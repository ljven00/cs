# CS 1105 — Module 3: Sequential Logic Design

---

## 1. What is Sequential Logic?

Unlike combinational circuits, **sequential circuits** have **memory** — their output depends on:

- Current inputs
- **Past state** (what happened before)

> Think of a light switch with a timer — the output depends not just on whether you press it, but on what state it was already in.

The "memory" element is the **flip-flop**.

---

## 2. Latches vs. Flip-Flops

| Feature        | Latch                        | Flip-Flop                    |
|----------------|------------------------------|------------------------------|
| Trigger        | Level-sensitive (while enabled) | Edge-triggered (on clock tick) |
| Stability      | Can be transparent/unstable  | Stable and predictable       |
| Common use     | Simple storage               | Registers, counters, FSMs    |

> In practice, **flip-flops** are preferred in synchronous (clocked) digital systems.

---

## 3. The SR Latch

The simplest memory element. Built from two cross-coupled NOR (or NAND) gates.

| S | R | Q (next) | Notes          |
|---|---|----------|----------------|
| 0 | 0 |  Q (hold)|  No change     |
| 1 | 0 |  1       |  Set           |
| 0 | 1 |  0       |  Reset         |
| 1 | 1 |  ?       |  **Invalid**   |

> The S=1, R=1 condition is forbidden — it creates an undefined state.

---

## 4. Flip-Flop Types

### 4.1 D Flip-Flop (Data / Delay)

The most commonly used. On each **rising clock edge**, Q captures the value of D.

| D | Q (next) |
|---|----------|
| 0 |    0     |
| 1 |    1     |

```
Q(next) = D
```

Simple rule: **whatever D is, Q becomes it on the clock edge.**

---

### 4.2 JK Flip-Flop

An improvement on SR — eliminates the invalid state. J=Set, K=Reset.

| J | K | Q (next)     |
|---|---|--------------|
| 0 | 0 | Q (no change)|
| 0 | 1 | 0 (reset)    |
| 1 | 0 | 1 (set)      |
| 1 | 1 | Q̄ (toggle)  |

```
Q(next) = J·Q̄ + K̄·Q
```

---

### 4.3 T Flip-Flop (Toggle)

A simplified JK where J=K=T. Toggles output when T=1.

| T | Q (next) |
|---|----------|
| 0 | Q (hold) |
| 1 | Q̄ (toggle)|

```
Q(next) = T ⊕ Q
```

> T flip-flops are the building block of **binary counters**.

---

## 5. Registers

A **register** is a group of flip-flops that store multiple bits together.

- An **8-bit register** = 8 D flip-flops sharing the same clock
- On each clock edge, all 8 bits are captured simultaneously

### Shift Register

Bits shift from one flip-flop to the next on each clock pulse.

```
Clock:  _____|‾|___|‾|___|‾|___
Input:  1
         FF1 → FF2 → FF3 → FF4
After 1 tick: [1, 0, 0, 0]
After 2 ticks:[0, 1, 0, 0]
After 3 ticks:[0, 0, 1, 0]
```

Used in serial communication and data conversion.

---

## 6. Counters

Counters use T or JK flip-flops to count clock pulses.

### 2-Bit Ripple Counter (counts 0 → 3 → 0 → ...)

```
State sequence:
00 → 01 → 10 → 11 → 00 → ...
 0 →  1 →  2 →  3 →  0 → ...
```

Each flip-flop's output feeds the clock of the next — that's what makes it a **ripple** counter (the carry "ripples" through).

> **Synchronous counters** use a shared clock — faster and more reliable, but slightly more complex.

---

## 7. Finite State Machines (FSMs)

An FSM is a model for any system that moves between a finite number of **states** based on inputs.

### Two types:

| Type  | Output depends on...         |
|-------|------------------------------|
| Moore | State only                   |
| Mealy | State **and** current input  |

### FSM Design Steps

1. Define states
2. Draw a **state diagram**
3. Build a **state transition table**
4. Derive flip-flop input equations
5. Derive output equations

---

## 8. Worked Example — Traffic Light FSM

**System:** A simple traffic light with 3 states: **GREEN → YELLOW → RED → GREEN...**

Each state lasts one clock cycle (simplified).

### State Diagram

```
    ┌─────────────────────────────┐
    ↓                             │
  GREEN  ──(always)──►  YELLOW  ──(always)──►  RED
    ▲                                            │
    └────────────────(always)────────────────────┘
```

### State Encoding

| State  | S1 | S0 |
|--------|----|----|
| GREEN  |  0 |  0 |
| YELLOW |  0 |  1 |
| RED    |  1 |  0 |

### State Transition Table

| Current S1 S0 | Next S1 S0 |
|---------------|------------|
|    0  0       |   0  1     |
|    0  1       |   1  0     |
|    1  0       |   0  0     |

### Next-State Equations (using D flip-flops)

```
D1 = S0
D0 = S1̄ · S0̄  →  simplifies to  D0 = ¬(S1 OR S0) ... 
     actually: D0 = 1 only when current = GREEN (00)
     so D0 = S1̄·S0̄
```

> In a real design you'd verify this with K-Maps before wiring it up.

---

## 📝 Module 3 Quiz

**Q1.** What is the fundamental difference between a latch and a flip-flop?

<details>
<summary>Show Answer</summary>

A **latch** is level-sensitive — it responds to its inputs as long as the enable signal is active. A **flip-flop** is edge-triggered — it only responds to inputs at the moment of a clock edge (rising or falling), making it more stable and predictable in synchronous systems.
</details>

---

**Q2.** A D flip-flop currently holds Q=0. The input D=1 arrives. What is Q after the next rising clock edge?

<details>
<summary>Show Answer</summary>

**Q = 1.** The D flip-flop copies D to Q on every rising clock edge. Q(next) = D = 1.
</details>

---

**Q3.** A JK flip-flop has J=1, K=1, and current Q=0. What is Q after the clock edge?

<details>
<summary>Show Answer</summary>

J=K=1 causes a **toggle**. Q was 0, so Q(next) = **1**.

```
Q(next) = J·Q̄ + K̄·Q = 1·1 + 0·0 = 1
```
</details>

---

**Q4.** A 2-bit ripple counter is at state `11`. What is the next state?

<details>
<summary>Show Answer</summary>

The counter wraps around: **`11` → `00`**

The sequence is: 00 → 01 → 10 → 11 → 00 → ...
</details>

---

**Q5.** In a Moore FSM, what does the output depend on?

<details>
<summary>Show Answer</summary>

In a **Moore FSM**, the output depends **only on the current state** — not on the current input. This makes outputs more stable (they only change when the state changes).
</details>

---

## 🔑 Key Takeaways

- Sequential circuits have memory — outputs depend on inputs **and** past state.
- Flip-flops are the basic memory unit; D is the most common in practice.
- JK flip-flops fix the SR invalid state; T flip-flops are ideal for counters.
- Registers store multiple bits; shift registers move bits serially.
- FSMs model any state-based system — mastering them is essential for hardware and software design.

---

*Next up → **Module 4: Computer Arithmetic***
