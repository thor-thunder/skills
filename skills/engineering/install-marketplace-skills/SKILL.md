---
name: install-marketplace-skills
description: Install skills from the marketplace with dependency resolution and full setup guidance.
disable-model-invocation: true
---

# Install Marketplace Skills

Install any skill collection from the marketplace (like `mattpocock/skills`) with proper dependency management and configuration.

## Overview

This skill automates the process of:
1. **Installing skills** from a marketplace repo with all dependencies
2. **Resolving dependencies** between skills (some skills require others to function)
3. **Running setup flows** like `/setup-matt-pocock-skills` that configure the repo
4. **Guiding you through configuration** step-by-step

## Process

### 1. Discover Available Skills

List all skills available in the target marketplace repo. Show:
- User-invoked vs model-invoked skills
- Which skills are dependencies for others
- Brief descriptions of what each skill does

### 2. Select Skills to Install

Present the available skills and ask which ones you want to install. Default: install all non-deprecated skills (skip `personal/`, `in-progress/`, `deprecated/` buckets).

Prompt:
> Which skills would you like to install? (Select multiple or pick all)
> - Each skill shows its dependencies (if any)
> - User-invoked skills are marked as "manual-invoke"
> - Model-invoked skills are marked as "auto-available"

### 3. Install via Marketplace

Run the marketplace installation command:
```bash
npx skills@latest add <owner>/<repo>
```

This installs:
- All selected skills to your agent
- Handles the interactive skill picker if running in terminal
- Shows progress of each skill being installed

### 4. Resolve Dependencies

After installation, check if any setup skills need to run. Common examples:
- `/setup-matt-pocock-skills` — configures the repo for Matt Pocock's skills
- Any other `setup-*` skills in the installed collection

Ask if you want to run the setup skill now.

### 5. Verify Installation

Check that all skills installed correctly:
```bash
# List installed skills
npm list -g skills
# Or check in your agent's skill list
```

Show a summary of what's now available.

## Example Flow

**User:** "I want to install Matt Pocock's skills"

**Skill:**
1. Shows all available skills in `mattpocock/skills` (sorted by bucket)
2. Asks which ones to install (defaults to all published ones)
3. Runs: `npx skills@latest add mattpocock/skills`
4. Detects that `/setup-matt-pocock-skills` needs to run
5. Offers to run setup immediately
6. Confirms all skills are available

## Why This Skill Exists

The marketplace installation process has several steps:
- Know which repo to install from
- Understand dependencies between skills
- Run the `npx skills` command correctly
- Execute any setup flows afterward

This skill bundles all of that into one guided flow, so you don't have to remember the steps or deal with configuration manually.

## Extending This Skill

To support a new marketplace repo (e.g., `your-org/skills`):
1. Ensure the target repo follows the skills directory structure
2. Ensure it has a `plugin.json` that lists all skills
3. Add a `README.md` that documents what each skill does
4. Run this skill with: `/install-marketplace-skills` and select your repo

The skill will auto-discover the repo structure and present available skills.
