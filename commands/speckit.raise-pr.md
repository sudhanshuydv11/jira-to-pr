---
description: Push the current feature branch and raise (or update) a pull request once quality gates and code review have passed.
argument-hint: "[optional: base branch override]"
---

input: $ARGUMENTS (optional — base branch override; otherwise resolved via base-branch protocol)

protocols:
  - .claude/instructions/protocols/base-branch.md
  - .claude/instructions/protocols/quality-gates.md

preconditions:
  - Quality gates (lint/test) already passed for this branch
  - Code review verdict is PASS, or MEDIUM-only findings remain open (see draft_if below)
  - No CRITICAL/HIGH findings open — refuse to create a PR if any exist

steps:
  - name: Preflight
    checks:
      - No incomplete required tasks in tasks.md
      - Working tree clean
      - Lint/test evidence present in quickstart.md
      - No .env/credentials staged

  - name: Sync
    action: fetch base, rebase current branch if behind base

  - name: Push
    action: git push -u origin <BRANCH>

  - name: Compose PR body
    template: |
      Check .github/pull_request_template.md first. Otherwise use:
      Summary (from spec.md) | Jira | Approach (from plan.md) | Changes (from diff) |
      Quality Gates | Speckit Artifacts | Background Findings | Risk & Rollback | Verification Notes

  - name: Create or update PR
    create: gh pr create --base <base> --head <BRANCH> --title "<TICKET>: <short>" --body-file <tmp>
    draft_if: MEDIUM findings remain open
    already_exists: report URL instead of creating duplicate
    gh_unauth: stop, report "gh auth refresh -h github.com"

output_sections:
  - "Pull Request — URL, base←head, draft status, or why no PR"
  - "Sync — rebase result (clean / commits replayed / conflicts)"
  - "Blockers — what stopped PR creation + fix, or None"

failure_handling:
  review_not_passed: refuse to run, report required verdict
  gh_unauth: branch pushed; report "gh auth refresh -h github.com"
  push_rejected: report conflict, do not force-push; stop and ask before rebasing further

rules:
  - PR only on PASS — CRITICAL/HIGH findings block unconditionally
  - Never force-push; never rewrite history
  - No fabricated evidence in PR body
