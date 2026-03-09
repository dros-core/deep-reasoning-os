# 🗺️ DROS Roadmap

> Deep Reasoning OS — architecture evolution timeline

This roadmap reflects the actual development trajectory of DROS. Items marked ✅ are live in production. Items marked 🔄 are in shadow/canary. Items marked 🎯 are planned.

---

## ✅ Completed (Live in Production)

### Core Architecture
- [x] 16-Agent collaborative pipeline (A0–A14)
- [x] SpacingOracleSSOT — Yang-Zhang volatility, single source of truth
- [x] CardSpec immutable signing with sha256 checksum
- [x] 7-Layer Entry Gate (Macro Sentiment → Card Freshness)
- [x] EnsembleN* dynamic grid count optimization

### Execution Layer
- [x] v11.20 Rolling grid execution engine (~11,000 lines)
- [x] ORPHAN position management (no forced close invariant)
- [x] AOSM v2 — Adaptive Orphan Slot Management
- [x] AQER — Async Queue Entry Router (StaleGate + StickyWindow)
- [x] PerLevelTP (PLT) — per-fill take-profit grid
- [x] Atomic checkpoint writes (write → fsync → rename)
- [x] BatchOrderRouter (AQER P2)

### Learning Pipeline
- [x] AWR Agent — per-heartbeat dense reward MDP
- [x] Thompson Sampling Bandit — per-rotation preset selection
- [x] BLS — Bayesian Learning Subprocess (isolated, memory-safe)
- [x] SparseSafeCalibrator — sample-adaptive calibration (Temperature / Beta / Isotonic)
- [x] CPCV + PBO overfit detection (PBO ≥ 0.3 → block)

### Safety & Monitoring
- [x] MERLUSDT liquidation event → 5-layer defense retrofit (2026-02-02)
- [x] CLO/USDT Neutral Zone gate (p_dir ±5%) (2026-02-04)
- [x] Direction Safety v8.9.2 — PSI, tail risk, abstain threshold
- [x] Kill Switch + circuit breaker (429/418 → 5-min backoff)

### AI Evolution Lab v3
- [x] EnhancerBus — Strangler Fig pattern, 8-enhancer pipeline
- [x] ACI Risk — Adaptive Conformal Inference controller
- [x] EventStore — SQLite WAL append-only event log
- [x] Digital Twin — EPE/FRE/LPE parity tracking
- [x] Counterfactual Lab — OPE Capped SNIPS policy evaluation
- [x] Black Swan Ensemble — 2/4 vote (ADWIN+CUSUM+BOCPD+Hawkes)
- [x] Alpha Foundry — pyribs MAP-Elites genome evolution
- [x] OODA Loop — offline strategy loop (03:00–09:00 KST)

---

## 🔄 In Progress (Shadow / Canary)

- [ ] **USE_PARALLEL_MAIN_LOOP** — parallel main_loop + rotation (shadow, v11.20)
- [ ] **USE_AQER_BATCH_ORDERS** — batch order execution (shadow → production 2026-03-15)
- [ ] **USE_EVOL_ENTROPY_REGIME** — Kozachenko-Leonenko entropy detector
- [ ] **USE_EVOL_ISING_PHASE** — Ising magnetization phase detector
- [ ] **USE_EVOL_LOTKA_VOLTERRA** — Prey/predator regime detector
- [ ] **USE_EVOL_ACO_SELECTOR** — Ant Colony pheromone symbol ranker
- [ ] **USE_EVOL_MCTS_REASONING** — Monte Carlo Tree Search grid optimizer
- [ ] **USE_EVOL_CFC_SENSOR** — MIT CfC microstructure anomaly sensor

---

## 🎯 Planned

### Near-term
- [ ] Multi-symbol correlation awareness in slot allocation
- [ ] Regime-conditioned Thompson Sampling priors
- [ ] Cross-symbol transfer learning (A15 prototype)
- [ ] Enhanced VPIN + Hawkes process microstructure fusion

### Research Pipeline
- [ ] Reinforcement Learning from Market Feedback (RLMF)
- [ ] Causal inference for regime change detection
- [ ] Portfolio-level Kelly allocation across symbols
- [ ] Federated learning across exchange venues (research phase)

---

## 💡 Research Exploration

> These are active research questions — no implementation commitment yet.

- Applying large language models as market microstructure interpreters (read-only, zero execution authority)
- Symbolic AI + neural hybrid for regime classification
- Adversarial robustness testing for the Black Swan ensemble
- Cross-asset regime spillover detection

---

## 📋 Version History (Selected)

| Version | Date | Key Changes |
| :--- | :--- | :--- |
| v11.20 | 2026-03 | N* 60s TTL caching, parallel main_loop shadow |
| v11.19 | 2026-03 | PLT-Reconciler 2-layer exemption fix |
| v11.18 | 2026-03 | Card registry type safety fix |
| v11.17 | 2026-03 | NCO GhostCard + inventory pressure |
| v11.16 | 2026-03 | PLT Recover Entry Guard |
| v11.15 | 2026-03 | AOSM v2 production |
| v11.14 | 2026-03 | PLT cancel preservation fix |
| v8.4+ | 2026-02 | MERL Prevention — 5-layer defense |
| v5.0 | 2025-Q4 | Direction Engine ensemble ML |
| v3.0 | 2025-Q3 | AI Evolution Lab foundation |

---

*→ See [Architecture](./docs/architecture.md) · [Evolution Lab](./docs/evolution-lab.md)*