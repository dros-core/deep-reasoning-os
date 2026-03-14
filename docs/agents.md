# Agent Pipeline (A0-A15)

DROS processes each symbol through a sequential agent pipeline. Each agent has a single defined responsibility. Cross-agent parameter reuse is banned — if A4 computes spacing_dec, A9 cannot recompute it.

---

## Pipeline Overview

```
A0  EventLoop         -- outer scheduler, symbol iteration
A1  DataFetcher       -- OHLCV, funding, OI from Binance
A2  ReasoningCore     -- market structure analysis, regime classification
A3  DirectionEngine   -- p_dir computation, calibration, Dead Zone
A4  CandidateGen      -- card creation, spacing_dec SSOT, grid parameters
A5  EntryGate         -- 5-check validation (macro/direction/cost/quality/alignment)
A6  PositionSizer     -- leverage, margin allocation
A7  SelfCorrection    -- parameter adjustment post-observation (only agent allowed to re-sign)
A8  ROIDebias         -- hierarchical Bayesian debias, backtest (CPCV+PBO)
A9  CardSpec          -- final signature, immutable after signing
A10 OrderManager      -- order placement, PLT coordination
A11 Reconciler        -- position reconciliation, orphan detection
A12 RiskLayer         -- Dynamic SL (3-Stack), tail risk, margin safety
A13 LearningBridge    -- Thompson update, AWR reward signal
A14 EvolutionBus      -- Leviathan integration, enhancer routing
A15 MarketingEmitter  -- hook triggers, signal routing to channels
```

---

## Key Agent Contracts

### A0 — EventLoop
- Outer loop, calls each symbol through A1-A15 pipeline
- Per-stage asyncio timeouts:
  - A1/A2 feature extraction: 30s
  - A3/A4/A5 gate: 30s
  - A8 backtest: 60s
  - A6/A14/A9 card: 30s
- Outer wrapper: 300s total

### A2 — ReasoningCore
- `box_width_min/max_dec` fields are in % units -- divide by 100 before use
- Regime output: ranging / trending / volatile

### A3 — DirectionEngine
- Dead Zone: p_dir in [0.38, 0.62] -> abstain (SSOT: direction_safety_contracts.py)
- Asymmetric thresholds: Long >= 0.58, Short <= 0.42
- adx_slope computed from _adx_cache (hardcoding 0.0 triggers FAIL_ADX_SLOPE_HARDCODED)
- OR condition for LiveDirectionCheck (AND triggers FAIL_LIVECHECKAND_CONDITION)

### A4 — CandidateGenerator
- spacing_dec is SSOT for this symbol/card (A9 cannot recompute)
- N_total >= 10 required (FAIL_N_BELOW_10)
- PSI hard gate: PSI <= -0.8 absolute veto; PSI <= -0.65 conditional veto
- Card TTL: constant from config/m4_pro_optimization.py

### A5 — EntryGate
Check sequence (all must pass):
1. Macro sentiment (PSI, FGI tempo gate)
2. Direction confidence (p_dir thresholds + Dead Zone)
3. Cost gate (gross ROI vs fees+slippage)
4. Quality score (quality_base_no_llm >= 85)
5. Live alignment (>= 2 indicators aligned)

Check 4/5 blocks -> SYMBOL_WATCH (not SYMBOL_CONFIRMED)
SYMBOL_CONFIRMED requires connected WATCH (FAIL_MKT_CONFIRMED_NO_WATCH)

### A7 — SelfCorrection
- Only agent permitted to modify and re-sign a CardSpec
- All other agents cannot modify signed cards (FAIL_CARDSPEC_MUTATION)
- Applies WARN_EFH_LOW (spacing down 10%) and WARN_FCR_LOW (spacing up 15%)

### A8 — ROIDebias
- Hierarchical Bayesian debias: symbol -> cluster -> global (FAIL_A8_DEBIAS_FLAT if flat)
- CPCV backtest with purge gap >= 24h (FAIL_CPCV_NO_PURGE)
- PBO >= 0.3 blocks learning (FAIL_PBO_OVERFIT)
- OPE report: DR estimator + Clopper-Pearson CI required (FAIL_OPE_INCOMPLETE)

### A9 — CardSpec Officer
- Signs card with sha256(code+contracts+cardspec)
- After signing: immutable (FAIL_CARDSPEC_MUTATION on any modification)
- Hash mismatch on verification: FAIL_HASH_MISMATCH

### A14 — EvolutionBus
- Routes to Leviathan enhancers via EnhancerBus
- Enhancers write to DecisionPacket.extra_context only (read-only for main fields)
- Each enhancer wrapped in try/except with debug logging (silent failure banned)

### A15 — MarketingEmitter
- Hook triggers for 8 signal types
- All hooks isolated: lazy import + try/except (hook failure cannot affect trading)
- Raw payload transmission to LLM banned (FAIL_MKT_RAW_PAYLOAD)
- Scope enforcement: Hook inserted only in correct variable scope (FAIL_MKT_HOOK_SCOPE_MISMATCH)

---

## SSOT Rules

| Parameter | Owner | Rule |
|-----------|-------|------|
| spacing_dec | A4 | SSOT for card; no recomputation downstream |
| p_dir | A3 | SSOT for direction; no post-hoc adjustment |
| quality_score | A5 | quality_base_no_llm for gates; quality_final for logging |
| net_roi_expected | A8 | SSOT for ROI gate |
| tail_risk | A12 | effective_tail_risk = max(card.risk_metrics, direction_safety) |

---

## LLM Integration Rules

- All LLM calls via LLM Broker (FAIL_LLM_DIRECT_CALL on direct Ollama)
- Timeout mandatory: trades 100ms, analysis 3000ms (FAIL_LLM_NO_TIMEOUT)
- Fallback mandatory: deterministic return on failure (FAIL_LLM_NO_FALLBACK)
- delta_confidence bounded: [-0.05, +0.05] (FAIL_LLM_OUT_OF_RANGE)
- LLM adjustment zeroed if Hard Gate conditions violated (LLM Hard-Safety Guard)
- Self-Refine banned on 4B models (NeurIPS 2023: performance degrades below 13B)

---

## Agent Constants SSOT

`config/m4_pro_optimization.py` is the SSOT for:
- Memory thresholds
- LLM model IDs (changing requires simultaneous update of 11 files: `grep "qwen2.5"`)
- Card TTL
- M4 Pro hardware limits

---

## Shared Rate Limit

All agents share a `SharedRateLimitBudget`:
- Weight: 800/1200 (66% ceiling)
- Max concurrent: 5 requests
- Circuit Breaker: 429/418 -> 5-minute backoff

Direct API calls without budget check are banned.

---

*Agent documentation for DROS v12.4b | 2026-03-14*
