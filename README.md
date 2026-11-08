# TTW — tiered implementation workflow for Claude Code

TTW is a Claude Code skill that every file-modifying task routes through.
`SKILL.md` is a router: it sizes the run into a tier from the diff the task
will produce, never from the words of the brief, then tells the agent which
step files under `steps/` to read. Steps load progressively, per tier, so a
typo fix never pays for the full high-stakes protocol.

## At a glance

How a run flows, from the tier gate to the end. Yellow = gates, tiers and
decisions (hexagons), blue = steps, dashed purple = runs only under its
flag, green pill = end of a run; grey panels group the Direct lane and the
phase pipeline. Each part is detailed in [How a run works](#how-a-run-works).

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="docs/overview-dark.svg">
  <img alt="TTW run overview: the tier gate sends a task to Pure research, Direct, Diagnosis, High-stakes or Standard. Direct runs inline (branch, baseline gate, red test, edit, machine gate, ttw-tier re-check, self-check, ship) and escalates to 02-plan when a path leaves Files:, the tier re-check fails or an AC is not met. Diagnosis, High-stakes and Standard run the pipeline 00-init, 01-analyze, 01b-obsidian, 02-plan, 02c-verify, test-author, 03-execute, 04-validate, then the terminal router to 07/08 tests, 05-examine, 06-resolve, 09-finish, 09b-obsidian-note and COMPLETE." src="docs/overview-light.svg">
</picture>

## Install

```
git clone https://github.com/AlxWrtl/ttw-skill.git ~/.claude/skills/ttw
```

Or clone anywhere and symlink the folder to `~/.claude/skills/ttw`.

For `-o` / `-n`, set the placeholders in `steps/step-01b-obsidian-context.md`
and `steps/step-09b-obsidian-note.md`:

- `{vault}` — your Obsidian vault root
- `{graph-vault}` — the graphify output root

## Usage

```
/ttw [flags] <task>
```

```
/ttw fix typo in README        # Direct
/ttw -t -pr add endpoint       # Standard + tests + PR
/ttw -q -x migrate schema      # Clarify first, then adversarial review
/ttw -e refactor auth guard    # + one cross-vendor read-only verify pass
```

Flag precedence, highest first:

1. Typed by the user — lowercase forces ON, uppercase forces OFF
   (`-PR` cancels the default `-pr`, `-T` the default `-t`).
2. The tier's default set.
3. OFF.

Never on by default, must be typed: `-q`, `-2`, `-p`, `-k`, `-v`. `-e` is a
default of High-stakes only. `-f` is a default of every tier except Direct,
which runs its own red test inline. The tier and the resolved flags are
written in `00-context.md`. An unknown flag is rejected and the valid flag
list is printed.

Pure questions or research with zero file modification do not use the
workflow: they are answered directly.

## Tier gate

Run first, before any edit. The coordinator writes two lines from the files
it will touch:

```
Files: {path}, {path}
Est-lines: {changed lines, added + removed}
```

Precedence, first match wins: Diagnosis > High-stakes > Direct > Standard.

| Mode | Default flags | When | Reads |
|------|---------------|------|-------|
| Direct | `-pr` | ≤ 4 files, ≤ 30 changed lines, no sensitive surface, not a bug | `steps/DIRECT.md` + `steps/COMMANDS.md` |
| Diagnosis | `-f -x -pr -o -n` | bug / error / crash / broken: reproduce first, debugger agent implements | as Standard |
| Standard / complex | `-t -f -pr -o -n` | everything else that changes files | `step-00-init.md`, `ORCHESTRATION.md`, `ROUTING.md`, `COMMANDS.md`, then steps as routed |
| High-stakes | `-t -f -x -pr -o -n -e` | the diff touches a sensitive surface | Standard list + `steps/HIGH-STAKES.md` |
| Pure research | none | zero file change | none — answer directly; no branch, no PR |

**Sensitive surface** — judged on the content of `Files:` and of the real diff:

- permission rules: `allow` / `deny` / `ask` entries, sandbox keys;
- secrets: `.env*`, `secrets/`, `*token*` / `*key*` / `*cert*` paths;
- a hook's guard decision path: the branch that denies or blocks
  (`deny`, `block`, `permissionDecision`, `exit 2`);
- irreversible work: data deletion, migrations, force-push or history rewrite;
- a production deploy.

Not high: adding or removing a notification hook or a settings value. A
brief that says "hook" or "settings" is not high-stakes; a diff that edits a
deny rule is.

The tier is re-checked on the real diff by `ttw-tier --base {trunk}`
(`{trunk}` = the branch the run cut from). A higher tier than planned
escalates, keeping the branch and the diff.

## Flags

Lowercase enables, uppercase disables.

| Flag | Disable | Description |
|------|---------|-------------|
| `-q` | `-Q` | Clarify — detected ambiguities become targeted questions (3 max) before planning. Never default. |
| `-x` | `-X` | Examine — adversarial review, checklist-driven. Default in Diagnosis and High-stakes. |
| `-t` | `-T` | Test — create + run tests after implementation. Default in Standard and High-stakes. |
| `-f` | `-F` | Test-first — a separate agent writes failing tests from the ACs before execute; read-only for the implementer. Default outside Direct. |
| `-2` | | Divergence — second independent implementation of the core logic, behavioural diff; high-stakes logic only. Never default. |
| `-p` | `-P` | Premises — force/forbid the Fable premises pass; it runs only when `-p` is typed. |
| `-e` | `-E` | External verify — one cross-vendor read-only pass (Codex/GPT) over the same diff. Default in High-stakes, opt-in elsewhere. |
| `-pr` | `-PR` | PR — commit + PR. Default in every tier that changes files. |
| `-k` | `-K` | Tasks — dependency breakdown into file-disjoint waves. Never default. |
| `-v` | `-V` | Verify — research the plan online; must trace at least one query or state why none. Never default. |
| `-o` | `-O` | Obsidian — load vault context; must cite the notes it read or state the vault is silent. Default outside Direct. |
| `-n` | `-N` | Note — session note at end. Default outside Direct. |

## Invariants (every tier, not flags)

- **Branch first.** On `main`/`master`, cut a branch before the first edit;
  on a feature branch, stay. `{trunk}` = the branch cut from.
- **On-disk persistence.** The run's files live under
  `.claude/output/ttw/{task-id}/` (`{task-id}` = zero-padded number + slug,
  e.g. `01-user-auth`). Each phase summary is written to `NN-{phase}.md` as
  it completes; the disk copy is the source of truth for a manual resume.
- **`00-context.md`** holds the task, tier, `Files:` / `Est-lines:`, flags,
  branch and trunk, git status and the baseline verdict, plus:
  - a `## Progress` table, kept live — a step goes `running` before its work
    and `complete` when it ends, one edit each. `ttw-board` reads it and puts
    a run in the column of its first `running` (else `pending`) step;
  - a `## Tokens` table — one row per spawned agent, copied from its usage
    report, plus the coordinator context at start and end. Absent values are
    written `unknown`. These are context sizes, never presented as cost.
- **Explicit `model` on every spawn** — a spawn never inherits the session model.

## How a run works

### Direct (`steps/DIRECT.md`)

The one inline tier: the coordinator edits the files itself and spawns no
agent. The safeguard is the machine gate plus the real-diff re-check. Default
flags are `-pr` only; typed `-t`, `-x` or `-e` are honoured after the gate.

1. **Branch** — cut per the invariant, record `{trunk}`.
2. **Context file** — write `00-context.md` (Direct variant with
   `Baseline:`, `Test-first:`, ACs, Progress rows `03-edit` / `04-gate` / `08-ship`, Tokens).
3. **Baseline gate** — run the project's own gate before the first edit, the
   same command step 6 runs. Already red → name the failing check; only it may
   later be blamed on the baseline. Only a `sudo` gate is skipped.
4. **Red first** — write the test, check or canary that proves the AC, run it,
   record the red line. No red, no edit. Exemption: a diff with no behaviour
   to test (doc, comment, formatting), written `Test-first: skipped — {reason}`.
5. **Edit inline**, only the paths in `Files:`. Any other path is a re-tier.
6. **Machine gate** — re-run the baseline command; read exit code and tail.
7. **Tier re-check** — `ttw-tier --base {trunk}`. Anything but `tier=direct`,
   or a red gate, escalates to Standard (High-stakes on `tier=high`): branch and
   diff kept, cause recorded as `Escalated: …`, then step-02-plan with that
   tier's default flags. The first implementer brief after an escalation runs on
   `model: opus` and carries `TTW-CORRECTION-ROUND: {run-id}`.
8. **Self-check** — read `git diff`, one line per AC: `AC — met | not met — evidence`.
   A not-met AC is an escalation, not a second inline edit.
9. **Ship** — commit, push, PR, merge per `COMMANDS.md`; `-PR` leaves the change
   uncommitted on the branch. Present tier, files, gate verdict, `ttw-tier` line, PR link.

Direct never: spawns an implementer, analyzer or verifier; skips `ttw-tier`
or trusts `Est-lines:` over it; edits outside `Files:` without re-tiering;
corrects a red gate inline; edits before a red test (unless nothing to test).

### Standard, Diagnosis, High-stakes (`steps/ORCHESTRATION.md`)

- **Coordinator** = the main `/ttw` run, the only agent allowed to spawn
  (depth capped at 1). It does not analyze or code itself: it spawns a phase
  agent, verifies and persists its summary, then spawns the next phase.
  Any parallel fan-out (Explore agents, `-k` waves) is done by the coordinator.
- **Phase agent** = a fresh subagent with a self-contained brief, returning
  only a bounded summary (~1-2k tokens); its raw context is discarded.
- **Separation** — the coordinator never grades its own work: a separate
  implementer writes the diff, the coordinator verifies it. The implementer is
  a typed agent by domain (nix-expert, frontend-expert, backend-expert,
  debugger for Diagnosis, quick-fix for spelled-out mechanical edits), never
  general-purpose.

Model routing, summarised:

| Phase | model |
|-------|-------|
| Analyze fan-out (Explore / codebase-navigator) | `haiku`, read-only search only; enforced by the research-model hook |
| Analyze synthesis, plan (`subagent_type: Plan`) | `opus` (effort high/max) |
| Execute | `sonnet` for spelled-out or mechanical tasks, `opus` when a design or logic decision is needed |
| Correction round | `opus`, always (hook-enforced) |
| Run tests, gate runner for long suites | `sonnet` |
| Self-verify, Direct | coordinator inline |
| Fable fallback verify / premises (`-p`) | `fable`, read-only, bounded verdict |
| External verify (`-e`) | codex CLI subprocess, read-only |

Haiku locates, maps and audits; a "why does X happen" question goes to a
`Plan` agent on `opus`. Prefer dialing effort over switching models.

**Phase brief** (in): objective; the output schema; context = the task plus
preceding distilled summaries and the on-disk plan path (never the raw
transcript; an implementer brief names `02-plan.md` and its `Files:`
boundary); tools and boundaries.

**Phase summary** (out), fixed schema:

```
PHASE: {analyze|plan|execute|validate}
OBJECTIVE_MET: yes | partial | no
DECISIONS: {key choices made}
ARTIFACTS: {files touched / created, plan path, ACs}
OPEN_RISKS: {anything the next phase must know}
HANDOFF: {the single most important thing for the next phase}
```

`BLOCKED: {reason}` with `OBJECTIVE_MET: no` is a valid result: the
coordinator re-briefs with what was missing, changes approach, or asks the
user — never re-spawns an identical brief, never restates it as done. Before
spawning phase N+1 it checks every field of phase N is filled.

### Step-by-step pipeline

- **step-00-init** — coordinator setup: creates `00-context.md` first, records
  task, tier, flags, cwd, git status, `{trunk}`, and runs the baseline gate
  (same commands step-04 will run). Red baseline: only the named check may be
  attributed to it later; dirty tree noted; a `sudo` baseline is skipped and
  then nothing may be dismissed as pre-existing; in Diagnosis the red baseline
  is the reproduction target. Prints a compact summary.
- **step-00b-save** — creates `.claude/output/ttw/{task-id}/` and the
  `00-context.md` template with the Progress and Tokens tables. Unconditional.
- **step-01-analyze** — runs when the files to touch are unknown; Diagnosis
  always runs it. The coordinator fans out haiku Explore agents (count scaled
  to scope), then synthesizes or spawns an analyzer. Output: requirements,
  affected files, style sources, conventions, installed versions from
  lockfiles, dependencies, risks, and a required Conflicts & Constraints
  section. Every probe reports `total, matched, sample`; a zero ships with a
  positive control. Under `-q` it lists ambiguities; the coordinator asks them
  in one question set (3 max) before the planner is spawned. Writes `01-analyze.md`.
- **step-01b-obsidian-context** (`-o`) — read-only: reads the project note,
  the 3 most recent sessions, decision titles, then queries the knowledge
  graph by domain keywords (freshness probe first; stale graph still queried
  and a background reindex fired; errors degrade to recency only, never block).
  Must cite every note read as a wikilink or say `vault silent on this topic`.
  Writes `01b-obsidian-context.md`.
- **step-02-plan** — the coordinator writes the plan itself when analyze was
  skipped (not High-stakes); otherwise a `Plan` agent on `opus` drafts it.
  Contents: Premises (target restated, business rules, environment state,
  excluded scope), each sourced and tagged `[M]` measured or `[I]` inferred;
  ordered tasks; ACs that each name the command proving them (or
  `[lecture] — reason` / `[deferred to user]`); testing strategy under `-t`;
  risks; a categorical `Docs:` line (`{lib}@{installed version} — {source}
  (rung N)`, or `Docs: n/a (text)`). The ACs alone go to `02-acs.md`, the only
  spec a verifier is handed. The coordinator presents Premises first; in
  High-stakes or under `-q` it waits for approval. A user contradiction is
  persisted, dated, before execute proceeds. The plan records which verify
  passes will run.
- **step-02b-tasks** (`-k`) — tasks with `Files:`, `Depends:`, `Agent:`,
  `Verify:`, grouped in waves that must be pairwise file-disjoint (collision →
  split the wave). `Files:` values are concrete paths, never globs or `TBD`.
- **step-02c-verify** (`-v`) — online check of the plan: deprecations,
  alternatives, CVEs, rung-5 `Docs:` entries. Must trace at least one query
  and URL or state `nothing to verify online because …`. Writes `02c-verify.md`.
- **step-03-execute** — a typed implementer follows the plan: one todo at a
  time, plan's style, no API outside the `Docs:` line (`UNSOURCED_API`), no
  file outside its `Files:` list (`OUT_OF_SCOPE_FILE`). Test-first tests are
  read-only.
- **step-04-validate** — coordinator-owned: machine gate first, ACs run by
  their command, drifted premises re-read, build/integration/smoke checks,
  test-first integrity, baseline comparison, docs coverage, and
  `ttw-tier --base {trunk}` recorded verbatim (`tier=high` on Standard
  escalates to High-stakes, adding `-x` and `-e`). Under `-e` it runs
  `ttw-verify-external` and writes `04-external-triage.md` with a
  `confirmed=N unique=U dismissed=D` tally. Under `-2` it diffs both
  implementations over generated inputs.
- **step-05-examine** (`-x`) — one blind code-reviewer (`opus`) gets the spec
  and the real diff, never the rationale, and reports every checklist box:
  premise, scope, lying tests, probe denominators, sourced labels, `[M]`/`[I]`
  tags, races, boundaries, error handling, shared state, conventions; security
  and data-integrity boxes added on signal. Findings sorted
  Critical/Important/Minor and triaged in `05-examine.md`.
- **step-06-resolve** — runs on Critical/Important findings: presents them by
  severity and waits; per finding Fix / Skip / Discuss; re-runs checks after fixing.
- **step-07-tests** (`-t`) — writes tests following existing project
  patterns; when test-first ran, only extends coverage in its own files.
- **step-08-run-tests** — test loop, maximum 10 attempts, then remaining
  failures go to the user.
- **step-09-finish** (`-pr`) — commit, push, PR (Summary / Changes / Testing /
  Acceptance Criteria), shown in the transcript, then squash-merge once the
  gate is green, return to `{trunk}`, pull, delete the local branch.
- **step-09b-obsidian-note** (`-n`) — writes a session note
  `YYYY-MM-DD - {slug}.md` under the project's `sessions/` folder (dedup by
  slug, never overwrites), optionally links it from the project note, then
  fires `graphify-reindex` in the background — the only graph write path.

