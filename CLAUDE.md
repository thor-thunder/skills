# CLAUDE.md

## Approach

Code and execution is done by feeling and heart. You don't understand yet what to do but you're so powerful go right at it!!! No; OK? But; OK, go! If path fails retry nothing and skip to another path, ctx7 + [Code language] will aid in most, if not all. No half solutions go ALL IN… with auto-triggers on your [tools] and necessary [skills]. Use a todo.md file as a checkbox list '[ ]' update finished tasks by removal of them, most of time add more boxes/tasks. And memory.md for what changes you done and future plans queried for later.

---

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**For agent behavior details** — how skills are discovered, triggered, and chained together — see **AGENTS.md**.

## What This Repository Is

A curated collection of reusable **skills** (slash commands and autonomous behaviors) for Claude Code. These are tools that fix common failure modes in AI-assisted software development: misalignment, verbose output, broken code, and architectural debt. All skills are designed around **fundamental software engineering principles**, not shortcuts.

## Core Principles

**Alignment over speed**: Skills like `/grill-with-docs` and `/grill-me` invest time upfront in sharp requirements, preventing rework downstream.

**Domain language as leverage**: A shared vocabulary (documented in `CONTEXT.md` and `docs/adr/`) reduces token waste and makes the codebase navigable for agents. Agents operating in projects with good domain models produce better code.

**Depth, not breadth**: Skills target specific workflows (grilling, triaging, prototyping, debugging, architecture). Each skill is small and composable. A user doesn't learn a platform — they layer skills onto their natural workflow.

## Directory Structure & Invocation Model

Skills live in bucket folders under `skills/`:
- `engineering/` — code work (triage, design, testing, architecture)
- `productivity/` — non-code workflows (grilling, handoffs, teaching)
- `misc/` — rarely-used utilities
- `in-progress/`, `personal/`, `deprecated/` — not promoted

**Critical**: Only `engineering/`, `productivity/`, and `misc/` appear in the top-level `README.md` and `.claude-plugin/plugin.json`. The other buckets are excluded.

### User-invoked vs Model-invoked

This is the single most important axis in the codebase. Read `docs/invocation.md` for the full definition.

**User-invoked** (`disable-model-invocation: true` in frontmatter):
- Reachable **only when the human types the slash command**
- Examples: `/grill-with-docs`, `/to-prd`, `/triage`, `/ask-matt`
- Orchestrate workflows and gather human intent
- Cannot invoke other user-invoked skills (to avoid loops)

**Model-invoked** (no frontmatter flag):
- Reachable by **model or user**
- Examples: `/tdd`, `/diagnosing-bugs`, `/domain-modeling`, `/codebase-design`
- Encapsulate reusable discipline (red-green-refactor loop, diagnostic procedure, vocabulary)
- Can be auto-invoked by Claude when the task fits
- May be invoked by user-invoked skills

**Why it matters**: User-invoked skills define the "seams" where humans steer. Model-invoked skills define the tools the model reaches for. This split keeps orchestration (human-controlled) separate from reusable tactics (model-available).

### Skill Chaining and Auto-Triggering: Depth Through Composition

The power of this architecture lies in **depth**. A user-invoked skill has a small interface — just the slash command — but a deep implementation that may trigger multiple model-invoked skills internally.

**Example: `/grill-with-docs`**

```
Interface (what user sees):
  /grill-with-docs

Implementation (what happens inside):
  1. reads CLAUDE.md, CONTEXT.md, docs/adr/
  2. triggers /grilling (model-invoked) — human interview loop
  3. detects domain complexity → triggers /domain-modeling (model-invoked)
  4. updates CONTEXT.md with new terms, creates/updates ADRs
  5. returns sharpened requirements + updated docs

Leverage: User types one command; gets interview + domain modeling + documentation
Locality: Grilling behavior lives in /grilling; domain modeling in /domain-modeling
```

Each skill is a **deep module**: lots of behavior behind a simple interface, placed at a clean seam (the slash command), testable through that interface (can verify the interview produced sharper requirements and docs were updated).

**Auto-triggering happens at two levels:**

