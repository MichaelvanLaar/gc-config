# gc-config: GitHub Copilot CLI Plugin — Design Spec

**Date:** 2026-05-11
**Status:** Approved

## Summary

Transform the current `github-copilot-config-skills` repository into `gc-config`: a GitHub Copilot CLI plugin that provides two skills for setting up and maintaining best-practice GitHub Copilot Coding Agent configurations. The plugin mirrors the structure of the companion `cc-config` Claude Code plugin but targets the GitHub Copilot CLI plugin system.

## Problem

The current repo distributes GitHub Copilot skills via a bash `install.sh` script — a fragile, manual approach that provides no versioning, no discovery, and no update mechanism. The GitHub Copilot CLI has a first-class plugin system that solves all of these problems. Moving to it means users install once, get auto-updates, and can discover the plugin through marketplaces.

## Goals

- Rename the project to `gc-config`.
- Restructure as a proper GitHub Copilot CLI plugin.
- Provide two Claude-Code-quality skills: `gc-config-init` and `gc-config-optimize`.
- Drop `copilot-update` (plugin system handles updates).
- Update all documentation to reflect the new distribution model.

## Repository Structure

```
gc-config/
├── .github/
│   └── plugin/
│       └── marketplace.json         Copilot CLI marketplace manifest
├── plugins/
│   └── gc-config/
│       ├── plugin.json              Plugin manifest
│       └── skills/
│           ├── gc-config-init/
│           │   └── SKILL.md
│           └── gc-config-optimize/
│               └── SKILL.md
├── docs/
│   └── superpowers/
│       └── specs/
│           └── 2026-05-11-gc-config-plugin-design.md
├── AGENTS.md                        Project instructions for this repo
├── install.sh                       Shim → /plugin install instructions
├── LICENSE
└── README.md
```

**Files removed:**

- `.github/prompts/copilot-init.prompt.md`
- `.github/prompts/copilot-optimize.prompt.md`
- `.github/prompts/copilot-update.prompt.md`
- (The `copilot-update` skill is dropped entirely; the plugin system handles updates.)

## Manifest Files

### `.github/plugin/marketplace.json`

Makes this repository a GitHub Copilot CLI marketplace. Users add it with:

```
/plugin marketplace add MichaelvanLaar/gc-config
```

```json
{
  "name": "gc-config",
  "metadata": {
    "description": "GitHub Copilot CLI skills for setting up and maintaining best-practice GitHub Copilot configurations",
    "version": "1.0.0"
  },
  "owner": { "name": "Michael van Laar" },
  "plugins": [
    {
      "name": "gc-config",
      "source": "./plugins/gc-config",
      "description": "Bootstrap and audit GitHub Copilot configurations",
      "version": "1.0.0",
      "skills": ["./skills/gc-config-init", "./skills/gc-config-optimize"]
    }
  ]
}
```

### `plugins/gc-config/plugin.json`

Plugin manifest for the `gc-config` plugin.

```json
{
  "name": "gc-config",
  "description": "Bootstrap and audit a best-practice GitHub Copilot configuration",
  "version": "1.0.0",
  "author": { "name": "Michael van Laar" },
  "homepage": "https://github.com/MichaelvanLaar/gc-config",
  "repository": "https://github.com/MichaelvanLaar/gc-config",
  "license": "MIT"
}
```

## Skills

### `gc-config-init`

**Source:** Adapted from `.github/prompts/copilot-init.prompt.md`.

**Purpose:** Bootstrap a best-practice GitHub Copilot Coding Agent configuration for a new or unconfigured project.

**Frontmatter:**

```yaml
---
name: gc-config-init
description: Bootstrap a best-practice GitHub Copilot Coding Agent configuration for a new or unconfigured project. Use when a user asks to set up Copilot, create copilot-instructions.md, or configure GitHub Copilot for the first time.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
argument-hint: "[optional: brief project description]"
---
```

**Body:** Content of `copilot-init.prompt.md`, lightly adapted:

- Add numbered step headers to match the style of the existing numbered steps.
- Reference Copilot CLI tool names where relevant.
- Keep the core logic (scan, detect, create, summarize) intact.
- Retain the feedback/learnings step at the end.

**What the skill creates:**

- `.github/copilot-instructions.md` — global agent instructions, kept under ~8,000 characters
- `.github/instructions/*.instructions.md` — path-specific instruction files (optional)
- `.github/workflows/copilot-setup-steps.yml` — pre-install workflow when a build system is detected (optional)
- `AGENTS.md` — only when evidence of a multi-tool AI environment is found

