# CCNA 200-301 — Domain 3 Lab 1: Static Routing and OSPF

## Lab Overview

**Domain:** 3 — IP Connectivity  
**Topic:** Static Routing and OSPF Single Area  
**Difficulty:** Beginner–Intermediate  
**Estimated Time:** 45–60 minutes  

---

## Topology

```
PC1 --- R1 ----------- R2 --- PC2
         |               |
     10.0.1.0/24     10.0.2.0/24
        LAN              LAN

         |---------------|
           10.0.12.0/24
             WAN link
```

---

## Device List

| Device | Model | Role |
|--------|-------|------|
| R1 | Cisco 2911 | Left edge router |
| R2 | Cisco 2911 | Right edge router |
| PC1 | PC-PT | End host — LAN 1 |
| PC2 | PC-PT | End host — LAN 2 |

---

## IP Addressing

| Device | Interface | IP Address | Subnet Mask |
|--------|-----------|------------|-------------|
| PC1 | NIC | 10.0.1.10 | 255.255.255.0 |
| PC2 | NIC | 10.0.2.10 | 255.255.255.0 |
| R1 | Gi0/0 (LAN) | 10.0.1.1 | 255.255.255.0 |
| R1 | Gi0/1 (WAN) | 10.0.12.1 | 255.255.255.0 |
| R2 | Gi0/0 (LAN) | 10.0.2.1 | 255.255.255.0 |
| R2 | Gi0/1 (WAN) | 10.0.12.2 | 255.255.255.0 |

**PC1 Default Gateway:** 10.0.1.1  
**PC2 Default Gateway:** 10.0.2.1  

---

## Part 1 — Static Routing

### Key Configurations

```
! R1 — static route to R2's LAN
ip route 10.0.2.0 255.255.255.0 10.0.12.2

! R2 — static route to R1's LAN
ip route 10.0.1.0 255.255.255.0 10.0.12.1
```

### Why Both Routes Are Needed
Routing must work in **both directions**:
- R1 needs a route to 10.0.2.0/24 for the forward path (PC1 → PC2)
- R2 needs a route to 10.0.1.0/24 for the return path (PC2 → PC1)
- Missing either route causes the ping to fail even if one direction works

### Verification Commands

```
show ip route
show ip interface brief
ping 10.0.2.10 source GigabitEthernet0/0
```

---

## Part 2 — OSPF Single Area

### Removing Static Routes

```
! R1
no ip route 10.0.2.0 255.255.255.0 10.0.12.2

! R2
no ip route 10.0.1.0 255.255.255.0 10.0.12.1
```

### OSPF Configuration

```
! R1
router ospf 1
 network 10.0.1.0 0.0.0.255 area 0
 network 10.0.12.0 0.0.0.255 area 0

! R2
router ospf 1
 network 10.0.2.0 0.0.0.255 area 0
 network 10.0.12.0 0.0.0.255 area 0
```

### Verification Commands

```
show ip ospf neighbor
show ip route
show ip ospf interface
```

### Expected OSPF Route in Routing Table

```
O    10.0.2.0/24 [110/2] via 10.0.12.2, GigabitEthernet0/1
```

---

## Key Concepts Covered

### Static Routing
Manually configured routes. The router only knows what you explicitly tell it.
- Advantage: simple, low overhead, predictable
- Disadvantage: doesn't scale, no automatic failover, admin must update manually

### OSPF (Open Shortest Path First)
A dynamic routing protocol that automatically discovers and shares routing information between routers.
- Process ID is **local only** — does not need to match between routers
- Uses **wildcard masks** — the inverse of subnet masks
- Routers form **neighbor adjacencies** and share Link State Advertisements (LSAs)

### Wildcard Mask Calculation
```
255.255.255.255
-  subnet mask
= wildcard mask

Example: /24
255.255.255.255
- 255.255.255.0
= 0.0.0.255
```

### Administrative Distance (AD)

| Route Source | AD |
|---|---|
| Connected | 0 |
| Static | 1 |
| OSPF | 110 |

Lower AD = more trusted = installed in routing table.

### OSPF Route Interpretation
```
O    10.0.2.0/24 [110/2] via 10.0.12.2
```
- `O` — learned via OSPF
- `110` — administrative distance
- `2` — OSPF cost (based on bandwidth)

### DR/BDR Election
On multi-access segments OSPF elects a Designated Router (DR) and Backup Designated Router (BDR):
1. **Highest OSPF priority wins** — default is 1, range 0–255
2. **Tie-breaker: highest Router ID** — highest loopback or active interface IP
3. **Priority 0** — router can never become DR or BDR
4. **Non-preemptive** — a new higher-priority router does NOT take over from an existing DR

### OSPF Neighbor Relationships Formula
```
n(n-1)/2
```
Where n = number of routers in a full mesh.

| Routers | Relationships |
|---------|--------------|
| 3 | 3 |
| 4 | 6 |
| 5 | 10 |
| 6 | 15 |
| 10 | 45 |

---

## Lab Objectives

1. Configure IP addresses on all interfaces and verify up/up status
2. Configure static routes on R1 and R2
3. Verify PC1 can ping PC2 via static routing
4. Remove static routes and verify ping fails
5. Configure OSPF area 0 on both routers
6. Verify OSPF neighbor adjacency — state should be FULL
7. Verify O route appears in routing table
8. Simulate link failure by shutting down Gi0/1 on R1
9. Observe OSPF route disappears automatically
10. Restore link and verify OSPF reconverges

---

## Exam Tips

- Routers only know **directly connected networks** by default — everything else needs static or dynamic routing
- Routing must work in **both directions** — always check return path
- OSPF process ID is **local only** — mismatched process IDs between routers is fine
- Wildcard mask = 255.255.255.255 minus subnet mask — always
- `show ip ospf neighbor` state must be **FULL** for routes to be exchanged
- OSPF **does not preempt** — changing priority only takes effect after DR/BDR failure or OSPF reset
- Static routes with **lower AD than OSPF** override OSPF routes for the same destination

---

## CCNA Exam Objectives Covered

- 3.2 Interpret the components of a routing table
- 3.3 Determine how a router makes a forwarding decision by default
- 3.4 Configure and verify IPv4 static routing
- 3.5 Configure and verify single-area OSPFv2
- 3.5a Neighbor adjacencies
- 3.5b Point-to-point
- 3.5c Broadcast (DR/BDR selection)
- 3.5d Router ID

---

## Files in This Lab

| File | Description |
|------|-------------|
| static_routing.pkt | Packet Tracer file — static routing configuration |
| ospf.pkt | Packet Tracer file — OSPF single area configuration |
| README.md | This file |

---

*Lab completed: March 2026*  
*Author:Kyle T. Crenshaw
*Certification target: CCNA 200-301 v1.1*
