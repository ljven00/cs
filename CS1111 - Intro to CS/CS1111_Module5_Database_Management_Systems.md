# CS 1111 – Introduction to Computer Science
## Module 5: Overview of Database Management Systems

---

## Table of Contents
1. [What is a Database?](#1-what-is-a-database)
2. [History and Evolution of Databases](#2-history-and-evolution-of-databases)
3. [The Database Management System (DBMS)](#3-the-database-management-system-dbms)
4. [The Relational Database Model](#4-the-relational-database-model)
5. [Structured Query Language (SQL)](#5-structured-query-language-sql)
6. [Database Design](#6-database-design)
7. [The ACID Properties](#7-the-acid-properties)
8. [Types of Database Models](#8-types-of-database-models)
9. [NoSQL Databases](#9-nosql-databases)
10. [Popular DBMS Software](#10-popular-dbms-software)
11. [Database Security](#11-database-security)
12. [Module Summary](#12-module-summary)
13. [Module Quiz](#13-module-quiz)

---

## 1. What is a Database?

A **database** is an organized, structured collection of related data stored electronically so that it can be easily accessed, managed, and updated.

Data by itself is raw facts — numbers, words, dates. A database gives that data **structure and context**, turning it into useful **information**.

> **Example:** The number `8834521` on its own is just data. But stored in a database alongside `Juan García`, `Monterrey`, and `Active`, it becomes meaningful information: a customer record.

### Database vs. Flat File Storage

Before databases, organizations stored data in **flat files** — simple text or spreadsheet files. Consider a university storing student records in a spreadsheet:

| Problem | Flat File Issue | Database Solution |
|---|---|---|
| **Redundancy** | Student address copied in every course enrollment row | Address stored once; linked by Student ID |
| **Inconsistency** | Address updated in one row but not others | Update once; all references reflect the change |
| **No concurrent access** | Two staff editing the same file = data corruption | DBMS handles multiple simultaneous users safely |
| **No security** | Anyone with file access sees everything | DBMS enforces per-user permissions |
| **No integrity** | Nothing stops entering age = -5 | DBMS enforces validation rules (constraints) |

A **Database Management System (DBMS)** solves all of these problems.

---

## 2. History and Evolution of Databases

### 2.1 The 1950s–1960s: File-Based Systems

Data was stored in sequential files on magnetic tapes. Programs were tightly coupled to their data files — changing the data format required rewriting all programs that used it. There was no separation between data and application logic.

### 2.2 The 1960s–1970s: Hierarchical and Network Models

The first formal database models emerged to add structure to data:

- **Hierarchical Model (IBM IMS, 1966):** Data organized as a tree — one parent, many children. Fast for known access paths but rigid; many real-world relationships don't fit a strict hierarchy.
- **Network Model (CODASYL standard):** Extended hierarchical model to allow records to have multiple parents. More flexible but complex to navigate — programmers had to know the physical structure of the data.

### 2.3 The 1970s: The Relational Model

In 1970, **Edgar F. Codd** (IBM researcher) published a landmark paper introducing the **relational model** — organizing data into tables (relations) and using mathematical set theory (relational algebra) to query it.

Key insight: **data should be accessed by its content, not its physical location.** Users describe *what* data they want, not *how* to find it.

This model became the dominant paradigm for the next 50+ years.

### 2.4 The 1970s–1980s: SQL and Commercial RDBMS

- **SQL (Structured Query Language)** was developed at IBM (1974) as the standard language for relational databases.
- **Oracle** (1979) — first commercially available RDBMS.
- **IBM DB2** (1983), **Microsoft SQL Server** (1989), **MySQL** (1995) followed.

### 2.5 The 2000s–Present: NoSQL and NewSQL

The explosion of the internet created new data challenges — massive scale, unstructured data (social media posts, images, logs), and need for extreme horizontal scalability. Traditional relational databases struggled.

- **NoSQL databases** (MongoDB, Cassandra, Redis) emerged to handle these use cases with flexible schemas and distributed architecture.
- **NewSQL** (Google Spanner, CockroachDB) attempts to combine the scalability of NoSQL with the ACID guarantees of relational databases.
- **Cloud databases** (Amazon RDS, Google Cloud SQL, Azure SQL) made database hosting managed services.

---

## 3. The Database Management System (DBMS)

A **Database Management System (DBMS)** is the software that manages a database — it sits between the database and the users/applications, handling all interactions with the data.

```
┌─────────────────────────────────────┐
│         Users / Applications        │
│  (Web App, Mobile App, Reports...)  │
└──────────────┬──────────────────────┘
               │  Queries & Commands
               ↓
┌─────────────────────────────────────┐
│        DBMS Software                │
│  ┌──────────┐  ┌──────────────────┐ │
│  │  Query   │  │  Transaction     │ │
│  │Processor │  │    Manager       │ │
│  └──────────┘  └──────────────────┘ │
│  ┌──────────┐  ┌──────────────────┐ │
│  │  Storage │  │  Security &      │ │
│  │  Engine  │  │  Access Control  │ │
│  └──────────┘  └──────────────────┘ │
└──────────────┬──────────────────────┘
               │
               ↓
┌─────────────────────────────────────┐
│         Physical Database           │
│    (Files on Disk / SSD / Cloud)    │
└─────────────────────────────────────┘
```

### Core Functions of a DBMS

| Function | Description |
|---|---|
| **Data Definition** | Define the structure (schema) of the database — tables, columns, types, constraints |
| **Data Manipulation** | Insert, update, delete, and retrieve data |
| **Data Storage** | Physically manage how data is stored and retrieved efficiently |
| **Transaction Management** | Ensure groups of operations either all succeed or all fail (ACID) |
| **Concurrency Control** | Allow multiple users to access data simultaneously without conflicts |
| **Security** | Authenticate users and enforce access permissions |
| **Backup & Recovery** | Protect data against loss; restore from failures |
| **Data Integrity** | Enforce rules that keep data accurate and consistent |

### The Three-Schema Architecture

Modern DBMS separates the database into three levels:

```
External Level   →  [ View A ] [ View B ] [ View C ]
                         (what different users see)
                              ↓
Conceptual Level →  [ Full Logical Schema ]
                         (complete logical structure)
                              ↓
Internal Level   →  [ Physical Storage Structure ]
                         (how data is actually stored on disk)
```

- **External Level:** Different users/apps see different customized views of the data.
- **Conceptual Level:** The complete logical description of all data — tables, relationships, constraints.
- **Internal Level:** How data is physically stored — file organization, indexes, storage paths.

This separation means physical storage changes don't require rewriting applications, and different users can see only the parts of the data they need.

---

## 4. The Relational Database Model

The **relational model** is the most widely used database model. Data is organized into **tables (relations)**, and relationships between data are represented through shared values (keys).

### 4.1 Tables, Rows, and Columns

| Term | Also Called | Description |
|---|---|---|
| **Table** | Relation | A collection of related data organized in rows and columns |
| **Row** | Record / Tuple | A single entry in a table — one instance of the entity |
| **Column** | Attribute / Field | A named property of the entity; holds one type of data |
| **Cell** | — | The intersection of a row and a column — holds one value |

**Example — `Students` Table:**

| student_id | first_name | last_name | email                    | enrollment_date |
|---|---|---|---|---|
| 1001       | Ana        | Martínez  | ana.m@university.edu     | 2024-08-20      |
| 1002       | Carlos     | López     | c.lopez@university.edu   | 2024-08-20      |
| 1003       | Sofía      | Herrera   | s.herrera@university.edu | 2023-08-19      |

- This table has **3 rows** (records) and **5 columns** (attributes).
- Each row represents one student.
- Each column holds one type of information about students.

### 4.2 Data Types

Each column has a defined **data type** that constrains what values it can hold:

| Data Type | Description | Example |
|---|---|---|
| `INT` / `INTEGER` | Whole numbers | `1001`, `-5`, `0` |
| `DECIMAL(p,s)` | Exact decimal numbers | `19.99`, `3.14` |
| `VARCHAR(n)` | Variable-length text, max n chars | `'Ana'`, `'Monterrey'` |
| `CHAR(n)` | Fixed-length text | `'MX'` (country code) |
| `DATE` | Calendar date | `2024-08-20` |
| `DATETIME` | Date and time | `2024-08-20 09:30:00` |
| `BOOLEAN` | True or False | `TRUE`, `FALSE` |
| `TEXT` | Large text blocks | Long descriptions |
| `BLOB` | Binary large objects | Images, files |

### 4.3 Keys

**Keys** are the mechanism that uniquely identifies rows and establishes relationships between tables.

#### Primary Key (PK)
A column (or combination of columns) that **uniquely identifies each row** in a table. Rules:
- Must be **unique** — no two rows can have the same primary key value.
- Must be **NOT NULL** — a primary key can never be empty.

> In the Students table, `student_id` is the primary key. No two students share the same ID.

#### Foreign Key (FK)
A column in one table that **references the primary key of another table**, creating a link between them. This enforces **referential integrity** — you cannot reference a student that doesn't exist.

**Example — `Enrollments` Table:**

| enrollment_id | student_id | course_id | grade |
|---|---|---|---|
| 5001          | 1001       | CS1111    | A     |
| 5002          | 1002       | CS1111    | B+    |
| 5003          | 1001       | MATH101   | A-    |

Here, `student_id` is a **foreign key** referencing the `Students` table. `course_id` is a **foreign key** referencing a `Courses` table.

#### Other Key Types

| Key Type | Description |
|---|---|
| **Candidate Key** | Any column that could serve as a primary key (unique + not null) |
| **Composite Key** | A primary key made of two or more columns combined |
| **Unique Key** | Like a primary key but allows one NULL value |
| **Index** | Not strictly a key, but speeds up searches on a column |

### 4.4 Relationships Between Tables

In a relational database, tables are connected through relationships:

#### One-to-One (1:1)
Each record in Table A relates to exactly one record in Table B.
> **Example:** Each employee has exactly one parking spot; each parking spot belongs to exactly one employee.

#### One-to-Many (1:N) — Most Common
One record in Table A can relate to many records in Table B, but each record in B relates to only one in A.
> **Example:** One student can have many enrollments, but each enrollment belongs to one student.

```
Students (1) ──────< Enrollments (Many)
```

#### Many-to-Many (M:N)
Records in Table A can relate to many records in Table B, and vice versa. Implemented using a **junction table** (also called a bridge or associative table).
> **Example:** Students can enroll in many courses; each course can have many students.

```
Students (Many) >──── Enrollments ────< Courses (Many)
                    (junction table)
```

The `Enrollments` table above is that junction table — it breaks the many-to-many into two one-to-many relationships.

---

## 5. Structured Query Language (SQL)

**SQL (Structured Query Language)** is the standard language for interacting with relational databases. It is declarative — you describe *what* you want, and the database figures out *how* to get it.

SQL is divided into sub-languages:

| Sub-language | Full Name | Purpose | Commands |
|---|---|---|---|
| **DDL** | Data Definition Language | Define/modify database structure | `CREATE`, `ALTER`, `DROP` |
| **DML** | Data Manipulation Language | Add, change, remove data | `INSERT`, `UPDATE`, `DELETE` |
| **DQL** | Data Query Language | Retrieve data | `SELECT` |
| **DCL** | Data Control Language | Manage permissions | `GRANT`, `REVOKE` |
| **TCL** | Transaction Control Language | Manage transactions | `COMMIT`, `ROLLBACK` |

---

### 5.1 DDL – Creating Structure

```sql
-- Create the Students table
CREATE TABLE Students (
    student_id      INT           PRIMARY KEY,
    first_name      VARCHAR(50)   NOT NULL,
    last_name       VARCHAR(50)   NOT NULL,
    email           VARCHAR(100)  UNIQUE NOT NULL,
    enrollment_date DATE          DEFAULT CURRENT_DATE
);

-- Create the Courses table
CREATE TABLE Courses (
    course_id   CHAR(10)     PRIMARY KEY,
    title       VARCHAR(100) NOT NULL,
    credits     INT          NOT NULL,
    department  VARCHAR(50)
);

-- Create the Enrollments junction table
CREATE TABLE Enrollments (
    enrollment_id INT  PRIMARY KEY,
    student_id    INT  NOT NULL,
    course_id     CHAR(10) NOT NULL,
    grade         CHAR(2),
    FOREIGN KEY (student_id) REFERENCES Students(student_id),
    FOREIGN KEY (course_id)  REFERENCES Courses(course_id)
);
```

```sql
-- Add a new column to an existing table
ALTER TABLE Students ADD COLUMN phone VARCHAR(15);

-- Delete an entire table (irreversible!)
DROP TABLE Enrollments;
```

---

### 5.2 DML – Manipulating Data

```sql
-- Insert a new student
INSERT INTO Students (student_id, first_name, last_name, email, enrollment_date)
VALUES (1001, 'Ana', 'Martínez', 'ana.m@university.edu', '2024-08-20');

-- Insert multiple rows at once
INSERT INTO Students (student_id, first_name, last_name, email, enrollment_date)
VALUES
    (1002, 'Carlos', 'López',   'c.lopez@university.edu',   '2024-08-20'),
    (1003, 'Sofía',  'Herrera', 's.herrera@university.edu', '2023-08-19');

-- Update a record
UPDATE Students
SET email = 'ana.martinez@university.edu'
WHERE student_id = 1001;

-- Delete a record
DELETE FROM Students
WHERE student_id = 1003;
```

> **Warning:** `DELETE FROM Students;` with no `WHERE` clause deletes **every row** in the table. Always double-check your WHERE clause before running DELETE or UPDATE.

---

### 5.3 DQL – Querying Data (SELECT)

The `SELECT` statement is the most powerful and frequently used SQL command.

**Basic syntax:**
```sql
SELECT  column1, column2, ...     -- What columns to show
FROM    table_name                -- Which table
WHERE   condition                 -- Filter rows (optional)
ORDER BY column ASC|DESC          -- Sort results (optional)
LIMIT   n;                        -- Maximum rows to return (optional)
```

**Examples:**

```sql
-- Retrieve all columns from Students
SELECT * FROM Students;

-- Retrieve only specific columns
SELECT first_name, last_name, email FROM Students;

-- Filter with WHERE
SELECT * FROM Students
WHERE enrollment_date >= '2024-01-01';

-- Sort results alphabetically by last name
SELECT first_name, last_name
FROM Students
ORDER BY last_name ASC;

-- Count the number of students
SELECT COUNT(*) AS total_students FROM Students;

-- Find students whose last name starts with 'M'
SELECT * FROM Students
WHERE last_name LIKE 'M%';
```

---

### 5.4 Aggregate Functions

```sql
-- Total number of enrollments
SELECT COUNT(*) FROM Enrollments;

-- Average, min, max (e.g., if we had a numeric grade column)
SELECT
    AVG(gpa)  AS average_gpa,
    MAX(gpa)  AS highest_gpa,
    MIN(gpa)  AS lowest_gpa,
    SUM(credits) AS total_credits
FROM Students;
```

---

### 5.5 GROUP BY and HAVING

```sql
-- Count how many students enrolled per year
SELECT
    YEAR(enrollment_date) AS year,
    COUNT(*)              AS students_enrolled
FROM Students
GROUP BY YEAR(enrollment_date);

-- Show only years with more than 50 enrollments
SELECT
    YEAR(enrollment_date) AS year,
    COUNT(*)              AS students_enrolled
FROM Students
GROUP BY YEAR(enrollment_date)
HAVING COUNT(*) > 50;
```

> **Rule:** `WHERE` filters individual rows *before* grouping. `HAVING` filters groups *after* aggregation.

---

### 5.6 JOIN – Combining Tables

A **JOIN** combines rows from two or more tables based on a related column.

**Types of JOINs:**

```
Table A:  [1, 2, 3, 4]
Table B:  [2, 3, 4, 5]

INNER JOIN  → [2, 3, 4]        (matching rows only)
LEFT JOIN   → [1, 2, 3, 4]     (all of A, matched B or NULL)
RIGHT JOIN  → [2, 3, 4, 5]     (all of B, matched A or NULL)
FULL JOIN   → [1, 2, 3, 4, 5]  (all rows from both)
```

**INNER JOIN — Show each student's enrolled course names:**

```sql
SELECT
    s.first_name,
    s.last_name,
    c.title     AS course_title,
    e.grade
FROM Enrollments e
INNER JOIN Students  s ON e.student_id = s.student_id
INNER JOIN Courses   c ON e.course_id  = c.course_id
ORDER BY s.last_name;
```

**Result:**

| first_name | last_name | course_title                       | grade |
|---|---|---|---|
| Ana        | Martínez  | Introduction to Computer Science   | A     |
| Ana        | Martínez  | Calculus I                         | A-    |
| Carlos     | López     | Introduction to Computer Science   | B+    |

**LEFT JOIN — Show all students, even those not enrolled in anything:**

```sql
SELECT
    s.first_name,
    s.last_name,
    c.title AS course_title
FROM Students s
LEFT JOIN Enrollments e ON s.student_id = e.student_id
LEFT JOIN Courses     c ON e.course_id  = c.course_id;
```

A student with no enrollments will appear with `NULL` in the `course_title` column.

---

## 6. Database Design

Good database design is critical for performance, integrity, and maintainability. The process involves **Entity-Relationship (ER) modeling** and **normalization**.

### 6.1 Entity-Relationship (ER) Model

An **ER diagram** is a visual blueprint of a database, showing:
- **Entities:** Things in the real world (Students, Courses, Professors).
- **Attributes:** Properties of entities (student_id, name, email).
- **Relationships:** How entities relate to each other (Students *enroll in* Courses).

**Notation:**
- Rectangles = Entities
- Ellipses = Attributes
- Diamonds = Relationships
- Lines with crow's foot / numbers = Cardinality (1:1, 1:N, M:N)

**Example ER Diagram (text representation):**
```
[Student] ────enrolls in──── [Course]
    |                            |
(student_id)              (course_id)
(first_name)              (title)
(last_name)               (credits)
(email)

Cardinality: Student (M) ────── (N) Course
```

### 6.2 Normalization

**Normalization** is the process of organizing a database to reduce **data redundancy** and improve **data integrity** by dividing large tables into smaller, related ones and defining relationships between them.

It is achieved through a series of **Normal Forms (NF)**.

---

#### First Normal Form (1NF)
**Rule:** Every cell must contain a single, atomic (indivisible) value. No repeating groups.

**Violation:**

| student_id | name | courses               |
|---|---|---|
| 1001       | Ana  | CS1111, MATH101, ENG1 |

*Problem:* The `courses` column holds multiple values.

**Fixed (1NF):**

| student_id | name | course  |
|---|---|---|
| 1001       | Ana  | CS1111  |
| 1001       | Ana  | MATH101 |
| 1001       | Ana  | ENG1    |

---

#### Second Normal Form (2NF)
**Rule:** Must be in 1NF, and every non-key column must depend on the **entire** primary key (no partial dependencies). Applies when the primary key is composite.

**Violation (composite key: student_id + course_id):**

| student_id | course_id | grade | course_title              |
|---|---|---|---|
| 1001       | CS1111    | A     | Intro to Computer Science |

*Problem:* `course_title` depends only on `course_id`, not the full composite key.

**Fixed (2NF):** Split into separate `Enrollments` and `Courses` tables.

---

#### Third Normal Form (3NF)
**Rule:** Must be in 2NF, and no non-key column should depend on another non-key column (no transitive dependencies).

**Violation:**

| student_id | zip_code | city       | state |
|---|---|---|---|
| 1001       | 64000    | Monterrey  | NL    |

*Problem:* `city` and `state` depend on `zip_code`, not on `student_id`.

**Fixed (3NF):** Move `zip_code → city, state` to a separate `ZipCodes` table.

---

**Summary of Normal Forms:**

| Normal Form | Eliminates |
|---|---|
| **1NF** | Repeating groups; multi-valued cells |
| **2NF** | Partial dependencies on composite keys |
| **3NF** | Transitive dependencies (non-key → non-key) |
| **BCNF** | Anomalies 3NF doesn't fully handle |
| **4NF, 5NF** | Multi-valued and join dependencies (advanced) |

> **Design goal:** Most production databases aim for **3NF** as the practical sweet spot between eliminating redundancy and maintaining query performance.

---

## 7. The ACID Properties

When multiple users access a database simultaneously, or when the system crashes mid-operation, data can become corrupted or inconsistent. **Transactions** and the **ACID properties** prevent this.

A **transaction** is a sequence of one or more SQL operations treated as a single logical unit of work. Either all operations succeed, or none do.

> **Classic example — Bank Transfer:**
> Transfer $500 from Account A to Account B:
> 1. `UPDATE Accounts SET balance = balance - 500 WHERE id = 'A';`
> 2. `UPDATE Accounts SET balance = balance + 500 WHERE id = 'B';`
>
> If the system crashes between step 1 and step 2, Account A loses $500 but Account B never receives it. A transaction wraps both steps — if either fails, both are rolled back.

### A – Atomicity
**"All or nothing."**
A transaction is treated as a single indivisible unit. Either **all** operations complete successfully (committed) or **none** of them take effect (rolled back).

### C – Consistency
**"The database stays valid before and after every transaction."**
A transaction brings the database from one valid state to another, respecting all defined rules (constraints, foreign keys, data types). A transaction that would violate consistency is rejected.

### I – Isolation
**"Concurrent transactions don't interfere with each other."**
While a transaction is in progress, its intermediate states are invisible to other transactions. Each transaction behaves as if it runs alone, even if thousands are running simultaneously.

> **Example:** If two airline booking systems try to reserve the last seat simultaneously, isolation ensures only one succeeds — not both.

### D – Durability
**"Committed data survives failures."**
Once a transaction is committed, its changes are permanent — even if the server crashes immediately afterward. This is achieved through write-ahead logging and persistent storage.

### TCL – Transaction Control in SQL

```sql
-- Start a transaction (implicit in most DBMS)
BEGIN TRANSACTION;

-- Debit Account A
UPDATE Accounts SET balance = balance - 500 WHERE account_id = 'A';

-- Credit Account B
UPDATE Accounts SET balance = balance + 500 WHERE account_id = 'B';

-- If all went well, make it permanent
COMMIT;

-- If something went wrong, undo everything
-- ROLLBACK;
```

---

## 8. Types of Database Models

Beyond the relational model, several other database models exist, each suited to different types of data and use cases.

### 8.1 Hierarchical Model

Data organized as a **tree structure** with parent-child relationships. Each child has exactly one parent.

```
Company
├── Engineering Dept
│   ├── Ana Martínez
│   └── Carlos López
└── Finance Dept
    └── Sofía Herrera
```

- **Pros:** Fast for hierarchical data (org charts, file systems).
- **Cons:** Rigid; can't easily represent many-to-many relationships; redundancy when a child has multiple parents.
- **Example:** IBM IMS, Windows Registry.

### 8.2 Network Model

An extension of the hierarchical model where records can have **multiple parents**, forming a graph structure.

- **Pros:** More flexible than hierarchical; can represent complex relationships.
- **Cons:** Extremely complex to navigate; developers must know the physical data structure.
- **Example:** CODASYL databases, early airline reservation systems.

### 8.3 Relational Model

Data in **tables** with rows and columns; relationships through shared key values; queried with SQL.

- **Pros:** Flexible, intuitive, powerful querying, data independence, widely supported.
- **Cons:** Can be slow for very large unstructured data; rigid schema can be hard to change.
- **Example:** MySQL, PostgreSQL, Oracle, SQL Server.

### 8.4 Object-Oriented Model

Data stored as **objects**, like in object-oriented programming. Objects have attributes and methods. Supports inheritance and encapsulation.

- **Pros:** Natural fit for OOP applications; handles complex data types (images, audio).
- **Cons:** Not as widely adopted; limited querying capabilities vs SQL.
- **Example:** db4o, ObjectDB.

### 8.5 Entity-Attribute-Value (EAV) Model

Data stored as triples: (entity, attribute, value). Extremely flexible — no fixed schema.

- **Pros:** Handles sparse data with many optional attributes.
- **Cons:** Querying is complex and slow; no type enforcement.
- **Example:** Electronic health records (patients have different sets of symptoms/measurements).

---

## 9. NoSQL Databases

**NoSQL** ("Not Only SQL") databases depart from the strict table-based relational model. They are designed for **massive scale, high availability, flexible schemas**, and types of data that don't fit neatly into tables.

The four main NoSQL categories:

### 9.1 Document Stores

Data is stored as **documents** (usually JSON or BSON format). Each document is self-contained and can have a different structure.

```json
{
  "student_id": 1001,
  "name": "Ana Martínez",
  "email": "ana.m@university.edu",
  "courses": [
    { "id": "CS1111", "grade": "A" },
    { "id": "MATH101", "grade": "A-" }
  ],
  "address": {
    "city": "Monterrey",
    "state": "NL"
  }
}
```

- **Best for:** Content management, user profiles, catalogs, any data with variable structure.
- **Examples:** **MongoDB**, CouchDB, Amazon DocumentDB.

### 9.2 Key-Value Stores

The simplest NoSQL type. Data stored as **key → value** pairs, like a giant hash map. Values can be anything (string, number, JSON, binary).

```
"session:abc123"  →  { "user_id": 1001, "expires": "2025-12-01" }
"price:AAPL"      →  "198.50"
"user:1001:cart"  →  ["item_A", "item_B", "item_C"]
```

- **Best for:** Caching, session management, shopping carts, real-time leaderboards.
- **Examples:** **Redis**, Amazon DynamoDB, Memcached.

### 9.3 Column-Family Stores (Wide-Column)

Data stored in **column families** — rows can have different sets of columns. Optimized for reading/writing large amounts of data across many rows for specific columns.

- **Best for:** Time-series data, analytics, IoT sensor data, write-heavy workloads.
- **Examples:** **Apache Cassandra**, Google Bigtable, HBase.

### 9.4 Graph Databases

Data stored as **nodes** (entities) and **edges** (relationships), with properties on both. Optimized for traversing complex, interconnected relationships.

```
(Ana) ──FRIENDS_WITH──> (Carlos)
(Ana) ──ENROLLED_IN───> (CS1111)
(CS1111) ──TAUGHT_BY──> (Prof. Ramírez)
```

- **Best for:** Social networks, recommendation engines, fraud detection, knowledge graphs.
- **Examples:** **Neo4j**, Amazon Neptune, ArangoDB.

### 9.5 SQL vs. NoSQL Comparison

| Feature | SQL (Relational) | NoSQL |
|---|---|---|
| **Data Model** | Structured tables | Documents, KV, Columns, Graphs |
| **Schema** | Fixed, predefined | Flexible, dynamic |
| **Query Language** | SQL (standardized) | Varies by system |
| **Relationships** | Excellent (JOINs) | Limited or application-level |
| **ACID Transactions** | Strong guarantees | Often eventual consistency |
| **Scalability** | Vertical (bigger server) | Horizontal (more servers) |
| **Best For** | Structured data, complex queries | Large scale, flexible/unstructured data |
| **Examples** | MySQL, PostgreSQL, Oracle | MongoDB, Redis, Cassandra, Neo4j |

> **Key insight:** Neither SQL nor NoSQL is universally better. The choice depends on your data's structure, scale, and access patterns. Many modern applications use **both** (polyglot persistence) — for example, a relational DB for orders/payments and Redis for caching session data.

---

## 10. Popular DBMS Software

### 10.1 MySQL

- **Type:** Open-source relational DBMS.
- **Owned by:** Oracle Corporation.
- **Strengths:** Extremely popular for web applications, fast reads, easy to set up, widely hosted.
- **Used by:** WordPress, Facebook (historically), Twitter, YouTube.
- **Best for:** Web applications, e-commerce, content management systems.

### 10.2 PostgreSQL

- **Type:** Open-source object-relational DBMS.
- **Strengths:** Advanced SQL features, excellent standards compliance, supports JSON natively, extensible, very reliable.
- **Used by:** Instagram, Spotify, Reddit, Apple.
- **Best for:** Complex queries, geospatial data (PostGIS), applications needing strict data integrity.

### 10.3 Microsoft SQL Server

- **Type:** Commercial relational DBMS.
- **Owned by:** Microsoft.
- **Strengths:** Deep Windows/Azure integration, excellent enterprise tooling (SSMS), Business Intelligence features.
- **Best for:** Enterprise applications on Windows infrastructure, .NET applications.

### 10.4 Oracle Database

- **Type:** Commercial relational DBMS.
- **Strengths:** Industry-leading performance for enterprise workloads, advanced security, RAC (Real Application Clusters) for high availability.
- **Used by:** Banks, airlines, government systems, large enterprises.
- **Best for:** Mission-critical enterprise systems requiring maximum reliability.

### 10.5 MongoDB

- **Type:** Open-source document-oriented NoSQL DBMS.
- **Strengths:** Flexible JSON-like documents, easy horizontal scaling, great for rapidly changing schemas.
- **Used by:** Uber, eBay, Forbes, Bosch.
- **Best for:** Content management, real-time analytics, applications with evolving data structures.

### 10.6 Redis

- **Type:** Open-source in-memory key-value store.
- **Strengths:** Extremely fast (data lives in RAM), supports rich data structures, pub/sub messaging.
- **Used by:** Twitter, GitHub, Snapchat (for caching and real-time features).
- **Best for:** Caching, session storage, real-time leaderboards, message queues.

### 10.7 Quick Comparison

| DBMS | Type | License | Best Known For |
|---|---|---|---|
| MySQL | Relational | Open-source | Web applications |
| PostgreSQL | Object-Relational | Open-source | Advanced SQL, reliability |
| SQL Server | Relational | Commercial | Microsoft ecosystem |
| Oracle DB | Relational | Commercial | Enterprise mission-critical |
| MongoDB | Document NoSQL | Open-source | Flexible schemas |
| Redis | Key-Value NoSQL | Open-source | Speed, caching |

---

## 11. Database Security

Databases often hold an organization's most sensitive data — customer records, financial information, medical records, passwords. Securing them is critical.

### 11.1 Authentication and Authorization

- **Authentication:** Verify who is connecting to the database (username + password, certificates, OS authentication).
- **Authorization:** Control what authenticated users can do.

```sql
-- Create a database user
CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'SecurePass!2024';

-- Grant only the necessary permissions (Principle of Least Privilege)
GRANT SELECT, INSERT, UPDATE ON university_db.Students TO 'app_user'@'localhost';

-- Revoke a permission
REVOKE INSERT ON university_db.Students FROM 'app_user'@'localhost';
```

### 11.2 SQL Injection

**SQL Injection** is the most common and dangerous database attack. It occurs when malicious SQL code is inserted into an input field and executed by the database.

**Vulnerable code (Python example):**
```python
# DANGEROUS — never do this!
username = input("Enter username: ")
query = "SELECT * FROM Users WHERE username = '" + username + "'"
```

**Attack:** User enters: `' OR '1'='1`
```sql
-- The query becomes:
SELECT * FROM Users WHERE username = '' OR '1'='1'
-- '1'='1' is always true → returns ALL users!
```

**Or worse:** User enters: `'; DROP TABLE Users; --`
```sql
SELECT * FROM Users WHERE username = ''; DROP TABLE Users; --'
-- Deletes the entire Users table!
```

**Prevention — Use Parameterized Queries / Prepared Statements:**
```python
# SAFE — user input is never interpreted as SQL
cursor.execute("SELECT * FROM Users WHERE username = %s", (username,))
```

### 11.3 Encryption

- **Encryption at rest:** The database files on disk are encrypted (e.g., MySQL's Transparent Data Encryption, Oracle TDE).
- **Encryption in transit:** Communications between the app and database use TLS/SSL.
- **Column-level encryption:** Sensitive columns (passwords, SSNs, credit card numbers) stored encrypted.

> **Password storage best practice:** Never store plain-text passwords. Store a **salted hash** (e.g., bcrypt, Argon2). Even if the database is stolen, passwords cannot be recovered.

### 11.4 Backups and Recovery

- **Regular backups:** Full, incremental, and differential backups on a schedule.
- **Point-in-time recovery:** Using transaction logs to restore to a specific moment.
- **The 3-2-1 rule:** Keep **3** copies of data, on **2** different media, with **1** offsite.

### 11.5 Auditing

Database **audit logs** record who accessed what data, when, and what changes were made. Essential for compliance (GDPR, HIPAA, PCI-DSS) and forensic investigation after a breach.

---

## 12. Module Summary

Key takeaways from Module 5:

- A **database** is an organized collection of related data. A **DBMS** is the software that manages it, solving problems of redundancy, inconsistency, concurrency, security, and integrity.
- Databases evolved from **flat files** → **hierarchical/network models** → **relational model** (Codd, 1970) → **NoSQL** and **cloud databases**.
- The **relational model** organizes data into **tables** with rows and columns; **primary keys** uniquely identify rows; **foreign keys** link tables; relationships are 1:1, 1:N, or M:N.
- **SQL** is the standard language: DDL (CREATE, ALTER, DROP), DML (INSERT, UPDATE, DELETE), and DQL (SELECT) with WHERE, JOIN, GROUP BY, HAVING, and aggregate functions.
- **Normalization** (1NF, 2NF, 3NF) eliminates data redundancy and dependency anomalies.
- **ACID properties** (Atomicity, Consistency, Isolation, Durability) guarantee reliable transactions even in the face of errors and concurrency.
- **NoSQL databases** (Document, Key-Value, Column-Family, Graph) offer flexible schemas and horizontal scalability for large-scale and unstructured data.
- **Database security** requires authentication, authorization (least privilege), protection against **SQL injection** (parameterized queries), encryption, backups, and auditing.

---

## 13. Module Quiz

Test your understanding of Module 5.

---

**Question 1**
Which of the following best describes a **primary key**?

- A) A column that can contain NULL values to represent missing data
- B) A column or set of columns that uniquely identifies each row in a table
- C) A column in one table that references a column in another table
- D) A special index used to speed up SELECT queries

---

**Question 2**
A university database has a `Students` table and a `Courses` table. A student can enroll in many courses, and each course can have many students. What type of relationship is this, and how is it typically implemented?

- A) One-to-One; add a foreign key to either table
- B) One-to-Many; add a foreign key to the Courses table
- C) Many-to-Many; create a junction (bridge) table with foreign keys to both tables
- D) Many-to-Many; duplicate the student record for each course

---

**Question 3**
Which SQL clause is used to filter results **after** grouping and aggregation?

- A) `WHERE`
- B) `HAVING`
- C) `ORDER BY`
- D) `LIMIT`

---

**Question 4**
A database table violates **First Normal Form (1NF)** if:

- A) A non-key column depends on another non-key column
- B) A column holds multiple values in a single cell (e.g., "CS1111, MATH101")
- C) Two tables share the same primary key
- D) A foreign key references a non-existent primary key