### Routing (`steps/ROUTING.md`)

The single place transitions are decided. Linear spine:

| From | Next |
|------|------|
| 00-init | Diagnosis: 01-analyze. Standard / High-stakes: 01-analyze if files unknown, else 01b if `-o`, else 02-plan |
| Direct escalation | 02-plan with Standard (or High-stakes) flags; branch and diff kept |
| 01-analyze | 01b if `-o`, else 02-plan |
| 01b-obsidian | 02-plan |
| 02-plan | 02c-verify if `-v`, else 03-execute |
| 02c-verify | 03-execute |
| 03-execute | 04-validate |

Shared terminal router, applied after 04-validate, 08-run-tests, 05-examine
and 06-resolve — first match wins:

1. From 04-validate only: `-t` → 07-tests.
2. `-x` and examine not yet run → 05-examine.
3. Critical/Important findings and resolve not yet run → 06-resolve.
4. `-pr` → 09-finish.
5. `-n` → 09b-obsidian-note.
6. Else → COMPLETE.

07-tests always goes to 08-run-tests, which applies the router skipping rule 1.
09-finish goes to 09b if `-n`, else COMPLETE. Invariant: with both `-pr` and
`-n`, the PR comes before the note, and 09-finish's tail is the only path to 09b.

### Test-first (`-f`)

