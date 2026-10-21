# CS 1104 Computer Systems
## Module 3: Combinational Logic and the ALU

---

## Overview

In Module 1 you learned Boolean gates. In Module 2 you learned binary arithmetic. This module is where they meet: we'll see how gates are wired together into **combinational circuits** — circuits whose output depends purely on their current inputs — and how those circuits combine to form the **Arithmetic Logic Unit (ALU)**, the component inside every CPU that does all computation.

---

## 1. What is Combinational Logic?

A **combinational circuit** has no memory. Given a set of inputs, it always produces the same outputs — instantly, with no history involved.

```
Inputs ──→ [ Combinational Circuit ] ──→ Outputs
```

This is in contrast to **sequential circuits** (Module 4), which can remember past states.

**Examples of combinational circuits:**
- Adders
- Multiplexers
- Decoders
- The ALU itself

---

## 2. Half Adder

The simplest useful circuit: adds two single bits.

| A | B | Sum | Carry |
|---|---|-----|-------|
| 0 | 0 |  0  |   0   |
| 0 | 1 |  1  |   0   |
| 1 | 0 |  1  |   0   |
| 1 | 1 |  0  |   1   |

Notice the pattern?
- **Sum = A XOR B**
- **Carry = A AND B**

So a half adder is just **one XOR gate + one AND gate**. That's it.

```
A ──┬──[XOR]──→ Sum
B ──┘──[AND]──→ Carry
```

**Limitation:** It can't handle a carry coming *in* from a previous bit position.

---

## 3. Full Adder

A **full adder** adds three bits: A, B, and a carry-in (Cin). This is what you need to chain multiple bits together.

| A | B | Cin | Sum | Cout |
|---|---|-----|-----|------|
| 0 | 0 |  0  |  0  |  0   |
| 0 | 0 |  1  |  1  |  0   |
| 0 | 1 |  0  |  1  |  0   |
| 0 | 1 |  1  |  0  |  1   |
| 1 | 0 |  0  |  1  |  0   |
| 1 | 0 |  1  |  0  |  1   |
| 1 | 1 |  0  |  0  |  1   |
| 1 | 1 |  1  |  1  |  1   |

The Boolean expressions:
```
Sum  = A XOR B XOR Cin
Cout = (A AND B) OR (Cin AND (A XOR B))
```

A full adder is built from **two half adders + one OR gate**.

---

## 4. Ripple Carry Adder

To add multi-bit numbers, chain full adders together — the carry-out of each bit feeds into the carry-in of the next.

**4-bit example: adding A₃A₂A₁A₀ + B₃B₂B₁B₀**

```
      A₀,B₀         A₁,B₁         A₂,B₂         A₃,B₃
Cin=0 → [FA] →Cout→ [FA] →Cout→ [FA] →Cout→ [FA] → Cout (overflow)
         ↓            ↓            ↓            ↓
         S₀           S₁           S₂           S₃
```

This is called a **ripple carry adder** because the carry "ripples" from right to left. It works correctly but is slow for large bit widths — the last bit can't be computed until all the carries have propagated. Modern CPUs use a faster design called a **carry-lookahead adder**, but the ripple carry adder is the foundation.

---

## 5. Multiplexer (MUX)

A **multiplexer** selects one of several inputs and forwards it to the output, based on a selector signal. Think of it as a data switch.

A **2-to-1 MUX** has:
- 2 data inputs: A, B
- 1 selector: S
- 1 output: Y

```
Y = (NOT S · A) OR (S · B)
```

| S | Y     |
|---|-------|
| 0 | A     |
| 1 | B     |

**Where it's used:** The ALU uses MUXes to choose which operation's result to output (add, subtract, AND, OR, etc.).

---

## 6. Decoder

A **decoder** takes an n-bit input and activates exactly one of 2ⁿ output lines.

**2-to-4 decoder example:**

| A | B | Y0 | Y1 | Y2 | Y3 |
|---|---|----|----|----|----|
| 0 | 0 |  1 |  0 |  0 |  0 |
| 0 | 1 |  0 |  1 |  0 |  0 |
| 1 | 0 |  0 |  0 |  1 |  0 |
| 1 | 1 |  0 |  0 |  0 |  1 |

**Where it's used:** Inside the ALU's control logic to decode the operation code (opcode) and activate the right functional unit.

---

## 7. The Arithmetic Logic Unit (ALU)

The **ALU** is the computational core of the CPU. It is a combinational circuit that performs:
- **Arithmetic operations:** addition, subtraction
- **Logical operations:** AND, OR, NOT, XOR
- **Comparison operations:** equal, less-than, greater-than (used for branching)