---

**Question 5**
Which ACID property ensures that if a system crashes in the middle of a bank transfer (after debiting Account A but before crediting Account B), **both operations are undone**?

- A) Consistency
- B) Isolation
- C) Durability
- D) Atomicity

---

**Question 6**
A developer builds an e-commerce website. They need to store product catalog data where each product can have a completely different set of attributes (a shirt has size/color, a laptop has RAM/storage). Which database type is most appropriate?

- A) Relational (SQL) with a fixed schema
- B) Document-oriented NoSQL (e.g., MongoDB)
- C) Graph database (e.g., Neo4j)
- D) Hierarchical database

---

**Question 7**
What is **SQL Injection**, and which of the following correctly prevents it?

- A) An attack where excess data overwrites memory; prevented by firewalls
- B) An attack inserting malicious SQL into input fields; prevented by **parameterized queries**
- C) An attack that encrypts database files; prevented by regular backups
- D) An attack exploiting weak passwords; prevented by multi-factor authentication

---

**Question 8**
Which SQL statement correctly retrieves the full name and course title for all students enrolled in a course, combining data from the `Students`, `Enrollments`, and `Courses` tables?

- A)
```sql
SELECT first_name, last_name, title
FROM Students, Courses;
```
- B)
```sql
SELECT s.first_name, s.last_name, c.title
FROM Enrollments e
INNER JOIN Students s ON e.student_id = s.student_id
INNER JOIN Courses  c ON e.course_id  = c.course_id;
```
- C)
```sql
SELECT first_name, last_name, title
FROM Students
WHERE course_id = Courses.course_id;
```
- D)
```sql
SELECT * FROM Students UNION SELECT * FROM Courses;
```

