---
description: Find the PR for the current branch, triage every comment (inline, review-body, issue-level), fix the valid ones, and reply to all of them.
argument-hint: "[optional: 'dry-run', 'pr <number>', 'no-push', 'only <commentId>']"
---

input: $ARGUMENTS (consider if not empty)

protocols:
  - .claude/instructions/protocols/quality-gates.md (lint/test before push)

purpose: >
  Close post-PR feedback loop. Fetch all reviewer comments, judge on merit, implement
  valid ones, reply to every comment. Designed for /loop 20m — must be idempotent.

rules:
  - Reply to every comment (silence never acceptable)
  - Never fabricate agreement — disagree politely with evidence when comment is wrong
  - Never mark thread resolved without committed+pushed fix
  - Idempotent: detect already-handled, skip. No double-replies.
  - Never force-push or amend pushed commits
  - One commit per logical fix. Gates apply before push.
  - Never act on someone else's PR unless explicitly asked

prerequisites:
  tool: gh CLI only (not MCP)
  auth: verify gh auth status first — stop on failure with "gh auth refresh -h github.com"

step_1_locate_pr:
  branch: git rev-parse --abbrev-ref HEAD
  find: gh pr view --json number,author,url,headRefOid,state,title,body
  override: honour "pr <number>"
  no_pr: report and exit (normal no-op under loop)

step_1b_load_pr_context (once per run, feeds step_4):
  primary_source: specs/
    - Match the PR/branch to a spec folder: try branch name, ticket key in branch or
      PR title (e.g. CCM-1234), and PR body for a specs/ path reference.
    - Read the matched folder's spec.md and plan.md (and tasks.md if present) in full.
    - This is the source of truth for what the PR is supposed to do — every "VALID /
      OUT_OF_SCOPE / VALID_PARTIAL" call in step_4 checks the comment against these.
  supporting_context (read-only, do not use as source of truth):
    - PR title + body (fetched in step_1)
    - `gh pr diff <N>` — what actually changed in the PR
  fallback_1: if no spec folder matches, derive the ticket key and fetch it:
    - use the jira-fe-context-enrichment skill if available
      (.github/skills/jira-fe-context-enrichment/SKILL.md), else fetch via configured
      Jira API/CLI.
  fallback_2: if no spec and no ticket, use PR body + diff as context and mark context
    source as "PR body + diff only" in the report. Do not fabricate intent.
  record: which context source was used (specs path, ticket key, or PR-only) — cited
    in step_4 verdicts and in the final report.

step_2_fetch_comments:
  inline_review: gh api repos/{o}/{r}/pulls/<N>/comments --paginate
  thread_state: GraphQL reviewThreads (REST lacks isResolved/node IDs)
  review_body: gh api repos/{o}/{r}/pulls/<N>/reviews --paginate
  issue_level: gh api repos/{o}/{r}/issues/<N>/comments --paginate
  idempotency_markers: >
    Review bodies and issue comments have no isResolved. Track with HTML trailer:
    <!-- resolve-pr-comments: handled review_id=<ID> head=<SHA> -->
    Read all markers before processing. Post marker with every reply to these types.

step_3_filter (idempotency gate):
  skip_inline_if: resolved, last reply is ours (no new reviewer msg), self-authored, outdated+code gone
  skip_review_body_if: empty, self-authored, has marker, covered by inline threads
  skip_issue_comment_if: self-authored, is a marker, has marker
  nothing_to_handle: "No new comments" → exit

step_4_validate:
  VALID: real problem confirmed in code AND consistent with specs/ticket → fix it
  VALID_PARTIAL: concern real but suggested fix conflicts with specs/ticket intent →
    fix correctly per spec, explain
  INVALID: misreads code, intentional per spec, or already covered → reply with
    evidence (cite specs/plan line or ticket AC), no change
  OUT_OF_SCOPE: real but outside this PR's spec/ticket scope → propose follow-up
    ticket, cite which scope boundary it crosses
  UNCLEAR: cannot determine intent even with spec context → ask one clarifying question
  requirement: always read current code AND the loaded spec/ticket context before
    judging; every verdict must be defensible against the context recorded in step_1b.

