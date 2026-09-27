---
description: Create or update the project constitution from interactive or provided principle inputs, ensuring all dependent templates stay in sync.
argument-hint: "[optional principle updates]"
---

input: $ARGUMENTS (consider if not empty)

purpose: >
  Update project constitution at .specify/memory/constitution.md. File is a template
  with [PLACEHOLDER] tokens. Collect/derive values, fill template, propagate amendments.

steps:
  - name: Load
    file: .specify/memory/constitution.md
    action: identify all [ALL_CAPS_IDENTIFIER] placeholder tokens
    note: respect user-specified principle count (may differ from template)

  - name: Collect Values
    priority: user input first, then infer from repo context (README, docs, prior versions)
    dates:
      RATIFICATION_DATE: original adoption (ask if unknown)
      LAST_AMENDED_DATE: today if changes made
    version: >
      CONSTITUTION_VERSION — semver increment:
      MAJOR (incompatible removals/redefinitions),
      MINOR (new principle/expanded guidance),
      PATCH (clarifications/typos)

  - name: Draft Updated Constitution
    actions:
      - Replace every placeholder with concrete text (no brackets left)
      - Each Principle: name, non-negotiable rules, rationale
      - Governance: amendment procedure, versioning policy, compliance review

  - name: Consistency Propagation
    verify_alignment:
      - .specify/templates/plan-template.md (Constitution Check section)
      - .specify/templates/spec-template.md (scope/requirements)
      - .specify/templates/tasks-template.md (task categorization)
      - .specify/templates/commands/*.md (no outdated refs)
      - Runtime docs (README, quickstart, agent files)

  - name: Sync Impact Report (HTML comment at top of file)
    include:
      - "Version: old → new"
      - Modified/added/removed principles
      - "Templates requiring updates (✅ updated / ⚠ pending)"
      - Deferred TODOs

  - name: Validate
    checks: no unexplained brackets, version matches report, dates ISO, principles declarative+testable

  - name: Write
    target: .specify/memory/constitution.md (overwrite)

  - name: Report
    include: new version + bump rationale, files needing follow-up, suggested commit message

style:
  - Preserve heading hierarchy from template
  - <100 chars per line, single blank line between sections, no trailing whitespace