1. **Prose invocation**: User-invoked skills reference model-invoked skills by name in their text: "Run the `/domain-modeling` skill." Claude recognizes this and executes the skill when appropriate.
2. **Pattern matching**: Model-invoked skills include rich trigger phrases in their description. When Claude evaluates a task, it reads these phrases and auto-invokes the skill if the pattern matches:
   - `/tdd` description: "Use when the user wants test-first development, mentions writing tests first…"
   - Agent asks: "Does the current task match?" → If yes, auto-invoke `/tdd`

**Why this design?**

- **Seam discipline**: Humans control which major workflow to enter (`/grill-with-docs`, `/to-prd`, `/triage`). Once inside, Claude executes tactics (grilling, domain modeling, documentation).
- **Composability**: A skill doesn't assume exclusive control. `/grill-with-docs` works alongside `/improve-codebase-architecture` without colliding because each owns its own interface.
- **Testability**: Can test a skill by verifying its interface worked: Did grilling produce sharper requirements? Did CONTEXT.md get updated? Did ADRs document new decisions?

**See AGENTS.md** for detailed mechanics of skill resolution, seams, and plugin configuration.

## Domain Language

Read `CONTEXT.md` for the full glossary, but key terms:

- **Issue tracker** — the tool hosting issues (GitHub Issues, Linear, local markdown). Not "backlog backend" or "issue host."
- **Issue** — a unit of work: bug, task, PRD, or vertical slice. Not "ticket."
- **Triage role** — a state-machine label (e.g., `needs-triage`, `ready-for-afk`) applied during triage. Maps to real labels via `docs/agents/triage-labels.md`.
- **Module** — any container with an interface and implementation (function, class, package). Scale-agnostic.
- **Depth** — leverage at the interface: behaviour-per-unit-of-complexity. Deep = small interface, lots of implementation. Shallow = large interface, little implementation (avoid).
- **Seam** — where a module's interface lives; a place to alter behaviour without editing there.
- **Adapter** — a concrete thing satisfying an interface at a seam (role, not substance).

## Skills as Deep Modules

Each skill is designed as a **deep module** using codebase design principles:

**Interface** (small):
- Slash command name (`/grill-with-docs`)
- Frontmatter metadata (name, description, invocation rules)
- Input: rough idea, code snippet, or design question
- Output: sharpened requirements, updated docs, or fixed code

**Implementation** (deep):
- SKILL.md: the main workflow
- Supporting docs: `.DEEPENING.md`, `.LOGIC.md`, `.UI.md` (details hidden inside)
- Scripts: shell utilities, test templates (organized in `scripts/` subfolder)
- Auto-triggers to other skills when appropriate (via prose invocation or pattern matching)

**Leverage** (what users get):
- One command invocation delivers multiple coordinated behaviors
- `/grill-with-docs` user doesn't need to know about `/grilling` or `/domain-modeling` — they're triggered automatically
- Reduces user's cognitive load (interface is simple) while providing rich behavior (implementation is deep)

**Locality** (where maintenance concentrates):
- Grilling behavior changes? Edit `/grilling` skill once
- Domain modeling logic needs updating? Edit `/domain-modeling` once
- Change ripples nowhere else because the seam is clean (other skills call it by name, not internals)
- Tests can verify each skill's interface independently

When designing or updating a skill, ask:
- Can I reduce the interface (fewer required inputs, simpler slash command)?
- Can I hide more complexity inside (what can move from user responsibility to skill implementation)?
- Can I trigger other skills automatically instead of asking the user to chain them?
- Is each behavior concentrated in one place (high locality)?

## Skill Structure

Every skill is a folder with:
- **SKILL.md** — the main file. Frontmatter includes `name`, `description`, and optionally `disable-model-invocation: true`.
  - Description is **human-facing** for user-invoked skills (one line; no trigger list).
  - Description is **model-facing** for model-invoked skills (rich trigger phrases: "Use when the user wants…, mentions…, asks for…").
- **Supporting docs** — additional `.md` files as needed (e.g., `.DEEPENING.md`, `.DESIGN-IT-TWICE.md` for `/codebase-design`, or `.LOGIC.md`, `.UI.md` for `/prototype`).
- **Scripts** — shell scripts in a `scripts/` subfolder (e.g., `diagnosing-bugs/scripts/hitl-loop.template.sh`).

