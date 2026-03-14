# 🛡️ Safety — Deep Reasoning OS (DROS)

> **Core principle**: Execution comes after validation. Always.

---

## 7-Layer Entry Gate

Every trade candidate passes seven sequential checks before any order is placed. All layers must pass — a single rejection halts the pipeline.

| Layer | Name | Description |
|:---:|:---|:---|
| 1 | **Macro Sentiment Veto** | Macro regime indicator falls below configured threshold → long veto |
| 2 | **Tail Risk Veto** | Tail risk score exceeds configured threshold → entry blocked |
| 3 | **Direction Uncertainty Block** | Prediction confidence below abstain threshold → no position |
| 4 | **Range Extreme Check** | Price at boundary of viable grid range → skip |
| 5 | **ToxicityShield / VPIN** | Informed-flow toxicity score exceeds threshold → skip |
| 6 | **Liquidation Probability** | Liquidation distance falls below configured safety margin → blocked |
| 7 | **AQER StaleGate** | Card freshness TTL expired → entry forbidden |

---

## Real Liquidation Events

DROS documents every liquidation event publicly. These events drove the safety architecture.

### MERL Event (2026-02-02)
- **Setup**: SHORT judgment on MERLUSDT
- **Outcome**: +37% LONG rally → 100% liquidation
- **Response**: 5 new safety layers added; full post-mortem in [Case Study](./case-study-merl/)

### CLO Event (2026-02-04)
- **Setup**: SHORT judgment on CLOUSDT
- **Outcome**: +24% LONG pump
- **Response**: Neutral Zone defense layer added

→ See full analysis: [docs/case-study-merl/](./case-study-merl/)

---

## Invariant Contracts

The system enforces 43 named invariants at runtime. Violations trigger immediate halt or self-correction.

### P0 — Immediate System Halt

| Code | Trigger |
|:---|:---|
| `FAIL_UNIT_PCT` | `_pct` unit used anywhere (only `_dec` permitted) |
| `FAIL_CARDSPEC_MUTATION` | Signed CardSpec modified after A9 signature |
| `FAIL_ORPHAN_FORCED_CLOSE` | Forced full liquidation of ORPHAN position |
| `FAIL_NCO_SCALE_IN_ATTEMPTED` | Scale-in on non-card orphan position |
| `FAIL_SPACING_SSOT_MISMATCH` | Spacing computed outside SpacingOracleSSOT |
| `FAIL_REDUCE_ONLY_BLOCKED_BY_MARGIN_BACKOFF` | Reduce-only level suppressed by margin logic |
| `FAIL_LLM_DIRECT_CALL` | LLM called directly without broker |
| `FAIL_IMPORT_RELOAD` | `importlib.reload()` invoked at runtime |

### P1 — Learning / Execution Halt

| Code | Trigger |
|:---|:---|
| `FAIL_N_BELOW_10` | Grid count falls below minimum |
| `FAIL_SPACING_TOO_TIGHT` | Spacing below fee-floor multiple |
| `FAIL_HASH_MISMATCH` | CardSpec hash does not match code state |
| `FAIL_MACRO_SENTIMENT_VETO` | Macro gate triggered |
| `FAIL_TAIL_RISK_VETO` | Tail risk gate triggered |
| `FAIL_PBO_OVERFIT` | Backtest overfitting detected |
| `FAIL_AOSM_UNSAFE_MARGIN_RELEASE` | Margin release without liquidation distance validation |

### Self-Correcting Warnings

| Code | Response |
|:---|:---|
| `WARN_EFH_LOW` | Spacing reduced automatically |
| `WARN_FCR_LOW` | Spacing increased automatically |
| `WARN_NEGATIVE_ROI` | Card rejected before execution |

---

## ORPHAN Position Contracts

Positions that lose their active CardSpec (e.g., after rotation or system restart) enter ORPHAN state with strict rules:

- **No forced liquidation** — absolute prohibition (P0)
- **No scale-in** — absolute prohibition (P0)
- **Rotation exempt** — `no_fills_30min` grace applies
- **Exit only via TP** — stop-loss forbidden; take-profit only
- **Drag ratio monitoring** — escalating alerts trigger manual review or accelerated exit

---

## Deployment Safety

All new strategies and features follow a staged rollout:

```
Shadow (full validation, no capital) → Canary (10% capital) → Production (100%)
```

- Minimum 7-day shadow period
- SPA statistical test (p < 0.01) required for graduation
- Any P0 violation at any stage → immediate rollback
