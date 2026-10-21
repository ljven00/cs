# CS 2204 – Communications and Networking
## Module 1: Introduction to Data Communication and Networking

---

## Table of Contents

1. [Overview](#1-overview)
2. [What is Data Communication?](#2-what-is-data-communication)
3. [Components of a Data Communication System](#3-components-of-a-data-communication-system)
4. [Data Flow Modes](#4-data-flow-modes)
5. [What is a Network?](#5-what-is-a-network)
6. [Types of Networks](#6-types-of-networks)
7. [Network Topologies](#7-network-topologies)
8. [Network Models: OSI vs. TCP/IP](#8-network-models-osi-vs-tcpip)
9. [Protocols and Standards](#9-protocols-and-standards)
10. [Standards Organizations](#10-standards-organizations)
11. [Summary](#11-summary)
12. [Key Terms](#12-key-terms)
13. [Review Questions](#13-review-questions)

---

## 1. Overview

Data communication and networking form the backbone of modern digital society. From sending an email to streaming video, every digital interaction depends on the principles covered in this module.

**Module Learning Objectives:**

By the end of this module, students should be able to:

- Define data communication and explain its fundamental components.
- Describe different data flow modes (simplex, half-duplex, full-duplex).
- Define a computer network and explain its purpose.
- Differentiate between LAN, MAN, WAN, and other network types.
- Describe common network topologies and their trade-offs.
- Explain the OSI and TCP/IP reference models and the purpose of layering.
- Define protocols and explain the role of standards organizations.

---

## 2. What is Data Communication?

**Data communication** is the exchange of data between two or more devices via a transmission medium (e.g., a cable or wireless signal).

For data communication to occur, the communicating devices must be part of a **communication system** made up of hardware and software.

### 2.1 Effectiveness Criteria

A data communication system is considered effective if it delivers data with the following characteristics:

| Criterion | Description |
|-----------|-------------|
| **Delivery** | Data must be delivered to the correct destination device and only that device. |
| **Accuracy** | Data must arrive without alteration. Corrupted data is unusable. |
| **Timeliness** | Data must arrive within an acceptable time frame. Late delivery can render data useless (e.g., live video). |
| **Jitter** | Variation in packet arrival time must be controlled, especially for audio/video applications. |

---

## 3. Components of a Data Communication System

Every data communication system consists of five essential components:

```
[Message] → [Sender] → [Transmission Medium] → [Receiver] → [Message]
                              ↑
                          [Protocol]
```

### 3.1 The Five Components

**1. Message**
The information (data) to be communicated. This can be text, numbers, images, audio, or video.

**2. Sender**
The device that sends the data message. This can be a computer, workstation, telephone, or camera.

**3. Receiver**
The device that receives the message. Like the sender, it can be any capable device.

**4. Transmission Medium**
The physical path through which a message travels from sender to receiver. Examples include:
- Twisted-pair wire
- Coaxial cable
- Fiber-optic cable
- Radio waves (wireless)

**5. Protocol**
A set of rules that govern data communication. It represents an agreement between the communicating devices. Without a protocol, two devices may be connected but unable to communicate — like two people speaking different languages.

---

## 4. Data Flow Modes

Communication between two devices can occur in one of three modes:

### 4.1 Simplex

- Communication is **one-directional**.
- Only one device transmits; the other can only receive.
- **Example:** Keyboard to computer, traditional broadcast television.

```
Device A  ───────────────────────►  Device B
          (one direction only)
```

### 4.2 Half-Duplex

- Communication is **two-directional**, but **not simultaneous**.
- Each device can both transmit and receive, but not at the same time.
- When one device sends, the other must receive and wait.
- **Example:** Walkie-talkies, older CB radios.

```
Device A  ◄───────────────────────  Device B
Device A  ───────────────────────►  Device B
          (one direction at a time)
```

### 4.3 Full-Duplex

- Communication is **two-directional and simultaneous**.
- Both devices can transmit and receive at the same time.
- **Example:** Telephone calls, modern network links.

```
Device A  ◄──────────────────────►  Device B
          (both directions at once)
```

---

## 5. What is a Network?

A **computer network** is a set of devices (nodes) connected by communication links. A **node** can be a computer, printer, or any other device capable of sending or receiving data.

### 5.1 Why Network?

Networks exist to enable:

- **Resource sharing** – Printers, storage, and software can be shared across users.
- **Communication** – Email, messaging, video conferencing.
- **Data sharing** – Files and databases accessible from multiple locations.
- **Centralized management** – Easier administration of users and security.

---

## 6. Types of Networks

Networks are commonly classified by their **geographic scope**:

### 6.1 Local Area Network (LAN)

- Covers a **small geographic area** such as an office, building, or campus.
- Typically owned and managed by a single organization.
- High data transfer rates (e.g., 100 Mbps to 10 Gbps).
- Uses technologies such as Ethernet and Wi-Fi.

### 6.2 Metropolitan Area Network (MAN)

- Covers a **city or large campus**.
- Larger than a LAN but smaller than a WAN.
- Often used by organizations with multiple buildings in a city.
- Example: A city-wide cable TV network or municipal Wi-Fi.

### 6.3 Wide Area Network (WAN)

- Covers a **large geographic area** — countries or continents.
- Usually made up of interconnected LANs.
- Uses public infrastructure (telephone lines, satellites, fiber).
- The Internet is the largest example of a WAN.

### 6.4 Personal Area Network (PAN)

- Covers the range of an **individual person** (a few meters).
- Used to connect personal devices.
- Example: Bluetooth connection between a phone and headphones.

### Summary Table

| Type | Geographic Scope | Typical Owner | Example |
|------|-----------------|---------------|---------|
| PAN | ~1–10 meters | Individual | Bluetooth devices |
| LAN | Building/Campus | Organization | Office Ethernet |
| MAN | City | Organization/ISP | Campus network |
| WAN | Country/Global | Multiple/Public | The Internet |

---

## 7. Network Topologies

**Topology** refers to the way nodes and links in a network are arranged. There are two aspects:

- **Physical topology** – The actual geometric layout of the hardware.
- **Logical topology** – How data actually flows through the network.

### 7.1 Bus Topology

- All devices are connected to a single shared cable (the **bus**).
- Data transmitted by one device travels the entire length of the bus.

**Advantages:** Simple, inexpensive, easy to install.
**Disadvantages:** A single cable failure disables the entire network; difficult to troubleshoot.

```
Device─┬─Device─┬─Device─┬─Device
       │        │        │
══════════════════════════════ (Bus)
```

### 7.2 Star Topology

- All devices are connected to a **central hub or switch**.
- Each device has a dedicated point-to-point link to the hub.

**Advantages:** Easy to add/remove devices; one link failure doesn't affect others.
**Disadvantages:** If the central hub fails, the entire network goes down.

```
        Device
           |
Device ── Hub/Switch ── Device
           |
        Device
```

### 7.3 Ring Topology

- Each device is connected to exactly **two other devices**, forming a ring.
- Data travels in one direction around the ring.

**Advantages:** Predictable performance; no collisions.
**Disadvantages:** A single device or link failure can disrupt the whole ring (unless dual-ring).

```
Device ── Device
  |              |
Device ── Device
```

### 7.4 Mesh Topology

- Every device is connected to **every other device**.
- Provides maximum redundancy and reliability.

**Advantages:** Highly fault-tolerant; multiple paths for data.
**Disadvantages:** Expensive and complex to install; number of links grows rapidly with nodes.

> For *n* devices, a full mesh requires **n(n–1)/2** links.

### 7.5 Hybrid Topology

- A **combination** of two or more topologies.
- Most real-world networks use hybrid topologies (e.g., star-bus, star-ring).

---

## 8. Network Models: OSI vs. TCP/IP

To reduce complexity, network functions are organized into **layers**. Each layer handles a specific set of tasks and communicates with the layers directly above and below it.

### 8.1 The OSI Reference Model

The **Open Systems Interconnection (OSI)** model, developed by ISO, defines seven layers:

| Layer | Number | Name | Primary Responsibility |
|-------|--------|------|------------------------|
| 7 | Application | Application | Network services to end-user applications |
| 6 | Presentation | Presentation | Data translation, encryption, compression |
| 5 | Session | Session | Establishing, managing, terminating sessions |
| 4 | Transport | Transport | End-to-end delivery, error recovery, flow control |
| 3 | Network | Network | Logical addressing and routing |
| 2 | Data Link | Data Link | Framing, physical addressing, error detection |
| 1 | Physical | Physical | Bit transmission over physical medium |

**Key Concept – Encapsulation:**
As data moves **down** the layers from sender to receiver, each layer adds its own header (and sometimes trailer) — a process called **encapsulation**. The reverse process (removing headers) at the receiver is called **decapsulation**.

### 8.2 The TCP/IP Model

The **TCP/IP model** (also called the Internet model) is the practical model used by the modern Internet. It has four layers:

| TCP/IP Layer | Corresponds to OSI Layers |
|---|---|
| Application | Application + Presentation + Session (OSI 5–7) |
| Transport | Transport (OSI 4) |
| Internet | Network (OSI 3) |
| Network Access | Data Link + Physical (OSI 1–2) |

### 8.3 OSI vs. TCP/IP Comparison

| Feature | OSI Model | TCP/IP Model |
|---------|-----------|--------------|
| Layers | 7 | 4 |
| Development | ISO standard (theoretical) | ARPANET (practical) |
| Usage | Reference/teaching | Real-world Internet |
| Transport protocols | Generic | TCP, UDP |

> The OSI model is primarily used as a **conceptual framework** for understanding and designing networks. TCP/IP is the **functional standard** used in practice.

---

## 9. Protocols and Standards

### 9.1 What is a Protocol?

A **protocol** is a set of rules that defines:

- **Syntax** – The structure or format of the data (what each field means).
- **Semantics** – The meaning of each section of bits (how to interpret each field).
- **Timing** – When data should be sent and how fast it can be transmitted.

Without agreed-upon protocols, devices from different manufacturers could not communicate.

### 9.2 What are Standards?

**Standards** ensure that hardware and software produced by different vendors can work together (**interoperability**).

Standards are classified into two types:

**De facto standards** – Widely used in practice but not officially approved by a standards body.
> Example: The QWERTY keyboard layout.

**De jure standards** – Officially legislated and approved by a recognized standards body.
> Example: ASCII character encoding.

---

## 10. Standards Organizations

Several key organizations develop and publish networking standards:

| Organization | Full Name | Role |
|---|---|---|
| **ISO** | International Organization for Standardization | Developed the OSI model |
| **ITU-T** | International Telecommunication Union – Telecom Sector | Telecommunications standards |
| **ANSI** | American National Standards Institute | US national standards |
| **IEEE** | Institute of Electrical and Electronics Engineers | LAN standards (e.g., IEEE 802.3 Ethernet, IEEE 802.11 Wi-Fi) |
| **EIA** | Electronic Industries Alliance | Physical layer standards (e.g., connectors, cabling) |
| **IETF** | Internet Engineering Task Force | Internet protocols (TCP, IP, HTTP, DNS) — publishes RFCs |
| **W3C** | World Wide Web Consortium | Web standards (HTML, CSS, XML) |

---

## 11. Summary

This module introduced the foundational concepts of data communications and networking:

- **Data communication** requires five components: message, sender, receiver, medium, and protocol.
- **Data flow** can be simplex, half-duplex, or full-duplex.
- **Networks** are classified by geographic scope: PAN, LAN, MAN, and WAN.
- **Network topology** describes the physical and logical arrangement of devices: bus, star, ring, mesh, and hybrid.
- The **OSI model** provides a 7-layer conceptual framework; the **TCP/IP model** is the practical 4-layer model used on the Internet.
- **Protocols** define the rules for communication (syntax, semantics, timing).
- **Standards organizations** (ISO, IEEE, IETF, etc.) develop and publish the standards that enable interoperability.

---

## 12. Key Terms

| Term | Definition |
|---|---|
| **Data Communication** | The exchange of data between devices via a transmission medium |
| **Protocol** | A set of rules governing data communication |
| **Network** | A set of devices connected by communication links |
| **Topology** | The geometric arrangement of a network's nodes and links |
| **LAN** | Local Area Network – covers a limited geographic area |
| **WAN** | Wide Area Network – covers a large geographic area |
| **OSI Model** | 7-layer reference model for network communications |
| **TCP/IP Model** | 4-layer practical model used in the Internet |
| **Simplex** | One-way data flow |
| **Half-Duplex** | Two-way data flow, but not simultaneous |
| **Full-Duplex** | Two-way simultaneous data flow |
| **Encapsulation** | Adding headers as data moves down the OSI layers |
| **Standard** | A set of rules agreed upon by standards bodies to ensure interoperability |
| **De facto standard** | A standard adopted through widespread use, not official decree |
| **De jure standard** | A standard formally approved by a recognized standards body |

---

## 13. Review Questions

**Conceptual Questions:**

1. What are the five components of a data communication system? Briefly describe each.
2. Explain the difference between half-duplex and full-duplex communication. Give a real-world example of each.
3. What is the difference between a LAN and a WAN? How does a MAN fit between them?
4. Compare the bus and star topologies. What are the failure implications of each?
5. Why is a mesh topology considered the most reliable? What is its main disadvantage?
6. What are the three key elements of a protocol (syntax, semantics, timing)? Explain each briefly.
7. What is the difference between a de facto and a de jure standard? Give one example of each.

**Application Questions:**

8. A company has offices in five buildings on the same city block, all connected to a central switch. What type of network and topology does this describe?
9. A walkie-talkie system allows only one person to speak at a time. Which data flow mode does this represent?
10. Map the following tasks to their OSI layer: (a) routing a packet, (b) establishing a session, (c) converting data to bits, (d) encrypting data.

---

*End of Module 1 | Next: Module 2 – The Physical Layer and Transmission Media*