Between plan approval and execute, a separate test-author agent (`opus`)
whose only input is the ACs writes tests that fail against the current code,
and nothing else. Those files are read-only for the implementer; editing,
deleting or weakening one is a finding at validate. In Diagnosis the failing
test is the reproduction. Direct spawns no test-author: it writes its own red
test inline. A diff with no behaviour to test skips it with the reason in
`00-context.md`.

### Verify loop

After every execute wave:

1. Machine gate first (parse / typecheck / lint / tests), output read in the tail.
2. Read the execute summary and the actual diff — never the summary alone.
3. The coordinator verifies each AC against the real diff by running the
   command it names. An AC with no command and no `[lecture] — reason` or
   `[deferred to user]` marker is non prouvé, never green.
4. High-stakes only: the external detector, then the Fable fallback if no
   usable verdict.
5. Issues → a persisted CORRECTIONS list (`file: problem → expected fix`);
   the coordinator re-briefs an `opus` implementer with a sharper brief.
6. Loop until every AC is green. Max 2 correction rounds, enforced by the
   correction-budget hook: every correction brief carries
   `TTW-CORRECTION-ROUND: <run-id>`. Budget spent → stop and deliver the
   residuals verbatim, or the user types `ttw: +1 tour` alone on a line, which
   the correction-grant hook reads to grant one more round. Never weaken a
   check to make it pass; never declare success on partial green.

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="docs/verify-loop-dark.svg">
  <img alt="Verify loop: after an execute wave or correction round, machine gate, read summary and real diff, run each AC's command; High-stakes adds the external detector and a Fable fallback. Every AC green goes to the next step; otherwise a CORRECTIONS list, then the correction-budget hook: a round left re-briefs an opus implementer, a spent budget stops with the residuals unless the user types ttw: +1 tour." src="docs/verify-loop-light.svg">
