# DROS — Deep Reasoning OS

**Systematic futures trading infrastructure for Binance.** Observations registered before outcomes, scored after. All states published.

[![Version](https://img.shields.io/badge/version-v12.4b-blue)](ROADMAP.md)
[![Safety](https://img.shields.io/badge/safety-8--layer%20gate-green)](#safety-architecture)
[![Tests](https://img.shields.io/badge/tests-500%2B-brightgreen)](#validation)
[![License](https://img.shields.io/badge/license-proprietary-lightgrey)](#license)

---

## What DROS Does

DROS is a closed-loop autonomous trading system operating on Binance USDT-M futures. It does not predict the future. It registers systematic observations before outcomes, scores them against realized results, and publishes all four states — confirmed, expired, contained, and protected.

**Core loop**: Observe → Gate → Execute → Score → Evolve

Every decision passes through eight sequential safety gates. No gate can be bypassed. Every fail code triggers an immediate halt with structured logging.

---

## How It Works

### 1. Register
System detects market conditions through five-axis sentiment analysis, directional signals, and regime classification. A candidate card is filed with timestamp before any position opens.

### 2. Gate
Eight sequential layers validate the candidate:
- PSI/Macro Sentiment Gate
- Tail Risk Gate
- Net ROI Gate (net_roi_expected < -5% blocks entry)
- Direction Gate (asymmetric thresholds: Long >=0.58, Short <=0.42)
- Entry Gate (5-check validation)
- Card Quality Gate (>=85 score required)
- Cost Gate (>=90% pass rate required)
- Position State Gate

### 3. Execute
Approved positions enter via grid execution with per-level take-profit orders. Dynamic stop-loss runs three stacked layers — Catastrophe Stop always active, Chandelier Trail regime-adaptive, Grid-Fail SL on execution degradation.

### 4. Score
Outcomes are observed and scored against pre-registered observations. Lead times, confirmation rates, and expiry counts are all published. Nothing is suppressed.

### 5. Evolve
Leviathan v4.0 (formerly AI Evolution Lab) runs shadow experiments — new strategies start in shadow, graduate through Sequential Probability Analysis to canary, then production. No direct deployment.

---

## Architecture Overview

```
Market Data
    |
    v
[5-Brain Perception Layer]
    |-- Macro Sentiment (PSI, FGI, funding)
    |-- Direction Engine (ADX, ER, RSI ensemble)
    |-- Microstructure (VPIN, order flow)
    |-- Regime Classifier (ranging/trending/volatile)
    `-- Symbol Sentiment (5-axis: Crowding/Participation/Aggression/SmartMoney/Execution)
    |
    v
[8-Layer Safety Gate]          -- hard enforcement, no bypass
    |
    v
[Grid Execution Engine]
    |-- Per-Level TP (reduce_only GTC orders)
    |-- Dynamic SL 3-Stack (A/B/C)
    |-- Position State Machine (ENTRY_CANDIDATE -> MANAGED_POSITION)
    |-- ORPHAN handler (TP-only, no forced close)
    |
    v
[Learning Pipeline]
    |-- Thompson Sampling (regime-aware, 30-min throttle)
    |-- AWR Grid Agent (off-policy, replay buffer)
    |-- SparseSafeCalibrator (< 50 samples: Temperature Scaling)
    |-- BLS (subprocess-isolated, 6GB gate)
    |
    v
[Leviathan v4.0 Shadow Ring]   -- all experiments shadow-first
    |-- Ring 1: Production strategies
    `-- Ring 2: Shadow candidates (SPA p<0.01 graduation gate)
```

---

## Safety Architecture

50+ named fail codes. 100+ feature flags. All safety checks are hard stops, not warnings.

**P0 — Immediate halt:**
`FAIL_UNIT_PCT` - `_pct` unit used instead of `_dec`
`FAIL_CARDSPEC_MUTATION` - signed card modified
`FAIL_ORPHAN_FORCED_CLOSE` - forced full liquidation attempted
`FAIL_NCO_SCALE_IN_ATTEMPTED` - scale-in on non-card orphan
`FAIL_SL_DISABLED` - global SL switched off
`FAIL_SL_ORDER_BYPASS` - stop order placed without wrapper

**P1 — Learning/execution halt:**
`FAIL_PSI_ABSOLUTE_VETO` - PSI veto overridden
`FAIL_NEGATIVE_ROI_ENTRY` - negative expected ROI entry
`FAIL_POSITION_UNREGISTERED` - position opened without PSM registration
`FAIL_SPACING_SSOT_MISMATCH` - spacing computed outside oracle

Full fail code reference: [docs/safety.md](docs/safety.md)

---

## Pre-Alert & Marketing System

DROS operates a transparent observation feed. Signals are registered **before** outcomes, confirmed or expired **after**, and published to all channels simultaneously.

**Signal lifecycle:**
```
SYMBOL_WATCH (filed) -> SYMBOL_CONFIRMED / SYMBOL_WATCH_EXPIRED
                     -> SYMBOL_ACTION_PROOF (30-min delayed proof)
```

All four states published. EXPIRED signals are never suppressed (Numerai transparency model).

Channels: X (proof distribution), Telegram Public (observation feed), Telegram Private (operator terminal).

---

## Disclaimer

Past performance does not guarantee future results. This system is not investment advice. All observations are systematic, timestamp-verified, and scored against realized outcomes. No returns are guaranteed or implied.

DROS operates within Binance exchange risk parameters. Users are responsible for their own capital allocation decisions.

---

## Repository Structure

```
docs/
  architecture.md       System design and component map
  safety.md             Fail codes, invariants, and enforcement
  execution.md          Grid execution, SL/TP, position management
  agents.md             Agent pipeline (A0-A15)
  learning.md           ML pipeline, Thompson, AWR, calibration
  evolution-lab.md      Leviathan v4.0 shadow/production ring system
  validation.md         Test infrastructure and contract testing
  performance.md        KPI baselines and benchmark results
  case-study-merl.md    MERL liquidation post-mortem and defense
  faq.md                Common questions
  open-architecture.md  Extension points and integration guide
  academic-references.md Research foundations
ROADMAP.md              Version history and planned features
```

---

*DROS v12.4b | Leviathan v4.0 | Marketing v6.2 | 2026-03-14*