When creating a new skill:
1. Decide: user-invoked or model-invoked? (Read `docs/invocation.md` for the test.)
2. Place it in the right bucket (`engineering/`, `productivity/`, or `misc`).
3. Write SKILL.md with sharp, actionable guidance. Don't explain "what" — explain "when to use" and "how" and "why this approach."
4. Update the bucket's `README.md` with a one-line entry linking to SKILL.md.
5. If it's in `engineering/`, `productivity/`, or `misc/`: add it to the top-level `README.md` and `.claude-plugin/plugin.json`.
6. Cross-skill references: use **prose invocation** ("Run the `/grilling` skill"), not deep file links. Shared docs live inside the skill that owns them.

## Key Workflows

### The Main Flow (ask-matt.SKILL.md)

**User types `/grill-with-docs`:**

```
┌──────────────────────────────────────────────┐
│ /grill-with-docs (user-invoked interface)    │
├──────────────────────────────────────────────┤
│ Inside: reads docs, triggers /grilling       │
│ ↓ (auto)                                      │
│ /grilling (model-invoked) — human interview  │
│ ↓ (auto-detected)                            │
│ /domain-modeling (model-invoked) — builds    │
│   glossary, stress-tests terms               │
│ ↓ (skill writes)                             │
│ Updates: CONTEXT.md, docs/adr/                │
│ Returns: sharpened idea + docs               │
└──────────────────────────────────────────────┘
```

**Main flow options:**

1. **`/grill-with-docs`** — sharpen idea by interview, update docs. Auto-triggers `/grilling` and `/domain-modeling`. **Start here for codebase work.**
2. **Branch**: Prototype to answer design questions via `/prototype` + `/handoff`.
3. **Branch**: Multi-session build?
   - `/to-prd` (user-invoked) → synthesizes PRD
   - `/to-issues` (user-invoked) → breaks PRD into vertical slices
   - Spawn fresh session per issue with `/implement`
   - **Context hygiene**: keep grill + PRD + issues in one window; each `/implement` session starts fresh, receiving only the issue it needs

### On-Ramps
- **`/triage`** — move raw bug reports and feature requests through triage roles. Produces agent-ready issues.
- **`/improve-codebase-architecture`** — keep codebase healthy for agent operations. Run periodically; each opportunity generates a task for the main flow.

### Crossing Sessions
- **`/handoff`** — compact conversation to markdown. Open a **new session** referencing that file (don't continue in place).
- **`/compact`** (built-in) — stay in same conversation; summarize early turns. Use at intentional phase breaks, not mid-phase.

## Failure Modes This Repo Addresses

1. **Misalignment** → grilling sessions (`/grill-me`, `/grill-with-docs`)
2. **Verbosity** → shared domain language (`CONTEXT.md`, ADRs)
3. **Broken code** → feedback loops (tests with `/tdd`, debugging with `/diagnosing-bugs`, design discipline with `/codebase-design`)
4. **Ball of mud** → architectural discipline (`/improve-codebase-architecture`, `/codebase-design` vocabulary)

## Contributing & Skill Development

When writing or improving a skill:
- **Consult the discipline, not the tool**. If a skill describes an AI workflow, ground it in real software engineering (TDD, domain modeling, architecture review, debugging procedures).
- **Compress jargon ruthlessly**. Use `CONTEXT.md` + shared vocabulary to cut unnecessary words. Depth of domain language is a feature, not a bug.
- **Make it composable**. A skill works with other skills; don't assume exclusive control.
- **Prioritize human control at seams**. User-invoked skills decide the route; model-invoked skills fill in the tactics.

## Releasing Changes

This repo uses [changesets](https://github.com/changesets/changesets):

```bash
npm run changeset          # Create a changeset
npm run version            # Bump versions and update CHANGELOG
```

Then push to trigger a release via GitHub Actions.

---

**Philosophy**: These skills are "my best effort at condensing software engineering fundamentals into repeatable practices." They're small, adaptable, and designed to be remixed. Hack around with them; make them your own.
