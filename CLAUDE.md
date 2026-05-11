# gc-config

GitHub Copilot CLI plugin providing two skills (`/gc-config-init`, `/gc-config-optimize`) for bootstrapping and auditing best-practice GitHub Copilot Coding Agent configurations. Content-only repo: Markdown + JSON, no build toolchain.

## Key Config Files

| File | Purpose |
|------|---------|
| `AGENTS.md` | Multi-tool agent instructions (Copilot, Codex, Cursor, etc.)      |
| `.claude/settings.json` | Permissions, environment variables                                |
| `.githooks/pre-commit` | Keeps Key Config Files table in sync                              |
| `.github/plugin/marketplace.json` | Copilot CLI marketplace manifest                                  |
| `.gitignore` | Git ignore patterns                                               |
| `plugins/gc-config/plugin.json` | Plugin manifest                                                   |
| `plugins/gc-config/skills/gc-config-init/SKILL.md` | Skill: bootstrap GitHub Copilot config                            |
| `plugins/gc-config/skills/gc-config-optimize/SKILL.md` | Skill: audit GitHub Copilot config                                |
| `scripts/sync-config-table.sh` | Auto-syncs Key Config Files table on commit                       |

## Setup

No build steps. Clone and start editing.

## References

@AGENTS.md

## Conventions

- Skill `SKILL.md` files use YAML frontmatter: `name`, `description`, `allowed-tools`, `argument-hint`
- Both skills end with the learnings/feedback step — keep them aligned
- Internal skill cross-references use `/gc-config-init` and `/gc-config-optimize`
- Plugin manifest fields follow the Copilot CLI plugin reference spec

## Don't

- Don't commit secrets or credentials
- Don't use `--force` git flags — fix the underlying issue instead

## Learnings

When the user corrects a mistake or points out a recurring issue, append a one-line
summary to `.claude/learnings.md`. Don't modify CLAUDE.md directly.

## Compact Instructions

When compacting, preserve: list of modified files, current test status, open TODOs, and key decisions made.
