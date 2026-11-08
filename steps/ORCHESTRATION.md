# TTW Orchestration — fresh context per phase

Goal: each phase (analyze, plan, execute, validate) runs in an ISOLATED
subagent with a clean context, so the workflow never loses or pollutes
context as it grows. The coordinator keeps only a chain of distilled
summaries — never the raw work of each phase.

## Roles

- **Coordinator** = the main `/ttw` run. It is the ONLY agent allowed to
  spawn (subagents cannot spawn subagents — depth is capped at 1). Outside
  Direct it does NOT do the analysis/coding itself; it spawns a phase agent, receives its
  summary, verifies it, persists it, then spawns the next phase.
- **Phase agent** = a fresh subagent (Agent tool) per phase. Receives a
  self-contained brief, works in its own window, returns ONLY a bounded
  summary (~1-2k tokens). Its raw context is discarded after it returns.
  A targeted question in a text file > ~30 KB that is not an instruction file
  (skill, step, plan): `jev-locate FILE "question"` (allowed_domains
  api.typesafe.ai) may replace the whole Read; instructions are read whole.

## Model routing — Opus 5.5 workhorse, Codex detector, Fable = fallback verifier

NEVER let a phase spawn inherit the session model — ALWAYS pass an explicit
`model` parameter on every Agent call. `opus` = Opus 5.5, the current workhorse.

| Phase | Agent | model |
|-------|-------|-------|
| Analyze fan-out | Explore / codebase-navigator | haiku — read-only search only |
| Analyze synthesis | analyzer phase agent (`subagent_type: Plan`, read-only — the coordinator writes 01-analyze.md) | `opus` (effort high) |
| Plan | coordinator inline when 01-analyze was skipped (Standard); plan phase agent (`subagent_type: Plan`, read-only — the coordinator writes 02-plan.md) only after 01-analyze or on High-stakes | `opus` (effort high/max) |
| Execute (parallel waves under `-k`, coordinator's call) | typed implementer (quick-fix / nix-expert / frontend-expert / backend-expert / debugger), never general-purpose | `sonnet` when the plan task spells the exact old→new text or a mechanical sweep with no design decision; `opus` when the task needs a design or logic decision |
| Correction round (verify loop step 5) | typed implementer by domain | `opus`, always — the correction-budget hook denies any other model |
| Bulk / large-context execute | typed implementer (quick-fix / nix-expert / frontend-expert / backend-expert / debugger), never general-purpose | `sonnet` |
| Direct (edit + self-check, DIRECT.md) | coordinator inline | none |
| Run tests | test-runner | sonnet |
| Self-verify (every task) | COORDINATOR inline (Opus 5.5) | none — fresh-context adversarial pass |
| High-stakes fallback verify (no usable external verdict) / premises (`-p`) | fable verifier subagent | `fable` — READ-ONLY, bounded verdict |
| Long test suites needing diagnosis | gate runner subagent | `sonnet` — never haiku; verdict quotes exit code + last failing lines |
| External verify (`-e`, default high-stakes) | codex CLI subprocess, not an Agent spawn | `gpt-6-astra` → `gpt-5.6-terra` — READ-ONLY, bounded verdict |

The Analyze fan-out row is enforced by the research-model hook (PreToolUse
on Agent): an Explore or codebase-navigator spawn whose `model` is not
`haiku`, absent included, is denied.

Haiku locates, maps and audits; it does not explain causes. A "why does X
happen" question (root cause, tracing a library's internals, a fix
recommendation) goes to `subagent_type: Plan`, `model: "opus"`, fed by the
haiku fan-out if needed. Bench 2026-10-14, 3 real project prompts, blind
opus judge: haiku 8.2-8.7/10 on locate and audit, 6.2 on root cause
(two false claims about react-router internals) against opus 9.0.

Effort-tiering first: prefer dialing Opus 5.5 effort (low↔max) over switching
models — a model switch pays the ~15× subagent/context tax. Switch model only
when the tier gap is real (sonnet for mechanical and bulk work, haiku for
read-only search on agents without Write, Edit or Bash).

Plan approval: when the coordinator wrote the plan itself, approval is its own
Files:/ACs/Docs:/Style: check; the planner/premises rules below apply when a
plan agent ran. The coordinator reads the returned plan, checks it against the
task + analyze summary, then approves it or re-briefs the planner — and
re-briefs any plan missing its `Docs:` line (step-02-plan.md). Execute
never starts on an unapproved plan. The planner drafts the premises but never
talks to the user — it has no user channel. So it is the COORDINATOR that
presents the plan's Premises to the user at approval time.
In high-stakes mode or under `-q`, it asks explicitly whether any is wrong and
waits for the answer; in every other mode it shows them and proceeds. It
collects any contradiction, whenever it comes. When one comes back, the
coordinator persists the correction itself, per the rule in step-02-plan, and
does so BEFORE spawning execute — the same applies to a contradiction raised
later, mid-run.

