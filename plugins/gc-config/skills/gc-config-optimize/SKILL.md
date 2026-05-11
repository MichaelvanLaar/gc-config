---
name: gc-config-optimize
description: Audit and improve an existing GitHub Copilot Coding Agent configuration. Checks the 8,000-character limit, anti-patterns, missing sections, invalid applyTo globs, missing dependency caching, hooks configuration, and accumulated learnings. Use when a user asks to optimize, audit, or improve their Copilot configuration.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
argument-hint: "[optional: focus area, e.g. 'length', 'caching', or 'hooks']"
---

Audit and improve the GitHub Copilot Coding Agent configuration in this project.

**If `.github/copilot-instructions.md` does not exist**, stop and suggest running `/gc-config-init` instead.

If `$ARGUMENTS` specifies a focus area (e.g. `length`, `caching`, `hooks`), prioritize that area in Step 2 but still complete the full inventory.

## Step 1 — Full inventory

Read and catalog everything:

- `.github/copilot-instructions.md`
- All files in `.github/instructions/` (path-specific instruction files)
- `.github/workflows/copilot-setup-steps.yml`
- `AGENTS.md`
- `.github/copilot-learnings.md`
- `.github/hooks/hooks.json` and any scripts it references
- Toolchain files: `package.json`, `Cargo.toml`, `go.mod`, `pyproject.toml`, etc. (to determine which formatters are present)

**Report this metrics snapshot to the user before auditing:**

| Metric                                    | Value                         |
| ----------------------------------------- | ----------------------------- |
| `copilot-instructions.md` character count | X / ~8,000 limit              |
| Path-specific instruction files           | N files                       |
| `copilot-setup-steps.yml`                 | exists / missing              |
| `AGENTS.md`                               | exists / missing              |
| `.github/hooks/hooks.json`                | exists (list hooks) / missing |
| `copilot-learnings.md` entries            | N entries / not present       |

## Step 2 — Audit against best practices

### 2a: `copilot-instructions.md` audit

- **Over 8,000 characters** → must fix; suggest which sections to extract to path-specific instruction files
- **Anti-patterns** → should fix:
  - Personality instructions ("be concise", "think carefully", "act as a senior engineer")
  - File-by-file codebase descriptions (Copilot can read files itself)
  - Rules that a configured linter or formatter already enforces
- **Missing Commands section** or only vague commands (not actual command strings) → should fix
- **No architecture overview** → nice to have

### 2b: `AGENTS.md` audit

- Does it exist? Should it? (yes when multiple AI tools are used — `.claude/`, `.gemini/`, `.codex/` directories present)
- Is it genuinely tool-agnostic? (no Copilot-specific syntax inside AGENTS.md)
- **Contradictions** between AGENTS.md and `copilot-instructions.md` → must fix

### 2c: Path-specific instruction files audit

For each file in `.github/instructions/*.instructions.md`:

- **Invalid `applyTo` glob** in frontmatter → must fix (invalid globs silently fail — Copilot applies the file to everything)
- **Missing `applyTo`** entirely → must fix

Also check:

- **No path-specific files** for a project with distinct subsystems (frontend/backend, tests, API) → nice to have

### 2d: `copilot-setup-steps.yml` audit

- **Wrong job name** (must be exactly `copilot-setup-steps`) → must fix; Copilot ignores the workflow otherwise
- **Missing dependency caching** → should fix (every agent session reinstalls all dependencies from scratch):
  - `actions/setup-node` without `cache: 'npm'`
  - `actions/setup-python` without `cache: 'pip'`
  - `actions/setup-go` without `cache: true`
  - Cargo builds without `actions/cache@v4` for `~/.cargo` and `target/`
- **Missing `copilot-setup-steps.yml`** when a build system is detected → should fix

### 2e: Hooks audit

Check `.github/hooks/hooks.json` and any referenced scripts:

- **No `hooks.json` but a formatter is present** (Prettier, ruff, rustfmt, gofmt) → should fix; the skills can create `hooks.json` with a `postToolUse` formatter hook and the accompanying script
- **`postToolUse` script exits non-zero on formatter failure** (no `|| true`) → should fix; postToolUse hooks must exit 0
- **Referenced script file does not exist** → should fix
- **Script not executable** (execute bit not set) → should fix
- **No `preToolUse` blocking hook** when `.env` or `secrets/` files exist in the project → nice to have (partial equivalent of `permissions.deny`)

When creating or fixing a missing `postToolUse` formatter hook, use this `hooks.json` structure:

```json
{
  "version": 1,
  "hooks": {
    "postToolUse": [
      {
        "type": "command",
        "bash": ".github/hooks/scripts/format.sh",
        "cwd": ".",
        "timeoutSec": 30
      }
    ]
  }
}
```

And this `format.sh` template (adapt the formatter command to the project ecosystem):

```bash
#!/usr/bin/env bash
set -euo pipefail

INPUT=$(cat)
FILE=$(echo "$INPUT" | jq -r '.toolInput.path // .toolInput.file_path // empty')

if [[ -z "$FILE" || ! -f "$FILE" ]]; then
  exit 0
fi

npx prettier --write "$FILE" || true   # Replace with the project's formatter
```

## Step 3 — Learnings review

If `.github/copilot-learnings.md` exists:

1. Read all entries.
2. Group similar entries — 3 or more corrections pointing to the same gap indicate a real missing rule.
3. For recurring patterns: propose promoting the rule into `copilot-instructions.md` or a path-specific instruction file.
4. For one-off entries that do not recur: propose deleting them.
5. Present the full list grouped as "promote to config" vs "delete as one-off" with rationale. Wait for approval before changing anything.

## Step 4 — Findings report

Present all findings grouped by severity before making any changes:

**Must fix** (correctness or consistency):

- `copilot-instructions.md` over 8,000 characters
- Invalid or missing `applyTo` in path-specific instruction files
- Wrong job name in `copilot-setup-steps.yml`
- Contradictions between `AGENTS.md` and `copilot-instructions.md`

**Should fix** (quality and effectiveness):

- Anti-patterns in `copilot-instructions.md`
- Missing or vague Commands section
- Missing `copilot-setup-steps.yml` when a build system is detected
- Missing dependency caching in `copilot-setup-steps.yml`
- Missing `postToolUse` formatter hook when a formatter is present in the project
- Broken hook scripts (non-zero exit on failure, missing files, not executable)
- Learnings entries that should be promoted to config

**Nice to have** (polish):

- Architecture overview missing from `copilot-instructions.md`
- No path-specific instruction files for a multi-subsystem project
- `preToolUse` blocking hook for sensitive files (`.env`, `secrets/`)
- Learnings entries that are one-offs (clean up the file)

## Step 5 — Apply approved changes and report

Wait for the user to approve the findings. Apply only approved changes (including learnings promotions and deletions).

For each learnings entry promoted to a config file, remove it from `copilot-learnings.md`. For entries marked as one-offs, remove them too. If all entries are processed, delete `copilot-learnings.md` entirely — it will be recreated naturally when the next correction occurs.

**After applying changes, report:**

- Every file modified or created, with a one-line description of changes
- Before/after metrics: character count, hook count, instruction file count, learnings entry count
- How many learnings entries were promoted, deleted, or remain

---

Did this output meet your expectations? If not, describe what was off and Copilot will log the correction to `.github/copilot-learnings.md`.

> **Note:** Corrections are not auto-loaded on every session. Run `/gc-config-optimize` periodically to review and incorporate them.
