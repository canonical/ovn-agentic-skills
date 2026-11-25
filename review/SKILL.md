---
name: review
description: "Review a proposed change (PR, diff, patch, or branch) as a project engineer and produce a concrete, verifiable verdict. Use when asked to review, sanity-check, or approve code; when a change claims to be ready to merge; or when deciding whether a fix or backport is correct, secure, and well-tested. Verifies claims by running read-only checks rather than trusting the description."
argument-hint: "what/where to review (PR number, branch, file, or diff)"
---

# Code review

Review the change as an engineer who owns the outcome — not a generic
proofread. The output is a verdict (does it work as-is? what must change
first?) plus concrete next steps.

## Workflow

1. **Read the context first.** Understand what the change is *for*: the linked
   issue/ticket, the surrounding code, and any conventions in the repo
   (`CONTRIBUTING`, `AGENT*` docs, CI config).
2. **Read the diff** end to end, not just the summary.
3. **Verify, don't trust.** Re-run the claimed checks yourself with read-only
   commands (`git log`/`diff`/`rev-parse`, `go build`/`test`/`vet`, the repo's
   lint/test invocation, `checkpatch`). Do not assert something passes because
   the author said so.
4. **Check each rubric** in [references/rubrics.md](references/rubrics.md) —
   correctness, security, style, tests, and performance/resilience.
5. **Write the review** in org/markdown with `*`/`**` headings and bullets,
   citing the exact file/branch/commit/line for every finding. End with a
   verdict and ordered next steps.

## Verdict format

Answer directly:

- **Will this work as-is?** yes / no / partially.
- **What must change first?** list blockers (must-fix) separately from
  non-blocking nits and design questions.
- **Recommended next steps** — the smallest concrete path to a mergeable state.

## Key rules

- Read-only verification: run checks freely, but do not edit, commit, push,
  upload, or change any external state while reviewing. If a confirmation
  would modify something, stop and report it instead of running it.
- Every finding cites evidence you can point at (a line, a SHA, a failing
  command), not a vague opinion.
- `Approved` means you would merge it yourself — not "looks plausible".
- Distinguish a defect in the change from a pre-existing or unrelated problem.

## Supporting files

- [references/rubrics.md](references/rubrics.md) — the five review rubrics
- [examples.md](examples.md) — a worked review and evaluation prompts