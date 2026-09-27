# Resume Detection Protocol

Shared protocol for detecting and resuming from incomplete pipeline state.

## Procedure

1. Run `.specify/scripts/bash/check-prerequisites.sh --json --paths-only` to get REPO_ROOT, BRANCH, FEATURE_DIR.
2. Probe completion state in order:

| Step | Complete when |
| --- | --- |
| specify | `FEATURE_DIR/spec.md` exists and on a feature branch |
| plan | `plan.md` exists (+ `research.md` if unknowns existed) |
| tasks | `tasks.md` exists with at least one task |
| implement | all required tasks in `tasks.md` are checked |
| gates | `quickstart.md` records passing lint + test evidence |
| review | review verdict recorded for current HEAD |
| PR | open PR exists for the branch |

3. Announce: `Resuming at step N (steps 1..N-1 already complete)`.
4. If everything is complete, report and stop — do not re-run.

## Rules

- Re-invoking after interruption is safe and must not redo finished work.
- Never redo completed work on resume.
