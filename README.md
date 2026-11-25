# ovn-agentic-skills

A collection of agent skills for the OVN product family at Canonical:
[MicroOVN](https://github.com/canonical/microovn), the legacy
[OVN charms](https://github.com/canonical/ovn-charms-v1), and OVN/OVS
packaging and release work.

Skills are small, self-contained task playbooks for a coding agent (OpenCode,
GitHub Copilot, Claude Code) and, at the same time, readable onboarding notes
for new contributors. Each skill lives in its own directory with a `SKILL.md`
entry point; bulky detail is pushed into `references/` and `scripts/`.

## Skills

| Skill | Description |
|---|---|
| [microovn-dev-loop](microovn-dev-loop/SKILL.md) | The everyday MicroOVN developer loop: `go build`/`test`/`vet`, gofmt, linters, the snap build, and the BATS functional tests — including the libdqlite (cgo) prerequisite. |
| [ovn-sru](ovn-sru/SKILL.md) | Package an OVN change as a quilt patch and ship it to users: Debian salsa packaging, Ubuntu sync, PPA upload, and the noble SRU / Launchpad merge request. |
| [ovn-charms](ovn-charms/SKILL.md) | Work on the legacy reactive OVN charms (`ovn-central`, `ovn-chassis`, `ovn-dedicated-chassis`): build, unit test, lint, and read the flaky functional CI. |
| [review](review/SKILL.md) | Review a change against fixed correctness / elegance / security rubrics and produce a clear, verifiable verdict with concrete next steps. |

## Authoring

See [docs/skill-writing-guidelines.md](docs/skill-writing-guidelines.md) for
how skills are structured and the checklist to pass before adding one.

## Install (OpenCode)

Skills are discovered from `~/.config/opencode/skills`.

```bash
mkdir -p ~/.config/opencode/skills
git clone https://github.com/canonical/ovn-agentic-skills ~/.config/opencode/ovn-agentic-skills
ln -s ~/.config/opencode/ovn-agentic-skills ~/.config/opencode/skills/ovn-agentic-skills
```

Restart OpenCode, then check `/skills` shows the new entries.

## Contributing

Pull requests welcome — new skills or improvements to existing ones. Follow
the [authoring guide](docs/skill-writing-guidelines.md).