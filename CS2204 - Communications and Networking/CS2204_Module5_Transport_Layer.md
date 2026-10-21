# CS 2204 – Communications and Networking
## Module 5: Transport Layer

---

## Table of Contents

1. [Overview](#1-overview)
2. [Transport Layer Responsibilities](#2-transport-layer-responsibilities)
3. [Port Numbers and Addressing](#3-port-numbers-and-addressing)
4. [User Datagram Protocol (UDP)](#4-user-datagram-protocol-udp)
5. [Transmission Control Protocol (TCP)](#5-transmission-control-protocol-tcp)
6. [TCP Connection Management](#6-tcp-connection-management)
7. [TCP Flow Control](#7-tcp-flow-control)
8. [TCP Congestion Control](#8-tcp-congestion-control)
9. [TCP Error Control](#9-tcp-error-control)
10. [TCP vs. UDP Comparison](#10-tcp-vs-udp-comparison)
11. [Transport Layer Security (TLS)](#11-transport-layer-security-tls)
12. [SCTP (Stream Control Transmission Protocol)](#12-sctp-stream-control-transmission-protocol)
13. [Socket Programming Concepts](#13-socket-programming-concepts)
14. [Summary](#14-summary)
15. [Key Terms](#15-key-terms)
16. [Review Questions](#16-review-questions)

---

## 1. Overview

The Transport Layer provides **end-to-end communication services** between application processes running on different hosts. While the Network Layer moves packets from host to host, the Transport Layer ensures that the correct application on each host sends and receives the right data. It is the layer that transforms raw network delivery into a reliable (or fast) communication service for applications.

**Module Learning Objectives:**

By the end of this module, students should be able to:

- Describe the responsibilities of the Transport Layer and how it differs from the Network Layer.
- Explain how port numbers enable process-to-process communication.
- Describe the UDP segment format and identify applications that use UDP.
- Describe the TCP segment format and explain each field.
- Explain TCP connection establishment (three-way handshake) and termination (four-way handshake).
- Explain TCP flow control using the sliding window and receive buffer.
- Describe TCP congestion control mechanisms: Slow Start, Congestion Avoidance, Fast Retransmit, and Fast Recovery.
- Explain TCP error control through sequence numbers, ACKs, and retransmission timers.
- Compare TCP and UDP and justify the choice of each for different applications.
- Describe the role of TLS in securing Transport Layer communications.

---

## 2. Transport Layer Responsibilities

The **Transport Layer (OSI Layer 4)** sits between the Network Layer (Layer 3) and the Session Layer (Layer 5). Its job is to provide **process-to-process** (application-to-application) communication.

### 2.1 Core Responsibilities

| Responsibility | Description |
|---|---|
| **Process-to-process delivery** | Uses port numbers to direct data to the correct application process on a host |
| **Segmentation and reassembly** | Breaks large application messages into segments for transmission; reassembles at the destination |
| **Connection management** | Establishes, maintains, and terminates logical connections (TCP only) |
| **Flow control** | Prevents a fast sender from overwhelming a slow receiver (TCP only) |
| **Error control** | Detects lost or corrupted segments and requests retransmission (TCP only) |
| **Congestion control** | Reduces transmission rate when the network is congested (TCP only) |
| **Multiplexing/Demultiplexing** | Multiple applications can share the network simultaneously using different port numbers |

### 2.2 Network Layer vs. Transport Layer

| Feature | Network Layer (Layer 3) | Transport Layer (Layer 4) |
|---|---|---|
| Delivery scope | Host-to-host | Process-to-process |
| Addressing | IP addresses | Port numbers |
| Unit of data | Packet | Segment (TCP) / Datagram (UDP) |
| Reliability | Best-effort (IP) | Optional (TCP = reliable; UDP = unreliable) |
| Path awareness | Yes (routing) | No |

### 2.3 Multiplexing and Demultiplexing

**Multiplexing (at sender):** Multiple application processes simultaneously pass data down to the Transport Layer. Each segment is tagged with a source port number to distinguish which application it came from.

**Demultiplexing (at receiver):** The Transport Layer receives incoming segments and delivers each to the correct application process based on the destination port number.

```
Application A (Port 5000) ─┐
Application B (Port 5001) ─┤──► [Transport Layer MUX] ──► Network
Application C (Port 5002) ─┘

Network ──► [Transport Layer DEMUX] ──► Port 5000 → App A
                                   ──► Port 5001 → App B
                                   ──► Port 5002 → App C
```

---

## 3. Port Numbers and Addressing

### 3.1 What is a Port Number?

A **port number** is a 16-bit integer (0–65535) that identifies a specific process or service on a host. Combined with an IP address, a port number forms a **socket address** that uniquely identifies a communication endpoint.

```
Socket = IP Address + Port Number
Example: 192.168.1.10 : 80  (web server on host 192.168.1.10)
```

A **connection** between two processes is uniquely identified by the **5-tuple**:
```
{ Protocol, Source IP, Source Port, Destination IP, Destination Port }
```

### 3.2 Port Number Ranges

| Range | Name | Description |
|---|---|---|
| 0 – 1023 | **Well-known ports** | Assigned by IANA to standard services; require root/admin privileges |
| 1024 – 49151 | **Registered ports** | Assigned by IANA for specific applications; available to any process |
| 49152 – 65535 | **Dynamic/Ephemeral ports** | Assigned temporarily by the OS to client processes |

### 3.3 Common Well-Known Port Numbers

| Port | Protocol | Service |
|---|---|---|
| 20 | TCP | FTP Data |
| 21 | TCP | FTP Control |
| 22 | TCP | SSH (Secure Shell) |
| 23 | TCP | Telnet |
| 25 | TCP | SMTP (Email sending) |
| 53 | UDP/TCP | DNS (Domain Name System) |
| 67/68 | UDP | DHCP (Server/Client) |
| 80 | TCP | HTTP (Web) |
| 110 | TCP | POP3 (Email retrieval) |
| 143 | TCP | IMAP (Email retrieval) |
| 443 | TCP | HTTPS (Secure Web) |
| 520 | UDP | RIP |
| 3389 | TCP | RDP (Remote Desktop) |

---

## 4. User Datagram Protocol (UDP)

### 4.1 What is UDP?

**UDP (User Datagram Protocol)**, defined in **RFC 768**, is a **connectionless, unreliable** Transport Layer protocol. It provides minimal service: it segments application data, adds source/destination port numbers, and passes the datagram to the Network Layer — with no guarantees of delivery, ordering, or error recovery.

### 4.2 UDP Segment Format

The UDP header is extremely simple — only **8 bytes**:

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
├─────────────────────────────────┬─────────────────────────────────┤
│          Source Port            │       Destination Port          │
├─────────────────────────────────┼─────────────────────────────────┤
│             Length              │            Checksum             │
├─────────────────────────────────┴─────────────────────────────────┤
│                            Data (variable)                        │
└───────────────────────────────────────────────────────────────────┘
```

| Field | Size | Description |
|---|---|---|
| **Source Port** | 16 bits | Port number of the sending application (optional; set to 0 if not used) |
| **Destination Port** | 16 bits | Port number of the receiving application |
| **Length** | 16 bits | Total length of the UDP datagram (header + data) in bytes; minimum = 8 |
| **Checksum** | 16 bits | Optional in IPv4 (mandatory in IPv6); covers header, data, and pseudo-header |

**Maximum UDP datagram size:** 65,535 bytes (limited by the 16-bit Length field), though practical size is constrained by IP MTU.

### 4.3 UDP Characteristics

| Feature | UDP Behavior |
|---|---|
| Connection | Connectionless — no setup or teardown |
| Reliability | None — no ACKs, no retransmission |
| Ordering | None — datagrams may arrive out of order |
| Flow control | None |
| Congestion control | None |
| Speed | Fast — minimal overhead |
| Header size | 8 bytes |

### 4.4 When to Use UDP

UDP is preferred when:

- **Speed matters more than reliability** — lost packets are tolerable.
- **Real-time communication** — retransmission would arrive too late to be useful.
- **Application handles its own reliability** — some apps implement their own error control.
- **Broadcast/multicast** — TCP is point-to-point; UDP supports one-to-many.

**Applications using UDP:**

| Application | Reason |
|---|---|
| **DNS** | Short request/response; retransmission handled by the app |
| **DHCP** | Bootstrap protocol; no prior connection possible |
| **VoIP / Video conferencing** | Real-time; dropped packets preferable to delay |
| **Live video streaming** | Slight loss acceptable; low latency critical |
| **Online gaming** | Frequent small updates; stale data is useless |
| **SNMP** | Simple management queries; high overhead not justified |
| **TFTP** | Simple file transfer with its own reliability layer |
| **RIP** | Routing updates over trusted internal networks |

---

## 5. Transmission Control Protocol (TCP)

### 5.1 What is TCP?

**TCP (Transmission Control Protocol)**, defined in **RFC 793**, is a **connection-oriented, reliable** Transport Layer protocol. It provides a guaranteed, ordered, error-checked byte stream between two applications.

TCP achieves reliability through:
- **Sequence numbers** — to detect missing or out-of-order segments.
- **Acknowledgments (ACKs)** — to confirm receipt of data.
- **Retransmission** — to recover from lost segments.
- **Flow control** — to match sender rate to receiver capacity.
- **Congestion control** — to adapt to network conditions.

### 5.2 TCP Segment Format

The TCP header is a minimum of **20 bytes** (up to 60 bytes with options):

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
├─────────────────────────────────┬─────────────────────────────────┤
│          Source Port            │       Destination Port          │
├─────────────────────────────────┴─────────────────────────────────┤
│                        Sequence Number                            │
├───────────────────────────────────────────────────────────────────┤
│                     Acknowledgment Number                         │
├─────────┬───────────┬─────────────────────────────────────────────┤
│  Data   │ Reserved  │  Control Flags   │      Window Size         │
│ Offset  │           │URG ACK PSH RST SYN FIN                     │
├─────────────────────────────────┬─────────────────────────────────┤
│            Checksum             │         Urgent Pointer          │
├───────────────────────────────────────────────────────────────────┤
│                    Options (if Data Offset > 5)                   │
└───────────────────────────────────────────────────────────────────┘
```

### 5.3 TCP Header Field Descriptions

| Field | Size | Description |
|---|---|---|
| **Source Port** | 16 bits | Sending application's port number |
| **Destination Port** | 16 bits | Receiving application's port number |
| **Sequence Number** | 32 bits | Byte number of the first byte in this segment's data; or ISN during SYN |
| **Acknowledgment Number** | 32 bits | Next byte number the receiver expects (ACKs all bytes up to this − 1) |
| **Data Offset (HLEN)** | 4 bits | Header length in 32-bit words; minimum = 5 (20 bytes) |
| **Reserved** | 6 bits | Must be zero |
| **URG** | 1 bit | Urgent pointer field is valid |
| **ACK** | 1 bit | Acknowledgment number field is valid |
| **PSH** | 1 bit | Push: deliver data to application immediately without buffering |
| **RST** | 1 bit | Reset: abort the connection immediately |
| **SYN** | 1 bit | Synchronize sequence numbers (used during connection setup) |
| **FIN** | 1 bit | Finish: sender has no more data to send (used during connection teardown) |
| **Window Size** | 16 bits | Size of the receiver's buffer (in bytes); used for flow control |
| **Checksum** | 16 bits | Error detection over header, data, and pseudo-header (mandatory in TCP) |
| **Urgent Pointer** | 16 bits | Offset from sequence number to last urgent byte (only used when URG = 1) |
| **Options** | Variable | Common options: MSS (Maximum Segment Size), Window Scale, SACK, Timestamps |

### 5.4 TCP Sequence and Acknowledgment Numbers

TCP treats data as a **continuous byte stream**. Sequence numbers track individual bytes:

- **Sequence Number (SEQ):** The byte offset of the first byte of data in this segment.
- **Acknowledgment Number (ACK):** The next byte the receiver expects — implicitly acknowledges all bytes before this number (**cumulative acknowledgment**).

**Example:**

```
Sender sends bytes 0–999:    SEQ = 0,   data = 1000 bytes
Receiver responds:           ACK = 1000  (expecting byte 1000 next)

Sender sends bytes 1000–1999: SEQ = 1000, data = 1000 bytes
Receiver responds:            ACK = 2000  (expecting byte 2000 next)
```

The **Initial Sequence Number (ISN)** is a random number chosen during the SYN step of the three-way handshake — preventing conflicts with old connections.

---

## 6. TCP Connection Management

### 6.1 Three-Way Handshake (Connection Establishment)

Before data transfer, TCP establishes a connection using a **three-way handshake**:

```
Client                                    Server
──────                                    ──────
SYN (SEQ=x)              ───────────────►
                         ◄─────────────── SYN-ACK (SEQ=y, ACK=x+1)
ACK (ACK=y+1)            ───────────────►
          ── Connection Established ──
```

| Step | Segment | Description |
|---|---|---|
| **Step 1** | SYN | Client sends SYN with its Initial Sequence Number (ISN = x). SYN flag = 1 |
| **Step 2** | SYN-ACK | Server acknowledges client's ISN (ACK = x+1) and sends its own ISN (SEQ = y). SYN = 1, ACK = 1 |
| **Step 3** | ACK | Client acknowledges server's ISN (ACK = y+1). Connection is now open |

**Purpose of the three-way handshake:**
- Both parties agree on initial sequence numbers.
- Both parties confirm the other is reachable and willing to communicate.
- Both parties synchronize their receive windows.

### 6.2 SYN Flooding Attack

A **SYN flood** is a DoS attack where an attacker sends many SYN segments with fake source IPs. The server allocates resources and sends SYN-ACKs, but never receives the final ACK — exhausting the server's connection table.

**Mitigation:** SYN cookies — the server does not allocate resources until the ACK is received; uses a cryptographic token (the ISN) to verify the client.

### 6.3 Four-Way Handshake (Connection Termination)

TCP is **full-duplex** — each direction must be closed independently. Termination uses a **four-way handshake**:

```
Client                                    Server
──────                                    ──────
FIN (SEQ=x)              ───────────────►   (Client done sending)
                         ◄─────────────── ACK (ACK=x+1)
                         ◄─────────────── FIN (SEQ=y)  (Server done sending)
ACK (ACK=y+1)            ───────────────►
        ── Connection Terminated ──
```

| Step | Segment | Description |
|---|---|---|
| **Step 1** | FIN | Active closer (client) sends FIN — it has no more data to send |
| **Step 2** | ACK | Passive closer (server) acknowledges the FIN; may still send data |
| **Step 3** | FIN | Server sends its own FIN when it has no more data |
| **Step 4** | ACK | Client acknowledges server's FIN |

**TIME_WAIT state:** After sending the final ACK, the client waits in **TIME_WAIT** for **2 × MSL** (Maximum Segment Lifetime, typically 60–120 seconds) before fully closing. This ensures:
- The final ACK reaches the server (in case it is lost and the server retransmits FIN).
- Old duplicate segments from the previous connection expire before a new connection reuses the same port.

### 6.4 TCP State Machine (Summary)

Key TCP states:

| State | Description |
|---|---|
| **CLOSED** | No connection exists |
| **LISTEN** | Server waiting for incoming SYN |
| **SYN_SENT** | Client sent SYN; waiting for SYN-ACK |
| **SYN_RECEIVED** | Server received SYN; sent SYN-ACK; waiting for ACK |
| **ESTABLISHED** | Connection open; data transfer in progress |
| **FIN_WAIT_1** | Sent FIN; waiting for ACK |
| **FIN_WAIT_2** | Received ACK of FIN; waiting for server's FIN |
| **TIME_WAIT** | Waiting 2×MSL before fully closing |
| **CLOSE_WAIT** | Received FIN; waiting for local app to close |
| **LAST_ACK** | Sent FIN; waiting for final ACK |

---

## 7. TCP Flow Control

### 7.1 The Receive Buffer and Window Size

TCP uses a **sliding window** mechanism for flow control. Each side maintains a **receive buffer** — a block of memory that holds incoming data until the application reads it.

The **Receiver Window (rwnd)** in the TCP header tells the sender how many bytes the receiver can currently accept:

```
Allowed to send = min(cwnd, rwnd)
```

Where **cwnd** = congestion window (set by congestion control) and **rwnd** = receiver's advertised window.

### 7.2 Window Size Operation

```
Receiver Buffer: [  Used  |  Available (rwnd)  ]
                  ◄──────► ◄──────────────────►
               Data not    Space sender can fill
               yet read

Sender can transmit up to rwnd bytes beyond the last acknowledged byte.
```

**Window size = 0 (Zero Window):**
When the receiver's buffer is full, it sends a segment with **window = 0**, telling the sender to stop transmitting. The sender then periodically sends **Window Probe** segments (1 byte) to check when the window reopens.

### 7.3 Silly Window Syndrome

**Silly Window Syndrome (SWS)** occurs when small amounts of data are exchanged, wasting bandwidth on TCP/IP overhead.

**On the receiver side:** The receiver advertises a small window when only a few bytes are freed.
- **Solution (Clark's Algorithm):** Receiver waits until the window is at least **min(MSS, half of buffer)** before advertising it.

**On the sender side:** The sender sends tiny segments instead of waiting to accumulate more data.
- **Solution (Nagle's Algorithm):** The sender buffers small data and sends it as one large segment once:
  - The sender has accumulated at least **MSS** bytes, OR
  - All previously sent data has been acknowledged (the pipe is empty).

---

## 8. TCP Congestion Control

### 8.1 What is Network Congestion?

**Congestion** occurs when the volume of data injected into the network exceeds what the network can handle — routers drop packets, delays increase, and throughput collapses. TCP must detect and respond to congestion without explicit signals from routers (in most cases).

**TCP interprets packet loss as a signal of congestion** and reduces its transmission rate accordingly.

### 8.2 The Congestion Window (cwnd)

TCP maintains a **congestion window (cwnd)** at the sender, which limits how much data can be in transit:

```
Effective window = min(cwnd, rwnd)
Bytes in flight ≤ min(cwnd, rwnd)
```

TCP adjusts **cwnd** based on network feedback.

### 8.3 Slow Start

When a new connection is established (or after a timeout), TCP begins in **Slow Start** mode:

- **Initial cwnd:** Typically 1–10 MSS (Maximum Segment Size).
- **Growth:** cwnd doubles every **RTT** (exponential growth — one new segment is added for every ACK received).
- **Threshold (ssthresh):** Slow Start continues until cwnd reaches the **Slow Start Threshold (ssthresh)**.

```
RTT 1: cwnd = 1 MSS  →  send 1 segment
RTT 2: cwnd = 2 MSS  →  send 2 segments
RTT 3: cwnd = 4 MSS  →  send 4 segments
RTT 4: cwnd = 8 MSS  →  ...
(continues until cwnd ≥ ssthresh)
```

> Despite the name "Slow Start," the window grows exponentially — it starts slow but accelerates quickly.

### 8.4 Congestion Avoidance

Once **cwnd ≥ ssthresh**, TCP switches to **Congestion Avoidance** mode:

- **Growth:** cwnd increases by **1 MSS per RTT** (linear/additive growth).
- This is the **AIMD (Additive Increase, Multiplicative Decrease)** phase.

```
RTT n:   cwnd = 8 MSS
RTT n+1: cwnd = 9 MSS
RTT n+2: cwnd = 10 MSS
...
```

### 8.5 Detecting Congestion: Loss Events

TCP detects two types of loss events and responds differently:

#### Triple Duplicate ACKs (3 DupACKs)

When the receiver gets an out-of-order segment, it sends **duplicate ACKs** for the last in-order segment received. Three duplicate ACKs suggest a segment was lost, but subsequent segments are still arriving — the network is still functional.

**Response (Fast Retransmit + Fast Recovery):**
1. Retransmit the lost segment immediately (without waiting for timeout).
2. Set `ssthresh = cwnd / 2`.
3. Set `cwnd = ssthresh + 3 MSS`.
4. Continue in **Congestion Avoidance** (not back to Slow Start).

#### Timeout (RTO expiry)

A timeout suggests severe congestion — no segments are getting through.

**Response:**
1. Set `ssthresh = cwnd / 2`.
2. Set `cwnd = 1 MSS`.
3. Re-enter **Slow Start** (more aggressive reduction).

### 8.6 TCP Congestion Control Summary

```
                    cwnd
                      │
                      │        /
                      │       /  (Congestion Avoidance: +1 MSS/RTT)
           ssthresh ──┼──────/─────────────────── 3 DupACK → ssthresh=cwnd/2
                      │     /                                  cwnd=ssthresh+3
                      │    /  (Slow Start: ×2/RTT)
                      │   /
                      │  /
                      │ /
                    1 ┼─────────────────────────────────────► RTT
                      0
```

### 8.7 TCP Variants and Modern Congestion Control

| Variant | Innovation |
|---|---|
| **TCP Tahoe** | Original: Slow Start, Congestion Avoidance, Fast Retransmit; goes to cwnd=1 on 3 DupACK |
| **TCP Reno** | Added Fast Recovery: doesn't drop to cwnd=1 on 3 DupACK (stays at ssthresh) |
| **TCP NewReno** | Improved Fast Recovery for multiple losses in one window |
| **TCP CUBIC** | Default in Linux; uses cubic function for cwnd growth; better for high-bandwidth-delay networks |
| **QUIC** | Google's UDP-based transport with built-in TLS; used in HTTP/3 |
| **BBR** | Bottleneck Bandwidth and RTT; Google's model-based congestion control; avoids buffer bloat |

---

## 9. TCP Error Control

### 9.1 Mechanisms for Reliable Delivery

TCP uses four mechanisms together to achieve reliable, ordered delivery:

**1. Sequence Numbers**
- Every byte of data has a unique sequence number.
- Allows the receiver to detect missing bytes, reorder out-of-order segments, and discard duplicates.

**2. Acknowledgment Numbers**
- Cumulative ACKs confirm receipt of all bytes up to ACK−1.
- If a segment is lost, the receiver continues to ACK the last in-order byte received, triggering duplicate ACKs at the sender.

**3. Retransmission Timer (RTO)**
- Each sent segment starts a retransmission timer.
- If the ACK is not received before the timer expires, the segment is retransmitted.
- **RTO** is dynamically calculated based on measured Round-Trip Time (RTT):

```
SRTT = (1 − α) × SRTT + α × RTT_sample         (smoothed RTT, α ≈ 0.125)
RTTVAR = (1 − β) × RTTVAR + β × |SRTT − RTT_sample|   (RTT variance, β ≈ 0.25)
RTO = SRTT + 4 × RTTVAR
```

- **Karn's Algorithm:** Do not use RTT samples from retransmitted segments (ambiguity about which transmission was ACKed); double RTO on each consecutive timeout (**exponential backoff**).

**4. Checksum**
- The TCP checksum detects corrupted segments; corrupted segments are silently discarded (the timeout then triggers retransmission).

### 9.2 Selective Acknowledgment (SACK)

Standard TCP uses **cumulative ACKs** — if segment 3 is lost but segments 4, 5, and 6 arrive, the receiver continues ACKing segment 2. The sender doesn't know segments 4–6 arrived safely and may retransmit them all (Go-Back-N behavior).

**SACK (RFC 2018)** is an optional TCP extension that allows the receiver to report **non-contiguous blocks** of received data:

```
Lost:      Segment 3
Received:  Segments 1, 2, 4, 5, 6

SACK option: ACK=3, SACK={4–6}
→ Sender retransmits only segment 3
```

SACK is negotiated during the three-way handshake via TCP options and is enabled in virtually all modern TCP implementations.

### 9.3 Out-of-Order Delivery

TCP guarantees **ordered delivery** to the application:
- Out-of-order segments are **buffered** at the receiver.
- They are passed to the application only after all preceding bytes have arrived and been reordered.

---

## 10. TCP vs. UDP Comparison

### 10.1 Direct Comparison

| Feature | TCP | UDP |
|---|---|---|
| **Connection** | Connection-oriented (3-way handshake) | Connectionless |
| **Reliability** | Guaranteed delivery (ACK + retransmit) | Best-effort; no guarantees |
| **Ordering** | Ordered delivery guaranteed | No ordering |
| **Flow control** | Yes (sliding window / rwnd) | No |
| **Congestion control** | Yes (Slow Start, AIMD) | No |
| **Error detection** | Checksum (mandatory) | Checksum (optional in IPv4) |
| **Header size** | 20–60 bytes | 8 bytes |
| **Speed** | Slower (overhead) | Faster (minimal overhead) |
| **Broadcast/Multicast** | No (point-to-point only) | Yes |
| **Application control** | Less (TCP handles reliability) | More (app controls behavior) |

### 10.2 Application Protocol Selection Guide

| Application | Protocol | Reason |
|---|---|---|
| Web browsing (HTTP/HTTPS) | TCP | Reliable delivery of web pages required |
| Email (SMTP, IMAP, POP3) | TCP | Reliable message transfer required |
| File transfer (FTP, SFTP) | TCP | No data loss acceptable |
| Remote login (SSH, Telnet) | TCP | Ordered, reliable command transmission |
| DNS queries | UDP | Fast; short request/response; app retries |
| VoIP (SIP/RTP) | UDP | Low latency; loss tolerable; real-time |
| Video streaming (live) | UDP | Real-time; slight loss preferred over delay |
| Video streaming (on-demand) | TCP (DASH/HLS) | Buffered; reliability more important |
| Online gaming | UDP | Frequent state updates; stale data useless |
| DHCP | UDP | No prior connection possible |
| SNMP | UDP | Simple polling; short messages |
| TFTP | UDP | Uses own reliability; simpler than FTP |

### 10.3 The Case for Each

**Choose TCP when:**
- Data integrity is critical (financial transactions, files, commands).
- The application cannot implement its own reliability.
- Data must arrive in order.
- Connection setup overhead is acceptable.

**Choose UDP when:**
- Speed and low latency are critical.
- Occasional data loss is tolerable.
- The application implements its own error handling.
- Broadcast or multicast delivery is needed.
- Short transactions make connection setup overhead too expensive.

---

## 11. Transport Layer Security (TLS)

### 11.1 What is TLS?

**TLS (Transport Layer Security)** is a cryptographic protocol that provides **security** over a TCP connection. Although it sits logically between TCP and the Application Layer, it is commonly discussed with the Transport Layer.

TLS provides three security services:

| Service | Description |
|---|---|
| **Confidentiality** | Data is encrypted; eavesdroppers cannot read it |
| **Integrity** | Data cannot be modified in transit without detection (HMAC) |
| **Authentication** | Server (and optionally client) identity is verified using digital certificates (X.509) |

**TLS replaces the older SSL (Secure Sockets Layer)**. The current version is **TLS 1.3 (RFC 8446)**, released in 2018.

### 11.2 TLS Handshake (TLS 1.3)

TLS establishes a secure session on top of a TCP connection. TLS 1.3 reduces the handshake to **1 RTT** (or even 0-RTT for resumption):

```
Client                                   Server
──────                                   ──────
ClientHello (supported ciphers, key share) ────────────────►
                         ◄──────────────── ServerHello + Certificate
                                            + Finished (encrypted)
Finished                  ────────────────►
     ── Application Data (encrypted) ──
```

**Key steps:**
1. **ClientHello:** Client sends supported TLS versions, cipher suites, and a key share (Diffie-Hellman public key).
2. **ServerHello + Certificate:** Server selects cipher suite, sends its certificate (for authentication), and its DH key share.
3. **Key Derivation:** Both sides independently derive the same session keys using the DH exchange.
4. **Finished:** Both sides confirm handshake integrity with an encrypted Finished message.
5. **Application Data:** All subsequent data is encrypted with the negotiated session keys.

### 11.3 TLS 1.3 vs. TLS 1.2

| Feature | TLS 1.2 | TLS 1.3 |
|---|---|---|
| Handshake RTTs | 2 RTT | 1 RTT (0-RTT optional) |
| Cipher suites | Many (including weak) | Only 5 strong suites |
| Forward secrecy | Optional | Mandatory (always ephemeral DH) |
| RSA key exchange | Supported | Removed |
| Record encryption | After handshake | Immediately after ServerHello |
| Downgrade protection | Limited | Built-in |

### 11.4 Common TLS Applications

| Application | Protocol | Port |
|---|---|---|
| Secure web browsing | HTTPS (HTTP over TLS) | 443 |
| Secure email (sending) | SMTPS / STARTTLS | 465 / 587 |
| Secure email (retrieval) | IMAPS / POP3S | 993 / 995 |
| VPN | TLS-based VPNs (OpenVPN) | Various |
| Secure FTP | FTPS | 990 |

---

## 12. SCTP (Stream Control Transmission Protocol)

### 12.1 What is SCTP?

**SCTP (Stream Control Transmission Protocol)**, defined in **RFC 4960**, is a Transport Layer protocol that combines features of both TCP and UDP. It was designed primarily for **telecom signaling** (SS7 over IP) but is used in modern applications requiring high reliability and multi-homing.

### 12.2 SCTP Features

| Feature | Description |
|---|---|
| **Connection-oriented** | Uses a four-way handshake ("association" setup) with cookie mechanism to prevent SYN flooding |
| **Multi-streaming** | Supports multiple **independent streams** within one association; head-of-line blocking in one stream doesn't block others |
| **Multi-homing** | An SCTP association can span multiple IP addresses per endpoint; if one path fails, SCTP switches to an alternate path automatically |
| **Message-oriented** | Preserves message boundaries (unlike TCP's byte stream); similar to UDP in this regard |
| **Ordered and unordered delivery** | Can deliver messages in order (within a stream) or out of order |
| **Reliable delivery** | Uses ACKs and retransmission like TCP |
| **Congestion control** | Similar to TCP congestion control |

### 12.3 SCTP vs. TCP vs. UDP

| Feature | TCP | UDP | SCTP |
|---|---|---|---|
| Reliability | Yes | No | Yes |
| Ordering | Yes (byte stream) | No | Per-stream |
| Message boundaries | No | Yes | Yes |
| Multi-streaming | No | No | Yes |
| Multi-homing | No | No | Yes |
| Head-of-line blocking | Yes | N/A | No (per stream) |
| Use case | General reliable | Fast/real-time | Telecom, WebRTC |

---

## 13. Socket Programming Concepts

### 13.1 What is a Socket?

A **socket** is a software abstraction that provides an endpoint for network communication. It is identified by:

```
Socket = (IP Address, Port Number, Protocol)
```

Sockets are the standard API through which application programs communicate over a network (BSD Sockets API, used by virtually all operating systems).

### 13.2 Socket Types

| Socket Type | Protocol | Description |
|---|---|---|
| **Stream socket (SOCK_STREAM)** | TCP | Reliable, ordered, byte-stream communication |
| **Datagram socket (SOCK_DGRAM)** | UDP | Connectionless, unreliable datagram communication |
| **Raw socket (SOCK_RAW)** | IP/ICMP | Direct access to lower-layer protocols; used for custom protocol development and network tools |

### 13.3 TCP Client-Server Communication Flow

```
SERVER                                    CLIENT
──────                                    ──────
socket()   → Create socket               socket()   → Create socket
bind()     → Assign port                 
listen()   → Mark as passive             connect()  → Initiate 3-way handshake
accept()   → Block; wait for client  ◄──────────── (3-way handshake)
              (returns new socket)
              
send()/recv() ◄──────────────────────────────────► send()/recv()
              (data exchange)
              
close()    → Close connection            close()    → Initiate 4-way handshake
```

### 13.4 UDP Client-Server Communication Flow

UDP has no connection setup:

```
SERVER                                    CLIENT
──────                                    ──────
socket()   → Create UDP socket           socket()   → Create UDP socket
bind()     → Assign port                 
recvfrom() → Wait for datagram      ◄─── sendto()  → Send datagram to server
sendto()   → Reply to client        ───► recvfrom() → Receive reply
close()                                  close()
```

### 13.5 The Concept of Well-Known vs. Ephemeral Ports in Practice

When a client connects to a server:
- **Server** uses a **well-known port** (e.g., port 80 for HTTP) — bound explicitly via `bind()`.
- **Client** is assigned an **ephemeral port** by the OS automatically (e.g., 54321) — does not call `bind()`.

The combination `(client IP, client ephemeral port, server IP, server well-known port)` uniquely identifies the connection, allowing a single server to handle thousands of simultaneous clients on the same port 80.

---

## 14. Summary

This module covered the Transport Layer — the fourth layer of the OSI model:

- The **Transport Layer** provides process-to-process communication using port numbers for multiplexing and demultiplexing.
- **Port numbers** (0–65535) identify application processes; well-known ports (0–1023) are reserved for standard services.
- **UDP** is connectionless and unreliable, with an 8-byte header and minimal overhead — ideal for real-time and short-transaction applications.
- **TCP** is connection-oriented and reliable, providing ordered, error-checked byte-stream delivery with flow and congestion control.
- **TCP connection establishment** uses a three-way handshake (SYN, SYN-ACK, ACK); **termination** uses a four-way handshake (FIN, ACK, FIN, ACK).
- **TCP flow control** uses the receiver window (rwnd) to prevent buffer overflow; Nagle's Algorithm and Clark's Algorithm address the Silly Window Syndrome.
- **TCP congestion control** uses Slow Start (exponential growth), Congestion Avoidance (linear growth), and reacts to loss via Fast Retransmit/Recovery (3 DupACKs) or full Slow Start restart (timeout).
- **TCP error control** relies on sequence numbers, cumulative ACKs, retransmission timers (RTO), and optional SACK for selective retransmission.
- **TLS** adds confidentiality, integrity, and authentication on top of TCP; TLS 1.3 achieves this in 1 RTT.
- **SCTP** extends TCP with multi-streaming and multi-homing, eliminating head-of-line blocking.
- **Sockets** are the programming interface for Transport Layer communication; stream sockets use TCP, datagram sockets use UDP.

---

## 15. Key Terms

| Term | Definition |
|---|---|
| **Transport Layer** | OSI Layer 4; provides process-to-process delivery, flow control, and error control |
| **Port number** | 16-bit identifier for an application process on a host |
| **Socket** | Endpoint identified by IP address, port, and protocol |
| **Multiplexing** | Combining multiple application data streams into one transport connection |
| **Demultiplexing** | Delivering incoming segments to the correct application using port numbers |
| **UDP** | User Datagram Protocol; connectionless, unreliable, fast |
| **TCP** | Transmission Control Protocol; connection-oriented, reliable, ordered |
| **Segment** | The unit of data at the Transport Layer (TCP); also called datagram for UDP |
| **Sequence number** | Byte offset of the first data byte in a TCP segment |
| **Acknowledgment number** | Next expected byte; confirms all prior bytes received |
| **Three-way handshake** | TCP connection setup: SYN → SYN-ACK → ACK |
| **Four-way handshake** | TCP connection teardown: FIN → ACK → FIN → ACK |
| **TIME_WAIT** | State after sending final ACK; lasts 2×MSL to ensure clean connection close |
| **Flow control** | Preventing sender from overwhelming receiver's buffer (rwnd) |
| **rwnd** | Receiver Window; advertised buffer space available at the receiver |
| **Congestion control** | Adapting sender rate to network capacity (cwnd-based) |
| **cwnd** | Congestion Window; sender-side limit on bytes in flight |
| **Slow Start** | TCP phase: exponential cwnd growth from 1 MSS until ssthresh |
| **Congestion Avoidance** | TCP phase: linear cwnd growth after ssthresh (AIMD) |
| **Fast Retransmit** | Retransmitting a lost segment after 3 duplicate ACKs without waiting for timeout |
| **Fast Recovery** | Staying in Congestion Avoidance (not Slow Start) after Fast Retransmit |
| **RTO** | Retransmission Timeout; dynamically calculated from measured RTT |
| **SACK** | Selective Acknowledgment; allows receiver to report non-contiguous received blocks |
| **Nagle's Algorithm** | Buffers small TCP segments until MSS data is ready or pipe is clear |
| **MSS** | Maximum Segment Size; largest amount of data TCP will send in one segment |
| **TLS** | Transport Layer Security; cryptographic protocol for confidentiality and integrity over TCP |
| **SCTP** | Stream Control Transmission Protocol; multi-streaming, multi-homing transport protocol |
| **SYN flood** | DoS attack exploiting TCP's half-open connection state; mitigated by SYN cookies |
| **Silly Window Syndrome** | Inefficiency from sending/receiving tiny TCP segments; solved by Nagle/Clark |
| **QUIC** | UDP-based transport with built-in TLS; used in HTTP/3 |

---

## 16. Review Questions

**Conceptual Questions:**

1. What is the difference between process-to-process and host-to-host communication? How do port numbers enable process-to-process delivery?
2. A server is running HTTP (port 80), HTTPS (port 443), and FTP (port 21) simultaneously. Explain how the Transport Layer directs incoming packets to the correct application.
3. Compare the UDP and TCP header formats. Why is the UDP header only 8 bytes while TCP requires at least 20 bytes?
4. Explain each step of the TCP three-way handshake. Why is the three-way handshake necessary rather than a two-way handshake?
5. What is the purpose of the TIME_WAIT state in TCP? What problems would occur if TCP skipped TIME_WAIT?
6. Explain the difference between flow control and congestion control in TCP. What information does each use, and what does each protect against?
7. A TCP connection has cwnd = 16 MSS and ssthresh = 8 MSS. Three duplicate ACKs are received. Describe what happens to cwnd and ssthresh immediately and in subsequent RTTs (using TCP Reno).
8. What is the Silly Window Syndrome? How do Nagle's Algorithm and Clark's Algorithm address it on the sender side and receiver side respectively?
9. What is SACK, and how does it improve on standard cumulative acknowledgments? Give an example of when SACK saves unnecessary retransmissions.
10. What three security services does TLS provide? Why is TLS 1.3's mandatory forward secrecy important?

**Application / Calculation Questions:**

11. A TCP sender has sent bytes 0–2999 and received an ACK for byte 2000. The window size (rwnd) is 4000 bytes. How many more bytes can the sender transmit?

12. A TCP connection begins with cwnd = 1 MSS and ssthresh = 16 MSS. Trace the value of cwnd at the end of each RTT for 10 RTTs, assuming no packet loss. Identify when Slow Start ends and Congestion Avoidance begins.

13. After 10 RTTs (from Question 12), a timeout occurs. What are the new values of ssthresh and cwnd? Trace the cwnd for 5 more RTTs.

14. Calculate the RTO given the following measurements:
    - Initial SRTT = 100 ms
    - New RTT sample = 120 ms
    - Initial RTTVAR = 10 ms
    - Use α = 0.125, β = 0.25

15. A UDP application sends video frames at 30 fps, each frame is 10 KB. The network has a bandwidth of 10 Mbps and a one-way propagation delay of 20 ms. Calculate: (a) the transmission time per frame, (b) the total delay per frame (transmission + propagation), and (c) whether the network can sustain this video rate. Show your work.

16. Explain why DNS uses UDP for standard queries but falls back to TCP for responses larger than 512 bytes (or 4096 bytes with EDNS). What characteristic of DNS makes UDP the right default choice?

17. A client at IP `10.0.0.5` connects to a web server at `203.0.113.10:443`. The OS assigns the client an ephemeral port of `52847`. Write the full 5-tuple that uniquely identifies this TCP connection. If the same client opens a second browser tab to the same server simultaneously, how does the network distinguish the two connections?

---

*End of Module 5 | Next: Module 6 – Session Layer and Presentation Layer*
