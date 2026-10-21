# CS 1105 — Module 2: Combinational Logic Design

---

## 1. What is Combinational Logic?

A **combinational circuit** is one where the output depends **only on the current inputs** — no memory, no history.

> Compare to *sequential logic* (Module 3), where past states matter.

**Examples:** Adders, multiplexers, encoders, decoders.

---

## 2. Boolean Expressions from Truth Tables

Given a truth table, you can extract a Boolean expression using the **Sum of Products (SOP)** method:

1. Look for rows where output = **1**
2. Write a **minterm** (AND expression) for each such row
3. OR all the minterms together

### Example

| A | B | C | F |
|---|---|---|---|
| 0 | 0 | 0 | 0 |
| 0 | 0 | 1 | 1 |
| 0 | 1 | 0 | 0 |
| 0 | 1 | 1 | 1 |
| 1 | 0 | 0 | 0 |
| 1 | 0 | 1 | 0 |
| 1 | 1 | 0 | 1 |
| 1 | 1 | 1 | 1 |

Rows where F = 1: `001, 011, 110, 111`

```
F = Ā·B̄·C + Ā·B·C + A·B·C̄ + A·B·C
```

---

## 3. Karnaugh Maps (K-Maps)

K-Maps are a visual method to **simplify Boolean expressions** — faster and less error-prone than algebra for up to 4 variables.

### Rules
- Arrange cells in **Gray code order** (only 1 bit changes between adjacent cells)
- Group 1s in powers of 2: **1, 2, 4, 8**
- Make groups as **large as possible**
- Groups can **wrap around** edges
- Each group eliminates one variable per doubling in size

### 2-Variable K-Map Layout

```
        B=0   B=1
A=0  |  m0  |  m1  |
A=1  |  m2  |  m3  |
```

### 3-Variable K-Map Example

Using the truth table from above:

```
        C=0   C=1
AB=00 |  0  |  1  |
AB=01 |  0  |  1  |
AB=11 |  1  |  1  |
AB=10 |  0  |  0  |
```

**Group 1:** Rows AB=00 and AB=01, C=1 → `Ā·C` (A cancels, B cancels... wait)

Let's trace it:
- `{001, 011}` → A=0, C=1, B varies → **Ā·C**
- `{011, 111}` → C=1, B=1, A varies → **B·C**

```
F = Ā·C + A·B
```

Simpler than the original 4-term expression!

---

## 4. Common Combinational Circuits

### 4.1 Half Adder

Adds two 1-bit numbers. Produces a **Sum** and a **Carry**.

| A | B | Sum | Carry |
|---|---|-----|-------|
| 0 | 0 |  0  |   0   |
| 0 | 1 |  1  |   0   |
| 1 | 0 |  1  |   0   |
| 1 | 1 |  0  |   1   |

```
Sum   = A ⊕ B
Carry = A · B
```

### 4.2 Full Adder

Adds two bits **plus a carry-in** from a previous stage.

| A | B | Cin | Sum | Cout |
|---|---|-----|-----|------|
| 0 | 0 |  0  |  0  |  0   |
| 0 | 1 |  0  |  1  |  0   |
| 1 | 1 |  0  |  0  |  1   |
| 1 | 1 |  1  |  1  |  1   |

```
Sum  = A ⊕ B ⊕ Cin
Cout = A·B + Cin·(A⊕B)
```

> Chain multiple Full Adders to build an **n-bit ripple carry adder**.

### 4.3 Multiplexer (MUX)

A **data selector** — routes one of several inputs to a single output based on select lines.

A **2-to-1 MUX**:

| S | Output |
|---|--------|
| 0 |   A    |
| 1 |   B    |

```
F = Ā·A_in + S·B_in
  (simplified: F = S ? B : A)
```

A **4-to-1 MUX** has 2 select lines (S1, S0) choosing from inputs I0–I3.

### 4.4 Decoder

Converts a binary code into one of **2ⁿ output lines**.

A **2-to-4 decoder** with inputs A, B:

| A | B | D0 | D1 | D2 | D3 |
|---|---|----|----|----|----|
| 0 | 0 |  1 |  0 |  0 |  0 |
| 0 | 1 |  0 |  1 |  0 |  0 |
| 1 | 0 |  0 |  0 |  1 |  0 |
| 1 | 1 |  0 |  0 |  0 |  1 |

```
D0 = Ā·B̄,  D1 = Ā·B,  D2 = A·B̄,  D3 = A·B
```

---

## 5. Worked Example — Building a Circuit from Scratch

**Problem:** Design a circuit that outputs 1 when the 2-bit input (A, B) represents an **odd number**.

**Step 1 — Truth Table:**

| A | B | Decimal | F |
|---|---|---------|---|
| 0 | 0 |    0    | 0 |
| 0 | 1 |    1    | 1 |
| 1 | 0 |    2    | 0 |
| 1 | 1 |    3    | 1 |

**Step 2 — SOP Expression:**

F = 1 when B = 1 (regardless of A)

```
F = Ā·B + A·B = B·(Ā + A) = B
```

**Step 3 — Circuit:**

Just wire input B directly to output. A single wire — no gates needed!

> This is why simplification matters. Always simplify before building.

---

## 📝 Module 2 Quiz

**Q1.** What is the key difference between combinational and sequential logic?

<details>
<summary>Show Answer</summary>

Combinational logic output depends **only on current inputs**. Sequential logic output depends on current inputs **and past states** (it has memory).
</details>

---

**Q2.** Write the SOP expression for this truth table:

| A | B | F |
|---|---|---|
| 0 | 0 | 1 |
| 0 | 1 | 0 |
| 1 | 0 | 0 |
| 1 | 1 | 1 |

<details>
<summary>Show Answer</summary>

Rows with F=1: `00` and `11`

```
F = Ā·B̄ + A·B
```

This is actually the **XNOR** function.
</details>

---

**Q3.** Group the 1s in this 2-variable K-Map and write the simplified expression:

```
        B=0   B=1
A=0  |   1  |   1  |
A=1  |   1  |   0  |
```

<details>
<summary>Show Answer</summary>

- Group top row `{00, 01}` → **Ā** (A=0, B varies)
- Group left column `{00, 10}` → **B̄** (B=0, A varies)

```
F = Ā + B̄
```

(Also verifiable via De Morgan's: this equals ¬(A·B) = NAND)
</details>

---

**Q4.** A Full Adder has inputs A=1, B=1, Cin=1. What are Sum and Cout?

<details>
<summary>Show Answer</summary>

```
Sum  = 1 ⊕ 1 ⊕ 1 = 1
Cout = 1·1 + 1·(1⊕1) = 1 + 0 = 1
```

Sum = **1**, Cout = **1**
</details>

---

**Q5.** A 4-to-1 MUX has select lines S1=1, S0=0. Which input is routed to the output?

<details>
<summary>Show Answer</summary>

S1S0 = `10` in binary = **2** in decimal → **I2** is selected.
</details>

---

## 🔑 Key Takeaways

- Combinational circuits have no memory — outputs depend only on present inputs.
- SOP (Sum of Products) lets you derive any Boolean expression from a truth table.
- K-Maps simplify expressions visually — always use the largest possible groupings.
- Half Adders and Full Adders are the foundation of all arithmetic circuits.
- MUXes and Decoders are essential data-routing building blocks.

---

*Next up → **Module 3: Sequential Logic Design***
