# CS 1111 – Introduction to Computer Science
## Module 3: Boolean Algebra and Logic Gates

---

## Table of Contents
1. [Introduction: The Language of Logic](#1-introduction-the-language-of-logic)
2. [Boolean Algebra Fundamentals](#2-boolean-algebra-fundamentals)
3. [Basic Boolean Operations](#3-basic-boolean-operations)
4. [Boolean Laws and Theorems](#4-boolean-laws-and-theorems)
5. [Boolean Expressions and Simplification](#5-boolean-expressions-and-simplification)
6. [Logic Gates](#6-logic-gates)
7. [Combining Gates: Logic Circuits](#7-combining-gates-logic-circuits)
8. [Universal Gates: NAND and NOR](#8-universal-gates-nand-and-nor)
9. [Real-World Applications](#9-real-world-applications)
10. [Module Summary](#10-module-summary)
11. [Module Quiz](#11-module-quiz)

---

## 1. Introduction: The Language of Logic

In Module 2, we learned that computers represent everything using binary — 0s and 1s. But how does a computer actually *make decisions* using those 0s and 1s? The answer lies in **Boolean algebra** and **logic gates**.

**Boolean algebra**, developed by mathematician George Boole in 1854, is a branch of mathematics that works with only two values: **TRUE** and **FALSE** (represented as **1** and **0**). It provides a formal system for reasoning about logical conditions — exactly the kind of reasoning a computer needs to do.

**Logic gates** are the physical hardware implementation of Boolean operations. They are tiny electronic circuits (built from transistors) that take binary inputs and produce a binary output based on a logical rule. An average modern CPU contains **billions** of logic gates.

Together, Boolean algebra and logic gates form the mathematical and physical foundation of all digital computing.

> **Real-world analogy:** Think of a smart door lock that opens only if **both** a correct PIN is entered **and** a valid fingerprint is detected. The door opening is the output; the PIN and fingerprint are inputs; "both must be true" is the Boolean logic.

---

## 2. Boolean Algebra Fundamentals

### 2.1 Boolean Variables

A **Boolean variable** can hold only one of two values: **0 (FALSE)** or **1 (TRUE)**.

Variables are usually represented by letters: `A`, `B`, `C`, `X`, `Y`.

- `A = 1` means "condition A is true"
- `B = 0` means "condition B is false"

### 2.2 Boolean Functions

A **Boolean function** takes one or more Boolean variables as inputs and produces a single Boolean output.

**Example:**
```
F(A, B) = A AND B
```
This function outputs 1 only when both A and B are 1.

### 2.3 Truth Tables

A **truth table** is a systematic way to list all possible input combinations for a Boolean function and their corresponding outputs.

For **n** input variables, there are **2ⁿ** possible input combinations.

| Inputs | Output |
|---|---|
| n = 1 variable | 2¹ = 2 rows |
| n = 2 variables | 2² = 4 rows |
| n = 3 variables | 2³ = 8 rows |
| n = 4 variables | 2⁴ = 16 rows |

---

## 3. Basic Boolean Operations

There are three fundamental Boolean operations: **AND**, **OR**, and **NOT**. All other operations are derived from these.

---

### 3.1 The AND Operation (Conjunction)

**Symbol:** `·` or `∧` or simply written as `AB`

**Rule:** Output is **1** only when **ALL** inputs are 1.

> **Plain English:** "Both A AND B must be true."

**Truth Table:**

| A | B | A AND B (A · B) |
|---|---|---|
| 0 | 0 | 0 |
| 0 | 1 | 0 |
| 1 | 0 | 0 |
| 1 | 1 | **1** |

**Real-world example:** A car starts only if the key is in the ignition (`A=1`) **AND** the brake pedal is pressed (`B=1`).

---

### 3.2 The OR Operation (Disjunction)

**Symbol:** `+` or `∨`

**Rule:** Output is **1** when **AT LEAST ONE** input is 1.

> **Plain English:** "Either A OR B (or both) must be true."

**Truth Table:**

| A | B | A OR B (A + B) |
|---|---|---|
| 0 | 0 | 0 |
| 0 | 1 | **1** |
| 1 | 0 | **1** |
| 1 | 1 | **1** |

**Real-world example:** A building alarm triggers if motion is detected (`A=1`) **OR** a door is opened (`B=1`).

---

### 3.3 The NOT Operation (Complement/Inversion)

**Symbol:** `'` or `¬` or a bar over the variable: `Ā`

**Rule:** Output is the **opposite** of the input.

> **Plain English:** "If A is true, NOT A is false, and vice versa."

**Truth Table:**

| A | NOT A (A') |
|---|---|
| 0 | **1** |
| 1 | **0** |

**Real-world example:** A night light turns ON when it is NOT daytime.

---

### 3.4 Derived Operations

From AND, OR, NOT we derive other important operations:

#### NAND (NOT AND)
Output is 0 **only** when ALL inputs are 1. (The opposite of AND.)
`F = (A · B)'`

| A | B | NAND |
|---|---|---|
| 0 | 0 | 1 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | **0** |

---

#### NOR (NOT OR)
Output is 1 **only** when ALL inputs are 0. (The opposite of OR.)
`F = (A + B)'`

| A | B | NOR |
|---|---|---|
| 0 | 0 | **1** |
| 0 | 1 | 0 |
| 1 | 0 | 0 |
| 1 | 1 | 0 |

---

#### XOR (Exclusive OR)
Output is 1 when inputs are **different** from each other.
**Symbol:** `⊕`
`F = A ⊕ B`

| A | B | XOR |
|---|---|---|
| 0 | 0 | 0 |
| 0 | 1 | **1** |
| 1 | 0 | **1** |
| 1 | 1 | 0 |

> **Key insight:** XOR outputs 1 when inputs are different; 0 when they are the same. This makes it extremely useful in arithmetic circuits and encryption.

---

#### XNOR (Exclusive NOR)
Output is 1 when inputs are **the same**.
`F = (A ⊕ B)'`

| A | B | XNOR |
|---|---|---|
| 0 | 0 | **1** |
| 0 | 1 | 0 |
| 1 | 0 | 0 |
| 1 | 1 | **1** |

---

## 4. Boolean Laws and Theorems

Just like regular algebra has rules (commutative, associative, etc.), Boolean algebra has its own set of laws. These laws are crucial for **simplifying** complex Boolean expressions.

### 4.1 Basic Identity Laws

| Law | AND Form | OR Form |
|---|---|---|
| **Identity** | A · 1 = A | A + 0 = A |
| **Null (Annihilator)** | A · 0 = 0 | A + 1 = 1 |
| **Idempotent** | A · A = A | A + A = A |
| **Complement** | A · A' = 0 | A + A' = 1 |
| **Double Negation** | (A')' = A | — |

---

### 4.2 Commutative Laws
The order of operands does not matter.
```
A · B = B · A
A + B = B + A
```

---

### 4.3 Associative Laws
Grouping does not matter.
```
(A · B) · C = A · (B · C)
(A + B) + C = A + (B + C)
```

---

### 4.4 Distributive Laws
```
A · (B + C) = (A · B) + (A · C)   [AND distributes over OR]
A + (B · C) = (A + B) · (A + C)   [OR distributes over AND]
```

> **Note:** The second form has no equivalent in regular algebra — it's unique to Boolean algebra!

---

### 4.5 Absorption Laws
```
A · (A + B) = A
A + (A · B) = A
```
> These are useful for eliminating redundant terms.

---

### 4.6 De Morgan's Theorems

**De Morgan's Theorems** are among the most important and widely used rules in Boolean algebra. They describe how to distribute a NOT operation over AND and OR.

**Theorem 1:**
```
(A · B)' = A' + B'
```
> "The complement of a product equals the sum of the complements."
> NOT (A AND B) = (NOT A) OR (NOT B)

**Theorem 2:**
```
(A + B)' = A' · B'
```
> "The complement of a sum equals the product of the complements."
> NOT (A OR B) = (NOT A) AND (NOT B)

**Practical rule to apply De Morgan's:**
1. **Complement** the entire expression.
2. **Change** every AND (·) to OR (+) and every OR (+) to AND (·).
3. **Complement** each individual variable.

**Example:**
```
F = (A · B · C)'
  = A' + B' + C'     ← AND becomes OR, each variable complemented
```

**Verification with truth table (2-variable case):**

| A | B | A' | B' | (A·B)' | A'+B' |
|---|---|---|---|---|---|
| 0 | 0 | 1 | 1 | 1 | 1 |
| 0 | 1 | 1 | 0 | 1 | 1 |
| 1 | 0 | 0 | 1 | 1 | 1 |
| 1 | 1 | 0 | 0 | 0 | 0 |

Both columns are identical — theorem confirmed ✓

---

## 5. Boolean Expressions and Simplification

### 5.1 Writing Boolean Expressions

A Boolean expression describes a logical function algebraically.

**Example:** "The alarm sounds if the door is open OR if both motion is detected AND it is night."

Let:
- `D` = door is open
- `M` = motion detected
- `N` = it is night

```
Alarm = D + (M · N)
```

---

### 5.2 Simplifying Boolean Expressions

Simplification reduces a Boolean expression to its simplest form — fewer terms and variables means simpler (cheaper, faster) circuits.

**Example 1:** Simplify `F = A·B + A·B'`

```
F = A·B + A·B'
  = A·(B + B')     [Distributive Law — factor out A]
  = A·(1)          [Complement Law: B + B' = 1]
  = A              [Identity Law: A·1 = A]
```

**Result:** `F = A` — the output depends only on A, regardless of B.

---

**Example 2:** Simplify `F = A + A'·B`

```
F = A + A'·B
  = (A + A')·(A + B)   [Distributive Law: A + (B·C) = (A+B)·(A+C)]
  = 1·(A + B)          [Complement Law: A + A' = 1]
  = A + B              [Identity Law: 1·X = X]
```

**Result:** `F = A + B`

---

**Example 3:** Simplify `F = A·B·C + A·B·C' + A·B'`

```
F = A·B·C + A·B·C' + A·B'
  = A·B·(C + C') + A·B'    [Factor out A·B from first two terms]
  = A·B·(1) + A·B'         [Complement Law]
  = A·B + A·B'             [Identity Law]
  = A·(B + B')             [Factor out A]
  = A·(1)                  [Complement Law]
  = A                      [Identity Law]
```

**Result:** `F = A` — three terms collapsed to one variable!

---

### 5.3 Sum of Products (SOP) Form

A Boolean expression in **Sum of Products** form is a sum (OR) of product (AND) terms, where each term may contain variables or their complements.

```
F = A·B' + A'·B·C + B·C'
```

Each AND term is called a **minterm**. SOP is the standard form most useful for building circuits.

---

### 5.4 Product of Sums (POS) Form

A Boolean expression in **Product of Sums** form is a product (AND) of sum (OR) terms.

```
F = (A + B') · (A' + C) · (B + C')
```

Each OR term is called a **maxterm**.

---

## 6. Logic Gates

Logic gates are the **physical hardware components** that implement Boolean operations. They are built from transistors and are the building blocks of all digital circuits.

---

### 6.1 AND Gate

- **Symbol:** D-shape with a flat input side
- **Operation:** Output is HIGH (1) only when **all** inputs are HIGH
- **Boolean:** `F = A · B`

```
    A ──┐
        ├──[AND]── F
    B ──┘

Truth Table:
A B | F
0 0 | 0
0 1 | 0
1 0 | 0
1 1 | 1
```

---

### 6.2 OR Gate

- **Symbol:** Curved D-shape with curved input side
- **Operation:** Output is HIGH when **at least one** input is HIGH
- **Boolean:** `F = A + B`

```
    A ──┐
        ├──[OR]─── F
    B ──┘

Truth Table:
A B | F
0 0 | 0
0 1 | 1
1 0 | 1
1 1 | 1
```

---

### 6.3 NOT Gate (Inverter)

- **Symbol:** Triangle with a small circle (bubble) at the output
- **Operation:** Inverts the input
- **Boolean:** `F = A'`

```
    A ──[NOT]── F

Truth Table:
A | F
0 | 1
1 | 0
```

---

### 6.4 NAND Gate

- **Symbol:** AND gate with a bubble at the output
- **Operation:** Opposite of AND — output is LOW only when **all** inputs are HIGH
- **Boolean:** `F = (A · B)'`

```
    A ──┐
        ├──[NAND]── F
    B ──┘

Truth Table:
A B | F
0 0 | 1
0 1 | 1
1 0 | 1
1 1 | 0
```

---

### 6.5 NOR Gate

- **Symbol:** OR gate with a bubble at the output
- **Operation:** Opposite of OR — output is HIGH only when **all** inputs are LOW
- **Boolean:** `F = (A + B)'`

```
    A ──┐
        ├──[NOR]─── F
    B ──┘

Truth Table:
A B | F
0 0 | 1
0 1 | 0
1 0 | 0
1 1 | 0
```

---

### 6.6 XOR Gate

- **Symbol:** OR gate with an extra curved line on the input side
- **Operation:** Output is HIGH when inputs are **different**
- **Boolean:** `F = A ⊕ B`

```
    A ──┐
        ├──[XOR]─── F
    B ──┘

Truth Table:
A B | F
0 0 | 0
0 1 | 1
1 0 | 1
1 1 | 0
```

---

### 6.7 XNOR Gate

- **Symbol:** XOR gate with a bubble at the output
- **Operation:** Output is HIGH when inputs are the **same**
- **Boolean:** `F = (A ⊕ B)'`

```
    A ──┐
        ├──[XNOR]── F
    B ──┘

Truth Table:
A B | F
0 0 | 1
0 1 | 0
1 0 | 0
1 1 | 1
```

---

### 6.8 Gate Summary Table

| Gate | Symbol | Boolean | Output is 1 when... |
|---|---|---|---|
| AND | `·` | `A · B` | All inputs are 1 |
| OR | `+` | `A + B` | At least one input is 1 |
| NOT | `'` | `A'` | Input is 0 |
| NAND | `(·)'` | `(A · B)'` | At least one input is 0 |
| NOR | `(+)'` | `(A + B)'` | All inputs are 0 |
| XOR | `⊕` | `A ⊕ B` | Inputs are different |
| XNOR | `(⊕)'` | `(A ⊕ B)'` | Inputs are the same |

---

## 7. Combining Gates: Logic Circuits

Real-world digital systems are built by combining multiple logic gates into **logic circuits**. Let's trace through some important examples.

---

### 7.1 Reading a Logic Circuit

To find the output of a logic circuit, trace the signal from inputs to output, evaluating each gate along the way.

**Example:** Find the output F for inputs A=1, B=0, C=1

```
Circuit: F = (A · B) + C'

Step 1: A · B = 1 · 0 = 0
Step 2: C' = 1' = 0
Step 3: F = 0 + 0 = 0
```

---

### 7.2 The Half Adder

A **half adder** is a circuit that adds two single bits together and produces a **Sum** and a **Carry** output. It is one of the foundational building blocks of arithmetic circuits inside a CPU.

**Inputs:** A, B (single bits)
**Outputs:**
- **Sum (S):** The result of adding A + B (without the carry)
- **Carry (C):** The carry bit (overflow into the next position)

**Truth Table:**

| A | B | Sum (S) | Carry (C) |
|---|---|---|---|
| 0 | 0 | 0 | 0 |
| 0 | 1 | 1 | 0 |
| 1 | 0 | 1 | 0 |
| 1 | 1 | 0 | 1 |

**Observations:**
- Sum = 1 when inputs are **different** → **XOR gate**
- Carry = 1 only when **both** inputs are 1 → **AND gate**

**Boolean Expressions:**
```
S = A ⊕ B
C = A · B
```

**Circuit Implementation:**
```
A ──┬──[XOR]──── S (Sum)
    │
B ──┴──[AND]──── C (Carry)
```

> **Example:** Add 1 + 1 in binary:
> - A=1, B=1
> - Sum = 1 XOR 1 = 0
> - Carry = 1 AND 1 = 1
> - Result: Carry=1, Sum=0 → binary `10` = decimal 2 ✓

---

### 7.3 The Full Adder

A **full adder** extends the half adder to also accept a **carry-in** bit from a previous addition. This allows adders to be chained together to add multi-bit numbers.

**Inputs:** A, B, Cᵢₙ (carry-in)
**Outputs:** Sum (S), Cₒᵤₜ (carry-out)

**Boolean Expressions:**
```
S    = A ⊕ B ⊕ Cᵢₙ
Cₒᵤₜ = (A · B) + (Cᵢₙ · (A ⊕ B))
```

**Implementation:** A full adder can be built from **two half adders** and one **OR gate**.

> **Significance:** By chaining multiple full adders, a CPU can add any size binary numbers. A 32-bit adder, for instance, chains 32 full adders together.

---

### 7.4 The Multiplexer (MUX)

A **multiplexer** is a circuit that selects one of several input signals and routes it to a single output, based on **selector inputs**.

A 2-to-1 MUX has:
- Inputs: `I₀`, `I₁`
- Selector: `S`
- Output: `F`

```
If S = 0 → F = I₀
If S = 1 → F = I₁
```

**Boolean Expression:** `F = (I₀ · S') + (I₁ · S)`

> **Analogy:** A TV remote control channel button is like a selector — it chooses which channel (input) to display on the screen (output).

---

## 8. Universal Gates: NAND and NOR

A **universal gate** is a single gate type from which **any other logic gate** can be constructed. Both NAND and NOR are universal gates.

This is extremely important in chip manufacturing — a factory only needs to produce one type of gate to build any circuit.

---

### 8.1 NAND as a Universal Gate

**NOT using NAND:**
```
Connect both inputs of a NAND gate together:
A ──┬──[NAND]── A'
    ┘

(A · A)' = A' (by Idempotent + De Morgan's)
```

**AND using NAND:**
```
A ──┐                  ┌──[NAND]──┐
    ├──[NAND]──────────┤          ├── A·B
B ──┘                  └──[NAND]──┘
                         (NOT the NAND output)
```
Put the NAND output through another NAND used as NOT:
`((A·B)')' = A·B`

**OR using NAND (using De Morgan's):**
```
A ──[NAND as NOT]──┐
                   ├──[NAND]── A+B
B ──[NAND as NOT]──┘

(A')·(B') = (A+B)' ... apply De Morgan's:
NAND of (A', B') = (A'·B')' = A + B  ✓
```

---

### 8.2 NOR as a Universal Gate

Similarly, NOR can implement NOT, AND, and OR:

**NOT using NOR:**
```
A ──┬──[NOR]── A'
    ┘
```

**OR using NOR:**
```
A ──┐                     ┌──[NOR as NOT]──┐
    ├──[NOR]──────────────┤                ├── A+B
B ──┘                     └────────────────┘
```

**AND using NOR (using De Morgan's):**
```
A ──[NOR as NOT]──┐
                  ├──[NOR]── A·B
B ──[NOR as NOT]──┘

(A+B)' → NOR of (A', B') = (A'+B')' = A·B  ✓
```

---

## 9. Real-World Applications

Boolean algebra and logic gates are not just theoretical — they are the foundation of virtually every digital system.

### 9.1 CPU Arithmetic and Logic Unit (ALU)

The ALU in every CPU is built entirely from logic gates. It uses:
- **Adder circuits** (half adders + full adders) for arithmetic
- **AND/OR/XOR gates** for logical operations
- **Comparator circuits** (using XNOR and subtractors) to compare values

### 9.2 Memory (RAM) Cells

A single bit of RAM is stored in a **flip-flop** — a circuit made from cross-connected NAND or NOR gates that can hold a 0 or 1 even after the input is removed.

### 9.3 Decision-Making in Programs

Every `if` statement in a programming language ultimately compiles down to Boolean logic executed on gates.

```python
# Python code
if age >= 18 and has_id:
    print("Access granted")
```

This becomes Boolean logic: `F = (age >= 18) · has_id`, which is evaluated by AND gates in the hardware.

### 9.4 Search Engines

Boolean logic powers database and search queries:
- **Google search:** `"climate change" AND "renewable energy" NOT "nuclear"` uses AND/NOT/OR logic to filter results.
- **Database queries:** `SELECT * FROM users WHERE age > 18 AND country = 'MX'` uses Boolean conditions.

### 9.5 Digital Security

Access control systems use Boolean logic:
- A vault opens only if `keycard = valid AND pin = correct AND biometric = match`
- Security alerts trigger if `intrusion_sensor = 1 OR fire_sensor = 1 OR tamper_sensor = 1`

### 9.6 Encryption (XOR Cipher)

The XOR operation is fundamental to many encryption algorithms. In a simple XOR cipher:
```
Encrypted = Plaintext ⊕ Key
Decrypted = Encrypted ⊕ Key   (XOR with the same key reverses it)
```

**Example:**
```
Plaintext:  1 0 1 1 0 1 0 0
Key:        1 1 0 0 1 1 0 1
            ───────────────
Encrypted:  0 1 1 1 1 0 0 1

Decrypt:
Encrypted:  0 1 1 1 1 0 0 1
Key:        1 1 0 0 1 1 0 1
            ───────────────
Decrypted:  1 0 1 1 0 1 0 0  ← Original plaintext recovered ✓
```

---

## 10. Module Summary

Key takeaways from Module 3:

- **Boolean algebra** works with only two values: 0 (FALSE) and 1 (TRUE), making it the mathematical foundation for digital computers.
- The three fundamental operations are **AND** (·), **OR** (+), and **NOT** ('), from which NAND, NOR, XOR, and XNOR are derived.
- **Truth tables** enumerate all possible input combinations and their outputs for a Boolean function.
- Boolean algebra has a rich set of laws: **Identity, Null, Idempotent, Complement, Commutative, Associative, Distributive, Absorption**, and the critically important **De Morgan's Theorems**.
- **De Morgan's Theorems** state: `(A·B)' = A'+B'` and `(A+B)' = A'·B'` — essential for simplifying and transforming expressions.
- **Logic gates** (AND, OR, NOT, NAND, NOR, XOR, XNOR) are the physical hardware components that implement Boolean operations using transistors.
- Gates are combined into **logic circuits** such as the **half adder** (XOR + AND), **full adder**, and **multiplexer**.
- **NAND and NOR are universal gates** — any other gate can be built from them alone.
- Boolean logic is the foundation of CPUs, RAM, programming decisions, search queries, security systems, and encryption.

---

## 11. Module Quiz

Test your understanding of Module 3.

---

**Question 1**
What is the output of `F = A · B + C'` when A=1, B=0, C=0?

- A) 0
- B) 1
- C) Cannot be determined
- D) Depends on gate order

---

**Question 2**
Which Boolean law states that `A + (A · B) = A`?

- A) Distributive Law
- B) De Morgan's Theorem
- C) Absorption Law
- D) Complement Law

---

**Question 3**
Apply De Morgan's Theorem to simplify `(A + B + C)'`:

- A) `A' · B' + C'`
- B) `A' + B' + C'`
- C) `A' · B' · C'`
- D) `(A · B · C)'`

---

**Question 4**
Which gate produces an output of 1 **only** when all its inputs are 0?

- A) NAND
- B) NOR
- C) XOR
- D) AND

---

**Question 5**
A half adder receives inputs A=1 and B=1. What are the Sum and Carry outputs?

- A) Sum=1, Carry=0
- B) Sum=0, Carry=0
- C) Sum=0, Carry=1
- D) Sum=1, Carry=1

