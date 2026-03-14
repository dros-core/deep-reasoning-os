# 📊 Performance — Deep Reasoning OS (DROS)

> **Note**: This page documents operational characteristics and design targets — not backtested returns or live PnL figures.

---

## Execution Layer

| Characteristic | Detail |
|:---|:---|
| **Runtime model** | Single-process asyncio daemon |
| **Checkpoint safety** | Atomic write (write → fsync → rename) |
| **Microstructure** | Order timing adapts to informed-flow signals |
| **Order jitter** | Game-theoretic randomization reduces adverse selection |
| **Reconciliation** | 5-step heartbeat reconciler |

---

## Safety Gate

| Characteristic | Detail |
|:---|:---|
| **Layers per entry** | 7 sequential gates — all must pass |
| **Enforcement** | Deterministic; no probabilistic bypass |
| **P0 violations** | Immediate system halt |
| **Card freshness** | Configurable TTL enforced at entry |
| **Liquidation check** | Per-entry margin safety validation |

---

## Learning Pipeline

| Characteristic | Detail |
|:---|:---|
| **AWR cadence** | Per-heartbeat parameter adaptation |
| **Thompson Sampling** | ~30-minute throttle per symbol |
| **BLS isolation** | Subprocess architecture for full OS memory reclamation |
| **Replay buffer** | Fixed-size deque; oldest entries auto-evicted |
| **Calibrator selection** | Sample-count adaptive (sparse-safe below threshold) |
| **Overfitting guard** | PBO gate blocks training when overfitting detected |

---

## AI Evolution Lab

| Characteristic | Detail |
|:---|:---|
| **Production modules** | 7 active (ACI Risk, EventStore, Digital Twin, Counterfactual Lab, Black Swan, Alpha Foundry, OODA) |
| **Shadow modules** | 8 under evaluation |
| **Minimum shadow period** | 7 days before canary promotion |
| **Canary allocation** | 10% of capital |
| **Graduation test** | SPA (p < 0.01) |
| **Black Swan ensemble** | 2-of-4 detector vote required |
| **OODA loop** | Offline only (03:00–09:00 KST) |

---

## Codebase Scale

| Component | Count |
|:---|:---|
| **Specialized agents** | 16 |
| **Named invariant contracts** | 43 |
| **Evolution modules** | 53 |
| **Feature flags** | 67 |

---

## KPI Targets

| KPI | Target |
|:---|:---|
| **Cost Gate pass rate** | Target threshold maintained |
| **Negative-ROI cards** | Target zero |
| **Live-Backtest gap** | ≤ 10% |
| **Card generation rate** | Target ≥ 80% |

*Actual measured values are not published publicly.*