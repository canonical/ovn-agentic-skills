# Skill Authoring Guide

A `SKILL.md` is a small, reusable task playbook for an agent: repeatable
workflows that benefit from richer instructions, references, or scripts,
without bloating always-on context. This is distinct from an `AGENTS.md`
(repo-wide, always-on defaults and guardrails).

## What goes where

- Each skill lives in its own directory.
- `SKILL.md` is the entry point and the only always-loaded file.
- `references/` — longer supporting material loaded on demand.
- `scripts/` — deterministic helpers the skill runs.
- `examples.md` — realistic worked examples and evaluation prompts.
- `templates/` — output skeletons.

## Core rules

1. **One skill, one job.** Narrow, concrete scope. `microovn-dev-loop` is a
   skill; "testing" is not.
2. **The description is the router.** The front-matter `description` says both
   what the skill does and when to use it, with trigger wording. It is the most
   important line in the file.
3. **Keep `SKILL.md` small** (under ~500 lines). Push bulk into `references/`.
4. **Write procedures, not essays.** Inspect these files → run this command →
   if this case, follow this branch → validate → fix and rerun.
5. **Script deterministic work.** If a step is a fixed sequence of commands,
   make it a `scripts/` helper with an explicit validator.
6. **Make validation explicit.** Every skill ends with a way to confirm the
   work is correct (a test run, a checker command, a diff review).
7. **Separate concerns.** Keep `SKILL.md` a dispatcher + workflow; keep facts
   and reference tables in `references/`.
8. **Evaluate with a few test prompts.** 2–4 prompts in `examples.md`: a clear
   positive trigger, a near-miss that should not trigger, and an edge case.

## Minimal template

```markdown
---
name: <skill-name>
description: <What this does and when to use it. Include trigger wording.>
---

# <Skill title>

## Workflow

1. Inspect `<path/to/file>`.
2. Run `<command>`.
3. If needed, follow [reference](references/reference.md).
4. Validate with `<validator>`.
5. If validation fails, fix and rerun.

## Key rules

- …

## Supporting files

- [Reference](references/reference.md) — deeper rules and edge cases
- [Examples](examples.md) — realistic examples and eval prompts
- `scripts/<name>` — deterministic helper
```

Optional sections, only when they earn their place: `Inputs`, `Outputs`,
`Stop and report if`, `Use this skill when`, `Do not use this skill when`.

## Checklist before adding a skill

- Is the scope narrow?
- Is the description specific and triggerable?
- Is the workflow executable as written?
- Is validation explicit?
- Did bulky detail move out of `SKILL.md`?
- Should any step be a script instead?
- Are there 2–4 evaluation prompts?