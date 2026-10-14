# Cisco Network & Core Services Lab
A comprehensive hands-on lab demonstrating **OSPF routing**, **HSRP gateway redundancy**, **Static NAT**, **PAT (overload)**, **DHCP Server**, **DHCP Relay**, **NTP Master/Client**, **NTP MD5 authentication**, **DNS**, **Syslog**, and **SNMP v2c/v3** — built and verified in **EVE-NG** with real Cisco IOS images.

## 🎯 Lab Objectives
- Configure **OSPF Area 0** across 5 routers
- Configure **HSRP** for gateway redundancy between R1 and R2
- Configure **Static NAT** on R1 for external access
- Configure **PAT (overload)** for LAN-to-Internet translation
- Configure **Cisco router as DHCP server** on R4
- Configure **DHCP Relay** on R1 for remote LAN clients
- Configure **NTP Master/Client** with MD5 authentication
- Configure **DNS server** on R1
- Configure **Syslog** to remote server R5
- Configure **SNMP v2c and v3** on R1
- Verify end-to-end connectivity and NAT/PAT translations
- Demonstrate **HSRP failover** and **NAT ACL selection**
- Troubleshoot **real-world issues** encountered during the build

## 🖧 Topology Overview
┌──────────────┐
│ Loopback │
│ 8.8.8.8/32 │ (Simulated Internet on R3)
└──────┬───────┘
│
┌──────┴───────┐
│ R3 │
│ Core/ISP │
└┬────┬─────┬──┘
│ │ │
Fa0/0 │ │ │ Fa0/1
┌──────┘ │ └─────┐
│ │ │
┌────┴────┐ ┌───┴────┐ ┌───┴────┐
│ R1 │ │ R4 │ │ R5 │
│ (HSRP │ │ (DHCP) │ │(NTP/S) │
│ NAT) │ └────────┘ └────────┘
└────┬────┘
│
│ Fa0/0
┌────┴────┐
│ R2 │
│ (HSRP │
│ Standby)│
└────┬────┘
│
┌────┴────┐
│ SW │
│(Switch) │
└────┬────┘
│
┌────┴────┐
│ VPC │
│ (DHCP) │
└─────────┘

## 📋 Device Table
| Device | Role | Model | Key Function |
|--------|------|-------|--------------|
| R1 | Edge Router | Cisco 3725 | HSRP Active + NAT + DHCP Relay + DNS + SNMP + Syslog |
| R2 | Edge Router | Cisco 3725 | HSRP Standby |
| R3 | Core Router | Cisco 3725 | OSPF backbone + simulated Internet |
| R4 | DHCP Server | Cisco 3725 | DHCP Server |
| R5 | Services | Cisco 3725 | NTP Master + Syslog receiver |
| SW | Access Switch | Cisco IOL L2 | Layer 2 switching |
| VPC | Client | VPCS | DHCP client |

## 🔗 Cabling Plan
| From | Interface | To | Interface | Purpose |
|------|-----------|-----|-----------|---------|
| R1 | Fa0/0 | R3 | Fa0/0 | Core uplink |
| R2 | Fa0/1 | R3 | Fa0/1 | Core uplink |
| R3 | Fa1/0 | R4 | Fa1/0 | To DHCP server |
| R3 | Fa2/0 | R5 | Fa0/0 | To services |
| R1 | Fa0/1 | SW | e0/0 | LAN (HSRP Active) |
| R2 | Fa0/0 | SW | e0/1 | LAN (HSRP Standby) |
| SW | e0/2 | VPC | eth0 | Client |