### 7.1 ALU Inputs and Outputs

```
         ┌─────────────┐
    A ──→│             │──→ Result
    B ──→│     ALU     │──→ Zero flag
  Op  ──→│             │──→ Carry flag
         └─────────────┘   ──→ Overflow flag
```

- **A, B:** The two operands (e.g., 8-bit, 16-bit, 32-bit, 64-bit)
- **Op (operation code):** Selects which operation to perform
- **Result:** The computed output
- **Flags:** Status bits that tell the CPU about the result

### 7.2 ALU Operation Table (simplified 3-bit opcode)

| Op Code | Operation    | Result         |
|---------|--------------|----------------|
| 000     | ADD          | A + B          |
| 001     | SUB          | A − B (via Two's Complement) |
| 010     | AND          | A AND B        |
| 011     | OR           | A OR B         |
| 100     | XOR          | A XOR B        |
| 101     | NOT          | NOT A          |
| 110     | SHL          | A shifted left by 1 |
| 111     | SHR          | A shifted right by 1 |

### 7.3 How the ALU Chooses Its Output

The ALU computes **all operations in parallel** and uses a **MUX** to select the correct result based on the opcode. This is more practical in hardware than trying to "turn off" unused circuits.

```
A,B → [Adder  ] → result₀ ──┐
A,B → [AND    ] → result₁ ──┤
A,B → [OR     ] → result₂ ──┤──→ [MUX] ──→ Final Result
A,B → [XOR    ] → result₃ ──┤       ↑
 A  → [NOT    ] → result₄ ──┘      Op
```

### 7.4 Status Flags

After every ALU operation, flags are set automatically:

| Flag         | Meaning                                      |
|--------------|----------------------------------------------|
| **Zero (Z)** | Result is 0 (used for equality checks)       |
| **Carry (C)**| Carry-out from the MSB (unsigned overflow)   |
| **Overflow (V)** | Signed result is out of range            |
| **Negative (N)** | Result's MSB is 1 (result is negative)   |

These flags feed directly into the CPU's control unit to make branching decisions — e.g., "if Zero flag is set, jump to this address."

---

## 8. Subtraction in the ALU

Recall from Module 2: to subtract, the ALU **negates B using Two's Complement, then adds**.

```
A − B  =  A + (NOT B) + 1
```

In hardware this is elegant:
1. Pass B through inverters (NOT gates) → gives NOT B
2. Set Cin of the first full adder to 1 (instead of 0) → adds the +1
3. Use the same adder circuit as for addition

So **the ALU reuses the adder for both ADD and SUB** — only the control signals change.

---

## 9. Putting It All Together

Here's a simplified view of a 4-bit ALU's internals:

```
          ┌──────────────────────────────┐
  A[3:0]──┤                              ├──→ Result[3:0]
  B[3:0]──┤   4× Full Adders (chained)   ├──→ Carry Out
          │   AND/OR/XOR gates (bitwise) │
  Op[2:0]─┤   NOT logic                  ├──→ Zero Flag
          │   Output MUX                 ├──→ Overflow Flag
          └──────────────────────────────┘
```

Every real CPU ALU — from a tiny microcontroller to a 64-core server chip — is a scaled-up, optimized version of exactly this structure.

---

## Summary

| Component      | Function                                      |
|----------------|-----------------------------------------------|
| Half Adder     | Adds 2 bits → Sum + Carry (XOR + AND)         |
| Full Adder     | Adds 3 bits (includes carry-in)               |
| Ripple Carry   | Chains full adders for multi-bit addition     |
| MUX            | Selects one of many signals based on control  |
| Decoder        | Activates one output line from an n-bit code  |
| ALU            | Performs arithmetic and logic using all above |
| Flags          | Z, C, V, N — report status of the result      |

---

## 🧠 Module Quiz

1. A half adder receives inputs A = 1, B = 1. What are the Sum and Carry outputs? Which two gates produce these?

2. Explain why a half adder is not sufficient for adding multi-bit numbers, and how a full adder solves this.

3. Trace through a 4-bit ripple carry adder to compute `0101 + 0111`. Show the carry at each bit position and give the final result in binary and decimal.

4. An ALU has opcode `001` mapped to subtraction. It computes A − B internally as A + (NOT B) + 1. Using 4-bit values, compute `1001 − 0011` this way and verify the result.

5. After an ALU operation, the Zero flag is set and the Negative flag is clear. What can you conclude about the result? Give a concrete example of an operation that would produce this.

6. Why does the ALU compute all operations simultaneously and use a MUX to select the output, rather than only running the selected operation? *(Think about how hardware works vs. software.)*
