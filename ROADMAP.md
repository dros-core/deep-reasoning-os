# DROS Roadmap

## Current Version: v12.4b (2026-03-14)

### v12.4b — Bootstrap Safety Gate + Marketing Research Product Layer
- **Bootstrap Safety Gate** (v12.3b): 5-condition unlock, quality_base_no_llm enforcement
- **Marketing v6.2**: Research Product Layer — Dual-Surface copy architecture
  - `psych_policy.py` SSOT: PSYCH_INTENT + COMPREHENSION_LEVEL + CONVERSION_MODE
  - `time_policy.py` SSOT: channel-by-channel time rendering (relative vs exact)
  - `evidence_surface.py` SSOT: per-signal public/hidden field policy
  - Public formatter: Surface A (plain English) + Surface B (evidence line)
  - X formatter: proof-card format, <=220 chars, zero jargon
  - Private formatter: 4-line operator memo, exact timestamps
- **Leviathan v4.0**: Renamed from AI Evolution Lab — 5-Brain Architecture formalized
- CB 3-budget independence: rotation / slot expansion / reliability fully decoupled
- FGI as Macro Tempo Gate only (FAIL_FGI_DIRECT_P_DIR_BOOST enforcement)

---

## Version History

### v12.3b (2026-03-13)
- Bootstrap Safety Gate: 5-condition simultaneous unlock
  - rank==1 AND positioned==0 AND quality_base_no_llm>=85 AND age<=60m AND live_alignment>=2
- FAIL_BOOTSTRAP_RAW_QUALITY_UNLOCK: quality_final (with LLM boost) cannot unlock
- FAIL_BOOTSTRAP_SHADOW_MUTATION: shadow mode threshold changes blocked
- FAIL_SL_DIRECT_FILL: _fill_open_slots() direct call after SL exit blocked
- BootstrapActionBucket persisted to DaemonCheckpoint (token restoration on restart)
- Marketing v6.1 production deployment:
  - emoji_policy.py SSOT (EMJ-001/002/003)
  - VALID_POSTURES frozenset enforcement
  - scheduled_marketing_events SQLite for ACTION_PROOF (no asyncio.call_later)
  - embedding_guard.py cosine similarity shadow mode
  - Signal fidelity: 15 conflicts resolved

### v12.2 (2026-03-12)
- **FGI & Macro Tempo Gate**: FGI is gate-only, FAIL_FGI_DIRECT_P_DIR_BOOST
- **Symbol Sentiment 5-axis**: Crowding + Participation + Aggression + SmartMoney + Execution
- **CB 3-budget**: rotation_timestamps / slot_expansion_budget / reliability_cb fully independent
- **Direction Accuracy**: Live ADX + ER indicators, OR condition enforcement
  - CONFLICT-001: LiveDirectionCheck AND -> OR (FAIL_LIVECHECKAND_CONDITION)
  - CONFLICT-002: Dead Zone SSOT (0.38/0.62, widened from 0.45/0.55)
  - CONFLICT-003: adx_slope=0.0 hardcoding removed, _adx_cache computed
  - Asymmetric thresholds: Long >=0.58, Short <=0.42
- Direction Revalidation Enhancer (shadow): 3-stage BLOCK/DEGRADE/PASS
- Thompson cluster warm-start prior (peer average -> card p_dir -> Beta(1,1))
- 4 new FAIL codes: FAIL_STALE_ENTRY_ADX, FAIL_LIVECHECKAND_CONDITION, FAIL_DEAD_ZONE_NOT_SSOT, FAIL_ADX_SLOPE_HARDCODED

### v12.1 (2026-03-11)
- **Dynamic SL 3-Stack Architecture**:
  - Stack A: Catastrophe Stop (always active, even in CONTROLLED_OFF)
  - Stack B: Chandelier Trail (regime-adaptive: ranging=2.5x, trending=1.5x, volatile=3.0x)
  - Stack C: Grid-Fail SL (3-factor: hours_no_fill + ADX>=25 + adverse_drift)
