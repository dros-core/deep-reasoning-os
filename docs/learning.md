# 🧠 Learning — Deep Reasoning OS (DROS)

> AWR + Thompson Sampling dual online learning pipeline

---

## Overview

DROS learns continuously from every trade outcome.  
Two learning layers operate in parallel, each solving a different problem:

| Layer | Algorithm | Problem | Update Frequency |
|-------|-----------|---------|-----------------|
| **Layer 1** | Thompson Sampling Bandit | Which preset for this symbol? | Per rotation (~30 min) |
| **Layer 2** | AWR Agent (MDP) | How to adjust spacing/N/leverage? | Per heartbeat (5 min dense) |

---

## Layer 1 — Thompson Sampling Bandit

**Goal:** Select the best preset configuration for each symbol at rotation time.

```python
# Bayesian Beta posterior per preset
alpha[preset] += 1  if net_roi > TS_SUCCESS_THRESHOLD
beta[preset]  += 1  if net_roi ≤ TS_SUCCESS_THRESHOLD

# Per-symbol throttle: 30-minute cooldown
# Success metric: net_roi (fees + funding deducted — not exit-type alone)
```

**Key rules:**
- Update requires `regime_id` context (`FAIL_REGIME_CONTEXT_MISSING`)
- Success = `net_roi > 0` (after fees and funding) — binary exit-type forbidden (`FAIL_TS_BINARY_SUCCESS`)
- Thompson skew alpha/beta > 100:1 → LearningCircuitBreaker trips

---

## Layer 2 — AWR Agent (Advantage-Weighted Regression)

**Goal:** Multi-step MDP for continuous grid parameter adaptation.

```
State:   [spacing_dec, N_total, leverage, regime, volatility, fill_rate]
Action:  [spacing_adjust, N_adjust, leverage_adjust, direction_bias]
Reward:  dense heartbeat reward (every 5 min) + terminal episode reward

AWR update:
  weight = exp(advantage / β)  (β = AWR_BETA, default 0.3)
  max weight cap: AWR_MAX_WEIGHT (default 10.0)
  weight explosion (max/mean > 100) → skip training → FAIL_AWR_WEIGHT_EXPLOSION
```

**Replay buffer:** `deque(maxlen=10,000)` — oldest auto-evicted, never cleared manually.  
**Training trigger:** buffer ≥ 128 samples AND reward_std > 0.

---

## Data Flow

```
daemon.py (25s heartbeat)
  ├── learning_store.write_heartbeat()   [5-min throttle]
  ├── learning_store.write_fill()        [on every fill]
  └── shadow_simulator._log_fill()       [fallback]
         ↓
autonomous_learning_daemon (30-min cycle)
  ├── compute_dense_reward()  → AWR buffer accumulates
  ├── Thompson posterior update (30-min throttle per symbol)
  └── awr_agent._train()  [buffer ≥ 128 + reward_std > 0]
         ↓
BLS subprocess (6-hour interval)
  └── Bayesian learning → OS fully reclaims RSS on exit
         ↓
LearningCircuitBreaker validation
  ├── Thompson skew        (alpha/beta > 100:1)
  ├── AWR weight explosion (max/mean > 100)
  ├── Write failures       (3 consecutive)
  └── Cursor staleness     (>2h no advance)
```

---

## Backtest Validation — CPCV + PBO

All strategy candidates are validated before production deployment.

```python
cv = CombinatoricalPurgedCV(
    n_splits   = 5,
    purge_gap  = 24,    # 24h — prevents information leakage
    embargo_pct = 0.01  # additional safety margin
)

# PBO ≥ 0.3 → FAIL_PBO_OVERFIT → learning blocked
```

**Required metrics:** Balanced Accuracy · MCC · Brier Score

---

## SparseSafeCalibrator

Direction probability calibration adapts to available sample size:

| Samples | Method | Reason |
|---------|--------|--------|
| < 50 | Temperature Scaling | Prevents Isotonic overfitting |
| 50–500 | Beta Calibration | Optimal for mid-range |
| > 500 | IsotonicRegression | Sufficient data guaranteed |

- State persisted after every calibration: `save_state()` required (`FAIL_CALIBRATOR_STATE_LOST`)
- Empirical Bayes kappa: marginal likelihood estimation — hardcoding forbidden (`FAIL_EB_HARDCODED_KAPPA`)

---

## BLS — Bayesian Learning Subprocess

**Problem:** Python GC does not return memory to OS after large allocations.  
**Solution:** BLS runs as a subprocess → OS fully reclaims all RSS on exit.

```
Memory available ≥ 6.0 GB → Full BLS (Bayesian + presets)
Memory available ≥ 3.0 GB → Lightweight BLS (skip Bayesian + presets)
Memory available < 3.0 GB → Skip entirely

Note: mem.percent is unreliable on macOS (inactive/cached inflate %)
      Use mem.available (actual allocatable bytes) — always.
```

---

## Learning Store — SQLite SSOT

```
Location: execution/state/learning_store.db
Tables:   heartbeats · fills · roi_snapshots · roi_events · game_events

ROI snapshot rules:
  → roi_snapshots table only (FAIL_ROI_SNAPSHOT_WRONG_TABLE)
  → unrealized ROI must NOT be written to roi_events
  → data_source column required, NULL forbidden (FAIL_ROI_NO_DATA_SOURCE)

Cursor-based tailing — staleness > 2h → LearningCircuitBreaker
```

---

## Hierarchical ROI Debias — A8 (INVARIANT-LEARNING-10)

A8 applies hierarchical Bayesian debiasing before using ROI for learning:

```
symbol → cluster → global

Flat debiasing forbidden → FAIL_A8_DEBIAS_FLAT
```

---

## Hot-Reload Safety

Learning state survives daemon restarts without `importlib.reload()`:

```
Forbidden: importlib.reload()  → FAIL_IMPORT_RELOAD
           (CPython #126548: stale refs, thread-unsafe)

Required:
  Atomic writes:    write → fsync → os.replace()
  SharedMemory:     track=False (cross-process persistence)
  Singleton reset:  module global → None → lazy re-init
  env var:          module-level os.environ.get() forbidden → FAIL_COLD_ENV_VAR
  Timestamps:       ALL fields adjusted on restore (prevent negative elapsed_h)
```

---

## KPI Targets

| KPI | Target | Current |
|-----|--------|---------|
| Cost Gate pass rate | ≥ 90% | 95% ✅ |
| Negative ROI cards | 0% | 0% ✅ |
| Live-Backtest gap | ≤ 10% | Measuring |
| Card generation rate | ≥ 80% | Measuring |

---

*→ See [Architecture](./architecture.md) · [Agents](./agents.md) · [Evolution Lab](./evolution-lab.md)*
