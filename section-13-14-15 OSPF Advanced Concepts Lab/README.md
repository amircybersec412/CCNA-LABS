# Cisco OSPF Advanced Concepts Lab
A comprehensive hands-on lab demonstrating **OSPF neighbor adjacency states**, **metric (cost) calculation**, **packet types**, **Router ID selection**, **router types**, **route types**, **DR/BDR election**, **equal-cost load balancing**, **path preference**, **Hello/Dead intervals**, **distribute-list filtering**, and **route summarization** — built and verified in **EVE-NG** with real Cisco IOS images.

## 🎯 Lab Objectives
- Configure OSPF in a multi-area topology (Area 0 backbone + Area 1)
- Observe **neighbor adjacency state transitions** (Down → Init → 2-Way → Exstart → Exchange → Loading → Full)
- Understand and verify **OSPF metric (cost)** calculation
- Capture all **5 OSPF packet types** with Wireshark (Hello, DBD, LSR, LSU, LSAck)
- Configure **OSPF Router ID** manually and observe effects
- Build and identify **Internal, Backbone, ABR, and ASBR routers**
- Identify **OSPF route types** in the routing table (O, O IA, O E1, O E2)
- Observe **DR/BDR election** with priorities and Router IDs
- Configure **equal-cost multi-path (ECMP)** load balancing
- Understand **OSPF path preference** order (O > O IA > E1 > E2)
- Change **Hello/Dead intervals** and observe timer mismatches
- Filter OSPF routes using **distribute-list** with ACLs
- Summarize routes at **ABR** and **ASBR**

## 🖧 Topology Overview
Area 0 (Backbone) Area 1
┌──────────────────────┐ ┌──────────────┐
│ │ │ │
[R1]────[R2]────[R3]────[R4] [R6]────[R7]────[R8]
ABR Backbone Backbone ASBR ABR Internal Internal
│ │ │
│ ┌───[R5]───┐ │ │
│ │ │ │ │
│ └──────────┘ │ │
│ │ │
└───────────────────────┴──────────┘
(R1↔R6 link carries Area 0)

### Device Table
| Device | Role | OSPF Area | Router Type | Router ID |
|--------|------|-----------|-------------|-----------|
| R1 | ABR + ASBR | 0 & 1 | Area Border Router | 1.1.1.1 |
| R2 | Backbone | 0 | Backbone Router | 2.2.2.2 |
| R3 | Backbone | 0 | Backbone Router | 3.3.3.3 |
| R4 | ASBR | 0 | Autonomous System Boundary Router | 4.4.4.4 |
| R5 | Backbone | 0 | Backbone Router | 5.5.5.5 |
| R6 | ABR | 0 & 1 | Area Border Router | 6.6.6.6 |
| R7 | Internal | 1 | Internal Router | 7.7.7.7 |
| R8 | Internal | 1 | Internal Router | 8.8.8.8 |

## 🔗 Cabling Plan
| From | Interface | To | Interface | Purpose |
|------|-----------|-----|-----------|---------|
| R1 | Fa0/0 | R2 | Fa0/0 | Area 0 backbone |
| R2 | Fa0/1 | R3 | Fa0/1 | Area 0 backbone |
| R3 | Fa0/0 | R4 | Fa0/0 | Area 0 backbone |
| R2 | Fa1/0 | R5 | Fa1/0 | Alternate Area 0 path |
| R5 | Fa0/1 | R4 | Fa0/1 | Alternate Area 0 path |
| R1 | Fa0/1 | R6 | Fa0/1 | Area 0 (ABR link) |
| R6 | Fa0/0 | R7 | Fa0/0 | Area 1 |
| R7 | Fa0/1 | R8 | Fa0/1 | Area 1 |

