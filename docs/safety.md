# Safety Architecture

DROS enforces safety through hard-coded invariants with named fail codes. These are not warnings or soft limits — violations trigger immediate halt and structured logging.

---

## 8-Layer Safety Gate

Every candidate position passes through eight sequential layers. No layer can be bypassed or reordered.

| Layer | Name | Rule |
|-------|------|------|
| 1 | Macro Sentiment Gate | PSI <= -0.5 blocks Long entry |
| 2 | Tail Risk Gate | tail_risk >= 0.8 blocks entry |
| 3 | Net ROI Gate | net_roi_expected < -5% blocks entry; < 0% triggers shadow log |
| 4 | Direction Gate | Asymmetric thresholds: Long p_dir >= 0.58, Short p_dir <= 0.42 |
| 5 | Entry Gate | 5-check validation (macro, direction, cost, quality, alignment) |
| 6 | Card Quality Gate | quality_base_no_llm >= 85 required |
| 7 | Cost Gate | >= 90% pass rate required |
| 8 | Position State Gate | PositionStateMachine registration required |

**Hard Gate First**: PSI/tail_risk/net_roi are evaluated before LLM soft scoring. LLM adjustment is zeroed if hard gate conditions are violated (LLM Hard-Safety Guard).

---

## Unit Contract

All values use `_dec` units. The `_pct` suffix is permanently banned.

```
CORRECT:   spacing_dec = 0.005   (0.5%)
FORBIDDEN: spacing_pct = 0.5     -> FAIL_UNIT_PCT (P0 immediate halt)
```

This applies across all agents, all files, all contexts. No exceptions.

---

## Fail Codes

### P0 — Immediate Halt

| Code | Trigger |
|------|---------|
| FAIL_UNIT_PCT | _pct unit detected |
| FAIL_CARDSPEC_MUTATION | Signed CardSpec modified after A9 signature |
| FAIL_ORPHAN_FORCED_CLOSE | Forced full liquidation of orphan position attempted |
| FAIL_NCO_SCALE_IN_ATTEMPTED | Scale-in attempted on non-card orphan |
| FAIL_SPACING_SSOT_MISMATCH | Spacing computed outside SpacingOracleSSOT |
| FAIL_REDUCE_ONLY_BLOCKED_BY_MARGIN_BACKOFF | Reduce-only level blocked by margin backoff |
| FAIL_PRESET_PATH_MISMATCH | learned_presets path not via LearningArtifactRegistry |
| FAIL_LLM_DIRECT_CALL | Direct Ollama call bypassing LLM Broker |
| FAIL_IMPORT_RELOAD | importlib.reload() detected |
| FAIL_SL_DISABLED | SL_ENABLED=0 global off attempted |
| FAIL_SL_CAT_DISABLED | Catastrophe Stop (Stack A) disabled |
| FAIL_SL_ORDER_BYPASS | STOP_MARKET placed without place_protective_stop() wrapper |

### P1 — Learning/Execution Halt

| Code | Trigger |
|------|---------|
| FAIL_N_BELOW_10 | N_total < 10 grids |
| FAIL_SPACING_TOO_TIGHT | spacing_dec < k_floor x (fees_dec + slippage_dec) |
| FAIL_HASH_MISMATCH | sha256(code+contracts+cardspec) mismatch |
| FAIL_MACRO_SENTIMENT_VETO | PSI veto overridden |
| FAIL_TAIL_RISK_VETO | tail_risk >= 0.8 entry allowed |
| FAIL_PSI_ABSOLUTE_VETO | PSI absolute veto bypassed |
| FAIL_NEGATIVE_ROI_ENTRY | net_roi_expected < -5% entry allowed |
| FAIL_POSITION_UNREGISTERED | Position opened without PSM registration |
| FAIL_INJECT_NON_CARD_ORIGIN | VCA inject without card_origin=True |
| FAIL_SL_LIQ_TOO_CLOSE | liq_buffer_pct <= 6% without Stack A tightening |
| FAIL_SL_BREAKGLASS_NOTT | BREAKGLASS_OFF without TTL + operator_reason |
| FAIL_REBUILD_THRASH | Spacing rebuild without 10% hysteresis + 30min cooldown |
| FAIL_AOSM_UNSAFE_MARGIN_RELEASE | Margin released without liq_distance bisection check |
| FAIL_ROI_HISTORY_LOST | roi_history not persisted to SQLite |
| FAIL_PBO_OVERFIT | PBO >= 0.3 in backtest |
| FAIL_TS_BINARY_SUCCESS | Thompson success based on exit-type, not net_roi |

