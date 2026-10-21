# CS 2204 – Communications and Networking
## Module 6: Session Layer and Presentation Layer

---

## Table of Contents

1. [Overview](#1-overview)
2. [The Session Layer (OSI Layer 5)](#2-the-session-layer-osi-layer-5)
3. [Session Layer Functions](#3-session-layer-functions)
4. [Session Layer Protocols and Real-World Mapping](#4-session-layer-protocols-and-real-world-mapping)
5. [The Presentation Layer (OSI Layer 6)](#5-the-presentation-layer-osi-layer-6)
6. [Data Translation and Encoding](#6-data-translation-and-encoding)
7. [Data Compression](#7-data-compression)
8. [Cryptography and Encryption at the Presentation Layer](#8-cryptography-and-encryption-at-the-presentation-layer)
9. [Data Formats and Standards](#9-data-formats-and-standards)
10. [The Relationship Between Layers 5, 6, and 7 in Practice](#10-the-relationship-between-layers-5-6-and-7-in-practice)
11. [Character Encoding Standards](#11-character-encoding-standards)
12. [Multimedia Encoding and Compression](#12-multimedia-encoding-and-compression)
13. [Abstract Syntax Notation (ASN.1)](#13-abstract-syntax-notation-asn1)
14. [Summary](#14-summary)
15. [Key Terms](#15-key-terms)
16. [Review Questions](#16-review-questions)

---

## 1. Overview

The Session Layer and Presentation Layer are the fifth and sixth layers of the OSI model. They are frequently the least discussed in practical networking because, in the dominant TCP/IP model, their functions are largely absorbed into the Application Layer. Nevertheless, understanding these layers is essential for grasping how networked applications manage communication sessions, represent data in a common format, and secure data in transit.

This module examines both layers in depth — their theoretical roles in the OSI model and their practical manifestations in real protocols and systems.

**Module Learning Objectives:**

By the end of this module, students should be able to:

- Describe the responsibilities of the Session Layer (OSI Layer 5).
- Explain session establishment, maintenance, synchronization, and termination.
- Identify real-world protocols that implement Session Layer functions.
- Describe the responsibilities of the Presentation Layer (OSI Layer 6).
- Explain data translation, encoding, compression, and encryption as Presentation Layer functions.
- Compare character encoding standards (ASCII, Unicode, UTF-8).
- Describe common data serialization formats (XML, JSON, ASN.1).
- Explain symmetric and asymmetric encryption concepts as they apply to the Presentation Layer.
- Identify common multimedia compression standards (JPEG, MPEG, MP3).
- Explain how Layers 5 and 6 map to real-world TCP/IP implementations.

---

## 2. The Session Layer (OSI Layer 5)

### 2.1 Position in the OSI Stack

```
┌───────────────────────────────┐
│   Layer 7 – Application       │
├───────────────────────────────┤
│   Layer 6 – Presentation      │
├───────────────────────────────┤
│   Layer 5 – Session           │  ← Focus of this section
├───────────────────────────────┤
│   Layer 4 – Transport         │
├───────────────────────────────┤
│   Layer 3 – Network           │
├───────────────────────────────┤
│   Layer 2 – Data Link         │
├───────────────────────────────┤
│   Layer 1 – Physical          │
└───────────────────────────────┘
```

### 2.2 What is a Session?

A **session** is a semi-permanent, interactive information exchange between two or more communicating devices. Unlike a Transport Layer connection (which concerns itself with reliable byte delivery), a session provides a **structured dialogue** — organizing and controlling the interaction between application processes over a potentially long period.

Think of a session as the equivalent of a phone call:
- The Transport Layer is the telephone line (physical connection).
- The Session Layer is the conversation itself — who speaks when, how the call is placed on hold, and how it is ended gracefully.

### 2.3 Session vs. Connection vs. Communication

| Concept | Layer | Scope |
|---|---|---|
| **Communication link** | Physical / Data Link | Raw bit/frame transfer between adjacent nodes |
| **Connection** | Transport | End-to-end byte-stream delivery (TCP) |
| **Session** | Session | Structured application dialogue with state |

---

## 3. Session Layer Functions

The Session Layer is responsible for six key functions:

### 3.1 Session Establishment

The Session Layer sets up a **session** between two communicating application processes:

- **Authentication:** Verifies the identity of both communicating parties before the session begins.
- **Authorization:** Verifies that the authenticated parties have permission to use the requested services.
- **Session negotiation:** Establishes parameters such as which party initiates communication, the type of dialogue (simplex, half-duplex, full-duplex), and session identifiers.

A session identifier allows a session to be **uniquely identified** and later **resumed** after an interruption, without re-authentication.

### 3.2 Session Maintenance

Once established, the Session Layer keeps the session alive and organized:

- **Keep-alive mechanisms:** Sends periodic signals to confirm both parties are still active, preventing the session from timing out during long pauses.
- **Re-establishment:** If the underlying connection is lost temporarily (e.g., due to a network interruption), the Session Layer can re-establish the Transport Layer connection and resume the session from where it left off, rather than starting over.
- **Session tokens:** Some implementations use tokens to control who has the right to transmit at any given moment (important in half-duplex communication).

### 3.3 Dialog Control

The Session Layer manages the **direction of data flow** between communicating parties:

| Mode | Description | Example |
|---|---|---|
| **Simplex** | One party transmits; the other only receives | Broadcast, monitoring |
| **Half-duplex** | Both parties can transmit, but not simultaneously; a token controls turns | Older videoconferencing, walkie-talkies |
| **Full-duplex** | Both parties can transmit simultaneously | Modern TCP-based applications |

In **half-duplex** sessions, the Session Layer uses a **data token** to coordinate who may transmit. Only the party holding the token may send data. When done, it passes (or releases) the token.

### 3.4 Synchronization and Checkpoints

One of the Session Layer's most important (and distinctive) functions is **synchronization** — inserting **checkpoints** (also called synchronization points) into the data stream so that, in the event of a failure, only the data after the last checkpoint needs to be retransmitted.

**Types of synchronization points:**

| Type | Description |
|---|---|
| **Major synchronization point** | A confirmed checkpoint — both parties must acknowledge it. Data before this point is committed and will not be retransmitted |
| **Minor synchronization point** | An unconfirmed checkpoint — used to structure data within a dialogue unit; does not require explicit acknowledgment |

**Example — File Transfer:**
A 1 GB file transfer could insert major checkpoints every 100 MB. If the connection drops at 650 MB, the transfer resumes from the 600 MB checkpoint (the last confirmed major sync point) rather than restarting from the beginning.

```
[0 MB]──►[100 MB]──►[200 MB]──►... ──►[600 MB]──►[650 MB ← FAILURE]
          Major      Major              Major
          sync       sync               sync ✓
                                        ↑ Resume from here
```

### 3.5 Session Termination

The Session Layer provides **orderly termination** of a session:

- **Graceful termination:** Both parties agree to end the session; any remaining data is flushed and delivered before closure.
- **Abrupt termination:** One party (or the network) closes the session without completing the exchange — data may be lost.
- **Release tokens:** Resources allocated for the session (memory, state, credentials) are freed.

### 3.6 Exception Reporting

The Session Layer reports certain conditions to the Presentation Layer:

- Notification of session interruptions.
- Notification of timeouts or failed keep-alive responses.
- Reporting of unrecoverable errors that cannot be handled at this layer.

---

## 4. Session Layer Protocols and Real-World Mapping

### 4.1 Why the Session Layer is "Invisible" in TCP/IP

The TCP/IP model does not have a distinct Session Layer. Instead, session-like functions are:

- **Embedded in application protocols** (e.g., HTTP cookies maintain session state; FTP session commands).
- **Handled by the Transport Layer** (TCP connections provide some session-like properties).
- **Managed by dedicated middleware** (e.g., RPC frameworks, web application frameworks).

This is why students studying practical networking often encounter the OSI Session Layer mainly as a conceptual framework rather than a distinct protocol layer.

### 4.2 Protocols with Session Layer Characteristics

| Protocol | Layer in TCP/IP | Session Layer Functions It Implements |
|---|---|---|
| **NetBIOS** | Application | Session establishment and naming between Windows hosts |
| **RPC (Remote Procedure Call)** | Application | Session management for distributed function calls |
| **NFS (Network File System)** | Application | Maintains session state for remote file access |
| **SMB (Server Message Block)** | Application | Windows file sharing; session establishment and authentication |
| **H.323** | Application | Multimedia session control (VoIP, videoconferencing) |
| **SIP (Session Initiation Protocol)** | Application | Establishes, modifies, and terminates VoIP/multimedia sessions |
| **PPTP / L2TP** | Network/Transport | Tunnel sessions for VPN connections |
| **SQL sessions** | Application | Database connections maintain session state across queries |
| **TLS session resumption** | Between Transport and Application | Resumes a prior TLS session without full handshake |

### 4.3 SIP: A Practical Session Layer Example

**SIP (Session Initiation Protocol)**, defined in RFC 3261, is one of the clearest real-world implementations of Session Layer concepts.

SIP manages **multimedia communication sessions** (voice, video, instant messaging):

**SIP Session Lifecycle:**

```
Caller (Alice)                          Callee (Bob)
──────────────                          ────────────
INVITE  ─────────────────────────────►
        ◄─────────────────────────────  100 Trying
        ◄─────────────────────────────  180 Ringing
        ◄─────────────────────────────  200 OK
ACK     ─────────────────────────────►
        ══ Media Session (RTP/UDP) ══
BYE     ─────────────────────────────►
        ◄─────────────────────────────  200 OK
```

- **INVITE:** Initiates the session (like the 3-way handshake at the Session Layer level).
- **ACK:** Confirms session establishment.
- **BYE:** Gracefully terminates the session.
- Actual media (audio/video) flows separately over **RTP (Real-time Transport Protocol)** on UDP.

### 4.4 HTTP Sessions and Cookies

HTTP is stateless by design — each request is independent. **Session state** is maintained at the application level through:

- **Cookies:** Small pieces of state stored in the browser; sent with each request to identify the session.
- **Session tokens:** A unique identifier (stored in a cookie or URL parameter) that maps to server-side session data.
- **JWT (JSON Web Tokens):** A self-contained token encoding session data, signed to prevent tampering.

```
Client                                    Server
──────                                    ──────
GET /login ──────────────────────────────►
           ◄──────────────────────────── Set-Cookie: session_id=abc123
GET /dashboard (Cookie: session_id=abc123) ──►
           ◄──────────────────────────── Dashboard page (user identified)
```

---

## 5. The Presentation Layer (OSI Layer 6)

### 5.1 What is the Presentation Layer?

The **Presentation Layer (OSI Layer 6)** is responsible for the **syntax and semantics** of the data exchanged between applications. Its job is to ensure that information sent by the Application Layer of one system is readable by the Application Layer of another — even if the two systems use different internal data representations.

Think of the Presentation Layer as a **translator and formatter**: it converts data between the format used by the application and a common network format.

### 5.2 Core Responsibilities

| Responsibility | Description |
|---|---|
| **Translation / Data conversion** | Converts data from sender's format to a common network format and from network format to receiver's format |
| **Encryption / Decryption** | Encrypts outgoing data for confidentiality; decrypts incoming data |
| **Compression / Decompression** | Reduces data size for efficient transmission; decompresses at the receiver |
| **Data formatting** | Ensures data is structured in a way the receiving application can interpret (e.g., character encoding, image formats, audio formats) |

### 5.3 The Translation Problem

Different computer systems represent data differently at the hardware and OS level:

| Issue | Example |
|---|---|
| **Character encoding** | ASCII (7-bit) vs. EBCDIC (IBM mainframes) vs. Unicode |
| **Byte order (endianness)** | Big-endian (network byte order) vs. little-endian (x86 processors) |
| **Integer representation** | Signed magnitude vs. two's complement |
| **Floating-point format** | IEEE 754 (most systems) vs. proprietary formats |
| **Date/time format** | ISO 8601 (`2024-01-15`) vs. US format (`01/15/2024`) vs. Unix timestamp |

The Presentation Layer translates between these formats so that data sent by one system is correctly interpreted by another.

---

## 6. Data Translation and Encoding

### 6.1 Byte Order (Endianness)

**Endianness** refers to the order in which bytes of a multi-byte value are stored in memory or transmitted over a network.

**Big-endian (network byte order):**
- Most significant byte stored/sent first.
- Used by: Network protocols (IP, TCP), SPARC processors, IBM mainframes.

**Little-endian:**
- Least significant byte stored first.
- Used by: x86/x64 processors (Intel, AMD), most modern PCs.

**Example — The 32-bit value `0x12345678`:**

| Byte Address | Big-Endian | Little-Endian |
|---|---|---|
| 0 (first) | 0x12 | 0x78 |
| 1 | 0x34 | 0x56 |
| 2 | 0x56 | 0x34 |
| 3 (last) | 0x78 | 0x12 |

Network protocols use **big-endian (network byte order)**. Applications on little-endian machines must call conversion functions (`htons()`, `htonl()`, `ntohs()`, `ntohl()`) to convert between host and network byte order.

### 6.2 Data Serialization

**Serialization** is the process of converting structured data (objects, records, data structures) into a flat sequence of bytes suitable for transmission or storage.

**Deserialization** is the reverse process — reconstructing the original data structure from the byte sequence at the receiver.

Common serialization formats:

| Format | Type | Description | Use |
|---|---|---|---|
| **JSON** | Text-based | Human-readable; key-value pairs; widely used in web APIs | REST APIs, configuration files |
| **XML** | Text-based | Hierarchical markup; verbose; self-describing | Web services (SOAP), configuration |
| **Protocol Buffers (protobuf)** | Binary | Google's compact binary serialization | gRPC, internal Google services |
| **MessagePack** | Binary | Binary JSON; more compact than JSON | High-performance APIs |
| **ASN.1 / BER/DER** | Binary | Standard for telecom and security protocols | X.509 certificates, SNMP, LDAP |
| **CBOR** | Binary | Concise Binary Object Representation; IoT-focused | IoT, COSE |
| **Avro** | Binary | Apache; schema-based; used in big data | Kafka, Hadoop |

---

## 7. Data Compression

### 7.1 Why Compress?

Data compression reduces the number of bits needed to represent data, resulting in:
- **Faster transmission** — fewer bits to send over a fixed-bandwidth link.
- **Lower storage requirements** — more data fits in the same space.
- **Reduced cost** — less bandwidth consumed.

### 7.2 Lossless vs. Lossy Compression

| Type | Description | Data Recovery | Use Cases |
|---|---|---|---|
| **Lossless** | Original data is perfectly reconstructed after decompression | Exact | Text, executables, financial data, PNG images |
| **Lossy** | Some data is permanently discarded; quality is reduced | Approximate | Images (JPEG), audio (MP3), video (H.264) |

### 7.3 Lossless Compression Algorithms

**Run-Length Encoding (RLE):**
- Replaces consecutive repeated values with a count and the value.
- Best for data with long runs of identical bytes.
- Example: `AAAAAABBBCC` → `6A3B2C`

**Huffman Coding:**
- Assigns shorter binary codes to more frequent symbols and longer codes to rare symbols.
- Variable-length codes; optimal for known symbol frequencies.
- Used in: DEFLATE (ZIP, gzip, PNG), JPEG (for header), MP3.

**LZ77 / LZ78 / LZW:**
- Dictionary-based compression — replaces repeated patterns with references to a dictionary.
- LZW used in: GIF images.
- LZ77 is the basis for: ZIP, gzip, zlib, DEFLATE.

**DEFLATE:**
- Combines LZ77 + Huffman coding.
- Used in: ZIP, gzip, zlib, PNG, HTTP content encoding (`Content-Encoding: gzip`).

### 7.4 Lossy Compression Principles

Lossy compression exploits limitations of human perception:

- **Psychoacoustics (audio):** Human hearing is less sensitive to high-frequency components and quieter sounds masked by louder ones. MP3/AAC discard inaudible frequencies.
- **Psychovisual (image/video):** Human vision is more sensitive to changes in luminance (brightness) than chrominance (color). JPEG reduces color resolution while preserving brightness detail.
- **Temporal redundancy (video):** Consecutive video frames are often very similar. Codecs only encode the differences between frames (inter-frame coding) rather than full frames every time.

---

## 8. Cryptography and Encryption at the Presentation Layer

### 8.1 Why Encryption at the Presentation Layer?

The Presentation Layer is conceptually responsible for **encrypting outgoing data** and **decrypting incoming data** to ensure confidentiality. In the OSI model, this transformation happens before the data is passed to the Session Layer for transmission.

In practice (TCP/IP), this is handled by **TLS/SSL** (between Transport and Application layers) or by application-level encryption.

### 8.2 Fundamental Cryptographic Concepts

**Plaintext:** The original, readable data.
**Ciphertext:** The encrypted, unreadable data.
**Key:** A secret value used to control the encryption/decryption algorithm.
**Cipher:** The algorithm used to encrypt or decrypt data.

### 8.3 Symmetric Encryption

In **symmetric encryption**, the same key is used for both encryption and decryption.

```
Plaintext + Key  →  [Encryption]  →  Ciphertext
Ciphertext + Key →  [Decryption]  →  Plaintext
```

**Properties:**
- Very fast; efficient for large data volumes.
- The key must be shared securely between parties in advance (**key distribution problem**).
- If the key is compromised, all encrypted data is exposed.

**Common Symmetric Algorithms:**

| Algorithm | Key Size | Block Size | Status | Notes |
|---|---|---|---|---|
| **DES** | 56 bits | 64 bits | Deprecated | Too short; broken by brute force |
| **3DES (Triple DES)** | 112/168 bits | 64 bits | Being phased out | DES applied 3 times; slow |
| **AES** | 128/192/256 bits | 128 bits | Current standard | Fast; used in TLS, Wi-Fi (WPA2/3), file encryption |
| **RC4** | 40–2048 bits | Stream | Deprecated | Used in old WEP, SSLv3; vulnerabilities found |
| **ChaCha20** | 256 bits | Stream | Modern | Used in TLS 1.3; fast on hardware without AES acceleration |

**Modes of Operation (for block ciphers like AES):**

| Mode | Name | Description | Use |
|---|---|---|---|
| **ECB** | Electronic Codebook | Same plaintext block → same ciphertext block | Never use (patterns visible) |
| **CBC** | Cipher Block Chaining | Each block XORed with previous ciphertext; uses IV | TLS 1.2 (legacy) |
| **CTR** | Counter Mode | Turns block cipher into stream cipher using counter | SSH, TLS |
| **GCM** | Galois/Counter Mode | CTR + authentication tag; provides AEAD | TLS 1.3, AES-GCM |

> **AEAD (Authenticated Encryption with Associated Data):** Provides both confidentiality and integrity in one operation. AES-GCM and ChaCha20-Poly1305 are the two AEAD ciphers in TLS 1.3.

### 8.4 Asymmetric Encryption (Public-Key Cryptography)

In **asymmetric encryption**, two mathematically related keys are used:
- **Public key:** Shared openly; used to encrypt data or verify signatures.
- **Private key:** Kept secret; used to decrypt data or create signatures.

```
Sender encrypts with receiver's PUBLIC key  →  Ciphertext
Receiver decrypts with their own PRIVATE key →  Plaintext
```

**Key property:** Data encrypted with the public key can only be decrypted with the corresponding private key — and vice versa.

**Common Asymmetric Algorithms:**

| Algorithm | Based On | Key Size | Use |
|---|---|---|---|
| **RSA** | Integer factorization | 2048–4096 bits | Key exchange, digital signatures, certificates |
| **DSA** | Discrete logarithm | 1024–3072 bits | Digital signatures only |
| **ECDSA** | Elliptic curve discrete log | 256–521 bits | Faster than RSA; used in TLS, SSH, Bitcoin |
| **ECDH / ECDHE** | Elliptic curve DH | 256–521 bits | Key exchange in TLS (forward secrecy) |
| **Ed25519** | Edwards curve | 256 bits | Fast, modern digital signatures; SSH, TLS |

**Asymmetric vs. Symmetric:**

| Feature | Symmetric | Asymmetric |
|---|---|---|
| Keys | One shared key | Public/private key pair |
| Speed | Fast | Slow (100–1000× slower) |
| Key distribution | Problem (must share securely) | Solved (public key is openly shared) |
| Use | Bulk data encryption | Key exchange, digital signatures, authentication |

**In practice:** Asymmetric encryption is used to securely exchange a symmetric session key. All bulk data is then encrypted with the symmetric key (this hybrid approach is used in TLS).

### 8.5 Hashing and Message Integrity

A **cryptographic hash function** produces a fixed-size **digest (hash)** from any input. Properties:

- **Deterministic:** Same input always gives the same hash.
- **One-way:** Cannot reverse the hash to find the input (pre-image resistance).
- **Avalanche effect:** A tiny change in input produces a completely different hash.
- **Collision resistant:** Computationally infeasible to find two inputs with the same hash.

**Common Hash Algorithms:**

| Algorithm | Output Size | Status | Use |
|---|---|---|---|
| **MD5** | 128 bits | Broken (collisions found) | Legacy; file checksums (non-security) |
| **SHA-1** | 160 bits | Deprecated (collisions found) | Legacy certificates; phased out |
| **SHA-256** | 256 bits | Current standard | TLS, certificates, Bitcoin, code signing |
| **SHA-384 / SHA-512** | 384/512 bits | Current standard | High-security applications |
| **SHA-3** | Variable | Modern alternative | Post-SHA-2 standard; different algorithm family |
| **BLAKE2 / BLAKE3** | Variable | Modern | Faster than SHA-256; used in many tools |

**HMAC (Hash-based Message Authentication Code):**
Combines a hash function with a secret key to provide both **integrity** and **authentication**:
```
HMAC(key, message) = Hash(key ⊕ opad || Hash(key ⊕ ipad || message))
```
Used in TLS for record integrity, API authentication, JWT signatures.

### 8.6 Digital Signatures

A **digital signature** provides:
- **Authentication** — proves the sender's identity.
- **Integrity** — proves the message was not altered.
- **Non-repudiation** — the sender cannot deny sending the message.

**How it works:**
```
Sender:
  1. Hash the message: H = Hash(message)
  2. Encrypt the hash with private key: Signature = Encrypt(H, private_key)
  3. Send: message + Signature

Receiver:
  1. Hash the received message: H' = Hash(message)
  2. Decrypt the signature with sender's public key: H = Decrypt(Signature, public_key)
  3. If H == H' → Signature is valid (message is authentic and unaltered)
```

### 8.7 Public Key Infrastructure (PKI) and X.509 Certificates

How does a receiver know that a public key actually belongs to who claims to own it? This is solved by **PKI (Public Key Infrastructure)**.

A **digital certificate (X.509)** is a digitally signed document that binds:
- A **public key** to an **identity** (domain name, organization, person).
- Issued and signed by a trusted **Certificate Authority (CA)**.

**X.509 Certificate Contents:**

| Field | Description |
|---|---|
| **Version** | X.509 version (usually v3) |
| **Serial Number** | Unique identifier assigned by the CA |
| **Subject** | Identity the certificate belongs to (e.g., `CN=www.example.com`) |
| **Issuer** | The CA that signed the certificate |
| **Validity period** | Not Before / Not After dates |
| **Public Key** | The subject's public key |
| **Subject Alternative Names (SAN)** | Additional domain names covered by the certificate |
| **Signature** | CA's digital signature over the certificate |

**Certificate Chain of Trust:**
```
Root CA (self-signed, pre-installed in OS/browser)
  └── Intermediate CA (signed by Root CA)
        └── End-Entity Certificate (signed by Intermediate CA)
              (e.g., www.example.com)
```

Browsers and operating systems ship with a list of trusted **Root CAs**. Any certificate signed by a trusted Root CA (directly or through intermediates) is automatically trusted.

---

## 9. Data Formats and Standards

### 9.1 XML (eXtensible Markup Language)

**XML** is a text-based markup language for encoding documents in a human-readable and machine-readable format.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<student>
    <id>12345</id>
    <name>Alice Johnson</name>
    <course>CS 2204</course>
    <grade>A</grade>
</student>
```

**Features:**
- Self-describing: tag names convey meaning.
- Hierarchical structure.
- Platform and language independent.
- Verbose — larger file size than binary formats.

**Use:** SOAP web services, configuration files (Maven, Spring), Microsoft Office documents (DOCX, XLSX are XML archives), RSS feeds.

### 9.2 JSON (JavaScript Object Notation)

**JSON** is a lightweight, text-based data interchange format derived from JavaScript object syntax.

```json
{
  "student": {
    "id": 12345,
    "name": "Alice Johnson",
    "course": "CS 2204",
    "grade": "A"
  }
}
```

**Features:**
- Human-readable but more compact than XML.
- Supports: strings, numbers, booleans, arrays, objects, null.
- Native to JavaScript; supported natively by all modern languages.
- No support for comments; no schema enforcement by default.

**Use:** REST APIs, web application configuration, NoSQL databases (MongoDB), data exchange.

### 9.3 JSON vs. XML

| Feature | JSON | XML |
|---|---|---|
| Readability | Easy | Moderate (verbose) |
| File size | Smaller | Larger (tags repeated) |
| Data types | Native (numbers, booleans) | String only (must be typed via schema) |
| Comments | Not supported | Supported |
| Schema validation | JSON Schema (optional) | XSD (XML Schema Definition) |
| Namespaces | Not supported | Supported |
| Use in web | Dominant (REST APIs) | Legacy (SOAP), configuration |

### 9.4 MIME (Multipurpose Internet Mail Extensions)

**MIME** extends the format of email (and HTTP) to support non-ASCII characters and binary attachments. It defines:

- **Content-Type:** Describes the type and format of the data.
- **Content-Transfer-Encoding:** Specifies how the data is encoded (e.g., base64, quoted-printable).

**Common MIME Types:**

| MIME Type | Description |
|---|---|
| `text/plain` | Plain text |
| `text/html` | HTML document |
| `text/css` | CSS stylesheet |
| `application/json` | JSON data |
| `application/xml` | XML data |
| `application/pdf` | PDF document |
| `image/jpeg` | JPEG image |
| `image/png` | PNG image |
| `audio/mpeg` | MP3 audio |
| `video/mp4` | MP4 video |
| `multipart/form-data` | Form data with file uploads |

---

## 10. The Relationship Between Layers 5, 6, and 7 in Practice

### 10.1 OSI vs. TCP/IP Layer Mapping

In the OSI model, Layers 5, 6, and 7 are distinct. In the TCP/IP model, they collapse into a single **Application Layer**:

```
OSI Model                    TCP/IP Model
─────────────────────────────────────────
Layer 7 – Application  ┐
Layer 6 – Presentation ├──► Application Layer
Layer 5 – Session      ┘
─────────────────────────
Layer 4 – Transport    ────► Transport Layer
─────────────────────────
Layer 3 – Network      ────► Internet Layer
─────────────────────────
Layer 2 – Data Link    ┐
Layer 1 – Physical     ├──► Network Access Layer
                       ┘
```

### 10.2 How TCP/IP Applications Handle Layers 5 and 6

| OSI Function | How TCP/IP Applications Handle It |
|---|---|
| Session establishment | Application-level (HTTP cookies, SIP INVITE, SQL connections) |
| Session tokens | HTTP session IDs, JWT, OAuth 2.0 tokens |
| Synchronization/checkpoints | Application-level (FTP restart markers, download resume) |
| Data translation | Application code (language libraries, serialization frameworks) |
| Encryption | TLS (between TCP and Application); application-level encryption |
| Compression | HTTP content encoding (gzip, Brotli); application-level (ZIP, codec) |
| Character encoding | Application specifies encoding (UTF-8 in HTTP headers, JSON) |

### 10.3 A Practical Example: HTTPS Web Request

When a browser fetches `https://www.example.com/data.json`, all OSI layers 5, 6, and 7 functions occur within the application interaction:

```
OSI Function          How It Is Performed in HTTPS
────────────────────────────────────────────────────────────────
Session (L5):         TCP connection + TLS session (TLS session ID
                      or ticket allows session resumption)
Presentation (L6):    TLS encrypts/decrypts data (confidentiality)
                      HTTP defines Content-Type: application/json
                      Content-Encoding: gzip (compression)
                      JSON format (serialization/data translation)
Application (L7):     HTTP request/response protocol
                      Application logic processes the JSON payload
```

---

## 11. Character Encoding Standards

Character encoding is a core Presentation Layer concern — ensuring that text is represented the same way on both ends of a communication.

### 11.1 ASCII (American Standard Code for Information Interchange)

- **Bit width:** 7-bit (128 characters).
- **Coverage:** English alphabet (upper/lower), digits 0–9, punctuation, and 33 control characters.
- **Range:** 0–127.
- **Limitation:** Cannot represent non-English characters (accented letters, non-Latin scripts).

**Key ASCII values:**

| Character | Decimal | Hex |
|---|---|---|
| `NUL` | 0 | 0x00 |
| `LF` (newline) | 10 | 0x0A |
| `CR` (carriage return) | 13 | 0x0D |
| Space | 32 | 0x20 |
| `0` – `9` | 48–57 | 0x30–0x39 |
| `A` – `Z` | 65–90 | 0x41–0x5A |
| `a` – `z` | 97–122 | 0x61–0x7A |

### 11.2 Extended ASCII and EBCDIC

**Extended ASCII (ISO 8859 series):**
- Uses 8 bits (256 characters).
- The lower 128 values match standard ASCII; the upper 128 encode additional characters.
- Different variants (ISO 8859-1 for Western European, ISO 8859-5 for Cyrillic, etc.) — not universally interoperable.

**EBCDIC (Extended Binary Coded Decimal Interchange Code):**
- IBM's proprietary encoding for mainframe computers.
- 8-bit, 256 characters; completely different code points from ASCII.
- Not compatible with ASCII — a Presentation Layer translation is required when IBM mainframes communicate with ASCII-based systems.

### 11.3 Unicode

**Unicode** is a universal character encoding standard that aims to represent every character in every written language.

- **Code points:** Unicode assigns a unique integer (code point) to every character.
- **Range:** U+0000 to U+10FFFF (over 1.1 million possible code points; ~150,000 assigned).
- **Notation:** U+ followed by 4–6 hex digits (e.g., `U+0041` = `A`, `U+4E2D` = 中).

**Unicode Encoding Forms (how code points are stored as bytes):**

| Encoding | Unit Size | Variable Length | BOM | Endianness | Common Use |
|---|---|---|---|---|---|
| **UTF-8** | 1–4 bytes | Yes | Optional | N/A (byte-oriented) | Web (default), Linux, JSON |
| **UTF-16** | 2 or 4 bytes | Yes | Required | Big/Little | Windows, Java, JavaScript (internal) |
| **UTF-32** | 4 bytes | No (fixed) | Required | Big/Little | Internal processing; rare in transmission |

### 11.4 UTF-8 in Depth

**UTF-8** is the most widely used encoding on the Internet (used by over 97% of web pages):

- **ASCII compatible:** The first 128 Unicode code points (U+0000 to U+007F) are encoded as single bytes identical to ASCII — ASCII text is valid UTF-8.
- **Variable length:** Characters use 1 to 4 bytes depending on the code point range.

**UTF-8 Encoding Scheme:**

| Code Point Range | Bytes Used | Byte Pattern |
|---|---|---|
| U+0000 – U+007F | 1 byte | `0xxxxxxx` |
| U+0080 – U+07FF | 2 bytes | `110xxxxx 10xxxxxx` |
| U+0800 – U+FFFF | 3 bytes | `1110xxxx 10xxxxxx 10xxxxxx` |
| U+10000 – U+10FFFF | 4 bytes | `11110xxx 10xxxxxx 10xxxxxx 10xxxxxx` |

**Example:** The euro sign `€` (U+20AC):
- Binary: `0010 0000 1010 1100`
- UTF-8: `11100010 10000010 10101100` = `0xE2 0x82 0xAC`

---

## 12. Multimedia Encoding and Compression

### 12.1 Image Formats

| Format | Compression | Transparency | Animation | Best For |
|---|---|---|---|---|
| **JPEG** (.jpg) | Lossy (DCT-based) | No | No | Photographs, complex images |
| **PNG** (.png) | Lossless (DEFLATE) | Yes (alpha) | No | Graphics, screenshots, logos |
| **GIF** (.gif) | Lossless (LZW) | Yes (1-bit) | Yes | Simple animations, limited colors |
| **WebP** (.webp) | Lossy or lossless | Yes | Yes | Web images (20–30% smaller than JPEG/PNG) |
| **AVIF** (.avif) | Lossy or lossless | Yes | Yes | Next-gen web images; excellent compression |
| **BMP** (.bmp) | None (uncompressed) | No | No | Raw image data; Windows legacy |
| **TIFF** (.tiff) | Lossless (optional) | Yes | No | Professional photography, printing |
| **SVG** (.svg) | N/A (vector) | Yes | Yes | Scalable graphics, icons, logos |

**JPEG Compression Process (DCT-based):**
1. Convert image from RGB to YCbCr (luminance + chrominance).
2. Downsample chrominance channels (human eye is less sensitive to color detail).
3. Divide image into 8×8 pixel blocks.
4. Apply **Discrete Cosine Transform (DCT)** to each block — converts spatial information to frequency information.
5. **Quantize** DCT coefficients (discard high-frequency detail — this is where loss occurs).
6. Apply **Huffman coding** (lossless compression of quantized values).

**Quality factor (1–100):** Higher quality = less quantization = larger file = less loss.

### 12.2 Audio Formats

| Format | Compression | Quality | Use |
|---|---|---|---|
| **WAV** (.wav) | None | Perfect | Raw PCM audio; large files |
| **MP3** (.mp3) | Lossy (psychoacoustic) | Good | Music, podcasts; universal compatibility |
| **AAC** (.aac, .m4a) | Lossy | Better than MP3 at same bitrate | iTunes, YouTube, streaming |
| **OGG Vorbis** (.ogg) | Lossy | Comparable to MP3 | Open-source alternative to MP3 |
| **FLAC** (.flac) | Lossless | Perfect | Audiophile music; archiving |
| **Opus** (.opus) | Lossy | Excellent (especially at low bitrates) | VoIP, WebRTC, streaming |
| **AIFF** (.aiff) | None | Perfect | Apple uncompressed audio |

**MP3 Compression (MPEG-1 Audio Layer III):**
- Uses a **psychoacoustic model** to determine which audio components are inaudible.
- Applies **Modified DCT (MDCT)** to convert time-domain audio to frequency domain.
- Removes frequencies masked by louder nearby frequencies (masking effect).
- Applies **Huffman coding** to the remaining frequency data.
- Typical bitrates: 128 kbps (acceptable), 192 kbps (good), 320 kbps (near-transparent).

### 12.3 Video Standards

Video compression must address both **spatial redundancy** (within a frame) and **temporal redundancy** (between frames).

**Frame Types:**
| Frame Type | Description |
|---|---|
| **I-frame (Intra)** | Complete, self-contained frame; like a JPEG; reference point |
| **P-frame (Predicted)** | Encodes differences from the previous I or P frame |
| **B-frame (Bidirectional)** | Encodes differences from both past and future frames; highest compression |

A **Group of Pictures (GOP)** is the sequence pattern: I, B, P, B, P, B, P, ... between I-frames.

**Common Video Codecs:**

| Codec | Standard | Year | Use |
|---|---|---|---|
| **H.264 (AVC)** | MPEG-4 Part 10 | 2003 | Dominant; streaming, Blu-ray, video calls |
| **H.265 (HEVC)** | MPEG-H Part 2 | 2013 | 50% better compression than H.264; 4K streaming |
| **H.266 (VVC)** | ISO/IEC 23090-3 | 2020 | 50% better than H.265; next-gen |
| **VP9** | Google/IETF | 2013 | YouTube, WebRTC; open-source H.265 alternative |
| **AV1** | Alliance for Open Media | 2018 | Open-source; Netflix, YouTube; royalty-free |

**Container Formats** (wrap video + audio + subtitles):

| Container | Extension | Common Codecs |
|---|---|---|
| **MP4** | .mp4, .m4v | H.264, H.265, AAC |
| **MKV (Matroska)** | .mkv | Almost anything |
| **WebM** | .webm | VP9, AV1, Opus |
| **AVI** | .avi | Legacy; various |
| **MOV** | .mov | Apple; H.264, ProRes |

---

## 13. Abstract Syntax Notation (ASN.1)

### 13.1 What is ASN.1?

**ASN.1 (Abstract Syntax Notation One)** is a formal language defined by **ITU-T** for describing data structures in a way that is independent of any machine architecture or programming language. It is the Presentation Layer standard for data structure description in telecommunications and security protocols.

ASN.1 separates:
- **Abstract syntax:** The logical structure of the data (defined in ASN.1 notation).
- **Transfer syntax:** How the data is encoded as bytes for transmission (defined by encoding rules).

### 13.2 ASN.1 Encoding Rules

| Encoding | Description | Use |
|---|---|---|
| **BER (Basic Encoding Rules)** | Original; allows multiple valid encodings for the same value | SNMP, X.509 (legacy) |
| **DER (Distinguished Encoding Rules)** | Subset of BER; exactly one valid encoding per value | X.509 certificates, TLS, PKCS |
| **PER (Packed Encoding Rules)** | Compact binary; minimal padding | Telecom (3GPP, LTE) |
| **XER (XML Encoding Rules)** | XML-based encoding of ASN.1 structures | Web services |
| **JSON Encoding Rules (JER)** | JSON-based encoding | Modern APIs using ASN.1 schemas |

### 13.3 ASN.1 in Practice

ASN.1 is widely used behind the scenes in security and network management:

- **X.509 digital certificates** — encoded in DER (binary) or PEM (base64-encoded DER).
- **PKCS standards** — cryptographic message formats (e.g., PKCS#12 for certificate stores).
- **SNMP (Simple Network Management Protocol)** — MIB data encoded in BER.
- **LDAP (Lightweight Directory Access Protocol)** — directory data encoded in BER.
- **LTE/5G air interface protocols** — PER-encoded ASN.1 structures.

**Example ASN.1 definition:**
```asn1
Student ::= SEQUENCE {
    id        INTEGER,
    name      UTF8String,
    course    UTF8String,
    grade     ENUMERATED { A, B, C, D, F }
}
```

This abstract definition can then be encoded as BER, DER, JSON, or XML without changing the schema — demonstrating the Presentation Layer principle of separating structure from encoding.

---

## 14. Summary

This module explored OSI Layers 5 and 6 — the Session Layer and Presentation Layer:

**Session Layer (OSI Layer 5):**
- Provides structured dialogue management between application processes.
- Key functions: session establishment/termination, dialog control (simplex/half-duplex/full-duplex), synchronization checkpoints, and exception reporting.
- In TCP/IP, session functions are absorbed into the Application Layer — implemented by protocols like SIP, HTTP cookies, SQL connections, and RPC.

**Presentation Layer (OSI Layer 6):**
- Responsible for data translation, encryption, and compression.
- Addresses the translation problem: byte order (endianness), character encoding, data serialization formats.
- **Encryption:** Symmetric (AES, ChaCha20) for bulk data; Asymmetric (RSA, ECDH) for key exchange; Hashing (SHA-256) for integrity; TLS combines all three.
- **Compression:** Lossless (DEFLATE, Huffman, LZ77) for text/files; Lossy (JPEG, MP3, H.264) for media.
- **Character encoding:** ASCII (7-bit, English only); Unicode/UTF-8 (universal, web standard).
- **Data formats:** JSON (REST APIs), XML (SOAP, config), ASN.1 (security/telecom protocols).
- **Multimedia:** JPEG (photos), PNG (graphics), MP3/AAC (audio), H.264/H.265/AV1 (video).
- In TCP/IP, presentation functions are handled by TLS, HTTP content negotiation, and application-level libraries.

---

## 15. Key Terms

| Term | Definition |
|---|---|
| **Session Layer** | OSI Layer 5; manages structured dialogues, synchronization, and session state |
| **Session** | A semi-permanent interactive exchange between two communicating processes |
| **Dialog control** | Managing the direction and turn-taking of communication (simplex, half-duplex, full-duplex) |
| **Synchronization point** | A checkpoint in a session that marks committed progress; enables recovery from failures |
| **Major sync point** | A confirmed checkpoint; data before it will not be retransmitted |
| **Minor sync point** | An unconfirmed checkpoint; structures dialogue within a segment |
| **SIP** | Session Initiation Protocol; establishes, modifies, and terminates multimedia sessions |
| **Presentation Layer** | OSI Layer 6; handles data translation, encryption, and compression |
| **Endianness** | Byte order: big-endian (MSB first) vs. little-endian (LSB first) |
| **Serialization** | Converting structured data into a byte sequence for transmission or storage |
| **Lossless compression** | Compression where original data is perfectly recovered (DEFLATE, Huffman) |
| **Lossy compression** | Compression that permanently discards some data (JPEG, MP3, H.264) |
| **Symmetric encryption** | Single key for both encryption and decryption (AES, ChaCha20) |
| **Asymmetric encryption** | Public/private key pair; public key encrypts, private key decrypts (RSA, ECDH) |
| **AES** | Advanced Encryption Standard; current symmetric encryption standard |
| **Hash function** | One-way function producing a fixed-size digest from any input |
| **SHA-256** | Secure Hash Algorithm; 256-bit output; current standard for integrity |
| **HMAC** | Hash-based Message Authentication Code; provides integrity and authentication |
| **Digital signature** | Encrypted hash proving authenticity, integrity, and non-repudiation |
| **PKI** | Public Key Infrastructure; system of CAs, certificates, and trust relationships |
| **X.509** | Standard for digital certificates binding a public key to an identity |
| **Certificate Authority (CA)** | Trusted entity that issues and signs digital certificates |
| **TLS** | Transport Layer Security; provides confidentiality, integrity, and authentication over TCP |
| **ASCII** | 7-bit character encoding for English text (128 characters) |
| **Unicode** | Universal character standard covering all world scripts (U+0000 to U+10FFFF) |
| **UTF-8** | Variable-length Unicode encoding; ASCII-compatible; dominant on the web |
| **MIME** | Multipurpose Internet Mail Extensions; defines content types for email and HTTP |
| **JSON** | JavaScript Object Notation; lightweight text-based data interchange format |
| **XML** | eXtensible Markup Language; hierarchical text-based data format |
| **ASN.1** | Abstract Syntax Notation One; formal language for describing data structures; used in X.509, SNMP |
| **DCT** | Discrete Cosine Transform; mathematical transform used in JPEG and MP3 compression |
| **I-frame** | Complete reference video frame; requires no other frames to decode |
| **GOP** | Group of Pictures; sequence of I, P, and B frames in a video stream |
| **DEFLATE** | Lossless compression combining LZ77 + Huffman; used in ZIP, gzip, PNG |

---

## 16. Review Questions

**Conceptual Questions:**

1. What is the difference between a Transport Layer connection and a Session Layer session? Use a real-world analogy to illustrate the distinction.
2. Explain the purpose of synchronization checkpoints in the Session Layer. How do major and minor synchronization points differ? Give an example of how checkpoints benefit a large file transfer.
3. Why is the Session Layer often described as "invisible" in TCP/IP networking? Name three protocols or mechanisms that implement Session Layer functions within the TCP/IP Application Layer.
4. What are the three core responsibilities of the Presentation Layer? For each, give a concrete example of how it is implemented in a modern TCP/IP application.
5. Explain the endianness problem. A 32-bit integer `0xDEADBEEF` is stored in big-endian order. How would it appear in memory in little-endian order? Why must network protocols specify byte order?
6. Compare lossless and lossy compression. For each, name two algorithms or formats and explain why that type of compression is appropriate for its use case.
7. Explain the difference between symmetric and asymmetric encryption. Why does TLS use both rather than just one? What role does each play in a TLS session?
8. What is a digital certificate (X.509)? What problem does PKI solve, and how does the chain of trust work?
9. Compare ASCII, Extended ASCII, and UTF-8. Why has UTF-8 become the dominant encoding standard for the web? What makes it backward-compatible with ASCII?
10. What is ASN.1, and why is it a Presentation Layer concept? Name three protocols that rely on ASN.1 encoding and identify which encoding rules each uses.

**Application / Analysis Questions:**

11. A UTF-8 encoded string contains the bytes `0x48 0x65 0x6C 0x6C 0x6F 0xE2 0x82 0xAC`. Identify each character. Which bytes are ASCII-compatible single-byte characters, and which represent a multi-byte UTF-8 sequence? What character does the multi-byte sequence represent?

12. A web server sends an HTTP response with headers:
    ```
    Content-Type: application/json; charset=utf-8
    Content-Encoding: gzip
    ```
    Identify which OSI layer each header element relates to (Session, Presentation, or Application), and explain what the receiving browser must do with this information before using the response body.

13. Explain the JPEG compression pipeline. A photographer saves a photo at quality 95 vs. quality 30. What is different in the compression process between the two settings, and what are the trade-offs in terms of file size and image quality?

14. AES-256-GCM is described as an AEAD cipher. Define AEAD and explain what AES-256-GCM provides that plain AES-256-CBC does not. Why is GCM preferred in TLS 1.3?

15. An organization uses a SIP-based VoIP system. Trace the full session lifecycle for a call from Alice to Bob: (a) How is the session established? (b) How is actual audio transmitted? (c) How is the session terminated? Identify the roles of SIP and RTP, and map each to the OSI layer where it operates.

16. A 2-hour uncompressed video at 1080p (30 fps) has the following parameters: resolution = 1920×1080 pixels, color depth = 24 bits/pixel, frame rate = 30 fps. Calculate: (a) the size of one uncompressed frame in bytes, (b) the total uncompressed file size, and (c) the storage space saved if H.264 achieves a 200:1 compression ratio.

---

*End of Module 6 | Next: Module 7 – The Application Layer and Network Security*