## 🖥️ IP Addressing
| Device | Interface | IP Address | Area | Purpose |
|--------|-----------|------------|------|---------|
| R1 | Fa0/0 | 192.168.12.1/24 | 0 | Link to R2 |
| R1 | Fa0/1 | 192.168.16.1/24 | 0 | Link to R6 |
| R1 | Lo1 | 172.16.1.1/24 | 0 | Summarization test |
| R1 | Lo2 | 172.16.2.1/24 | 0 | Summarization test |
| R1 | Lo3 | 172.16.3.1/24 | 0 | Summarization test |
| R1 | Lo4 | 172.16.4.1/24 | 0 | Summarization test |
| R2 | Fa0/0 | 192.168.12.2/24 | 0 | Link to R1 |
| R2 | Fa0/1 | 192.168.23.2/24 | 0 | Link to R3 |
| R2 | Fa1/0 | 192.168.25.2/24 | 0 | Link to R5 |
| R2 | Lo2 | 2.2.2.2/32 | 0 | Router ID |
| R3 | Fa0/1 | 192.168.23.3/24 | 0 | Link to R2 |
| R3 | Fa0/0 | 192.168.34.3/24 | 0 | Link to R4 |
| R3 | Lo3 | 3.3.3.3/32 | 0 | Router ID |
| R4 | Fa0/0 | 192.168.34.4/24 | 0 | Link to R3 |
| R4 | Fa0/1 | 192.168.45.4/24 | 0 | Link to R5 |
| R4 | Lo4 | 4.4.4.4/32 | 0 | Router ID |
| R5 | Fa1/0 | 192.168.25.5/24 | 0 | Link to R2 |
| R5 | Fa0/1 | 192.168.45.5/24 | 0 | Link to R4 |
| R5 | Lo5 | 5.5.5.5/32 | 0 | Router ID |
| R6 | Fa0/1 | 192.168.16.6/24 | 0 | Link to R1 |
| R6 | Fa0/0 | 192.168.67.6/24 | 1 | Link to R7 |
| R6 | Lo6 | 6.6.6.6/32 | 1 | Router ID |
| R7 | Fa0/0 | 192.168.67.7/24 | 1 | Link to R6 |
| R7 | Fa0/1 | 192.168.78.7/24 | 1 | Link to R8 |
| R7 | Lo7 | 7.7.7.7/32 | 1 | Router ID |
| R8 | Fa0/1 | 192.168.78.8/24 | 1 | Link to R7 |
| R8 | Lo8 | 8.8.8.8/32 | 1 | Router ID |

## 📋 Configuration Files
### R1 — ABR + ASBR
en
conf ter
hostname R1
interface fastEthernet 0/0
ip address 192.168.12.1 255.255.255.0
no shutdown
exit
interface fastEthernet 0/1
ip address 192.168.16.1 255.255.255.0
no shutdown
exit
interface loopback 1
ip address 172.16.1.1 255.255.255.0
exit
interface loopback 2
ip address 172.16.2.1 255.255.255.0
exit
interface loopback 3
ip address 172.16.3.1 255.255.255.0
exit
interface loopback 4
ip address 172.16.4.1 255.255.255.0
exit
router ospf 1
router-id 1.1.1.1
network 192.168.12.0 0.0.0.255 area 0
network 192.168.16.0 0.0.0.255 area 0
network 172.16.1.0 0.0.0.255 area 0
network 172.16.2.0 0.0.0.255 area 0
network 172.16.3.0 0.0.0.255 area 0
network 172.16.4.0 0.0.0.255 area 0
area 0 range 172.16.0.0 255.255.0.0
exit
end
write memory

### R2 — Backbone Router
en
conf ter
hostname R2
interface fastEthernet 0/0
ip address 192.168.12.2 255.255.255.0
no shutdown
exit
interface fastEthernet 0/1
ip address 192.168.23.2 255.255.255.0
no shutdown
exit
interface fastEthernet 1/0
ip address 192.168.25.2 255.255.255.0
no shutdown
exit
interface loopback 2
ip address 2.2.2.2 255.255.255.255
exit
router ospf 1
router-id 2.2.2.2
network 192.168.12.0 0.0.0.255 area 0
network 192.168.23.0 0.0.0.255 area 0
network 192.168.25.0 0.0.0.255 area 0
network 2.2.2.2 0.0.0.0 area 0
exit
end
write memory

### R3 — Backbone Router
en
conf ter
hostname R3
interface fastEthernet 0/1
ip address 192.168.23.3 255.255.255.0
no shutdown
exit
interface fastEthernet 0/0
ip address 192.168.34.3 255.255.255.0
no shutdown
exit
interface loopback 3
ip address 3.3.3.3 255.255.255.255
exit
router ospf 1
router-id 3.3.3.3
network 192.168.23.0 0.0.0.255 area 0
network 192.168.34.0 0.0.0.255 area 0
network 3.3.3.3 0.0.0.0 area 0
exit
end
write memory

