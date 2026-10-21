# CS 2204 – Communications and Networking
## Module 2: The Physical Layer and Transmission Media

---

## Table of Contents

1. [Overview](#1-overview)
2. [The Physical Layer in the OSI Model](#2-the-physical-layer-in-the-osi-model)
3. [Data and Signals](#3-data-and-signals)
4. [Analog vs. Digital Signals](#4-analog-vs-digital-signals)
5. [Signal Impairments](#5-signal-impairments)
6. [Data Rate Limits](#6-data-rate-limits)
7. [Transmission Media: Overview](#7-transmission-media-overview)
8. [Guided (Wired) Transmission Media](#8-guided-wired-transmission-media)
9. [Unguided (Wireless) Transmission Media](#9-unguided-wireless-transmission-media)
10. [Transmission Media Comparison](#10-transmission-media-comparison)
11. [Digital Transmission: Line Coding](#11-digital-transmission-line-coding)
12. [Analog Transmission: Modulation](#12-analog-transmission-modulation)
13. [Multiplexing](#13-multiplexing)
14. [Switching](#14-switching)
15. [Summary](#15-summary)
16. [Key Terms](#16-key-terms)
17. [Review Questions](#17-review-questions)

---

## 1. Overview

The Physical Layer is the foundation of all network communication. It defines how raw bits are converted into electrical, optical, or radio signals and transported across a physical medium. Understanding this layer is critical because all higher-level protocols depend on the physical delivery of bits.

**Module Learning Objectives:**

By the end of this module, students should be able to:

- Describe the responsibilities of the Physical Layer in the OSI model.
- Distinguish between analog and digital data and signals.
- Explain how signal impairments affect data transmission.
- Apply the Nyquist and Shannon theorems to calculate maximum data rates.
- Compare and contrast guided and unguided transmission media.
- Describe common line coding schemes for digital transmission.
- Explain amplitude, frequency, and phase modulation for analog transmission.
- Describe the purpose and types of multiplexing.
- Compare circuit switching and packet switching.

---

## 2. The Physical Layer in the OSI Model

The **Physical Layer (Layer 1)** is the lowest layer of the OSI model. It is responsible for the transmission of raw, unstructured **bit streams** over a physical medium.

### 2.1 Key Responsibilities

| Responsibility | Description |
|---|---|
| **Bit representation** | Defines how bits (0s and 1s) are encoded as signals (electrical, optical, or electromagnetic) |
| **Data rate** | Defines the number of bits sent per second (transmission rate) |
| **Synchronization** | Ensures the sender and receiver clocks are synchronized |
| **Physical topology** | Defines how devices are physically connected (bus, star, ring, etc.) |
| **Transmission mode** | Defines simplex, half-duplex, or full-duplex operation |
| **Line configuration** | Point-to-point or multipoint |

### 2.2 What the Physical Layer Does NOT Do

The Physical Layer deals only with the movement of individual bits. It does not:
- Interpret the meaning of those bits.
- Detect or correct errors (that is the job of the Data Link Layer).
- Address devices (that is handled by higher layers).

---

## 3. Data and Signals

To be transmitted, data must be converted into a **signal** — an electromagnetic representation that can travel through a medium.

### 3.1 Data Types

**Analog data** – Takes continuous values within a range.
> Example: Sound waves, temperature readings, analog voice.

**Digital data** – Takes discrete values (finite set of states).
> Example: Text files, integers, binary data.

### 3.2 Signal Types

**Analog signal** – A continuous waveform that varies smoothly over time.
- Described by: amplitude, frequency, and phase.

**Digital signal** – A signal that uses a limited number of discrete levels (typically two: high and low, representing 1 and 0).

### 3.3 Periodic vs. Non-Periodic Signals

**Periodic signal** – Repeats a pattern over a time period (cycle).
- Characterized by its **period (T)** and **frequency (f)**, where f = 1/T.

**Non-periodic (aperiodic) signal** – Does not repeat. Most data signals are non-periodic.

### 3.4 Key Properties of Analog Signals

| Property | Description | Unit |
|---|---|---|
| **Amplitude** | Peak value (strength) of the signal | Volts (V) |
| **Frequency** | Number of cycles per second | Hertz (Hz) |
| **Period** | Duration of one cycle (T = 1/f) | Seconds (s) |
| **Phase** | Position of the waveform relative to a reference point | Degrees (°) or Radians |
| **Wavelength** | Distance a signal travels in one period (λ = v/f) | Meters (m) |

### 3.5 Frequency and the Electromagnetic Spectrum

The electromagnetic spectrum spans a range of frequencies used in different communication technologies:

| Frequency Range | Technology |
|---|---|
| 3 Hz – 300 Hz | Power lines |
| 3 kHz – 300 kHz | AM radio, submarine communication |
| 300 kHz – 3 MHz | AM radio broadcast |
| 3 MHz – 300 MHz | FM radio, VHF TV |
| 300 MHz – 3 GHz | UHF TV, cellular, Wi-Fi, Bluetooth |
| 3 GHz – 300 GHz | Microwave, satellite, 5G mmWave |
| > 300 GHz | Infrared, visible light, fiber optics |

---

## 4. Analog vs. Digital Signals

### 4.1 Bandwidth

**Analog bandwidth** – The range of frequencies a medium can carry, measured in Hertz (Hz).

**Digital bandwidth** – The maximum data transfer rate, measured in bits per second (bps).

> Note: These two uses of the word "bandwidth" are related but different. A medium with a wider analog bandwidth can carry a higher digital bandwidth.

### 4.2 Composite Signals and Fourier Analysis

Real data signals (digital or complex analog) are **composite signals** — they consist of many sine waves of different frequencies combined. **Fourier analysis** shows that any periodic signal can be decomposed into a sum of sine waves.

- A **digital signal** has an **infinite bandwidth** in theory.
- The range of frequencies in a composite signal is called its **bandwidth**.

### 4.3 Digital vs. Analog Transmission

| Feature | Analog Transmission | Digital Transmission |
|---|---|---|
| Signal type | Continuous | Discrete |
| Degradation | Signal weakens (attenuation) and accumulates noise | Signal can be regenerated exactly |
| Cost | Less expensive hardware (historically) | Modern hardware is cost-effective |
| Noise sensitivity | High — noise is amplified with signal | Low — noise can be filtered by regeneration |
| Security | Harder to encrypt | Easier to encrypt |

---

## 5. Signal Impairments

As a signal travels through a medium, it can suffer degradation. The three most common forms of impairment are:

### 5.1 Attenuation

**Attenuation** is the loss of signal energy (strength) as it travels through a medium. Energy is absorbed by the medium.

- Measured in **decibels (dB)**: `dB = 10 log₁₀(P₂/P₁)`
- A negative dB value indicates loss; a positive value indicates gain (amplification).
- Solution: **Amplifiers** (analog) or **repeaters/regenerators** (digital) restore signal strength.

**Example:**
> If a signal enters a cable with power P₁ = 100 mW and exits with P₂ = 50 mW:
> dB = 10 log₁₀(50/100) = 10 × (−0.301) = **−3 dB** (3 dB loss)

### 5.2 Distortion

**Distortion** occurs when the shape (form) of the signal changes. In composite signals, different frequency components travel at slightly different speeds, causing them to arrive out of phase.

- Common in digital signals where high-frequency components are absorbed faster.
- Can cause received signals to look different from the transmitted signal.

### 5.3 Noise

**Noise** is any unwanted signal that is added to the data signal during transmission. Types include:

| Noise Type | Description |
|---|---|
| **Thermal noise** | Caused by random electron motion; present in all electronic devices |
| **Induced noise** | From external electromagnetic sources (motors, power lines) |
| **Crosstalk** | Signal from one wire couples into an adjacent wire |
| **Impulse noise** | Short bursts of high energy (lightning, electrical spikes); most damaging |

**Signal-to-Noise Ratio (SNR):**
A key measure of signal quality:

```
SNR = Signal Power / Noise Power
SNR_dB = 10 log₁₀(SNR)
```

A higher SNR means a cleaner signal and better communication quality.

---

## 6. Data Rate Limits

Two theoretical models define the maximum data rate for a channel:

### 6.1 Nyquist Theorem (Noiseless Channel)

For a **noiseless** channel, the maximum data rate is:

```
Maximum Data Rate = 2 × Bandwidth × log₂(L)
```

Where:
- **Bandwidth** = channel bandwidth in Hz
- **L** = number of signal levels (discrete states)

**Example:**
> A noiseless channel with a bandwidth of 3000 Hz and 8 signal levels:
> Max Rate = 2 × 3000 × log₂(8) = 2 × 3000 × 3 = **18,000 bps**

> Increasing signal levels (L) increases data rate, but requires higher signal-to-noise ratio to be distinguishable.

### 6.2 Shannon's Theorem (Noisy Channel)

For a **noisy** channel, Shannon's Capacity Formula gives the theoretical maximum:

```
Maximum Data Rate = Bandwidth × log₂(1 + SNR)
```

Where:
- **SNR** = Signal-to-Noise Ratio (as a power ratio, not in dB)

**Example:**
> A channel with bandwidth = 3000 Hz and SNR = 1000:
> Capacity = 3000 × log₂(1001) ≈ 3000 × 9.97 ≈ **29,900 bps**

### 6.3 Using Both Theorems Together

In practice, both limits apply:
- Shannon's theorem sets the **absolute upper bound**.
- Nyquist's theorem helps determine the **number of signal levels needed** to approach that bound.
- The actual data rate must be **at or below both limits**.

---

## 7. Transmission Media: Overview

**Transmission media** are the physical paths through which signals travel from sender to receiver. They are classified into:

- **Guided media** – Signal is physically guided along a solid medium (wired).
- **Unguided media** – Signal propagates through the air or space without a physical guide (wireless).

```
Transmission Media
├── Guided (Wired)
│   ├── Twisted-Pair Cable (UTP / STP)
│   ├── Coaxial Cable
│   └── Fiber-Optic Cable
└── Unguided (Wireless)
    ├── Radio Waves
    ├── Microwaves
    └── Infrared
```

---

## 8. Guided (Wired) Transmission Media

### 8.1 Twisted-Pair Cable

**Twisted-pair cable** consists of two insulated copper wires twisted together. The twisting helps cancel electromagnetic interference (EMI) from external sources and reduces crosstalk between adjacent pairs.

**Types:**

| Type | Full Name | Shielding | Use |
|---|---|---|---|
| **UTP** | Unshielded Twisted Pair | None | Most common; telephone, Ethernet (LANs) |
| **STP** | Shielded Twisted Pair | Metallic foil shield | Higher noise environments |

**UTP Categories:**

| Category | Bandwidth | Max Data Rate | Application |
|---|---|---|---|
| Cat 3 | 16 MHz | 10 Mbps | Traditional telephone, 10BASE-T |
| Cat 5 | 100 MHz | 100 Mbps | 100BASE-TX (Fast Ethernet) |
| Cat 5e | 100 MHz | 1 Gbps | Gigabit Ethernet |
| Cat 6 | 250 MHz | 1 Gbps (10 Gbps short distances) | Gigabit and 10G Ethernet |
| Cat 6a | 500 MHz | 10 Gbps | 10GBASE-T Ethernet |
| Cat 7 | 600 MHz | 10 Gbps | Shielded, data centers |

**Advantages of Twisted-Pair:**
- Low cost; widely available.
- Easy to install.
- Supports both analog and digital signals.

**Disadvantages:**
- Susceptible to noise over long distances.
- Limited bandwidth compared to coaxial or fiber.

---

### 8.2 Coaxial Cable

**Coaxial cable** consists of a central copper conductor surrounded by an insulating layer, a metallic shield (braid or foil), and an outer plastic jacket. The shielding provides excellent noise resistance.

**Structure:**
```
[Outer Jacket] → [Metallic Shield] → [Insulating Layer] → [Inner Conductor]
```

**Types:**

| Type | RG Rating | Impedance | Use |
|---|---|---|---|
| Thinnet | RG-58 | 50 Ω | 10BASE2 Ethernet |
| Thicknet | RG-8 | 50 Ω | 10BASE5 Ethernet |
| TV/Cable | RG-59 / RG-6 | 75 Ω | Cable TV, broadband Internet |

**Advantages of Coaxial Cable:**
- Higher bandwidth than twisted-pair.
- Better noise rejection due to shielding.
- Longer distance without amplification.

**Disadvantages:**
- More expensive than twisted-pair.
- Bulkier and less flexible.
- Largely replaced by fiber for backbone applications.

---

### 8.3 Fiber-Optic Cable

**Fiber-optic cable** transmits data as **pulses of light** through a glass or plastic core. Light travels down the core via **total internal reflection** — reflecting off the cladding back into the core.

**Structure:**
```
[Protective Jacket] → [Buffer Coating] → [Cladding] → [Glass/Plastic Core]
```

**Propagation Modes:**

| Mode | Core Diameter | Light Source | Distance | Use |
|---|---|---|---|---|
| **Multimode (MMF)** | 50–62.5 µm | LED | Short (up to ~2 km) | LAN, intra-building |
| **Single-mode (SMF)** | ~8–10 µm | Laser diode | Long (up to 100+ km) | WAN, backbone, telecom |

**Advantages of Fiber-Optic Cable:**
- Extremely high bandwidth (tens of Tbps).
- Immune to electromagnetic interference.
- Very low attenuation over long distances.
- Lightweight and small diameter.
- Secure — difficult to tap without detection.

**Disadvantages:**
- More expensive than copper.
- Requires specialized tools and skills to splice and connect.
- More fragile (glass core can crack).

---

## 9. Unguided (Wireless) Transmission Media

Wireless communication transmits signals through free space. The signal propagates as **electromagnetic waves**.

### 9.1 Propagation Methods

| Method | Description | Use |
|---|---|---|
| **Ground propagation** | Signals travel along the Earth's surface | AM radio (low frequency) |
| **Sky propagation** | Signals reflect off the ionosphere | Shortwave radio |
| **Line-of-sight (LOS)** | Signals travel in a straight line | Microwave, satellite, Wi-Fi |

### 9.2 Radio Waves

- **Frequency range:** 3 Hz to 1 GHz
- **Omnidirectional** — broadcast in all directions.
- Can penetrate walls; travels long distances.
- Subject to multipath interference.

**Uses:** AM/FM radio, cellular networks (3G/4G LTE), wireless LANs (Wi-Fi), Bluetooth.

**Sub-bands:**

| Band | Frequency Range | Application |
|---|---|---|
| VLF (Very Low Frequency) | 3–30 kHz | Submarine communication |
| LF (Low Frequency) | 30–300 kHz | Navigation |
| MF (Medium Frequency) | 300 kHz – 3 MHz | AM radio |
| HF (High Frequency) | 3–30 MHz | Shortwave radio |
| VHF (Very High Frequency) | 30–300 MHz | FM radio, VHF TV |
| UHF (Ultra High Frequency) | 300 MHz – 3 GHz | Cellular, UHF TV, Wi-Fi |

### 9.3 Microwaves

- **Frequency range:** 1 GHz to 300 GHz
- **Unidirectional** — focused beam requires line-of-sight alignment.
- Cannot penetrate solid obstacles easily.

**Uses:**

| Type | Description |
|---|---|
| **Terrestrial microwave** | Point-to-point links between towers; long-distance telephone, TV backbone |
| **Satellite microwave** | Uses geostationary (GEO), medium-earth orbit (MEO), or low-earth orbit (LEO) satellites |

**Satellite Orbit Comparison:**

| Type | Altitude | Latency | Coverage | Example |
|---|---|---|---|---|
| GEO | ~35,786 km | ~270 ms | 1/3 of Earth | Traditional satellite TV |
| MEO | 5,000–15,000 km | ~50–150 ms | Mid-range coverage | GPS |
| LEO | 200–2,000 km | ~20–40 ms | Requires constellation | Starlink, OneWeb |

### 9.4 Infrared

- **Frequency range:** 300 GHz to 400 THz
- Very short range; **cannot penetrate walls**.
- Used in line-of-sight, room-scale communication.

**Uses:** TV remote controls, short-range device communication (IrDA), some point-of-sale systems.

**Limitation:** Blocked by solid objects; susceptible to interference from sunlight and fluorescent lighting.

---

## 10. Transmission Media Comparison

| Medium | Bandwidth | Distance | Cost | EMI Susceptibility | Use Case |
|---|---|---|---|---|---|
| UTP (Cat 5e/6) | Up to 250 MHz | Up to 100 m | Low | High | Office LAN, telephone |
| STP | Up to 600 MHz | Up to 100 m | Medium | Low | Industrial, noisy environments |
| Coaxial Cable | Up to 1 GHz | Up to several km | Medium | Low | Cable TV, broadband |
| Fiber-Optic (MMF) | Tens of GHz | Up to 2 km | High | None | LAN backbone |
| Fiber-Optic (SMF) | Tens of THz | 100+ km | Very High | None | WAN, ISP backbone |
| Radio Wave | Up to 1 GHz | Very wide area | Variable | High | Cellular, Wi-Fi, broadcasting |
| Microwave | 1–300 GHz | LOS (km–GEO) | High | Medium | Satellite, point-to-point links |
| Infrared | 300 GHz–400 THz | Very short (<10 m) | Low | Medium | Remotes, IrDA |

---

## 11. Digital Transmission: Line Coding

**Line coding** is the process of converting binary data (0s and 1s) into a digital signal. Different schemes are used to improve synchronization, reduce DC components, and improve error detection.

### 11.1 Categories of Line Coding

**1. Unipolar NRZ (Non-Return to Zero)**
- Uses only one voltage level (e.g., +V for 1, 0 V for 0).
- Simple but has DC component and synchronization issues.

**2. Polar NRZ**
- Uses two voltage levels (e.g., +V for 1, −V for 0).
- *NRZ-L:* Voltage level defines bit value.
- *NRZ-I:* Voltage changes at the start of a 1-bit; no change for 0-bit.

**3. Return to Zero (RZ)**
- Signal returns to 0 between each bit.
- Provides built-in synchronization (self-clocking) but requires more bandwidth.

**4. Manchester Encoding**
- Each bit has a **transition in the middle**: Low-to-High = 1, High-to-Low = 0.
- Used in **10BASE-T Ethernet**.
- Self-clocking; no DC component.

**5. Differential Manchester**
- Always has a mid-bit transition (for clocking).
- Bit value determined by whether there is a transition **at the start** of the bit:
  - Transition at start = 0
  - No transition at start = 1
- Used in **Token Ring** networks.

**6. Bipolar AMI (Alternate Mark Inversion)**
- 0 is represented by no voltage.
- 1 alternates between +V and −V.
- No DC component; easy error detection (a violation of alternation indicates an error).

### 11.2 Line Coding Comparison

| Scheme | DC Component | Self-Clocking | Bandwidth | Error Detection |
|---|---|---|---|---|
| NRZ-L | Yes | No | Low | Poor |
| NRZ-I | Yes | Partial | Low | Poor |
| Manchester | No | Yes | Double NRZ | Good |
| Diff. Manchester | No | Yes | Double NRZ | Good |
| Bipolar AMI | No | Partial | Medium | Good |

---

## 12. Analog Transmission: Modulation

When digital data must be transmitted over an analog medium (e.g., a telephone line), or when an analog signal must be transmitted at a different frequency, **modulation** is used.

**Modulation** encodes data by varying one or more properties of a **carrier wave**: amplitude, frequency, or phase.

### 12.1 Amplitude Shift Keying (ASK)

- The **amplitude** of the carrier is changed to represent binary data.
- Binary 1 → Higher amplitude; Binary 0 → Lower amplitude (or 0).
- Simple but susceptible to noise (noise affects amplitude directly).

### 12.2 Frequency Shift Keying (FSK)

- The **frequency** of the carrier is changed to represent data.
- Binary 1 → Higher frequency; Binary 0 → Lower frequency.
- Less susceptible to amplitude noise; used in radio modems.

### 12.3 Phase Shift Keying (PSK)

- The **phase** of the carrier is changed to represent data.
- Binary 1 → Phase shift (e.g., 180°); Binary 0 → No change.
- **BPSK (Binary PSK):** Two phases — one per bit.
- **QPSK (Quadrature PSK):** Four phases — two bits per symbol (more efficient).

### 12.4 Quadrature Amplitude Modulation (QAM)

- Combines **both ASK and PSK** — varies both amplitude and phase simultaneously.
- More bits per symbol; very high data rates.
- Used in: DSL, cable modems, Wi-Fi (802.11), 4G/5G LTE.

**Example — 16-QAM:** 4 amplitude levels × 4 phase shifts = 16 symbols = **4 bits per symbol**.

### 12.5 Modulation Summary

| Technique | Property Changed | Noise Resistance | Efficiency | Common Use |
|---|---|---|---|---|
| ASK | Amplitude | Low | Low | Optical fiber, simple RF |
| FSK | Frequency | Medium | Medium | Radio modems, RFID |
| PSK (BPSK/QPSK) | Phase | High | Medium–High | Wi-Fi, satellite |
| QAM | Amplitude + Phase | Medium | Very High | DSL, cable, cellular |

---

## 13. Multiplexing

**Multiplexing** is a technique that allows multiple signals to share a single transmission medium simultaneously, making efficient use of bandwidth.

```
Signal 1 ─┐
Signal 2 ─┤──► [MUX] ──► [Medium] ──► [DEMUX] ──► Signal 1
Signal 3 ─┘                                        Signal 2
                                                    Signal 3
```

A **Multiplexer (MUX)** combines signals at the sender; a **Demultiplexer (DEMUX)** splits them back at the receiver.

### 13.1 Frequency Division Multiplexing (FDM)

- Divides the total bandwidth into **separate frequency bands** (channels).
- Each signal is modulated onto a different carrier frequency.
- All signals travel simultaneously on the same medium.
- Used for **analog signals**.

**Examples:** AM/FM radio stations, cable TV channels, traditional telephone landlines (POTS), ADSL.

### 13.2 Wavelength Division Multiplexing (WDM)

- Same concept as FDM but applied to **light frequencies** in fiber-optic cables.
- Each data stream is carried by a different wavelength (color) of light.
- **Dense WDM (DWDM):** Hundreds of wavelengths on one fiber — enables terabit-per-second links.

**Examples:** Long-haul fiber-optic backbone networks, undersea cables.

### 13.3 Time Division Multiplexing (TDM)

- Divides time into fixed **time slots**; each signal gets a slot in rotation.
- All signals take turns using the entire channel bandwidth.
- Used for **digital signals**.

**Types:**

| Type | Description |
|---|---|
| **Synchronous TDM** | Fixed time slots assigned to each source, even if the source has no data |
| **Statistical (Async) TDM** | Slots allocated dynamically — only sources with data transmit; more efficient |

**Examples:** T-carrier (T1/T3) telephone systems, SONET, ISDN.

### 13.4 Code Division Multiplexing / Multiple Access (CDM/CDMA)

- Each signal is encoded with a unique **spreading code** (chip sequence).
- All signals transmit simultaneously over the **same frequency and time**.
- Receivers use the matching code to extract only their signal.

**Examples:** 3G cellular networks (CDMA2000), GPS.

---

## 14. Switching

When data must travel through a network from source to destination, it passes through intermediate devices. **Switching** defines how that path is established.

### 14.1 Circuit Switching

- A **dedicated physical path** is established between sender and receiver before data transfer begins.
- The path remains reserved for the entire duration of the communication.
- Resources are **guaranteed** but can be wasted when idle.

**Phases:**
1. **Connection establishment** — Path is set up end-to-end.
2. **Data transfer** — Data flows along the dedicated path.
3. **Connection teardown** — Path is released when done.

**Advantages:**
- Guaranteed bandwidth and low latency.
- Suitable for real-time voice.

**Disadvantages:**
- Inefficient — resources reserved even during silences or pauses.
- Slow connection setup time.

**Example:** Traditional telephone networks (PSTN).

### 14.2 Packet Switching

- Data is divided into small **packets**, each containing a header with addressing information.
- Packets travel **independently** through the network, possibly along different paths.
- No dedicated path is established beforehand.

**Types:**

| Type | Description |
|---|---|
| **Datagram (connectionless)** | Each packet is routed independently; no fixed path. Used by the Internet (IP). |
| **Virtual circuit (connection-oriented)** | A logical path is established before transmission; packets follow the same route. Used in ATM, MPLS. |

**Advantages:**
- Efficient use of network resources.
- Fault-tolerant — packets can be rerouted if a link fails.

**Disadvantages:**
- Variable delay (jitter) due to queuing.
- Packets may arrive out of order and need reassembly.

**Example:** The Internet (IP-based packet switching).

### 14.3 Circuit Switching vs. Packet Switching

| Feature | Circuit Switching | Packet Switching |
|---|---|---|
| Path | Dedicated, fixed | Dynamic, per packet |
| Resources | Reserved | Shared |
| Delay | Low and constant | Variable (jitter) |
| Efficiency | Low when idle | High |
| Failure handling | Connection lost | Packets rerouted |
| Example | PSTN (telephone) | Internet (IP) |

---

## 15. Summary

This module covered the Physical Layer — the lowest level of network communication:

- The **Physical Layer** is responsible for bit transmission, data rate, synchronization, and physical topology.
- **Data and signals** can be analog (continuous) or digital (discrete); both can be impaired by attenuation, distortion, and noise.
- **Nyquist's theorem** gives the maximum data rate in a noiseless channel; **Shannon's theorem** gives the theoretical limit for noisy channels.
- **Guided media** include twisted-pair (UTP/STP), coaxial cable, and fiber-optic; each has different bandwidth, distance, and cost trade-offs.
- **Unguided media** include radio waves, microwaves, and infrared, each used in different wireless contexts.
- **Line coding** converts digital data into digital signals (Manchester, NRZ, Bipolar AMI, etc.).
- **Modulation** converts digital data for analog transmission (ASK, FSK, PSK, QAM).
- **Multiplexing** (FDM, WDM, TDM, CDMA) allows multiple signals to share a medium efficiently.
- **Switching** techniques (circuit vs. packet) determine how data is routed through a network.

---

## 16. Key Terms

| Term | Definition |
|---|---|
| **Physical Layer** | OSI Layer 1; responsible for transmitting raw bits over a physical medium |
| **Analog signal** | A continuously varying electromagnetic wave |
| **Digital signal** | A signal with discrete levels representing binary data |
| **Attenuation** | Loss of signal strength over distance |
| **SNR** | Signal-to-Noise Ratio; measure of signal quality relative to background noise |
| **Nyquist Theorem** | Defines maximum data rate for a noiseless channel |
| **Shannon's Theorem** | Defines the theoretical maximum data rate considering noise |
| **UTP** | Unshielded Twisted Pair; most common LAN cabling |
| **Fiber-optic cable** | Transmits data as light pulses through glass/plastic core |
| **Guided media** | Physical wired transmission paths (UTP, coax, fiber) |
| **Unguided media** | Wireless transmission through air/space (radio, microwave, infrared) |
| **Line coding** | Converting digital bits to digital signals for transmission |
| **Modulation** | Encoding data by varying a carrier wave's amplitude, frequency, or phase |
| **Multiplexing** | Combining multiple signals onto one medium |
| **FDM** | Frequency Division Multiplexing; splits medium into frequency bands |
| **TDM** | Time Division Multiplexing; splits medium into time slots |
| **WDM** | Wavelength Division Multiplexing; uses different light wavelengths in fiber |
| **Circuit switching** | Dedicated path established before data transfer (e.g., PSTN) |
| **Packet switching** | Data broken into packets routed independently (e.g., the Internet) |
| **QAM** | Quadrature Amplitude Modulation; varies both amplitude and phase for high efficiency |

---

## 17. Review Questions

**Conceptual Questions:**

1. What are the five main responsibilities of the Physical Layer in the OSI model?
2. What is the difference between analog and digital data? Give one real-world example of each.
3. A signal loses 50% of its power traveling through a cable. Calculate the attenuation in dB. Show your work.
4. How do Nyquist's and Shannon's theorems differ? Under what conditions would you apply each?
5. Compare UTP, coaxial cable, and fiber-optic cable in terms of bandwidth, cost, and susceptibility to EMI.
6. What is the difference between guided and unguided transmission media? Give two examples of each.
7. Why does Manchester encoding use double the bandwidth of NRZ but is still preferred for some applications?
8. Explain the difference between FDM and TDM. Which is used for analog signals, and which for digital?
9. What are the three phases of circuit switching? Why is it considered inefficient?
10. How does packet switching handle network failures better than circuit switching?

**Application / Calculation Questions:**

11. A noiseless channel has a bandwidth of 4000 Hz and supports 4 signal levels. Using the Nyquist theorem, calculate the maximum data rate.
12. A channel has a bandwidth of 5 MHz and an SNR of 1023. Using Shannon's theorem, calculate the maximum channel capacity.
13. An organization needs to connect two buildings 80 meters apart within a noisy industrial facility. Which transmission medium would you recommend, and why?
14. A company uses QPSK modulation. How many bits are transmitted per symbol? If the symbol rate is 4 million symbols per second, what is the bit rate?
15. A fiber-optic backbone carries 40 wavelengths, each supporting 10 Gbps using WDM. What is the total capacity of the link?

---

*End of Module 2 | Next: Module 3 – Data Link Layer*
