# AGENTS.md

How Claude Code agents discover, trigger, and chain skills.

## Agent Skill Resolution

When Claude Code starts in a project with `.claude/settings.json`, it:

1. **Reads enabled plugins** from `enabledPlugins` (e.g., `"codebase-design@skills": true`)
2. **Discovers skill files** in each enabled plugin's folder (e.g., `skills/engineering/codebase-design/SKILL.md`)
3. **Parses frontmatter** to extract `name`, `description`, and `disable-model-invocation` flag
4. **Makes skills available** via slash commands (`/codebase-design`) and auto-invocation (if model-invocable)

### The Interface

**For the user** (human typing):
- Sees a slash command in IDE autocomplete or types it directly
- Provides input if the skill requires it
- Observes the skill's behavior unfold

**For the agent** (Claude evaluating task):
- Reads the skill's `description` field in frontmatter
- Model-invocable skills have rich trigger phrases: "Use when the user wants…, mentions…, asks for…"
- Agent reaches for the skill when task matches the description

## Skill Invocation: User-Controlled vs Agent-Driven

### User-Invoked Skills (`disable-model-invocation: true`)

**Seam**: Only active when human types the slash command.

```
human types /grill-with-docs
         ↓
SKILL.md executes (reads CLAUDE.md, CONTEXT.md, etc.)
         ↓
within skill, triggers /grilling (model-invoked)
```

**Why**: Orchestration decisions (which workflow to run, which issue to tackle, which design to explore) belong to the human. Skills like `/ask-matt`, `/to-prd`, `/triage` are seams where the human steers.

**Constraint**: Cannot invoke other user-invoked skills (would create loops). Only invokes model-invoked skills.

### Model-Invoked Skills (no flag)

**Seam**: Active when human types OR when agent auto-reaches for it.

```
scenario 1: human types /tdd
         ↓
SKILL.md executes (runs test-first loop)

scenario 2: agent evaluates "write a failing test first"
         ↓
description matches, agent auto-invokes /tdd
         ↓
SKILL.md executes (same behavior as scenario 1)
```

**Why**: Tactics (how to build a feature, how to debug, how to model a domain) can be agent-chosen. When Claude recognizes the pattern, it reaches for the right skill without waiting for human direction.

**Constraint**: None. Can be invoked by users, other skills, or auto-triggered by agents.

## Skill Chaining: Deep Interfaces

A user-invoked skill's **interface** is simple: just the slash command and its input. Its **implementation** is deep — it may trigger multiple model-invoked skills.

### Example: `/grill-with-docs`

**Interface** (small):
```
/grill-with-docs
  ← human provides rough idea
```

**Implementation** (deep):
```
1. reads CLAUDE.md, CONTEXT.md, existing ADRs
2. triggers /grilling (model-invoked) — human interview loop
3. detects complexity → triggers /domain-modeling (model-invoked)
4. updates CONTEXT.md and creates/updates ADRs
5. returns sharpened idea + updated docs
```

**Leverage**: User types one command; gets interview + domain refinement + documentation updates. All the complexity is hidden inside.

**Locality**: If the grilling interview logic needs to change, edit `/grilling` in one place. If domain-modeling behavior changes, edit `/domain-modeling` in one place. Changes concentrate in dedicated skills.

### Example: `/improve-codebase-architecture`

**Interface** (small):
```
/improve-codebase-architecture
  ← analyzes codebase automatically
```

**Implementation** (deep):
```
1. scans for architectural opportunities (modules, seams, shallow interfaces)
2. generates visual HTML report
3. triggers /grilling on whichever opportunity human picks
4. for each explored option, may trigger /codebase-design (model-invoked)
5. updates docs or suggests refactors
```

**Leverage**: User gets full architecture review + targeted grill session + design exploration. One command; lots of behavior.

## Skill Integration via Claude's Native Commands

Skills integrate with Claude's built-in commands and behaviors:

### `/compact`
Built-in command to summarize early conversation turns. Skills don't call this — users invoke it at phase breaks to keep context efficient.

