# IP Addressing Plan

## Campus

| Network | Purpose |
|---|---|
| 192.168.10.0/24 | IT |
| 192.168.20.0/24 | HR |
| 192.168.30.0/24 | ADMIN |
| 192.168.40.0/24 | SOC |
| 192.168.50.0/24 | SERVERS |
| 192.168.60.0/24 | MANAGEMENT |

## Data Center

| Network | Purpose |
|---|---|
| 192.168.100.0/24 | Web/App |
| 192.168.110.0/24 | Database |
| 192.168.120.0/24 | SOC |

## WAN

| Link | Network |
|---|---|
| HQ-ISP | 10.0.0.0/30 |
| ISP-Cloud | 10.0.0.4/30 |
| ISP-SOHO | 10.0.0.8/30 |

## Data Center Routed Links

| Link | Network |
|---|---|
| CORE-LEAF3 | 10.10.10.0/30 |
| SPINE1-LEAF1 | 10.20.1.0/30 |
| SPINE1-LEAF2 | 10.20.1.4/30 |
| SPINE1-LEAF3 | 10.20.1.8/30 |
| SPINE2-LEAF1 | 10.20.1.12/30 |
| SPINE2-LEAF2 | 10.20.1.16/30 |
| SPINE2-LEAF3 | 10.20.1.20/30 |