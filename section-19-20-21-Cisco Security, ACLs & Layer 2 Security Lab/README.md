# Cisco Security, ACLs & Layer 2 Security Lab
A comprehensive hands-on lab demonstrating **device security**, **Access Control Lists (ACLs)**, **Layer 2 security**, **VPN concepts**, **wireless security protocols**, **AAA**, **QoS**, **DHCP Snooping**, **Dynamic ARP Inspection (DAI)**, **Port Security**, and **IP Source Guard** — built and verified in **EVE-NG** with real Cisco IOS images.

## 🎯 Lab Objectives
- Configure device access control using **local passwords** and **SSH**
- Define key security concepts: threats, vulnerabilities, and mitigation
- Configure password policies and understand **multifactor authentication**
- Describe the capabilities and function of **TFTP** and **FTP** in the network
- Introduction, theory, and concept of **Quality of Service (QoS)**
- Configure and verify **Standard**, **Extended**, **Named**, and **Time-Based ACLs**
- Describe the concept of **Remote Access** and **Site-to-Site VPNs**
- Configure and verify **Layer 2 Security Features**:
  - **Port Security**
  - **DHCP Snooping**
  - **Dynamic ARP Inspection (DAI)**
  - **Infrastructure ACL / IP Source Guard**
- Differentiate **Authentication**, **Authorization**, and **Accounting (AAA)**
- Describe **Wireless Security Protocols** (WPA, WPA2, WPA3)

## 🖧 Topology Overview
┌─────────────┐
│ R4 (TFTP/ │
│ FTP) │
└──────┬──────┘
│ 10.30.30.0/24
│
R2 (WAN Peer) │
│ │
│ 200.1.1.0/24 │
│ │
[R1]───────────────────────[R3]
│ Fa0/1 │ Fa0/1
│ │
[SW] │
│ │
┌───┴────┬────────┬─────────┐ │
│ │ │ │ │
VPC6 VPC7 VPC8 VPC9 │
(Client) (Attacker) (Server) (DHCP) │

### Device Table
| Device | Role | Model | Purpose |
|--------|------|-------|---------|
| **R1** | Edge Router | Cisco 3725 | SSH, AAA, QoS, ACLs, DHCP Relay, NAT |
| **R2** | WAN Peer | Cisco 3725 | Site-to-Site VPN endpoint |
| **R3** | Core Router | Cisco 3725 | Routing, ACL application point |
| **R4** | Server | Cisco 3725 | TFTP, FTP server |
| **SW** | Access Switch | Cisco IOL L2 | DHCP Snooping, DAI, Port Security, IP Source Guard |
| **VPC6** | Client | VPCS | Normal traffic (DHCP) |
| **VPC7** | Attacker | VPCS | Spoofing, ARP poisoning (Static 192.168.1.66) |
| **VPC8** | Server | VPCS | Legitimate server (Static 192.168.1.30) |
| **VPC9** | DHCP Client | VPCS | DHCP client |

## 🔗 Cabling Plan
| From | Interface | To | Interface | Purpose |
|------|-----------|-----|-----------|---------|
| R1 | Fa1/0 | R2 | Fa1/0 | WAN / VPN |
| R1 | Fa0/0 | R3 | Fa0/0 | Core Link |
| R1 | Fa0/1 | SW | e0/1 | LAN Gateway |
| R3 | Fa0/1 | R4 | Fa0/0 | Server Link |
| SW | e0/0 | VPC6 | eth0 | Client |
| SW | e0/2 | VPC7 | eth0 | Attacker |
| SW | e0/3 | VPC8 | eth0 | Server |
| SW | e1/0 | VPC9 | eth0 | DHCP Client |

## 🖥️ IP Addressing Plan
| Device | Interface | Subnet | IP Address | Purpose |
|--------|-----------|--------|------------|---------|
| **R1** | Fa1/0 | 200.1.1.0/24 | 200.1.1.1 | WAN |
| **R1** | Fa0/0 | 10.0.0.0/24 | 10.0.0.1 | Core |
| **R1** | Fa0/1 | 192.168.1.0/24 | 192.168.1.1 | LAN |
| **R1** | Lo1 | — | 1.1.1.1/32 | RID |
| **R2** | Fa1/0 | 200.1.1.0/24 | 200.1.1.2 | WAN |
| **R2** | Lo2 | — | 2.2.2.2/32 | RID |
| **R3** | Fa0/0 | 10.0.0.0/24 | 10.0.0.3 | Core |
| **R3** | Fa0/1 | 10.30.30.0/24 | 10.30.30.1 | Server Link |
| **R3** | Lo3 | — | 3.3.3.3/32 | RID |
| **R4** | Fa0/0 | 10.30.30.0/24 | 10.30.30.2 | Server |
| **R4** | Lo4 | — | 4.4.4.4/32 | RID |
| **SW** | VLAN 1 | 192.168.1.0/24 | 192.168.1.2 | Mgmt |
| **VPC6** | eth0 | 192.168.1.0/24 | DHCP (192.168.1.10) | Client |
| **VPC7** | eth0 | 192.168.1.0/24 | 192.168.1.66 | Attacker |
| **VPC8** | eth0 | 192.168.1.0/24 | 192.168.1.30 | Server |
| **VPC9** | eth0 | 192.168.1.0/24 | DHCP (192.168.1.11) | Client |

