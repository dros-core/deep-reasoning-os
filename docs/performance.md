# Performance Baselines

Current KPI targets and measurement methodology for DROS v12.4b.

---

## KPI Targets

| KPI | Target | Current Status |
|-----|--------|---------------|
| Cost Gate pass rate | >= 90% | 95% |
| Negative ROI card ratio | 0% | 0% |
| Live-Backtest divergence | <= 10% | Measuring |
| Card creation rate | >= 80% | Measuring |
| SL whipsaw rate | 0% | 0% (post v12.1) |
| Slot utilization | >= 70% | Measuring |
| Reconciler drift ratio | < 50% per cycle | < 5% |

---

## Cost Gate Methodology

The Cost Gate validates that gross ROI exceeds total execution costs:

```
cost_gate_pass = gross_roi_dec > k_floor x (fees_dec + slippage_dec)
```

Where:
- `fees_dec`: Binance maker/taker fee in decimal
- `slippage_dec`: estimated slippage based on order book depth
- `k_floor`: safety multiplier (contract constant)
- `gross_roi_dec`: expected gross return per grid cycle

Pass rate is measured over rolling 30-day window. Drop below 90% triggers parameter review.

---

## Spacing Oracle Performance

Yang-Zhang volatility estimator used as sigma_d baseline:

| Estimator | Bias | Noise | Preferred |
|-----------|------|-------|-----------|
| Yang-Zhang | Low | Low | Yes |
| ATR | Medium | High | Fallback only |
| Close-to-Close | High | High | Never |

Yang-Zhang requires OHLC data. ATR used only when OHLC unavailable.

---

## Direction Accuracy (Post v12.2)

Measured from Direction Revalidation Enhancer shadow log:

| Metric | Target | Description |
|--------|--------|-------------|
| conflict_precision | >= 60% | Accuracy of LiveCheck conflict signals |
| false_positive_rate | <= 20% | Rate of false conflict blocks |
| ADX stale events | 0 | Entries with stale ADX (TTL 300s) |

Shadow promotion requires: conflict_precision >= 60%, false_positive_rate <= 20%, >= 50 events, >= 7 shadow days, SPA p < 0.01.

---

## Dynamic SL Validation (Post v12.1)

Validated over 11.7-day shadow period with 459 events:

| Result | Value |
|--------|-------|
| ranging SL widening rate | 100% |
| whipsaw events | 0 |
| Stack A false fires | 0 |
| Chandelier trail coverage | 100% of active positions |

---

## Learning Pipeline Performance

| Metric | Target | Notes |
|--------|--------|-------|
| Thompson update latency | < 30 min throttle | regime-aware |
| BLS memory (lightweight) | < 2.5 GB RSS | subprocess-isolated |
| BLS memory (full) | < 6.0 GB RSS | subprocess-isolated |
| CPCV purge gap | >= 24h | prevents data leakage |
| PBO threshold | < 0.3 | overfitting gate |
| Calibration samples | >= 50 for IsotonicReg | SparseSafeCalibrator |

---

## Marketing Signal Performance

Measured from WatchRegistry over rolling 30-day window:

| Metric | Description |
|--------|-------------|
| watch_confirmation_rate | CONFIRMED / (CONFIRMED + EXPIRED) |
| median_lead_time | Median seconds from WATCH to CONFIRMED |
| false_positive_rate | EXPIRED / total WATCH events |
| action_proof_coverage | ACTION_PROOF per CONFIRMED |

All four states published. EXPIRED events are never suppressed.

Target confirmation rate: > 55% (Numerai baseline: 50% = random)

---

## Benchmark Environment

- **Hardware**: Apple M4 Pro (12-core CPU, 30-core GPU, 48GB unified memory)
- **OS**: macOS Sonoma
- **Python**: 3.13
- **Exchange**: Binance USDT-M Futures (hedge mode)
- **Symbols**: Dynamic selection based on Symbol Sentiment Tier

---

## Latency Profile

| Operation | Target Latency |
|-----------|---------------|
| LLM call (trading) | < 100ms |
| LLM call (analysis) | < 3000ms |
| Order placement | < 200ms |
| Reconciler cycle | < 60s |
| A0-A15 pipeline (per symbol) | < 30s (stage timeout) |
| Checkpoint write | < 1s (atomic write) |

---

## Test Coverage

- **Total tests**: 500+
- **Regression suite**: ~179 core execution tests
- **Integration tests**: ~6 pipeline end-to-end
- **Contract tests**: One test per fail code
- **Skip rate**: < 10% (pre-existing timeout: test_day9_kpi)

All tests run via dros-test-runner after any modification to execution/, contracts/, services/, or utils/.

---

*Performance documentation for DROS v12.4b | 2026-03-14*
