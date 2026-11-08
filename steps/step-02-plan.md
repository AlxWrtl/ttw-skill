# Step 02: Plan

YOU ARE A PLANNER, not an implementer. Do NOT write any code yet.

Files known (01-analyze skipped, not High-stakes): the COORDINATOR writes 02-plan.md itself, short: Files:, ordered steps, command-verifiable ACs, Docs: line, Style: line. No plan subagent.

Per ORCHESTRATION.md: the coordinator, when 01-analyze ran or on
High-stakes, spawns this as a fresh planner agent (`subagent_type: Plan`, `model: opus`) whose input is the analyze phase summary (not
the raw transcript). Return the plan phase summary schema with the full plan
in it: the Plan agent is read-only (no Write tool), so the coordinator
persists it as 02-plan.md. The coordinator (Opus 5.5) then reviews the plan and approves or
re-briefs before execute — execute never starts on an unapproved plan.

## Clarify first (`-q`)

If `-q` is active and step-01 raised ambiguities, the coordinator has already
put them to the user — one AskUserQuestion, 3 questions maximum, concrete
options — before spawning you: the answers arrive in your brief and you plan
on them. A question asked after the plan is written is a question asked too
late — the plan has already committed.

## ULTRA THINK

Before writing the plan, mentally simulate the entire implementation:
- Walk through every file you'll create or modify
- Consider the order of changes (what depends on what)
- Identify where things could go wrong
- Think about edge cases the user didn't mention

## Create Implementation Plan

Produce a structured plan with:

1. **Premises** — what you are taking as true, stated BEFORE any task.
   A task list cannot show that you misunderstood the domain; this section
   can. It is what the user must be able to contradict in ten seconds.
   - **The target, restated** — what you believe is being asked, in your own
     words. Misread intent is the most common premise error, and the one a
     task list hides best.
   - **Assumed business rules** — how the product actually behaves, stated
     flatly: "a journalist can never see whether a car is available".
   - **Assumed environment state** — what exists right now: rows in a table,
     deployed version, accounts present, branch state.
   - **Explicitly excluded scope** — nearby things this plan does NOT touch.

   **Every premise carries its source.** Admissible: a command and its
   output, a `file:line` you actually read, or a verbatim quote from the user
   or a spec. Not admissible: recollection, or a truncated read presented as
   exhaustive. When the analyze summary is your only input, say so and name
   what that summary rested on — do not launder it into a fact. A premise you
   cannot source is written as an open question; that is a valid outcome, not
   a failure.

   **Tag every premise.** The FIRST token of a premise line is its tag:
   `[M] measured or [I] inferred`. No third value, no untagged premise.
   - `[M]` is legal ONLY with its evidence inline on the same line: the
     command and its output, or a `file:line` that was actually opened.
     No evidence written down, no `[M]`.
   - `[I]` covers everything else — INCLUDING whatever the analyze summary
     asserts without a source of its own. Inherited confidence is still
     inference; a claim does not become measured by changing hands.

   Hard rule: no `[I]` premise becomes a string the user sees, a number
   written into the code, or a statement reported as fact. It is measured
   before it is used, or it is written as an open question.

   The tag is `never self-audited`. A tag re-read only by the process that
   wrote it measures nothing — that process already believes it. It counts
   only when ANOTHER process reads it, and that reader is NAMED: the step-05-examine
   reviewer's premise box carries it on every run, plus the Fable premises
   pass whenever `-p` adds one. A tag that nobody re-reads is decoration.

   Premise and scope errors dominate user corrections, and most surface only
   after code exists. A stronger planner does not fix this: reasoning models
   rarely flag a false premise on their own.

2. **Tasks** — numbered, ordered by dependency:
   ```
   T1: Create types/interfaces in types.ts
   T2: Add database migration
   T3: Implement API endpoint (depends on T1, T2)
   T4: Create UI component (depends on T1)
   T5: Wire up route (depends on T3, T4)
   ```

3. **Acceptance Criteria** — each one names the command that proves it,
   with the exit code or output expected:
   ```
   AC1: `pnpm test -- items.create` exits 0: the form creates an item
   AC2: `pnpm test -- items.validation` exits 0: errors display inline
   AC3: `pnpm exec playwright test items-redirect.spec.ts` exits 0: success redirects to the list
   AC4: [lecture] — the error wording matches the product copy; no command judges tone
   ```
   An AC that names no command is reported non prouvé at validate, never
   green. Two exceptions, declared on the AC itself: `[lecture] — reason`
   when no command can prove it (the reason says why), and
   `[deferred to user]` when the proof needs the user (sudo, a password).
   Use them rarely: the share of `[lecture]` ACs is measured apart, and the
   ttw-metrics-probe hook lists the unproven ACs each time 02-acs.md is written.

