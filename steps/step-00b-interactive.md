# Step 00b: Interactive Configuration

Present all available flags to the user as a toggle menu.
Use AskUserQuestion to let them enable/disable each flag.

Display current flag state and let user toggle:
```
Current flags:
[ ] -a  Auto (skip confirmations)
[ ] -x  Examine (adversarial review)
[ ] -s  Save (persist outputs)
[ ] -t  Test (create + run tests)
[ ] -e  Economy (no subagents)
[ ] -b  Branch (create git branch)
[ ] -pr Pull Request (commit + PR)
[ ] -k  Tasks (task breakdown)
[ ] -m  Teams (parallel execution)
```

After user confirms, update the active flags and return to step-00-init flow.
