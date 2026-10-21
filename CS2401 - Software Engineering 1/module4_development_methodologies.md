# Module 4: Software Development Methodologies

**Course:** CS 2401 – Software Engineering 1
**Module:** 4 of 8

---

## Learning Objectives

By the end of this module, students will be able to:
- Describe traditional vs. agile methodologies
- Explain Scrum roles, events, and artifacts
- Understand Kanban boards and WIP limits
- Apply XP practices (TDD, pair programming, CI)
- Choose the right methodology for a given project context

---

## 4.1 Overview of Development Methodologies

A **software development methodology** is a structured process that guides how a team plans, builds, and delivers software. Choosing the right methodology can be the difference between a successful project and a failed one.

### Traditional vs. Agile

```
TRADITIONAL (Plan-Driven)          AGILE (Value-Driven)
────────────────────────────────   ────────────────────────────────
Requirements fixed upfront          Requirements evolve
Sequential phases                   Iterative and incremental
Heavy documentation                 Working software focus
Change is expensive                 Change is expected and welcomed
Delivery at the end                 Frequent deliveries
Low customer involvement            Continuous customer collaboration
```

---

## 4.2 Waterfall Methodology (Revisited in Context)

Covered in Module 1, but in practice:

**When teams try to use Waterfall for dynamic projects:**
- Scope creep occurs after requirements are "frozen"
- Developers discover design flaws only during implementation
- QA finds critical bugs months after the code was written

**The result:** The "Big Bang" delivery — a large release at the end with a high chance of missing the mark.

---

## 4.3 Agile Manifesto and Principles

Published in 2001 by 17 software practitioners, the Agile Manifesto defines 4 values and 12 principles.

### 12 Agile Principles (Summary)

1. Satisfy the customer through early and continuous delivery
2. Welcome changing requirements, even late in development
3. Deliver working software frequently (weeks, not months)
4. Business people and developers work together daily
5. Build projects around motivated individuals
6. Face-to-face conversation is the most efficient communication
7. Working software is the primary measure of progress
8. Agile processes promote sustainable development
9. Continuous attention to technical excellence
10. Simplicity — the art of maximizing work NOT done
11. Best architectures emerge from self-organizing teams
12. Teams regularly reflect and adjust their behavior

---

## 4.4 Scrum Framework

**Scrum** is the most widely adopted Agile framework. It organizes work into **Sprints** — fixed-length iterations (usually 2 weeks).

### Scrum Roles

| Role | Responsibilities |
|---|---|
| **Product Owner (PO)** | Owns the Product Backlog; prioritizes features; represents the customer |
| **Scrum Master (SM)** | Facilitates Scrum events; removes impediments; coaches the team |
| **Development Team** | Self-organizing; builds the product increment (typically 3–9 people) |

---

### Scrum Artifacts

| Artifact | Description |
|---|---|
| **Product Backlog** | Ordered list of all desired features (user stories, bugs, tasks) |
| **Sprint Backlog** | Items selected for the current Sprint + plan to deliver them |
| **Increment** | The working, potentially shippable product at the end of each Sprint |

---

### Scrum Events

```
┌─────────────────────────────────────────────────────────────────┐
│                        SCRUM SPRINT (2 weeks)                    │
│                                                                  │
│  Sprint       Daily      Sprint       Sprint      Sprint         │
│  Planning  →  Scrum   →  Review   →  Retro    →  (next begins)  │
│  (Day 1)     (Daily)    (Last day)  (Last day)                   │
│                                                                  │
│  [Product Backlog] → [Sprint Backlog] → [Increment]              │
└─────────────────────────────────────────────────────────────────┘
```

| Event | Duration | Purpose |
|---|---|---|
| **Sprint Planning** | Up to 8 hrs | Team selects backlog items and creates Sprint plan |
| **Daily Scrum** | 15 min | Daily sync on progress, plans, and blockers |
| **Sprint Review** | Up to 4 hrs | Demo the increment to stakeholders; gather feedback |
| **Sprint Retrospective** | Up to 3 hrs | Team reflects: what went well, what to improve |

---

### Scrum in Practice: Example Sprint

**Team:** 5 developers, 1 Scrum Master, 1 Product Owner
**Sprint Goal:** "Users can register and log in to the platform"

**Sprint Backlog:**
- User Story 1: Register with email/password (8 points)
- User Story 2: Email verification (5 points)
- User Story 3: Login with JWT tokens (8 points)
- User Story 4: Password reset flow (5 points)

**Daily Scrum (Example):**
> Maria: "Yesterday I finished the registration API. Today I'll start email verification. No blockers."
> Carlos: "Yesterday I set up the JWT library. Today I'll implement the login endpoint. I'm blocked — need DB credentials for the dev environment."
> Scrum Master: "I'll get you those credentials by 9:30 AM, Carlos."

---

### Story Points and Velocity

**Story Points** are a relative measure of effort (complexity + uncertainty + work).

**Fibonacci scale commonly used:** 1, 2, 3, 5, 8, 13, 21

**Velocity** = Total story points completed per Sprint. Used to forecast future Sprints.

---

## 4.5 Kanban

**Kanban** is a visual workflow management method focused on limiting Work In Progress (WIP) to improve flow.

### Kanban Board

```
┌────────────┬──────────────┬─────────────┬──────────┐
│  Backlog   │  In Progress │   Review    │   Done   │
│            │   WIP: max 3 │  WIP: max 2 │          │
├────────────┼──────────────┼─────────────┼──────────┤
│ Task A     │ Task C       │ Task E      │ Task G   │
│ Task B     │ Task D       │             │ Task H   │
│ Task F     │              │             │ Task I   │
│ Task J     │              │             │          │
└────────────┴──────────────┴─────────────┴──────────┘
```

