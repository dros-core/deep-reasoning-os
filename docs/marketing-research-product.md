# Marketing v6.2 — Research Product Layer

DROS operates a transparent observation feed. The system registers signals before outcomes, scores them after, and publishes all states — confirmed, expired, contained, and protected.

**Core philosophy**: "We recorded first, results disclosed on same ledger."

---

## Research Product Layer

DROS marketing is not a signal service. It is an observation ledger with verifiable timestamps.

### Pre-register -> Score -> Disclose

```
1. SYMBOL_WATCH    -- observation filed (timestamp T0)
2. Market develops
3. SYMBOL_CONFIRMED or SYMBOL_WATCH_EXPIRED (timestamp T1)
4. If confirmed: SYMBOL_ACTION_PROOF (T1 + 30min delay)
```

All four states published on the same record. No cherry-picking. EXPIRED signals are never suppressed.

This follows the Registered Reports model from academic research: method fixed before results, outcomes disclosed on same ledger regardless of direction.

---

## Dual-Surface Message Architecture

Every public message has two surfaces:

**Surface A** — Plain English (first line, zero jargon)
```
BTC moved into defensive monitoring.
```

**Surface B** — Evidence (second line, one verifiable data point)
```
Selling pressure rose above the watch threshold.
```

Surface A is readable by anyone. Surface B provides the evidence layer for those who want to verify. Technical terms (PSI, ADX) never appear in Surface A.

---

## Channel Architecture

### X — Proof Distribution

- Format: proof card (160-220 chars)
- Jargon: 0 terms
- First sentence: self-explanatory without context
- CTA: proof signals only (SYMBOL_CONFIRMED, SYMBOL_ACTION_PROOF)
- Alert signals (KILL_SWITCH, REGIME_SHIFT): no CTA

```
BTC -- posture shifted ~90m before the visible move.
The watch resolved before confirmation spread.
t.me/dros_quant_lab
```

### Telegram Public — Observation Feed

- Format: Surface A + Surface B
- Emoji: 1 (severity marker only: critical=shield, high=chart-down, medium=brain, low=satellite)
- CTA: soft, only on SYMBOL_CONFIRMED and SYMBOL_ACTION_PROOF
- Daily limit: 5 messages

```
[chart-down] BTC moved into defensive monitoring.
Selling pressure rose above the watch threshold.
Observation window: ~4h.
```

### Telegram Private — Operator Terminal

- Format: 4-line operator memo
- Emoji: 0 (absolutely banned)
- Timestamps: exact (2026-03-14 13:04 KST + Lead: 90m)
- Daily limit: 12 messages

```
[CONFIRMED] BTCUSDT Ref W...8f3a
Confirmed after ~90m with two aligned signals.
Resolution: 2-factor aligned within window.
Posture: defensive maintained.
```

---

## Signal Types and Psychology

| Signal | Channel | Psychology | Conversion |
|--------|---------|------------|-----------|
| SYMBOL_WATCH | Private | Open loop (Zeigarnik) | none |
| SYMBOL_CONFIRMED | Public + Private | Evidence resolution | proof_cta |
| SYMBOL_ACTION_PROOF | X + Public + Private | Foresight proof | proof_cta |
| SYMBOL_WATCH_EXPIRED | Private only | Trust transparency | none |
| KILL_SWITCH | All | Trust preservation | none |
| REGIME_SHIFT | All | Uncertainty reframing | none |
| WEEKLY_TRANSPARENCY | Private only | Retention anchor | none |

---

## SSOT Modules (v6.2)

### psych_policy.py
- PSYCH_INTENT: signal -> psychological intent mapping
- COMPREHENSION_LEVEL: channel -> max technical terms, first-sentence rules
- CONVERSION_MODE: signal -> CTA policy (none / proof_cta / soft_scorecard)

