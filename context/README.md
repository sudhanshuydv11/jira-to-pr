# Context Sources

Cache of context about repos/sources this project integrates with. Declared in
`.claude/config.json` under `contextSources`. Refresh with `/sync-context`
(see `.claude/commands/sync-context.md` for the full protocol).

Each file's frontmatter records `source`, `branch`/`commit` (or equivalent), and
`lastUpdated`/`generated`. Always check these against the live source before trusting
the cache for anything time-sensitive.

| name | source | branch | target file |
|---|---|---|---|
| cmp-portal | https://github.com/bct-adidas/cfccm-cmp-portal | main | `cmp-portal.md` |
| libraries-api | local: `package.json` | — | `libraries-api.md` |
