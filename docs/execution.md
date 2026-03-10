# ⚡ Execution Engine — Deep Reasoning OS (DROS)

The execution layer translates validated CardSpecs into live exchange orders with microstructure-aware timing, atomic state management, and invariant enforcement.

---

## Core Architecture

The production execution core is a single-process asyncio daemon coordinating all runtime components: order management, position tracking, safety gates, learning feedback, and hot-reload checkpointing.

**Design principles**:
- Single source of truth for every parameter (SSOT)
- Atomic checkpoint writes (write → fsync → rename — no partial state)
- No `importlib.reload()` — prevents state corruption on hot-reload
- All LLM calls routed through broker with timeout and deterministic fallback

---

## Reconciler — 5-Step Process

At each heartbeat, the reconciler ensures live exchange state matches expected state:

1. Collect live positions from exchange
2. Cross-reference CardPositionRegistry
3. Check PLT (Per-Level Take Profit) exemption
4. Check AQER StickyWindow exemption
5. Handle mismatches (ORPHAN detection, stale order cancellation)

---

## AQER — Active-Queue Entry Router

Three-phase architecture ensuring entry quality:

| Phase | Name | Role |
|:---:|:---|:---|
| P0 | **StaleGate** | Card freshness validation — GREEN / YELLOW / RED states |
| P1 | **StickyWindow** | Queue preservation via 4-condition filter |
| P2 | **BatchOrderRouter** | Groups compatible orders into single TCP batch (shadow phase) |

---

## AOSM — Adaptive Orphan Slot Manager

Manages capital allocation across orphan and active card positions:

| Engine | Name | Role |
|:---:|:---|:---|
| A | **CapitalUnlockController** | Binary-search margin release with liquidation distance validation |
| B | **DeterministicNStarDitherer** | Hash-based N* variation (deterministic, no `random()`) |
| C | **BetaGatedHedgeSlots** | Correlation and beta-gated hedge slot allocation |

---

## ORPHAN Position Management

Positions that outlive their CardSpec (rotation, restart) enter ORPHAN state:

- **No forced liquidation** (P0 — `FAIL_ORPHAN_FORCED_CLOSE`)
- **No scale-in** (P0 — `FAIL_NCO_SCALE_IN_ATTEMPTED`)
- **Exit only via TP** — stop-loss forbidden
- Drag ratio monitoring triggers escalating alerts and accelerated exit

---

## PLT — Per-Level Take Profit

Take-profit orders placed at each individual grid level:

- `reduce_only=True`, GTC order type
- Quantity equals the fill quantity at that level
- Price set at fill price ± spacing
- Active PLT orders suppress aggregate TP to prevent double-exit

---

## Microstructure-Aware Execution

Order timing adapts to detected market microstructure signals:

- Informed-flow detection (VPIN-based toxicity scoring)
- Order jitter to reduce adverse selection
- Game-theoretic defense against front-running patterns

---

## Hot-Reload & Checkpoint Safety

| Rule | Detail |
|:---|:---|
| `importlib.reload()` forbidden | `FAIL_IMPORT_RELOAD` — prevents module state corruption |
| Atomic checkpoint | write → fsync → rename sequence |
| SharedMemory | `track=False` required — `FAIL_SHM_ATTACH` |
| Cold env vars | Module-level `os.environ.get()` forbidden — `FAIL_COLD_ENV_VAR` |

---

## Staged Rollout

```
Shadow → Canary (10%) → Production
```

All new execution features begin in shadow mode (full validation, zero capital). Graduation requires SPA statistical test (p < 0.01) and minimum 7-day shadow period.