### WIP Limits
WIP limits prevent the team from starting too many tasks at once, which causes context switching and delays.

**Rule:** If the In Progress column is at its WIP limit, no new work can be pulled until something moves forward.

### Scrum vs. Kanban

| Feature | Scrum | Kanban |
|---|---|---|
| Cadence | Fixed Sprints | Continuous flow |
| Roles | PO, SM, Dev Team | No prescribed roles |
| WIP Limits | Per Sprint | Per column |
| Changes mid-cycle | Not during Sprint | Anytime |
| Best for | New product development | Maintenance, support teams |

---

## 4.6 Extreme Programming (XP)

**XP** is an Agile methodology that focuses on **engineering practices** for high-quality code.

### Core XP Practices

#### 1. Test-Driven Development (TDD)
Write the test **before** the code.

```
RED → GREEN → REFACTOR

1. Write a failing test (RED)
2. Write the minimum code to pass (GREEN)
3. Improve the code without breaking tests (REFACTOR)
```

**Example:**
```python
# Step 1: Write the test first
def test_add():
    assert add(2, 3) == 5     # Fails — function doesn't exist yet

# Step 2: Write the function to pass the test
def add(a, b):
    return a + b              # Test now passes

# Step 3: Refactor if needed
```

#### 2. Pair Programming
Two developers work together at one workstation. One writes code (Driver), the other reviews (Navigator), swapping roles frequently.

**Benefits:**
- Catches bugs in real time
- Knowledge sharing
- Better code quality

#### 3. Continuous Integration (CI)
Developers integrate their code into the shared repository **multiple times per day**. Automated tests run on every commit.

#### 4. Refactoring
Continuously improving code structure without changing behavior.

#### 5. Simple Design
Build only what is needed now. No over-engineering.

---

## 4.7 SAFe, LeSS, and Scaled Agile

For large organizations with 50–1000+ developers, scaling frameworks exist:

| Framework | Description |
|---|---|
| **SAFe** (Scaled Agile Framework) | Organizes teams into Agile Release Trains (ARTs); widely used in enterprises |
| **LeSS** (Large Scale Scrum) | Applies Scrum principles at scale with minimal additional process |
| **Nexus** | Scrum.org's scaling framework; integrates 3–9 Scrum teams |

---

## 4.8 Choosing the Right Methodology

| Project Characteristics | Recommended Methodology |
|---|---|
| Fixed requirements, regulated industry | Waterfall or V-Model |
| Evolving requirements, fast delivery | Scrum or Kanban |
| High engineering quality critical | XP |
| Maintenance/support team | Kanban |
| Large enterprise, multiple teams | SAFe or LeSS |
| Mixed (research + product) | Spiral |

---

## 4.9 Case Study: Startup Building a Mobile App

**Context:** 6-person startup, building a social fitness tracking app. Requirements will evolve based on user feedback.

**Methodology chosen:** Scrum

**Why:**
- Small, co-located team
- Need to ship features quickly and adjust based on feedback
- Product Owner can prioritize the backlog weekly

**Sprint 1 Goal:** "Users can create profiles and log workouts"

**Outcome after 4 Sprints:** A working MVP (Minimum Viable Product) is released to 200 beta users. Feedback drives the Sprint 5 backlog.

---

## 4.10 Summary

- Agile methodologies embrace change, deliver frequently, and prioritize working software.
- Scrum uses Sprints, defined roles, and regular ceremonies to manage iterative development.
- Kanban visualizes workflow and uses WIP limits for continuous delivery.
- XP focuses on technical practices: TDD, pair programming, CI, and simple design.
- The right methodology depends on team size, project type, and requirements stability.

---

## Quiz – Module 4

---

**1. In Scrum, who is responsible for prioritizing the Product Backlog?**

- A) Scrum Master
- B) Development Team
- C) Product Owner ✅
- D) Project Manager

---

**2. What is the purpose of WIP limits in Kanban?**

- A) Estimate story points
- B) Define sprint length
- C) Prevent too many tasks from being in progress simultaneously ✅
- D) Assign tasks to team members

---

**3. In Test-Driven Development, what is the correct order?**

- A) Code → Test → Refactor
- B) Test → Refactor → Code
- C) Test → Code → Refactor ✅
- D) Code → Refactor → Test

---

**4. Which Scrum event is a daily 15-minute synchronization meeting?**

- A) Sprint Planning
- B) Sprint Review
- C) Sprint Retrospective
- D) Daily Scrum ✅

---

**5. Which methodology is best for a team doing ongoing maintenance and support?**

- A) Scrum
- B) Waterfall
- C) Kanban ✅
- D) Spiral

---

**6. Short Answer:** Describe the three Scrum roles and their primary responsibilities.

---

**7. Practical Exercise:** Your team is building a banking compliance system for a government client. Requirements are strict and fully defined upfront. Which methodology would you choose and why? What are the trade-offs?

---

## Further Reading

- Schwaber, K. & Sutherland, J. *The Scrum Guide* (free at scrumguides.org)
- Beck, K. *Extreme Programming Explained* (2nd ed.)
- Anderson, D.J. *Kanban: Successful Evolutionary Change for Your Technology Business*
- [Agile Alliance Resources](https://www.agilealliance.org)

---

*End of Module 4 | Next: Module 5 – Software Testing and Quality Assurance*
