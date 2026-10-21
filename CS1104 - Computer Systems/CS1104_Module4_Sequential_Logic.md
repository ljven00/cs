# CS 1104 Computer Systems
## Module 4: Sequential Logic

---

## Overview

In Module 3, we built combinational circuits whose outputs depend only on *current* inputs. But a computer also needs **memory** — the ability to remember past states. That's what **sequential logic** provides. Sequential circuits have outputs that depend on both current inputs **and stored state**. This is how registers, memory, and the CPU's control unit work.

---

## 1. The Key Difference: State

```
Combinational:  Inputs ──→ [Circuit] ──→ Output
                                              (no memory)

Sequential:     Inputs ──→ [Circuit] ──→ Output
                               ↑    ↓
                            [State / Memory]
```

The stored **state** feeds back into the circuit, so the same input can produce different outputs depending on what happened before.

---

## 2. The SR Latch (The Simplest Memory Element)

The **SR Latch** (Set-Reset) is the most basic memory circuit. It stores **one bit**.

It's built from two cross-coupled NOR gates:

```
S ──→[NOR]──→ Q
        ↑      |
        └──────┘
R ──→[NOR]──→ Q̄  (Q-bar, the complement of Q)
        ↑      |
        └──────┘
```

### Behavior

| S | R | Q (next) | Meaning          |
|---|---|----------|------------------|
| 0 | 0 |  Q (hold)| Remember state   |
| 1 | 0 |    1     | Set Q to 1       |
| 0 | 1 |    0     | Reset Q to 0     |
| 1 | 1 | ⚠️ Invalid | Forbidden state |

- **S=1 (Set):** Forces Q to 1 and stores it
- **R=1 (Reset):** Forces Q to 0 and stores it
- **S=0, R=0:** Holds the current value — this is **memory**
- **S=1, R=1:** Both outputs try to be 1 simultaneously — undefined and forbidden

> **This is the origin of computer memory.** The hold condition (S=0, R=0) is a circuit quietly remembering a bit even with no input.

---

## 3. The D Latch

The SR latch's forbidden state is inconvenient. The **D Latch** (Data latch) fixes this with a single data input and an **Enable** signal.

```
D ──→[AND]──→ S ──→[SR Latch]──→ Q
     [NOT]──→ R
Enable──────────────────────────↑
```

| Enable | D | Q (next) |
|--------|---|----------|
|   0    | X | Q (hold) |
|   1    | 0 |    0     |
|   1    | 1 |    1     |

- When **Enable = 1**: Q follows D (transparent)
- When **Enable = 0**: Q holds its last value (latched)

The D latch is **level-triggered** — it responds to the input the entire time Enable is high.

---

## 4. The D Flip-Flop

Latches are problematic because they're transparent — input changes while Enable is high can cause glitches. A **D Flip-Flop** solves this by being **edge-triggered**: it only captures D at the precise moment the clock signal transitions (usually rising edge).

```
         ┌──────────────┐
    D ──→│              │──→ Q
CLK ──→│  D Flip-Flop  │──→ Q̄
         └──────────────┘
```

| CLK Edge | D | Q (next) |
|----------|---|----------|
| ↑ (rise) | 0 |    0     |
| ↑ (rise) | 1 |    1     |
| No edge  | X | Q (hold) |

The flip-flop **samples D only on the rising clock edge** and holds that value until the next rising edge.

> **This is why the clock is so fundamental to CPUs.** Every flip-flop in the system updates simultaneously on the clock edge — creating a perfectly synchronized, step-by-step machine.

---

## 5. The Clock Signal

The **clock** is a periodic square wave that drives all sequential logic in a synchronous circuit.

```
    ┌─┐ ┌─┐ ┌─┐ ┌─┐
────┘ └─┘ └─┘ └─┘ └───
    ↑   ↑   ↑   ↑
  Rising edges — flip-flops update here
```

- **Clock frequency:** How many cycles per second (Hz). A 3 GHz CPU has 3 billion rising edges per second.
- **Clock period:** Time between rising edges = 1 / frequency
- **One clock cycle = one "tick" of the machine**

Between clock edges, combinational logic (like the ALU) computes its result. On the next clock edge, flip-flops capture and store that result.

---

## 6. Registers

A **register** is simply a group of D flip-flops sharing a common clock and enable signal. An 8-bit register stores 8 bits simultaneously.

```
D[0] ──→ [DFF] ──→ Q[0]
D[1] ──→ [DFF] ──→ Q[1]
  ...               ...
D[7] ──→ [DFF] ──→ Q[7]
            ↑
           CLK (shared)
```

Registers are the **fastest storage** in a computer — they live inside the CPU itself. You'll work with them directly in the Machine Language and Assembler modules.

---

## 7. A Practical Example: 2-bit Counter

Let's build something real. A **2-bit counter** counts 00 → 01 → 10 → 11 → 00 → ...

It uses two D flip-flops (Q₁, Q₀) and some combinational logic:

```
D₀ = NOT Q₀              (bit 0 always toggles)
D₁ = Q₁ XOR Q₀           (bit 1 toggles when Q₀ = 1)
```

| Clock Cycle | Q₁ | Q₀ | Decimal |
|-------------|----|----|---------|
| 0           | 0  | 0  | 0       |
| 1           | 0  | 1  | 1       |
| 2           | 1  | 0  | 2       |
| 3           | 1  | 1  | 3       |
| 4           | 0  | 0  | 0 (wraps) |

Notice: the counter uses **both combinational logic** (the XOR and NOT gates computing D) **and sequential logic** (the flip-flops storing state). This pattern — combinational logic feeding into registers, clocked by the same signal — is the blueprint for the entire CPU.

---

## 8. Combinational vs. Sequential: Side by Side

| Feature          | Combinational          | Sequential                  |
|------------------|------------------------|-----------------------------|
| Output depends on| Current inputs only    | Current inputs + stored state |
| Has memory?      | No                     | Yes                         |
| Clock needed?    | No                     | Yes (for synchronous)       |
| Examples         | ALU, MUX, Decoder      | Registers, counters, CPU    |
| Modules covered  | Module 3               | This module                 |

---

## Summary

| Component     | What it does                                              |
|---------------|-----------------------------------------------------------|
| SR Latch      | Stores 1 bit using cross-coupled NOR gates                |
| D Latch       | Cleaner 1-bit storage; transparent when Enable = 1        |
| D Flip-Flop   | Captures input only on clock edge; foundation of registers |
| Clock         | Synchronizes all flip-flops; defines the CPU's rhythm     |
| Register      | n flip-flops grouped together to store n bits             |
| Counter       | Flip-flops + combinational logic cycling through states   |

---

## 🧠 Module Quiz

1. What is the fundamental difference between a combinational circuit and a sequential circuit? Give one example of each from the modules so far.

2. An SR Latch currently has Q = 1. Inputs change to S = 0, R = 0. What is the new value of Q, and why?

3. Why is the S = 1, R = 1 condition forbidden in an SR Latch?

4. A D Flip-Flop has Q = 0 and D = 1. The clock has not yet ticked. What is Q? What happens to Q after the next rising clock edge?

5. A CPU runs at 2 GHz. How much time (in nanoseconds) does the combinational logic have to compute its result within a single clock cycle?

6. Extend the 2-bit counter from Section 7: what Boolean expressions for D₂ would you need to make it a **3-bit counter** (counting 0 through 7)?  
   *(Hint: when should bit 2 toggle?)*
