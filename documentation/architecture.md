# Network Architecture

## 1. Overview

The **Enterprise HQ–Branch Network** is a Cisco Packet Tracer-based enterprise network designed to demonstrate practical Layer 2, Layer 3, WAN, routing, redundancy, and network-service concepts.

The architecture connects:

- Headquarters users
- Internal servers
- Guest users
- A branch office
- Redundant multilayer switches
- An HQ edge router
- A simulated ISP/Internet

The design focuses on **scalability, redundancy, logical segmentation, dynamic routing, and structured troubleshooting**.

---

## 2. High-Level Architecture

```text
                         SIMULATED INTERNET
                                |
                              ISP
                         203.0.113.1
                                |
                         203.0.113.2
                              R1-HQ
                         /       |       \
                        /        |        \
                       /         |         \
                    OSPF        OSPF       OSPF
                     /           |           \
                    /            |            \
             CORE-SW1 ========= CORE-SW2
               ACTIVE             STANDBY
                  \                 /
                   \               /
                    \             /
                     ACCESS LAYER
                    /             \
             ACCESS-SW1         ACCESS-SW2
              /   |   \          /   |   \
           Sales  IT PCs       Guest  Servers


                              R1-HQ
                                |
                           OSPF WAN
                                |
                           R2-BRANCH
                                |
                          BRANCH-SW1
                                |
                          BRANCH-PC1
```

---

## 3. Device Roles

### R1-HQ

R1-HQ is the headquarters edge router.

Responsibilities:

- ISP connectivity
- HQ-to-Branch WAN connectivity
- NAT/PAT
- Default routing toward the ISP
- OSPF routing

### R2-BRANCH

R2-BRANCH provides Layer 3 connectivity for the branch.

Responsibilities:

- Branch default gateway
- Router-on-a-stick
- DHCP relay
- OSPF routing
- WAN connectivity to HQ

### CORE-SW1

CORE-SW1 is the primary multilayer core/distribution switch.

Responsibilities:

- Inter-VLAN routing
- HSRP Active gateway
- OSPF
- DHCP relay
- STP root bridge
- LACP EtherChannel

### CORE-SW2

CORE-SW2 provides redundancy for the primary core switch.

Responsibilities:

- Inter-VLAN routing
- HSRP Standby gateway
- OSPF
- STP secondary role
- LACP EtherChannel

### ACCESS-SW1

ACCESS-SW1 provides connectivity for the main user groups.

Connected devices include:

- SALES-PC1
- SALES-PC2
- IT-PC1
- IT-PC2

### ACCESS-SW2

ACCESS-SW2 provides connectivity for:

- GUEST-PC1
- SRV-DHCP-DNS
- SRV-WEB

### BRANCH-SW1

BRANCH-SW1 provides Layer 2 connectivity for the branch user network.

Connected device:

- BRANCH-PC1

### ISP

The ISP router represents an external Internet provider.

Its Loopback interface `8.8.8.8/32` is used as a simulated Internet destination.

---

## 4. Layer 2 Architecture

The Layer 2 infrastructure uses:

- VLAN segmentation
- 802.1Q trunking
- Rapid-PVST+
- PortFast
- BPDU Guard
- LACP EtherChannel
- Native VLAN 999
- Allowed VLAN lists

User devices are assigned to VLANs according to their function.

```text
SALES-PC1
    |
VLAN 10
    |
ACCESS-SW1
    |
802.1Q Trunk
    |
CORE-SW1 / CORE-SW2
```

