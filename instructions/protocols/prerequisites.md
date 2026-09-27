# Prerequisites Script Protocol

Shared protocol for running the Speckit prerequisites check.

## Command

```bash
.specify/scripts/bash/check-prerequisites.sh [FLAGS]
```

Common flag combos:
- `--json` — basic mode, returns FEATURE_DIR, AVAILABLE_DOCS
- `--json --paths-only` — returns REPO_ROOT, BRANCH, FEATURE_DIR only
- `--json --require-tasks --include-tasks` — requires tasks.md exists, includes task content

## Rules

- All paths from the output must be treated as absolute.
- For single quotes in args: use `'I'\''m Groot'` or `"I'm Groot"`.
- Parse the JSON output; do not rely on stdout text parsing.
