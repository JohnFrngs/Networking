# Static Routing Lab — Cisco Packet Tracer

**Course:** Computer Networks | MSc Informatics, University of Piraeus  
**Tool:** Cisco Packet Tracer Student v6.2  
**Topic:** Static routing configuration across a multi-router topology

---

## Network Topology

The topology consists of three routers (Edge, Hermes, Bono) interconnecting
five LAN segments (A–E) and one server network (Atom), using both /29 and
/30 subnets.

![Topology Diagram](./topology.png)

---

## IP Addressing Scheme

| Network | CIDR  | Default Gateway | Host IP        | Subnet Mask     | Role        |
|---------|-------|-----------------|----------------|-----------------|-------------|
| A       | /29   | 10.1.1.25       | 10.1.1.26      | 255.255.255.248 | LAN segment |
| B       | /29   | 10.1.1.33       | 10.1.1.34      | 255.255.255.248 | LAN segment |
| C       | /29   | 10.1.1.41       | 10.1.1.42      | 255.255.255.248 | LAN segment |
| D       | /29   | 10.1.1.73       | 10.1.1.74      | 255.255.255.248 | LAN segment |
| E       | /29   | 10.1.1.65       | 10.1.1.66      | 255.255.255.248 | LAN segment |
| F       | /30   | 10.10.60.1      | 10.10.60.2     | 255.255.255.252 | WAN link    |
| Atom    | /30   | 10.10.60.22     | 10.10.60.21    | 255.255.255.252 | Server      |
| H-B     | /30   | 10.10.60.5      | 10.10.60.6     | 255.255.255.252 | WAN link    |
| B-E     | /30   | 10.10.60.10     | 10.10.60.9     | 255.255.255.252 | WAN link    |

**Design rationale:**  
- **/29** used for LAN segments → 6 usable hosts per subnet, sufficient for
  small workgroups  
- **/30** used for point-to-point WAN links → only 2 usable IPs needed,
  minimizing address waste

---

## Total Usable IP Count

| Subnet type | Count | Hosts/subnet | Total IPs |
|-------------|-------|--------------|-----------|
| /29 (LANs)  | 5     | 6            | 30        |
| /30 (WAN)   | 4     | 2            | 8         |
| **Total**   |       |              | **38**    |

---

## Static Routing Configuration (example)

Each router was manually configured with explicit routes to all remote networks.  
Example command on Edge router:

```bash
ip route 10.1.1.32 255.255.255.248 10.10.60.5
ip route 10.1.1.40 255.255.255.248 10.10.60.5
ip route 10.10.60.8  255.255.255.252 10.10.60.5
```

---

## Connectivity Verification

End-to-end connectivity was verified using `ping` across all network pairs:

| Source              | Destination           | Result |
|---------------------|-----------------------|--------|
| B (10.1.1.34)       | A (10.1.1.26)         | ✅     |
| A (10.1.1.26)       | Atom (10.10.60.21)    | ✅     |
| D (10.1.1.74)       | E (10.1.1.66)         | ✅     |

---

## Static vs Dynamic Routing — When to Use Which

Static routing was chosen here because the topology is small and fixed.
In larger or frequently changing networks, dynamic protocols such as
**OSPF** or **EIGRP** would be preferred, as they automatically update
routing tables without manual intervention, reducing administrative overhead
and improving fault tolerance.

---

## Skills Demonstrated

- IPv4 subnetting (VLSM, CIDR notation)
- Static route configuration on Cisco routers
- Multi-router topology design
- Network troubleshooting with ping/traceroute
