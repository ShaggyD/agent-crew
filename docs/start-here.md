# Start Here

Welcome to agent-crew. You've just acquired a library of 25+ agent personas organized as three companies at three stages. This guide gets you from zero to your first running crew in under 30 minutes.

## What you bought

Three things, really:

**Profiles.** 25+ agent personas, each with a SOUL.md (full character: voice, principles, push-back triggers) and a README.md (quick reference: when to use, what problems they solve). Every profile is designed to argue with you — these agents don't say "yes" to bad ideas.

**Stages.** The profiles are organized as three companies — SeedCo (5 people), GrowthCo (15 people), ScaleCo (50 people) — so you can pick the org structure that fits your current size.

**Interactions.** Cross-team workflows, communication protocols, and escalation paths that adapt to each stage.

## Quick start (5 min)

### 1. Pick your size

How many people are in your crew?

| Your size | Start here | Profiles |
|---|---|---|
| 1-8 people | [SeedCo stage →](../stages/seed-co.md) | 5 scrappy generalists |
| 8-25 people | [GrowthCo stage →](../stages/growth-co.md) | 14 specialists in departments |
| 25-80 people | [ScaleCo stage →](../stages/scale-co.md) | 25+ with management layer |

Don't see your exact size? Read the [Customization Guide](customization-guide.md#6-how-to-adjust-stage-mapping) for mix-and-match advice.

### 2. Read the org chart

Open your stage doc. Look at the org chart. Understand who reports to whom, how decisions flow, and what the key dynamics are at that size.

Each stage doc includes a **personality profile table** showing how conservative/optimistic the whole company is across resource allocation, technical decisions, process, risk tolerance, and communication.

### 3. Open a few profiles

Pick two or three roles that matter most to you right now. Read their SOUL.md. Pay attention to:

- **What they push back on** — This is the most important section. It tells you when this agent will disagree with you, and that's the whole point of these profiles.
- **How they research** — This is how the agent reaches conclusions. Different research methods create productive tension.
- **Principles** — These are non-negotiable. If an agent violates its own principles, something is wrong.

### 4. Choose your framework

Your profiles are in markdown. They work with any agent system. Ready-made integrations:

| Framework | Guide | What's included |
|---|---|---|
| **CrewAI** | [Integration Guide](integration-guide.md#3-crewai-integration) | Field mapping, profile loader, full 3-agent crew example |
| **AutoGen** | [Integration Guide](integration-guide.md#4-autogen-integration) | System message builder, push-back conversation patterns |
| **LangGraph** | [Integration Guide](integration-guide.md#5-langgraph-integration) | Supervisor-worker graph, push-back routing |
| **Any system** | [Integration Guide](integration-guide.md#2-generic-json-export) | JSON export script for custom harnesses |

### 5. Run your first crew

Copy the quick-start config from the Integration Guide for your framework. Swap in your API key. Run it. Watch two personas disagree productively.

That's the moment it clicks.

## The full reading path

When you're ready to go deeper:

```
Start Here (this doc)
  └── Your stage doc (stages/seed-co.md or growth-co.md or scale-co.md)
       ├── Read the org chart and dynamics
       ├── Open each profile in your department
       │    ├── SOUL.md (the personality)
       │    └── README.md (the quick reference)
       └── Read the interactions (interactions/...)
            ├── SDLC Pipeline
            ├── Communications Protocol
            └── Cross-Functional Workflows
```

## What makes these profiles different

Most agent profiles describe what an agent *does*. These describe who an agent *is*.

- **They push back.** Every profile has defined disagreement triggers. These agents challenge assumptions, argue for better solutions, and resist bad scope.
- **They research.** Every profile has a defined method for reaching conclusions. They don't guess — they look things up.
- **They have stage-appropriate tendencies.** SeedCo agents are more conservative with resources. ScaleCo agents think in systems and process. Each profile's bias is intentional.
- **They create productive friction.** A CEO who always agrees with the CTO is useless. These profiles are designed to generate healthy disagreement that leads to better decisions.

## If something doesn't work

| Problem | Solution |
|---|---|
| A persona doesn't fit my company culture | [Customization Guide → How to adjust voice/tone](customization-guide.md#2-how-to-adjust-voice--tone) |
| I need a role that doesn't exist yet | [Customization Guide → How to add new profiles](customization-guide.md#4-how-to-add-new-profiles) |
| The reporting structure is wrong for my org | [Customization Guide → How to adjust reporting lines](customization-guide.md#5-how-to-adjust-reporting-lines) |
| My crew has 8 people, not 5 or 15 | [Customization Guide → How to adjust stage mapping](customization-guide.md#6-how-to-adjust-stage-mapping) |
| I want to export profiles as JSON | [Integration Guide → JSON Export](integration-guide.md#2-generic-json-export) |

## What to read next

| If you want to... | Read this |
|---|---|
| See the full catalog of profiles | [profiles.json](../profiles.json) |
| Understand decision making and escalation | [Reporting Lines](../stages/reporting-lines.md) |
| Adapt the crew to your needs | [Customization Guide](customization-guide.md) |
| Hook profiles into your agent framework | [Integration Guide](integration-guide.md) |
