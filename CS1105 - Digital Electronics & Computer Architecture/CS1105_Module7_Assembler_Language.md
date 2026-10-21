# CS 1105 — Module 7: Assembler Language: Concepts and Techniques

---

## 1. What is Assembly Language?

Assembly language is the **lowest-level human-readable programming language** — a thin layer above raw machine code (binary instructions).

```
High-Level (C):     int x = a + b;
Assembly:           ADD R1, R2, R3
Machine Code:       0000 0001 0010 0011
```

Each assembly instruction corresponds **directly** to one machine instruction. The program that converts assembly to machine code is called an **assembler**.

> Assembly is used when you need maximum control over hardware — operating systems, device drivers, embedded systems, and performance-critical routines.

---

## 2. Assembly Language Structure

A typical assembly source file has four columns:

```
[Label:]    [Mnemonic]   [Operands]      [; Comment]

START:      MOV          R1, #10         ; Load 10 into R1
            ADD          R1, R1, R2      ; R1 = R1 + R2
            STR          R1, [R3]        ; Store R1 to address in R3
DONE:       B            DONE            ; Infinite loop (halt)
```

| Column      | Purpose                                              |
|-------------|------------------------------------------------------|
| **Label**   | Optional name for an address — used for jumps/calls  |
| **Mnemonic**| The operation name (MOV, ADD, LDR, STR, B...)       |
| **Operands**| What the instruction operates on                    |
| **Comment** | Human notes — ignored by assembler                  |

---

## 3. Common Instructions (Generic / ARM-style)

### Data Movement

| Instruction         | Meaning                              |
|---------------------|--------------------------------------|
| `MOV R1, #5`        | Load immediate value 5 into R1       |
| `MOV R1, R2`        | Copy R2 into R1                      |
| `LDR R1, [R2]`      | Load from memory address in R2 → R1  |
| `STR R1, [R2]`      | Store R1 to memory address in R2     |

### Arithmetic & Logic

| Instruction         | Meaning                              |
|---------------------|--------------------------------------|
| `ADD R1, R2, R3`    | R1 = R2 + R3                         |
| `SUB R1, R2, R3`    | R1 = R2 − R3                         |
| `MUL R1, R2, R3`    | R1 = R2 × R3                         |
| `AND R1, R2, R3`    | R1 = R2 AND R3 (bitwise)             |
| `ORR R1, R2, R3`    | R1 = R2 OR R3  (bitwise)             |
| `EOR R1, R2, R3`    | R1 = R2 XOR R3 (bitwise)             |
| `CMP R1, R2`        | Set flags based on R1 − R2 (no store)|

### Branching

| Instruction         | Meaning                              |
|---------------------|--------------------------------------|
| `B LABEL`           | Unconditional branch to LABEL        |
| `BEQ LABEL`         | Branch if Zero flag set (equal)      |
| `BNE LABEL`         | Branch if Zero flag clear (not equal)|
| `BGT LABEL`         | Branch if greater than               |
| `BLT LABEL`         | Branch if less than                  |
| `BL LABEL`          | Branch and Link (call a subroutine)  |
| `MOV PC, LR`        | Return from subroutine               |

---

## 4. Addressing Modes in Assembly

Revisiting addressing modes from Module 6 — now with real assembly syntax:

| Mode        | Example              | What it means                        |
|-------------|----------------------|--------------------------------------|
| Immediate   | `MOV R1, #42`        | R1 ← 42                              |
| Register    | `ADD R1, R2, R3`     | R1 ← R2 + R3                         |
| Direct      | `LDR R1, =0x200`     | R1 ← value at address 0x200          |
| Indirect    | `LDR R1, [R2]`       | R1 ← value at address stored in R2   |
| Indexed     | `LDR R1, [R2, #4]`   | R1 ← value at address R2 + 4         |

---

## 5. Assembler Directives

Directives are **instructions to the assembler**, not to the CPU — they control how code is assembled.

| Directive        | Purpose                                      |
|------------------|----------------------------------------------|
| `.data`          | Start of data section                        |
| `.text`          | Start of code section                        |
| `.equ VAL, 10`   | Define constant VAL = 10                     |
| `.byte 0xFF`     | Reserve 1 byte with value 0xFF               |
| `.word 1000`     | Reserve a word (4 bytes) with value 1000     |
| `.space 20`      | Reserve 20 bytes of zeroed memory            |
| `.ascii "Hello"` | Store ASCII string                           |

---

## 6. The Assembly Process

```
Source code (.asm)
       ↓
  [ Assembler ]
       ↓
  Object file (.obj / .o)   ← machine code + symbol table
       ↓
  [ Linker ]                ← combines multiple object files
       ↓
  Executable (.exe / binary)
```

### Two-Pass Assembler

Most assemblers make **two passes** through the source code:

