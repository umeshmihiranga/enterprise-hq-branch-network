# VLAN Design

## Overview

The network uses VLAN segmentation to logically separate different types of users, infrastructure, and services.

VLANs reduce broadcast domains, improve network organization, and provide a scalable foundation for inter-VLAN routing and access control.

The HQ network uses VLANs 10–99, while the branch uses VLAN 100.

---

## VLAN Assignment

| VLAN ID | VLAN Name        | Purpose                                |
| ------: | ---------------- | -------------------------------------- |
|      10 | SALES            | Sales department users                 |
|      20 | IT               | IT department users                    |
|      30 | SERVERS          | Infrastructure and application servers |
|      40 | GUEST            | Guest user devices                     |
|      50 | VOICE            | Voice/VoIP devices                     |
|      99 | MANAGEMENT       | Network device management              |
|     100 | BRANCH-USERS     | Branch user devices                    |
|     999 | NATIVE-BLACKHOLE | Unused native VLAN                     |

> For IP subnets, gateways, and HSRP virtual IPs corresponding to these VLANs, see [ip-addressing.md](ip-addressing.md).

---

## VLAN Purposes

### VLAN 10 — SALES

Used for Sales department endpoints.
- DHCP enabled
- Example devices: SALES-PC1, SALES-PC2

### VLAN 20 — IT

Used for IT department endpoints.
- DHCP enabled
- Example devices: IT-PC1, IT-PC2

### VLAN 30 — SERVERS

Dedicated server network for infrastructure and application services.
- Static addressing
- Example devices:
  - SRV-DHCP-DNS
  - SRV-WEB

Services include: DHCP, DNS, HTTP.

### VLAN 40 — GUEST

Used for guest devices. The VLAN provides logical separation from internal departmental networks.
- DHCP enabled
- Example device: GUEST-PC1

### VLAN 50 — VOICE

Reserved for future VoIP deployment. No voice endpoints are currently deployed, but the VLAN is included to demonstrate an enterprise-ready network design.

### VLAN 99 — MANAGEMENT

Dedicated management VLAN for network infrastructure. This VLAN is reserved for management interfaces and future administrative access.

### VLAN 100 — BRANCH-USERS

Used for users at the branch site. The branch VLAN is routed through R2-BRANCH and advertised through OSPF.
- DHCP enabled
- Example device: BRANCH-PC1

---

## VLAN 999 — Native Blackhole VLAN

VLAN 999 is configured as the native VLAN on trunk links.

It is intentionally unused for normal endpoint traffic.

Purpose:
- Prevent normal user traffic from using the default native VLAN.
- Provide a dedicated native VLAN for trunking.
- Reduce the risk of accidental VLAN leakage.
- Demonstrate enterprise trunk configuration practices.

No end devices are assigned to VLAN 999. Unused access ports are also assigned to VLAN 999 and administratively shut down.

---

## Access Port Design

End-user ports are configured as access ports and assigned to the appropriate VLAN.

Access ports connected to end devices use:
- **PortFast**: Allows endpoints to transition to forwarding quickly.
- **BPDU Guard**: Protects access ports against unexpected BPDU transmission.

---

## Trunk Design

Trunks are used to carry multiple VLANs between network devices.

HQ trunk links use:
- Native VLAN: 999
- Allowed VLANs: 10,20,30,40,50,99

HQ trunk connections include:
- CORE-SW1 ↔ CORE-SW2
- CORE-SW1 ↔ ACCESS-SW1
- CORE-SW2 ↔ ACCESS-SW1
- CORE-SW1 ↔ ACCESS-SW2
- CORE-SW2 ↔ ACCESS-SW2

The branch uses a dedicated trunk between:
- R2-BRANCH ↔ BRANCH-SW1

The branch trunk carries:
- Native VLAN: 999
- Allowed VLAN: 100

---

## VLAN Propagation

The network does not rely on VTP for VLAN distribution. VLANs are explicitly configured on the switches.

This provides:
- Predictable VLAN configuration
- Reduced dependency on a centralized VTP server
- Easier troubleshooting
- Better visibility of the intended VLAN configuration

> For details on inter-VLAN routing and DHCP relay, see [routing-design.md](routing-design.md).
> For details on STP and HSRP gateway redundancy for these VLANs, see [redundancy.md](redundancy.md).
> For traffic flow examples, see [architecture.md § Network Services Flow](architecture.md#15-network-services-flow).

---

## Design Principles

The VLAN architecture follows these principles:

- **Departmental segmentation**: Users are separated based on organizational function.
- **Server isolation**: Infrastructure services are placed in a dedicated server VLAN.
- **Guest separation**: Guest devices use their own VLAN rather than sharing internal user networks.
- **Dedicated management network**: Network management traffic has a separate VLAN.
- **Future voice support**: A dedicated voice VLAN is reserved for future VoIP deployment.
- **Branch segmentation**: Branch users use a separate VLAN and subnet.
- **Controlled trunking**: Trunks explicitly allow only required VLANs.
- **Dedicated native VLAN**: VLAN 999 is used as the native VLAN and contains no normal endpoints.
- **Layer 3 gateway redundancy**: HSRP provides resilient default gateways for the HQ VLANs.

---

## Verification Commands

Useful commands for validating the VLAN design include:

- Display VLANs: `show vlan brief`
- Display trunk status: `show interfaces trunk`
- Display SVI status: `show ip interface brief`
- Display HSRP status: `show standby brief`
- Display spanning-tree status: `show spanning-tree vlan 10`
- Test inter-VLAN connectivity: `ping 192.168.30.20`

**Verify DHCP operation:** Check the client IP configuration and confirm that the correct VLAN gateway and DNS server are received.
