# Review rubrics

Work through these five lenses. Record blockers as must-fix; everything else
as nits or open questions. Cite evidence for each.

## Correctness

- Does the change actually do what it claims? Trace the happy path and the
  error paths.
- Are the boundary conditions right (empty input, nil/absent value, off-by-one,
  concurrency/races)?
- Does it integrate with the surrounding code as it exists today (current
  branches, versions, APIs) — not an imagined future state?
- Are the cited SHAs, paths, and versions real? Re-verify against the repos.

## Security

- No secrets, keys, or credentials introduced or logged.
- No injection (SQL/command/template) or unsafe deserialisation.
- Privilege boundaries preserved (don't add root/`sudo`/privileged actions
  without justification).
- Secrets/keys handled via existing mechanisms, not ad hoc.

## Style & conventions

- Matches the repo's existing patterns, naming, and framework choices.
- Commit subjects follow the project convention (often: imperative, ≤ a char
  limit, trailing full stop).
- Attribution trailers present and correct (`Signed-off-by`, `Assisted-by`
  where required).
- No dead code, stray comments, or unrelated changes bundled in.

## Tests

- Is the change covered by a test that would fail without it?
- Were the tests actually run? Re-run the relevant suite if possible.
- Does the test assert the behaviour, or just that it ran?
- Is the test deterministic (no sleeps/timing dependence unless justified)?

## Performance, resilience & scope

- Any obvious complexity blow-up or unbounded work?
- Failure/error handling reasonable; retries/backoff where the operation is
  transient?
- Scope matches the stated goal — flag scope creep that belongs in another
  change.