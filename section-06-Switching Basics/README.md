# CCNA Section 06 — Cisco Switching, MAC Address Table, VLANs, SVI & Switch Hardening
## Overview
This lab covers the main concepts from **CCNA Section 06**, including Cisco switching, MAC address tables, access ports, VLANs, trunking, default VLANs, Switch Virtual Interfaces (SVIs), Layer 2 and Layer 3 switching, inter-VLAN routing, Auto-MDIX, and basic switch hardening.
The lab was implemented using **Cisco Packet Tracer**.
## Learning Objectives
By completing this lab, I learned how to:
- Understand how Cisco switches learn MAC addresses.
- Examine the MAC address table.
- Understand dynamic and static MAC addresses.
- Configure access ports.
- Create and configure VLANs.
- Understand the default VLAN.
- Configure trunk ports.
- Configure a Switch Virtual Interface (SVI).
- Configure a Layer 3 switch.
- Configure inter-VLAN routing.
- Understand Layer 2 vs Layer 3 switching.
- Configure Auto-MDIX.
- Apply basic Cisco switch hardening.
- Configure SSH access.
- Verify switch configuration.
- Troubleshoot VLAN and IP connectivity problems.
# 1. Lab Topology
## Devices
| Device | Packet Tracer Default Name | Model | Role |
|---|---|---|---|
| SW1-L2 | Switch0 | Cisco 2960 | Layer 2 Switch |
| SW2-L3 | Switch1 | Cisco 3560 | Layer 3 Switch |
| PC1 | PC0 | PC | IT Client |
| PC2 | PC1 | PC | HR Client |
## Device Renaming
The Packet Tracer devices were renamed as follows:
Switch0 → SW1-L2
Switch1 → SW2-L3
PC0 → PC1
PC1 → PC2
## Physical Connections
| Device | Interface | Device | Interface | Cable |
|---|---|---|---|---|
| PC1 | Fa0 | SW1-L2 | Fa0/1 | Copper Straight-Through |
| PC2 | Fa0 | SW1-L2 | Fa0/2 | Copper Straight-Through |
| SW1-L2 | Gi0/1 | SW2-L3 | Gi0/1 | Copper Straight-Through |
# 2. VLAN and IP Addressing Plan
## VLAN Plan
| VLAN | Name | Network | Purpose |
|---|---|---|---|
| 10 | IT | 192.168.10.0/24 | IT users |
| 20 | HR | 192.168.20.0/24 | HR users |
| 99 | MANAGEMENT | 192.168.99.0/24 | Switch management |
## IP Addressing
| Device | Interface | IP Address | Subnet Mask | Default Gateway |
|---|---|---|---|---|
| SW1-L2 | VLAN 99 | 192.168.99.2 | 255.255.255.0 | 192.168.99.1 |
| SW2-L3 | VLAN 10 | 192.168.10.1 | 255.255.255.0 | — |
| SW2-L3 | VLAN 20 | 192.168.20.1 | 255.255.255.0 | — |
| SW2-L3 | VLAN 99 | 192.168.99.1 | 255.255.255.0 | — |
| PC1 | Fa0 | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 |
| PC2 | Fa0 | 192.168.20.10 | 255.255.255.0 | 192.168.20.1 |
# 3. VLAN Configuration
VLANs divide a Layer 2 network into separate broadcast domains.
In this lab:
- VLAN 10 represents IT.
- VLAN 20 represents HR.
- VLAN 99 is used for management.
# 4. SW1-L2 Configuration
Enter the following configuration on **SW1-L2**:
enable
configure terminal
hostname SW1-L2
no ip domain-lookup
vlan 10
name IT
exit
vlan 20
name HR
exit
vlan 99
name MANAGEMENT
exit
interface fastEthernet 0/1
switchport mode access
switchport access vlan 10
spanning-tree portfast
no shutdown
exit
interface fastEthernet 0/2
switchport mode access
switchport access vlan 20
spanning-tree portfast
no shutdown
exit
interface vlan 99
ip address 192.168.99.2 255.255.255.0
no shutdown
exit
ip default-gateway 192.168.99.1
interface gigabitEthernet 0/1
switchport mode trunk
no shutdown
exit
end
copy running-config startup-config
# 5. SW2-L3 Configuration
Enter the following configuration on **SW2-L3**:
enable
configure terminal
hostname SW2-L3
no ip domain-lookup
vlan 10
name IT
exit
vlan 20
name HR
exit
vlan 99
name MANAGEMENT
exit
interface gigabitEthernet 0/1
switchport mode trunk
no shutdown
exit
interface vlan 10
ip address 192.168.10.1 255.255.255.0
no shutdown
exit
interface vlan 20
ip address 192.168.20.1 255.255.255.0
no shutdown
exit
interface vlan 99
ip address 192.168.99.1 255.255.255.0
no shutdown
exit
ip routing
end
copy running-config startup-config
# 6. PC Configuration
## PC1
Configure PC1 with:
IP Address: 192.168.10.10
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.1
## PC2
Configure PC2 with:
IP Address: 192.168.20.10
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.20.1
# 7. Access Ports
An access port normally carries traffic belonging to a single VLAN.
In this lab:
SW1-L2 Fa0/1 → VLAN 10 → PC1
SW1-L2 Fa0/2 → VLAN 20 → PC2
Example configuration:
interface fastEthernet 0/1
switchport mode access
switchport access vlan 10
The second access port was configured for VLAN 20:
interface fastEthernet 0/2
switchport mode access
switchport access vlan 20
# 8. Trunk Port
The connection between SW1-L2 and SW2-L3 is configured as a trunk.
SW1-L2:
interface gigabitEthernet 0/1
switchport mode trunk
no shutdown
SW2-L3:
interface gigabitEthernet 0/1
switchport mode trunk
no shutdown
The trunk allows multiple VLANs to travel between the switches.
# 9. Default VLAN
Cisco switches initially place ports into **VLAN 1**.
Important characteristics of VLAN 1:
- VLAN 1 exists by default.
- Switch ports initially belong to VLAN 1.
- VLAN 1 is the default VLAN.
- VLAN 1 cannot be renamed.
- VLAN 1 cannot be deleted.
- Some Cisco control traffic is associated with VLAN 1.
Verify VLAN information with:
show vlan brief
# 10. MAC Address Table
A switch uses MAC addresses to make Layer 2 forwarding decisions.
The MAC address is a **48-bit hardware address**.
A MAC address consists of:
48 bits = 24-bit OUI + 24-bit device-specific portion
The OUI identifies the manufacturer, while the remaining portion identifies the network interface.
A Cisco MAC address table contains information such as:
- VLAN ID
- MAC address
- Address type
- Switch port
Example:
Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
10      xxxx.xxxx.xxxx    DYNAMIC     Fa0/1
20      xxxx.xxxx.xxxx    DYNAMIC     Fa0/2
# 11. Dynamic MAC Addresses
A switch automatically learns the source MAC address of incoming Ethernet frames.
For example:
PC1 → SW1-L2
When SW1-L2 receives a frame from PC1, it can learn PC1's MAC address and associate it with the incoming port.
Check the MAC address table:
show mac address-table
Check only dynamically learned addresses:
show mac address-table dynamic
# 12. Unknown Destination MAC Addresses
If a switch receives an Ethernet frame with a destination MAC address that is not currently present in its MAC address table, the switch floods the frame out the appropriate ports within the VLAN, except the port on which the frame was received.
Once the destination device responds, the switch can learn the source MAC address from the returning frame.
# 13. MAC Address Aging
Cisco switches use MAC address aging to remove old dynamically learned MAC addresses.
The commonly used default aging time on Cisco switches is:
300 seconds
The aging timer can be modified by the administrator when required.
# 14. Static MAC Address
A static MAC address can be manually assigned to a specific VLAN and switch interface.
First, obtain the actual MAC address of PC1.
On PC1:
ipconfig /all
Find the MAC address shown as the physical address.
Then configure the actual MAC address on SW1-L2:
enable
configure terminal
mac address-table static <PC1-MAC-ADDRESS> vlan 10 interface fastEthernet 0/1
end
For example, if Packet Tracer shows:
00E0.8F12.3456
the command would be:
mac address-table static 00E0.8F12.3456 vlan 10 interface fastEthernet 0/1
Do not use the example MAC address unless it is actually the MAC address of PC1.
Verify the static MAC address:
show mac address-table static
# 15. Switch Virtual Interface (SVI)
An SVI is a logical Layer 3 interface associated with a VLAN.
On the Layer 3 switch, the following SVIs were configured:
interface vlan 10
ip address 192.168.10.1 255.255.255.0
no shutdown
interface vlan 20
ip address 192.168.20.1 255.255.255.0
no shutdown
interface vlan 99
ip address 192.168.99.1 255.255.255.0
no shutdown
SVIs can be used for:
- Switch management
- Testing
- Layer 3 routing
- Inter-VLAN communication
# 16. Layer 2 vs Layer 3 Switch
## Layer 2 Switch
A Layer 2 switch primarily forwards Ethernet frames using MAC addresses.
In this lab:
SW1-L2
is used as the Layer 2 switch.
## Layer 3 Switch
A Layer 3 switch can perform both switching and IP routing.
In this lab:
SW2-L3
is used as the Layer 3 switch.
The Layer 3 switch performs routing between VLANs.
# 17. Inter-VLAN Routing
PC1 belongs to VLAN 10:
192.168.10.10/24
PC2 belongs to VLAN 20:
192.168.20.10/24
Because these are different IP networks, communication between them requires Layer 3 routing.
SW2-L3 performs this routing using the configured SVIs.
The following command enables Layer 3 routing:
ip routing
The basic communication path is:
PC1
192.168.10.10
     |
   VLAN 10
     |
  SW1-L2
     |
   TRUNK
     |
  SW2-L3
     |
   VLAN 20
     |
