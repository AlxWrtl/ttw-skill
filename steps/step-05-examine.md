# Step 05: Examine

YOU ARE A SKEPTICAL REVIEWER, not a defender of this code.
Your job is to find problems, not to validate the implementation.

## Adversarial Code Review

Launch ONE blind code-reviewer agent (`model: opus`, effort high) working a merged checklist.
The explicit `model: opus` override is required (the per-invocation param
beats the agent frontmatter). The coordinator arbitrates its findings. No
Fable pass runs over them.

**Blind review.** Each reviewer receives the spec (task, ACs, the plan's
premises) and the real diff — and never the implementer's rationale. No
transcript, no execute summary, no "here is what I was going for". A
reviewer told why the code is right stops looking for the reason it is not.

**Checklist, not free reading.** The reviewer works its boxes explicitly and
reports every one as pass / fail / not-applicable. Free-form review drifts
toward style; the checklist is what keeps the expensive categories covered.

### Checklist: premise, scope, tests that lie, correctness
- [ ] Does the diff do what the ACs asked, or something adjacent to it?
- [ ] Any premise from the plan contradicted by the code as written?
- [ ] Scope creep: changes no AC asked for.
- [ ] Tests asserting nothing: no assertion, mocked subject, tautology.
- [ ] Tests that pass because the assertion was bent to fit the code.
- [ ] Any probe cited in the summary: is its denominator shown? A `matched`
      with no `total` is not a measurement, and a zero it returns is not
      an absence.
- [ ] Any visible label, string or `aria-label` the diff introduces: is its
      SOURCE cited — a capture, a spec, or the user's own words? An
      invented label is a finding, not a detail.
- [ ] Every `[M]` premise: does the cited evidence actually exist as cited?
      Every `[I]` premise: did it stay out of the code? This box is NOT
      optional — step-02-plan names it as the independent reader that makes
      the tags mean anything, so without it the tag rule lies.
- [ ] Races: non-atomic read-modify-write, unawaited async, lost updates.
- [ ] Null/undefined, empty collections, off-by-one at the boundaries.
- [ ] Error handling: swallowed errors, missing boundaries, wrong fallback.
- [ ] State shared across requests/instances that should not be.
- [ ] Convention violations (step-01), dead code, needless complexity.

### Security & data integrity (added on signal)
The security and data-integrity boxes are added to the reviewer's checklist when the run is High-stakes
(SKILL.md tier gate), `ttw-tier` reports `tier=high`, or the real diff touches
a HIGH path, auth, shell/SQL/template construction, or permission/sandbox
config.
- [ ] AuthN/authZ gaps; missing input validation; data exposed in responses.
- [ ] Injection surfaces: SQL, shell, template, XSS, CSRF.
- [ ] Secrets or credentials in code, logs, or error messages.
- [ ] Destructive or irreversible operations without a guard.
- [ ] Writes/migrations that can half-apply and leave broken state.

## Collect Findings

Aggregate all findings and sort by severity:
- **Critical**: security vulnerabilities, data loss risks
- **Important**: logic bugs, error handling gaps
- **Minor**: naming, style, minor improvements

Then write the triage into `.claude/output/ttw/{task-id}/05-examine.md`:
one line per finding — `id | file:line | confirmed | evidence` or
`id | file:line | dismissed | reason` — then the tally
`confirmed=N unique=U dismissed=D`, same format as step-04.

## Next Step

Apply the shared terminal router in [ROUTING.md](ROUTING.md): Critical/Important
findings trigger rule 3 (→ 06-resolve); otherwise fall through to pr/note/COMPLETE.
