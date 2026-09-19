# IPv4 Addressing

## Lab Network
192.168.10.0/24
255.255.255.0

| Device | Interface | IPv4 Address | Mask | Gateway |
|---|---|---|---|---|
| R1 | G0/0 | 192.168.10.1 | 255.255.255.0 | — |
| PC1 | Fa0 | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 |
| PC2 | Fa0 | 192.168.10.11 | 255.255.255.0 | 192.168.10.1 |

## Router Configuration

interface gigabitEthernet 0/0
ip address 192.168.10.1 255.255.255.0
no shutdown


## Verification
show ip interface brief


From PC1:
ping 192.168.10.1
ping 192.168.10.11

