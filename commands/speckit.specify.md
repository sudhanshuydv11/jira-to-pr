---
description: Create or update the feature specification from a natural language feature description.
argument-hint: "<feature description>"
---

input: $ARGUMENTS (required — error if empty)

protocols:
  - .claude/instructions/protocols/extension-hooks.md (before_specify, after_specify)

pre_execution: Run extension hooks for phase before_specify

steps:
  - name: Generate Short Name
    format: 2-4 words, kebab-case, action-noun (e.g. "add-user-auth", "fix-payment-timeout")
    rule: preserve technical terms/acronyms

  - name: Create Feature Branch
    check: .specify/init-options.json for branch_numbering ("timestamp" → add --timestamp)
    command: .specify/scripts/bash/create-new-feature.sh "$ARGUMENTS" --json --short-name "<slug>" "<description>"
    parse: BRANCH_NAME, SPEC_FILE from JSON output
    rule: run only once, never pass --number

  - name: Load Template
    file: .specify/templates/spec-template.md

  - name: Generate Spec
    from: feature description (actors, actions, data, constraints)
    unclear_aspects: make informed guesses, max 3 [NEEDS CLARIFICATION] markers
    clarification_criteria: only for decisions significantly impacting scope/UX with no reasonable default
    sections: User Scenarios, Functional Requirements (testable), Success Criteria (measurable, tech-agnostic), Key Entities
    write_to: SPEC_FILE

  - name: Quality Validation
    create: FEATURE_DIR/checklists/requirements.md (content quality, requirement completeness, feature readiness)
    validate: spec against checklist
    on_fail: fix and re-validate (max 3 iterations)
    needs_clarification: auto-resolve each — analyze options, select best-practice, log "Auto-resolved Q[N]: [answer] — Reason: [rationale]"

  - name: Report
    include: branch name, spec path, checklist results, readiness for /speckit.clarify or /speckit.plan

post_execution: Run extension hooks for phase after_specify

guidelines:
  - Focus on WHAT and WHY — avoid HOW (no tech stack, APIs, code structure)
  - Written for business stakeholders, not developers
  - Mandatory sections completed; optional sections removed if N/A
  - Success criteria: measurable, technology-agnostic, user-focused, verifiable
  - Never prompt the user — auto-select recommended approach
