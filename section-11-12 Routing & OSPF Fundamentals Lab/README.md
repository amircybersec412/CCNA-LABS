# Cisco Static Routing & OSPF Fundamentals Lab
A comprehensive hands-on lab demonstrating **static routing**, **host routes**, **longest match rule**, **floating static routes**, **Administrative Distance**, and **OSPF dynamic routing** — built and verified in **EVE-NG** with real Cisco IOS images.

## 🎯 Lab Objectives
- Configure **static network routes** between routers
- Understand and verify **host routes** (`/32` prefix)
- Demonstrate the **longest match rule** in route lookup
- Configure **floating static routes** with higher AD as backup
- Observe **Administrative Distance** priority between route sources
- Configure **OSPF** dynamic routing in Area 0
- Verify **OSPF neighbor adjacency** (`FULL` state)
- Understand **OSPF metric (cost)** calculation
- Verify **OSPF routing table**, **neighbor table**, and **database**
- Test end-to-end connectivity through multi-router topology

## 🖧 Topology Overview

VPC5 ──── R1 ──── R2 ──── R3 ──── VPC6
10.1.1.0/24 10.12.12.0/24 10.23.23.0/24 10.3.3.0/24
│
Loopback3
3.3.3.3/32

### Device Table
| Device | Role | Model | Key Function |
|--------|------|-------|--------------|
| R1 | Edge Router | Cisco 3725 | Connects to VPC5 + transit to R2 |
| R2 | Transit Router | Cisco 3725 | Connects R1 and R3 |
| R3 | Edge Router | Cisco 3725 | Connects to VPC6 + Loopback |
| VPC5 | Test Host | VPCS | Source host (10.1.1.0/24) |
| VPC6 | Test Host | VPCS | Destination host (10.3.3.0/24) |

## 🔗 Cabling Plan
| From | Interface | To | Interface |
|------|-----------|-----|-----------|
| R1 | Fa0/0 | R2 | Fa0/0 |
| R2 | Fa0/1 | R3 | Fa0/1 |
| R1 | Fa0/1 | VPC5 | eth0 |
| R3 | Fa0/0 | VPC6 | eth0 |

## 🖥️ IP Addressing
| Device | Interface | IP Address | Gateway | Purpose |
|--------|-----------|------------|---------|---------|
| R1 | Fa0/0 | 10.12.12.1/24 | — | Link to R2 |
| R1 | Fa0/1 | 10.1.1.1/24 | — | Link to VPC5 |
| R2 | Fa0/0 | 10.12.12.2/24 | — | Link to R1 |
| R2 | Fa0/1 | 10.23.23.2/24 | — | Link to R3 |
| R3 | Fa0/1 | 10.23.23.3/24 | — | Link to R2 |
| R3 | Fa0/0 | 10.3.3.1/24 | — | Link to VPC6 |
| R3 | Lo3 | 3.3.3.3/32 | — | Loopback (host route) |
| VPC5 | eth0 | 10.1.1.10/24 | 10.1.1.1 | Test host |
| VPC6 | eth0 | 10.3.3.10/24 | 10.3.3.1 | Test host |

## 📋 Configuration Files
### R1
en
conf ter
hostname R1
## ! Interfaces
interface fastEthernet 0/0
ip address 10.12.12.1 255.255.255.0
no shutdown
exit
interface fastEthernet 0/1
ip address 10.1.1.1 255.255.255.0
no shutdown
exit
## ! Static Routes
ip route 3.3.3.3 255.255.255.255 10.12.12.2
ip route 10.3.3.0 255.255.255.0 10.12.12.2
ip route 10.23.23.0 255.255.255.0 10.12.12.2
## ! Floating Static (backup)
ip route 10.3.3.0 255.255.255.0 fastEthernet 0/1 200
## ! OSPF
router ospf 1
network 10.1.1.0 0.0.0.255 area 0
network 10.12.12.0 0.0.0.255 area 0
exit
end
write memory

