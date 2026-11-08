# Step 01: Analyze
<!-- effort: high -->

Skipped when the files to touch are already known: Standard goes straight
to 02-plan (ROUTING.md linear spine). Run it only when they are not.
Diagnosis never skips it: it reproduces the error first, files known or not.

YOU ARE AN EXPLORER, not a planner. Do NOT plan or implement yet.
Your only job is to deeply understand the codebase and the task.

## Who runs this (per ORCHESTRATION.md)

- The COORDINATOR does the parallel Explore fan-out itself
  (a phase agent cannot spawn — depth=1), collects the bounded Explore
  summaries, then either synthesizes directly or spawns one analyzer agent
  with those summaries as input. Return the analyze phase summary schema.

## Strategy

Evaluate task complexity across 4 dimensions:
- **Scope**: how many files/modules affected?
- **Libraries**: unfamiliar dependencies?
- **Patterns**: existing conventions to follow?
- **Uncertainty**: unclear requirements?

The coordinator launches parallel Explore agents (`model: haiku`, enforced by the research-model hook), count scaled to scope:
- 1-2 files: 1-2 agents
- 3-5 files: 3-5 agents
- 6+ files: 5-10 agents

Each agent explores a different aspect:
- File structure and conventions
- Existing patterns and utilities
- Related components and dependencies
- Test patterns (if -t flag active)

**Each agent must return a condensed, distilled summary (~1-2k tokens):
files, conventions, risks — NOT raw file dumps.** Their bounded summary is
what enters your context; do not paste full file contents back.

## Output

Document your findings:
- **Requirements**: what exactly needs to be built
- **Affected files**: list of files to create/modify
- **Style sources**: each of CLAUDE.md, AGENTS.md, `.claude/rules/`,
  CONTRIBUTING*, .editorconfig, lint/format configs, docs/*style* that
  exists, cited by path as read — or `none`. A fixed list, not a sample.
- **Conventions**: patterns to follow (naming, structure, imports), each
  traced to a style source or a `file:line` example
- **Versions**: installed version of each library or flake input the task
  touches, read from its lockfile, never from memory (feeds the plan's `Docs:` line)
- **Dependencies**: libraries, utilities, types to use
- **Risks**: potential issues or unknowns

## Probe contract (every ad hoc probe)

A probe is anything you write on the spot to decide a question: a grep, a
find, a one-off script, a pipeline. Every probe reports three fields
together — `total, matched, sample` — and never `matched` alone:
- `total`: how many units the probe actually READ (files opened, lines
  scanned). Not how many exist in theory.
- `matched`: how many of them satisfied the predicate.
- `sample`: up to 3 matches quoted VERBATIM. No sample, no match.

A count with no denominator is unfalsifiable, which is why `matched` alone
is never a report.

`matched: 0` with `total: 0` is a broken instrument, not a finding. The walk
read nothing, so the predicate never ran; the zero describes the harness.
The rule: a predicate that cannot tell "nothing to find" from "nothing
read" is not a predicate.

Before reporting ANY zero: re-run the SAME probe against a case that MUST
match — a line you already have in front of you — and show it returning
non-zero. That positive control ships with the zero. An absence reported
without it is an untested claim, and step-05-examine treats it as a finding.

## Conflicts & Constraints (REQUIRED — confront task vs codebase)

This is what makes step-01 analysis, not just exploration. You MUST fill every
bullet. Write "none found" only after actually looking — never leave blank:
- **Constraining patterns**: existing conventions/architecture that constrain
  HOW this must be built (e.g. "all DB access goes through repo layer X").
- **Divergences**: where the task as asked would break or contradict an
  existing pattern — name the file/pattern and the conflict.
- **Decisions needed from user**: ambiguities that change the design and that
  you cannot resolve from the codebase. Surface them before planning; under
  `-q` they are asked, not merely listed (see below).
- **Out-of-scope temptations**: nearby things that look broken but are NOT
  this task — list them so the plan does not creep into them.

## Clarify (`-q`)

If `-q` is active, close this phase by LISTING the ambiguities in your
SUMMARY. You have no user channel — you never ask them yourself. What
qualifies: the "Decisions needed from user" above, and every premise the plan
will not be able to source:
Unsourceable premises become questions, never silent assumptions.

The COORDINATOR owns the asking: it reads this list and puts it to the user
as a SINGLE AskUserQuestion — at most 3 questions, each with concrete options
— BEFORE spawning the planner, so no plan text exists yet when they are asked.

Three, not ten: asking upfront is the best-measured error reducer in the
literature, and the first question carries most of the gain. Nothing here
blocks a run without `-q` — it just plans on the safest reading and says so.

## Absences and numbers

There is no separate analyze pass by Fable: every absence and every number
the plan rests on reaches a re-reader as a premise with its command — the
examine reviewer when -x runs, replayed only under -p.

## If save mode (-s):
Write findings to `.claude/output/ttw/{task-id}/01-analyze.md`

## Next Step

If obsidian mode (-o) is active:
  Read [step-01b-obsidian-context.md](step-01b-obsidian-context.md) and execute it.
Else:
  Read [step-02-plan.md](step-02-plan.md) and execute it.
