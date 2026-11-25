# MicroOVN — the data path

## The analogy to lead with

Think of the virtual network as a building:

- Each **tenant VM** is a room; its NIC plugs into the local **switch** (OVS).
- **Logical switches and routers** are the rooms and hallways an architect drew
  on the blueprint (the NB). They are *intent*, not furniture.
- **ovn-controller / OVS** on each machine is the local post office that
  actually moves the letters (packets) according to that blueprint.
- When the destination is in a *different* building (hypervisor), the post
  office seals the letter in an **envelope** (GENEVE) and sends it down the
  trunk line (tunnel) to the other building's post office.

## The four steps (VM → VM, cross host)

```
VM A ─► br-int (OVS) ─► [ingress: tag this as logical port A]
                          └─► [logical decision: A wants to reach B]
                               └─► B is remote → encap GENEVE → tunnel ──► remote chassis
                                                                             └─► decap → [egress: ACL/NAT/LB]
                                                                                  └─► output to VM B's port
```

1. **Ingress** — the VM's frame enters OVS through its VIF. The ingress flows
   stamp it with the identity of the logical port it came from.
2. **Logical decision** — the flows (computed by ovn-controller from the SB)
   resolve "logical port A to logical address of B". If B is local, output to
   B's port and you're done.
3. **Tunnel** — if B is on another chassis, the frame is encapsulated in GENEVE
   and sent to that chassis's tunnel endpoint. Remote VMs thus look adjacent.
4. **Egress** — the remote chassis decapsulates, runs the egress pipeline
   (ACLs, NAT, load balancing), and outputs to B's VIF.

## Same-host shortcut

If both VMs are on the same hypervisor, steps 3–4 collapse: the flows output
directly from A's ingress to B's port on the local bridge — no tunnel. (This
is why "traffic stays on the host" is a real OVN property, not an
optimisation hack.)

## The one-liner to close with

The network is **logical upstream** (switches/routers/ACLs in the NB model)
and **flows downstream** (OpenFlow on OVS, GENEVE under the covers). OVN
decides what *should* happen; OVS makes the packets *actually* do it.