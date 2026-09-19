# Enterprise Cybersecurity Network Mega Lab
A comprehensive Cisco Packet Tracer enterprise network project designed to demonstrate practical knowledge of enterprise networking, network architecture, routing, switching, WAN connectivity, cloud networking, data-center architecture, and network security.

## Project Overview

This project simulates a multi-site enterprise network containing:

- Enterprise headquarters
- 3-tier campus architecture
- VLAN-based departmental segmentation
- Layer 3 core switching
- Layer 2 distribution and access layers
- Enterprise servers
- WAN connectivity
- ISP network
- Public cloud environment
- SOHO branch office
- Data-center spine-leaf architecture
- OSPF dynamic routing
- SSH management
- Port security
- PortFast
- BPDU Guard
- Unused-port security
- End-to-end connectivity testing
- Security verification and troubleshooting

The project was built and tested using Cisco Packet Tracer.

## Objectives

The main objectives of this project were to:

1. Design an enterprise network architecture.
2. Implement a hierarchical 3-tier campus network.
3. Configure VLAN segmentation.
4. Configure Layer 3 inter-VLAN routing.
5. Deploy enterprise server networks.
6. Implement WAN connectivity.
7. Connect the enterprise to a cloud environment.
8. Build a SOHO branch network.
9. Implement a data-center spine-leaf architecture.
10. Configure OSPF dynamic routing.
11. Implement basic network security controls.
12. Perform end-to-end connectivity testing.
13. Troubleshoot network failures.
14. Document the complete network implementation.

# Network Architecture

## Enterprise Campus

The headquarters uses a hierarchical 3-tier architecture:

                 HQ-R1
                   |
               CORE-SW1
              /         \
        DIST-SW1       DIST-SW2
           |              |
       ACCESS-SW1     ACCESS-SW2
        / | \          / | \
      IT  IT HR       HR ADM SOC

## Layers
Core Layer
CORE-SW1

## Distribution Layer
DIST-SW1
DIST-SW2

## Access Layer
ACCESS-SW1
ACCESS-SW2

## Data Center Architecture
The data center uses a spine-leaf architecture:
                 SPINE-SW1
                /    |    \
               /     |     \
          LEAF1    LEAF2    LEAF3
             \       |       /
              \      |      /
                 SPINE-SW2

The leaf switches provide connectivity to data-center workloads.
LEAF-SW1 → DC-WEB
LEAF-SW1 → DC-APP
LEAF-SW2 → DC-DB
LEAF-SW3 → SOC-SRV
LEAF-SW3 also acts as the border leaf connecting the data center to the enterprise core.      

## WAN Architecture
Enterprise HQ
      |
    HQ-R1
      |
    ISP-R1
     /   \
    /     \
Cloud     SOHO

The WAN connects:
Headquarters
ISP
Public cloud
SOHO branch

## VLAN Plan
| VLAN | Name        | Network          | Gateway       |
| ---- | ----------- | ---------------- | ------------- |
| 10   | IT          | 192.168.10.0/24  | 192.168.10.1  |
| 20   | HR          | 192.168.20.0/24  | 192.168.20.1  |
| 30   | ADMIN       | 192.168.30.0/24  | 192.168.30.1  |
| 40   | SOC         | 192.168.40.0/24  | 192.168.40.1  |
| 50   | SERVERS     | 192.168.50.0/24  | 192.168.50.1  |
| 60   | MANAGEMENT  | 192.168.60.0/24  | 192.168.60.1  |
| 100  | DC-WEB-APP  | 192.168.100.0/24 | 192.168.100.1 |
| 110  | DC-DATABASE | 192.168.110.0/24 | 192.168.110.1 |
| 120  | DC-SOC      | 192.168.120.0/24 | 192.168.120.1 |

## IP Addressing
Enterprise Servers
| Device      | IP Address    |
| ----------- | ------------- |
| DNS-SERVER  | 192.168.50.10 |
| WEB-SERVER  | 192.168.50.20 |
| FILE-SERVER | 192.168.50.30 |
| SOC-SERVER  | 192.168.50.40 |

## Data Center
| Device  | IP Address     |
| ------- | -------------- |
| DC-WEB  | 192.168.100.10 |
| DC-APP  | 192.168.100.20 |
| DC-DB   | 192.168.110.10 |
| SOC-SRV | 192.168.120.10 |

## Cloud
| Device    | IP Address    |
| --------- | ------------- |
| CLOUD-WEB | 172.16.100.10 |

## SOHO
| Device   | IP Address     |
| -------- | -------------- |
| SOHO-PC1 | 192.168.200.10 |
| SOHO-PC2 | 192.168.200.11 |

## Routing
OSPF process 10 is used as the primary dynamic routing protocol.
OSPF Process: 10
Area: 0
OSPF is used across:
Core
Border leaf
Spine switches
Leaf switches
WAN routers
The enterprise core also provides the default route toward HQ-R1.

## Security Controls
The project implements several basic network security controls.

## SSH
Cisco device management is secured using SSH.
Local administrator account
SSH authentication
SSH-only VTY access
Telnet disabled
Privileged EXEC password protection

## Port Security
Access and server-facing ports use:
Maximum MAC address: 1
Sticky MAC addresses
Violation mode: restrict

## PortFast
PortFast is enabled on end-device ports to allow hosts to transition to forwarding state quickly.

## BPDU Guard
BPDU Guard is enabled on endpoint ports to protect against unauthorized switches being connected to access ports.

## Unused Ports
Unused switch interfaces are administratively shut down.

## Login Protection
Login protection is configured where supported by the Packet Tracer IOS image.

## Testing
The network was tested using:
ICMP ping
HTTP access
OSPF neighbor verification
Routing table verification
VLAN verification
Trunk verification
Port-security verification
PortFast/BPDU Guard verification
SSH authentication
Telnet protection testing

##Troubleshooting
During development, connectivity problems were deliberately investigated and corrected.
One issue involved the enterprise WEB-SERVER not being reachable because its switch port was not correctly assigned to VLAN 50.
The issue was resolved by configuring the server-facing interface as an access port in VLAN 50.
This demonstrated practical troubleshooting involving:
VLAN assignment
Access-port configuration
Layer 2 forwarding
End-to-end connectivity testing

## Technologies Used
Cisco Packet Tracer
Cisco IOS
IPv4
VLAN
802.1Q Trunking
Inter-VLAN Routing
OSPF
SSH
Port Security
PortFast
BPDU Guard
WAN
Cloud Networking
SOHO Networking
3-Tier Architecture
Spine-Leaf Architecture

## Project Evidence
Screenshots documenting the implementation are available in the Screenshots directory.
Configuration files are available in the Configuration directory.

## Learning Outcomes
Through this project, I learned how to:
Design an enterprise network.
Build a hierarchical 3-tier architecture.
Configure VLAN segmentation.
Configure trunk links.
Implement inter-VLAN routing.
Configure enterprise server networks.
Build WAN connections.
Connect enterprise networks to cloud environments.
Build SOHO networks.
Design a spine-leaf data-center network.
Configure OSPF.
Secure Cisco devices using SSH.
Configure port security.
Configure PortFast and BPDU Guard.
Disable unused switch ports.
Test enterprise connectivity.
Troubleshoot VLAN and connectivity problems.
Document a complete cybersecurity networking project.

## Author
Cybersecurity student building practical networking and security skills through hands-on laboratory projects.
