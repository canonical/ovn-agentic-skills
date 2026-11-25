---
name: microovn-intro
description: "What MicroOVN is and how it fits together, for onboarding. Use when explaining MicroOVN's architecture, the relationship between OVN / OVS / microcluster / dqlite, what each snap service runs, how a packet travels the logical → physical path, or mapping a MicroOVN component to its upstream OVN/OVS equivalent. Explain analogy-first."
argument-hint: "concept to explain (e.g. 'what is OVN', 'what runs in the snap', 'how does a packet flow', 'central vs chassis')"
---

# MicroOVN — intro

## Resources

| Resource | Use when |
|---|---|
| [concepts.md](references/concepts.md) | Explaining any component or the layering, analogy-first |
| [data-path.md](references/data-path.md) | Walking a packet from one VM to another (same host and cross host) |

---

## Quick Procedures

### 1 — Explain what MicroOVN is

Say it in one line, then layer up:

1. One line: MicroOVN packages **OVN + OVS** into a snap and automates deploying/managing them as a cluster.
2. The three layers: **MicroOVN** (orchestration) → **OVN** (control plane, the logical network) → **OVS** (data plane, the actual forwarding).
3. The two engines it wraps: OVN = the brain, OVS = the switch — OVN computes the logic, OVS moves the packets.

See [concepts.md](references/concepts.md).

### 2 — Walk the OVN data path

1. Start from the analogy (letter / room / hallway / envelope).
2. Spelunke the four steps: ingress → logical decision → tunnel → egress.
3. Finish with the one-liner: the network is *logical* upstream and *flows* downstream.

See [data-path.md](references/data-path.md).

### 3 — Map a component to upstream OVN/OVS

1. Open [concepts.md](references/concepts.md#components) — the component table.
2. Match the MicroOVN snap service (`central`, `chassis`, `switch`, `bird`) to the upstream daemons it runs.
3. Note what `microovnd` (microcluster + dqlite) is and why it is not OVN at all.

### 4 — Explain central vs chassis

- `central` = the databases + the translator (NB/SB + `ovn-northd`).
- `chassis` = the worker on each hypervisor (`ovn-controller` + `ovs-vswitchd`).
- A node can be central, chassis, or both; the cluster keeps them consistent.

---

## Key Rules

1. OVN is the **control plane**, OVS is the **data plane** — never describe them as the same thing.
2. North is intent (what you want), south is reality (what is). `northd` sits between them and compiles one into the other.
3. The logical network (switches/routers/ACLs) is a *model*; what actually forwards packets is OpenFlow flows on OVS.
4. `microovnd` (microcluster + dqlite) is MicroOVN's *own* cluster/control store — it is a layer MicroOVN adds, not part of OVN.
5. Clustering is optional: a single-node MicroOVN has the same services, minus the distributed coordination.