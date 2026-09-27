---
description: Identify underspecified areas in the current feature spec by asking up to 5 highly targeted clarification questions and encoding answers back into the spec.
argument-hint: "[optional context]"
---

input: $ARGUMENTS (consider if not empty)

protocols:
  - .claude/instructions/protocols/prerequisites.md

purpose: Detect and reduce ambiguity in active feature spec. Expected to run BEFORE /speckit.plan.

steps:
  - name: Setup
    command: .specify/scripts/bash/check-prerequisites.sh --json --paths-only
    parse: FEATURE_DIR, FEATURE_SPEC
    on_fail: abort, instruct to re-run /speckit.specify

  - name: Ambiguity Scan
    load: spec file
    categories_to_assess (mark Clear/Partial/Missing):
      - Functional Scope & Behavior (goals, out-of-scope, roles)
      - Domain & Data Model (entities, identity, state, scale)
      - Interaction & UX Flow (journeys, error/empty/loading, a11y)
      - Non-Functional (perf, scalability, reliability, observability, security, compliance)
      - Integration & External Dependencies (APIs, formats, protocols)
      - Edge Cases & Failure Handling (negatives, rate limiting, conflicts)
      - Constraints & Tradeoffs
      - Terminology & Consistency
      - Completion Signals (testability, DoD)
      - Placeholders (TODOs, vague adjectives)

  - name: Generate Questions (max 5, internal queue)
    criteria:
      - Answerable with multiple-choice (2-5 options) or short-phrase (≤5 words)
      - Must materially impact architecture, data, tasks, tests, UX, ops, or compliance
      - Balance across highest-impact categories
      - Prioritize by Impact × Uncertainty

  - name: Auto-Resolve All (non-interactive)
    multiple_choice: analyze options, select best by best-practices/risk-reduction/context
    short_answer: determine best answer from context
    log: "Auto-selected: Option [X] - <reasoning>" or "Auto-resolved: <answer> - <reasoning>"
    stop_when: all critical ambiguities resolved OR 5 questions done

  - name: Integrate Into Spec (after each answer)
    ensure: "## Clarifications" > "### Session YYYY-MM-DD" section exists
    append: "- Q: <question> → A: <answer>"
    apply_to: appropriate section (functional→requirements, data→model, NFR→success criteria, etc.)
    rule: replace invalidated statements, no contradictions
    save: after each integration

  - name: Validate
    checks: one bullet per answer, total ≤5, no lingering placeholders, no contradictions, valid markdown, consistent terminology

  - name: Write
    target: FEATURE_SPEC

  - name: Report
    include: questions asked/answered, path, sections touched, coverage summary (Resolved/Deferred/Clear/Outstanding), next command

rules:
  - Never prompt the user — always auto-select best-practice
  - No ambiguities found: "No critical ambiguities detected." → suggest proceeding
  - Spec missing: instruct to run /speckit.specify first
  - Max 5 questions total, never exceed
  - Quota exhausted with high-impact remaining: flag under Deferred

context_for_prioritization: $ARGUMENTS
