# TTW — tiered implementation workflow for Claude Code

TTW is a Claude Code skill that routes each task to a tier sized to its risk.
`SKILL.md` is a small router: it decides the tier, then tells the agent which
step files under `steps/` to read. Steps are loaded progressively, per tier,
so a typo fix never pays for the full high-stakes protocol.

## Tiers

| Mode | Default flags | When | Read |
|------|---------------|------|------|
| Direct | `-pr` | ≤ 4 files, ≤ 30 changed lines, no sensitive surface, not a bug | `steps/DIRECT.md` + `steps/COMMANDS.md` |
| Diagnosis | `-f -x -pr -o -n` | bug / error / crash / broken: reproduce first, debugger agent implements | as Standard |
| Standard / complex | `-t -f -pr -o -n` | everything else that changes files | `steps/step-00-init.md`, `steps/ORCHESTRATION.md`, `steps/ROUTING.md`, `steps/COMMANDS.md`, then steps as routed |
| High-stakes | `-t -f -x -pr -o -n -e` | the diff touches a sensitive surface | Standard list + `steps/HIGH-STAKES.md` |
| Pure research | none | zero file change | none — answer directly; no branch, no PR |

## Flags

Lowercase enables, uppercase disables.

| Flag | Disable | Description |
|------|---------|-------------|
| -q | -Q | Clarify — detected ambiguities become targeted questions (3 max) before planning |
| -x | -X | Examine — adversarial review, checklist-driven |
| -t | -T | Test — create + run tests after implementation |
| -f | -F | Test-first — a separate agent writes failing tests from the ACs before execute; read-only for the implementer |
| -2 | | Divergence — second independent implementation of the core logic, behavioral diff; high-stakes logic only |
| -p | -P | Premises — force/forbid the premises pass (runs only when -p is typed) |
| -e | -E | External verify — one cross-vendor read-only pass over the same diff |
| -pr | -PR | PR — commit + PR |
| -k | -K | Tasks — dependency breakdown |
| -v | -V | Verify — research the plan online |
| -o | -O | Obsidian — load vault context |
| -n | -N | Note — session note at end |

See `SKILL.md` for the authoritative tables.

## Usage

```
/ttw fix typo in README        # Direct
/ttw -t -pr add endpoint       # Standard + tests + PR
/ttw -q -x migrate schema      # Clarify first, then adversarial review
/ttw -e refactor auth guard    # + one cross-vendor read-only verify pass
```

## Install

```
git clone https://github.com/AlxWrtl/ttw-skill.git ~/.claude/skills/ttw
```

Or clone anywhere and symlink the folder to `~/.claude/skills/ttw`.

For `-o` / `-n`, set the placeholders in `steps/step-01b-obsidian-context.md`
and `steps/step-09b-obsidian-note.md`:

- `{vault}` — your Obsidian vault root
- `{graph-vault}` — the graphify output root

## Structure

```
.
├── README.md
├── SKILL.md              # router: tier gate, flags, contract
├── eval-suite.json
└── steps/
    ├── COMMANDS.md
    ├── DIRECT.md
    ├── HIGH-STAKES.md
    ├── ORCHESTRATION.md
    ├── ROUTING.md
    ├── step-00-init.md
    ├── step-00b-save.md
    ├── step-01-analyze.md
    ├── step-01b-obsidian-context.md
    ├── step-02-plan.md
    ├── step-02b-tasks.md
    ├── step-02c-verify.md
    ├── step-03-execute.md
    ├── step-04-validate.md
    ├── step-05-examine.md
    ├── step-06-resolve.md
    ├── step-07-tests.md
    ├── step-08-run-tests.md
    ├── step-09-finish.md
    └── step-09b-obsidian-note.md
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

MIT
