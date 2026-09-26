# Cisco EtherChannel & VLAN Lab
A comprehensive hands-on lab demonstrating **EtherChannel bundling**, **802.1Q VLAN trunking**, **Native VLAN security**, and **Router-on-a-Stick inter-VLAN routing** — built and verified in **EVE-NG** using real Cisco IOS images.

## 🎯 Lab Objectives
- Configure and verify **LACP EtherChannel** (IEEE 802.3ad) between two switches
- Configure and verify **PAgP EtherChannel** (Cisco proprietary) between two switches
- Configure and verify **Static ON EtherChannel** (manual bundling)
- Configure **802.1Q trunking** across all EtherChannel bundles
- Implement **Native VLAN 99** for security best practice
- Deploy **Router-on-a-Stick** for inter-VLAN routing
- Verify **Spanning Tree Protocol (STP)** convergence over EtherChannels
- Enable **EtherChannel Misconfiguration Guard** and **`vlan dot1q tag native`**
- Practice **EtherChannel troubleshooting** and failure scenarios
- Test end-to-end connectivity across same-VLAN and inter-VLAN paths

## 🖧 Topology Overview
| Device | Role | Model | Key Function |
|--------|------|-------|--------------|
| SW1 | Core Switch | Cisco IOU L2 | LACP + Static ON EtherChannels |
| SW2 | Core Switch | Cisco IOU L2 | LACP + Static ON EtherChannels |
| SW3 | Access Switch | Cisco IOU L2 | PAgP + Static ON EtherChannels |
| SW4 | Access Switch | Cisco IOU L2 | PAgP + Static ON EtherChannels |
| R1 | Edge Router | Cisco 3725 (Dynamips) | Router-on-a-Stick |
| VPC1–VPC4 | Test Hosts | VPCS | End-to-end connectivity |

## 🔗 EtherChannel Summary
| Port-Channel | Protocol | Members | Endpoints | Status |
|--------------|----------|---------|-----------|--------|
| Po1 | LACP (802.3ad) | 4 links | SW1 ↔ SW2 | ✅ Up |
| Po2 | PAgP (Cisco) | 2 links | SW3 ↔ SW4 | ✅ Up |
| Po3 | Static ON | 1 link | SW1 ↔ SW3 | ✅ Up |
| Po3 | Static ON | 1 link | SW2 ↔ SW4 | ✅ Up |

## 🌐 VLAN Design

| VLAN | Name | Subnet | Purpose |
|------|------|--------|---------|
| 10 | Sales | 192.168.10.0/24 | Sales department |
| 20 | Engineering | 192.168.20.0/24 | Engineering department |
| 30 | Management | 192.168.30.0/24 | Management department |
| 99 | Native | — | Native VLAN (untagged baseline) |

## 🖥️ IP Addressing

| Device | Interface | IP Address | Gateway | VLAN |
|--------|-----------|------------|---------|------|
| VPC1 | eth0 | 192.168.10.10/24 | 192.168.10.1 | 10 |
| VPC2 | eth0 | 192.168.20.10/24 | 192.168.20.1 | 20 |
| VPC3 | eth0 | 192.168.30.10/24 | 192.168.30.1 | 30 |
| VPC4 | eth0 | 192.168.10.11/24 | 192.168.10.1 | 10 |
| R1 | Fa0/0.10 | 192.168.10.1/24 | — | 10 |
| R1 | Fa0/0.20 | 192.168.20.1/24 | — | 20 |
| R1 | Fa0/0.30 | 192.168.30.1/24 | — | 30 |

