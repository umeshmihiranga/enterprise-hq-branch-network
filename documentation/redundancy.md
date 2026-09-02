# Redundancy Design

## Overview

Redundancy is implemented throughout the HQ network to improve availability and reduce the impact of individual link or device failures.

The design uses three primary technologies:
- **HSRP** for default-gateway redundancy
- **EtherChannel** for redundant core links
- **Rapid PVST+** for Layer 2 loop prevention and redundant-path recovery

The objective is to maintain connectivity when a network component fails without creating Layer 2 loops.

---

## 1. HSRP Gateway Redundancy

### Purpose

HSRP (Hot Standby Router Protocol) provides a virtual default gateway for the HQ VLANs. Instead of clients using the physical IP address of a single core switch, they use a shared virtual IP address.

### HSRP Roles

CORE-SW1 is configured as the preferred HSRP device with `Priority: 110`.
CORE-SW2 is configured with `Priority: 100`.

Both switches use `preempt`. This allows CORE-SW1 to regain the active role when it becomes available again.

> For the full list of HSRP virtual IP addresses and physical SVI IPs, see [ip-addressing.md § HSRP Addressing](ip-addressing.md#3-hsrp-addressing).

### HSRP Failure Scenario

Under normal operation, clients send traffic to the HSRP VIP, which is handled by CORE-SW1 (ACTIVE).

If CORE-SW1 becomes unavailable, CORE-SW2 (STANDBY) assumes the ACTIVE role. Clients continue using the same default gateway. No client gateway configuration needs to change.

---

## 2. EtherChannel Redundancy

### Purpose

The two core switches are connected using an EtherChannel.

The bundle consists of `Fa0/23` and `Fa0/24` on both core switches. These physical interfaces form `Port-channel1`.

LACP is used to negotiate the EtherChannel.

### LACP Configuration

The member interfaces use:
```
channel-group 1 mode active
```

This places the interfaces in active LACP negotiation mode.

### EtherChannel Benefits

Without EtherChannel, multiple parallel Layer 2 links could create a switching loop. With EtherChannel, the switch treats the physical links as a single logical connection.

Benefits include:
- Increased available bandwidth
- Link-level redundancy
- Faster recovery from a member-link failure
- Simplified STP topology
- Reduced risk of Layer 2 loops

### EtherChannel Failure

If one physical member fails, traffic can continue through the remaining EtherChannel member. The entire logical connection does not need to fail because of one physical link failure.

---

## 3. Rapid PVST+ Redundancy

### Purpose

The network contains redundant Layer 2 paths between the core and access switches. These paths provide physical redundancy but could create Layer 2 loops.

Rapid PVST+ is therefore used:
```
spanning-tree mode rapid-pvst
```

### STP Root Design

CORE-SW1 is configured as the preferred STP root:
```
spanning-tree vlan 10,20,30,40,50,99 priority 24576
```

CORE-SW2 acts as the secondary root.

### Redundant Path Blocking

Because redundant Layer 2 links exist, STP places selected interfaces into a blocking/alternate state. The alternate path is retained as a backup. It is not a failure when STP blocks a redundant path; this is expected behavior.

### STP Failure Recovery

If the primary Layer 2 path fails, Rapid PVST+ can transition an alternate path into forwarding, effectively restoring connectivity.

---

## 4. Access-Link Redundancy

Access switches have redundant uplinks.

For example, ACCESS-SW1 connects to both CORE-SW1 (`Fa0/24`) and CORE-SW2 (`Fa0/23`).

This prevents a single uplink failure from completely isolating an access switch. STP controls which path forwards traffic under normal conditions.

---

## 5. Multi-Layer Redundancy

The network provides redundancy at multiple layers, creating several independent protection mechanisms.

### Redundancy Matrix

| Component | Primary Function | Failure Protection |
|---|---|---|
| HSRP | Default gateway | Core switch failure |
| EtherChannel | Core interconnect | Individual link failure |
| Rapid PVST+ | Loop prevention | Redundant Layer 2 paths |
| Dual access uplinks | Access connectivity | Single uplink failure |
| OSPF | Dynamic routing | Routed-path changes |

### OSPF Redundancy

OSPF provides dynamic routing between the Layer 3 devices. The core has separate routed connections toward R1-HQ, providing two routed paths toward the HQ edge router. OSPF can maintain routing information through both core devices.

---

## 6. Failure Testing

The redundancy design can be validated by intentionally failing individual components.

- **HSRP Test**: Run `show standby brief`. Shut down the active core SVI and verify that CORE-SW2 becomes active. Test client connectivity.
- **EtherChannel Test**: Run `show etherchannel summary`. A single member can be taken offline to verify that the Port-channel remains operational.
- **STP Test**: Run `show spanning-tree vlan 10`. Verify CORE-SW1 is the root bridge and alternate interfaces can transition when the active path fails.
- **OSPF Test**: Check OSPF neighbors with `show ip ospf neighbor` and verify learned routes with `show ip route ospf`.

### Expected Failure Behavior

- **Single HSRP Gateway Failure** → Standby Core Becomes Active → Clients Keep Same Virtual Gateway
- **Single EtherChannel Member Failure** → Remaining Member Continues Forwarding → Port-channel Remains Operational
- **Primary Layer 2 Path Failure** → Rapid PVST+ Detects Failure → Alternate Path Moves Toward Forwarding → Connectivity Restored
- **Routed Path Change** → OSPF Detects Topology Change → Routing Table Recalculated → Traffic Uses Available Route

---

## 7. Design Rationale

Redundancy was implemented to avoid relying on a single network component for connectivity. Together, these technologies create a more resilient enterprise network architecture capable of maintaining connectivity during individual link, path, or core-device failures.
