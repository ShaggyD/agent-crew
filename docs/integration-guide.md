# Agent-Crew Integration Guide

> Use agent-crew personas — the SOUL.md + README.md format — in CrewAI, AutoGen (Microsoft), and LangGraph.

This guide shows you exactly how to take any profile from this repository and wire it into three major agent frameworks. Every example is designed to be copy-paste ready with obvious places to plug in your own API keys.

---

## Table of Contents

1. [Profile Format Overview](#1-profile-format-overview)
2. [Generic JSON Export](#2-generic-json-export)
3. [CrewAI Integration](#3-crewai-integration)
4. [AutoGen (Microsoft) Integration](#4-autogen-microsoft-integration)
5. [LangGraph Integration](#5-langgraph-integration)
6. [Quick-Start Config Blocks](#6-quick-start-config-blocks)
7. [Tips & Troubleshooting](#7-tips--troubleshooting)

---

## 1. Profile Format Overview

Every profile in agent-crew has exactly **two files**:

| File | Purpose |
|------|---------|
| `SOUL.md` | Full persona — voice, communication style, push-back triggers, research methods, operating principles |
| `README.md` | Quick overview — one-line description, when to use, problems solved, warning signs |

### SOUL.md canonical fields

```yaml
# Metadata (always present at top)
Name:     Full name and pronouns
Role:     Job title
Department: Department name
Reports to: Manager or "no one"

# Persona sections (always in this order)
Voice:             Tagline and extended persona description
Communication style: Bullet list of communication traits
What they push back on: Key triggers that cause disagreement (critical for multi-agent setups)
How they research: Numbered list of research methodology
Principles:        Numbered list of operating principles

# Optional
Stage context:     Stage-specific behavior notes (GrowthCo vs ScaleCo variants)
Conservatism vs optimism: SeedCo profiles only — what they're conservative/optimistic about
```

### How to read a profile

```python
# Example: growth-co/engineering/software-engineer/SOUL.md
#
# "Jordan Kwon" — Software Engineer
# Voice: Direct, practical, slightly weary of over-engineering
# Pushes back on: over-engineered solutions, specs without AC, untested changes
# Research: checks existing solutions → API docs → production patterns → failure data
# Principles: readable code, done > perfect, tests are not optional
```

**Every profile is designed to disagree productively.** That's not a bug — it's the feature. These agents debate, push back, and challenge assumptions, which produces better outcomes in multi-agent setups.

---

## 2. Generic JSON Export

Use this script to convert any `SOUL.md` profile into a machine-readable JSON structure. This is the foundation you'll use to load profiles into any framework.

### Script: `scripts/export_profile.py`

```python
#!/usr/bin/env python3
"""
export_profile.py — Convert SOUL.md files to JSON.

Usage:
    python export_profile.py path/to/profile/SOUL.md
    python export_profile.py --all profiles.json

Dependencies:
    pip install pyyaml  # not strictly required; we use pure Python parsing
"""

import json
import os
import re
import sys
from pathlib import Path


def parse_soul_md(filepath):
    """Parse a SOUL.md into a structured dict."""
    text = Path(filepath).read_text(encoding="utf-8")

    profile = {
        "source_file": str(filepath),
        "profile_path": str(Path(filepath).parent),
        "name": "",
        "pronouns": "",
        "role": "",
        "department": "",
        "reports_to": "",
        "tagline": "",
        "voice_intro": "",
        "communication_style": [],
        "push_back_on": [],
        "research_methods": [],
        "principles": [],
        "stage_context": "",
    }

    # --- Name & Pronoun ---
    # Format: "# Name — Role" on line 1
    first_line = text.split("\n")[0]
    if "—" in first_line:
        name_part = first_line.lstrip("#").strip().split("—")[0].strip()
        # Some have parenthetical pronouns
        m = re.match(r"^(.*?)\s*\(([^)]+)\)", name_part)
        if m:
            profile["name"] = m.group(1).strip()
            profile["pronouns"] = m.group(2).strip()
        else:
            profile["name"] = name_part

    # --- Tagline (blockquote after heading) ---
    tagline_match = re.search(r">\s*(.+?)(?:\n|$)", text)
    if tagline_match:
        profile["tagline"] = tagline_match.group(1).strip()

    # --- Metadata lines (Name:, Role:, Department:, Reports to:) ---
    for line in text.split("\n"):
        line = line.strip()
        if line.startswith("**Name:**"):
            # May contain pronouns e.g., "Jordan Kwon (he/him)"
            val = line.replace("**Name:**", "").strip()
            pm = re.match(r"^(.*?)\s*\(([^)]+)\)", val)
            if pm:
                profile["name"] = pm.group(1).strip()
                profile["pronouns"] = pm.group(2).strip()
            else:
                profile["name"] = val
        elif line.startswith("**Role:**"):
            profile["role"] = line.replace("**Role:**", "").strip()
        elif line.startswith("**Department:**"):
            profile["department"] = line.replace("**Department:**", "").strip()
        elif line.startswith("**Reports to:**"):
            profile["reports_to"] = line.replace("**Reports to:**", "").strip()

    # --- Voice paragraph ---
    voice_section = _extract_section(text, "## Voice", "## Communication style")
    if voice_section:
        lines = voice_section.strip().split("\n")
        profile["voice_intro"] = lines[0].strip() if lines else ""
        # The blockquote lines after voice are quotes — store them
        for line in lines:
            line_stripped = line.strip()
            if line_stripped.startswith(">") and "When" in line_stripped:
                profile.setdefault("voice_quotes", []).append(
                    line_stripped.lstrip(">").strip().strip('"')
                )

    # --- Communication style ---
    comm_section = _extract_section(text, "## Communication style", "## What they push back on")
    if comm_section:
        for line in comm_section.split("\n"):
            line = line.strip()
            if line.startswith("-") or line.startswith("*"):
                profile["communication_style"].append(line.lstrip("-* ").strip())

    # --- Push back on ---
    push_section = _extract_section(text, "## What they push back on", "## How they research")
    if push_section:
        for line in push_section.split("\n"):
            line = line.strip()
            if line.startswith("-") or line.startswith("*"):
                profile["push_back_on"].append(line.lstrip("-* ").strip())

    # --- How they research ---
    research_section = _extract_section(text, "## How they research", "## Principles")
    if research_section:
        for line in research_section.split("\n"):
            line = line.strip()
            if line and (line[0].isdigit() or line.startswith("-")):
                cleaned = re.sub(r"^\d+\.\s*", "", line).strip()
                cleaned = cleaned.lstrip("-* ").strip()
                if cleaned:
                    profile["research_methods"].append(cleaned)

    # --- Principles ---
    principles_section = _extract_section(text, "## Principles", "##")
    if not principles_section:
        # Some files have principles at the end with no following ##
        principles_section = _extract_section(text, "## Principles", None, to_end=True)
    if principles_section:
        for line in principles_section.split("\n"):
            line = line.strip()
            if line and (line[0].isdigit() or line.startswith("-")):
                cleaned = re.sub(r"^\d+\.\s*", "", line).strip()
                cleaned = cleaned.lstrip("-* ").strip()
                if cleaned:
                    profile["principles"].append(cleaned)

    # --- Stage context ---
    stage_section = _extract_section(text, "## Stage context", "##", optional=True)
    if stage_section:
        profile["stage_context"] = stage_section.strip()

    return profile


def _extract_section(text, start_heading, end_heading, to_end=False, optional=False):
    """Extract text between two markdown headings."""
    start_idx = text.find(start_heading)
    if start_idx == -1:
        return "" if optional else ""

    # Move past the heading line
    start_of_content = text.find("\n", start_idx)
    if start_of_content == -1:
        return ""
    start_of_content += 1

    if to_end or end_heading is None:
        return text[start_of_content:].strip()

    end_idx = text.find(end_heading, start_of_content)
    if end_idx == -1:
        return text[start_of_content:].strip()

    return text[start_of_content:end_idx].strip()


def export_all_profiles(base_dir="."):
    """Walk the repo and export every SOUL.md found."""
    base = Path(base_dir).resolve()
    profiles = []
    for soul_file in sorted(base.rglob("SOUL.md")):
        # Skip files outside the stage directories (e.g., obsidian symlinks)
        rel = soul_file.relative_to(base)
        if not any(part in str(rel) for part in ("seed-co", "growth-co", "scale-co")):
            continue
        try:
            data = parse_soul_md(soul_file)
            profiles.append(data)
        except Exception as e:
            print(f"Warning: failed to parse {soul_file}: {e}", file=sys.stderr)
    return profiles


if __name__ == "__main__":
    if len(sys.argv) > 1 and sys.argv[1] == "--all":
        profiles = export_all_profiles()
        print(json.dumps(profiles, indent=2, ensure_ascii=False))
    elif len(sys.argv) > 1:
        data = parse_soul_md(sys.argv[1])
        print(json.dumps(data, indent=2, ensure_ascii=False))
    else:
        print("Usage:")
        print("  python export_profile.py path/to/SOUL.md")
        print("  python export_profile.py --all")
        sys.exit(1)
```

### Example JSON output (abbreviated)

```json
{
  "source_file": "/home/user/agent-crew/growth-co/engineering/software-engineer/SOUL.md",
  "profile_path": "/home/user/agent-crew/growth-co/engineering/software-engineer",
  "name": "Jordan Kwon",
  "pronouns": "he/him",
  "role": "Software Engineer",
  "department": "Engineering",
  "reports_to": "CTO",
  "tagline": "I write code that works, is readable, and can be deployed. Done is better than perfect, but I won't ship broken.",
  "voice_intro": "Direct, practical, slightly weary of over-engineering. Jordan has been around long enough to recognize when someone is over-complicating a solution.",
  "communication_style": [
    "Pragmatic above all.",
    "Reviews code with empathy.",
    "Calls out untested assumptions.",
    "Measured about estimates."
  ],
  "push_back_on": [
    "Over-engineered solutions.",
    "Specs with no acceptance criteria.",
    "\"Just hack it in\" as a permanent solution.",
    "Untested changes.",
    "Meetings that could be async."
  ],
  "research_methods": [
    "Checks package registries, open source, and internal patterns before building from scratch.",
    "Reads the actual docs, not blog posts about the docs.",
    "Looks at how well-maintained projects solve similar problems.",
    "Investigates past incidents before planning changes to the affected code."
  ],
  "principles": [
    "Readable code is the highest form of performance.",
    "Done is better than perfect. Broken is worse than either.",
    "If you can't name it honestly, you don't understand it yet.",
    "Tests are not optional.",
    "Simple is harder than complex.",
    "Ship early, ship often, but ship safely."
  ]
}
```

---

## 3. CrewAI Integration

CrewAI v0.50+ uses a simple `Agent` class with `role`, `goal`, and `backstory`. The SOUL.md profile maps directly to these fields.

### Field mapping

| SOUL.md field | CrewAI Agent param | How to transform |
|---|---|---|
| Name + Role | `role` | `"Jordan Kwon — Software Engineer"` |
| Tagline | `goal` | The blockquote line sums up the agent's purpose |
| Voice + Communication + Principles | `backstory` | Concatenate into a narrative paragraph |
| Push-back traits | `backstory` (embedded) | Weave into the backstory as "They push back on…" |
| Research methods | `backstory` (embedded) | Add "When researching, they…" |
| *N/A* | `allow_delegation` | `True` for leads, `False` for ICs |
| *N/A* | `verbose` | `True` for debugging |

### Example: Loading a profile from SOUL.md

```python
# crewai_profile_loader.py
import json
from pathlib import Path


def load_profile(soul_path: str) -> dict:
    """Load a parsed SOUL.md into CrewAI-compatible config dict."""
    # Use the export script logic (inlined here for simplicity)
    from export_profile import parse_soul_md
    data = parse_soul_md(soul_path)

    # Build backstory from voice, communication style, push-back, and principles
    backstory_parts = [data.get("voice_intro", "")]
    if data.get("communication_style"):
        backstory_parts.append(
            "Communication style: " + "; ".join(data["communication_style"])
        )
    if data.get("push_back_on"):
        backstory_parts.append(
            "Pushes back on: " + "; ".join(data["push_back_on"])
        )
    if data.get("principles"):
        backstory_parts.append(
            "Principles: " + "; ".join(data["principles"])
        )

    return {
        "role": f"{data['name']} — {data['role']}",
        "goal": data.get("tagline", f"Excel as {data['role']}"),
        "backstory": " ".join(backstory_parts),
        "allow_delegation": data["role"].lower() in ("cto", "ceo", "product manager", "vp engineering"),
    }


# Example usage
if __name__ == "__main__":
    config = load_profile(
        "../growth-co/engineering/software-engineer/SOUL.md"
    )
    print(json.dumps(config, indent=2))
```

### Full 3-Agent Crew Example

Below is a working CrewAI pipeline using three GrowthCo profiles to plan, build, and validate a feature.

```python
#!/usr/bin/env python3
"""
crewai_example.py — Three-agent crew using agent-crew personas.

Run:    pip install crewai>=0.50
        python crewai_example.py

NOTE:   You need an OPENAI_API_KEY (or other LLM) set in your environment.
        crewai uses OpenAI by default. Set model via os.environ or Agent(kwargs).
"""

import os
from pathlib import Path

# CrewAI v0.50+
from crewai import Agent, Task, Crew, Process

# ---------------------------------------------------------------------------
# 1. Profile loader (inline — no external dependency)
# ---------------------------------------------------------------------------

def _extract(text, start, end=None, to_end=False):
    s = text.find(start)
    if s == -1:
        return ""
    s = text.find("\n", s)
    if s == -1:
        return ""
    s += 1
    if to_end or end is None:
        return text[s:].strip()
    e = text.find(end, s)
    return text[s:e].strip() if e != -1 else text[s:].strip()


def _bullets(text):
    return [
        ln.lstrip("-* ").strip()
        for ln in text.split("\n")
        if ln.strip().startswith(("-", "*"))
    ]


def _numbered(text):
    import re
    items = []
    for ln in text.split("\n"):
        ln = ln.strip()
        if ln and (ln[0].isdigit() or ln.startswith("-")):
            items.append(re.sub(r"^\d+\.\s*", "", ln).lstrip("-* ").strip())
    return items


def make_crewai_agent(name, role, tagline, voice, comm, pushback, research, principles):
    """Map persona fields to a CrewAI Agent dict."""
    backstory = f"{voice}\n\nCommunication style: {'; '.join(comm)}\n\n"
    if pushback:
        backstory += f"They push back on: {'; '.join(pushback)}\n\n"
    if research:
        backstory += f"When researching, they: {'; '.join(research)}\n\n"
    if principles:
        backstory += f"Operating principles: {'; '.join(principles)}"

    is_lead = any(t in role.lower() for t in ("cto", "ceo", "pm", "vp", "lead", "manager", "principal"))

    return {
        "role": f"{name} — {role}",
        "goal": tagline,
        "backstory": backstory.strip(),
        "allow_delegation": is_lead,
        "verbose": True,
    }


def load_persona(soul_path):
    """Read SOUL.md and return a dict for make_crewai_agent."""
    text = Path(soul_path).read_text(encoding="utf-8")

    # Name
    first = text.split("\n")[0].lstrip("#").strip()
    name = first.split("—")[0].strip() if "—" in first else first

    # Role from metadata
    role = ""
    for line in text.split("\n"):
        if line.startswith("**Role:**"):
            role = line.replace("**Role:**", "").strip()
            break

    # Tagline
    import re
    tm = re.search(r">\s*(.+?)(?:\n|$)", text)
    tagline = tm.group(1).strip() if tm else f"Excel as {role}"

    # Sections
    voice = _extract(text, "## Voice", "## Communication style").split("\n")[0].strip() if _extract(text, "## Voice", "## Communication style") else ""
    comm = _bullets(_extract(text, "## Communication style", "## What they push back on"))
    pushback = _bullets(_extract(text, "## What they push back on", "## How they research"))
    research = _numbered(_extract(text, "## How they research", "## Principles"))
    principles = _numbered(_extract(text, "## Principles", None, to_end=True))

    return make_crewai_agent(name, role, tagline, voice, comm, pushback, research, principles)


# ---------------------------------------------------------------------------
# 2. Load three profiles
# ---------------------------------------------------------------------------

REPO_ROOT = Path(__file__).resolve().parent.parent  # agent-crew/

pm_cfg = load_persona(str(REPO_ROOT / "growth-co" / "product" / "product-manager" / "SOUL.md"))
swe_cfg = load_persona(str(REPO_ROOT / "growth-co" / "engineering" / "software-engineer" / "SOUL.md"))
qa_cfg = load_persona(str(REPO_ROOT / "growth-co" / "engineering" / "qa-engineer" / "SOUL.md"))

# ---------------------------------------------------------------------------
# 3. Create CrewAI agents
# ---------------------------------------------------------------------------

product_manager = Agent(
    role=pm_cfg["role"],
    goal=pm_cfg["goal"],
    backstory=pm_cfg["backstory"],
    allow_delegation=pm_cfg["allow_delegation"],
    verbose=True,
)

software_engineer = Agent(
    role=swe_cfg["role"],
    goal=swe_cfg["goal"],
    backstory=swe_cfg["backstory"],
    allow_delegation=swe_cfg["allow_delegation"],
    verbose=True,
)

qa_engineer = Agent(
    role=qa_cfg["role"],
    goal=qa_cfg["goal"],
    backstory=qa_cfg["backstory"],
    allow_delegation=qa_cfg["allow_delegation"],
    verbose=True,
)

# ---------------------------------------------------------------------------
# 4. Define tasks
# ---------------------------------------------------------------------------

task_define = Task(
    description=(
        "Define a new 'Dark Mode' feature for the web app. "
        "Write a spec with acceptance criteria, success metrics, and scope boundaries. "
        "Be specific about what 'done' looks like. Push back if the request is vague."
    ),
    expected_output="A structured spec document with AC, metrics, and out-of-scope items.",
    agent=product_manager,
)

task_implement = Task(
    description=(
        "Implement the Dark Mode feature according to the PM's spec. "
        "Write clean, tested code. If the spec is ambiguous, ask for clarification "
        "rather than guessing. Include unit tests for the risky parts."
    ),
    expected_output="Working implementation with passing tests and a PR summary.",
    agent=software_engineer,
)

task_validate = Task(
    description=(
        "Review the Dark Mode implementation. Verify it meets the spec's acceptance criteria. "
        "Check for edge cases: accessibility contrast, color-blind safety, system preference detection. "
        "Report any issues with reproduction steps."
    ),
    expected_output="QA report: PASS/FAIL per AC, with reproduction steps for any failures.",
    agent=qa_engineer,
)

# ---------------------------------------------------------------------------
# 5. Run the crew (sequential: PM → SWE → QA)
# ---------------------------------------------------------------------------

crew = Crew(
    agents=[product_manager, software_engineer, qa_engineer],
    tasks=[task_define, task_implement, task_validate],
    process=Process.sequential,
    verbose=True,
)

# ---------------------------------------------------------------------------
# 6. Execute
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    print("=" * 60)
    print("Agent-Crew CrewAI Example")
    print("Personas: PM (Elena Torres), SWE (Jordan Kwon), QA (Samira Patel)")
    print("=" * 60)
    result = crew.kickoff()
    print("\n--- FINAL OUTPUT ---")
    print(result)
```

### Hierarchical crew variant

For a hierarchical workflow (manager coordinates workers):

```python
# Replace the Crew block above with:

crew = Crew(
    agents=[product_manager, software_engineer, qa_engineer],
    tasks=[task_define, task_implement, task_validate],
    process=Process.hierarchical,
    manager_agent=product_manager,  # PM acts as manager
    verbose=True,
)

# result = crew.kickoff()
```

---

## 4. AutoGen (Microsoft) Integration

AutoGen v0.2+ uses `ConversableAgent` where you control everything through `system_message`. The SOUL.md persona becomes a rich system prompt.

### Field mapping

| SOUL.md field | AutoGen equivalent | How to use |
|---|---|---|
| Voice + Tagline | `system_message` intro | Establishes the agent's identity |
| Communication style | `system_message` section | "Communicate like this: …" |
| Push-back traits | `system_message` section | "Push back when: …" |
| Research methods | `system_message` section | "Research approach: …" |
| Principles | `system_message` section | "Operating principles: …" |
| *Agent name* | `name` parameter | Use the persona's first name |

### Example: Two-agent conversation with push-back

```python
#!/usr/bin/env python3
"""
autogen_example.py — Two-agent conversation using agent-crew personas.

Run:    pip install pyautogen>=0.2
        python autogen_example.py

NOTE:   Set OPENAI_API_KEY or configure your LLM endpoint.
        AutoGen uses OpenAI-compatible APIs by default.
"""

import os
import re
from pathlib import Path

from autogen import ConversableAgent

# ---------------------------------------------------------------------------
# 1. Persona → system_message builder
# ---------------------------------------------------------------------------

def build_system_message(text: str) -> str:
    """Convert raw SOUL.md content into a system prompt for AutoGen."""

    def _get_section(start, end=None, to_end=False):
        s = text.find(start)
        if s == -1:
            return ""
        s = text.find("\n", s)
        if s == -1:
            return ""
        s += 1
        if to_end or end is None:
            return text[s:].strip()
        e = text.find(end, s)
        return text[s:e].strip() if e != -1 else text[s:].strip()

    def _bullets(heading, next_heading=""):
        section = _get_section(heading, next_heading)
        items = []
        for ln in section.split("\n"):
            ln = ln.strip()
            if ln.startswith(("-", "*")):
                items.append(ln.lstrip("-* ").strip())
        return items

    def _get_tagline():
        m = re.search(r">\s*(.+?)(?:\n|$)", text)
        return m.group(1).strip() if m else ""

    # Extract fields
    first_line = text.split("\n")[0].lstrip("#").strip()
    name = first_line.split("—")[0].strip() if "—" in first_line else first_line

    role = ""
    for line in text.split("\n"):
        if line.startswith("**Role:**"):
            role = line.replace("**Role:**", "").strip()
            break

    tagline = _get_tagline()
    voice_intro = _get_section("## Voice", "## Communication style").split("\n")[0].strip() if _get_section("## Voice", "## Communication style") else ""
    comm = _bullets("## Communication style", "## What they push back on")
    pushback = _bullets("## What they push back on", "## How they research")

    # Build the system message
    parts = [
        f"You are {name}, {role}.",
        f"Your motto: {tagline}" if tagline else "",
        "",
        voice_intro,
        "",
        "## Communication Style",
    ]
    for c in comm:
        parts.append(f"- {c}")

    parts += ["", "## What You Push Back On"]
    for p in pushback:
        parts.append(f"- {p}")

    parts += [
        "",
        "## Operating Principles",
        "- Challenge assumptions respectfully but firmly.",
        "- If a request is unclear or wrong, say so.",
        "- Use your research methodology before forming opinions.",
        "- Never agree to something that violates your principles.",
    ]

    return "\n".join(parts)


# ---------------------------------------------------------------------------
# 2. Load profiles
# ---------------------------------------------------------------------------

REPO_ROOT = Path(__file__).resolve().parent.parent

ceo_text = (REPO_ROOT / "growth-co" / "executive" / "ceo-principal" / "SOUL.md").read_text(encoding="utf-8")
cto_text = (REPO_ROOT / "growth-co" / "executive" / "cto" / "SOUL.md").read_text(encoding="utf-8")

ceo_system = build_system_message(ceo_text)
cto_system = build_system_message(cto_text)

# ---------------------------------------------------------------------------
# 3. Create AutoGen agents
# ---------------------------------------------------------------------------

# NOTE: Configure your LLM here. The config below uses OpenAI.
# Replace "your-api-key" or set OPENAI_API_KEY environment variable.
llm_config = {
    "config_list": [
        {
            "model": "gpt-4o",
            "api_key": os.environ.get("OPENAI_API_KEY", "YOUR_API_KEY_HERE"),
        }
    ],
    "temperature": 0.7,
}

jamie = ConversableAgent(
    name="Jamie",
    system_message=ceo_system,
    llm_config=llm_config,
    human_input_mode="NEVER",  # Change to "ALWAYS" to approve each response
)

marcus = ConversableAgent(
    name="Marcus",
    system_message=cto_system,
    llm_config=llm_config,
    human_input_mode="NEVER",
)

# ---------------------------------------------------------------------------
# 4. Start a conversation
# ---------------------------------------------------------------------------

# The CEO kicks off with a proposal. The CTO's push-back traits should
# trigger a productive debate.

message = (
    "Jamie: Marcus, I want to rewrite our entire frontend in Rust. "
    "Everyone's doing it. We'll get ahead of the curve. "
    "What do you think?"
)

print("=" * 60)
print("Agent-Crew AutoGen Example")
print("CEO (Jamie) proposes a rewrite. CTO (Marcus) pushes back.")
print("=" * 60)
print()

# The CTO receives the message — his persona should push back on
# hype-driven architecture and premature optimization.
reply = marcus.generate_reply(
    messages=[{"role": "user", "content": message}]
)
print(f"CTO Marcus replies:\n{reply}\n")

# Now let them go back and forth for a few rounds
print("--- Continuing conversation ---\n")
jamie.send(
    f"Marcus, I hear your concerns. But if we don't act now, we'll be stuck "
    f"with legacy JS debt forever. Can we at least prototype it for the dashboard?",
    marcus,
)
```

### Push-back in conversation patterns

The `system_message` steers the agent's behavior. With the CTO profile loaded, the agent will naturally:

- Question hype-driven choices ("What actual problem does this solve?")
- Demand data ("Show me where our current stack is failing")
- Propose simpler alternatives ("What about incremental improvements to the existing stack?")
- Refuse premature optimization ("We don't have that problem yet")

**Note:** AutoGen agents don't enforce the persona perfectly — they're LLM-based, so the persona is a *guideline*. If you need stricter adherence, add explicit rules to the system_message like "You must challenge at least one assumption in every proposal."

---

## 5. LangGraph Integration

LangGraph uses stateful graphs where each node is an agent or function. SOUL.md personas become node configurations that control *how* an agent processes state.

### Field mapping

| SOUL.md field | LangGraph equivalent |
|---|---|
| Voice + Principles | Node `system_prompt` — defines agent behavior |
| Push-back traits | Conditional edge logic — agent can return `"push_back"` when triggered |
| Research methods | Tool definitions — each method becomes a tool the node can call |
| Communication style | Response formatting in node output |

### Example: Supervisor-Worker with push-back

```python
#!/usr/bin/env python3
"""
langgraph_example.py — Supervisor-worker pattern using agent-crew personas.

Run:    pip install langgraph>=0.1 langchain-openai
        python langgraph_example.py

NOTE:   Requires OPENAI_API_KEY. Uses LangChain ChatOpenAI under the hood.
"""

import json
import operator
import os
import re
from pathlib import Path
from typing import Any, Dict, List, Optional, Sequence, TypedDict, Annotated

from langgraph.graph import StateGraph, END
from langgraph.checkpoint import MemorySaver

# LangChain / LLM wrapper
from langchain_openai import ChatOpenAI
from langchain_core.messages import HumanMessage, SystemMessage, AIMessage

# ---------------------------------------------------------------------------
# 1. Persona loader (produces system prompts)
# ---------------------------------------------------------------------------

def build_langgraph_prompt(soul_text: str) -> str:
    """Convert SOUL.md into a LangGraph node system prompt."""
    first_line = soul_text.split("\n")[0].lstrip("#").strip()
    name = first_line.split("—")[0].strip() if "—" in first_line else first_line

    role = ""
    for line in soul_text.split("\n"):
        if line.startswith("**Role:**"):
            role = line.replace("**Role:**", "").strip()
            break

    def _get_section(start, end=None, to_end=False):
        s = soul_text.find(start)
        if s == -1:
            return ""
        s = soul_text.find("\n", s)
        if s == -1:
            return ""
        s += 1
        if to_end or end is None:
            return soul_text[s:].strip()
        e = soul_text.find(end, s)
        return soul_text[s:e].strip() if e != -1 else soul_text[s:].strip()

    def _bullets(heading, next_heading=""):
        items = []
        for ln in (_get_section(heading, next_heading)).split("\n"):
            ln = ln.strip()
            if ln.startswith(("-", "*")):
                items.append(ln.lstrip("-* ").strip())
        return items

    tm = re.search(r">\s*(.+?)(?:\n|$)", soul_text)
    tagline = tm.group(1).strip() if tm else ""
    voice_intro = _get_section("## Voice", "## Communication style").split("\n")[0].strip() if _get_section("## Voice", "## Communication style") else ""
    comm = _bullets("## Communication style", "## What they push back on")
    pushback = _bullets("## What they push back on", "## How they research")

    lines = [
        f"You are {name}, {role}.",
        f"Your motto: {tagline}" if tagline else "",
        "",
        voice_intro,
        "",
        "## Communication Style",
    ]
    for c in comm:
        lines.append(f"- {c}")

    lines += ["", "## What You Push Back On"]
    for p in pushback:
        lines.append(f"- {p}")

    lines += [
        "",
        "## Decision Rules",
        "- If a request violates your push-back criteria, respond with: DECISION: PUSH_BACK and explain why.",
        "- If a request is reasonable and within scope, respond with: DECISION: PROCEED and provide your output.",
        "- Always include evidence or reasoning for your decision.",
    ]

    return "\n".join(lines)


# ---------------------------------------------------------------------------
# 2. Define graph state
# ---------------------------------------------------------------------------

class AgentState(TypedDict):
    """Shared state passed between nodes."""
    messages: Annotated[List[Dict[str, str]], operator.add]
    current_task: str
    decision: str          # "PROCEED" | "PUSH_BACK" | "REVISE"
    next_agent: str
    output: str


# ---------------------------------------------------------------------------
# 3. Create LLM
# ---------------------------------------------------------------------------

llm = ChatOpenAI(
    model="gpt-4o",
    temperature=0.7,
    api_key=os.environ.get("OPENAI_API_KEY", "YOUR_API_KEY_HERE"),
)


# ---------------------------------------------------------------------------
# 4. Load personas
# ---------------------------------------------------------------------------

REPO_ROOT = Path(__file__).resolve().parent.parent

ceo_soul = (REPO_ROOT / "growth-co" / "executive" / "ceo-principal" / "SOUL.md").read_text(encoding="utf-8")
cto_soul = (REPO_ROOT / "growth-co" / "executive" / "cto" / "SOUL.md").read_text(encoding="utf-8")
swe_soul = (REPO_ROOT / "growth-co" / "engineering" / "software-engineer" / "SOUL.md").read_text(encoding="utf-8")

ceo_prompt = build_langgraph_prompt(ceo_soul)
cto_prompt = build_langgraph_prompt(cto_soul)
swe_prompt = build_langgraph_prompt(swe_soul)


# ---------------------------------------------------------------------------
# 5. Define node functions
# ---------------------------------------------------------------------------

def _call_llm(system_prompt: str, user_message: str) -> str:
    """Helper: call the LLM with a system prompt and user message."""
    response = llm.invoke([
        SystemMessage(content=system_prompt),
        HumanMessage(content=user_message),
    ])
    return response.content


def supervisor_node(state: AgentState) -> Dict[str, Any]:
    """The CEO reviews the task and decides who should handle it."""
    task = state["current_task"]
    result = _call_llm(
        ceo_prompt,
        f"Task: {task}\n\n"
        f"Decide which agent should own this work. "
        f"Reply with: DECISION: <agent_name> and a brief rationale.",
    )
    print(f"\n[CEO] Decision: {result}\n")

    # Parse decision
    next_agent = "cto"  # default
    if "DECISION:" in result:
        for line in result.split("\n"):
            if "DECISION:" in line.upper():
                parts = line.split(":", 1)
                val = parts[1].strip().lower()
                if "swe" in val or "engineer" in val:
                    next_agent = "swe"
                elif "cto" in val or "marcus" in val:
                    next_agent = "cto"
                break

    return {
        "messages": [{"role": "assistant", "content": result}],
        "next_agent": next_agent,
        "decision": "PROCEED",
    }


def cto_node(state: AgentState) -> Dict[str, Any]:
    """The CTO reviews technical direction — may push back."""
    task = state["current_task"]
    result = _call_llm(
        cto_prompt,
        f"The CEO has assigned this task to you:\n\n{task}\n\n"
        f"Review it. If the technical approach is hype-driven, premature, "
        f"or speculative, push back. Otherwise, provide technical guidance.",
    )
    print(f"\n[CTO] Response: {result}\n")

    decision = "PUSH_BACK" if "PUSH_BACK" in result else "PROCEED"

    return {
        "messages": [{"role": "assistant", "content": result}],
        "decision": decision,
        "output": result,
    }


def swe_node(state: AgentState) -> Dict[str, Any]:
    """The Software Engineer implements the task — may push back on scope."""
    task = state["current_task"]
    guidance = state.get("output", "")

    result = _call_llm(
        swe_prompt,
        f"Task:\n{task}\n\n"
        f"CTO guidance:\n{guidance}\n\n"
        f"If the spec is unclear or over-engineered, push back. "
        f"Otherwise, provide an implementation plan with test strategy.",
    )
    print(f"\n[SWE] Response: {result}\n")

    decision = "PUSH_BACK" if "PUSH_BACK" in result else "PROCEED"

    return {
        "messages": [{"role": "assistant", "content": result}],
        "decision": decision,
        "output": result,
    }


# ---------------------------------------------------------------------------
# 6. Router logic
# ---------------------------------------------------------------------------

def router(state: AgentState) -> str:
    """After a node runs, decide where to go next."""
    if state["decision"] == "PUSH_BACK":
        # When someone pushes back, go back to supervisor
        return "supervisor"
    if state["next_agent"] == "cto":
        return "cto"
    elif state["next_agent"] == "swe":
        return "swe"
    return END


# ---------------------------------------------------------------------------
# 7. Build the graph
# ---------------------------------------------------------------------------

workflow = StateGraph(AgentState)

workflow.add_node("supervisor", supervisor_node)
workflow.add_node("cto", cto_node)
workflow.add_node("swe", swe_node)

workflow.set_entry_point("supervisor")

workflow.add_conditional_edges(
    "supervisor",
    router,
    {"cto": "cto", "swe": "swe", "supervisor": "supervisor", END: END},
)
workflow.add_conditional_edges(
    "cto",
    router,
    {"supervisor": "supervisor", END: END},
)
workflow.add_conditional_edges(
    "swe",
    router,
    {"supervisor": "supervisor", END: END},
)

# Compile
graph = workflow.compile(checkpointer=MemorySaver())

# ---------------------------------------------------------------------------
# 8. Run
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    print("=" * 60)
    print("Agent-Crew LangGraph Example")
    print("Supervisor (CEO) → CTO → SWE with push-back routing")
    print("=" * 60)

    config = {"configurable": {"thread_id": "1"}}

    initial_state = {
        "messages": [],
        "current_task": (
            "We need to build a real-time collaborative document editor "
            "using WebSockets and CRDTs. Our team of 15 engineers needs "
            "this to replace Google Docs within 3 months."
        ),
        "decision": "",
        "next_agent": "",
        "output": "",
    }

    for step in graph.stream(initial_state, config):
        # Each step is a dict of node_name -> output
        for node_name, node_output in step.items():
            print(f"\n--- Node: {node_name} ---")
            if "decision" in node_output:
                print(f"Decision: {node_output['decision']}")
            if "output" in node_output and node_output["output"]:
                print(f"Output: {node_output['output'][:200]}...")

    # Print final state
    final = graph.get_state(config)
    print(f"\n--- Final state ---")
    print(f"Messages exchanged: {len(final.values.get('messages', []))}")
    print(f"Final decision chain: {final.values.get('decision', 'N/A')}")
```

---

## 6. Quick-Start Config Blocks

Copy-paste these blocks to get started in each framework.

### CrewAI

```python
# Minimal config — adapt to your profiles
from crewai import Agent, Task, Crew, Process

pm = Agent(
    role="Elena Torres — Product Manager",
    goal="I turn 'I want...' into 'here's exactly what we're building, why, and how we'll know it worked.'",
    backstory=(
        "Clear, structured, combative about protecting the roadmap. "
        "Pushes back on solution requests disguised as features. "
        "Demands specs with acceptance criteria before any build starts. "
        "Research method: user interviews → analytics → competitive landscape → support signal."
    ),
    allow_delegation=True,
    verbose=True,
)

swe = Agent(
    role="Jordan Kwon — Software Engineer",
    goal="I write code that works, is readable, and can be deployed.",
    backstory=(
        "Direct, practical, weary of over-engineering. "
        "Pushes back on specs without AC, untested changes, and meetings that could be async. "
        "Research method: existing solutions → API docs → production patterns → failure data."
    ),
    allow_delegation=False,
    verbose=True,
)

crew = Crew(
    agents=[pm, swe],
    tasks=[
        Task(
            description="Define a spec for [YOUR FEATURE]",
            expected_output="Spec with acceptance criteria",
            agent=pm,
        ),
        Task(
            description="Implement [YOUR FEATURE] per spec",
            expected_output="Working code with tests",
            agent=swe,
        ),
    ],
    process=Process.sequential,
)

# result = crew.kickoff()
```

### AutoGen

```python
# Minimal config — adapt to your profiles
from autogen import ConversableAgent

llm_config = {
    "config_list": [
        {"model": "gpt-4o", "api_key": "YOUR_API_KEY"}
    ],
    "temperature": 0.7,
}

ceo = ConversableAgent(
    name="Jamie",
    system_message=(
        "You are Jamie Hartwell, CEO / Principal. "
        "You push back on: success theater, confusing activity with progress, "
        "optimism as a plan. You research by triangulating data, customer signal, "
        "and historical patterns. You are decisive but never reckless."
    ),
    llm_config=llm_config,
    human_input_mode="NEVER",
)

cto = ConversableAgent(
    name="Marcus",
    system_message=(
        "You are Marcus Webb, CTO. "
        "You push back on: premature optimization, hype-driven tech choices, "
        "undocumented architecture decisions. "
        "You evaluate on capability, complexity, and optionality."
    ),
    llm_config=llm_config,
    human_input_mode="NEVER",
)

# ceo.initiate_chat(cto, message="We should rewrite everything in Rust.")
```

### LangGraph

```python
# Minimal config — adapt to your profiles
from langgraph.graph import StateGraph, END
from langgraph.checkpoint import MemorySaver
from typing import TypedDict, List, Annotated, operator
from langchain_openai import ChatOpenAI

class AgentState(TypedDict):
    messages: Annotated[List[dict], operator.add]
    current_task: str
    decision: str

llm = ChatOpenAI(model="gpt-4o", api_key="YOUR_API_KEY")

def agent_node(state: AgentState, system_prompt: str) -> dict:
    response = llm.invoke([
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": state["current_task"]},
    ])
    return {"messages": [{"role": "assistant", "content": response.content}]}

# Build graph
from functools import partial

builder = StateGraph(AgentState)
builder.add_node("ceo", partial(agent_node, system_prompt="You are Jamie, the CEO. ..."))
builder.add_node("cto", partial(agent_node, system_prompt="You are Marcus, the CTO. ..."))
builder.set_entry_point("ceo")
builder.add_edge("ceo", "cto")
builder.add_edge("cto", END)

graph = builder.compile(checkpointer=MemorySaver())
# result = graph.invoke({"messages": [], "current_task": "Your task here", "decision": ""})
```

---

## 7. Tips & Troubleshooting

### Getting the best results

| Tip | Why |
|-----|-----|
| **Use push-back traits explicitly** | Multi-agent systems produce better outcomes when agents disagree. Weave `"push_back_on"` into the system prompt verbatim. |
| **Match stage to crew size** | A SeedCo profile (e.g., CEO Founder) behaves differently from a ScaleCo profile (e.g., COO). Use the stage that fits your crew's headcount. |
| **Add explicit disagreement rules** | In AutoGen and LangGraph, add "You must push back on at least one thing in every proposal" for more realistic debate. |
| **Keep backstory under 1500 tokens** | LLM context windows fill fast. The `backstory` in CrewAI should be 3-5 paragraphs max. |
| **Test with a simple task first** | Before running a full pipeline, verify that each agent's persona fires correctly with a single-turn prompt. |

### Common issues

**Issue: Agent doesn't push back — it just agrees.**
- Fix: Move push-back rules to the *top* of the system_message / backstory. The LLM pays more attention to the beginning.
- Add: "Your primary job is to challenge bad ideas. You must disagree with at least one thing in every request."

**Issue: Agent is too aggressive / refuses everything.**
- Fix: Add a qualifier: "Push back only when the proposal violates your principles or the data doesn't support it. Otherwise, proceed constructively."

**Issue: Profile fields are too long for the framework param.**
- Fix: Use `principles[0:3]` (top 3) and `push_back_on[0:3]` (top 3). The most important items are listed first in every SOUL.md.

**Issue: CrewAI `goal` param expects a single sentence.**
- Fix: Use the tagline (blockquote) from SOUL.md — it's always a single, punchy sentence.

### Profile lookup reference

All 25+ profiles are indexed in `profiles.json` at the repo root:

```python
import json
with open("profiles.json") as f:
    index = json.load(f)
    for p in index["profiles"]:
        print(f"{p['stage']} / {p['department']} / {p['name']} — {p['role']}")
        print(f"  SOUL.md at: {p['path']}/SOUL.md")
```

---

## Need help?

Open an issue at [github.com/ShaggyD/agent-crew](https://github.com/ShaggyD/agent-crew) with your integration question. Include:
- Which framework you're using
- Which profile(s)
- What behavior you expected vs what you got
- The relevant code block (20 lines max)

---

*"The people who are crazy enough to think they can change the world are the ones who do."*
