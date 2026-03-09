# 🔬 Evolution Lab — Deep Reasoning OS (DROS)

> AI Evolution Lab v3 — 13-module self-evolution system

---

## Overview

The AI Evolution Lab is DROS's controlled strategy evolution framework.  
New strategies are never deployed directly to production.  
Every idea must survive a **4-layer scientific validation pipeline** before going live.

> *"Shadow first. Always."*

**Implementation completed:** 2026-03-09  
**Active modules:** 7 Production · 6 Shadow  
**Architecture pattern:** EnhancerBus (Strangler Fig)

---

## 7-Kernel Orchestra

| Kernel | Role | Key Module |
|--------|------|-----------|
| **Perception** | Market state detection | `entropy_regime.py` · `ising_phase.py` |
| **Forecast** | Direction prediction | `mlx_direction_predictor.py` (Apple MLX) |
| **Microstructure** | Liquidity sensing | `cfc_sensor.py` (MIT CfC neural network) |
| **Risk** | Black Swan detection | `black_swan_ensemble.py` (2/4 vote) |
| **Execution** | Optimal execution | `mcts_engine.py` · `inverse_reasoning.py` |
| **Research** | Hypothesis testing | `popper_engine.py` (E-value based) |
| **Evolution** | Strategy evolution | `qd_optimizer.py` (pyribs MAP-Elites) |

---

## EnhancerBus — Plugin Architecture

```
Location: services/evolution/enhancer_bus.py
Pattern:  Strangler Fig (gradual replacement of existing logic)

Rule:     DecisionPacket main fields → READ-ONLY
          Only extra_context may be written
          Violation → FAIL_EVOL_BUS_WRITE_VIOLATION

Optimization: _sorted_cache for 8-enhancer priority ordering
              Invalidated on register() / unregister()
```

All 13 evolution modules plug into the EnhancerBus without modifying core decision logic.

---

## 4-Layer Validation Pipeline

Every new strategy must pass all 4 layers before production:

```
Layer 1: Research Lab
  POPPER E-value hypothesis testing
  E-value ≥ 1/α → reject hypothesis
  Pre-registration required — post-hoc hypotheses forbidden
  → FAIL_EVOL_POST_HOC_HYPOTHESIS

Layer 2: Counterfactual Lab
  OPE Capped SNIPS (cap=10.0)
  Bootstrap CI with n ≥ 1,000 samples required
  → FAIL_EVOL_OPE_NO_CI

Layer 3: Digital Twin
  MirrorEngine computes EPE / FRE / LPE simultaneously
  All 3 parity metrics must be < 5%
  → FAIL_EVOL_PARITY_INCOMPLETE

Layer 4: Shadow → Canary → Production
  Minimum 7 days shadow
  SPA test p < 0.01 required for promotion
  Minimum 100 events before graduation
  10% Canary traffic before 100% production
```

---

## Active Modules

### Production (Live)

| Module | Technology | Role |
|--------|-----------|------|
| **ACI Risk** | Adaptive Conformal Inference | Risk boundary calibration |
| **EventStore** | SQLite WAL append-only | Immutable event log |
| **Digital Twin** | MirrorEngine EPE/FRE/LPE | Shadow-production parity |
| **Counterfactual Lab** | OPE Capped SNIPS | Policy evaluation |
| **Black Swan** | ADWIN+CUSUM+BOCPD+Hawkes | 2/4 vote ensemble detection |
| **Alpha Foundry** | pyribs MAP-Elites | Genome quality-diversity evolution |
| **OODA Loop** | Boyd OODA cycle | Offline strategy adaptation |

### Shadow (Validation in progress)

| Module | Technology | Role |
|--------|-----------|------|
| **Entropy Regime** | Kozachenko-Leonenko estimator | Market entropy regime detection |
| **Ising Phase** | Ising magnetization model | Phase transition detection |
| **CfC Sensor** | MIT Closed-form Continuous-time NN | Microstructure anomaly detection |
| **MCTS Reasoning** | UCB1 + OU-process rollout | Grid parameter optimization |
| **ACO Selector** | Ant Colony pheromone | Symbol ranking |
| **Lotka-Volterra** | Prey-predator dynamics | Regime detection |

---

## Alpha Foundry — MAP-Elites Genome Evolution

```python
# Strategy genome: grid parameters as evolvable chromosome
genome = StrategyGenome(
    spacing_dec  = 0.0025,
    N_total      = 16,
    leverage     = 5,
    grid_type    = "geometric",
    direction    = "long"
)

# Quality-Diversity optimization via pyribs
# Explores diverse high-performing strategies
# Memory leak prevention: gc.collect() per iteration (INVARIANT-EVOL-09)
# Mutation rate: plateau-adaptive, never hardcoded (INVARIANT-EVOL-07)
```

