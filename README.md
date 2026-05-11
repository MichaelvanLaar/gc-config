# gc-config

Two GitHub Copilot CLI skills for setting up and maintaining a best-practice GitHub Copilot Coding Agent configuration, distributed as a Copilot CLI plugin.

**`/gc-config-init`** bootstraps a lean configuration for a new or unconfigured project — creating `copilot-instructions.md`, path-specific instruction files, `copilot-setup-steps.yml`, and `AGENTS.md` where applicable.

**`/gc-config-optimize`** audits and improves an existing Copilot configuration against current best practices — checking for the 8,000-character limit, anti-patterns, missing sections, consistency between files, and accumulated learnings.

Both skills are grounded in the official GitHub Copilot Coding Agent documentation and research on agent instruction design.

## Table of Contents

- [gc-config](#gc-config)
  - [Table of Contents](#table-of-contents)
  - [What problem do these skills solve?](#what-problem-do-these-skills-solve)
  - [Installation](#installation)
    - [Keeping skills current](#keeping-skills-current)
    - [Uninstalling](#uninstalling)
  - [Usage](#usage)
    - [`/gc-config-init` — Bootstrap a new project](#gc-config-init--bootstrap-a-new-project)
    - [`/gc-config-optimize` — Audit and improve an existing setup](#gc-config-optimize--audit-and-improve-an-existing-setup)
    - [Recommended workflow](#recommended-workflow)
  - [What the skills create and check](#what-the-skills-create-and-check)
    - [Configuration files](#configuration-files)
    - [Key best practices applied](#key-best-practices-applied)
  - [What these skills cannot configure](#what-these-skills-cannot-configure)
  - [Compatibility](#compatibility)
  - [Contributing](#contributing)
  - [License](#license)

## What problem do these skills solve?

GitHub Copilot's Coding Agent reads configuration from several sources — a global instructions file, path-specific instruction files, a setup workflow, and `AGENTS.md`. Without guidance, these files tend to accumulate noise: personality instructions, file-by-file descriptions, rules that a linter already enforces, and content that blows past the 8,000-character limit without warning.

These skills take a different approach:

- **`/gc-config-init`** creates the minimum viable configuration that's correct from day one: a focused `copilot-instructions.md` (under the character limit), path-specific instruction files scoped with correct `applyTo` globs, and a properly structured `copilot-setup-steps.yml` if a build system is detected.

- **`/gc-config-optimize`** treats your existing configuration as a codebase to audit. It inventories every config file, measures character counts, checks for known anti-patterns, and presents findings in three tiers — must fix, should fix, nice to have — before touching anything.

## Installation

Open GitHub Copilot CLI in any project and run:

```
/plugin marketplace add MichaelvanLaar/gc-config
/plugin install gc-config@gc-config
```

That's it. GitHub Copilot CLI downloads the skills and makes `/gc-config-init` and `/gc-config-optimize` available immediately.

> **Note:** Skills are not updated automatically. See [Keeping skills current](#keeping-skills-current) to update.

### Keeping skills current

Skills are not updated automatically. To update to the latest version, run:

```
/plugin update gc-config@gc-config
```

### Uninstalling

To remove the plugin and the marketplace in one step:

```
/plugin marketplace remove gc-config
```

Removing the marketplace automatically uninstalls any plugins installed from it. To remove only the plugin while keeping the marketplace:

```
/plugin uninstall gc-config@gc-config
```

## Usage

### `/gc-config-init` — Bootstrap a new project

Start GitHub Copilot CLI in your project directory and invoke the skill:

```
/gc-config-init
```

Or with a brief project description to skip some questions:

```
/gc-config-init Next.js 14 e-commerce platform with Stripe and Postgres
/gc-config-init Rust CLI tool with clap, targeting Linux and macOS
```

The skill will:

1. **Scan** for existing Copilot config files and abort if `copilot-instructions.md` already exists (use `/gc-config-optimize` instead).
2. **Detect** the project's toolchain to propose relevant `copilot-setup-steps.yml` content and infer what path-specific instruction files might be useful.
3. **Create** up to four outputs:
   - `.github/copilot-instructions.md` — global agent instructions, kept under 8,000 characters
   - `.github/instructions/*.instructions.md` — path-specific instruction files with correct `applyTo` glob patterns (offered, not forced)
   - `.github/workflows/copilot-setup-steps.yml` — pre-install workflow when a build system is detected (job name is always `copilot-setup-steps`, runtime capped at 59 minutes)
   - `AGENTS.md` — only when evidence of a multi-tool AI environment is found
4. **Summarize** what was created, what was skipped, and why.

### `/gc-config-optimize` — Audit and improve an existing setup

Invoke the skill at any time:

```
/gc-config-optimize
```

Or focused on a specific area:

```
/gc-config-optimize length
/gc-config-optimize caching
```

The skill will:

1. **Inventory** all Copilot config files and take a metrics snapshot: character counts, number of path-specific instruction files, presence of `copilot-setup-steps.yml`, `AGENTS.md`, and `copilot-learnings.md`.
2. **Audit** against best practices, checking for:
   - **Must fix**: over the 8,000-character limit, invalid `applyTo` globs, wrong job name in `copilot-setup-steps.yml`, contradictions between `AGENTS.md` and `copilot-instructions.md`
   - **Should fix**: anti-patterns (personality instructions, file-by-file descriptions, linter-enforced rules), missing Commands section, missing `copilot-setup-steps.yml` when a build system is detected, missing dependency caching in `copilot-setup-steps.yml`
   - **Nice to have**: missing architecture overview, no path-specific files for a multi-subsystem project
3. **Review learnings**: if `.github/copilot-learnings.md` exists, classify each entry as a recurring pattern (promote into config) or a one-off (delete), and include findings in the report.
4. **Present** all findings grouped by severity tier before touching anything.
5. **Wait for your approval**, then apply only the changes you approve.
6. **Report** before/after metrics for every modified file, plus how many learnings entries were promoted, deleted, or remain.

### Recommended workflow

```
Day 1:    /gc-config-init                   ← Bootstrap config for a new project
          ... start coding ...

Week 1:   /gc-config-optimize               ← First audit pass with real code context
          ... continue building ...

Ongoing:  /gc-config-optimize               ← Periodic hygiene checks; incorporates accumulated learnings
          /gc-config-optimize length        ← When copilot-instructions.md has grown
```

## What the skills create and check

### Configuration files

| File                                        | Created by                        | Purpose                                                                                                                          |
| ------------------------------------------- | --------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `.github/copilot-instructions.md`           | `/gc-config-init`                 | Global agent instructions, loaded every session (limit: ~8,000 characters)                                                       |
| `.github/instructions/*.instructions.md`    | `/gc-config-init` (optional)      | Path-specific instructions, scoped via `applyTo` glob in frontmatter                                                             |
| `.github/workflows/copilot-setup-steps.yml` | `/gc-config-init` (optional)      | Pre-install dependencies before Copilot runs; job must be named `copilot-setup-steps`; max 59 minutes                            |
| `AGENTS.md`                                 | `/gc-config-init` (optional)      | Vendor-neutral agent instructions for multi-tool AI environments                                                                 |
| `.github/copilot-learnings.md`              | Created by Copilot on corrections | Accumulates one-line corrections from skill feedback steps; reviewed and incorporated by `/gc-config-optimize`                   |
| `.github/hooks/hooks.json`                  | `/gc-config-init` (optional)      | PostToolUse formatter hook (runs after file edits) and optional preToolUse blocking hook (rejects tool calls on sensitive files) |

### Key best practices applied

- **Stay under 8,000 characters**: `copilot-instructions.md` is loaded on every agent session. Content past ~8,000 characters is silently truncated. The skills measure this and flag overruns.
- **Path-specific scoping**: use `.github/instructions/*.instructions.md` with `applyTo` glob patterns to deliver frontend, backend, or test-specific context only when relevant — avoids bloating the global file.
- **`applyTo` correctness**: invalid glob patterns silently fail. The skills check that every `applyTo` value is a valid glob.
- **`copilot-setup-steps.yml` constraints**: the workflow job must be named exactly `copilot-setup-steps` or Copilot ignores it. Runtime is capped at 59 minutes. The skills enforce both.
- **Dependency caching in setup steps**: without caching, every agent session reinstalls all dependencies from scratch. The skills generate and check for ecosystem-specific caching: `actions/setup-node@v4` with `cache: 'npm'`, `actions/setup-python@v5` with `cache: 'pip'`, `actions/setup-go@v5` with `cache: true`, and `actions/cache@v4` for Cargo.
- **Remove anti-patterns**: personality instructions ("be concise"), file-by-file descriptions, and rules that a linter already enforces all waste the character budget without adding value.
- **Commands section**: a `## Commands` section with the project's build, test, and lint commands gives Copilot the verification loop it needs to self-check its own work.
- **`AGENTS.md` consistency**: when both `AGENTS.md` and `copilot-instructions.md` exist, the skills check for contradictions between them.
- **Learning and improvement**: each skill ends with a feedback step. When Copilot makes a mistake, the correction is logged as a one-line entry in `.github/copilot-learnings.md`. Running `/gc-config-optimize` periodically reviews these entries, promotes recurring patterns into the configuration, and deletes one-offs — keeping the learnings file lean or removing it until the next correction cycle.
- **PostToolUse formatter hook**: `.github/hooks/hooks.json` with a `postToolUse` hook runs a formatter script automatically after every file edit — same effect as Claude Code's PostToolUse hook. The skills create and audit this hook. Hook scripts must exit 0 (they cannot block after the fact); use `|| true` on the formatter command.

## What these skills cannot configure

Unlike the companion [cc-config](https://github.com/MichaelvanLaar/cc-config) for Claude Code, these skills target GitHub Copilot's configuration surface. Several Claude Code features have no Copilot equivalent:

| Claude Code feature                                     | Status in GitHub Copilot                                                                                                                                                        |
| ------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `permissions.deny` / `permissions.allow`                | `preToolUse` hooks in `.github/hooks/hooks.json` provide a partial equivalent — blocking scripts that can reject tool calls before they execute                                 |
| Autocompact control (`CLAUDE_AUTOCOMPACT_PCT_OVERRIDE`) | No equivalent                                                                                                                                                                   |
| `@`-import progressive disclosure                       | `copilot-instructions.md` loads in full every session                                                                                                                           |
| Learnings auto-loading                                  | `CLAUDE.md` can reference `learnings.md` so corrections load automatically; `copilot-learnings.md` is passive — run `/gc-config-optimize` explicitly to incorporate corrections |
| MCP server automation via files                         | Copilot CLI supports `~/.copilot/mcp-config.json` (file-based, per machine); the Coding Agent requires GitHub repository Settings → Copilot → MCP servers                       |
| `.claude/context/` shared domain folder                 | No equivalent; content must live in `copilot-instructions.md` or path-specific files                                                                                            |
| Content exclusions (file patterns to hide from Copilot) | Configured in GitHub UI only — Organization or repository Settings → Copilot → Content exclusion                                                                                |
| Spending limits / budget caps                           | Configured in GitHub UI only — Organization Settings → Billing → GitHub Copilot                                                                                                 |

## Compatibility

- Works with any programming language, framework, or build tool.
- Works with content projects (static sites, documentation sets, Markdown-driven workflows).
- Supports multi-tool AI environments (Claude Code, Cursor, Gemini) via `AGENTS.md`.
- Requires GitHub Copilot CLI.

## Contributing

Issues and pull requests are welcome. If you've found a best practice that isn't covered, or a pattern that the skills should detect, please open an issue.

## License

[MIT](LICENSE)