### R4 — ASBR (OSPF + RIP)
en
conf ter
hostname R4
interface fastEthernet 0/0
ip address 192.168.34.4 255.255.255.0
no shutdown
exit
interface fastEthernet 0/1
ip address 192.168.45.4 255.255.255.0
no shutdown
exit
interface loopback 4
ip address 4.4.4.4 255.255.255.255
exit
router ospf 1
router-id 4.4.4.4
network 192.168.34.0 0.0.0.255 area 0
network 4.4.4.4 0.0.0.0 area 0
redistribute rip subnets metric-type 1
summary-address 5.0.0.0 255.0.0.0
exit
router rip
version 2
no auto-summary
network 192.168.45.0
network 4.0.0.0
exit
end
write memory

### R5 — Backbone with RIP
en
conf ter
hostname R5
interface fastEthernet 1/0
ip address 192.168.25.5 255.255.255.0
no shutdown
exit
interface fastEthernet 0/1
ip address 192.168.45.5 255.255.255.0
no shutdown
exit
interface loopback 5
ip address 5.5.5.5 255.255.255.255
exit
router ospf 1
router-id 5.5.5.5
network 192.168.25.0 0.0.0.255 area 0
network 5.5.5.5 0.0.0.0 area 0
exit
router rip
version 2
no auto-summary
network 192.168.45.0
network 5.0.0.0
exit
end
write memory

### R6 — ABR (Area 0 + Area 1)
en
conf ter
hostname R6
interface fastEthernet 0/1
ip address 192.168.16.6 255.255.255.0
no shutdown
exit
interface fastEthernet 0/0
ip address 192.168.67.6 255.255.255.0
no shutdown
exit
interface loopback 6
ip address 6.6.6.6 255.255.255.255
exit
router ospf 1
router-id 6.6.6.6
network 192.168.16.0 0.0.0.255 area 0
network 192.168.67.0 0.0.0.255 area 1
network 6.6.6.6 0.0.0.0 area 1
exit
end
write memory

### R7 — Internal Router (Area 1)
en
conf ter
hostname R7
interface fastEthernet 0/0
ip address 192.168.67.7 255.255.255.0
no shutdown
exit
interface fastEthernet 0/1
ip address 192.168.78.7 255.255.255.0
no shutdown
exit
interface loopback 7
ip address 7.7.7.7 255.255.255.255
exit
router ospf 1
router-id 7.7.7.7
network 192.168.67.0 0.0.0.255 area 1
network 192.168.78.0 0.0.0.255 area 1
network 7.7.7.7 0.0.0.0 area 1
exit
end
write memory

### R8 — Internal Router (Area 1)
en
conf ter
hostname R8
interface fastEthernet 0/1
ip address 192.168.78.8 255.255.255.0
no shutdown
exit
interface loopback 8
ip address 8.8.8.8 255.255.255.255
exit
router ospf 1
router-id 8.8.8.8
network 192.168.78.0 0.0.0.255 area 1
network 8.8.8.8 0.0.0.0 area 1
exit
end
write memory

## 🛠️ Technologies & Concepts
### 1. OSPF Neighbor Adjacency States
OSPF routers go through **seven states** before forming a full adjacency.
| State | Description |
|-------|-------------|
| **Down** | No hello received yet |
| **Init** | Hello received, but bidirectional communication not established |
| **2-Way** | Bidirectional hello exchange; DR/BDR election happens |
| **Exstart** | Master/slave negotiation begins; DBD exchange starts |
| **Exchange** | Full DBD packets exchanged |
| **Loading** | LSR/LSU/LSAck exchange to sync databases |
| **Full** | Adjacency complete; LSDBs are synchronized |
**Debug command:** `debug ip ospf adj`
**Reset command:** `clear ip ospf process`
### 2. OSPF Metric (Cost)
OSPF uses **cost** as its metric, calculated as:
Cost = Reference Bandwidth ÷ Interface Bandwidth

