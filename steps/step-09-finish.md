# Step 09: Finish

## Ship

Stage, commit, push, open the PR and squash-merge it per
[COMMANDS.md](COMMANDS.md) "Ship (`-pr`)" — the command shapes live there
only. Commit message: conventional (`feat:` / `fix:` …), body for a large
diff.

## Pull Request content

- **Title**: conventional format matching the commit
- **Body**: structured with:
  - ## Summary (what was done)
  - ## Changes (bullet list of key changes)
  - ## Testing (how it was tested)
  - ## Acceptance Criteria (checklist from plan)

## Before creating it
Show the PR title and body in the transcript, then create it — no approval
wait: the `-pr` default already authorizes commit, push and PR on the run's
branch. Once the gate is green, merge it yourself, then return to
`{trunk}`, pull and delete the local branch (COMMANDS.md shapes).
Force-pushing and rewriting pushed history still need the user's
explicit go. Once `gh pr create` succeeds, the ttw-metrics-probe hook
names the run files this run still lacks (02-acs.md, the examine or
external tally): write them before merging.

## COMPLETE

Present final summary:
```
TTW Complete
Branch: {branch}
Commit: {hash}
PR: {url}
Steps completed: {list}
```

## Next Step — Obsidian Note

# INVARIANT: when both -pr and -n are set, steps 04/05/06/08 route to step-09
# FIRST (pr takes precedence), so this tail is the ONLY path to step-09b in that case.
# Do not remove this check without restoring -n branches in predecessors.
If note mode (-n) is active:
  Read [step-09b-obsidian-note.md](step-09b-obsidian-note.md) and execute it.
