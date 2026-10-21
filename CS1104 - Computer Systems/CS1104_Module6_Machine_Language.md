# CS 1104 Computer Systems
## Module 6: Machine Language

---

## Overview

In Module 5, you saw instructions like `ADD R3, R1, R2` being fetched and executed. But the CPU doesn't actually understand words — it only understands **binary**. **Machine language** is the actual binary representation of instructions that the CPU reads directly from memory. This module covers how instructions are encoded, what types exist, and how a real CPU interprets them.

---

## 1. What is Machine Language?

Machine language is the **lowest level of programming** — raw binary numbers that the CPU executes directly with no translation needed.

Every instruction is a fixed-width binary pattern stored in memory. When the CPU fetches it into the IR, the Control Unit decodes each field of that pattern to figure out exactly what to do.

```
Human-readable:   ADD  R3, R1, R2
Assembly:         ADD  R3, R1, R2    (one step above machine code)
Machine code:     0001 0011 0001 0010  (what the CPU actually sees)
```

We'll use a simple **16-bit instructon set** throughout this module to keep examples manageable. Real architectures like x86-64 use 32 or 64-bit instructions, but the principles are identical.

---

## 2. Instruction Format

An instruction is divided into **fields**. Each field has a specific meaning and position within the binary word.

### Our 16-bit Instruction Format

```
 15      12  11    9   8     6   5     3   2     0
┌──────────┬────────┬────────┬────────┬──────────┐
│  OPCODE  │   RD   │   RS1  │   RS2  │  (unused)│
│  4 bits  │ 3 bits │ 3 bits │ 3 bits │  3 bits  │
└──────────┴────────┴────────┴────────┴──────────┘
```

| Field | Bits | Meaning |
|-------|------|---------|
| **OPCODE** | 15–12 | Which operation to perform |
| **RD** | 11–9 | Destination register |
| **RS1** | 8–6 | Source register 1 |
| **RS2** | 5–3 | Source register 2 |

With 4 opcode bits → 2⁴ = **16 possible instructions**.  
With 3 register bits → 2³ = **8 registers** (R0 through R7).

---

## 3. Instruction Types

Not all instructions work the same way. There are three main formats:

### 3.1 R-Type (Register)

Operates entirely on registers. All three register fields are used.

```
┌──────────┬────────┬────────┬────────┬──────────┐
│  OPCODE  │   RD   │  RS1   │  RS2   │  unused  │
└──────────┴────────┴────────┴────────┴──────────┘
```

**Example:** `ADD R3, R1, R2` → R3 = R1 + R2

Encoded: opcode=`0001`, RD=R3=`011`, RS1=R1=`001`, RS2=R2=`010`
```
0001  011  001  010  000
```

### 3.2 I-Type (Immediate)

One operand is a **constant value** (called an immediate) baked directly into the instruction instead of coming from a register.

```
┌──────────┬────────┬────────┬──────────────────┐
│  OPCODE  │   RD   │  RS1   │    IMMEDIATE      │
│  4 bits  │ 3 bits │ 3 bits │     6 bits        │
└──────────┴────────┴────────┴──────────────────┘
```

**Example:** `ADDI R1, R0, 5` → R1 = R0 + 5

Encoded: opcode=`0010`, RD=R1=`001`, RS1=R0=`000`, imm=5=`000101`
```
0010  001  000  000101
```

Immediate values are useful for things like `R1 = R1 + 1` (a loop counter) without wasting a register to hold the constant 1.

### 3.3 J-Type (Jump)

Used for jumps and branches. Most of the instruction is taken up by a **target address**.

```
┌──────────┬──────────────────────────────────────┐
│  OPCODE  │           TARGET ADDRESS              │
│  4 bits  │              12 bits                  │
└──────────┴──────────────────────────────────────┘
```

**Example:** `JUMP 0x00A` → PC = 0x00A (jump to address 10)

Encoded: opcode=`1100`, address=`000000001010`
```
1100  000000001010
```

---

## 4. Our Instruction Set

Here is the complete instruction set for our simple 16-bit CPU:

| Opcode | Mnemonic | Type | Operation |
|--------|----------|------|-----------|
| 0000   | NOP      | —    | No operation |
| 0001   | ADD      | R    | RD = RS1 + RS2 |
| 0010   | ADDI     | I    | RD = RS1 + imm |
| 0011   | SUB      | R    | RD = RS1 − RS2 |
| 0100   | AND      | R    | RD = RS1 AND RS2 |
| 0101   | OR       | R    | RD = RS1 OR RS2 |
| 0110   | XOR      | R    | RD = RS1 XOR RS2 |
| 0111   | NOT      | R    | RD = NOT RS1 |
| 1000   | LOAD     | I    | RD = Memory[RS1 + imm] |
| 1001   | STORE    | I    | Memory[RS1 + imm] = RD |
| 1010   | MOV      | R    | RD = RS1 |
| 1011   | CMP      | R    | Set flags from RS1 − RS2 (no result stored) |
| 1100   | JUMP     | J    | PC = address |
| 1101   | JEQ      | J    | If Zero flag set: PC = address |
| 1110   | JNE      | J    | If Zero flag clear: PC = address |
| 1111   | HALT     | —    | Stop execution |

