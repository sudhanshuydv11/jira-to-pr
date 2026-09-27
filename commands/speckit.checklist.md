---
description: Generate a custom checklist for the current feature based on user requirements.
argument-hint: "<domain, e.g. ux, security, perf>"
---

input: $ARGUMENTS (required)

protocols:
  - .claude/instructions/protocols/prerequisites.md

core_concept: >
  Checklists are "Unit Tests for English" — they validate REQUIREMENTS QUALITY,
  not implementation behavior. Test what's WRITTEN (or not written) in the spec.

steps:
  - name: Setup
    command: .specify/scripts/bash/check-prerequisites.sh --json
    parse: FEATURE_DIR, AVAILABLE_DOCS

  - name: Clarify Intent (up to 3 dynamic questions)
    source: user input + spec signals
    criteria: only ask what materially changes checklist content
    archetypes: [scope refinement, risk prioritization, depth calibration, audience framing, boundary exclusion, scenario gap]
    follow_ups: up to 2 more (max 5 total) if major scenario classes unclear

  - name: Load Feature Context
    files: [spec.md, plan.md (if exists), tasks.md (if exists)]
    strategy: progressive disclosure, load only relevant portions

  - name: Generate Checklist
    location: FEATURE_DIR/checklists/<domain>.md
    if_exists: append from last CHK ID (never delete existing)
    if_new: start from CHK001
    group_by_dimensions:
      - Requirement Completeness
      - Requirement Clarity
      - Requirement Consistency
      - Acceptance Criteria Quality
      - Scenario Coverage
      - Edge Case Coverage
      - Non-Functional Requirements
      - Dependencies & Assumptions
      - Ambiguities & Conflicts
    soft_cap: 40 items (merge near-duplicates, consolidate low-impact edge cases)
    traceability: ≥80% items must include [Spec §X.Y] or [Gap/Ambiguity/Conflict/Assumption]

  - name: Report
    include: path, item count, new vs append, focus areas, depth, actor/timing

item_format: "- [ ] CHK### - <question about requirement quality> [Dimension, Spec §X.Y or Gap]"

required_patterns:
  - "Are [requirements] defined/specified for [scenario]?"
  - "Is [vague term] quantified with specific criteria?"
  - "Are requirements consistent between [A] and [B]?"
  - "Can [requirement] be objectively measured?"
  - "Does the spec define [missing aspect]?"

prohibited_patterns:
  - "Verify", "Test", "Confirm", "Check" + implementation behavior
  - "Displays correctly", "works properly", "functions as expected"
  - "Click", "navigate", "render", "load", "execute"
  - Test cases, test plans, implementation details
