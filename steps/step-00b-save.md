# Step 00b: Save Mode Setup

Create output directory for this task:
```
.claude/output/apex/{task-id}/
```

Where `{task-id}` is a zero-padded sequential number + short slug (e.g., `01-user-auth`).

Create initial context file:
```
.claude/output/apex/{task-id}/00-context.md
```

With content:
```markdown
# APEX: {task description}
Date: {current date}
Flags: {active flags}
Branch: {branch name}

## Progress
| Step | Status | Notes |
|------|--------|-------|
| 00-init | complete | |
| 01-analyze | pending | |
| 02-plan | pending | |
| 03-execute | pending | |
| 04-validate | pending | |
```

After each step completes, update this progress table.

Return to step-00-init flow.