---

**Question 6**
Simplify the Boolean expression: `F = A·B + A·B'`

- A) `F = B`
- B) `F = A + B`
- C) `F = A`
- D) `F = 1`

---

**Question 7**
Which of the following is true about NAND gates?

- A) NAND gates can only implement AND and NOT operations
- B) NAND gates are universal — any Boolean function can be built using only NAND gates
- C) NAND output is 1 only when all inputs are 1
- D) NAND gates cannot implement an OR gate

---

**Question 8**
A security system triggers an alarm (`F=1`) under this rule: "Trigger if the window sensor is activated, OR if both the motion sensor is active AND the time is after midnight."

Let `W` = window sensor, `M` = motion sensor, `T` = after midnight. Which expression correctly represents this?

- A) `F = W · (M + T)`
- B) `F = W + M · T`
- C) `F = (W + M) · T`
- D) `F = W · M + T`

---

**Question 9**
Using De Morgan's Theorem, which expression is equivalent to `(A · B')'`?

- A) `A' · B`
- B) `A' + B`
- C) `A + B'`
- D) `A' · B'`

---

**Question 10** *(Short Answer)*
Explain what a **full adder** is, how it differs from a **half adder**, and why full adders are necessary for adding multi-bit binary numbers. Provide an example.

---

### Answer Key