## 🖥️ IP Addressing Plan
| Router | Interface | To | Subnet | IP Address |
|--------|-----------|-----|--------|------------|
| R1 | Fa0/0 | R3 | 10.0.0.0/24 | 10.0.0.1 |
| R1 | Fa0/1 | SW | 192.168.1.0/24 | 192.168.1.1 |
| R1 | Lo1 | — | — | 1.1.1.1/32 |
| R2 | Fa0/1 | R3 | 10.0.1.0/24 | 10.0.1.2 |
| R2 | Fa0/0 | SW | 192.168.1.0/24 | 192.168.1.2 |
| R2 | Lo2 | — | — | 2.2.2.2/32 |
| R3 | Fa0/0 | R1 | 10.0.0.0/24 | 10.0.0.3 |
| R3 | Fa0/1 | R2 | 10.0.1.0/24 | 10.0.1.3 |
| R3 | Fa1/0 | R4 | 10.10.10.0/24 | 10.10.10.1 |
| R3 | Fa2/0 | R5 | 10.20.20.0/24 | 10.20.20.1 |
| R3 | Lo3 | — | — | 3.3.3.3/32 |
| R3 | Lo6 | — | — | 8.8.8.8/32 |
| R4 | Fa1/0 | R3 | 10.10.10.0/24 | 10.10.10.2 |
| R4 | Lo4 | — | — | 4.4.4.4/32 |
| R5 | Fa0/0 | R3 | 10.20.20.0/24 | 10.20.20.2 |
| R5 | Lo5 | — | — | 5.5.5.5/32 |
| HSRP | Virtual | LAN | 192.168.1.0/24 | 192.168.1.254 |
| VPC | eth0 | SW | DHCP | 192.168.1.11+ |

## 📋 Configuration Files
### R1 Configuration
en
conf ter
hostname R1
! Interfaces
interface fastEthernet 0/0
ip address 10.0.0.1 255.255.255.0
no shutdown
exit
interface fastEthernet 0/1
ip address 192.168.1.1 255.255.255.0
ip helper-address 10.10.10.2
ip nat inside
speed 100
duplex full
standby 1 ip 192.168.1.254
standby 1 priority 150
standby 1 preempt
standby 1 authentication md5 key-string cisco
no shutdown
exit
interface loopback 1
ip address 1.1.1.1 255.255.255.255
exit
! NAT
access-list 10 deny 10.0.0.0 0.0.255.255
access-list 10 deny 10.10.0.0 0.0.255.255
access-list 10 deny 10.20.0.0 0.0.255.255
access-list 10 deny 192.168.0.0 0.0.255.255
access-list 10 permit any
ip nat inside source list 10 interface fastEthernet 0/0 overload
ip nat inside source static 4.4.4.4 10.0.0.100
! OSPF
router ospf 1
router-id 1.1.1.1
passive-interface default
no passive-interface FastEthernet0/0
no passive-interface FastEthernet0/1
network 10.0.0.0 0.0.0.255 area 0
network 192.168.1.0 0.0.0.255 area 0
network 1.1.1.1 0.0.0.0 area 0
exit
! NTP
ntp authentication-key 1 md5 NTPpass123
ntp authenticate
ntp trusted-key 1
ntp server 10.20.20.2 key 1
! Syslog
service timestamps log datetime msec
service timestamps debug datetime msec
logging host 10.20.20.2
logging trap informational
logging buffered 16384 informational
! SNMP
snmp-server community public RO
snmp-server community private RW
snmp-server contact admin@lab.local
snmp-server location DataCenter-Rack-1
snmp-server group ADMIN v3 priv
snmp-server user snmpuser ADMIN v3 auth md5 AuthPass123 priv aes 128 PrivPass123
! DNS
ip dns server
ip host r1.lab.local 192.168.1.1
ip host r3.lab.local 10.0.0.3
ip host r4.lab.local 10.10.10.2
ip host r5.lab.local 10.20.20.2
ip host isp.lab.local 8.8.8.8
ip name-server 8.8.8.8
ip domain-lookup
end
write memory

### R2 Configuration
en
conf ter
hostname R2
interface fastEthernet 0/1
ip address 10.0.1.2 255.255.255.0
ip ospf cost 100
no shutdown
exit
interface fastEthernet 0/0
ip address 192.168.1.2 255.255.255.0
speed 100
duplex full
standby 1 ip 192.168.1.254
standby 1 priority 100
standby 1 preempt
standby 1 authentication md5 key-string cisco
no shutdown
exit
interface loopback 2
ip address 2.2.2.2 255.255.255.255
exit
router ospf 1
router-id 2.2.2.2
network 10.0.1.0 0.0.0.255 area 0
network 192.168.1.0 0.0.0.255 area 0
network 2.2.2.2 0.0.0.0 area 0
exit
end
write memory

