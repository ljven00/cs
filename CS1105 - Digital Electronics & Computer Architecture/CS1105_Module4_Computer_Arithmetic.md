# CS 1105 — Module 4: Computer Arithmetic

---

## 1. Why Computer Arithmetic?

Computers perform all math using **binary hardware** — adders, subtractors, multipliers built from logic gates. Understanding how numbers are represented and operated on is essential for writing efficient code and designing reliable hardware.

---

## 2. Integer Representation

### Unsigned Integers

All bits represent magnitude. An n-bit number can store values from **0 to 2ⁿ−1**.

```
8-bit example:
11111111₂ = 255₁₀  (max)
00000000₂ =   0₁₀  (min)
```

### Signed Integers — Two's Complement

The standard for representing negative numbers in hardware.

- **MSB (leftmost bit) = sign bit**: 0 → positive, 1 → negative
- Range for n bits: **−2ⁿ⁻¹ to 2ⁿ⁻¹ − 1**

```
8-bit range: −128 to +127
```

**Converting negative decimal to two's complement:**

```
Step 1: Write magnitude in binary
Step 2: Flip all bits (one's complement)
Step 3: Add 1

Example: −19 in 8 bits
+19  = 0001 0011
Flip → 1110 1100
+1  → 1110 1101  = −19 ✓
```

**Verify by converting back:**

```
1110 1101
Flip → 0001 0010
+1  → 0001 0011 = 19 ✓
```

---

## 3. Binary Addition & Subtraction

### Addition

Same rules as decimal, with carries:

```
  0011 1010   ( 58)
+ 0001 1110   ( 30)
-----------
  0101 1000   ( 88) ✓
```

### Subtraction via Two's Complement

Subtraction is just **adding a negative** — no separate subtractor circuit needed!

```
A − B  =  A + (−B)  =  A + (two's complement of B)

Example: 58 − 30

30  = 0001 1110
−30 = 1110 0010  (two's complement)

  0011 1010   ( 58)
+ 1110 0010   (−30)
-----------
  0001 1100   ( 28) ✓
(carry out discarded for same-sign result)
```

---

## 4. Overflow

Overflow occurs when a result is **too large or too small** to fit in the available bits.

### Detecting Overflow (Two's Complement)

Overflow happens when:
- Two **positive** numbers add to a **negative** result
- Two **negative** numbers add to a **positive** result

```
Example (4-bit):
  0110  ( 6)
+ 0011  ( 3)
------
  1001  ← reads as −7, but answer should be 9!
```

9 doesn't fit in 4-bit signed range (−8 to +7) → **OVERFLOW**

> Rule: Overflow occurs when the carry **into** the sign bit ≠ carry **out** of the sign bit.

---

## 5. Binary Multiplication

### Shift-and-Add Method

Multiply each bit of the multiplier by the full multiplicand, shifting left for each position.

```
    1011   (11)
  × 0110   ( 6)
  ------
    0000   ← 1011 × 0 (bit 0)
   1011    ← 1011 × 1 (bit 1, shift left 1)
  1011     ← 1011 × 1 (bit 2, shift left 2)
 0000      ← 1011 × 0 (bit 3, shift left 3)
---------
 1000010   (66) ✓
```

---

## 6. Binary Division

### Restoring Division (Conceptual)

Similar to long division in decimal:

```
1100 ÷ 0011  →  12 ÷ 3

Step 1: Does 0011 go into 0001? No  → bring down next bit
Step 2: Does 0011 go into 0011? Yes → quotient bit = 1, remainder = 0
...
Result: Quotient = 0100 (4), Remainder = 0000 ✓
```

> Hardware implementations use repeated subtraction and shifting, managed by the ALU.

---

## 7. Floating-Point Representation (IEEE 754)

Integers can't represent fractions or very large numbers efficiently. **IEEE 754** is the standard for floating-point.

### Single Precision (32 bits)

```
| 1 bit  |  8 bits   |     23 bits      |
|  Sign  |  Exponent |     Mantissa     |
```

### Formula

```
Value = (−1)^Sign × 1.Mantissa × 2^(Exponent − 127)
```

The **bias of 127** allows the exponent to represent both positive and negative powers of 2.

### Example: Represent 6.75 in IEEE 754