### `/help`
Built-in command to show available commands. Skill descriptions appear here. A skill's `description` in frontmatter is its help text.

### Model Auto-Invocation
When Claude recognizes a task pattern, it auto-invokes matching model-invocable skills without waiting for user direction. Example:

```
user: "Write a test first for this feature"
     ↓
Claude reads /tdd skill description: "Use when the user wants test-first development…"
     ↓
description matches, auto-invokes /tdd
     ↓
/tdd skill runs (executes red-green-refactor loop)
```

### Prose Invocation
Skills reference each other by name in text ("Run the `/grilling` skill"), not by file path. When a skill says "trigger `/domain-modeling`", it's using the skill's registered name from frontmatter.

## Skill Configuration & Activation

### `.claude/settings.json`
Controls which skill plugins are enabled in this project:

```json
{
  "enabledPlugins": {
    "codebase-design@skills": true,
    "tdd@skills": true,
    "code-review@claude-official-plugins": true
  }
}
```

Only enabled plugins make their skills available. Skills in disabled plugins cannot be invoked (user or auto).

### Skill Metadata (frontmatter)
Each SKILL.md sets rules for its own invocation:

```yaml
---
name: grill-with-docs
description: Sharpen a plan by interview, update CONTEXT.md and ADRs
disable-model-invocation: true
---
```

- **name**: Appears in slash commands (`/grill-with-docs`)
- **description**: Help text for `/help` and trigger phrases for agent auto-invocation
- **disable-model-invocation**: If `true`, only human typing `/grill-with-docs` triggers it; agent cannot auto-invoke

## Skill Auto-Trigger Conditions

### A Skill Auto-Triggers When:

1. **It is model-invocable** (no `disable-model-invocation: true` flag)
2. **It is enabled** (in `enabledPlugins` or globally available)
3. **Its description matches the task** — agent reads trigger phrases and decides if this skill solves the problem

### Example Trigger Phrases in Descriptions:

```
/tdd: "Use when the user wants test-first development, 
       mentions writing tests first, or needs a red-green-refactor loop."

/domain-modeling: "Use when the user is naming domain terms, 
                   asking about invariants, or building a glossary."

/diagnosing-bugs: "Use when debugging a hard bug, need to reproduce 
                   and minimize, or suspect a performance regression."
```

Agent evaluates: Does the current task match any of these? If yes, auto-invoke the skill.

## Seams and Boundaries

### The Main Seam: User-Invoked → Model-Invoked

User-invoked skills are the **seams** where human control lives. The human decides which workflow to enter. Once inside, model-invoked skills fill in the tactics.

```
Human at seam ↓
        /grill-with-docs
        ↓ (inside, skill may trigger)
        /grilling (model-invoked)
        ↓ (Claude may auto-trigger)
        /domain-modeling (model-invoked)
        ↓ (Claude chooses)
```

**Why separate?**
- **Orchestration** (human) vs **tactic** (agent) stay distinct
- Humans make high-level decisions; agents execute discipline
- Easier to test and reason about workflow

### The Plugin Seam

`enabledPlugins` in `.claude/settings.json` is a **seam** between the project's needs and available skills.

```
project needs /tdd for red-green-refactor feedback loop
         ↓
add "tdd@skills": true to enabledPlugins
         ↓
Claude can now reach for /tdd when building features
```

Change the adapter (add/remove plugins) without changing how skills work internally.

## Skill Discovery for New Users

When a user opens this project:

1. Claude reads `.claude/settings.json` and discovers enabled skills
2. Claude reads each skill's frontmatter (name, description)
3. Skill names appear in IDE autocomplete (slash commands)
4. Skill descriptions appear in `/help` and influence auto-invocation

The **interface** for skill discovery is simple: just the enabled list. The **implementation** is deep: includes reading and parsing, metadata extraction, registration, and trigger pattern matching.

---

**Summary**: Skills are composable modules with small interfaces (slash commands + descriptions) and deep implementations (can trigger other skills, update docs, run loops). User-invoked skills orchestrate; model-invoked skills execute discipline. Configuration via `enabledPlugins` and metadata via frontmatter define the activation seams.
