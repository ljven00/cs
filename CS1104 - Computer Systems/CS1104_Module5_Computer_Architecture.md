# CS 1104 Computer Systems
## Module 5: Computer Architecture

---

## Overview

The previous four modules gave you the building blocks: gates, arithmetic, combinational circuits, and memory elements. Now we assemble them into a complete computer. This module covers the **architecture** of a CPU — how its components are organized, how data moves between them, and how a machine actually executes a program, one instruction at a time.

---

## 1. The Big Picture: Von Neumann Architecture

Almost every computer you'll ever encounter is based on the **Von Neumann model** (proposed by John Von Neumann in 1945). It has five key components:

```
         ┌─────────────────────────────┐
         │            CPU              │
         │  ┌──────────┐ ┌─────────┐  │
         │  │  Control │ │   ALU   │  │
         │  │   Unit   │ │         │  │
         │  └──────────┘ └─────────┘  │
         │       ┌────────────┐        │
         │       │  Registers │        │
         └───────┴─────┬──────┴────────┘
                       │  (Bus)
          ┌────────────┼────────────┐
          ↓            ↓            ↓
       [Memory]     [Input]      [Output]
```

### The Components

| Component | Role |
|-----------|------|
| **CPU** | Executes instructions |
| **Memory (RAM)** | Stores both program instructions and data |
| **Input** | Keyboard, mouse, sensors, etc. |
| **Output** | Screen, speakers, etc. |
| **Bus** | Wires connecting everything |

> **Von Neumann's key insight:** Instructions and data are stored in the **same memory**, in the same format (binary). This is called the **stored-program concept** — it's what makes software possible.

---

## 2. Inside the CPU

The CPU has three main parts:

### 2.1 Registers

Small, ultra-fast storage built from flip-flops (Module 4). There are a handful of special-purpose registers:

| Register | Name | Purpose |
|----------|------|---------|
| **PC** | Program Counter | Holds the address of the **next** instruction to fetch |
| **IR** | Instruction Register | Holds the **current** instruction being executed |
| **MAR** | Memory Address Register | Holds the memory address to read/write |
| **MDR** | Memory Data Register | Holds data being transferred to/from memory |
| **ACC** | Accumulator | Holds ALU results (in simple architectures) |

Plus several **general-purpose registers** (e.g., R0–R7) that programs use freely.

### 2.2 The ALU

The ALU from Module 3 — performs all arithmetic and logic operations on register values.

### 2.3 The Control Unit (CU)

The **brain of the CPU**. It reads instructions and generates the control signals that tell every other component what to do and when. It uses a decoder (Module 3) to interpret opcodes.

---

## 3. Memory

**RAM (Random Access Memory)** is organized as an array of addressable locations. Each location stores a fixed number of bits (typically 8 bits = 1 byte).

```
Address │ Contents
────────┼──────────
  0x00  │ 0010 1100
  0x01  │ 1000 0001
  0x02  │ 0000 0101
  0x03  │ 0110 1110
  ...   │ ...
```

- To **read**: put the address in MAR → memory puts the value in MDR
- To **write**: put the address in MAR, the value in MDR → memory stores it

Memory is **much slower** than registers. A register access takes 1 clock cycle; RAM can take hundreds. This gap is bridged by **cache memory** (a smaller, faster memory between the CPU and RAM).

---

## 4. The Bus

The **bus** is a set of shared wires connecting the CPU, memory, and I/O devices. There are three types:

| Bus | Carries | Width |
|-----|---------|-------|
| **Address Bus** | Memory addresses (CPU → Memory) | 32-bit = 4 GB addressable |
| **Data Bus** | Actual data (bidirectional) | 32 or 64 bits |
| **Control Bus** | Read/Write signals, clock, interrupts | Varies |

---

## 5. The Fetch-Decode-Execute Cycle

This is the heartbeat of every CPU. It repeats endlessly, billions of times per second.

```
┌─────────┐     ┌─────────┐     ┌─────────┐
│  FETCH  │────→│ DECODE  │────→│ EXECUTE │
└─────────┘     └─────────┘     └─────────┘
     ↑                                │
     └────────────────────────────────┘
```

### Step 1: FETCH
1. Copy PC → MAR
2. Read memory at MAR → MDR
3. Copy MDR → IR  *(now the instruction is in IR)*
4. Increment PC  *(point to the next instruction)*

### Step 2: DECODE
The Control Unit examines the opcode in IR and figures out:
- What operation is this? (ADD, LOAD, JUMP, etc.)
- What operands does it need?
- Which registers or memory addresses are involved?