```
Step 1: Convert to binary
  6   = 110
  .75 = .11   (0.75 × 2 = 1.5 → 1, 0.5 × 2 = 1.0 → 1)
  6.75 = 110.11₂

Step 2: Normalize
  110.11 = 1.1011 × 2²

Step 3: Encode
  Sign     = 0 (positive)
  Exponent = 2 + 127 = 129 = 10000001₂
  Mantissa = 1011 0000 0000 0000 0000 000  (drop leading 1)

Result:
  0 | 10000001 | 10110000000000000000000
```

---

## 8. The Arithmetic Logic Unit (ALU)

The ALU is the core computational unit of a CPU. It performs:

- **Arithmetic:** ADD, SUB, MUL, DIV
- **Logical:** AND, OR, NOT, XOR
- **Comparison:** equal, less-than, greater-than (sets flags)

### ALU Flags

| Flag     | Meaning                              |
|----------|--------------------------------------|
| Zero (Z) | Result is 0                          |
| Carry (C)| Unsigned overflow / carry out        |
| Overflow (V) | Signed overflow                  |
| Negative (N) | Result is negative (MSB = 1)     |

These flags drive **conditional branching** in programs.

---

## 9. Worked Example — Full ALU Operation

**Problem:** Using an 8-bit ALU, compute −15 + 22. Check for overflow.

**Step 1 — Represent −15:**
```
+15 = 0000 1111
Flip→ 1111 0000
+1 → 1111 0001  = −15 ✓
```

**Step 2 — Add:**
```
  1111 0001   (−15)
+ 0001 0110   ( 22)
-----------
  0000 0111   (  7) ✓
```

**Step 3 — Check overflow:**
- Carry into sign bit: 1
- Carry out of sign bit: 1
- Equal → **No overflow**

Result = **7**, flags: Z=0, C=1, V=0, N=0

---

## 📝 Module 4 Quiz

**Q1.** What is the 8-bit two's complement representation of −45?

<details>
<summary>Show Answer</summary>

```
+45 = 0010 1101
Flip→ 1101 0010
+1 → 1101 0011  = −45 ✓
```
</details>

---

**Q2.** Add these 8-bit signed numbers: `0110 0000 + 0101 0000`. Is there overflow?

<details>
<summary>Show Answer</summary>

```
  0110 0000  ( 96)
+ 0101 0000  ( 80)
-----------
  1011 0000  (−80 in signed) ← should be 176!
```

Two positives produced a negative → **OVERFLOW detected.**

Carry into sign bit = 0, carry out = 0 — but the sign changed incorrectly.
Actually: carry into MSB = 1, carry out of MSB = 0 → they differ → **OVERFLOW** ✓
</details>

---

**Q3.** Multiply `0101 × 0011` using shift-and-add.

<details>
<summary>Show Answer</summary>

```
    0101  (5)
  × 0011  (3)
  ------
    0101   ← bit 0 = 1
   0101    ← bit 1 = 1, shift left 1
   0000    ← bit 2 = 0
  0000     ← bit 3 = 0
---------
  0001111  = 15 ✓
```
</details>

---

**Q4.** What is the bias used in IEEE 754 single-precision exponent, and why?

<details>
<summary>Show Answer</summary>

The bias is **127**. It's used so the 8-bit exponent field can represent both positive and negative exponents without needing a separate sign bit. The stored exponent = actual exponent + 127. For example, exponent 0 is stored as 127 (0111 1111).
</details>

---

**Q5.** An ALU sets the Zero flag (Z=1) after an operation. What does that tell you?

<details>
<summary>Show Answer</summary>

The result of the operation was **0**. This is commonly used in conditional branches — for example, after subtracting two values, Z=1 means they were **equal**.
</details>

---

## 🔑 Key Takeaways

- Two's complement is the universal standard for signed integer arithmetic in hardware.
- Subtraction is implemented as addition of the two's complement — no extra circuit needed.
- Overflow is detectable by comparing the carry into and out of the sign bit.
- IEEE 754 enables floating-point math using sign, biased exponent, and mantissa fields.
- The ALU performs all arithmetic and logic operations and sets flags used by the CPU.

---

*Next up → **Module 5: Memory and Programmable Logic***
