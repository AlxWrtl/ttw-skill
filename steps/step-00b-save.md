# Step 00b: Save Mode Setup

Create output directory for this task:
```
.claude/output/ttw/{task-id}/
```

Where `{task-id}` is a zero-padded sequential number + short slug (e.g., `01-user-auth`).

Create initial context file:
```
.claude/output/ttw/{task-id}/00-context.md
```

With content:
```markdown
# TTW: {task description}
Date: {current date}
Tier: {tier}
Flags: {active flags}
Branch: {branch name}
Baseline: {gate} -> {green | red: check | skipped: reason}

## Progress
| Step | Status | Notes |
|------|--------|-------|
| 00-init | running | |
| 01-analyze | pending | |
| 02-plan | pending | |
| 03-execute | pending | |
| 04-validate | pending | |

## Tokens
Context growth, not billed tokens.

| phase | agent | model | tokens |
|-------|-------|-------|--------|

coordinator context: {start}→{end}
```

When 01-analyze is skipped (files known, ROUTING.md), mark it `skipped`.

Write this file as init STARTS, before the baseline, and keep the table
live: a step goes `running` before its work begins and `complete` when
it ends, one edit each, never batched at ship time. ttw-board puts a
run in the column of its first `running` (else `pending`) step, so a
table written late or filled at the end shows only Execute and Terminé.

`## Tokens` — one row per spawned agent, `tokens` copied from its
task-notification `<usage><subagent_tokens>`. `coordinator context`:
`{start}` from the ttw-flags hook line `TTW start: context=N`, `{end}`
read at end of run. Any value absent → write `unknown`, never a guess.
These are context sizes, not billed tokens: never present them as cost.

Return to step-00-init flow.
