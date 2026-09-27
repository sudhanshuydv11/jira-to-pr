---
description: Run the full Speckit delivery pipeline end to end (specify through code review) and raise a pull request automatically when the review gate passes.
argument-hint: "<feature description from Jira, or ticket ID>"
---

input: $ARGUMENTS (required — stop and ask if empty)

protocols:
  - .claude/instructions/protocols/subagent-model.md
  - .claude/instructions/protocols/resume-detection.md
  - .claude/instructions/protocols/base-branch.md
  - .claude/instructions/protocols/quality-gates.md
  - .claude/instructions/protocols/extension-hooks.md

rules:
  - PR only on PASS — CRITICAL/HIGH findings block unconditionally
  - Max 3 remediation attempts per gate, max 3 review cycles
  - No silent scope growth — build only what tasks.md defines
  - No skipped gates. No fabricated evidence. No history rewriting (never force-push).
  - No user prompts — always auto-select best-practice approach
  - Clarify + analyze always run as parallel subagents after tasks

steps:
  - name: Intake
    command: /jira.specify-input <TICKET>
    when: input is a ticket ID (skip for descriptions)

  - name: Specify
    command: /speckit.specify
    produces: feature branch + spec.md

  - name: Plan
    command: /speckit.plan
    produces: plan.md, research.md, data-model.md, contracts/

  - name: Tasks
    command: /speckit.tasks
    produces: tasks.md (dependency-ordered)

  - name: Clarify + Analyze
    parallel: true
    commands: [/speckit.clarify, /speckit.analyze]
    blocking: CRITICAL findings from analyze
    clarify_mode: auto-resolves all questions (no prompts)

  - name: Implement
    command: /speckit.implement
    note: only step that writes source code

  - name: Quality Gates
    run: [npm run lint, npm test]
    on_fail: fresh implement subagent with failure output
    retries: 3

  - name: Code Review
    command: /code.review
    on_fail: loop to Implement
    max_cycles: 3

  - name: Commit Artifacts
    action: commit spec/plan/tasks/evidence to feature branch

  - name: Raise PR
    command: /speckit.raise-pr
    condition: review PASS
    note: preflight, sync, PR body, gh create, draft/duplicate/auth handling all defined in speckit.raise-pr

output_sections:
  - "Pipeline Status — steps 1-10 with status, artifacts, duration"
  - "Resume Point — where run started and why"
  - "Quality Gates — lint/test/review verdicts with evidence"
  - "Background Findings — clarify + analyze results, or None"
  - "Pull Request — URL, base←head, draft status, or why no PR"
  - "Blockers — what stopped pipeline + fix, or None"
  - "Next Steps — DEV deploy/testing, or remediation needed"

failure_handling:
  step_fails: report step, error, artifacts; state resume command
  review_fails_3x: no PR; report persistent findings
  raise_pr_fails: see /speckit.raise-pr failure_handling (gh_unauth, push_rejected, review_not_passed)
