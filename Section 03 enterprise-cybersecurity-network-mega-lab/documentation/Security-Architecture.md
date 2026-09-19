# Security Architecture

## Device Hardening

Cisco devices were hardened using:

- Disabled DNS lookup
- Password encryption
- Local administrative authentication
- Privileged EXEC protection
- Console timeout
- SSH-only management
- Security banner

## Layer 2 Security

The following controls were implemented:

### Port Security

Endpoint-facing interfaces use:

- Maximum one MAC address
- Sticky MAC learning
- Restrict violation mode

### PortFast

PortFast is enabled on endpoint interfaces.

### BPDU Guard

BPDU Guard is enabled on endpoint interfaces to protect the access layer from unauthorized spanning-tree participation.

### Unused Interfaces

Unused switch interfaces are administratively shut down.

## Management Security

Remote management uses SSH rather than Telnet.

Telnet access is disabled on VTY lines.