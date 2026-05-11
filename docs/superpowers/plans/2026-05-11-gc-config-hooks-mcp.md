# gc-config Hooks and MCP Enhancement Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rewrite both gc-config skills to add Copilot hooks support (postToolUse formatter, preToolUse blocking), correct MCP documentation, and remove the outdated "Limitations" footers.

**Architecture:** Three file edits — rewrite `gc-config-init/SKILL.md` (63 → ~200 lines, 7 named steps), rewrite `gc-config-optimize/SKILL.md` (55 → ~230 lines, 5 named steps with sub-sections), update `README.md` (three targeted edits: add hooks.json row, add best-practices bullet, fix three table rows).

**Tech Stack:** Markdown, YAML frontmatter, bash (for hook script templates embedded in the skill text). Content-only repo — no build toolchain.

---

## File Map

| File                                                   | Change               |
| ------------------------------------------------------ | -------------------- |
| `plugins/gc-config/skills/gc-config-init/SKILL.md`     | Full rewrite         |
| `plugins/gc-config/skills/gc-config-optimize/SKILL.md` | Full rewrite         |
| `README.md`                                            | Three targeted edits |

---

## Task 1: Rewrite `gc-config-init/SKILL.md`

**Files:**

- Modify: `plugins/gc-config/skills/gc-config-init/SKILL.md`

- [ ] **Step 1: Read the current file**

```bash
cat plugins/gc-config/skills/gc-config-init/SKILL.md
```

Confirm the file is 63 lines and ends with the "Limitations vs. Claude Code" footer.

- [ ] **Step 2: Write the new content**

Replace the entire file with:

````markdown
---
name: gc-config-init
description: Bootstrap a best-practice GitHub Copilot Coding Agent configuration for a new or unconfigured project. Use when a user asks to set up GitHub Copilot, create copilot-instructions.md, or configure GitHub Copilot for the first time. Also use when the user says things like "set up my Copilot config", "bootstrap Copilot", or "initialize GitHub Copilot". Grounded in official GitHub Copilot Coding Agent best practices.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
argument-hint: "[optional: brief project description]"
---

Set up a GitHub Copilot Coding Agent configuration for this project.

## Step 1 — Gather context

Before creating any files:

1. If `.github/copilot-instructions.md` already exists, stop and suggest running `/gc-config-optimize` instead.
2. Scan for toolchain clues: `package.json`, `Cargo.toml`, `go.mod`, `pyproject.toml`, `requirements.txt`, `Makefile`, `README.md`, and similar.
3. Check if `.github/hooks/hooks.json` already exists (skip Step 2 if present).
4. Note which formatters are present: Prettier (`.prettierrc*`, `prettier` in `package.json`), ruff (`ruff.toml`, `[tool.ruff]` in `pyproject.toml`), rustfmt (`rustfmt.toml`), gofmt (any Go project), php-cs-fixer (`.php-cs-fixer.php`).
5. If no project description was provided in `$ARGUMENTS` and the toolchain cannot be determined, ask: what does this project produce and what stack is involved?

## Step 2 — Create `.github/hooks/hooks.json`

Skip this step if `.github/hooks/hooks.json` already exists.

If a formatter was detected in Step 1, create `.github/hooks/hooks.json`:

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
````

Also create `.github/hooks/scripts/format.sh`. The script receives the hook payload as JSON on stdin — extract the edited file path from it:

```bash
#!/usr/bin/env bash
set -euo pipefail

INPUT=$(cat)
FILE=$(echo "$INPUT" | jq -r '.toolInput.path // .toolInput.file_path // empty')

if [[ -z "$FILE" || ! -f "$FILE" ]]; then
  exit 0
fi
```

Then append the formatter call for the detected ecosystem:

- **JS/TS/Markdown (Prettier):** `npx prettier --write "$FILE" || true`
- **Python (ruff):** `ruff format "$FILE" || true`
- **Rust (rustfmt):** `rustfmt "$FILE" || true`
- **Go (gofmt):** `gofmt -w "$FILE" || true`
- **PHP (php-cs-fixer):** `php-cs-fixer fix "$FILE" || true`

Make the script executable: run `chmod +x .github/hooks/scripts/format.sh` after creating it.

**PostToolUse scripts must exit 0.** They run after the tool completes and cannot block it — use `|| true` on the formatter command so a formatter error never crashes the hook.

**Optional — preToolUse guard for sensitive files:**

Ask the user: "Would you like a preToolUse hook that blocks writes to `.env` and `secrets/` files? This is the Copilot equivalent of `permissions.deny`."

If yes, add a `preToolUse` entry to `hooks.json`:

```json
"preToolUse": [
  {
    "type": "command",
    "bash": ".github/hooks/scripts/guard.sh",
    "cwd": ".",
    "timeoutSec": 10
  }
]
```

And create `.github/hooks/scripts/guard.sh`:

```bash
#!/usr/bin/env bash
INPUT=$(cat)
TARGET=$(echo "$INPUT" | jq -r '.toolInput.path // .toolInput.file_path // empty')

if [[ -n "$TARGET" ]] && [[ "$TARGET" =~ (^|/)\\.env($|\\.) || "$TARGET" =~ (^|/)secrets/ ]]; then
  echo "Blocked: writes to .env and secrets/ are not allowed" >&2
  exit 1
fi
exit 0
```

Make the script executable: `chmod +x .github/hooks/scripts/guard.sh`

**PreToolUse scripts block by exiting 1.** The script reads a JSON payload on stdin with `.toolName` and `.toolInput`. Exiting 0 allows the tool call to proceed.

If no formatter was detected in Step 1, skip hooks.json creation and note it in the Step 7 summary.

## Step 3 — Create `.github/copilot-instructions.md`

Create the file with this structure (keep total under ~8,000 characters):

```markdown
# <Project Name>

<One-line description and stack summary.>

## Commands

- Build: `<command or TODO>`
- Test: `<command or TODO>`
- Lint: `<command or TODO>`

## Architecture

<Key directories and patterns — only non-obvious parts. Omit if too new to know.>

## Conventions

<Concrete rules that deviate from defaults or that Copilot commonly gets wrong.
Never standard language conventions. Omit if the project is too new.>

## Don't

- Don't commit secrets or credentials to git
- Don't use `--force` git flags — fix the underlying issue instead
```

Fill in what the toolchain scan revealed. Use `TODO` placeholders for commands that cannot be determined yet.

## Step 4 — Create path-specific instruction files _(optional)_

Offer to create `.github/instructions/*.instructions.md` files when the project has distinct subsystems with different conventions (frontend/backend, tests, API layer). Each file requires an `applyTo` frontmatter field:

```markdown
---
applyTo: "src/frontend/**"
---

<Frontend-specific conventions here.>
```

Do not create these if the project is too new or the subsystems are not yet clear. Explain that they can be created later.

## Step 5 — Create `copilot-setup-steps.yml` _(optional)_

Create `.github/workflows/copilot-setup-steps.yml` only when a package manager or build tool is detected. The job name must be exactly `copilot-setup-steps` and runtime must stay under 59 minutes. Always include dependency caching — without it, every agent session reinstalls all dependencies from scratch.

Example for a Node.js project:

```yaml
name: Copilot Setup Steps
on: workflow_call

jobs:
  copilot-setup-steps:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"
      - run: npm ci
```

Adapt caching to the detected ecosystem:

- Python: `actions/setup-python@v5` with `cache: 'pip'`
- Go: `actions/setup-go@v5` with `cache: true`
- Rust: `actions/cache@v4` targeting `~/.cargo` and `target/`

## Step 6 — Create `AGENTS.md` _(optional)_

Create `AGENTS.md` only when there is evidence that other AI coding tools are used (`.claude/`, `.gemini/`, `.codex/` directories, or Cursor config files present). `AGENTS.md` is the vendor-neutral standard read by Codex, Amp, Cursor, Copilot, and others.

Keep it focused on universal concerns: setup commands, architecture boundaries, code style, testing conventions, and safety rules. Do not include Copilot-specific syntax.

## Step 7 — Summary

List every file created, note any `TODO` placeholders that still need filling in, and include:

- **Fill in TODO commands** once the build/test/lint commands are known.
- **MCP servers:** For Copilot CLI, MCP servers can be configured in `~/.copilot/mcp-config.json`. For the Coding Agent, use GitHub repository Settings → Copilot → MCP servers.
- **Run `/gc-config-optimize`** once the project has more content for a full audit.
- **Learnings:** When Copilot makes a mistake and the user corrects it, Copilot logs a one-line correction to `.github/copilot-learnings.md`. Run `/gc-config-optimize` periodically to incorporate these into the configuration.

---

Did this output meet your expectations? If not, describe what was off and Copilot will log the correction to `.github/copilot-learnings.md`.

> **Note:** Corrections are not auto-loaded on every session. Run `/gc-config-optimize` periodically to review and incorporate them.

````

- [ ] **Step 3: Verify the rewrite**

Read the file and confirm:
- YAML frontmatter is present with `name`, `description`, `allowed-tools`, `argument-hint`
- File contains exactly 7 `## Step N` headings
- Step 2 contains the `hooks.json` JSON block and `format.sh` bash block
- Step 7 contains the MCP servers note
- The file does **not** contain the string "Limitations vs. Claude Code"

- [ ] **Step 4: Commit**

```bash
git add plugins/gc-config/skills/gc-config-init/SKILL.md
git commit -m "feat: ✨ rewrite gc-config-init with hooks support and 7-step format"
````

---

## Task 2: Rewrite `gc-config-optimize/SKILL.md`

**Files:**

- Modify: `plugins/gc-config/skills/gc-config-optimize/SKILL.md`

- [ ] **Step 1: Read the current file**

```bash
cat plugins/gc-config/skills/gc-config-optimize/SKILL.md
```

Confirm the file is 55 lines and ends with the "Limitations vs. Claude Code" footer.

- [ ] **Step 2: Write the new content**

Replace the entire file with:

````markdown
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
````

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

````

- [ ] **Step 3: Verify the rewrite**

Read the file and confirm:
- YAML frontmatter is present with `name`, `description`, `allowed-tools`, `argument-hint`
- `argument-hint` now includes `'hooks'` as an example focus area
- File contains exactly 5 `## Step N` headings
- Step 2 contains sub-sections `### 2a` through `### 2e`
- Section `### 2e` contains the `hooks.json` JSON block and `format.sh` bash block
- The file does **not** contain the string "Limitations vs. Claude Code"