> For the complete VLAN assignment table and per-VLAN details, see [vlan-design.md](vlan-design.md).
> For trunk and access port configuration specifics, see [vlan-design.md § Access Port Design / Trunk Design](vlan-design.md#access-port-design).
> For EtherChannel and STP details, see [redundancy.md](redundancy.md).

---

## 5. VLAN Segmentation

The network is divided into logical broadcast domains:

| VLAN | Name |
|---:|---|
| 10 | SALES |
| 20 | IT |
| 30 | SERVERS |
| 40 | GUEST |
| 50 | VOICE |
| 99 | MANAGEMENT |
| 999 | NATIVE-BLACKHOLE |

The branch uses:

| VLAN | Name |
|---:|---|
| 100 | BRANCH-USERS |

VLAN 999 is used as the native VLAN on trunk links and is not assigned to normal end devices.

> For subnets, gateways, and DHCP pool details, see [ip-addressing.md](ip-addressing.md).
> For VLAN purposes and design rationale, see [vlan-design.md](vlan-design.md).

---

## 6. Layer 3 Architecture

CORE-SW1 and CORE-SW2 operate as multilayer switches and provide Layer 3 routing using SVIs.

Each production VLAN has a Layer 3 gateway.

HSRP provides a shared virtual gateway so that end devices use a single virtual IP as their default gateway. If the active switch fails, the standby takes over transparently.

> For HSRP addresses and SVI IP assignments, see [ip-addressing.md § HSRP Addressing](ip-addressing.md#3-hsrp-addressing).
> For HSRP roles, priorities, and failure scenarios, see [redundancy.md § HSRP Gateway Redundancy](redundancy.md#1-hsrp-gateway-redundancy).

---

## 7. Routing Architecture

OSPF is used as the internal dynamic routing protocol.

The enterprise routing domain uses OSPF Area 0.

```text
                         R1-HQ
                        /     \
                     OSPF     OSPF
                      /         \
                CORE-SW1      CORE-SW2
                      \         /
                       \       /
                         OSPF
                           |
                      R2-BRANCH
```

OSPF provides dynamic route exchange between the HQ core, HQ router, and branch router.

> For the full OSPF design including router IDs, passive interfaces, and route advertisement, see [routing-design.md](routing-design.md).

---

## 8. Headquarters Architecture

The headquarters contains:

- Sales users
- IT users
- Guest users
- Internal servers
- Redundant multilayer core switches
- Access switches
- Edge router

High-level HQ design:

```text
                       R1-HQ
                      /     \
                     /       \
               CORE-SW1     CORE-SW2
                  /  \         /  \
                 /    \       /    \
        ACCESS-SW1      ACCESS-SW2
          /   |   \        /    \
       Sales IT PCs     Guest  Servers
```

---

## 9. Server Architecture

The server network is located in VLAN 30.

### SRV-DHCP-DNS

- IP Address: `192.168.30.10`
- VLAN: 30

Services:

- DHCP
- DNS

The server provides DHCP scopes for:

- SALES
- IT
- GUEST
- BRANCH

It also provides internal DNS resolution.

### SRV-WEB

- IP Address: `192.168.30.20`
- VLAN: 30

Service:

- HTTP

The internal DNS record:

```text
www.enterprise.local → 192.168.30.20
```

---

## 10. DHCP Architecture

DHCP is centralized on SRV-DHCP-DNS.

The Layer 3 gateways use DHCP relay to forward client requests to `192.168.30.10`.

```text
Client
   |
VLAN
   |
SVI / Gateway
   |
DHCP Relay
   |
192.168.30.10
   |
DHCP Server
```

This allows clients in different VLANs and the remote branch to obtain addresses from a centralized DHCP server.

> For DHCP pool details, see [ip-addressing.md § DHCP Address Pools](ip-addressing.md#7-dhcp-address-pools).
> For DHCP relay routing configuration, see [routing-design.md § DHCP Routing](routing-design.md#dhcp-routing).

---

## 11. Branch Architecture

The branch uses VLAN 100:

- Network: `192.168.100.0/24`
- Gateway: `192.168.100.1`

Topology:

```text
BRANCH-PC1
     |
     |
BRANCH-SW1
     |
802.1Q Trunk
     |
R2-BRANCH
     |
     |
OSPF WAN
     |
R1-HQ
```

R2-BRANCH uses a router subinterface for VLAN 100:

```
GigabitEthernet0/0.100
```

The branch DHCP requests are relayed to the centralized DHCP server at HQ.

> For the router-on-a-stick configuration, see [routing-design.md § Branch Router-on-a-Stick](routing-design.md#branch-router-on-a-stick).

---

## 12. WAN Architecture

The HQ and Branch are connected using a routed WAN link.

```text
R1-HQ
10.0.0.1/30
    |
    |
10.0.0.2/30
R2-BRANCH
```

OSPF dynamically advertises the branch network across this link.

The branch network `192.168.100.0/24` is therefore reachable from the HQ network.

> For all WAN and transit IP assignments, see [ip-addressing.md § WAN and Transit Addressing](ip-addressing.md#4-wan-and-transit-addressing).

---

## 13. Internet Architecture

R1-HQ connects the enterprise network to the simulated ISP.

```text
Internal Networks
       |
     R1-HQ
203.0.113.2
       |
       |
203.0.113.1
       |
      ISP
       |
       |
    8.8.8.8
```

The ISP uses `8.8.8.8/32` as a simulated external Internet destination.

R1-HQ performs NAT/PAT for internal private addresses.

> For the NAT/PAT configuration and traffic flow, see [routing-design.md § NAT/PAT](routing-design.md#natpat).

---

## 14. Redundancy Architecture

The network uses multiple redundancy mechanisms:

- **HSRP** — Redundant default gateways for all HQ VLANs
- **LACP EtherChannel** — Core interconnect link aggregation
- **Rapid-PVST+** — Layer 2 loop prevention with redundant-path recovery

> For full redundancy details including failure scenarios and testing procedures, see [redundancy.md](redundancy.md).

---

## 15. Network Services Flow

A typical user accessing the internal web server follows this path:

```text
SALES-PC1
    |
VLAN 10
    |
ACCESS-SW1
    |
802.1Q Trunk
    |
CORE-SW1 / CORE-SW2
    |
VLAN 30
    |
SRV-WEB
192.168.30.20
    |
HTTP Response
    |
SALES-PC1
```

DNS resolution occurs through:

```text
SALES-PC1
    |
    |
SRV-DHCP-DNS
192.168.30.10
    |
    |
www.enterprise.local
    |
    |
192.168.30.20
```

---

## 16. Management and Native VLAN Design

VLAN 99 is reserved for network management:

```
192.168.99.0/24
```

VLAN 999 is used as the native VLAN on trunk links.

Production VLANs are explicitly defined in trunk allowed lists:

```
10,20,30,40,50,99
```

> For the native VLAN design rationale, see [vlan-design.md § VLAN 999](vlan-design.md#vlan-999--native-blackhole-vlan).