### P1 — Marketing Halt

| Code | Trigger |
|------|---------|
| FAIL_MKT_RAW_PAYLOAD | Raw payload (check_id, condition, layer) passed to LLM |
| FAIL_MKT_FORBIDDEN_POSTURE | Posture value outside VALID_POSTURES frozenset |
| FAIL_MKT_HOOK_SCOPE_MISMATCH | Hook inserted outside correct variable scope |
| FAIL_MKT_CONFIRMED_NO_WATCH | SYMBOL_CONFIRMED fired from Check4/5 (WATCH only allowed) |
| FAIL_MKT_FLAT_DAILY_MAX | rate_limiter DAILY_MAX directly modified |
| FAIL_MKT_EXPIRED_HIDDEN | EXPIRED watch result suppressed |
| FAIL_MKT_LAYER_VIOLATION | execution emitter re-called from marketing layer |
| FAIL_MKT_SCHEDULE_LOST | asyncio.call_later() used instead of SQLite schedule |

### Direction Entry Codes

| Code | Trigger |
|------|---------|
| FAIL_STALE_ENTRY_ADX | _fetch_live_indicators() missing ADX or ER |
| FAIL_LIVECHECKAND_CONDITION | LiveDirectionCheck uses AND (OR required) |
| FAIL_DEAD_ZONE_NOT_SSOT | Dead zone not referencing direction_safety_contracts.py |
| FAIL_ADX_SLOPE_HARDCODED | adx_slope=0.0 hardcoded instead of cache-computed |

### Evolution Lab Codes

| Code | Trigger |
|------|---------|
| FAIL_EVOL_POST_HOC_HYPOTHESIS | Hypothesis registered after testing |
| FAIL_EVOL_DIRECT_DEPLOY | Evolved strategy deployed without shadow stage |
| FAIL_EVOL_FIXED_MUTATION | Mutation rate hardcoded |
| FAIL_EVOL_SINGLE_DETECTOR | Black Swan fired by single detector |
| FAIL_EVOL_OODA_LIVE_DECIDE | OODA Decide/Act during live market hours |
| FAIL_EVOL_EVENT_MUTATION | EventStore modified outside append/prune |
| FAIL_EVOL_MLX_NO_LOCK | mx.eval() called without _gpu_lock |
| FAIL_EVOL_FORK_MLX | MLX subprocess using fork context |

---

## Invariant Classes

### INVARIANT-ORPHAN
- INVARIANT-ORPHAN-01: CardPositionRegistry registration required
- INVARIANT-ORPHAN-02: ORPHAN exempt from no_fills rotation; SL banned; TP-only exit
- INVARIANT-ORPHAN-07: Forced full liquidation absolutely forbidden (P0)

### INVARIANT-SLOT
- INVARIANT-SLOT-02: Slot count = _positioned_symbols() (not len(_symbols))
- INVARIANT-SLOT-04: _non_card_orphan_symbols persisted to DaemonCheckpoint
- INVARIANT-SLOT-05: NCO margin >= 50% equity suppresses _fill_open_slots()

