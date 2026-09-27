---
description: Execute the implementation plan by processing and executing all tasks defined in tasks.md
argument-hint: "[optional context]"
---

input: $ARGUMENTS (consider if not empty)

protocols:
  - .claude/instructions/protocols/extension-hooks.md (before_implement, after_implement)
  - .claude/instructions/protocols/prerequisites.md

pre_execution: Run extension hooks for phase before_implement

steps:
  - name: Setup
    action: Run .specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks
    parse: FEATURE_DIR, AVAILABLE_DOCS

  - name: Check Checklists
    when: FEATURE_DIR/checklists/ exists
    action: count total/completed/incomplete items, display status table
    on_incomplete: log warning but auto-proceed

  - name: Load Context
    required: [tasks.md, plan.md, .claude/context/libraries-api.md]
    if_exists: [data-model.md, contracts/, research.md, quickstart.md]

  - name: Project Setup Verification
    action: create/verify ignore files (.gitignore, .eslintignore, etc.) based on detected setup
    rule: only add missing critical patterns to existing files

  - name: Parse tasks.md
    extract: [phases, dependencies, task IDs, file paths, parallel markers [P]]

  - name: Execute Phase-by-Phase
    rules:
      - Complete each phase before next
      - Respect dependencies; parallel [P] tasks can run together
      - TDD: test tasks before implementation tasks
      - Same-file tasks run sequentially
      - Mark completed tasks as [X] in tasks.md

  - name: Implementation Order
    sequence: Setup → Tests → Core → Integration → Polish
    on_failure:
      non_parallel: halt execution
      parallel: continue others, report failed
    progress: report after each completed task

  - name: Completion Validation
    checks:
      - All required tasks completed
      - Features match spec
      - Tests pass, coverage meets requirements
      - Implementation follows plan

post_execution: Run extension hooks for phase after_implement

note: If tasks.md incomplete/missing, suggest running /speckit.tasks first.
