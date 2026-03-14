# Validation and Testing

DROS maintains 500+ tests covering contract enforcement, integration pipelines, and property-based invariant checking.

---

## Test Infrastructure

### Test Runner

All tests run via `dros-test-runner` agent (haiku model, background):

**Trigger conditions**:
- Any modification to execution/, contracts/, services/, or utils/
- After bug fixes or new feature implementations
- After any agent pipeline change

**Output**: Failures only (500+ tests, pass cases suppressed for readability)

---

## Test Categories

### Contract Tests (Unit Level)

Each fail code has a dedicated unit test. Example structure:

```
test_FAIL_UNIT_PCT         -- _pct unit blocked
test_FAIL_CARDSPEC_MUTATION -- signed card immutable
test_FAIL_SL_DISABLED      -- SL fully-off blocked
test_FAIL_LIVECHECKAND     -- AND condition blocked in LiveCheck
test_FAIL_ORPHAN_FORCED    -- forced liquidation blocked
```

Contract tests verify that invariant violations halt execution and log structured output. They do not test business logic -- only enforcement.

### Integration Tests (Pipeline Level)

End-to-end pipeline tests that simulate full A0-A15 execution:

- **Single symbol entry**: BTCUSDT turbo mode, validates card_created flag
- **8-gate validation**: Each gate independently vetoed, verifies block
- **SL trigger**: Stack A fires at catastrophe threshold
- **PLT coordination**: PLT orders placed and cancelled on position close
- **ORPHAN detection**: Non-card position classified correctly
- **Reconciler cycle**: Drift detection and resolution

### Property-Based Tests (Hypothesis)

Hypothesis library tests for invariant properties:

- spacing_dec always >= k_floor x (fees_dec + slippage_dec)
- N_total always >= 10
- p_dir always in [0, 1] after calibration
- net_roi always uses _dec unit
- All PLT orders have reduce_only=True
- quality_base_no_llm always <= quality_final

Property tests use example databases in .hypothesis/constants/ for reproducible shrinking.

### Learning Pipeline Tests

- CPCV purge gap verification (>= 24h enforced)
- PBO threshold gate (>= 0.3 blocks)
- Thompson update regime_id requirement
- SparseSafeCalibrator method selection by sample count
- AWR weight explosion detection (max/mean > 100)

### Marketing Tests

- event_normalizer: blocked keys not in LLM payload
- post_cleaner: hard_block_check fires on banned phrases
- channel_tone_consistency: emoji counts, CTA rules, operator phrases
- repetition_guard: Jaccard > 0.65 blocks
- VALID_POSTURES: invalid posture value blocked
- EXPIRED watch: never suppressed

---

## Validation Gates

### Pre-execution Checklist

Before any production deployment:

```
[ ] N_total >= 10 (FAIL_N_BELOW_10)
[ ] spacing_dec >= k_floor x (fees_dec + slippage_dec)
[ ] No _pct units in any agent output (FAIL_UNIT_PCT)
[ ] CardSpec STRICT mode (FAIL_CARDSPEC_MUTATION)
[ ] SL_MODE != disabled (FAIL_SL_DISABLED)
[ ] All PLT orders reduce_only=True (INVARIANT-PLT-01)
```

### Post-execution Checklist

After every trading session:

```
[ ] Cost Gate pass rate >= 90%
[ ] Negative ROI entries = 0
[ ] Live-Backtest divergence <= 10%
[ ] Telemetry persisted (roi_snapshots, roi_events)
[ ] No FAIL_* codes in daemon logs
[ ] Reconciler drift ratio < 50%
```

---

## Hash Verification

Every CardSpec is verified with sha256(code + contracts + cardspec):

- Hash computed at A9 signature
- Verified at every downstream agent access
- Mismatch triggers FAIL_HASH_MISMATCH (P1 halt)

This prevents parameter drift between signature and execution.

---

## Shadow Validation

New features run in shadow mode before production:

**Shadow = would-have logging only**:
- No capital at risk
- All gate decisions logged with "would-have" prefix
- Metrics computed against shadow decisions
- Production gate decisions unchanged

Shadow promotion criteria (example: Direction Revalidation Enhancer):
- conflict_precision >= 60%
- false_positive_rate <= 20%
- events >= 50
- shadow_days >= 7
- SPA p < 0.01

---

## Digital Twin Parity

Leviathan shadow strategies must maintain parity with production:

| Metric | Threshold |
|--------|-----------|
| EPE (Execution Parity Error) | < 5% |
| FRE (Fill Rate Error) | < 10% |
| LPE (Latency Parity Error) | < 20% |

Parity failure blocks graduation regardless of ROI performance.

---

## Regression Protection

Core regression suite (~179 tests) covers:

- All FAIL codes (one test per code)
- All INVARIANT rules
- Edge cases identified in post-mortems (MERL, DEXE, DENT incidents)
- SL stack behavior per mode (STRICT / CONTROLLED_OFF / BREAKGLASS_OFF)
- Bootstrap 5-condition simultaneous unlock
- Thompson sampling regime context

---

## Test Environment

```bash
# Quick single-symbol validation
export CHECKPOINT_LOGGING=1 FAIL_FAST_STRICT=1
python3 -c "
from core.a0_event_loop import EventLoop
r = EventLoop().process_single_symbol('BTCUSDT', mode='turbo')
assert r.get('card_created'), 'FAIL'
"
tail -20 logs_v5.2/checkpoint_BTCUSDT.jsonl

# Full pipeline
./run_safe_mode.sh
```

---

*Validation documentation for DROS v12.4b | 2026-03-14*
