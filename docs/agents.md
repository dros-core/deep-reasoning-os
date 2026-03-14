# 🤖 Agents — Deep Reasoning OS (DROS)

DROS coordinates 16 specialized AI agents through a structured pipeline. Each agent owns exactly one responsibility — no agent recomputes another's output (SSOT principle).

---

## Agent Reference Table

| Agent | Role | Key Output | Critical Contracts |
|:---|:---|:---|:---|
| **A0** | Orchestrator | Execution order, exception routing | MassGen layer6 |
| **A0.5** | Macro Regime Engine | BULL / BEAR / SIDEWAYS judgment | PSI threshold → long veto |
| **A1** | Data & Feature Collection | Yang-Zhang σ, ATR, fees, sentiment | — |
| **A2** | Reasoning Engine | LLM-structured bias, ensemble ML output | Kelly Risk layer6 |
| **A3** | Safety Guardian | Pre-entry risk block | Blocks on uncertainty |
| **A4** | Candidate Generator | N_total (EnsembleN*), spacing_dec | **SSOT owner of spacing_dec** |
| **A5** | Cost Validator | PASS / FAIL + minimum spacing | Fee-floor compliance |
| **A6** | Scorer & Selector | Top-K grid configuration confirmed | MoE Router layer6 |
| **A7** | Self-Corrector | Repaired candidate after FAIL | **Sole agent permitted to re-sign CardSpec** |
| **A8** | Backtest Simulator | ROI, CVaR, EFH, FCR | CPCV + PBO overfitting guard |
| **A9** | CardSpec Officer | Immutable sha256-sealed CardSpec | Immutable after signature |
| **A10** | Execution Manager | FSM: Staging → Observe → Live | — |
| **A11** | Risk Governance | Exposure, leverage, loss limits | Kelly layer6 |
| **A12** | SRE Monitor | KPI tracking, runbook automation | — |
| **A13** | Debug & QA | Contract enforcement, `_pct` blocking | LogSimilarity layer6 |
| **A14** | Leviathan Bridge | 5-Brain enhancer routing, shadow/production ring dispatch | INVARIANT-EVOL-01 |
| **A15** | Marketing Emitter | 8 hook points, pre-alert system, channel routing | FAIL_MKT_EXPIRED_HIDDEN, FAIL_MKT_LEAD_TIME_RAW |

---

## Call Chains

### Normal Card Creation
```
A0 → A1 → A2 → A4 → A5 → A6 → A9
```

### Self-Correction Loop
```
A5 (FAIL) → A7 (repair) → A4 (regenerate) → A5 (re-validate) → A6 → A9
```

### Backtest Integration
```
A9 (card) → A8 → A6 (re-score) → A9 (final seal)
```

### Parameter Propagation
```
mode (turbo/steady):  A0 → A2 → A4
k_floor:              A2 → A4 → A5
spacing_dec:          A4 → A5 → A7 → A9
```

---

## Critical Invariants

| Code | Severity | Rule |
|:---|:---:|:---|
| `FAIL_UNIT_PCT` | P0 | Only `_dec` units permitted — `_pct` triggers immediate halt |
| `FAIL_CARDSPEC_MUTATION` | P0 | CardSpec is immutable after A9 signature |
| `FAIL_SPACING_SSOT_MISMATCH` | P0 | Spacing must be computed by SpacingOracleSSOT only |
| `FAIL_N_BELOW_10` | P1 | Grid count minimum enforced |
| `FAIL_SPACING_TOO_TIGHT` | P1 | Spacing must meet fee-floor multiple |
| `FAIL_PBO_OVERFIT` | P1 | A8 blocks training when backtest overfitting is detected |

---

## Layer6 Components

Layer6 activates automatically when task complexity exceeds threshold (≥ 0.5).

| Component | Role |
|:---|:---|
| **Fibonacci** | Task decomposition into optimal phases |
| **PhiContext** | File read optimization |
| **Kelly Risk** | Pre-execution risk assessment |
| **LogSimilarity** | Pattern reuse from past similar tasks |
| **MoE Router** | Expert mixture routing |
| **ExpLearning** | Exponential learning rate tracking |
| **Consensus** | Multi-agent majority validation |
| **MassGen** | Parallel bulk generation (A0) |

---

## Design Principles

**Single Responsibility**: Each agent computes one thing. A4 computes spacing; A5 validates it; A9 seals it. No overlap.

**SSOT Enforcement**: Parameters flow in one direction. A7 may re-sign after self-correction — no other agent has this permission.

**Deterministic Contracts**: Agent behavior is governed by named invariant contracts enforced at runtime. AI assists; deterministic code decides.

**Advisory-Only LLM**: LLM outputs are structured suggestions. They do not directly trigger orders. All LLM calls route through a broker with timeout and deterministic fallback.

---

### A14 — Leviathan Bridge (formerly Reliability Analyst)
- Routes decisions to Leviathan v4.0 5-Brain enhancers via EnhancerBus
- Enhancers write to `DecisionPacket.extra_context` only (INVARIANT-EVOL-01)
- Wrap in try/except with debug logging (silent failure banned)
- Shadow ring: would-have logging only, no capital impact

### A15 — Marketing Emitter
- 8 hook points (Hook 1-5: core signals, Hook 6-8: pre-alert)
- Hook 6: PSI threshold → SYMBOL_WATCH (direction_safety.py)
- Hook 7: grid execution degradation → SYMBOL_WATCH (daemon.py)
- Hook 8: Entry gate confirmed block → SYMBOL_CONFIRMED (entry_gate.py)
- All hooks: lazy import + try/except isolation (hook failure cannot affect trading)
- EXPIRED results always published, never suppressed (FAIL_MKT_EXPIRED_HIDDEN)
- lead_time_s must route through humanize_lead_time() (FAIL_MKT_LEAD_TIME_RAW)
