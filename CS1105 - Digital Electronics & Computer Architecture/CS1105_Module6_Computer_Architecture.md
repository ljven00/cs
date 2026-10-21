# CS 1105 — Module 6: Computer Architecture

---

## 1. What is Computer Architecture?

Computer architecture defines how a computer system is **organized and how its components interact** — the CPU, memory, I/O, and the buses connecting them.

Two landmark design philosophies dominate:

| Model       | Full Name                        | Key Trait                              |
|-------------|----------------------------------|----------------------------------------|
| **Von Neumann** | Von Neumann Architecture     | Single shared memory for data & code   |
| **Harvard**  | Harvard Architecture            | Separate memories for data & code      |

> Most modern CPUs are Von Neumann at the system level but use Harvard-style **internal caches** for speed.

---

## 2. The Von Neumann Architecture

```
         ┌──────────────────────────────┐
         │             CPU              │
         │  ┌──────────┐  ┌──────────┐ │
         │  │   ALU    │  │ Control  │ │
         │  │          │  │  Unit    │ │
         │  └──────────┘  └──────────┘ │
         │        ↑ Registers ↑        │
         └───────────────┬─────────────┘
                         │
                    System Bus
                         │
         ┌───────────────┼───────────────┐
         │               │               │
    ┌────┴────┐     ┌────┴────┐    ┌────┴────┐
    │  Memory │     │   I/O   │    │ Storage │
    │(RAM/ROM)│     │ Devices │    │         │
    └─────────┘     └─────────┘    └─────────┘
```

### The Von Neumann Bottleneck

Because data and instructions share one bus, the CPU is often idle waiting for memory. This is a fundamental limitation — mitigated by **cache** and **pipelining**.

---

## 3. CPU Components

### 3.1 The Control Unit (CU)

Directs all CPU activity. It:
- Fetches instructions from memory
- Decodes what each instruction means
- Signals other components to execute it

### 3.2 The ALU

Performs all arithmetic and logic operations (covered in Module 4).

### 3.3 Registers

Small, ultra-fast storage **inside the CPU**. Key registers:

| Register | Name                     | Purpose                                    |
|----------|--------------------------|--------------------------------------------|
| PC       | Program Counter          | Address of the **next** instruction        |
| IR       | Instruction Register     | Holds the **current** instruction          |
| MAR      | Memory Address Register  | Address being read from/written to         |
| MDR      | Memory Data Register     | Data being transferred to/from memory      |
| ACC      | Accumulator              | Holds ALU results                          |
| SP       | Stack Pointer            | Points to top of the stack                 |

---

## 4. The Fetch-Decode-Execute Cycle

Every instruction a CPU runs goes through this cycle — millions of times per second.

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│   1. FETCH      PC → MAR → Memory → MDR → IR       │
│                 PC increments to next instruction   │
│                                                     │
│   2. DECODE     Control Unit interprets IR          │
│                 Identifies opcode & operands        │
│                                                     │
│   3. EXECUTE    ALU computes / memory accessed /    │
│                 result stored in register           │
│                                                     │
└─────────────────────────────────────────────────────┘
          ↑_____________repeat_________________↑
```

### Example — ADD instruction

```
Instruction: ADD R1, R2   (R1 ← R1 + R2)

Fetch:   Load instruction from memory[PC] into IR
Decode:  CU identifies ADD opcode, operands R1 and R2
Execute: ALU adds R1 + R2, result stored in R1
         Flags updated (Zero, Carry, Overflow, Negative)
