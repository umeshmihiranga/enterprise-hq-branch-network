Enterprise HQ–Branch Network

A redundant enterprise network designed and implemented using Cisco Packet Tracer, simulating a real-world headquarters, branch office, internal services, and Internet-connected enterprise environment.

📌 Project Overview

This project demonstrates the design, configuration, and troubleshooting of an enterprise network using Cisco routers and multilayer/access switches.

The network was built with a focus on:

Scalability
Redundancy
Dynamic routing
VLAN segmentation
High availability
Centralized network services
Layer 2 and Layer 3 troubleshooting

The project also documents real configuration issues encountered during implementation and the troubleshooting process used to resolve them.

🏗️ Network Architecture
┌──────────────┐
│ ISP │
│ 8.8.8.8/32 │
└──────┬───────┘
│
203.0.113.0/30
│
┌──────┴───────┐
│ R1-HQ │
│ HQ Edge/WAN │
└──┬────┬───┬──┘
│ │ │
OSPF │ │ │ OSPF
│ │ │
┌───────┘ │ └────────┐
│ │ │
┌──────┴─────┐ │ ┌──────┴─────┐
│ CORE-SW1 │═════╪═════│ CORE-SW2 │
│ HSRP │ LACP│ │ HSRP │
│ Active │ │ │ Standby │
└──────┬─────┘ │ └──────┬─────┘
│ │ │
┌──────┴──────┐ │ ┌──────┴──────┐
│ ACCESS-SW1 │ │ │ ACCESS-SW2 │
└──────┬──────┘ │ └──────┬──────┘
│ │ │
┌────┴────┐ │ ┌────┴─────────┐
│ Users │ │ │ Guest/Servers│
└─────────┘ │ └──────────────┘
│
┌──────┴──────┐
│ R2-BRANCH │
│ OSPF │
└──────┬──────┘
│
┌──────┴──────┐
│ BRANCH-SW1 │
└──────┬──────┘
│
Branch User
🖥️ Devices
Routers
Device Role
R1-HQ HQ edge, WAN, NAT/PAT
R2-BRANCH Branch routing
ISP Simulated Internet
Multilayer Switches
Device Role
CORE-SW1 Primary Layer 3 core/distribution
CORE-SW2 Secondary Layer 3 core/distribution
Access Switches
Device Role
ACCESS-SW1 User access
ACCESS-SW2 Guest and server access
BRANCH-SW1 Branch access
End Devices
SALES-PC1
SALES-PC2
IT-PC1
IT-PC2
GUEST-PC1
BRANCH-PC1
SRV-DHCP-DNS
SRV-WEB
🌐 VLAN Design
VLAN Name Network Purpose
10 SALES 192.168.10.0/24 Sales users
20 IT 192.168.20.0/24 IT users
30 SERVERS 192.168.30.0/24 Network services
40 GUEST 192.168.40.0/24 Guest users
50 VOICE 192.168.50.0/24 Reserved voice network
99 MANAGEMENT 192.168.99.0/24 Network management
999 NATIVE-BLACKHOLE — Native/unused VLAN
100 BRANCH-USERS 192.168.100.0/24 Branch users
🔀 Switching Technologies

The project implements:

VLAN segmentation
802.1Q trunking
Native VLAN configuration
Allowed VLAN lists
Rapid-PVST+
STP root bridge selection
PortFast
BPDU Guard
LACP EtherChannel

The two core switches use an LACP EtherChannel:

CORE-SW1 Fa0/23 ─────┐
├── Port-Channel1
CORE-SW1 Fa0/24 ─────┘
║
║
CORE-SW2 Fa0/23 ─────┐
├── Port-Channel1
CORE-SW2 Fa0/24 ─────┘
🚦 Layer 3 Routing
Inter-VLAN Routing

CORE-SW1 and CORE-SW2 perform Layer 3 routing using SVIs.

Each production VLAN has a redundant HSRP gateway.

Example:

VLAN 10

CORE-SW1 192.168.10.2
CORE-SW2 192.168.10.3
HSRP VIP 192.168.10.1
OSPF

OSPF process 10 is used across the routed enterprise infrastructure.

Device OSPF Router ID
R1-HQ 1.1.1.1
CORE-SW1 2.2.2.2
CORE-SW2 3.3.3.3
R2-BRANCH 4.4.4.4

OSPF Area 0 provides dynamic route exchange between:

HQ
Core switches
Branch
🔄 High Availability
HSRP

HSRP provides redundant default gateways.

CORE-SW1

