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

## A wave is FILE-DISJOINT, not merely dependency-independent

Two tasks with no dependency on each other can still write the same file,
and the coordinator runs a wave's implementers concurrently. The second
writer wins, silently, and the loss surfaces later as a change that
"reverted itself".

So the `Files:` field above is not documentation, it is the partition key:

- Take the `Files:` lists of the tasks in the wave and compare them
  PAIRWISE. If one path is named by more than one task, the wave is
  INVALID. Do not deduplicate first: a union hides exactly the repeat you
  are looking for.
- Fix it by splitting, never by hoping: move one of the colliding tasks
  into a later wave, or merge the two tasks into one that owns the file.
  A collision is never a blocker — the wave just gets narrower, and a wave
  of width 1 is a correct answer, not a failure.
- Two tasks editing different regions of one large file still collide.
  This repo's `home/claude-code/skills.nix` holds every skill, so any two
  skill tasks serialise. Barrel files are the common case: the task that
  adds a module also owns its `index.ts` re-export, and no other task in
  that wave may name `index.ts`.
- `Files:` must be concrete repository paths, comma-separated. A
  directory, a glob, `(various)`, `TBD` or "the auth module" is NOT a
  `Files:` value — it is an admission the task is not ready to be
  scheduled. Give it its own wave, or send it back to step-02-plan. "It
  might touch a few things" is how a collision gets planned in.

State the disjointness check per wave, so the coordinator can see it held
rather than assume it:

```
Wave 1: T1, T2 — pairwise disjoint
        T1: src/types.ts, src/index.ts   T2: migrations/003.sql
Wave 2: T3, T4 — pairwise disjoint
        T3: src/api/user.ts              T4: src/ui/Form.tsx
```

This is cheaper than isolating the agents from each other, and it removes
the collision at the source rather than containing it. Isolation is the
answer to a DIFFERENT problem — see the worktree rule in ORCHESTRATION.md.

Return to step-02-plan flow.
