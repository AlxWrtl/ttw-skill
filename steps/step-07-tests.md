# Step 07: Tests

YOU ARE A TEST ENGINEER, not an implementer.

## Analyze Test Patterns

Before writing any tests:
1. Find existing test files in the project (Glob for `**/*.test.*`, `**/*.spec.*`, `**/__tests__/**`)
2. Read 2-3 existing tests to understand:
   - Test framework (Jest, Vitest, Playwright, etc.)
   - Naming conventions
   - Setup/teardown patterns
   - Assertion style
   - Mock patterns

## Create Tests

Based on the acceptance criteria from step-02:
1. Write unit tests for pure logic/utilities
2. Write integration tests for API endpoints/data flow
3. Write component tests for UI (if applicable)

Follow the EXISTING test patterns exactly. Don't introduce new testing paradigms.

## If test-first (`-f`) already ran

A test-author agent wrote the AC tests before execute; those files stay
read-only for the implementer and step-04-validate checks their integrity.
Your job here is to EXTEND coverage — the cases the ACs did not spell out
(edge values, error paths, integration seams) — in files of your own. Never
rewrite, weaken, reorganize or duplicate a test-author file: a duplicate of
an AC test is coverage theatre, and an edit to one destroys the evidence
validate needs. A test-author file that looks wrong is a finding you report,
not a file you fix.

## Rules

- Each test should be independent
- Use descriptive test names that explain the behavior
- Test edge cases, not just happy paths
- Mock external dependencies, not internal ones

## Next Step

Read [step-08-run-tests.md](step-08-run-tests.md) and execute it.
