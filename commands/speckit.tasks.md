---
description: Generate an actionable, dependency-ordered tasks.md for the feature based on available design artifacts.
argument-hint: "[optional context, e.g. 'base branch: develop']"
---

input: $ARGUMENTS (consider if not empty)

protocols:
  - .claude/instructions/protocols/extension-hooks.md (before_tasks, after_tasks)
  - .claude/instructions/protocols/base-branch.md (for changed-lines coverage)
  - .claude/instructions/protocols/prerequisites.md

pre_execution: Run extension hooks for phase before_tasks

steps:
  - name: Setup
    command: .specify/scripts/bash/check-prerequisites.sh --json
    parse: FEATURE_DIR, AVAILABLE_DOCS

  - name: Resolve Base Branch
    per_protocol: used for changed-lines coverage gate tasks

  - name: Load Design Docs
    required: [plan.md, spec.md, .claude/context/libraries-api.md]
    if_exists: [data-model.md, contracts/, research.md, quickstart.md]

  - name: Generate Tasks
    from: tech stack/libraries (plan), user stories with priorities (spec), entities, contracts, research
    organize_by: user story (primary structure)
    template: .specify/templates/tasks-template.md

  - name: Write tasks.md
    phases:
      - "Phase 1: Setup (project init)"
      - "Phase 2: Foundational (blocking prerequisites — MUST complete before stories)"
      - "Phase 3+: One per user story in priority order"
      - "Final: Polish & Cross-Cutting"
    within_story_order: Tests → Types/State/Config → API/Queries → UI → Integration → Quality Gate

  - name: Report
    include: path, total count, per-story count, parallel opportunities, base branch, suggested MVP

post_execution: Run extension hooks for phase after_tasks

task_format: "- [ ] [TaskID] [P?] [Story?] Description with file path"
  components:
    checkbox: always "- [ ]"
    task_id: T001, T002… (sequential)
    parallel: "[P]" only if parallelizable (different files, no deps on incomplete tasks)
    story: "[US1]", "[US2]"… REQUIRED for story phases, NOT for Setup/Foundational/Polish
    description: clear action with exact file path
  examples:
    - "- [ ] T001 Create project structure per implementation plan"
    - "- [ ] T012 [P] [US1] Create User model in src/models/user.ts"

quality_gate_per_story:
  lint: npm run lint
  coverage: >
    git fetch --no-tags --prune origin <base> &&
    MB=$(git merge-base HEAD origin/<base>) &&
    npm test -- --coverage.changed=$MB

rules:
  - Tasks organized by user story for independent implementation/testing
  - TDD: test tasks before implementation tasks
  - Each task must be immediately executable by an LLM without extra context
