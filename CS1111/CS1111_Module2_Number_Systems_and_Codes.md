# CS 1111 – Introduction to Computer Science
## Module 2: Number Systems and Codes

---

## Table of Contents
1. [Why Number Systems Matter in Computing](#1-why-number-systems-matter-in-computing)
2. [Positional Number Systems](#2-positional-number-systems)
3. [The Decimal System (Base-10)](#3-the-decimal-system-base-10)
4. [The Binary System (Base-2)](#4-the-binary-system-base-2)
5. [The Octal System (Base-8)](#5-the-octal-system-base-8)
6. [The Hexadecimal System (Base-16)](#6-the-hexadecimal-system-base-16)
7. [Number Base Conversions](#7-number-base-conversions)
8. [Binary Arithmetic](#8-binary-arithmetic)
9. [Character Encoding: Representing Text](#9-character-encoding-representing-text)
10. [Other Codes in Computing](#10-other-codes-in-computing)
11. [Module Summary](#11-module-summary)
12. [Module Quiz](#12-module-quiz)

---

## 1. Why Number Systems Matter in Computing

Computers are electronic machines built from billions of tiny switches called **transistors**. Each switch has only two states: **ON** or **OFF**. This is why computers use the **binary number system** (base-2) — it maps perfectly to these two electrical states:

| Electrical State | Binary Digit |
|---|---|
| OFF (low voltage) | 0 |
| ON (high voltage) | 1 |

Every piece of data a computer handles — text, images, audio, video, instructions — is ultimately represented as a sequence of **0s and 1s**.

However, binary numbers get very long very quickly. To make them more manageable for humans to read and write, we also use **octal (base-8)** and **hexadecimal (base-16)** systems. Understanding all of these number systems is fundamental to understanding how computers work at their core.

---

## 2. Positional Number Systems

All the number systems we study here are **positional** (also called **place-value**) systems. This means the value of a digit depends on:

1. The digit itself.
2. Its **position** in the number.

Each position represents a **power of the base**.

> **General Rule:**
> In a number system with base **b**, the digit in position **n** (counting from 0 at the right) has a place value of **b^n**.

For example, in a base-**b** number `d₃d₂d₁d₀`:

```
Value = (d₃ × b³) + (d₂ × b²) + (d₁ × b¹) + (d₀ × b⁰)
```

This rule works for every number system — decimal, binary, octal, and hexadecimal.

---

## 3. The Decimal System (Base-10)

This is the number system humans use every day. It has **10 digits**: `0, 1, 2, 3, 4, 5, 6, 7, 8, 9`.

The base is **10**, so each position is a power of 10.

### Example: Breaking down 3,472

```
3,472 = (3 × 10³) + (4 × 10²) + (7 × 10¹) + (2 × 10⁰)
      = (3 × 1000) + (4 × 100) + (7 × 10) + (2 × 1)
      = 3000 + 400 + 70 + 2
      = 3,472 ✓
```

| Position | 3 | 4 | 7 | 2 |
|---|---|---|---|---|
| Place Value | 10³ = 1000 | 10² = 100 | 10¹ = 10 | 10⁰ = 1 |

This is the foundation. We'll use the same logic for all other bases.

---

## 4. The Binary System (Base-2)

The binary system has only **2 digits**: `0` and `1`. These are called **bits** (binary digits).

The base is **2**, so each position is a power of 2.

### Place Values in Binary

| Position | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|---|---|---|---|---|---|---|---|---|
| Place Value | 2⁷=128 | 2⁶=64 | 2⁵=32 | 2⁴=16 | 2³=8 | 2²=4 | 2¹=2 | 2⁰=1 |

### Example: Breaking down binary `1011`

```
1011₂ = (1 × 2³) + (0 × 2²) + (1 × 2¹) + (1 × 2⁰)
       = (1 × 8) + (0 × 4) + (1 × 2) + (1 × 1)
       = 8 + 0 + 2 + 1
       = 11₁₀
```

So `1011` in binary equals `11` in decimal.

> **Notation Tip:** We use a subscript to indicate the base. `1011₂` is binary, `11₁₀` is decimal.

### Important Binary Terms

| Term | Meaning |
|---|---|
| **Bit** | A single binary digit (0 or 1) — the smallest unit of data |
| **Nibble** | 4 bits (e.g., `1011`) |
| **Byte** | 8 bits (e.g., `10110110`) |
| **Kilobyte (KB)** | 1,024 bytes (2¹⁰) |
| **Megabyte (MB)** | 1,024 KB = 1,048,576 bytes |
| **Gigabyte (GB)** | 1,024 MB |
| **Terabyte (TB)** | 1,024 GB |

> **Why 1,024 and not 1,000?** Because computers work in powers of 2, and 2¹⁰ = 1,024 is the closest power of 2 to 1,000.

---

## 5. The Octal System (Base-8)

The octal system has **8 digits**: `0, 1, 2, 3, 4, 5, 6, 7`.

The base is **8**, so each position is a power of 8.

Octal is a convenient shorthand for binary because **one octal digit always represents exactly 3 binary bits** (since 2³ = 8).

### Example: Breaking down octal `347`

```
347₈ = (3 × 8²) + (4 × 8¹) + (7 × 8⁰)
      = (3 × 64) + (4 × 8) + (7 × 1)
      = 192 + 32 + 7
      = 231₁₀
```

### Octal ↔ Binary Shortcut

Group binary digits in sets of **3** from the right and convert each group:

| Octal | Binary |
|---|---|
| 0 | 000 |
| 1 | 001 |
| 2 | 010 |
| 3 | 011 |
| 4 | 100 |
| 5 | 101 |
| 6 | 110 |
| 7 | 111 |

**Example:** Convert binary `110 101 011` to octal:
```
110 → 6
101 → 5
011 → 3

Result: 653₈
```

> **Where is octal used?** Unix/Linux file permission codes use octal. For example, the permission `chmod 755` uses octal digits 7, 5, and 5 to set read/write/execute permissions.

---

## 6. The Hexadecimal System (Base-16)

The hexadecimal (hex) system has **16 digits**. Since we only have 10 numeric digits (0–9), the letters A–F represent values 10–15:

| Hex | Decimal | Binary |
|---|---|---|
| 0 | 0 | 0000 |
| 1 | 1 | 0001 |
| 2 | 2 | 0010 |
| 3 | 3 | 0011 |
| 4 | 4 | 0100 |
| 5 | 5 | 0101 |
| 6 | 6 | 0110 |
| 7 | 7 | 0111 |
| 8 | 8 | 1000 |
| 9 | 9 | 1001 |
| A | 10 | 1010 |
| B | 11 | 1011 |
| C | 12 | 1100 |
| D | 13 | 1101 |
| E | 14 | 1110 |
| F | 15 | 1111 |

The base is **16**, so each position is a power of 16. One hex digit represents exactly **4 binary bits** (since 2⁴ = 16) — making hex an extremely compact way to represent binary.

### Example: Breaking down hex `2F3`

```
2F3₁₆ = (2 × 16²) + (F × 16¹) + (3 × 16⁰)
        = (2 × 256) + (15 × 16) + (3 × 1)
        = 512 + 240 + 3
        = 755₁₀
```

### Hex ↔ Binary Shortcut

Group binary digits in sets of **4** from the right:

**Example:** Convert binary `1010 1111 0011` to hex:
```
1010 → A
1111 → F
0011 → 3

Result: AF3₁₆
```

> **Where is hex used?**
> - **Memory addresses:** `0x7FFF3A00` (the `0x` prefix denotes hexadecimal)
> - **Colors in web design:** `#FF5733` (red=FF, green=57, blue=33)
> - **MAC addresses:** `00:1A:2B:3C:4D:5E`
> - **Error codes:** `0x0000007B` (Windows blue screen codes)

---

## 7. Number Base Conversions

This is one of the most important practical skills in this module. We'll cover the most common conversions systematically.

---

### 7.1 Decimal → Binary (Repeated Division by 2)

**Method:** Divide the decimal number by 2 repeatedly. Record the **remainders** from bottom to top.

**Example:** Convert `45₁₀` to binary

```
45 ÷ 2 = 22  remainder 1  ← LSB (Least Significant Bit)
22 ÷ 2 = 11  remainder 0
11 ÷ 2 =  5  remainder 1
 5 ÷ 2 =  2  remainder 1
 2 ÷ 2 =  1  remainder 0
 1 ÷ 2 =  0  remainder 1  ← MSB (Most Significant Bit)

Read remainders bottom to top: 101101₂
```

**Verification:** `101101₂ = 32 + 8 + 4 + 1 = 45₁₀` ✓

---

### 7.2 Binary → Decimal (Sum of Place Values)

**Method:** Multiply each bit by its place value (power of 2) and add them up.

**Example:** Convert `110110₂` to decimal

```
Position:  5    4    3    2    1    0
Bit:       1    1    0    1    1    0
Value:    32   16    0    4    2    0

Sum = 32 + 16 + 0 + 4 + 2 + 0 = 54₁₀
```

---

### 7.3 Decimal → Hexadecimal (Repeated Division by 16)

**Method:** Same as decimal to binary, but divide by 16. Remember to use A–F for remainders 10–15.

**Example:** Convert `475₁₀` to hex

```
475 ÷ 16 = 29  remainder 11 → B  ← LSD (Least Significant Digit)
 29 ÷ 16 =  1  remainder 13 → D
  1 ÷ 16 =  0  remainder  1 → 1  ← MSD (Most Significant Digit)

Read remainders bottom to top: 1DB₁₆
```

**Verification:** `1DB₁₆ = (1×256) + (13×16) + (11×1) = 256 + 208 + 11 = 475₁₀` ✓

---

### 7.4 Hexadecimal → Binary (Direct Substitution)

Replace each hex digit with its **4-bit binary equivalent**.

**Example:** Convert `A7C₁₆` to binary

```
A → 1010
7 → 0111
C → 1100

Result: 1010 0111 1100₂
```

---

### 7.5 Binary → Hexadecimal (Group by 4)

Group binary digits in sets of **4 from the right** (pad with leading zeros if needed), then convert each group.

**Example:** Convert `10111101₂` to hex

```
1011 → B
1101 → D

Result: BD₁₆
```

---

### 7.6 Conversion Summary Table

| From \ To | Decimal | Binary | Octal | Hex |
|---|---|---|---|---|
| **Decimal** | — | Divide by 2 repeatedly | Divide by 8 repeatedly | Divide by 16 repeatedly |
| **Binary** | Sum place values | — | Group by 3 bits | Group by 4 bits |
| **Octal** | Sum place values (×8ⁿ) | Expand each digit to 3 bits | — | Convert via binary |
| **Hex** | Sum place values (×16ⁿ) | Expand each digit to 4 bits | Convert via binary | — |

---

### 7.7 Quick Reference: Decimal 0–16 in All Bases

| Decimal | Binary | Octal | Hexadecimal |
|---|---|---|---|
| 0 | 0000 | 0 | 0 |
| 1 | 0001 | 1 | 1 |
| 2 | 0010 | 2 | 2 |
| 3 | 0011 | 3 | 3 |
| 4 | 0100 | 4 | 4 |
| 5 | 0101 | 5 | 5 |
| 6 | 0110 | 6 | 6 |
| 7 | 0111 | 7 | 7 |
| 8 | 1000 | 10 | 8 |
| 9 | 1001 | 11 | 9 |
| 10 | 1010 | 12 | A |
| 11 | 1011 | 13 | B |
| 12 | 1100 | 14 | C |
| 13 | 1101 | 15 | D |
| 14 | 1110 | 16 | E |
| 15 | 1111 | 17 | F |
| 16 | 10000 | 20 | 10 |

---

## 8. Binary Arithmetic

Computers perform all arithmetic in binary. The rules are simple but powerful.

### 8.1 Binary Addition

There are only four rules:

| Rule | Result | Carry |
|---|---|---|
| 0 + 0 | 0 | 0 |
| 0 + 1 | 1 | 0 |
| 1 + 0 | 1 | 0 |
| 1 + 1 | 0 | 1 (carry) |
| 1 + 1 + 1 | 1 | 1 (carry) |

**Example:** Add `1011₂ + 0110₂`

```
    1011   (= 11₁₀)
  + 0110   (=  6₁₀)
  ------
   10001   (= 17₁₀)
```

Step-by-step (right to left):
- Column 0: 1 + 0 = 1
- Column 1: 1 + 1 = 0, carry 1
- Column 2: 0 + 1 + 1(carry) = 0, carry 1
- Column 3: 1 + 0 + 1(carry) = 0, carry 1
- Column 4: carry 1 → write 1

Result: `10001₂` = 17₁₀ ✓

---

### 8.2 Binary Subtraction

Four rules (borrowing works like in decimal):

| Rule | Result |
|---|---|
| 0 - 0 | 0 |
| 1 - 0 | 1 |
| 1 - 1 | 0 |
| 0 - 1 | 1 (borrow 1 from next column) |

**Example:** Subtract `1010₂ - 0011₂`

```
    1010   (= 10₁₀)
  - 0011   (=  3₁₀)
  ------
    0111   (=  7₁₀)
```

Result: `0111₂` = 7₁₀ ✓

---

### 8.3 Two's Complement (Representing Negative Numbers)

Computers represent negative numbers using **Two's Complement**, because it makes subtraction as simple as addition.

**How to find the Two's Complement of a number:**

1. Write the number in binary.
2. **Invert all bits** (0 → 1, 1 → 0). This is called the **One's Complement**.
3. **Add 1** to the result.

**Example:** Represent `-5` in 8-bit two's complement

```
Step 1: 5 in binary (8-bit) →  00000101
Step 2: Invert all bits      →  11111010  (One's Complement)
Step 3: Add 1                →  11111011  (Two's Complement of -5)
```

So `11111011` in two's complement represents `-5`.

**How to recognize a negative number:** In two's complement, if the **leftmost bit (MSB) is 1**, the number is negative.

> **Why is this useful?** To compute `8 - 5`, a computer simply computes `8 + (-5)`:
> ```
>   00001000  (8)
> + 11111011  (-5 in two's complement)
> ----------
>   00000011  (3) ← the overflow bit is discarded
> ```
> Result: 3 ✓ — No special subtraction hardware needed!

---

## 9. Character Encoding: Representing Text

Numbers are straightforward to represent in binary. But how does a computer store text like the letter `A` or the symbol `@`?

The answer is **character encoding** — a standard that assigns a unique number to every character, and that number is then stored in binary.

---

### 9.1 ASCII (American Standard Code for Information Interchange)

**ASCII** (1963) was the first widely adopted character encoding. It uses **7 bits**, allowing for **128 characters** (2⁷ = 128): uppercase and lowercase letters, digits 0–9, punctuation, and control characters.

#### Selected ASCII Values

| Character | Decimal | Binary |
|---|---|---|
| `A` | 65 | 0100 0001 |
| `B` | 66 | 0100 0010 |
| `Z` | 90 | 0101 1010 |
| `a` | 97 | 0110 0001 |
| `z` | 122 | 0111 1010 |
| `0` | 48 | 0011 0000 |
| `9` | 57 | 0011 1001 |
| Space | 32 | 0010 0000 |
| `!` | 33 | 0010 0001 |

> **Useful Pattern:** The difference between an uppercase letter and its lowercase version is always **32** (or binary `0010 0000`). So `A`=65 and `a`=97 (65+32=97). This is why toggling bit 5 switches case!

**Example:** The word `Hi` in ASCII binary:
```
H = 72  → 0100 1000
i = 105 → 0110 1001

"Hi" = 0100 1000  0110 1001
```

#### Limitations of ASCII
ASCII only covers **English** characters. It cannot represent accented characters (é, ñ, ü), Arabic, Chinese, Japanese, or emoji.

---

### 9.2 Extended ASCII

An 8-bit extension of ASCII that adds 128 more characters (values 128–255) to include some Western European characters. However, different systems used different extensions, causing compatibility problems.

---

### 9.3 Unicode

**Unicode** is a universal character encoding standard designed to represent **every character from every language** in the world — plus symbols, emoji, and more.

Unicode assigns each character a unique **code point**, written as `U+` followed by a hex number.
- `U+0041` = `A`
- `U+00F1` = `ñ`
- `U+4E2D` = `中` (Chinese character for "middle/China")
- `U+1F600` = 😀

#### UTF-8: The Most Common Encoding

**UTF-8** is the most widely used implementation of Unicode. Key properties:
- **Variable-length:** Characters use 1 to 4 bytes depending on the character.
- **Backward compatible with ASCII:** The first 128 characters (0–127) are encoded identically to ASCII.
- Used by over **98% of websites**.

| Character Range | Bytes Used |
|---|---|
| ASCII characters (0–127) | 1 byte |
| Western European characters | 2 bytes |
| Most other languages | 3 bytes |
| Rare characters, emoji | 4 bytes |

> **Example:** The word `café` in UTF-8:
> - `c` → 1 byte (ASCII)
> - `a` → 1 byte (ASCII)
> - `f` → 1 byte (ASCII)
> - `é` → 2 bytes (non-ASCII)
> - Total: 5 bytes

---

## 10. Other Codes in Computing

### 10.1 BCD – Binary Coded Decimal

**BCD** encodes each decimal digit separately using **4 binary bits**.

| Decimal Digit | BCD |
|---|---|
| 0 | 0000 |
| 1 | 0001 |
| 2 | 0010 |
| ... | ... |
| 9 | 1001 |

**Example:** Represent `39` in BCD:
```
3 → 0011
9 → 1001
BCD: 0011 1001
```

Note: BCD is **not** the same as converting 39 directly to binary (`100111`). Each digit is encoded separately.

> **Where is BCD used?** Digital clocks, calculators, and cash registers — where decimal precision matters and direct binary conversion could introduce rounding errors in financial calculations.

---

### 10.2 Gray Code

**Gray code** is a binary sequence where **only one bit changes** between consecutive values. This is important in hardware applications (like rotary encoders) to prevent errors when transitioning between states.

| Decimal | Binary | Gray Code |
|---|---|---|
| 0 | 000 | 000 |
| 1 | 001 | 001 |
| 2 | 010 | 011 |
| 3 | 011 | 010 |
| 4 | 100 | 110 |
| 5 | 101 | 111 |
| 6 | 110 | 101 |
| 7 | 111 | 100 |

> Notice in regular binary, going from 3 (`011`) to 4 (`100`) changes **all 3 bits** at once — this can cause brief erroneous readings in hardware. Gray code going from 3 (`010`) to 4 (`110`) changes only **1 bit**, eliminating this problem.

---

### 10.3 Parity Bits (Error Detection)

When data is transmitted or stored, bits can occasionally flip due to electrical interference. A **parity bit** is an extra bit added to a binary sequence to detect such errors.

- **Even parity:** The parity bit is set so the total number of `1`s (including the parity bit) is **even**.
- **Odd parity:** The parity bit is set so the total number of `1`s is **odd**.

**Example (Even Parity):** Transmit `1011001`

```
Count of 1s in 1011001 = 4 (already even)
Parity bit = 0

Transmitted: 1011001 0
```

**Example (Odd Parity):** Transmit `1011001`

```
Count of 1s = 4 (even, so we need to make it odd)
Parity bit = 1

Transmitted: 1011001 1
```

If the receiver counts the `1`s and the parity doesn't match, it knows an error occurred and can request the data to be resent.

---

## 11. Module Summary

Key takeaways from Module 2:

- Computers use **binary (base-2)** because hardware transistors have two states: ON (1) and OFF (0).
- All number systems we covered are **positional** — digit value depends on its position (power of the base).
- **Decimal (base-10)** uses digits 0–9; **Binary (base-2)** uses 0–1; **Octal (base-8)** uses 0–7; **Hexadecimal (base-16)** uses 0–9 and A–F.
- One **octal digit = 3 bits**; one **hex digit = 4 bits** — making them compact representations of binary.
- To convert **decimal to binary/octal/hex**, use repeated division by the target base and read remainders bottom to top.
- To convert **binary to decimal**, multiply each bit by its place value (power of 2) and sum.
- **Binary addition** follows simple rules with carrying; **Two's Complement** is how computers represent and work with negative numbers.
- **ASCII** encodes 128 characters using 7 bits. **Unicode/UTF-8** encodes every character in every language, using 1–4 bytes per character.
- **BCD** encodes each decimal digit as 4 bits separately. **Gray code** changes only one bit between values. **Parity bits** detect transmission errors.

---

## 12. Module Quiz

Test your understanding of Module 2.

---

**Question 1**
What is the decimal value of the binary number `10110₂`?

- A) 16
- B) 20
- C) 22
- D) 24

---

**Question 2**
Convert the decimal number `58₁₀` to binary using repeated division.

- A) `111010₂`
- B) `110100₂`
- C) `101110₂`
- D) `111100₂`

---

**Question 3**
What is the hexadecimal equivalent of binary `1101 0110₂`?

- A) `C6₁₆`
- B) `D6₁₆`
- C) `E6₁₆`
- D) `B6₁₆`

