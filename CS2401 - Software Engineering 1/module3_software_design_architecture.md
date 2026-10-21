# Module 3: Software Design and Architecture

**Course:** CS 2401 – Software Engineering 1
**Module:** 3 of 8

---

## Learning Objectives

By the end of this module, students will be able to:
- Understand the principles of software design (SOLID, DRY, KISS)
- Distinguish between high-level and low-level design
- Describe common architectural patterns (Layered, MVC, Microservices)
- Create basic UML diagrams (Class Diagram, Sequence Diagram)
- Apply design patterns (Singleton, Observer, Factory)

---

## 3.1 What is Software Design?

**Software Design** is the process of transforming requirements into a blueprint that guides implementation. It sits between requirements analysis and coding.

Good design produces software that is:
- **Correct** – Does what it's supposed to do
- **Modular** – Divided into manageable, independent components
- **Maintainable** – Easy to update and extend
- **Reusable** – Components can be used in other contexts
- **Testable** – Behavior can be verified in isolation

### Design Levels

| Level | Focus | Output |
|---|---|---|
| **Architectural Design** (HLD) | System structure, major components | Architecture diagram |
| **Component Design** (LLD) | Internal logic of each component | Class diagrams, pseudocode |
| **Interface Design** | APIs, data formats between components | API specs, data models |

---

## 3.2 Design Principles

### SOLID Principles

SOLID is a set of five principles for writing clean, maintainable object-oriented code.

#### S – Single Responsibility Principle
> A class should have only one reason to change.

**Bad:**
```python
class User:
    def save_to_database(self): ...
    def send_welcome_email(self): ...
    def generate_report(self): ...
```

**Good:**
```python
class User: ...
class UserRepository:
    def save(self, user: User): ...
class EmailService:
    def send_welcome(self, user: User): ...
```

#### O – Open/Closed Principle
> Software entities should be open for extension, but closed for modification.

#### L – Liskov Substitution Principle
> Subtypes must be substitutable for their base types.

#### I – Interface Segregation Principle
> Clients should not be forced to depend on interfaces they don't use.

#### D – Dependency Inversion Principle
> High-level modules should not depend on low-level modules. Both should depend on abstractions.

---

### Other Key Principles

