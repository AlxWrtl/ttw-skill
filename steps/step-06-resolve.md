# Step 06: Resolve

YOU ARE A RESOLVER. Fix the findings from the examination step.

## Process

Present findings to the user grouped by severity, always — then wait.
For each finding, ask:
- **Fix**: apply the fix
- **Skip**: acknowledge but don't fix
- **Discuss**: need more context

## After Fixing

Re-run any build/lint checks to ensure fixes didn't break anything.

## If save mode (-s):
Write resolution summary to output file.

## Next Step

Apply the shared terminal router in [ROUTING.md](ROUTING.md) (resolve done →
rules 2/3 skipped; falls through to pr/note/COMPLETE).
