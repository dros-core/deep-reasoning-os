# System Architecture

DROS is a single-process, event-driven autonomous trading system. All components run within one Python process on Apple Silicon (M4 Pro), with subprocess isolation for GPU and memory-intensive workloads.

---

## Process Architecture

```
execution-daemon (main process)
  |-- asyncio event loop
  |-- A0 EventLoop (symbol scheduler)
  |-- A1-A15 agent pipeline (per symbol, sequential)
  |-- Reconciler (position sync, 60s cycle)
  |-- RiskLayer (SL/TP management, continuous)
  |-- LearningBridge (Thompson/AWR updates)
  |-- MarketingEmitter (hook triggers, async isolated)
  |
  |-- [subprocess] BLS (Bayesian Learning System, 6GB gate)
  |-- [subprocess] MLX (Metal GPU, spawn context only)

continuous-daemon (companion process)
  |-- Macro regime monitoring
  |-- FGI polling
  |-- Symbol sentiment aggregation
  |-- Marketing scheduled events (SQLite outbox)
```

---

## Data Flow

### Entry Path

```
Market Data (Binance WebSocket + REST)
    |
    v
A1 DataFetcher
    |-- OHLCV (1m, 5m, 15m, 1h, 4h)
    |-- Funding rate
    |-- Open interest
    |-- Order book depth
    |
    v
A2 ReasoningCore
    |-- Regime classification (ranging/trending/volatile)
    |-- Volatility (Yang-Zhang preferred, ATR fallback)
    |-- Market structure (support/resistance, momentum)
    |
    v
A3 DirectionEngine
    |-- PSI (Proprietary Sentiment Index, 5-axis)
    |-- p_dir computation (ensemble: RSI, ADX, ER, momentum)
    |-- Calibration (SparseSafeCalibrator)
    |-- Dead Zone [0.38, 0.62] -> abstain
    |
    v
A4 CandidateGenerator (card creation)
    |-- spacing_dec (SpacingOracleSSOT, Yang-Zhang volatility)
    |-- N_total (grid count, >= 10)
    |-- leverage (Kelly-derived, FGI < 20 -> max 2.5% normal size)
    |-- PSI hard gate check
    |
    v
A5 EntryGate (5-check)
    |-- Check 1: Macro (PSI, FGI tempo)
    |-- Check 2: Direction (p_dir thresholds)
    |-- Check 3: Cost gate (ROI vs fees)
    |-- Check 4: Quality (quality_base_no_llm >= 85)
    |-- Check 5: Live alignment (>= 2 indicators)
    |
    v
A8 ROIDebias
    |-- CPCV backtest (purge gap >= 24h)
    |-- PBO validation (< 0.3 required)
    |-- Hierarchical Bayesian debias
    |-- net_roi_expected gate (< -5% blocked)
    |
    v
A9 CardSpec (sha256 signature)
    |
    v
PositionStateMachine.register(ENTRY_CANDIDATE)
    |
    v
A10 OrderManager (grid placement)
    |-- Per-level TP (reduce_only GTC)
    |-- Dynamic SL registration (Stack A/B/C)
    |
    v
PositionStateMachine.transition(MANAGED_POSITION)
```

### Exit Path

```
Exit trigger (SL / TP / manual)
    |
    v
A10 OrderManager
    |-- Cancel all active orders (exchange + PerLevelTPStore)
    |-- Place close order
    |
    v
PositionStateMachine.deregister()
    |
    v
A13 LearningBridge
    |-- net_roi calculation (fees + funding deducted)
    |-- Thompson update (regime-aware, 30-min throttle)
    |-- AWR reward signal (replay buffer)
    |
    v
roi_snapshots table (SQLite, not roi_events)
```

---

## Storage Architecture

