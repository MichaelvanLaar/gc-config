# AI Coding Assistant Skills

Reusable skills and prompt configurations for AI coding assistants — Claude Code (`.claude/`) and GitHub Copilot (`.github/`), managed with OpenSpec.

## Key Config Files

| File | Purpose |
|------|---------|
| `AGENTS.md` | Shared instructions for all AI coding agents   |
| `.claude/settings.json` | Claude Code permissions, hooks, env vars       |
| `.claude/skills/cc-init/SKILL.md` | TODO: add description  |
| `.claude/skills/cc-optimize/SKILL.md` | TODO: add description  |
| `.claude/skills/cc-update/SKILL.md` | TODO: add description  |
| `.claude/skills/openspec-apply-change/SKILL.md` | TODO: add description  |
| `.claude/skills/openspec-archive-change/SKILL.md` | TODO: add description  |
| `.claude/skills/openspec-explore/SKILL.md` | TODO: add description  |
| `.claude/skills/openspec-propose/SKILL.md` | TODO: add description  |
| `.gitignore` | Git ignore patterns                            |

## Commands

```bash
openspec list                                                  # list active changes
openspec status --change "<name>" --json                       # check change status
openspec instructions apply --change "<name>" --json           # get implementation instructions
```

## Structure

- `.claude/skills/` — Claude Code skills (`SKILL.md` per skill)
- `.claude/commands/opsx/` — Claude Code slash commands for OpenSpec operations
- `.github/skills/` — GitHub Copilot skills (mirrors `.claude/skills/`)
- `.github/prompts/` — GitHub Copilot prompt files
- `openspec/` — OpenSpec change workflow (proposals, specs, tasks)

## Conventions

- Each skill lives in its own subdirectory as `SKILL.md`
- GitHub Copilot prompts in `.github/prompts/` mirror Claude commands in `.claude/commands/`
- When editing a skill, update both the Claude Code and GitHub Copilot versions
- `openspec/config.yaml` holds the project context shown to AI — keep it current

## Don't

- Don't commit secrets or credentials to git
- Don't use `--force` flags — fix the underlying issue instead

## Learnings

When the user corrects a mistake or points out a recurring issue, append a one-line
summary to `.claude/learnings.md`. Don't modify CLAUDE.md directly.

## Compact Instructions

When compacting, preserve: list of modified files, current test status, open TODOs, and key decisions made.
