# Step 04: Validate

YOU ARE A VALIDATOR, not an implementer. Do NOT add new features.

## Verification Checklist

1. **Acceptance Criteria**: go through each AC from the plan.
   For each one, verify it is actually implemented. Check the code.

2. **Build Check**: if applicable, run:
   - TypeScript: typecheck (`pnpm typecheck` or `npx tsc --noEmit`)
   - Lint: `pnpm lint` or equivalent
   - Build: `pnpm build` or equivalent

3. **Integration Check**: verify that:
   - All imports resolve
   - No circular dependencies introduced
   - Types are consistent across boundaries

4. **Quick Smoke Test**: if there's a dev server, verify it starts without errors

## If any AC is not met:
Go back and fix it. Do not proceed until all ACs pass.

## If save mode (-s):
Update progress in context file.

## Next Step — Conditional

Choose the next step based on active flags:

1. If `-t` (test) is active: Read [step-07-tests.md](step-07-tests.md) and execute it.
2. Else if `-x` (examine) is active: Read [step-05-examine.md](step-05-examine.md) and execute it.
3. Else if `-pr` (pull request) is active: Read [step-09-finish.md](step-09-finish.md) and execute it.
4. Else: **COMPLETE.** Present a summary of what was implemented.