| Principle | Meaning |
|---|---|
| **DRY** (Don't Repeat Yourself) | Avoid duplicating logic; extract shared code |
| **KISS** (Keep It Simple, Stupid) | Prefer simple solutions over complex ones |
| **YAGNI** (You Aren't Gonna Need It) | Don't build features until they are needed |
| **Separation of Concerns** | Different parts of a system handle different responsibilities |

---

## 3.3 Software Architecture Patterns

An **architectural pattern** provides a reusable solution to a recurring structural problem.

### 1. Layered (N-Tier) Architecture

The system is organized into horizontal layers, each with a distinct responsibility.

```
┌──────────────────────────────────┐
│         Presentation Layer       │  ← UI, web pages, mobile screens
├──────────────────────────────────┤
│         Business Logic Layer     │  ← Rules, calculations, workflows
├──────────────────────────────────┤
│         Data Access Layer        │  ← Database queries, ORM
├──────────────────────────────────┤
│         Database Layer           │  ← MySQL, PostgreSQL, MongoDB
└──────────────────────────────────┘
```

**Pros:** Simple, easy to understand, good separation of concerns
**Cons:** Can be slow due to many layer traversals; changes in one layer can ripple

**Best for:** Traditional enterprise applications, web apps

---

### 2. MVC (Model-View-Controller)

Separates an application into three components:

```
         User Input
              │
              ▼
        ┌──────────┐
        │Controller│  ← Handles input, coordinates
        └────┬─────┘
             │
    ┌────────┴────────┐
    ▼                 ▼
┌───────┐        ┌────────┐
│ Model │◄──────►│  View  │
│(Data) │        │  (UI)  │
└───────┘        └────────┘
```

- **Model:** Manages data and business logic
- **View:** Displays data to the user
- **Controller:** Handles user input and updates Model/View

**Examples:** Django (Python), Ruby on Rails, ASP.NET MVC

---

### 3. Microservices Architecture

The application is composed of **small, independent services**, each responsible for a specific business function.

```
                    ┌─────────────────┐
                    │   API Gateway   │
                    └────────┬────────┘
          ┌─────────┬────────┼────────┬─────────┐
          ▼         ▼        ▼        ▼         ▼
    ┌──────────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐
    │  User    │ │Order │ │Pay-  │ │Notif.│ │Inven-│
    │  Service │ │Svc   │ │ment  │ │Svc   │ │tory  │
    └──────────┘ └──────┘ └──────┘ └──────┘ └──────┘
         │           │        │        │         │
      [DB]        [DB]     [DB]     [DB]       [DB]
```

**Pros:** Independent deployment, scalability, tech flexibility
**Cons:** Complex infrastructure, network latency, harder debugging

**Examples:** Netflix, Amazon, Uber use microservices

---

### 4. Event-Driven Architecture

Components communicate through **events** rather than direct calls.

**Example:** When a user places an order → an `OrderPlaced` event is fired → Inventory Service, Payment Service, and Notification Service each react to the event independently.

---

### 5. Client-Server Architecture

Classic two-tier model: **Client** makes requests, **Server** handles them.

```
[Browser / Mobile App] ──── HTTP ────► [Web Server] ──► [Database]
```

---

## 3.4 UML Diagrams

**UML (Unified Modeling Language)** is the standard notation for visualizing software designs.

### Class Diagram

Shows the structure of a system — classes, attributes, methods, and relationships.

```
┌──────────────────────────────┐
│           Order              │
├──────────────────────────────┤
│ - orderId: int               │
│ - date: DateTime             │
│ - status: String             │
├──────────────────────────────┤
│ + placeOrder(): void         │
│ + cancelOrder(): void        │
│ + getTotal(): double         │
└──────────────────────────────┘
          │  contains
          ▼  (1 to many)
┌──────────────────────────────┐
│          OrderItem           │
├──────────────────────────────┤
│ - product: Product           │
│ - quantity: int              │
│ - price: double              │
├──────────────────────────────┤
│ + getSubtotal(): double      │
└──────────────────────────────┘
```

### Sequence Diagram

Shows how objects interact **over time** to accomplish a use case.

```
User          LoginController     AuthService      Database
  │                  │                 │               │
  │─── login(email, pwd) ─────────────►│               │
  │                  │                 │──findUser()──►│
  │                  │                 │◄── user ──────│
  │                  │                 │               │
  │                  │                 │── validatePwd()
  │                  │◄── token ───────│               │
  │◄── redirect to dashboard ─────────│               │
```

---

## 3.5 Design Patterns

**Design patterns** are proven, reusable solutions to common software design problems. They are organized into three categories:

| Category | Patterns |
|---|---|
| **Creational** | Singleton, Factory, Builder, Prototype |
| **Structural** | Adapter, Decorator, Facade, Proxy |
| **Behavioral** | Observer, Strategy, Command, Iterator |

### Singleton Pattern (Creational)

Ensures a class has **only one instance** and provides a global access point.

**Use case:** Database connection pool, configuration manager

```python
class DatabaseConnection:
    _instance = None

    @classmethod
    def get_instance(cls):
        if cls._instance is None:
            cls._instance = cls()
        return cls._instance

    def query(self, sql):
        # Execute SQL
        pass

# Usage
db = DatabaseConnection.get_instance()
db.query("SELECT * FROM users")
```

---

### Observer Pattern (Behavioral)

Defines a **one-to-many dependency** so that when one object changes state, its dependents are notified automatically.

**Use case:** Event systems, UI frameworks, notifications

```python
class EventEmitter:
    def __init__(self):
        self._observers = []

    def subscribe(self, observer):
        self._observers.append(observer)

    def notify(self, event):
        for observer in self._observers:
            observer.update(event)

class EmailNotifier:
    def update(self, event):
        print(f"Sending email for: {event}")

class SMSNotifier:
    def update(self, event):
        print(f"Sending SMS for: {event}")

# Usage
emitter = EventEmitter()
emitter.subscribe(EmailNotifier())
emitter.subscribe(SMSNotifier())
emitter.notify("OrderPlaced")
```

---

### Factory Pattern (Creational)

Defines an interface for creating objects but lets subclasses decide which class to instantiate.

```python
class NotificationFactory:
    @staticmethod
    def create(notification_type):
        if notification_type == "email":
            return EmailNotifier()
        elif notification_type == "sms":
            return SMSNotifier()
        raise ValueError("Unknown type")

# Usage
notifier = NotificationFactory.create("email")
notifier.update("Welcome!")
```

---

## 3.6 Case Study: Designing an E-Commerce Platform

**Requirements Summary:**
- Users can browse, search, and purchase products
- Admins can manage inventory and view orders
- System must support 100,000+ users

**Architectural Decision:** Layered architecture for initial release, with a path to microservices as the platform grows.

**Key Design Choices:**
1. MVC pattern for the web frontend
2. REST API for mobile clients
3. Repository pattern for database access
4. Observer pattern for order notifications (email, SMS, push)
5. Singleton for database connection pooling

---

## 3.7 Summary

- Software design transforms requirements into a blueprint for implementation.
- SOLID, DRY, and KISS are fundamental design principles.
- Architectural patterns (Layered, MVC, Microservices) provide proven structural solutions.
- UML diagrams (Class, Sequence) are the standard notation for design documentation.
- Design patterns (Singleton, Observer, Factory) solve common recurring problems.

---

## Quiz – Module 3

---

**1. Which SOLID principle states a class should have only one reason to change?**

- A) Open/Closed
- B) Single Responsibility ✅
- C) Liskov Substitution
- D) Dependency Inversion

---

**2. In MVC architecture, which component handles user input?**

- A) Model
- B) View
- C) Controller ✅
- D) Service

---

**3. Which architectural pattern is used by Netflix and Amazon for independent scalability?**

- A) Layered Architecture
- B) MVC
- C) Client-Server
- D) Microservices ✅

---

**4. A Sequence Diagram is used to show:**

- A) Class attributes and methods
- B) Interactions between objects over time ✅
- C) Database schema
- D) Deployment environments

---

**5. Which design pattern ensures only one instance of a class exists?**

- A) Factory
- B) Observer
- C) Singleton ✅
- D) Adapter

---

**6. Short Answer:** Explain the difference between High-Level Design (HLD) and Low-Level Design (LLD). Give an example of an output from each.

---

**7. Design Exercise:** You are building a ride-sharing app (like Uber). Identify:
- Which architectural pattern you would use and why
- Three design patterns that would be applicable
- Draw a simple Class Diagram with at least 3 classes (Driver, Rider, Trip)

---

## Further Reading

- Gamma et al. *Design Patterns: Elements of Reusable Object-Oriented Software* (GoF book)
- Martin, R.C. *Clean Architecture*
- Fowler, M. *Patterns of Enterprise Application Architecture*
- [UML Reference at uml.org](https://www.uml.org)

---

*End of Module 3 | Next: Module 4 – Software Development Methodologies*
