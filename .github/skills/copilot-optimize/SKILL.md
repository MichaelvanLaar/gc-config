---
name: copilot-optimize
description: Audit and optimize an existing GitHub Copilot Coding Agent configuration against current best practices. Use when a user asks to review, improve, or clean up their Copilot setup, copilot-instructions.md, path-specific instruction files, or setup steps workflow.
license: MIT
compatibility: GitHub Copilot with file read/write access.
metadata:
  author: MichaelvanLaar
  version: "1.0"
  generatedBy: "1.0.0"
---

# Optimize GitHub Copilot Configuration

You are auditing and improving an existing GitHub Copilot Coding Agent setup. Your job is to identify what's good (preserve it), what's missing, what's bloated, and what violates current best practices — then fix it with the user's approval.

## Step 1: Full inventory

Read and catalog everything that exists before suggesting any changes.

### Configuration files to read

- `.github/copilot-instructions.md`
- All `.github/instructions/*.instructions.md` files
- `.github/workflows/copilot-setup-steps.yml`
- `AGENTS.md` (project root)
- `CLAUDE.md` and `GEMINI.md` (project root, if present — for cross-tool consistency)
- `.github/copilot-learnings.md` (if it exists — accumulated corrections from past skill runs)

If `.github/copilot-instructions.md` does not exist, **stop** — tell the user there is no Copilot configuration to optimize and suggest running `copilot-init` instead.

### Project context

- Package manager and dependencies (`package.json`, `Cargo.toml`, `go.mod`, `requirements.txt`, etc.)
- Build/test/lint commands
- CI/CD configuration in `.github/workflows/`
- Other AI tool directories: `.claude/`, `.gemini/`, `.codex/`

### Metrics snapshot

Report before proposing any changes:

- `copilot-instructions.md` character count (limit: ~8 000)
- Number of path-specific instruction files in `.github/instructions/`
- Whether `copilot-setup-steps.yml` exists
- Whether `AGENTS.md` exists

## Step 2: Length audit

Check that `copilot-instructions.md` stays within the ~8 000 character limit. Content beyond this limit may be silently truncated by the Coding Agent.

- **Under 8 000 characters** → no length finding
- **8 000–10 000 characters** → flag as "should fix"; suggest sections to move to path-specific instruction files
- **Over 10 000 characters** → flag as "must fix"; report the excess and list the top candidates for extraction

## Step 3: Content quality audit

Check `copilot-instructions.md` for missing essentials and known anti-patterns.

### Anti-patterns to flag as "should fix"

- Personality instructions ("act as a senior engineer", "think carefully")
- File-by-file codebase descriptions (the Coding Agent can read files itself)
- Rules that the configured linter/formatter already enforces
- Duplicate content that also exists in `AGENTS.md`

### Missing essentials to flag as "should fix"

- No Commands section or only vague commands (the Coding Agent needs exact command strings to avoid CI failures)

### Missing structure to flag as "nice to have"

- No architecture overview or directory layout section

## Step 4: Path-specific instruction coverage

Assess whether `.github/instructions/*.instructions.md` files would improve the setup.

- **Multi-language or multi-subsystem project with no path-specific files** → flag as "nice to have" and suggest glob patterns (e.g., `src/frontend/**`, `**/*.test.ts`)
- **Existing path-specific files with missing or malformed `applyTo` frontmatter** → flag as "must fix"; a missing `applyTo` field means the file never applies

## Step 5: copilot-setup-steps.yml audit

If the project has a build system, package manager, or dependency manager:

- **Build system detected but `copilot-setup-steps.yml` absent** → flag as "should fix"; pre-installing dependencies reduces failed agent sessions and token waste
- **`copilot-setup-steps.yml` exists** → verify it has the required `copilot-setup-steps` job name; flag any structural issues as "must fix"
- **Missing dependency caching** → flag as "should fix"; without caching, every agent session reinstalls all dependencies from scratch, wasting GitHub Actions minutes:
  - `actions/setup-node` without `cache: 'npm'` (or `'yarn'`/`'pnpm'`) → suggest adding `cache:` parameter
  - `actions/setup-python` without `cache: 'pip'` → suggest adding `cache:` parameter
  - `actions/setup-go` without `cache: true` → suggest adding `cache:` parameter
  - Raw `cargo install` / `cargo build` without `actions/cache@v4` targeting `~/.cargo/registry`, `~/.cargo/git`, and `target/` → suggest adding cache step

