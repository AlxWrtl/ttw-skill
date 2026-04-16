# Step 06: Resolve

YOU ARE A RESOLVER. Fix the findings from the examination step.

## Process

### If auto mode (-a):
Automatically fix all Critical and Important findings.
Skip Minor findings unless they're trivial to fix.

### If NOT auto mode:
Present findings to the user grouped by severity.
For each finding, ask:
- **Fix**: apply the fix
- **Skip**: acknowledge but don't fix
- **Discuss**: need more context

## After Fixing

Re-run any build/lint checks to ensure fixes didn't break anything.

## If save mode (-s):
Write resolution summary to output file.

## Next Step

If `-pr` (pull request) is active:
  Read [step-09-finish.md](step-09-finish.md) and execute it.
Else:
  **COMPLETE.** Present the resolution summary.