| Pass   | Job                                                      |
|--------|----------------------------------------------------------|
| Pass 1 | Scan all labels, build a **symbol table** with addresses |
| Pass 2 | Translate instructions, replacing labels with addresses  |

This solves the **forward reference problem** — using a label before it's defined.

---

## 7. The Stack

The stack is a region of memory used for **temporary storage**, function calls, and saving registers.

- Operates **LIFO** (Last In, First Out)
- Managed by the **Stack Pointer (SP)**
- **PUSH** decrements SP and writes data
- **POP** reads data and increments SP

```
Before PUSH R1:           After PUSH R1:
  SP → [   ?   ]            SP → [  R1  ]  ← SP moved down
       [   ?   ]                 [   ?  ]
       [   ?   ]                 [   ?  ]
```

### Subroutine Call with Stack

```
; Caller
    BL  MY_FUNC     ; Branch to MY_FUNC, save return address in LR

; Subroutine
MY_FUNC:
    PUSH {LR}       ; Save return address
    ; ... do work ...
    POP  {PC}       ; Restore PC = return to caller
```

---

## 8. Worked Example — Sum of an Array

**Task:** Add up 4 numbers stored in memory starting at address 0x300. Store result in R0.

```
; Memory layout:
; 0x300: 5
; 0x304: 8
; 0x308: 3
; 0x30C: 7

        MOV  R0, #0         ; R0 = accumulator = 0
        MOV  R1, #4         ; R1 = counter = 4
        LDR  R2, =0x300     ; R2 = pointer to array start

LOOP:   CMP  R1, #0         ; Is counter = 0?
        BEQ  DONE           ; If yes, exit loop
        LDR  R3, [R2]       ; Load current element into R3
        ADD  R0, R0, R3     ; accumulator += element
        ADD  R2, R2, #4     ; move pointer to next word
        SUB  R1, R1, #1     ; decrement counter
        B    LOOP           ; repeat

DONE:   ; R0 now holds 5+8+3+7 = 23
```

**Trace:**

| Iteration | R0 | R1 | R3 |
|-----------|----|----|----|
| Start     | 0  | 4  | —  |
| 1         | 5  | 3  | 5  |
| 2         | 13 | 2  | 8  |
| 3         | 16 | 1  | 3  |
| 4         | 23 | 0  | 7  |

---

## 📝 Module 7 Quiz

**Q1.** What is the role of a **label** in assembly language?

<details>
<summary>Show Answer</summary>

A label is a **symbolic name for a memory address**. It allows the programmer to refer to locations in code (for branching) or data (for loading/storing) by name rather than by raw numeric address. The assembler replaces all labels with their actual addresses during Pass 2.
</details>

---

**Q2.** What is the difference between `MOV R1, #10` and `LDR R1, [R2]`?

<details>
<summary>Show Answer</summary>

`MOV R1, #10` uses **immediate addressing** — the value 10 is loaded directly into R1 from the instruction itself. No memory access occurs.

`LDR R1, [R2]` uses **indirect addressing** — R2 contains a memory address, and R1 is loaded with the value stored at that address. A memory read occurs.
</details>

---

**Q3.** Why does a two-pass assembler need two passes?

<details>
<summary>Show Answer</summary>

To solve the **forward reference problem**. If an instruction branches to a label that appears later in the code, the assembler doesn't yet know the label's address on the first read. **Pass 1** builds a complete symbol table of all label addresses. **Pass 2** uses that table to fill in the correct addresses when generating machine code.
</details>

---

**Q4.** Write the assembly instructions to swap the values in R1 and R2 using R3 as a temporary register.

<details>
<summary>Show Answer</summary>

```
MOV R3, R1    ; R3 = R1 (save R1)
MOV R1, R2    ; R1 = R2
MOV R2, R3    ; R2 = old R1
```

After these three instructions, R1 and R2 have swapped values.
</details>

---

**Q5.** What happens to the Stack Pointer (SP) when you PUSH a value, and when you POP one?

<details>
<summary>Show Answer</summary>

**PUSH:** SP is **decremented** (moved to a lower address) and the value is written to the new SP location. The stack grows downward in memory.

**POP:** The value at the current SP address is read into the target register, then SP is **incremented** (moved back up). The stack shrinks.
</details>

---

## 🔑 Key Takeaways

- Assembly is a 1-to-1 mapping to machine instructions — full hardware control, no abstraction.
- Instructions fall into three categories: data movement, arithmetic/logic, and branching.
- Addressing modes define where operand data comes from: immediate, register, direct, indirect, or indexed.
- The two-pass assembler builds a symbol table on Pass 1 and generates machine code on Pass 2.
- The stack enables subroutine calls, local storage, and register preservation — managed by the SP.

---

*Next up → **Module 8: Compiler Processes Overview***
