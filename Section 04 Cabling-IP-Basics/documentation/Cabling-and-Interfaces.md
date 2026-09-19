# Cabling and Interfaces

## Physical Interfaces
Common Cisco Ethernet interfaces include FastEthernet (`Fa`), GigabitEthernet (`Gi`), and TenGigabitEthernet (`Te`).

Examples:

Fa0/1
Gi0/1


## Ethernet Cabling
Common Ethernet media include copper and fiber. Typical copper connections include PC-to-switch and router-to-switch. Historically, crossover cables were commonly used between similar devices; modern equipment often supports automatic MDI/MDIX.

## Fiber Optics
**Single-mode fiber:** generally used for longer-distance communication.
**Multimode fiber:** generally used for shorter-distance communication, including building and data-center environments.

## Lab Connections
R1 ↔ SW1
PC1 ↔ SW1
PC2 ↔ SW1
IP-PHONE1 ↔ SW1
SW1 ↔ SW2


## Verification
show ip interface brief
show interfaces status

