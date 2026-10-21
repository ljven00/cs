# CS 1104 Computer Systems
## Module 8: Assembler II

---

## Overview

In Module 7, Pass 1 scanned the entire assembly source and built a Symbol Table mapping every label to its address. Now comes **Pass 2**: a second scan through the source where each instruction is fully encoded into binary using the Symbol Table, the instruction format from Module 6, and the operand translation rules from Module 7. By the end of Pass 2, the assembler outputs a complete binary file ready to load into memory and execute.

---

## 1. What Pass 2 Does

Pass 2 reads the source top to bottom **a second time**. For each instruction line it:

1. Looks up the **opcode** from the mnemonic table
2. Encodes each **operand** (register → 3-bit code, immediate → binary, label → address from Symbol Table)
3. Assembles all fields into a complete **binary word**
4. Writes that word to the output at the current address

```
Source line:   ADD  R3, R1, R2
                │    │   │   │
                ↓    ↓   ↓   ↓
Opcode lookup: 0001  011 001 010
                              ↓
Output word:   0001 011 001 010 000
```

---

## 2. The Encoding Algorithm

For each instruction type, Pass 2 applies a fixed encoding procedure:

### R-Type Encoding

```
bits 15-12: opcode
bits 11- 9: RD  (destination register)
bits  8- 6: RS1 (source register 1)
bits  5- 3: RS2 (source register 2)
bits  2- 0: 000 (unused)
```

**Example:** `SUB R4, R2, R1`

| Field  | Value | Binary |
|--------|-------|--------|
| Opcode | SUB = 0011 | `0011` |
| RD     | R4 = 4 | `100` |
| RS1    | R2 = 2 | `010` |
| RS2    | R1 = 1 | `001` |
| Unused | —     | `000` |

**Result:** `0011 100 010 001 000`

---

### I-Type Encoding

```
bits 15-12: opcode
bits 11- 9: RD
bits  8- 6: RS1
bits  5- 0: immediate (6-bit signed)
```

**Example:** `ADDI R3, R1, 12`

| Field     | Value | Binary |
|-----------|-------|--------|
| Opcode    | ADDI = 0010 | `0010` |
| RD        | R3 = 3 | `011` |
| RS1       | R1 = 1 | `001` |
| Immediate | 12    | `001100` |

**Result:** `0010 011 001 001100`

