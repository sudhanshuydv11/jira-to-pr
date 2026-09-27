---
description: Read the source list from .claude/config.json, check whether each repo has changed since last sync, and refresh only the stale context files in .claude/context/.
argument-hint: "[optional: 'check' (report staleness only), 'force', 'only <NAME>']"
---

input: $ARGUMENTS (consider if not empty)

purpose: >
  Maintain .claude/context/ as a cheap, trustworthy cache of related repo context.
  Sources declared in .claude/config.json. Checks whether source moved, regenerates only stale.

rules:
  - Never serve stale cache silently — if freshness unverifiable, say so with generated date
  - Never regenerate what hasn't changed (matching SHA = current)
  - Never invent content about a source repo — everything from files actually read
  - Preserve "## Local Notes" section across refreshes
  - Never write secrets into context files
  - .claude/config.json is only source of truth for which repos to sync
  - Never overwrite user-set fields. Never invent a "why".

config:
  location: .claude/config.json under contextSources
  user_provides: url only
  derived_fields: name, branch, paths, why (written back on first sync)
  skip: entries with template: true

step_1_resolve_sources:
  - Parse URL: extract owner/repo from github URL
  - Derive name: strip prefixes (cfccm-, cmp-, bct-) and suffixes (-ui, -service, -api, -app), lowercase
  - Derive branch: gh api repos/<o>/<r> --jq '.default_branch'
  - Derive paths: list repo root + src/, select integration-surface dirs (cap 5, empty = whole repo)
  - Derive why: check dependency on this package, static resource usage, API calls. If no link found: "Relationship not established from code — verify with the user."
  - Write derived fields back to config.json
  - Honour "only <NAME>" filter

step_2_freshness_check:
  command: gh api repos/<o>/<r>/commits/<branch> --jq '{sha: .sha, lastUpdated: .commit.committer.date}'
  use: commit.committer.date (NOT pushed_at, NOT author.date)
  verdicts:
    file_missing: GENERATE
    remote_newer: REFRESH
    same_time_different_sha: REFRESH (force-push)
    both_match: CURRENT (skip)
    frontmatter_broken: REFRESH
    probe_failed: UNVERIFIED (keep, flag)
    force_passed: REFRESH all
  if_check_mode: report table and exit without writing

step_3_generate_or_refresh:
  read_via: gh api (no clone) — directory listings + file contents
  extract: public API surface, integration contracts, conventions, breaking-change constraints
  omit: internal implementation detail. Target <200 lines.
  frontmatter: source, branch, commit, lastUpdated, generated, paths
  preserve: existing "## Local Notes"
  update: .claude/context/README.md table

output_sections:
  - "Sources — table: name, repo, branch, target file"
  - "Derived — fields filled in (omit if nothing derived)"
  - "Freshness — table: name, cached/remote commit, verdict"
  - "Actions — files generated/refreshed + skipped"
  - "Unverified — sources whose freshness couldn't be checked, or None"
  - "Next Steps — why needing confirmation, sources to add, auth fixes, or None"

failure_handling:
  no_sources: report with pointer to config; exit cleanly
  config_missing_or_invalid: stop and report
  gh_unauth: report "gh auth refresh -h github.com"; fallback git ls-remote for SHA
  source_inaccessible: mark UNVERIFIED, keep cache, continue others
  rate_limited: report remaining quota + skipped sources
