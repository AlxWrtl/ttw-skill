# Step 00: Initialize

YOU ARE AN INITIALIZER, not an executor. Do NOT start implementing yet.

## Parse Flags

Extract flags from $ARGUMENTS. Default: all flags OFF.
- If `-pr` is set, auto-enable `-b` (branch)
- If `-m` is set, auto-enable `-k` (tasks)
- Uppercase flag disables (e.g., `-A` disables auto)

## Initialize State

Record the following:
- **Task**: the user's request (everything after flags)
- **Flags**: which flags are active
- **Working directory**: current project path
- **Git status**: current branch, clean/dirty, uncommitted changes

## Present Summary

Display a compact summary:
```
APEX initialized
Task: {description}
Flags: {active flags}
Branch: {current branch}
Status: {clean/dirty}
```

## Conditional Sub-Steps

Execute these in order, ONLY if the corresponding flag is active:

1. If `-i` (interactive): Read [step-00b-interactive.md](step-00b-interactive.md) and execute it.
2. If `-b` or `-pr`: Read [step-00b-branch.md](step-00b-branch.md) and execute it.
3. If `-e` (economy): Read [step-00b-economy.md](step-00b-economy.md) and execute it.
4. If `-s` (save): Read [step-00b-save.md](step-00b-save.md) and execute it.
5. If `-r` (resume): Look for saved state in `.claude/output/apex/` and restore context. Skip to the last incomplete step.

## Next Step

Read [step-01-analyze.md](step-01-analyze.md) and execute it.
