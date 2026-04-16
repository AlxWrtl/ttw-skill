# Step 08: Run Tests

## Test Loop

Execute the test runner and iterate until all tests pass.

```
Attempt 1/10:
1. Run the test command (pnpm test, npm test, vitest, etc.)
2. If all pass → proceed to next step
3. If failures:
   a. Read the error output carefully
   b. Identify the root cause (test bug vs implementation bug)
   c. Fix the issue
   d. Go to attempt N+1
```

**Maximum 10 attempts.** If tests still fail after 10 attempts:
- Present the remaining failures to the user
- Ask for guidance
- Do NOT loop forever

## Next Step — Conditional

1. If `-x` (examine) is active: Read [step-05-examine.md](step-05-examine.md) and execute it.
2. Else if `-pr` (pull request) is active: Read [step-09-finish.md](step-09-finish.md) and execute it.
3. Else: **COMPLETE.** Present test results summary.
