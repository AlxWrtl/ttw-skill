---
name: ttw
description: "Universal task workflow (TTW) — EVERY task that modifies files routes through TTW, in any project, any size or type: feature, fix, bug, refactor, config, docs. A tier gate sizes the run from the diff: Direct (inline), Diagnosis, Standard, High-stakes. Not for pure questions or research with zero file modification."
---

# TTW: tiered implementation workflow

This file is the router. Read ONLY the file for your tier — every other
step file stays unread unless your tier's route reaches it.

## Tier gate (run FIRST, before any edit)

The coordinator first writes two lines, from the files it will touch:

```
Files: {path}, {path}
Est-lines: {changed lines, added + removed}
```

Tier is decided on the diff, never on the brief's words. A brief that says
"hook" or "settings" is not high-stakes; a diff that edits a deny rule is.
Precedence, first match wins: Diagnosis > High-stakes > Direct > Standard.

| Mode | Default flags | When | Read |
|------|---------------|------|------|
| Direct | `-pr` | ≤ 4 files, ≤ 30 changed lines, no sensitive surface, not a bug | `steps/DIRECT.md` + `steps/COMMANDS.md` |
| Diagnosis | `-f -x -pr -o -n` | bug / error / crash / broken: reproduce first, debugger agent implements | as Standard |
| Standard / complex | `-t -f -pr -o -n` | everything else that changes files | `steps/step-00-init.md`, `steps/ORCHESTRATION.md`, `steps/ROUTING.md`, `steps/COMMANDS.md`, then steps as routed |
| High-stakes | `-t -f -x -pr -o -n -e` | the diff touches a sensitive surface | Standard list + `steps/HIGH-STAKES.md` |
| Pure research | none | zero file change | none — answer directly; no branch, no PR |

Sensitive surface — judged on the CONTENT of `Files:` and of the real diff:
- permission rules: `allow` / `deny` / `ask` entries, sandbox keys;
- secrets: `.env*`, `secrets/`, `*token*` / `*key*` / `*cert*` paths;
- a hook's guard decision path: the branch that denies or blocks
  (`deny`, `block`, `permissionDecision`, `exit 2`);
- irreversible work: data deletion, migrations, force-push or history
  rewrite;
- a production deploy.

Adding or removing a notification hook or a settings value is NOT high.

The tier is re-checked on the real diff by `ttw-tier --base {trunk}`
(`{trunk}` = the branch this run cut from). A higher tier than planned
escalates, keeping the branch and the diff (DIRECT.md, ROUTING.md).

## Available Flags

| Flag | Disable | Description |
|------|---------|-------------|
| -q | -Q | Clarify — detected ambiguities become targeted questions (3 max) before planning |
| -x | -X | Examine — adversarial review, checklist-driven |
| -t | -T | Test — create + run tests after implementation |
| -f | -F | Test-first — a SEPARATE agent writes failing tests from the ACs before execute; read-only for the implementer; default outside Direct |
| -2 | | Divergence — second independent implementation of the core logic, behavioral diff; high-stakes logic only |
| -p | -P | Premises — force/forbid the Fable premises pass (runs ONLY when -p is typed) |
| -e | -E | External verify — one cross-vendor read-only pass (Codex/GPT) over the same diff; default in high-stakes, opt-in elsewhere |
| -pr | -PR | PR — commit + PR |
| -k | -K | Tasks — dependency breakdown |
| -v | -V | Verify — research the plan online; must trace at least one query or state why none |
| -o | -O | Obsidian — load vault context; must cite the notes it read or state the vault is silent |
| -n | -N | Note — session note at end |

Branch-first and on-disk persistence are INVARIANTS of every tier, not
options: the branch is cut per the Git section of CLAUDE.md, and the run's
files live under `.claude/output/ttw/{task-id}/`. Every spawn passes an
explicit `model` per CLAUDE.md — never inherit.

Flag precedence, highest first:
1. Typed by the user — lowercase forces ON, uppercase forces OFF
   (`-PR` cancels the default `-pr`, `-T` the default `-t`).
2. The tier's default set (table above).
3. OFF.

Never on by default — must be typed: `-q`, `-2`, `-p`, `-k`, `-v`.
`-e` is a default of High-stakes only. `-f` is a default of every tier
except Direct, which runs its own red test inline (`steps/DIRECT.md`).
State the tier and the resolved flags in `00-context.md`.

## Common Usage

```
/ttw fix typo in README        # Direct
/ttw -t -pr add endpoint       # Standard + tests + PR
/ttw -q -x migrate schema      # Clarify first, then adversarial review
/ttw -e refactor auth guard    # + one cross-vendor read-only verify pass
```

## Execution

Write `Files:` + `Est-lines:`, pick the tier, then read the files in its
Read column now and follow them.


## Input/Output Contract
- **Expects:** task description with optional flags. Example: /ttw -q -t implement user auth
- **Produces:** the change on its own branch, verified by the machine gate, shipped per -pr; Standard and High-stakes run init → analyze → plan → execute → validate.
- **Side effects:** modifies source files, optionally creates tests, commits, creates PRs.


## Scope
- **Use this skill when:** EVERY task that modifies files, in any project, any size — the tier gate sizes the run from the diff.
- **Do NOT use for:** Pure questions or research with zero file modification → answer directly, no workflow.


## Error handling

- A failed step (blocked task, red build/tests): stop, surface the raw
  error and failing criteria; never fabricate success or push past red.
- Missing prerequisite (no git repo, no test framework): warn and fall
  back (manual verification / skip the gated step), never swallow it.
- Unknown flag → reject it and print the valid flag list.
- Re-running is safe: branch/commit steps are no-ops when applied, and
  the summaries under `.claude/output/ttw/` let a run resume by hand.


## Handoffs
- Diagnosis stays INSIDE ttw — execute phase spawns the debugger agent (model: opus).
- If scope is unclear → run /discuss first, then return to ttw.
- High-stakes verify (external detector, Fable fallback, `-p` premises) → steps/HIGH-STAKES.md.

