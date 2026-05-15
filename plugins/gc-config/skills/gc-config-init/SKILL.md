---
name: gc-config-init
description: Bootstrap a best-practice GitHub Copilot Coding Agent configuration for a new or unconfigured project. Use when a user asks to set up GitHub Copilot, create copilot-instructions.md, or configure GitHub Copilot for the first time. Also use when the user says things like "set up my Copilot config", "bootstrap Copilot", or "initialize GitHub Copilot". Grounded in official GitHub Copilot Coding Agent best practices.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
argument-hint: "[optional: brief project description]"
---

Set up a GitHub Copilot Coding Agent configuration for this project.

## Step 0 — Recall learnings

If `.github/copilot-learnings.md` exists, read all entries and apply them silently to inform this run. The `[skill-name]` tag on each entry is provenance only — all entries apply regardless of which skill wrote them. Do not announce that learnings were loaded.

If the file does not exist, proceed without mention.

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
- **Learnings:** Project-specific discoveries are automatically stored in `.github/copilot-learnings.md` and recalled on future runs of either skill. Run `/gc-config-optimize` periodically to promote recurring patterns into the configuration.

## Store learnings

Before responding, review this run against the notice criteria below. For each entry that qualifies, append one line to `.github/copilot-learnings.md` (create the file if it does not exist), tagged `[gc-config-init]`. Skip any entry that duplicates one already in the file. If nothing qualifies, do not create or modify the file — no user notification either way.

**Qualifies:**

- Something about this repo that differs from what this skill assumes on a generic project
- A suggestion the user explicitly accepted or rejected that deviates from skill defaults
- A constraint or fact discovered that would change how this skill behaves next time

**Does not qualify:**

- Standard skill behavior applied without deviation
- Facts already present in AGENTS.md, CLAUDE.md, or other config files
- Anything a reader could determine from the repo without this skill having run

Entry format:

```
[gc-config-init] <concise fact about this project>
```

---

Did this output meet your expectations? If not, describe what was off and Copilot will log the correction to `.github/copilot-learnings.md`.

> **Note:** Learnings are automatically recalled on the next run of either skill. Run `/gc-config-optimize` periodically to promote recurring patterns into the configuration.
