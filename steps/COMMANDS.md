# TTW Commands — classify, then run, escalate or delegate

The ONE place that says how a command is run. Every tier reads it; no step
file restates it. Before running ANY command (coordinator or phase agent),
classify it — by capability, not by task: do not special-case nix.

- **Safe** — read-only, parse, test, edit a file in the repo,
  `git status/add`, `nix-instantiate --parse`, grep, build steps that do
  not touch the system: execute directly. Bare `nix flake check` and
  `darwin-rebuild build` (no sudo) you run yourself, long ones with
  `run_in_background`.
- **Password-interactive** — `sudo …` (including `sudo darwin-rebuild
  switch`) and system package installs that prompt for a password: DO NOT
  execute; add the exact command to a **"Run yourself" list** in the phase
  summary / final output. Only this bullet goes to that list. A baseline
  that needs one is recorded `baseline: skipped — {command} — {reason}`.
- **Blocked automatically** (classifier or sandbox) — not a hand-off: ask
  the user in the conversation, naming the exact action
  (« je le lance ? … »), and on their explicit yes run it yourself. A no
  from the user — in the permission box or in the conversation — is final:
  do not ask again.
  Never hand the user a command to type, never a `! cmd`.
- **Sandbox-excluded** — `git push`, `git commit`, `git pull`, `git fetch`,
  `gh`, `codex`, `claude-dispatch` and bare `nix flake check` run OUTSIDE the sandbox only
  when the Bash call is that command alone:
  run them as standalone commands — one per Bash call, from the repo cwd, no `cd … &&`, no
  `git -C`, no `&&` chain, no redirection, heredoc or `$(…)` (multi-line
  text: `git commit -F <file>`, `gh pr create --body-file <file>`). Any of
  those keeps the call sandboxed and it fails. If one fails on sandbox
  evidence, fix the shape; do not escalate. Long ones run with
  `run_in_background`; read the tail of the output file.
- **Read-only `.git/config`** — inside the sandbox, `.git/config` and
  `.git/hooks` are read-only in every repo: `git branch -d/-m/-u`,
  `git checkout -b <x> origin/<y>`, `git remote`, `git config --local`,
  `git init` and `git clone` fail there — run them with
  dangerouslyDisableSandbox (one retry, permission box).
- **Sandbox-blocked** — docker, or any command that just failed with clear
  sandbox evidence (a refused local DB socket, permission denied on allowed
  work, socket/auth failure): retry ONCE with `dangerouslyDisableSandbox: true`.
  The `ask` permission rule shows the user a confirmation box — they
  approve or refuse; a refusal is an answer, not an obstacle to work
  around. COORDINATOR ONLY: phase agents do not escalate; they surface the
  command in their summary and the coordinator decides. Never weaken the
  sandbox config itself and never touch secrets to make a command pass.
- **In doubt** — ask the user, unless already durably authorized this
  session.

Why: the confirmation box keeps the user in control while avoiding
dead-end "Run yourself" lists for one-click approvals. Only password
prompts stay delegated (the password belongs in the user's terminal).
Acceptance criteria that need a delegated command (e.g. "switch applied")
are marked **deferred to user** in the validate summary, not failed.

## Ship (`-pr`)

Commit on the run's branch (`git commit -F <file>`, English, imperative,
type prefix), push, `gh pr create --body-file <file>`, then squash-merge
the PR once its checks are green — each a standalone command per the
rule above. Never commit to `main`/`master`; never force-push or rewrite
pushed history without asking.
