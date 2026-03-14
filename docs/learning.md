# Learning Pipeline

DROS uses offline reinforcement learning with Bayesian methods for adaptive parameter optimization. All learning runs on historical realized outcomes — no simulation data enters production models.

---

## Architecture Overview

```
Realized Outcomes (net_roi, fill_rate, regime)
    |
    v
roi_snapshots (SQLite)          [not roi_events — INVARIANT-LEARNING-11]
    |
    v
Thompson Sampling               [regime-aware Beta updates]
    |
    v
AWR Grid Agent                  [off-policy, replay buffer 10K]
    |
    v
A8 ROIDebias                    [hierarchical Bayesian, CPCV+PBO]
    |
    v
SparseSafeCalibrator            [sample-count adaptive]
    |
    v
LearningCircuitBreaker          [gates production deployment]
    |
    v
learned_presets (via LearningArtifactRegistry)
```

---

## Thompson Sampling

Each symbol+regime combination maintains a Beta(alpha, beta) distribution:

- **Success**: net_roi > 0 (after fees and funding) -> alpha += 1
- **Failure**: net_roi <= 0 -> beta += 1
- **Throttle**: 30-minute minimum between updates per symbol (WARN_TS_FLOOD on excess)
- **Context**: regime_id required at update time (FAIL_REGIME_CONTEXT_MISSING if missing)
- **Warm start**: cluster-based prior (peer average -> card p_dir -> Beta(1,1) fallback)

**INVARIANT-LEARNING-04/05**: Success criterion is net_roi, not exit-type. A TP exit with net_roi <= 0 is a failure. Exit-type-based success triggers FAIL_TS_BINARY_SUCCESS.

---

## AWR (Advantage-Weighted Regression)

Off-policy batch RL replacing PPO:

| Property | Value |
|----------|-------|
| Mode | Off-policy |
| Buffer | Replay buffer, deque(maxlen=10000) |
| Eviction | Oldest auto-evicted (never cleared) |
| Weight explosion | max/mean > 100 -> skip training (FAIL_AWR_WEIGHT_EXPLOSION) |
| Feature flag | USE_AWR_GRID_AGENT=1 |

AWR is more sample-efficient than PPO for irregular market data. Off-policy training allows reuse of old observations without environment re-sampling.

---

## A8 ROIDebias

### Hierarchical Bayesian Debias

Three-level hierarchy (FAIL_A8_DEBIAS_FLAT if flat):
```
symbol level (most specific)
    |
    v
cluster level (peer group)
    |
    v
global level (system-wide prior)
```

Flat (symbol-only) debias banned. Hierarchical structure prevents overfitting to sparse symbol history.

### CPCV+PBO Validation

- **CPCV**: Combinatorial Purge Cross-Validation
  - Purge gap: >= 24h between train and test (FAIL_CPCV_NO_PURGE)
  - Embargo: additional buffer after purge
- **PBO**: Probability of Backtest Overfitting
  - Threshold: < 0.3 required (FAIL_PBO_OVERFIT if >= 0.3)
  - PBO >= 0.3 indicates likely overfit — learning blocked

### OPE (Off-Policy Evaluation)

Required for every learning report (FAIL_OPE_INCOMPLETE if missing):
- DR (Doubly Robust) estimator
- Clopper-Pearson confidence interval

---

## SparseSafeCalibrator

Sample-count adaptive calibration (INVARIANT-LEARNING-16):

| Sample Count | Method |
|-------------|--------|
| < 50 | Temperature Scaling |
| 50 - 500 | Beta Calibration |
| > 500 | IsotonicRegression |

IsotonicRegression on < 50 samples triggers FAIL_ISOTONIC_SPARSE.

Calibration output must be in [0, 1]. Out-of-range values are clamped (FAIL_CALIBRATION_OUT_OF_RANGE if not clamped).

Calibrator state persisted after every calibration run (FAIL_CALIBRATOR_STATE_LOST if not saved).

---

## BLS (Bayesian Learning System)

Memory-gated subprocess:

| Mode | RSS Gate |
|------|----------|
| Lightweight | < 2.5 GB |
| Full | < 6.0 GB |

