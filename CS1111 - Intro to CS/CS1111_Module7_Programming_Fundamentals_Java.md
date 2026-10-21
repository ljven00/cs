# CS 1111 – Introduction to Computer Science
## Module 7: Programming Fundamentals (Java)

---

## Table of Contents
1. [What is Programming?](#1-what-is-programming)
2. [Setting Up Java](#2-setting-up-java)
3. [Your First Java Program](#3-your-first-java-program)
4. [Variables and Data Types](#4-variables-and-data-types)
5. [Operators](#5-operators)
6. [Control Flow](#6-control-flow)
7. [Loops](#7-loops)
8. [Methods](#8-methods)
9. [Arrays](#9-arrays)
10. [Module Summary](#10-module-summary)
11. [Module Quiz](#11-module-quiz)

---

## 1. What is Programming?

A **program** is a set of instructions that tells a computer what to do. **Programming** (or coding) is the process of writing those instructions in a language the computer can understand and execute.

Computers only understand machine language (binary — 0s and 1s), but writing in binary is impractical. **High-level programming languages** like Java let us write human-readable instructions that are then translated into machine code.

### Why Java?

Java is one of the most widely used programming languages in the world. Key reasons:

- **Platform independence:** Java code runs on any device with a Java Virtual Machine (JVM) — "Write Once, Run Anywhere."
- **Widely used:** Android apps, enterprise software, banking systems, web back-ends.
- **Beginner-friendly:** Strict, clear syntax helps build good habits.
- **Huge ecosystem:** Vast libraries, tools, and community support.

### How Java Code Runs

```
You write:    MyProgram.java   (source code — human readable)
      ↓  Java Compiler (javac)
Produces:     MyProgram.class  (bytecode — platform-neutral)
      ↓  Java Virtual Machine (JVM)
Executes on:  Any OS (Windows, macOS, Linux)
```

---

## 2. Setting Up Java

To write and run Java programs you need:

1. **JDK (Java Development Kit):** Download from [https://adoptium.net](https://adoptium.net) (free). Includes the compiler (`javac`) and runtime (`java`).
2. **A code editor or IDE:** Options include:
   - **IntelliJ IDEA** (recommended — free Community edition)
   - **VS Code** with the Java extension
   - **Eclipse**

Once installed, verify in your terminal:
```bash
java -version
javac -version
```

---

## 3. Your First Java Program

Every Java program starts from the same basic structure. Let's write the classic first program:

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

**Breaking it down line by line:**

| Line | Explanation |
|---|---|
| `public class HelloWorld` | Every Java program lives inside a **class**. The filename must match the class name (`HelloWorld.java`). |
| `public static void main(String[] args)` | The **main method** — the entry point. Java starts executing here. |
| `System.out.println(...)` | Prints text to the console, followed by a new line. |
| `{ }` | Curly braces define the beginning and end of a block of code. |

**To run it:**
```bash
javac HelloWorld.java   # Compile
java HelloWorld         # Run
# Output: Hello, World!
```

> **Note:** Java is **case-sensitive**. `System` is not the same as `system`. Every statement ends with a **semicolon** `;`.

---

## 4. Variables and Data Types

A **variable** is a named container that stores a value in memory. In Java, every variable must have a declared **type**.

### 4.1 Primitive Data Types

| Type | Size | Description | Example |
|---|---|---|---|
| `int` | 32-bit | Whole numbers | `int age = 20;` |
| `double` | 64-bit | Decimal numbers | `double gpa = 3.85;` |
| `boolean` | 1-bit | True or False | `boolean passed = true;` |
| `char` | 16-bit | A single character | `char grade = 'A';` |
| `long` | 64-bit | Large whole numbers | `long population = 8000000000L;` |

### 4.2 The String Type

`String` is not a primitive — it is an **object** that represents a sequence of characters. It is used so frequently it feels like a built-in type.

```java
String name = "Ana Martínez";
String city = "Monterrey";
```

### 4.3 Declaring and Using Variables

```java
public class Variables {
    public static void main(String[] args) {

        // Declaration and assignment
        int age = 21;
        double height = 1.68;
        boolean isStudent = true;
        char initial = 'A';
        String name = "Carlos";

        // Print variables
        System.out.println("Name: " + name);
        System.out.println("Age: " + age);
        System.out.println("Height: " + height + "m");
        System.out.println("Student: " + isStudent);
    }
}
```

**Output:**
```
Name: Carlos
Age: 21
Height: 1.68m
Student: true
```

### 4.4 Constants

A **constant** is a variable whose value cannot change after it is set. Use the `final` keyword:

```java
final double PI = 3.14159;
final int MAX_STUDENTS = 30;
```

By convention, constants are written in `ALL_CAPS_WITH_UNDERSCORES`.

### 4.5 Getting User Input

To read input from the user, use the `Scanner` class:

```java
import java.util.Scanner;  // Import Scanner

public class UserInput {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.print("Enter your name: ");
        String name = scanner.nextLine();

        System.out.print("Enter your age: ");
        int age = scanner.nextInt();

        System.out.println("Hello, " + name + "! You are " + age + " years old.");

        scanner.close();
    }
}
```

---

## 5. Operators

### 5.1 Arithmetic Operators

| Operator | Meaning | Example | Result |
|---|---|---|---|
| `+` | Addition | `5 + 3` | `8` |
| `-` | Subtraction | `10 - 4` | `6` |
| `*` | Multiplication | `3 * 4` | `12` |
| `/` | Division | `10 / 3` | `3` (integer division!) |
| `%` | Modulus (remainder) | `10 % 3` | `1` |

> **Important:** Dividing two `int` values discards the decimal. `10 / 3 = 3`, not `3.33`. To get a decimal result, use `double`: `10.0 / 3 = 3.33`.

```java
int a = 10, b = 3;
System.out.println(a + b);   // 13
System.out.println(a - b);   // 7
System.out.println(a * b);   // 30
System.out.println(a / b);   // 3  (not 3.33!)
System.out.println(a % b);   // 1  (remainder of 10 ÷ 3)
```

### 5.2 Comparison Operators

Return a `boolean` (`true` or `false`):

| Operator | Meaning | Example |
|---|---|---|
| `==` | Equal to | `5 == 5` → `true` |
| `!=` | Not equal to | `5 != 3` → `true` |
| `>` | Greater than | `7 > 3` → `true` |
| `<` | Less than | `2 < 9` → `true` |
| `>=` | Greater than or equal | `5 >= 5` → `true` |
| `<=` | Less than or equal | `4 <= 3` → `false` |

### 5.3 Logical Operators

| Operator | Meaning | Example |
|---|---|---|
| `&&` | AND — both must be true | `age >= 18 && hasID` |
| `\|\|` | OR — at least one must be true | `isAdmin \|\| isOwner` |
| `!` | NOT — inverts the value | `!isLoggedIn` |

### 5.4 Assignment Shorthand

```java
int x = 10;
x += 5;   // same as x = x + 5  →  x is now 15
x -= 3;   // x = x - 3          →  x is now 12
x *= 2;   // x = x * 2          →  x is now 24
x /= 4;   // x = x / 4          →  x is now 6
x++;      // x = x + 1          →  x is now 7
x--;      // x = x - 1          →  x is now 6
```

---

## 6. Control Flow

Control flow determines **which block of code runs** based on conditions.

### 6.1 if / else if / else

```java
int score = 75;

if (score >= 90) {
    System.out.println("Grade: A");
} else if (score >= 80) {
    System.out.println("Grade: B");
} else if (score >= 70) {
    System.out.println("Grade: C");
} else {
    System.out.println("Grade: F");
}
// Output: Grade: C
```

### 6.2 switch Statement

Useful when checking one variable against many possible values:

```java
int day = 3;

switch (day) {
    case 1:
        System.out.println("Monday");
        break;
    case 2:
        System.out.println("Tuesday");
        break;
    case 3:
        System.out.println("Wednesday");
        break;
    default:
        System.out.println("Another day");
}
// Output: Wednesday
```

> **Important:** Always include `break;` after each case, or execution will "fall through" to the next case.

### 6.3 Practical Example — Grade Calculator

```java
import java.util.Scanner;

public class GradeCalculator {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.print("Enter your score (0-100): ");
        int score = scanner.nextInt();

        String grade;
        if (score >= 90)      grade = "A";
        else if (score >= 80) grade = "B";
        else if (score >= 70) grade = "C";
        else if (score >= 60) grade = "D";
        else                  grade = "F";

        System.out.println("Your grade is: " + grade);
        scanner.close();
    }
}
```

---

## 7. Loops

Loops repeat a block of code multiple times, avoiding the need to write the same code over and over.

### 7.1 for Loop

Best when you know **exactly how many times** to repeat:

```java
// Print numbers 1 to 5
for (int i = 1; i <= 5; i++) {
    System.out.println("Count: " + i);
}
```

**Output:**
```
Count: 1
Count: 2
Count: 3
Count: 4
Count: 5
```

**Structure:** `for (initialization; condition; update)`
- **Initialization:** `int i = 1` — runs once at the start.
- **Condition:** `i <= 5` — checked before each iteration; loop stops when false.
- **Update:** `i++` — runs after each iteration.

### 7.2 while Loop

Best when you repeat **as long as a condition is true**, and you don't know in advance how many iterations:

```java
int number = 1;

while (number <= 5) {
    System.out.println("Number: " + number);
    number++;
}
```

> **Warning:** Always ensure the condition eventually becomes `false` — otherwise you create an **infinite loop** that never stops.

### 7.3 do-while Loop

Like `while`, but **always executes at least once** (checks condition after the first run):

```java
Scanner scanner = new Scanner(System.in);
int input;

do {
    System.out.print("Enter a positive number: ");
    input = scanner.nextInt();
} while (input <= 0);

System.out.println("You entered: " + input);
```

This keeps asking until the user enters a positive number — guaranteed to run at least once.

### 7.4 Loop Control: break and continue

```java
// break — exit the loop immediately
for (int i = 1; i <= 10; i++) {
    if (i == 5) break;          // Stop when i reaches 5
    System.out.println(i);      // Prints 1, 2, 3, 4
}

// continue — skip the current iteration, continue the loop
for (int i = 1; i <= 5; i++) {
    if (i == 3) continue;       // Skip 3
    System.out.println(i);      // Prints 1, 2, 4, 5
}
```

### 7.5 Practical Example — Multiplication Table

```java
public class MultiplicationTable {
    public static void main(String[] args) {
        int n = 5;

        for (int i = 1; i <= 10; i++) {
            System.out.println(n + " x " + i + " = " + (n * i));
        }
    }
}
```

**Output:**
```
5 x 1 = 5
5 x 2 = 10
...
5 x 10 = 50
```

---

## 8. Methods

A **method** is a named, reusable block of code that performs a specific task. Methods help organize code, avoid repetition, and make programs easier to read and maintain.

### 8.1 Defining and Calling a Method

```java
public class Methods {

    // Method definition
    public static void greet(String name) {
        System.out.println("Hello, " + name + "!");
    }

    public static void main(String[] args) {
        greet("Ana");      // Method call
        greet("Carlos");   // Reuse the same method
        greet("Sofía");
    }
}
```

**Output:**
```
Hello, Ana!
Hello, Carlos!
Hello, Sofía!
```

### 8.2 Methods That Return a Value

```java
public class Calculator {

    // This method takes two ints and returns their sum
    public static int add(int a, int b) {
        return a + b;
    }

    public static double average(int a, int b) {
        return (a + b) / 2.0;
    }

    public static void main(String[] args) {
        int result = add(10, 5);
        System.out.println("Sum: " + result);           // Sum: 15
        System.out.println("Average: " + average(10, 5)); // Average: 7.5
    }
}
```

**Method anatomy:**
```
public static int add(int a, int b)
│      │      │   │   └──────────┘
│      │      │   │    Parameters (inputs)
│      │      │   └── Method name
│      │      └─────── Return type (int)
│      └────────────── static (belongs to the class, not an object)
└───────────────────── Access modifier
```

Use `void` as the return type when the method doesn't return anything.

### 8.3 Practical Example — Temperature Converter

```java
public class TempConverter {

    public static double celsiusToFahrenheit(double celsius) {
        return (celsius * 9.0 / 5.0) + 32;
    }

    public static double fahrenheitToCelsius(double fahrenheit) {
        return (fahrenheit - 32) * 5.0 / 9.0;
    }

    public static void main(String[] args) {
        System.out.println("25°C = " + celsiusToFahrenheit(25) + "°F");
        System.out.println("98.6°F = " + fahrenheitToCelsius(98.6) + "°C");
    }
}
```

**Output:**
```
25°C = 77.0°F
98.6°F = 37.0°C
```

---

## 9. Arrays

An **array** stores multiple values of the same type in a single variable, accessible by an **index** (starting at 0).

### 9.1 Declaring and Using Arrays

```java
// Declare an array of 5 integers
int[] scores = {85, 92, 78, 95, 88};

// Access elements by index (starts at 0)
System.out.println(scores[0]);  // 85
System.out.println(scores[4]);  // 88

// Get the length of the array
System.out.println(scores.length);  // 5

// Modify an element
scores[2] = 80;
```

> **Important:** Array indices start at **0**, not 1. An array of 5 elements has indices 0, 1, 2, 3, 4. Accessing `scores[5]` causes an `ArrayIndexOutOfBoundsException` — a common beginner mistake.

### 9.2 Iterating Over an Array

```java
int[] scores = {85, 92, 78, 95, 88};

// Using a for loop
for (int i = 0; i < scores.length; i++) {
    System.out.println("Score " + (i + 1) + ": " + scores[i]);
}

// Using an enhanced for-each loop (cleaner)
for (int score : scores) {
    System.out.println(score);
}
```

### 9.3 Practical Example — Class Average

```java
public class ClassAverage {

    public static double calculateAverage(int[] grades) {
        int sum = 0;
        for (int grade : grades) {
            sum += grade;
        }
        return (double) sum / grades.length;
    }

    public static void main(String[] args) {
        int[] grades = {78, 85, 92, 88, 76, 95, 83};

        double average = calculateAverage(grades);
        System.out.printf("Class average: %.2f%n", average);
    }
}
```

**Output:**
```
Class average: 85.29
```

---

## 10. Module Summary

Key takeaways from Module 7:

- A **program** is a set of instructions for a computer. Java is **compiled to bytecode** and run by the JVM, making it platform-independent.
- Every Java program has a **class** and a `main` method — the entry point of execution.
- **Variables** store data and have a type (`int`, `double`, `boolean`, `char`, `String`). Constants use `final`.
- **Operators** perform arithmetic (`+`, `-`, `*`, `/`, `%`), comparisons (`==`, `>`, `<`), and logic (`&&`, `||`, `!`).
- **Control flow** with `if/else if/else` and `switch` lets the program make decisions.
- **Loops** (`for`, `while`, `do-while`) repeat code. `break` exits a loop; `continue` skips an iteration.
- **Methods** are reusable named blocks of code. They can accept **parameters** and **return** values.
- **Arrays** store multiple values of the same type, accessed by a zero-based index.

---

## 11. Module Quiz

---

**Question 1**
Which of the following correctly declares an integer variable named `age` with a value of 20?

- A) `int age = 20;`
- B) `integer age = 20;`
- C) `Age int = 20;`
- D) `int = age 20;`

---

**Question 2**
What is the output of the following code?
```java
int a = 10;
int b = 3;
System.out.println(a % b);
```

- A) 3
- B) 3.33
- C) 1
- D) 0

---

**Question 3**
What is the output of this code?
```java
int score = 85;
if (score >= 90) {
    System.out.println("A");
} else if (score >= 80) {
    System.out.println("B");
} else {
    System.out.println("C");
}
```

- A) A
- B) B
- C) C
- D) No output

