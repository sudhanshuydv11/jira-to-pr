# Quality Gates Protocol

Shared protocol for lint/test quality gates.

## Commands

- Lint: `npm run lint`
- Tests: `npm test`

## Rules

- Gates always run — never skipped or bypassed.
- On failure: hand back to a fresh implement subagent with failure output.
- Maximum 3 remediation attempts, then stop and report.
- Evidence recorded verbatim (with timestamps) in `quickstart.md`.
- Never fabricate evidence — only real command output.
