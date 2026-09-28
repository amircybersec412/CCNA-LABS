# Cisco STP & Routing Fundamentals Lab
A comprehensive hands-on lab demonstrating **Spanning Tree Protocol (STP)**, **PVST+**, **Rapid PVST+ (RSTP)**, **BPDU operation**, **Root Bridge election**, **port roles/states**, and **IPv4 static/default/floating routing** — built and verified in **EVE-NG** with real Cisco IOS images.

## 🎯 Lab Objectives
- Build a redundant Layer 2 topology and observe STP behavior
- Understand **Root Bridge election** (Bridge Priority + MAC)
- Identify **port roles**: Root, Designated, Alternate, Blocking
- Observe **port states**: Blocking, Listening, Learning, Forwarding
- Compare **PVST+** (slow convergence) vs **Rapid PVST+** (fast convergence)
- Manipulate **Root Bridge** per VLAN for load balancing
- Manipulate **port cost** and **port priority** to influence path selection
- Capture and analyze **BPDUs** (`debug spanning-tree events`)
- Configure **static routes**, **default routes**, and **floating static routes**
- Understand **routed protocols** (IPv4) vs **routing protocols** (RIP/OSPF/EIGRP)
- Verify **Administrative Distance** behavior with floating static routes

## 🖧 Topology Overview
R1
/
/
SW1──────SW2
| \ / |
| \ / |
| SW3 |
| / \ |
| / \ |
SW4──────SW5
| |
VPC7 VPC8
VLAN 10 VLAN 20

### Device Table
| Device | Role | Model | Key Function |
|--------|------|-------|--------------|
| SW1 | Core Switch | Cisco IOU L2 | Root Bridge for VLAN 10 |
| SW2 | Core Switch | Cisco IOU L2 | Root Bridge for VLAN 20 |
| SW3 | Distribution | Cisco IOU L2 | Intermediate STP transit |
| SW4 | Access Switch | Cisco IOU L2 | VLAN 10 access, STP blocking |
| SW5 | Access Switch | Cisco IOU L2 | VLAN 20 access, STP blocking |
| R1 | Edge Router | Cisco 3725 | Inter-VLAN routing + static routes |
| VPC7 | Test Host | VPCS | VLAN 10 client |
| VPC8 | Test Host | VPCS | VLAN 20 client |

## 🌐 VLAN Design
| VLAN | Name | Subnet | Purpose |
|------|------|--------|---------|
| 1 | Default | — | Default VLAN |
| 10 | Users | 192.168.10.0/24 | User traffic |
| 20 | Servers | 192.168.20.0/24 | Server traffic |
| 99 | Management | — | Native VLAN |

## 🖥️ IP Addressing
| Device | Interface | IP Address | Gateway | VLAN |
|--------|-----------|------------|---------|------|
| VPC7 | eth0 | 192.168.10.10/24 | 192.168.10.1 | 10 |
| VPC8 | eth0 | 192.168.20.10/24 | 192.168.20.1 | 20 |
| R1 | Fa0/0 | 192.168.10.1/24 | — | 10 |
| R1 | Fa0/1 | 192.168.20.1/24 | — | 20 |

## 🔗 Cabling Plan
| From | Interface | To | Interface |
|------|-----------|-----|-----------|
| R1 | Fa0/0 | SW1 | e0/0 |
| R1 | Fa0/1 | SW2 | e0/1 |
| SW1 | e0/1 | SW3 | e0/1 |
| SW1 | e0/2 | SW2 | e0/2 |
| SW1 | e0/3 | SW4 | e0/3 |
| SW2 | e0/0 | SW3 | e0/0 |
| SW2 | e0/3 | SW5 | e0/3 |
| SW3 | e0/2 | SW4 | e0/2 |
| SW3 | e1/0 | SW5 | e1/0 |
| SW4 | e0/0 | SW5 | e0/0 |
| SW4 | e0/1 | VPC7 | eth0 |
| SW5 | e0/1 | VPC8 | eth0 |

## 📋 Configuration Files
### SW1 — Root Bridge for VLAN 10
en
conf ter
hostname SW1
vlan 10
name Users
vlan 20
name Servers
vlan 99
name Management
exit
! Access port to R1
interface e0/0
switchport mode access
switchport access vlan 10
no shutdown
exit
! Trunks to other switches
interface range e0/1-3
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,99
no shutdown
exit
! Set as Root Bridge for VLAN 10
spanning-tree vlan 10 root primary
! Security
spanning-tree etherchannel guard misconfig
end
write memory

### SW2 — Root Bridge for VLAN 20
en
conf ter
hostname SW2
vlan 10
name Users
vlan 20
name Servers
vlan 99
name Management
exit
interface e0/1
switchport mode access
switchport access vlan 20
no shutdown
exit
interface range e0/0, e0/2-3
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,99
no shutdown
exit
! Set as Root Bridge for VLAN 20
spanning-tree vlan 20 root primary
spanning-tree etherchannel guard misconfig
end
write memory

### SW3 — Distribution Switch
en
conf ter
hostname SW3
vlan 10
name Users
vlan 20
name Servers
vlan 99
name Management
exit
interface range e0/0-2, e1/0
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,99
no shutdown
exit
spanning-tree etherchannel guard misconfig
end
write memory

