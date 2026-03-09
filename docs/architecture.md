# 🏗️ Architecture — Deep Reasoning OS (DROS)

> Full system architecture and 16-Agent collaborative pipeline

---

## System Overview

DROS is a **layered operating architecture** built around one principle:  
**Execution comes only after deterministic validation.**

```
┌─────────────────────────────────────────────────────┐
│              DEEP REASONING OS — Full Flow           │
└─────────────────────────────────────────────────────┘

[Binance Market Data]
  WebSocket (real-time trades / order book)
  REST API  (OHLCV, positions, balance)
         ↓
[A1: Data & Feature Collection]
  Yang-Zhang σ̂, ATR, depth proxy, funding rate, sentiment
         ↓
[A2: Reasoning Engine]        [A0.5: Macro Regime Engine]
  LLM structuring + ensemble    BULL / BEAR / SIDEWAYS
         ↓                              ↓
[A3: Safety Guardian] ←── reject → STOP
  p_dir uncertainty / Tail Risk / Range Extreme
         ↓
[A4: Candidate Generator]
  N_total (EnsembleN*) + spacing_dec (SpacingOracleSSOT)
         ↓
[A5: Cost Validator]          [A7: Self-Correction]
  k_floor × fees ≥ spacing      FAIL → spacing↑ / reanchor
         ↓ PASS
[A6: Scoring & Selection]
  Top-K combination → leverage / direction / grid type
         ↓
[A8: Backtest Simulator]
  CPCV + PBO validation / CVaR / EFH / FCR
         ↓ PASS
[A9: CardSpec Officer]
  sha256 signature → immutable CardSpec
         ↓
[A10: Execution Manager]      [AQER: Entry Quality Router]
  Staging → Observe → Live     StaleGate + StickyWindow
         ↓
[Binance Order Execution]
  Reconciler → PLT (PerLevelTP) → Position Management
         ↓
[Learning Feedback Loop]
  AWR Agent + Thompson Sampling + BLS → next cycle

[A11–A14: Risk · SRE · Reliability] (parallel, always-on)
[AI Evolution Lab] (EnhancerBus → shadow → canary → production)
```

---

## Agent Pipeline

| Agent | Role | Key Output | Layer6 |
|-------|------|-----------|--------|
| **A0** | Orchestrator | Execution order / exception management | MassGen |
| **A0.5** | Macro Regime | BULL / BEAR / SIDEWAYS classification | — |
| **A1** | Data & Features | σ̂, ATR, depth_proxy, fees, sentiment | — |
| **A2** | Reasoning Engine | mode_hint, k_floor, bias | Kelly |
| **A3** | Safety Guardian | Risk correction / entry block | Kelly |
| **A4** | Candidate Generator | N_total, spacing_dec, grid_type | Fibonacci |
| **A5** | Cost Validator | PASS/FAIL + spacing_dec_min | LogSimilarity |
| **A6** | Scoring & Selection | Top-K combination finalized | MoE Router |
| **A7** | Self-Correction | FAIL → auto-repair → regenerate | ExpLearning |
| **A8** | Backtest Simulator | ROI, CVaR, EFH, FCR | — |
| **A9** | CardSpec Officer | STRICT CardSpec + sha256 seal | Consensus |
| **A10** | Execution Manager | Staging → Observe → Live FSM | FSM |
| **A11** | Risk Governance | Exposure / leverage / loss limits | Kelly |
| **A12** | SRE Monitor | KPI / alerts / runbook | — |
| **A13** | Debug & QA | `_pct` block / complexity check | LogSimilarity |
| **A14** | Reliability Analyst | Reliability ≥ 0.85 validation | — |

---

## Call Chains

```
[Normal Card Creation]
A0 → A1 → A2 → A4 → A5 → A6 → A9

[Self-Correction]
A5(FAIL) → A7 → A4 → A5(re-validate) → A6

[Backtest Integration]
A9(card) → A8 → A6(re-score) → A9(final sign)

[Parameter Propagation]
mode (turbo/steady) : A0 → A2 → A4
k_floor             : A2 → A4 → A5
spacing_dec         : A4 → A5 → A7 → A9
```

---

## SpacingOracleSSOT

Single source of truth for grid spacing. All agents reference one oracle.

```
Yang-Zhang Volatility (Open-High-Low-Close integrated)
    ↓
σ_d estimation → ATR supplement
    ↓
SpacingOracleSSOT.compute_spacing()
    → spacing_dec ≥ k_floor × (fees_dec + slippage_rt_dec)
    → 10% hysteresis + 30-min cooldown
```

- ATR-only σ_d estimation forbidden (`INVARIANT-SPACING-04`)
- Rebuild requires `should_rebuild()` confirmation

---

## EnsembleN* — Dynamic Grid Count

Five factors determine optimal grid count:

| Factor | Effect |
|--------|--------|
| Volatility (σ_d) | Higher vol → fewer grids |
| Capital efficiency | Margin-to-leverage optimization |
| Fill frequency (EFH) | Charge frequency adjustment |
| Regime (Macro) | BULL/BEAR/SIDEWAYS tuning |
| ATR multiplier | Absolute range density |

→ N_total ≥ 10 always enforced (`FAIL_N_BELOW_10`)  
→ Turbo mode: 10–20 grids · Steady mode: 16–32 grids

---

## CardSpec — Immutable Trade Specification

```python
CardSpec(
    symbol      = "BTCUSDT",
    direction   = "long",        # long / short / neutral
    N_total     = 16,
    spacing_dec = 0.0025,        # 0.25% — _dec unit only, _pct forbidden
    leverage    = 5,
    grid_type   = "geometric",
    entry_price = 95_000.0,
    checksum    = sha256(...)    # immutable seal
)
```

- Only A7 self-correction may re-sign (sole exception)
- Violation: `FAIL_CARDSPEC_MUTATION` (P0 immediate halt)

---

## Layer6 Components (auto-activated at complexity ≥ 0.5)

| Component | Role |
|-----------|------|
| Fibonacci | Golden-ratio task decomposition |
| PhiContext | File-read golden-ratio optimization |
| Kelly Risk | Pre-execution risk assessment |
| LogSimilarity | Historical pattern reuse |
| MoE Router | Mixture-of-experts routing |
| ExpLearning | Exponential learning tracker |
| Consensus | Majority-vote validation |
| MassGen | Parallel bulk generation |

---

*→ See [Agents](./agents.md) · [Safety](./safety.md) · [Execution](./execution.md)*
