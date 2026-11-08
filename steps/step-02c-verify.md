# Step 02c: Verify Plan

YOU ARE A RESEARCHER, not an implementer. Do NOT write any code yet.
Your job is to verify that the plan from step-02 is based on correct, up-to-date information.
The plan's `Docs:` lines (step-02-plan.md) already source each API and option
at its installed version. Do not redo the plan's `Docs:` lines: `-v` is the extra
online pass — deprecations, better alternatives, CVEs, recency — and it
re-researches online every `Docs:` entry that fell to rung 5.

## Process

For each major technical decision in the plan, verify it against current reality:

1. **APIs & Libraries**: WebSearch only for what the `Docs:` line does not settle:
   - Has the API been deprecated?
   - Are there newer/better alternatives?
   - Check version compatibility.

2. **Patterns & Best Practices**: WebSearch for current recommended patterns.
   - Is the proposed pattern still the recommended approach?
   - Has the framework/tool introduced a better way since your training cutoff?
   - Check official docs, not just blog posts.

3. **Configuration & Syntax**: If touching config files (nix, tsconfig, eslint, etc.):
   - WebFetch the official documentation page
   - Verify rung-5 entries, and options renamed/removed/deprecated since the
     installed version; names/types already sourced in the `Docs:` line are
     not re-verified

4. **Security**: If the plan involves auth, crypto, or sensitive data:
   - Verify the recommended approach hasn't changed
   - Check for known CVEs in proposed dependencies

## How to Research

- Use WebSearch for broad questions ("nextjs 15 best practices server actions 2026")
- Use WebFetch for specific doc pages (official docs URLs)
- Launch parallel research agents if multiple topics need verification
- Focus on OFFICIAL sources: framework docs, GitHub repos, RFCs — not Medium articles

## Output

For each item verified, report:
```
✅ {item} — confirmed correct ({source})
⚠️ {item} — outdated, recommended: {new approach} ({source})
❌ {item} — wrong/deprecated, must change: {correction} ({source})
```

## If issues found:
Update the plan and TodoWrite checklist to reflect corrections.
Present the changes to the user. In high-stakes mode or under `-q`, wait for
their answer before execute; otherwise proceed.

## Trace or nothing (what makes `-v` real)

This step MUST end with at least one web query traced — the query string and
the source URL, in the output — or with the explicit sentence `nothing to
verify online because` followed by the reason (nothing version-dependent, no
external API, offline). A `-v` run with neither is a FAILED step, not a
no-op: research nobody can trace is indistinguishable from research nobody
did.

## If save mode (-s):
Write verification results to `.claude/output/ttw/{task-id}/02c-verify.md`

## Next Step

Consult [ROUTING.md](ROUTING.md) → 03-execute.
