---
description: Fetch all tickets assigned to me in the current sprint, determine which are independent, then run the full Speckit pipeline on them in parallel where safe and sequentially where not.
argument-hint: "[optional: 'dry-run', 'only CCM-1234,CCM-5678', 'max-parallel N', 'sprint <id>']"
---

input: $ARGUMENTS (consider if not empty)

protocols:
  - .claude/instructions/protocols/base-branch.md (resolve once, reuse for all tickets)
  - .claude/instructions/protocols/worktree-isolation.md
  - .claude/instructions/protocols/quality-gates.md

context:
  jira_project: CCM
  board_id: "69061"
  board_name: Campaign Portal Sprints-2026
  board_type: scrum
  sprint_source: active sprint on that board

rules:
  - Print full execution plan and require confirmation before starting (skip with --yes)
  - Never run two pipelines in same working tree
  - Default to sequential when independence uncertain
  - max-parallel default 3
  - One PR per ticket, never combine
  - Never reorder or skip silently — report every exclusion with reason
  - One ticket failing does not abort unrelated tickets; dependents skipped

steps:
  - name: Resolve Sprint
    action: jira_get_sprints_from_board
    params: { board_id: "69061", state: active }
    note: If multiple active sprints, ask which. Skip if user passed "sprint <id>".

  - name: Fetch Tickets
    jql: >
      assignee = currentUser() AND sprint in openSprints()
      AND statusCategory != Done ORDER BY priority DESC, key ASC
    fields: [summary, status, assignee, priority, issuelinks, parent, labels, description]
    filters: honour "only CCM-..." as allowlist
    report: list every excluded ticket with reason

  - name: Independence Analysis
    dependent_if_any:
      - "Jira links: blocks / is blocked by / depends on between the two"
      - Parent/subtask relationship
      - "Predicted file overlap — high-collision paths that auto-trigger dependency:"
      - "  package.json, package-lock.json, webpack.config.js, webpack.publish.config.js"
      - "  .babelrc, jest.config.js, src/components/Calendar.js"
      - "Semantic ordering: dep upgrade before refactors using new API, etc."
    changelog_caveat: >
      Every ticket edits CHANGELOG.md and bumps package.json version — exclude from
      overlap test but flag that PRs need version-bump rebase before merge.
    requirement: Record conclusions with evidence. Never assert independence without basis.

  - name: Schedule
    actions:
      - Build directed dependency graph
      - Detect cycles → run cycled tickets sequentially by priority
      - Partition into waves by topological level (wave N deps all in waves < N)
      - Within wave: up to max-parallel concurrent, rest queued
      - Waves run strictly in order

  - name: Print Plan & Confirm
    format: |
      Sprint: <name> (<start> → <end>)
      Base:   <resolved base branch>
      Wave 1 (parallel, 3 tickets): CCM-A, CCM-B, CCM-C
        └ independent: no Jira links, no predicted file overlap
      Wave 2 (sequential, after CCM-A): CCM-D
        └ depends on CCM-A: both modify webpack.config.js
      Excluded: CCM-E (status Closed), CCM-F (not in allowlist)
    stop_if: dry-run

  - name: Execute
    per_ticket:
      - Create worktree and provision (per worktree-isolation protocol)
      - Run /speckit.run-full-pr <ticket> inside worktree
      - Record: status, PR URL, gate results, worktree path
      - Skip dependents of failed as "skipped (blocked by <ticket>)"
      - Clean successful worktrees; retain failed for inspection

output_sections:
  - "Sprint — name, dates, board, ticket counts (fetched/selected/excluded)"
  - "Dependency Analysis — table of pairs with verdict + evidence"
  - "Execution Plan — waves with parallel/sequential rationale"
  - "Results — per ticket: status, PR URL, lint/test/review verdicts"
  - "Skipped & Failed — reason + resume command for each"
  - "Retained Worktrees — paths + cleanup commands, or None"
  - "Next Steps — PRs awaiting review, version-bump rebases, DEV deploy"

failure_handling:
  jira_unreachable: Stop immediately (VPN required)
  no_active_sprint_or_tickets: Report and stop
  worktree_creation_fails: Downgrade ticket to sequential
  pipeline_fails: Retain worktree, report failing step, continue others
  dirty_main_tree: Stop and report (never stash/discard user work)
