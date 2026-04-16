# Step 02b: Task Decomposition

Break the plan into individual task files with a dependency graph.

For each task, create a structured entry:
```
Task: T{n} — {name}
Files: {files to create/modify}
Depends: {T1, T2, ...} or none
Agent: {suggested agent type}
Instructions: {specific implementation details}
Verify: {how to verify this task is done}
```

Order tasks by dependency — tasks with no dependencies first.
Group independent tasks into waves for parallel execution.

```
Wave 1: T1, T2 (no deps — can run in parallel)
Wave 2: T3, T4 (depend on wave 1)
Wave 3: T5 (depends on wave 2)
```

Return to step-02-plan flow.
