# CS 2204 – Communications and Networking
## Module 4: Network Layer

---

## Table of Contents

1. [Overview](#1-overview)
2. [Network Layer Responsibilities](#2-network-layer-responsibilities)
3. [Logical Addressing: IPv4](#3-logical-addressing-ipv4)
4. [IPv4 Address Classes and Special Addresses](#4-ipv4-address-classes-and-special-addresses)
5. [Subnetting and CIDR](#5-subnetting-and-cidr)
6. [Network Address Translation (NAT)](#6-network-address-translation-nat)
7. [IPv4 Packet Format](#7-ipv4-packet-format)
8. [IPv6](#8-ipv6)
9. [Routing Concepts](#9-routing-concepts)
10. [Routing Algorithms](#10-routing-algorithms)
11. [Routing Protocols](#11-routing-protocols)
12. [Internet Control Message Protocol (ICMP)](#12-internet-control-message-protocol-icmp)
13. [Address Resolution Protocol (ARP)](#13-address-resolution-protocol-arp)
14. [Dynamic Host Configuration Protocol (DHCP)](#14-dynamic-host-configuration-protocol-dhcp)
15. [Summary](#15-summary)
16. [Key Terms](#16-key-terms)
17. [Review Questions](#17-review-questions)

---

## 1. Overview

The Network Layer is the layer responsible for the end-to-end delivery of individual packets across multiple networks. While the Data Link Layer handles hop-to-hop delivery on a single link, the Network Layer ensures that a packet reaches its ultimate destination regardless of how many routers and networks lie in between. It introduces **logical addressing**, **routing**, and **packet forwarding** — the core mechanisms of the modern Internet.

**Module Learning Objectives:**

By the end of this module, students should be able to:

- Describe the responsibilities of the Network Layer.
- Explain IPv4 addressing, including classes, subnetting, and CIDR.
- Perform subnetting calculations (network address, broadcast, host range, subnet mask).
- Explain the purpose and operation of NAT.
- Describe the IPv4 packet format and the role of each field.
- Compare IPv4 and IPv6, and describe the IPv6 address format.
- Explain the difference between routing and forwarding.
- Compare distance vector and link-state routing algorithms.
- Describe common routing protocols (RIP, OSPF, BGP).
- Explain the role of ICMP, ARP, and DHCP as Network Layer support protocols.

---

## 2. Network Layer Responsibilities

The **Network Layer (OSI Layer 3)** is responsible for the delivery of packets from the original source host to the final destination host — across potentially many intermediate networks and routers.

### 2.1 Core Responsibilities

| Responsibility | Description |
|---|---|
| **Logical addressing** | Assigns globally unique IP addresses to identify source and destination hosts across networks |
| **Routing** | Determines the best path for a packet to travel from source to destination |
| **Forwarding** | Moving a packet from an input interface to the correct output interface on a router |
| **Packetizing** | Encapsulating the Transport Layer segment into a Network Layer packet (adding IP header) |
| **Fragmentation** | Breaking large packets into smaller fragments when they exceed the MTU of a link |
| **Error reporting** | Using ICMP to notify sources of delivery problems |

### 2.2 Routing vs. Forwarding

These two terms are often confused:

| Concept | Description | Analogy |
|---|---|---|
| **Routing** | The process of building and maintaining routing tables (the "map") | Planning a road trip route |
| **Forwarding** | The act of moving a specific packet to the correct output port based on the routing table | The driver following the plan at each junction |

### 2.3 Connection Services

The Network Layer can offer two types of service:

**Connectionless (Datagram) service:**
- Each packet is treated independently.
- No pre-established path; packets may take different routes.
- Used by the **Internet Protocol (IP)**.
- Simple and flexible, but delivery is not guaranteed.

**Connection-oriented (Virtual Circuit) service:**
- A logical path is established before data transfer.
- All packets follow the same path.
- Used by **ATM, Frame Relay, MPLS**.
- Provides predictable QoS but requires setup overhead.

---

## 3. Logical Addressing: IPv4

### 3.1 What is an IP Address?

An **IPv4 address** is a 32-bit number that uniquely identifies a device (interface) on an IP network. It is typically written in **dotted-decimal notation** — four decimal numbers (octets) separated by dots.

```
Binary:         11000000 . 10101000 . 00000001 . 00000001
Dotted-decimal:   192    .   168    .    1     .    1
```

Each octet ranges from 0 to 255. The full range of IPv4 addresses is 0.0.0.0 to 255.255.255.255 — approximately **4.3 billion** unique addresses.

### 3.2 Network vs. Host Portion

An IP address has two parts:

```
┌──────────────────────┬──────────────────────┐
│    Network Portion   │     Host Portion     │
│  (identifies the     │  (identifies the     │
│   network/subnet)    │   device on that     │
│                      │   network)           │
└──────────────────────┴──────────────────────┘
```

The **subnet mask** (or prefix length) defines where the network portion ends and the host portion begins.

### 3.3 Subnet Mask

A subnet mask is a 32-bit number with consecutive 1s for the network portion and 0s for the host portion.

| Subnet Mask | CIDR Notation | Network Bits | Host Bits | Hosts per Network |
|---|---|---|---|---|
| 255.0.0.0 | /8 | 8 | 24 | 16,777,214 |
| 255.255.0.0 | /16 | 16 | 16 | 65,534 |
| 255.255.255.0 | /24 | 24 | 8 | 254 |
| 255.255.255.128 | /25 | 25 | 7 | 126 |
| 255.255.255.192 | /26 | 26 | 6 | 62 |
| 255.255.255.224 | /27 | 27 | 5 | 30 |
| 255.255.255.240 | /28 | 28 | 4 | 14 |
| 255.255.255.252 | /30 | 30 | 2 | 2 |

**Number of usable hosts = 2ʰ − 2** (subtract 2 for network address and broadcast address).

---

## 4. IPv4 Address Classes and Special Addresses

### 4.1 Classful Addressing

Originally, IPv4 addresses were divided into five classes based on the leading bits of the first octet:

| Class | Leading Bits | First Octet Range | Default Mask | # Networks | Hosts/Network | Use |
|---|---|---|---|---|---|---|
| **A** | 0 | 1 – 126 | /8 (255.0.0.0) | 126 | 16,777,214 | Large organizations |
| **B** | 10 | 128 – 191 | /16 (255.255.0.0) | 16,384 | 65,534 | Medium organizations |
| **C** | 110 | 192 – 223 | /24 (255.255.255.0) | 2,097,152 | 254 | Small organizations |
| **D** | 1110 | 224 – 239 | N/A | N/A | N/A | Multicast |
| **E** | 1111 | 240 – 255 | N/A | N/A | N/A | Reserved/experimental |

> Note: Address `127.x.x.x` is reserved for loopback (127.0.0.1 = localhost).

Classful addressing was **wasteful** (a company needing 300 hosts wasted most of a Class B block) and was replaced by **CIDR**.

### 4.2 Private IP Address Ranges (RFC 1918)

Certain address ranges are designated for **private use** — they are not routed on the public Internet and can be reused by any organization internally:

| Class | Private Range | CIDR | # Addresses |
|---|---|---|---|
| A | 10.0.0.0 – 10.255.255.255 | 10.0.0.0/8 | 16,777,216 |
| B | 172.16.0.0 – 172.31.255.255 | 172.16.0.0/12 | 1,048,576 |
| C | 192.168.0.0 – 192.168.255.255 | 192.168.0.0/16 | 65,536 |

Devices using private addresses require **NAT** to communicate with the public Internet.

### 4.3 Special IPv4 Addresses

| Address | Purpose |
|---|---|
| `0.0.0.0` | This host (used during boot/DHCP request) |
| `255.255.255.255` | Limited broadcast (all hosts on local network) |
| `127.0.0.1` | Loopback (testing local TCP/IP stack) |
| `169.254.x.x` | APIPA — auto-assigned when DHCP fails |
| `x.x.x.0` | Network address (host bits all 0) |
| `x.x.x.255` | Directed broadcast for a subnet (host bits all 1) |

---

## 5. Subnetting and CIDR

### 5.1 Why Subnet?

**Subnetting** divides a single large network into multiple smaller **subnetworks (subnets)**. Benefits include:

- **Reduced broadcast domains** — fewer devices receive each broadcast.
- **Improved security** — traffic between subnets can be filtered by routers.
- **Efficient address allocation** — assign address blocks proportional to actual need.
- **Organizational structure** — subnets can mirror departments, floors, or locations.

### 5.2 Subnetting Process

To subnet a network, bits are **borrowed from the host portion** to create additional network bits.

**Key formulas:**
```
Number of subnets  = 2ˢ          (s = number of borrowed bits)
Hosts per subnet   = 2ʰ − 2      (h = remaining host bits)
Subnet size        = 2ʰ          (total addresses including network and broadcast)
```

### 5.3 Subnetting Example

**Problem:** Subnet the network `192.168.1.0/24` into 4 equal subnets.

**Step 1 — Determine bits to borrow:**
- Need 4 subnets → 2² = 4 → borrow **2 bits**.
- New prefix: /24 + 2 = **/26**
- New subnet mask: `255.255.255.192`

**Step 2 — Determine subnet size:**
- Host bits remaining: 32 − 26 = 6 bits
- Addresses per subnet: 2⁶ = **64**
- Usable hosts per subnet: 64 − 2 = **62**

**Step 3 — List the subnets:**

| Subnet | Network Address | First Host | Last Host | Broadcast |
|---|---|---|---|---|
| 1 | 192.168.1.0 | 192.168.1.1 | 192.168.1.62 | 192.168.1.63 |
| 2 | 192.168.1.64 | 192.168.1.65 | 192.168.1.126 | 192.168.1.127 |
| 3 | 192.168.1.128 | 192.168.1.129 | 192.168.1.190 | 192.168.1.191 |
| 4 | 192.168.1.192 | 192.168.1.193 | 192.168.1.254 | 192.168.1.255 |

### 5.4 CIDR (Classless Inter-Domain Routing)

**CIDR** (RFC 4632) abandons the rigid class-based boundaries of classful addressing. Any prefix length (/0 to /32) can be assigned, allowing more precise allocation.

**CIDR notation:** `IP address / prefix length` → e.g., `203.0.113.0/25`

**Supernetting (route aggregation):**
CIDR also allows multiple contiguous networks to be combined into a single **supernet** (summary route), reducing the size of routing tables.

**Example:**
```
192.168.0.0/24
192.168.1.0/24
192.168.2.0/24
192.168.3.0/24
─────────────────────────
Can be summarized as: 192.168.0.0/22  (covers all four /24 networks)
```

This is called **route summarization** or **supernetting** and is essential for keeping Internet routing tables manageable.

### 5.5 Variable-Length Subnet Masking (VLSM)

**VLSM** allows subnets of different sizes to be created from the same address block — each subnet gets only the number of addresses it needs.

**Example:** Assign subnets from `10.0.0.0/24` for departments of sizes 100, 50, and 2 hosts:

| Department | Hosts Needed | Hosts Allocated | Subnet | Mask |
|---|---|---|---|---|
| Engineering | 100 | 126 | 10.0.0.0/25 | /25 |
| Sales | 50 | 62 | 10.0.0.128/26 | /26 |
| WAN Link | 2 | 2 | 10.0.0.192/30 | /30 |

---

## 6. Network Address Translation (NAT)

### 6.1 Why NAT?

IPv4's ~4.3 billion addresses are exhausted. **NAT** allows many devices with **private IP addresses** to share one or a few **public IP addresses** when accessing the Internet.

### 6.2 How NAT Works

A NAT-enabled router maintains a **NAT translation table** that maps private (inside local) IP address:port combinations to public (inside global) IP address:port combinations.

```
Private Network              NAT Router             Public Internet
─────────────────────────────────────────────────────────────────────
192.168.1.10:5000 ──►  [NAT Table]  ──► 203.0.113.5:10001 ──► Server
192.168.1.11:5000 ──►  [NAT Table]  ──► 203.0.113.5:10002 ──► Server
192.168.1.12:5001 ──►  [NAT Table]  ──► 203.0.113.5:10003 ──► Server
```

**NAT Table Example:**

| Inside Local (Private) | Inside Global (Public) | Outside Global (Server) |
|---|---|---|
| 192.168.1.10:5000 | 203.0.113.5:10001 | 8.8.8.8:80 |
| 192.168.1.11:5000 | 203.0.113.5:10002 | 8.8.8.8:80 |

### 6.3 NAT Types

| Type | Description |
|---|---|
| **Static NAT** | One-to-one mapping of private to public address; used for servers that need a fixed public IP |
| **Dynamic NAT** | Maps private addresses to a pool of public addresses; no fixed mapping |
| **PAT (Port Address Translation) / NAT Overload** | Many private addresses share **one** public IP; distinguished by unique port numbers. Most common type (used in home routers) |

### 6.4 NAT Advantages and Disadvantages

| Advantages | Disadvantages |
|---|---|
| Conserves public IPv4 addresses | Breaks end-to-end connectivity model |
| Provides a degree of security (internal IPs hidden) | Complicates some protocols (VoIP, FTP, IPSec) |
| Transparent to internal devices | Increases router processing overhead |
| Delays IPv4 exhaustion | Impedes peer-to-peer applications |

---

## 7. IPv4 Packet Format

The **IPv4 packet** (also called a **datagram**) consists of a header and a data section. The header is a minimum of **20 bytes**.

### 7.1 IPv4 Header Fields

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
├─────────┬─────────┬─────────────────────────────────────────────┤
│ Version │   IHL   │      DSCP/TOS       │    Total Length       │
├─────────────────────────────────┬────┬──────────────────────────┤
│        Identification           │Flgs│    Fragment Offset       │
├─────────────────────────────────┴────┴──────────────────────────┤
│      TTL        │    Protocol   │       Header Checksum         │
├─────────────────────────────────────────────────────────────────┤
│                       Source IP Address                         │
├─────────────────────────────────────────────────────────────────┤
│                    Destination IP Address                       │
├─────────────────────────────────────────────────────────────────┤
│                    Options (if IHL > 5)                         │
└─────────────────────────────────────────────────────────────────┘
```

### 7.2 Field Descriptions

| Field | Size | Description |
|---|---|---|
| **Version** | 4 bits | IP version number (4 for IPv4) |
| **IHL** (Internet Header Length) | 4 bits | Header length in 32-bit words; minimum value = 5 (20 bytes) |
| **DSCP/TOS** | 8 bits | Differentiated Services Code Point; used for QoS prioritization |
| **Total Length** | 16 bits | Total packet size (header + data) in bytes; max = 65,535 bytes |
| **Identification** | 16 bits | Unique ID assigned to each original packet; used to reassemble fragments |
| **Flags** | 3 bits | Bit 0: Reserved (0); Bit 1: DF (Don't Fragment); Bit 2: MF (More Fragments) |
| **Fragment Offset** | 13 bits | Position of this fragment in the original packet (in 8-byte units) |
| **TTL** (Time to Live) | 8 bits | Decremented by 1 at each router; packet discarded when TTL = 0 (prevents infinite loops) |
| **Protocol** | 8 bits | Identifies the upper-layer protocol: 6 = TCP, 17 = UDP, 1 = ICMP, 89 = OSPF |
| **Header Checksum** | 16 bits | Error check for the header only (not the data) |
| **Source IP Address** | 32 bits | IP address of the originating host |
| **Destination IP Address** | 32 bits | IP address of the intended recipient |
| **Options** | Variable | Optional fields for testing, debugging, security; rarely used |

### 7.3 Fragmentation

When a packet is larger than the **MTU (Maximum Transmission Unit)** of a link it must traverse, the router **fragments** it into smaller pieces.

- **MTU:** Maximum packet size a link can carry. Ethernet MTU = **1500 bytes**.
- Each fragment has its own IP header with the same Identification field.
- **MF flag:** Set to 1 for all fragments except the last.
- **Fragment Offset:** Indicates where in the original packet this fragment's data belongs.
- Reassembly happens **only at the destination host** (not at intermediate routers).

**Fragmentation Example:**

A 4000-byte packet crosses an Ethernet link (MTU = 1500 bytes):

| Fragment | Data Size | Offset | MF Flag |
|---|---|---|---|
| 1 | 1480 bytes | 0 | 1 |
| 2 | 1480 bytes | 185 (1480/8) | 1 |
| 3 | 1040 bytes | 370 (2960/8) | 0 |

> Note: Each fragment's data must be a multiple of 8 bytes (except the last), because the offset field is in 8-byte units.

---

## 8. IPv6

### 8.1 Why IPv6?

IPv4 address exhaustion, along with the need for better security, QoS, and auto-configuration, drove the development of **IPv6 (Internet Protocol version 6)**, standardized in RFC 2460 (now RFC 8200).

**Key improvements over IPv4:**

| Feature | IPv4 | IPv6 |
|---|---|---|
| Address size | 32 bits (~4.3 billion) | 128 bits (~3.4 × 10³⁸) |
| Header size | 20–60 bytes (variable) | 40 bytes (fixed) |
| Header checksum | Yes | No (removed; upper layers handle it) |
| Fragmentation | By routers and hosts | By source host only |
| NAT | Required | Not needed (abundant addresses) |
| IPSec | Optional | Built-in (mandatory in original spec) |
| Auto-configuration | DHCP required | Stateless Address Autoconfiguration (SLAAC) |
| Broadcast | Yes | No (replaced by multicast and anycast) |
| QoS | Limited (TOS field) | Flow Label field for QoS |

### 8.2 IPv6 Address Format

IPv6 addresses are **128 bits** written as eight groups of four hexadecimal digits, separated by colons:

```
2001:0DB8:0000:0000:0000:FF00:0042:8329
```

**Simplification rules:**

1. **Leading zeros** in a group can be omitted:
   `2001:DB8:0:0:0:FF00:42:8329`

2. **One consecutive group of all-zero groups** can be replaced by `::` (only once per address):
   `2001:DB8::FF00:42:8329`

**Examples:**

| Full Address | Compressed |
|---|---|
| `2001:0DB8:0000:0000:0000:0000:0000:0001` | `2001:DB8::1` |
| `0000:0000:0000:0000:0000:0000:0000:0001` | `::1` (loopback) |
| `0000:0000:0000:0000:0000:0000:0000:0000` | `::` (unspecified) |
| `FE80:0000:0000:0000:0200:F8FF:FE21:67CF` | `FE80::200:F8FF:FE21:67CF` |

### 8.3 IPv6 Address Types

| Type | Description | Example Prefix |
|---|---|---|
| **Unicast** | One-to-one; identifies a single interface | `2001:DB8::/32` (global) |
| **Multicast** | One-to-many; sent to all members of a group | `FF00::/8` |
| **Anycast** | One-to-nearest; sent to the nearest interface with that address | Same unicast format |
| **Link-local** | Auto-configured; valid only on the local link | `FE80::/10` |
| **Loopback** | Local loopback testing | `::1/128` |
| **Unspecified** | Default source before address is assigned | `::/128` |

> IPv6 has **no broadcast** — broadcast functions are replaced by multicast.

### 8.4 IPv6 Header Format

The IPv6 header is **fixed at 40 bytes** — simpler than IPv4:

| Field | Size | Description |
|---|---|---|
| **Version** | 4 bits | Always 6 |
| **Traffic Class** | 8 bits | Similar to IPv4 DSCP/TOS; for QoS |
| **Flow Label** | 20 bits | Identifies packets belonging to the same flow for QoS |
| **Payload Length** | 16 bits | Length of the data (excluding the 40-byte header) |
| **Next Header** | 8 bits | Identifies the next header (e.g., 6=TCP, 17=UDP, 58=ICMPv6, or extension header) |
| **Hop Limit** | 8 bits | Same as IPv4 TTL — decremented at each hop |
| **Source Address** | 128 bits | IPv6 address of sender |
| **Destination Address** | 128 bits | IPv6 address of receiver |

**Extension headers** (e.g., for fragmentation, routing, authentication) are chained after the base header via the Next Header field — keeping the base header clean and fixed-size.

### 8.5 IPv6 Transition Mechanisms

Because IPv4 and IPv6 must coexist during the long transition period, several mechanisms exist:

| Mechanism | Description |
|---|---|
| **Dual stack** | A device runs both IPv4 and IPv6 simultaneously |
| **Tunneling** | IPv6 packets are encapsulated inside IPv4 packets to cross IPv4 networks |
| **Translation (NAT64)** | Translates between IPv6 and IPv4 packets at the network boundary |

---

## 9. Routing Concepts

### 9.1 The Routing Table

Every router maintains a **routing table** — a database of known network destinations and how to reach them.

**Typical routing table entry:**

| Destination Network | Subnet Mask | Next Hop | Interface | Metric | Source |
|---|---|---|---|---|---|
| 192.168.1.0 | /24 | — | eth0 | 0 | Direct |
| 10.0.0.0 | /8 | 192.168.1.1 | eth0 | 1 | Static |
| 0.0.0.0 | /0 | 203.0.113.1 | eth1 | 1 | Default |

- **Destination network:** The network this entry applies to.
- **Next hop:** The IP address of the next router toward the destination.
- **Interface:** The local interface to use for forwarding.
- **Metric:** A cost value used to compare routes (lower is better).
- **Source:** How the route was learned (directly connected, static, or dynamic protocol).

### 9.2 Longest Prefix Match

When multiple entries match a destination IP, the router uses the **most specific match** — the entry with the **longest prefix** (most network bits).

**Example:**

| Entry | Network | Match? |
|---|---|---|
| A | 192.168.1.0/24 | Yes |
| B | 192.168.1.128/25 | Yes (more specific) |
| C | 0.0.0.0/0 | Yes (default) |

For destination `192.168.1.200` → Router chooses **Entry B** (/25 is the longest match).

### 9.3 Types of Routes

| Type | Description |
|---|---|
| **Directly connected** | Networks directly attached to a router interface; added automatically |
| **Static routes** | Manually configured by an administrator; does not adapt to topology changes |
| **Default route** | A catch-all route (`0.0.0.0/0`) used when no more specific match exists |
| **Dynamic routes** | Learned automatically via routing protocols (RIP, OSPF, BGP) |

### 9.4 Administrative Distance

When a router learns the same route from multiple sources, **administrative distance (AD)** determines which source is trusted most (lower AD = higher trust):

| Route Source | AD Value |
|---|---|
| Directly connected | 0 |
| Static route | 1 |
| EIGRP (internal) | 90 |
| OSPF | 110 |
| RIP | 120 |
| External BGP | 20 |
| Unknown | 255 (unusable) |

---

## 10. Routing Algorithms

### 10.1 Distance Vector Routing

**How it works:**
- Each router maintains a table of distances (metrics) to every known network.
- Routers periodically share their **entire routing table** with directly connected neighbors.
- Each router updates its table using the **Bellman-Ford algorithm**:

```
D(x, y) = min over all neighbors v { cost(x, v) + D(v, y) }
```

Where D(x, y) = best known distance from router x to destination y.

**Characteristics:**
- Routers know only distances, not the full topology.
- Simple to implement; low CPU/memory requirements.
- **Slow convergence** — changes propagate gradually hop by hop.
- Subject to the **count-to-infinity problem**: a routing loop where routers keep incrementing the metric for an unreachable destination.

**Solutions to count-to-infinity:**
- **Split horizon:** Never advertise a route back to the neighbor from which it was learned.
- **Poison reverse:** Advertise a failed route with metric = infinity (16 for RIP) back to the source neighbor.
- **Triggered updates:** Send updates immediately when a topology change occurs (don't wait for the next periodic interval).

### 10.2 Link-State Routing

**How it works:**
- Each router discovers its directly connected neighbors and the cost of each link.
- Each router floods its **link-state advertisement (LSA)** to every other router in the network.
- Every router builds a complete, identical **topological map** of the entire network.
- Each router independently runs **Dijkstra's Shortest Path First (SPF) algorithm** to compute the shortest path to every destination.

**Dijkstra's Algorithm (simplified):**
1. Start at the source router; set its distance to 0 and all others to ∞.
2. Visit the unvisited node with the smallest known distance.
3. Update the distances of its neighbors.
4. Repeat until all nodes are visited.

**Characteristics:**
- **Fast convergence** — LSAs are flooded immediately upon topology change.
- Routers have a complete view of the topology.
- Higher CPU and memory requirements than distance vector.
- More complex to implement.

### 10.3 Comparison

| Feature | Distance Vector | Link-State |
|---|---|---|
| Algorithm | Bellman-Ford | Dijkstra (SPF) |
| Information shared | Routing table (distances only) | Link-state advertisements (full topology) |
| Knowledge of topology | Partial (neighbors only) | Complete (entire network) |
| Convergence speed | Slow | Fast |
| Bandwidth usage | Periodic full table | Triggered updates (LSAs) |
| CPU/Memory | Low | High |
| Loop prevention | Split horizon, poison reverse | Complete topology view |
| Example protocols | RIP, EIGRP | OSPF, IS-IS |

### 10.4 Path Vector Routing

Used in **inter-domain routing** (between autonomous systems). Similar to distance vector but each route advertisement carries the **full path (list of ASes)** to the destination, preventing routing loops.

- Used by **BGP (Border Gateway Protocol)**.

---

## 11. Routing Protocols

### 11.1 Classification of Routing Protocols

```
Routing Protocols
├── Interior Gateway Protocols (IGP) — within an autonomous system (AS)
│   ├── Distance Vector: RIP, IGRP
│   ├── Hybrid: EIGRP
│   └── Link-State: OSPF, IS-IS
└── Exterior Gateway Protocol (EGP) — between autonomous systems
    └── Path Vector: BGP
```

An **Autonomous System (AS)** is a collection of networks under a single administrative authority (e.g., an ISP or large enterprise), identified by a unique **AS Number (ASN)**.

### 11.2 RIP (Routing Information Protocol)

- **Type:** Distance vector IGP.
- **Metric:** Hop count (number of routers to destination).
- **Maximum hop count:** 15 (16 = infinity / unreachable). Limits RIP to small networks.
- **Update interval:** Every **30 seconds** (full table broadcast to all neighbors).
- **Versions:**
  - **RIPv1:** Classful; does not send subnet mask in updates.
  - **RIPv2:** Classless; includes subnet mask; supports VLSM and CIDR; uses multicast (224.0.0.9).
  - **RIPng:** IPv6 version of RIP.

**Advantages:** Simple, easy to configure.
**Disadvantages:** Limited scalability (max 15 hops); slow convergence; high bandwidth usage from periodic full-table broadcasts.

### 11.3 OSPF (Open Shortest Path First)

- **Type:** Link-state IGP.
- **Metric:** **Cost**, based on link bandwidth: `Cost = 10⁸ / bandwidth (bps)`. Lower bandwidth = higher cost.
- **Algorithm:** Dijkstra's SPF.
- **Updates:** Triggered (only when topology changes); flooded as **LSAs (Link-State Advertisements)**.
- **Multicast addresses:** `224.0.0.5` (all OSPF routers), `224.0.0.6` (DR/BDR).
- **Authentication:** Supports MD5 authentication.

**OSPF Areas:**
For large networks, OSPF uses a hierarchical **area** structure to limit LSA flooding and reduce routing table sizes:

```
          ┌──────────────────────┐
          │     Area 0           │
          │   (Backbone Area)    │
          └──────────┬───────────┘
          ┌──────────┴───────────┐
       Area 1               Area 2
   (Non-backbone)        (Non-backbone)
```

- All non-backbone areas must connect to **Area 0 (backbone area)**.
- **ABR (Area Border Router):** Connects a non-backbone area to Area 0.
- **ASBR (Autonomous System Boundary Router):** Connects the OSPF domain to external routing domains.

**OSPF DR/BDR:** On multi-access networks (e.g., Ethernet), OSPF elects a **Designated Router (DR)** and **Backup DR (BDR)** to reduce LSA flooding overhead.

**Advantages:** Fast convergence; scalable; supports VLSM/CIDR; loop-free.
**Disadvantages:** More complex to configure; higher CPU/memory usage.

### 11.4 BGP (Border Gateway Protocol)

- **Type:** Path vector EGP (Exterior Gateway Protocol).
- **Purpose:** Routes traffic **between autonomous systems** on the Internet.
- **Version:** BGP-4 (RFC 4271) is the current standard; supports CIDR.
- **Transport:** Runs over **TCP port 179** (reliable delivery guaranteed by TCP).
- **Metric (attributes):** Does not use a simple numeric metric. Uses a set of **path attributes** for route selection, most importantly:
  - **AS_PATH:** List of ASes the route has traversed (used for loop prevention).
  - **NEXT_HOP:** IP address of the next-hop router.
  - **LOCAL_PREF:** Preference for outbound traffic within an AS.
  - **MED:** Multi-Exit Discriminator; suggests preferred entry point into an AS.

**BGP Types:**
- **eBGP (External BGP):** Between routers in **different** ASes.
- **iBGP (Internal BGP):** Between routers **within the same** AS.

**BGP is the routing protocol of the Internet** — it maintains the global routing table connecting all ISPs and large networks worldwide (~900,000+ routes as of 2025).

**Advantages:** Highly scalable; policy-based routing; loop prevention via AS_PATH.
**Disadvantages:** Slow convergence (designed for stability, not speed); complex configuration.

### 11.5 Routing Protocol Comparison

| Feature | RIP | OSPF | BGP |
|---|---|---|---|
| Type | Distance vector | Link-state | Path vector |
| Scope | IGP (intra-AS) | IGP (intra-AS) | EGP (inter-AS) |
| Metric | Hop count | Cost (bandwidth) | Path attributes |
| Max hops | 15 | Unlimited | Unlimited |
| Convergence | Slow | Fast | Slow (stable) |
| Algorithm | Bellman-Ford | Dijkstra SPF | Path vector |
| Transport | UDP port 520 | IP protocol 89 | TCP port 179 |
| Scalability | Small networks | Large enterprise | Internet-scale |

---

## 12. Internet Control Message Protocol (ICMP)

### 12.1 What is ICMP?

**ICMP** is a Network Layer support protocol used by IP devices to send **error messages** and **operational information** about network conditions. It is defined in **RFC 792** (ICMPv4) and **RFC 4443** (ICMPv6).

ICMP messages are encapsulated in IP packets (Protocol field = **1** for ICMPv4, **58** for ICMPv6).

> ICMP does not fix errors — it reports them to the source. Correction is left to higher layers.

### 12.2 Common ICMP Message Types

| Type | Code | Message | Triggered When |
|---|---|---|---|
| 0 | 0 | **Echo Reply** | Response to Echo Request (ping) |
| 3 | 0–15 | **Destination Unreachable** | Packet cannot be delivered (network, host, port, protocol unreachable) |
| 4 | 0 | **Source Quench** | Router congestion (deprecated in RFC 6633) |
| 5 | 0–3 | **Redirect** | Router tells host to use a better route |
| 8 | 0 | **Echo Request** | Used by ping to test reachability |
| 11 | 0 | **Time Exceeded** | TTL expired in transit (used by traceroute) |
| 11 | 1 | **Fragment Reassembly Timeout** | Fragments did not arrive in time |
| 12 | 0 | **Parameter Problem** | Malformed IP header |

### 12.3 Key ICMP Applications

#### ping

Uses **ICMP Echo Request (Type 8)** and **Echo Reply (Type 0)** to:
- Test whether a remote host is reachable.
- Measure round-trip time (RTT).

```
Host A  ──── ICMP Echo Request ────►  Host B
Host A  ◄─── ICMP Echo Reply   ────  Host B
```

#### traceroute

Uses ICMP **Time Exceeded (Type 11)** messages to map the path packets take:

1. Send a packet with **TTL = 1** → First router decrements TTL to 0, discards packet, and sends ICMP Time Exceeded back.
2. Send with **TTL = 2** → Second router responds.
3. Repeat with increasing TTL until the destination responds with Echo Reply.

Each hop's IP address and RTT are recorded, revealing the full path.

---

## 13. Address Resolution Protocol (ARP)

### 13.1 The ARP Problem

IP routing delivers packets to the correct **network**. But to actually deliver a frame on a local network, the sender needs the **MAC address** of the next hop (not just the IP address).

**ARP bridges the gap between Layer 3 (IP addresses) and Layer 2 (MAC addresses).**

### 13.2 ARP Operation

1. **ARP Request:** The sender broadcasts an ARP request frame to `FF:FF:FF:FF:FF:FF`:
   *"Who has IP address 192.168.1.1? Tell 192.168.1.10."*

2. **ARP Reply:** The device with that IP address responds with a **unicast** ARP reply:
   *"192.168.1.1 is at MAC address 00:1A:2B:3C:4D:5E."*

3. **ARP Cache:** The sender stores the IP-to-MAC mapping in its **ARP cache** to avoid repeating the request for a period of time (typically 2–20 minutes).

```
Sender                                    Target
──────                                    ──────
ARP Request (broadcast) ───────────────► All devices on LAN
                         ◄─── ARP Reply (unicast) ──── Target
```

### 13.3 ARP Cache and Gratuitous ARP

- The **ARP cache** stores recently resolved IP-to-MAC mappings. Entries expire after a timeout.
- **Gratuitous ARP:** A device sends an ARP request for its own IP address — used to announce its presence, update other caches, or detect IP address conflicts.

### 13.4 ARP for Routed Traffic

When a device sends traffic to a **different network** (not the same subnet), it sends the frame to the **default gateway (router)** — not directly to the destination.

- The frame's **destination MAC** = MAC address of the default gateway.
- The **destination IP** = final destination host.
- The router strips the frame, consults its routing table, and re-encapsulates in a new frame with the next-hop MAC.

### 13.5 Proxy ARP and ARP Security

**Proxy ARP:** A router responds to ARP requests on behalf of devices on another network, allowing devices without a default gateway to communicate off-subnet.

**ARP Spoofing (ARP Poisoning):** A malicious device sends fake ARP replies, associating its MAC address with another device's IP — enabling man-in-the-middle attacks.
- **Mitigation:** Dynamic ARP Inspection (DAI) on switches; static ARP entries; encrypted communications.

---

## 14. Dynamic Host Configuration Protocol (DHCP)

### 14.1 What is DHCP?

**DHCP** automates the assignment of IP configuration to network devices, eliminating the need for manual (static) IP address configuration. It operates at the **Application Layer** but supports Network Layer addressing and is typically discussed here.

DHCP is defined in **RFC 2131** and operates over **UDP (port 67 server, port 68 client)**.

### 14.2 DHCP DORA Process

DHCP uses a four-step handshake called **DORA**:

```
Client                              DHCP Server
──────                              ───────────
DHCP Discover (broadcast) ────────────────────►
              ◄──────────── DHCP Offer (unicast/broadcast)
DHCP Request  (broadcast) ────────────────────►
              ◄──────────── DHCP Acknowledgment (ACK)
```

| Step | Message | Description |
|---|---|---|
| **D**iscover | Client → Broadcast | Client broadcasts to find available DHCP servers |
| **O**ffer | Server → Client | Server offers an available IP address and configuration |
| **R**equest | Client → Broadcast | Client requests the offered IP (broadcast so other servers know) |
| **A**cknowledge | Server → Client | Server confirms the lease; client configures its interface |

### 14.3 DHCP Lease and Configuration

The DHCP server leases an IP address for a configurable **lease time** (e.g., 24 hours, 8 days). The client must renew before the lease expires.

**Information provided by DHCP:**
- IP address
- Subnet mask
- Default gateway
- DNS server address(es)
- Lease duration
- Domain name

### 14.4 DHCP Relay Agent

DHCP uses broadcasts, which do not cross router boundaries. In networks with multiple subnets, a **DHCP relay agent** (configured on the router) forwards DHCP Discover messages from clients to a centralized DHCP server on another subnet.

### 14.5 DHCPv6

For IPv6 networks, **DHCPv6** (RFC 3315) provides stateful address assignment. IPv6 also supports **SLAAC (Stateless Address Autoconfiguration)** — devices generate their own address using the network prefix from router advertisements + their interface ID (derived from MAC address), without needing a DHCP server.

---

## 15. Summary

This module covered the Network Layer — the third layer of the OSI model:

- The **Network Layer** is responsible for logical addressing, routing, forwarding, and packet delivery across multiple networks.
- **IPv4** uses 32-bit addresses in dotted-decimal notation; addresses are divided into a network and host portion by a subnet mask.
- **Classful addressing** divided addresses into Classes A–E; **CIDR** replaced this with flexible prefix lengths.
- **Subnetting** borrows host bits to create multiple subnetworks; **VLSM** allows variable-sized subnets from the same block.
- **NAT** (especially PAT) allows many private-addressed devices to share a single public IP address.
- The **IPv4 packet header** contains critical fields including TTL (prevents loops), Protocol (identifies upper layer), and fragment fields for reassembly.
- **IPv6** uses 128-bit addresses, a fixed 40-byte header, no broadcast, and built-in support for auto-configuration and IPSec.
- **Routing** (building the table) and **forwarding** (using the table) are distinct operations; routers use **longest prefix match** to select routes.
- **Distance vector** protocols (RIP) use Bellman-Ford and share routing tables; **link-state** protocols (OSPF) use Dijkstra and share topology maps.
- **BGP** is the path-vector protocol that routes traffic between autonomous systems on the Internet.
- **ICMP** provides error reporting and diagnostic tools (ping, traceroute).
- **ARP** resolves IP addresses to MAC addresses on local networks.
- **DHCP** automatically assigns IP configuration to devices using the DORA process.

---

## 16. Key Terms

| Term | Definition |
|---|---|
| **Network Layer** | OSI Layer 3; responsible for logical addressing and end-to-end packet delivery |
| **IPv4** | 32-bit Internet Protocol version 4; dominant protocol for packet delivery |
| **IP Address** | 32-bit logical address identifying a host interface on an IP network |
| **Subnet mask** | 32-bit mask defining the network and host portions of an IP address |
| **CIDR** | Classless Inter-Domain Routing; flexible prefix-length IP addressing |
| **Subnetting** | Dividing a network into smaller subnetworks by borrowing host bits |
| **VLSM** | Variable-Length Subnet Masking; subnets of different sizes from the same block |
| **NAT** | Network Address Translation; maps private IPs to public IPs |
| **PAT** | Port Address Translation; many private IPs share one public IP using port numbers |
| **TTL** | Time to Live; decremented at each hop to prevent infinite packet loops |
| **Fragmentation** | Breaking a large packet into smaller pieces to fit link MTU |
| **MTU** | Maximum Transmission Unit; largest packet size a link can carry |
| **IPv6** | 128-bit Internet Protocol version 6; successor to IPv4 |
| **SLAAC** | Stateless Address Autoconfiguration; IPv6 self-addressing without DHCP |
| **Routing table** | A router's database of known destinations and how to reach them |
| **Longest prefix match** | Rule to use the most specific matching entry in a routing table |
| **Administrative distance** | Trust ranking of routing information sources |
| **Distance vector** | Routing algorithm where routers share distance tables with neighbors |
| **Link-state** | Routing algorithm where routers share full topology maps and run SPF |
| **RIP** | Routing Information Protocol; distance vector IGP; hop-count metric; max 15 hops |
| **OSPF** | Open Shortest Path First; link-state IGP; cost metric; uses Dijkstra's algorithm |
| **BGP** | Border Gateway Protocol; path-vector EGP; routes between autonomous systems |
| **Autonomous System (AS)** | A network or group of networks under a single administrative authority |
| **ICMP** | Internet Control Message Protocol; error reporting and diagnostics |
| **ARP** | Address Resolution Protocol; resolves IP addresses to MAC addresses |
| **DHCP** | Dynamic Host Configuration Protocol; auto-assigns IP configuration |
| **DORA** | DHCP handshake: Discover, Offer, Request, Acknowledge |

---

## 17. Review Questions

**Conceptual Questions:**

1. What is the difference between routing and forwarding? Use an analogy to explain each.
2. Why was classful IPv4 addressing considered wasteful? How did CIDR address this problem?
3. Explain the purpose of the TTL field in an IPv4 packet. What happens when TTL reaches 0?
4. A packet of 5000 bytes must cross an Ethernet link with an MTU of 1500 bytes. How many fragments will be created? What are the size, offset, and MF flag of each fragment? (Assume a 20-byte IP header.)
5. Compare distance vector and link-state routing algorithms in terms of: information shared, convergence speed, and loop prevention.
6. What is the count-to-infinity problem in distance vector routing? How do split horizon and poison reverse help?
7. Why does BGP use path vectors rather than distances? How does the AS_PATH attribute prevent routing loops?
8. What is the role of ICMP? Explain how traceroute uses ICMP to map network paths.
9. Explain the ARP process. What happens when a host sends traffic to a device on a different subnet?
10. What is the DHCP DORA process? Why does the client use a broadcast for the Request step even after receiving an Offer?

**Application / Calculation Questions:**

11. You are given the network `172.16.0.0/16`. Subnet it to support at least 30 subnets, each with at least 500 hosts. What is the new prefix length? How many hosts are supported per subnet? List the first three subnet addresses, their broadcast addresses, and usable host ranges.

12. Given the IP address `192.168.10.200` with subnet mask `255.255.255.240`:
    - What is the network address?
    - What is the broadcast address?
    - What is the range of usable host addresses?
    - How many usable hosts does this subnet support?

13. A router has the following routing table entries for a destination packet to `10.1.2.130`:

    | Network | Prefix |
    |---|---|
    | 10.0.0.0 | /8 |
    | 10.1.0.0 | /16 |
    | 10.1.2.128 | /25 |
    | 0.0.0.0 | /0 |

    Which route is selected and why?

14. Using VLSM, allocate IP addresses from `192.168.5.0/24` for the following requirements: Department A needs 60 hosts, Department B needs 28 hosts, Department C needs 12 hosts, and one WAN link needs 2 hosts. Show the subnet, mask, first host, last host, and broadcast for each.

15. OSPF assigns a cost of `10⁸ / bandwidth`. Calculate the OSPF cost for: (a) a 100 Mbps Fast Ethernet link, (b) a 1 Gbps Gigabit Ethernet link, and (c) a 10 Mbps link. Which path would OSPF prefer for a route that can use either a 100 Mbps link or two 1 Gbps links in series?

---

*End of Module 4 | Next: Module 5 – Transport Layer*