- [ ] **Step 4: Commit**

```bash
git add plugins/gc-config/skills/gc-config-optimize/SKILL.md
git commit -m "feat: ✨ rewrite gc-config-optimize with hooks audit and named-step format"
````

---

## Task 3: Update `README.md`

**Files:**

- Modify: `README.md`

- [ ] **Step 1: Add `.github/hooks/hooks.json` row to the configuration files table**

The configuration files table currently ends at line 154 (the `copilot-learnings.md` row). Insert a new row after that row and before the closing blank line:

Find this exact text:

```
| `.github/copilot-learnings.md`              | Created by Copilot on corrections | Accumulates one-line corrections from skill feedback steps; reviewed and incorporated by `/gc-config-optimize` |
```

Replace with:

```
| `.github/copilot-learnings.md`              | Created by Copilot on corrections | Accumulates one-line corrections from skill feedback steps; reviewed and incorporated by `/gc-config-optimize` |
| `.github/hooks/hooks.json`                  | `/gc-config-init` (optional)      | PostToolUse formatter hook (runs after file edits) and optional preToolUse blocking hook (rejects tool calls on sensitive files) |
```

- [ ] **Step 2: Add PostToolUse formatter bullet to "Key best practices applied"**

The section currently ends at line 166. Add a new bullet after the existing last bullet ("Learning and improvement: ..."):

Find this exact text:

```
- **Learning and improvement**: each skill ends with a feedback step. When Copilot makes a mistake, the correction is logged as a one-line entry in `.github/copilot-learnings.md`. Running `/gc-config-optimize` periodically reviews these entries, promotes recurring patterns into the configuration, and deletes one-offs — keeping the learnings file lean or removing it until the next correction cycle.
```

Replace with:

```
- **Learning and improvement**: each skill ends with a feedback step. When Copilot makes a mistake, the correction is logged as a one-line entry in `.github/copilot-learnings.md`. Running `/gc-config-optimize` periodically reviews these entries, promotes recurring patterns into the configuration, and deletes one-offs — keeping the learnings file lean or removing it until the next correction cycle.
- **PostToolUse formatter hook**: `.github/hooks/hooks.json` with a `postToolUse` hook runs a formatter script automatically after every file edit — same effect as Claude Code's PostToolUse hook. The skills create and audit this hook. Hook scripts must exit 0 (they cannot block after the fact); use `|| true` on the formatter command.
```

- [ ] **Step 3: Fix three rows in the "What these skills cannot configure" table**

The table spans lines 172–182. Apply three changes:

**Changes 1 & 2 — Remove the PostToolUse row and update the permissions row in one edit:**

Find this exact block:

```
| `permissions.deny` / `permissions.allow`                | No file-level access control                                                                                                                                                    |
| PostToolUse hooks (auto-formatter)                      | No hook system                                                                                                                                                                  |
| Autocompact control (`CLAUDE_AUTOCOMPACT_PCT_OVERRIDE`) | No equivalent                                                                                                                                                                   |
```

Replace with:

```
| `permissions.deny` / `permissions.allow`                | `preToolUse` hooks in `.github/hooks/hooks.json` provide a partial equivalent — blocking scripts that can reject tool calls before they execute                                  |
| Autocompact control (`CLAUDE_AUTOCOMPACT_PCT_OVERRIDE`) | No equivalent                                                                                                                                                                   |
```

**Change 2 — Update the MCP row**:

Find:

```
| MCP server automation via files                         | MCP servers are configured in GitHub repository settings UI only                                                                                                                |
```

Replace with:

```
| MCP server automation via files                         | Copilot CLI supports `~/.copilot/mcp-config.json` (file-based, per machine); the Coding Agent requires GitHub repository Settings → Copilot → MCP servers                      |
```

- [ ] **Step 4: Verify the README changes**

Read lines 148–183 of `README.md` and confirm:

- The configuration files table has 6 rows (`.github/copilot-instructions.md`, `.github/instructions/`, `copilot-setup-steps.yml`, `AGENTS.md`, `copilot-learnings.md`, `hooks.json`)
- The "Key best practices applied" section ends with the PostToolUse formatter bullet
- The "What these skills cannot configure" table has **no** PostToolUse row
- The `permissions.deny` row now mentions `preToolUse` hooks
- The MCP row mentions `~/.copilot/mcp-config.json`

- [ ] **Step 5: Commit**

```bash
git add README.md
git commit -m "docs: 📝 update README for hooks support and correct MCP/permissions rows"
```
