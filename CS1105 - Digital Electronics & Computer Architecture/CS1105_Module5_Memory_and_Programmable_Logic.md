# CS 1105 — Module 5: Memory and Programmable Logic

---

## 1. Why Memory Matters

Every computer needs two things: a place to **compute** (the CPU) and a place to **store** data and instructions (memory). Understanding memory types, organization, and hierarchy is essential for understanding how computers actually run programs.

---

## 2. Memory Basics

### Key Terms

| Term        | Meaning                                          |
|-------------|--------------------------------------------------|
| **Bit**     | Smallest unit — a single 0 or 1                  |
| **Byte**    | 8 bits                                           |
| **Word**    | CPU's natural data size (e.g., 32-bit or 64-bit) |
| **Address** | Unique identifier for each memory location       |
| **Capacity**| Total storage = 2ⁿ locations × word size        |

### Address Space

If a memory has **n address lines**, it can access **2ⁿ** unique locations.

```
Example: 16 address lines → 2¹⁶ = 65,536 locations (64 KB)
```

---

## 3. Memory Types

### 3.1 RAM — Random Access Memory

Read and write during normal operation. **Volatile** — loses data when power is off.

| Type  | Full Name               | How it stores data            | Speed  | Cost   |
|-------|-------------------------|-------------------------------|--------|--------|
| SRAM  | Static RAM              | Flip-flops (6 transistors)    | Fast   | High   |
| DRAM  | Dynamic RAM             | Capacitor + transistor        | Slower | Low    |

> **SRAM** is used for CPU cache. **DRAM** is used for main memory (your RAM sticks).

DRAM must be **refreshed** periodically because capacitors leak charge.

---

### 3.2 ROM — Read-Only Memory

**Non-volatile** — retains data without power. Typically written once or infrequently.

| Type   | Description                                      |
|--------|--------------------------------------------------|
| ROM    | Mask-programmed at factory; never changeable     |
| PROM   | Programmable once by the user (fuses)            |
| EPROM  | Erasable with UV light, then reprogrammable      |
| EEPROM | Electrically erasable, byte by byte              |
| Flash  | Block-erasable EEPROM — used in SSDs, USB drives |

> Your computer's BIOS/UEFI firmware is stored in Flash memory.

---

## 4. Memory Hierarchy

Faster memory is more expensive and smaller. Systems use a **hierarchy** to balance speed and cost.

```
         ┌─────────────┐
         │  Registers  │  ← Fastest, smallest (~bytes), inside CPU
         ├─────────────┤
         │    Cache    │  ← Very fast (SRAM), small (KB–MB)
         │  (L1/L2/L3) │
         ├─────────────┤
         │  Main RAM   │  ← Fast (DRAM), medium (GB)
         ├─────────────┤
         │  SSD / HDD  │  ← Slow, large (TB), non-volatile
         └─────────────┘
         ↑ Speed / Cost
         ↓ Capacity
```

### Locality of Reference

Cache works because programs tend to:
- **Temporal locality** — reuse the same data soon after accessing it
- **Spatial locality** — access data near recently accessed locations

---

## 5. Cache Memory

Cache sits between the CPU and main RAM, holding **recently used data**.

### Cache Hit vs. Miss

| Event      | Meaning                                  | Result          |
|------------|------------------------------------------|-----------------|
| Cache Hit  | Data found in cache                      | Fast access      |
| Cache Miss | Data not in cache, fetched from RAM      | Slower access    |

### Cache Mapping Strategies

| Method        | How it works                                      |
|---------------|---------------------------------------------------|
| Direct Mapped | Each RAM block maps to exactly one cache line     |
| Fully Assoc.  | Any RAM block can go in any cache line            |
| Set Assoc.    | Compromise — groups of cache lines per set        |

---

## 6. Programmable Logic Devices (PLDs)

Instead of building circuits from individual gates, PLDs allow you to **program logic into a chip**.

### 6.1 PLA — Programmable Logic Array

Both the AND plane and OR plane are programmable.

```
Inputs → [Programmable AND plane] → Product terms
       → [Programmable OR plane]  → Outputs
```

Can implement **any** sum-of-products expression.

---

### 6.2 PAL — Programmable Array Logic

Only the **AND plane** is programmable; OR connections are fixed.

- Simpler and faster than PLA
- Less flexible — fixed number of product terms per output
- Common in older embedded systems

---