### INVARIANT-MARGIN
- INVARIANT-MARGIN-01: reduce_only levels always execute (margin backoff cannot block)
- INVARIANT-MARGIN-09: SHORT BUY price must not exceed entry_price
- INVARIANT-MARGIN-10: LONG SELL price must not be below entry_price

### INVARIANT-SL
- INVARIANT-SL-01: SL_ENABLED=0 auto-maps to CONTROLLED_OFF (never fully off)
- INVARIANT-SL-02: CONTROLLED_OFF preserves Stack A (Catastrophe Stop)
- INVARIANT-SL-03: BREAKGLASS_OFF requires symbol-scope + TTL + operator_reason
- INVARIANT-SL-04: liq_buffer_pct <= 6% forces Stack A tight floor
- INVARIANT-SL-05: All STOP_MARKET orders via place_protective_stop() wrapper

### INVARIANT-BOOTSTRAP
- INVARIANT-BOOTSTRAP-01: Unlock requires quality_base_no_llm (LLM boost excluded)
- INVARIANT-BOOTSTRAP-02: Shadow mode threshold changes absolutely forbidden
- INVARIANT-BOOTSTRAP-03: SL exit refill via queued trigger only
- INVARIANT-BOOTSTRAP-04: 5-condition simultaneous unlock required
- INVARIANT-BOOTSTRAP-05: F4 CASCADE and ReliabilityGate are independent components
- INVARIANT-BOOTSTRAP-06: BootstrapActionBucket persisted to DaemonCheckpoint

### INVARIANT-MACRO
- INVARIANT-MACRO-01: FGI for Macro Tempo Gate only; direct p_dir_blended modification banned
- INVARIANT-MACRO-02: PSI veto cannot be reversed by FGI recovery
- INVARIANT-MACRO-03: FGI returns bucket forecast (1-3d/4-7d/8-14d/>14d), not single ETA

### INVARIANT-CB
- INVARIANT-CB-01: Rotation budget and Slot Expansion budget share no timestamps
- INVARIANT-CB-02: no_fills_30min is one factor of five; standalone binary trigger banned
- INVARIANT-CB-03: Reliability CB blocks new entries only; reconcile/SL/safety unaffected

---

## Position State Machine

Three states with defined transitions:

```
ENTRY_CANDIDATE
    |-- (all 8 gates pass) --> MANAGED_POSITION
    `-- (any gate fails)   --> rejected (no position)

MANAGED_POSITION
    |-- (exit trigger)     --> deregistered
    `-- (orphan detected)  --> EXTERNAL_ORPHAN

EXTERNAL_ORPHAN
    |-- TP-only exit        --> deregistered
    `-- forced close        --> FAIL_ORPHAN_FORCED_CLOSE (P0)
```

All positions must be registered in PositionStateMachine before first order placement. Unregistered positions trigger FAIL_POSITION_UNREGISTERED.

---

## Dynamic Stop-Loss Architecture

### Stack A: Catastrophe Stop (Always Active)
- Active in all SL_MODE states including CONTROLLED_OFF
- Fires at liq_buffer_pct floor
- Cannot be disabled by any flag

### Stack B: Chandelier Trail (Regime-Adaptive)
- ranging: 2.5x ATR (ER-adaptive floor: 2.0 x (1 + 0.8 x (1-ER)))
- trending: 1.5x ATR
- volatile: 3.0x ATR
- Only active when SL_MODE=STRICT

### Stack C: Grid-Fail SL (3-Factor Trigger)
- hours_no_fill >= threshold
- ADX >= 25 (trend confirmation)
- adverse_drift detected
- All three required simultaneously
- Feature flag: USE_GRID_FAIL_SL

### SL Mode Transitions
```
STRICT          -- default, all stacks active
CONTROLLED_OFF  -- Stack B+C suppressed, Stack A preserved, TTL 2h, audit log required
BREAKGLASS_OFF  -- symbol-scoped only, TTL required, operator_reason required
```

---

*Safety documentation for DROS v12.4b | 2026-03-14*
