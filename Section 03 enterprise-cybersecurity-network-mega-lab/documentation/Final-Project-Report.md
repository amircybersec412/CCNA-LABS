# Final Project Report

## Project

Enterprise Cybersecurity Network Mega Lab

## Platform

Cisco Packet Tracer

## Summary

This project simulates a complete enterprise network environment combining campus networking, data-center networking, WAN connectivity, cloud services, and a SOHO branch.

The network was designed and implemented from the physical topology through routing, connectivity, security hardening, testing, troubleshooting, and documentation.

## Architecture

The enterprise campus uses a 3-tier architecture consisting of:

- Core
- Distribution
- Access

The data center uses a spine-leaf architecture.

The WAN connects the headquarters, ISP, cloud, and SOHO branch.

## Routing

OSPF was implemented as the primary dynamic routing protocol.

The enterprise core provides the internal default route toward the WAN.

## Security

Security controls included:

- SSH
- Local authentication
- Privileged EXEC protection
- Port security
- Sticky MAC addresses
- PortFast
- BPDU Guard
- Unused port shutdown
- Login protection
- Security banner

## Testing

The network was tested using:

- ICMP
- HTTP
- OSPF neighbor verification
- Routing tables
- VLAN verification
- Trunk verification
- Security verification
- SSH testing
- Telnet protection testing

## Troubleshooting

A server connectivity issue was identified during testing.

The issue was traced to an incorrect VLAN assignment on the server-facing switch port.

After correcting the VLAN configuration, end-to-end connectivity was restored.

## Conclusion

The project provided practical experience with enterprise networking and foundational network security.

It demonstrates the ability to design, configure, secure, test, troubleshoot, and document a multi-architecture Cisco network.