| Interface Type | Bandwidth | Default Cost |
|----------------|-----------|--------------|
| Ethernet | 10 Mbps | 10 |
| FastEthernet | 100 Mbps | 1 |
| GigabitEthernet | 1 Gbps | 1 |
| Serial | 1544 Kbps | 64 |
| Loopback | — | 1 |

**Reference bandwidth:** 100 Mbps (default)
The router selects the path with the **lowest cumulative cost**. Cost is counted from **outgoing interfaces** along the path.
**Verification:** `show ip ospf interface` or `show ip route ospf`

### 3. OSPF Packet Types
OSPF uses **5 packet types**:
| Type | Name | Multicast | Purpose |
|------|------|-----------|---------|
| **1** | Hello | 224.0.0.5 | Discover and maintain neighbors |
| **2** | DBD (Database Description) | Unicast | Summarize LSDB |
| **3** | LSR (Link State Request) | Unicast | Request missing LSAs |
| **4** | LSU (Link State Update) | Unicast/224.0.0.6 | Flood LSAs to neighbors |
| **5** | LSAck (Link State Acknowledgment) | Unicast | Acknowledge received LSAs |

**OSPF multicast MAC:** `01:00:5e:00:00:05`
**OSPF multicast IP:** `224.0.0.5` (AllSPFRouters), `224.0.0.6` (AllDRouters)

### 4. OSPF Router ID (RID)
The **Router ID** is a unique 32-bit identifier for each OSPF router.
**Selection order:**
1. **Manually configured** via `router-id X.X.X.X` command
2. **Highest IP** on a **loopback** interface
3. **Highest IP** on a **physical** interface
**Best practice:** Configure RID manually or use loopback interfaces for stability.
**Verification:** `show ip ospf | include Router ID`

### 5. OSPF Router Types
| Router Type | Definition |
|-------------|------------|
| **Internal Router** | All OSPF interfaces in the same area |
| **Backbone Router** | At least one interface in Area 0 |
| **Area Border Router (ABR)** | At least one interface in Area 0 and one in another area |
| **Autonomous System Boundary Router (ASBR)** | One interface in OSPF and one in another routing protocol (RIP, EIGRP, BGP) |
**Verification:**
- ABR: `show ip ospf | include area border`
- ASBR: `show ip ospf | include autonomous`
### 6. OSPF Route Types
| Code | Type | Preference |
|------|------|-----------|
| **O** | Intra-Area | 1st (highest) |
| **O IA** | Inter-Area | 2nd |
| **O E1** | External Type 1 | 3rd |
| **O E2** | External Type 2 | 4th |
| **O N1** | NSSA External Type 1 | 5th |
| **O N2** | NSSA External Type 2 | 6th |

**Preference order:** `O > O IA > E1 > E2 > N1 > N2`
- **E1:** Cumulative cost (external + internal)
- **E2:** Static external cost
- Intra-area routes always preferred over inter-area, regardless of cost

### 7. OSPF DR/BDR Election
On multi-access segments, OSPF elects a **Designated Router (DR)** and **Backup DR (BDR)**.
**Election criteria:**
1. Highest **OSPF priority** wins (default = 1, range 0-255)
2. Priority **0** = never DR/BDR
3. Tiebreaker: highest **Router ID**
4. **No preemption** — DR stays DR until failure
**Multicast addresses:**
- DR sends updates to `224.0.0.5` (AllSPFRouters)
- Other routers send to `224.0.0.6` (AllDRouters)
**Verification:** `show ip ospf neighbor` (State = FULL/DR, FULL/BDR, FULL/DROTHER)

### 8. OSPF Equal-Cost Load Balancing
OSPF supports **Equal-Cost Multi-Path (ECMP)** load balancing across multiple paths with the **same metric**.
- By default, OSPF installs up to **4 equal-cost paths**
- Can be increased with `maximum-paths N` (up to **16** or **32**)
- Traffic is load-balanced 50/50 between equal-cost paths