### time_policy.py
- TIME_RENDERING: channel -> relative or exact
  - telegram_public: relative (~90m, ~4h)
  - telegram_private: exact (2026-03-14 13:04 KST + Lead: 90m)
  - x_delayed: relative
  - pinned_scorecard / weekly_digest: exact

### evidence_surface.py
- EVIDENCE_SURFACE_PUBLIC: per-signal allowed fields for public/X channels
- EVIDENCE_HIDDEN_FIELDS: fields banned from all external channels
  - check_id, layer, reason_code, resolution_type, lead_time_s, condition

---

## Time Display Rules

| Channel | Format | Example |
|---------|--------|---------|
| telegram_public | relative | ~90 minutes, ~4h |
| x_delayed | relative | after ~90m |
| telegram_private | exact | 2026-03-14 13:04 KST, Lead: 90m |
| pinned_scorecard | exact | Updated: 2026-03-14 09:00 KST |

`lead_time_s` raw value is never exposed. Must route through `humanize_lead_time()` (FAIL_MKT_LEAD_TIME_RAW).

---

## 6 Psychology Levers

| Lever | Implementation | Forbidden |
|-------|---------------|-----------|
| Uncertainty | Posture frame ("Posture: defensive") | "Big crash coming" |
| Loss aversion | "Expansion paused", "Exposure contained" | Return guarantees |
| Weak control | Bot 5-command interface (query-only) | Real position control |
| Transparent failure | EXPIRED always published | Suppressing expired |
| Information scarcity | Free = observation, Paid = operator terminal | Fake urgency |
| Plain language | First line = zero jargon | PSI/ADX in first line |

---

## Bot Interface (5 Commands)

Provides weak control (Algorithm Aversion reduction per INFORMS mnsc.2016.2643):

```
/scorecard    -- 30d: {confirmed}/{watches} ({rate}%) | Median lead: {median}min
/watch BTC    -- BTC -- [PENDING] Ref W...{tail} | Opened: ~{age} ago | Remaining: ~{ttl}
/accuracy 30d -- 30d rate: {rate}% | Hook6={h6}% | Hook7={h7}% | Hook8={h8}%
/expired 7d   -- {count} expired | SYMBOL | Window | Expired_at
/regime       -- {regime} | PSI: {psi} | ADX: {adx} | Since: {since}
```

Query-only. No modification commands. Weak control improves algorithmic acceptance without granting execution access.

---

## Weekly Transparency (Monday 09:00 KST)

Fixed 6-metric digest published to Private channel:

```
--- DROS Weekly (W{week}/{year}) ---
Watches: {total} | Confirmed: {confirmed} ({rate}%)
Expired: {expired} | Median lead: {median}min
KILL_SWITCH: {count} | Uptime: {uptime}%
```

Idempotency key: `weekly_{YYYY}_{WW}` (prevents duplicate on restart). EXPIRED count always included.

---

## Invariants (v6.1-6.2)

| Code | Rule |
|------|------|
| MKT-17 | Title from title_map.py SSOT only |
| MKT-18 | LLM input: event_normalizer typed schema only |
| MKT-21 | telegram_public: 1-3 emoji required |
| MKT-22 | telegram_private: 0 emoji (absolute) |
| MKT-25 | Channel tone consistency required |
| MKT-26 | Jaccard > 0.65 repetition blocked |
| MKT-28 | watch_id from WatchRegistry.new_watch_id() only |
| MKT-29 | EXPIRED never suppressed |
| MKT-33 | SYMBOL_CONFIRMED requires linked WATCH |
| MKT-34 | lead_time_s via humanize_lead_time() only |
| MKT-35 | Daily channel limits enforced |

---

## SEC / MiCA Compliance

All public communications:
- No PnL or return guarantees
- No implied future performance
- Methodology observable and documented
- All outcomes published (confirmed and expired)

Standard disclaimer on all channels:
> Past performance does not guarantee future results. This is not investment advice.

---

*Marketing v6.2 documentation for DROS v12.4b | 2026-03-14*
