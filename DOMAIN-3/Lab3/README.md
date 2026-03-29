# CCNA 200-301 — Domain 3 Lab 3: HSRP (Hot Standby Router Protocol)

## Lab Overview

**Domain:** 3 — IP Connectivity  
**Topic:** First Hop Redundancy — HSRP  
**Difficulty:** Intermediate  
**Estimated Time:** 30–45 minutes  

---

## Topology

```
PC1 --- SW1 --- R1 (Active)  --- ISP --- 8.8.8.8
         |                    /
         +------ R2 (Standby)-
```

---

## Device List

| Device | Model | Role |
|--------|-------|------|
| R1 | Cisco 2911 | HSRP Active router |
| R2 | Cisco 2911 | HSRP Standby router |
| SW1 | Cisco 2960 | Layer 2 switch |
| ISP | Cisco 2911 | Simulated internet gateway |
| PC1 | PC-PT | End host |

---

## IP Addressing

| Device | Interface | IP Address | Subnet Mask |
|--------|-----------|------------|-------------|
| PC1 | NIC | 10.0.1.10 | 255.255.255.0 |
| R1 | Gi0/0 (LAN) | 10.0.1.1 | 255.255.255.0 |
| R1 | Gi0/1 (WAN) | 10.0.12.1 | 255.255.255.0 |
| R2 | Gi0/0 (LAN) | 10.0.1.2 | 255.255.255.0 |
| R2 | Gi0/1 (WAN) | 10.0.13.2 | 255.255.255.0 |
| ISP | Gi0/0 | 10.0.12.3 | 255.255.255.0 |
| ISP | Gi0/1 | 10.0.13.1 | 255.255.255.0 |
| ISP | Loopback0 | 8.8.8.8 | 255.255.255.255 |
| **HSRP Virtual IP** | **—** | **10.0.1.254** | **255.255.255.0** |

**PC1 Default Gateway: 10.0.1.254 (HSRP Virtual IP — NOT R1 or R2's real IP)**

---

## HSRP Configuration

### R1 (Active)
```
interface GigabitEthernet0/0
 standby 1 ip 10.0.1.254
 standby 1 priority 110
 standby 1 preempt
```

### R2 (Standby)
```
interface GigabitEthernet0/0
 standby 1 ip 10.0.1.254
 standby 1 priority 90
 standby 1 preempt
```

---

## Lab Objectives

1. Configure IP addresses on all interfaces and verify up/up status
2. Set PC1 gateway to HSRP virtual IP 10.0.1.254
3. Configure HSRP on R1 and R2 with correct priorities
4. Verify HSRP state with `show standby brief`
5. Confirm R1 is Active and R2 is Standby
6. Simulate R1 failure by shutting down Gi0/0
7. Verify R2 takes over Active role automatically
8. Restore R1 and verify preempt returns Active role to R1

---

## Verification Commands

```
show standby brief
show standby
show ip interface brief
```

### Expected `show standby brief` output on R1 (Active):
```
Interface   Grp  Pri P State    Active   Standby    Virtual IP
Gi0/0       1    110 P Active   local    10.0.1.2   10.0.1.254
```

### Expected `show standby brief` output on R2 (Standby):
```
Interface   Grp  Pri P State    Active   Standby    Virtual IP
Gi0/0       1    90  P Standby  10.0.1.1 local      10.0.1.254
```

---

## Key Concepts Covered

### What Problem HSRP Solves
Without HSRP, PC1 has a single default gateway (e.g. R1's real IP). If R1 fails, PC1 still tries to send traffic to R1 — traffic stops with no automatic recovery. HSRP provides a **virtual IP shared between two routers** so end devices always have a reachable gateway regardless of which physical router is active.

### HSRP Election
| Factor | Rule |
|--------|------|
| Default priority | 100 |
| Higher priority | wins Active role |
| Tie-breaker | highest IP address |
| Preempt | allows recovered router to reclaim Active |

### HSRP vs Floating Static Route
| Feature | Floating Static | HSRP |
|---------|----------------|------|
| Type | Router redundancy | Gateway redundancy |
| Protects | Path between routers | Default gateway for end devices |
| Configured on | Routers | Routers (virtual IP used by PCs) |
| End device aware | No | No — transparent to PC |

### Preempt Behavior
- **With preempt:** Recovered higher-priority router automatically reclaims Active role
- **Without preempt:** Recovered router stays Standby even if it has higher priority
- Unlike OSPF DR/BDR election — HSRP **is preemptive** when configured with `standby 1 preempt`

### Failover Behavior
When Active router fails:
1. Standby router stops receiving Hello packets from Active
2. Dead timer expires
3. Standby transitions to Active and takes over virtual IP
4. Brief interruption (typically under 10 seconds) then traffic resumes
5. PC1 never changes its gateway — completely transparent

---

## HSRP Timers

| Timer | Default | Purpose |
|-------|---------|---------|
| Hello | 3 seconds | How often Active sends hellos |
| Hold | 10 seconds | How long before Standby declares Active dead |

---

## Exam Tips

- PC1 gateway must be set to the **virtual IP** — never R1 or R2's real IP
- Default HSRP priority is **100** — higher wins Active
- Tie-breaker is **highest IP address** — not Router ID like OSPF
- `standby 1 preempt` is required for automatic failback after recovery
- HSRP group number (the `1` in `standby 1`) must match on both routers
- `show standby brief` — P in the output means preempt is configured
- HSRP is Cisco proprietary — VRRP is the open standard equivalent

---

## CCNA Exam Objectives Covered

- 3.6 Configure and verify IPv4 and IPv6 static routing
- 3.7 Configure and verify single-area OSPFv2
- 3.8 Describe the purpose, functions, and concepts of first hop redundancy protocols (HSRP)

---

*Lab completed: March 2026*  
*Author: Kyle T. Crenshaw  
*Certification target: CCNA 200-301 v1.1*
