# CS 1103 – Programming 2 (Java)
## Module 1: Exception Handling and String Handling

---

## Table of Contents

1. [What Is an Exception?](#1-what-is-an-exception)
2. [The Exception Hierarchy](#2-the-exception-hierarchy)
3. [try / catch / finally](#3-try--catch--finally)
4. [Catching Multiple Exceptions](#4-catching-multiple-exceptions)
5. [Throwing Exceptions](#5-throwing-exceptions)
6. [Custom Exceptions](#6-custom-exceptions)
7. [Checked vs Unchecked Exceptions](#7-checked-vs-unchecked-exceptions)
8. [Introduction to String Handling](#8-introduction-to-string-handling)
9. [Essential String Methods](#9-essential-string-methods)
10. [String Comparison](#10-string-comparison)
11. [StringBuilder — Efficient String Building](#11-stringbuilder--efficient-string-building)
12. [Regular Expressions Basics](#12-regular-expressions-basics)
13. [Putting It All Together](#13-putting-it-all-together)
14. [Exercises](#14-exercises)

---

## 1. What Is an Exception?

When a Java program runs into a problem it cannot handle normally — dividing by zero, opening a file that doesn't exist, or converting "hello" to an integer — it doesn't just silently fail. It **throws an exception**: a signal that something went wrong.

If that exception is not handled, the program crashes and prints a **stack trace** — a report showing exactly what went wrong and where.

```
Exception in thread "main" java.lang.ArithmeticException: / by zero
    at Main.divide(Main.java:5)
    at Main.main(Main.java:10)
```

Reading a stack trace is one of the most important debugging skills in Java. It tells you:
- **The exception type** — `ArithmeticException`
- **The message** — `/ by zero`
- **The call chain** — which method called which, and on which line

Without exception handling, any unexpected input or condition would crash your program. With it, you can detect problems, respond gracefully, and keep the program running.

---

## 2. The Exception Hierarchy

All exceptions in Java are objects. They live in a class hierarchy rooted at `Throwable`:

```
Throwable
├── Error                        ← serious JVM problems, don't catch these
│   ├── OutOfMemoryError
│   └── StackOverflowError
└── Exception                   ← things programs should handle
    ├── RuntimeException         ← unchecked (don't need to declare or catch)
    │   ├── ArithmeticException         (e.g. divide by zero)
    │   ├── NullPointerException        (e.g. calling a method on null)
    │   ├── ArrayIndexOutOfBoundsException
    │   ├── NumberFormatException       (e.g. Integer.parseInt("abc"))
    │   └── IllegalArgumentException
    └── IOException              ← checked (must handle or declare)
        ├── FileNotFoundException
        └── EOFException
```

**Key takeaway:**
- `Error` — severe JVM-level problems. Don't catch them.
- `RuntimeException` — programming mistakes. You *can* catch them, but better to fix the code.
- Other `Exception` subclasses — expected external problems (files, networks, databases). You *must* handle these.

---

## 3. `try` / `catch` / `finally`

The core mechanism for handling exceptions in Java is the `try-catch` block.

### Basic Syntax

```java
try {
    // code that might throw an exception
} catch (ExceptionType e) {
    // code that runs if that exception occurs
} finally {
    // code that ALWAYS runs, exception or not
}
```

### Example — Catching a Division Error

```java
public class DivisionDemo {
    public static void main(String[] args) {
        int a = 10;
        int b = 0;

        try {
            int result = a / b;
            System.out.println("Result: " + result);
        } catch (ArithmeticException e) {
            System.out.println("Error: Cannot divide by zero.");
            System.out.println("Details: " + e.getMessage());
        }

        System.out.println("Program continues normally.");
    }
}
```

Output:
```
Error: Cannot divide by zero.
Details: / by zero
Program continues normally.
```

Without the `try-catch`, the program would have crashed. With it, we handle the problem and continue.

### The `finally` Block

`finally` runs whether an exception occurred or not. It's the right place to release resources (close files, close database connections):

```java
try {
    System.out.println("Trying...");
    int result = 10 / 2;
    System.out.println("Result: " + result);
} catch (ArithmeticException e) {
    System.out.println("Caught an exception.");
} finally {
    System.out.println("Finally block always runs.");
}
```

Output:
```
Trying...
Result: 5
Finally block always runs.
```

Even if an exception is thrown and caught, `finally` still runs. Even if no exception is thrown, `finally` still runs.

### Useful Exception Methods

| Method | What it returns |
|---|---|
| `e.getMessage()` | The error message string |
| `e.getClass().getName()` | The full exception class name |
| `e.printStackTrace()` | Prints the full stack trace to the console |

---

## 4. Catching Multiple Exceptions

A single `try` block may throw different types of exceptions. You can handle each one differently with multiple `catch` blocks.

### Multiple catch Blocks

```java
import java.util.Scanner;

public class MultiCatch {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        try {
            System.out.print("Enter a number: ");
            int number = Integer.parseInt(scanner.nextLine());   // may throw NumberFormatException

            int[] arr = {10, 20, 30};
            System.out.println("Array value: " + arr[number]);  // may throw ArrayIndexOutOfBoundsException

        } catch (NumberFormatException e) {
            System.out.println("That's not a valid integer.");
        } catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("Index out of range. Enter 0, 1, or 2.");
        } catch (Exception e) {
            // Catches anything else — acts as a safety net
            System.out.println("Something went wrong: " + e.getMessage());
        }

        scanner.close();
    }
}
```

> **Important:** Place more specific exceptions before more general ones. Since all exceptions extend `Exception`, putting `catch (Exception e)` first would catch everything — the specific blocks would never run.

### Multi-catch (Java 7+)

If you want to handle two exceptions the same way, combine them with `|`:

```java
} catch (NumberFormatException | ArrayIndexOutOfBoundsException e) {
    System.out.println("Invalid input: " + e.getMessage());
}
```

---

## 5. Throwing Exceptions

You're not limited to catching exceptions — you can **throw** them yourself when your code detects an invalid state.

Use the `throw` keyword with an exception object:

```java
static double divide(double a, double b) {
    if (b == 0) {
        throw new ArithmeticException("Divisor cannot be zero.");
    }
    return a / b;
}
```

```java
try {
    System.out.println(divide(10, 2));    // 5.0
    System.out.println(divide(10, 0));    // throws
} catch (ArithmeticException e) {
    System.out.println("Caught: " + e.getMessage());
}
```

Output:
```
5.0
Caught: Divisor cannot be zero.
```

### The `throws` Keyword

If a method can throw a **checked exception** (more on those shortly) but doesn't handle it internally, it must declare that in its signature using `throws`. This tells callers they need to handle it:

```java
static void readFile(String path) throws IOException {
    // code that might throw IOException
}
```

```java
try {
    readFile("data.txt");
} catch (IOException e) {
    System.out.println("Could not read file: " + e.getMessage());
}
```

Note the difference:
- `throw` — actually throws an exception object (action).
- `throws` — declares that a method *might* throw an exception (declaration).

---

## 6. Custom Exceptions

You can define your own exception types by extending `Exception` (for checked) or `RuntimeException` (for unchecked). Custom exceptions make your code more expressive — the exception name itself conveys what went wrong.

### Creating a Custom Exception

```java
public class InsufficientFundsException extends Exception {

    private double amount;

    public InsufficientFundsException(double amount) {
        super("Insufficient funds. Short by: $" + amount);
        this.amount = amount;
    }

    public double getAmount() {
        return amount;
    }
}
```

### Using the Custom Exception

```java
public class BankAccount {

    private String owner;
    private double balance;

    BankAccount(String owner, double balance) {
        this.owner = owner;
        this.balance = balance;
    }

    public void withdraw(double amount) throws InsufficientFundsException {
        if (amount > balance) {
            throw new InsufficientFundsException(amount - balance);
        }
        balance -= amount;
        System.out.printf("Withdrew $%.2f. Balance: $%.2f%n", amount, balance);
    }
}
```

```java
BankAccount account = new BankAccount("Alice", 100.00);

try {
    account.withdraw(60.00);
    account.withdraw(70.00);   // only $40 left — not enough
} catch (InsufficientFundsException e) {
    System.out.println("Transaction failed: " + e.getMessage());
    System.out.printf("You are short $%.2f%n", e.getAmount());
}
```

Output:
```
Withdrew $60.00. Balance: $40.00
Transaction failed: Insufficient funds. Short by: $30.0
You are short $30.00
```

Custom exceptions are especially useful when building libraries or APIs — they give callers specific, meaningful error information.

---

## 7. Checked vs Unchecked Exceptions

This distinction is fundamental to Java's exception system.

### Unchecked Exceptions (RuntimeException and subclasses)

- The compiler does **not** require you to handle them.
- They usually represent **programming mistakes** — bugs you should fix, not catch.
- Examples: `NullPointerException`, `ArrayIndexOutOfBoundsException`, `NumberFormatException`.

```java
// No compile error even though this could throw NullPointerException
String s = null;
System.out.println(s.length());   // crashes at runtime
```

### Checked Exceptions (Everything else under Exception)

- The compiler **forces** you to either catch them or declare them with `throws`.
- They represent **expected external problems** the program should anticipate.
- Examples: `IOException`, `FileNotFoundException`, `SQLException`.

```java
// COMPILE ERROR — must handle or declare IOException
public static void main(String[] args) {
    FileReader reader = new FileReader("file.txt");   // won't compile
}

// CORRECT — handled
public static void main(String[] args) {
    try {
        FileReader reader = new FileReader("file.txt");
    } catch (FileNotFoundException e) {
        System.out.println("File not found.");
    }
}
```

### Summary

| | Checked | Unchecked |
|---|---|---|
| Compiler enforcement | Yes — must handle or declare | No |
| Represents | External problems (files, network) | Programming mistakes |
| Extends | `Exception` (but not `RuntimeException`) | `RuntimeException` |
| Examples | `IOException`, `SQLException` | `NullPointerException`, `NumberFormatException` |

---

## 8. Introduction to String Handling

A `String` in Java is an **immutable** object — once created, its content cannot be changed. Any operation that appears to "modify" a string actually creates a brand new `String` object.

```java
String s = "Hello";
s = s + " World";   // "Hello" still exists in memory; s now points to a new object
```

This immutability makes strings safe to share but can be inefficient for heavy modification (use `StringBuilder` instead — covered in Section 11).

### Creating Strings

```java
String s1 = "Hello";                        // string literal (preferred)
String s2 = new String("Hello");            // using constructor (avoid this)
String s3 = String.valueOf(42);             // from a number → "42"
String s4 = String.valueOf(true);           // from boolean → "true"
```

### String Length

```java
String name = "Maria";
System.out.println(name.length());   // 5
```

### Character Access

```java
char first = name.charAt(0);   // 'M'
char last  = name.charAt(name.length() - 1);   // 'a'
```

---

## 9. Essential String Methods

### Case Conversion

```java
String s = "Hello World";
System.out.println(s.toUpperCase());   // HELLO WORLD
System.out.println(s.toLowerCase());   // hello world
```

### Trimming Whitespace

```java
String padded = "   hello   ";
System.out.println(padded.trim());    // "hello"
System.out.println(padded.strip());   // "hello" (Java 11+, Unicode-aware)
```

### Checking Contents

```java
String sentence = "Java is fun";

System.out.println(sentence.contains("fun"));          // true
System.out.println(sentence.startsWith("Java"));       // true
System.out.println(sentence.endsWith("fun"));          // true
System.out.println(sentence.isEmpty());                // false
System.out.println("".isEmpty());                      // true
System.out.println("   ".isBlank());                   // true (Java 11+)
```

### Searching

```java
String s = "banana";

System.out.println(s.indexOf('a'));        // 1 — first occurrence
System.out.println(s.lastIndexOf('a'));    // 5 — last occurrence
System.out.println(s.indexOf("nan"));     // 2 — substring search
System.out.println(s.indexOf('z'));        // -1 — not found
```

### Extracting Substrings

```java
String s = "Hello, World!";

System.out.println(s.substring(7));       // "World!" — from index 7 to end
System.out.println(s.substring(7, 12));   // "World" — index 7 up to (not including) 12
```

### Replacing

```java
String s = "I like cats and cats like me";

System.out.println(s.replace("cats", "dogs"));
// I like dogs and dogs like me

System.out.println(s.replace(' ', '-'));
// I-like-cats-and-cats-like-me
```

### Splitting

`split()` breaks a string into an array using a delimiter:

```java
String csv = "Alice,Bob,Carlos,Diana";
String[] names = csv.split(",");

for (String name : names) {
    System.out.println(name);
}
```

Output:
```
Alice
Bob
Carlos
Diana
```

```java
String sentence = "The quick brown fox";
String[] words = sentence.split(" ");
System.out.println(words.length);   // 4
```

### Joining

`String.join()` is the opposite of `split()`:

```java
String joined = String.join(", ", "Alice", "Bob", "Carlos");
System.out.println(joined);   // Alice, Bob, Carlos

String[] parts = {"2026", "05", "14"};
System.out.println(String.join("-", parts));   // 2026-05-14
```

### String.format

`String.format()` builds a formatted string (like `printf` but returns a String instead of printing):

```java
String name = "Alice";
int age = 30;
double gpa = 3.85;

String report = String.format("Name: %-10s | Age: %3d | GPA: %.2f", name, age, gpa);
System.out.println(report);
// Name: Alice      | Age:  30 | GPA: 3.85
```

Common format specifiers:

| Specifier | Meaning |
|---|---|
| `%s` | String |
| `%d` | Integer |
| `%f` | Float/double |
| `%.2f` | Float with 2 decimal places |
| `%n` | Newline |
| `%10s` | Right-aligned in 10 characters |
| `%-10s` | Left-aligned in 10 characters |

---

## 10. String Comparison

This is one of the most common sources of bugs in Java. **Never compare strings with `==`.**

### Why `==` is Wrong for Strings

`==` compares **references** (memory addresses), not the actual content:

```java
String a = new String("hello");
String b = new String("hello");

System.out.println(a == b);        // false — different objects in memory
System.out.println(a.equals(b));   // true  — same content
```

With string literals Java may reuse the same object (string interning), so `==` might accidentally return `true` sometimes — making the bug inconsistent and hard to track down.

### Always Use `.equals()` or `.equalsIgnoreCase()`

```java
String input = "Yes";

if (input.equals("Yes")) {
    System.out.println("User said yes.");
}

if (input.equalsIgnoreCase("yes")) {
    System.out.println("Case-insensitive match!");
}
```

### Comparing Alphabetically with `compareTo`

```java
String s1 = "apple";
String s2 = "banana";

int result = s1.compareTo(s2);
// negative → s1 comes before s2
// zero     → they are equal
// positive → s1 comes after s2

System.out.println(result);   // negative number
```

### Safe Comparison When a String Might be `null`

Calling `.equals()` on a `null` reference throws a `NullPointerException`. A safe pattern is to put the known (non-null) string on the left:

```java
String input = null;

// Unsafe
if (input.equals("quit")) { ... }   // NullPointerException!

// Safe
if ("quit".equals(input)) { ... }   // false — no crash
```

---

## 11. `StringBuilder` — Efficient String Building

Because `String` is immutable, building a string by concatenating in a loop creates many temporary objects:

```java
String result = "";
for (int i = 0; i < 1000; i++) {
    result += i;   // creates a new String object every iteration — slow!
}
```

**`StringBuilder`** solves this. It maintains a mutable buffer internally and only creates a final `String` when you call `toString()`:

```java
StringBuilder sb = new StringBuilder();

for (int i = 0; i < 1000; i++) {
    sb.append(i);   // modifies the buffer — no new objects
}

String result = sb.toString();
```

### Common StringBuilder Methods

```java
StringBuilder sb = new StringBuilder("Hello");

sb.append(" World");           // Hello World
sb.insert(5, ",");             // Hello, World
sb.replace(7, 12, "Java");     // Hello, Java
sb.delete(5, 7);               // HelloJava
sb.reverse();                  // avaJolleH
sb.toString();                 // convert to regular String

System.out.println(sb.length());   // current character count
```

### When to Use StringBuilder

Use `StringBuilder` when:
- You're building a string inside a loop.
- You're concatenating more than a handful of strings.
- Performance matters.

For simple one-off concatenation (`"Hello, " + name + "!"`), the regular `+` operator is fine — the compiler optimizes it automatically.

---

## 12. Regular Expressions Basics

A **regular expression** (regex) is a pattern used to match, search, or validate strings. Java's `String` class has built-in regex support.

### Common Regex Symbols

| Pattern | Matches |
|---|---|
| `.` | Any single character |
| `\d` | Any digit (0–9) |
| `\w` | Any word character (letters, digits, underscore) |
| `\s` | Any whitespace character |
| `[abc]` | Any of a, b, or c |
| `[a-z]` | Any lowercase letter |
| `[^abc]` | Anything except a, b, or c |
| `*` | Zero or more of the preceding |
| `+` | One or more of the preceding |
| `?` | Zero or one of the preceding |
| `{n}` | Exactly n of the preceding |
| `^` | Start of string |
| `$` | End of string |

> In Java strings, `\d` must be written as `"\\d"` because `\` is an escape character.

### `matches()` — Does the whole string match a pattern?

```java
String phone = "555-1234";
System.out.println(phone.matches("\\d{3}-\\d{4}"));   // true

String email = "user@example.com";
System.out.println(email.matches("[\\w.]+@[\\w.]+\\.[a-z]{2,}"));   // true

String code = "ABC";
System.out.println(code.matches("[A-Z]{3}"));   // true
```

### `replaceAll()` — Replace using a pattern

```java
String messy = "Hello    World   Java";
String clean = messy.replaceAll("\\s+", " ");   // collapse multiple spaces to one
System.out.println(clean);   // Hello World Java

String noDigits = "abc123def456".replaceAll("\\d", "");
System.out.println(noDigits);   // abcdef
```

### `split()` with Regex

`split()` accepts a regex, not just a plain string:

```java
String data = "one,two;three four";
String[] parts = data.split("[,; ]+");   // split on comma, semicolon, or space

for (String part : parts) {
    System.out.println(part);
}
```

Output:
```
one
two
three
four
```

---

## 13. Putting It All Together

Here's a complete program that validates and processes user input using exception handling, string methods, and regex:

```java
import java.util.Scanner;
import java.util.ArrayList;
import java.util.List;

public class ContactValidator {

    // Custom exception for invalid contact data
    static class InvalidContactException extends Exception {
        InvalidContactException(String message) {
            super(message);
        }
    }

    static void validateEmail(String email) throws InvalidContactException {
        if (!email.matches("[\\w.]+@[\\w.]+\\.[a-z]{2,}")) {
            throw new InvalidContactException("Invalid email: " + email);
        }
    }

    static void validatePhone(String phone) throws InvalidContactException {
        if (!phone.matches("\\d{3}-\\d{4}")) {
            throw new InvalidContactException("Invalid phone (use NNN-NNNN): " + phone);
        }
    }

    static String formatName(String name) {
        // Capitalize the first letter of each word
        String[] parts = name.trim().toLowerCase().split("\\s+");
        StringBuilder formatted = new StringBuilder();
        for (String part : parts) {
            if (part.length() > 0) {
                formatted.append(Character.toUpperCase(part.charAt(0)));
                formatted.append(part.substring(1));
                formatted.append(" ");
            }
        }
        return formatted.toString().trim();
    }

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        List<String> contacts = new ArrayList<>();

        System.out.println("=== Contact Entry (type 'done' to finish) ===\n");

        while (true) {
            try {
                System.out.print("Name (or 'done'): ");
                String name = scanner.nextLine();
                if (name.equalsIgnoreCase("done")) break;

                if (name.isBlank()) {
                    throw new InvalidContactException("Name cannot be blank.");
                }

                System.out.print("Email: ");
                String email = scanner.nextLine().trim();
                validateEmail(email);

                System.out.print("Phone (NNN-NNNN): ");
                String phone = scanner.nextLine().trim();
                validatePhone(phone);

                String formatted = String.format("%-20s | %-25s | %s",
                    formatName(name), email.toLowerCase(), phone);
                contacts.add(formatted);
                System.out.println("Contact saved!\n");

            } catch (InvalidContactException e) {
                System.out.println("Error: " + e.getMessage() + " — try again.\n");
            }
        }

        System.out.println("\n=== Saved Contacts ===");
        if (contacts.isEmpty()) {
            System.out.println("No contacts saved.");
        } else {
            contacts.forEach(System.out::println);
        }

        scanner.close();
    }
}
```

This program demonstrates:
- A custom exception class.
- Validation methods that throw exceptions.
- A loop that catches errors and lets the user retry.
- String trimming, case conversion, and formatting.
- Regex for email and phone validation.
- `StringBuilder` for name formatting.

---

## 14. Exercises

---

### Exercise 1 – Safe Number Parser

Write a method `safeParseInt(String input)` that:
- Returns the parsed integer if the input is valid.
- Returns `0` if the input is not a valid integer (catch `NumberFormatException`).

In `main`, test it with these inputs: `"42"`, `"hello"`, `"3.14"`, `"-7"`, `""`.

**Expected Output:**
```
"42"    → 42
"hello" → 0
"3.14"  → 0
"-7"    → -7
""      → 0
```

---

### Exercise 2 – Guarded Division

Write a method `divide(int a, int b)` that:
- Returns the result of `a / b` as a `double`.
- Throws a custom exception `DivisionByZeroException` if `b` is zero.

Create the `DivisionByZeroException` class. In `main`, test with several inputs including `b = 0`.

---

### Exercise 3 – String Statistics

Write a program that takes a sentence from the user and prints:
1. The number of characters (excluding spaces).
2. The number of words.
3. The number of vowels (a, e, i, o, u — any case).
4. The sentence reversed word by word (not character by character).
5. The sentence with every word capitalized.

**Example input:** `"the quick brown fox"`

**Expected output:**
```
Characters (no spaces): 16
Words: 4
Vowels: 5
Reversed words: fox brown quick the
Title case: The Quick Brown Fox
```

---

### Exercise 4 – Input Validator

Write a program that asks the user to enter:
1. An **email address** — must match the pattern `word@word.tld`.
2. A **ZIP code** — must be exactly 5 digits.
3. A **date** — must match `DD/MM/YYYY`.

Keep re-asking for each field until the user enters a valid value. Use `matches()` with regex for all three validations.

---

### Exercise 5 – CSV Parser

Write a program that processes this CSV string:

```java
String data = "Alice,30,Engineering\nBob,25,Marketing\nCarlos,35,Engineering\nDiana,28,HR";
```

Parse it and:
1. Print each person's name, age, and department.
2. Print only the people in the `"Engineering"` department.
3. Print the average age of all people.
4. Handle any line that has fewer than 3 fields by printing a warning and skipping it.

---

### Exercise 6 – Exception Chain *(Challenge)*

Create a three-level method chain:

- `method1()` calls `method2()`.
- `method2()` calls `method3()`.
- `method3()` throws an `IllegalArgumentException` with the message `"Invalid input in method3"`.

In `method2`, catch the exception, print `"method2 caught it and is rethrowing"`, then **rethrow** it (use `throw e`).

In `main`, catch it in a `try-catch` and print the final message.

Observe how the stack trace shows the full chain.

---

### Exercise 7 – StringBuilder Word Reversal *(Challenge)*

Write a method that takes a sentence and uses `StringBuilder` to:
1. Reverse the entire string character by character.
2. Then reverse each individual word back to normal.

The net effect: the words appear in reversed order but each word's letters are correct.

**Example:**
- Input:  `"Hello World Java"`
- Output: `"Java World Hello"`

**Hint:** You can do this by reversing the whole string, then using `split` and `StringBuilder` on each word, or by managing character positions manually.

---

## Module Summary

Here's what you covered in Module 1:

- **Exceptions** are Java's way of signalling that something went wrong at runtime.
- The `try-catch-finally` block handles exceptions gracefully without crashing.
- Use **multiple catch blocks** for different exception types — specific ones first.
- `throw` raises an exception; `throws` declares a method might throw one.
- **Custom exceptions** give errors meaningful, domain-specific names.
- **Checked exceptions** must be handled (compiler enforces it); **unchecked** ones are usually programming bugs.
- `String` is immutable — methods return new strings, never modify the original.
- Key String methods: `charAt`, `substring`, `split`, `replace`, `trim`, `contains`, `indexOf`, `format`.
- **Never compare strings with `==`** — always use `.equals()` or `.equalsIgnoreCase()`.
- **`StringBuilder`** is efficient for building strings in loops or with many concatenations.
- **Regular expressions** validate and transform strings using pattern matching.

---

**Next up:** Module 2 – Packages in Java

---

*CS 1103 Programming 2 (Java) — Module 1 of 8*
