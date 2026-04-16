# Step 09: Finish

## If teams were used (-m):
Ensure all team agents have completed and shut down cleanly.

## Git Operations

1. **Stage changes**: `git add` all modified/created files
2. **Commit**: use conventional commit format
   - `feat: {description}` for new features
   - `fix: {description}` for bug fixes
   - Include a body with key changes if the diff is large
3. **Push**: `git push -u origin {branch-name}`

## Create Pull Request

Use `gh pr create` with:
- **Title**: conventional format matching the commit
- **Body**: structured with:
  - ## Summary (what was done)
  - ## Changes (bullet list of key changes)
  - ## Testing (how it was tested)
  - ## Acceptance Criteria (checklist from plan)

## If NOT auto mode:
Show the PR title and body for approval before creating.

## COMPLETE

Present final summary:
```
APEX Complete
Branch: {branch}
Commit: {hash}
PR: {url}
Steps completed: {list}
```