### Step 3: EXECUTE
Carry out the operation. Depending on the instruction, this may involve:
- Sending operands to the ALU and storing the result
- Reading from or writing to memory
- Updating the PC to a new address (for jumps/branches)

Then the cycle **immediately repeats** — FETCH the next instruction.

---

## 6. A Worked Example

Suppose memory contains this simple program (we'll use symbolic names for now):

```
Address  │  Instruction
─────────┼──────────────────
  0x00   │  LOAD R1, 0x10     ← Load value at address 0x10 into R1
  0x01   │  LOAD R2, 0x11     ← Load value at address 0x11 into R2
  0x02   │  ADD  R3, R1, R2   ← R3 = R1 + R2
  0x03   │  STORE R3, 0x12    ← Store R3 into address 0x12

  0x10   │  0000 0101         ← Data: 5
  0x11   │  0000 0011         ← Data: 3
```

**Trace:**

| Cycle | PC  | Action                          | State after         |
|-------|-----|---------------------------------|---------------------|
| 1     | 0x00| Fetch LOAD R1, 0x10             | IR = LOAD R1, 0x10  |
| 1     | 0x01| Decode + Execute: R1 ← mem[0x10]| R1 = 5              |
| 2     | 0x01| Fetch LOAD R2, 0x11             | IR = LOAD R2, 0x11  |
| 2     | 0x02| Decode + Execute: R2 ← mem[0x11]| R2 = 3              |
| 3     | 0x02| Fetch ADD R3, R1, R2            | IR = ADD R3, R1, R2 |
| 3     | 0x03| Decode + Execute: R3 ← 5 + 3   | R3 = 8              |
| 4     | 0x03| Fetch STORE R3, 0x12            | IR = STORE R3, 0x12 |
| 4     | 0x04| Decode + Execute: mem[0x12] ← 8 | Memory[0x12] = 8    |

---

## 7. Harvard vs. Von Neumann Architecture

Modern CPUs often use a variation called **Harvard Architecture**, which uses **separate memories** for instructions and data:

| Feature | Von Neumann | Harvard |
|---------|-------------|---------|
| Memory | Single shared | Separate instruction + data |
| Bus | Shared | Separate buses |
| Bottleneck | Can't fetch+read data simultaneously | Can do both at once |
| Used in | Most general-purpose CPUs (logically) | Microcontrollers, CPUs with cache |

In practice, modern CPUs use Von Neumann logically but Harvard-style **caches** internally — getting the best of both worlds.

---

## 8. The Memory Hierarchy

Not all memory is equal. There's a fundamental trade-off between **speed** and **size**:

```
         Fastest / Smallest / Most Expensive
         ┌──────────────┐
         │   Registers  │  ~1 ns   (handful of locations)
         ├──────────────┤
         │   L1 Cache   │  ~4 ns   (KB range)
         ├──────────────┤
         │   L2 Cache   │  ~12 ns  (MB range)
         ├──────────────┤
         │   L3 Cache   │  ~40 ns  (MB–GB range)
         ├──────────────┤
         │     RAM      │  ~100 ns (GB range)
         ├──────────────┤
         │     Disk     │  ~1 ms   (TB range)
         └──────────────┘
         Slowest / Largest / Cheapest
```

The CPU always looks for data in the fastest level first, falling back to slower levels only when needed (a **cache miss**).

---

## Summary

| Concept | Key Idea |
|---------|----------|
| Von Neumann Model | CPU + Memory + I/O connected by a bus; instructions and data share memory |
| Registers | Ultra-fast in-CPU storage (PC, IR, MAR, MDR, general purpose) |
| Control Unit | Decodes instructions and drives control signals |
| Fetch-Decode-Execute | The CPU's endless loop; one iteration per instruction |
| Address/Data/Control Bus | Three types of wires connecting CPU to the rest of the system |
| Memory Hierarchy | Speed/size trade-off from registers down to disk |

---

## 🧠 Module Quiz

1. What is the **stored-program concept**, and why is it significant? Which architecture introduced it?

2. Trace the Fetch-Decode-Execute cycle for the instruction `ADD R3, R1, R2`. What happens in each step? Which registers are involved?

3. A CPU has a 16-bit address bus. What is the maximum amount of memory it can address? Show your calculation.

4. What is the role of the **Program Counter (PC)**? What happens to it during a normal instruction fetch? What happens to it during a JUMP instruction?

5. Explain the difference between the **Address Bus** and the **Data Bus**. Can they carry the same kinds of information? Why are they kept separate?

6. A program runs slowly because data is frequently needed from RAM rather than cache. What is this problem called, and what programming practice could help reduce it?
