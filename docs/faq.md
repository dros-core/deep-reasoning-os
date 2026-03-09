# ❓ Frequently Asked Questions

---

## About DROS

### What is DROS?

Deep Reasoning OS is a multi-agent autonomous trading system for Binance USDT perpetual futures. It is a research and engineering project — not a managed fund, not a trading service, not a bot you can download and run.

### Is this open source?

DROS operates as **open architecture, closed execution**. The architecture documentation, design principles, invariant contracts, and research methodology are publicly shared here. The production trading infrastructure — execution code, ML model weights, runtime configuration, and strategy parameters — are private.

### Can I run DROS myself?

The production system is not publicly released. This repository documents how DROS works architecturally. If you are interested in a commercial or research collaboration, see [Enterprise & Partnerships](../README.md#-enterprise--partnerships).

### Why share the architecture but not the code?

Sharing the architecture creates value for the research community, enables collaboration on hard problems (market microstructure, safety-critical systems, online learning), and demonstrates technical depth. The execution layer contains operational details that would require extensive context and infrastructure to use correctly.

---

## Safety & Risk

### Does DROS lose money?

Like any trading system, DROS operates in an uncertain environment. DROS was designed with survivability as a first-order concern — the 7-Layer Entry Gate, invariant contract system, and shadow-to-production deployment pipeline all exist to minimize catastrophic outcomes. Past operational data is not published here. Nothing in this repository is a performance guarantee.

### What happened with the MERLUSDT liquidation?

On 2026-02-02, DROS held a SHORT position in MERLUSDT that experienced a +37% LONG rally, resulting in a 100% liquidation event. This event directly triggered the addition of 5 new safety layers. The full architectural response is documented in [Safety](./safety.md).

### Is this financial advice?

No. Nothing in this repository constitutes financial advice, investment solicitation, or a guarantee of future performance.

---

## Community & Contact

### How do I join the research community?

The DROS Research Lab Telegram channel shares public-safe regime alerts, architecture release notes, and research discussions.

**[→ Join DROS Research Lab on Telegram](https://t.me/deepreasoningos)**

### How do I contact DROS for a business discussion?

For technology licensing, strategic partnerships, or institutional deployment consulting:

📩 **[enterprise@deepreasoningos.com](mailto:enterprise@deepreasoningos.com)**

Private architecture briefings are available under NDA. Please do not use GitHub Issues for business inquiries.

### How do I report a security concern?

📩 **[security@deepreasoningos.com](mailto:security@deepreasoningos.com)**

See [SECURITY.md](../SECURITY.md) for full details.

---

## Technical

### What is Yang-Zhang volatility?

Yang-Zhang is a volatility estimator that uses Open, High, Low, and Close prices to compute a more accurate estimate than close-to-close methods. DROS uses it as the primary input to SpacingOracleSSOT for dynamic grid spacing. ATR-only estimation is explicitly forbidden (`INVARIANT-SPACING-04`).

### What is VPIN?

Volume-synchronized Probability of Informed Trading. DROS uses VPIN fused with Order Flow Imbalance (OFI) to detect toxic order flow. High VPIN triggers the Toxicity Shield gate.

### What is Thompson Sampling doing here?

Thompson Sampling is used as a Bayesian bandit for preset selection. At each rotation cycle, the system samples from posterior distributions over preset performance (regime-conditioned), observes the outcome, and updates the posterior. This replaces manual parameter tuning with adaptive online learning.

### What does "Shadow → Canary → Production" mean?

Every new strategy or model change goes through three stages:

1. **Shadow**: Runs alongside production, generates signals, does not execute orders. Minimum 7 days.
2. **Canary**: Routes 10% of live traffic. SPA test (p < 0.01) required before promotion.
3. **Production**: Full deployment.

### What is AWR?

Advantage-Weighted Regression — an off-policy reinforcement learning algorithm. DROS uses AWR as the primary RL agent for grid parameter optimization, replacing the previous PPO implementation.

---

*→ See [Architecture](./architecture.md) · [Safety](./safety.md) · [Learning](./learning.md)*