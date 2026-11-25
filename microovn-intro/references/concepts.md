# MicroOVN — core concepts

## The one-liner

MicroOVN packages **OVN** and **OVS** into a single snap and automates
deploying, clustering, and managing them. It is "OVN made boring" — the brain
and the switch, pre-wired, with clustering on top.

## The three layers

```
    ┌─────────────────────────────────────────────┐
    │ MicroOVN        orchestration / lifecycle   │  snap, microcluster, dqlite
    ├─────────────────────────────────────────────┤
    │ OVN             control plane / logical net  │  NB, northd, SB, ovn-controller
    ├─────────────────────────────────────────────┤
    │ OVS             data plane / forwarding      │  ovs-vswitchd, flows, tunnels
    └─────────────────────────────────────────────┘
```

MicroOVN deploys and manages OVN; OVN computes the logical network; OVS does
the actual packet forwarding.

## The pieces, analogy-first

| Component | Analogy | What it actually is |
|---|---|---|
| **OVN** | the architect + air-traffic control | distributed SDN controller over OVS. Logical switches, routers, ACLs, NAT, LB. |
| **OVN NB DB** | the blueprint ("desired state") | the *logical* topology the operator/cloud wrote: `Logical_Switch`, `Logical_Router`, `Logical_Switch_Port`, ACLs, NAT. |
| **ovn-northd** | the translator / compiler | daemon that reads the NB blueprint and compiles it into logical *flows* in the SB. |
| **OVN SB DB** | the wiring diagram ("reality") | physical bindings + computed flows: which chassis hosts which port (`Chassis`, `Datapath_Binding`, `Port_Binding`, tunnels). |
| **ovn-controller** | the electrician on each machine | runs per hypervisor; reads the SB, computes the concrete OpenFlow flows, programs OVS. |
| **OVS (ovs-vswitchd)** | the actual switch and wires | the forwarding engine (kernel or userspace/DPDK/DOCA) that moves packets per the flows. |
| **GENEVE tunnel** | the sealed envelope on the trunk line | carries logical traffic between hypervisors, making remote VMs look adjacent. |
| **microcluster** | the membership + trust office | MicroOVN's clustering layer: mTLS auth, members, tokens, service enablement across nodes. |
| **dqlite** | the shared ledger | distributed SQLite (Raft) storing microcluster's cluster state — the cluster's source of truth. |

Key asymmetry to remember: **north is intent, south is reality.** `northd`
sits in the middle and compiles the former into the latter.

## Snap services → upstream components

| Snap service | Runs (upstream) | Role |
|---|---|---|
| `microovnd` | microcluster + dqlite | MicroOVN's own control plane: cluster membership, state, service orchestration. **Not OVN.** |
| `central` | `ovsdb-server` (NB + SB) + `ovn-northd` | the OVN control plane: databases + the translator. |
| `chassis` | `ovn-controller` + `ovs-vswitchd` | the data plane on a hypervisor: programs and forwards. |
| `switch` | `ovs-vswitchd` only | standalone OVS, no OVN (for OVS without the SDN layer). |
| `bird` | BIRD daemon | BGP speaker for the OVN underlay/overlay (managed or attach modes). |

A node can be central, chassis, or both; `microovn status` reports each
service's health.

## Where the product family sits

| Product | Layer / role |
|---|---|
| **MicroOVN** (snap) | runs OVN+OVS on a node / cluster. The engine. |
| **microovn-operator** (Juju charm) | drives MicroOVN at scale across many machines (relations, tokens, integration). |
| **ovn-charms-v1** (reactive charms) | the *legacy* way to deploy OVN (charm-ovn-central / -chassis / -dedicated-chassis). |
| **ovn / ovs / ovn-vif** (upstream) | the engines themselves; MicroOVN and the charms are distribution + lifecycle around them. |

## How clustering fits (briefly)

MicroOVN uses microcluster (mTLS + dqlite) to keep a set of nodes acting as
one: who is a member, what services each runs, and the shared cluster state.
This is *separate* from OVN's own clustering (OVN's NB/SB are themselves
clustered via RAFT) — MicroOVN layers its own control store above OVN.