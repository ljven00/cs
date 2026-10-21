# CS 1104 Computer Systems
## Module 7: Assembler I

---

## Overview

In Module 6, you wrote programs in machine code — tedious, error-prone binary strings. In this module we introduce **assembly language**: a human-readable text representation of machine code where binary opcodes become mnemonics like `ADD`, `LOAD`, and `JEQ`. We also build the first half of an **assembler** — the program that translates assembly into binary. By the end of this module, you'll understand how that translation works and be able to do it by hand.

---

## 1. Assembly Language vs. Machine Code

Assembly language is a **thin layer of abstraction** over machine code. There is (almost) a one-to-one correspondence: one assembly instruction → one machine instruction.

```
Assembly text           Machine code (binary)
─────────────────────   ──────────────────────
ADD  R3, R1, R2    →    0001 011 001 010 000
LOAD R1, R0, 0x20  →    1000 001 000 100000
HALT               →    1111 000000000000
```

Unlike high-level languages (Python, C), assembly doesn't have `for` loops, functions, or types. You are one step above the bare metal.

---

## 2. Anatomy of an Assembly Program

A typical assembly source file has four columns:

```
[label:]   mnemonic   [operands]   [; comment]
```

| Column | Optional? | Purpose |
|--------|-----------|---------|
| **Label** | Yes | A name for this address; used as a jump target |
| **Mnemonic** | No | The instruction name (ADD, LOAD, etc.) |
| **Operands** | Depends | Registers, immediates, or labels |
| **Comment** | Yes | Human notes after a semicolon — ignored by assembler |

### Example

```asm
; Compute R3 = R1 + R2 and store result
        ADDI  R1, R0, 7    ; R1 = 7
        ADDI  R2, R0, 4    ; R2 = 4
        ADD   R3, R1, R2   ; R3 = 7 + 4 = 11
        STORE R3, R0, 0x22 ; Memory[0x22] = 11
        HALT               ; Done
```

---

## 3. What an Assembler Does

An assembler is a program that reads assembly source text and outputs binary machine code. It performs two major tasks:

1. **Translating mnemonics** to opcodes
2. **Resolving labels** to addresses

```
source.asm  ──→  [ Assembler ]  ──→  program.bin
(text)                               (binary)
```

The assembler we're building processes the source in **two passes** — this module covers Pass 1, and Module 8 covers Pass 2.

---

## 4. Pass 1: Building the Symbol Table

The first problem the assembler must solve: **forward references**.

Consider this code:

```asm
        JEQ   done       ; Jump to "done" — but where IS "done"?
        ADD   R3, R1, R2
done:   HALT
```

When the assembler sees `JEQ done` on line 1, it hasn't encountered the label `done` yet. It doesn't know the address to encode. This is a **forward reference**.

**Pass 1's job:** scan the entire source file, find every label, and record its address in a **Symbol Table**.

### How Pass 1 Works

The assembler maintains a **Location Counter (LC)** that tracks the current instruction's address, starting at 0.

```
LC = 0

For each line:
    If line has a label → add (label, LC) to Symbol Table
    If line has an instruction → LC = LC + 1
    (skip blank lines and comment-only lines)
```

### Example: Building the Symbol Table

```asm
; Count down from 3 to 0
        ADDI  R1, R0, 3   ; address 0x00
        ADDI  R2, R0, 1   ; address 0x01
loop:   SUB   R1, R1, R2  ; address 0x02  ← "loop" maps to 0x02
        CMP   R1, R0      ; address 0x03
        JNE   loop        ; address 0x04
        HALT              ; address 0x05
done:                     ; address 0x06  ← "done" maps to 0x06 (no instruction)
```

**Symbol Table after Pass 1:**

| Label | Address |
|-------|---------|
| loop  | 0x02    |
| done  | 0x06    |

Now when Pass 2 encounters `JNE loop`, it can look up `loop` and encode address `0x02` into the instruction.

---

## 5. Handling Assembler Directives

Assembly programs often include **directives** — commands to the assembler itself rather than instructions for the CPU. They control how the output is organized.

| Directive | Meaning |
|-----------|---------|
| `.org 0x10` | Set the Location Counter to 0x10 (place code here) |
| `.data 0x05` | Reserve a word in memory initialized to the value 5 |
| `.word 0` | Reserve a word initialized to 0 |

### Example with Directives

```asm
        .org  0x00         ; Start placing code at address 0x00

        LOAD  R1, R0, num1 ; R1 = Memory[address of num1]
        LOAD  R2, R0, num2 ; R2 = Memory[address of num2]
        ADD   R3, R1, R2
        HALT

        .org  0x10         ; Place data starting at address 0x10
num1:   .data 7            ; Memory[0x10] = 7
num2:   .data 4            ; Memory[0x11] = 4
```

**Symbol Table after Pass 1:**

| Label | Address |
|-------|---------|
| num1  | 0x10    |
| num2  | 0x11    |