### R2
en
conf ter
hostname R2
interface fastEthernet 0/0
ip address 10.12.12.2 255.255.255.0
no shutdown
exit
interface fastEthernet 0/1
ip address 10.23.23.2 255.255.255.0
no shutdown
exit
! Static Routes
ip route 3.3.3.3 255.255.255.255 10.23.23.3
ip route 10.1.1.0 255.255.255.0 10.12.12.1
ip route 10.3.3.0 255.255.255.0 10.23.23.3
! OSPF
router ospf 1
network 10.12.12.0 0.0.0.255 area 0
network 10.23.23.0 0.0.0.255 area 0
exit
end
write memory
text

### R3
en
conf ter
hostname R3
interface fastEthernet 0/1
ip address 10.23.23.3 255.255.255.0
no shutdown
exit
interface fastEthernet 0/0
ip address 10.3.3.1 255.255.255.0
no shutdown
exit
interface loopback 3
ip address 3.3.3.3 255.255.255.255
exit
! Static Routes
ip route 10.1.1.0 255.255.255.0 10.23.23.2
ip route 10.12.12.0 255.255.255.0 10.23.23.2
! OSPF
router ospf 1
network 3.3.3.3 0.0.0.0 area 0
network 10.3.3.0 0.0.0.255 area 0
network 10.23.23.0 0.0.0.255 area 0
exit
end
write memory
text

### VPCs
! VPC5
ip 10.1.1.10/24 10.1.1.1
save
! VPC6
ip 10.3.3.10/24 10.3.3.1
save

## 🛠️ Technologies & Concepts
### Static Routing
| Feature | Detail |
|---------|--------|
| Definition | Manually configured route by admin |
| Default AD | 1 |
| Command | `ip route <network> <mask> <next-hop>` |
| Permanent keyword | Keeps route even if interface goes down |
| Use case | Small networks, stub networks |

### Network Route
- Route for an entire **classful network** or subnet
- Example: `ip route 10.3.3.0 255.255.255.0 10.12.12.2`
- Most routing table entries are network routes

### Host Route
- Route to a **single device** with `/32` mask (IPv4)
- Example: `ip route 3.3.3.3 255.255.255.255 10.12.12.2`
- **Auto-installed** when IP is configured on router interface (connected host route)
- More specific than network routes — wins in longest match

### Longest Match Rule
- Router chooses the route with the **longest prefix** (most specific)
- Example: `3.3.3.3` matches both `/8` and `/32` — the `/32` wins
- **Overrides all other criteria** — specificity wins first

### Floating Static Route
- Static route with **higher AD** than the primary route
- Stays inactive until the primary route fails
- Example: `ip route 10.3.3.0 255.255.255.0 fa0/1 200`
- Backup route for redundancy

### Administrative Distance (AD)
| Route Source | Default AD |
|--------------|-----------|
| Connected | 0 |
| Static | 1 |
| EIGRP Summary | 5 |
| External BGP | 20 |
| EIGRP | 90 |
| OSPF | 110 |
| IS-IS | 115 |
| RIP | 120 |
| External EIGRP | 170 |
| Internal BGP | 200 |
| Unknown | 255 |

**Lower AD = more trusted.**
### OSPF (Open Shortest Path First)
| Feature | Detail |
|---------|--------|
| Type | Link-State dynamic routing protocol |
| Standard | Open (RFC 2328) |
| Algorithm | Shortest Path First (Dijkstra / SPF) |
| IP Protocol | 89 (not TCP/UDP) |
| Multicast Addresses | 224.0.0.5 (all OSPF routers), 224.0.0.6 (DR/BDR) |
| Hello Timer | 10 seconds |
| Dead Timer | 40 seconds |
| AD | 110 |
| Metric | Cost (based on bandwidth) |
| Classes | Classless (supports VLSM) |
| Areas | Area 0 = backbone |
| Tables | Neighbor, Topology (LSDB), Routing |

