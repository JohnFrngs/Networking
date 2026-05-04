# Cisco Networking Labs

> **Academic context:** These labs were completed as coursework assignments for the *Computer Networks* course of the MSc in Informatics at the University of Piraeus. Despite their academic origin, they cover real-world networking concepts implemented hands-on in Cisco Packet Tracer v6.2.

***

## Theoretical Background

Modern computer networks rely on two fundamental mechanisms to move data between hosts: **routing** (deciding which path a packet takes) and **addressing** (uniquely identifying every device). This repo demonstrates both, progressively, across two labs.

**Subnetting & CIDR** divides a large IP space into smaller logical segments. A prefix length (e.g. `/29`, `/19`) defines how many bits belong to the network — the remaining bits identify individual hosts. Point-to-point WAN links typically use `/30` (2 usable hosts), while LAN segments use wider prefixes depending on how many hosts they need to support.

**Routing** can be configured in two ways:

- **Static routing** — an administrator manually enters every route on every router. Simple and predictable, but does not scale and requires manual updates when the topology changes.
- **Dynamic routing (OSPF)** — routers automatically discover their neighbors, exchange link-state information, and converge on an optimal path. OSPF (Open Shortest Path First) is a link-state protocol that operates within *Areas*; Area 0 is the backbone to which all other areas must connect.

**NAT Overload (PAT — Port Address Translation)** allows an entire private network to share a single public IP address by mapping each internal session to a unique port number. This is how most home and enterprise networks access the internet today.

**Access Control Lists (ACLs)** are ordered rule sets applied to router interfaces that permit or deny traffic based on criteria such as source/destination IP, protocol, and port. Extended ACLs allow filtering on all four of these dimensions simultaneously, enabling fine-grained security policies.

***

## Repository Structure

Each lab lives in its own branch:

```
main                  ← this README
├── branch: lab1      ← Static Routing
└── branch: lab2      ← OSPF + NAT + ACLs
```

***

## Labs at a Glance

| Feature | Lab 1 — Static Routing | Lab 2 — OSPF + NAT + ACLs |
|---|---|---|
| **Routing type** | Manual static routes | OSPF dynamic (Area 0) |
| **Subnet prefixes used** | /29, /30 | /19, /30, /24 |
| **No. of routers** | 3 (Edge, Hermes, Bono) | 3 internal + ISP router |
| **Internet simulation** | ✗ | ✅ ISP router + DNS server |
| **NAT** | ✗ | ✅ PAT / Overload |
| **Access control** | ✗ | ✅ Extended ACLs (121, 104) |
| **Hosts / PCs** | 5 PCs + 1 server | 6 PCs + 1 DNS server |
| **Key verification tools** | `ping` | `ping`, `traceroute`, `show ip nat`, `show ip ospf neighbor` |
| **Primary skill** | VLSM, static `ip route` | OSPF convergence, PAT, traffic filtering |

***

## Lab 1 — Static Routing

**Topology:** Three routers (Edge, Hermes, Bono) interconnecting five `/29` LAN segments (A–E) and one server network, linked over `/30` point-to-point WAN interfaces.

**Key concept:** Every router holds explicit `ip route <network> <mask> <next-hop>` entries for all remote networks. While straightforward on a small fixed topology, this approach becomes unmanageable at scale — motivating the move to OSPF in Lab 2.

> 📁 See branch `lab1` for full configuration, IP table, and topology diagram.

***

## Lab 2 — OSPF, NAT Overload & ACLs

**Topology:** Same three-router backbone, extended with a simulated ISP connection (`143.233.173.0/30`) and a public DNS server at `8.8.8.8`. LAN segments now use `/19` prefixes (8190 usable hosts each).

**Key concepts:**

- **OSPF Area 0** — Bono acts as the central hub; `show ip ospf neighbor` confirms FULL adjacency with both Hermes and Edge. A static default route on Bono (`ip route 0.0.0.0 0.0.0.0 143.233.173.1`) handles internet-bound traffic not covered by OSPF.
- **NAT Overload** — configured on Bono's ISP-facing interface; `show ip nat translations` confirms multiple internal IPs (from different hosts) mapping to a single public IP, distinguished by port.
- **ACL 121** — restricts outbound internet traffic to HTTP only (`tcp any host 8.8.8.8 eq 80`), applied outbound on the ISP interface.
- **ACL 104** — limits inbound traffic toward Hermes's networks to ICMP and OSPF only, applied inbound on Hermes's serial interface toward Bono.

> 📁 See branch `lab2` for full configuration, IP table, and topology diagram.

***

## Skills Demonstrated

- IPv4 subnetting and VLSM across multiple prefix lengths
- Static route configuration on Cisco IOS
- OSPF single-area dynamic routing and neighbor state verification
- NAT Overload (PAT) for many-to-one address translation
- Extended ACL design for protocol- and port-level traffic filtering
- Internet simulation with ISP router and DNS server in Cisco Packet Tracer
