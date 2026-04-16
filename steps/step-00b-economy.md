# Step 00b: Economy Mode Override

ECONOMY MODE ACTIVE. These 6 rules override ALL subsequent steps:

1. **No subagents.** Never use the Agent tool. Use Glob, Grep, Read directly.
2. **No parallel exploration.** Explore sequentially, one file at a time.
3. **Minimal scope.** Read only files directly relevant to the task.
4. **Skip optional steps.** If examine (-x) is also set, do a self-review checklist instead of launching review agents.
5. **No TodoWrite.** Track progress mentally, don't create formal todo lists.
6. **Concise outputs.** Shorter summaries, no detailed analysis documents.

These rules save ~70% tokens. Apply them to every subsequent step.

Return to step-00-init flow.