## 📋 Configuration Files
### R1 Configuration
enable
configure terminal
hostname R1
! Interfaces
interface fastEthernet 1/0
ip address 200.1.1.1 255.255.255.0
speed 100
duplex full
no shutdown
exit
interface fastEthernet 0/0
ip address 10.0.0.1 255.255.255.0
speed 100
duplex full
no shutdown
exit
interface fastEthernet 0/1
ip address 192.168.1.1 255.255.255.0
ip helper-address 10.30.30.2
speed 100
duplex full
no shutdown
exit
interface loopback 1
ip address 1.1.1.1 255.255.255.255
exit
! OSPF
router ospf 1
router-id 1.1.1.1
network 10.0.0.0 0.0.0.255 area 0
network 192.168.1.0 0.0.0.255 area 0
network 200.1.1.0 0.0.0.255 area 0
network 1.1.1.1 0.0.0.0 area 0
exit
! DHCP Server
ip dhcp excluded-address 192.168.1.1 192.168.1.9
ip dhcp excluded-address 192.168.1.250 192.168.1.254
ip dhcp pool LAN_POOL
network 192.168.1.0 255.255.255.0
default-router 192.168.1.1
dns-server 8.8.8.8
lease 0 8 0
exit
! SSH & Local Users
ip domain-name lab.local
crypto key generate rsa modulus 1024
ip ssh version 2
username admin privilege 15 secret AdminPass123!
enable secret Cisco123!
service password-encryption
line console 0
password Console123!
login
exit
line vty 0 4
transport input ssh
login local
exit
! AAA
aaa new-model
aaa authentication login default local
aaa authorization exec default local
security passwords min-length 8
login block-for 120 attempts 3 within 60

! QoS
class-map match-any VOICE
match dscp ef
exit
policy-map WAN-POLICY
class VOICE
priority percent 30
class class-default
fair-queue
exit
exit
interface fastEthernet 1/0
service-policy output WAN-POLICY
exit
! Extended ACL
access-list 100 permit tcp any host 10.30.30.2 eq 80
access-list 100 permit tcp any host 10.30.30.2 eq 443
access-list 100 deny tcp any host 10.30.30.2 eq 23
access-list 100 permit ip any any
interface fastEthernet 0/1
ip access-group 100 in
exit
end
write memory

### R2 Configuration
enable
configure terminal
hostname R2
interface fastEthernet 1/0
ip address 200.1.1.2 255.255.255.0
speed 100
duplex full
no shutdown
exit
interface loopback 2
ip address 2.2.2.2 255.255.255.255
exit
router ospf 1
router-id 2.2.2.2
network 200.1.1.0 0.0.0.255 area 0
network 2.2.2.2 0.0.0.0 area 0
exit
end
write memory

### R3 Configuration
enable
configure terminal
hostname R3
interface fastEthernet 0/0
ip address 10.0.0.3 255.255.255.0
speed 100
duplex full
no shutdown
exit
interface fastEthernet 0/1
ip address 10.30.30.1 255.255.255.0
speed 100
duplex full
no shutdown
exit
interface loopback 3
ip address 3.3.3.3 255.255.255.255
exit
router ospf 1
router-id 3.3.3.3
network 10.0.0.0 0.0.0.255 area 0
network 10.30.30.0 0.0.0.255 area 0
network 3.3.3.3 0.0.0.0 area 0
exit
! Named ACL
ip access-list extended BLOCK_ATTACKER
deny ip host 192.168.1.66 any
permit ip any any
exit
! Time-Based ACL
time-range WORKING_HOURS
periodic weekdays 8:00 to 18:00
exit
ip access-list extended TIME_BASED
deny ip host 192.168.1.66 any time-range WORKING_HOURS
permit ip any any
exit
end
write memory

### R4 Configuration
enable
configure terminal
hostname R4
interface fastEthernet 0/0
ip address 10.30.30.2 255.255.255.0
speed 100
duplex full
no shutdown
exit
interface loopback 4
ip address 4.4.4.4 255.255.255.255
exit
router ospf 1
router-id 4.4.4.4
network 10.30.30.0 0.0.0.255 area 0
network 4.4.4.4 0.0.0.0 area 0
exit
! TFTP Server
tftp-server nvram:startup-config
! FTP Server
ip ftp username cisco
ip ftp password CiscoFTP123
username ftpuser privilege 15 secret FtpPass123!
end
write memory

### SW Configuration
enable
configure terminal
hostname SW
interface vlan 1
ip address 192.168.1.2 255.255.255.0
no shutdown
exit
ip default-gateway 192.168.1.1
interface range e0/0-3, e1/0
switchport mode access
switchport access vlan 1
no shutdown
exit
! DHCP Snooping
ip dhcp snooping
ip dhcp snooping vlan 1
no ip dhcp snooping information option

