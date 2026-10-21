# CS 1104 Computer Systems
## Module 2: Binary Arithmetic

---

## Overview

Now that you understand Boolean Logic, it's time to see how computers actually count and do math. Computers don't use the decimal system (base-10) that we use daily — they use **binary (base-2)**, where every number is represented using only 0s and 1s. This module covers how binary numbers work, how to do arithmetic with them, and how negative numbers are handled.

---

## 1. Positional Number Systems

Before diving into binary, let's recall how decimal works — then the jump to binary will feel natural.

In **decimal (base-10)**, each digit position represents a power of 10:

```
  3  4  2
  |  |  └─  2 × 10⁰ =   2
  |  └────  4 × 10¹ =  40
  └───────  3 × 10² = 300
                      ─────
                       342
```

In **binary (base-2)**, each digit (called a **bit**) represents a power of 2:

```
  1  0  1  1
  |  |  |  └─  1 × 2⁰ =  1
  |  |  └────  1 × 2¹ =  2
  |  └───────  0 × 2² =  0
  └─────────   1 × 2³ =  8
                          ──
                          11  (decimal)
```

> **Key insight:** With *n* bits, you can represent 2ⁿ different values (0 through 2ⁿ − 1).

---

## 2. Converting Between Binary and Decimal

### Binary → Decimal

Multiply each bit by its positional power of 2 and sum the results.

**Example:** Convert `1101` to decimal

| Bit | Position | Power of 2 | Value |
|-----|----------|------------|-------|
| 1   | 3        | 2³ = 8     | 8     |
| 1   | 2        | 2² = 4     | 4     |
| 0   | 1        | 2¹ = 2     | 0     |
| 1   | 0        | 2⁰ = 1     | 1     |

**Result:** 8 + 4 + 0 + 1 = **13**

### Decimal → Binary

Repeatedly divide by 2 and record the remainders, then read them bottom-up.

**Example:** Convert 13 to binary

```
13 ÷ 2 = 6  remainder  1  ↑ (LSB)
 6 ÷ 2 = 3  remainder  0  |
 3 ÷ 2 = 1  remainder  1  |
 1 ÷ 2 = 0  remainder  1  ↑ (MSB)

Result: 1101
```

---

## 3. Binary Addition

Binary addition follows the same rules as decimal addition, but with only two digits.

### Rules

| A | B | Sum | Carry |
|---|---|-----|-------|
| 0 | 0 |  0  |   0   |
| 0 | 1 |  1  |   0   |
| 1 | 0 |  1  |   0   |
| 1 | 1 |  0  |   1   |

The carry is exactly what XOR and AND give you — which is why XOR is the heart of an adder circuit!

### Example: Add `0110` + `0101`

```
    0 1 1 0   (6)
  + 0 1 0 1   (5)
  ─────────
  Carry: 0 1 0 0
  ─────────────
    1 0 1 1   (11) ✓
```

Work right to left:
- Col 0: 0 + 1 = 1, carry 0
- Col 1: 1 + 0 = 1, carry 0
- Col 2: 1 + 1 = 0, carry 1
- Col 3: 0 + 0 + 1(carry) = 1

---

## 4. Binary Subtraction

### Method 1: Direct Subtraction

| A | B | Difference | Borrow |
|---|---|------------|--------|
| 0 | 0 |     0      |   0    |
| 0 | 1 |     1      |   1    |
| 1 | 0 |     1      |   0    |
| 1 | 1 |     0      |   0    |

### Method 2: Using Two's Complement (Preferred)

Computers almost never subtract directly. Instead, they **negate and add**. This is covered in Section 6.

---

## 5. Overflow

Overflow occurs when the result of an addition is too large to fit in the available number of bits.

**Example with 4-bit numbers:**

```
  1 1 0 0   (12)
+ 0 1 0 1   ( 5)
─────────
1 0 0 0 1   (17) ← requires 5 bits!
```

The 5th bit is lost if we only have 4-bit storage, giving us `0001` = 1, which is **wrong**. Hardware detects this with a carry-out flag.

---

## 6. Representing Negative Numbers: Two's Complement

How do we store negative numbers using only 0s and 1s? The standard answer is **Two's Complement**.

### The Rule

To negate a number in Two's Complement:
1. **Flip all the bits** (bitwise NOT)
2. **Add 1**

**Example:** Represent −6 in 8 bits

```
Step 1:  6 in binary       →  0000 0110
Step 2:  Flip all bits     →  1111 1001
Step 3:  Add 1             →  1111 1010   ← This is −6
```

### Why It's Clever

With Two's Complement, **subtraction becomes addition**:

```
  7 − 3  =  7 + (−3)

  7  =  0000 0111
 −3  =  1111 1101   (flip 0000 0011, then add 1)
      ───────────
        0000 0100  =  4  ✓
```

The hardware needs **only one circuit** for both addition and subtraction!

### Range for n-bit Two's Complement

| Bits | Min value | Max value |
|------|-----------|-----------|
| 4    | −8        | +7        |
| 8    | −128      | +127      |
| 16   | −32,768   | +32,767   |
| 32   | −2,147,483,648 | +2,147,483,647 |

> **Sign bit:** In Two's Complement, the most significant bit (MSB) is always 1 for negative numbers and 0 for positive numbers.

---

## 7. Hexadecimal: A Convenient Shorthand

Writing long binary strings is tedious and error-prone. **Hexadecimal (base-16)** is commonly used as a shorthand — each hex digit represents exactly 4 bits.

| Decimal | Binary | Hex |
|---------|--------|-----|
| 0       | 0000   | 0   |
| 1       | 0001   | 1   |
| ...     | ...    | ... |
| 9       | 1001   | 9   |
| 10      | 1010   | A   |
| 11      | 1011   | B   |
| 12      | 1100   | C   |
| 13      | 1101   | D   |
| 14      | 1110   | E   |
| 15      | 1111   | F   |

**Example:** Binary `1010 1111` → Hex `0xAF`

You'll see hex constantly when working with memory addresses and machine code in later modules.

---

## Summary

| Concept | Key Idea |
|---------|----------|
| Binary (base-2) | Only 0s and 1s; each bit is a power of 2 |
| Binary addition | Same as decimal, but carry happens at 2 (not 10) |
| Overflow | Result too large for the bit width |
| Two's Complement | Standard way to represent negatives; flip bits + add 1 |
| Subtraction | Done by adding the Two's Complement negative |
| Hexadecimal | Shorthand for binary; 1 hex digit = 4 bits |

---

## 🧠 Module Quiz

1. Convert the binary number `10110` to decimal.

2. Convert the decimal number `45` to binary using repeated division.

3. Perform the following binary addition and verify by converting to decimal:  
   `0111 + 0110`

4. Using 8-bit Two's Complement, represent the number **−19**.  
   *(Hint: start with 19 in binary, then apply the two steps.)*

5. A 4-bit register holds the value `1111`. Is this number positive or negative in Two's Complement? What is its decimal value?

6. Why do computers prefer Two's Complement over simply storing a sign bit alongside a magnitude?  
   *(Think about what it means for the hardware.)*
