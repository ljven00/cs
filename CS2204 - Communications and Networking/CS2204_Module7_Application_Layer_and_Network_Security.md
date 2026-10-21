# CS 2204 – Communications and Networking
## Module 7: The Application Layer and Network Security

---

## Table of Contents

1. [Overview](#1-overview)
2. [Application Layer Fundamentals](#2-application-layer-fundamentals)
3. [Domain Name System (DNS)](#3-domain-name-system-dns)
4. [Hypertext Transfer Protocol (HTTP and HTTPS)](#4-hypertext-transfer-protocol-http-and-https)
5. [Electronic Mail Protocols](#5-electronic-mail-protocols)
6. [File Transfer Protocols](#6-file-transfer-protocols)
7. [Remote Access Protocols](#7-remote-access-protocols)
8. [Network Management: SNMP](#8-network-management-snmp)
9. [Introduction to Network Security](#9-introduction-to-network-security)
10. [Types of Network Attacks](#10-types-of-network-attacks)
11. [Firewalls](#11-firewalls)
12. [Intrusion Detection and Prevention Systems](#12-intrusion-detection-and-prevention-systems)
13. [Virtual Private Networks (VPN)](#13-virtual-private-networks-vpn)
14. [Network Access Control and Authentication](#14-network-access-control-and-authentication)
15. [Wireless Security](#15-wireless-security)
16. [Summary](#16-summary)
17. [Key Terms](#17-key-terms)
18. [Review Questions](#18-review-questions)

---

## 1. Overview

The Application Layer is the topmost layer of both the OSI and TCP/IP models. It is the layer closest to the end user — the layer where network services are delivered directly to applications. Every time a web page is loaded, an email is sent, or a file is downloaded, Application Layer protocols are at work.

This module also introduces **network security** — a topic that spans all layers but is most visible at the Application Layer. Security concerns are woven into every modern protocol, and understanding threats, defenses, and security protocols is essential for any networking professional.

**Module Learning Objectives:**

By the end of this module, students should be able to:

- Describe the role and responsibilities of the Application Layer.
- Explain how DNS resolves domain names to IP addresses using a hierarchical system.
- Describe HTTP and HTTPS operation, methods, status codes, and versions.
- Explain how email is sent, relayed, and retrieved using SMTP, POP3, and IMAP.
- Describe FTP, SFTP, and TFTP and their security trade-offs.
- Explain SSH and Telnet for remote access.
- Describe SNMP and its role in network management.
- Define key security concepts: CIA triad, threats, vulnerabilities, and attacks.
- Describe common network attacks: DoS/DDoS, MITM, phishing, replay, SQL injection.
- Explain how firewalls, IDS/IPS, and VPNs protect networks.
- Describe authentication mechanisms and wireless security protocols.

---

## 2. Application Layer Fundamentals

### 2.1 Role of the Application Layer

The **Application Layer (OSI Layer 7 / TCP/IP Application Layer)** provides **network services directly to end-user applications**. It defines the protocols that applications use to communicate over the network.

Important distinctions:
- The Application Layer is **not** the application itself (e.g., a web browser).
- It is the **protocol layer** through which applications access network services (e.g., HTTP is the Application Layer protocol; Chrome is the application using HTTP).

### 2.2 Client-Server vs. Peer-to-Peer

Application Layer architectures fall into two primary models:

**Client-Server Model:**
- A **server** provides a service and listens on a well-known port.
- A **client** initiates a request to the server.
- Servers are always on; clients communicate when needed.
- Examples: HTTP (web), SMTP (email), DNS, FTP.

```
Client ──── Request ────► Server
Client ◄─── Response ─── Server
```

**Peer-to-Peer (P2P) Model:**
- Every node can act as both client and server.
- No central server required; resources are shared directly between peers.
- Examples: BitTorrent, blockchain networks, early Skype, Gnutella.

```
Peer A ◄────────────────► Peer B
  ▲                          ▲
  └──────────► Peer C ◄──────┘
```

**Hybrid Model:**
- Combines both: a central server for coordination/indexing, but direct peer-to-peer for data transfer.
- Example: Modern Skype (signaling via Microsoft servers; media via P2P), Napster (original design).

### 2.3 Application Layer Protocol Data Unit

The unit of data at the Application Layer is a **message**. This message is passed to the Transport Layer, which segments it into TCP segments or UDP datagrams.

```
Application Layer:  Message
Transport Layer:    Segment (TCP) / Datagram (UDP)
Network Layer:      Packet
Data Link Layer:    Frame
Physical Layer:     Bits
```

---

## 3. Domain Name System (DNS)

### 3.1 Why DNS?

IP addresses (e.g., `142.250.80.46`) are difficult for humans to remember. **DNS (Domain Name System)** is a distributed, hierarchical naming system that translates human-readable **domain names** (e.g., `www.google.com`) into **IP addresses**.

DNS is defined in **RFC 1034** and **RFC 1035**. It operates primarily over **UDP port 53** (and TCP port 53 for responses > 512 bytes or zone transfers).

### 3.2 DNS Hierarchy

The DNS namespace is organized as a hierarchical tree:

```
                        . (Root)
           ┌────────────┼────────────┐
          com          org          edu
     ┌─────┴─────┐
  google       amazon
 ┌──┴──┐
www   mail
```

**Levels:**
- **Root (`.`):** The top of the hierarchy; managed by IANA/ICANN; 13 root server clusters (A through M).
- **Top-Level Domain (TLD):** `.com`, `.org`, `.net`, `.edu`, `.gov`, country codes (`.us`, `.uk`, `.de`).
- **Second-Level Domain (SLD):** `google`, `amazon`, `bbc` — registered by organizations.
- **Subdomain:** `www`, `mail`, `ftp` — configured by the domain owner.
- **Fully Qualified Domain Name (FQDN):** The complete name including all levels: `www.example.com.` (note trailing dot for root).

### 3.3 DNS Resolution Process

When a user types `www.example.com` in a browser:

```
Browser          Resolver         Root NS       TLD NS (.com)    Auth NS
───────          ────────         ───────       ─────────────    ───────
Query ──────────►
                 (Check local cache — miss)
                 Query ──────────►
                         ◄─────── Refer to .com NS
                 Query ──────────────────────────►
                         ◄─────────────────────── Refer to example.com NS
                 Query ──────────────────────────────────────────►
                         ◄────────────────────────────────────── A: 93.184.216.34
◄────────────────
```

**Participants:**
1. **DNS Stub Resolver:** Built into the OS; forwards queries to the recursive resolver.
2. **Recursive Resolver (DNS Resolver):** Operated by ISP or service (e.g., Google 8.8.8.8, Cloudflare 1.1.1.1); does the heavy lifting.
3. **Root Name Servers:** Direct queries to the appropriate TLD name servers.
4. **TLD Name Servers:** Direct queries to the authoritative name server for the domain.
5. **Authoritative Name Server:** Holds the actual DNS records for the domain; returns the answer.

### 3.4 DNS Record Types

| Record Type | Description | Example |
|---|---|---|
| **A** | Maps a hostname to an IPv4 address | `www.example.com → 93.184.216.34` |
| **AAAA** | Maps a hostname to an IPv6 address | `www.example.com → 2606:2800:220:1:248:1893:25c8:1946` |
| **CNAME** | Canonical Name; alias pointing to another hostname | `mail.example.com → example-com.mail.protection.outlook.com` |
| **MX** | Mail Exchange; specifies mail servers for the domain | `example.com → mail.example.com (priority 10)` |
| **NS** | Name Server; identifies authoritative name servers | `example.com → ns1.example.com` |
| **PTR** | Pointer; reverse DNS — IP address to hostname | `93.184.216.34 → www.example.com` |
| **TXT** | Text record; used for SPF, DKIM, DMARC, domain verification | `example.com → "v=spf1 include:_spf.google.com ~all"` |
| **SOA** | Start of Authority; administrative info about the zone | Serial, refresh, retry, expire times |
| **SRV** | Service record; specifies hostname/port for specific services | `_sip._tcp.example.com → 10 20 5060 sip.example.com` |
| **CAA** | Certification Authority Authorization; restricts which CAs can issue certs | `example.com → 0 issue "letsencrypt.org"` |

### 3.5 DNS Caching and TTL

- Every DNS record has a **TTL (Time to Live)** value (in seconds).
- Resolvers and clients **cache** DNS responses for the duration of the TTL.
- Caching reduces DNS query traffic and speeds up resolution.
- After TTL expires, the resolver must query again to get a fresh answer.

### 3.6 DNS Security

**DNS Spoofing / Cache Poisoning:**
An attacker injects forged DNS responses into a resolver's cache, redirecting users to malicious IP addresses.

**DNSSEC (DNS Security Extensions):**
Adds digital signatures to DNS records. Resolvers can verify that responses are authentic and have not been tampered with. Uses asymmetric cryptography (public keys published as DNSKEY records).

**DNS over HTTPS (DoH) and DNS over TLS (DoT):**
Encrypts DNS queries to prevent eavesdropping and manipulation by intermediaries (ISPs, network attackers).

---

## 4. Hypertext Transfer Protocol (HTTP and HTTPS)

### 4.1 What is HTTP?

**HTTP (Hypertext Transfer Protocol)** is the foundation of data communication on the World Wide Web. It is an **application-level protocol** for distributed, hypermedia information systems.

- **RFC 7230–7235** (HTTP/1.1), **RFC 7540** (HTTP/2), **RFC 9114** (HTTP/3).
- **Default port:** 80 (HTTP), 443 (HTTPS).
- **Stateless:** Each request-response pair is independent; no session state retained by default.

### 4.2 HTTP Request-Response Model

```
Client (Browser)                          Web Server
────────────────                          ──────────
HTTP Request ──────────────────────────►
             ◄───────────────────────── HTTP Response
```

**HTTP Request Format:**
```
GET /index.html HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0
Accept: text/html,application/xhtml+xml
Accept-Encoding: gzip, deflate
Connection: keep-alive
```

**HTTP Response Format:**
```
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Content-Length: 1234
Content-Encoding: gzip
Server: Apache/2.4.51

<html>...</html>
```

### 4.3 HTTP Methods

| Method | Description | Body | Safe | Idempotent |
|---|---|---|---|---|
| **GET** | Retrieve a resource | No | Yes | Yes |
| **POST** | Submit data to be processed | Yes | No | No |
| **PUT** | Replace a resource entirely | Yes | No | Yes |
| **PATCH** | Partially update a resource | Yes | No | No |
| **DELETE** | Delete a resource | Optional | No | Yes |
| **HEAD** | Same as GET but returns headers only | No | Yes | Yes |
| **OPTIONS** | Retrieve supported methods for a resource | No | Yes | Yes |
| **CONNECT** | Establish a tunnel (used for HTTPS proxies) | No | No | No |
| **TRACE** | Echo the request (used for diagnostics) | No | Yes | Yes |

- **Safe:** Does not modify server state.
- **Idempotent:** Multiple identical requests have the same effect as one.

### 4.4 HTTP Status Codes

| Code Range | Category | Examples |
|---|---|---|
| **1xx** | Informational | 100 Continue, 101 Switching Protocols |
| **2xx** | Success | 200 OK, 201 Created, 204 No Content |
| **3xx** | Redirection | 301 Moved Permanently, 302 Found, 304 Not Modified |
| **4xx** | Client Error | 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 429 Too Many Requests |
| **5xx** | Server Error | 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable |

### 4.5 HTTP Versions

**HTTP/1.0:**
- One request per TCP connection (connection closed after each response).
- Inefficient: TCP handshake overhead for every resource.

**HTTP/1.1:**
- **Persistent connections:** TCP connection kept open for multiple requests (`Connection: keep-alive`).
- **Pipelining:** Client sends multiple requests without waiting for responses (limited adoption).
- **Host header:** Mandatory; allows virtual hosting (multiple domains on one IP).
- **Chunked transfer encoding:** Response body sent in chunks; useful for streaming.
- Still dominant in practice but has **Head-of-Line (HoL) blocking** at the request level.

**HTTP/2 (RFC 7540):**
- **Binary framing layer:** Data sent as binary frames instead of text.
- **Multiplexing:** Multiple requests/responses over a single TCP connection simultaneously — eliminates HoL blocking at the HTTP level.
- **Header compression (HPACK):** Reduces overhead from repetitive headers.
- **Server push:** Server proactively sends resources the client will need.
- **Stream prioritization:** Client can indicate resource priority.
- Still suffers from TCP-level HoL blocking.

**HTTP/3 (RFC 9114):**
- Runs over **QUIC** (UDP-based transport) instead of TCP.
- Eliminates TCP HoL blocking entirely (each QUIC stream is independent).
- Built-in TLS 1.3 — encrypted by default, faster connection setup.
- Faster connection establishment: 0-RTT or 1-RTT.
- Adopted by Google, Facebook, Cloudflare; growing adoption.

### 4.6 HTTPS

**HTTPS = HTTP + TLS**

- TLS encrypts the HTTP traffic between client and server.
- Provides confidentiality, integrity, and server authentication.
- The browser verifies the server's X.509 certificate before the connection proceeds.
- Indicated by `https://` in the URL and a padlock icon in the browser.

**HTTP Strict Transport Security (HSTS):**
A server-side response header that instructs browsers to always use HTTPS for the domain — preventing SSL stripping attacks:
```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

---

## 5. Electronic Mail Protocols

### 5.1 Email Architecture

Email delivery involves multiple components and protocols:

```
Sender's MUA ──SMTP──► Sender's MTA ──SMTP──► Receiver's MTA ──► Mailbox
(Mail User Agent)     (Mail Transfer Agent)                    ↓
(e.g., Outlook)       (e.g., Postfix)         (e.g., Exchange)  Receiver's MUA
                                                                 accesses via
                                                                 POP3 or IMAP
```

| Component | Role | Protocol |
|---|---|---|
| **MUA (Mail User Agent)** | Email client (Outlook, Thunderbird, Gmail web) | SMTP (send), IMAP/POP3 (receive) |
| **MTA (Mail Transfer Agent)** | Mail server that routes/relays messages | SMTP |
| **MDA (Mail Delivery Agent)** | Delivers mail to the user's mailbox | Local delivery |
| **MSA (Mail Submission Agent)** | Accepts mail from MUA for submission | SMTP (port 587) |

### 5.2 SMTP (Simple Mail Transfer Protocol)

**SMTP** (RFC 5321) is the protocol for **sending and relaying** email between mail servers.

- **Ports:** 25 (server-to-server), **587** (submission, client-to-server with auth), 465 (SMTPS — SMTP over TLS).
- **Transport:** TCP.
- **Push protocol:** The sender initiates delivery to the receiver's mail server.

**SMTP Session Example:**
```
Client                        Server
──────                        ──────
                ◄──────────── 220 mail.example.com SMTP ready
EHLO client.com ─────────────►
                ◄──────────── 250-mail.example.com Hello
                              250-STARTTLS
                              250 AUTH LOGIN PLAIN
MAIL FROM:<alice@a.com> ─────►
                ◄──────────── 250 OK
RCPT TO:<bob@b.com> ─────────►
                ◄──────────── 250 OK
DATA ────────────────────────►
                ◄──────────── 354 Start input
Subject: Hello
From: alice@a.com
To: bob@b.com

Hello Bob!
. ───────────────────────────►
                ◄──────────── 250 OK Message accepted
QUIT ────────────────────────►
                ◄──────────── 221 Closing
```

**SMTP Commands:**

| Command | Purpose |
|---|---|
| `EHLO` / `HELO` | Identify the sending server |
| `MAIL FROM` | Specify the sender's address |
| `RCPT TO` | Specify the recipient's address |
| `DATA` | Begin message body transmission |
| `QUIT` | Close the connection |
| `STARTTLS` | Upgrade connection to TLS |
| `AUTH` | Authenticate the client |

### 5.3 Email Authentication (SPF, DKIM, DMARC)

To combat spam and email spoofing:

**SPF (Sender Policy Framework):**
A TXT DNS record that lists which IP addresses are authorized to send email for a domain.
```
example.com TXT "v=spf1 ip4:192.0.2.0/24 include:_spf.google.com ~all"
```

**DKIM (DomainKeys Identified Mail):**
The sending server **digitally signs** each outgoing email. The receiving server retrieves the public key from DNS and verifies the signature.

**DMARC (Domain-based Message Authentication, Reporting, and Conformance):**
Builds on SPF and DKIM — instructs receiving servers what to do with mail that fails authentication (none, quarantine, reject) and provides reporting to domain owners.

### 5.4 POP3 (Post Office Protocol v3)

**POP3** (RFC 1939) is a protocol for **retrieving email** from a mail server.

- **Port:** 110 (POP3), 995 (POP3S over TLS).
- **Transport:** TCP.
- **Model:** Downloads messages to the local device and **deletes them from the server** (by default).
- **Suitable for:** Single-device access; offline reading.
- **Not suitable for:** Multi-device access (messages disappear from server after download).

**POP3 Commands:**

| Command | Purpose |
|---|---|
| `USER` | Specify username |
| `PASS` | Provide password |
| `LIST` | List messages |
| `RETR n` | Retrieve message n |
| `DELE n` | Mark message n for deletion |
| `QUIT` | End session (deletions committed) |

### 5.5 IMAP (Internet Message Access Protocol)

**IMAP** (RFC 3501 / RFC 9051 for IMAP4rev2) is a more powerful protocol for **accessing and managing email on the server**.

- **Port:** 143 (IMAP), 993 (IMAPS over TLS).
- **Transport:** TCP.
- **Model:** Messages remain on the server; client **synchronizes** a local view.
- **Suitable for:** Multi-device access (phone, laptop, web — all see the same inbox).
- **Features:** Server-side folders, search, flags (read/unread/answered), partial fetch.

### 5.6 POP3 vs. IMAP

| Feature | POP3 | IMAP |
|---|---|---|
| Messages stored | Locally (after download) | On server |
| Multi-device sync | No | Yes |
| Server-side folders | No | Yes |
| Partial message fetch | No | Yes |
| Offline access | Easy (downloaded) | Possible (with sync) |
| Server storage usage | Low | High |
| Suitable for | Single-device, limited server storage | Multiple devices, modern use |

---

## 6. File Transfer Protocols

### 6.1 FTP (File Transfer Protocol)

**FTP** (RFC 959) is a standard protocol for transferring files between a client and a server.

- **Ports:** 21 (control), 20 (data — active mode).
- **Transport:** TCP (two connections: one for control, one for data).
- **Authentication:** Username and password (sent in plaintext — insecure).
- **Security:** None (all data including credentials transmitted unencrypted).

**FTP Modes:**

| Mode | How Data Connection is Established | Firewall Issues |
|---|---|---|
| **Active mode** | Server initiates data connection to client port | Client-side firewalls often block inbound connections |
| **Passive mode** | Client initiates both control and data connections to server | Firewall-friendly for clients |

**FTP Commands (over control connection):**

| Command | Purpose |
|---|---|
| `USER` | Send username |
| `PASS` | Send password |
| `LIST` | List directory contents |
| `RETR filename` | Download a file |
| `STOR filename` | Upload a file |
| `PWD` | Print working directory |
| `CWD directory` | Change working directory |
| `QUIT` | Close session |

### 6.2 SFTP (SSH File Transfer Protocol)

**SFTP** is a secure file transfer protocol that runs over an **SSH (Secure Shell)** connection.

- **Port:** 22 (same as SSH).
- **Transport:** TCP, encrypted end-to-end.
- **Authentication:** Password or SSH key pair.
- **Not FTP over SSH:** SFTP is a completely different protocol that happens to run over SSH.
- Provides: confidentiality, integrity, authentication, directory operations, resume capability.

### 6.3 FTPS (FTP Secure)

**FTPS** adds TLS encryption to the standard FTP protocol.

- **Ports:** 990 (implicit TLS), 21 with STARTTLS (explicit TLS).
- Unlike SFTP, FTPS is true FTP with encryption layered on top.
- More complex to configure through firewalls than SFTP.

### 6.4 TFTP (Trivial File Transfer Protocol)

**TFTP** (RFC 1350) is a simple, lightweight file transfer protocol.

- **Port:** 69.
- **Transport:** UDP (not TCP).
- **No authentication:** No username or password.
- **No directory listing:** Can only read or write files by name.
- **Use cases:** Booting diskless workstations, uploading firmware to routers/switches, PXE network boot.
- **Reliability:** Implements its own simple Stop-and-Wait ARQ over UDP.

### 6.5 File Transfer Protocol Comparison

| Feature | FTP | SFTP | FTPS | TFTP |
|---|---|---|---|---|
| Transport | TCP | TCP (over SSH) | TCP (over TLS) | UDP |
| Encryption | None | Yes (SSH) | Yes (TLS) | None |
| Authentication | Username/Password | Password or Key | Username/Password | None |
| Firewall-friendly | Difficult | Yes | Difficult | Yes |
| Directory operations | Yes | Yes | Yes | No |
| Use case | Legacy transfers | Secure transfers | Secure FTP | Network boot, firmware |

---

## 7. Remote Access Protocols

### 7.1 Telnet

**Telnet** (RFC 854) provides a **virtual terminal** service — a text-based interface to a remote machine.

- **Port:** 23.
- **Transport:** TCP.
- **Security:** None — all data, including passwords, transmitted in **plaintext**.
- **Status:** Deprecated for most use; replaced by SSH in virtually all production environments.
- Still used occasionally for: quick testing of TCP services (e.g., `telnet server 80` to test HTTP), troubleshooting legacy systems.

### 7.2 SSH (Secure Shell)

**SSH** (RFC 4251–4254) provides a **secure, encrypted channel** for remote login, command execution, and tunneling.

- **Port:** 22.
- **Transport:** TCP.
- **Security:** Strong encryption (AES, ChaCha20), integrity (HMAC), authentication.

**SSH Authentication Methods:**

| Method | Description |
|---|---|
| **Password** | Username and password (encrypted in transit, but susceptible to brute force) |
| **Public key** | Client proves identity using a private key; server verifies with the stored public key — most secure and common |
| **Keyboard-interactive** | Challenge-response (used for MFA/OTP integration) |
| **GSSAPI** | Kerberos-based authentication (used in enterprise environments) |

**SSH Architecture — Three Layers:**

| Layer | Protocol | Function |
|---|---|---|
| **Transport** | SSH-TRANS | Provides encryption, server authentication, integrity |
| **Authentication** | SSH-USERAUTH | Handles user authentication |
| **Connection** | SSH-CONNECT | Multiplexes multiple logical channels over one SSH connection |

**SSH Features Beyond Remote Login:**

| Feature | Description |
|---|---|
| **SCP (Secure Copy)** | File copying using SSH |
| **SFTP** | Full file management over SSH |
| **Port forwarding (tunneling)** | Forwards TCP traffic through an encrypted SSH tunnel |
| **X11 forwarding** | Forwards graphical Linux applications over SSH |
| **SSH agent** | Caches SSH private keys to avoid repeated passphrase entry |

**SSH Known Hosts:**
On first connection to a new server, SSH presents the server's **host key fingerprint** and asks the user to verify it. This fingerprint is stored in `~/.ssh/known_hosts`. On subsequent connections, SSH verifies the fingerprint automatically — protecting against man-in-the-middle attacks.

---

## 8. Network Management: SNMP

### 8.1 What is SNMP?

**SNMP (Simple Network Management Protocol)** is the standard protocol for **monitoring and managing network devices** (routers, switches, servers, printers).

- **RFC 1157** (SNMPv1), **RFC 1901** (SNMPv2c), **RFC 3411–3418** (SNMPv3).
- **Port:** 161 (SNMP agent), 162 (SNMP trap receiver).
- **Transport:** UDP.

### 8.2 SNMP Architecture

| Component | Description |
|---|---|
| **SNMP Manager** | The network management system (NMS) — polls devices and receives traps |
| **SNMP Agent** | Software running on managed devices (router, switch); responds to manager queries |
| **MIB (Management Information Base)** | A hierarchical database defining the variables (objects) that can be monitored/controlled |
| **OID (Object Identifier)** | A unique dotted-decimal identifier for each MIB object (e.g., `1.3.6.1.2.1.1.1` = sysDescr) |

### 8.3 SNMP Operations

| Operation | Direction | Description |
|---|---|---|
| **GET** | Manager → Agent | Request the value of a specific MIB object |
| **GET-NEXT** | Manager → Agent | Request the value of the next object in the MIB tree |
| **GET-BULK** | Manager → Agent | Request multiple MIB objects in one query (SNMPv2+) |
| **SET** | Manager → Agent | Set the value of a MIB object (e.g., change interface speed) |
| **TRAP** | Agent → Manager | Unsolicited notification from agent when an event occurs |
| **INFORM** | Agent → Manager | Like TRAP but requires acknowledgment (SNMPv2+) |

### 8.4 SNMP Versions

| Version | Security | Community String | Notes |
|---|---|---|---|
| **SNMPv1** | None | Plaintext | Original; no authentication or encryption |
| **SNMPv2c** | None | Plaintext | Adds GET-BULK, 64-bit counters; still uses community strings |
| **SNMPv3** | Full | N/A | Adds authentication (HMAC-MD5/SHA) and encryption (AES/3DES); recommended |

**Community strings** (SNMPv1/v2c) function like simple passwords:
- `public` — read-only (default, widely known — major security risk).
- `private` — read-write (even more sensitive).

**SNMPv3 Security Models:**
- **noAuthNoPriv:** No authentication, no encryption.
- **authNoPriv:** Authentication (HMAC), no encryption.
- **authPriv:** Authentication + encryption (AES) — recommended configuration.

---

## 9. Introduction to Network Security

### 9.1 The CIA Triad

The foundation of information security is the **CIA Triad**:

```
        ┌─────────────────────┐
        │   Confidentiality   │
        │  (only authorized   │
        │  parties can read)  │
        └──────────┬──────────┘
                   │
       ┌───────────┴────────────┐
       │                        │
┌──────┴──────┐          ┌──────┴──────┐
│  Integrity  │          │Availability │
│ (data is    │          │ (systems are│
│  accurate & │          │  accessible │
│  unaltered) │          │  when needed│
└─────────────┘          └─────────────┘
```

| Property | Definition | Threat | Control |
|---|---|---|---|
| **Confidentiality** | Information is accessible only to those authorized | Eavesdropping, data theft | Encryption, access control |
| **Integrity** | Information is accurate and has not been tampered with | Data modification, corruption | Hashing, digital signatures, checksums |
| **Availability** | Systems and data are accessible to authorized users when needed | DoS/DDoS attacks, hardware failure | Redundancy, backups, DDoS protection |

**Extended properties (sometimes added):**
- **Authentication:** Verifying identity of users, devices, or services.
- **Non-repudiation:** Proof that a party performed an action; they cannot deny it later.
- **Authorization:** Ensuring authenticated entities can only access what they are permitted to.

### 9.2 Security Terminology

| Term | Definition |
|---|---|
| **Threat** | Any potential danger to a system or data |
| **Vulnerability** | A weakness in a system that could be exploited |
| **Exploit** | A technique or tool that takes advantage of a vulnerability |
| **Attack** | A deliberate action to violate security |
| **Risk** | The probability that a threat will exploit a vulnerability × the impact |
| **Countermeasure** | A control that reduces a risk |
| **Zero-day** | A vulnerability unknown to the vendor; no patch yet exists |

### 9.3 Threat Actors

| Actor | Motivation | Sophistication |
|---|---|---|
| **Script kiddies** | Curiosity, notoriety | Low (use pre-written tools) |
| **Hacktivists** | Political/social agenda | Medium |
| **Cybercriminals** | Financial gain | Medium to high |
| **Insider threats** | Grudge, financial, espionage | Variable (trusted access) |
| **Nation-state actors** | Espionage, sabotage | Very high (APT — Advanced Persistent Threat) |

---

## 10. Types of Network Attacks

### 10.1 Denial of Service (DoS) and Distributed DoS (DDoS)

**DoS:** An attacker overwhelms a target system with traffic or requests, making it unavailable to legitimate users.

**DDoS:** The attack is launched from **many distributed sources simultaneously** (a botnet), making it much harder to block.

**Common DDoS Techniques:**

| Technique | Description |
|---|---|
| **SYN flood** | Sends many SYN packets with spoofed source IPs; server allocates resources for half-open connections |
| **UDP flood** | Sends large volumes of UDP packets to random ports; target wastes resources sending ICMP "port unreachable" replies |
| **ICMP flood (Ping flood)** | Overwhelms target with ICMP Echo Requests |
| **HTTP flood** | Sends many legitimate-looking HTTP GET/POST requests; application-layer attack |
| **Amplification attack** | Uses protocols with large response-to-request ratios (DNS, NTP, SSDP) to amplify traffic toward the victim |
| **Smurf attack** | Sends ICMP Echo Requests with spoofed victim IP as source to a broadcast address; all hosts reply to victim |

**DDoS Mitigation:**
- Traffic scrubbing centers (cloud-based DDoS protection: Cloudflare, Akamai).
- Rate limiting and traffic shaping.
- Anycast network diffusion.
- BGP blackholing (null-routing attack traffic).

### 10.2 Man-in-the-Middle (MITM) Attack

An attacker **secretly intercepts** and potentially **modifies** communication between two parties who believe they are communicating directly.

**Attack variants:**

| Variant | Method |
|---|---|
| **ARP poisoning** | Attacker sends fake ARP replies to associate their MAC with a gateway's IP; intercepts LAN traffic |
| **DNS spoofing** | Forged DNS responses redirect victims to attacker-controlled servers |
| **SSL stripping** | Attacker downgrades an HTTPS connection to HTTP between the victim and the real server |
| **Rogue Wi-Fi AP** | Attacker sets up a fake access point; victims connect and all traffic passes through the attacker |
| **BGP hijacking** | Attacker announces fraudulent BGP routes, diverting Internet traffic through their network |

**Mitigations:** TLS/HTTPS, HSTS, DNSSEC, certificate pinning, dynamic ARP inspection (DAI).

### 10.3 Phishing and Social Engineering

**Phishing:** Fraudulent communications (usually email) designed to trick users into revealing credentials, clicking malicious links, or downloading malware.

**Variants:**

| Type | Description |
|---|---|
| **Phishing** | Mass emails impersonating trusted organizations |
| **Spear phishing** | Targeted phishing crafted for a specific individual or organization |
| **Whaling** | Spear phishing targeting senior executives (CEO, CFO) |
| **Smishing** | Phishing via SMS |
| **Vishing** | Voice phishing via phone calls |
| **Pharming** | Redirecting users from legitimate websites to fraudulent ones (via DNS/host file manipulation) |

**Mitigations:** User training and awareness, multi-factor authentication (MFA), email filtering, DMARC/DKIM/SPF, anti-phishing browser features.

### 10.4 Replay Attack

An attacker **captures a valid data transmission** (e.g., an authentication credential) and **replays it later** to gain unauthorized access.

**Example:** Capturing a session token from HTTP traffic and resending it to impersonate the user.

**Mitigations:**
- **Timestamps:** Include current time in authentication messages; reject messages older than a threshold.
- **Nonces (number used once):** A unique random value included in each request; server rejects any message reusing a previous nonce.
- **Sequence numbers:** Track expected sequence of messages; reject out-of-order replays.
- TLS encryption (prevents capture in transit).

### 10.5 SQL Injection

**SQL injection** occurs when an attacker inserts malicious SQL code into an input field that is incorporated into a database query.

**Vulnerable code example (PHP):**
```php
$query = "SELECT * FROM users WHERE username='" . $_GET['user'] . "'";
```

**Malicious input:** `' OR '1'='1`

**Resulting query:**
```sql
SELECT * FROM users WHERE username='' OR '1'='1'
```
This returns all users because `'1'='1'` is always true.

**Impact:** Data exfiltration, authentication bypass, data modification, database destruction.

**Mitigations:**
- **Parameterized queries / prepared statements** (the correct fix — input is never interpreted as code).
- **Input validation and sanitization.**
- **Least privilege:** Database accounts should have only necessary permissions.
- **Web Application Firewall (WAF).**

### 10.6 Cross-Site Scripting (XSS)

An attacker injects malicious **JavaScript** into web pages viewed by other users.

**Types:**

| Type | Description |
|---|---|
| **Stored XSS** | Malicious script stored in the database; served to all visitors |
| **Reflected XSS** | Script included in a URL parameter; reflected back in the response |
| **DOM-based XSS** | Script manipulates the DOM in the browser without server interaction |

**Impact:** Cookie theft, session hijacking, credential harvesting, defacement.

**Mitigations:** Output encoding, Content Security Policy (CSP), HTTPOnly and Secure cookie flags, input validation.

### 10.7 Password Attacks

| Attack | Description |
|---|---|
| **Brute force** | Tries every possible password combination |
| **Dictionary attack** | Tries words from a wordlist |
| **Credential stuffing** | Uses leaked username/password pairs from other breaches |
| **Password spraying** | Tries a few common passwords against many accounts (avoids lockout) |
| **Rainbow table** | Precomputed hash-to-password table; defeated by salted hashes |

**Mitigations:** Strong passwords, account lockout policies, MFA, salted hashing (bcrypt, Argon2), breach monitoring.

### 10.8 Malware

| Type | Description |
|---|---|
| **Virus** | Self-replicating code that attaches to legitimate programs |
| **Worm** | Self-propagating; spreads across networks without user action |
| **Trojan** | Disguises itself as legitimate software; creates backdoor |
| **Ransomware** | Encrypts victim's files; demands payment for decryption key |
| **Spyware** | Covertly monitors user activity; steals information |
| **Rootkit** | Hides malicious activity at OS or firmware level |
| **Botnet** | Network of compromised machines (bots) controlled by an attacker (C&C server) |
| **Keylogger** | Records keystrokes to capture credentials |

---

## 11. Firewalls

### 11.1 What is a Firewall?

A **firewall** is a network security device (hardware or software) that **monitors and controls incoming and outgoing network traffic** based on predefined security rules. It establishes a barrier between a trusted internal network and untrusted external networks (e.g., the Internet).

### 11.2 Firewall Types

**1. Packet-Filtering Firewall (Stateless)**

- Operates at the **Network Layer (Layer 3)** and optionally Transport Layer (Layer 4).
- Inspects each packet **independently** based on:
  - Source/destination IP address.
  - Source/destination port number.
  - Protocol (TCP, UDP, ICMP).
- **Stateless:** Does not track connection state; cannot distinguish new connections from established ones.
- Fast but limited — cannot inspect packet content or understand context.

**Advantages:** Simple, fast, low overhead.
**Disadvantages:** Cannot detect attacks that span multiple packets; easy to spoof.

**2. Stateful Inspection Firewall**

- Operates at **Layers 3–4**.
- Maintains a **state table** of all active connections.
- Only allows incoming packets that are part of an already-established outbound connection.
- Can detect TCP session hijacking, invalid flag combinations, and out-of-state packets.

**Advantages:** More secure than stateless; understands connection context.
**Disadvantages:** Cannot inspect application-layer content.

**3. Application Layer Firewall (Proxy Firewall)**

- Operates at **Layer 7 (Application Layer)**.
- Acts as a **proxy**: intercepts all communication between client and server; establishes separate connections on each side.
- Can inspect HTTP content, filter URLs, detect XSS/SQL injection in traffic, enforce application policies.
- Also called a **Web Application Firewall (WAF)** when focused on HTTP/HTTPS.

**Advantages:** Deep inspection; application-aware.
**Disadvantages:** Higher latency; must understand each protocol it inspects.

**4. Next-Generation Firewall (NGFW)**

- Combines stateful inspection + application layer inspection + additional services:
  - **Deep Packet Inspection (DPI):** Inspects payload content.
  - **Intrusion Prevention System (IPS).**
  - **SSL/TLS inspection:** Decrypts TLS traffic for inspection.
  - **User identity awareness:** Applies rules based on user, not just IP.
  - **Threat intelligence integration.**
- Examples: Palo Alto Networks, Fortinet FortiGate, Cisco Firepower, Check Point.

### 11.3 Firewall Rules (ACLs)

Firewalls use **Access Control Lists (ACLs)** — ordered sets of rules evaluated top-to-bottom:

```
Rule #  Action    Protocol  Source          Destination     Port
──────  ──────    ────────  ──────          ───────────     ────
1       ALLOW     TCP       192.168.1.0/24  ANY             80    (HTTP outbound)
2       ALLOW     TCP       192.168.1.0/24  ANY             443   (HTTPS outbound)
3       ALLOW     UDP       192.168.1.0/24  8.8.8.8         53    (DNS to Google)
4       DENY      ANY       ANY             ANY             23    (Block Telnet)
5       DENY      ANY       ANY             ANY             ANY   (Default deny all)
```

**Default policy:** Most firewalls follow a **default deny** (implicit deny all) policy — anything not explicitly permitted is blocked.

### 11.4 DMZ (Demilitarized Zone)

A **DMZ** is a network segment that sits between the trusted internal network and the untrusted Internet, used to host **publicly accessible servers** (web, email, DNS) while protecting the internal network.

```
Internet ──► [Outer Firewall] ──► DMZ (Web/Mail/DNS servers) ──► [Inner Firewall] ──► Internal LAN
```

- The outer firewall allows specific traffic to reach DMZ servers.
- The inner firewall blocks DMZ servers from accessing the internal LAN.
- If a DMZ server is compromised, the attacker cannot directly reach the internal network.

---

## 12. Intrusion Detection and Prevention Systems

### 12.1 IDS vs. IPS

| Feature | IDS (Intrusion Detection System) | IPS (Intrusion Prevention System) |
|---|---|---|
| **Function** | Monitors traffic and **alerts** on suspicious activity | Monitors traffic and **blocks** suspicious activity in real time |
| **Position** | Out-of-band (passive monitor; copy of traffic) | Inline (traffic flows through the IPS) |
| **Response** | Passive — logs and alerts only | Active — drops packets, resets connections, blocks IPs |
| **Risk** | False negatives: attacks may be missed | False positives: legitimate traffic may be blocked |

### 12.2 Detection Methods

**Signature-Based Detection:**
- Compares traffic against a database of known attack signatures (patterns).
- **Advantages:** Very accurate for known attacks; low false positive rate.
- **Disadvantages:** Cannot detect unknown (zero-day) attacks; signature database must be kept up to date.

**Anomaly-Based (Behavioral) Detection:**
- Establishes a **baseline** of normal network behavior; alerts when traffic deviates significantly.
- **Advantages:** Can detect novel and zero-day attacks.
- **Disadvantages:** Higher false positive rate; difficult to establish a reliable baseline.

**Stateful Protocol Analysis:**
- Compares observed protocol behavior against expected protocol state and behavior standards.
- Detects protocol violations and unusual state transitions.

### 12.3 IDS/IPS Placement

```
Internet ──► [Firewall] ──► [IPS (inline)] ──► Internal Network
                                ▲
                          Drops malicious
                          packets before
                          they reach network
```

A **SIEM (Security Information and Event Management)** system aggregates logs from firewalls, IDS/IPS, endpoints, and other sources, correlating events to detect complex attacks and providing centralized visibility.

---

## 13. Virtual Private Networks (VPN)

### 13.1 What is a VPN?

A **VPN (Virtual Private Network)** creates a **secure, encrypted tunnel** through a public network (typically the Internet), allowing remote users or branch offices to communicate as if they were on the same private network.

```
Remote User ──[Encrypted Tunnel through Internet]──► VPN Gateway ──► Corporate Network
```

### 13.2 VPN Use Cases

| Use Case | Description |
|---|---|
| **Remote access VPN** | Individual users connect securely to the corporate network from anywhere |
| **Site-to-site VPN** | Two branch offices connect their networks securely over the Internet |
| **Consumer privacy VPN** | Individual users encrypt their traffic to hide it from ISPs and regional restrictions |

### 13.3 VPN Protocols

**IPSec (Internet Protocol Security):**
- Operates at the **Network Layer (Layer 3)**.
- Provides confidentiality (encryption), integrity, and authentication for IP packets.
- Two modes:
  - **Transport mode:** Encrypts the IP payload only; original IP header is preserved. Used for host-to-host VPNs.
  - **Tunnel mode:** Encrypts the entire original IP packet and encapsulates it in a new IP packet. Used for site-to-site VPNs.
- Two protocols:
  - **AH (Authentication Header):** Provides integrity and authentication; no encryption.
  - **ESP (Encapsulating Security Payload):** Provides integrity, authentication, and encryption. Most commonly used.
- Key exchange: **IKE (Internet Key Exchange)** — negotiates security associations (SAs).

**SSL/TLS VPN:**
- Operates at the **Application Layer** using TLS.
- Accessible through a web browser or lightweight client — no special network configuration needed.
- More firewall-friendly than IPSec (uses HTTPS port 443).
- Examples: OpenVPN (uses TLS), Cisco AnyConnect, Citrix Gateway.

**WireGuard:**
- Modern, lightweight VPN protocol using state-of-the-art cryptography (ChaCha20, Curve25519, BLAKE2).
- Much simpler code base than IPSec (~4,000 lines vs. ~400,000).
- Fast performance; built into the Linux kernel (v5.6+).
- Used by many commercial VPN providers; increasingly popular.

**PPTP (Point-to-Point Tunneling Protocol):**
- Older protocol; **deprecated** — serious security vulnerabilities.
- Should not be used in any security-conscious environment.

**L2TP/IPSec:**
- L2TP provides the tunnel; IPSec provides encryption.
- Widely supported but slower than IKEv2 and WireGuard.

### 13.4 VPN Protocol Comparison

| Protocol | Layer | Security | Speed | Firewall-Friendly | Use Case |
|---|---|---|---|---|---|
| **IPSec (IKEv2)** | Network | Strong | Fast | Moderate | Enterprise site-to-site |
| **SSL/TLS (OpenVPN)** | Application | Strong | Moderate | Yes (port 443) | Remote access |
| **WireGuard** | Network | Very strong | Very fast | Moderate | Modern VPNs |
| **L2TP/IPSec** | Data Link + Network | Strong | Moderate | Moderate | Legacy enterprise |
| **PPTP** | Data Link | Broken | Fast | Yes | **Do not use** |

---

## 14. Network Access Control and Authentication

### 14.1 AAA Framework

**AAA (Authentication, Authorization, and Accounting)** is the framework for controlling network access:

| Component | Purpose | Question Answered |
|---|---|---|
| **Authentication** | Verify the identity of the user or device | "Who are you?" |
| **Authorization** | Determine what the authenticated entity is permitted to do | "What can you do?" |
| **Accounting** | Record what the entity did; for auditing and billing | "What did you do?" |

**AAA Protocols:**

| Protocol | Description |
|---|---|
| **RADIUS (Remote Authentication Dial-In User Service)** | RFC 2865; client-server; UDP; used for network access (Wi-Fi, VPN, dial-up); encrypts passwords only |
| **TACACS+ (Terminal Access Controller Access Control System Plus)** | Cisco proprietary; TCP; encrypts entire payload; separates AAA; used for device administration |
| **Diameter** | Successor to RADIUS; TCP/SCTP; used in 4G/5G LTE core networks |

### 14.2 Authentication Factors

**Authentication factors** are categories of credentials used to verify identity:

| Factor | Type | Example |
|---|---|---|
| **Something you know** | Knowledge | Password, PIN, security question |
| **Something you have** | Possession | Smart card, hardware token (YubiKey), phone (OTP app) |
| **Something you are** | Inherence/Biometric | Fingerprint, face scan, iris scan, voice |
| **Somewhere you are** | Location | GPS location, IP address range |
| **Something you do** | Behavior | Typing pattern, walking gait |

**Multi-Factor Authentication (MFA):** Requires two or more factors from different categories. Even if one factor is compromised (e.g., password stolen), the account remains protected.

**Single Sign-On (SSO):** A user authenticates once and gains access to multiple systems without re-authenticating. Implemented via protocols like:
- **SAML (Security Assertion Markup Language)** — XML-based; enterprise SSO (Okta, Azure AD).
- **OAuth 2.0** — Authorization framework; allows third-party apps to access resources on behalf of a user.
- **OpenID Connect (OIDC)** — Authentication layer on top of OAuth 2.0; used by Google Sign-In, Apple Sign-In.
- **Kerberos** — Ticket-based authentication for Active Directory (Windows domain environments).

### 14.3 IEEE 802.1X (Port-Based Network Access Control)

**802.1X** is a standard for **port-based access control** — a device cannot access the network until it is authenticated.

**Components:**
- **Supplicant:** The client device requesting access.
- **Authenticator:** The network device (switch, AP) that enforces access control.
- **Authentication Server:** Typically a RADIUS server that verifies credentials.

**Protocol:** Uses **EAP (Extensible Authentication Protocol)** transported over LAN (EAPOL) to the authenticator, then RADIUS to the authentication server.

```
[Supplicant] ──EAPOL──► [Authenticator/Switch] ──RADIUS──► [Auth Server]
                                ↓
                      Port blocked until
                      authentication succeeds
```

---

## 15. Wireless Security

### 15.1 Wireless Security Challenges

Wireless networks present unique security challenges:
- The **medium is shared and open** — anyone within radio range can receive (and attempt to transmit) signals.
- **No physical perimeter** — an attacker can be in a parking lot or adjacent building.
- **Rogue access points** — attackers can set up fake APs to intercept traffic (evil twin attacks).

### 15.2 Wi-Fi Security Protocols

**WEP (Wired Equivalent Privacy):**
- First Wi-Fi security standard; used RC4 stream cipher.
- **Completely broken** — vulnerable to passive attacks; should never be used.
- A WEP key can be cracked in minutes with freely available tools.

**WPA (Wi-Fi Protected Access):**
- Interim standard replacing WEP; used **TKIP (Temporal Key Integrity Protocol)** to fix WEP's key reuse flaws.
- Better than WEP but TKIP has known vulnerabilities.
- **Legacy** — should not be used for new deployments.

**WPA2 (IEEE 802.11i):**
- Replaced TKIP with **AES-CCMP** — a strong, standardized encryption algorithm.
- Two modes:
  - **WPA2-Personal (WPA2-PSK):** Uses a **Pre-Shared Key** (Wi-Fi password) for home/small office.
  - **WPA2-Enterprise:** Uses **802.1X** and a RADIUS server for per-user authentication (corporate).
- **KRACK vulnerability** (2017) affected WPA2 but was mitigated by patches.

**WPA3 (IEEE 802.11ax):**
- Current standard; significantly stronger than WPA2.
- **WPA3-Personal:** Replaces PSK with **SAE (Simultaneous Authentication of Equals)** — a password-authenticated key exchange that prevents offline dictionary attacks even with a weak password.
- **WPA3-Enterprise:** Adds 192-bit security suite for high-security environments.
- **Forward secrecy:** Session keys cannot be recovered even if the password is later compromised.
- **OWE (Opportunistic Wireless Encryption):** Encrypts open networks (no password) — used in coffee shops, airports.

### 15.3 Wi-Fi Security Best Practices

| Practice | Reason |
|---|---|
| Use WPA3 (or WPA2 at minimum) | Strong encryption and authentication |
| Use strong, unique Wi-Fi passwords | Prevents brute-force attacks on PSK |
| Disable WPS (Wi-Fi Protected Setup) | WPS PIN is vulnerable to brute force |
| Use WPA2/WPA3-Enterprise for business | Per-user credentials; 802.1X authentication |
| Segment guest Wi-Fi onto a separate VLAN | Prevents guests from accessing internal resources |
| Enable network monitoring and rogue AP detection | Detect evil twin and unauthorized APs |
| Disable unused SSIDs and bands | Reduce attack surface |
| Use a VPN on untrusted Wi-Fi networks | Protects traffic even on compromised networks |

### 15.4 Wireless Attack Types

| Attack | Description |
|---|---|
| **Evil Twin** | Attacker creates a rogue AP with the same SSID as a legitimate network |
| **Deauthentication attack** | Sends forged 802.11 deauth frames to disconnect clients from an AP |
| **PMKID attack** | Captures PMKID from WPA2 4-way handshake initiation; allows offline password cracking |
| **WPS PIN brute force** | Exploits the WPS PIN mechanism to recover the Wi-Fi password |
| **Wardriving** | Driving around scanning for open/weak wireless networks |
| **Karma attack** | Rogue AP responds to any probe request, luring clients to connect automatically |

---

## 16. Summary

This module covered the Application Layer and Network Security:

**Application Layer:**
- The Application Layer provides network services directly to applications, using the client-server or P2P model.
- **DNS** resolves domain names to IPs through a hierarchical recursive process; secured by DNSSEC, DoH, and DoT.
- **HTTP/HTTPS** drives the web; HTTP/2 adds multiplexing; HTTP/3 moves to QUIC for HoL blocking elimination.
- **Email** uses SMTP for sending, IMAP/POP3 for retrieval, and SPF/DKIM/DMARC for anti-spoofing.
- **FTP** is insecure; **SFTP** (over SSH) and **FTPS** (over TLS) are secure alternatives; **TFTP** is simple/UDP-based for network booting.
- **SSH** provides encrypted remote access, replacing the insecure Telnet.
- **SNMP** manages network devices; SNMPv3 with authPriv is the secure version.

**Network Security:**
- The **CIA Triad** (Confidentiality, Integrity, Availability) is the foundation of security.
- Common attacks: **DoS/DDoS, MITM, phishing, replay, SQL injection, XSS, malware.**
- **Firewalls** control traffic by packet filtering (stateless), stateful inspection, application proxy, or NGFW capabilities.
- **IDS** detects and alerts; **IPS** detects and blocks; both use signature-based and anomaly-based detection.
- **VPNs** create encrypted tunnels: IPSec for network-layer VPNs, SSL/TLS for remote access, WireGuard for modern high-performance use.
- **AAA** (RADIUS, TACACS+) manages authentication, authorization, and accounting for network access.
- **MFA** and **802.1X** strengthen authentication for users and devices.
- **WPA3** is the current wireless security standard; WEP and WPA are deprecated.

---

## 17. Key Terms

| Term | Definition |
|---|---|
| **Application Layer** | OSI Layer 7 / TCP/IP top layer; provides network services to applications |
| **DNS** | Domain Name System; translates domain names to IP addresses |
| **FQDN** | Fully Qualified Domain Name; complete domain name including all hierarchy levels |
| **Recursive resolver** | DNS server that performs the full resolution on behalf of the client |
| **Authoritative name server** | DNS server holding the actual records for a domain |
| **DNSSEC** | DNS Security Extensions; digitally signs DNS records to prevent spoofing |
| **HTTP** | Hypertext Transfer Protocol; application protocol for web communication |
| **HTTPS** | HTTP over TLS; encrypted web communication |
| **HSTS** | HTTP Strict Transport Security; forces HTTPS use for a domain |
| **HTTP/2** | Binary, multiplexed HTTP over TCP with header compression |
| **HTTP/3** | HTTP over QUIC (UDP-based); eliminates TCP head-of-line blocking |
| **SMTP** | Simple Mail Transfer Protocol; sends and relays email (port 25/587) |
| **IMAP** | Internet Message Access Protocol; accesses email on server (port 143/993) |
| **POP3** | Post Office Protocol v3; downloads email from server (port 110/995) |
| **SPF** | Sender Policy Framework; DNS record specifying authorized mail senders |
| **DKIM** | DomainKeys Identified Mail; digitally signs outgoing email |
| **DMARC** | Email authentication policy framework building on SPF and DKIM |
| **FTP** | File Transfer Protocol; unencrypted file transfer (port 20/21) |
| **SFTP** | SSH File Transfer Protocol; secure file transfer over SSH (port 22) |
| **TFTP** | Trivial File Transfer Protocol; simple UDP-based file transfer (port 69) |
| **SSH** | Secure Shell; encrypted remote access and tunneling (port 22) |
| **Telnet** | Unencrypted remote terminal protocol (port 23); deprecated |
| **SNMP** | Simple Network Management Protocol; monitors/manages network devices (port 161) |
| **MIB** | Management Information Base; hierarchical database of manageable objects |
| **CIA Triad** | Confidentiality, Integrity, Availability — the foundation of information security |
| **DoS / DDoS** | Denial of Service / Distributed DoS; attack to make a service unavailable |
| **MITM** | Man-in-the-Middle; attacker secretly intercepts and relays communication |
| **Phishing** | Fraudulent communication to trick users into revealing credentials |
| **SQL injection** | Attack inserting malicious SQL into input fields to manipulate databases |
| **XSS** | Cross-Site Scripting; injecting malicious scripts into web pages |
| **Firewall** | Device/software controlling network traffic based on security rules |
| **Stateful inspection** | Firewall tracking connection state to allow only legitimate return traffic |
| **NGFW** | Next-Generation Firewall; combines stateful inspection, IPS, DPI, and application awareness |
| **DMZ** | Demilitarized Zone; isolated network segment for publicly accessible servers |
| **IDS** | Intrusion Detection System; monitors and alerts on suspicious activity |
| **IPS** | Intrusion Prevention System; monitors and blocks suspicious activity inline |
| **VPN** | Virtual Private Network; encrypted tunnel over a public network |
| **IPSec** | Internet Protocol Security; Network Layer encryption for VPNs |
| **WireGuard** | Modern lightweight VPN protocol with strong cryptography |
| **AAA** | Authentication, Authorization, Accounting; framework for access control |
| **RADIUS** | Remote Authentication Dial-In User Service; AAA protocol for network access |
| **MFA** | Multi-Factor Authentication; requires multiple credential categories |
| **802.1X** | Port-based network access control standard using EAP and RADIUS |
| **WPA2** | Wi-Fi Protected Access 2; uses AES-CCMP; current widely deployed standard |
| **WPA3** | Wi-Fi Protected Access 3; uses SAE; current best Wi-Fi security standard |
| **SAE** | Simultaneous Authentication of Equals; WPA3 key exchange resistant to dictionary attacks |

---

## 18. Review Questions

**Conceptual Questions:**

1. Explain the difference between the Application Layer and the application itself. Why is this distinction important when studying network protocols?
2. Trace the full DNS resolution process for the domain `mail.university.edu`, starting from a user's browser on a fresh system with an empty cache. Identify every participant in the process and what each does.
3. Compare HTTP/1.1, HTTP/2, and HTTP/3. What specific problem does each version solve that the previous version did not? Why does HTTP/3 use QUIC instead of TCP?
4. Explain the roles of SMTP, IMAP, and POP3 in email communication. Why would a corporate environment prefer IMAP over POP3 for employee email access?
5. Why is FTP considered insecure? In what situations might SFTP be preferred over FTPS, and vice versa?
6. Explain the CIA Triad. For each property, give one example of a network attack that violates it and one countermeasure that protects it.
7. Compare signature-based and anomaly-based IDS detection methods. What type of attack can each detect that the other cannot? What is the main drawback of each?
8. Explain the difference between IPSec Transport Mode and Tunnel Mode. In which scenario would each be used?
9. Why is WEP considered completely broken? What specific improvements does WPA3's SAE mechanism make over WPA2's Pre-Shared Key system?
10. What is 802.1X, and why is it important for enterprise network security? Describe the role of each of its three components (supplicant, authenticator, authentication server).

**Application / Analysis Questions:**

11. A company's web server responds with the following HTTP headers:
    ```
    HTTP/2 200 OK
    Content-Type: application/json
    Content-Encoding: br
    Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
    X-Content-Type-Options: nosniff
    ```
    For each header: (a) identify its purpose, (b) identify which security risk or performance concern it addresses.

12. An attacker performs an ARP poisoning attack on a LAN to intercept traffic between a user (`192.168.1.50`) and the gateway (`192.168.1.1`). Describe step-by-step how the attack works. What does the attacker's ARP cache table look like after the attack? What countermeasures can the network administrator deploy?

13. Analyze the following SQL query vulnerability:
    ```sql
    SELECT * FROM accounts WHERE user='" + username + "' AND pass='" + password + "'";
    ```
    (a) What input would an attacker provide for `username` to log in without a valid password? (b) Write the resulting SQL query and explain why it succeeds. (c) Rewrite the query using a parameterized statement (in any language).

14. A network administrator must choose between deploying an IDS and an IPS for a hospital network that handles patient records. (a) What are the trade-offs of each choice in this context? (b) Which would you recommend and why? (c) Where in the network architecture would you place your chosen solution?

15. Calculate the total DNS query resolution time for `www.shop.example.com` given the following delays (round-trip):
    - Client → Recursive Resolver: 5 ms
    - Resolver → Root NS: 80 ms
    - Resolver → .com TLD NS: 60 ms
    - Resolver → example.com Auth NS: 40 ms
    - All records have a TTL of 3600 seconds.
    What is the resolution time for the first query? What is the resolution time if the same user queries the same domain 30 minutes later? Explain the difference.

16. A penetration tester discovers that an internal web application reflects user input directly into the page without sanitization. Write an example XSS payload that would steal a session cookie and send it to an attacker's server at `evil.com`. Then describe three mitigations the development team should implement to prevent this attack.

17. Compare RADIUS and TACACS+ across the following dimensions: transport protocol, encryption scope, AAA separation, and primary use case. In a scenario where a network administrator needs to manage configuration access to 500 Cisco routers with granular per-command authorization, which protocol would be more appropriate and why?

---

*End of Module 7 | Next: Module 8 – Introduction to Next-Generation Communication Technologies*