- SL_MODE: STRICT / CONTROLLED_OFF (TTL 2h, Stack A preserved) / BREAKGLASS_OFF
- FAIL_SL_DISABLED, FAIL_SL_CAT_DISABLED, FAIL_SL_BREAKGLASS_NOTT, FAIL_SL_LIQ_TOO_CLOSE, FAIL_SL_ORDER_BYPASS
- place_protective_stop() wrapper: STOP_MARKET via API with closePosition auto-branch
- ranging sl_mult: 1.5 -> 2.0 (ER-adaptive: 0.8 coefficient)
- Production deployment: USE_ADAPTIVE_SL_PRODUCTION=1, USE_GRID_FAIL_SL=1, USE_LIQ_BUFFER_SL_FLOOR=1
- Validation: 11.7d shadow, 459 events, 0 whipsaws

### v12.0 (2026-03-11)
- **Position State Machine**: ENTRY_CANDIDATE -> MANAGED_POSITION / EXTERNAL_ORPHAN
- **8-Layer Safety Gate**: Hard Gate (PSI/tail_risk/net_roi) before LLM soft scoring
- PSI Double Gate: card creation + entry decision both checked
- LLM Hard-Safety Guard: PSI/tail_risk/net_roi violations zero out llm_adj
- MANAGED_POSITION refresh: event-driven 30-min cycle (ADWIN + ATR + conf triggers)
- INJECT gate: card_origin=True AND 3-gate pass required
- FAIL_PSI_ABSOLUTE_VETO, FAIL_NEGATIVE_ROI_ENTRY, FAIL_POSITION_UNREGISTERED, FAIL_INJECT_NON_CARD_ORIGIN

### v11.26 (2026-03-10)
- MLX subprocess spawn context (fork() banned: FAIL_EVOL_FORK_MLX)
- Evidence-based graduation gate v3: ESS + coverage + truncated mSPRT
- AlphaFoundryClock 72h control (INVARIANT-EVOL-19)
- MLX GPU fitness proxy (shadow)
- OODA regime drift acceleration: research 12h fast-track on drift

### v11.25 (2026-03-10)
- MLX warmup deadlock fix: warmup() deferred to asyncio background task
- Python 3.13 pandas/numpy Cython import lock fix: preload before daemon import
- OpenSSL EVP deadlock: OPENSSL_CONF=/dev/null in plist
- LightGBM + ChromaDB SIGSEGV: n_jobs=1 enforcement, OMP_NUM_THREADS=1

### v11.0 (2026-03-06)
- Game Theory layer: VPIN toxicity detection, order flow analysis
- INVARIANT-GT-01: observation data only (no synthetic injection)
- Binance hedge mode positionSide enforcement (FAIL_AOSM_HEDGE_MODE_MISSING_SIDE)
- N* Dithering: hash(epoch_hour, symbol) deterministic, random() banned

### v10.x — v6.x (Historical)
- v10.37: Learning pipeline — AWR, BLS subprocess, SparseSafeCalibrator
- v9.3.0: TP/SL rotation, per-level take-profit GTC orders
- v9.9.3: FLAT Flip Guard
- v8.9.2: DirectionSafety PSI veto, macro sentiment gate
- v8.7.4.1: Surge detection
- v8.4+: MERL prevention (post-liquidation defense system)
- v6.x: Marketing automation baseline (hook system, LLM copy)

---

## Planned

### Near-term (Q2 2026)
- Weekly Transparency digest (Monday 09:00 KST, Private channel)
- Bot 5-command interface: /scorecard /watch /accuracy /expired /regime
- GitHub public docs: methodology.md, scoring_policy.md, signal_taxonomy.md

### Medium-term (Q3 2026)
- Direction Revalidation Enhancer: shadow -> production (>=60% conflict_precision, <=20% false_positive_rate)
- MLX Batch Indicators: shadow -> production
- Leviathan Ring 2: MCTS grid optimizer production graduation
- Black Swan ensemble: ADWIN+CUSUM+BOCPD+Hawkes 2/4 vote production

### Long-term (Q4 2026+)
- Multi-exchange support (architecture groundwork)
- OODA loop: offline strategy improvement (03:00-09:00 KST window)
- AlphaFoundry QD MAP-Elites: full genome evolution pipeline
- Academic publication: CPCV+PBO+SPA graduation methodology

---

*Updated: 2026-03-14*
