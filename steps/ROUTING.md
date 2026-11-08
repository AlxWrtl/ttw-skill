# TTW Routing Table — single source of truth

Every step's "Next Step" section says "consult ROUTING.md". Do NOT duplicate
transition logic inside step files. This table is the only place that decides
where to go next. Edit transitions HERE, nowhere else.

Direct does not use this table until it escalates: its whole run is
steps/DIRECT.md.

## Linear spine

| From | Next (unconditional) |
|------|----------------------|
| 00-init | Diagnosis: 01-analyze always (reproduction first). Standard / High-stakes: 01-analyze IF the files to touch are unknown, else 01b-obsidian IF `-o`, else 02-plan |
| Direct escalation | 02-plan with the Standard (or High-stakes) flags; branch and diff kept, cause in 00-context.md |
| 01-analyze | 01b-obsidian IF `-o`, else 02-plan |
| 01b-obsidian | 02-plan |
| 02-plan | 02c-verify IF `-v`, else 03-execute |
| 02c-verify | 03-execute |
| 03-execute | 04-validate |

## Post-validate / post-tests / post-resolve — shared terminal router

Steps 04-validate, 08-run-tests, 05-examine, 06-resolve all end by applying
THIS ordered router. Take the FIRST matching rule:

1. From 04-validate ONLY: IF `-t` (test) → 07-tests.
2. IF `-x` (examine) AND examine not yet run → 05-examine.
3. IF 05-examine produced Critical/Important findings AND resolve not yet run → 06-resolve.
4. IF `-pr` (pull request) → 09-finish.   ← takes precedence over `-n`
5. IF `-n` (note) → 09b-obsidian-note.
6. Else → COMPLETE (present summary).

## 07-tests / 08-run-tests

- 07-tests → 08-run-tests (always).
- 08-run-tests → apply the shared terminal router above (skip rule 1).

## 09-finish

- 09-finish → 09b-obsidian-note IF `-n`, else COMPLETE.

## INVARIANT

When both `-pr` and `-n` are set, rule 4 (pr) fires before rule 5 (note), so
09-finish is reached first; 09-finish's own tail is then the ONLY path to
09b. Do not reorder rules 4/5 without updating 09-finish.