### 6.3 CPLD — Complex PLD

Multiple PAL-like blocks connected by a programmable interconnect.

- Larger capacity than a single PAL
- Predictable timing (good for control logic)
- Still used in industrial and embedded applications

---

### 6.4 FPGA — Field-Programmable Gate Array

The most powerful and flexible PLD. Instead of AND/OR planes, FPGAs use:

- **LUTs (Look-Up Tables)** — small RAM that implements any logic function
- **Flip-flops** — for sequential logic
- **Interconnect** — programmable wiring between blocks
- **DSP blocks, RAM blocks, I/O** — specialized hard resources

```
FPGA workflow:
  Write HDL (VHDL or Verilog)
       ↓
  Synthesize → Map → Place → Route
       ↓
  Program bitstream onto chip
```

> FPGAs are used in prototyping, signal processing, networking hardware, and AI accelerators.

---

## 7. Memory Organization in a System

A typical system connects CPU, RAM, and ROM over a shared **bus**.

```
     ┌─────────────┐
     │     CPU     │
     └──────┬──────┘
            │ Address / Data / Control Bus
     ┌──────┴──────────────┐
     │                     │
┌────┴─────┐         ┌─────┴────┐
│   RAM    │         │   ROM    │
│(program  │         │ (BIOS /  │
│  data)   │         │firmware) │
└──────────┘         └──────────┘
```

### Memory-Mapped I/O

Peripheral devices (keyboard, display) are assigned **memory addresses** so the CPU can communicate with them using the same read/write instructions as memory.

---

## 8. Worked Example — Memory Capacity

**Problem:** A memory chip has 12 address lines and stores 8-bit words. What is its total capacity?

```
Locations = 2¹² = 4,096 = 4K locations
Word size = 8 bits = 1 byte

Total capacity = 4K × 8 bits = 32 Kbits = 4 KB
```

**Problem 2:** How many address lines are needed for a 1 MB memory with 8-bit words?

```
1 MB = 2²⁰ locations
→ Need 20 address lines
```

---

## 📝 Module 5 Quiz

**Q1.** What is the main difference between SRAM and DRAM?

<details>
<summary>Show Answer</summary>

**SRAM** uses flip-flops to store bits — it is faster, more expensive, and does not need refreshing. It is used for CPU cache.

**DRAM** uses capacitors — it is slower, cheaper, denser, but must be refreshed periodically to prevent data loss. It is used for main memory.
</details>

---

**Q2.** A memory has 14 address lines and a 16-bit word size. What is its total capacity in KB?

<details>
<summary>Show Answer</summary>

```
Locations = 2¹⁴ = 16,384
Word size = 16 bits = 2 bytes

Total = 16,384 × 2 bytes = 32,768 bytes = 32 KB
```
</details>

---

**Q3.** What is a cache miss, and what happens when one occurs?

<details>
<summary>Show Answer</summary>

A **cache miss** occurs when the CPU requests data that is not currently in the cache. The system must fetch the data from slower main RAM, load it into the cache, then deliver it to the CPU — resulting in a delay called a **miss penalty**.
</details>

---

**Q4.** What is the key difference between a PLA and a PAL?

<details>
<summary>Show Answer</summary>

In a **PLA**, both the AND plane and OR plane are programmable — offering maximum flexibility.

In a **PAL**, only the AND plane is programmable; the OR connections are fixed. This makes PALs faster and simpler, but less flexible (fixed number of product terms per output).
</details>

---

**Q5.** What does an FPGA use instead of traditional AND/OR logic planes?

<details>
<summary>Show Answer</summary>

FPGAs use **Look-Up Tables (LUTs)** — small configurable RAM blocks that can implement any logic function for a given number of inputs. LUTs are combined with flip-flops, programmable interconnect, and specialized blocks (DSP, RAM, I/O) to build complete digital systems on a single chip.
</details>

---

## 🔑 Key Takeaways

- Memory is classified by volatility (RAM vs. ROM) and access method.
- SRAM is fast but expensive (cache); DRAM is dense and cheap (main memory).
- The memory hierarchy balances speed and cost using registers, cache, RAM, and storage.
- Cache exploits temporal and spatial locality to speed up memory access.
- PLDs (PLA, PAL, CPLD, FPGA) let designers implement custom logic without building from scratch — FPGAs being the most powerful and widely used today.

---

*Next up → **Module 6: Computer Architecture***
