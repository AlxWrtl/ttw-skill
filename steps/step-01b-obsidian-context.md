# Step 01b: Obsidian Context Load

YOU ARE A CONTEXT GATHERER, not a planner. Do NOT plan or implement yet.
Your job is to load project knowledge from the Obsidian vault so TTW stays aligned
with what the user already knows/decided about this project.

## Vault

Root: `{vault}` — your Obsidian vault root (e.g. `~/Vaults/MyVault`); set it once here.

## Process

1. **Load global conventions** — Read `00-Meta/CLAUDE.md` (if exists) for vault-wide rules.

2. **Detect project name** in this priority order:
   a. Explicit hint in task description: `project X`, `projet X`, or `--project X`.
   b. Current working directory basename, lowercased and kebab-cased.
   Keep both the normalized slug (for folder/file matching) and the original display casing.
   Example: `~/code/my-app` → slug `my-app`.

3. **Find the project note** (case-insensitive match):
   - `Glob` `02-Projets/*/*.md`, filter where folder-name matches slug (case-insensitive)
     OR file stem matches slug.
   - 0 matches → report "no project note found for '{slug}'", suggest creating
     it via `-n` at the end, then SKIP steps 4-6 and GO STRAIGHT TO STEP 7.
     Do NOT skip to step-02: recency retrieval is keyed BY PROJECT and has
     nothing to read, but the graph is keyed BY CONTENT — it ignores project
     folders entirely and relates entities across all of `02-Projets`. A
     missing project note does not make it mute.
     The cwd basename is NOT a reliable key for the graph. A code repo can
     carry a name that exists nowhere as a vault folder while its notes live
     under another project: measured on a repo whose notes sit under a differently named
     project folder — this early exit sent the run to step-02, and the graph,
     which held the answer, was never asked.
   - 1 match → use it.
   - 2+ matches → use `AskUserQuestion` to let the user pick; do NOT silently pick a "closest match".

4. **Read the project note** — extract:
   - Current goals / status
   - Architecture decisions already made
   - Open questions / known issues
   - Links to sub-notes (decisions, sessions, etc.)

5. **Scan recent sessions** — `Glob` pattern: `02-Projets/[project]/sessions/*.md`
   - Read the 3 most recent session notes (by filename date `YYYY-MM-DD`)
   - Extract: decisions, next steps, blockers, context that informs the current task

6. **Scan decisions** (if folder exists) — `Glob` pattern: `02-Projets/[project]/decisions/*.md`
   - Read titles + summaries only (do NOT modify files in `decisions/`)

