---
description: Bootstrap a GitHub Copilot Coding Agent configuration (copilot-instructions.md and optional supporting files). Lighter alternative to the full copilot-init skill.
---

Set up a GitHub Copilot Coding Agent configuration for this project.

**Before creating any files:**

1. If `.github/copilot-instructions.md` already exists, stop and suggest running `copilot-optimize` instead.
2. Scan for toolchain clues: `package.json`, `Cargo.toml`, `go.mod`, `requirements.txt`, `Makefile`, `README.md`, and similar.
3. If the project description was not provided and the toolchain cannot be determined, ask what the project produces and what stack is involved.

**Create `.github/copilot-instructions.md`** with this structure (keep under ~8 000 characters):

```markdown
# <Project Name>

<One-line description and stack summary.>

## Commands

- Build: `<command or TODO>`
- Test: `<command or TODO>`
- Lint: `<command or TODO>`

## Architecture

<Key directories and patterns — only non-obvious parts. Omit if too new.>

## Conventions

<Concrete rules that deviate from defaults. Never standard language conventions.>

## Don't

- Don't commit secrets or credentials to git
- Don't use `--force` git flags — fix the underlying issue instead
```

**Optionally also create:**

- `.github/instructions/*.instructions.md` — if the project has distinct subsystems with different conventions; use `applyTo: "<glob>"` frontmatter to scope each file
- `.github/workflows/copilot-setup-steps.yml` — if a package manager or build tool is detected; job name must be `copilot-setup-steps`, runtime under 59 min; always include dependency caching (`actions/setup-node@v4` with `cache: 'npm'`, `actions/setup-python@v5` with `cache: 'pip'`, `actions/setup-go@v5` with `cache: true`, or `actions/cache@v4` for Cargo) — without caching every agent session reinstalls all dependencies from scratch
- `AGENTS.md` — if other AI tool directories (`.claude/`, `.gemini/`, `.codex/`) are present

**After creating files**, list what was created, note any TODO placeholders, and mention:

- Fill in TODO commands once they are known
- Run `copilot-optimize` once the project has more content
- Add MCP servers via GitHub repository settings (Settings → Copilot → MCP servers)
- When Copilot makes a mistake and the user corrects it, Copilot logs a one-line summary to `.github/copilot-learnings.md` instead of modifying config files directly. Run `copilot-optimize` periodically to incorporate accumulated learnings into the configuration.

**Limitations vs. Claude Code's cc-init:** No permissions deny/allow, no PostToolUse hooks, no autocompact control, no `@`-import progressive disclosure (so `copilot-learnings.md` is not auto-loaded — `copilot-optimize` is required to apply it), no MCP automation via files.

---

Did this output meet your expectations? If not, describe what was off and Copilot will log the correction to `.github/copilot-learnings.md`.

> **Note:** Corrections are not auto-loaded on every session. Run `copilot-optimize` periodically to review and incorporate them.
