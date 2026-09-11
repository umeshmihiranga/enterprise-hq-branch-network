# Enterprise HQ–Branch Network

A redundant Cisco enterprise network designed and implemented using **Cisco Packet Tracer**, simulating a real-world headquarters, branch office, internal network services, and Internet-connected enterprise environment.

The project demonstrates practical **Layer 2 and Layer 3 networking, dynamic routing, redundancy, network services, and structured troubleshooting**.

---

## 📌 Project Overview

This project simulates a multi-site enterprise network consisting of:

- Headquarters (HQ)
- Branch office
- Redundant Layer 3 core switches
- Access-layer switching
- VLAN segmentation
- Inter-VLAN routing
- OSPF dynamic routing
- HSRP first-hop redundancy
- LACP EtherChannel
- Rapid-PVST+ Spanning Tree
- Centralized DHCP
- DHCP Relay
- Internal DNS
- Internal web server
- NAT/PAT Internet connectivity
- HQ-to-Branch connectivity
- Network failure and troubleshooting scenarios

The network was designed with a focus on **availability, scalability, redundancy, and operational troubleshooting**.

---

## 🏗️ Network Topology

![Enterprise HQ–Branch Network](topology/network-topology.png)

### High-Level Architecture

```text
                         ┌───────────────┐
                         │      ISP      │
                         │    8.8.8.8    │
                         └───────┬───────┘
                                 │
                          203.0.113.0/30
                                 │
                         ┌───────┴───────┐
                         │     R1-HQ     │
                         │  HQ Edge/WAN  │
                         │   NAT / OSPF  │
                         └───┬───────┬───┘
                             │       │
                           OSPF     OSPF
                             │       │
                 ┌───────────┴───────┴───────────┐
                 │                               │
          ┌──────┴──────┐                 ┌──────┴──────┐
          │  CORE-SW1   │════ LACP ══════│  CORE-SW2   │
          │ L3 / HSRP   │                 │ L3 / HSRP   │
          │  STP Root   │                 │  Secondary  │
          └──────┬──────┘                 └──────┬──────┘
                 │                               │
          ┌──────┴──────┐                 ┌──────┴──────┐
          │ ACCESS-SW1  │                 │ ACCESS-SW2  │
          └──────┬──────┘                 └──────┬──────┘
                 │                               │
             HQ Users                       Guests / Servers


                    HQ ↕ OSPF WAN ↕ Branch

                         ┌──────────────┐
                         │  R2-BRANCH   │
                         │    OSPF      │
                         └──────┬───────┘
                                │
                         ┌──────┴───────┐
                         │ BRANCH-SW1   │
                         └──────┬───────┘
                                │
                          Branch Users
```

---

## 🎯 Project Objectives

| Objective                   | Implementation                         |
| --------------------------- | -------------------------------------- |
| VLAN segmentation           | 802.1Q VLANs                           |
| Inter-VLAN communication    | Layer 3 SVIs                           |
| Default gateway redundancy  | HSRP                                   |
| Layer 2 loop prevention     | Rapid-PVST+                            |
| Link redundancy             | LACP EtherChannel                      |
| Dynamic routing             | OSPF                                   |
| Branch connectivity         | OSPF over WAN                          |
| Automatic IP addressing     | DHCP                                   |
| DHCP across VLANs           | DHCP Relay                             |
| Name resolution             | DNS                                    |
| Internal application access | HTTP                                   |
| Internet connectivity       | NAT/PAT                                |
| Network resilience          | Redundant core and gateways            |
| Troubleshooting             | Layer 1 → Layer 2 → Layer 3 → Services |

---

# 🖥️ Network Devices

## Routers

| Device        | Role                                 |
| ------------- | ------------------------------------ |
| **R1-HQ**     | HQ edge router, WAN gateway, NAT/PAT |
| **R2-BRANCH** | Branch router and WAN routing        |
| **ISP**       | Simulated Internet provider          |

## Multilayer Switches

| Device       | Role                                       |
| ------------ | ------------------------------------------ |
| **CORE-SW1** | Primary Layer 3 core/distribution switch   |
| **CORE-SW2** | Secondary Layer 3 core/distribution switch |

## Access Switches

| Device         | Role                    |
| -------------- | ----------------------- |
| **ACCESS-SW1** | HQ user access          |
| **ACCESS-SW2** | Guest and server access |
| **BRANCH-SW1** | Branch user access      |

## End Devices

