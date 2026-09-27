---
description: Perform a non-destructive cross-artifact consistency and quality analysis across spec.md, plan.md, and tasks.md after task generation.
argument-hint: "[optional context]"
---

input: $ARGUMENTS (consider if not empty)

protocols:
  - .claude/instructions/protocols/prerequisites.md

purpose: >
  Identify inconsistencies, duplications, ambiguities, and underspecified items across
  spec.md, plan.md, tasks.md before implementation. STRICTLY READ-ONLY — no file edits.

steps:
  - name: Initialize
    command: .specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks
    parse: FEATURE_DIR, derive SPEC/PLAN/TASKS paths
    on_missing: abort, instruct to run prerequisite command

  - name: Load Artifacts (minimal context)
    spec.md: Overview, Functional Requirements, Success Criteria, User Stories, Edge Cases
    plan.md: Architecture/stack, Data Model refs, Phases, Technical constraints
    tasks.md: Task IDs, descriptions, phase grouping, [P] markers, file paths
    constitution: .specify/memory/constitution.md — principles and MUST/SHOULD rules

  - name: Build Semantic Models (internal, not output)
    models:
      - Requirements inventory (FR-###, SC-### keys; exclude post-launch KPIs from SC)
      - User story/action inventory with acceptance criteria
      - Task → requirement coverage mapping
      - Constitution rule set

  - name: Detection Passes (max 50 findings)
    A_duplication: near-duplicate requirements
    B_ambiguity: vague adjectives without metrics, unresolved placeholders (TODO, ???)
    C_underspecification: verbs missing objects/outcomes, stories missing acceptance, tasks referencing undefined components
    D_constitution: conflicts with MUST principles, missing mandated gates
    E_coverage_gaps: requirements with zero tasks, tasks with no requirement, buildable SC not in tasks
    F_inconsistency: terminology drift, entities in one file but not another, ordering contradictions, conflicting requirements

  - name: Severity Assignment
    CRITICAL: constitution MUST violation, missing core artifact, zero-coverage requirement blocking baseline
    HIGH: duplicate/conflicting requirement, ambiguous security/perf, untestable acceptance
    MEDIUM: terminology drift, missing NFR coverage, underspecified edge case
    LOW: style/wording, minor redundancy

  - name: Produce Report
    sections:
      - "Findings table — ID, Category, Severity, Location(s), Summary, Recommendation"
      - "Coverage Summary — Requirement Key, Has Task?, Task IDs, Notes"
      - "Constitution Alignment Issues"
      - "Unmapped Tasks"
      - "Metrics — Total Requirements/Tasks, Coverage %, Ambiguity/Duplication/Critical counts"
      - "Next Actions — CRITICAL → resolve before implement; LOW/MEDIUM → may proceed"
      - "Remediation Suggestions — top 5 issues with concrete fixes (auto-included)"

rules:
  - NEVER modify files
  - NEVER hallucinate missing sections
  - Constitution violations always CRITICAL
  - Deterministic: rerun without changes → consistent IDs and counts
  - Auto-include remediation for top 5 — never prompt user
