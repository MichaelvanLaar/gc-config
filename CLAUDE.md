# AI Coding Assistant Skills

Reusable skills and prompt configurations for AI coding assistants — Claude Code (`.claude/`) and GitHub Copilot (`.github/`), managed with OpenSpec.

@AGENTS.md

## Key Config Files

| File | Purpose |
|------|---------|
| `.claude/settings.json` | Claude Code permissions, hooks, env vars                                                                |
| `.claude/skills/cc-init/SKILL.md` | Bootstraps a new Claude Code project configuration (CLAUDE.md, settings.json, hooks, git setup)         |
| `.claude/skills/cc-optimize/SKILL.md` | Audits and improves an existing Claude Code configuration for quality and cost                          |
| `.claude/skills/cc-update/SKILL.md` | Downloads the latest cc-init, cc-optimize, and cc-update skills from the source repository              |
| `.claude/skills/copilot-init/SKILL.md` | TODO: add description   |
| `.claude/skills/copilot-optimize/SKILL.md` | TODO: add description   |
| `.claude/skills/copilot-update/SKILL.md` | TODO: add description   |
| `.claude/skills/openspec-apply-change/SKILL.md` | Implements pending tasks from an OpenSpec change                                                        |
| `.claude/skills/openspec-archive-change/SKILL.md` | Archives a completed OpenSpec change and optionally syncs delta specs                                   |
| `.claude/skills/openspec-explore/SKILL.md` | Explores and maps the codebase to inform OpenSpec proposals                                             |
| `.claude/skills/openspec-propose/SKILL.md` | Creates a new OpenSpec change with proposal, specs, and task artifacts                                  |
| `.gitignore` | Git ignore patterns                                                                                     |
| `AGENTS.md` | Shared instructions for all AI coding agents                                                            |

## Commands

```bash
openspec list                                          # list active changes
openspec status --change "<name>" --json               # check change status
openspec instructions apply --change "<name>" --json   # get implementation instructions
```

## Learnings

When the user corrects a mistake or points out a recurring issue, append a one-line
summary to `.claude/learnings.md`. Don't modify CLAUDE.md directly.

## Compact Instructions

When compacting, preserve: list of modified files, current test status, open TODOs, and key decisions made.