---

**Question 4**
How many times will the following loop print "Hello"?
```java
for (int i = 0; i < 4; i++) {
    System.out.println("Hello");
}
```

- A) 3
- B) 4
- C) 5
- D) Infinite

---

**Question 5**
What is the value of `scores[2]` in the following array?
```java
int[] scores = {10, 20, 30, 40, 50};
```

- A) 20
- B) 40
- C) 30
- D) 10

---

**Question 6**
What keyword is used to send a value back from a method to the caller?

- A) `send`
- B) `output`
- C) `return`
- D) `result`

---

**Question 7**
Which loop is **guaranteed to execute its body at least once**, even if the condition is false from the start?

- A) `for` loop
- B) `while` loop
- C) `do-while` loop
- D) `foreach` loop

---

**Question 8** *(Short Answer)*
Write a Java method called `isEven` that takes an integer as a parameter and returns `true` if the number is even, `false` if it is odd. Then show how you would call it in `main` for the number 8.

---

### Answer Key

| Question | Answer | Explanation |
|---|---|---|
| 1 | **A** | Java syntax: `type variableName = value;`. The type keyword is `int` (lowercase), and the declaration ends with `;`. |
| 2 | **C** | The `%` operator returns the **remainder** of division. `10 ÷ 3 = 3` remainder **1**. |
| 3 | **B** | `score = 85`. The first condition `85 >= 90` is false. The second `85 >= 80` is **true** → prints "B". The `else` is skipped. |
| 4 | **B** | `i` starts at 0 and runs while `i < 4`. Iterations: i=0, i=1, i=2, i=3 → **4 times**. |
| 5 | **C** | Arrays are zero-indexed. Index 0=10, index 1=20, index **2=30**, index 3=40, index 4=50. |
| 6 | **C** | The `return` keyword sends a value back to the caller and exits the method. |
| 7 | **C** | The `do-while` loop executes the body **first**, then checks the condition — guaranteeing at least one execution. |
| 8 | *(See below)* | |

**Question 8 – Sample Answer:**
```java
public static boolean isEven(int number) {
    return number % 2 == 0;
}

public static void main(String[] args) {
    boolean result = isEven(8);
    System.out.println("Is 8 even? " + result);  // Is 8 even? true
}
```
> If `number % 2 == 0`, the remainder when dividing by 2 is zero — meaning the number is even. The method returns that boolean expression directly.

---

*End of Module 7 — Up next: **Module 8: Overview of Emerging Trends***
