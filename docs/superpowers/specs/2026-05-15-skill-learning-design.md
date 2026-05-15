# Skill Learning System — Design Spec

**Date:** 2026-05-15
**Scope:** `gc-config-init` and `gc-config-optimize` SKILL.md files

## Problem

The existing learning mechanism (`copilot-learnings.md`) only captures entries when the user explicitly corrects Copilot. In practice this never triggers, so the file never appears and learnings never accumulate. The end-of-run feedback question is also routinely skipped.

## Solution: Observe-Notice-Store-Recall (Option B)

A three-step loop baked into both skills as silent bookends around the existing flow.

### Step 0 — Recall (new, before Step 1)

At the start of each skill run:

1. Check if `.github/copilot-learnings.md` exists.
2. If yes, read all entries and apply them silently — they inform decisions during the run with no user-visible output and no announcement. The `[skill-name]` tag on each entry is provenance only, not a filter; entries from either skill apply to both.
3. If the file does not exist, proceed normally with no mention of learnings.

### Existing steps — unchanged

All existing skill steps remain intact and unchanged in sequence.

### Final step — Store (new, after last substantive step, before feedback question)

After the skill's final substantive step:

1. Review the run against the notice criteria below.
2. If one or more entries qualify, append them to `.github/copilot-learnings.md` (create the file if it doesn't exist).
3. If nothing qualifies, do nothing — no file created, no user notification.
4. Skip any entry that duplicates something already in the file.
5. The existing feedback question follows as normal.

## Notice Criteria

An entry is written if and only if one of these conditions is true:

| Condition                                                                            | Example                                                                      |
| ------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------- |
| **Project deviation** — repo differs from skill's generic assumptions                | `monorepo: applyTo globs should cover packages/*/`                           |
| **User choice** — user accepted or rejected a suggestion that deviates from defaults | `user prefers minimal hooks — don't suggest additions unless clearly absent` |
| **Discovery** — a constraint found that changes future behavior                      | `project has existing pre-commit hook — don't overwrite`                     |

What does **not** qualify:

- Standard skill behavior applied without deviation
- Facts already present in AGENTS.md, CLAUDE.md, or other config files
- Anything obvious from reading the repo

## Entry Format

One line per entry, tagged by originating skill:

```
[gc-config-init] monorepo: applyTo globs should cover packages/*/
[gc-config-optimize] user prefers minimal hooks — don't suggest additions unless clearly absent
```

## Interaction with gc-config-optimize

`gc-config-optimize` already has a Learnings review step that promotes entries to config and removes processed ones. This step is unchanged — it now receives entries automatically rather than depending on manual user corrections.

Both skills participate in capture (Store) and consumption (Recall), so learnings written by `gc-config-init` are available to `gc-config-optimize` on the next run and vice versa.

## What changes in each SKILL.md

| File                          | Change                                                                                 |
| ----------------------------- | -------------------------------------------------------------------------------------- |
| `gc-config-init/SKILL.md`     | Add Recall before Step 1; add Store after final step                                   |
| `gc-config-optimize/SKILL.md` | Add Recall before Step 1; add Store after final step (Learnings review step unchanged) |

## Out of scope

- Structured entry categories (`[preference]`, `[avoid]`, etc.) — over-engineering at this stage
- Any user-visible recall announcements
- Cross-project learning (each repo maintains its own file)
