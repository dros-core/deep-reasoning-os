# Frequently Asked Questions

---

## System Design

**Q: Why does DROS use a grid strategy instead of directional prediction?**

Grid strategies profit from volatility regardless of direction. DROS combines grid execution with directional gating -- the grid captures volatility premium, while the direction engine ensures grids are only deployed when the bias is sufficiently clear. Ambiguous signals produce no position (Dead Zone abstain).

---

**Q: What does "Dead Zone" mean?**

When p_dir (directional probability) falls in [0.38, 0.62], the system abstains from taking a position. This range represents insufficient directional conviction. The zone was widened from [0.45, 0.55] after the MERL incident showed that marginal direction signals led to losses.

---

**Q: Why are there 8 safety gates?**

Each gate addresses a distinct failure mode observed in production or analysis. They are deliberately redundant -- multiple gates can block the same entry for different reasons. The redundancy is intentional: a single gate failure should never lead to a catastrophic position.

---

**Q: Can I disable the stop-loss?**

Stack A (Catastrophe Stop) cannot be disabled by any means. SL_ENABLED=0 auto-maps to CONTROLLED_OFF, which preserves Stack A. BREAKGLASS_OFF requires symbol-scope, TTL, and operator_reason -- and still preserves Stack A. There is no fully-off SL state.

---

**Q: What is the difference between MANAGED_POSITION and EXTERNAL_ORPHAN?**

MANAGED_POSITION is a position created by DROS through the full 8-gate pipeline. EXTERNAL_ORPHAN is any position that exists but was not created through the pipeline (pre-existing positions, positions from manual trading, or positions that lost their card association). ORPHAN positions are managed to natural exit (TP-only) and cannot be force-closed or scaled-in.

---

## Learning and Evolution

**Q: How does Thompson Sampling work in DROS?**

Each symbol+regime combination maintains a Beta distribution representing success probability. On exit, if net_roi > 0 (after fees and funding), the success count increments. If net_roi <= 0, the failure count increments. Future entries sample from this distribution. Updates are throttled to 30 minutes per symbol to prevent reward flooding.

**Key**: Success is defined by net_roi, not exit type. A position that exits via TP but has net_roi <= 0 is a failure.

---

**Q: What is CPCV+PBO and why is it required?**

CPCV (Combinatorial Purge Cross-Validation) with PBO (Probability of Backtest Overfitting) measures whether backtest performance is due to genuine edge or parameter fitting. PBO >= 0.3 means the strategy is likely overfit and learning is blocked. Purge gap >= 24h prevents data leakage between train and test periods.

---

**Q: What is the graduation pipeline for new strategies?**

All new strategies start in Shadow (Ring 2). Promotion requires:
1. 100+ events
2. 7 days shadow duration
3. EPE < 5% (Digital Twin parity)
4. ROI improvement vs baseline
5. SPA sequential test: p < 0.01

Canary deployment comes next (small capital allocation), then Production if canary metrics hold.

---

**Q: Can I add a new evolution experiment without shadow mode?**

No. INVARIANT-EVOL-03 requires all evolved strategies to start at deployment_stage="shadow". Direct production deployment triggers FAIL_EVOL_DIRECT_DEPLOY.

---

## Marketing and Signals

**Q: What is the difference between SYMBOL_WATCH and SYMBOL_CONFIRMED?**

SYMBOL_WATCH is filed when the system detects an early condition that may develop. It is a pre-observation -- no confirmation has occurred yet. SYMBOL_CONFIRMED is filed when the conditions described in a WATCH have materialized within the observation window.

WATCH and CONFIRMED are always linked -- a CONFIRMED without a preceding WATCH triggers FAIL_MKT_CONFIRMED_NO_WATCH.

---

**Q: What happens when a WATCH expires?**

SYMBOL_WATCH_EXPIRED is published to the Private channel. Expired observations are never suppressed (INVARIANT-MKT-29). This transparency model (from Numerai) means the full record -- including failed predictions -- is available.

---

**Q: Why are EXPIRED signals published?**

Because suppressing failures creates survivorship bias in the observable record. The watch confirmation rate (confirmed / total) is only meaningful if all outcomes are reported. Suppressing EXPIRED would inflate apparent accuracy.

---

**Q: What is the "same ledger" principle?**

Observations registered before outcomes, disclosed on the same ledger afterward. WATCH files the observation. CONFIRMED or EXPIRED discloses the outcome. The timestamps are verifiable. This is structurally equivalent to Registered Reports in academic research -- method fixed before results.

---

## Operations

**Q: What does the daily limit mean for marketing signals?**

| Channel | Daily Limit |
|---------|------------|
| X | 2 |
| Telegram Public | 5 |
| Telegram Private | 12 |

Exceeding the limit triggers FAIL_MKT_DAILY_LIMIT_EXCEEDED. Limits exist to prevent alert fatigue (Reuters research: excessive alerts cause subscriber churn).

---

**Q: What is the Bootstrap Safety Gate?**

The Bootstrap gate prevents the system from opening new positions after a restart or in a cold-start state until conditions are safe. Five conditions must all be met simultaneously:
1. rank == 1 (best quality candidate)
2. positioned == 0 (slot is empty)
3. quality_base_no_llm >= 85 (quality without LLM boost)
4. age <= 60 minutes (fresh signal)
5. live_alignment >= 2 (at least 2 indicators confirm)

LLM quality boost (quality_final) cannot be used for unlock -- only quality_base_no_llm.

---

**Q: Why is importlib.reload() banned?**

Hot reload via importlib.reload() in a running async process causes race conditions between module-level state and in-flight coroutines. DROS uses SIGHUP for configuration reload, which triggers a safe checkpoint-and-restart sequence instead.

---

**Q: What is the _pct ban?**

All numeric trading parameters must use `_dec` suffix (decimal representation). `_pct` suffix (percentage representation) is permanently banned to prevent decimal/percentage confusion bugs. Example: spacing_dec = 0.005 (0.5%), never spacing_pct = 0.5. Violation triggers FAIL_UNIT_PCT (P0 immediate halt).

---

*FAQ for DROS v12.4b | 2026-03-14*
