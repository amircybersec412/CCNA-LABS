Incident
Unauthorized Guest Network Access Attempt

Source
192.168.40.0/24

Target
Internal enterprise networks

Detection
R1 GUEST-RESTRICTION ACL

Action
Traffic denied

Evidence
ACL match counters

Response
1. Identify source network
2. Confirm ACL activity
3. Verify internal networks remain available
4. Confirm Guest isolation
5. Monitor for repeated attempts

Status
CONTAINED