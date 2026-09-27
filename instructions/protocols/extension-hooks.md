# Extension Hooks Protocol

Shared protocol for checking `.specify/extensions.yml` hooks before/after a command step.

## Procedure

Given a hook phase (e.g. `before_specify`, `after_implement`):

1. Check if `.specify/extensions.yml` exists. If not, skip silently.
2. Read it; if YAML is invalid, skip silently.
3. Look for entries under `hooks.<phase>`.
4. Filter out hooks where `enabled` is explicitly `false`. Treat missing `enabled` as enabled.
5. Do NOT evaluate `condition` expressions — if a hook has a non-empty `condition`, skip it.
6. For each executable hook, output based on `optional` flag:

**Optional** (`optional: true`):
```
## Extension Hooks

**Optional Hook**: {extension}
Command: `/{command}`
Description: {description}

Prompt: {prompt}
To execute: `/{command}`
```

**Mandatory** (`optional: false`):
```
## Extension Hooks

**Automatic Hook**: {extension}
Executing: `/{command}`
EXECUTE_COMMAND: {command}

Wait for the result before proceeding.
```
