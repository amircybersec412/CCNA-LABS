# Network Architecture

## Overview

The SecureTech Corporation network is designed using a segmented enterprise architecture.
The network separates users, servers, guests, security operations, and network administration into different VLANs.

## Architecture Layers

### Router

R1 provides inter-VLAN routing and acts as the default gateway for the VLANs.

### Distribution Switch

SW1 provides the central switching and trunking infrastructure.

### Access Switches

SW2 and SW3 connect end-user devices, servers, and other endpoints to their assigned VLANs.

## Security Architecture

Network segmentation is used to establish logical security boundaries.
The Guest VLAN is separated from internal business networks.
The Server VLAN provides a dedicated segment for internal services.
The SOC VLAN provides a dedicated area for security operations.
The Admin VLAN is reserved for network administration.
Further access-control and hardening controls will be implemented in later phases.