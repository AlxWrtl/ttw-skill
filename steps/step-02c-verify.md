# Step 02c: Verify Plan

YOU ARE A RESEARCHER, not an implementer. Do NOT write any code yet.
Your job is to verify that the plan from step-02 is based on correct, up-to-date information.

## Process

For each major technical decision in the plan, verify it against current reality:

1. **APIs & Libraries**: WebSearch for the latest docs of any library/framework used.
   - Is the API still current? Has it been deprecated?
   - Are there newer/better alternatives?
   - Check version compatibility.

2. **Patterns & Best Practices**: WebSearch for current recommended patterns.
   - Is the proposed pattern still the recommended approach?
   - Has the framework/tool introduced a better way since your training cutoff?
   - Check official docs, not just blog posts.

3. **Configuration & Syntax**: If touching config files (nix, tsconfig, eslint, etc.):
   - WebFetch the official documentation page
   - Verify option names, types, and default values
   - Check if options have been renamed, removed, or deprecated

4. **Security**: If the plan involves auth, crypto, or sensitive data:
   - Verify the recommended approach hasn't changed
   - Check for known CVEs in proposed dependencies

## How to Research

- Use WebSearch for broad questions ("nextjs 15 best practices server actions 2026")
- Use WebFetch for specific doc pages (official docs URLs)
- Launch parallel research agents if multiple topics need verification (unless economy mode)
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
If not in auto mode (-a), present changes for user approval.

## If save mode (-s):
Write verification results to `.claude/output/apex/{task-id}/02c-verify.md`

## Next Step

If teams mode (-m) is active:
  Read [step-03-execute-teams.md](step-03-execute-teams.md) and execute it.
Else:
  Read [step-03-execute.md](step-03-execute.md) and execute it.
