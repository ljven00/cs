# Module 6: Software Maintenance and Evolution

**Course:** CS 2401 – Software Engineering 1
**Module:** 6 of 8

---

## Learning Objectives

By the end of this module, students will be able to:
- Classify the four types of software maintenance
- Explain why software evolves over time (Lehman's Laws)
- Understand technical debt and its consequences
- Apply refactoring techniques to improve code quality
- Describe legacy system strategies (rewrite vs. reengineer)

---

## 6.1 Why Maintenance Matters

Software maintenance is not a cleanup phase — it is **the longest and most expensive phase** of the software lifecycle.

**Industry statistics:**
- Maintenance consumes **60–80% of total software lifecycle costs**
- The average enterprise application is maintained for **10–20 years**
- Most developers spend more time **reading and modifying existing code** than writing new code

> "Any fool can write code that a computer can understand. Good programmers write code that humans can understand." — Martin Fowler

---

## 6.2 Types of Software Maintenance

**IEEE Std 14764** defines four types of software maintenance:

### 1. Corrective Maintenance
Fixing **defects** discovered after release.

**Examples:**
- Patching a security vulnerability
- Fixing a crash triggered by an edge case
- Correcting a calculation error in a tax module

**Trigger:** Bug reports from users or monitoring systems

---

### 2. Adaptive Maintenance
Modifying software to **adapt to environmental changes** (new OS, database, regulations, APIs).

**Examples:**
- Updating the app to support iOS 18
- Migrating from MySQL 5.7 to MySQL 8.0
- Updating API calls after a third-party provider changes their interface
- Compliance with GDPR or new tax laws

**Trigger:** External environment changes

---

### 3. Perfective Maintenance
Improving software **performance or maintainability** — not fixing bugs, just making it better.

**Examples:**
- Optimizing a slow database query
- Refactoring messy code for readability
- Adding better logging and monitoring
- Restructuring the codebase for easier testing

**Trigger:** Developer initiative or performance bottlenecks

---

### 4. Preventive Maintenance
Making changes to **prevent future problems**.

**Examples:**
- Updating deprecated library versions before they become unsupported
- Adding unit tests to poorly covered legacy code
- Documenting undocumented modules

**Trigger:** Risk analysis, code audits

---

### Maintenance Type Summary

```
         Problem EXISTS?
              │
      ┌───────┴────────┐
     YES               NO
      │                 │
  Corrective        External change?
  (fix the bug)      │         │
                    YES        NO
                     │         │
                 Adaptive   Improve system?
               (environment)   │       │
                              YES      NO
                               │        │
                           Perfective  Preventive
                           (quality)   (prevention)
```

---

## 6.3 Lehman's Laws of Software Evolution

Manny Lehman studied how large software systems evolve over time and proposed a set of laws (1974–1996).

| Law | Name | Description |
|---|---|---|
| I | Continuing Change | A system must be continually adapted or it becomes progressively less useful |
| II | Increasing Complexity | As software evolves, its complexity increases unless work is done to reduce it |
| III | Self-Regulation | Software evolution is a self-regulating process |
| IV | Conservation of Organizational Stability | The average rate of global activity in an evolving system tends to remain constant |
| V | Conservation of Familiarity | The amount of new content added per release must remain roughly constant |
| VI | Continuing Growth | Functionality must grow to maintain user satisfaction |
| VII | Declining Quality | Quality declines unless rigorously maintained |
| VIII | Feedback System | Evolution processes constitute multi-level, multi-loop feedback systems |

**Key takeaway:** Software that is not actively maintained will become less useful and more fragile over time. This is inevitable.

---

## 6.4 Technical Debt

**Technical debt** is a metaphor (coined by Ward Cunningham) for the hidden cost of shortcuts and poor design decisions in code.

Like financial debt, technical debt accumulates **interest** — the longer it goes unaddressed, the harder it becomes to work with the codebase.

### Types of Technical Debt

| Type | Example |
|---|---|
| **Code debt** | Duplicated logic, unclear variable names |
| **Design debt** | Missing abstraction layers, monolithic functions |
| **Testing debt** | Missing test coverage |
| **Documentation debt** | Undocumented APIs, missing comments |
| **Infrastructure debt** | Using outdated servers or deprecated tools |

### Technical Debt Quadrant (Martin Fowler)

```
                    Reckless           Prudent
                 ┌─────────────┬──────────────────────┐
Deliberate       │ "We don't   │  "We must ship now   │
                 │  have time  │   and deal with       │
                 │  for design"│   consequences later" │
                 ├─────────────┼──────────────────────┤
Inadvertent      │ "What's     │  "Now we know how     │
                 │  layering?" │   we should have      │
                 │             │   done it"            │
                 └─────────────┴──────────────────────┘
```

The **Prudent + Deliberate** quadrant is the only acceptable form — consciously taking on debt with a plan to pay it back.

---

## 6.5 Refactoring

**Refactoring** is the process of restructuring existing code **without changing its external behavior**.

> "Refactoring is a controlled technique for improving the design of an existing code base." — Martin Fowler

### Common Refactoring Techniques

#### 1. Extract Method
Pull duplicated or complex code into a named function.

**Before:**
```python
def process_order(order):
    # validate
    if order.items is None or len(order.items) == 0:
        raise ValueError("Order has no items")
    if order.customer_id is None:
        raise ValueError("No customer")
    # calculate total
    total = sum(item.price * item.qty for item in order.items)
    tax = total * 0.08
    return total + tax
```

**After:**
```python
def validate_order(order):
    if not order.items:
        raise ValueError("Order has no items")
    if order.customer_id is None:
        raise ValueError("No customer")

def calculate_total(order):
    subtotal = sum(item.price * item.qty for item in order.items)
    return subtotal * 1.08

def process_order(order):
    validate_order(order)
    return calculate_total(order)
```

---

#### 2. Rename Variable / Method
Give names that reveal intent.

```python
# Before
d = 86400          # What is d?

# After
SECONDS_PER_DAY = 86400
```

---

#### 3. Replace Magic Numbers with Constants

```python
# Before
if user.age < 18:
    raise ValueError("Too young")

# After
MINIMUM_AGE = 18
if user.age < MINIMUM_AGE:
    raise ValueError(f"Must be at least {MINIMUM_AGE}")
```

---

#### 4. Replace Conditional with Polymorphism

```python
# Before
def get_payment_fee(payment_type, amount):
    if payment_type == "credit":
        return amount * 0.03
    elif payment_type == "paypal":
        return amount * 0.029 + 0.30
    elif payment_type == "crypto":
        return amount * 0.01

# After (using polymorphism)
class CreditCardPayment:
    def calculate_fee(self, amount): return amount * 0.03

class PayPalPayment:
    def calculate_fee(self, amount): return amount * 0.029 + 0.30

class CryptoPayment:
    def calculate_fee(self, amount): return amount * 0.01
```

---

## 6.6 Code Smells

**Code smells** are symptoms in the code that suggest deeper problems. They are not bugs — but they indicate areas likely to cause problems.

| Smell | Description | Fix |
|---|---|---|
| **Long Method** | Function is too long (> 20-30 lines) | Extract Method |
| **God Class** | One class does everything | Split responsibilities |
| **Feature Envy** | Method uses data from another class more than its own | Move method |
| **Magic Numbers** | Unexplained numeric literals | Extract constant |
| **Duplicate Code** | Same logic in multiple places | Extract shared method |
| **Dead Code** | Code that is never executed | Delete it |
| **Shotgun Surgery** | One change requires edits in many places | Consolidate logic |

---

## 6.7 Legacy Systems

A **legacy system** is an old system that still performs business-critical functions but is difficult to maintain, often using outdated technology.

### Legacy System Strategies

| Strategy | Description | When to Use |
|---|---|---|
| **Leave it alone** | Keep using it as-is | System is stable, rarely changed |
| **Reengineering** | Restructure and improve while preserving functionality | System works but code is poor |
| **Replacement (Big Bang)** | Replace entirely with a new system | System is beyond repair |
| **Strangler Fig Pattern** | Gradually replace parts with a new system running in parallel | Low-risk incremental migration |

### Strangler Fig Pattern (Martin Fowler)

```
Phase 1:    [Legacy System] ──► [All Traffic]

Phase 2:    [New Feature X] ──► [Traffic for Feature X]
            [Legacy System] ──► [Everything Else]

Phase 3:    [New System]    ──► [Most Traffic]
            [Legacy System] ──► [Remaining legacy parts]

Phase 4:    [New System]    ──► [All Traffic]  ← Legacy retired
```

---

## 6.8 Version Control and Changelogs

Good maintenance requires disciplined version control.

### Git Best Practices for Maintenance

- Use **semantic versioning**: MAJOR.MINOR.PATCH (e.g., 2.4.1)
  - MAJOR: Breaking changes
  - MINOR: New backward-compatible features
  - PATCH: Bug fixes

- Write meaningful **commit messages:**
  ```
  # Bad:
  git commit -m "fixed stuff"

  # Good:
  git commit -m "fix: correct tax calculation for zero-rated items (BUG-0234)"
  ```

- Maintain a **CHANGELOG.md** documenting what changed in each release.

---

## 6.9 Case Study: Evolving a Monolithic System

**Scenario:** A 10-year-old e-commerce platform built as a PHP monolith. It processes $5M/day in transactions.

**Problems:**
- 250,000 lines of code with no tests
- 3-week deployment cycles (fear of breaking things)
- Any change to checkout affects the entire app
- Can't scale horizontally

**Decision:** Use the Strangler Fig Pattern to incrementally extract services.

**Year 1:** Extract Payment Service → Microservice with full test coverage
**Year 2:** Extract User Service → Microservice
**Year 3:** Extract Catalog Service → Microservice
**Year 4:** Legacy monolith retired; 95% test coverage on all new services

---

## 6.10 Summary

- Software maintenance is the longest, most expensive phase of the SDLC.
- Four types: Corrective, Adaptive, Perfective, and Preventive.
- Lehman's Laws confirm software inevitably grows in complexity without active effort.
- Technical debt accumulates interest — proactive refactoring keeps it manageable.
- Legacy systems can be reengineerd, replaced, or migrated incrementally using the Strangler Fig pattern.

---

## Quiz – Module 6

---

**1. Which type of maintenance involves adapting the software to a new operating system version?**

- A) Corrective
- B) Adaptive ✅
- C) Perfective
- D) Preventive

