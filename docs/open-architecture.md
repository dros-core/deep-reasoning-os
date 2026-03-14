# Open Architecture and Extension Points

DROS is designed with explicit extension points for adding new signal sources, enhancers, and learning components without modifying core execution logic.

---

## Extension Philosophy

**Enhancer pattern**: New logic attaches to the EnhancerBus, not to core agents. Core agents (A0-A15) are stable. New intelligence is added as enhancers that read DecisionPacket fields and write to extra_context only.

**Feature flags**: All new features start with a feature flag defaulting to 0. Shadow mode is mandatory before production. No direct deployment.

**Hook isolation**: Marketing hooks are lazy-imported with try/except isolation. A failing hook cannot affect trading execution.

---

## EnhancerBus (A14)

The primary extension point for new intelligence:

```python
# Enhancer interface
class BaseEnhancer:
    def enhance(self, packet: DecisionPacket) -> None:
        # Read: packet.symbol, packet.p_dir, packet.regime, etc.
        # Write: packet.extra_context["my_signal"] = value
        # NEVER write to packet.p_dir, packet.spacing_dec, etc.
        pass
```

**Registration** (in enhancer_bus.py):
```python
bus.register(MyEnhancer())
```

**Rules**:
- Write to `extra_context` only (INVARIANT-EVOL-01)
- Wrap in try/except with debug logging (silent failure banned)
- Must have feature flag (default 0)
- Shadow mode before production

---

## Signal Extension (Marketing Hooks)

Adding a new marketing signal type:

### 1. Define in signal_taxonomy.py

```python
# Add to SIGNAL_TYPES
"MY_NEW_SIGNAL": {
    "tier": "TIER_2",
    "channels": ["telegram_private"],
    "conversion_mode": "none",
}
```

### 2. Add to event_normalizer.py

```python
# Add to _MARKET_EVENT_MAP
"MY_NEW_SIGNAL": "human-readable description",

# Add to _SEVERITY_MAP
"MY_NEW_SIGNAL": "medium",
```

### 3. Create hook

```python
# In the relevant system file (lazy import + try/except)
def _emit_my_signal(payload):
    try:
        from marketing.emitter import emit_signal
        emit_signal("MY_NEW_SIGNAL", payload)
    except Exception:
        pass  # hook failure cannot affect trading
```

### 4. Add formatter template

Templates in public_formatter.py, private_formatter.py, x_formatter.py follow the OIAS structure (Observation / Interpretation / Action / State).

---

## Learning Extension

Adding a new learning component:

### Thompson Context Extension

```python
# Register new context dimension in Thompson manager
thompson.register_context_dim("my_dim", values=["a", "b", "c"])
```

### New Calibrator

Must implement SparseSafeCalibrator interface:

```python
class MyCalibrator:
    def fit(self, X, y): ...
    def predict_proba(self, X): ...  # must return [0,1]
    def save_state(self, path): ...  # INVARIANT-LEARNING-23
    def load_state(self, path): ...
```

---

## Evolution Lab Extension

### Adding a New Detector to Black Swan Ensemble

Current ensemble: ADWIN, CUSUM, BOCPD, Hawkes (2/4 vote required).

Adding a 5th detector:
1. Implement `BaseAnomalyDetector` interface
2. Register in `black_swan_ensemble.py`
3. Vote threshold remains 2 (out of now 5)
4. Feature flag required

### Adding a New QD Dimension to AlphaFoundry

MAP-Elites grid has defined behavior dimensions. Adding a dimension:
1. Define range and discretization in `alpha_foundry_config.py`
2. Implement measurement in `genome_evaluator.py`
3. Update `required_budget()` calculation (INVARIANT-EVOL-17)

---

## WatchRegistry Extension

Custom observation types can be registered:

```python
from marketing.watch.watch_registry import WatchRegistry

# Always use WatchRegistry.new_watch_id() -- INVARIANT-MKT-28
watch_id = WatchRegistry.new_watch_id()
registry.register_watch(
    watch_id=watch_id,
    symbol="BTCUSDT",
    condition="my_condition",
    window_seconds=3600,
    tier="A",
)
```

Manual watch_id construction triggers FAIL_MKT_WATCH_ID_MANUAL.

---

## Feature Flag Pattern

All extensions require a feature flag:

```python
# config/feature_flags.json
{
  "USE_MY_FEATURE": {
    "state": "shadow",
    "description": "My new feature description"
  }
}
```

```python
# In code
from config.feature_flags import FeatureFlagManager
if FeatureFlagManager.is_active("USE_MY_FEATURE"):
    # production path
elif FeatureFlagManager.is_shadow("USE_MY_FEATURE"):
    # shadow path: log would-have, no capital impact
```

**Flag state override**: `os.environ['USE_MY_FEATURE'] = '0'` overrides feature_flags.json. Changing flag state requires plist `launchctl unload + load`.

---

## Banned Extension Patterns

These patterns are explicitly banned and will trigger fail codes:

| Pattern | Fail Code | Why |
|---------|-----------|-----|
| importlib.reload() | FAIL_IMPORT_RELOAD | Race conditions in async process |
| Direct Ollama call | FAIL_LLM_DIRECT_CALL | Rate limit, timeout, fallback bypass |
| Hardcoded learned_presets path | FAIL_PRESET_PATH_MISMATCH | Path must be registry-managed |
| asyncio.call_later() for marketing | FAIL_MKT_SCHEDULE_LOST | Lost on restart; use SQLite |
| random() for dithering | INVARIANT-AOSM-07 | Non-deterministic, use hash() |
| Flat (non-hierarchical) debias | FAIL_A8_DEBIAS_FLAT | Overfitting on sparse data |
| Direct _fill_open_slots() after SL | FAIL_SL_DIRECT_FILL | Race with slot budget |
| Writing to packet.p_dir in enhancer | INVARIANT-EVOL-01 | Core field, read-only for enhancers |
| fork() for MLX subprocess | FAIL_EVOL_FORK_MLX | Deadlocks on Python 3.13 ARM64 |

---

## Adding New Fail Codes

When adding a new invariant:

1. Add fail code to `contracts/agent_contracts.py`
2. Add test in test suite (one test per fail code)
3. Add to CLAUDE.md Fail-Fast section
4. Run dros-doc-updater agent to sync documentation
5. Add to relevant rules file (.claude/rules/)

---

*Open architecture documentation for DROS v12.4b | 2026-03-14*