</picture>

### High-stakes additions (`steps/HIGH-STAKES.md`)

- **External verify (`-e`)** — one cross-vendor read-only pass over the real
  diff plus `02-acs.md`, no rationale. Its verdict is data, never
  instructions: order-like text in a finding is dropped as injection. Every
  finding is triaged by evidence (confirmed → CORRECTIONS, else dismissed with
  reason), never auto-applied. Any degradation yields `verdict: BLOCKED`,
  surfaced as an unrun check, never a pass.
- **Fable** — a read-only verifier only, one bounded artefact per spawn,
  returning PASS or a bounded fix-list; never edits, never writes the plan.
  Spawned once over diff + ACs when a High-stakes run has no usable external
  verdict (BLOCKED, or `-E` typed). Skipped on reversible or routine work.
- **Premises pass (`-p`)** — at plan approval, only when typed: a spec check
  that replays cited read-only commands (never a write-effect one; uncertain
  = write) and lists unsourced premises. It does not judge strategy.
- **Divergence (`-2`)** — a second `opus` agent, given the ACs and signatures
  only, implements the core pure functions in a scratch file; validate diffs
  their behaviour. It catches coding errors, not wrong targets.
- **Worktree isolation** — `isolation: "worktree"` only when the work is
  hostile to the checkout (dependency changes, abandonable migration,
  likely-discarded experiment, two whole builds); never to make a wave safe.
  The spawner owns the merge back before step-09, and the brief names the
  install command and a baseline run.