---

**Question 9**
Redis is being used in a social media application. Which use case is Redis **most** appropriate for in this context?

- A) Storing the permanent user profile data including bio, posts, and followers
- B) Caching the "trending topics" list that needs to refresh every few seconds
- C) Running complex analytical queries across millions of historical posts
- D) Storing the relationship graph between millions of users and their friends

---

**Question 10** *(Short Answer)*
Explain the **ACID properties** of database transactions using the example of an online shopping checkout process (customer places an order, payment is charged, and inventory is reduced). Describe what could go wrong without each property, and how ACID prevents it.

---

### Answer Key

| Question | Answer | Explanation |
|---|---|---|
| 1 | **B** | A primary key uniquely identifies each row, must be unique and NOT NULL. Option C describes a foreign key. |
| 2 | **C** | Students ↔ Courses is Many-to-Many. It is implemented with a junction table (e.g., `Enrollments`) that holds foreign keys to both tables, breaking M:N into two 1:N relationships. |
| 3 | **B** | `HAVING` filters after aggregation/grouping. `WHERE` filters individual rows before grouping. |
| 4 | **B** | 1NF requires atomic (single) values in each cell. Storing multiple courses in one cell violates atomicity. Option A describes a 3NF violation. |
| 5 | **D** | **Atomicity** — the entire transaction is treated as one unit. If any step fails, the whole transaction is rolled back, leaving the database unchanged. |
| 6 | **B** | Document-oriented NoSQL (MongoDB) stores each product as a flexible JSON document where different products can have entirely different fields — ideal for variable-attribute catalog data. |
| 7 | **B** | SQL Injection embeds malicious SQL in user input. Parameterized queries separate SQL code from data, so user input is never interpreted as SQL commands. |
| 8 | **B** | Option B correctly uses INNER JOINs with proper aliases and ON conditions to link the three tables through the junction `Enrollments` table. Option A creates a cartesian product (every student × every course). |
| 9 | **B** | Redis is an in-memory key-value store optimized for speed — perfect for caching frequently-read, rapidly-changing data like trending topics. User profiles need a document/relational DB (C would need analytical DB, D would need a graph DB). |
| 10 | *(See below)* | |

