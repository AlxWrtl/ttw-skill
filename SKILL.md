---
name: apex
description: "Systematic implementation using APEX methodology"
effort: high
---

# APEX: Systematic Implementation Workflow

You are about to execute a structured, multi-step implementation workflow.
Each step is a separate file that you will read and execute one at a time.
This keeps instructions fresh in your context for maximum attention.

## Available Flags

| Enable | Disable | Description |
|--------|---------|-------------|
| -a | -A | Auto — skip confirmations |
| -x | -X | Examine — adversarial code review |
| -s | -S | Save — persist outputs to files |
| -t | -T | Test — create and run tests |
| -e | -E | Economy — no subagents, direct tools only |
| -b | -B | Branch — create git branch |
| -pr | -PR | Pull request — commit + PR (implies -b) |
| -k | -K | Tasks — task breakdown with dependency graph |
| -m | -M | Teams — Agent Teams parallel execution (implies -k) |
| -v | -V | Verify — research plan online before executing |
| -i | | Interactive — configure flags via menu |
| -r | | Resume — continue previous task |

## Common Usage

```
/apex add feature                    # Basic
/apex -a -s implement auth           # Autonomous + save
/apex -a -x -s fix bug              # Full autonomous with review
/apex -a -t -pr add endpoint        # Auto + tests + PR
/apex -e simple fix                  # Economy mode (save tokens)
/apex -a -x -t -pr full feature    # Everything enabled
```

## Execution

Read [steps/step-00-init.md](steps/step-00-init.md) and execute it now.


## Input/Output Contract
- **Expects:** task description with optional flags. Example: /apex -a -x implement user auth
- **Produces:** complete implementation through progressive steps: init → analyze → plan → execute → validate (+ optional: tests, examine, resolve, finish).
- **Side effects:** modifies source files, optionally creates tests, commits, creates PRs.


## Scope
- **Use this skill when:** Implementing features, modules, or tasks that benefit from structured multi-step execution with quality gates.
- **Do NOT use for:** Quick fixes (<20 lines) → use quick-fix. Debugging → use /debug. Research → use Explore agent.


## Handoffs
    - If task is broken and needs diagnosis → use debug skill instead.
    - If scope is unclear → run /discuss first.
    - After tests fail repeatedly → hand off to debugger agent.
    - After finish on L/XL changes → hand off to code-reviewer agent.

