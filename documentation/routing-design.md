# Routing Design

## Overview

The network uses a combination of Layer 3 switching, OSPF dynamic routing, static routing, and NAT/PAT to provide connectivity between the HQ, branch, server, and simulated Internet networks.

The routing architecture is designed to provide:

- Inter-VLAN routing
- Dynamic HQ-to-Branch routing
- Redundant core connectivity
- Centralized default-route propagation
- Internet connectivity
- Scalable routing between network segments

---

## Routing Architecture

The routing hierarchy is:

```text
                    Simulated Internet
                         8.8.8.8
                            |
                            |
                          ISP
                     203.0.113.1
                            |
                            |
                     203.0.113.2
                          R1-HQ
                     /      |      \
                    /       |       \
             10.0.1.1     10.0.1.5   10.0.0.1
                /             \          \
               /               \          \
        CORE-SW1             CORE-SW2     R2-BRANCH
        10.0.1.2             10.0.1.6      10.0.0.2
          |                     |             |
          |                     |             |
       HQ VLANs              HQ VLANs     VLAN 100
```

---

## Layer 3 Core

CORE-SW1 and CORE-SW2 operate as multilayer switches.

Layer 3 routing is enabled using:

```
ip routing
```

The core switches provide:
- SVI gateways
- Inter-VLAN routing
- OSPF participation
- Redundant default gateways through HSRP
- Routed connections toward R1-HQ

### Switched Virtual Interfaces (SVIs)

Each major HQ VLAN has an SVI on both core switches. These interfaces provide Layer 3 connectivity for their respective VLANs.

> For the specific IP addresses assigned to the SVIs and HSRP virtual gateways, see [ip-addressing.md § HSRP Addressing](ip-addressing.md#3-hsrp-addressing).
> For HSRP roles and redundancy details, see [redundancy.md § HSRP Gateway Redundancy](redundancy.md#1-hsrp-gateway-redundancy).

### Routed Core-to-Router Links

The core switches use dedicated Layer 3 links to R1-HQ. These links are routed interfaces rather than Layer 2 switchports.

Example:
```
interface GigabitEthernet0/1
 no switchport
 ip address 10.0.1.2 255.255.255.252
```

This provides Layer 3 connectivity between the core and the edge router.

---

## OSPF Design

### OSPF Overview

Open Shortest Path First (OSPF) is used as the dynamic routing protocol.

The network uses:
- OSPF Process ID: 10
- Area: 0

All routed enterprise links participate in OSPF Area 0. OSPF provides automatic route exchange between R1-HQ, R2-BRANCH, CORE-SW1, and CORE-SW2.

### OSPF Router IDs

Explicit router IDs make OSPF neighbor identification easier during troubleshooting.

| Device | Router ID |
|---|---|
| R1-HQ | `1.1.1.1` |
| CORE-SW1 | `2.2.2.2` |
| CORE-SW2 | `3.3.3.3` |
| R2-BRANCH | `4.4.4.4` |

### OSPF Network Structure

The core switches have OSPF adjacency with R1-HQ. R1-HQ has an OSPF adjacency with R2-BRANCH across the WAN link.

### OSPF Passive Interfaces

User VLAN interfaces do not need OSPF neighbor relationships. Therefore, the core switches use `passive-interface default` and explicitly enable OSPF on the routed interface toward R1-HQ:

```
no passive-interface GigabitEthernet0/1
```

R2-BRANCH similarly uses the WAN serial interface for the OSPF adjacency. This reduces unnecessary OSPF neighbor formation on user-facing interfaces.

---

## HQ-to-Branch Routing

The HQ and branch networks communicate through R1-HQ and R2-BRANCH over the WAN link. 

OSPF advertises the branch network from R2-BRANCH. Therefore, HQ devices can dynamically learn the branch route.

Example route:
```text
O    192.168.100.0/24
```

### Branch Router-on-a-Stick

The branch uses router-on-a-stick routing. R2-BRANCH connects to BRANCH-SW1 through a trunk.

R2 uses a subinterface to provide the default gateway for branch users:
```
interface GigabitEthernet0/0.100
 encapsulation dot1Q 100
 ip address 192.168.100.1 255.255.255.0
```

---

## Internet Routing

### Default Route

R1-HQ connects to the ISP and has a static default route sending unknown destinations toward the ISP:

```
ip route 0.0.0.0 0.0.0.0 203.0.113.1
```

### OSPF Default Route Advertisement

R1-HQ redistributes its default route into OSPF using:
```
router ospf 10
 default-information originate
```

This allows the core switches to learn the default route dynamically without requiring a manually configured Internet default route.

---

## NAT/PAT

R1-HQ performs NAT overload (PAT) for internal networks. PAT allows multiple internal devices to share the public IP address of R1-HQ.

Example configuration:
```
access-list 1 permit 192.168.0.0 0.0.255.255

ip nat inside source list 1 interface GigabitEthernet0/0 overload
```

The interfaces are classified as:
- G0/0 → NAT outside
- G0/1 → NAT inside
- G0/2 → NAT inside
- S0/0/0 → NAT inside

### NAT Traffic Flow

```text
SALES-PC1
192.168.10.101
      |
      ↓
CORE-SW1
      |
      ↓
R1-HQ (192.168.10.101)
      |
      ↓
     PAT
      |
      ↓
203.0.113.2
      |
      ↓
     ISP
      |
      ↓
   8.8.8.8
```

The private source address is translated before traffic reaches the simulated ISP.

---

## DHCP Routing

DHCP is centralized on SRV-DHCP-DNS. Because DHCP broadcasts normally do not cross Layer 3 boundaries, DHCP relay is configured on the client VLAN interfaces.

Example:
```
ip helper-address 192.168.30.10
```

This is configured for VLAN 10, VLAN 20, VLAN 40, and the branch router's VLAN 100 subinterface. This allows a single DHCP server to provide addresses to multiple networks.

---

## DNS and Web Routing

DNS resolves `www.enterprise.local` to the web server IP. A client can therefore access the internal web server using `http://www.enterprise.local`.

Traffic is routed from the client's VLAN to VLAN 30 through the core Layer 3 gateway.

---

## Verification

### Routing Verification

The following commands are used to verify the routing architecture:
- Check Routing Table: `show ip route`
- Check OSPF Neighbors: `show ip ospf neighbor`
- Check OSPF Routes: `show ip route ospf`
- Check OSPF Configuration: `show ip protocols`
- Check HSRP: `show standby brief`
- Check NAT Translations: `show ip nat translations`
- Check NAT Statistics: `show ip nat statistics`

### Design Rationale

- **Layer 3 Switching**: Provides efficient local routing within the HQ.
- **OSPF**: Removes the need to manually configure every internal route and adapts to changes.
- **Static Default Route**: Simplifies upstream connectivity to the ISP.
- **NAT/PAT**: Secures private addresses behind a single public address.
- **DHCP Relay**: Allows centralized address management while maintaining separate Layer 3 networks.
