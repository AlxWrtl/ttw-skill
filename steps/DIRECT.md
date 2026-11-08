# TTW Direct — the one inline tier
<!-- effort: low -->

Direct is for a diff of ≤ 4 files and ≤ 30 changed lines that touches no
sensitive surface and is not a bug (SKILL.md tier table). The coordinator
edits the files itself and spawns no implementer: no analyze phase, no
plan agent, no subagent at all.

Why inline: this revokes the 2026-08-17 'no inline tier': the coordinator does grade its own diff here; the safeguard is the machine gate plus the real-diff re-check, not self-grading.
A 28-line settings diff measured ~220k tokens through the full chain; the
same change inline costs a fraction, and the two checks below are
deterministic, so they do not care who wrote the diff.

Default flags: `-pr` only; -o and -n run only when typed — a vault
read and a session note buy nothing on a one-line change. Typed `-t`,
`-x` or `-e` are honoured: they run after step 6 as in Standard.

## Steps (in order)

1. **Branch.** Per the Git section of CLAUDE.md: on `main`/`master`, cut
   `{type}/{task-id}` before the first edit; on a feature branch, stay.
   Record `{trunk}` = the branch you cut from.
2. **Context file.** Write `.claude/output/ttw/{task-id}/00-context.md`:

   ```markdown
   # TTW: {task}
   Tier: Direct
   Flags: {resolved flags}
   Branch: {branch} (trunk: {trunk})
   Files: {path}, {path}
   Est-lines: {N}
   Baseline: {gate} -> {green | red: check | skipped: reason}
   Test-first: {command} -> red: {line} | skipped — {reason}

   ## Progress
   | Step | Status | Notes |
   |------|--------|-------|
   | 03-edit | running | |
   | 04-gate | pending | |
   | 08-ship | pending | |

   ## Tokens
   | Point | Coordinator context |
   |-------|---------------------|
   | start | {n} |
   | end | {n} |
   ```

   ACs go in the same file under `## ACs`, one line each. The Progress
   table is what ttw-board reads (NN prefix → column): without it a
   Direct run sits in Init until it goes stale. One edit per change:
   04-gate goes `running` (03-edit `complete`) at step 6, 08-ship
   `running` at step 9 and `complete` once merged (`-PR`: once the
   diff is left on the branch). An escalation replaces the table with
   step-00b's.
3. **Baseline gate.** Run the project's own gate BEFORE the first edit —
   the same command step 6 will run (`pnpm typecheck && pnpm lint`,
   `nix flake check --no-build`, `cargo check`). Record its verdict.
   Already red → name the failing check; only that check may be blamed
   on the baseline later. Only a `sudo` gate is skipped (COMMANDS.md),
   and a skipped baseline dismisses nothing as pre-existing.
4. **Red first.** Before the edit, write the test, check or canary that
   proves the AC (its path goes in `Files:`), run it, and record the red
   output in `00-context.md` as `Test-first: {command} -> red: {line}`.
   Step 6 re-runs it with the gate and it must be green. A test that
   cannot fail proves nothing: no red, no edit. The one exemption is a
   diff with no behaviour to test (doc, comment, formatting), written as
   `Test-first: skipped — {reason}` (ORCHESTRATION.md "Test-first").
5. **Edit inline**, only the paths in `Files:`. A path outside `Files:`
   is not a quiet addition — it is a re-tier: go to step 7's escalation.
6. **Machine gate.** Re-run the baseline command; read the exit code and
   the tail only (ORCHESTRATION.md "Coordinator context").
7. **Tier re-check.** Run `ttw-tier --base {trunk}` and read its verdict
   line. Escalate when it is anything but `tier=direct`, OR the gate is
   red (a red gate never gets a fix inline):
   - Direct escalates to Standard (or to High-stakes when `ttw-tier`
     says `tier=high`): keep the branch and the diff, record the cause
     in `00-context.md` (`Escalated: {ttw-tier verdict | gate red: check}`),
     then go to step-02-plan with the Standard or High-stakes default
     flags. High-stakes also reads `steps/HIGH-STAKES.md`.
   - Direct never runs a correction round inline. The first implementer
     brief after an escalation runs on `model: opus` and carries `TTW-CORRECTION-ROUND: {run-id}`
     ({run-id} = this run's `.claude/output/ttw/` dir name), so the
     correction budget counts it.
8. **Self-check.** Read `git diff` of the touched files and check each AC
   against it, one line per AC: `AC — met | not met — evidence`. A not-met
   AC is an escalation (step 7), not a second inline edit.
9. **Ship.** Commit, push, PR and merge per `steps/COMMANDS.md`
   (`-pr` default; `-PR` opts out of the whole ship step — no commit, the
   change stays uncommitted on the run's branch, as in Standard). Fill the `## Tokens`
   end row, then present: tier, files, gate verdict, `ttw-tier` line,
   PR link.

## What Direct never does

- Spawn an implementer, an analyzer or a verifier.
- Skip `ttw-tier --base {trunk}`, or trust `Est-lines:` over its verdict.
- Edit a path outside `Files:` without re-tiering.
- Correct a red gate inline.
- Edit before a red test, unless the diff has no behaviour to test.
