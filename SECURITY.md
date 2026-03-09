# 🛡️ Security

## Scope of This Repository

This repository contains **architecture documentation only**.

There is no executable trading code, no API keys, no credentials, and no deployment infrastructure in this repository. Security vulnerabilities in the sense of CVEs or exploitable code do not apply here.

---

## DROS Safety Architecture (7-Layer Entry Gate)

DROS enforces a deterministic multi-layer validation before any order is placed.

| Layer | Gate | Trigger |
| :--- | :--- | :--- |
| 1 | **Macro Sentiment** | PSI ≤ −0.5 → Long entry blocked |
| 2 | **Tail Risk** | tail_risk ≥ 0.8 → All entry blocked |
| 3 | **Direction Uncertainty** | \|p_dir − 0.5\| ≤ 0.05 → Neutral zone blocked |
| 4 | **Range Extreme** | Price at range boundary → blocked |
| 5 | **Toxicity Shield** | VPIN toxicity score exceeds threshold → blocked |
| 6 | **Liquidation Probability** | liq_distance < safety margin → blocked |
| 7 | **Card Freshness** | card_age > 5,400s → slot fill blocked |

**Result:** No order is sent unless all 7 gates pass. This is enforced deterministically — not probabilistically.

---

## Invariant Contract System

DROS uses a **Fail-Fast invariant system** with named failure codes.

**P0 Invariants (immediate halt on violation):**

| Code | Rule |
| :--- | :--- |
| `FAIL_UNIT_PCT` | `_pct` unit usage anywhere in the codebase |
| `FAIL_CARDSPEC_MUTATION` | CardSpec modified after A9 signing |
| `FAIL_ORPHAN_FORCED_CLOSE` | Forced full liquidation of orphan position |
| `FAIL_NCO_SCALE_IN_ATTEMPTED` | Scale-in on non-card orphan |
| `FAIL_SPACING_SSOT_MISMATCH` | SpacingOracleSSOT bypassed |
| `FAIL_LLM_DIRECT_CALL` | Direct Ollama call bypassing LLM Broker |
| `FAIL_IMPORT_RELOAD` | `importlib.reload()` used (hot-reload violation) |

Every P0 invariant violation halts the system immediately and logs a structured event.

---

## No Credentials in This Repository

This repository does not contain and must never contain:

- Binance API keys or secrets
- Exchange WebSocket credentials
- Private configuration files (`.env`, `config.yaml`, etc.)
- ML model weights or trained parameters
- User account information of any kind

If you discover that credentials have been accidentally committed, please report immediately (see below).

---

## Responsible Disclosure

If you find any issue related to this public repository (e.g., accidentally committed sensitive data, a broken link to a security resource, or documentation that could mislead users into unsafe behavior):

📩 **[security@deepreasoningos.com](mailto:security@deepreasoningos.com)**

Please do not open a public GitHub Issue for potential security concerns. Use the email above.

We will acknowledge receipt within 72 hours.

---

## What DROS Is Not

- DROS is not a managed trading service
- DROS is not financial advice
- DROS does not hold or manage user funds
- This repository does not enable anyone to run the production trading system

*→ See [Safety Architecture](./docs/safety.md) for full gate specifications*