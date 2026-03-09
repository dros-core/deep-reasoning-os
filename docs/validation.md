# 🔬 Validation Standards

> DROS applies academic-grade validation at every stage — from strategy development through production deployment.

---

## Core Principle

> **"A strategy that cannot be validated is a strategy that cannot be trusted."**

DROS validation pipeline is designed to catch overfitting, data leakage, distributional shift,
and regime mismatch before any strategy reaches production capital.

---

## Backtesting Standard — CPCV + PBO

Standard backtesting suffers from a fundamental problem: strategies can be optimized to look good on historical data without any genuine predictive power. DROS addresses this with two controls.

### Combinatorial Purged Cross-Validation (CPCV)

Based on López de Prado (2018, 2020).

```
n_splits   = 5
purge_gap  = 24 hours   (prevents information leakage between train/test)
embargo_pct = 1%        (additional safety margin after purge)
```

The purge gap is critical: in financial time series, adjacent observations are correlated.
Without purging, information leaks from training into test sets — producing artificially optimistic results.

### Probability of Backtest Overfitting (PBO)

```
PBO ≥ 0.3 → FAIL_PBO_OVERFIT → learning blocked
```

PBO estimates the probability that the selected strategy configuration was chosen due to
overfitting rather than genuine predictive power. A threshold of 0.3 means: if there is
a 30% or greater chance the result is noise, the strategy is rejected.

This runs automatically. No human override.

---

## Calibration — SparseSafeCalibrator

Direction probability outputs must be calibrated — raw model scores are not probabilities.
DROS uses an adaptive calibration pipeline that selects method based on available sample size:

| Samples | Method | Rationale |
|---------|--------|-----------|
| < 50 | Temperature Scaling | Isotonic overfitting risk too high |
| 50–500 | Beta Calibration | Optimal bias-variance balance |
| > 500 | Isotonic Regression | Sufficient data for non-parametric fit |

Calibrator state is persisted after every calibration cycle (`FAIL_CALIBRATOR_STATE_LOST` if not saved).
Empirical Bayes kappa is estimated from marginal likelihood — hardcoded values are forbidden.

---

## Drift Detection — Black Swan Ensemble

Four independent drift detectors run in parallel. **A minimum of 2/4 must agree** before a Black Swan signal is raised.

| Detector | Algorithm | Detects |
|----------|-----------|---------|
| ADWIN | Adaptive Windowing | Gradual concept drift |
| CUSUM | Cumulative Sum Control | Abrupt change points |
| BOCPD | Bayesian Online Changepoint Detection | Probabilistic regime shifts |
| Hawkes | Self-exciting point process | Clustering of extreme events |

The 2/4 vote requirement prevents false positives from any single detector while maintaining sensitivity to genuine regime changes.

---

## Deployment Validation — Shadow → Canary → Production

No strategy reaches production without passing a staged deployment pipeline:

```
Stage 1: Research Lab
  → Hypothesis registered before testing (FAIL_EVOL_POST_HOC_HYPOTHESIS if violated)
  → POPPER E-value validation

Stage 2: Counterfactual Lab
  → Off-Policy Evaluation (OPE)
  → Doubly Robust estimator + Clopper-Pearson confidence intervals

Stage 3: Digital Twin
  → Mirror Engine parity check (EPE / FRE / LPE < 5% divergence)

Stage 4: Shadow Mode
  → Minimum 7 days of parallel execution
  → Zero production capital at risk

Stage 5: Canary (10% traffic)
  → Sequential Probability Ratio Test (SPA, p < 0.01)
  → Rollback on failure

Stage 6: Production
  → Full deployment after canary validation
```

Each stage has an automated gate. Human promotion is not required — but human override to skip stages is not permitted.

---

## Invariant Contract Enforcement

43 named invariant contracts enforce correctness properties throughout the system.
Violations raise named FAIL codes and halt the relevant subsystem immediately.

Key validation-related invariants:

| INVARIANT | Rule |
|-----------|------|
| `INVARIANT-LEARNING-13` | A8 backtest requires CPCV+PBO — PBO ≥ 0.3 blocks learning |
| `INVARIANT-LEARNING-16` | Isotonic Regression forbidden with < 50 samples |
| `INVARIANT-LEARNING-18` | Empirical Bayes kappa must be estimated, not hardcoded |
| `INVARIANT-LEARNING-19` | CPCV purge gap ≥ 24h — shorter gaps are invalid |
| `INVARIANT-EVOL-02` | Hypotheses registered before testing — post-hoc forbidden |
| `INVARIANT-EVOL-03` | Evolved strategies start at shadow stage — direct deployment forbidden |
| `INVARIANT-EVOL-08` | Black Swan requires 2/4 ensemble vote |

---

## Academic Foundation

DROS validation methodology is grounded in peer-reviewed research:

| Reference | Application |
|-----------|-------------|
| López de Prado (2018) — *Advances in Financial Machine Learning* | CPCV, Triple-Barrier Labels, Feature Importance |
| López de Prado (2020) — *Machine Learning for Asset Managers* | PBO, walk-forward validation |
| Adams & MacKay (2007) — *Bayesian Online Changepoint Detection* | BOCPD drift detector |
| Easley, López de Prado, O'Hara (2012) — *VPIN: Flow Toxicity* | Microstructure toxicity detection |
| Albers et al. (2025) — *Adverse Selection in Binance BTC Perpetuals* | Exchange-specific adverse flow modeling |
| Grünwald et al. (2023) — *E-values: Calibration, Combining and Beyond* | POPPER E-value hypothesis testing |
| Mouret & Clune (2015) — *Illuminating search spaces by mapping elites* | MAP-Elites quality-diversity search |
| Hasani et al. (2022) — *Closed-form continuous-time neural networks* | CfC microstructure anomaly detection |

---

## What We Do Not Validate

In the interest of transparency:

- **Live PnL is not published** — production performance figures are not included in this repository
- **Backtested returns are not claimed** — past validation results do not predict future performance
- **Market regime coverage is partial** — DROS is calibrated on observed regimes; novel regimes may not be covered

---

*→ See [Learning](./learning.md) · [Safety](./safety.md) · [Evolution Lab](./evolution-lab.md)*
