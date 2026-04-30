---
description: Audit and optimize an existing GitHub Copilot Coding Agent configuration. Lighter alternative to the full copilot-optimize skill.
---

Audit and improve the GitHub Copilot Coding Agent configuration in this project.

**If `.github/copilot-instructions.md` does not exist**, stop and suggest running `copilot-init` instead.

**Step 1 — Inventory and metrics snapshot**

Read all Copilot config files and report:

- `copilot-instructions.md` character count (limit: ~8 000)
- Number of files in `.github/instructions/`
- Whether `.github/workflows/copilot-setup-steps.yml` exists
- Whether `AGENTS.md` exists

**Step 2 — Inventory** also reads `.github/copilot-learnings.md` if it exists.

**Step 3 — Audit findings** (group as must fix / should fix / nice to have)

Check for:

- **Length**: over 8 000 characters → must fix; suggest which sections to extract
- **Anti-patterns**: personality instructions, file-by-file descriptions, linter-enforced rules → should fix
- **Missing Commands section** or only vague commands → should fix
- **Invalid `applyTo`** in any `.github/instructions/*.instructions.md` file → must fix
- **Missing `copilot-setup-steps.yml`** when a build system is detected → should fix
- **Missing dependency caching** in `copilot-setup-steps.yml` (`actions/setup-node` without `cache:`, `actions/setup-python` without `cache:`, `actions/setup-go` without `cache: true`, raw Cargo commands without `actions/cache@v4`) → should fix; every agent session otherwise reinstalls all dependencies from scratch
- **Wrong job name** in `copilot-setup-steps.yml` (must be `copilot-setup-steps`) → must fix
- **Contradictions** between `AGENTS.md` and `copilot-instructions.md` → must fix
- **No architecture overview** → nice to have
- **No path-specific instruction files** for a multi-subsystem project → nice to have

**Step 4 — Learnings review** (if `.github/copilot-learnings.md` exists)

- Classify each entry: recurring pattern (promote to a config file) or one-off (delete).
- Include promoted entries in the findings report under the appropriate tier.
- When applying changes: add promoted content to config files and remove processed entries from `copilot-learnings.md`; delete the file entirely if all entries are resolved.

**Step 5 — Wait for approval**, then apply only approved changes (including any learnings changes).

**Step 6 — Report** before/after metrics, list every file modified, and note how many learnings entries were promoted, deleted, or remain.

**Limitations vs. Claude Code's cc-optimize:** No permissions deny/allow, no PostToolUse hooks, no autocompact control, no `@`-import progressive disclosure (so `copilot-learnings.md` is not auto-loaded — this skill is required to apply it), no MCP automation via files.

---

Did this output meet your expectations? If not, describe what was off and Copilot will log the correction to `.github/copilot-learnings.md`.

> **Note:** Corrections are not auto-loaded on every session. Run `copilot-optimize` periodically to review and incorporate them.