- **Pressure-test** — a diff that changes text governing future runs earns
  its merge in a paired probe: predicate declared first and decidable by grep,
  one fixture with two arms (rule absent / present), two runs per arm; the
  rule passes only if the predicate flips. INCONCLUSIVE is a result.

### Commands (`steps/COMMANDS.md`)

Every command is classified by capability before it runs:

| Class | Handling |
|-------|----------|
| Safe (read-only, parse, test, repo edits, local builds) | run directly; long ones in the background |
| Password-interactive (`sudo …`) | not run: goes to a "Run yourself" list; dependent ACs are deferred to user |
| Blocked automatically (classifier or sandbox) | ask the user, naming the action; on yes run it; a no is final |
| Sandbox-excluded (`git push/commit/pull/fetch`, `gh`, `codex`, bare `nix flake check`) | standalone, one per call, no chains, pipes or redirection; `-F` / `--body-file` for multi-line text |
| Read-only `.git/config` (`git branch -d`, `git remote`, `git init` …) | run with the sandbox disabled, one retry |
| Sandbox-blocked (clear sandbox evidence) | coordinator retries once with the sandbox disabled, behind a confirmation box; phase agents only surface it |
| In doubt | ask the user |

Ship (`-pr`): commit on the run's branch, push, open the PR, squash-merge
once checks are green. Never commit to `main`/`master`; never force-push or
rewrite pushed history without asking.