If no build system is detected, skip this step.

## Step 6: AGENTS.md consistency check

If `AGENTS.md` exists:

- Compare coding rules and commands between `AGENTS.md` and `copilot-instructions.md`
- **Contradictory rules** → flag as "must fix" and show both conflicting passages
- **Duplicate content** → flag as "should fix" and suggest which file should be the single source of truth

## Step 7: Learnings review

If `.github/copilot-learnings.md` exists:

1. Read all entries and count them.
2. Classify each entry:
   - **Recurring pattern** — the same theme appears multiple times, or the correction points to a persistent gap in the configuration (e.g., a missing convention, an incorrect command). Candidate for promotion into `copilot-instructions.md`, a path-specific instruction file, or `AGENTS.md`.
   - **One-off correction** — specific to a single past interaction, unlikely to recur. Candidate for deletion.
3. Include learnings findings in the report (Step 8). Promoted corrections map to the "nice to have" or "should fix" tier depending on their significance.
4. When applying approved changes:
   - For promoted entries: add the substance to the appropriate config file, then remove the entry from `copilot-learnings.md`.
   - For one-off entries: remove the entry from `copilot-learnings.md`.
   - If all entries are processed: delete `copilot-learnings.md` entirely — it will be recreated naturally when the next correction occurs.

If `.github/copilot-learnings.md` does not exist, skip this step.

> **Learnings graduation:** Recurring corrections should graduate into durable config — a rule in `copilot-instructions.md`, a path-specific instruction, or an `AGENTS.md` entry. One-off corrections get deleted. The file stays lean until the next cycle.

## Step 8: Generate findings report

Present all findings grouped by category. For each finding, state what the issue is, why it matters, and what you'd change.

### Must fix (correctness issues)

- `copilot-instructions.md` exceeds 8 000 characters
- Path-specific instruction file missing or invalid `applyTo` field
- `copilot-setup-steps.yml` missing required `copilot-setup-steps` job name
- Contradictory rules between `AGENTS.md` and `copilot-instructions.md`

### Should fix (quality improvements)

- Anti-patterns in `copilot-instructions.md`
- Missing Commands section
- Build system present but no `copilot-setup-steps.yml`
- Duplicate content between `AGENTS.md` and `copilot-instructions.md`

### Nice to have (polish)

- No architecture overview
- No path-specific instruction files for a multi-subsystem project
- No `AGENTS.md` when multiple AI tools are in use
- Learnings entries that should be promoted to a config file (from Step 7)

**Wait for user approval before making any changes.**

## Step 9: Apply approved changes

Make only the approved changes. For each modified file, show a brief before/after summary. Process any approved learnings changes as described in Step 7.

## Step 10: Final summary

After all changes:

1. List every file modified with a one-line description of changes
2. Report new metrics: character count, number of instruction files, etc.
3. Compare to before (e.g., "`copilot-instructions.md`: 12 400 chars → 5 800 chars")
4. If learnings were reviewed: report how many entries were promoted, how many deleted, and how many remain in `.github/copilot-learnings.md`
5. Note anything intentionally left unchanged and why
6. Remind the user to commit the changes

### What this skill cannot do

Unlike the Claude Code equivalent (`cc-optimize`), this skill cannot configure:

- **No permissions deny/allow** — Copilot has no file-level access control
- **No PostToolUse hooks** — no hook system for auto-formatting after edits
- **No autocompact control** — no equivalent to `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE`
- **No `@`-import progressive disclosure** — `copilot-instructions.md` loads in full on every session; unlike Claude Code's `CLAUDE.md`, it cannot `@`-import `copilot-learnings.md` automatically
- **No MCP automation** — MCP servers are configured in GitHub repository settings UI, not via files

## What NOT to do

- Don't modify files before getting user approval
- Don't refactor things that work well — if a config is correct and clean, say so
- Don't suggest MCP servers here — those are configured via the GitHub UI

---

Did this output meet your expectations? If not, describe what was off and Copilot will log the correction to `.github/copilot-learnings.md`.

> **Note:** Unlike the Claude Code equivalent (`cc-optimize`), corrections are not auto-loaded on every session — Copilot has no `@`-import support. Run `copilot-optimize` periodically to review accumulated learnings and promote them into your configuration.
