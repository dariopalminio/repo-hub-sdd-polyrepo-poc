---
name: speckit-multi-repo-sync-sync
description: Create the current feature branch in the affected sub-repositories recorded
  in plan.md (idempotent; supports dry-run)
compatibility: Requires spec-kit project structure with .specify/ directory
metadata:
  author: github-spec-kit
  source: extension:multi-repo-sync
---

# Multi Repo Sync Sync Skill

# Multi-Repo Branch Sync — Sync

Create the root repository's **current feature branch** in the sub-repositories
and git submodules that the feature **affects**, so the developer does not have
to create the matching branch by hand in each component.

"Affected" repositories are the ones recorded in the **Affected Repositories**
table that the companion `speckit.multi-repo-sync.analyze` command writes into
`plan.md` during the `after_plan` hook. This command reads that table and creates
the branch only in those repositories — it does not branch the whole tree.

This command **only** manages branches in sub-repositories. It never modifies
Spec Kit core files and never changes the root repository's branch. It is
**idempotent**: sub-repositories that already have the branch are skipped.

## When this runs

It runs automatically via the `after_tasks` hook declared in this extension's
`extension.yml`, after `analyze` has recorded the affected repositories during
`after_plan`. It can also be invoked manually at any time.

## User Input

Optional flags may be passed by the user after the command:

- `--dry-run` — report what would happen without creating any branch.
- `--no-switch` — create the branch but do not check it out in each sub-repo
  (by default the branch is also switched to, mirroring `git checkout -b`, on a
  clean working tree).
- `--branch <name>` — use a specific branch name instead of the root's current.
- `--verbose` / `--quiet` — adjust logging.

## Execution

1. **Resolve the affected repositories.** Locate the current feature's `plan.md`
   (the one for the active branch under `specs/`). Read its **Affected
   Repositories** section — a Markdown table with `Repo Path`, `Type`, and
   `Reason` columns. Collect the values of the `Repo Path` column into a
   comma-separated list (no spaces), e.g. `services/api,libs/shared`.
   - If the table's only row is the `_none_` sentinel (the `Repo Path` cell is
     `_none_` or `—`, which `analyze` writes when no sub-repository is affected),
     there is nothing to do. Do **not** run the script with that sentinel as a
     target — it is not a real path and would be reported as a failure. Instead,
     report that the feature touches no sub-repository and stop.
   - If `plan.md` has **no** Affected Repositories section (for example the
     `analyze` hook did not run), fall back to scanning the whole tree: run the
     script **without** `--targets` and note in your report that no affected list
     was found so every detected sub-repository was considered.

2. **Run the platform script** from the installed extension directory and rely on
   its JSON output — do not re-implement detection or branch logic yourself.
   Pass the affected list via `--targets` (omit it for the fallback above).

   - **Bash / macOS / Linux:**

     ```bash
     .specify/extensions/multi-repo-sync/scripts/bash/sync-feature-branches.sh --json --targets "<paths>" {$ARGUMENTS}
     ```

   - **PowerShell / Windows:**

     ```powershell
     .specify/extensions/multi-repo-sync/scripts/powershell/sync-feature-branches.ps1 -Json -Targets "<paths>" {$ARGUMENTS}
     ```

   (Translate `{$ARGUMENTS}` to the platform's flag style: `--dry-run`→`-DryRun`,
   `--no-switch`→`-NoSwitch`, `--branch X`→`-Branch X`.)

## Reporting

Parse the JSON the script prints on stdout and report back concisely:

1. The branch being created and the affected repositories it was applied to.
2. A short table of each target's `path` and `status`
   (`created` / `exists` / `would_create` / `failed`). A `created` target was
   also switched onto the branch unless its working tree was dirty.
3. If `summary.failed > 0`, list the failed targets with their `message` and
   note that the failures were isolated — **the native plan/tasks run was not
   affected**. Do not retry automatically; surface the cause (dirty tree,
   missing remote, uninitialized submodule, unborn HEAD) so the user can act.

Notes:
- A script exit code of `1` means at least one sub-repository failed; this is
  informational and must not be treated as a failure of the planning/tasks step.
- If there are no affected repositories and none are detected
  (`reason: no_targets`), report that the project appears to be single-repo, or
  that the feature does not touch any sub-repository, and that there is nothing
  to do.
