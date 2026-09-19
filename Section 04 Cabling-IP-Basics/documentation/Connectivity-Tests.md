# Connectivity Tests

## Objective
Verify basic Layer 1, Layer 2, and Layer 3 operation.

## Router
On R1:
show ip interface brief


## Switch
On SW1:
show interfaces status


## PC1
ping 192.168.10.1
ping 192.168.10.11

## Troubleshooting Exercise
Temporarily configure PC2 as:
192.168.20.11

with `/24` mask. From PC1:
ping 192.168.20.11
The test should fail because PC2 is in a different subnet and no routing for that subnet exists.

Restore PC2:
192.168.10.11

Then test:
ping 192.168.10.11


## Final Result
The lab should demonstrate correct physical connectivity, IPv4 addressing, local communication, and basic troubleshooting.
