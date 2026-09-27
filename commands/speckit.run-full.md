---
description: Run the full Speckit delivery pipeline (specify through code review) as sequential subagents with resume detection, stopping before PR creation.
argument-hint: "<feature description from Jira, or ticket ID>"
---

input: $ARGUMENTS (required — stop and ask if empty)

protocols:
  - .claude/instructions/protocols/subagent-model.md
  - .claude/instructions/protocols/resume-detection.md
  - .claude/instructions/protocols/quality-gates.md
  - .claude/instructions/protocols/extension-hooks.md

rules:
  - Max 3 remediation attempts per gate, max 3 review cycles — report and stop
  - No silent scope growth — implement only what tasks.md defines
  - No skipped gates. No fabricated evidence. No user prompts.
  - Clarify + analyze always parallel after tasks
  - This command never creates a PR — that is /speckit.run-full-pr

steps:
  - name: Intake
    command: /jira.specify-input <TICKET>
    when: input is a ticket ID

  - name: Specify
    command: /speckit.specify
    produces: feature branch + spec.md

  - name: Plan
    command: /speckit.plan
    produces: plan.md + optional artifacts

  - name: Tasks
    command: /speckit.tasks
    produces: tasks.md

  - name: Clarify + Analyze
    parallel: true
    commands: [/speckit.clarify, /speckit.analyze]
    blocking: CRITICAL findings from analyze
    clarify_mode: auto-resolves (no prompts)

  - name: Implement
    command: /speckit.implement
    note: only step writing source code

  - name: Quality Gates
    run: [npm run lint, npm test]
    on_fail: fresh implement subagent with failure output
    retries: 3

  - name: Code Review
    command: /code.review
    on_pass: pipeline complete
    on_fail: loop to Implement (max 3 cycles)

  - name: Commit Artifacts
    action: commit spec/plan/tasks/evidence to feature branch

output_sections:
  - "Pipeline Status — steps with status, artifacts, duration"
  - "Resume Point — where run started and why"
  - "Quality Gates — lint/test/review verdicts with evidence"
  - "Background Findings — clarify + analyze results, or None"
  - "Blockers — what stopped pipeline + fix, or None"
  - "Next Steps — typically /speckit.run-full-pr or remediation needed"

failure_handling:
  step_fails: report step, error, artifacts; state resume command
  interrupted: resume detection makes re-invocation safe and cheap
