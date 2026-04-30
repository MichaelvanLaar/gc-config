# github-copilot-config-skills

Three skills for setting up and maintaining a best-practice [GitHub Copilot Coding Agent](https://docs.github.com/en/copilot/using-github-copilot/using-claude-sonnet-in-github-copilot) configuration.

**`copilot-init`** bootstraps a lean configuration for a new or unconfigured project — creating `copilot-instructions.md`, path-specific instruction files, `copilot-setup-steps.yml`, and `AGENTS.md` where applicable.

**`copilot-optimize`** audits and improves an existing Copilot configuration against current best practices — checking for the 8,000-character limit, anti-patterns, missing sections, consistency between files, and accumulated learnings.

**`copilot-update`** fetches the latest versions of all installed skills from this repository — run it any time you want to pick up improvements.

Each skill is available in two formats:

| Format               | Location           | How to use                                        |
| -------------------- | ------------------ | ------------------------------------------------- |
| Copilot native skill | `.github/skills/`  | Reference by name in Copilot Chat                 |
| Copilot prompt file  | `.github/prompts/` | Attach via the prompt-file picker in Copilot Chat |

All three skills are grounded in the official GitHub Copilot Coding Agent documentation and research on agent instruction design.

## Table of Contents

- [What problem do these skills solve?](#what-problem-do-these-skills-solve)
- [Installation](#installation)
- [Usage](#usage)
  - [`/copilot-init` — Bootstrap a new project](#copilot-init--bootstrap-a-new-project)
  - [`/copilot-optimize` — Audit and improve an existing setup](#copilot-optimize--audit-and-improve-an-existing-setup)
  - [`/copilot-update` — Keep skills current](#copilot-update--keep-skills-current)
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

- **`copilot-init`** creates the minimum viable configuration that's correct from day one: a focused `copilot-instructions.md` (under the character limit), path-specific instruction files scoped with correct `applyTo` globs, and a properly structured `copilot-setup-steps.yml` if a build system is detected.

- **`copilot-optimize`** treats your existing configuration as a codebase to audit. It inventories every config file, measures character counts, checks for known anti-patterns, and presents findings in three tiers — must fix, should fix, nice to have — before touching anything.

## Installation

Run the install script from your project directory:

```bash
curl -fsSL https://raw.githubusercontent.com/MichaelvanLaar/github-copilot-config-skills/main/install.sh | bash
```

This downloads both the native skill files and the prompt files for `copilot-init`, `copilot-optimize`, and `copilot-update` into `.github/skills/` and `.github/prompts/`.

To install into a specific directory, or to pin to a release tag:

```bash
# Specific directory
curl -fsSL https://raw.githubusercontent.com/MichaelvanLaar/github-copilot-config-skills/main/install.sh | bash -s path/to/project

# Pin to a specific tag or commit
curl -fsSL https://raw.githubusercontent.com/MichaelvanLaar/github-copilot-config-skills/main/install.sh | REF=v1.0.0 bash
```

If you prefer to inspect the script before running it:

```bash
curl -fsSL https://raw.githubusercontent.com/MichaelvanLaar/github-copilot-config-skills/main/install.sh -o install.sh
# Review install.sh, then:
bash install.sh
rm install.sh
```

### Manual installation

Copy the files you want directly into your project:

- `.github/skills/copilot-*/SKILL.md` — native skill files
- `.github/prompts/copilot-*.prompt.md` — prompt files

### Directory structure after installation

```
your-project/
└── .github/
    ├── skills/
    │   ├── copilot-init/
    │   │   └── SKILL.md
    │   ├── copilot-optimize/
    │   │   └── SKILL.md
    │   └── copilot-update/
    │       └── SKILL.md
    └── prompts/
        ├── copilot-init.prompt.md
        ├── copilot-optimize.prompt.md
        └── copilot-update.prompt.md
```

After running `copilot-init`, additional files are created in your project (see [What the skills create and check](#what-the-skills-create-and-check)).

## Usage

### `copilot-init` — Bootstrap a new project

Attach the `copilot-init` skill or prompt in Copilot Chat. You can optionally include a brief project description to skip some questions:

> Bootstrap my GitHub Copilot config. This is a Next.js 14 e-commerce platform with Stripe and Postgres.

The skill will:

1. **Scan** for existing Copilot config files and abort if `copilot-instructions.md` already exists (use `copilot-optimize` instead).
2. **Detect** the project's toolchain to propose relevant `copilot-setup-steps.yml` content and infer what path-specific instruction files might be useful.
3. **Create** up to four outputs:
   - `.github/copilot-instructions.md` — global agent instructions, kept under 8,000 characters
   - `.github/instructions/*.instructions.md` — path-specific instruction files with correct `applyTo` glob patterns (offered, not forced)
   - `.github/workflows/copilot-setup-steps.yml` — pre-install workflow when a build system is detected (job name is always `copilot-setup-steps`, runtime capped at 59 minutes)
   - `AGENTS.md` — only when evidence of a multi-tool AI environment is found
4. **Summarize** what was created, what was skipped, and why.

### `copilot-optimize` — Audit and improve an existing setup

Attach the `copilot-optimize` skill or prompt in Copilot Chat. You can optionally focus on a specific area:

> Audit my Copilot config, focusing on the length of copilot-instructions.md.

The skill will:

1. **Inventory** all Copilot config files and take a metrics snapshot: character counts, number of path-specific instruction files, presence of `copilot-setup-steps.yml`, `AGENTS.md`, and `copilot-learnings.md`.
2. **Audit** against best practices, checking for:
   - **Must fix**: over the 8,000-character limit, invalid `applyTo` globs, wrong job name in `copilot-setup-steps.yml`, contradictions between `AGENTS.md` and `copilot-instructions.md`
   - **Should fix**: anti-patterns (personality instructions, file-by-file descriptions, linter-enforced rules), missing Commands section, missing `copilot-setup-steps.yml` when a build system is detected
   - **Nice to have**: missing architecture overview, no path-specific files for a multi-subsystem project
3. **Review learnings**: if `.github/copilot-learnings.md` exists, classify each entry as a recurring pattern (promote into config) or a one-off (delete), and include findings in the report.
4. **Present** all findings grouped by severity tier before touching anything.
5. **Wait for your approval**, then apply only the changes you approve.
6. **Report** before/after metrics for every modified file, plus how many learnings entries were promoted, deleted, or remain.

### `copilot-update` — Keep skills current

Attach the `copilot-update` skill or prompt in Copilot Chat any time you want to pull the latest versions:

> Update my copilot-\* skills to the latest versions.

It updates `copilot-init`, `copilot-optimize`, and itself — only for skills already installed in the project. Skills you have not installed are never added.

For each installed skill it updates all file locations if they exist: `.github/skills/` and `.github/prompts/`. `copilot-update` itself is always updated, even when running alone.

### Recommended workflow

```
Day 1:    copilot-init                   ← Bootstrap config for a new project
          ... start coding ...

Week 1:   copilot-optimize               ← First audit pass with real code context
          ... continue building ...

Ongoing:  copilot-optimize               ← Periodic hygiene checks; incorporates accumulated learnings
          copilot-optimize (length)      ← When copilot-instructions.md has grown
          copilot-update                 ← After pulling updates from this repo
```

## What the skills create and check

### Configuration files

| File                                        | Created by                        | Purpose                                                                                                     |
| ------------------------------------------- | --------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `.github/copilot-instructions.md`           | `copilot-init`                    | Global agent instructions, loaded every session (limit: ~8,000 characters)                                  |
| `.github/instructions/*.instructions.md`    | `copilot-init` (optional)         | Path-specific instructions, scoped via `applyTo` glob in frontmatter                                        |
| `.github/workflows/copilot-setup-steps.yml` | `copilot-init` (optional)         | Pre-install dependencies before Copilot runs; job must be named `copilot-setup-steps`; max 59 minutes       |
| `AGENTS.md`                                 | `copilot-init` (optional)         | Vendor-neutral agent instructions for multi-tool AI environments                                            |
| `.github/copilot-learnings.md`              | Created by Copilot on corrections | Accumulates one-line corrections from skill feedback steps; reviewed and incorporated by `copilot-optimize` |

### Key best practices applied

- **Stay under 8,000 characters**: `copilot-instructions.md` is loaded on every agent session. Content past ~8,000 characters is silently truncated. The skills measure this and flag overruns.
- **Path-specific scoping**: use `.github/instructions/*.instructions.md` with `applyTo` glob patterns to deliver frontend, backend, or test-specific context only when relevant — avoids bloating the global file.
- **`applyTo` correctness**: invalid glob patterns silently fail. The skills check that every `applyTo` value is a valid glob.
- **`copilot-setup-steps.yml` constraints**: the workflow job must be named exactly `copilot-setup-steps` or Copilot ignores it. Runtime is capped at 59 minutes. The skills enforce both.
- **Dependency caching in setup steps**: without caching, every agent session reinstalls all dependencies from scratch. The skills generate and check for ecosystem-specific caching: `actions/setup-node@v4` with `cache: 'npm'`, `actions/setup-python@v5` with `cache: 'pip'`, `actions/setup-go@v5` with `cache: true`, and `actions/cache@v4` for Cargo.
- **Remove anti-patterns**: personality instructions ("be concise"), file-by-file descriptions, and rules that a linter already enforces all waste the character budget without adding value.
- **Commands section**: a `## Commands` section with the project's build, test, and lint commands gives Copilot the verification loop it needs to self-check its own work.
- **`AGENTS.md` consistency**: when both `AGENTS.md` and `copilot-instructions.md` exist, the skills check for contradictions between them.
- **Learning and improvement**: each skill ends with a feedback step. When Copilot makes a mistake, the correction is logged as a one-line entry in `.github/copilot-learnings.md`. Running `copilot-optimize` periodically reviews these entries, promotes recurring patterns into the configuration, and deletes one-offs — keeping the learnings file lean or removing it until the next correction cycle.

## What these skills cannot configure

Unlike the companion [claude-code-config-skills](https://github.com/MichaelvanLaar/claude-code-config-skills), these skills target GitHub Copilot's configuration surface. Several Claude Code features have no Copilot equivalent:

| Claude Code feature                                     | Status in GitHub Copilot                                                                                                                                                     |
| ------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `permissions.deny` / `permissions.allow`                | No file-level access control                                                                                                                                                 |
| PostToolUse hooks (auto-formatter)                      | No hook system                                                                                                                                                               |
| Autocompact control (`CLAUDE_AUTOCOMPACT_PCT_OVERRIDE`) | No equivalent                                                                                                                                                                |
| `@`-import progressive disclosure                       | `copilot-instructions.md` loads in full every session                                                                                                                        |
| Learnings auto-loading                                  | `CLAUDE.md` can reference `learnings.md` so corrections load automatically; `copilot-learnings.md` is passive — run `copilot-optimize` explicitly to incorporate corrections |
| MCP server automation via files                         | MCP servers are configured in GitHub repository settings UI only                                                                                                             |
| `.claude/context/` shared domain folder                 | No equivalent; content must live in `copilot-instructions.md` or path-specific files                                                                                         |
| Content exclusions (file patterns to hide from Copilot) | Configured in GitHub UI only — Organization or repository Settings → Copilot → Content exclusion                                                                             |
| Spending limits / budget caps                           | Configured in GitHub UI only — Organization Settings → Billing → GitHub Copilot                                                                                              |

## Compatibility

- Works with any programming language, framework, or build tool.
- Works with content projects (static sites, documentation sets, Markdown-driven workflows).
- Supports multi-tool AI environments (Claude Code, Cursor, Gemini) via `AGENTS.md`.
- The native skills (`.github/skills/`) require GitHub Copilot with file read/write access.
- The prompt files (`.github/prompts/`) work with any GitHub Copilot Chat interface.

## Contributing

Issues and pull requests are welcome. If you've found a best practice that isn't covered, or a pattern that the skills should detect, please open an issue.

## License

[MIT](LICENSE)
