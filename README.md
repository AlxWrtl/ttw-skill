# APEX Skill for Claude Code

A structured, multi-step implementation methodology for [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

APEX breaks complex tasks into progressive steps with quality gates, keeping instructions fresh in context at each phase.

## Pipeline

```
Init → Analyze → Plan → Execute → Validate → [Tests] → [Examine] → [Resolve] → [Finish]
 00      01       02      03        04         07         05          06          09
```

## Flags

| Enable | Disable | Description |
|--------|---------|-------------|
| `-a` | `-A` | Auto — skip confirmations |
| `-x` | `-X` | Examine — adversarial code review |
| `-s` | `-S` | Save — persist outputs to files |
| `-t` | `-T` | Test — create and run tests |
| `-e` | `-E` | Economy — no subagents, direct tools only |
| `-b` | `-B` | Branch — create git branch |
| `-pr` | `-PR` | Pull request — commit + PR (implies `-b`) |
| `-k` | `-K` | Tasks — task breakdown with dependency graph |
| `-m` | `-M` | Teams — parallel agent execution (implies `-k`) |
| `-v` | `-V` | Verify — research plan online before executing |
| `-i` | | Interactive — configure flags via menu |
| `-r` | | Resume — continue previous task |

## Usage

```
/apex add feature                    # Basic
/apex -a -s implement auth           # Autonomous + save
/apex -a -x -s fix bug              # Full autonomous with review
/apex -a -t -pr add endpoint        # Auto + tests + PR
/apex -e simple fix                  # Economy mode (save tokens)
/apex -a -x -t -pr full feature    # Everything enabled
```

## Installation

Copy the skill into your Claude Code skills directory:

```bash
# Clone
git clone https://github.com/AlxWrtl/apex-skill.git

# Copy to Claude Code skills
cp -r apex-skill ~/.claude/skills/apex
```

Or symlink it:

```bash
git clone https://github.com/AlxWrtl/apex-skill.git ~/apex-skill
ln -s ~/apex-skill ~/.claude/skills/apex
```

## Structure

```
SKILL.md                    # Skill entry point (read by Claude Code)
eval-suite.json             # Evaluation test cases
steps/
  step-00-init.md           # Initialize: parse flags, check git, setup
  step-00b-interactive.md   # Interactive flag configuration
  step-00b-branch.md        # Git branch creation
  step-00b-economy.md       # Economy mode rules
  step-00b-save.md          # Save mode output setup
  step-01-analyze.md        # Explore codebase, evaluate complexity
  step-02-plan.md           # Create implementation plan + acceptance criteria
  step-02b-tasks.md         # Task decomposition with dependency graph
  step-02c-verify.md        # Research/verify plan against current docs
  step-03-execute.md        # Implement changes task-by-task
  step-03-execute-teams.md  # Parallel agent team execution
  step-04-validate.md       # Verify acceptance criteria + build checks
  step-05-examine.md        # Adversarial code review (security, logic, clean code)
  step-06-resolve.md        # Fix examination findings
  step-07-tests.md          # Create tests following project patterns
  step-08-run-tests.md      # Run tests with retry loop (max 10)
  step-09-finish.md         # Git commit, push, create PR
```

## How It Works

APEX uses **progressive context loading**: instead of dumping all instructions at once, each step is read and executed independently. This keeps the relevant instructions fresh in Claude's context window, improving attention and accuracy on complex tasks.

Steps are connected via relative links — each step tells Claude which file to read next based on active flags and outcomes.

## License

MIT