---

**Question 4**
Which of the following correctly states the relationship between one hexadecimal digit and binary?

- A) One hex digit represents 3 binary bits
- B) One hex digit represents 4 binary bits
- C) One hex digit represents 8 binary bits
- D) One hex digit represents 2 binary bits

---

**Question 5**
What is the Two's Complement representation of `-12` in 8 bits?

- A) `11110011`
- B) `11110100`
- C) `10001100`
- D) `11001100`

---

**Question 6**
In ASCII, the letter `M` has a decimal value of 77. What is the ASCII decimal value of the lowercase letter `m`?

- A) 77
- B) 99
- C) 109
- D) 113

---

**Question 7**
A web developer sets the background color of a website to `#1A2BFF`. What number system is being used here, and what are the decimal values of the red, green, and blue components?

- A) Octal; Red=1, Green=2, Blue=255
- B) Hexadecimal; Red=26, Green=43, Blue=255
- C) Binary; Red=1, Green=1, Blue=255
- D) Hexadecimal; Red=1, Green=2, Blue=255

---

**Question 8**
Add the following binary numbers: `10101₂ + 01110₂`

- A) `100001₂`
- B) `100011₂`
- C) `11111₂`
- D) `100111₂`

---

**Question 9**
What is the BCD representation of the decimal number `74`?