## Error handling

- A failed step (blocked task, red build/tests): stop, surface the raw error
  and failing criteria; never fabricate success or push past red.
- Missing prerequisite (no git repo, no test framework): warn and fall back
  (manual verification / skip the gated step), never swallow it.
- Unknown flag → rejected, valid flag list printed.
- Re-running is safe: branch/commit steps are no-ops when applied, and the
  summaries under `.claude/output/ttw/` let a run resume by hand.

## Structure

```
.
├── README.md
├── LICENSE
├── SKILL.md                          # router: tier gate, flags, contract
├── eval-suite.json                   # skill eval cases
├── docs/                             # README diagrams
│   ├── overview.d2, verify-loop.d2   # D2 sources
│   ├── palette.d2, dark.d2           # light palette, dark override
│   └── *-light.svg, *-dark.svg       # rendered, committed
└── steps/
    ├── COMMANDS.md                   # command classes and ship shapes
    ├── DIRECT.md                     # the whole Direct run
    ├── HIGH-STAKES.md                # -e, Fable, -p, -2, worktree, pressure-test
    ├── ORCHESTRATION.md              # coordinator, models, briefs, verify loop
    ├── ROUTING.md                    # every step transition
    ├── step-00-init.md               # init + baseline
    ├── step-00b-save.md              # run dir + 00-context.md
    ├── step-01-analyze.md            # exploration + conflicts
    ├── step-01b-obsidian-context.md  # vault + graph context (-o)
    ├── step-02-plan.md               # premises, tasks, ACs, Docs line
    ├── step-02b-tasks.md             # file-disjoint waves (-k)
    ├── step-02c-verify.md            # online plan check (-v)
    ├── step-03-execute.md            # implementer rules
    ├── step-04-validate.md           # gate, ACs, tier re-check
    ├── step-05-examine.md            # blind adversarial review (-x)
    ├── step-06-resolve.md            # fix / skip / discuss findings
    ├── step-07-tests.md              # write tests (-t)
    ├── step-08-run-tests.md          # test loop
    ├── step-09-finish.md             # commit, PR, merge (-pr)
    └── step-09b-obsidian-note.md     # session note (-n)
```

`docs/*.d2` are the diagram sources (D2 0.9.0, ELK layout). Regenerate
both themes from `docs/` in bash with:

```
E='--elk-nodeNodeBetweenLayers 36 --elk-edgeNodeBetweenLayers 16 --elk-padding [top=32,left=24,bottom=48,right=24]'
for f in overview verify-loop; do d2 $E $f.d2 $f-light.svg; cat $f.d2 dark.d2 | d2 $E - $f-dark.svg; done
```

## Companion tools

The steps call tools this repo does not ship:

- `ttw-tier` — re-checks the tier on the real diff
- `ttw-board` — run board reading the Progress table
- `ttw-verify-external` — cross-vendor (Codex) verify for `-e`
- hooks — `require-ttw` gate, `ttw-flags`, correction-budget / correction-grant,
  research-model, `ttw-metrics-probe`
- `graphify` and `enquire` MCP servers — vault context for `-o`
- `claude-dispatch`, `jev-locate`, `libdocs`

They live in the author's private configuration, which is not public.
Without them, adapt or skip the steps that call them; `SKILL.md` asks the
agent to warn and fall back on a missing prerequisite, not to fail silently.

## License

MIT — see [LICENSE](LICENSE).
