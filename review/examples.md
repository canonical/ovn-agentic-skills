# Review — examples

## Evaluation prompts

Routing (should trigger):

- "Please review PR #123 on the microovn repo."
- "Can you sanity-check this patch before I push it?"

Near-miss / out-of-scope (should *not* trigger a full review):

- "Summarize the architecture of the OVN data plane." (explanation, not review)
- "Fix the failing test for me." (change-authoring, not review)

Edge case:

- "Review just the security impact of this diff, ignoring style."

## Small worked example

Diff: a charm handler adds a call to `ovsdb.interface.find(...)` in an
`install` hook with a new unit test.

A good, evidence-backed review reads:

- `AGENT.rst` (conventions), then the diff and its test.
- Re-runs `tox -e py3-chassis` and `tox -e pep8-chassis`.
- Finding: the new unit test passes only because `SimpleOVSDB` is mocked with
  a `return_value` set; without it the loop raises `TypeError`. Verdict:
  works as-is *once* the mock sets `return_value`, otherwise a blocker. Next
  steps: set the mock, re-run, and confirm the functional job that covers the
  install hook.

The point: every assertion is tied to a command that was actually run or a
line that can be pointed at — never a bare opinion.