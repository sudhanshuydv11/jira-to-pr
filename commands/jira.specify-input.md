---
description: Prepare FE-only speckit.specify-ready feature input from a Jira ticket by consuming the generic Jira context extraction skill output.
argument-hint: "<JIRA-KEY or Jira URL>"
---

input: $ARGUMENTS (required)

purpose: Produce a /speckit.specify input block from a Jira ticket, keeping scope FE-only.

required_skill: >
  jira-fe-context-enrichment (.github/skills/jira-fe-context-enrichment/SKILL.md) —
  source of truth for Jira retrieval, normalization, FE/BE/SHARED classification,
  Confluence enrichment, M3 FE UI library extraction, FE Context Package generation.

workflow:
  - Parse Jira key (accept CCM-1234 or full URL). Error if undetermined.
  - Run skill with target "specify"
  - If "NO FE SCOPE" returned, pass through unchanged
  - Refine into specification-ready business language:
      focus: WHAT/WHY, not implementation
      include: Confluence-derived requirement facts
      include: UI-library constraints only if affecting scope/acceptance/behavior
      assumptions: explicit
      clarifications: up to 3 NEEDS CLARIFICATION items

output_structure:
  jira_snapshot:
    - Issue Key, Title, Status, Priority, Summary
    - Description Highlights, Acceptance Criteria
    - Comments Used, External Refs, Confluence Insights
    - UI Library Constraints (scope-impacting only)
    - Scope Classification (FE/BE/SHARED)
    - Non-FE Context (kept out), Gaps/Unknowns
  refined_input:
    - Feature, User/Role, Problem, Goal
    - In scope, Out of scope
    - Acceptance criteria
    - Enriched functional requirements
    - UI library constraints affecting scope/acceptance
    - Constraints, Dependencies
    - Performance expectations, Test expectations
    - Open questions

quality_bar:
  - No hallucination — mark uncertain as assumptions
  - Testable, unambiguous acceptance criteria
  - Concise bullets over prose
  - Exclude BE tasks from refined input
