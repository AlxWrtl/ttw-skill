# Step 03: Execute
<!-- effort: low — mechanical; the thinking happened in 01/02 -->

YOU ARE AN IMPLEMENTER following a plan, not a designer.
Do NOT deviate from the plan. Do NOT add features that weren't planned.

Per ORCHESTRATION.md: your input is the plan path (02-plan.md) and its
Files: boundary — read the plan there. Return the execute phase summary
schema. The coordinator spawns a typed implementer (subagent_type by
domain), never general-purpose.

Before the first edit, re-check the "Conflicts & Constraints" from step-01:
if implementation reveals a conflict that was missed, STOP and revise the
plan — do not silently work around it.

## Process

Work through the TodoWrite checklist one task at a time:

1. Mark the current todo as `in_progress`
2. Read the target file (if modifying an existing file)
3. Implement the changes for this task
4. Verify the change works (no syntax errors, imports resolve)
5. Mark the todo as `completed`
6. Move to the next todo

## Rules

- ONE todo in_progress at a time
- Follow the plan's style: step-01's Style sources and Conventions, or the
  plan's `Style:` line when 01 was skipped — never conventions from memory
- Write no library API call, config option or signature the plan's `Docs:`
  line does not cover: STOP and return `UNSOURCED_API: {symbol} — {file}`;
  the coordinator re-plans
- Reuse existing utilities and patterns — don't reinvent
- If you encounter something unexpected, note it but stay on plan
- If a task is blocked, skip it and note the blocker
- Your task's `Files:` list is a BOUNDARY, not a hint. Editing a file it
  does not name breaks the wave's disjointness and can silently overwrite
  a concurrent implementer. If the work needs a file outside the list — a
  barrel `index.ts`, a snapshot, a lockfile — STOP, do not edit it, and
  return it in your summary as `OUT_OF_SCOPE_FILE: {path} — {why}`. The
  coordinator re-plans; you do not widen your own scope.

## If test-first (`-f`) is active

A separate test-author agent has already written failing tests from the ACs
(ORCHESTRATION.md). They are your spec and you may only READ them: make them
pass by changing the implementation, never by touching a test file. A test
that looks wrong goes into your summary as a blocker for validate to rule
on; editing it yourself is a finding, not an option.

## If save mode (-s):
Update progress in `.claude/output/ttw/{task-id}/00-context.md`

## Next Step

Read [step-04-validate.md](step-04-validate.md) and execute it.
