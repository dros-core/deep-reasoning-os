# Case Study: MERL Liquidation and Defense System

## Incident Summary

**Symbol**: MERLUSDT
**Position**: SHORT
**Outcome**: 100% liquidation

**Sequence**: MERLUSDT SHORT position entered during apparent ranging market. A +37% LONG rally followed, exceeding all grid levels. Position liquidated at full loss.

This incident drove the MERL Prevention System (v8.4+), which introduced five defensive layers now active in production.

---

## Root Cause Analysis

### Contributing Factors

1. **Macro sentiment not gated**: PSI veto not enforced at card creation time. PSI was elevated (bullish bias) but entry was allowed.

2. **Tail risk underweighted**: effective_tail_risk used card-level risk metrics only. direction_safety computed a higher risk independently. The max() merge was not implemented.

3. **LLM soft score absorbed hard signals**: LLM confidence adjustment partially offset the directional uncertainty signal, allowing a marginal entry through.

4. **No Net ROI gate**: Expected net ROI was negative after fees and slippage. Entry was not blocked.

5. **Dead Zone too narrow**: p_dir Dead Zone was [0.45, 0.55]. Direction signal at 0.43 was classified as SHORT rather than abstain.

---

## Post-Incident Changes

### v8.4+ — MERL Prevention System

Five defensive layers introduced:

| Layer | Description |
|-------|-------------|
| Entry Gate | PSI hard veto at card creation (FAIL_PSI_ABSOLUTE_VETO) |
| Funding SSOT | Funding rate as independent kill signal |
| Kill Switch | Global entry suspension on drawdown threshold |
| Tail Risk | effective_tail_risk = max(card.risk_metrics, direction_safety) |
| Microstructure | VPIN toxicity detection, order flow analysis |

### v12.0 — Hard Gate Separation

The MERL lesson was formalized into the 8-Layer Safety Gate architecture:

- **Layer 1 (PSI/Macro)**: Enforced at both card creation AND entry decision
- **Layer 2 (Tail Risk)**: effective_tail_risk = max() merge enforced as INVARIANT (TailRiskSSOT)
- **Layer 3 (Net ROI Gate)**: net_roi_expected < -5% blocks entry (FAIL_NEGATIVE_ROI_ENTRY)
- **LLM Hard-Safety Guard**: LLM adjustment zeroed if any hard gate condition violated

### v12.2 — Direction Dead Zone Expansion

Dead Zone widened from [0.45, 0.55] to [0.38, 0.62]:
- Signals in [0.38, 0.62] abstain -- no position
- Asymmetric thresholds: Long requires p_dir >= 0.58, Short requires p_dir <= 0.42
- SHORT at 0.43 would now be classified as abstain (Dead Zone)

### v12.1 — Dynamic SL Architecture

Three-stack SL system prevents runaway loss:
- **Stack A**: Catastrophe Stop always active, cannot be disabled
- **Stack B**: Chandelier Trail, regime-adaptive (ranging: 2.5x ATR minimum)
- **Stack C**: Grid-Fail SL on execution degradation

A MERL-equivalent incident would now be contained by Stack A before full liquidation.

---

## Current Defenses Against MERL-Class Events

1. **PSI Double Gate**: PSI veto checked at card creation AND entry. Cannot be bypassed by FGI recovery (INVARIANT-MACRO-02).

2. **effective_tail_risk**: Always max(card, direction_safety). Single-source tail risk banned.

3. **Net ROI Gate**: Negative expected ROI blocked before position opens.

4. **Dead Zone [0.38, 0.62]**: Marginal directional signals produce no position.

5. **Stack A Catastrophe Stop**: Always active. Cannot be disabled by SL_ENABLED or any flag.

6. **Symbol Sentiment 5-axis**: Crowding + Participation + Aggression + SmartMoney + Execution all checked. Single-axis signal cannot drive entry.

7. **VPIN Toxicity Detection**: Order flow analysis detects informed trading before position opens.

8. **FGI Kelly Cap**: FGI < 20 (extreme fear) caps position size at 2.5% normal size (Fractional Kelly).

---

## Metrics Since MERL Prevention Deployment

| Metric | Pre-MERL | Post v12.1 |
|--------|----------|-----------|
| Catastrophic loss events | Yes | 0 |
| SL whipsaw events | N/A | 0 |
| Negative ROI entries | Yes | 0% |
| Dead Zone abstain rate | 0% (no dead zone) | Active |

---

## Lessons Applied System-Wide

The MERL incident established three architectural principles now enforced as invariants:

1. **Hard gates before soft scores**: No LLM confidence adjustment can override PSI, tail_risk, or net_roi gate.

2. **max() merge for risk**: When two subsystems compute the same risk metric independently, always use the higher value.

3. **Stack A is unconditional**: A stop-loss mechanism that can be disabled provides false safety. Stack A cannot be disabled by any means.

---

*MERL post-mortem and defense system for DROS v12.4b | 2026-03-14*