### `gc-config-optimize`

**Source:** Adapted from `.github/prompts/copilot-optimize.prompt.md`.

**Purpose:** Audit and improve an existing GitHub Copilot Coding Agent configuration.

**Frontmatter:**

```yaml
---
name: gc-config-optimize
description: Audit and improve an existing GitHub Copilot Coding Agent configuration. Checks character limits, anti-patterns, missing sections, and dependency caching. Incorporates accumulated learnings from copilot-learnings.md.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
argument-hint: "[optional: focus area, e.g. 'length' or 'caching']"
---
```

**Body:** Content of `copilot-optimize.prompt.md`, lightly adapted (same approach as init).

## `install.sh` (Shim)

Replace the current install script with a shim that redirects to the plugin system:

```bash
#!/usr/bin/env bash
# install.sh — DEPRECATED
#
# Skills are now distributed as a GitHub Copilot CLI plugin.
# Install via the Copilot CLI plugin system instead.

echo ""
echo "This install script is no longer used."
echo ""
echo "Install the gc-config skills via the GitHub Copilot CLI plugin system:"
echo ""
echo "  1. In GitHub Copilot CLI, run:"
echo "       /plugin marketplace add MichaelvanLaar/gc-config"
echo ""
echo "  2. Then install the plugin:"
echo "       /plugin install gc-config@gc-config"
echo ""
echo "  3. To enable auto-updates, go to /plugin → Marketplaces tab"
echo "     and turn on auto-update for MichaelvanLaar/gc-config."
echo ""

exit 1
```

## `AGENTS.md`

Replace the current generic `AGENTS.md` with focused project instructions for AI assistants working in this repository:

```markdown
# gc-config

GitHub Copilot CLI plugin providing two skills for setting up and maintaining
best-practice GitHub Copilot Coding Agent configurations.

## Key Config Files

| File                                                   | Purpose                                    |
| ------------------------------------------------------ | ------------------------------------------ |
| `.github/plugin/marketplace.json`                      | Copilot CLI marketplace manifest           |
| `plugins/gc-config/plugin.json`                        | Plugin manifest                            |
| `plugins/gc-config/skills/gc-config-init/SKILL.md`     | Skill: bootstrap GitHub Copilot config     |
| `plugins/gc-config/skills/gc-config-optimize/SKILL.md` | Skill: audit GitHub Copilot config         |
| `install.sh`                                           | Deprecated shim pointing to plugin install |

## Setup

No build steps required. This is a content repository of Markdown and JSON files.

## Conventions

- Plugin manifest fields follow the Copilot CLI plugin reference spec.
- Skill SKILL.md files use YAML frontmatter (name, description, allowed-tools, argument-hint).
- Keep skill content aligned between gc-config-init and gc-config-optimize (both end with the learnings/feedback step).

## Don't

- Don't commit secrets or credentials.
- Don't use `--force` git flags — fix the underlying issue instead.
```

## `README.md`

Rewritten to document the Copilot CLI plugin. Sections:

1. **Header + intro**: two skills, one-line descriptions
2. **What problem do these skills solve?**: manual config drift, character limits, missing best practices
3. **Installation**: `/plugin marketplace add` + `/plugin install`, keeping skills current (auto-update opt-in), uninstalling
4. **Usage**: invocation examples for each skill, recommended workflow table
5. **What the skills create and check**: config files table + key best practices
6. **What these skills cannot configure**: limitations vs. Claude Code (retained from current README, updated)
7. **Compatibility**
8. **Contributing / License**

## GitHub Repository Rename

The GitHub repository must be renamed from `github-copilot-config-skills` to `gc-config`. This is a one-time manual step:

```bash
gh repo rename gc-config
```

Or via GitHub Settings → General → Repository name. GitHub automatically redirects the old URL.

## Decisions Made

| Decision               | Choice                                  | Reason                            |
| ---------------------- | --------------------------------------- | --------------------------------- |
| `copilot-update` skill | Dropped                                 | Plugin system handles updates     |
| Skill naming           | `gc-config-init`, `gc-config-optimize`  | Mirrors cc-config naming pattern  |
| Old install.sh         | Shim (same pattern as cc-config)        | Clean break; no legacy dual-mode  |
| Old `.github/prompts/` | Removed                                 | Content moves into SKILL.md files |
| Plugin format          | GitHub Copilot CLI (`/.github/plugin/`) | User requirement                  |