### R3 Configuration
en
conf ter
hostname R3
interface fastEthernet 0/0
ip address 10.0.0.3 255.255.255.0
no shutdown
exit
interface fastEthernet 0/1
ip address 10.0.1.3 255.255.255.0
no shutdown
exit
interface fastEthernet 1/0
ip address 10.10.10.1 255.255.255.0
no shutdown
exit
interface fastEthernet 2/0
ip address 10.20.20.1 255.255.255.0
no shutdown
exit
interface loopback 3
ip address 3.3.3.3 255.255.255.255
exit
interface loopback 6
ip address 8.8.8.8 255.255.255.255
exit
router ospf 1
router-id 3.3.3.3
maximum-paths 1
network 10.0.0.0 0.0.0.255 area 0
network 10.0.1.0 0.0.0.255 area 0
network 10.10.10.0 0.0.0.255 area 0
network 10.20.20.0 0.0.0.255 area 0
network 3.3.3.3 0.0.0.0 area 0
network 8.8.8.8 0.0.0.0 area 0
exit
end
write memory

### R4 Configuration
en
conf ter
hostname R4
interface fastEthernet 1/0
ip address 10.10.10.2 255.255.255.0
no shutdown
exit
interface loopback 4
ip address 4.4.4.4 255.255.255.255
exit
! DHCP Server
ip dhcp excluded-address 192.168.1.1 192.168.1.10
ip dhcp excluded-address 192.168.1.250 192.168.1.254
ip dhcp pool LAN_POOL
network 192.168.1.0 255.255.255.0
default-router 192.168.1.254
dns-server 8.8.8.8
domain-name lab.local
lease 0 8 0
exit
router ospf 1
router-id 4.4.4.4
network 10.10.10.0 0.0.0.255 area 0
network 4.4.4.4 0.0.0.0 area 0
exit
end
write memory

### R5 Configuration
en
conf ter
hostname R5
interface fastEthernet 0/0
ip address 10.20.20.2 255.255.255.0
no shutdown
exit
interface loopback 5
ip address 5.5.5.5 255.255.255.255
exit
router ospf 1
router-id 5.5.5.5
network 10.20.20.0 0.0.0.255 area 0
network 5.5.5.5 0.0.0.0 area 0
exit
! NTP Master
ntp master 3
ntp authentication-key 1 md5 NTPpass123
ntp authenticate
ntp trusted-key 1
end
write memory

### SW Configuration
en
conf ter
hostname SW
interface e0/0
switchport mode access
switchport access vlan 1
no shutdown
exit
interface e0/1
switchport mode access
switchport access vlan 1
no shutdown
exit
interface e0/2
switchport mode access
switchport access vlan 1
no shutdown
exit
end
write memory

### VPC Configuration
ip 192.168.1.11/24 192.168.1.1
save
Or via DHCP:
ip dhcp
save

## 🛠️ Technologies & Concepts
### OSPF (Open Shortest Path First)
| Feature | Detail |
|---------|--------|
| Type | Link-State dynamic routing protocol |
| Standard | Open (RFC 2328) |
| Algorithm | Shortest Path First (Dijkstra) |
| IP Protocol | 89 (not TCP/UDP) |
| Multicast Addresses | 224.0.0.5, 224.0.0.6 |
| Hello Timer | 10 seconds |
| Dead Timer | 40 seconds |
| AD | 110 |
| Metric | Cost (based on bandwidth) |
| Areas | Area 0 = backbone |
| Tables | Neighbor, Topology (LSDB), Routing |

### HSRP (Hot Standby Router Protocol)
| Feature | Detail |
|---------|--------|
| Type | Cisco proprietary FHRP |
| States | Initial, Listen, Speak, Standby, Active |
| Default Priority | 100 |
| Preempt | Disabled by default |
| Virtual MAC (v1) | 0000.0c07.acXX |
| Hello Timer | 3 seconds |
| Hold Timer | 10 seconds |
| Multicast | 224.0.0.2 |

### NAT (Network Address Translation)
| Type | Description |
|------|-------------|
| Static NAT | One-to-one mapping |
| Dynamic NAT | Pool-based mapping |
| PAT (Overload) | Many-to-one with ports |
| Inside Local | Private IP of internal host |
| Inside Global | Public IP representing internal host |
| Outside Local | IP of external host as seen internally |
| Outside Global | Public IP of external host |

### DHCP (Dynamic Host Configuration Protocol)
| Feature | Detail |
|---------|--------|
| DORA Process | Discover, Offer, Request, Acknowledgment |
| Server Port | UDP 67 |
| Client Port | UDP 68 |
| Relay Command | `ip helper-address X.X.X.X` |
| Verification | `show ip dhcp binding`, `show ip dhcp pool` |

