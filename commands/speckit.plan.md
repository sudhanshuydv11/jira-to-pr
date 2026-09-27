---
description: Execute the implementation planning workflow using the plan template to generate design artifacts.
argument-hint: "[optional planning context]"
---

input: $ARGUMENTS (consider if not empty)

protocols:
  - .claude/instructions/protocols/extension-hooks.md (before_plan, after_plan)

pre_execution: Run extension hooks for phase before_plan

steps:
  - name: Setup
    command: .specify/scripts/bash/setup-plan.sh --json
    parse: FEATURE_SPEC, IMPL_PLAN, SPECS_DIR, BRANCH

  - name: Load Context
    files: [FEATURE_SPEC, .specify/memory/constitution.md, .claude/context/libraries-api.md, IMPL_PLAN template]

  - name: Execute Plan Workflow (follow IMPL_PLAN template structure)
    actions:
      - Fill Technical Context (mark unknowns as "NEEDS CLARIFICATION")
      - Fill Constitution Check from constitution
      - Evaluate gates (ERROR if violations unjustified)
      - "Phase 0: Generate research.md (resolve all NEEDS CLARIFICATION)"
      - "Phase 1: Generate data-model.md, contracts/, quickstart.md"
      - "Phase 1: Run .specify/scripts/bash/update-agent-context.sh claude"
      - Re-evaluate Constitution Check post-design

  - name: Report
    include: branch, IMPL_PLAN path, generated artifacts

post_execution: Run extension hooks for phase after_plan

phase_0_research:
  trigger: unknowns from Technical Context
  per_unknown: "Research {unknown} for {feature context}"
  output: research.md with Decision, Rationale, Alternatives considered

phase_1_design:
  prerequisites: research.md complete
  entities: extract from spec → data-model.md (fields, relationships, validations, state transitions)
  contracts: define interface contracts → contracts/ (only if project has external interfaces)
  agent_context: run update script

rules:
  - Use absolute paths
  - No user prompts — auto-resolve all NEEDS CLARIFICATION using best-practice defaults
  - ERROR on gate failures only
  - Reference .claude/context/libraries-api.md for technology decisions