4. **Testing Strategy** (if -t flag active):
   ```
   - Unit tests for validation logic
   - Integration test for API endpoint
   - Component test for form submission
   ```

5. **Risks & Mitigations**

## Docs line — categorical, not a confidence score

A plan with any task that writes or changes a library API call, a config
option, a function signature or a dependency version carries a `Docs:` line
per language or library.
Nothing of that kind (prose, comments, a typo) → `Docs: n/a (text)`.
The trigger is the category of the change, never how sure you feel.
Format: `Docs: {lib}@{installed version} — {source} (rung N)`.

Installed version first: read from the lockfile BEFORE any search —
package.json + pnpm-lock.yaml, flake.lock, pyproject.toml / uv.lock, Cargo.lock.
Then the ladder; stop at the first rung that answers:
1. Pinned `libdocs <name> "<question>"` (`libdocs --list`).
2. `libdocs --search <lib>`, keeping only an id filtered to the installed major.
3. Official site: WebFetch the doc page, or WebSearch restricted to the official domain.
4. Source at the pinned version: `node_modules/{pkg}`, the `/nix/store` tree of the flake.lock rev, the GitHub tag.
5. Nothing found: run the mechanical probe NOW, at plan time (`nix eval`, a
   typecheck stub), and cite its output in the `Docs:` line as `[M]`; no
   probe possible → it is an open question, never a task.
Mechanical proof beats a doc page when one exists (`nix eval`, typecheck): name it in the `Docs:` line.

A plan whose tasks touch one of these categories and carries no `Docs:` line is REJECTED at approval and re-briefed, never executed.

## Create TodoWrite Checklist

Convert tasks into a TodoWrite checklist. Only ONE todo can be in_progress at a time.

## Persist the acceptance criteria ALONE

Besides the plan, write the Acceptance Criteria — and nothing else, no
premises, no rationale, no task list — to
`.claude/output/ttw/{task-id}/02-acs.md`.
Copy every `Docs:` line into it as one criterion, `AC-docs:` — a condition
on the diff, not rationale — so validate, Fable and `-e` check it without the plan.
A verifier is handed 02-acs.md and never the plan. That file is the ONLY thing a
verifier is ever handed as the spec. The plan is not a substitute: it
carries the reasoning, and a reviewer told why the code is right stops
looking for the reason it is not. Every later step that names an ACs file
means this one.
Each AC keeps its command, or its `[lecture] — reason` / `[deferred to user]` marker, in that file: the verifier sees no other proof.

## If tasks mode (-k):
Read [step-02b-tasks.md](step-02b-tasks.md) and execute it before proceeding.

## User Approval

The coordinator, not this planner, talks to the user (ORCHESTRATION.md "Plan approval"):
- **Present the Premises FIRST**, before the task list. An unsourced premise
  must be visible in the transcript before the code that rests on it exists.
- Then present the rest of the plan
- In high-stakes mode or under `-q`: ask explicitly whether any premise is
  wrong — that is the question worth a round trip — and wait for approval
  before proceeding.
- Every other mode: show them and proceed without waiting; a contradiction
  raised later follows the rule below.

**Whenever the user contradicts a premise — at plan approval or later, mid-run
— persist the correction before execute proceeds.** The rule holds at any
moment of the run: a contradiction raised while execute is already running is
persisted before the work resumes, not at the end. One line, minimal scope —
the correction as stated, never a generalization of it (inferring broader
scope from a single counter-example is the documented failure mode). Date it
and mark it `corrected by user`. Destination, exactly one of three:
- Rule true for the whole team → the project's `.claude/rules/*.md` (versioned, reviewed in PR).
- Project fact that does not generalize into a rule → the project `CLAUDE.md`.
- Personal preference of this user → their `~/.claude/CLAUDE.md`, never versioned in the repo.

A rule that must be IMPOSSIBLE to violate goes into a hook, not prose —
instruction files are context, not enforcement. This is what makes a
correction permanent: the same premise, re-corrected in a later session,
means this step was skipped.

## Verify passes — record them HERE

`-p` forces the Fable premises pass; without `-p` typed, no premises pass runs.
The premises pass is a spec check: it replays the
cited read-only commands, never a write-effect one, and never rewrites the
plan.

**Record in the plan which verify passes will run** — the external detector
(`-e`), the Fable fallback if its trigger fires (no usable external verdict
on high-stakes), and the Fable premises pass if `-p` is typed. That record
is the audit trail of every Fable spawn in the run.

## Next Step

Consult [ROUTING.md](ROUTING.md): if `-v` go to 02c-verify, else go to
03-execute.
