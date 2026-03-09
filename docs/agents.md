# 🤖 Agents — Deep Reasoning OS (DROS)

> 16-Agent roles, contracts, and interaction patterns

---

## Overview

DROS operates through **16 specialized agents** that collaborate in a structured pipeline.  
Each agent owns a specific domain — no agent duplicates another's computation.

**Core principle:** Every parameter has exactly one owner (SSOT).  
`spacing_dec` is computed by A4. A9 does not recompute it. No exceptions.

---

## Agent Roles & Contracts

### A0 — Orchestrator
- **Role:** Controls execution order, manages exceptions, triggers sub-pipelines
- **Layer6:** MassGen (parallel bulk generation)
- **Key rule:** Entry point for all symbol processing cycles

### A0.5 — Macro Regime Engine
- **Role:** Classifies market regime as BULL / BEAR / SIDEWAYS
- **Outputs:** `regime`, `psi_score`, `macro_sentiment`
- **Key rule:** PSI ≤ −0.5 → Long veto (`FAIL_MACRO_SENTIMENT_VETO`)

### A1 — Data & Feature Collection
- **Role:** Collects and computes all market features
- **Outputs:** `σ̂` (Yang-Zhang), `ATR`, `depth_proxy`, `fees_dec`, `sentiment`
- **Key rule:** Source of all raw market data — no agent fetches data independently

### A2 — Reasoning Engine
- **Role:** LLM-assisted structuring + ensemble ML inference
- **Outputs:** `mode_hint` (turbo/steady), `k_floor`, `bias`
- **Layer6:** Kelly Risk
- **Key rule:** LLM is a **structurer only** — never a decision maker

### A3 — Safety Guardian
- **Role:** Pre-entry risk correction and hard blocks
- **Outputs:** PASS / BLOCK + reason code
- **Layer6:** Kelly Risk
- **Key rule:** Direction uncertainty |p_dir − 0.5| ≤ 0.05 → block

### A4 — Candidate Generator
- **Role:** Generates grid candidates using EnsembleN* and SpacingOracleSSOT
- **Outputs:** `N_total`, `spacing_dec`, `grid_type`
- **Layer6:** Fibonacci
- **SSOT owner:** `spacing_dec` — no other agent may recompute this

### A5 — Cost Validator
- **Role:** Validates spacing against fee floor
- **Outputs:** PASS / `FAIL_SPACING_TOO_TIGHT` + `spacing_dec_min`
- **Layer6:** LogSimilarity
- **Formula:** `spacing_dec ≥ k_floor × (fees_dec + slippage_rt_dec)`

### A6 — Scoring & Selection
- **Role:** Selects top-K combinations from validated candidates
- **Outputs:** Final `direction`, `leverage`, `grid_type`, composite score
- **Layer6:** MoE Router

### A7 — Self-Correction
- **Role:** Repairs failed candidates automatically
- **Outputs:** Corrected candidate → returned to A4/A5
- **Layer6:** ExpLearning
- **Key rule:** **Sole agent allowed to re-sign** CardSpec (only during self-correction)
- **Actions:** `spacing_increase` / `reanchor` / `N_recalculate`

### A8 — Backtest Simulator
- **Role:** Validates candidates via historical simulation
- **Outputs:** `roi`, `cvar`, `efh` (expected fill hours), `fcr` (fill completion rate)
- **Validation:** CPCV + PBO — PBO ≥ 0.3 → `FAIL_PBO_OVERFIT`

### A9 — CardSpec Officer
- **Role:** Signs and seals the final CardSpec
- **Outputs:** `CardSpec(STRICT)` + `sha256` checksum
- **Layer6:** Consensus
- **Key rule:** Once signed, CardSpec is **immutable** → `FAIL_CARDSPEC_MUTATION` (P0)

### A10 — Execution Manager
- **Role:** Manages the execution FSM (Staging → Observe → Live)
- **Outputs:** Order placement, position state transitions
- **Layer6:** FSM

### A11 — Risk Governance
- **Role:** Monitors exposure, leverage, and loss limits in real time
- **Layer6:** Kelly Risk
- **Key rule:** Enforces position-level and portfolio-level risk caps

### A12 — SRE Monitor
- **Role:** System reliability, KPI tracking, alert generation, runbook automation
- **Monitors:** Cost Gate ≥ 90% · Negative ROI = 0 · Live-Backtest gap ≤ 10%

### A13 — Debug & QA
- **Role:** Runtime contract enforcement
- **Layer6:** LogSimilarity
- **Blocks:** Any `_pct` unit usage → `FAIL_UNIT_PCT` (P0)

### A14 — Reliability Analyst
- **Role:** Confidence and reliability scoring
- **Target:** Reliability ≥ 0.85

---

## Interaction Patterns

### Normal Card Creation
```
A0 → A1 → A2 → A4 → A5 → A6 → A9
```

### Self-Correction Loop
```
A5 (FAIL_SPACING_TOO_TIGHT)
  → A7 (repair: spacing_increase / reanchor / N_recalculate)
  → A4 (regenerate)
  → A5 (re-validate)
  → A6 (score)
  → A9 (sign)
```

### Backtest Integration
```
A9 (signed card)
  → A8 (CPCV backtest)
  → A6 (re-score with backtest result)
  → A9 (final sign)
```

---

## Parameter Propagation (SSOT Chain)

| Parameter | Owner | Consumers |
|-----------|-------|-----------|
| `mode` (turbo/steady) | A0 | A2 → A4 |
| `k_floor` | A2 | A4 → A5 |
| `spacing_dec` | A4 | A5 → A7 → A9 |
| `σ̂` (volatility) | A1 | A4 (via SpacingOracleSSOT) |
| `regime` | A0.5 | A2 → A4 |

---

## Critical Invariants

| Code | Rule | Severity |
|------|------|----------|
| `FAIL_UNIT_PCT` | `_pct` unit usage anywhere | P0 — immediate halt |
| `FAIL_CARDSPEC_MUTATION` | CardSpec modified after A9 signing | P0 — immediate halt |
| `FAIL_N_BELOW_10` | N_total < 10 | P1 — learning halt |
| `FAIL_SPACING_TOO_TIGHT` | spacing < k_floor × fees | P1 — learning halt |
| `FAIL_PBO_OVERFIT` | PBO ≥ 0.3 in backtest | P1 — learning halt |
| `FAIL_SPACING_SSOT_MISMATCH` | SpacingOracle bypassed | P0 — immediate halt |

---

*→ See [Architecture](./architecture.md) · [Safety](./safety.md)*
