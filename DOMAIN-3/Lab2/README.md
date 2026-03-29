# CCNA 200-301 — Domain 3 Lab 2: Default Routes and Floating Static Routes

## Lab Overview

**Domain:** 3 — IP Connectivity  
**Topic:** Default Routes and Floating Static Routes  
**Difficulty:** Intermediate  
**Estimated Time:** 45–60 minutes  

---

## Topology

```
PC1 --- R1 ----------- R2 --- R3 --- PC2
         \             |     /
          \------------+----/
           direct link (backup path)
                       |
                      ISP
                       |
                   8.8.8.8 (Loopback)
```

---

## Device List

| Device | Model | Role |
|--------|-------|------|
| R1 | Cisco 2911 | Edge router — primary and backup paths |
| R2 | Cisco 2911 | Core router — connects to ISP |
| R3 | Cisco 2911 | Edge router — alternate path |
| ISP | Cisco 2911 | Simulated internet gateway |
| PC1 | PC-PT | End host — LAN 1 |
| PC2 | PC-PT | End host — LAN 2 |

---

## IP Addressing

| Device | Interface | IP Address | Subnet Mask |
|--------|-----------|------------|-------------|
| PC1 | NIC | 10.0.1.10 | 255.255.255.0 |
| PC2 | NIC | 10.0.3.10 | 255.255.255.0 |
| R1 | Gi0/0 | 10.0.1.1 | 255.255.255.0 |
| R1 | Gi0/1 | 10.0.12.1 | 255.255.255.0 |
| R1 | Gi0/2 | 10.0.13.1 | 255.255.255.0 |
| R2 | Gi0/0 | 10.0.12.2 | 255.255.255.0 |
| R2 | Gi0/1 | 10.0.23.1 | 255.255.255.0 |
| R2 | Gi0/2 | 10.0.99.1 | 255.255.255.0 |
| R3 | Gi0/0 | 10.0.23.2 | 255.255.255.0 |
| R3 | Gi0/1 | 10.0.3.1 | 255.255.255.0 |
| R3 | Gi0/2 | 10.0.13.2 | 255.255.255.0 |
| ISP | Gi0/0 | 10.0.99.2 | 255.255.255.0 |
| ISP | Loopback0 | 8.8.8.8 | 255.255.255.255 |

---

## Key Configurations

### Default Routes

```
! R1 — primary default route to R2
ip route 0.0.0.0 0.0.0.0 10.0.12.2 1

! R1 — floating static backup route via R3 (AD 200)
ip route 0.0.0.0 0.0.0.0 10.0.13.2 200

! R2 — default route to ISP
ip route 0.0.0.0 0.0.0.0 10.0.99.2

! R3 — default route to R2
ip route 0.0.0.0 0.0.0.0 10.0.23.1

! ISP — summary route back to internal networks
ip route 10.0.0.0 255.0.0.0 10.0.99.1

! R2 — routes back to LAN segments
ip route 10.0.1.0 255.255.255.0 10.0.12.1
ip route 10.0.3.0 255.255.255.0 10.0.23.2
```

---

## Lab Objectives

1. Configure IP addresses on all interfaces and verify up/up status
2. Configure default routes on R1, R2, R3 and ISP
3. Verify PC1 can ping 8.8.8.8
4. Configure floating static route on R1 via R3 with AD 200
5. Verify only the primary route appears in `show ip route`
6. Simulate primary link failure by shutting down R1 Gi0/1
7. Verify floating static route takes over automatically
8. Restore primary link and verify failback

---

## Verification Commands

```
show ip route
show ip interface brief
ping 8.8.8.8 source GigabitEthernet0/0
```

---

## Key Concepts Covered

### Default Route
A route of last resort — matches any destination with no more specific entry in the routing table.
```
ip route 0.0.0.0 0.0.0.0 [next-hop]
```
Appears in routing table as `S*` with gateway of last resort set.

### Floating Static Route
A static route with a manually elevated AD so it stays hidden while a better route exists.
```
ip route 0.0.0.0 0.0.0.0 [next-hop] [AD]
```
Only installs in the routing table when the primary route disappears.

### Administrative Distance (AD) Table

| Route Source | AD |
|---|---|
| Connected | 0 |
| Static | 1 |
| EIGRP | 90 |
| OSPF | 110 |
| RIP | 120 |
| Floating Static (this lab) | 200 |
| Unusable | 255 |

**Lower AD = more trusted = installed in routing table.**

---

## Exam Tips

- `S*` in the routing table means static default route — the `*` indicates gateway of last resort
- Floating static AD must be **higher** than the primary route's AD to stay hidden
- Always verify return path — both directions must have valid routes
- `show ip route` only shows the **best** route per destination — floating static will not appear while primary is active
- Default route syntax: `ip route 0.0.0.0 0.0.0.0` — the two zeros mean "match everything"

---

## CCNA Exam Objectives Covered

- 3.2 Interpret the components of a routing table
- 3.3 Determine how a router makes a forwarding decision by default
- 3.4 Configure and verify IPv4 static routing
- 3.4a Default route
- 3.4b Network route
- 3.4c Host route
- 3.4d Floating static

---

*Lab completed: March 2026*  
*Author:Kyle T. Crenshaw
*Certification target: CCNA 200-301 v1.1*
