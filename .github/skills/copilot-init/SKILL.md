---
name: copilot-init
description: Bootstrap a best-practice GitHub Copilot Coding Agent configuration for a new or unconfigured project. Use when a user asks to set up Copilot, configure copilot-instructions.md, or prepare a repo for the Copilot Coding Agent.
license: MIT
compatibility: GitHub Copilot with file read/write access.
metadata:
  author: MichaelvanLaar
  version: "1.0"
  generatedBy: "1.0.0"
---

# Bootstrap GitHub Copilot Configuration

You are setting up a GitHub Copilot Coding Agent configuration from scratch. Your goal is to create lean, high-quality baseline files that help the Coding Agent understand the project and avoid trial-and-error on tool installation and conventions.

## Step 1: Gather context

Before creating any files, understand what you're working with.

1. Check for existing Copilot config: if `.github/copilot-instructions.md` already exists, **stop** — tell the user this skill is for fresh setups and suggest running `copilot-optimize` instead.
2. Look for other AI tool directories: `.claude/` (Claude Code), `.gemini/`, `.codex/` — relevant for AGENTS.md in Step 5.
3. Scan for toolchain clues:
   - **Build/package systems**: `package.json`, `composer.json`, `Cargo.toml`, `pyproject.toml`, `go.mod`, `Makefile`, `Gemfile`, `pom.xml`, `build.gradle`, `*.sln`, `*.csproj`, `requirements.txt`
   - **Content/static sites**: `hugo.toml`, `_config.yml`, `astro.config.*`, `mkdocs.yml`, dominant `.md` files
   - Always check `README.md` for project purpose
4. Check for existing quality tools: `.eslintrc*`, `.prettierrc*`, `.editorconfig`, CI configs in `.github/workflows/`
5. Check for sensitive files: `.env`, `.env.*`, `secrets/`

If the user provided a project description in the chat, use it to infer the stack and toolchain without asking further.

If the project is empty or the toolchain cannot be determined, ask the user:

- What does this project produce?
- What stack or toolchain is involved?

## Step 2: Create .github/copilot-instructions.md

Create `.github/copilot-instructions.md`. This file is loaded automatically on every Copilot Coding Agent session — keep it lean (under ~8 000 characters / ~2 pages). The Coding Agent reads it before doing any work.

Use this structure:

```markdown
# <Project Name>

<One-line description. Stack or toolchain summary.>

## Commands

<List exact commands. Examples:

- Build: `npm run build`
- Test: `npm test`
- Lint: `npm run lint`
  If commands are unknown, insert `TODO:` placeholders rather than guessing.>

## Architecture

<Brief overview of key directories and patterns — only non-obvious parts.
Omit if the project is too new to have a meaningful layout.>

## Conventions

<Only concrete rules that deviate from defaults or that the Coding Agent commonly gets wrong.
Never include standard language conventions.>

## Don't

- Don't commit secrets or credentials to git
- Don't use `--force` git flags — fix the underlying issue instead
```

When detecting commands from `package.json`, `Makefile`, etc., include the exact validated strings. Never guess command names.

## Step 3: Offer path-specific instruction files

If the project has distinct subsystems with different conventions (e.g., TypeScript frontend + Python backend, or test files requiring different rules), offer to create one or more `.github/instructions/*.instructions.md` files.

Each file uses an `applyTo` frontmatter glob to scope which files it applies to:

```markdown
---
applyTo: "src/frontend/**"
---

<Instructions specific to files matching this glob.>
```

If the project has uniform conventions throughout, skip this step — global instructions are sufficient.

## Step 4: Offer copilot-setup-steps.yml

If a package manager or build tool is detected (`package.json`, `requirements.txt`, `Cargo.toml`, `go.mod`, etc.), offer to create `.github/workflows/copilot-setup-steps.yml`.

This workflow runs before each Coding Agent session to pre-install dependencies, reducing failed runs and wasted tokens. Requirements: the job name **must** be `copilot-setup-steps`; total runtime must stay under 59 minutes.