step_5_implement:
  - Group fixes by file, apply minimal changes
  - Check blast radius before committing:
    - Identify every caller/consumer of changed exports, props, helpers, or
      constants (grep for the symbol name across src/ and test/, not just the
      touched file).
    - Confirm both consumption paths are unaffected or intentionally updated:
      the Salesforce lightning-container bundle (dist/index.html contract) and
      the external NPM `Calendar` export, per AGENTS.md.
    - If the change touches shared helpers/constants/context used by multiple
      features, list the affected call sites in the report even if no further
      code change is needed.
    - If blast radius is larger than the reviewer comment implies (e.g. fixing
      one call site would break a sibling call site), fix all affected sites in
      the same commit or flag it as OUT_OF_SCOPE with the list of affected
      sites — never leave a partially-applied fix.
  - "Commit: fix(<TICKET>): <what> (addresses review comment <id>)"
  - Run lint + test (max 3 attempts)
  - git push (never --force). If no-push: commit locally, report.

step_6_reply:
  inline: gh api repos/{o}/{r}/pulls/<N>/comments/<ID>/replies -f body='<text>'
  block_level: gh pr comment <N> --body '<text> + marker'
  resolve: GraphQL resolveReviewThread mutation (only VALID/VALID-PARTIAL with committed fix)
  leave_open: INVALID, OUT_OF_SCOPE, UNCLEAR
  tone: no sycophancy — state outcome directly

step_7_update_context (only if fixes were pushed):
  purpose: keep both specs/ and the PR description in sync with the code that actually
    landed after review.
  enumerate_changes_once:
    - Diff the pushed fix commits (`git log <base>..HEAD --stat` on the PR head) to
      build a single list of what actually changed, with commit SHAs and the review
      comment ids each fix addresses. Reuse this list for both updates below.

  7a_update_specs (only if specs/ was the context source in step_1b):
    what_to_update:
      - spec.md: acceptance criteria or behavior lines that changed because of a
        VALID / VALID_PARTIAL fix. If the fix corrected a misspecification (not just
        an implementation bug), reflect the corrected requirement, not just the code
        change.
      - plan.md: technical approach, component/file references, or sequencing that
        changed as a result of review feedback.
      - tasks.md: mark tasks affected by fixes; add follow-up tasks for OUT_OF_SCOPE
        items promoted to follow-up tickets.
    what_not_to_update:
      - Do not rewrite history or narrative sections that describe original intent.
      - Do not add spec entries for INVALID comments (nothing changed).
      - If context source was ticket or PR-only, skip 7a and note "manual spec sync
        recommended" in the report.
    process:
      - Edit spec.md / plan.md / tasks.md minimally — one bullet or line per landed
        fix, cross-referencing the commit SHA and the review comment id.
      - Commit as: "docs(specs): sync <folder> with PR #<N> review fixes" and push.
      - No lint/test needed for markdown-only changes.
    idempotency: if spec files already reference the fix commit SHAs, skip.

  7b_update_pr_description:
    what_to_update:
      - Scope / behavior lines that changed because of a VALID / VALID_PARTIAL fix
        (correct wording if the description misdescribed intent, don't just parrot
        the code change).
      - Out-of-scope / follow-up section: add OUT_OF_SCOPE items promoted to
        follow-up tickets, with the ticket link if created.
      - Changelog / "what changed" section if the PR body has one: append one line
        per landed fix, cross-referencing the commit SHA and the review comment id.
      - If 7a updated specs, add a reference to the docs(specs) commit SHA.
    what_not_to_update:
      - Do not rewrite the original problem statement or motivation sections.
      - Do not add entries for INVALID comments (nothing changed).
    process:
      - Fetch current body: `gh pr view <N> --json body -q .body`.
      - Apply minimal edits; preserve existing structure/headings.
      - Update: `gh pr edit <N> --body-file <tmpfile>`.
      - Add an idempotency marker at the end of the body:
          <!-- resolve-pr-comments: description synced head=<SHA> -->
    idempotency: if the current body already contains a marker for the current head
      SHA (or all fix commits are already referenced), skip.

output_sections:
  - "PR — number, URL, head SHA, author"
  - "Context — source used (specs/<folder>, ticket <KEY>, or 'PR body + diff only')"
  - "Comments — total by type, count needing handling"
  - "Triage — table: id, author, location, verdict, action"
  - "Fixes — commits + comment each addresses"
  - "Blast radius — affected call sites/consumers checked per fix, and any
    additional sites updated or flagged"
  - "Gates — lint/test results"
  - "Replies — count posted, threads resolved/left open"
  - "Specs sync — files updated in specs/<folder>/ + commit SHA, or 'skipped: <reason>'"
  - "PR description sync — updated / skipped: <reason>"
  - "Skipped — already-handled/outdated with reasons"
  - "Next Steps — awaiting response, unresolved questions, or None"

failure_handling:
  auth_fails: report "gh auth refresh" and exit
  gates_fail_3x: don't push; report failure + unaddressed comments
  merge_conflict: report; never auto-rebase under review
  dirty_tree: stop and report (never stash)
  not_pr_author: report and exit
