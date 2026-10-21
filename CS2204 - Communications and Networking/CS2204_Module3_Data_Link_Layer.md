# CS 2204 – Communications and Networking
## Module 3: Data Link Layer

---

## Table of Contents

1. [Overview](#1-overview)
2. [Data Link Layer Responsibilities](#2-data-link-layer-responsibilities)
3. [Framing](#3-framing)
4. [Flow Control](#4-flow-control)
5. [Error Detection and Correction](#5-error-detection-and-correction)
6. [Error Control Protocols](#6-error-control-protocols)
7. [Data Link Layer Sublayers: LLC and MAC](#7-data-link-layer-sublayers-llc-and-mac)
8. [Multiple Access Protocols](#8-multiple-access-protocols)
9. [Ethernet (IEEE 802.3)](#9-ethernet-ieee-8023)
10. [Wireless LANs (IEEE 802.11 – Wi-Fi)](#10-wireless-lans-ieee-80211--wi-fi)
11. [Point-to-Point Protocol (PPP)](#11-point-to-point-protocol-ppp)
12. [Switching at the Data Link Layer](#12-switching-at-the-data-link-layer)
13. [Virtual LANs (VLANs)](#13-virtual-lans-vlans)
14. [Summary](#14-summary)
15. [Key Terms](#15-key-terms)
16. [Review Questions](#16-review-questions)

---

## 1. Overview

The Data Link Layer is responsible for reliable, node-to-node delivery of data — that is, the movement of a frame from one hop (device) to the next adjacent hop over a single link. While the Physical Layer moves raw bits, the Data Link Layer packages those bits into meaningful units called **frames**, manages access to the shared medium, and handles error detection.

**Module Learning Objectives:**

By the end of this module, students should be able to:

- Describe the responsibilities of the Data Link Layer.
- Explain how framing structures data for node-to-node transmission.
- Compare and apply flow control mechanisms (Stop-and-Wait, Sliding Window).
- Describe error detection techniques (parity, CRC, checksum).
- Explain the difference between error detection and error correction.
- Describe ARQ protocols (Stop-and-Wait, Go-Back-N, Selective Repeat).
- Explain the MAC and LLC sublayers and their roles.
- Describe multiple access protocols (TDMA, FDMA, CSMA/CD, CSMA/CA).
- Describe the structure and operation of Ethernet and Wi-Fi (IEEE 802.11).
- Explain how switches and bridges operate at the Data Link Layer.
- Describe the concept of VLANs.

---

## 2. Data Link Layer Responsibilities

The Data Link Layer (OSI Layer 2) sits between the Physical Layer (Layer 1) and the Network Layer (Layer 3). It is responsible for **node-to-node** (hop-to-hop) communication along a single link, as opposed to end-to-end delivery handled by the Transport Layer.

### 2.1 Core Responsibilities

| Responsibility | Description |
|---|---|
| **Framing** | Packages bits from the Physical Layer into structured units called frames |
| **Physical addressing** | Adds MAC addresses to frames to identify the sender and receiver on the local link |
| **Flow control** | Prevents a fast sender from overwhelming a slow receiver |
| **Error control** | Detects (and sometimes corrects) errors introduced during transmission |
| **Access control** | Manages which device can use the shared medium at a given time |

### 2.2 Node-to-Node vs. End-to-End

It is critical to understand the scope of the Data Link Layer:

- A packet from source to destination may pass through many intermediate nodes (routers/switches).
- The Data Link Layer handles **each individual hop** separately — from Node A to Node B, then Node B to Node C, and so on.
- Each hop may use a **different Data Link Layer protocol** (e.g., Ethernet on a LAN, then Wi-Fi, then a fiber link).

```
[Host A] ──Ethernet──► [Switch] ──Ethernet──► [Router] ──Wi-Fi──► [Host B]
  DLL: Ethernet          DLL: Ethernet           DLL: Wi-Fi
```

---

## 3. Framing

**Framing** is the process of breaking a bitstream into manageable units called **frames**. A frame adds a header and trailer around the data (payload) from the Network Layer.

### 3.1 General Frame Structure

```
┌──────────────┬────────────────────────┬──────────────┐
│    Header    │        Data (Payload)  │    Trailer   │
│ (Addresses,  │  (Network Layer PDU)   │  (Error      │
│  control)    │                        │   check)     │
└──────────────┴────────────────────────┴──────────────┘
```

### 3.2 Framing Methods

**1. Fixed-Size Framing**
- Each frame has a predetermined, fixed length.
- No delimiter needed — frame boundaries are implicit.
- Simple but wastes space when data is smaller than the frame size.
- **Example:** ATM uses fixed 53-byte cells.

**2. Variable-Size Framing**

Because data does not always fit neatly into fixed sizes, variable-length frames are more common. They require a way to identify frame boundaries:

**a) Character count**
- A field in the header specifies the number of characters in the frame.
- Problem: If the count field is corrupted, frame synchronization is lost.

**b) Character (byte) stuffing**
- Special flag characters mark the start and end of a frame (e.g., FLAG = 01111110 in HDLC).
- If the flag byte appears in the data, an **escape character (ESC)** is inserted before it.
- The receiver removes escape characters to reconstruct the original data.

**c) Bit stuffing**
- Used in bit-oriented protocols (e.g., HDLC).
- Flag pattern is `01111110` (six consecutive 1s between two 0s).
- If five consecutive 1s appear in the data, a **0 is stuffed** after them.
- Receiver removes any 0 that follows five consecutive 1s.

---

## 4. Flow Control

**Flow control** prevents a fast sender from overwhelming a slow receiver by regulating the rate of data transmission.

### 4.1 Stop-and-Wait

- The sender transmits **one frame** and then **waits** for an acknowledgment (ACK) before sending the next.
- Simple and easy to implement.
- Very inefficient: the sender is idle while waiting for the ACK, wasting bandwidth.

```
Sender:    [Frame 0] ─────────────────────────► Receiver
                     ◄─── ACK 0 ─────────────── Receiver
Sender:    [Frame 1] ─────────────────────────► Receiver
                     ◄─── ACK 1 ─────────────── Receiver
```

**Efficiency:**
```
Efficiency = 1 / (1 + 2a)   where a = propagation delay / transmission time
```

On high-latency links (large `a`), Stop-and-Wait is very inefficient.

### 4.2 Sliding Window

- The sender can transmit **multiple frames** before waiting for an ACK — up to a **window size (W)** of unacknowledged frames.
- The window "slides" forward as ACKs are received.
- Dramatically improves efficiency on high-latency links.

```
Sender Window (W=4):
[Frame 0][Frame 1][Frame 2][Frame 3] ──► (can all be in transit simultaneously)
◄── ACK 0 (window slides: now can send Frame 4)
◄── ACK 1 (window slides: now can send Frame 5)
```

**Efficiency:**
```
If W ≥ 1 + 2a:   Efficiency = 1       (pipe is always full)
If W < 1 + 2a:   Efficiency = W / (1 + 2a)
```

The sliding window is the basis for the **Go-Back-N** and **Selective Repeat** ARQ protocols.

---

## 5. Error Detection and Correction

Transmission errors occur when bits are flipped (0→1 or 1→0) due to noise, attenuation, or distortion. The Data Link Layer must detect these errors.

### 5.1 Types of Errors

| Error Type | Description |
|---|---|
| **Single-bit error** | Only one bit in a data unit is changed |
| **Burst error** | Two or more consecutive bits are changed; more common in real networks |

### 5.2 Redundancy

Error detection works by adding **redundant bits** (extra bits calculated from the data) to the frame. The receiver recalculates these bits and compares them to the received redundancy — a mismatch indicates an error.

### 5.3 Parity Check

**Simple (single-bit) parity:**
- One parity bit is added so the total number of 1-bits is either even (even parity) or odd (odd parity).
- Detects **all single-bit errors**.
- Cannot detect errors where two bits are flipped (even number of errors).

**Two-dimensional (2D) parity:**
- Data is organized into a matrix; parity bits are added for each row and each column.
- Can detect **all 1-, 2-, and 3-bit errors** and some burst errors.
- Can also **correct single-bit errors** (the row and column parity together identify the position).

### 5.4 Checksum

- Used primarily at higher layers (UDP, IP) but conceptually part of error detection.
- Data is divided into segments; segments are summed using one's complement arithmetic.
- The **complement of the sum** is sent as the checksum.
- Receiver adds all segments including the checksum; result should be all 1s (no error).

**Advantages:** Simple and fast.
**Disadvantages:** Less powerful than CRC — burst errors may cancel out in the sum and go undetected.

### 5.5 Cyclic Redundancy Check (CRC)

CRC is the most powerful and widely used error-detection method at the Data Link Layer.

**Principle:**
- Treats the message as a large binary number (dividend).
- Divides it by a predetermined **generator polynomial** (divisor) using **modulo-2 division** (XOR arithmetic).
- The **remainder** (CRC bits / FCS – Frame Check Sequence) is appended to the message.
- Receiver divides the received frame (including CRC) by the same generator; a remainder of **0** indicates no error.

**Steps (Sender):**
1. Append r zeros to the message (where r = degree of the generator polynomial).
2. Divide the augmented message by the generator using modulo-2 division.
3. Replace the appended zeros with the remainder (CRC).

**Steps (Receiver):**
1. Divide the received frame (data + CRC) by the same generator.
2. If remainder = 0 → No error detected.
3. If remainder ≠ 0 → Error detected; frame is discarded.

**CRC Detection Power:**
- Detects **all single-bit errors**.
- Detects **all burst errors of length ≤ r** (where r = number of CRC bits).
- Detects **most burst errors of length > r** with very high probability.

**Common CRC Standards:**

| Standard | Generator Degree | Used In |
|---|---|---|
| CRC-8 | 8 bits | ATM header |
| CRC-16 | 16 bits | USB, Bluetooth |
| CRC-32 | 32 bits | Ethernet, ZIP, PNG |
| CRC-CCITT | 16 bits | HDLC, X.25 |

### 5.6 Error Correction: Hamming Code

**Error correction** (forward error correction / FEC) allows the receiver to **fix** errors without retransmission. This is important when retransmission is too costly (e.g., deep-space communication).

**Hamming Code:**
- Adds redundant bits at positions that are powers of 2 (positions 1, 2, 4, 8, ...).
- Each redundant bit covers a specific set of data bits.
- At the receiver, the positions of failing parity checks form a binary number that identifies the **exact bit position** of the error.

**Minimum number of redundancy bits (r)** for a data word of m bits:
```
2^r ≥ m + r + 1
```

**Example:** For m = 4 data bits → need r = 3 check bits (since 2³ = 8 ≥ 4 + 3 + 1 = 8).

---

## 6. Error Control Protocols

**Error control** combines error detection with retransmission requests (ARQ – Automatic Repeat reQuest). There are three main ARQ protocols:

### 6.1 Stop-and-Wait ARQ

- Sender transmits one frame and waits for ACK or NAK (negative acknowledgment).
- **ACK received** → Send the next frame.
- **NAK received or timeout** → Retransmit the current frame.
- Uses **1-bit sequence numbers** (0 and 1) to detect duplicate frames.

**Problem:** Very low efficiency due to idle wait time.

### 6.2 Go-Back-N ARQ (GBN)

- Uses a **sliding window** of size W = 2ⁿ − 1 (for n-bit sequence numbers).
- Sender can have up to W unacknowledged frames in transit.
- If an error is detected (or timeout occurs), the receiver **discards** the erroneous frame and all subsequent frames.
- Sender must **retransmit the erroneous frame and all frames that followed it** (go back to N).

**Advantages:** Simpler receiver — no need to buffer out-of-order frames.
**Disadvantages:** Retransmits correctly received frames unnecessarily.

**Window size:** Maximum window = 2ⁿ − 1 (to avoid ambiguity with sequence numbers).

### 6.3 Selective Repeat ARQ (SR)

- Uses a sliding window of size W = 2ⁿ⁻¹ (for n-bit sequence numbers).
- Only the **specific erroneous frame** is retransmitted.
- The receiver buffers correctly received out-of-order frames and reorders them after the missing frame arrives.

**Advantages:** Most efficient — minimal retransmissions.
**Disadvantages:** More complex receiver (needs buffer for out-of-order frames); smaller maximum window size.

### 6.4 ARQ Protocol Comparison

| Protocol | Window Size | Retransmission | Receiver Buffer | Efficiency |
|---|---|---|---|---|
| Stop-and-Wait | 1 | 1 frame | Not needed | Low |
| Go-Back-N | 2ⁿ − 1 | Erroneous + all after | Not needed | Medium |
| Selective Repeat | 2ⁿ⁻¹ | Erroneous frame only | Required | High |

---

## 7. Data Link Layer Sublayers: LLC and MAC

In IEEE standards (used in LANs), the Data Link Layer is divided into two sublayers:

```
┌─────────────────────────────────────┐ ← Network Layer (Layer 3)
│  LLC (Logical Link Control)         │  ← Upper DLL sublayer
├─────────────────────────────────────┤
│  MAC (Medium Access Control)        │  ← Lower DLL sublayer
└─────────────────────────────────────┘ ← Physical Layer (Layer 1)
```

### 7.1 Logical Link Control (LLC) — IEEE 802.2

- Provides a **uniform interface** to the Network Layer, independent of the underlying physical medium.
- Handles **flow control** and **error control** for the data link.
- Multiplexes different network protocols (IPv4, IPv6, etc.) over the same physical link.

### 7.2 Medium Access Control (MAC)

- Governs **which device can access the shared transmission medium** at a given time.
- Contains the **MAC address** (physical/hardware address) of the device.
- Specific to each LAN technology (Ethernet MAC ≠ Wi-Fi MAC).

### 7.3 MAC Addresses

A **MAC address** is a 48-bit (6-byte) hardware address burned into the NIC (Network Interface Card).

**Format:** `XX:XX:XX:XX:XX:XX` (hexadecimal, e.g., `00:1A:2B:3C:4D:5E`)

| Bits | Purpose |
|---|---|
| First 3 bytes (OUI) | Organizationally Unique Identifier — identifies the manufacturer |
| Last 3 bytes | Device-specific identifier assigned by the manufacturer |

**Special MAC addresses:**
- `FF:FF:FF:FF:FF:FF` — Broadcast address (sent to all devices on the local network)
- Multicast addresses start with `01:XX:XX:XX:XX:XX`

---

## 8. Multiple Access Protocols

When multiple devices share a single transmission medium, a **multiple access protocol** is needed to coordinate access and avoid collisions.

### 8.1 Categories

```
Multiple Access Protocols
├── Random Access (Contention-Based)
│   ├── ALOHA
│   ├── Slotted ALOHA
│   ├── CSMA
│   ├── CSMA/CD (Ethernet)
│   └── CSMA/CA (Wi-Fi)
├── Controlled Access
│   ├── Reservation
│   ├── Polling
│   └── Token Passing
└── Channelization
    ├── FDMA
    ├── TDMA
    └── CDMA
```

### 8.2 Random Access Protocols

In random access, any device can transmit at any time if the channel is free. If two devices transmit simultaneously, a **collision** occurs.

#### ALOHA

Developed at the University of Hawaii for radio networks.

**Pure ALOHA:**
- Any station transmits whenever it has data.
- If a collision occurs (no ACK received within timeout), the station waits a random time and retransmits.
- **Throughput:** Maximum ~18.4% of channel capacity.

**Slotted ALOHA:**
- Time is divided into equal slots; stations may only transmit at the start of a slot.
- Reduces the chance of partial overlaps.
- **Throughput:** Maximum ~36.8% of channel capacity.

#### CSMA (Carrier Sense Multiple Access)

- Before transmitting, a station **listens** (senses the carrier) to check if the channel is idle.
- Reduces collisions but cannot eliminate them (propagation delay means two stations may both sense the channel as idle and transmit simultaneously).

**CSMA Persistence Strategies:**

| Strategy | Description |
|---|---|
| **1-persistent** | If channel is idle, transmit immediately (high collision risk) |
| **Non-persistent** | If channel is busy, wait a random time and sense again |
| **p-persistent** | If channel is idle, transmit with probability p; wait with probability (1-p) |

#### CSMA/CD (Carrier Sense Multiple Access with Collision Detection)

Used in **wired Ethernet** (IEEE 802.3).

**Operation:**
1. Sense the channel. If idle, start transmitting.
2. While transmitting, **monitor the channel for collisions**.
3. If a collision is detected, send a **jam signal** to alert all stations.
4. Wait a **random backoff time** (binary exponential backoff) and retry.

**Binary Exponential Backoff:**
- After the 1st collision: wait 0 or 1 slot (random).
- After the 2nd collision: wait 0, 1, 2, or 3 slots.
- After the nth collision: wait 0 to 2ⁿ − 1 slots.
- After 16 failed attempts: the frame is discarded and an error is reported.

**Minimum frame size:** To detect collisions reliably, a frame must be long enough that its transmission time ≥ 2 × propagation delay (round-trip time). For 10 Mbps Ethernet, this is **64 bytes**.

#### CSMA/CA (Carrier Sense Multiple Access with Collision Avoidance)

Used in **wireless networks (Wi-Fi / IEEE 802.11)**.

Collision **detection** is not practical in wireless networks (a transmitting station cannot easily hear simultaneous transmissions from others). Instead, collisions are **avoided**:

**Operation:**
1. Sense the channel. If idle for a period called **DIFS** (Distributed Interframe Space), proceed.
2. Choose a random **backoff** time; count it down while the channel is idle.
3. Transmit the frame.
4. Wait for an ACK. If no ACK received (collision assumed), retry with longer backoff.

**Optional: RTS/CTS (Request to Send / Clear to Send)**
- Addresses the **hidden node problem** (two stations that can both reach the AP but not each other).
- Sender sends an **RTS** to the receiver; receiver replies with **CTS**.
- All nearby stations hearing the CTS defer transmission.

### 8.3 Controlled Access Protocols

**Polling:**
- A central **controller (primary station)** invites each station in turn to transmit.
- No collisions; efficient but creates a bottleneck at the primary station.

**Token Passing:**
- A special control frame called a **token** circulates around the network.
- A station may only transmit when it holds the token.
- **Token Ring (IEEE 802.5):** Stations arranged in a logical ring.
- No collisions; deterministic — maximum wait time is bounded.

### 8.4 Channelization Protocols

Divide the medium into channels (frequency, time, or code) assigned to individual stations. Covered in detail in Module 2 (FDM, TDM, CDMA).

---

## 9. Ethernet (IEEE 802.3)

**Ethernet** is the dominant wired LAN technology, standardized by IEEE as **802.3**.

### 9.1 Evolution of Ethernet

| Standard | Speed | Medium | Notes |
|---|---|---|---|
| 10BASE5 (Thicknet) | 10 Mbps | Thick coaxial | Original Ethernet |
| 10BASE2 (Thinnet) | 10 Mbps | Thin coaxial | Cheaper, more flexible |
| 10BASE-T | 10 Mbps | UTP Cat 3 | Introduced star topology |
| 100BASE-TX (Fast Ethernet) | 100 Mbps | UTP Cat 5 | CSMA/CD retained |
| 1000BASE-T (Gigabit Ethernet) | 1 Gbps | UTP Cat 5e/6 | Full-duplex; CSMA/CD rarely used |
| 10GBASE-T | 10 Gbps | UTP Cat 6a | Full-duplex; point-to-point switched |
| 40/100 GbE | 40/100 Gbps | Fiber or copper | Data center backbone |

### 9.2 Ethernet Frame Format (IEEE 802.3)

```
┌──────────┬────────┬──────────┬──────────┬──────────┬──────────┐
│ Preamble │  SFD   │  Dest.   │  Source  │  Length/ │   Data   │  Pad  │  FCS  │
│ 7 bytes  │ 1 byte │  MAC     │  MAC     │  Type    │ 46–1500  │       │ 4 bytes│
│          │        │ 6 bytes  │ 6 bytes  │  2 bytes │  bytes   │       │ (CRC) │
└──────────┴────────┴──────────┴──────────┴──────────┴──────────┘
```

| Field | Size | Description |
|---|---|---|
| **Preamble** | 7 bytes | Alternating 10101010 pattern; allows receiver to synchronize clock |
| **SFD** (Start Frame Delimiter) | 1 byte | `10101011` — signals the start of the frame |
| **Destination MAC** | 6 bytes | MAC address of the intended receiver |
| **Source MAC** | 6 bytes | MAC address of the sender |
| **Length/Type** | 2 bytes | If ≤ 1500: length of data field; if ≥ 1536: EtherType (e.g., 0x0800 = IPv4) |
| **Data and Padding** | 46–1500 bytes | The payload (Network Layer packet); padded to minimum 46 bytes |
| **FCS** (Frame Check Sequence) | 4 bytes | CRC-32 for error detection |

**Minimum frame size:** 64 bytes (preamble excluded) — ensures CSMA/CD collision detection works.
**Maximum frame size (MTU):** 1500 bytes of data — larger packets must be fragmented.

### 9.3 Ethernet Addressing

- Uses **MAC addresses** (48-bit) for source and destination.
- Frames with destination `FF:FF:FF:FF:FF:FF` are broadcast to all devices.
- Devices accept frames addressed to their own MAC, their multicast group, or broadcast.

---

## 10. Wireless LANs (IEEE 802.11 – Wi-Fi)

**IEEE 802.11** defines wireless LAN standards, commonly known as **Wi-Fi**.

### 10.1 Wi-Fi Standards

| Standard | Year | Frequency | Max Speed | Notes |
|---|---|---|---|---|
| 802.11b | 1999 | 2.4 GHz | 11 Mbps | First widespread Wi-Fi |
| 802.11a | 1999 | 5 GHz | 54 Mbps | Less interference; shorter range |
| 802.11g | 2003 | 2.4 GHz | 54 Mbps | Backward compatible with 802.11b |
| 802.11n (Wi-Fi 4) | 2009 | 2.4/5 GHz | 600 Mbps | Introduced MIMO |
| 802.11ac (Wi-Fi 5) | 2013 | 5 GHz | ~3.5 Gbps | MU-MIMO, wider channels |
| 802.11ax (Wi-Fi 6/6E) | 2019+ | 2.4/5/6 GHz | ~9.6 Gbps | OFDMA, improved dense environments |

### 10.2 Wi-Fi Architecture

**Infrastructure mode (BSS – Basic Service Set):**
- Devices (stations) communicate through an **Access Point (AP)**.
- Multiple BSSs can be connected via a **Distribution System (DS)** to form an **Extended Service Set (ESS)**.
- Most home and enterprise networks use this mode.

**Ad-hoc mode (IBSS – Independent BSS):**
- Devices communicate **directly** with each other (peer-to-peer).
- No AP required; used for temporary, small networks.

### 10.3 IEEE 802.11 Frame Format

The Wi-Fi frame is more complex than Ethernet due to wireless-specific requirements:

| Field | Description |
|---|---|
| **Frame Control** | 2 bytes; type, subtype, flags (ToDS, FromDS, retry, etc.) |
| **Duration** | Time the channel will be used (for virtual carrier sensing via NAV) |
| **Address 1** | Destination MAC (receiver) |
| **Address 2** | Source MAC (transmitter) |
| **Address 3** | Filtering address (e.g., AP's BSSID or router MAC) |
| **Sequence Control** | Fragment and sequence numbers |
| **Address 4** | Used only in WDS (wireless distribution system) |
| **Data** | Payload |
| **FCS** | CRC-32 for error detection |

### 10.4 CSMA/CA and Hidden Node Problem

Wi-Fi uses **CSMA/CA** (covered in Section 8.2). The **hidden node problem** occurs when:

- Station A and Station C can both reach AP (B) but cannot hear each other.
- A and C may transmit simultaneously, causing a collision at B.
- Solution: **RTS/CTS handshake** — stations overhearing the CTS defer transmission.

### 10.5 Wi-Fi Security

| Protocol | Status | Description |
|---|---|---|
| **WEP** (Wired Equivalent Privacy) | Deprecated (broken) | RC4 encryption; easily cracked |
| **WPA** (Wi-Fi Protected Access) | Legacy | Temporary fix for WEP; TKIP encryption |
| **WPA2** | Standard | AES-CCMP encryption; most widely used |
| **WPA3** | Current best | SAE handshake; stronger individual encryption |

---

## 11. Point-to-Point Protocol (PPP)

**PPP** is a Data Link Layer protocol used for **point-to-point links** — direct connections between exactly two nodes (e.g., dial-up, DSL, and many WAN links).

### 11.1 PPP Services

- **Framing:** Defines the frame format for encapsulating Network Layer packets.
- **Authentication:** Supports PAP (Password Authentication Protocol) and CHAP (Challenge Handshake Authentication Protocol).
- **Network Layer multiplexing:** Can carry multiple Network Layer protocols (IP, IPv6, IPX) over the same link using the NCP (Network Control Protocol).
- **Link management:** LCP (Link Control Protocol) establishes, configures, and terminates connections.

### 11.2 PPP Frame Format

```
┌──────┬──────────┬──────────┬──────────────┬────────────┬──────┐
│ Flag │ Address  │ Control  │  Protocol    │    Data    │  FCS │
│ 1 B  │  1 B     │  1 B     │   1–2 B      │ Variable   │ 2–4 B│
└──────┴──────────┴──────────┴──────────────┴────────────┴──────┘
```

- **Flag:** `01111110` — marks frame boundaries.
- **Address:** Always `11111111` (broadcast) — PPP does not use MAC addresses.
- **Protocol:** Identifies the Network Layer protocol of the payload (e.g., 0x0021 = IP).
- **FCS:** CRC for error detection.

---

## 12. Switching at the Data Link Layer

### 12.1 Hubs vs. Bridges vs. Switches

| Device | OSI Layer | Function |
|---|---|---|
| **Hub** | Layer 1 | Broadcasts all received bits to all ports; all ports share one collision domain |
| **Bridge** | Layer 2 | Connects two LAN segments; filters traffic based on MAC addresses |
| **Switch** | Layer 2 | Multi-port bridge; creates a separate collision domain per port; uses MAC table |

### 12.2 How a Switch Works

A switch builds and maintains a **MAC address table** (also called a CAM table or forwarding table) that maps MAC addresses to switch ports.

**Learning process:**
1. When a frame arrives on port X from MAC address A, the switch records: `MAC A → Port X`.
2. The switch looks up the destination MAC in its table.
3. If found → **forward** the frame only to the correct port (unicast forwarding).
4. If not found → **flood** the frame to all ports except the incoming port (unknown unicast flooding).
5. Broadcast frames are always flooded to all ports.

**Benefits of switching:**
- Each port is its own **collision domain** — eliminates collisions between devices on different ports.
- Allows **simultaneous conversations** (e.g., A↔B and C↔D at the same time).
- Higher aggregate throughput than hub-based networks.

### 12.3 Spanning Tree Protocol (STP – IEEE 802.1D)

In networks with redundant switch links (for fault tolerance), **loops** can form. A loop causes **broadcast storms** — frames circle forever, consuming all bandwidth.

**STP** automatically detects and eliminates loops by:
1. Electing a **Root Bridge** (switch with lowest Bridge ID).
2. Computing the **shortest path** from each switch to the Root Bridge.
3. Placing **redundant ports into a blocking state** — they do not forward traffic but can be activated if the active path fails.

**STP port states:** Blocking → Listening → Learning → Forwarding → Disabled.

---

## 13. Virtual LANs (VLANs)

### 13.1 What is a VLAN?

A **VLAN (Virtual LAN)** is a logical grouping of devices on one or more switches, regardless of their physical location, that behave as if they were on the same LAN segment.

Without VLANs, all devices on all switch ports belong to the same broadcast domain — every broadcast is sent to every device.

### 13.2 Benefits of VLANs

| Benefit | Description |
|---|---|
| **Broadcast control** | Broadcasts are limited to the VLAN; reduces unnecessary traffic |
| **Security** | Devices in different VLANs cannot communicate directly without a router |
| **Flexibility** | Users can be grouped logically (e.g., by department) regardless of physical location |
| **Performance** | Smaller broadcast domains reduce congestion |

### 13.3 VLAN Tagging (IEEE 802.1Q)

When frames travel between switches (on **trunk links**), they need to carry VLAN information. **IEEE 802.1Q** adds a **4-byte VLAN tag** inside the Ethernet frame:

```
┌────────────┬─────────┬──────────┬──────────┬─────────┬──────┬─────┐
│  Dest MAC  │ Src MAC │  0x8100  │  TCI     │  Type   │ Data │ FCS │
│  6 bytes   │ 6 bytes │ (2 B)    │ (2 B)    │         │      │     │
│            │         │ Tag Type │ PCP+DEI+ │         │      │     │
│            │         │          │  VID(12b)│         │      │     │
└────────────┴─────────┴──────────┴──────────┴─────────┴──────┴─────┘
```

- **VID (VLAN ID):** 12-bit field; supports up to **4094 VLANs** (0 and 4095 are reserved).
- **Access ports:** Connect to end devices; no VLAN tag (device unaware of VLAN).
- **Trunk ports:** Connect switches to each other; carry tagged frames for multiple VLANs.

### 13.4 Inter-VLAN Routing

Devices in different VLANs cannot communicate at Layer 2 — they need a **router** or a **Layer 3 switch** to route packets between VLANs. This is called **inter-VLAN routing**.

---

## 14. Summary

This module explored the Data Link Layer — the second layer of the OSI model:

- The **Data Link Layer** is responsible for framing, physical addressing, flow control, error control, and medium access control.
- **Framing** packages bits into structured units using character stuffing or bit stuffing to delimit variable-length frames.
- **Flow control** (Stop-and-Wait, Sliding Window) prevents receiver overflow.
- **Error detection** (parity, checksum, CRC) identifies corrupted frames; **CRC-32** is the standard for Ethernet.
- **Error correction** (Hamming code) allows receivers to fix errors without retransmission.
- **ARQ protocols** (Stop-and-Wait, Go-Back-N, Selective Repeat) combine error detection with retransmission for reliable delivery.
- The DLL is divided into **LLC** (interface to Network Layer) and **MAC** (medium access control) sublayers.
- **CSMA/CD** governs wired Ethernet; **CSMA/CA** governs Wi-Fi; both detect or avoid collisions on shared media.
- **Ethernet (IEEE 802.3)** uses a 64–1518 byte frame with CRC-32 and MAC addressing.
- **Wi-Fi (IEEE 802.11)** uses CSMA/CA with an optional RTS/CTS mechanism for the hidden node problem.
- **Switches** operate at Layer 2, build MAC address tables, and create separate collision domains per port; **STP** prevents loops.
- **VLANs** (IEEE 802.1Q) logically segment networks for broadcast control, security, and flexibility.

---

## 15. Key Terms

| Term | Definition |
|---|---|
| **Data Link Layer** | OSI Layer 2; handles node-to-node framing, addressing, and error control |
| **Frame** | A structured unit of data at the Data Link Layer |
| **MAC Address** | 48-bit hardware address that identifies a device on a local network |
| **Flow control** | Mechanism to prevent a fast sender from overwhelming a slow receiver |
| **Sliding window** | Flow control allowing multiple unacknowledged frames in transit |
| **CRC** | Cyclic Redundancy Check; a powerful polynomial-based error detection method |
| **Hamming Code** | An error-correction code that identifies and fixes single-bit errors |
| **ARQ** | Automatic Repeat reQuest; error control using retransmission |
| **Go-Back-N** | ARQ protocol that retransmits erroneous frame and all subsequent frames |
| **Selective Repeat** | ARQ protocol that retransmits only the erroneous frame |
| **LLC** | Logical Link Control; upper DLL sublayer providing interface to Network Layer |
| **MAC** | Medium Access Control; lower DLL sublayer governing channel access |
| **CSMA/CD** | Carrier Sense Multiple Access with Collision Detection; used in wired Ethernet |
| **CSMA/CA** | Carrier Sense Multiple Access with Collision Avoidance; used in Wi-Fi |
| **Ethernet** | Most common wired LAN technology; IEEE 802.3 standard |
| **IEEE 802.11** | Wireless LAN standard; commonly known as Wi-Fi |
| **Switch** | Layer 2 device that forwards frames based on MAC address table |
| **STP** | Spanning Tree Protocol; prevents loops in switched networks |
| **VLAN** | Virtual LAN; logical grouping of devices independent of physical location |
| **IEEE 802.1Q** | Standard for VLAN tagging on trunk links |
| **Hidden node problem** | Wi-Fi issue where two stations cannot hear each other but both reach the AP |
| **PPP** | Point-to-Point Protocol; Data Link protocol for direct two-node links |

---

## 16. Review Questions

**Conceptual Questions:**

1. What are the five main responsibilities of the Data Link Layer? How do they differ from the responsibilities of the Physical Layer?
2. Explain the difference between character stuffing and bit stuffing. Why is bit stuffing preferred in bit-oriented protocols?
3. Compare Stop-and-Wait and Sliding Window flow control. What is the efficiency formula for each, and what does the variable `a` represent?
4. A sender uses Go-Back-N ARQ with a window size of 7 (3-bit sequence numbers). Frame 4 is corrupted. Which frames must be retransmitted? Why?
5. Why is CRC more powerful than a simple parity check for detecting burst errors? What does the generator polynomial determine?
6. Explain the difference between the LLC and MAC sublayers. Why is this division useful?
7. What is the hidden node problem in Wi-Fi? How does RTS/CTS resolve it?
8. How does a switch build and use its MAC address table? What happens when a destination MAC address is not yet in the table?
9. Why is Spanning Tree Protocol (STP) necessary in switched networks? Describe the three main steps STP uses to eliminate loops.
10. What is a VLAN? What are the benefits of using VLANs? How does IEEE 802.1Q enable VLAN trunking?

**Application / Calculation Questions:**

11. Using CRC, divide the message `110101` by the generator `1011` (use modulo-2 division). What is the CRC remainder? What is the transmitted frame?
12. A data word has m = 7 bits. How many Hamming redundancy bits are required for single-bit error correction? What are the positions of the redundancy bits in the codeword?
13. A sliding window protocol uses 4-bit sequence numbers with Go-Back-N ARQ. What is the maximum window size? If the propagation delay is 10 ms and the transmission time for one frame is 1 ms, what is the efficiency?
14. A switch receives a frame on port 3 with source MAC `AA:BB:CC:DD:EE:01` and destination MAC `AA:BB:CC:DD:EE:05`. The MAC table contains no entries. What does the switch do? What entry is now added to the table?
15. An organization has 3 departments (Sales, Engineering, HR) sharing a single 24-port switch. Describe how VLANs could be configured to isolate department traffic, and explain what is needed for the departments to communicate with each other.

---

*End of Module 3 | Next: Module 4 – Network Layer*