| Question | Answer | Explanation |
|---|---|---|
| 1 | **B** | A·B = 1·0 = 0; C' = 0' = 1; F = 0 + 1 = **1** |
| 2 | **C** | The Absorption Law states A + (A·B) = A and A·(A+B) = A. |
| 3 | **C** | By De Morgan's: `(A+B+C)' = A'·B'·C'` — OR becomes AND, each variable complemented. |
| 4 | **B** | NOR outputs 1 only when all inputs are 0 (it is the complement of OR). |
| 5 | **C** | S = A⊕B = 1⊕1 = 0; C = A·B = 1·1 = 1. So Sum=0, Carry=1 (binary 1+1 = 10). |
| 6 | **C** | A·B + A·B' = A·(B+B') = A·1 = **A** (Distributive + Complement + Identity Laws). |
| 7 | **B** | NAND is a universal gate — NOT, AND, OR (and all others) can all be built from NAND gates alone. |
| 8 | **B** | "W OR (M AND T)" → `F = W + M·T`. In Boolean algebra, AND (·) has higher precedence than OR (+). |
| 9 | **B** | `(A·B')' = A' + (B')' = A' + B` (De Morgan's: product complement becomes sum of complements; double negation: (B')' = B). |
| 10 | *(See below)* | |

**Question 10 – Sample Answer:**
> A **half adder** adds two single-bit binary inputs (A and B) and produces two outputs: a **Sum** (S = A ⊕ B) and a **Carry** (C = A · B). It is called "half" because it cannot account for a carry coming in from a previous addition stage.
>
> A **full adder** adds three inputs: A, B, and a **carry-in (Cᵢₙ)** from the previous bit position. It produces a Sum (`S = A ⊕ B ⊕ Cᵢₙ`) and a carry-out (`Cₒᵤₜ`). A full adder can be built from two half adders and one OR gate.
>
> **Why full adders are necessary:** When adding multi-bit numbers, each bit position (except the first) may receive a carry from the position to its right. A half adder has no carry-in input, so it can only be used for the **least significant bit**. All higher bit positions require a full adder to properly handle carries.
>
> **Example:** Add `011₂ + 011₂` (3 + 3 = 6):
> - Bit 0: Half Adder: 1+1 → Sum=0, Carry=1
> - Bit 1: Full Adder: 1+1+1(carry) → Sum=1, Carry=1
> - Bit 2: Full Adder: 0+0+1(carry) → Sum=1, Carry=0
> - Result: `110₂` = 6₁₀ ✓

---

*End of Module 3 — Up next: **Module 4: Operating Systems***