## 📋 Configuration Files
### SW1 — Core Switch
hostname SW1
! --- VLANs ---
vlan 10
name Sales
vlan 20
name Engineering
vlan 30
name Management
vlan 99
name Native
exit
! --- Access port: VPC1 (VLAN 10) ---
interface e1/0
switchport mode access
switchport access vlan 10
no shutdown
exit
! --- LACP EtherChannel to SW2 (Port-channel 1) ---
interface range e0/0-3
switchport
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,30,99
channel-protocol lacp
channel-group 1 mode active
no shutdown
exit
interface port-channel 1
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,30,99
no shutdown
exit
! --- Static ON EtherChannel to SW3 (Port-channel 3) ---
interface e1/1
switchport
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,30,99
channel-group 3 mode on
no shutdown
exit
interface port-channel 3
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,30,99
no shutdown
exit
! --- Trunk to R1 (Router-on-a-Stick) ---
interface e2/0
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,30,99
no shutdown
exit
! --- Security & Best Practices ---
spanning-tree etherchannel guard misconfig
vlan dot1q tag native
! --- Load Balancing ---
port-channel load-balance src-dst-ip
end
write memory

### SW2 — Core Switch
hostname SW2
vlan 10
name Sales
vlan 20
name Engineering
vlan 30
name Management
vlan 99
name Native
exit
interface e1/0
switchport mode access
switchport access vlan 20
no shutdown
exit
! --- LACP EtherChannel to SW1 (Port-channel 1) ---
interface range e0/0-3
switchport
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,30,99
channel-protocol lacp
channel-group 1 mode passive
no shutdown
exit
interface port-channel 1
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,30,99
no shutdown
exit
! --- Static ON EtherChannel to SW4 (Port-channel 3) ---
interface e1/1
switchport
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,30,99
channel-group 3 mode on
no shutdown
exit
interface port-channel 3
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,30,99
no shutdown
exit
spanning-tree etherchannel guard misconfig
vlan dot1q tag native
port-channel load-balance src-dst-ip
end
write memory

### SW3 — Access Switch
hostname SW3
vlan 10
name Sales
vlan 20
name Engineering
vlan 30
name Management
vlan 99
name Native
exit
interface e1/0
switchport mode access
switchport access vlan 30
no shutdown
exit
! --- PAgP EtherChannel to SW4 (Port-channel 2) ---
interface range e0/0-1
switchport
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,30,99
channel-protocol pagp
channel-group 2 mode desirable
no shutdown
exit
interface port-channel 2
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,30,99
no shutdown
exit
! --- Static ON EtherChannel to SW1 (Port-channel 3) ---
interface e1/1
switchport
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,30,99
channel-group 3 mode on
no shutdown
exit
interface port-channel 3
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,30,99
no shutdown
exit
spanning-tree etherchannel guard misconfig
vlan dot1q tag native
end
write memory

### SW4 — Access Switch
hostname SW4
vlan 10
name Sales
vlan 20
name Engineering
vlan 30
name Management
vlan 99
name Native
exit
interface e1/0
switchport mode access
switchport access vlan 10
no shutdown
exit
! --- PAgP EtherChannel to SW3 (Port-channel 2) ---
interface range e0/0-1
switchport
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,30,99
channel-protocol pagp
channel-group 2 mode auto
no shutdown
exit
interface port-channel 2
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,30,99
no shutdown
exit
! --- Static ON EtherChannel to SW2 (Port-channel 3) ---
interface e1/1
switchport
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,30,99
channel-group 3 mode on
no shutdown
exit
interface port-channel 3
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,30,99
no shutdown
exit
spanning-tree etherchannel guard misconfig
vlan dot1q tag native
end
write memory

### R1 — Edge Router (Router-on-a-Stick)
hostname R1
! --- Physical interface to SW1 ---
interface fastEthernet 0/0
duplex full
speed 100
no shutdown
exit
! --- Sub-interface for VLAN 10 ---
interface fastEthernet 0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0
no shutdown
exit
! --- Sub-interface for VLAN 20 ---
interface fastEthernet 0/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0
no shutdown
exit
! --- Sub-interface for VLAN 30 ---
interface fastEthernet 0/0.30
encapsulation dot1Q 30
ip address 192.168.30.1 255.255.255.0
no shutdown
exit
end
write memory