BLS runs in subprocess for memory isolation (RSS reclaimed after completion). `mem.percent` is banned as gate metric -- RSS used directly (FAIL_BLS_OOM).

---

## Empirical Bayes Shrinkage

EB shrinkage kappa estimated from marginal likelihood (FAIL_EB_HARDCODED_KAPPA if hardcoded):

```
kappa estimated via marginal likelihood maximization
  not: kappa = 0.5  (hardcoded -- banned)
```

---

## Data Source Tracking

Every roi_events record requires data_source column (FAIL_ROI_NO_DATA_SOURCE if NULL):

| Source | Weight |
|--------|--------|
| live | 1.0 |
| backtest | 0.5 |
| synthetic | 0.1 |

Warm-start bootstrap synthetic data must be tagged data_source="synthetic" (FAIL_BOOTSTRAP_NO_SOURCE_TAG if missing).

HTM (Hierarchical Temporal Memory) updates require data_source -- synthetic weighted at 0.1 (FAIL_DATA_SOURCE_CONTAMINATION).

Propensity scores recorded at selection time, never recomputed post-hoc (FAIL_PROPENSITY_POST_HOC).

---

## Learning Circuit Breaker

LearningCircuitBreaker gates production deployment of learned parameters:

- Trips on: net_roi degradation, PBO failure, calibration drift
- Tripped CB blocks: LearningArtifactRegistry.get_path() for production writes
- FAIL_LEARNING_CB_TRIPPED: production write attempted with tripped CB

---

## Storage

| Store | Location | Invariant |
|-------|----------|-----------|
| ROI snapshots | roi_history.db, roi_snapshots table | INVARIANT-LEARNING-11 |
| ROI events | roi_history.db, roi_events table | data_source required |
| Learning state | execution/state/learning_store.db (WAL) | SSOT |
| Learned presets | via LearningArtifactRegistry.get_path() only | INVARIANT-LEARNING-01 |
| Calibrator state | saved after every calibration | INVARIANT-LEARNING-23 |

**INVARIANT-LEARNING-01**: learned_presets path must come from LearningArtifactRegistry.get_path("learned_presets"). Any hardcoded path triggers FAIL_PRESET_PATH_MISMATCH.

**INVARIANT-LEARNING-02**: roi_history persisted to SQLite. In-memory dict as sole store triggers FAIL_ROI_HISTORY_LOST.

---

## Full Invariant List

| Code | Rule |
|------|------|
| INVARIANT-LEARNING-01 | learned_presets path via LearningArtifactRegistry only |
| INVARIANT-LEARNING-02 | roi_history SQLite-persisted |
| INVARIANT-LEARNING-03 | Thompson update requires regime_id context |
| INVARIANT-LEARNING-04/05 | TS success = net_roi positive (not exit-type) |
| INVARIANT-LEARNING-06 | OPE: DR estimator + Clopper-Pearson CI |
| INVARIANT-LEARNING-08 | Calibration output clamped to [0,1] |
| INVARIANT-LEARNING-10 | A8 debias: hierarchical Bayesian (symbol/cluster/global) |
| INVARIANT-LEARNING-11 | Unrealized PnL not recorded in roi_events |
| INVARIANT-LEARNING-13 | A8 backtest: CPCV+PBO, PBO < 0.3 |
| INVARIANT-LEARNING-14 | HTM update: data_source explicit, synthetic weight 0.1 |
| INVARIANT-LEARNING-15 | Propensity score at selection-time, no post-hoc |
| INVARIANT-LEARNING-16 | SparseSafeCalibrator: method by sample count |
| INVARIANT-LEARNING-18 | EB kappa from marginal likelihood, not hardcoded |
| INVARIANT-LEARNING-19 | CPCV purge gap >= 24h |
| INVARIANT-LEARNING-22 | roi_events: data_source column, no NULL |
| INVARIANT-LEARNING-23 | Calibrator state saved after every calibration |
| INVARIANT-LEARNING-26 | warm_start_bootstrap synthetic tagged data_source="synthetic" |

---

*Learning documentation for DROS v12.4b | 2026-03-14*
