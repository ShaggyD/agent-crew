# Agent Crew — Customization Guide

> **Audience:** Paying clients who have purchased the Agent Crew profile pack.
> **Goal:** Adapt the 25+ profiles to your company's names, voice, industry, and org structure — without breaking anything.
> **Prerequisites:** You've cloned or downloaded the `agent-crew` repo. You're editing files in your own working copy.

---

## Table of Contents

1. [How to Rename a Persona](#1-how-to-rename-a-persona)
2. [How to Adjust Voice / Tone](#2-how-to-adjust-voice--tone)
3. [How to Add Industry-Specific Context](#3-how-to-add-industry-specific-context)
4. [How to Add New Profiles](#4-how-to-add-new-profiles)
5. [How to Adjust Reporting Lines](#5-how-to-adjust-reporting-lines)
6. [How to Adjust Stage Mapping](#6-how-to-adjust-stage-mapping)
7. [How to Maintain Consistency](#7-how-to-maintain-consistency)
8. [Customizing for Different Industries — Quick Reference](#8-customizing-for-different-industries--quick-reference)

---

## 1. How to Rename a Persona

Every profile has a **name** scattered across three places. You must update **all three** or cross-references will break.

### Files That Store the Name

| File | Field / Header | Purpose |
|---|---|---|
| `profiles/<stage>/<department>/<role>/SOUL.md` | `# Agent Name` (line 1, Markdown H1) | The canonical name used by the runtime |
| `profiles/<stage>/<department>/<role>/README.md` | `# Agent Name` (line 1, Markdown H1) | Human-readable docs that mirror the SOUL heading |
| `profiles.json` | `"name"` key inside the profile object | The string the orchestrator matches at load time |

### Example: Rename `"Sarah Chen"` → `"Alex Rivera"`

**Before — `profiles/seedco/engineering/senior-fullstack-engineer/SOUL.md`:**
```markdown
# Sarah Chen

**Role:** Senior Full-Stack Engineer
```

**After:**
```markdown
# Alex Rivera

**Role:** Senior Full-Stack Engineer
```

**Before — same path for `README.md`:**
```markdown
# Sarah Chen

Senior Full-Stack Engineer at SeedCo.
```

**After:**
```markdown
# Alex Rivera

Senior Full-Stack Engineer at SeedCo.
```

**Before — `profiles.json`:**
```json
{
  "id": "seedco-senior-fullstack-engineer",
  "name": "Sarah Chen",
  "role": "Senior Full-Stack Engineer",
  "stage": "seedco",
  "department": "engineering",
  "reports_to": "ceo"
}
```

**After:**
```json
{
  "id": "seedco-senior-fullstack-engineer",
  "name": "Alex Rivera",
  "role": "Senior Full-Stack Engineer",
  "stage": "seedco",
  "department": "engineering",
  "reports_to": "ceo"
}
```

### ⚠️ What to Watch For

- **Internal references.** If another profile's SOUL.md or README.md says *"escalate to Sarah Chen"*, update that mention too. Search your entire repo:
  ```
  grep -ri "Sarah Chen" profiles/
  ```
- **Runtime config files** (e.g., JSON, YAML, `.agent` files) that reference the name string. Update those in lockstep.
- **The `"id"` field** in `profiles.json` is a stable key — leave it alone unless you have a reason. If you do change it, update every place that references `"id"` (department READMEs, orchestration rules).

### ✅ Checklist for Renaming

- [ ] SOUL.md H1 updated
- [ ] README.md H1 updated
- [ ] profiles.json `"name"` updated
- [ ] All cross-references (other SOUL/README files) updated
- [ ] Runtime config files checked
- [ ] `"id"` changed (only if necessary) + all references updated

---

## 2. How to Adjust Voice / Tone

The "Voice" section in each SOUL.md defines how the agent communicates. You can change it per-profile or set a company-wide voice by editing a shared style template.

### Where Voice Lives

Every SOUL.md has a block like this:

```markdown
## Voice

- **Tone:** Direct, constructive, optimistic
- **Formality:** Moderate — uses professional language but avoids jargon
- **Temper:** Calm under pressure, assumes good intent
- **Communication style:** Bullet points for clarity, short paragraphs for context
- **Quirks:** Occasionally uses startup metaphors ("ship it," "unblock me")
```

### Before / After Examples

#### Making It More Formal (Enterprise / Regulated Industry)

**Before (default startup voice):**
```markdown
## Voice

- **Tone:** Direct, constructive, optimistic
- **Formality:** Moderate — professional but approachable
- **Communication style:** Concise, uses bullet points, occasional casual language
- **Quirks:** References "shipping," "sprints," "unblocking"
```

**After (enterprise-formal):**
```markdown
## Voice

- **Tone:** Precise, diplomatic, solution-oriented
- **Formality:** Formal — complete sentences, no contractions, industry-standard terminology
- **Communication style:** Structured arguments, context-first, numbered recommendations
- **Quirks:** Prefers data over opinions, frames trade-offs as risk assessments
```

#### Making It More Casual (Consumer Tech / Early Stage)

**Before (default):**
```markdown
## Voice

- **Tone:** Direct, constructive, optimistic
- **Formality:** Moderate
```

**After (casual):**
```markdown
## Voice

- **Tone:** Friendly, direct, energetic
- **Formality:** Casual — uses contractions, can crack a joke, startup-native language
- **Communication style:** Short and punchy, emoji where appropriate, TL;DR summaries
- **Quirks:** Calls out assumptions, default yes on experiments
```

#### Making It More Technical (Deep Tech / Infrastructure)

**Before (default):**
```markdown
## Voice

- **Tone:** Direct, constructive, optimistic
```

**After (technical):**
```markdown
## Voice

- **Tone:** Precise, evidence-backed, intellectually honest
- **Formality:** Technical — uses domain terminology correctly, explains when needed
- **Communication style:** Data-first: metrics, benchmarks, confidence intervals
- **Quirks:** Over-explains edge cases, pushes back on fuzzy requirements
```

### Bulk Voice Updates

For company-wide tone changes, edit the voice template in a shared config or use search-and-replace across all SOUL.md files:

```
# Replace casual voice quirks with formal ones across all profiles
find profiles/ -name "SOUL.md" -exec sed -i 's/ship it/execute/g' {} \;
```

---

## 3. How to Add Industry-Specific Context

The default profiles are startup-agnostic. To make them feel native to your industry, you inject context into three sections of each SOUL.md:

1. **Role / Summary** — Replace generic startup language with industry-specific framing.
2. **Key Responsibilities** — Add sector-specific duties.
3. **Push-Back Notes** (if present) — Add compliance or domain-specific guardrails.

### Example: Fintech for a Senior Full-Stack Engineer

**Before (generic):**
```markdown
# Sarah Chen

**Role:** Senior Full-Stack Engineer at SeedCo

## Summary

Full-stack engineer who builds customer-facing features end-to-end.
Owns the frontend architecture and bridges product and backend.
```

**After (fintech):**
```markdown
# Alex Rivera

**Role:** Senior Full-Stack Engineer at SeedFin

## Summary

Full-stack engineer specializing in payment flow UX and financial
data visualization. Owns the frontend architecture for KYC onboarding,
transaction history, and compliance dashboards. Bridges product,
back-end, and compliance teams.
```

**After — Key Responsibilities (fintech additions in bold):**
```markdown
## Key Responsibilities

- Build and maintain customer-facing payment flows (checkout, transfers, statements)
- **Implement KYC/KYB onboarding UI with document upload and verification status**
- **Ensure transaction history views meet audit trail requirements**
- **Build compliance dashboards for the risk team**
- Bridge product requirements with backend API design
- Maintain design system components
```

**After — Push-Back Notes (fintech additions):**
```markdown
## When to Push Back

- Scope creep that compromises **regulatory deadlines (P27, SOC2 audit windows)**
- Feature requests that bypass **compliance review**
- **Any UX that obscures fee disclosures or regulatory disclaimers**
```

### Example: Healthcare for a Product Manager

**Before (generic Key Responsibilities):**
```markdown
## Key Responsibilities

- Define and prioritize the product roadmap
- Write PRDs and user stories
- Run user research sessions
- Coordinate with engineering on sprint planning
```

**After (healthcare):**
```markdown
## Key Responsibilities

- Define and prioritize the product roadmap with **HIPAA compliance as a non-negotiable constraint**
- Write PRDs and user stories **with PHI/PII handling requirements baked in**
- Run user research sessions **with IRB-aligned consent workflows**
- Coordinate with engineering on sprint planning **and security review cycles**
```

### Where to Add Industry Context Per Profile

| SOUL.md Section | What to Change |
|---|---|
| `# Title` / `**Role:**` | Add domain tag (e.g., "at PayFlow Inc.") |
| `## Summary` | Lead with the industry problem the agent solves |
| `## Key Responsibilities` | Add 2–4 industry-specific bullets |
| `## Knowledge` / `## Tools` | Add domain-specific tools (Stripe, Epic, AWS HealthLake) |
| `## When to Push Back` | Add regulatory or compliance guardrails |
| `## Voice` | Adjust formality to match industry norms |

---

## 4. How to Add New Profiles

Use this workflow to add a brand-new persona to the crew.

### Step 1: Choose the Right Location

The repo organizes profiles by company stage then department:

```
profiles/
├── seedco/           # Pre-product-market fit (~5-10 people)
│   ├── ceo/
│   ├── engineering/
│   ├── product/
│   └── go-to-market/
├── growthco/         # Post-PMF, scaling (~10-50 people)
│   ├── ceo/
│   ├── engineering/
│   ├── product/
│   ├── go-to-market/
│   └── operations/
└── scaleco/          # Multi-team, established (~50+ people)
    ├── executive/
    ├── engineering/
    ├── product/
    ├── marketing/
    ├── sales/
    ├── operations/
    └── data/
```

Decide which stage and department the new profile belongs to.

### Step 2: Create the Directory

```bash
# Example: Add a "Data Engineer" to GrowthCo engineering
mkdir -p profiles/growthco/engineering/data-engineer
```

### Step 3: Write `SOUL.md`

Follow the established format. Every SOUL.md has this structure:

```markdown
# Agent Name

**Role:** Specific Role Title

## Summary
2–3 sentences explaining what this agent does and why it exists.

## Key Responsibilities
- Bullet list of 5–8 primary duties
- Each bullet is a concrete action

## Knowledge
- Domain expertise this agent has
- Internal systems it knows about
- Industry context it understands

## Tools / Integrations
- Tools this agent can use or request
- APIs, databases, SaaS products

## Voice
- **Tone:** ...
- **Formality:** ...
- **Communication style:** ...
- **Quirks:** ...

## Escalation Path
- Who this agent reports to
- When to escalate vs. decide independently

## When to Push Back
- Situations where this agent pushes back
- Ethical guardrails
- Quality standards

## Working with Other Agents
- Key peer relationships
- What this agent provides vs. consumes
```

**Important:** Copy the exact section headings and formatting from an existing profile in the same department. Consistency matters — the runtime may parse specific section headers.

### Step 4: Write `README.md`

The README is the human-friendly version. Keep the same structure but slightly more explanatory:

```markdown
# Agent Name

**Role:** Specific Role Title | Stage | Department

## Overview

A longer narrative version of the summary. Why this role exists,
what gap it fills, what success looks like.

## When to Deploy This Agent

- Scenario 1: ...
- Scenario 2: ...
- Scenario 3: ...

## Dependencies

- **Reports to:** [Who]
- **Works closely with:** [Peer agents]
- **Provides:** [Deliverables this agent produces]
- **Consumes:** [Input it needs from other agents]

## Key Behaviors

- Behavior 1
- Behavior 2
```

### Step 5: Register in `profiles.json`

Add a new entry following the exact shape of existing entries:

```json
{
  "id": "growthco-data-engineer",
  "name": "Maya Patel",
  "role": "Data Engineer",
  "stage": "growthco",
  "department": "engineering",
  "reports_to": "growthco-vp-engineering",
  "enabled": true,
  "tags": ["data", "engineering", "analytics"]
}
```

### Step 6: Reference in Department README

Update `profiles/growthco/engineering/README.md` to list the new profile:

```markdown
## Engineering Team

- Maya Patel — Data Engineer (new)
- (existing profiles listed here)
```

### ✅ Checklist for Adding a New Profile

- [ ] Directory created under correct stage + department
- [ ] SOUL.md written with all required sections (match an existing profile's structure exactly)
- [ ] README.md written with overview, dependencies, and key behaviors
- [ ] profiles.json entry added with unique `"id"`
- [ ] Department README.md updated to list the new profile
- [ ] Cross-references checked (other profiles that should mention this new role)
- [ ] Voice section matches the stage-appropriate tone
- [ ] `reports_to` ID in profiles.json points to an existing profile

---

## 5. How to Adjust Reporting Lines

Reporting lines are defined in **two places** that must stay in sync.

### Where Reporting Lines Live

| File | Field | What It Does |
|---|---|---|
| `profiles.json` | `"reports_to"` | The `"id"` of the manager profile |
| `SOUL.md` | `## Escalation Path` | Human-readable description of the reporting relationship |

### Example: Change Who a Profile Reports To

Say the default has the Senior Full-Stack Engineer reporting to the CEO (SeedCo). You want them reporting to a new VP of Engineering instead.

**Before — `profiles.json`:**
```json
{
  "id": "seedco-senior-fullstack-engineer",
  "name": "Alex Rivera",
  "role": "Senior Full-Stack Engineer",
  "reports_to": "seedco-ceo"
}
```

**After:**
```json
{
  "id": "seedco-senior-fullstack-engineer",
  "name": "Alex Rivera",
  "role": "Senior Full-Stack Engineer",
  "reports_to": "seedco-vp-engineering"
}
```

**Before — `SOUL.md`:**
```markdown
## Escalation Path

- **Reports to:** CEO (Jamie Kwan)
- **Escalate when:** Technical debt threatens roadmap delivery, or architectural decisions need founder input
```

**After:**
```markdown
## Escalation Path

- **Reports to:** VP of Engineering (Taylor Reed)
- **Escalate when:** Technical debt threatens roadmap delivery, architectural decisions need sign-off, or cross-team coordination stalls
```

### ⚠️ What to Watch For

- **Circular reporting.** If A reports to B and B reports to A, the runtime loops. Do a quick sanity check after changes.
- **Orphan profiles.** Every profile's `reports_to` must point to an existing `"id"` in `profiles.json`. If you remove a manager, update all their reports.
- **Department consistency.** If you move someone to a different manager, make sure the department README still makes sense. Update the department header if needed.
- **Stage boundaries.** A SeedCo profile reporting to a ScaleCo profile may confuse stage-specific logic. Stay within stage unless you have a cross-stage workflow.

### Changing the Org Structure (Batch)

If you're adopting a completely different org chart, the fastest approach is:

1. Audit every `"reports_to"` in `profiles.json` against your real org.
2. Update each one in a single pass.
3. Regenerate or bulk-edit the `## Escalation Path` sections in SOUL.md files.
4. Update department READMEs to reflect the new structure.

---

## 6. How to Adjust Stage Mapping

You have 25+ profiles spread across three stages. If your company has 8 people and doesn't fit neatly into one stage, here's how to choose.

### The Three Stages at a Glance

| Stage | Company Size | What It Models |
|---|---|---|
| **SeedCo** | ~5–10 people | Pre-product-market fit. Generalists. Founder-led. |
| **GrowthCo** | ~10–50 people | Post-PMF, early scale. First specialists, first VPs. |
| **ScaleCo** | ~50–200 people | Multi-team. Deep specialists. Formal departments. |

### Strategy 1: Pick the Closest Stage, Then Add/Remove

If you're a 12-person company, **GrowthCo** is your closest match. Take all GrowthCo profiles, then:

- **Remove** profiles you don't need (e.g., if you have no data team yet, drop the Data Engineer).
- **Add** 1–2 SeedCo profiles that fill gaps (e.g., if you're still founder-led in some areas, keep the SeedCo CEO profile).
- **Rename** profiles to match your actual people.

### Strategy 2: Mix and Match (Most Common for 8-Person Teams)

Most 8-person teams look like this hybrid:

| Department | Use Stage | Rationale |
|---|---|---|
| CEO / Founder | **SeedCo** | Founder still does everything |
| Engineering (2–3) | **GrowthCo** | Already have specialists |
| Product (1) | **GrowthCo** | First dedicated PM |
| Marketing (1) | **SeedCo** or **GrowthCo** | First marketer, still generalist |
| Sales (1) | **SeedCo** | Founder-led sales still |
| Operations (1) | **GrowthCo** | First ops hire |

### Quick Selection Matrix

| Your Company Profile | Best Base Stage | Modifications |
|---|---|---|
| 5–7 people, pre-revenue | SeedCo — all profiles | Trim departments you don't have |
| 8–15 people, early revenue | GrowthCo base + SeedCo CEO | Add SeedCo founder profiles, remove unneeded specialists |
| 15–30 people, scaling | GrowthCo — all profiles | May borrow 1–2 ScaleCo profiles for deep specialists |
| 30+ people | ScaleCo base + GrowthCo bridge | Keep a few GrowthCo profiles for roles that are still evolving |

### Disabling Profiles You Don't Need

In `profiles.json`, set `"enabled": false` rather than deleting entries. This preserves the structure and makes it easy to re-enable later:

```json
{
  "id": "growthco-data-engineer",
  "name": "Maya Patel",
  "enabled": false
}
```

The runtime will skip disabled profiles. Remove the entry entirely only when you're certain you'll never use it.

---

## 7. How to Maintain Consistency

When you write or edit profiles, follow these standards so the crew stays coherent.

### Voice Standards

| Attribute | SeedCo | GrowthCo | ScaleCo |
|---|---|---|---|
| **Tone** | Energetic, direct, scrappy | Constructive, strategic | Polished, data-driven, executive |
| **Formality** | Low–moderate | Moderate | Moderate–high |
| **Decision scope** | "Figure it out" | "Recommend with data" | "Present options with risk analysis" |
| **Quirks** | Startup idioms, optimistic | Process-aware,偶尔 casual | Industry terms, measured |

### Principle Formatting

Principles (e.g., "Move fast," "Default open") appear in some profiles. Always format as:

```markdown
## Principles

1. **Move Fast, Stay Solvent** — Speed is our edge, but not at the cost of runway.
2. **Default Open** — Share early, share often. Information silos are the enemy.
3. **Strong Opinions, Weakly Held** — Advocate for your view, then pivot when data says otherwise.
```

Number them. Bold the principle name. Follow with a one-sentence explanation.

### Research Method Conventions

When a profile includes how it gathers information, use this structure:

```markdown
## Research Methods

| Method | Used When | Output |
|---|---|---|
| User interviews | Validating a new feature | Thematic analysis doc |
| Data analysis | Investigating a metric drop | Dashboard + written summary |
| Competitive audit | Entering a new market | Competitive landscape doc |
| Technical spike | Evaluating a new technology | RFC with recommendations |
```

### Push-Back Section Conventions

Every `## When to Push Back` section follows this logic:

```markdown
## When to Push Back

- **[Condition]** — [Why it matters]
- **[Condition]** — [Why it matters]
```

Conditions are concrete, not vague. Example:

- **"Scope that adds 2+ weeks without a corresponding timeline adjustment"** (not "unrealistic deadlines")
- **"Requests that bypass security review"** (not "bad ideas")

### Profile Completeness Checklist

Every profile must have all of these sections in SOUL.md:

- [ ] `# Name` — H1 with the agent's name
- [ ] `**Role:**` — Bold role line right after the H1
- [ ] `## Summary` — 2–3 sentences
- [ ] `## Key Responsibilities` — 5–8 bullets
- [ ] `## Knowledge` — 3–6 domain areas
- [ ] `## Tools / Integrations` — 3–6 tools
- [ ] `## Voice` — All 4 sub-attributes (Tone, Formality, Communication style, Quirks)
- [ ] `## Escalation Path` — Who they report to, when to escalate
- [ ] `## When to Push Back` — 3–5 concrete conditions
- [ ] `## Working with Other Agents` — 3–5 peer relationships

Every profile must have these in README.md:

- [ ] `# Name` — H1 matching SOUL.md
- [ ] `## Overview` — 3–5 sentences
- [ ] `## When to Deploy This Agent` — 3–5 scenarios
- [ ] `## Dependencies` — Reports to, works with, provides, consumes
- [ ] `## Key Behaviors` — 3–5 behaviors

### Style Rules

| Rule | Correct | Incorrect |
|---|---|---|
| Use **bold** for labels, not code ticks | `**Reports to:**` | `Reports to:` |
| Use `##` for sections, never `#` after the title | `## Summary` | `# Summary` |
| Bullets end with a period only if multiple sentences | No period for single sentence | Random periods |
| Names in SOUL.md are human names | `Sarah Chen` | `senior-fullstack-engineer` |
| profile.json `"id"` uses kebab-case | `seedco-vp-engineering` | `SeedCo_VP_Engineering` |

---

## 8. Customizing for Different Industries — Quick Reference

Use this table to know what changes per industry. The key insight: **every industry changes the same five sections** — Knowledge, Tools, Voice formality, Push-Back guardrails, and Key Responsibilities.

| Section to Change | Fintech | Healthcare | Enterprise SaaS | Consumer Tech |
|---|---|---|---|---|
| **Knowledge** | PCI-DSS, KYC/KYB, AML, P27, SOC2, fraud patterns | HIPAA, HITECH, FHIR, HL7, PHI/PII handling, ICD-10, CPT codes | Okta/SAML/SSO, RBAC, SLA frameworks, enterprise sales cycles, procurement | Growth loops, viral mechanics, DAU/MAU metrics, cohort retention, A/B test design |
| **Tools** | Stripe, Plaid, Alloy, Chainalysis, TaxBit | Epic, Cerner, Health Gorilla, Twilio HIPAA, AWS HealthLake | Salesforce, HubSpot, Zendesk, Jira Service Management, Confluence | Amplitude, Mixpanel, Braze, Appcues, Segment |
| **Voice Formality** | High — regulatory disclosures require precise language | High — patient safety and legal liability demand formality | Moderate–High — professional but can be direct | Low–Moderate — casual, fast, experimental |
| **Push-Back Guardrails** | Never bypass compliance; never obscure fees; never approve a release without legal sign-off | Never expose PHI; never bypass BAA; never approve a clinical workflow without clinician input | Never skip security review; never over-promise in responses; never bypass procurement | Never ship without experiment results; never break the core loop; never ignore user feedback signals |
| **Key Responsibilities (additions)** | Implement payment flows, compliance dashboards, audit logs, fraud detection | Implement clinical workflows, HIPAA-compliant auth, FHIR API integrations, patient portal UX | Implement SSO/SAML, RBAC, admin dashboards, SLA monitoring, enterprise onboarding flows | Implement onboarding funnels, referral mechanics, A/B test frameworks, real-time collaboration features |
| **Regulatory Overhead** | High — compliance is a feature | High — patient safety and privacy are non-negotiable | Medium — enterprise security reviews add cycles | Low — speed to market > compliance |
| **Escalation Speed** | Medium — compliance can slow things down | Medium — clinical review cycles add lead time | Medium — enterprise sales cycles dictate rhythm | Fast — move fast, break things (within reason) |

### Industry-Specific Profile Examples

**Fintech — "When to Push Back" addition:**
```markdown
- **Regulatory shortcut** — Never skip compliance review for speed. Every user-facing financial feature needs legal sign-off before launch.
```

**Healthcare — "Knowledge" addition:**
```markdown
## Knowledge

- HIPAA Privacy Rule, Security Rule, Breach Notification Rule
- FHIR R4 API specification and SMART on FHIR authorization
- PHI/PII data classification and de-identification techniques
- BAA (Business Associate Agreement) requirements
```

**Enterprise SaaS — "Voice" adjustment:**
```markdown
## Voice

- **Tone:** Confident, consultative, ROI-focused
- **Formality:** Professional — enterprise stakeholders expect polished communication
- **Communication style:** Structured, executive-summary-first, data-backed
- **Quirks:** Avoids startup slang, frames everything in business value
```

**Consumer Tech — "Key Responsibilities" addition:**
```markdown
## Key Responsibilities

- Design and analyze A/B tests for onboarding, activation, and retention
- Build and monitor growth loop metrics (viral coefficient, time-to-value)
- Implement real-time collaboration features that drive network effects
- Optimize core user flows for conversion (sign-up, first action, referral)
```

---

## Appendix A: File Reference Cheat Sheet

| Action | Files to Modify | Runtime Impact |
|---|---|---|
| Rename a persona | SOUL.md + README.md + profiles.json | High — name is the lookup key |
| Change voice/tone | SOUL.md only | Low — cosmetic, no structural change |
| Add industry context | SOUL.md (multiple sections) | Low — additive, existing structure unchanged |
| Add a new profile | New dir + SOUL.md + README.md + profiles.json + department README | Medium — new entry, existing profiles unaffected |
| Change reporting line | profiles.json + SOUL.md | Medium — changes agent graph, check for orphans/cycles |
| Disable a profile | profiles.json (`enabled: false`) | Low — skipped at load time |
| Change stage mapping | profiles.json (re-assign `stage`) + move directory | Medium — path change, update all references |
| Bulk industry pivot | All SOUL.md files (5 sections) + profiles.json tags | High — systematic, test the runtime after |

## Appendix B: Validation Commands

Run these after any customization to catch issues early.

```bash
# 1. Verify all names match between SOUL.md and profiles.json
echo "=== Checking name consistency ==="
for profile in $(find profiles/ -name "SOUL.md"); do
  soul_name=$(head -1 "$profile" | sed 's/^# //')
  json_name=$(jq -r --arg path "$profile" '.[] | select(.name == $soul_name)' profiles.json)
  # Manual check recommended
done

# 2. Verify all reports_to references exist
echo "=== Checking for orphan profiles ==="
jq -r '.[].reports_to' profiles.json | sort -u | while read manager; do
  if ! jq -e --arg id "$manager" '.[] | select(.id == $id)' profiles.json > /dev/null; then
    echo "MISSING: $manager is referenced but has no profile"
  fi
done

# 3. Check for duplicate IDs
echo "=== Checking for duplicate IDs ==="
jq -r '.[].id' profiles.json | sort | uniq -d

# 4. Verify directory structure matches stage/department consistency
echo "=== Checking directory vs. JSON stage/department ==="
jq -r '.[] | "\(.id) \(.stage)/\(.department)/\(.name | gsub(" "; "-") | ascii_downcase)"' profiles.json
```

---

> **Need help?** Open an issue in the agent-crew repo or contact support. We can also do custom profile packs for specific industries — reach out for a quote.
