# CS 1105 — Module 1: Introduction to Digital Design

---

## 1. What is Digital Electronics?

Digital electronics deals with systems that use **discrete signals** — typically two voltage levels represented as:

- **0** → LOW (e.g., 0V)
- **1** → HIGH (e.g., 3.3V or 5V)

This is the foundation of all modern computers, smartphones, and embedded systems.

> **Analog vs. Digital:** Analog signals are continuous (infinite values). Digital signals are discrete (only 0 or 1). Digital systems are more reliable and noise-resistant.

---

## 2. Number Systems

Computers don't use decimal (base-10). You need to understand these systems:

| System      | Base | Digits Used       |
|-------------|------|-------------------|
| Binary      | 2    | 0, 1              |
| Octal       | 8    | 0–7               |
| Decimal     | 10   | 0–9               |
| Hexadecimal | 16   | 0–9, A–F          |

### Conversions

**Decimal → Binary**
Repeatedly divide by 2 and record remainders (bottom to top).

```
13 ÷ 2 = 6 R 1
 6 ÷ 2 = 3 R 0
 3 ÷ 2 = 1 R 1
 1 ÷ 2 = 0 R 1
         ↑ read upward
13₁₀ = 1101₂
```

**Binary → Hexadecimal**
Group bits in sets of 4 from right to left.

```
1101 0110₂
= D    6  ₁₆
= D6₁₆
```

---

## 3. Binary Arithmetic

### Addition

```
  1011   (11)
+ 0110   ( 6)
------
 10001   (17)
```

Rules: `0+0=0`, `0+1=1`, `1+1=10` (write 0, carry 1)

### Two's Complement (Representing Negative Numbers)

1. Write the positive number in binary
2. Flip all bits (one's complement)
3. Add 1

```
+5  =  0101
Flip → 1010
Add 1→ 1011  = –5 in two's complement
```

---

## 4. Logic Gates

Logic gates are the **building blocks** of all digital circuits.

| Gate | Symbol | Output |
|------|--------|--------|
| AND  | A · B  | 1 only if both inputs are 1 |
| OR   | A + B  | 1 if at least one input is 1 |
| NOT  | Ā      | Inverts the input |
| NAND | ¬(A·B) | Opposite of AND |
| NOR  | ¬(A+B) | Opposite of OR |
| XOR  | A ⊕ B  | 1 if inputs are **different** |

### Example: AND Gate Truth Table

| A | B | A AND B |
|---|---|---------|
| 0 | 0 |    0    |
| 0 | 1 |    0    |
| 1 | 0 |    0    |
| 1 | 1 |    1    |

---

## 5. Boolean Algebra Basics

Boolean algebra lets us simplify logic expressions.

### Key Laws

| Law            | Expression                        |
|----------------|-----------------------------------|
| Identity       | A + 0 = A, A · 1 = A             |
| Null           | A + 1 = 1, A · 0 = 0             |
| Complement     | A + Ā = 1, A · Ā = 0             |
| Idempotent     | A + A = A, A · A = A             |
| De Morgan's    | ¬(A·B) = Ā+B̄, ¬(A+B) = Ā·B̄  |

### Example: Simplification

```
F = A·B + A·B̄
  = A·(B + B̄)     ← Factor out A
  = A·1            ← Complement law
  = A              ← Identity law
```

---

## 6. Worked Example — Full Circuit

**Problem:** Design a circuit that outputs 1 only when exactly one of two inputs is 1.

**Solution:** This is an **XOR gate**.

Truth table:

| A | B | Output |
|---|---|--------|
| 0 | 0 |   0    |
| 0 | 1 |   1    |
| 1 | 0 |   1    |
| 1 | 1 |   0    |

Boolean expression: `F = A⊕B = A·B̄ + Ā·B`

---

## 📝 Module 1 Quiz

**Q1.** Convert `42₁₀` to binary.

<details>
<summary>Show Answer</summary>

`42 = 101010₂`

```
42 ÷ 2 = 21 R 0
21 ÷ 2 = 10 R 1
10 ÷ 2 =  5 R 0
 5 ÷ 2 =  2 R 1
 2 ÷ 2 =  1 R 0
 1 ÷ 2 =  0 R 1
→ Read upward: 101010
```
</details>

---

**Q2.** What is `–7` in 4-bit two's complement?

<details>
<summary>Show Answer</summary>

```
+7  = 0111
Flip→ 1000
+1 → 1001  = –7
```
</details>

---

**Q3.** Simplify: `F = A·B + Ā·B`

<details>
<summary>Show Answer</summary>

```
F = B·(A + Ā)
  = B·1
  = B
```
</details>

---

**Q4.** Which gate produces output 1 only when **both** inputs are 0?

- A) AND  
- B) OR  
- C) NOR  
- D) NAND  

<details>
<summary>Show Answer</summary>

**C) NOR** — NOR outputs 1 only when all inputs are 0.
</details>

---

**Q5.** Convert `B4₁₆` to binary.

<details>
<summary>Show Answer</summary>

```
B = 1011
4 = 0100
B4₁₆ = 10110100₂
```
</details>

---

## 🔑 Key Takeaways

- Digital systems use binary (0 and 1) to represent all data.
- You must be comfortable converting between binary, decimal, octal, and hex.
- Logic gates are the physical implementation of Boolean operations.
- Boolean algebra helps simplify complex circuits.
- Two's complement is the standard way to represent negative integers.

---

*Next up → **Module 2: Combinational Logic Design***
