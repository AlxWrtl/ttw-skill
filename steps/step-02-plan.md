# Step 02: Plan

YOU ARE A PLANNER, not an implementer. Do NOT write any code yet.

## ULTRA THINK

Before writing the plan, mentally simulate the entire implementation:
- Walk through every file you'll create or modify
- Consider the order of changes (what depends on what)
- Identify where things could go wrong
- Think about edge cases the user didn't mention

## Create Implementation Plan

Produce a structured plan with:

1. **Tasks** — numbered, ordered by dependency:
   ```
   T1: Create types/interfaces in types.ts
   T2: Add database migration
   T3: Implement API endpoint (depends on T1, T2)
   T4: Create UI component (depends on T1)
   T5: Wire up route (depends on T3, T4)
   ```

2. **Acceptance Criteria** — specific, verifiable conditions:
   ```
   AC1: User can create a new item via the form
   AC2: Validation errors display inline
   AC3: Success redirects to the list page
   ```

3. **Testing Strategy** (if -t flag active):
   ```
   - Unit tests for validation logic
   - Integration test for API endpoint
   - Component test for form submission
   ```

4. **Risks & Mitigations**

## Create TodoWrite Checklist

Convert tasks into a TodoWrite checklist. Only ONE todo can be in_progress at a time.

## If tasks mode (-k) or teams mode (-m):
Read [step-02b-tasks.md](step-02b-tasks.md) and execute it before proceeding.

## User Approval

If auto mode (-a) is NOT active:
- Present the complete plan
- Ask the user if they want to modify anything
- Wait for approval before proceeding

If auto mode (-a) IS active:
- Display the plan briefly
- Proceed automatically

## Next Step

If verify mode (-v) is active:
  Read [step-02c-verify.md](step-02c-verify.md) and execute it.
Else if teams mode (-m) is active:
  Read [step-03-execute-teams.md](step-03-execute-teams.md) and execute it.
Else:
  Read [step-03-execute.md](step-03-execute.md) and execute it.