Example templates (adapt to detected toolchain — always include dependency caching to reduce Actions minutes consumed on every agent session):

**Node.js (npm):**

```yaml
name: "Copilot Setup Steps"
on: workflow_dispatch

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

**Python (pip):**

```yaml
name: "Copilot Setup Steps"
on: workflow_dispatch

jobs:
  copilot-setup-steps:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
          cache: "pip"
      - run: pip install -r requirements.txt
```

**Go:**

```yaml
name: "Copilot Setup Steps"
on: workflow_dispatch

jobs:
  copilot-setup-steps:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: "stable"
          cache: true
      - run: go mod download
```

**Rust (Cargo):**

```yaml
name: "Copilot Setup Steps"
on: workflow_dispatch

jobs:
  copilot-setup-steps:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/cache@v4
        with:
          path: |
            ~/.cargo/registry
            ~/.cargo/git
            target/
          key: ${{ runner.os }}-cargo-${{ hashFiles('**/Cargo.lock') }}
      - run: cargo fetch
```

Use `actions/setup-node@v4 cache: 'npm'` (or `'yarn'`/`'pnpm'`), `actions/setup-python@v5 cache: 'pip'`, `actions/setup-go@v5 cache: true`, and `actions/cache@v4` for Cargo. Caching is not optional — without it, every agent session reinstalls all dependencies from scratch.

If no build system is found, skip this step and note it in the summary.

## Step 5: Create AGENTS.md (if multi-tool environment)

Create `AGENTS.md` only when evidence of multiple AI coding tools exists:

- `.claude/` directory present (Claude Code in use)
- Other AI tool config directories found (`.gemini/`, `.codex/`, cursor-related)
- User confirms a multi-tool setup

AGENTS.md is vendor-neutral and read by Copilot, Claude Code, Codex, Amp, and others. Focus on: setup commands, architecture boundaries, code style rules, testing conventions, safety rules.

If only GitHub Copilot is configured, skip this step.

## Step 6: Present summary

After creating files, give the user a concise summary:

1. Every file created with a one-line description
2. TODO placeholders that still need filling in
3. What was intentionally left out and why
4. Three next steps:
   - Fill in TODO command placeholders once they are known
   - Run `copilot-optimize` once the project has more content to review
   - Add MCP servers via GitHub repository settings (Settings → Copilot → MCP servers)
5. Explain the Learnings mechanism:
   - When Copilot makes a mistake and the user corrects it at the end of a skill run, Copilot logs a one-line summary to `.github/copilot-learnings.md` instead of modifying config files directly.
   - Unlike the Claude Code equivalent (`cc-init`), these corrections are **not** auto-loaded into every session — Copilot has no `@`-import or dynamic file injection. Run `copilot-optimize` periodically to review accumulated learnings and promote them into the configuration.

### What this skill cannot do

Unlike the Claude Code equivalent (`cc-init`), this skill cannot configure:

- **No permissions deny/allow** — Copilot has no file-level access control
- **No PostToolUse hooks** — no hook system for auto-formatting after edits
- **No autocompact control** — no equivalent to `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE`
- **No `@`-import progressive disclosure** — `copilot-instructions.md` loads in full on every session
- **No MCP automation** — MCP servers are configured in GitHub repository settings UI, not via files

## What NOT to do

- Don't create `.claude/settings.json`, hooks, or `.mcp.json` — these are Claude Code–specific
- Don't create stub files for unsupported features
- Don't over-engineer — an accurate 30-line `copilot-instructions.md` beats an 80-line one full of guesses
- Don't include information you're not confident about — `TODO:` is better than wrong instructions

---

Did this output meet your expectations? If not, describe what was off and Copilot will log the correction to `.github/copilot-learnings.md`.

> **Note:** Unlike the Claude Code equivalent (`cc-init`), corrections are not auto-loaded on every session — Copilot has no `@`-import support. Run `copilot-optimize` periodically to review accumulated learnings and promote them into your configuration.