7. **Graph relations (graphify)** — steps 4-6 cover the FRESH tail by
   recency; this step recovers the OLD relational body: notes related to
   the current task that recency retrieval is structurally blind to.

   RUNS EVEN WITH NO PROJECT NOTE (step 3, 0 matches). The query is built
   from the TASK, never from the project name — which the keyword rule in
   (b) already bans — so an undetected project changes nothing about it.
   When there is no project, this step IS the whole vault context.

   a. **Freshness probe** (cheap, no MCP): Bash
      `ls -l {graph-vault}/graphify-out/graph.json` and compare its mtime
      to the newest note under `{vault}/02-Projets`.
      - File absent → graph status `absent`: skip the rest of this step
        (recency-only report). Do NOT build the graph here — the initial
        full build is manual and long.
      - Older than the newest note → status `stale ({graph date})`:
        STILL query it (relations are durable; only the freshest notes
        are missing, and recency already covered those), and fire the
        catch-up: Bash `graphify-reindex` with `run_in_background: true`.
        NEVER wait for it, never poll it.
      - Otherwise → status `fresh`.
   b. `mcp__graphify__query_graph` with `depth` 1 (the tool defaults to 3)
      and `token_budget` 1200. `question` = 3-6 DOMAIN KEYWORDS, never a
      sentence. Ban the meta-words `session`, `décision`, `projet`, `note`
      and the project name: they match the hub notes and drag unrelated
      seeds in. Measured on this vault, same graph, same budget:
        "décisions et sessions liées au reset de mot de passe et à la
         délivrabilité email, projet X"  → 89 nodes, 22 shown, 0 EDGES
        "reset mot de passe délivrabilité email invitation"
                                              → 16 nodes, all shown, 9 edges
      The junk seeds appear in the `Start:` list itself, so this is seed
      selection, not traversal — lowering `depth` alone does NOT fix it.

      READ THE BANNER, it decides whether the call was worth anything:
      - `[!] TRUNCATED` → the answer carries NO edges at all. Edges are only
        emitted once every node fits ("Edges are never dropped once every
        node fits"), so a truncated reply drops exactly what this step came
        for. Treat it as a failed query, not a partial one: reformulate ONCE
        with fewer, sharper keywords. Still truncated → drop the graph for
        this run and say so in the status line.
        (This supersedes the old "NEVER issue a second query_graph" rule,
        written before that behaviour was measured. One reformulation is
        cheaper than a relation-free answer; a third is not.)
      - `[i] Complete answer over budget` → this is the GOOD outcome: every
        node and edge is there. It may overrun the requested budget 4-6x
        (~5-7k tokens of context). Accept it, do not try to shrink it.
   c. Optionally, at most TWO `mcp__graphify__get_neighbors` calls on the
      1-2 returned entities most central to the task.
   d. Keep only what recency did NOT already surface: related notes
      outside {project note, 3 recent sessions, decisions read}. Read at
      most 3 of them from the vault (Read tool) — the graph is the
      pointer, the note is the truth. Cite them as `[[wikilinks]]` like
      every other note read.
   e. Any MCP error, timeout, or empty/degraded response → ONE attempt
      only: set status `unavailable` and continue with recency alone.
      This step NEVER blocks and NEVER fails the run — a missing graph
      degrades to exactly the pre-graphify behavior.

## Tool routing — enquire vs graphify (never double-query)

- `mcp__enquire__*` = what the vault WRITES: find/read notes, keyword +
  semantic search, and the EXPLICIT wikilink graph (backlinks, note
  neighbors, paths between notes).
- `mcp__graphify__*` = what the vault IMPLIES: LLM-extracted entities and
  relations across note contents, thematic communities, hubs — links that
  no wikilink materializes.
- Route: "find/read notes about X", "what links to note N" → enquire.
  "how does concept X relate to concept Y", "what clusters around entity
  X" → graphify. The same question never goes to both.

### The graph's scope is what the reindex covers, NOT what the graph holds

graphify is reindexed from `02-Projets` alone (`graphify-reindex.sh`), so
that folder is the whole of its declared scope. Finding a node from
somewhere else does NOT widen it.

Measured 2026-09-09: of 2963 nodes, 2 carry a `source_file` relative to the
VAULT ROOT (`02-Projets/my-app/my-app.md`,
`04-Resources/Outils de récupération web pour agents IA.md`) instead of the
normal base relative to `02-Projets` (`client-a/…`, `my-app/…`). They are
residue from a run scoped differently, and `graphify-reindex.sh` is
incremental-only — a full rebuild is a separate manual gesture — so those
nodes will never be refreshed and never removed.

Consequence for a run: trust the declared scope, never the contents. A node
under `04-Resources` is not evidence that `04-Resources` is covered; it is a
frozen snapshot of one moment, and the note behind it may have changed or
been deleted since. Outside `02-Projets`, read the note itself.

## Output — Context Report

Produce a compact report:

```
## Obsidian Context — {project}

### Project note: [[02-Projets/{project}/{project}]]
{1-3 line summary of current project state}

### Recent sessions
- [[02-Projets/{project}/sessions/YYYY-MM-DD - title]] — {key takeaway}
- [[02-Projets/{project}/sessions/YYYY-MM-DD - title]] — {key takeaway}

### Relevant decisions
- [[02-Projets/{project}/decisions/slug]] — {one-liner}

### Graph relations (graphify)
- [[02-Projets/{project}/...]] — {relation to the task, per the graph}
- Graph status: fresh | stale ({graph date}) | absent | unavailable
  ALWAYS print this line, including when no project note was found — it is
  the only way to tell "the graph said nothing" from "the graph was skipped".

### Implications for current task
- {how this context changes/informs the plan}
- {constraints or prior choices to respect}
- {any conflict between the task and prior decisions — flag it}
```

## Rules

- Read-only. Do NOT write to the vault in this step (that's step-09b).
  The `graphify-reindex` catch-up writes only to `{graph-vault}`, never to
  the vault — firing it does not break this rule.
- Never modify `decisions/` files.
- If no project note exists, say so and suggest creating one via `-n` flag at the end.
- Use full-path wikilinks always: `[[02-Projets/Project/Project]]`.

## Trace or nothing (what makes `-o` real)

This step MUST end with a summary that cites every note it actually read as
a `[[wikilink]]`, or with the exact sentence `vault silent on this topic`.
A `-o` run that produces neither is a FAILED step, not a no-op: report it as
a failure and say what blocked it (vault unreachable, no match, empty note).
An `-o` that silently changes nothing is the whole reason this rule exists.

## If save mode (-s):
Target file: `.claude/output/ttw/{task-id}/01b-obsidian-context.md`.
- If file absent → `Write` with the full context report.
- If file present → `Edit` to append `\n\n## Run {ISO-timestamp}\n\n{report}`.

## Merge into Analysis

Feed the "Implications for current task" section into the Step 01 findings so the
subsequent plan (Step 02) reflects the vault knowledge.

## Next Step

Read [step-02-plan.md](step-02-plan.md) and execute it.