> **Sign extension:** If the immediate is negative (e.g., −2 = `111110` in 6-bit Two's Complement), the CPU sign-extends it to the full word width before using it. This is handled automatically by the hardware.

---

### J-Type Encoding

```
bits 15-12: opcode
bits 11- 0: target address (12-bit)
```

**Example:** `JEQ 0x04`

| Field   | Value | Binary |
|---------|-------|--------|
| Opcode  | JEQ = 1101 | `1101` |
| Address | 0x04 = 4 | `000000000100` |

**Result:** `1101 000000000100`

**Example with a label:** `JEQ equal` where Symbol Table says equal = 0x04

The assembler substitutes: `JEQ 0x04` → same encoding as above. This is the payoff from Pass 1.

---

## 3. A Complete Pass 2 Walkthrough

Let's fully assemble the "find larger number" program from Module 7. We already have the Symbol Table:

| Label    | Address |
|----------|---------|
| equal    | 0x04    |
| r1bigger | 0x06    |
| done     | 0x09    |

Now Pass 2 encodes each instruction:

---

**Address 0x00:** `CMP R1, R2`
- Opcode: `1011`, RD: unused=`000`, RS1: R1=`001`, RS2: R2=`010`, unused=`000`
- **Binary:** `1011 000 001 010 000`

---

**Address 0x01:** `JEQ equal` → `JEQ 0x04`
- Opcode: `1101`, Address: 0x04=`000000000100`
- **Binary:** `1101 000000000100`

---

**Address 0x02:** `SUB R4, R1, R2`
- Opcode: `0011`, RD: R4=`100`, RS1: R1=`001`, RS2: R2=`010`, unused=`000`
- **Binary:** `0011 100 001 010 000`

---

**Address 0x03:** `JNE r1bigger` → `JNE 0x06`
- Opcode: `1110`, Address: 0x06=`000000000110`
- **Binary:** `1110 000000000110`

---

**Address 0x04:** `MOV R3, R1`  *(label `equal` points here)*
- Opcode: `1010`, RD: R3=`011`, RS1: R1=`001`, RS2: unused=`000`, unused=`000`
- **Binary:** `1010 011 001 000 000`

---

**Address 0x05:** `JUMP done` → `JUMP 0x09`
- Opcode: `1100`, Address: 0x09=`000000001001`
- **Binary:** `1100 000000001001`

---

**Address 0x06:** `MOV R3, R1`  *(label `r1bigger` points here)*
- **Binary:** `1010 011 001 000 000`  *(same encoding as 0x04)*

---

**Address 0x07:** `JUMP done` → `JUMP 0x09`
- **Binary:** `1100 000000001001`

---

**Address 0x08:** `MOV R3, R2`
- Opcode: `1010`, RD: R3=`011`, RS1: R2=`010`, RS2: unused=`000`, unused=`000`
- **Binary:** `1010 011 010 000 000`

---

**Address 0x09:** `HALT`  *(label `done` points here)*
- Opcode: `1111`, rest=`000000000000`
- **Binary:** `1111 000000000000`

---

### Final Output Binary

```
Address │ Binary Word
────────┼──────────────────────
  0x00  │ 1011 000 001 010 000
  0x01  │ 1101 000000000100
  0x02  │ 0011 100 001 010 000
  0x03  │ 1110 000000000110
  0x04  │ 1010 011 001 000 000
  0x05  │ 1100 000000001001
  0x06  │ 1010 011 001 000 000
  0x07  │ 1100 000000001001
  0x08  │ 1010 011 010 000 000
  0x09  │ 1111 000000000000
```

This is a complete, executable binary program.

---

## 4. Error Detection in Pass 2

Pass 2 catches errors that couldn't be detected in Pass 1:

| Error | Example | Why caught in Pass 2 |
|-------|---------|----------------------|
| Undefined label | `JMP nowhere` (never defined) | Symbol Table lookup fails |
| Immediate out of range | `ADDI R1, R0, 200` (max is 31 for 6-bit signed) | Value doesn't fit in field |
| Wrong operands for instruction | `HALT R1` (HALT takes no operands) | Operand count mismatch |

---

## 5. The Assembler Output: Object Files

In real systems, the assembler doesn't always produce a raw binary directly. It often produces an **object file** containing:

```
┌─────────────────────────────────┐
│           Object File           │
├─────────────────────────────────┤
│  Header (code size, data size)  │
├─────────────────────────────────┤
│  Code segment (machine code)    │
├─────────────────────────────────┤
│  Data segment (initialized data)│
├─────────────────────────────────┤
│  Symbol table (exported labels) │
├─────────────────────────────────┤
│  Relocation table               │
└─────────────────────────────────┘
```

The **relocation table** lists addresses that need to be adjusted when the program is loaded at a different memory location than assumed. A separate program called a **linker** combines multiple object files and resolves these addresses before execution.

---

## 6. The Full Assembler Pipeline

Putting both passes together — and zooming out to the full tool chain:

```
source.asm
    │
    ▼
┌─────────────────────────────────────────────┐
│                  ASSEMBLER                  │
│                                             │
│  Pass 1: scan → build Symbol Table          │
│  Pass 2: encode → produce object code       │
└─────────────────────────────────────────────┘
    │
    ▼
source.obj  (object file, may have unresolved references)
    │
    ▼
┌───────────┐
│  LINKER   │  ← combines multiple .obj files
└───────────┘
    │
    ▼
program.bin  (fully resolved binary)
    │
    ▼
┌───────────┐
│  LOADER   │  ← OS places binary into RAM
└───────────┘
    │
    ▼
CPU executes via Fetch-Decode-Execute cycle
```

---

## 7. Bringing It All Together

You've now traced a complete path from the lowest level to a running program:

| Module | What you learned |
|--------|-----------------|
| 1 | Boolean Logic — gates are the atoms of computation |
| 2 | Binary Arithmetic — numbers and math in binary |
| 3 | Combinational Logic & ALU — gates form functional units |
| 4 | Sequential Logic — flip-flops give circuits memory |
| 5 | Computer Architecture — components form a complete CPU |
| 6 | Machine Language — CPU instructions in binary |
| 7 | Assembler I — assembly syntax and Pass 1 (Symbol Table) |
| 8 | Assembler II — Pass 2 encoding and the full tool chain |

From a NOR gate to a running program — that's the full stack of a computer system.

---

## Summary

| Concept | Key Idea |
|---------|----------|
| Pass 2 | Encodes each instruction to binary using opcode table + Symbol Table |
| R/I/J encoding | Each type has a fixed field layout; fields are packed into one word |
| Sign extension | Negative immediates are extended to full word width by hardware |
| Undefined label | Caught in Pass 2 when Symbol Table lookup fails |
| Object file | Assembler output with code, data, symbol, and relocation sections |
| Linker | Combines object files and resolves cross-file label references |
| Loader | OS places the final binary into RAM for the CPU to execute |

---

## 🧠 Module Quiz

1. Fully encode the following instruction into a 16-bit binary word:  
   `LOAD R5, R2, 6`  
   Show each field and its binary value.

2. Using the Symbol Table below, encode `JNE retry` into binary:

   | Label | Address |
   |-------|---------|
   | retry | 0x03    |
   | done  | 0x08    |

3. What is **sign extension** and when does it matter? Give an example using a 6-bit immediate of `−3`.

4. An assembler processes `ADDI R1, R0, 50`. The immediate field is 6 bits signed. What is the maximum positive value it can hold? Does 50 fit? What error would the assembler report?

5. What is the difference between an **assembler**, a **linker**, and a **loader**? What does each one take as input and produce as output?

6. Trace the entire two-pass assembly process (Pass 1 + Pass 2) for this short program. Produce the Symbol Table and the final binary output:
   ```asm
           ADDI  R1, R0, 3
   check:  CMP   R1, R0
           JEQ   done
           ADDI  R1, R1, -1
           JUMP  check
   done:   HALT
   ```
