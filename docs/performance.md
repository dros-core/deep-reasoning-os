# 📊 System Operational Metrics

> Operational characteristics of DROS v11.20 — **not** financial performance claims.
>
> This page documents how the system behaves technically. No backtested returns, no live PnL figures.

---

## Execution Layer

| Metric | Value |
| :--- | :--- |
| **Core daemon size** | ~11,000 lines (10,973 as of v11.20) |
| **Tick processing** | asyncio event loop, non-blocking |
| **Checkpoint write** | Atomic: write → fsync → rename |
| **Microstructure latency** | VPIN+OFI via SharedMemory, lock-free, <0.01ms |
| **Order jitter** | ±15% randomization + Poisson timing |
| **Slot management** | Dynamic allocation via AOSM v2 |

---

## Safety Gate

| Metric | Value |
| :--- | :--- |
| **Gates per entry attempt** | 7 (all must pass) |
| **Gate enforcement** | Deterministic (not probabilistic) |
| **P0 invariant violations** | Immediate system halt |
| **Card freshness TTL** | 5,400 seconds (90 minutes) |
| **Liquidation probability check** | Per-entry, real-time |

---

## Learning Pipeline

| Metric | Value |
| :--- | :--- |
| **AWR update frequency** | Per heartbeat |
| **Thompson Sampling update** | Per rotation cycle, 30-min throttle per symbol |
| **BLS isolation** | Separate subprocess (RSS fully reclaimed after run) |
| **Memory gate (BLS)** | 2.5 GB lightweight / 6.0 GB full mode |
| **Replay buffer** | deque(maxlen=10,000), oldest auto-evicted |
| **Calibrator selection** | <50 samples: Temperature Scaling / 50–500: Beta / >500: Isotonic |
| **PBO overfit threshold** | ≥ 0.3 → training blocked |

---

## Evolution Lab

| Metric | Value |
| :--- | :--- |
| **Active production modules** | 7 (ACI Risk, EventStore, Digital Twin, Counterfactual Lab, Black Swan, Alpha Foundry, OODA) |
| **Shadow modules** | 8 (in testing) |
| **Shadow minimum duration** | 7 days |
| **Canary traffic** | 10% |
| **Canary significance test** | SPA (p < 0.01) |
| **Black Swan vote threshold** | 2/4 ensemble detectors |
| **OODA offline window** | 03:00–09:00 KST (Decide/Act phase only) |

---

## Codebase

| Metric | Value |
| :--- | :--- |
| **Total Python lines** | ~1,090,000 |
| **Test files** | 442 |
| **Service modules** | 100 |
| **Contract definitions** | 43 |
| **Evolution modules** | 53 |
| **Named invariant codes** | 40+ (P0 + P1 + WARN) |

---

## API Constraints

| Constraint | Value |
| :--- | :--- |
| **Weight limit target** | ≤ 800/1,200 (66% of limit) |
| **Max concurrent requests** | 5 |
| **Circuit breaker trigger** | HTTP 429 or 418 → 5-minute backoff |
| **Cache-first policy** | API direct calls minimized via in-memory cache |

---

## KPI Targets

These are internal operational targets — not financial return targets.

| KPI | Target |
| :--- | :--- |
| **Cost Gate pass rate** | ≥ 90% |
| **Negative-ROI card rate** | 0% |
| **Live-Backtest divergence** | ≤ 10% |
| **Card generation rate** | ≥ 80% |

---

> Nothing on this page constitutes financial advice or a guarantee of future system behavior.

*→ See [Architecture](./architecture.md) · [Safety](./safety.md) · [Learning](./learning.md)*