# Module 1: Introduction to Software Engineering and Software Development Life Cycle

**Course:** CS 2401 – Software Engineering 1
**Module:** 1 of 8

---

## Learning Objectives

By the end of this module, students will be able to:
- Define software engineering and explain its importance
- Describe the major phases of the Software Development Life Cycle (SDLC)
- Compare different SDLC models
- Understand the role of a software engineer in real-world projects

---

## 1.1 What is Software Engineering?

**Software Engineering** is the systematic application of engineering principles to the design, development, testing, and maintenance of software. It bridges computer science theory with practical project delivery.

> "Software engineering is the establishment and use of sound engineering principles in order to economically obtain software that is reliable and works efficiently on real machines." — Fritz Bauer, 1969

### Why Software Engineering Matters

Without structured software engineering practices, projects tend to suffer from:

- **Cost overruns** – Projects exceeding budget due to poor estimation
- **Schedule delays** – Missed deadlines from unclear scope
- **Poor quality** – Defects that reach end users
- **Maintenance nightmares** – Code that becomes impossible to update

### Real-World Example: The Ariane 5 Rocket Failure (1996)

On June 4, 1996, the Ariane 5 rocket exploded 37 seconds after launch. The cause? A software bug — a 64-bit floating point number was converted to a 16-bit integer without proper validation, causing an overflow. The mission cost over $370 million.

**Lesson:** Software errors in critical systems can be catastrophic. This is why engineering discipline matters.

---

## 1.2 Software Development Life Cycle (SDLC)

The **SDLC** is a structured process used to plan, create, test, and deploy software. It provides a framework for managing the complexity of software projects.

### The 7 Phases of the SDLC

```
┌─────────────────────────────────────────────────────┐
│                  SDLC Phases                        │
│                                                     │
│  1. Planning  ──►  2. Requirements  ──►  3. Design  │
│                                              │      │
│  7. Maintenance  ◄──  6. Deployment  ◄──  4. Impl.  │
│                                              │      │
│                       5. Testing  ◄──────────       │
└─────────────────────────────────────────────────────┘
```

#### Phase 1 – Planning
Define the project scope, feasibility, timeline, and resources.

**Key outputs:** Project charter, feasibility report, resource plan

#### Phase 2 – Requirements Analysis
Gather and document what the software must do (functional) and how well it must do it (non-functional).

**Key outputs:** Software Requirements Specification (SRS)

#### Phase 3 – System Design
Translate requirements into a blueprint for the system architecture, database, UI, and components.

**Key outputs:** High-level design (HLD), Low-level design (LLD)

#### Phase 4 – Implementation (Coding)
Developers write code according to the design documents.

**Key outputs:** Source code, unit tests

#### Phase 5 – Testing
Verify the software works as expected. Find and fix defects.

**Key outputs:** Test reports, defect logs

#### Phase 6 – Deployment
Release the product to users, either all at once or in stages.

**Key outputs:** Deployed system, release notes

#### Phase 7 – Maintenance
Fix bugs discovered in production, improve performance, and add new features over time.

**Key outputs:** Patches, updated documentation

---

## 1.3 SDLC Models

Different project contexts call for different SDLC models. Here are the most common:

### Waterfall Model

A **sequential** model where each phase must be completed before the next begins.

```
Planning → Requirements → Design → Implementation → Testing → Deployment → Maintenance
```

**Pros:**
- Simple, easy to understand
- Works well for small projects with clear requirements

**Cons:**
- Inflexible — changes are expensive
- Testing happens late; bugs found late are costly

**Best for:** Projects with fixed, well-understood requirements (e.g., government contracts)

---

### V-Model (Verification & Validation)

An extension of Waterfall where each development phase has a corresponding testing phase.

```
Requirements ──────────────────► Acceptance Testing
  System Design ────────────► System Testing
    HLD ──────────────────► Integration Testing
      LLD ────────────► Unit Testing
           Coding (bottom of V)
```

**Best for:** Safety-critical systems (medical devices, aviation software)

