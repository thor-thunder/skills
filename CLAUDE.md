# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

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

## Domain Language

Read `CONTEXT.md` for the full glossary, but key terms:

- **Issue tracker** — the tool hosting issues (GitHub Issues, Linear, local markdown). Not "backlog backend" or "issue host."
- **Issue** — a unit of work: bug, task, PRD, or vertical slice. Not "ticket."
- **Triage role** — a state-machine label (e.g., `needs-triage`, `ready-for-afk`) applied during triage. Maps to real labels via `docs/agents/triage-labels.md`.
- **Module** — any container with an interface and implementation (function, class, package). Scale-agnostic.
- **Depth** — leverage at the interface: behaviour-per-unit-of-complexity. Deep = small interface, lots of implementation. Shallow = large interface, little implementation (avoid).
- **Seam** — where a module's interface lives; a place to alter behaviour without editing there.
- **Adapter** — a concrete thing satisfying an interface at a seam (role, not substance).

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
1. **`/grill-with-docs`** — sharpen idea by interview. Updates `CONTEXT.md` and ADRs inline. **Start here for codebase work.**
2. **Branch**: Prototype to answer design questions via `/prototype` + `/handoff`.
3. **Branch**: Multi-session build? `/to-prd` → `/to-issues` → spawn fresh session per issue with `/implement`.
   - **Context hygiene**: keep grill + PRD + issues in one window; each `/implement` starts fresh.

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