| Device       | Purpose               |
| ------------ | --------------------- |
| SALES-PC1    | Sales workstation     |
| SALES-PC2    | Sales workstation     |
| IT-PC1       | IT workstation        |
| IT-PC2       | IT workstation        |
| GUEST-PC1    | Guest workstation     |
| BRANCH-PC1   | Branch workstation    |
| SRV-DHCP-DNS | DHCP and DNS services |
| SRV-WEB      | Internal web server   |

---

# 🌐 VLAN & IP Addressing Design

| VLAN | Name             | Network            | Purpose                |
| ---: | ---------------- | ------------------ | ---------------------- |
|   10 | SALES            | `192.168.10.0/24`  | Sales users            |
|   20 | IT               | `192.168.20.0/24`  | IT users               |
|   30 | SERVERS          | `192.168.30.0/24`  | Network services       |
|   40 | GUEST            | `192.168.40.0/24`  | Guest users            |
|   50 | VOICE            | `192.168.50.0/24`  | Reserved voice network |
|   99 | MANAGEMENT       | `192.168.99.0/24`  | Network management     |
|  100 | BRANCH-USERS     | `192.168.100.0/24` | Branch users           |
|  999 | NATIVE-BLACKHOLE | —                  | Unused native VLAN     |

The VLAN structure separates users, servers, guests, management traffic, and branch users into independent broadcast domains.

---

# 🔀 Layer 2 Switching

The switching infrastructure implements several enterprise Layer 2 technologies:

- VLAN segmentation
- 802.1Q trunking
- Native VLAN configuration
- Allowed VLAN lists
- Rapid-PVST+
- STP root bridge selection
- PortFast
- BPDU Guard
- LACP EtherChannel

---

# 🔗 LACP EtherChannel

The two core switches are connected using an LACP EtherChannel.

```text
CORE-SW1                         CORE-SW2

Fa0/23 ────────────────┐   ┌────────────── Fa0/23
                       │   │
                       ├───┤
                   Port-Channel1
                       │   │
Fa0/24 ────────────────┘   └────────────── Fa0/24
```

### EtherChannel Members

**CORE-SW1**

```text
Fa0/23
Fa0/24
```

**CORE-SW2**

```text
Fa0/23
Fa0/24
```

LACP provides:

- Link redundancy
- Increased aggregate bandwidth
- Logical link abstraction
- Reduced STP complexity
- Continued connectivity if one physical link fails

---

# 🌳 Spanning Tree

The switching environment uses **Rapid-PVST+** to prevent Layer 2 loops while maintaining redundant paths.

### STP Design

**CORE-SW1**

- Preferred STP root
- Primary Layer 3 core
- HSRP Active for production VLANs

**CORE-SW2**

- Secondary STP path
- Secondary Layer 3 core
- HSRP Standby

Additional Layer 2 protections include:

- PortFast
- BPDU Guard
- Root bridge selection
- Trunk verification

---

# 🚦 Layer 3 Routing

The multilayer core switches perform **Inter-VLAN Routing using Switched Virtual Interfaces (SVIs)**.

Each production VLAN has an SVI on both core switches.

### Example — VLAN 10

```text
CORE-SW1
192.168.10.2

CORE-SW2
192.168.10.3

HSRP Virtual Gateway
192.168.10.1
```

Hosts use the HSRP virtual address:

```text
192.168.10.1
```

as their default gateway.

This allows the network to maintain gateway availability even if one core switch becomes unavailable.

---

# 🛡️ HSRP High Availability

HSRP provides redundant default gateways for production VLANs.

### Example

```text
                HSRP Virtual Gateway
                    192.168.10.1
                          │
                 ┌────────┴────────┐
                 │                 │
          CORE-SW1             CORE-SW2
          192.168.10.2         192.168.10.3
          Priority: 110        Priority: 100
          ACTIVE               STANDBY
          Preempt Enabled      Preempt Enabled
```

### HSRP Roles

| Device   | Priority | Role    |
| -------- | -------: | ------- |
| CORE-SW1 |      110 | Active  |
| CORE-SW2 |      100 | Standby |

If CORE-SW1 becomes unavailable, CORE-SW2 can assume the virtual gateway role.

This removes the default gateway as a single point of failure.

---

# 🔄 OSPF Dynamic Routing

The routed infrastructure uses:

```text
OSPF Process 10
Area 0
```

OSPF provides dynamic route exchange between the HQ, core switches, and branch router.

### OSPF Router IDs

| Device    | Router ID |
| --------- | --------- |
| R1-HQ     | `1.1.1.1` |
| CORE-SW1  | `2.2.2.2` |
| CORE-SW2  | `3.3.3.3` |
| R2-BRANCH | `4.4.4.4` |

