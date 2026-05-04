# Lab 2 — OSPF, NAT Overload & Access Control Lists

**Course:** Computer Networks | MSc Informatics, University of Piraeus  
**Tool:** Cisco Packet Tracer Student v6.2  
**Topics:** Dynamic routing (OSPF), NAT Overload (PAT), Extended ACLs,
Internet simulation

---

## Network Topology

Three internal routers (Edge, Hermes, Bono) connect six LAN/WAN segments
to a simulated ISP with a DNS server (8.8.8.8).

![Topology Diagram](./topology.png)

---

## IP Addressing Scheme

| Segment          | CIDR               | Default Gateway  | Host IP          | Subnet Mask     | Role        |
|------------------|--------------------|------------------|------------------|-----------------|-------------|
| PC A             | 172.16.160.0/19    | 172.16.160.1     | 172.16.160.2     | 255.255.224.0   | LAN segment |
| PC B             | 172.16.32.0/19     | 172.16.32.1      | 172.16.32.2      | 255.255.224.0   | LAN segment |
| PC C             | 172.16.64.0/19     | 172.16.64.1      | 172.16.64.2      | 255.255.224.0   | LAN segment |
| PC D             | 172.16.128.0/19    | 172.16.128.1     | 172.16.128.2     | 255.255.224.0   | LAN segment |
| PC E             | 172.16.96.0/19     | 172.16.96.1      | 172.16.96.2      | 255.255.224.0   | LAN segment |
| PC F             | 172.16.192.8/30    | 172.16.192.9     | 172.16.192.10    | 255.255.255.252 | WAN link    |
| Hermes–Bono      | 172.16.192.0/30    | 172.16.192.1     | 172.16.192.2     | 255.255.255.252 | WAN link    |
| Bono–Edge        | 172.16.192.4/30    | 172.16.192.5     | 172.16.192.6     | 255.255.255.252 | WAN link    |
| Bono–ISP         | 143.233.173.0/30   | 143.233.173.1    | 143.233.173.2    | 255.255.255.252 | Uplink/ISP  |
| ISP DNS Server   | 8.8.8.0/24         | 8.8.8.1          | 8.8.8.8          | 255.255.255.0   | DNS Server  |

**Design rationale:**  
- **/19** for LANs → 8190 usable hosts, appropriate for larger internal segments  
- **/30** for point-to-point WAN links → minimal address waste  
- **Public range (143.233.173.x)** used to simulate ISP uplink

---

## Dynamic Routing — OSPF (Area 0)

All three internal routers run OSPF in Area 0 (Backbone).
Bono acts as the central hub with FULL adjacency to both Hermes and Edge.

| Router | Router ID       | OSPF Networks Advertised               | Neighbors (FULL state) |
|--------|-----------------|----------------------------------------|------------------------|
| Hermes | 172.16.192.2    | B, C, E subnets + Hermes–Bono link     | Bono                   |
| Edge   | 172.16.192.9    | A, F subnets + Edge–Bono link          | Bono                   |
| Bono   | 172.16.192.5    | D subnet + both WAN links              | Hermes, Edge           |

Internet access is handled via a static default route on Bono:
```bash
ip route 0.0.0.0 0.0.0.0 143.233.173.1
```

---

## NAT Overload (PAT)

NAT Overload was configured on Bono to allow all internal hosts to share
a single public IP for internet access, differentiated by port number.

```bash
# Mark internal interfaces
ip nat inside   → Serial (Hermes, Edge links) + GigabitEthernet (PC D)
ip nat outside  → GigabitEthernet1/0 (ISP uplink)

# Define which addresses to translate
access-list 1 permit 172.16.0.0 0.0.255.255

# Enable PAT using the outside interface IP
ip nat inside source list 1 interface GigabitEthernet1/0 overload
```

`show ip nat translations` confirmed active ICMP translations with
distinct port numbers per internal host sharing one public IP.

---

## Access Control Lists (ACLs)

### Internet access — HTTP only (ACL 121)
Traffic from internal hosts to the ISP server is restricted to HTTP (port 80):
```bash
access-list 121 permit tcp any host 8.8.8.8 eq 80
interface GigabitEthernet1/0
  ip access-group 121 out
```

### Hermes networks — ICMP/OSPF only (ACL 104)
Networks behind Hermes are reachable from other routers only via
ping/traceroute (ICMP) and OSPF updates:
```bash
access-list 104 permit icmp any any
access-list 104 permit ospf any any
interface Serial3/0
  ip access-group 104 in
```

Verified: PC A (172.16.160.2) successfully pinged and tracerouted to
PC E (172.16.96.2) via Bono → Hermes. Non-ICMP traffic was blocked.

---

## Skills Demonstrated

- OSPF dynamic routing configuration and neighbor adjacency verification
- VLSM subnetting across /19, /30, /24 prefix lengths
- NAT Overload (PAT) for many-to-one address translation
- Extended ACLs for granular traffic filtering (protocol + port-level)
- Internet simulation with ISP router and DNS server in Packet Tracer
