# ⚡ Execution Engine — Deep Reasoning OS (DROS)

> v11 execution engine, AQER entry quality router, AOSM capital management

---

## Overview

The execution layer is the operational core of DROS — responsible for converting validated CardSpecs into live positions, managing capital allocation, and maintaining grid integrity around the clock.

**Core file:** `execution/daemon.py` — production daemon (~11,000 lines)

---

## v11 Feature Timeline

| Version | Feature | Description |
|---------|---------|-------------|
| v11.0 | Game Theory | VPIN + OFI + CFR + Stealth execution |
| v11.6 | NCO GhostCard | Non-Card Orphan grid management |
| v11.8 | Vampire Capital + Microstructure | Capital reallocation + CfC sensor |
| v11.14 | PLT Cancel Fix | `preserve_plt=True` parameter |
| v11.15 | AOSM v2 | 3-engine Adaptive Orphan Slot Manager |
| v11.16 | PLT Recover Entry Guard | SHORT BUY tp>entry blocked |
| v11.17 | NCO Inventory Pressure | A-S inventory pressure spacing |
| v11.18 | CardRegistry type fix | set→dict type mismatch defense |
| v11.19 | PLT-Reconciler 2-Layer Exempt | Infinite loop fix |
| v11.20 | N* 60s TTL Cache | `_get_n_star_cached()` |

---

## Reconciler — 5-Step Process

Every cycle, the Reconciler validates live exchange state against internal records:

```
Step 1: Collect live positions from exchange
Step 2: Cross-reference CardPositionRegistry
Step 3: Check PLT exemption (cid pattern + PerLevelTPStore DB)
Step 4: Check AQER StickyWindow exemption
Step 5: Handle mismatches (ORPHAN registration / order cancellation)
```

---

## AQER — Active-Queue Entry Router

Based on the insight that Binance `PUT /fapi/v1/order` (modify) **re-queues orders**, losing priority.

> Academic basis: Albers et al. 2025 — fill probability correlates positively with adverse selection risk  
> "Well-filled" ≠ "Good fill" → nearest-K maximization is suboptimal.

### 3-Phase Architecture

| Phase | Feature | Status |
|-------|---------|--------|
| P0 | StaleGate (card freshness) | Production |
| P1 | StickyWindow (queue preservation) | Production |
| P2 | BatchOrderRouter (single TCP batch) | Shadow |

### StaleGate — 3 States

```
GREEN  →  Fresh card → normal entry allowed
YELLOW →  Aging card → slot expansion blocked
RED    →  Stale card → _fill_open_slots() immediate return
           → FAIL_AQER_STALE_ENTRY (P0)
```

### StickyWindow — 4-Condition Queue Preservation

Prevents unnecessary order cancel + requeue cycles that lose queue priority.  
Orders are kept in place unless all 4 replacement conditions are met.

### BatchOrderRouter

Groups multiple order placements into a single TCP connection.  
Reduces API weight consumption and latency variance.

---

## AOSM v2 — Adaptive Orphan Slot Manager

**Problem:** NCO (Non-Card Orphan) positions locked 92% of equity → 0 new entries for 30 days.

### 3-Engine Structure

| Engine | Name | Role |
|--------|------|------|
| A | CapitalUnlockController | Binary-search margin release + order budget K≤8 |
| B | DeterministicNStarDitherer | Hash-based deterministic N* dithering |
| C | BetaGatedHedgeSlots | Correlation/beta-gated hedge discount |

### Margin Release Formula

```
liq (Short) ≈ entry + remaining_margin / qty
liq (Long)  ≈ entry − remaining_margin / qty

liq_distance = |mark_price − liq_price| / mark_price

Safety condition:
  liq_distance ≥ AOSM_MIN_LIQ_DISTANCE_K × σ_1h

Violation → FAIL_AOSM_UNSAFE_MARGIN_RELEASE (P1)
```

### N* Dithering

```python
# Deterministic (not random) — reproducible across restarts
dither_offset = hash(epoch_hour, symbol) % dither_range
N_star_dithered = N_star_base + dither_offset

# random() forbidden → FAIL_AOSM_DITHER_RANDOM
```

---

## ORPHAN Position Management

Positions not tracked by an active CardSpec require special handling:

```
INVARIANT-ORPHAN-02:
  no_fills_30min → rotation exempt
  SL forbidden
  TP-only exit allowed

INVARIANT-ORPHAN-07:
  Forced full liquidation ABSOLUTELY FORBIDDEN
  → FAIL_ORPHAN_FORCED_CLOSE (P0)

INVARIANT-NCO-09:
  Scale-In ABSOLUTELY FORBIDDEN
  → FAIL_NCO_SCALE_IN_ATTEMPTED (P0)

INVARIANT-NCO-07:
  drag_reversion_ratio > 5.0 → MANUAL_REVIEW
  drag_reversion_ratio > 2.0 → EXIT_ACCELERATE
```

---

## PLT — Per-Level Take Profit

TP orders placed at each individual grid level:

```
INVARIANT-PLT-01:
  reduce_only=True, GTC, qty=fill_qty, price=fill×(1±spacing)

INVARIANT-PLT-02:
  On symbol exit: cancel at exchange AND PerLevelTPStore.cancel_all_active() simultaneously

INVARIANT-PLT-03:
  has_active_orders(symbol)==True → aggregate TP suppressed

INVARIANT-MARGIN-09: SHORT BUY order price > entry_price → FORBIDDEN
INVARIANT-MARGIN-10: LONG SELL order price < entry_price → FORBIDDEN
```

---

## Vampire Capital Allocator

```
Strategy: Extract margin from stagnant positions,
          reallocate to new high-potential opportunities.
          → Minimize idle capital, maximize active slot usage.

Module: execution/cvw_allocator.py (252 lines — pure functions)
        Extracted from daemon.py (−169 lines)
```

---

## Hot-Reload & Checkpoint Safety

```
Forbidden:  importlib.reload() → FAIL_IMPORT_RELOAD
            (CPython #126548: stale references, thread-unsafe)

Atomic checkpoint:
  write → fsync → os.replace()  (APFS atomicity guaranteed)

SharedMemory:
  track=False  (cross-process persistence required)

SQLite connections:
  timeout=30, PRAGMA busy_timeout=5000
  (prevents hang when zombie pytest holds .db-shm lock)
```

---

## Daemon Architecture

```
launchd managed processes:

com.deep-reasoning-os.continuous.plist        → run_forever() (card generation)
com.deep-reasoning-os.continuous-hybrid.plist → --single-cycle (15-min interval)
com.deep-reasoning-os.execution-daemon.plist  → execution daemon (order management)
com.deep-reasoning-os.autonomous-learning.plist → learning daemon
com.deep-reasoning-os.card-lifecycle.plist    → card lifecycle manager
```

---

## API Rate Limit Management

```
Weight limit:    800 / 1,200  (66% cap)
Per-second:      max 5 requests
Circuit Breaker: 429 / 418 error → 5-minute backoff
Cache-First:     minimize direct API calls

SharedRateLimitBudget: all agents route through shared budget
```

---

## Fork Safety (Apple Silicon)

```python
# Prevents EXC_BREAKPOINT crash on ARM64 M4 Pro
# Core fix: execution/__main__.py
multiprocessing.set_start_method("spawn")  # fork → spawn
```

---

*→ See [Architecture](./architecture.md) · [Safety](./safety.md) · [Evolution Lab](./evolution-lab.md)*