### OSPF Architecture

```text
                  R1-HQ
                 1.1.1.1
                    │
                  OSPF
                    │
          ┌─────────┴─────────┐
          │                   │
      CORE-SW1             CORE-SW2
       2.2.2.2              3.3.3.3
          │                   │
          └─────────┬─────────┘
                    │
                  OSPF
                    │
                R2-BRANCH
                 4.4.4.4
```

The branch network:

```text
192.168.100.0/24
```

is advertised through OSPF.

---

# 🛜 WAN Connectivity

The HQ and branch networks are connected through a routed WAN link.

```text
R1-HQ
  │
  │ 10.0.0.0/30
  │
R2-BRANCH
```

The WAN connection allows the branch network to dynamically exchange routes with the HQ infrastructure through OSPF.

---

# 🌍 Internet Connectivity

R1-HQ connects the enterprise network to a simulated ISP.

```text
Enterprise Network
        │
        │
     R1-HQ
        │
        │ 203.0.113.0/30
        │
       ISP
        │
        │
     8.8.8.8
```

The ISP uses a loopback address representing a simulated Internet destination:

```text
8.8.8.8
```

---

# 🔐 NAT / PAT

R1-HQ performs NAT/PAT for internal private IP addresses.

### Example

```text
Internal Host
192.168.10.101
       │
       │ NAT/PAT
       ▼
R1-HQ
203.0.113.2
       │
       ▼
      ISP
       │
       ▼
    8.8.8.8
```

NAT/PAT allows internal private hosts to access the simulated Internet using the HQ router's public-facing address.

---

# 🖥️ Network Services

## DHCP

A centralized DHCP server provides IP addresses to multiple VLANs.

DHCP services are provided for:

- Sales
- IT
- Guest
- Branch users

Because DHCP broadcasts normally remain within a local broadcast domain, **DHCP Relay** is configured on the appropriate Layer 3 interfaces.

This allows clients in different VLANs to obtain addresses from the centralized DHCP server.

---

## DNS

The internal DNS server provides name resolution for:

```text
www.enterprise.local
```

which resolves to:

```text
192.168.30.20
```

---

## 🌐 Internal Web Server

The internal web server is hosted at:

```text
192.168.30.20
```

HTTP is enabled and can be accessed using:

```text
http://www.enterprise.local
```

This demonstrates the integration of:

```text
DNS → IP Resolution → Routing → HTTP
```

within the enterprise network.

---

# 🧪 Verification & Testing

The network was verified using Cisco IOS commands and end-to-end connectivity tests.

## VLAN Verification

```text
show vlan brief
```

Used to verify:

- VLAN existence
- VLAN assignment
- Access-port membership

---

## Trunk Verification

```text
show interfaces trunk
```

Used to verify:

- Trunk state
- Native VLAN
- Allowed VLANs
- VLAN propagation

---

## STP Verification

```text
show spanning-tree
```

Used to verify:

- Root bridge
- Port roles
- Forwarding/blocking states
- STP topology

---

## EtherChannel Verification

```text
show etherchannel summary
```

Used to verify:

- Port-Channel status
- LACP operation
- Member interfaces
- Bundled links

---

## HSRP Verification

```text
show standby
```

Used to verify:

- Active router
- Standby router
- Virtual IP
- HSRP priority
- Preemption

---

## OSPF Verification

```text
show ip ospf neighbor
show ip ospf
```

Used to verify:

- OSPF neighbor relationships
- OSPF process
- Router IDs
- Area configuration

---

## Routing Verification

```text
show ip route
```

Used to verify:

- Connected routes
- OSPF routes
- Default routes
- Branch network reachability

---

## DHCP Verification

```text
show ip dhcp binding
```

Used to verify:

- DHCP leases
- Client addresses
- Address allocation

---

## Connectivity Testing

```text
ping
traceroute
```

Used for end-to-end validation between:

- Hosts and gateways
- VLANs
- HQ and Branch
- Internal services
- Enterprise network and simulated Internet

---

# 🔧 Troubleshooting

A major part of this project was troubleshooting configuration failures encountered during implementation.

The project does not only demonstrate successful configuration; it also documents the process of identifying and resolving network problems.

### Issues Investigated

- DHCP failure caused by incorrect trunk configuration
- Native VLAN mismatch
- Incorrect access-switch trunk configuration
- Branch DHCP failure
- DNS resolution failure
- Temporary Internet connectivity failure during redundancy testing

---

# 🧭 Troubleshooting Methodology