**Question 10 – Sample Answer:**

> Consider the checkout process: (1) Create order record, (2) Charge the customer's credit card, (3) Reduce product inventory.
>
> **Atomicity** — *"All or nothing."* Without it: the payment could be charged but the order record never created (customer pays but gets no confirmation), or inventory could be reduced but payment fails (store loses stock without payment). With Atomicity, if any step fails, all steps are rolled back — the customer is not charged and no inventory is reduced.
>
> **Consistency** — *"Rules are always respected."* Without it: inventory could go below zero (e.g., selling -2 items), or an order could reference a product that doesn't exist. With Consistency, the DBMS enforces constraints (e.g., `inventory >= 0`, valid foreign keys), rejecting any transaction that would violate them.
>
> **Isolation** — *"Concurrent transactions don't interfere."* Without it: if two customers simultaneously buy the last item in stock, both could read `inventory = 1`, both succeed, and inventory goes to -1. With Isolation, one transaction completes first; the second then sees `inventory = 0` and fails appropriately.
>
> **Durability** — *"Committed data survives failures."* Without it: after a successful payment and COMMIT, a server crash could lose the order record — the customer is charged but the order disappears. With Durability, committed transactions are written to persistent storage (via write-ahead logging), so they survive crashes and power failures.

---

*End of Module 5 — Up next: **Module 6: Computer Networks and Security***
