# Leviathan v4.0 — AI Evolution Lab

Formerly "AI Evolution Lab v3." Renamed to Leviathan v4.0 to reflect its matured 5-Brain Architecture and dual-ring production/shadow structure.

---

## Architecture: 5-Brain System

Leviathan operates through five specialized neural subsystems, each responsible for a distinct cognitive function:

| Brain | Function | Key Components |
|-------|----------|---------------|
| Perception | Market state classification | PSI, FGI, regime detector, symbol sentiment 5-axis |
| Forecast | Directional probability | Direction engine, Thompson prior, calibration |
| Microstructure | Order flow analysis | VPIN, toxicity detection, fill rate monitoring |
| Risk | Position sizing and safety | Tail risk, Dynamic SL, liq_buffer floor |
| Execution | Order management | Grid engine, PLT, Position State Machine |

---

## Dual-Ring Architecture

### Ring 1: Production
Strategies that have completed the full graduation pipeline. Operate on live capital with full enforcement of all invariants and fail codes.

### Ring 2: Shadow
All new strategies and features start here. Shadow mode means:
- No capital at risk
- Would-have logs only
- Full metric tracking (EPE, FRE, LPE parity with production)
- Cannot be promoted without SPA gate

**Promotion path**: Shadow -> SPA (p<0.01) -> Canary -> Production

Minimum requirements for promotion:
- 100+ events
- 7 days shadow duration
- EPE < 5% divergence from Digital Twin
- ROI improvement vs baseline
- SPA sequential test: p < 0.01

---

## PNL Efficiency Engine (8-Phase)

The PNL Efficiency Engine evaluates strategy quality through eight sequential phases before any capital allocation:

1. **Signal Quality** - Directional accuracy, calibration score
2. **Execution Quality** - Fill rate, slippage vs model
3. **Regime Fit** - Performance per regime (ranging/trending/volatile)
4. **Cost Gate** - Gross ROI vs fee + slippage (>=90% pass rate)
5. **Tail Risk** - Max drawdown, tail_risk score
6. **Lead Time** - Signal-to-execution latency
7. **Stability** - Cross-validation stability (CPCV + PBO < 0.3)
8. **Net ROI** - Post-all-cost expected return (must be positive)

---

## AlphaFoundry QD MAP-Elites

Quality-Diversity optimization using pyribs MAP-Elites for genome evolution:

- **Genome**: Grid parameters (spacing, N_total, leverage, regime weights)
- **Quality**: Net ROI after CPCV+PBO validation
- **Diversity**: Coverage across regime x symbol_tier grid
- **Mutation**: Adaptive rate (adjusts on plateau, INVARIANT-EVOL-07)
- **Clock**: AlphaFoundryClock 72h control cycle (INVARIANT-EVOL-19)

Direct `qd_optimizer.optimize()` calls are banned. Must route through `AlphaFoundryClock.run()`.

---

## OODA Loop (Offline Strategy Improvement)

Boyd OODA loop runs in the offline window (03:00-09:00 KST) only.

| Phase | Timing | Action |
|-------|--------|--------|
| Observe | Continuous | Data collection, event logging |
| Orient | Offline | Regime drift detection (on_regime_drift()) |
| Decide | Offline only | Strategy selection (decide_and_act()) |
| Act | Offline only | Parameter update, shadow registration |

**INVARIANT-EVOL-12**: Decide/Act phases offline-only. Live execution during market hours is forbidden (FAIL_EVOL_OODA_LIVE_DECIDE).

**INVARIANT-EVOL-18**: `on_regime_drift()` sets flag only. Execution deferred to decide_and_act() offline window.

---

## Black Swan Ensemble

Four independent detectors, 2/4 vote required (INVARIANT-EVOL-08):

1. **ADWIN** - Adaptive Windowing change detection
2. **CUSUM** - Cumulative sum control chart
3. **BOCPD** - Bayesian Online Changepoint Detection
4. **Hawkes** - Hawkes process event clustering

Single detector firing is insufficient. Reduces false positive rate significantly.

Feature flag: `USE_EVOL_BLACK_SWAN` (currently shadow)

---

## Evidence-Based Graduation Gate v3

Three sequential evidence requirements before production graduation:

1. **ESS (Effective Sample Size)**: AR(1) autocorrelation-adjusted. Prevents pseudoreplication from correlated market events.

