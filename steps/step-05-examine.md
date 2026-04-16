# Step 05: Examine

YOU ARE A SKEPTICAL REVIEWER, not a defender of this code.
Your job is to find problems, not to validate the implementation.

## Adversarial Code Review

Launch 3 parallel code-reviewer agents, each with a different focus:

### Agent 1: Security Review
- Authentication/authorization gaps
- Input validation missing
- Data exposure in responses
- SQL injection, XSS, CSRF risks
- Secrets or credentials in code

### Agent 2: Logic Review
- Race conditions
- Null/undefined edge cases
- Error handling gaps
- Off-by-one errors
- State management issues
- Missing error boundaries

### Agent 3: Clean Code Review
- Naming consistency
- Dead code
- Unnecessary complexity
- Missing type safety
- Convention violations (from step-01 analysis)

### If economy mode (-e):
Do NOT launch agents. Instead, go through each review dimension yourself
as a self-review checklist. Be thorough but use no subagents.

## Collect Findings

Aggregate all findings and sort by severity:
- **Critical**: security vulnerabilities, data loss risks
- **Important**: logic bugs, error handling gaps
- **Minor**: naming, style, minor improvements

## Next Step

If there are Critical or Important findings:
  Read [step-06-resolve.md](step-06-resolve.md) and execute it.
Else if `-pr` (pull request) is active:
  Read [step-09-finish.md](step-09-finish.md) and execute it.
Else:
  **COMPLETE.** Present the review summary.
