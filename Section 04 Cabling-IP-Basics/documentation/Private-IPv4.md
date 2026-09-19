# Private IPv4 Addressing

Private IPv4 addresses are used inside private networks such as homes, offices, campuses, and laboratories.

## RFC 1918 Ranges

| Range | CIDR |
|---|---|
| 10.0.0.0 – 10.255.255.255 | 10.0.0.0/8 |
| 172.16.0.0 – 172.31.255.255 | 172.16.0.0/12 |
| 192.168.0.0 – 192.168.255.255 | 192.168.0.0/16 |

This lab uses:
192.168.10.0/24


Private addresses let organizations use IPv4 internally without requiring a globally unique public address for every endpoint. They are not directly routable across the public Internet; NAT is commonly used when private hosts communicate with public destinations.

Examples:
10.10.10.10      Private
172.16.5.20      Private
172.32.5.20      Not RFC 1918 private
192.168.1.50     Private
8.8.8.8          Not RFC 1918 private