## Test-first (`-f`) — a SEPARATE agent writes the failing tests

If `-f` is active, between plan approval and execute the coordinator spawns a
test-author agent (`model: opus`) whose only input is the ACs. It writes
tests that FAIL against the current code, and nothing else — no
implementation, no fixture that quietly makes them pass. The resulting files
are then read-only for the implementer: editing, deleting or weakening one is
a finding at validate, never an option on the table. The separation is the
point — the agent that writes the code cannot also be the agent that decides
what "passing" means. In Diagnosis, the test-author's failing test IS the
reproduction: it fails on the bug before the debugger agent touches the code.

`-f` is a default of Diagnosis, Standard and High-stakes; `-F` cancels it.
Direct spawns no test-author: it writes its own red test inline (DIRECT.md).
The one exemption is a diff with no behaviour to test (doc, comment,
formatting): the test-author is skipped and the reason is written in
`00-context.md` as `Test-first: skipped — {reason}`.

## High-stakes blocks live in HIGH-STAKES.md

Divergence (`-2`), the Fable rule and premises pass,
External verify (`-e`), Worktree isolation and the Pressure-test are in
`steps/HIGH-STAKES.md`. Read it on High-stakes, or when one of those
flags is typed.

## Coordinator context — command output stays small

The coordinator reads command output in the tail only: a command whose
output can exceed ~30 lines runs in the background or with its output cut to
the tail, and the coordinator reads the exit code plus the last failing lines.
A verdict with no exit code is an unrun check.
Sandbox-excluded commands (bare `nix flake check`, `git push`, `gh`, `codex`)
stay standalone — no pipe — so run them with `run_in_background` and read
the tail of the output file.
A gate runner subagent (`model: sonnet`, never haiku) is reserved for
multi-command or long test suites whose failure needs diagnosis; its verdict
quotes the command, its exit code and the last failing lines.
The coordinator keeps reading `git diff --stat` and the hunks it must judge.

## Verify loop (Opus 5.5 self-verify; external detector on high-stakes)

After EVERY execute wave, verify — depth scaled to blast-radius:
1. Machine gate FIRST (free): parse / typecheck / lint / tests, output read in
   the tail (Coordinator context above). Never spend a
   model to find what a compiler finds.
2. Read the execute summary AND the actual diff (`git diff --stat` + the diff
   of touched files). Never trust the summary alone.
