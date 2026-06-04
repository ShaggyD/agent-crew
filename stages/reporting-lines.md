# Reporting Lines & Decision Flow

How decisions reach the CEO, how escalations work, and who owns what at each stage.

This document is the cross-stage reference. For stage-specific reporting structures with org charts, see [stages/](../stages/).

---

## Reporting structure by stage

Reporting structure is stage-dependent. The key shift: at SeedCo, everyone reports to the CEO. At GrowthCo, engineering reports to CTO and the rest report to CEO. At ScaleCo, executives manage departments and report to CEO.

| Stage | Structure | Detail |
|---|---|---|
| [SeedCo](../stages/seed-co.md) | All ICs → CEO | 5 people, flat. CEO manages everyone. No management layers. |
| [GrowthCo](../stages/growth-co.md) | Engineering → CTO, everyone else → CEO | 15 people, functional leads report to CEO directly. No COO. |
| [ScaleCo](../stages/scale-co.md) | Executives → CEO, departments → execs | 50 people, CEO manages exec team. Each exec manages their function. |

## Why Engineering reports to CTO (GrowthCo and ScaleCo)

Engineering velocity and quality are technical decisions that require technical leadership. The CTO handles code review standards, architecture decisions, and technical debt prioritization. Engineering work is still *planned* through the company's priority framework — the CTO coordinates with the COO (ScaleCo) or CEO (GrowthCo) on what gets built and when, while controlling *how* it gets built.

## Decision ownership

These apply across all stages. At smaller stages, the individual listed might be the only person in the role.

| Decision | Owner |
|---|---|
| Product spec and acceptance criteria | Product Manager |
| Implementation approach | Software Engineer (with technical lead oversight) |
| Design direction | Product Designer (or founder at SeedCo) |
| Test coverage / release gate | QA Engineer |
| Deployment timing | DevOps Engineer |
| Content publication | Content Lead |
| Campaign budget and timing | Growth/Marketing Lead (or CMO at ScaleCo) |
| Deal pricing and terms | Sales Lead (within bounds set by CEO) |
| Client communication and retention | Customer Success |
| Inbox triage and scheduling priority | Comms/Admin Manager |
| Budget allocation within plan | Finance/Ops (or CFO at ScaleCo) |
| Cross-functional scope disputes | CTO + domain lead (GrowthCo) or COO (ScaleCo) |
| Strategic product roadmap | CPO (ScaleCo) or CEO (GrowthCo/SeedCo) |

## Escalation path

```
SeedCo:  Any IC → CEO (immediate, direct)
GrowthCo: IC → functional lead → CEO → CTO (for tech issues)
ScaleCo: IC → team lead → department head → COO → CEO (strategic only)
```

**Escalation is not failure.** Expectation: escalate with a recommendation, not just a problem.

## When the CEO gets involved (all stages)

The CEO only touches decisions that:
1. **Change company direction** — new market, new product line, pivot
2. **Require funding** — hiring, tooling, infrastructure investment beyond plan
3. **Are existential** — legal, compliance, major client risk
4. **Cannot be resolved below** — two department heads agree they can't decide

Everything else is handled below the CEO. At SeedCo this means the CEO is in everything. At ScaleCo it means the CEO only sees strategic or existential decisions.

## Communication flow by stage

See [interactions/communications-protocol.md](../interactions/communications-protocol.md) for detailed communication cadence and channel structure by stage.

## Escalation triggers (all stages)

| Signal | Action |
|---|---|
| Blocked > 4 hours | Escalate within your function |
| Blocked > 24 hours | Escalate to department lead (or CEO at SeedCo/GrowthCo) |
| Client at risk | Immediate escalation to COO (ScaleCo) or CEO (SeedCo/GrowthCo) |
| Security / compliance issue | Immediate to CTO → CEO |
| Budget variance > 10% | Flag to Finance → COO (ScaleCo) or CEO (GrowthCo/SeedCo) |
