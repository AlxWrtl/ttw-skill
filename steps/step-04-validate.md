# Step 04: Validate

YOU ARE A VALIDATOR, not an implementer. Do NOT add new features.

Per ORCHESTRATION.md: the COORDINATOR (Opus 5.5) owns this step and its
verdict; it runs the machine gate with output read in the tail
(ORCHESTRATION.md "Coordinator context"). Machine
gate first (parse/typecheck/lint/tests — free), then Opus 5.5 self-verifies the
real diff against the ACs. Before declaring green,
re-validate the premises whose truth can have DRIFTED since the plan
(environment state: rows in a table, deployed version, branch) — a cheap
re-read, never a write-effect command. Input: the plan + execute phase
summaries AND the real diff. Produce the validate phase summary schema and
persist it.

If `-e` is active, run the external cross-vendor pass after the machine
gate: `ttw-verify-external --base {trunk} --acs
.claude/output/ttw/{task-id}/02-acs.md --out
.claude/output/ttw/{task-id}/04-external-verify.json`. `{trunk}` is the
branch this run cut from, not a constant — read it, never assume `master`.
The ACs file is the one step-02-plan persisted alone; passing the plan
instead hands the reviewer the rationale the whole pass exists to withhold.
It is a subprocess, not an Agent spawn,
and its brief is the same bounded one Fable gets — diff plus ACs, never
the rationale. Arbitrate its findings yourself:
an external BLOCKED verdict is an unrun check, never a green one.
Write `.claude/output/ttw/{task-id}/04-external-triage.md`: one line per
finding — `id | file:line | confirmed | evidence` or
`id | file:line | dismissed | reason` — then a tally
`confirmed=N unique=U dismissed=D`, where unique = confirmed and found by
no other pass in this run. The tally is what the metrics probe reads as
B5 (did the pass change the run); a hook recalls a missing one when
04-validate.md is written.

No usable external verdict on high-stakes (BLOCKED, or -E typed) → spawn the Fable fallback
(ORCHESTRATION.md) and record its cause in 04-validate.md.

## Verification Checklist

1. **Acceptance Criteria**: go through each AC of 02-acs.md and run the
   command it names; its exit code and output decide, not a reading of the code.
   An AC with no command and no `[lecture] — reason` / `[deferred to user]` marker is
   reported **non prouvé** and blocks a green verdict: send it back to plan
   for a command, never pass it on a reading of the diff.
   `[lecture]` and `[deferred to user]` ACs are listed apart in
   `04-validate.md` (lecture: what was read and the verdict; deferred: the
   command for the user), never counted as proven by a command.
   `python3 -I ~/.claude/hooks/ttw-metrics-probe.py --acs .claude/output/ttw/{task-id}/02-acs.md`
   prints the class of each AC.

2. **Build Check**: run only the SAFE checks (see ORCHESTRATION.md), and
   run the SAME commands step-00 recorded as the baseline:
   - TypeScript: typecheck (`pnpm typecheck` or `npx tsc --noEmit`)
   - Lint: `pnpm lint` or equivalent
   - Build: `pnpm build` or equivalent
   - Nix: `nix-instantiate --parse` (safe); run `darwin-rebuild build`
     (no sudo) yourself. Only `sudo darwin-rebuild switch` is privileged:
     mark such ACs **deferred to user** and add the command to the
     "Run yourself" list.

3. **Integration Check**: verify that:
   - All imports resolve
   - No circular dependencies introduced
   - Types are consistent across boundaries

4. **Quick Smoke Test**: if there's a dev server, verify it starts without errors

5. **Test-first integrity (`-f`)**: diff the test files against the version
   the test-author agent produced. Any edit, deletion or weakened assertion
   by the implementer is a finding — the tests were the spec, not a draft.

6. **Baseline comparison**: compare every result to the step-00 baseline
   verdict, before opening a correction round. Red at init and red now →
   report `pre-existing (baseline red at init)`. Green at init and red now
   → finding. Baseline recorded as skipped → no red may be dismissed at
   all; say so explicitly rather than guessing which side it came from.
7. **Docs coverage**: every library API call, config option, signature or
   version pin the real diff adds must have its source in `AC-docs:` (02-acs.md).
   One without is a finding, sent back to plan, never waived; `Docs: n/a (text)`
   over a diff that adds one is a finding too. AC-docs is an AC, so the `-e`
   pass and any Fable pass check it with no extra brief.
8. **Tier re-check**: run `ttw-tier --base {trunk}` on the real diff and
   record its line (`tier=… lines=… files=… reasons=…`) verbatim in
   `04-validate.md`. `tier=high` on a Standard run escalates to
   High-stakes before the terminal router: keep the branch and the diff,
   add the High-stakes default flags (`-x`, `-e`), read
   `steps/HIGH-STAKES.md`, record the cause in `00-context.md`, then run
   the `-e` pass above.

## Divergence check (`-2`)

If `-2` ran, a second independent implementation of the core pure functions
exists (ORCHESTRATION.md). Execute BOTH over generated inputs — edge values,
empty, boundaries, a spread of random cases — and diff the behaviour. Any
divergence is a finding: name the input, both outputs, and which one the ACs
say is right. Never settle it by preferring the nicer code, and never call it
a tie. Once the diff is clean, discard the second implementation: it was a
check, never a deliverable.

## If any AC is not met:
Go back and fix it. Do not proceed until all ACs pass. A non prouvé AC is not met.

## If save mode (-s):
Update progress in context file.

## Next Step

Apply the shared terminal router in [ROUTING.md](ROUTING.md) (this is
04-validate → rule 1 `-t` is in play).
