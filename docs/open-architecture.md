# 🔓 Open Architecture, Closed Execution

> A third path between fully open source and fully proprietary systems.

---

## The Model

DROS operates under a **dual-layer transparency model**:

| Layer | Status | Rationale |
| :--- | :--- | :--- |
| System architecture | **Open** | Transparency builds community trust |
| Agent pipeline design | **Open** | Research contribution and peer review |
| Safety gate principles | **Open** | Accountability and reproducibility |
| Learning methodology | **Open** | Academic validation and critique |
| Evolution framework | **Open** | Ecosystem development |
| Production daemon code | **Closed** | Execution edge protection |
| Learned parameters | **Closed** | Strategy confidentiality |
| Live configuration | **Closed** | Operational security |
| Model weights | **Closed** | Proprietary optimization |

---

## Why Not Fully Open Source?

A fully open system would expose the precise calibration that makes the execution layer effective.
In adversarial markets, execution parameters are the edge. Publishing them reduces their value for everyone — including the community.

More importantly: **the architecture is the contribution worth sharing**.

The design decisions — how 16 agents coordinate without conflicts, how an 8-layer safety gate prevents cascading failures, how evolution happens under scientific discipline — these are the ideas that advance the field.

The specific threshold values and learned weights are implementation details that do not generalize.

---

## Why Not Fully Closed?

A fully closed system cannot be critiqued, improved upon, or trusted.

**Transparency enables:**
- Academic peer review of methodology
- Community identification of design flaws
- Independent replication of architectural patterns
- Institutional due diligence without NDA friction

**Closed execution enables:**
- Competitive edge preservation
- Adversarial environment protection
- IP protection for enterprise licensing discussions

---

## What We Open

### Architecture and Design
- 16-agent pipeline structure and interaction contracts
- SpacingOracleSSOT concept (Yang-Zhang volatility approach)
- 8-Layer Safety Gate design and layer rationale
- Named FAIL codes and INVARIANT contracts (principles, not values)
- Shadow → Canary → Production deployment framework

**Pre-alert signals**: SYMBOL_WATCH / SYMBOL_CONFIRMED / SYMBOL_WATCH_EXPIRED / SYMBOL_ACTION_PROOF
- Hook 6 (direction_safety.py): PSI threshold → SYMBOL_WATCH
- Hook 7 (daemon.py): grid execution degradation → SYMBOL_WATCH
- Hook 8 (entry_gate.py): entry confirmed block → SYMBOL_CONFIRMED

### Methodology
- AWR + Thompson Sampling dual learning loop design
- CPCV + PBO validation pipeline (academic standard)
- Black Swan Ensemble architecture (ADWIN + CUSUM + BOCPD + Hawkes, 2/4 vote)
- AI Evolution Lab: Strangler Fig pattern, MAP-Elites genome search
- SparseSafeCalibrator adaptive selection by sample count

### Failure Cases
- MERL liquidation event (Feb 2, 2026) — root cause and response
- CLO uncertainty event (Feb 4, 2026) — direction confidence failure
- Every safety layer has its documented origin failure

### Academic Foundation
- 12 peer-reviewed references (López de Prado, Easley et al., Adams & MacKay, Hasani et al., Albers et al.)
- Methodology aligned with academic backtesting standards

---

### Banned Extension Patterns

| Pattern | Fail Code | Required Alternative |
|:---|:---|:---|
| asyncio.call_later() for ACTION_PROOF | FAIL_MKT_SCHEDULE_LOST | Use SQLite scheduled_marketing_events |
| Manual watch_id construction | FAIL_MKT_WATCH_ID_MANUAL | Use WatchRegistry.new_watch_id() |

---

## What We Keep Closed

### Execution Layer
- `execution/` daemon implementation
- Order routing logic and timing parameters
- Exact safety threshold values (calibrated per market regime)

### Learned State
- Thompson Sampling posterior values
- AWR replay buffer and trained weights
- SparseSafeCalibrator fitted state
- Preset parameter configurations

### Operational Data
- Live positions and portfolio state
- Historical PnL and drawdown figures
- Active symbol configurations
- API credentials and runtime environment

---

## For Enterprise Partners

Institutional partners (exchanges, quant funds, strategic investors) receive access to additional layers through NDA-gated briefings:

- Full architecture walkthrough with implementation details
- Performance attribution methodology
- Deployment infrastructure documentation
- Licensing and acquisition discussions

→ [Enterprise inquiries](mailto:enterprise@deepreasoningos.com)

---

## For Researchers

Academic researchers may cite DROS architecture in publications using:

```bibtex
@software{dros2026,
  author = {DROS Core Team},
  title  = {Deep Reasoning OS: A Multi-Agent Autonomous Trading Architecture},
  year   = {2026},
  url    = {https://github.com/dros-core/deep-reasoning-os}
}
```

→ [Full citation format](../CITATION.cff)

---

## For Community Members

The DROS Research Lab shares public-safe observations about:
- Market regime transitions and structural shifts
- Architecture updates and design decisions
- Research methodology and validation approaches

No financial signals. No alpha claims. Engineering transparency only.

→ [Join DROS Research Lab](https://t.me/deepreasoningos)

---

*→ See [Architecture](./architecture.md) · [Safety](./safety.md) · [FAQ](./faq.md)*