---

## OODA Loop — Offline Strategy Adaptation

Boyd's OODA cycle applied to strategy evolution:

```
Observe  → Collect market intelligence (async, all times)
Orient   → Regime classification + pattern recognition
Decide   → Strategy selection (OFFLINE ONLY: 03:00–09:00 KST)
Act      → Deploy to shadow for validation (OFFLINE ONLY)

INVARIANT-EVOL-12:
  Decide and Act phases: 03:00–09:00 KST only
  Live-hours execution → FAIL_EVOL_OODA_LIVE_DECIDE
```

---

## Black Swan Ensemble — 2/4 Vote

Four independent detectors must reach 2/4 consensus before a black swan alert:

| Detector | Method |
|----------|--------|
| ADWIN | Adaptive Windowing (distribution shift) |
| CUSUM | Cumulative Sum (sequential change detection) |
| BOCPD | Bayesian Online Changepoint Detection |
| Hawkes | Self-exciting point process (event clustering) |

```
Single detector alert → ignored
2/4 vote → BLACK_SWAN event registered
3/4 vote → elevated alert
4/4 vote → maximum response

INVARIANT-EVOL-08: Single detector sufficient → FAIL_EVOL_SINGLE_DETECTOR
```

---

## EventStore — Immutable Event Log

```
Implementation: SQLite WAL mode
Rule:           Append-only — no updates, no deletes
                Only prune() may remove old records
                Violation → FAIL_EVOL_EVENT_MUTATION (P0)

INVARIANT-EVOL-05: WAL mode required → FAIL_EVOL_NO_WAL
```

---

## Complete INVARIANT Reference

| INVARIANT | Rule | Fail Code |
|-----------|------|-----------|
| EVOL-01 | EnhancerBus: extra_context write only | `FAIL_EVOL_BUS_WRITE_VIOLATION` |
| EVOL-02 | Hypothesis pre-registration required | `FAIL_EVOL_POST_HOC_HYPOTHESIS` |
| EVOL-03 | Evolved strategies start in shadow | `FAIL_EVOL_DIRECT_DEPLOY` |
| EVOL-04 | ACI gamma ∈ (0, 0.1] | `FAIL_EVOL_ACI_GAMMA_RANGE` |
| EVOL-05 | EventStore: WAL mode required | `FAIL_EVOL_NO_WAL` |
| EVOL-06 | Digital Twin: EPE/FRE/LPE simultaneous | `FAIL_EVOL_PARITY_INCOMPLETE` |
| EVOL-07 | Mutation rate: plateau-adaptive only | `FAIL_EVOL_FIXED_MUTATION` |
| EVOL-08 | Black Swan: 2/4 vote required | `FAIL_EVOL_SINGLE_DETECTOR` |
| EVOL-09 | Alpha Foundry: gc.collect() per iter | `FAIL_EVOL_MEMORY_LEAK` |
| EVOL-10 | POPPER: E-value ≥ 1/α → reject | `FAIL_EVOL_EVALUE_THRESHOLD` |
| EVOL-11 | CfC: Z-score normalization required | `FAIL_EVOL_CFC_NO_NORM` |
| EVOL-12 | OODA Decide/Act: 03:00–09:00 KST only | `FAIL_EVOL_OODA_LIVE_DECIDE` |
| EVOL-14 | OPE: Bootstrap CI n ≥ 1,000 | `FAIL_EVOL_OPE_NO_CI` |
| EVOL-15 | MCTS: UCB1 + OU-process rollout | `FAIL_EVOL_MCTS_INVALID_ROLLOUT` |
| EVOL-16 | EventStore: append-only, prune only | `FAIL_EVOL_EVENT_MUTATION` |

---

## Graduation Gate — Shadow to Production

```
Minimum requirements for production promotion:

  ✓ 100+ events recorded in shadow
  ✓ 7+ days shadow runtime
  ✓ EPE < 5% (execution parity error)
  ✓ SPA test p < 0.01
  ✓ ROI improvement vs baseline
  ✓ 10% Canary validation passed
```

---

## Academic Foundation

| Paper | Application |
|-------|------------|
| Grünwald et al. — *E-values* (2023) | POPPER hypothesis testing |
| Adams & MacKay — *BOCPD* (2007) | Black Swan ensemble |
| Mouret & Clune — *MAP-Elites* (2015) | Alpha Foundry |
| Hasani et al. — *CfC Networks* (2022) | Microstructure sensor |
| Rackauckas et al. — *Universal DEs* (2020) | Dynamics modeling |

---

*→ See [Architecture](./architecture.md) · [Learning](./learning.md) · [Safety](./safety.md)*
