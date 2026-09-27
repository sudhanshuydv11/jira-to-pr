# Subagent Orchestration Protocol

Shared protocol for multi-step pipeline commands that run steps as subagents.

## Model

Steps run as **independent sequential subagents** to protect the context window.
The orchestrator retains only step status and artifact paths — never full subagent transcripts.

## Subagent Return Format

Each subagent returns a compact status object:

```json
{ "step": "<name>", "status": "complete|blocked|failed",
  "artifacts": ["<paths>"], "notes": "<=2 sentences", "blockers": [] }
```

## Parallel Steps

When steps are independent (e.g. `clarify` + `analyze`), launch them as **parallel background subagents** and collect both results before proceeding.

## Rules

- A subagent receives: step goal, `FEATURE_DIR` path, needed artifacts — never prior transcripts.
- Maximum 3 remediation attempts per failing gate.
- Maximum 3 review cycles total.
- Never exceed loop bounds — report and stop instead.
