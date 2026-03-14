# 📉 Case Study: The MERL Liquidation Event

> *"Every safety layer in DROS has a real failure behind it."*

---

## Overview

On **February 2, 2026**, DROS held a SHORT position on MERLUSDT.  
The token rallied **+37%** in a sustained LONG move.  
The position reached **100% liquidation**.

This document describes what happened, why the existing system failed to prevent it,
and how DROS was redesigned in response.

---

## What Happened

| Item | Detail |
| :--- | :--- |
| **Symbol** | MERLUSDT |
| **DROS judgment** | SHORT |
| **Actual market move** | +37% LONG rally |
| **Outcome** | 100% liquidation |
| **Date** | 2026-02-02 |

The event was not a data error or execution bug.  
The system made a directional judgment — and the market moved against it with no mechanism to intervene.

---

## Root Cause Analysis

Post-event investigation identified five contributing factors:

| ID | Root Cause | Category |
|----|-----------|----------|
| E1 | Funding rate signal misinterpreted — bullish funding treated as neutral | Signal quality |
| E2 | Direction model relied on lagging indicators only (MACD, RSI, ADX) | Model quality |
| E3 | No Tail Risk KPI defined — extreme rally probability unmodeled | Risk coverage |
| W4 | No real-time Kill Switch — position could not be exited on signal | Execution gap |
| W7 | No Tail Risk model in production at time of event | Risk coverage |

The core problem: the system had no mechanism to recognize and respond to an extreme short-squeeze scenario in real time.

---

## System Response

The MERL event became the foundation of DROS v8.4+ safety architecture.

**Defenses added directly in response:**

### 1. Macro Sentiment Veto (Layer 1)
A regime-level directional block. When macro sentiment indicators cross a defined threshold,
Long entries are completely blocked — regardless of other signals.

### 2. Tail Risk Model (Layer 2)
A dedicated model for estimating extreme event probability.
When tail risk exceeds its threshold, all entries are blocked — direction-agnostic.
This would have caught the MERLUSDT short-squeeze scenario.

### 3. Dynamic Kill Switch
Real-time position exit capability triggered by configurable signal conditions.
Addresses the W4 gap: positions can now be exited before liquidation.

### 4. Funding Rate SSOT
`contracts/funding_ssot.py` — Funding rate signal now has a single source of truth
with explicit interpretation contracts. Addresses E1.

### 5. Microstructure Signals
VPIN and OFI integrated into the direction pipeline.
Addresses E2 — direction no longer relies on lagging indicators alone.

---

## What the 7-Layer Gate Looks Like Now

The MERL event was the catalyst for formalizing the entry validation architecture:

```
Entry Request
    ├─ Layer 1: Macro Sentiment Veto       ← MERL defense
    ├─ Layer 2: Tail Risk Veto             ← MERL defense
    ├─ Layer 3: Direction Uncertainty Block ← CLO/USDT defense (Feb 4)
    ├─ Layer 4: Range Extreme Check
    ├─ Layer 5: Toxicity Shield (VPIN)
    ├─ Layer 6: Liquidation Probability Gate
    └─ Layer 7: Card Freshness Gate (AQER)
```

Every layer exists because of a documented failure or risk scenario.

---

## The CLO Event — Two Days Later

On **February 4, 2026**, a second event occurred:

| Item | Detail |
| :--- | :--- |
| **Symbol** | CLO/USDT |
| **p_dir (direction confidence)** | 0.50 — maximum uncertainty |
| **DROS judgment** | SHORT |
| **Actual move** | +24% LONG pump |

The system entered SHORT despite a direction confidence of exactly 50% — statistically equivalent to a coin flip.

**Defense added — Layer 3 (Direction Uncertainty Block):**

Any entry where direction confidence falls within a neutral zone around 50% is now blocked.
The system will not execute when it cannot distinguish signal from noise.

---

## Engineering Philosophy

These events shaped a core DROS principle:

> **"We do not try to predict the unpredictable.  
> We compute what is directly computable — liquidation probability, regime state, microstructure toxicity —  
> and block entries that fail these checks."**

The 43 invariant contracts in DROS production code exist as a direct consequence of this philosophy.
Each contract is a formalized rule that prevents a category of failure from recurring.

---

## Transparency Note

We publish this case study because we believe engineering honesty builds more trust than performance claims.

The MERL event was a real loss from a real production system.  
The response was systematic, documented, and verifiable in the architecture.

---

*→ See [Safety](./safety.md) · [Architecture](./architecture.md) · [Evolution Lab](./evolution-lab.md)*