interface e0/1
ip dhcp snooping trust
exit
interface range e0/0, e0/2, e0/3, e1/0
ip dhcp snooping limit rate 10
exit
! Dynamic ARP Inspection
ip arp inspection vlan 1
interface e0/1
ip arp inspection trust
exit
interface range e0/0, e0/2, e0/3, e1/0
ip arp inspection limit rate 15 burst interval 1
exit
! Port Security
interface range e0/0, e0/2, e0/3, e1/0
switchport port-security
switchport port-security maximum 2
switchport port-security violation shutdown
switchport port-security mac-address sticky
exit
! IP Source Guard
interface range e0/0, e0/2, e0/3, e1/0
ip verify source
exit
end
write memory
### VPC Configurations
**VPC6 (Client):**
ip dhcp
save
**VPC7 (Attacker):**
ip 192.168.1.66/24 192.168.1.1
save
**VPC8 (Server):**
ip 192.168.1.30/24 192.168.1.1
save
**VPC9 (DHCP Client):**
ip dhcp
save

## 🛠️ Technologies & Concepts
### Security Basics
| Feature | Description |
|---------|-------------|
| **Local Passwords** | `enable secret`, `username`, `line console`, `line vty` |
| **SSH** | `crypto key generate rsa`, `ip ssh version 2` |
| **Password Policies** | `security passwords min-length`, `login block-for` |
| **AAA** | `aaa new-model`, `aaa authentication login default local` |
| **TFTP** | `tftp-server nvram:startup-config`, port 69 |
| **FTP** | `ip ftp username`, `ip ftp password`, port 20/21 |
| **QoS** | `class-map`, `policy-map`, DSCP EF (46) for voice |

### Access Control Lists (ACLs)
| Type | Syntax | Use Case |
|------|--------|----------|
| **Standard** | `access-list 10 deny host X.X.X.X` | Filter by source only |
| **Extended** | `access-list 100 permit tcp any host X.X.X.X eq 80` | Filter by src/dst/proto/port |
| **Named** | `ip access-list extended NAME` | Readable ACLs |
| **Time-Based** | `time-range NAME`, `periodic weekdays 8:00 to 18:00` | Time-based filtering |

### Layer 2 Security
| Feature | Description |
|---------|-------------|
| **Port Security** | `switchport port-security maximum 2` — limits MACs per port |
| **DHCP Snooping** | `ip dhcp snooping` — prevents rogue DHCP servers |
| **DAI** | `ip arp inspection vlan 1` — prevents ARP poisoning |
| **IP Source Guard** | `ip verify source` — prevents IP spoofing |

### AAA
| Component | Description |
|-----------|-------------|
| **Authentication** | Who are you? (password, token, biometric) |
| **Authorization** | What can you do? (privilege level, command sets) |
| **Accounting** | What did you do? (log commands, sessions, traffic) |

### Wireless Security
| Protocol | Encryption | Authentication | Status |
|----------|-----------|----------------|--------|
| **WEP** | RC4 | None | ❌ Broken |
| **WPA** | TKIP | PSK/Enterprise | ⚠️ Deprecated |
| **WPA2** | AES-CCMP | PSK/Enterprise | ✅ Standard |
| **WPA3** | AES-GCM | SAE | ✅ Best Practice |

### VPN Concepts
| Type | Description |
|------|-------------|
| **Remote Access VPN** | Client-to-site (IPsec, SSL) |
| **Site-to-Site VPN** | IPsec tunnel between two sites (R1 ↔ R2) |

## ✅ Verification Commands
### Security Verification
show ip ssh
show running-config | include aaa
show running-config | include username
show running-config | include crypto key
### ACL Verification
show access-lists
show ip interface fastEthernet 0/1 | include access list
show time-range
### DHCP Verification
show ip dhcp binding
show ip dhcp pool
show ip dhcp conflict
show ip interface fa0/1 | include helper
### Layer 2 Security Verification
show ip dhcp snooping
show ip dhcp snooping binding
show ip arp inspection
show ip arp inspection statistics
show port-security
show port-security interface e0/0
show ip verify source
### QoS Verification
show policy-map
show policy-map interface fastEthernet 1/0
show class-map
### End-to-End Testing
! From VPC6
show ip
ping 192.168.1.1
ping 10.30.30.2
! From VPC7
show ip
ping 10.30.30.2

## 🎓 Key Learnings
Static NAT is bidirectional — allows inbound access to internal hosts/
PAT uses overload — many-to-one with port translation/
ACLs are processed top-down — first match wins/
Standard ACL — filter by source only/
Extended ACL — filter by src/dst/proto/port/
Named ACL — more readable and easier to edit/
Time-Based ACL — active only during configured periods/
Port Security — limits MACs per port; violations can shut down ports/
DHCP Snooping — prevents rogue DHCP servers/
DAI — prevents ARP poisoning/
IP Source Guard — prevents IP spoofing/
AAA — Authentication, Authorization, Accounting/
WPA3 — best wireless security protocol/
TFTP — lightweight, no authentication (port 69)/
FTP — authentication + file transfer (port 20/21)
