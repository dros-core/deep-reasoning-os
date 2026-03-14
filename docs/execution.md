# Execution Layer

Grid-based futures execution with per-level take-profit, dynamic stop-loss, and invariant-enforced position management.

---

## Grid Architecture

DROS operates a bidirectional grid around an entry price. Two modes:

| Mode | Grids | Use Case |
|------|-------|----------|
| Turbo | 10-20 | High-frequency fill environments |
| Steady | 16-32 | Stable trending regimes |

**Minimum grids**: N_total >= 10 (FAIL_N_BELOW_10 at P1)

**Spacing**: spacing_dec >= k_floor x (fees_dec + slippage_dec). The k_floor multiplier ensures grid revenue covers execution costs at minimum fill rate.

Spacing is computed exclusively by `SpacingOracleSSOT.compute_spacing()`. Any spacing computation outside this oracle triggers FAIL_SPACING_SSOT_MISMATCH.

---

## Per-Level Take-Profit (PLT)

Each grid level has an independent reduce-only GTC take-profit order.

**INVARIANT-PLT-01**: All PLT orders are reduce_only=True, GTC, qty=fill_qty, price=fill x (1 +/- spacing)
**INVARIANT-PLT-02**: On position close, exchange cancellation AND PerLevelTPStore.cancel_all_active() must execute simultaneously
**INVARIANT-PLT-03**: If has_active_orders(symbol)==True, aggregate TP is suppressed

PLT orders cannot be batched (INVARIANT-AQER-05). Each level's order is placed individually.

---

## Dynamic Stop-Loss (3-Stack)

### Stack A: Catastrophe Stop
- Always active, in all SL_MODE states
- Fires at liq_buffer_pct floor:
  - liq_buffer_pct <= 3%: floor 0.8%
  - liq_buffer_pct <= 6%: floor 1.5%
- Cannot be disabled

### Stack B: Chandelier Trail
- Regime-adaptive multipliers:
  - ranging: 2.5x ATR (minimum: 2.0 x (1 + 0.8 x (1-ER)))
  - trending: 1.5x ATR
  - volatile: 3.0x ATR
- Only active in SL_MODE=STRICT
- Fixed 2.0x multiplier is banned

### Stack C: Grid-Fail SL
- 3-factor simultaneous trigger:
  1. hours_no_fill >= threshold
  2. ADX >= 25
  3. adverse_drift detected
- Feature flag: USE_GRID_FAIL_SL

### SL Mode
```
STRICT         -- all stacks active (default)
CONTROLLED_OFF -- B+C suppressed, A preserved, 2h TTL, audit log
BREAKGLASS_OFF -- symbol-scoped, TTL required, operator_reason required
```

All STOP_MARKET orders must be placed via `place_protective_stop()` wrapper. Direct API calls trigger FAIL_SL_ORDER_BYPASS.

---

## Position State Machine

Positions transition through three defined states:

```
ENTRY_CANDIDATE  (pre-entry, after 8-gate validation)
      |
      v
MANAGED_POSITION (active position, 30-min refresh cycle)
      |
      v
[exit] -> deregistered
```

Non-card positions detected post-execution become EXTERNAL_ORPHAN:
- TP-only exit (no SL, no rotation pressure)
- Forced full close triggers FAIL_ORPHAN_FORCED_CLOSE (P0)
- Scale-in triggers FAIL_NCO_SCALE_IN_ATTEMPTED (P0)

All positions registered in PositionStateMachine before first order placement.

---

## MANAGED_POSITION Refresh

Active positions are refreshed every 30 minutes plus event-triggered:

**Event triggers**:
- ADWIN change detection on fill rate
- ATR expansion > threshold
- Direction confidence drop

Refresh re-evaluates card quality, updates SL levels, and checks regime transitions. Card is not recreated -- only parameters are updated within the signed CardSpec bounds.

---

## ORPHAN Position Handling

Non-card positions (opened outside DROS, or pre-existing at startup) are classified as EXTERNAL_ORPHAN.

**Rules**:
- Exempt from no_fills_30min rotation
- SL banned (TP-only exit path)
- Scale-in absolutely forbidden
- Forced liquidation absolutely forbidden
- Must persist in _non_card_orphan_symbols through DaemonCheckpoint

ORPHAN positions are managed to natural exit only. They are never force-closed.

---

## Slot Management

Available trading slots determined by `_positioned_symbols()`. Using `len(_symbols)` is banned (INVARIANT-SLOT-02).

**Slot suppression**: If NCO margin >= 50% of equity, `_fill_open_slots()` is suppressed to prevent over-leverage.

**Restart behavior**:
- STARTUP_SLOT_EXPANSION_DELAY_S (default 300s): no new entries for 5 minutes after restart
- DynAlloc trim on startup uses _positioned_symbols() count

**SL exit refill**: After a position closes via SL, slot refill is queued through `_slot_fill_requests` (direct `_fill_open_slots()` call triggers FAIL_SL_DIRECT_FILL).

---

## Reduce-Only Orders

Reduce-only levels always execute regardless of margin backoff (INVARIANT-MARGIN-01). Margin pressure cannot block a reduce-only order -- blocking triggers FAIL_REDUCE_ONLY_BLOCKED_BY_MARGIN_BACKOFF.

**Price constraints**:
- SHORT BUY price must not exceed entry_price (INVARIANT-MARGIN-09)
- LONG SELL price must not be below entry_price (INVARIANT-MARGIN-10)

---

## Spacing Rebuild

`SpacingOracleSSOT.compute_spacing()` determines when to rebuild:
- Requires `should_rebuild()` check first
- 10% hysteresis prevents oscillation
- 30-minute cooldown between rebuilds
- Rebuilding without these checks triggers FAIL_REBUILD_THRASH

Yang-Zhang volatility is used as sigma_d estimator. ATR-only sigma_d estimation is banned (INVARIANT-SPACING-04).

---

## Margin Safety

Before any margin release, AOSM performs bisection search to verify liq_distance is safe (INVARIANT-AOSM-01). Unsafe margin release triggers FAIL_AOSM_UNSAFE_MARGIN_RELEASE.

In Binance hedge mode, positionSide must be specified in all positionMargin requests (INVARIANT-AOSM-03).

N* Dithering uses `hash(epoch_hour, symbol)` for deterministic slot count variation. `random()` is banned (INVARIANT-AOSM-07).

---

## Hot Reload

The daemon supports configuration hot-reload via SIGHUP:
- `importlib.reload()` is banned (FAIL_IMPORT_RELOAD)
- Checkpoint writes are atomic: write -> fsync -> rename (FAIL_CHECKPOINT_SAVE on failure)
- SharedMemory uses track=False (FAIL_SHM_ATTACH on failure)
- Module-level `os.environ.get()` is banned (FAIL_COLD_ENV_VAR)

---

## AQER (Adaptive Queue Execution and Reconciliation)

- **StaleGate**: card_age > 5400s triggers FAIL_AQER_STALE_ENTRY (blocks entry)
- **StickyWindow**: Retained card ID exempts from reconciler ORPHAN cancellation
- **PLT batch ban**: PLT orders placed individually, never batched

---

## API Rate Limits

- Weight budget: 800/1200 (66% ceiling)
- Max concurrent requests: 5
- Circuit Breaker: 429/418 response -> 5-minute backoff
- Cache-first: API calls minimized through local state

---

*Execution documentation for DROS v12.4b | 2026-03-14*