```

---

## 5. Instruction Set Architecture (ISA)

The ISA is the **contract between hardware and software** — it defines every instruction the CPU can execute.

### Instruction Format

```
| Opcode | Operand 1 | Operand 2 | Operand 3 |
```

- **Opcode** — what operation to do (ADD, LOAD, JUMP...)
- **Operands** — what data to operate on (registers, memory addresses, constants)

### Addressing Modes

| Mode          | Description                              | Example              |
|---------------|------------------------------------------|----------------------|
| Immediate     | Value is in the instruction itself       | ADD R1, #5           |
| Register      | Value is in a register                   | ADD R1, R2           |
| Direct        | Value is at a given memory address       | LOAD R1, 0x200       |
| Indirect      | Address of value is in a register        | LOAD R1, [R2]        |
| Indexed       | Base address + offset                    | LOAD R1, 0x200[R2]   |

---

## 6. RISC vs. CISC

Two competing ISA design philosophies:

| Feature          | RISC                          | CISC                          |
|------------------|-------------------------------|-------------------------------|
| Full name        | Reduced Instruction Set       | Complex Instruction Set       |
| Instructions     | Few, simple, fixed-length     | Many, complex, variable-length|
| Execution        | 1 cycle per instruction (goal)| Multiple cycles per instruction|
| Memory access    | Load/Store only               | Any instruction can access memory|
| Examples         | ARM, RISC-V, MIPS             | x86, x86-64 (Intel/AMD)       |

> Your phone almost certainly runs **ARM** (RISC). Your laptop likely runs **x86-64** (CISC).

---

## 7. Pipelining

Instead of finishing one instruction before starting the next, pipelining **overlaps stages**.

### 5-Stage Pipeline

```
Cycle:    1     2     3     4     5     6     7
Inst 1:  [IF]  [ID]  [EX]  [MEM] [WB]
Inst 2:        [IF]  [ID]  [EX]  [MEM] [WB]
Inst 3:              [IF]  [ID]  [EX]  [MEM] [WB]
```

- **IF** — Instruction Fetch
- **ID** — Instruction Decode
- **EX** — Execute (ALU)
- **MEM** — Memory Access
- **WB** — Write Back to register

Ideal speedup: **N times faster** for an N-stage pipeline.

### Pipeline Hazards

| Hazard Type  | Cause                                              | Solution                    |
|--------------|----------------------------------------------------|-----------------------------|
| Data hazard  | Instruction needs result not yet written back      | Forwarding, stalling        |
| Control hazard | Branch destination unknown until execute stage  | Branch prediction           |
| Structural hazard | Two instructions need the same hardware      | Duplicate hardware, stalling|

---

## 8. The System Bus

Three buses connect the CPU to memory and I/O:

| Bus          | Carries                         | Direction         |
|--------------|---------------------------------|-------------------|
| Address Bus  | Memory address to access        | CPU → Memory/I/O  |
| Data Bus     | Actual data being transferred   | Bidirectional     |
| Control Bus  | Read/Write signals, interrupts  | Bidirectional     |

Width of the address bus determines **addressable memory**.
Width of the data bus determines **how much data moves per transfer**.

---

## 9. Worked Example — Tracing the Fetch-Decode-Execute Cycle

**Scenario:** Memory at address 0x100 contains the instruction `LOAD R1, 0x200`.  
Memory at 0x200 contains the value `42`. PC starts at 0x100.

```
FETCH:
  MAR ← PC         (MAR = 0x100)
  MDR ← Mem[MAR]   (MDR = LOAD R1, 0x200)
  IR  ← MDR        (IR  = LOAD R1, 0x200)
  PC  ← PC + 1     (PC  = 0x101)

DECODE:
  CU reads IR: opcode = LOAD, dest = R1, src_addr = 0x200

EXECUTE:
  MAR ← 0x200
  MDR ← Mem[MAR]   (MDR = 42)
  R1  ← MDR        (R1  = 42)
```

Result: **R1 now holds the value 42.**

---

## 📝 Module 6 Quiz

**Q1.** What is the Von Neumann bottleneck and how is it mitigated?

<details>
<summary>Show Answer</summary>

The Von Neumann bottleneck is the limitation caused by a **single shared bus** between the CPU and memory for both instructions and data. The CPU is often idle waiting for memory fetches. It is mitigated by **cache memory** (storing frequently used data close to the CPU) and **pipelining** (overlapping instruction stages to keep the CPU busy).
</details>

---

**Q2.** What does the Program Counter (PC) hold, and when does it update?

<details>
<summary>Show Answer</summary>

The **Program Counter** holds the memory address of the **next instruction** to be fetched. It increments automatically during the Fetch stage of each cycle. It can also be overwritten by branch or jump instructions to redirect program flow.
</details>

---

**Q3.** What is the difference between immediate and indirect addressing?

<details>
<summary>Show Answer</summary>

**Immediate addressing:** The operand value is embedded directly in the instruction itself. Example: `ADD R1, #10` — the value 10 is used directly.

**Indirect addressing:** The instruction contains a register whose contents are a **memory address**, and the actual data is at that address. Example: `LOAD R1, [R2]` — R2 holds the address where the data lives.
</details>

---

**Q4.** A 5-stage pipeline processes 100 instructions. Ignoring hazards, how many cycles does it take compared to no pipeline?

<details>
<summary>Show Answer</summary>

**Without pipeline:** 100 instructions × 5 cycles each = **500 cycles**

**With pipeline:** 5 cycles to fill the pipeline + 99 more instructions = **104 cycles**

The pipeline is approximately **4.8× faster** in this case. Ideal speedup approaches 5× as instruction count grows.
</details>

---

**Q5.** What is the key architectural difference between RISC and CISC?

<details>
<summary>Show Answer</summary>

**RISC** uses a small set of simple, fixed-length instructions that mostly execute in one clock cycle. Memory is accessed only via dedicated Load/Store instructions.

**CISC** uses a large set of complex, variable-length instructions where a single instruction can perform multiple operations (including direct memory access). This reduces program size but makes hardware design more complex.
</details>

---

## 🔑 Key Takeaways

- The Von Neumann model uses a single shared memory for code and data — most real systems use it with cache to overcome the bottleneck.
- The CPU's core components are the Control Unit, ALU, and registers — each with a specific role in the fetch-decode-execute cycle.
- The ISA defines every instruction a CPU understands, including addressing modes.
- RISC favors simplicity and speed; CISC favors compact, powerful instructions.
- Pipelining dramatically increases throughput but introduces hazards that must be managed.

---

*Next up → **Module 7: Assembler Language: Concepts and Techniques***