### NTP (Network Time Protocol)
| Feature | Detail |
|---------|--------|
| Port | UDP 123 |
| Stratum Levels | 0 (atomic) to 15 (farthest) |
| Stratum 16 | Unsynchronized |
| Default Router Stratum | 8 |
| Auth | MD5, SHA1 |
| Modes | Server, Client, Server/Client, Peer, Master |

### DNS (Domain Name System)
| Feature | Detail |
|---------|--------|
| Port | UDP 53 |
| Records | A, AAAA, CNAME, MX, PTR, NS, SOA |
| Root Servers | 13 (a.root-servers.net to m.root-servers.net) |
| Command | `ip dns server`, `ip host`, `ip name-server` |

### Syslog
| Severity | Level Name | Description |
|----------|-----------|-------------|
| 0 | Emergency | System unusable |
| 1 | Alert | Immediate action needed |
| 2 | Critical | Critical condition |
| 3 | Error | Error message |
| 4 | Warning | Warning message |
| 5 | Notice | Normal but significant |
| 6 | Informational | Information message |
| 7 | Debugging | Debug messages |

**Logging Destinations:** Console, Buffer, Terminal, Syslog Server
### SNMP (Simple Network Management Protocol)
| Feature | Detail |
|---------|--------|
| Port | UDP 161 (agent), 162 (traps) |
| Manager | Network Management System |
| Agent | Software on managed device |
| MIB | Management Information Base |
| v1/v2c | Community strings (clear text) |
| v3 | Authentication + Encryption |
| Security Levels | noAuthNoPriv, AuthNoPriv, AuthPriv |

## ✅ Verification Commands
### OSPF Verification
```cisco
show ip ospf neighbor
show ip route ospf
show ip ospf interface brief
show ip protocols
show ip ospf database
### HSRP Verification
cisco
show standby brief
show standby
### NAT Verification
cisco
show ip nat translations
show ip nat statistics
show access-lists
clear ip nat translation *
### DHCP Verification
cisco
show ip dhcp binding
show ip dhcp pool
show ip dhcp conflict
show ip interface fa0/1 | include helper
### NTP Verification
cisco
show ntp status
show ntp associations
show clock
show clock detail
### DNS Verification
cisco
show hosts
ping r4.lab.local
### Syslog Verification
cisco
show logging
### SNMP Verification
cisco
show snmp
show snmp community
show snmp user
show snmp group
### End-to-End Testing
bash
! From VPC
show ip
arp
ping 192.168.1.1
ping 192.168.1.2
ping 10.0.0.3
ping 3.3.3.3
ping 8.8.8.8
ping 4.4.4.4
ping 5.5.5.5
trace 8.8.8.8
## 🎓 Key Learnings
OSPF requires matching subnets on both ends of a link.
HSRP uses virtual MAC (0000.0c07.acXX) — VPCs must ARP for the VIP.
Static NAT is bidirectional — allows inbound access to internal hosts.
PAT uses overload — many-to-one with port translation.
DHCP Relay uses ip helper-address — required when server is on different subnet.
NTP authentication adds MD5 keys — must match on both sides.
DNS server can be configured on Cisco router with ip dns server.
Syslog severity levels range from 0 (Emergency) to 7 (Debugging).
SNMPv3 is the secure version — supports authentication and encryption.
NAT ACL must exclude RFC1918 destinations to prevent breaking internal routing.
ECMP breaks asymmetric NAT — use single path or align traffic flow.
NVI0 interface appears with NAT — make it passive in OSPF.
Duplex mismatch is a common cause of one-way ping failures.
VPC phantom cables in EVE-NG — always verify L2 connectivity first.
Debug IP ICMP is the fastest way to find where ICMP packets break.

## 📌 Lab Summary
This lab demonstrates a production-grade network with routing, redundancy, translation, and management services. Key accomplishments:
### Routing:
OSPF Area 0 across 5 routers
Full mesh of neighbors
Metric and cost manipulation
ECMP handling

### Redundancy:
HSRP Active/Standby between R1 and R2
Preempt configured
Authentication MD5

### Translation:
Static NAT for R4 loopback
PAT overload for LAN to Internet
ACL selection for internal vs external

### Services:
DHCP server on R4
DHCP relay on R1
NTP master on R5
NTP authentication MD5
DNS server on R1
Syslog to R5
SNMP v2c + v3

### Verification:
End-to-end connectivity
NAT translations live
HSRP failover
DHCP lease
NTP synchronization
