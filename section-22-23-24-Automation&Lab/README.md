# Cisco Network Automation & Discovery Protocols Lab
A comprehensive hands-on lab demonstrating **CDP**, **LLDP concepts**, **JSON Data Formatting**, **Software-Defined Networking (SDN)**, **Northbound/Southbound APIs**, and **Network Automation using a Control Node** — built and verified in **EVE-NG**.

## 🎯 Lab Objectives
- Configure and verify **Cisco Discovery Protocol (CDP)**.
- Understand **LLDP** (Link Layer Discovery Protocol) and vendor interoperability.
- Interpret **JSON (JavaScript Object Notation)** data formats.
- Demonstrate **Network Automation** using a Linux Control Node and SSH scripting.
- Understand **Configuration Management mechanisms** (Ansible, Puppet, Chef).
- Understand **SDN Architecture** (Control Plane vs. Data Plane).

## 🖧 Topology Overview

   [R1]───────────[SW]──────────────────[R2]
  (Fa0/0)          │                   (Fa0/0)
                   │ (e0/2)
                   │
               [ Linux ] 
              (TinyCore)
### Device Table
Device	Role	Interface	IP Address
R1	Automation Target	Fa0/0	192.168.1.11/24
R2	Automation Target	Fa0/0	192.168.1.12/24
SW	Layer 2 Switch	VLAN 1	192.168.1.10/24
Linux	Automation Control Node	eth0	192.168.1.50/24
## 📋 Configuration Files
### R1 Configuration
enable
configure terminal
hostname R1
interface fastEthernet 0/0
 ip address 192.168.1.11 255.255.255.0
 duplex full
 speed 100
 no shutdown
exit
ip domain-name lab.local
crypto key generate rsa modulus 1024
username admin privilege 15 password 123
line vty 0 4
 transport input ssh
 login local
exit
end
write memory

### R2 Configuration
enable
configure terminal
hostname R2

interface fastEthernet 0/0
 ip address 192.168.1.12 255.255.255.0
 duplex full
 speed 100
 no shutdown
exit
ip domain-name lab.local
crypto key generate rsa modulus 1024
username admin privilege 15 password 123
line vty 0 4
 transport input ssh
 login local
exit
end
write memory

### SW Configuration
enable
configure terminal
hostname SW
interface vlan 1
 ip address 192.168.1.10 255.255.255.0
 no shutdown
exit
ip default-gateway 192.168.1.1
interface range e0/0-2
 switchport mode access
 switchport access vlan 1
 no shutdown
exit
end
write memory

### Linux (TinyCore) Configuration
bash
# Configure interface
sudo ip link set eth0 up
sudo ip addr add 192.168.1.50/24 dev eth0
sudo ip route add default via 192.168.1.1

## 🛠️ Technologies & Concepts
### 1. Layer 2 Discovery Protocols
Protocol	Standard	Description
CDP	Cisco Proprietary	Discovers directly connected Cisco devices. Shares platform, IP, and port info. Enabled by default.
LLDP	IEEE 802.1AB	Vendor-neutral discovery protocol. Requires manual enabling (lldp run).
# (Note: The Cisco 3725 IOS 12.4 image used in this lab does not support LLDP. The concept is documented for completeness).

### 2. JSON (JavaScript Object Notation)
JSON is a lightweight, text-based, human-readable format used for data exchange. It is heavily used in REST APIs and Network Automation.

# Structure:
Objects: Enclosed in curly braces {} (Key-Value pairs).
Arrays: Enclosed in square brackets [] (Lists).
JSON Equivalent of show ip interface brief:

json
{
  "TABLE_intf": {
    "ROW_intf": {
      "intf-name": "FastEthernet0/0",
      "proto-state": "up",
      "link-state": "up",
      "prefix": "192.168.1.11"
    }
  }
}

### 3. SDN Architecture
Software-Defined Networking separates the Control Plane (routing decisions) from the Data Plane (packet forwarding).
Northbound APIs: Connect the SDN Controller to business applications (e.g., REST API).
Southbound APIs: Connect the SDN Controller to network devices (e.g., OpenFlow, NETCONF, CLI).

### 4. Configuration Management Tools
Tool	Language	Model	Agent
Ansible	Python/YAML	Push	Agentless
Puppet	Ruby	Pull	Agent-based
Chef	Ruby	Pull	Agent-based
SaltStack	Python/YAML	Push	Agent-based

## ✅ Automation Verification (The Lab Output)
Since the isolated lab environment lacks internet access to install Ansible or Python on the minimal Linux node, a Bash Shell Script was used to demonstrate the exact same "Automation" concept (executing commands across multiple devices without manual intervention).

### The Automation Script (auto.sh)
bash
#!/bin/sh
for ip in 192.168.1.11 192.168.1.12; do
  echo "--- Automating Router: $ip ---"
  ssh -o StrictHostKeyChecking=no admin@$ip "show run | include hostname"
done
### Execution Output
text
gns3@box:~$ ./auto.sh
--- Automating Router: 192.168.1.11 ---
Warning: Permanently added '192.168.1.11' (RSA) to the list of known hosts.
Password:
hostname R1
--- Automating Router: 192.168.1.12 ---
Warning: Permanently added '192.168.1.12' (RSA) to the list of known hosts.
Password:
hostname R2
Result: The Control Node successfully automated the retrieval of configurations from two separate routers simultaneously.
### 🎓 Key Learnings
CDP is enabled by default on Cisco devices but only works between Cisco gear.
LLDP is the open-standard alternative but requires newer IOS versions and manual activation.
JSON is the data format of choice for modern APIs and automation scripts.
SDN shifts intelligence to a central controller, communicating via Northbound (Apps) and Southbound (Devices) APIs.
Automation transforms network engineering from manual CLI typing into programmable, scalable scripts.
Configuration Management tools (like Ansible) allow you to manage thousands of devices using code (Infrastructure as Code).