Priority: 110
Role: Active
Preempt: Enabled

CORE-SW2

Priority: 100
Role: Standby
Preempt: Enabled

If the primary gateway becomes unavailable, CORE-SW2 can take over the virtual gateway.

EtherChannel

LACP combines multiple physical links into a single logical Port-Channel.

Benefits include:

Link redundancy
Increased aggregate bandwidth
Reduced STP complexity
Continued operation if one physical member fails
STP

Rapid-PVST+ prevents Layer 2 loops while maintaining redundant paths.

CORE-SW1 is configured as the preferred STP root for the production VLANs.

CORE-SW2 provides the secondary path.

🛜 WAN Connectivity

HQ and Branch are connected through:

R1-HQ
│
│ 10.0.0.0/30
│
R2-BRANCH

The branch network:

192.168.100.0/24

is advertised through OSPF.

🌍 Internet Connectivity

R1-HQ connects the enterprise network to a simulated ISP.

R1-HQ
203.0.113.2
│
│
ISP
203.0.113.1
│
│
8.8.8.8

8.8.8.8 represents a simulated Internet destination using the ISP's Loopback interface.

R1-HQ performs NAT/PAT so internal private addresses can access the simulated Internet.

Example:

192.168.10.101
│
│ NAT/PAT
▼
203.0.113.2
│
▼
8.8.8.8
🖥️ Network Services
DHCP

A centralized DHCP server provides addresses to:

Sales
IT
Guest
Branch

DHCP relay is configured on the appropriate Layer 3 interfaces.

DNS

Internal DNS provides:

www.enterprise.local

which resolves to:

192.168.30.20
Web Server

The internal web server is:

192.168.30.20

HTTP is enabled and accessible using:

http://www.enterprise.local
🧪 Verification

The following functionality has been successfully tested:

VLAN configuration
802.1Q trunking
Inter-VLAN routing
Rapid-PVST+
LACP EtherChannel
HSRP
OSPF
DHCP
DHCP relay
DNS resolution
Internal HTTP server
HQ ↔ Branch connectivity
NAT/PAT
Simulated Internet connectivity

Verification commands and screenshots will be documented in the evidence/ directory.

🔧 Troubleshooting

This project involved troubleshooting real configuration problems encountered during implementation.

Issues included:

DHCP failure caused by incorrect trunk configuration
Native VLAN mismatches
Incorrect access-switch trunk configuration
Branch DHCP failure
DNS resolution failure
Temporary Internet connectivity failure during redundancy testing

The troubleshooting process followed a structured Layer 1 → Layer 2 → Layer 3 → Services approach.

Detailed incident reports will be available under:

/troubleshooting/
📸 Evidence

The repository will contain verification screenshots demonstrating:

VLANs
Trunks
STP
EtherChannel
HSRP
OSPF
Routing
DHCP
DNS
HTTP
NAT
Branch connectivity
Failover testing

All evidence is based on actual Packet Tracer configuration and verification.

📂 Repository Structure
enterprise-hq-branch-network/
│
├── README.md
│
├── topology/
│ ├── network-topology.png
│ └── topology-description.md
│
├── documentation/
│ ├── architecture.md
│ ├── ip-addressing.md
│ ├── vlan-design.md
│ ├── routing-design.md
│ ├── redundancy.md
│ └── troubleshooting.md
│
├── configuration/
│ ├── R1-HQ.txt
│ ├── R2-BRANCH.txt
│ ├── ISP.txt
│ ├── CORE-SW1.txt
│ ├── CORE-SW2.txt
│ ├── ACCESS-SW1.txt
│ ├── ACCESS-SW2.txt
│ └── BRANCH-SW1.txt
│
├── evidence/
│
└── troubleshooting/
🎯 Skills Demonstrated

Networking

Cisco IOS
Cisco Packet Tracer
VLANs
IPv4 subnetting
Layer 2 switching
Layer 3 switching
Inter-VLAN routing
OSPF
HSRP
STP
LACP EtherChannel
DHCP
DHCP Relay
DNS
NAT/PAT
WAN routing

Troubleshooting

VLAN troubleshooting
Trunk troubleshooting
STP analysis
EtherChannel verification
OSPF troubleshooting
DHCP troubleshooting
DNS troubleshooting
NAT verification
Connectivity testing
Redundancy testing
📈 Project Status

Core network implementation: Completed

Evidence collection: In Progress

Final failure/failover testing: Pending

Documentation: In Progress

👤 Author

Umesh Mihiranga

Cisco Enterprise Networking Project
Cisco Packet Tracer
