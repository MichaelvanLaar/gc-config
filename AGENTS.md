# AI Coding Assistant Skills

Repository of reusable skills and prompt configurations for Claude Code and GitHub Copilot.

## Setup

No build steps required. This is a content repository of Markdown and YAML files.

## Structure

- `.claude/skills/` — Claude Code skills (SKILL.md per skill)
- `.claude/commands/opsx/` — Claude Code slash commands
- `.github/skills/` — GitHub Copilot skills (SKILL.md per skill)
- `.github/prompts/` — GitHub Copilot prompt files
- `openspec/` — Change workflow management (proposals, specs, tasks)

## Conventions

- Each skill lives in its own subdirectory with a `SKILL.md` file
- GitHub Copilot prompts mirror Claude Code commands in purpose and content
- When editing a skill, update the corresponding file in both `.claude/skills/` and `.github/skills/`
- `openspec/config.yaml` holds the project context shown to AI when creating artifacts — keep it current

## Safety

- Do not commit secrets or credentials
- Do not use `--force` git flags — fix the underlying issue instead
