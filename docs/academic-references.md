# Academic References

DROS design decisions are grounded in peer-reviewed research. This document maps each key paper to the specific component it informed.

---

## Market Microstructure

| Paper | Application in DROS |
|:------|:--------------------|
| Easley, López de Prado & O'Hara (2012) — *Flow Toxicity and Liquidity in a High-Frequency World* | **ToxicityShield**: Safety Gate Layer 5 — VPIN-based informed-flow detection before order placement |
| Albers, Durbin & Phan (2025) — *Queue Position Science* | **AQER**: fill probability estimation vs. adverse selection cost in the async queue entry router |
| Avellaneda & Stoikov (2008) — *High-Frequency Trading in a Limit Order Book* | **SpacingOracle**: intensity function λ(δ) = A·exp(−κ·δ) for grid level sizing |
| Kyle (1985) — *Continuous Auctions and Insider Trading* | Game-theoretic order sizing and adverse selection awareness |
| Glosten & Milgrom (1985) — *Bid, Ask and Transaction Prices in a Specialist Market* | Microstructure-aware execution timing and spread modeling |

---

## Volatility Estimation

| Paper | Application in DROS |
|:------|:--------------------|
| Yang & Zhang (2000) — *Drift-Independent Volatility Estimation Based on High, Low, Open, and Close Prices* | **SpacingOracleSSOT** — primary σ_d estimator (OHLCV 4-factor, drift-independent, 14× more efficient than close-to-close) |
| Rogers & Satchell (1991) — *Estimating Variance from High, Low and Closing Prices* | Fallback volatility estimator when Yang-Zhang is unavailable |

---

## Machine Learning & Backtesting

| Paper | Application in DROS |
|:------|:--------------------|
| López de Prado (2018) — *Advances in Financial Machine Learning* | **A8 Backtest Simulator**: CPCV (Combinatorially Purged Cross-Validation) + PBO overfitting guard |
| Bailey, Borwein, López de Prado & Zhu (2016) — *The Probability of Backtest Overfitting* | PBO ≥ 0.3 → FAIL_PBO_OVERFIT blocks learning pipeline promotion |
| Angelopoulos & Bates (2022) — *Conformal Risk Control* | **ACI Risk Controller**: adaptive conformal inference for dynamic risk bounds (Evolution Lab) |

---

## Reinforcement Learning

| Paper | Application in DROS |
|:------|:--------------------|
| Peng, Chang, Grace Zhang, Pieter Abbeel & Sergey Levine (2019) — *Advantage-Weighted Regression: Simple and Scalable Off-Policy Reinforcement Learning* | **AWR Agent**: per-heartbeat fast learning loop for grid parameter adaptation |
| Shafer & Vovk (2019) — *Game-Theoretic Statistics and Safe Anytime-Valid Inference* | **POPPER hypothesis testing**: E-value based safe inference in the Evolution Lab |

---

## Evolutionary Computation

| Paper | Application in DROS |
|:------|:--------------------|
| Mouret & Clune (2015) — *Illuminating Search Spaces by Mapping Elites* | **Alpha Foundry**: MAP-Elites quality-diversity genome search over grid parameter space (pyribs implementation) |

---

## Systems & Game Theory

| Paper | Application in DROS |
|:------|:--------------------|
| Boyd (1987) — *Patterns of Conflict* (OODA Loop) | **OODA Loop module**: offline Observe-Orient-Decide-Act strategy loop, restricted to 03:00–09:00 KST to avoid live-market interference |
| Zinkevich, Johanson, Bowling & Piccione (2007) — *Regret Minimization in Games with Incomplete Information* | **DCFR**: Discounted Counterfactual Regret Minimization for Nash equilibrium in 50-state × 5-action game model |
| Shapley (1953) — *A Value for n-Person Games* | Shapley-weighted symbol allocation across the active slot pool |

---

## Transparency and Trust

| Paper | Application in DROS |
|:------|:--------------------|
| Nosek, B.A., et al. (2018) — *The preregistration revolution*. PNAS. | **Research Product Layer**: Registered Reports model — method fixed before results, outcomes disclosed on same ledger. WATCH filed before outcome → CONFIRMED/EXPIRED published on same ledger regardless of direction. EXPIRED watch signals never suppressed (INVARIANT-MKT-29). |

---

## Notes on Application

These papers were not applied speculatively. Each is referenced in a specific, verifiable component:

- Yang-Zhang σ is the **only** volatility estimator in SpacingOracleSSOT (INVARIANT-SPACING-04: ATR-only estimation is banned)
- CPCV+PBO are **mandatory** before any backtest result promotes to production (LEARNING-13)
- MAP-Elites genomes start in shadow and require SPA p < 0.01 before canary deployment (INVARIANT-EVOL-03)

→ [Back to README](../README.md) · [Safety documentation](./safety.md) · [Learning documentation](./learning.md)
