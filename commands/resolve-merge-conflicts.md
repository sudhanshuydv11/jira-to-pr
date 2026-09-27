---
description: Merge a source branch into a destination branch and resolve any merge conflicts by reconciling the incoming (source) logic with the destination's current ticket context.
argument-hint: "<source-branch> <destination-branch>"
---

input: $ARGUMENTS → parse as "<source> <destination>". If only one branch is provided, treat
  it as <source> and use the current checked-out branch as <destination>. Error only if
  no arguments are provided at all.

purpose: >
  Merge <source> into <destination> as a single merge commit. On conflict, don't blindly
  pick a side or hand-edit markers — reconcile semantically: keep the incoming (source)
  logic as the substance, but bring it into line with what <destination> is currently
  trying to achieve (its ticket/spec), and never leave duplicated logic behind. Source's
  commit history is preserved (no squash, no rebase).

protocols:
  - .claude/instructions/protocols/quality-gates.md (lint/test before finalizing)

conflict_resolution_model:
  ancestor: git merge-base <source> <destination>  (single common baseline both sides diverged from)
  ours: <destination> — the current/target branch, reflects the current ticket's intent
  theirs (incoming): <source> — treat its logic as the older/base version to preserve and carry forward
  priority_rule: >
    Prefer the incoming (source) code's logic and intent over destination's conflicting
    hunks — but never take it verbatim. Adapt it so it satisfies destination's current
    ticket/requirements, keeping the incoming implementation's behavior wherever the
    ticket doesn't require a change.
  no_duplication_rule: >
    If ancestor→ours and ancestor→theirs changed the same logic in compatible ways
    (e.g. both extracted a helper, both handled the same edge case), consolidate to a
    single implementation. Never leave both versions side by side (duplicate functions,
    duplicate branches, duplicate imports, redundant conditionals).

step_1_preflight:
  - git rev-parse --verify <source> and <destination> — stop if either doesn't exist
  - git status --porcelain on current tree — stop and report if dirty (never stash)
  - git fetch origin (best effort) so refs are current
  - record starting SHA of <destination> (for rollback in the report)

step_2_start_merge:
  - git checkout <destination>
  - git merge --no-commit --no-ff <source>
  - clean_merge: no conflicts → skip to step_5 (still run gates before committing)
  - conflicts: git diff --name-only --diff-filter=U → list of conflicted files

step_3_gather_context (once per merge session):
  ticket_lookup_order:
    1. Search specs/**/spec.md and specs/**/plan.md for the feature tied to <destination>
       (match branch name / ticket key against spec front matter or folder name).
    2. If not found in specs/, derive the ticket key from <destination>'s branch name or
       recent commit messages (e.g. CCM-1234 pattern), then fetch it:
       - use the jira-fe-context-enrichment skill if available
         (.github/skills/jira-fe-context-enrichment/SKILL.md), else
       - fetch directly via the configured Jira API/CLI.
    3. If no ticket can be identified, proceed using only what's inferable from
       <destination>'s commit history and code — mark ticket context as
       "not found, inferred from commits" in the report. Do not fabricate requirements.
  ancestor_extraction (per conflicted file):
    - git show <merge-base-sha>:<path>   (ancestor version)
    - git show :2:<path>                  (ours — destination-side, from the index)
    - git show :3:<path>                  (theirs — incoming source-side, from the index)

step_4_resolve_each_file:
  - Diff ancestor→ours and ancestor→theirs separately to see what each side actually
    changed (not just the conflict markers, which only show the overlapping hunk).
  - Rewrite the conflicted region starting from theirs (incoming) as the base logic.
  - Apply the minimum changes needed to satisfy ours' intent (the current ticket):
      - only touch what the ticket/spec requires, or what ours changed for a reason
        that still applies (e.g. renamed API, new required prop, bug fix)
      - preserve incoming's behavior/logic where the ticket is silent on it
  - If both sides implement equivalent logic, consolidate to a single implementation
    (prefer the version that matches destination's current naming/patterns), remove
    the now-redundant one, and update call sites.
  - Remove conflict markers (<<<<<<<, =======, >>>>>>>) — never commit them.
  - Re-check imports/exports touched by the resolution for now-unused or duplicate entries.
  - git add <resolved file>

step_5_validate:
  - git diff --check for leftover conflict markers as a hard gate before committing.
  - npm run lint and npm test (quality-gates protocol) — max 3 remediation attempts on
    resolution-introduced failures, then stop and report unresolved files.

step_6_finalize:
  - git commit (default message: "Merge <source> into <destination>, resolve conflicts"
    + one line per file noting what was reconciled)
  - Do not push unless explicitly asked.

output_sections:
  - "Merge — source, destination, merge-base SHA, destination starting SHA (for rollback)"
  - "Conflicts — file list, or 'none'"
  - "Ticket context — key + source (specs/ file, fetched ticket, or 'inferred from commits')"
  - "Resolutions — per file: what incoming logic was kept, what changed and why, any
    duplication removed"
  - "Gates — lint/test results"
  - "Next steps — commit created (SHA) or blocked, with reason"
  - "Rollback — exact command using the recorded starting SHA"

failure_handling:
  dirty_tree: stop and report (never stash)
  missing_branch: report and exit
  unresolvable_semantic_conflict: git merge --abort, report file with both original hunks
    quoted; do not guess business intent
  gates_fail_3x: keep merge uncommitted, report failing files and errors
  user_wants_to_abort_midway: git merge --abort restores destination to starting SHA
