---
description: Prepare FE-only speckit.plan-ready planning input from a Jira ticket by consuming the generic Jira context extraction skill output.
argument-hint: "<JIRA-KEY or Jira URL>"
---

input: $ARGUMENTS (required)

purpose: Produce a /speckit.plan input block from a Jira ticket, keeping scope FE-only.

required_skill: >
  jira-fe-context-enrichment (.github/skills/jira-fe-context-enrichment/SKILL.md) —
  source of truth for Jira retrieval, normalization, FE/BE/SHARED classification,
  Confluence enrichment, M3 FE UI library extraction, FE Context Package generation.

workflow:
  - Parse Jira key (accept CCM-1234 or full URL). Error if undetermined.
  - Run skill with target "plan"
  - If "NO FE SCOPE" returned, pass through unchanged
  - Convert FE context into planning-oriented language:
      be_items: as dependencies/blockers, not tasks
      highlight: architecture/contract impact and unknowns
      prioritize: M3 FE UI library component guidance (props, constraints)
      incorporate: Confluence-derived constraints for planning/sequencing

output_structure:
  jira_snapshot:
    - Issue Key, Title, Status, Priority, Summary
    - Description Highlights, Acceptance Criteria
    - Comments Used, External Refs, Confluence Insights
    - M3 FE Component Insights
    - Scope Classification (FE/BE/SHARED)
    - Non-FE Context (kept out), Gaps/Unknowns
  refined_input:
    - Planning objective, FE workstreams
    - Architecture/contract impact
    - Technical constraints, Confluence-derived constraints
    - Component + props usage
    - Dependencies/blockers, Risks
    - Unknowns requiring research
    - Suggested research/data-model/contract inputs

quality_bar:
  - No hallucination — mark uncertain as assumptions
  - FE scope explicit and implementation-ready
  - BE scope as dependency context only