Directives are processed during Pass 1 (they affect the Location Counter) but do not generate instructions.

---

## 6. Translating Operands

Pass 1 also validates that operands are syntactically correct. There are three kinds:

### 6.1 Register Operands

Named R0–R7 in our ISA, mapped to 3-bit codes:

| Name | Binary |
|------|--------|
| R0   | 000    |
| R1   | 001    |
| R2   | 010    |
| R3   | 011    |
| R4   | 100    |
| R5   | 101    |
| R6   | 110    |
| R7   | 111    |

### 6.2 Immediate Values

Can be written in decimal (`5`), hex (`0x05`), or binary (`0b0101`). The assembler converts all formats to the same binary field.

```
ADDI R1, R0, 5      → immediate field = 000101
ADDI R1, R0, 0x05   → immediate field = 000101  (same)
```

### 6.3 Labels as Operands

When a label appears as an operand (e.g., `JEQ done`), the assembler looks it up in the Symbol Table and substitutes the address. This happens in **Pass 2**.

---

## 7. A Full Pass 1 Walkthrough

Let's run Pass 1 on a complete program:

```asm
; Find the larger of two numbers in R1 and R2, store in R3
        CMP   R1, R2       ; compare R1 and R2
        JEQ   equal        ; if equal, jump
        SUB   R4, R1, R2   ; R4 = R1 - R2
        JNE   r1bigger     ; if R4 != 0, R1 > R2 (since no borrow)
equal:  MOV   R3, R1       ; they're equal, store either
        JUMP  done
r1bigger: MOV R3, R1       ; R1 is larger
        JUMP  done
        MOV   R3, R2       ; R2 is larger (never reached here — bug!)
done:   HALT
```

**Pass 1 trace:**

| LC   | Line                    | Action                          |
|------|-------------------------|---------------------------------|
| 0x00 | `CMP R1, R2`           | No label; LC → 0x01             |
| 0x01 | `JEQ equal`            | No label; LC → 0x02             |
| 0x02 | `SUB R4, R1, R2`       | No label; LC → 0x03             |
| 0x03 | `JNE r1bigger`         | No label; LC → 0x04             |
| 0x04 | `equal: MOV R3, R1`    | **Add equal=0x04**; LC → 0x05   |
| 0x05 | `JUMP done`            | No label; LC → 0x06             |
| 0x06 | `r1bigger: MOV R3, R1` | **Add r1bigger=0x06**; LC → 0x07|
| 0x07 | `JUMP done`            | No label; LC → 0x08             |
| 0x08 | `MOV R3, R2`           | No label; LC → 0x09             |
| 0x09 | `done: HALT`           | **Add done=0x09**; LC → 0x0A    |

**Final Symbol Table:**

| Label    | Address |
|----------|---------|
| equal    | 0x04    |
| r1bigger | 0x06    |
| done     | 0x09    |

Pass 1 is complete. In Module 8 (Pass 2), we use this table to fully encode every instruction into binary.

---

## 8. Error Detection in Pass 1

Even in Pass 1, the assembler can catch some errors:

| Error Type | Example | Detected? |
|------------|---------|-----------|
| Duplicate label | `loop:` defined twice | ✅ Yes — in Pass 1 |
| Unknown mnemonic | `ADDD R1, R2` | ✅ Yes |
| Wrong operand count | `ADD R1, R2` (missing RS2) | ✅ Yes |
| Undefined label | `JMP nowhere` (nowhere never defined) | ❌ Not until Pass 2 |

---

## Summary

| Concept | Key Idea |
|---------|----------|
| Assembly language | Human-readable mnemonics; 1-to-1 with machine instructions |
| Assembler | Translates assembly text to binary machine code |
| Two-pass assembly | Pass 1 builds symbol table; Pass 2 encodes instructions |
| Location Counter | Tracks current address during assembly |
| Symbol Table | Maps label names to their memory addresses |
| Forward reference | A label used before it is defined — solved by Pass 1 |
| Directives | Assembler commands (`.org`, `.data`) not translated to instructions |

---

## 🧠 Module Quiz

1. What is a **forward reference** and why does it require a two-pass assembler? Give an example.

2. Run Pass 1 on the following program and produce the Symbol Table:
   ```asm
           ADDI  R1, R0, 10
           ADDI  R2, R0, 1
   top:    CMP   R1, R0
           JEQ   end
           SUB   R1, R1, R2
           JUMP  top
   end:    HALT
   ```

3. What is the Location Counter, and what two events cause it to change during Pass 1?

4. Explain the difference between an **assembler directive** and an **instruction**. Does `.data 5` appear in the final machine code binary? What does it produce?

5. In the full walkthrough (Section 7), there is a comment saying `; never reached here — bug!`. Why is the `MOV R3, R2` instruction at address `0x08` unreachable? What would you fix to make the program correct?

6. Why does Pass 1 detect duplicate labels but **not** undefined labels? At what point in the assembly process are undefined labels caught?
