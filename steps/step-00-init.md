# Step 00: Initialize (Standard / Diagnosis / High-stakes)
<!-- effort: medium -->

YOU ARE THE COORDINATOR, not an executor. Do NOT do the analysis or coding
yourself: spawn each phase as a fresh subagent and keep only its summary,
per [ORCHESTRATION.md](ORCHESTRATION.md). Direct never reads this file —
its whole run is DIRECT.md.

- Tier, default flags and flag precedence: SKILL.md (single source).
- How to run any command (privileged, sandbox-excluded, blocked):
  [COMMANDS.md](COMMANDS.md).
- Spawn models: CLAUDE.md and the model table in ORCHESTRATION.md.
- Branch first: per the Git section of CLAUDE.md (protect-main enforces
  it) — on `main`/`master`, cut the branch before the first edit; record
  `{trunk}` = the branch you cut from.

## Initialize State

First create `00-context.md` per [step-00b-save.md](step-00b-save.md),
with 00-init `running`; then record the following in it:
- **Task**: the user's request (everything after flags)
- **Tier**: from the SKILL.md tier gate, with its `Files:` and
  `Est-lines:` lines. That `Files:` line is the pre-analyze guess and is
  never rewritten: the plan's `Files:` line carries the final list.
- **Flags**: which flags are active, resolved per SKILL.md
- **Working directory**: current project path
- **Git status**: current branch, `{trunk}`, clean/dirty, uncommitted changes
- **Baseline**: the project's own gate, run BEFORE the first edit, and its
  verdict recorded. Use the cheapest gate that still discriminates, and
  use the SAME commands step-04 will run — a gate compared against a
  different gate proves nothing. `pnpm typecheck && pnpm lint`,
  `nix flake check --no-build`, `cargo check`.
  - **Already red.** Name the failing check verbatim. The run continues,
    and at validate ONLY that named failure may be attributed to the
    baseline — every other red is this run's.
  - **Dirty tree.** A baseline taken on a dirty tree measures someone
    else's uncommitted work. Record the tree as dirty beside the verdict.
  - **Privileged.** Bare `nix flake check` runs outside the sandbox: run
    it yourself (in the background if long). Only a `sudo` command goes on
    the "Run yourself" list per COMMANDS.md, then record
    `baseline: skipped — {command} — {reason}`. A skipped baseline is an
    unknown one: nothing at validate may then be dismissed as
    pre-existing.
  - **Diagnosis mode.** The red baseline IS the subject of the run.
    Record it as the reproduction target, never as an excuse.

## Present Summary

Display a compact summary:
```
TTW initialized
Task: {description}
Tier: {tier}
Flags: {active flags}
Branch: {current branch} (trunk: {trunk})
Status: {clean/dirty}
Baseline: {gate} -> {green | red: check | skipped: reason}
```

## Save

Read [step-00b-save.md](step-00b-save.md) and execute it — unconditional:
the on-disk summary chain is an invariant, not a flag.

`-o` and `-n` are not init-time sub-steps. `-o` fires before
step-02-plan: at the end of step-01-analyze, or straight from init when
analyze is skipped (vault + knowledge graph before planning); `-n` fires
at the terminal steps: it writes the session note, THEN runs
`graphify-reindex` in the background — the only thing keeping the graph
current, so a run that skips `-n` silently degrades the next run's `-o`.

## Next Step

Consult [ROUTING.md](ROUTING.md).