- A) `0111 0100`
- B) `0110 0100`
- C) `1001 0100`
- D) `0111 1000`

---

**Question 10** *(Short Answer)*
Explain why **UTF-8** is considered superior to ASCII for modern computing, and give two real-world examples where ASCII would be insufficient.

---

### Answer Key

| Question | Answer | Explanation |
|---|---|---|
| 1 | **C** | `10110₂ = 16 + 4 + 2 = 22₁₀` (positions 4, 2, 1 are set) |
| 2 | **A** | 58÷2=29 r**0**, 29÷2=14 r**1**, 14÷2=7 r**0**, 7÷2=3 r**1**, 3÷2=1 r**1**, 1÷2=0 r**1** → read bottom to top: `111010` |
| 3 | **B** | `1101` = D, `0110` = 6 → `D6₁₆` |
| 4 | **B** | One hex digit represents 4 binary bits (since 2⁴ = 16). |
| 5 | **B** | 12 = `00001100`; invert → `11110011`; add 1 → `11110100` |
| 6 | **C** | `m` = `M` + 32 = 77 + 32 = 109. (Lowercase letters are always 32 more than uppercase in ASCII.) |
| 7 | **B** | `#1A2BFF` is hex. `1A₁₆` = 26, `2B₁₆` = 43, `FF₁₆` = 255. |
| 8 | **B** | `10101`(21) + `01110`(14) = 35 = `100011₂`. Verify: 32+2+1=35 ✓ |
| 9 | **A** | BCD encodes each digit separately: 7 → `0111`, 4 → `0100` → `0111 0100` |
| 10 | *(See below)* | |

**Question 10 – Sample Answer:**
> ASCII is limited to 128 characters, all of which are based on the English alphabet and common punctuation. This makes it insufficient for representing any language other than English and unable to encode modern symbols like emoji. **UTF-8**, as an implementation of Unicode, can represent over 1.1 million characters covering virtually every language and symbol system in use.
>
> Two real-world examples where ASCII falls short:
> 1. **International websites** — A website displaying Spanish content with characters like `ñ`, `á`, or `ü` cannot use ASCII, as these characters don't exist in the ASCII table. UTF-8 handles them natively.
> 2. **Social media and messaging apps** — Emoji such as 😂 or 🎉 are part of Unicode (e.g., `U+1F602`, `U+1F389`) and cannot be represented in ASCII at all. Any platform that supports emoji must use Unicode encoding.

---

*End of Module 2 — Up next: **Module 3: Boolean Algebra and Logic Gates***
