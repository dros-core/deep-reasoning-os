# 🛡️ Safety — Deep Reasoning OS (DROS)

> 7-Layer Entry Gate, invariant contracts, and liquidation defense

---

## Core Principle

> **"Execution comes after validation. Always."**

DROS does not optimize for maximum trade frequency.  
It optimizes for **survivable, consistent performance** by blocking entries that fail any safety condition.

---

## 7-Layer Entry Gate

Every entry passes through 7 sequential gates. **Any single failure blocks execution.**

```
Entry Request
    │
    ├─ Layer 1: Macro Sentiment Veto
    │     PSI ≤ −0.5 → Long completely blocked
    │     → FAIL_MACRO_SENTIMENT_VETO
    │
    ├─ Layer 2: Tail Risk Veto
    │     tail_risk ≥ 0.8 → All entry blocked (direction-agnostic)
    │     → FAIL_TAIL_RISK_VETO
    │
    ├─ Layer 3: Direction Uncertainty Block
    │     |p_dir − 0.5| ≤ 0.05 → Neutral Zone → blocked
    │     (CLO/USDT event defense — 2026-02-04)
    │
    ├─ Layer 4: Range Extreme Check
    │     Price at range boundary → entry blocked
    │
    ├─ Layer 5: ToxicityShield (Game Theory)
    │     VPIN toxicity_score > threshold → blocked
    │     (institutional adverse flow defense)
    │
    ├─ Layer 6: Liquidation Probability (v8.6)
    │     liq_distance < AOSM_MIN_LIQ_DISTANCE_K × σ_1h
    │     → margin release blocked
    │     → FAIL_AOSM_UNSAFE_MARGIN_RELEASE
    │
    └─ Layer 7: AQER StaleGate
          card_age > 5,400s → slot fill completely blocked
          → FAIL_AQER_STALE_ENTRY
```

---

## Gate Reference Table

| Gate | Condition | Action | Code |
|------|-----------|--------|------|
| Macro Sentiment | PSI ≤ −0.5 | Block Long | `FAIL_MACRO_SENTIMENT_VETO` |
| Tail Risk | tail_risk ≥ 0.8 | Block all | `FAIL_TAIL_RISK_VETO` |
| Direction Uncertainty | \|p_dir−0.5\| ≤ 0.05 | Block entry | — |
| Range Extreme | Price at boundary | Block entry | — |
| Toxicity Shield | VPIN > threshold | Block entry | — |
| Liquidation Probability | liq_distance < safety | Block margin | `FAIL_AOSM_UNSAFE_MARGIN_RELEASE` |
| Card Freshness | card_age > 5,400s | Block slots | `FAIL_AQER_STALE_ENTRY` |

---

## Liquidation Defense — v8.6

> *"Predicting unknown patterns is near-impossible. Computing liquidation probability directly and blocking it is the ultimate solution."*

```
liq (Short) ≈ entry + remaining_margin / qty
liq (Long)  ≈ entry − remaining_margin / qty

liq_distance = |mark_price − liq_price| / mark_price

Safety condition:
  liq_distance ≥ AOSM_MIN_LIQ_DISTANCE_K × σ_1h

Violation → FAIL_AOSM_UNSAFE_MARGIN_RELEASE (P1)
```

---

## Real Liquidation Events — Lessons Learned

### MERL Event (2026-02-02)

| Item | Detail |
|------|--------|
| Symbol | MERLUSDT |
| System judgment | SHORT |
| Actual result | +37% LONG rally |
| Outcome | 100% liquidation |

**Root causes identified:**

| ID | Root Cause | Resolution Module |
|----|-----------|-------------------|
| E1 | Funding rate misinterpretation | `contracts/funding_ssot.py` |
| E2 | Lagging indicators only (MACD, RSI, ADX) | `services/adaptive_direction_engine.py` |
| E3 | Tail Risk KPI undefined | `contracts/decision_policy_contracts.py` |
| W4 | No real-time Kill Switch | `services/dynamic_kill_switch.py` |
| W7 | No Tail Risk model | `services/tail_risk_model.py` |

