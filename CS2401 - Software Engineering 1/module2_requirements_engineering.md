# Module 2: Software Requirements Engineering and Analysis

**Course:** CS 2401 – Software Engineering 1
**Module:** 2 of 8

---

## Learning Objectives

By the end of this module, students will be able to:
- Distinguish between functional and non-functional requirements
- Write clear, verifiable user stories and use cases
- Create Use Case Diagrams
- Identify common requirements elicitation techniques
- Produce a basic Software Requirements Specification (SRS)

---

## 2.1 What Are Software Requirements?

A **software requirement** is a description of what a system should do or a constraint on how it should operate.

Requirements are the foundation of the entire software development process. Getting them wrong — or missing them — is one of the most expensive mistakes a team can make.

> "The hardest single part of building a software system is deciding precisely what to build." — Fred Brooks, *The Mythical Man-Month*

### Types of Requirements

#### Functional Requirements (FR)
Describe **what the system should do** — the behaviors, features, and functions.

**Examples:**
- The system shall allow users to register with an email and password.
- The system shall send a confirmation email after a successful purchase.
- The system shall generate a monthly sales report in PDF format.

#### Non-Functional Requirements (NFR)
Describe **how the system should perform** — quality attributes and constraints.

| Category | Example |
|---|---|
| Performance | The system shall respond to search queries in under 2 seconds |
| Security | All passwords shall be stored using bcrypt hashing |
| Availability | The system shall maintain 99.9% uptime |
| Scalability | The system shall support up to 10,000 concurrent users |
| Usability | First-time users shall complete registration in under 3 minutes |
| Maintainability | The codebase shall achieve at least 80% unit test coverage |

#### Domain Requirements
Rules and constraints from the problem domain (industry-specific regulations).

**Example:** A banking app must comply with PCI-DSS standards for credit card data.

---

## 2.2 Requirements Elicitation Techniques

Elicitation is the process of **gathering requirements** from stakeholders.

### 1. Interviews
One-on-one or group conversations with stakeholders to understand their needs.

**Tips:**
- Prepare questions in advance
- Ask both open-ended and closed questions
- Record and document responses

**Example Question:** "What are the biggest pain points with your current appointment scheduling process?"

### 2. Surveys and Questionnaires
Useful for gathering input from large groups of users.

**Best for:** Validating assumptions or collecting preferences at scale.

### 3. Workshops / Focus Groups
Bring multiple stakeholders together to collaboratively identify requirements.

### 4. Observation (Ethnographic Study)
Watch how users interact with the current system in their natural environment.

**Best for:** Uncovering implicit requirements users cannot articulate.

### 5. Document Analysis
Review existing documents, manuals, reports, and legacy systems.

### 6. Prototyping
Build a quick prototype (wireframe or mockup) so users can react to something tangible.

---

## 2.3 Writing User Stories

**User Stories** are short, informal descriptions of a feature from the user's perspective. Popularized by Agile.

### Format:

```
As a [type of user],
I want to [perform some action],
So that [I can achieve some goal].
```

### Examples:

**Story 1:**
> As a **registered customer**, I want to **view my order history**, so that **I can track past purchases**.

**Story 2:**
> As an **admin**, I want to **deactivate inactive user accounts**, so that **I can maintain system security**.

**Story 3:**
> As a **new visitor**, I want to **reset my password via email**, so that **I can regain access if I forget it**.

### Acceptance Criteria
Each story should include acceptance criteria — conditions that must be true for the story to be considered "done."

**Example (for Story 1):**
- User can see a paginated list of all past orders
- Each order shows date, items, total cost, and status
- Orders can be filtered by date range
- Page loads in under 1.5 seconds

---

## 2.4 Use Cases

A **Use Case** describes a sequence of interactions between a user (actor) and the system to achieve a specific goal.

### Use Case Template

| Field | Description |
|---|---|
| **Use Case Name** | Login to System |
| **Actor** | Registered User |
| **Precondition** | User has a valid account |
| **Main Flow** | 1. User navigates to login page; 2. User enters email and password; 3. System validates credentials; 4. System grants access and redirects to dashboard |
| **Alternative Flow** | 3a. Credentials are invalid → System shows error message and allows retry |
| **Postcondition** | User is authenticated and logged in |
| **Exception** | After 5 failed attempts, account is temporarily locked |

---

## 2.5 Use Case Diagram

Use Case Diagrams show the **actors** and their **interactions** with the system at a high level.

