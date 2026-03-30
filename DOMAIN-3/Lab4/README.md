# CCNA 200-301 — Domain 3 Lab 4: IPv6 Static Routing and OSPFv3

## Lab Overview

**Domain:** 3 — IP Connectivity  
**Topic:** IPv6 Static Routing and OSPFv3  
**Difficulty:** Intermediate  
**Estimated Time:** 45–60 minutes  

---

## Topology

```
PC1 --- R1 ----------- R2 --- PC2

         |               |
     2001:db8:1::/64   2001:db8:2::/64
        LAN              LAN

         |---------------|
           2001:db8:12::/64
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

## IPv6 Addressing

| Device | Interface | IPv6 Address |
|--------|-----------|-------------|
| PC1 | NIC | 2001:db8:1::10/64 |
| PC2 | NIC | 2001:db8:2::10/64 |
| R1 | Gi0/0 (LAN) | 2001:db8:1::1/64 |
| R1 | Gi0/1 (WAN) | 2001:db8:12::1/64 |
| R2 | Gi0/0 (LAN) | 2001:db8:2::1/64 |
| R2 | Gi0/1 (WAN) | 2001:db8:12::2/64 |

**PC1 Default Gateway:** 2001:db8:1::1  
**PC2 Default Gateway:** 2001:db8:2::1  

---

## Part 1 — IPv6 Static Routing

### Enable IPv6 Routing (Required)
```
! Both routers
ipv6 unicast-routing
```

### Interface Configuration
```
! R1
interface GigabitEthernet0/0
 ipv6 address 2001:db8:1::1/64
 no shutdown
interface GigabitEthernet0/1
 ipv6 address 2001:db8:12::1/64
 no shutdown

! R2
interface GigabitEthernet0/0
 ipv6 address 2001:db8:2::1/64
 no shutdown
interface GigabitEthernet0/1
 ipv6 address 2001:db8:12::2/64
 no shutdown
```

### Static Routes
```
! R1 — route to R2's LAN
ipv6 route 2001:db8:2::/64 2001:db8:12::2

! R2 — route to R1's LAN
ipv6 route 2001:db8:1::/64 2001:db8:12::1
```

### Verification
```
show ipv6 interface brief
show ipv6 route
```

---

## Part 2 — OSPFv3

### Remove Static Routes
```
! R1
no ipv6 route 2001:db8:2::/64 2001:db8:12::2

! R2
no ipv6 route 2001:db8:1::/64 2001:db8:12::1
```

### OSPFv3 Configuration
```
! R1
ipv6 router ospf 1
 router-id 1.1.1.1
exit
interface GigabitEthernet0/0
 ipv6 ospf 1 area 0
exit
interface GigabitEthernet0/1
 ipv6 ospf 1 area 0

! R2
ipv6 router ospf 1
 router-id 2.2.2.2
exit
interface GigabitEthernet0/0
 ipv6 ospf 1 area 0
exit
interface GigabitEthernet0/1
 ipv6 ospf 1 area 0
```

### Verification
```
show ipv6 ospf neighbor
show ipv6 route
```

### Expected OSPFv3 Route
```
O   2001:DB8:1::/64 [110/2]
     via FE80::207:ECFF:FE92:8A02, GigabitEthernet0/1
```

Note: OSPFv3 uses **Link-Local addresses** as next-hop — this is normal.

---

## Key Concepts Covered

### IPv6 Address Format
- 128-bit address written in hexadecimal
- 8 groups of 4 hex digits separated by colons
- Leading zeros in each group can be omitted
- One consecutive run of all-zero groups can be replaced with `::`
- `::` can only be used **once** per address

### IPv6 Abbreviation Rules
```
2001:0db8:0000:0000:0000:0000:0000:0001
= 2001:db8::1

2001:0db8:0000:0000:0001:0000:0000:0001
= 2001:db8::1:0:0:1  (:: replaces longest run only)
```

### IPv6 Address Types

| Type | Prefix | Purpose |
|------|--------|---------|
| Global Unicast | 2000::/3 | Routable on internet |
| Link-Local | FE80::/10 | Local link only, auto-configured |
| Multicast | FF00::/8 | One-to-many, replaces broadcast |
| Loopback | ::1/128 | Equivalent to 127.0.0.1 |

### OSPFv2 vs OSPFv3 Comparison

| Feature | OSPFv2 (IPv4) | OSPFv3 (IPv6) |
|---------|--------------|--------------|
| Configuration | network statement | per interface |
| Next-hop | Global unicast | Link-Local address |
| Router-ID | auto from IPv4 | must configure manually |
| AD | 110 | 110 |
| Area concept | same | same |
| DR/BDR election | same | same |
| Neighbor states | same | same |

### Link-Local Addresses
- Automatically generated on every IPv6 enabled interface
- Start with `FE80::/10`
- Used for OSPF neighbor relationships and next-hop routing
- Not routable beyond the local link
- Always present regardless of global unicast configuration

### `ipv6 unicast-routing`
- **Required** on all routers to forward IPv6 packets between interfaces
- Without it the router drops IPv6 packets — will not route
- IPv6 equivalent of `ip routing` on a Layer 3 switch

---

## Lab Objectives

1. Enable `ipv6 unicast-routing` on both routers
2. Configure IPv6 addresses on all interfaces
3. Verify Link-Local addresses auto-generated with `show ipv6 interface brief`
4. Configure IPv6 static routes — both directions
5. Verify PC1 can ping PC2
6. Remove static routes and verify ping fails
7. Configure OSPFv3 per interface on both routers with manual router-id
8. Verify OSPFv3 neighbor state is FULL
9. Verify O route appears in `show ipv6 route`
10. Verify PC1 can ping PC2 via OSPFv3

---

## Exam Tips

- `ipv6 unicast-routing` is required — forgetting this is a common mistake
- OSPFv3 is configured **per interface** — no network statements
- Router-ID must be **manually configured** in OSPFv3 — no IPv4 to derive from
- OSPFv3 next-hop is always a **Link-Local address** — this is normal
- `::` can only be used **once** per IPv6 address
- There is **no broadcast** in IPv6 — multicast replaces it
- Link-Local addresses start with `FE80` — auto-generated, always present
- IPv6 loopback is `::1` — equivalent to `127.0.0.1` in IPv4
- AD for OSPFv3 is still **110** — same as OSPFv2

---

## CCNA Exam Objectives Covered

- 1.8 Configure and verify IPv6 addressing and prefix
- 1.9 Describe IPv6 address types
- 3.4 Configure and verify IPv4 and IPv6 static routing
- 3.5 Configure and verify single-area OSPFv2 (concepts apply to OSPFv3)

---

*Lab completed: March 2026*  
*Author: Kyle T. Crenshaw*  
*Certification target: CCNA 200-301 v1.1*
