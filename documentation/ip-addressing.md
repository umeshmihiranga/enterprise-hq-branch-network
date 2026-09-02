# IP Addressing Plan

## 1. Overview

The network uses private IPv4 addressing for internal networks and dedicated point-to-point subnets for routed WAN and transit links.

The addressing scheme is organized by VLAN function to make the network easier to understand, troubleshoot, and scale.

---

## 2. VLAN Addressing

| VLAN | Name             | Network            | Subnet Mask     | HSRP Virtual Gateway |
| ---: | ---------------- | ------------------ | --------------- | -------------------- |
|   10 | SALES            | `192.168.10.0/24`  | `255.255.255.0` | `192.168.10.1`       |
|   20 | IT               | `192.168.20.0/24`  | `255.255.255.0` | `192.168.20.1`       |
|   30 | SERVERS          | `192.168.30.0/24`  | `255.255.255.0` | `192.168.30.1`       |
|   40 | GUEST            | `192.168.40.0/24`  | `255.255.255.0` | `192.168.40.1`       |
|   50 | VOICE            | `192.168.50.0/24`  | `255.255.255.0` | `192.168.50.1`       |
|   99 | MANAGEMENT       | `192.168.99.0/24`  | `255.255.255.0` | `192.168.99.1`       |
|  999 | NATIVE-BLACKHOLE | No host network    | —               | —                    |
|  100 | BRANCH-USERS     | `192.168.100.0/24` | `255.255.255.0` | `192.168.100.1`      |

---

## 3. HSRP Addressing

The main HQ VLANs use HSRP for default-gateway redundancy.

| VLAN | CORE-SW1       | CORE-SW2       | HSRP VIP       |
| ---: | -------------- | -------------- | -------------- |
|   10 | `192.168.10.2` | `192.168.10.3` | `192.168.10.1` |
|   20 | `192.168.20.2` | `192.168.20.3` | `192.168.20.1` |
|   30 | `192.168.30.2` | `192.168.30.3` | `192.168.30.1` |
|   40 | `192.168.40.2` | `192.168.40.3` | `192.168.40.1` |
|   50 | `192.168.50.2` | `192.168.50.3` | `192.168.50.1` |
|   99 | `192.168.99.2` | `192.168.99.3` | `192.168.99.1` |

> For HSRP roles, priorities, and failover behavior, see [redundancy.md § HSRP Gateway Redundancy](redundancy.md#1-hsrp-gateway-redundancy).

---

## 4. WAN and Transit Addressing

Point-to-point links use /30 networks.

| Connection     | Network          | Device    | IP Address    |
| -------------- | ---------------- | --------- | ------------- |
| R1 ↔ R2        | `10.0.0.0/30`   | R1-HQ     | `10.0.0.1`   |
| R1 ↔ R2        | `10.0.0.0/30`   | R2-BRANCH | `10.0.0.2`   |
| R1 ↔ CORE-SW1  | `10.0.1.0/30`   | R1-HQ     | `10.0.1.1`   |
| R1 ↔ CORE-SW1  | `10.0.1.0/30`   | CORE-SW1  | `10.0.1.2`   |
| R1 ↔ CORE-SW2  | `10.0.1.4/30`   | R1-HQ     | `10.0.1.5`   |
| R1 ↔ CORE-SW2  | `10.0.1.4/30`   | CORE-SW2  | `10.0.1.6`   |
| R1 ↔ ISP       | `203.0.113.0/30` | R1-HQ     | `203.0.113.2` |
| R1 ↔ ISP       | `203.0.113.0/30` | ISP       | `203.0.113.1` |

Using /30 networks minimizes address waste on point-to-point links.

---

## 5. Server Addressing

Servers use static IP addresses because they provide infrastructure services.

| Device       | IP Address      | Network | Gateway        | Services   |
| ------------ | --------------- | ------- | -------------- | ---------- |
| SRV-DHCP-DNS | `192.168.30.10` | VLAN 30 | `192.168.30.1` | DHCP, DNS  |
| SRV-WEB      | `192.168.30.20` | VLAN 30 | `192.168.30.1` | HTTP       |

---

## 6. End-Device Addressing

User devices receive their IPv4 configuration dynamically through DHCP.

### Sales

- Network: `192.168.10.0/24`
- Gateway: `192.168.10.1`
- DHCP Server: `192.168.30.10`
- Example: SALES-PC1 → `192.168.10.101`

### IT

- Network: `192.168.20.0/24`
- Gateway: `192.168.20.1`
- DHCP Server: `192.168.30.10`

### Guest

- Network: `192.168.40.0/24`
- Gateway: `192.168.40.1`
- DHCP Server: `192.168.30.10`

### Branch

- Network: `192.168.100.0/24`
- Gateway: `192.168.100.1`
- DHCP Server: `192.168.30.10`

---

## 7. DHCP Address Pools

The centralized DHCP server provides separate address pools for each client network.

| Pool   | Network            | Gateway          | Starting Address  |
| ------ | ------------------ | ---------------- | ----------------- |
| SALES  | `192.168.10.0/24`  | `192.168.10.1`   | `192.168.10.100`  |
| IT     | `192.168.20.0/24`  | `192.168.20.1`   | `192.168.20.100`  |
| GUEST  | `192.168.40.0/24`  | `192.168.40.1`   | `192.168.40.100`  |
| BRANCH | `192.168.100.0/24` | `192.168.100.1`  | `192.168.100.100` |

The DHCP server also provides:

- DNS Server: `192.168.30.10`

---

## 8. DNS Addressing

The internal DNS server is:

```
192.168.30.10
```

DNS record:

```text
www.enterprise.local → 192.168.30.20
```

This allows clients to access the internal web server using a hostname rather than its IP address.

---

## 9. Simulated Internet Addressing

The ISP connection uses:

- R1-HQ: `203.0.113.2/30`
- ISP: `203.0.113.1/30`

The simulated Internet destination is:

- ISP Loopback0: `8.8.8.8/32`

The address represents an external destination within the Packet Tracer simulation.

---

## 10. NAT Addressing

Internal networks use private RFC 1918 addresses.

R1-HQ translates eligible internal traffic to its external interface address:

- Inside Local: `192.168.x.x`
- Inside Global: `203.0.113.2`

PAT allows multiple internal hosts to share the same external address.

> For the NAT/PAT configuration and traffic flow details, see [routing-design.md § NAT/PAT](routing-design.md#natpat).

---

## 11. Addressing Summary

```text
HQ VLANs
------------------------------
VLAN 10   192.168.10.0/24
VLAN 20   192.168.20.0/24
VLAN 30   192.168.30.0/24
VLAN 40   192.168.40.0/24
VLAN 50   192.168.50.0/24
VLAN 99   192.168.99.0/24

Branch
------------------------------
VLAN 100  192.168.100.0/24

WAN / Transit
------------------------------
R1-R2          10.0.0.0/30
R1-SW1         10.0.1.0/30
R1-SW2         10.0.1.4/30
R1-ISP         203.0.113.0/30

Simulated Internet
------------------------------
ISP Loopback   8.8.8.8/32
```

---

## 12. Addressing Design Principles

The addressing plan was designed to provide:

- Logical separation by department and function
- Simple VLAN-to-subnet mapping
- Predictable default gateways
- Dedicated point-to-point WAN subnets
- Static addresses for infrastructure servers
- DHCP for end-user devices
- Addressing that can be expanded as the network grows
