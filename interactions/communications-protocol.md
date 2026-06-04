# Communications Protocol

How agents communicate with each other, escalate when blocked, and route information across the crew.

The protocol changes dramatically by stage. SeedCo communicates synchronously because everyone needs to know everything. ScaleCo communicates async by default because synchronous doesn't scale.

## Communication patterns by stage

### SeedCo: Full transparency, synchronous

Everyone sees everything. A single Slack channel or daily standup covers the company. Bad news surfaces immediately because everyone's in the same room (or thread).

**Channels:**
- One main channel for everything
- Direct messages for quick coordination
- No formal standup — "what's happening" is a question, not a meeting

**Escalation:**
- Any team member can escalate directly to the CEO
- Expectation: escalate with a proposed solution, not just a problem
- No formal SLAs — respond when you see it

**Notification rules:**
- Customer complaints → immediate to Priya (CS) + Rhea (CEO)
- Production issues → immediate to Marcus (Engineer)
- Budget questions → immediate to Jordan (Ops)
- Everything else → when you get to it

### GrowthCo: Functional channels, lightweight rituals

Departments use their own channels. Cross-functional coordination happens through direct conversation, not routed through a central hub.

**Channels:**
- #general (company-wide announcements, wins)
- #eng (engineering — PRs, deployments, incidents)
- #product (specs, feedback, user research)
- #marketing (content, campaigns, analytics)
- #revenue (deals, pipeline, customer stories)
- #ops (finance, legal, admin)

**Rhythm:**
- **Daily:** Engineering standup (15 min, async check-in). CEO sync with anyone who needs it.
- **Weekly:** Monday priorities thread in #general. Friday wins thread.
- **Monthly:** All-hands (30 min) — CEO presents metrics, milestones, direction.

**Escalation:**
- Individual → department lead → CEO
- Engineering escalations go through CTO
- Everyone else escalates to CEO
- SLA: 4-hour response for blockers, 24-hour for non-urgent

**Notification rules:**
- @here for urgent messages (production down, security issue)
- @channel for department-relevant announcements
- No @everyone unless it's company-critical

### ScaleCo: Async-first, structured escalation

Process replaces proximity. Communication runs through defined channels with clear ownership and response SLAs.

**Channels:**
- Departmental channels with sub-channels for projects
- #exec for executive team discussions
- #announcements (read-only, company-wide updates)
- Incident channels (#incident-active with auto-creation for production issues)

**Rhythm:**
- **Daily:** Async check-in in each department channel. Engineering standup (15 min).
- **Weekly:** Exec staff meeting (60 min). Department standups (30 min each). Friday wrap-up post.
- **Bi-weekly:** All-hands (45 min) — executive updates, dept highlights, AMA.
- **Quarterly:** Offsite / strategy session. OKR review.

**Escalation:**
1. IC → Team lead → Department head → COO → CEO (only for strategic or existential issues)
2. Engineering: IC → Staff engineer → CTO → COO
3. Customer: CS lead → VP Sales → COO
4. Incident: DevOps → CTO (immediate, no waiting)

**SLA by severity:**
- **Critical** (production down, security breach, customer data risk): Respond within 15 min, any hour.
- **High** (feature broken for a segment, P1 customer blocked): Respond within 1 hour, business hours.
- **Medium** (bug with workaround, process gap): Respond within 24 hours.
- **Low** (improvement, nice-to-have): Respond within 1 week.

## Escalation principles (all stages)

1. **Escalate with a recommendation, not just a problem.** "X is broken. I've tried Y. I think we should do Z. Do you agree?"
2. **Bad news doesn't get better with time.** Surface it the moment you know.
3. **Escalation is not failure.** It's how the org learns where the bottlenecks are.
4. **If a decision doesn't change the company's direction, make it yourself.**
5. **CC is a permission structure, not a notification.** If you're CC'd, you're expected to read. If you don't need to act, you shouldn't be CC'd.
