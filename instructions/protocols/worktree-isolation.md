# Worktree Isolation Protocol

Shared protocol for running parallel pipelines in isolated git worktrees.

## Creation

```bash
git worktree add ../.sprint-run/<TICKET> -b feat/<TICKET>-<slug> origin/<base>
```

## Provisioning

`.claude/` and `.specify/` are gitignored, so a fresh worktree won't have them.

1. Copy in: `cp -R .claude .specify AGENTS.md CLAUDE.md <worktree>/`
2. Verify: `<worktree>/.specify/scripts/bash/check-prerequisites.sh` is present and executable.
3. Abort that ticket if provisioning fails.
4. Install deps: `npm ci` in the worktree (node_modules not shared).

## Cleanup

- **Success** (branch pushed): remove the worktree.
- **Failure**: leave in place for inspection.

## Rules

- Never run two pipelines in the same working tree.
- If worktree creation fails, downgrade that ticket to sequential execution in the main tree.
- Main tree must be clean before sequential work — never stash/discard user work.
