# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Release workflow

```bash
npm run changeset   # describe a change; creates a .changeset/*.md file
npm run version     # bump versions from pending changesets
```

No build step. No test suite. The only scripts are changeset management.

## Skill structure

Every skill is a directory with a single `SKILL.md`. Frontmatter is required:

```yaml
---
name: skill-name           # kebab-case; becomes /skill-name when invoked
description: "One line."   # human-facing if user-invoked, trigger-phrase-rich if model-invoked
disable-model-invocation: true   # omit this line for model-invoked skills
---
```

Additional reference files (`AGENT-BRIEF.md`, `OUT-OF-SCOPE.md`, etc.) live alongside `SKILL.md` in the same directory.

## Adding a skill

1. Create `skills/<bucket>/<skill-name>/SKILL.md` with the frontmatter above.
2. Add the path to `.claude-plugin/plugin.json`'s `skills` array (stable buckets only).
3. Add a bullet to the bucket's `README.md` under the correct invocation heading.
4. Add a bullet to the top-level `README.md` under the correct bucket and invocation heading, with the link pointing to `./skills/<bucket>/<skill-name>/SKILL.md`.

## Marketplace installation

Install via: `npx skills@latest add thor-thunder/skills`

The plugin manifest is `.claude-plugin/plugin.json`. Its `name` field is `thor-thunder-skills`.

---

Skills are organized into bucket folders under `skills/`:

- `engineering/` — daily code work
- `productivity/` — daily non-code workflow tools
- `misc/` — kept around but rarely used
- `personal/` — tied to my own setup, not promoted
- `in-progress/` — drafts not yet ready to ship
- `deprecated/` — no longer used

Every skill in `engineering/`, `productivity/`, or `misc/` must have a reference in the top-level `README.md` and an entry in `.claude-plugin/plugin.json`. Skills in `personal/`, `in-progress/`, and `deprecated/` must not appear in either.

Each skill entry in the top-level `README.md` must link the skill name to its `SKILL.md`.

Each bucket folder has a `README.md` that lists every skill in the bucket with a one-line description, with the skill name linked to its `SKILL.md`. Bucket `README.md`s and the top-level `README.md` group entries into **User-invoked** and **Model-invoked**.

Every `SKILL.md` is either user-invoked (`disable-model-invocation: true`, reachable only by the human) or model-invoked (model- or user-reachable). For the full definitions, description conventions, and why a user-invoked skill can invoke model-invoked skills but never another user-invoked one, see [docs/invocation.md](./docs/invocation.md).