**Verification:** `show ip route <network>` shows multiple next-hops
### 9. OSPF Path Preference
OSPF uses a **two-tier selection**:
1. **Type of path** (O > O IA > E1 > E2)
2. **Metric (cost)**
**Preference order:** `O > O IA > E1 > E2 > N1 > N2`
**Key rule:** Intra-area routes are always preferred over inter-area, regardless of cost.
### 10. OSPF Hello & Dead Intervals
OSPF uses two timers to maintain neighbor relationships.
| Network Type | Hello Interval | Dead Interval |
|--------------|----------------|---------------|
| Broadcast | 10 sec | 40 sec |
| Point-to-Point | 10 sec | 40 sec |
| NBMA | 30 sec | 120 sec |
- **Dead Interval** = 4 × Hello Interval
- Both timers **must match** on both sides of a link for adjacency
**Change timers:** `ip ospf hello-interval N` / `ip ospf dead-interval N`

### 11. OSPF Filtering with Distribute-List
Steps to filter OSPF routes:
1. **Define which routes to filter**
2. **Create an ACL** to match those routes
3. **Create a distribute-list** referencing the ACL with a direction (in/out)
4. **Verify** the route is removed

**Configuration:**
ip access-list standard Block_R4
deny host 4.4.4.4
permit any
router ospf 1
distribute-list Block_R4 in

**Note:** The LSA is still in the database — only the routing table entry is removed.
### 12. OSPF Summarization at ABR & ASBR
OSPF summarization consolidates multiple routes into one advertisement. Cannot be done within an area.
**Inter-Area Summarization (ABR):**
router ospf 1
area <area-id> range <network> <mask>

**External Summarization (ASBR):**
router ospf 1
summary-address <network> <mask>

**OSPF can summarize LSA type 3 and 5 only.**
**Benefits:** Saves memory, bandwidth, CPU cycles; improves stability.

## ✅ Verification Commands
### Per-Router Verification
! OSPF neighbors
show ip ospf neighbor
! OSPF routes
show ip route ospf
! OSPF database
show ip ospf database
! OSPF process
show ip ospf
! OSPF interfaces
show ip ospf interface
show ip ospf interface brief
! Routing protocols
show ip protocols
! RIP database (on ASBR)
show ip rip database
! Interface cost
show ip ospf interface <interface> | include Cost
! Maximum paths
show ip ospf | include Maximum

### OSPF Debug Commands
debug ip ospf adj
debug ip ospf events
debug ip ospf packet
undebug all

### Wireshark Filters
| Filter | Shows |
|--------|-------|
| `ospf` | All OSPF packets |
| `ospf.msg.type == 1` | Hello packets |
| `ospf.msg.type == 2` | DBD packets |
| `ospf.msg.type == 3` | LSR packets |
| `ospf.msg.type == 4` | LSU packets |
| `ospf.msg.type == 5` | LSAck packets |

## 🧪 Test Results Summary
| Test | Result |
|------|--------|
| OSPF neighbor adjacency (all 8 routers) | ✅ FULL state |
| Area 0 backbone functioning | ✅ |
| Area 1 functioning | ✅ |
| ABR (R1, R6) inter-area routing | ✅ |
| ASBR (R4) RIP redistribution | ✅ |
| Internal Router (R7, R8) | ✅ |
| Backbone Router (R2, R3, R5) | ✅ |
| Route types visible (O, O IA, E1) | ✅ |
| DR/BDR election | ✅ |
| Equal-cost load balancing (ECMP) | ✅ |
| Path preference order | ✅ |
| Hello/Dead timer changes | ✅ |
| Distribute-list filtering | ✅ |
| ABR summarization (172.16.0.0/16) | ✅ |
| ASBR summarization (5.0.0.0/8) | ✅ |
| Wireshark captures — all 5 packet types | ✅ |

## 📌 Lab Summary
This lab demonstrates a **production-grade multi-area OSPF topology** with all advanced concepts:
**OSPF Fundamentals:**
- Multi-area topology (Area 0 + Area 1)
- Neighbor adjacency states
- DR/BDR election
- Metric (cost) calculation
- Router ID selection
**Advanced OSPF:**
- ABR and ASBR router types
- Route types (O, O IA, E1, E2)
- Equal-cost load balancing
- Path preference
- Hello/Dead timer management
- Distribute-list filtering
- Route summarization
**Verification:**
- All 8 routers forming FULL adjacencies
- Wireshark captures of all 5 OSPF packet types
- Route type visibility across ABR and ASBR boundaries
- RIP-to-OSPF redistribution

**Built as part of CCNA/CCNP study path** — October 2026
