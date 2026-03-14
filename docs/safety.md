# 🛡️ Safety — Deep Reasoning OS (DROS)

> **Core principle**: Execution comes after validation. Always.

---

## 8-Layer Safety Gate

Every trade candidate passes eight sequential checks before any order is placed. All layers must pass — a single rejection halts the pipeline.

| Layer | Name | Description |
|:---:|:---|:---|
| 1 | **Macro Sentiment Veto** | Macro regime indicator falls below configured threshold → long veto |
| 2 | **Tail Risk Veto** | Tail risk score exceeds configured threshold → entry blocked |
| 3 | **Net ROI Gate** | Negative expected ROI blocks entry — hard gate before LLM scoring |
| 4 | **Direction Uncertainty Block** | Prediction confidence below abstain threshold → no position |
| 5 | **Range Extreme Check** | Price at boundary of viable grid range → skip |
| 6 | **ToxicityShield / VPIN** | Informed-flow toxicity score exceeds threshold → skip |
| 7 | **Liquidation Probability** | Liquidation distance falls below configured safety margin → blocked |
| 8 | **Position State Gate** | PositionStateMachine registration required before capital allocation |

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

The system enforces 50+ named invariants at runtime. Violations trigger immediate halt or self-correction.

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
| `FAIL_DIRECTION_CACHE_STALE` | hot_reload_cards() hysteresis bypassed without StateStore position confirmation |

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
| `FAIL_MKT_LEAD_TIME_RAW` | lead_time_s raw value exposed without humanize_lead_time() |
| `FAIL_MKT_WATCH_ID_MANUAL` | watch_id constructed manually outside WatchRegistry.new_watch_id() |
| `FAIL_MKT_EXPIRED_HIDDEN` | EXPIRED watch result suppressed instead of published |
| `FAIL_MKT_CONFIRM_NO_WATCH` | SYMBOL_CONFIRMED fired without linked WATCH |
| `FAIL_MKT_DAILY_LIMIT_EXCEEDED` | Channel daily message limit exceeded |

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

---

### INVARIANT-BOOTSTRAP

| Code | Rule |
|------|------|
| INVARIANT-BOOTSTRAP-01 | Unlock requires quality_base_no_llm (LLM boost excluded) |
| INVARIANT-BOOTSTRAP-02 | Shadow mode threshold changes absolutely forbidden |
| INVARIANT-BOOTSTRAP-03 | SL exit refill via queued trigger only |
| INVARIANT-BOOTSTRAP-04 | Multi-condition simultaneous unlock required |
| INVARIANT-BOOTSTRAP-05 | F4 CASCADE and ReliabilityGate are independent components |
| INVARIANT-BOOTSTRAP-06 | BootstrapActionBucket persisted to DaemonCheckpoint |

### INVARIANT-LEVIATHAN

| Code | Rule |
|------|------|
| INVARIANT-LEVIATHAN-01 | Shadow mode — log only, no capital impact |
| INVARIANT-LEVIATHAN-08 | Post-Entry Sentinel — Hard Gate only, LLM calls banned |
| INVARIANT-LEVIATHAN-09 | Ring 1 black lab code banned from production daemon import |
| INVARIANT-LEVIATHAN-11 | _build_leviathan_telemetry() key names must be exact |