---

### Iterative Model

Software is developed in **repeated cycles (iterations)**, each adding more features.

```
Iteration 1: Plan → Build → Test → Review
Iteration 2: Plan → Build → Test → Review (adds more features)
...
```

**Best for:** Projects where requirements evolve over time

---

### Spiral Model

Combines iterative development with systematic risk analysis.

Each loop of the spiral has four quadrants:
1. **Determine objectives**
2. **Identify and resolve risks**
3. **Development and testing**
4. **Plan next iteration**

**Best for:** Large, complex, high-risk projects

---

### Agile Model

An **iterative and incremental** approach focused on collaboration, flexibility, and delivering working software frequently.

**Core values (Agile Manifesto):**
- Individuals and interactions > Processes and tools
- Working software > Comprehensive documentation
- Customer collaboration > Contract negotiation
- Responding to change > Following a plan

**Best for:** Dynamic projects with evolving requirements (most modern web/mobile apps)

---

## 1.4 Roles in a Software Engineering Team

| Role | Responsibilities |
|---|---|
| Project Manager | Plans, schedules, and oversees the project |
| Business Analyst | Gathers and documents requirements |
| Software Architect | Designs high-level system structure |
| Developer (Frontend/Backend) | Writes code |
| QA Engineer | Tests the software |
| DevOps Engineer | Manages infrastructure and CI/CD pipelines |
| UX Designer | Designs user interfaces and experiences |

---

## 1.5 Real-World Case Study: Healthcare App Development

**Scenario:** A hospital wants a mobile app to let patients schedule appointments.

| SDLC Phase | Activity |
|---|---|
| Planning | Define scope: book/cancel appointments, view records |
| Requirements | Gather needs from doctors, nurses, and patients |
| Design | Design database schema, UI wireframes, API structure |
| Implementation | Build the React Native app + Node.js backend |
| Testing | Test booking flows, security, load under 1,000 concurrent users |
| Deployment | Release to App Store and Google Play |
| Maintenance | Fix login bug reported by users in week 2 |

---

## 1.6 Summary

- Software engineering applies systematic principles to build reliable, high-quality software.
- The SDLC provides a structured framework with 7 key phases.
- Different SDLC models (Waterfall, Agile, Spiral, etc.) suit different project needs.
- Software failures have real-world consequences — from lost revenue to loss of life.

---

## Quiz – Module 1

**Instructions:** Choose the best answer for each question.

---

**1. Which SDLC model is best suited for projects with well-defined, fixed requirements?**

- A) Agile
- B) Spiral
- C) Waterfall ✅
- D) Iterative

---

**2. In the V-Model, what does each development phase have?**

- A) A parallel marketing phase
- B) A corresponding testing phase ✅
- C) A corresponding deployment phase
- D) An independent risk analysis phase

---

**3. What was the root cause of the Ariane 5 rocket failure?**

- A) Hardware malfunction
- B) Poor project management
- C) A software integer overflow bug ✅
- D) Incorrect requirements

---

**4. Which Agile Manifesto value is correct?**

- A) Processes and tools over individuals
- B) Contract negotiation over customer collaboration
- C) Working software over comprehensive documentation ✅
- D) Following a plan over responding to change

---

**5. Which SDLC phase produces the Software Requirements Specification (SRS)?**

- A) Planning
- B) Design
- C) Requirements Analysis ✅
- D) Testing

---

**6. Short Answer:** Explain in your own words why software engineering is necessary. Give one example of what can go wrong without it.

---

**7. Short Answer:** Compare Waterfall and Agile. When would you choose one over the other?

---

## Further Reading

- Pressman, R.S. *Software Engineering: A Practitioner's Approach* (8th ed.) – Chapter 1
- Sommerville, I. *Software Engineering* (10th ed.) – Chapter 1
- [Agile Manifesto](https://agilemanifesto.org)
- IEEE Standard 12207 – Software Life Cycle Processes

---

*End of Module 1 | Next: Module 2 – Software Requirements Engineering and Analysis*
