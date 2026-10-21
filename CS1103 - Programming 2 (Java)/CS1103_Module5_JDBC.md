# CS 1103 – Programming 2 (Java)
## Module 5: Introduction to Java Database Connectivity (JDBC)

---

## Table of Contents

1. [What Is JDBC?](#1-what-is-jdbc)
2. [Relational Databases — A Quick Primer](#2-relational-databases--a-quick-primer)
3. [Setting Up — SQLite](#3-setting-up--sqlite)
4. [The JDBC Architecture](#4-the-jdbc-architecture)
5. [Connecting to a Database](#5-connecting-to-a-database)
6. [Creating Tables](#6-creating-tables)
7. [Inserting Data](#7-inserting-data)
8. [Querying Data — SELECT](#8-querying-data--select)
9. [Updating and Deleting Data](#9-updating-and-deleting-data)
10. [PreparedStatement — Safe and Efficient Queries](#10-preparedstatement--safe-and-efficient-queries)
11. [Transactions](#11-transactions)
12. [A Clean JDBC Pattern](#12-a-clean-jdbc-pattern)
13. [Putting It All Together](#13-putting-it-all-together)
14. [Exercises](#14-exercises)

---

## 1. What Is JDBC?

So far, your programs store data in variables, files, or serialized objects. These work for simple cases, but they don't scale well when you need to:

- Store thousands (or millions) of records efficiently.
- Search and filter data quickly.
- Allow multiple users to access the same data simultaneously.
- Enforce relationships between data (e.g. every order must belong to a customer).

This is what **relational databases** are built for.

**JDBC (Java Database Connectivity)** is Java's standard API for communicating with relational databases. It is part of the JDK (`java.sql`) and provides a consistent interface regardless of which database you use — MySQL, PostgreSQL, SQLite, Oracle, and others.

Think of JDBC as a universal remote control: the same buttons (API) work regardless of which TV brand (database) is connected. The specific TV (database driver) handles the actual communication.

---

## 2. Relational Databases — A Quick Primer

A **relational database** organizes data into **tables** — rows and columns, like a spreadsheet. Each table represents one type of thing (students, products, orders).

```
students table:
+----+--------+-----+-------+
| id | name   | age | gpa   |
+----+--------+-----+-------+
|  1 | Alice  |  20 |  3.85 |
|  2 | Bob    |  22 |  3.42 |
|  3 | Carlos |  21 |  3.91 |
+----+--------+-----+-------+
```

You interact with a database using **SQL (Structured Query Language)**:

| Operation | SQL Command | What it does |
|---|---|---|
| Create table | `CREATE TABLE` | Define the structure |
| Insert data | `INSERT INTO` | Add a row |
| Read data | `SELECT` | Query rows |
| Update data | `UPDATE` | Modify rows |
| Delete data | `DELETE` | Remove rows |

These four operations — Create, Read, Update, Delete — are called **CRUD**.

### Key SQL Concepts You'll Need

```sql
-- Create a table
CREATE TABLE students (
    id   INTEGER PRIMARY KEY,
    name TEXT    NOT NULL,
    age  INTEGER,
    gpa  REAL
);

-- Insert a row
INSERT INTO students (name, age, gpa) VALUES ('Alice', 20, 3.85);

-- Query all rows
SELECT * FROM students;

-- Query with a condition
SELECT name, gpa FROM students WHERE gpa > 3.5;

-- Update a row
UPDATE students SET gpa = 3.90 WHERE id = 1;

-- Delete a row
DELETE FROM students WHERE id = 2;
```

---

## 3. Setting Up — SQLite

For this module we use **SQLite** — a lightweight, file-based database that requires no installation or server. It stores the entire database in a single `.db` file, making it perfect for learning and small applications.

### Adding the SQLite Driver

JDBC needs a **driver** — a JAR file that knows how to talk to a specific database.

**If you're using IntelliJ IDEA:**
1. Go to `File → Project Structure → Libraries`.
2. Click `+` → `From Maven`.
3. Search for `org.xerial:sqlite-jdbc` and add the latest version.

**If you're using the command line**, download `sqlite-jdbc-x.x.x.jar` from:
https://github.com/xerial/sqlite-jdbc/releases

Then compile and run with the JAR on the classpath:
```bash
javac -cp sqlite-jdbc.jar Main.java
java  -cp .:sqlite-jdbc.jar Main      # macOS/Linux
java  -cp .;sqlite-jdbc.jar Main      # Windows
```

**JDBC URL for SQLite:**
```
jdbc:sqlite:school.db       ← creates/opens school.db in the current directory
jdbc:sqlite::memory:        ← in-memory database (gone when the program ends)
```

---

## 4. The JDBC Architecture

```
Your Java Code
      ↓
  JDBC API (java.sql)
      ↓
  Database Driver (sqlite-jdbc.jar)
      ↓
  Database (school.db file)
```

The four core JDBC interfaces — all in `java.sql`:

| Interface | Purpose |
|---|---|
| `Connection` | Represents the link to the database |
| `Statement` | Executes a static SQL string |
| `PreparedStatement` | Executes a precompiled SQL query with parameters |
| `ResultSet` | Holds the rows returned by a SELECT query |

---

## 5. Connecting to a Database

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class ConnectionDemo {
    public static void main(String[] args) {
        String url = "jdbc:sqlite:school.db";

        try (Connection conn = DriverManager.getConnection(url)) {
            System.out.println("Connected to database.");
            System.out.println("Driver: " + conn.getMetaData().getDriverName());
        } catch (SQLException e) {
            System.out.println("Connection failed: " + e.getMessage());
        }
    }
}
```

`DriverManager.getConnection(url)` opens the connection. SQLite creates `school.db` automatically if it doesn't exist.

Always use **try-with-resources** for `Connection` — it closes the connection automatically, releasing the database resources.

---

## 6. Creating Tables

Use a `Statement` to execute DDL (Data Definition Language) — SQL that creates or modifies the database structure.

```java
import java.sql.*;

public class CreateTableDemo {

    static final String URL = "jdbc:sqlite:school.db";

    public static void main(String[] args) {
        String createSQL = """
            CREATE TABLE IF NOT EXISTS students (
                id    INTEGER PRIMARY KEY AUTOINCREMENT,
                name  TEXT    NOT NULL,
                age   INTEGER NOT NULL,
                gpa   REAL    NOT NULL
            )
            """;

        try (Connection conn = DriverManager.getConnection(URL);
             Statement stmt = conn.createStatement()) {

            stmt.execute(createSQL);
            System.out.println("Table 'students' ready.");

        } catch (SQLException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
```

Key SQL notes:
- `IF NOT EXISTS` — creates the table only if it doesn't already exist. Safe to run every time.
- `PRIMARY KEY AUTOINCREMENT` — SQLite automatically assigns a unique, incrementing `id` to each row.
- `NOT NULL` — the database enforces that these fields must have a value.

---

## 7. Inserting Data

```java
import java.sql.*;

public class InsertDemo {

    static final String URL = "jdbc:sqlite:school.db";

    public static void main(String[] args) {
        String insertSQL = "INSERT INTO students (name, age, gpa) VALUES (?, ?, ?)";

        try (Connection conn = DriverManager.getConnection(URL);
             PreparedStatement pstmt = conn.prepareStatement(insertSQL)) {

            // Insert first student
            pstmt.setString(1, "Alice");
            pstmt.setInt(2, 20);
            pstmt.setDouble(3, 3.85);
            pstmt.executeUpdate();

            // Reuse the same PreparedStatement for the second student
            pstmt.setString(1, "Bob");
            pstmt.setInt(2, 22);
            pstmt.setDouble(3, 3.42);
            pstmt.executeUpdate();

            pstmt.setString(1, "Carlos");
            pstmt.setInt(2, 21);
            pstmt.setDouble(3, 3.91);
            pstmt.executeUpdate();

            System.out.println("3 students inserted.");

        } catch (SQLException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
```

Notice the `?` placeholders — this is a `PreparedStatement`, covered fully in Section 10. The `?` marks are replaced by actual values using `setString()`, `setInt()`, `setDouble()`, etc. Always use `PreparedStatement` for INSERT/UPDATE/DELETE — never concatenate user input directly into SQL strings.

---

## 8. Querying Data — SELECT

Use `executeQuery()` to run a SELECT and get back a `ResultSet` — a cursor that steps through the returned rows.

```java
import java.sql.*;

public class QueryDemo {

    static final String URL = "jdbc:sqlite:school.db";

    public static void main(String[] args) {
        String selectSQL = "SELECT * FROM students ORDER BY gpa DESC";

        try (Connection conn = DriverManager.getConnection(URL);
             Statement stmt  = conn.createStatement();
             ResultSet rs    = stmt.executeQuery(selectSQL)) {

            System.out.printf("%-5s %-12s %-5s %-5s%n", "ID", "Name", "Age", "GPA");
            System.out.println("-".repeat(30));

            while (rs.next()) {   // advance to the next row; returns false at end
                int    id   = rs.getInt("id");
                String name = rs.getString("name");
                int    age  = rs.getInt("age");
                double gpa  = rs.getDouble("gpa");

                System.out.printf("%-5d %-12s %-5d %.2f%n", id, name, age, gpa);
            }

        } catch (SQLException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
```

Output:
```
ID    Name         Age   GPA  
------------------------------
3     Carlos       21    3.91
1     Alice        20    3.85
2     Bob          22    3.42
```

### ResultSet Methods

| Method | Returns |
|---|---|
| `rs.next()` | `true` if another row exists, advances the cursor |
| `rs.getInt(column)` | `int` value of a column (by name or index) |
| `rs.getString(column)` | `String` value |
| `rs.getDouble(column)` | `double` value |
| `rs.getBoolean(column)` | `boolean` value |
| `rs.wasNull()` | `true` if the last value read was SQL NULL |

You can reference columns by **name** (`rs.getString("name")`) or by **index** (`rs.getString(2)`). Names are safer and more readable.

### Querying with a Condition

```java
String sql = "SELECT * FROM students WHERE gpa >= ?";

try (Connection conn = DriverManager.getConnection(URL);
     PreparedStatement pstmt = conn.prepareStatement(sql)) {

    pstmt.setDouble(1, 3.80);

    try (ResultSet rs = pstmt.executeQuery()) {
        while (rs.next()) {
            System.out.println(rs.getString("name") + " — " + rs.getDouble("gpa"));
        }
    }
} catch (SQLException e) {
    System.out.println("Error: " + e.getMessage());
}
```

Output:
```
Carlos — 3.91
Alice — 3.85
```

---

## 9. Updating and Deleting Data

Both UPDATE and DELETE use `executeUpdate()`, which returns the number of rows affected.

### UPDATE

```java
String updateSQL = "UPDATE students SET gpa = ? WHERE name = ?";

try (Connection conn = DriverManager.getConnection(URL);
     PreparedStatement pstmt = conn.prepareStatement(updateSQL)) {

    pstmt.setDouble(1, 3.95);
    pstmt.setString(2, "Alice");

    int rowsAffected = pstmt.executeUpdate();
    System.out.println("Rows updated: " + rowsAffected);   // 1

} catch (SQLException e) {
    System.out.println("Error: " + e.getMessage());
}
```

### DELETE

```java
String deleteSQL = "DELETE FROM students WHERE id = ?";

try (Connection conn = DriverManager.getConnection(URL);
     PreparedStatement pstmt = conn.prepareStatement(deleteSQL)) {

    pstmt.setInt(1, 2);   // delete student with id = 2
    int rowsAffected = pstmt.executeUpdate();
    System.out.println("Rows deleted: " + rowsAffected);   // 1

} catch (SQLException e) {
    System.out.println("Error: " + e.getMessage());
}
```

### `executeUpdate` vs `execute` vs `executeQuery`

| Method | Use for | Returns |
|---|---|---|
| `executeQuery()` | SELECT | `ResultSet` |
| `executeUpdate()` | INSERT, UPDATE, DELETE | `int` (rows affected) |
| `execute()` | Any SQL | `boolean` (true if result is a ResultSet) |

---

## 10. `PreparedStatement` — Safe and Efficient Queries

This section deserves special attention — it covers one of the most important security concepts in JDBC.

### SQL Injection — The Danger of String Concatenation

Never build SQL queries by concatenating user input:

```java
// DANGEROUS — never do this
String name = getUserInput();   // what if user types: ' OR '1'='1
String sql  = "SELECT * FROM students WHERE name = '" + name + "'";
// resulting query: SELECT * FROM students WHERE name = '' OR '1'='1'
// This returns ALL rows — authentication bypassed!
```

This is called **SQL injection** — an attacker manipulates the query by injecting SQL code through user input. It's one of the most common security vulnerabilities in applications.

### PreparedStatement Prevents This

A `PreparedStatement` precompiles the SQL template first. Parameters are then passed separately — they are **never interpreted as SQL code**:

```java
// SAFE — the ? is never treated as SQL
String sql = "SELECT * FROM students WHERE name = ?";

PreparedStatement pstmt = conn.prepareStatement(sql);
pstmt.setString(1, name);   // even if name contains SQL, it's treated as plain text
ResultSet rs = pstmt.executeQuery();
```

### Benefits of PreparedStatement

| Benefit | Detail |
|---|---|
| **Security** | Prevents SQL injection completely |
| **Performance** | SQL is compiled once, executed many times efficiently |
| **Readability** | Parameters are set with typed methods — clearer than string building |
| **Null safety** | `pstmt.setNull(1, Types.VARCHAR)` handles null values cleanly |

### Setter Methods Reference

```java
pstmt.setString(1, "Alice");
pstmt.setInt(2, 20);
pstmt.setDouble(3, 3.85);
pstmt.setBoolean(4, true);
pstmt.setDate(5, java.sql.Date.valueOf(LocalDate.now()));
pstmt.setNull(6, java.sql.Types.VARCHAR);   // explicitly set to SQL NULL
```

Parameters are numbered from **1**, not 0.

---

## 11. Transactions

By default, JDBC runs in **auto-commit mode** — every statement is immediately and permanently saved to the database. This is convenient but dangerous for operations that must succeed or fail as a unit.

Consider a bank transfer: deduct $100 from Alice and add $100 to Bob. If the deduction succeeds but the credit fails (due to an error), Alice loses money but Bob never receives it. That's a corrupted state.

**Transactions** solve this: multiple operations are grouped, and either all succeed (**commit**) or all are undone (**rollback**).

```java
import java.sql.*;

public class TransactionDemo {

    static final String URL = "jdbc:sqlite:school.db";

    public static void main(String[] args) {
        String deductSQL = "UPDATE accounts SET balance = balance - ? WHERE name = ?";
        String creditSQL = "UPDATE accounts SET balance = balance + ? WHERE name = ?";

        Connection conn = null;

        try {
            conn = DriverManager.getConnection(URL);
            conn.setAutoCommit(false);   // begin transaction manually

            try (PreparedStatement deduct = conn.prepareStatement(deductSQL);
                 PreparedStatement credit = conn.prepareStatement(creditSQL)) {

                deduct.setDouble(1, 100.0);
                deduct.setString(2, "Alice");
                deduct.executeUpdate();

                // Simulate an error mid-transaction
                // if (true) throw new SQLException("Simulated failure");

                credit.setDouble(1, 100.0);
                credit.setString(2, "Bob");
                credit.executeUpdate();

                conn.commit();   // all steps succeeded — save permanently
                System.out.println("Transaction committed successfully.");
            }

        } catch (SQLException e) {
            System.out.println("Transaction failed: " + e.getMessage());
            if (conn != null) {
                try {
                    conn.rollback();   // undo everything since setAutoCommit(false)
                    System.out.println("Transaction rolled back.");
                } catch (SQLException ex) {
                    System.out.println("Rollback failed: " + ex.getMessage());
                }
            }
        } finally {
            if (conn != null) {
                try { conn.close(); } catch (SQLException e) {}
            }
        }
    }
}
```

### Transaction Checklist

1. `conn.setAutoCommit(false)` — turn off auto-commit before starting.
2. Execute all related statements.
3. `conn.commit()` — if everything succeeds.
4. `conn.rollback()` — in the `catch` block if anything fails.
5. `conn.close()` — in the `finally` block.

---

## 12. A Clean JDBC Pattern

Real projects don't scatter database logic across every class. The standard pattern is a **DAO (Data Access Object)** — a class dedicated to database operations for one entity.

```java
import java.sql.*;
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

public class StudentDAO {

    private final String url;

    public StudentDAO(String url) {
        this.url = url;
    }

    // --- Create ---
    public void insert(String name, int age, double gpa) throws SQLException {
        String sql = "INSERT INTO students (name, age, gpa) VALUES (?, ?, ?)";
        try (Connection conn = DriverManager.getConnection(url);
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            pstmt.setString(1, name);
            pstmt.setInt(2, age);
            pstmt.setDouble(3, gpa);
            pstmt.executeUpdate();
        }
    }

    // --- Read all ---
    public List<String> findAll() throws SQLException {
        List<String> results = new ArrayList<>();
        String sql = "SELECT * FROM students ORDER BY name";
        try (Connection conn = DriverManager.getConnection(url);
             Statement stmt = conn.createStatement();
             ResultSet rs   = stmt.executeQuery(sql)) {
            while (rs.next()) {
                results.add(String.format("[%d] %-12s age %-3d GPA %.2f",
                    rs.getInt("id"), rs.getString("name"),
                    rs.getInt("age"), rs.getDouble("gpa")));
            }
        }
        return results;
    }

    // --- Read by ID ---
    public Optional<String> findById(int id) throws SQLException {
        String sql = "SELECT * FROM students WHERE id = ?";
        try (Connection conn  = DriverManager.getConnection(url);
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            pstmt.setInt(1, id);
            try (ResultSet rs = pstmt.executeQuery()) {
                if (rs.next()) {
                    return Optional.of(String.format("[%d] %s — GPA %.2f",
                        rs.getInt("id"), rs.getString("name"), rs.getDouble("gpa")));
                }
            }
        }
        return Optional.empty();
    }

    // --- Update ---
    public boolean updateGpa(int id, double newGpa) throws SQLException {
        String sql = "UPDATE students SET gpa = ? WHERE id = ?";
        try (Connection conn = DriverManager.getConnection(url);
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            pstmt.setDouble(1, newGpa);
            pstmt.setInt(2, id);
            return pstmt.executeUpdate() > 0;
        }
    }

    // --- Delete ---
    public boolean delete(int id) throws SQLException {
        String sql = "DELETE FROM students WHERE id = ?";
        try (Connection conn = DriverManager.getConnection(url);
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            pstmt.setInt(1, id);
            return pstmt.executeUpdate() > 0;
        }
    }

    // --- Count ---
    public int count() throws SQLException {
        String sql = "SELECT COUNT(*) FROM students";
        try (Connection conn = DriverManager.getConnection(url);
             Statement stmt = conn.createStatement();
             ResultSet rs   = stmt.executeQuery(sql)) {
            return rs.next() ? rs.getInt(1) : 0;
        }
    }
}
```

**Using the DAO:**
```java
StudentDAO dao = new StudentDAO("jdbc:sqlite:school.db");

dao.insert("Alice",  20, 3.85);
dao.insert("Bob",    22, 3.42);
dao.insert("Carlos", 21, 3.91);

System.out.println("Total students: " + dao.count());

dao.findAll().forEach(System.out::println);

dao.updateGpa(1, 3.95);
System.out.println("After update: " + dao.findById(1).orElse("Not found"));

dao.delete(2);
System.out.println("After delete — Total: " + dao.count());
```

---

## 13. Putting It All Together

Here's a complete console-based student management application using the DAO pattern:

```java
import java.sql.*;
import java.util.Scanner;

public class StudentApp {

    static final String URL = "jdbc:sqlite:school.db";
    static StudentDAO dao   = new StudentDAO(URL);
    static Scanner scanner  = new Scanner(System.in);

    public static void main(String[] args) throws SQLException {
        initDatabase();

        boolean running = true;
        while (running) {
            printMenu();
            int choice = readInt("Choice: ");
            System.out.println();

            switch (choice) {
                case 1 -> addStudent();
                case 2 -> listStudents();
                case 3 -> findStudent();
                case 4 -> updateGpa();
                case 5 -> deleteStudent();
                case 6 -> { System.out.println("Goodbye!"); running = false; }
                default -> System.out.println("Invalid choice.");
            }
        }
        scanner.close();
    }

    static void initDatabase() throws SQLException {
        String sql = """
            CREATE TABLE IF NOT EXISTS students (
                id    INTEGER PRIMARY KEY AUTOINCREMENT,
                name  TEXT    NOT NULL,
                age   INTEGER NOT NULL,
                gpa   REAL    NOT NULL
            )""";
        try (Connection conn = DriverManager.getConnection(URL);
             Statement stmt  = conn.createStatement()) {
            stmt.execute(sql);
        }
    }

    static void printMenu() {
        System.out.println("\n=== Student Manager ===");
        System.out.println("1. Add Student");
        System.out.println("2. List All Students");
        System.out.println("3. Find by ID");
        System.out.println("4. Update GPA");
        System.out.println("5. Delete Student");
        System.out.println("6. Exit");
    }

    static void addStudent() throws SQLException {
        String name = readString("Name: ");
        int    age  = readInt("Age: ");
        double gpa  = readDouble("GPA: ");
        dao.insert(name, age, gpa);
        System.out.println("Student added.");
    }

    static void listStudents() throws SQLException {
        var students = dao.findAll();
        if (students.isEmpty()) System.out.println("No students found.");
        else students.forEach(System.out::println);
    }

    static void findStudent() throws SQLException {
        int id = readInt("Enter student ID: ");
        System.out.println(dao.findById(id).orElse("Student not found."));
    }

    static void updateGpa() throws SQLException {
        int    id     = readInt("Student ID: ");
        double newGpa = readDouble("New GPA: ");
        boolean ok    = dao.updateGpa(id, newGpa);
        System.out.println(ok ? "GPA updated." : "Student not found.");
    }

    static void deleteStudent() throws SQLException {
        int id   = readInt("Student ID to delete: ");
        boolean ok = dao.delete(id);
        System.out.println(ok ? "Student deleted." : "Student not found.");
    }

    static String readString(String prompt) {
        System.out.print(prompt);
        return scanner.nextLine().trim();
    }

    static int readInt(String prompt) {
        System.out.print(prompt);
        int val = Integer.parseInt(scanner.nextLine().trim());
        return val;
    }

    static double readDouble(String prompt) {
        System.out.print(prompt);
        return Double.parseDouble(scanner.nextLine().trim());
    }
}
```

---

## 14. Exercises

---

### Exercise 1 – First Database

Set up SQLite in your project and write a program that:
1. Creates a `products` table with columns: `id` (auto-increment), `name` (text), `category` (text), `price` (real), `stock` (integer).
2. Inserts 5 products.
3. Queries and prints all products sorted by price.
4. Prints the total number of products.

---

### Exercise 2 – Full CRUD

Extend Exercise 1 with full CRUD operations via a menu:
1. Add a new product.
2. View all products.
3. Search by category (case-insensitive, use SQL `LIKE`).
4. Update the price of a product by ID.
5. Delete a product by ID.
6. Exit.

Use `PreparedStatement` for all INSERT, UPDATE, DELETE, and parameterized SELECT queries.

---

### Exercise 3 – DAO Pattern

Create a `BookDAO` class for a `books` table with fields: `id`, `title`, `author`, `year`, `available` (integer, 0 or 1).

Implement these methods:
- `insert(String title, String author, int year)`
- `findAll()` → `List<String>`
- `findByAuthor(String author)` → `List<String>`
- `markAvailable(int id, boolean available)` → `boolean`
- `delete(int id)` → `boolean`
- `countAvailable()` → `int`

Test all methods from `main`.

---

### Exercise 4 – Aggregation Queries

Using a `grades` table with columns `student_name`, `subject`, `score`:

1. Insert at least 10 rows (multiple subjects per student).
2. Write queries to find:
   - The **average score** per student.
   - The **highest score** in each subject.
   - The **student with the highest overall average**.
   - All students with an average **below 60** (at risk).

Use `GROUP BY`, `AVG()`, `MAX()`, and `HAVING` in your SQL — execute them through JDBC and display the results formatted.

---

### Exercise 5 – Transaction: Bank Transfer

Create an `accounts` table with `id`, `name`, `balance`.

Insert two accounts: Alice ($1000) and Bob ($500).

Write a method `transfer(Connection conn, String from, String to, double amount)` that:
1. Checks the sender has sufficient funds — throws a custom `InsufficientFundsException` if not.
2. Deducts from sender and adds to receiver **in a single transaction**.
3. Rolls back if anything fails.

Test with:
- A valid transfer ($200 from Alice to Bob).
- An invalid transfer ($2000 from Bob to Alice — should roll back).

Print balances before and after each transfer.

---

### Exercise 6 – Library System *(Challenge)*

Build a two-table library database:

**Table `books`:** `id`, `title`, `author`, `total_copies`

**Table `loans`:** `id`, `book_id` (foreign key), `member_name`, `loan_date`, `return_date` (nullable)

Implement:
- `addBook(title, author, copies)`
- `loanBook(bookId, memberName)` — only if a copy is available (total_copies minus active loans > 0); uses a transaction.
- `returnBook(loanId)` — sets the return date to today.
- `getAvailableBooks()` — books with at least one copy not on loan.
- `getActiveLoans()` — all loans where `return_date` is null.

Test the full flow from `main`: add books, loan some, return one, list available.

---

## Module Summary

Here's what you covered in Module 5:

- **JDBC** is Java's standard API for connecting to relational databases — the same code works across different database systems.
- **SQL CRUD** — `CREATE TABLE`, `INSERT`, `SELECT`, `UPDATE`, `DELETE` — is the language of relational databases.
- Use `DriverManager.getConnection(url)` to open a connection — always with try-with-resources.
- `Statement` executes static SQL; `PreparedStatement` executes parameterized SQL.
- **Never concatenate user input into SQL strings** — always use `PreparedStatement` to prevent SQL injection.
- `executeQuery()` returns a `ResultSet` for SELECT; `executeUpdate()` returns rows affected for INSERT/UPDATE/DELETE.
- Iterate a `ResultSet` with `while (rs.next())` and retrieve values with typed getters.
- **Transactions** group operations — use `setAutoCommit(false)`, `commit()`, and `rollback()` for atomic operations.
- The **DAO pattern** keeps database logic organized — one DAO class per entity with clean CRUD methods.

---

**Next up:** Module 6 – Generic Programming

---

*CS 1103 Programming 2 (Java) — Module 5 of 8*
