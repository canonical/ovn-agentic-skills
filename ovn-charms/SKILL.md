---
name: ovn-charms
description: "Work on the legacy v1 OVN Juju charms (ovn-central / ovn-chassis / ovn-dedicated-chassis, repo canonical/ovn-charms-v1): build with charmcraft, run unit/lint/coverage via tox, and read the flaky zaza functional CI. Use when asked to change or debug these machine charms, invoke their tests, or judge whether a CI failure is real."
argument-hint: "the charm (central/chassis/dedicated) and task (build/test/lint/functional)"
---

# Legacy OVN charms (ovn-charms-v1)

Monorepo of the three reactive machine charms. Repo-level conventions live in
`AGENT.rst` (commit subject ≤70 chars + trailing `.`, `Assisted-by` trailer,
single logical change per commit). This skill is the agent-facing workflow plus
the operational gotchas that are not in the README.

## Workflow

Build (charmcraft drives an LXD build container; each charm has its own
`charmcraft.yaml`):

```bash
cd ovn-chassis            # or ovn-central / ovn-dedicated-chassis
charmcraft clean
charmcraft pack -v
```

Unit / lint / coverage, from the monorepo root:

```bash
tox -e pep8-chassis       # flake8
tox -e py3-chassis        # unit tests (stestr)
tox -e cover-chassis      # coverage
```

Functional (zaza), from the monorepo root:

```bash
tox -e func-target-chassis -- noble-caracal
```

## Reading the CI

The zaza functional matrix is **not a reliable regression oracle** — it fails
on infrastructure. Known non-code failures (do not treat as regressions):

- `cannot resolve charm or bundle "vault" ... not found for latest/edge` on
  `amd64/ubuntu/22.04` (vault unpublished for 22.04; intermittent).
- COS-lite: `test_grafana_dashboards` → `401`, and `grafana-agent/0 in error state`.
- GitHub runner backlog leaves jobs `pending` for hours (the suite serialises).

Judge a change by build + lint + unit + coverage (all green ⇒ good) plus the one
functional job that actually exercises it, not the whole matrix.

## Gotchas in the layer unit tests

- Layer tests mock `SimpleOVSDB`. When you add code that calls
  `ovsdb.interface.find(...)`, set its `return_value` in the mock or the loop
  iterates a `MagicMock` and raises `TypeError`.
- `SimpleOVSDB('ovs-vsctl').interface` rows have no `__getitem__`; read via
  `.find(<condition>)` / iteration.
- `.remove(rec, col, value)` runs `ovs-vsctl remove`, which exits non-zero when
  the key is absent — make removals conditional on presence.

## Key rules

- One logical change per commit; build + unit + lint must be green.
- The functional CI is advisory — drive decisions from unit + lint + coverage.