PC2
192.168.20.10
# 18. Auto-MDIX
Auto-MDIX allows a compatible Cisco interface to automatically determine whether the connected Ethernet cable requires straight-through or crossover pin configuration.
Configure Auto-MDIX:
enable
configure terminal
interface fastEthernet 0/1
mdix auto
exit
end
Speed and duplex should normally remain on automatic negotiation:
speed auto
duplex auto
Verify interface information with:
show interfaces fastEthernet 0/1
# 19. Basic Switch Hardening
Basic switch hardening helps reduce unauthorized access to the network device.
The following security controls were configured.
## Enable Secret
enable
configure terminal
enable secret C1scoEnable!
The enable secret protects privileged EXEC mode.
# 20. Local Administrator Account
Create a local administrator account:
username admin privilege 15 secret AdminSecure!
This account can be used for local authentication, including SSH.
# 21. Password Encryption
Enable password encryption:
service password-encryption
This causes plaintext passwords in relevant portions of the running configuration to be obfuscated.
# 22. Console Authentication
Configure console authentication:
line console 0
password ConsoleSecure!
login
exit
# 23. Login Banner
Configure a warning banner:
banner motd #AUTHORIZED ACCESS ONLY#
This displays a warning message when users access the device.
# 24. SSH Configuration
SSH provides encrypted remote management access.
Configure the domain name:
ip domain-name cyberlab.local
Create RSA keys:
crypto key generate rsa
When prompted for the modulus size, use:
2048
Enable SSH version 2:
ip ssh version 2
Configure the VTY lines:
line vty 0 15
login local
transport input ssh
exit
This configuration:
- Uses the local username database.
- Allows SSH.
- Prevents Telnet access through these VTY lines.
# 25. SSH Verification
Check SSH status:
show ip ssh
Check the VTY configuration:
show running-config | section line vty
The configuration should show the local login method and SSH-only transport.
# 26. VLAN Verification
On SW1-L2:
show vlan brief
The output should show:
VLAN 10    IT
VLAN 20    HR
VLAN 99    MANAGEMENT
The access ports should also appear under their corresponding VLANs.
# 27. Trunk Verification
On SW1-L2:
show interfaces trunk
On SW2-L3:
show interfaces trunk
The Gi0/1 connection should appear as a trunk.
# 28. SVI Verification
On SW2-L3:
show ip interface brief
Expected relevant interfaces:
Vlan10    192.168.10.1
Vlan20    192.168.20.1
Vlan99    192.168.99.1
The interfaces should be operational when their associated VLANs and ports are active.
# 29. Routing Table Verification
On SW2-L3:
show ip route
The routing table should contain the connected networks:
192.168.10.0/24
192.168.20.0/24
192.168.99.0/24
# 30. MAC Address Table Verification
On SW1-L2:
show mac address-table
Dynamic MAC addresses:
show mac address-table dynamic
Static MAC addresses:
show mac address-table static
# 31. PC Connectivity Testing
## PC1 to Its Gateway
From PC1:
ping 192.168.10.1
Expected result:
Reply from 192.168.10.1
## PC2 to Its Gateway
From PC2:
ping 192.168.20.1
Expected result:
Reply from 192.168.20.1
# 32. Inter-VLAN Connectivity Test
From PC1:
ping 192.168.20.10
Expected result:
Reply from 192.168.20.10
This confirms that:
- PC1 has correct IP configuration.
- VLAN 10 is functioning.
- The trunk is functioning.
- SW2-L3 is routing.
- VLAN 20 is functioning.
- PC2 has correct IP configuration.
# 33. Troubleshooting Exercise
To simulate an incorrect IP configuration, temporarily change PC2's IP address.
## Incorrect PC2 Configuration
IP Address: 192.168.30.10
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.20.1
From PC1:
ping 192.168.30.10
The test should fail because PC2 is no longer using the expected VLAN 20 IP subnet.
# 34. Restore PC2 Configuration
Restore PC2:
IP Address: 192.168.20.10
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.20.1
Test again from PC1:
ping 192.168.20.10
The ping should succeed.
This demonstrates how an incorrect IP configuration can cause connectivity problems even when the physical network and VLAN configuration are functioning.
# 35. Final Verification Checklist
Before completing the lab, verify the following.
## SW1-L2
Run:
show vlan brief
show interfaces trunk
show mac address-table
show mac address-table dynamic
show mac address-table static
show ip interface brief
show running-config
## SW2-L3
Run:
show vlan brief
show interfaces trunk
show ip interface brief
show ip route
show mac address-table
show ip ssh
show running-config | section line vty
## PC1
Run:
ipconfig
ping 192.168.10.1
ping 192.168.20.10
## PC2
Run:
ipconfig
ping 192.168.20.1
ping 192.168.10.10
# 36. Screenshots to Capture
The following screenshots should be included in the GitHub repository.
| Screenshot | Description |
|---|---|
| S6-01 | Final Packet Tracer topology |
| S6-02 | SW1 `show vlan brief` |
| S6-03 | SW1 `show interfaces trunk` |
| S6-04 | `show mac address-table` |
| S6-05 | Dynamic and static MAC address table |
| S6-06 | SW2 `show ip interface brief` |
| S6-07 | SW2 `show ip route` |
| S6-08 | Successful PC1 → PC2 ping |
| S6-09 | SSH and switch security configuration |
| S6-10 | Failed ping during troubleshooting |
| S6-11 | Successful ping after fixing PC2 |
# 37. Troubleshooting Concepts Practiced
During this lab, I practiced troubleshooting problems related to:
- Incorrect VLAN assignment
- Incorrect IP addressing
- Incorrect default gateway
- Trunk configuration
- SVI configuration
- Layer 3 routing
- MAC address learning
- Static MAC configuration
- Switch management
- SSH configuration
- Physical and logical connectivity
A basic troubleshooting process used in this lab was:
1. Check physical connectivity.
2. Check VLAN assignment.
3. Check trunk status.
4. Check IP addressing.
5. Check default gateway.
6. Check SVI status.
7. Check routing table.
8. Check MAC address table.
9. Test connectivity with ping.
10. Correct the configuration and retest.
# 38. Security Relevance
The concepts practiced in this lab are important for cybersecurity because network defenders need to understand how switches forward traffic and how VLANs separate network segments.
The lab demonstrated:
- Network segmentation using VLANs.
- Layer 2 MAC address learning.
- Layer 3 routing between VLANs.
- Secure remote administration using SSH.
- Basic device hardening.
- Management network separation.
- MAC address visibility and analysis.
- Network troubleshooting.
Understanding these concepts provides a foundation for later cybersecurity topics such as network monitoring, IDS/IPS, access control, network segmentation, and incident investigation.
# 39. Lessons Learned
- I learned how Cisco switches use MAC addresses to forward Ethernet frames.
- I learned how dynamic MAC addresses are learned automatically.
- I learned how static MAC addresses can be configured manually.
- I learned how to create VLANs.
- I learned how to configure access ports.
- I learned how to configure trunk ports.
- I learned the purpose of the default VLAN.
- I learned how to configure an SVI.
- I learned the difference between Layer 2 and Layer 3 switches.
- I learned how inter-VLAN routing works.
- I learned how to configure Auto-MDIX.
- I learned how to configure basic switch security.
- I learned how to configure SSH.
- I learned how to verify VLANs, trunks, SVIs, routes, and MAC tables.
- I learned how incorrect IP addressing can cause connectivity problems.
- I learned a basic structured approach to network troubleshooting.
# 40. Skills Demonstrated
- Cisco IOS
- Cisco Packet Tracer
- VLAN configuration
- Access-port configuration
- Trunk configuration
- MAC address-table analysis
- Dynamic MAC learning
- Static MAC configuration
- Layer 2 switching
- Layer 3 switching
- SVI configuration
- Inter-VLAN routing
- Auto-MDIX
- Network segmentation
- SSH configuration
- Basic network hardening
- Network troubleshooting
- Connectivity testing
# 41. Lab Summary
This lab combined the major concepts from CCNA Section 06 into one practical Cisco Packet Tracer environment.
The final topology used:
PC1
 |
VLAN 10
 |
SW1-L2
 |
TRUNK
 |
SW2-L3
 |
VLAN 20
 |
PC2
SW1-L2 was responsible primarily for Layer 2 switching, VLAN assignment, and MAC address learning.
SW2-L3 provided Layer 3 functionality through SVIs and performed inter-VLAN routing.
Basic switch hardening and SSH were also configured to demonstrate secure network-device administration.
# 42. Lab Status
**Lab:** CCNA Section 06 — Cisco Switching, MAC Address Table, VLANs, SVI & Switch Hardening
**Platform:** Cisco Packet Tracer
**Status:** Completed
**Primary Focus:** Switching, VLANs, MAC Address Tables, Trunking, SVIs, Inter-VLAN Routing, Auto-MDIX, SSH, Hardening and Troubleshooting
