# Lab 05 — IP Addressing, Subnetting, Interface Troubleshooting, TCP/UDP & Virtualization

## Overview

This practical lab covers the following CCNA Section 05 topics:

- Verifying IP parameters on Windows and Linux
- Identifying interface status, speed, and duplex information
- Understanding IPv4 subnetting
- Comparing TCP and UDP
- Understanding virtualization and virtual machines
- Performing basic connectivity and physical-interface troubleshooting

All planned activities were completed successfully.

## Learning Objectives

After completing this lab, I can:

1. Configure and verify IPv4 addresses on Cisco router interfaces.
2. Configure IPv4 parameters on Packet Tracer PCs.
3. Divide a `/24` network into `/25` and `/26` subnets.
4. Verify interface status, speed, and duplex information.
5. Identify a disconnected switch interface.
6. Use Windows and Linux commands to inspect IP configuration and routing.
7. Explain the main differences between TCP and UDP.
8. Explain how a virtual machine uses a virtual network interface.
9. Troubleshoot basic connectivity problems.

## Topology

                    R1
                 Cisco 2911
                /           \
             G0/0           G0/1
               |             |
             G0/1           G0/1
              SW1            SW2
               |              |
             Fa0/1          Fa0/1
               |              |
        PC1-Windows       PC2-Windows

## Device Mapping

| Default Device | Final Name |
|---|---|
| Router0 | R1 |
| Switch0 | SW1 |
| Switch1 | SW2 |
| PC0 | PC1-Windows |
| PC1 | PC2-Windows |

## IPv4 Address Plan

| Device | Interface | IPv4 Address | Mask | Gateway |
|---|---|---|---|---|
| R1 | G0/0 | 192.168.10.1 | 255.255.255.128 | — |
| PC1-Windows | Fa0 | 192.168.10.10 | 255.255.255.128 | 192.168.10.1 |
| R1 | G0/1 | 192.168.10.129 | 255.255.255.128 | — |
| PC2-Windows | Fa0 | 192.168.10.130 | 255.255.255.128 | 192.168.10.129 |

## Router Configuration

enable
configure terminal
hostname R1
no ip domain-lookup

interface gigabitEthernet 0/0
 description LAN-1-to-SW1
 ip address 192.168.10.1 255.255.255.128
 no shutdown
 exit

interface gigabitEthernet 0/1
 description LAN-2-to-SW2
 ip address 192.168.10.129 255.255.255.128
 no shutdown
 exit
end


## Verification Commands

show ip interface brief
show running-config
show interfaces status
show interfaces fa0/1

On Windows:

#powershell
ipconfig
ipconfig /all
ping 192.168.10.1
ping 192.168.10.130


On Ubuntu:

#bash
ip addr
ip route
getent hosts google.com
ping -c 4 8.8.8.8


## IPv4 Subnetting Summary

### Two `/25` Subnets from `192.168.10.0/24`

`192.168.10.0/25` — usable range `192.168.10.1–192.168.10.126`, broadcast `192.168.10.127`
`192.168.10.128/25` — usable range `192.168.10.129–192.168.10.254`, broadcast `192.168.10.255`

Each `/25` subnet contains 128 total addresses and 126 usable host addresses.

### Four `/26` Subnets from `192.168.20.0/24`

| Network | Usable Range | Broadcast |
|---|---|---|
| `192.168.20.0/26` | `.1–.62` | `.63` |
| `192.168.20.64/26` | `.65–.126` | `.127` |
| `192.168.20.128/26` | `.129–.190` | `.191` |
| `192.168.20.192/26` | `.193–.254` | `.255` |

## TCP vs UDP

| Feature | TCP | UDP |
|---|---|---|
| Connection | Connection-oriented | Connectionless |
| Reliability | Reliable delivery | No built-in guarantee |
| Ordering | Maintains order | No built-in ordering |
| Retransmission | Yes | No |
| Overhead | Higher | Lower |
| Examples | HTTPS, SSH, FTP | DNS, DHCP, VoIP, streaming |

Understanding TCP and UDP is important for firewall configuration, packet analysis, port scanning, traffic monitoring, and incident investigation.

## Virtualization Summary

Physical Computer
        |
Windows Host OS
        |
VMware Workstation
        |
Ubuntu Virtual Machine
        |
Virtual Network Adapter
        |
VMware NAT / Virtual Network
        |
External Network


The Ubuntu VM used the `ens33` interface. The following commands were used to inspect its network configuration and connectivity:

#bash
ip addr
ip route
ping -c 4 8.8.8.8
getent hosts google.com


A hostname may resolve successfully even when the destination does not answer ICMP ping. Therefore, DNS resolution and ICMP reachability should be tested separately.

## Troubleshooting Activity

The connection between `SW1 Fa0/1` and `PC1-Windows Fa0` was temporarily disconnected.

The switch was checked with:
show interfaces status


The affected interface showed `notconnect`. After reconnecting the cable, the interface returned to `connected`, and connectivity was tested again.


## Final Result

The router, switches, Packet Tracer PCs, and Ubuntu virtual machine were checked successfully. IPv4 addressing, subnetting, interface verification, TCP/UDP concepts, virtualization, and basic physical troubleshooting were completed.