3. Opus 5.5 coordinator self-verifies each acceptance criterion against the real
   diff (fresh-context adversarial pass — Opus 5.5's strength) by running the
   command the AC names; an AC with no command and no `[lecture] — reason` or
   `[deferred to user]` marker is non prouvé, never self-verified green.
4. HIGH-STAKES ONLY (irreversible / security / architecture / prod): the
   external detector (`-e`) over diff + ACs, findings triaged by evidence;
   no usable external verdict → the Fable fallback pass (HIGH-STAKES.md).
5. Issues found → CORRECTIONS list (persisted): one line per issue —
   `file: problem → expected fix`. The coordinator re-briefs an Opus 5.5
   implementer (`model: opus`) with a SHARPER brief each round (root cause,
   exact files/lines, expected end state, exact command that must pass), then
   re-verifies the new diff.
   Correction rounds always run on `model: opus`, whatever model ran the task (quick-fix on sonnet is for planned execute tasks only); the correction-budget hook denies any other model, absent included, before counting the round.
6. Loop until every acceptance criterion is green —
   an AC reported non prouvé is not green. Max 2 correction rounds,
   enforced by the correction-budget hook (PreToolUse on Agent): every
   correction brief carries the line `TTW-CORRECTION-ROUND: <run-id>`
   (<run-id> = this run's `.claude/output/ttw/` dir name) or, for a run
   outside the session's repo, its absolute `…/.claude/output/ttw/<run-id>`
   path (no spaces); never create a stub run dir in the session's repo. Any other brief
   naming 06-resolve.md or a correction round carries
   `TTW-CORRECTION-ROUND: none`. A round goes through Agent or SendMessage,
   both counted, and counts once the spawn/send is attempted, even if later
   denied. A SendMessage correction goes only to an agent that was spawned on `model: opus`; otherwise spawn a new opus agent.
   Denied (budget spent) → STOP: deliver with
   the residuals list verbatim and the failing output, or ask the user to type `ttw: +1 tour` alone on a line — the correction-grant hook (UserPromptSubmit) reads the raw prompt and grants that run one more round per prompt; a plain "continue" grants nothing. Without a grant, never run the round inline: the coordinator never grades its own work.
   Never weaken a check to make it pass, never declare success on partial
   green.

## When this applies

- Standard, Diagnosis and High-stakes. Direct is the one inline tier
  (steps/DIRECT.md): the coordinator edits a small, non-sensitive diff
  itself, guarded by the machine gate and `ttw-tier` on the real diff,
  and escalates here the moment either says no.
- In Standard/High-stakes the coordinator never grades its own work: a
  separate implementer subagent writes the diff, the coordinator verifies.
- Standard skips the analyze phase when `Files:` is already known
  (ROUTING.md): the coordinator writes 02-plan.md itself and the implementer is the first spawn.
- A plan subagent runs only when 01-analyze ran or the tier is High-stakes.
  A plan is not graded code: the implementer stays a separate agent and the coordinator verifies its diff, so writer and verifier stay separate.
- The implementer is a typed agent picked by domain, never general-purpose.
  Mapping: nix-expert for `.nix`; frontend-expert / backend-expert by file
  type; debugger for Diagnosis; quick-fix (`model: sonnet`, explicit) for
  edit-only mechanical execute tasks the plan spells out — it has no Write tool; never for a correction round, which runs on `model: opus`. No domain match →
  quick-fix if edit-only, else backend-expert.
  general-purpose bootstrap measured ~53k on decide-only probes vs ~33-38k for nix-expert/Plan runs (2026-10-07).
- Forces `-s` (save) ON: the chain of summaries is also persisted to disk so
  it survives compaction and enables manual resume from disk. Fresh context + external
  memory are two halves of the same mechanism; do not enable one without the other.

## Fan-out lives in the COORDINATOR, not the phase

Because a phase agent cannot itself spawn (depth=1), any parallel fan-out is
done by the coordinator, which then hands the synthesis to the phase agent:
- **Analyze**: coordinator spawns the parallel Explore agents (`model: haiku`), collects their
  bounded summaries, THEN spawns the analyzer agent with those summaries as
  input. The analyzer produces the Conflicts & Constraints synthesis.
- **Execute (parallel waves)**: when `-k` produced independent waves, the coordinator spawns the implementer agents per wave
  directly; there is no separate "execute agent" wrapping them.

Before spawning a wave, re-check its file-disjointness (step-02b) by
re-reading the persisted task list at the plan path — the phase summary is
bounded and does not carry the per-task `Files:` lists. The coordinator
schedules the concurrency, so it owns the collision.

## Phase brief (what the coordinator passes IN)

Every spawn must include, per Anthropic's worker-brief contract:
1. **Objective** — the one job of this phase.
2. **Output format** — the exact summary schema below (mandatory).
3. **Context** — the task + the PRECEDING phase summaries (distilled), plus
   the on-disk plan path. Never the raw transcript.
   An implementer brief names the plan path (02-plan.md) and its Files: boundary; it never pastes the plan.
4. **Tools & boundaries** — which tools to use, what NOT to touch. For an
   implementer in a `-k` wave, "what NOT to touch" is literal: pass the
   task's `Files:` list verbatim as the only paths it may write.

## Phase summary (what each phase returns OUT — fixed schema)

```
PHASE: {analyze|plan|execute|validate}
OBJECTIVE_MET: yes | partial | no
DECISIONS: {key choices made}
ARTIFACTS: {files touched / created, plan path, ACs}
OPEN_RISKS: {anything the next phase must know}
HANDOFF: {the single most important thing for the next phase}
```

Target 1-2k tokens. Distilled, not raw. Over-compression loses subtle info
whose importance only appears later — keep every decision and open risk.

## BLOCKED is a valid result

Any phase agent may return `OBJECTIVE_MET: no` plus a `BLOCKED: {reason}`
line instead of forcing an answer it does not have. The coordinator treats
BLOCKED as a result, never as a failure to paper over: read the reason, then
re-brief with what was missing, change approach, or put the question to the
user. An agent that invents a plausible completion because "no" felt like
failing costs far more than one that stops and says why. Never re-spawn an
identical brief hoping for a different answer, and never restate a BLOCKED
phase as done in the run summary.

## Completeness check (mitigates path dependency)

Before spawning phase N+1, the coordinator verifies phase N's summary has all
schema fields filled and OBJECTIVE_MET is yes/partial. An empty field →
re-spawn with a sharper brief. OBJECTIVE_MET no → handle the BLOCKED reason
per the rule above. An omission here propagates silently all the way to
validate.

## Persistence

Write each phase summary to `.claude/output/ttw/{task-id}/NN-{phase}.md` as
it completes. The coordinator's live context holds only the summaries; the
disk copy is the source of truth for manual resume.
## Commands

Privileged and sandbox command classification: `steps/COMMANDS.md`.

## Cost note

Measured 2026-10-07: a 28-line settings diff run through the full chain
cost ~220k tokens (analyze 70k + implementer 38k + coordinator ~114k).
Multi-agent work costs ≈15× a plain chat, and every subagent pays ~20k of
bootstrap before it reads a line. That is the price of clean context on
work that needs it — not a price to pay on a one-line change. Size the
run on the diff (SKILL.md tier table): Direct for the small and
non-sensitive, this chain for the rest.