---

## 5. Decoding an Instruction: Step by Step

Let's decode a raw binary instruction from memory:

```
1000  010  001  000011
```

1. **OPCODE = 1000** → This is a `LOAD`
2. **RD = 010** → Destination is R2
3. **RS1 = 001** → Base address is in R1
4. **Immediate = 000011** → Offset is 3

**Result:** Load the value from memory address (R1 + 3) into R2.

In human-readable assembly: `LOAD R2, R1, 3`

---

## 6. A Complete Program in Machine Code

Let's write a program that computes the sum of two numbers stored in memory and saves the result.

**Assume:**
- Address `0x20` holds the value **7**
- Address `0x21` holds the value **4**
- R0 always holds **0** (a common convention)

**Program:**

| Address | Machine Code        | Assembly             | Meaning                    |
|---------|---------------------|----------------------|----------------------------|
| 0x00    | 1000 001 000 100000 | LOAD R1, R0, 0x20   | R1 = Memory[0 + 32] = 7   |
| 0x01    | 1000 010 000 100001 | LOAD R2, R0, 0x21   | R2 = Memory[0 + 33] = 4   |
| 0x02    | 0001 011 001 010 000| ADD  R3, R1, R2     | R3 = 7 + 4 = 11            |
| 0x03    | 1001 011 000 100010 | STORE R3, R0, 0x22  | Memory[0x22] = 11          |
| 0x04    | 1111 000000000000   | HALT                | Stop                       |

---

## 7. Branches and Loops

The real power of a computer is making decisions and repeating work. This is done with **conditional jump instructions** that check the ALU flags.

### Example: Count down from 3 to 0

```
Address  │ Assembly              │ Meaning
─────────┼───────────────────────┼──────────────────────────
  0x00   │ ADDI  R1, R0, 3      │ R1 = 3  (counter)
  0x01   │ ADDI  R2, R0, 1      │ R2 = 1  (decrement amount)
  0x02   │ SUB   R1, R1, R2     │ R1 = R1 − 1       ← loop starts here
  0x03   │ CMP   R1, R0         │ Set Zero flag if R1 == 0
  0x04   │ JNE   0x02           │ If not zero, jump back to 0x02
  0x05   │ HALT                 │ R1 is 0, done
```

**Trace:**

| Iteration | R1 before SUB | R1 after SUB | Zero flag | Jump? |
|-----------|---------------|--------------|-----------|-------|
| 1         | 3             | 2            | 0         | Yes   |
| 2         | 2             | 1            | 0         | Yes   |
| 3         | 1             | 0            | 1         | No    |

The CPU halts with R1 = 0. This loop pattern — decrement, compare, branch — is the machine-level foundation of every `for` and `while` loop you've ever written in a high-level language.

---

## 8. Why Not Just Program in Machine Code?

You technically can, but:

- **It's tedious:** Every instruction is a long binary string
- **It's error-prone:** A single flipped bit is a different instruction
- **It's not readable:** `1000 010 001 000011` vs `LOAD R2, R1, 3`
- **Addresses are hard-coded:** If you insert one instruction, all jump addresses shift

This is exactly why **assembly language** was invented — a human-readable text representation of machine code, with a program called an **assembler** converting it to binary. That's Modules 7 and 8.

---

## Summary

| Concept | Key Idea |
|---------|----------|
| Machine language | Binary instructions executed directly by the CPU |
| Instruction fields | Opcode + register/immediate fields packed into fixed-width bits |
| R-Type | All operands are registers |
| I-Type | One operand is a constant (immediate) embedded in the instruction |
| J-Type | Encodes a target address for jumps |
| Decoding | Control Unit reads each field and routes signals accordingly |
| Branches | Conditional jumps check ALU flags to implement loops and decisions |

---

## 🧠 Module Quiz

1. Decode the following 16-bit instruction using the format from this module:  
   `0011  100  010  011  000`  
   What operation is performed? On which registers?

2. Encode the instruction `ADDI R4, R2, 7` into binary using the I-Type format. Show each field.

3. Why does the I-Type format use fewer bits for the immediate value than a full register operand would allow? What trade-off does this create?

4. Trace the fetch-decode-execute cycle (from Module 5) for the instruction `ADD R3, R1, R2`. Now that you know instruction encoding, describe what the Control Unit sees in the IR at the decode step.

5. In the countdown loop example (Section 7), what would happen if the `CMP` instruction were removed and the `JNE` checked the result of `SUB` directly? Would the program still work? Why or why not?

6. A program has a `JUMP 0x08` instruction at address `0x05`. After this instruction executes, what is the value in the Program Counter? What would the PC have been if the jump had not been there?
