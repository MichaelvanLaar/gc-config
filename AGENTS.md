# gc-config

GitHub Copilot CLI plugin providing two skills for setting up and maintaining
best-practice GitHub Copilot Coding Agent configurations.

## Key Config Files

| File                                                   | Purpose                                |
| ------------------------------------------------------ | -------------------------------------- |
| `.github/plugin/marketplace.json`                      | Copilot CLI marketplace manifest       |
| `plugins/gc-config/plugin.json`                        | Plugin manifest                        |
| `plugins/gc-config/skills/gc-config-init/SKILL.md`     | Skill: bootstrap GitHub Copilot config |
| `plugins/gc-config/skills/gc-config-optimize/SKILL.md` | Skill: audit GitHub Copilot config     |

## Setup

No build steps required. This is a content repository of Markdown and JSON files.

## Conventions

- Plugin manifest fields follow the Copilot CLI plugin reference spec.
- Skill SKILL.md files use YAML frontmatter (name, description, allowed-tools, argument-hint).
- Keep skill content aligned: both skills end with the learnings/feedback step.
- Internal skill cross-references use `/gc-config-init` and `/gc-config-optimize`.

## Don't

- Don't commit secrets or credentials.
- Don't use `--force` git flags — fix the underlying issue instead.