Troubleshooting followed a structured network-layer approach.

```text
┌─────────────────────┐
│       Layer 1       │
│ Physical / Interfaces│
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│       Layer 2       │
│ VLAN / Trunk / STP  │
│ EtherChannel        │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│       Layer 3       │
│ SVI / Routing /     │
│ OSPF / HSRP         │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│      Services       │
│ DHCP / DNS / NAT    │
│ HTTP                │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ End-to-End Testing  │
└─────────────────────┘
```

Detailed troubleshooting reports are maintained under:

```text
/troubleshooting/
```

---

# 📸 Evidence & Verification

Verification screenshots are maintained under:

```text
/evidence/
```

The evidence directory documents the actual implementation and verification of:

- VLANs
- Trunks
- STP
- EtherChannel
- HSRP
- OSPF
- Routing
- DHCP
- DNS
- HTTP
- NAT/PAT
- HQ-to-Branch connectivity
- Failover testing

# 📂 Repository Structure

```text
enterprise-hq-branch-network/
│
├── README.md
│
├── documentation/
│   ├── architecture.md
│   ├── ip-addressing.md
│   ├── vlan-design.md
│   ├── routing-design.md
│   ├── redundancy.md
│   └── troubleshooting.md
│
├── configuration/
│   ├── R1-HQ.txt
│   ├── R2-BRANCH.txt
│   ├── ISP.txt
│   ├── CORE-SW1.txt
│   ├── CORE-SW2.txt
│   ├── ACCESS-SW1.txt
│   ├── ACCESS-SW2.txt
│   └── BRANCH-SW1.txt
│
├── evidence/
│   ├── vlan/
│   ├── trunks/
│   ├── stp/
│   ├── etherchannel/
│   ├── hsrp/
│   ├── ospf/
│   ├── routing/
│   ├── dhcp/
│   ├── dns/
│   ├── http/
│   ├── nat/
│   └── connectivity/
│
└── Enterprise Multi-Site Network Infrastructure & Routing Lab.pkt
```

---

# 🧠 Skills Demonstrated

## Networking

- Cisco IOS
- Cisco Packet Tracer
- IPv4 addressing
- IPv4 subnetting
- VLANs
- 802.1Q trunking
- Layer 2 switching
- Layer 3 switching
- Inter-VLAN routing
- OSPF
- HSRP
- Rapid-PVST+
- LACP
- EtherChannel
- DHCP
- DHCP Relay
- DNS
- NAT/PAT
- WAN routing

## Troubleshooting

- VLAN troubleshooting
- Trunk troubleshooting
- Native VLAN mismatch analysis
- STP analysis
- EtherChannel verification
- OSPF troubleshooting
- HSRP verification
- DHCP troubleshooting
- DNS troubleshooting
- NAT verification
- End-to-end connectivity testing
- Redundancy testing

# 🏆 Project Outcome

This project demonstrates the design and implementation of a complete enterprise network rather than isolated networking configurations.

Multiple technologies were integrated into a single working infrastructure:

```text
VLANs
  │
  ▼
802.1Q Trunking
  │
  ▼
Rapid-PVST+
  │
  ▼
EtherChannel
  │
  ▼
Layer 3 SVIs
  │
  ▼
HSRP
  │
  ▼
OSPF
  │
  ▼
DHCP / DNS
  │
  ▼
NAT/PAT
  │
  ▼
Simulated Internet
```

The project also emphasizes **verification and troubleshooting**, demonstrating the ability to investigate configuration failures systematically across Layer 1, Layer 2, Layer 3, and network services.

---

# 🎓 Key Takeaways

Through this project, the following practical networking concepts were implemented and tested:

- Designing a hierarchical enterprise network
- Separating users and services using VLANs
- Implementing Layer 3 switching
- Providing redundant default gateways
- Building resilient switch-to-switch links
- Implementing dynamic routing
- Connecting HQ and Branch environments
- Providing centralized network services
- Translating private addresses for Internet access
- Troubleshooting real configuration problems
- Verifying network behavior using Cisco IOS commands

---

# 👤 Author

**Umesh Mihiranga**

Cisco Enterprise Networking Project  
Cisco Packet Tracer

---

## ⭐ Technologies

`Cisco IOS` `Cisco Packet Tracer` `VLAN` `802.1Q` `STP` `Rapid-PVST+` `LACP` `EtherChannel` `HSRP` `OSPF` `DHCP` `DHCP Relay` `DNS` `NAT` `PAT` `IPv4` `Layer 2` `Layer 3`