### VPCs
! On VPC1
ip 192.168.10.10/24 192.168.10.1
save
! On VPC2
ip 192.168.20.10/24 192.168.20.1
save
! On VPC3
ip 192.168.30.10/24 192.168.30.1
save
! On VPC4
ip 192.168.10.11/24 192.168.10.1
save

---

## 🛠️ Technologies & Concepts
### EtherChannel
#### LACP (Link Aggregation Control Protocol)
- **Standard:** IEEE 802.3ad
- **Modes:** Active (initiates), Passive (responds)
- **Multicast MAC:** 0180:C200:0002
- **Max ports:** 16 (8 active at a time)
- **Requirement:** At least one side must be `active`

#### PAgP (Port Aggregation Protocol)
- **Standard:** Cisco proprietary
- **Modes:** Desirable (initiates), Auto (responds)
- **Multicast MAC:** 0100:0CCC:CCCC
- **Max ports:** 8
- **Requirement:** At least one side must be `desirable`

#### Static ON (Manual)
- **Protocol:** None (bypasses negotiation)
- **Use case:** When remote device doesn't support LACP/PAgP
- **Requirement:** Both sides must be `on`

#### EtherChannel Requirements
All member interfaces must match:
- Speed
- Duplex
- Native VLAN
- Allowed VLANs
- Switchport mode (trunk/access)
- MTU (Layer 3 port-channels)
- Storm control settings

#### Load Balancing
- **Method:** Hash-based on header fields
- **Default L2:** Source + Destination MAC
- **Default L3:** Source + Destination IP
- **Note:** Per-flow, not per-packet
- **Commands:**
  - `port-channel load-balance src-mac`
  - `port-channel load-balance dst-mac`
  - `port-channel load-balance src-dst-ip`
  - `show etherchannel load-balance`

### VLAN & Trunking
#### 802.1Q
- **Standard:** IEEE 802.1Q
- **Tag size:** 4 bytes inserted into Ethernet frame
- **Tag fields:** Type, Priority, Flag, VLAN ID (12 bits)
- **Supports:** Normal (1–1005) and Extended (1006–4094) VLANs
#### Native VLAN
- **Function:** Untagged traffic on a trunk belongs to the Native VLAN
- **Default:** VLAN 1
- **Best practice:** Change to a dedicated VLAN (99 in this lab)
- **Security:** Prevents VLAN hopping attacks
- **Requirement:** Must match on both ends of a trunk

#### `vlan dot1q tag native`
- Forces tagging of Native VLAN traffic too
- Prevents VLAN hopping / double-tagging attacks
- Must be enabled on **all** switches in the path
### Spanning Tree
- STP treats an EtherChannel as a **single logical link**
- **Cost:** Based on total bandwidth of the port-channel
- **One root port per VLAN** on each non-root switch
- **Blocking:** Only one path is active; others are standby
- **Result:** Loop-free topology with redundancy
### Router-on-a-Stick
- **Concept:** Single physical router interface carries multiple VLANs
- **Mechanism:** Sub-interfaces with 802.1Q encapsulation
- **Config:** `interface Fa0/0.10` → `encapsulation dot1Q 10` → `ip address`
- **Gateway:** Each sub-interface acts as the default gateway for its VLAN
### Security & Best Practices

#### EtherChannel Misconfiguration Guard
spanning-tree etherchannel guard misconfig
#### verification commands
! EtherChannel state
show etherchannel summary
show etherchannel port-channel
! Protocol neighbors
show lacp neighbor      ! LACP switches (SW1, SW2)
show pagp neighbor      ! PAgP switches (SW3, SW4)
! Trunk status
show interfaces trunk
! VLAN database
show vlan brief
! Spanning Tree per VLAN
show spanning-tree vlan 10
show spanning-tree vlan 20
show spanning-tree vlan 30
! Topology discovery
show cdp neighbors
! MAC learning
show mac address-table
! Security features
show spanning-tree summary
show vlan dot1q tag native
! Load balancing
show etherchannel load-balance