2. **Coverage**: Strategy must perform across regime diversity grid. Cannot graduate on single-regime performance.

3. **Truncated mSPRT**: Sequential probability ratio test with truncation. Maintains type-I error control under optional stopping.

Only when all three pass with p < 0.01 does the SPA gate open for canary deployment.

---

## MLX GPU Subsystem

Apple Silicon Metal GPU integration for indicator computation:

- **Isolation**: MLX runs in subprocess (spawn context only, fork banned: FAIL_EVOL_FORK_MLX)
- **Lock**: mx.eval() calls require _gpu_lock ownership (INVARIANT-EVOL-20)
- **Warmup**: Deferred to asyncio background task after daemon init (prevents import-lock deadlock)
- **Fitness proxy**: MLX batch evaluation for genome fitness in AlphaFoundry (shadow)

Feature flag: `USE_EVOL_MLX_SUBPROCESS` (active), `USE_EVOL_MLX_FITNESS_PROXY` (shadow)

---

## Digital Twin Parity

The Digital Twin mirrors production execution for shadow validation:

| Metric | Description | Threshold |
|--------|-------------|-----------|
| EPE | Execution Parity Error | < 5% |
| FRE | Fill Rate Error | < 10% |
| LPE | Latency Parity Error | < 20% |

Shadow strategies failing parity thresholds cannot graduate regardless of ROI performance.

---

## Feature Flags

All Leviathan features default to 0 (off). Enable via environment variable or feature_flags.json:

| Flag | Description | Status |
|------|-------------|--------|
| USE_EVOL_BLACK_SWAN | 4-detector anomaly ensemble | shadow |
| USE_EVOL_ALPHA_FOUNDRY | QD MAP-Elites genome evolution | shadow |
| USE_EVOL_OODA_LOOP | Boyd OODA offline loop | shadow |
| USE_EVOL_EVIDENCE_GRADUATION | ESS+coverage+mSPRT gate | active |
| USE_EVOL_MLX_SUBPROCESS | MLX Metal GPU isolation | active |
| USE_EVOL_MLX_FITNESS_PROXY | GPU-accelerated fitness evaluation | shadow |
| USE_EVOL_MCTS_REASONING | Monte Carlo Tree Search grid optimizer | shadow |
| USE_EVOL_DIGITAL_TWIN | Mirror Engine parity tracking | shadow |
| USE_EVOL_COUNTERFACTUAL_LAB | OPE policy evaluation | shadow |
| USE_EVOL_ENTROPY_REGIME | Kozachenko-Leonenko entropy detector | shadow |

---

## Invariants (Critical)

| Code | Rule |
|------|------|
| INVARIANT-EVOL-01 | EnhancerBus writes to DecisionPacket.extra_context only (read-only main fields) |
| INVARIANT-EVOL-02 | Hypotheses registered before testing, not after (FAIL_EVOL_POST_HOC_HYPOTHESIS) |
| INVARIANT-EVOL-03 | Evolved strategies start at deployment_stage="shadow" (FAIL_EVOL_DIRECT_DEPLOY) |
| INVARIANT-EVOL-07 | Mutation rate adapts on plateau, never hardcoded (FAIL_EVOL_FIXED_MUTATION) |
| INVARIANT-EVOL-08 | Black Swan ensemble requires 2/4 vote (FAIL_EVOL_SINGLE_DETECTOR) |
| INVARIANT-EVOL-12 | OODA Decide/Act offline-only (FAIL_EVOL_OODA_LIVE_DECIDE) |
| INVARIANT-EVOL-16 | EventStore append-only, prune() is only deletion (FAIL_EVOL_EVENT_MUTATION) |
| INVARIANT-EVOL-19 | Graduation budget via required_budget() only (FAIL_EVOL_HARDCODED_GRAD_PARAMS) |
| INVARIANT-EVOL-20 | mx.eval() requires _gpu_lock (FAIL_EVOL_MLX_NO_LOCK) |
| INVARIANT-EVOL-21 | sequential_evidence_score is pre-filter only; final graduation via SPA (FAIL_EVOL_SEQUENTIAL_BYPASS_SPA) |
| INVARIANT-EVOL-22 | MLX subprocess uses spawn context; fork() banned (FAIL_EVOL_FORK_MLX) |

---

*Leviathan v4.0 | DROS v12.4b | 2026-03-14*