---

**2. According to Lehman's Laws, what happens to software complexity over time without intervention?**

- A) It decreases
- B) It stays the same
- C) It increases ✅
- D) It becomes unpredictable

---

**3. What is "technical debt"?**

- A) Money owed for software licenses
- B) The cost of poor design choices that must eventually be addressed ✅
- C) Bugs introduced by new features
- D) Documentation that is out of date

---

**4. What does the "Extract Method" refactoring technique do?**

- A) Deletes unused code
- B) Renames variables for clarity
- C) Pulls complex or duplicated code into a named function ✅
- D) Replaces conditionals with polymorphism

---

**5. The Strangler Fig Pattern is used for:**

- A) Writing unit tests for legacy code
- B) Gradually replacing a legacy system with a new one ✅
- C) Migrating databases
- D) Detecting code smells automatically

---

**6. Short Answer:** Explain the difference between refactoring and rewriting. When would you choose each?

---

**7. Practical Exercise:** Identify at least 3 code smells in the following code and explain how you would fix each:

```python
def f(x, y, t):
    r = x * y
    if t == 1:
        return r * 0.21
    elif t == 2:
        return r * 0.16
    elif t == 3:
        return r * 0.08
    else:
        return r
```

---

## Further Reading

- Fowler, M. *Refactoring: Improving the Design of Existing Code* (2nd ed.)
- Lehman, M.M. "Programs, Life Cycles, and Laws of Software Evolution" (1980)
- Feathers, M. *Working Effectively with Legacy Code*
- [Refactoring Guru](https://refactoring.guru) – catalog of refactoring patterns and code smells

---

*End of Module 6 | Next: Module 7 – Software Quality Assurance, Security, and Ethical Considerations*