```
┌─────────────────────────────────────────────────┐
│              E-Commerce System                  │
│                                                 │
│  ┌────────────────┐    ┌────────────────────┐   │
│  │   Browse        │    │   Place Order      │   │
│  │   Products      │    │                    │   │
│  └──────┬─────────┘    └─────────┬──────────┘   │
│         │                        │              │
│         ▼                        ▼              │
│  ┌────────────────┐    ┌────────────────────┐   │
│  │   Search        │    │   Track Order      │   │
│  │   Products      │    │                    │   │
│  └────────────────┘    └────────────────────┘   │
│                                                 │
│  ┌────────────────────────────────────────────┐ │
│  │           Manage Inventory (Admin)          │ │
│  └────────────────────────────────────────────┘ │
└────────────────────┬────────────────────────────┘
                     │
        ┌────────────┴─────────────┐
        │                         │
    [Customer]               [Admin]
```

**Key notation:**
- **Actor** – A person or system that interacts with the system (shown outside the boundary)
- **Use Case** – An action or goal (shown as ovals inside the boundary)
- **System Boundary** – The rectangle representing the system
- **Association** – Lines connecting actors to use cases

---

## 2.6 Requirements Documentation: The SRS

The **Software Requirements Specification (SRS)** is the formal document that captures all requirements.

### Typical SRS Structure (IEEE 830 Standard)

```
1. Introduction
   1.1 Purpose
   1.2 Scope
   1.3 Definitions and Abbreviations
   1.4 References

2. Overall Description
   2.1 Product Perspective
   2.2 Product Functions
   2.3 User Classes and Characteristics
   2.4 Assumptions and Dependencies

3. Specific Requirements
   3.1 Functional Requirements
   3.2 Non-Functional Requirements
   3.3 External Interface Requirements

4. Appendices
```

### Qualities of Good Requirements

A requirement is well-written if it is:

| Quality | Meaning |
|---|---|
| **Complete** | Nothing essential is left out |
| **Unambiguous** | Can only be interpreted one way |
| **Verifiable** | Can be tested or measured |
| **Consistent** | Does not conflict with other requirements |
| **Traceable** | Can be linked to a source and to code |
| **Feasible** | Technically and economically achievable |

**Bad requirement:** "The system should be fast."
**Good requirement:** "The system shall load the home page in under 1.5 seconds on a standard broadband connection."

---

## 2.7 Case Study: Library Management System

**Scenario:** A university wants a Library Management System (LMS).

### Stakeholders
- Students (borrowers)
- Librarians (admins)
- IT Department

### Sample Requirements

**Functional:**
- FR-01: Students shall be able to search books by title, author, or ISBN.
- FR-02: Students shall be able to reserve up to 3 books at a time.
- FR-03: Librarians shall be able to add, update, and remove book records.
- FR-04: The system shall send an email reminder 2 days before a book is due.

**Non-Functional:**
- NFR-01: The system shall support up to 500 concurrent users.
- NFR-02: Book search results shall appear within 1 second.
- NFR-03: The system shall be available 24/7 with planned downtime not exceeding 2 hours/month.

---

## 2.8 Summary

- Requirements are divided into functional, non-functional, and domain requirements.
- Elicitation techniques include interviews, workshops, observation, and prototyping.
- User stories describe features from the user's perspective.
- Use cases describe detailed interaction flows between actors and the system.
- The SRS document formalizes all requirements following IEEE 830 standards.

---

## Quiz – Module 2

**Instructions:** Choose the best answer or provide a written response.

---

**1. Which of the following is a non-functional requirement?**

- A) Users shall be able to log in with an email and password
- B) The system shall send a welcome email after registration
- C) The system shall process transactions in under 500 milliseconds ✅
- D) Admins shall be able to export reports in CSV format

---

**2. What is the primary purpose of Requirements Elicitation?**

- A) Writing code based on known features
- B) Gathering needs and constraints from stakeholders ✅
- C) Testing the software against specifications
- D) Designing the system architecture

---

**3. Which user story format is correct?**

- A) "The login feature needs email and password fields."
- B) "Users should log in."
- C) "As a user, I want to log in with my email, so that I can access my dashboard." ✅
- D) "The system logs users in."

---

**4. In a Use Case Diagram, what does the rectangle represent?**

- A) An actor
- B) A use case
- C) The system boundary ✅
- D) A database

---

**5. Which quality means a requirement can only be interpreted one way?**

- A) Complete
- B) Feasible
- C) Unambiguous ✅
- D) Traceable

---

**6. Short Answer:** Rewrite this bad requirement as a good one:
> "The app should handle a lot of users."

*(Sample answer: "The application shall support up to 50,000 concurrent users with response times not exceeding 2 seconds under peak load.")*

---

**7. Practical Exercise:** You are building an online food delivery app. Write:
- 3 functional requirements
- 2 non-functional requirements
- 2 user stories with acceptance criteria

---

## Further Reading

- Sommerville, I. *Software Engineering* (10th ed.) – Chapters 4 & 5
- Wiegers, K. *Software Requirements* (3rd ed.)
- IEEE Std 830-1998 – IEEE Recommended Practice for SRS

---

*End of Module 2 | Next: Module 3 – Software Design and Architecture*
