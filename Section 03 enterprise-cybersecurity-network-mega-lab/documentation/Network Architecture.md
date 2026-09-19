# Network Architecture

## Overview

The network combines multiple enterprise architectures into one practical laboratory.

The design contains:

1. Three-tier campus architecture
2. Spine-leaf data-center architecture
3. WAN architecture
4. Cloud environment
5. SOHO branch


## Three-Tier Campus

The headquarters follows the traditional hierarchical model.

### Core

CORE-SW1 provides:

- Layer 3 routing
- Inter-VLAN routing
- Default route toward HQ-R1
- Connectivity to distribution switches
- Connectivity to the data-center border leaf

### Distribution

DIST-SW1 and DIST-SW2 provide:

- VLAN aggregation
- Trunk connectivity
- Server connectivity
- Redundant paths toward the core

### Access

ACCESS-SW1 and ACCESS-SW2 provide endpoint connectivity.


## Spine-Leaf Data Center

The data center uses:

- SPINE-SW1
- SPINE-SW2
- LEAF-SW1
- LEAF-SW2
- LEAF-SW3

Each leaf has connectivity to both spine switches.

LEAF-SW3 operates as the border leaf connecting the data center to CORE-SW1.


## WAN

The WAN consists of:

- HQ-R1
- ISP-R1
- CLOUD-R1
- SOHO-R1

OSPF provides dynamic routing across the routed infrastructure.

## Cloud

The cloud environment contains:

- CLOUD-R1
- CLOUD-SW1
- CLOUD-WEB
The cloud web server provides HTTP service for testing.


## SOHO
The branch office contains:

- SOHO-R1
- SOHO-SW1
- SOHO-PC1
- SOHO-PC2

The SOHO network connects through the ISP to the enterprise environment.