**Defenses added:** 5-layer Entry Gate · Funding SSOT · Dynamic Kill Switch · Tail Risk Model · Microstructure signals

---

### CLO Event (2026-02-04)

| Item | Detail |
|------|--------|
| Symbol | CLO/USDT |
| p_dir_12h | **0.50** (50% — maximum uncertainty) |
| System judgment | SHORT |
| Actual result | +24% LONG pump |

**Root cause:** Direction entered SHORT despite p_dir = 0.5 (pure coin flip).

**Defense added — Entry Gate Check Layer 3:**
```python
# Neutral Zone ±5% from 0.5
if abs(p_dir_12h - 0.5) <= 0.05:
    return "BLOCKED", "DIRECTION_TOO_UNCERTAIN"
```

---

## Fail Code Reference

### P0 — Immediate System Halt

| Code | Trigger |
|------|---------|
| `FAIL_UNIT_PCT` | `_pct` unit used anywhere |
| `FAIL_CARDSPEC_MUTATION` | CardSpec modified after signing |
| `FAIL_ORPHAN_FORCED_CLOSE` | Forced full liquidation of orphan |
| `FAIL_NCO_SCALE_IN_ATTEMPTED` | Scale-In on NCO position |
| `FAIL_SPACING_SSOT_MISMATCH` | SpacingOracle bypassed |
| `FAIL_REDUCE_ONLY_BLOCKED_BY_MARGIN_BACKOFF` | Reduce-only order blocked |
| `FAIL_LLM_DIRECT_CALL` | Direct Ollama call without broker |
| `FAIL_IMPORT_RELOAD` | `importlib.reload()` used |

### P1 — Learning / Execution Halt

| Code | Trigger |
|------|---------|
| `FAIL_N_BELOW_10` | N_total < 10 |
| `FAIL_SPACING_TOO_TIGHT` | spacing < k_floor × fees |
| `FAIL_HASH_MISMATCH` | sha256 integrity check failed |
| `FAIL_MACRO_SENTIMENT_VETO` | PSI ≤ −0.5 |
| `FAIL_TAIL_RISK_VETO` | tail_risk ≥ 0.8 |
| `FAIL_PBO_OVERFIT` | PBO ≥ 0.3 in backtest |
| `FAIL_AOSM_UNSAFE_MARGIN_RELEASE` | Liquidation distance insufficient |

### Warnings — Self-Correcting

| Code | Trigger | Auto-action |
|------|---------|-------------|
| `WARN_EFH_LOW` | Fill frequency low | spacing ↓ 10% |
| `WARN_FCR_LOW` | Fill completion low | spacing ↑ 15% |
| `WARN_NEGATIVE_ROI` | Negative ROI card | Card rejected |

---

## ORPHAN Position Contracts

Non-Card Orphan (NCO) positions follow strict rules:

```
INVARIANT-ORPHAN-02: no_fills_30min → rotation exempt, SL forbidden, TP-only exit
INVARIANT-ORPHAN-07: Forced full liquidation ABSOLUTELY FORBIDDEN (P0)
INVARIANT-NCO-09:    Scale-In ABSOLUTELY FORBIDDEN (P0)
INVARIANT-NCO-07:    drag_reversion_ratio > 5.0 → MANUAL_REVIEW
                     drag_reversion_ratio > 2.0 → EXIT_ACCELERATE
```

---

## Direction Safety Rules

```
p_dir ∈ [0, 1]  (calibrated probability)

Neutral Zone:    |p_dir − 0.5| ≤ 0.05  →  entry blocked
Abstain:         confidence < ABSTAIN_THRESHOLD  →  direction withheld
Macro Veto:      PSI ≤ −0.5  →  Long blocked

Dynamic threshold table (no hardcoded values):
  threshold = table[regime][cluster]
  if p_win ≥ threshold: proceed
```

---

*→ See [Architecture](./architecture.md) · [Execution](./execution.md) · [Evolution Lab](./evolution-lab.md)*