| Store | Format | Purpose |
|-------|--------|---------|
| DaemonCheckpoint | JSON | Position state, slot budget, _non_card_orphan_symbols, SL mode |
| roi_history.db | SQLite | ROI snapshots (INVARIANT-LEARNING-02) |
| roi_events | SQLite | Event log with data_source column (not unrealized PnL) |
| learning_store.db | SQLite WAL | Learning state SSOT |
| scheduled_marketing_events | SQLite | ACTION_PROOF delay queue (restart-safe) |
| PerLevelTPStore | SQLite | Active PLT orders |
| WatchRegistry | SQLite | Pre-alert observation records |
| CardPositionRegistry | SQLite | Card-to-position mapping |

---

## Binance API Integration

- **Mode**: USDT-M Futures, hedge mode (positionSide required)
- **Auth**: API key + secret via environment variable
- **Rate limit**: 800/1200 weight (66% ceiling)
- **Concurrency**: max 5 simultaneous requests
- **Circuit breaker**: 429/418 -> 5-minute exponential backoff
- **closePosition**: cannot combine with quantity/reduceOnly (API constraint)

---

## Memory Architecture (M4 Pro Optimization)

BLS (Bayesian Learning System) runs in subprocess:
- Lightweight mode: RSS < 2.5GB
- Full mode: RSS < 6.0GB
- OOM gate uses RSS, not mem.percent
- Subprocess reclaimed after completion (prevents RSS accumulation)

MLX (Apple Silicon GPU):
- Always in subprocess with spawn context (fork banned)
- warmup() deferred to asyncio background task
- _gpu_lock required for mx.eval() calls
- OPENSSL_CONF=/dev/null in plist (prevents EVP deadlock)
- OMP_NUM_THREADS=1 (prevents LightGBM+ChromaDB SIGSEGV)

---

## Feature Flag System

100+ feature flags managed via:
1. `config/feature_flags.json` (state: "shadow", "active", "disabled")
2. Environment variable override (explicit 0 to disable shadow flags)
3. plist EnvironmentVariables (process-level defaults)

Flag states:
- `active` (1): fully enabled
- `shadow`: would-have logging only, no capital impact
- `disabled` (0): off

Changing flag state requires plist `launchctl unload + load` (stop/start insufficient for env var changes).

---

## Daemon Startup Sequence

```
1. plist EnvironmentVariables loaded
2. Python import preload (pandas, numpy -- before daemon import)
3. ExecutionDaemon.__init__
   |-- DaemonCheckpoint restore
   |-- PositionStateMachine restore
   |-- SharedRateLimitBudget init
   |-- BootstrapActionBucket restore
4. asyncio.create_task(_warmup_mlx_background)  [deferred, non-blocking]
5. IntelligenceBridge wire
6. Reconciler first pass (detect EXTERNAL_ORPHAN)
7. STARTUP_SLOT_EXPANSION_DELAY_S (300s) -- no new entries
8. Normal operation loop
```

---

## Marketing Architecture

```
Trading Events
    |
    v
Hook Points (8 hooks, lazy import + try/except)
    |
    v
event_normalizer.normalize_event()   [typed schema, no raw payload]
    |
    v
LLM Broker                           [if copy generation needed]
    |
    v
post_cleaner + hard_block_check      [mandatory pipeline]
    |
    v
Channel Formatters
    |-- x_formatter (proof card, <= 220 chars, 0 jargon)
    |-- public_formatter (Surface A + B, 1 emoji)
    |-- private_formatter (4-line operator memo, 0 emoji)
    |
    v
Marketing Outbox (SQLite)
    |-- immediate: WATCH, CONFIRMED, KILL_SWITCH, REGIME_SHIFT
    |-- delayed (30m): ACTION_PROOF (SQLite scheduled_marketing_events)
    |-- weekly (Mon 09:00 KST): WEEKLY_TRANSPARENCY
    |
    v
Channel Delivery (Telegram Bot API, X API)
```

---

*Architecture documentation for DROS v12.4b | 2026-03-14*