### OSPF Metric (Cost)
- **Reference bandwidth:** 100 Mbps (default)
- **Cost formula:** Reference bandwidth ÷ Interface bandwidth
- FastEthernet cost = 10^8 / 10^7 = **10**
- Loopback cost = **1**
- End-to-end cost = sum of all link costs along the path

### OSPF Neighbor States
| State | Meaning |
|-------|---------|
| DOWN | No hello received |
| INIT | Hello received |
| 2WAY | Bidirectional (DR/BDR election) |
| EXSTART | Master/slave negotiation |
| EXCHANGE | DBD packets exchanged |
| LOADING | LSR/LSU exchange |
| **FULL** | **Adjacency complete** ✅ |

### OSPF DR/BDR Election
- Elected on **multi-access segments**
- Highest **Router Priority** wins (default = 1)
- Tie breaker: highest **Router ID**
- DR = Designated Router, BDR = Backup DR
- Other routers = DROTHER

### OSPF Router ID
- Highest active **loopback** IP address, OR
- Highest active **physical** interface IP, OR
- Manually configured with `router-id X.X.X.X`

### IP Routing Flow (Packet Handling)
1. PC checks if destination is on local subnet or remote
2. If remote, PC ARPs for its default gateway
3. Router receives frame, checks FCS and destination MAC
4. Router de-encapsulates IP packet from frame
5. Router looks up destination IP in routing table (longest match)
6. Router decrements TTL, recalculates checksum
7. Router encapsulates IP packet in new Layer 2 frame
8. Router rewrites source MAC (own) and destination MAC (next-hop)
9. Frame is transmitted toward destination
10. Process repeats hop by hop until packet reaches destination

### Routing Table Components
| Component | Description |
|-----------|-------------|
| Route Source | Code (C, S, O, D, R, B, etc.) |
| Destination Network | CIDR notation (e.g., 10.3.3.0/24) |
| Administrative Distance | Trustworthiness of route |
| Metric | Value assigned to reach remote network |
| Next Hop | IPv4 address of next router |
| Route Timestamp | Time since route learned |
| Outgoing Interface | Exit interface for packet |

### Metric Comparison
| Protocol | Metric |
|----------|--------|
| Static | Administrator decides |
| OSPF | Cost |
| RIP | Hop Count |
| EIGRP | Bandwidth + Delay |
| BGP | Path Counts |

## ✅ Verification Commands
### Static Route Verification
show ip route
show ip route static
show ip route <network>
show run | include ip route

### OSPF Verification
cisco
show ip route ospf
show ip ospf neighbor
show ip ospf database
show ip ospf
show ip ospf interface brief
show ip protocols
### OSPF Debug (use carefully)
cisco
debug ip ospf events
debug ip ospf packet
debug ip ospf adj
undebug all
### End-to-End Testing
bash
## ! From VPC5
show ip
ping 10.1.1.1
ping 10.12.12.2
ping 10.23.23.3
ping 10.3.3.1
ping 10.3.3.10
ping 3.3.3.3
trace 10.3.3.10

## Key Learnings
Static routes need to be configured on EVERY router in the path — not just the edge routers/
Host routes (/32) are auto-installed on router interfaces with IP addresses/
Longest match overrides all other criteria — specificity wins first/
Floating static routes provide backup without dynamic protocols/
AD determines trust between routing sources — lower is better/
OSPF uses cost as metric, based on bandwidth/
OSPF Router ID = highest loopback, else highest physical interface IP/
OSPF neighbors must match on subnet, area, timers, and authentication/
OSPF FULL state = adjacency complete, LSAs exchanged/
DR/BDR election reduces network traffic on multi-access segments/
Frame rewrite — MAC addresses change hop-by-hop, IP addresses stay the same/
TTL decrements at each router hop, preventing infinite loops