### SW4 — Access Switch (VLAN 10)
en
conf ter
hostname SW4
vlan 10
name Users
vlan 20
name Servers
vlan 99
name Management
exit
interface range e0/0, e0/2-3
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,99
no shutdown
exit
interface e0/1
switchport mode access
switchport access vlan 10
no shutdown
exit
spanning-tree etherchannel guard misconfig
end
write memory

### SW5 — Access Switch (VLAN 20)
en
conf ter
hostname SW5
vlan 10
name Users
vlan 20
name Servers
vlan 99
name Management
exit
interface range e0/0, e0/3, e1/0
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,99
no shutdown
exit
interface e0/1
switchport mode access
switchport access vlan 20
no shutdown
exit
spanning-tree etherchannel guard misconfig
end
write memory

### R1 — Router with Static/Default/Floating Routes
en
conf ter
hostname R1
! Physical interfaces
interface fastEthernet 0/0
ip address 192.168.10.1 255.255.255.0
no shutdown
exit
interface fastEthernet 0/1
ip address 192.168.20.1 255.255.255.0
no shutdown
exit
! Static route (AD=1)
ip route 10.0.0.0 255.255.255.0 fastEthernet 0/0
! Default route (gateway of last resort)
ip route 0.0.0.0 0.0.0.0 192.168.10.254
! Floating static route (AD=200 — backup)
ip route 10.0.0.0 255.255.255.0 fastEthernet 0/1 200
end
write memory

### VPCs
! VPC7
ip 192.168.10.10/24 192.168.10.1
save
! VPC8
ip 192.168.20.10/24 192.168.20.1
save

## 🛠️ Technologies & Concepts
### Spanning Tree Protocol (STP)
| Feature | Detail |
|---------|--------|
| Standard | IEEE 802.1D |
| Purpose | Prevent Layer 2 loops in redundant topologies |
| Convergence | Slow — 30–50 seconds |
| BPDU Interval | Every 2 seconds |
| BPDU Multicast | 01:80:C2:00:00:00 |
| Port States | Blocking, Listening, Learning, Forwarding, Disabled |
| Port Roles | Root, Designated, Non-Designated |

### PVST+ (Per-VLAN Spanning Tree Plus)
- Cisco proprietary implementation
- Runs **one STP instance per VLAN**
- Supports **802.1Q trunking**
- Default STP mode on Cisco switches
- Enables **per-VLAN load balancing**

### Rapid PVST+ (RSTP — IEEE 802.1w)
- Cisco implementation of RSTP
- **Convergence < 10 seconds**
- **3 port states:** Discarding, Learning, Forwarding
- **4 port roles:** Root, Designated, Alternate, Backup
- **UplinkFast and BackboneFast built-in** (no separate config)

### Root Bridge Election
1. Lowest **Bridge Priority** wins (default 32768)
2. If priority ties, lowest **MAC address** wins
3. Bridge Priority increments in steps of 4096
4. `spanning-tree vlan X root primary` sets priority to 24576
5. `spanning-tree vlan X root secondary` sets priority to 28672

### Path Cost
| Bandwidth | Short Mode | Long Mode |
|-----------|------------|-----------|
| 10 Mbps | 100 | 2,000,000 |
| 100 Mbps | 19 | 200,000 |
| 1 Gbps | 4 | 20,000 |
| 10 Gbps | 2 | 2,000 |

### Root Port Selection Order
1. Lowest **accumulated path cost** to Root Bridge
2. Lowest sender **Bridge Priority**
3. Lowest sender **MAC address**
4. Lowest sender **Port Priority**
5. Lowest sender **Port Number**

### Port States vs Port Roles
| Port State | Forwards BPDUs | Forwards Data | Learns MACs |
|------------|---------------|---------------|-------------|
| Blocking | YES | NO | NO |
| Listening | YES | NO | NO |
| Learning | YES | NO | YES |
| Forwarding | YES | YES | YES |
| Disabled | NO | NO | NO |

### Routed vs Routing Protocols
| Category | Examples | Purpose |
|----------|----------|---------|
| **Routed Protocol** | IPv4, IPv6 | Carries user data |
| **Routing Protocol** | RIP, OSPF, EIGRP, BGP | Discovers paths |

### Static Routing
- Manually configured by admin
- Default AD = 1
- Not scalable; no VLSM support
- `ip route network mask next-hop [AD]`

### Default Route
- Special static route: `0.0.0.0 0.0.0.0`
- Used when no specific route matches
- Gateway of last resort
- `ip route 0.0.0.0 0.0.0.0 <next-hop>`

### Floating Static Route
- Backup static route with **higher AD**
- Only installed when primary route fails
- `ip route network mask next-hop 200`

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

## ✅ Verification Commands
### STP Verification
! Summary — shows mode, root bridge role, blocking ports per VLAN
show spanning-tree summary
! Per-VLAN topology
show spanning-tree vlan 10
show spanning-tree vlan 20
! Detailed BPDU info
show spanning-tree vlan 10 detail
! Port-level STP
show spanning-tree interface e0/1
! Root Bridge info only
show spanning-tree root
! Ports with STP inconsistencies
show spanning-tree inconsistentports
### Routing Verification
show ip interface brief
show ip route
show ip route static
show ip route 10.0.0.0
show ip protocols
show arp
### BPDU Debug
debug spanning-tree events
! Wait 10 seconds
undebug all
### End-to-End
! From VPC7
show ip
ping 192.168.10.1
ping 192.168.20.10
trace 192.168.20.10
