# VLAN Plan

## SecureTech Corporation

| VLAN | Name   | Purpose                | Network         | Gateway |
|      |        |                        |
| 10   | IT     | IT Department          | 192.168.10.0/24 | 192.168.10.1 |
| 20   | HR     | Human Resources        | 192.168.20.0/24 | 192.168.20.1 |
| 30   | MGM    | Management             | 192.168.30.0/24 | 192.168.30.1 |
| 40   | GUEST  | Guest Users            | 192.168.40.0/24 | 192.168.40.1 |
| 50   | SRV    | Internal Servers       | 192.168.50.0/24 | 192.168.50.1 |
| 60   | SOC    | Security Operations    | 192.168.60.0/24 | 192.168.60.1 |
| 70   | ADMIN  | Network Administration | 192.168.70.0/24 | 192.168.70.1 |

## Security Purpose

The network is segmented into separate VLANs to reduce unnecessary communication
between departments and to provide a foundation for access-control policies and security monitoring.