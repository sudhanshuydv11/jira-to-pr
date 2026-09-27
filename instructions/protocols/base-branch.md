# Base Branch Resolution Protocol

Shared protocol for resolving the base branch. Used by: `sprint-run`, `run-full-pr`, `code.review`, `tasks`.

## Procedure

1. Explicit base in user arguments wins.
2. Otherwise read `BASE_BRANCH` from repo-root `.env`:
   `node -e "require('dotenv').config();console.log(process.env.BASE_BRANCH ?? '')"`
3. Prefer `origin/<BASE_BRANCH>`; fall back to local ref of same name.
4. If unset/empty, default to `main` **and warn explicitly**.
5. Report the resolved base and its source (`.env`, argument, or fallback).

## Constraints

- `.env` is gitignored — if read fails, stop and ask (don't guess).
- In multi-ticket runs (`sprint-run`), resolve **once** and reuse for all worktrees/reviews/PRs.
- Same value must be used for review merge-base and PR base to prevent drift.
