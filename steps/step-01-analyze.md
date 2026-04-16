# Step 01: Analyze

YOU ARE AN EXPLORER, not a planner. Do NOT plan or implement yet.
Your only job is to deeply understand the codebase and the task.

## Strategy

Evaluate task complexity across 4 dimensions:
- **Scope**: how many files/modules affected?
- **Libraries**: unfamiliar dependencies?
- **Patterns**: existing conventions to follow?
- **Uncertainty**: unclear requirements?

### If economy mode is active:
Use Glob and Grep directly. Read only the most relevant files. No agents.

### If economy mode is NOT active:
Launch parallel Explore agents based on complexity:
- Simple (1-2 files): 1-2 agents
- Medium (3-5 files): 3-5 agents
- Complex (6+ files): 5-10 agents

Each agent should explore a different aspect:
- File structure and conventions
- Existing patterns and utilities
- Related components and dependencies
- Test patterns (if -t flag active)

## Output

Document your findings:
- **Requirements**: what exactly needs to be built
- **Affected files**: list of files to create/modify
- **Conventions**: patterns to follow (naming, structure, imports)
- **Dependencies**: libraries, utilities, types to use
- **Risks**: potential issues or unknowns

## If save mode (-s):
Write findings to `.claude/output/apex/{task-id}/01-analyze.md`

## Next Step

Read [step-02-plan.md](step-02-plan.md) and execute it.
