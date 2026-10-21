# CS 1102 – Programming 1 (Java)
## Module 1: Introduction to Java Programming

---

## Table of Contents

1. [What is Java?](#1-what-is-java)
2. [How Java Works](#2-how-java-works)
3. [Setting Up Your Environment](#3-setting-up-your-environment)
4. [Your First Java Program](#4-your-first-java-program)
5. [Understanding the Code](#5-understanding-the-code)
6. [Variables and Data Types](#6-variables-and-data-types)
7. [Basic Input and Output](#7-basic-input-and-output)
8. [Comments in Java](#8-comments-in-java)
9. [Exercises](#9-exercises)

---

## 1. What is Java?

Java is a **general-purpose programming language** created by Sun Microsystems in 1995 (now maintained by Oracle). It is one of the most widely used languages in the world, powering everything from Android apps to large enterprise systems.

A few things make Java stand out:

- **Platform-independent** – Java programs can run on Windows, macOS, Linux, and more without changing your code. You'll understand exactly why in the next section.
- **Object-oriented** – Java organizes code around "objects", which makes large programs easier to manage. You'll learn about this in later modules.
- **Strongly typed** – Java requires you to declare what kind of data a variable holds. This catches many mistakes before your program even runs.
- **Widely used** – Java is used in Android development, banking software, web servers, and much more.

---

## 2. How Java Works

Most programming languages compile your code directly into instructions for your specific computer. Java takes a different approach:

```
Your Java Code (.java)
        ↓
   Java Compiler
        ↓
  Bytecode (.class)
        ↓
Java Virtual Machine (JVM)
        ↓
   Runs on any OS
```

1. You write Java code in a `.java` file.
2. The **Java Compiler** (`javac`) converts it into **bytecode** — a format no computer understands natively.
3. The **Java Virtual Machine (JVM)** reads that bytecode and runs it on whatever computer you're using.

This is why Java is often described as **"Write Once, Run Anywhere"** — the same bytecode runs on any machine that has a JVM installed.

---

## 3. Setting Up Your Environment

To write and run Java programs, you need two things:

### Step 1 – Install the JDK (Java Development Kit)

The JDK includes the compiler and everything else you need to build Java programs.

- Go to: https://adoptium.net (a free, community-maintained JDK)
- Download the **LTS (Long-Term Support)** version for your operating system.
- Run the installer and follow the steps.

To confirm it installed correctly, open a terminal (Command Prompt on Windows, Terminal on macOS/Linux) and type:

```bash
java -version
```

You should see something like:

```
openjdk version "21.0.1" ...
```

### Step 2 – Install a Code Editor

You can write Java in any text editor, but a dedicated IDE (Integrated Development Environment) makes life much easier. A popular free choice is:

- **IntelliJ IDEA Community Edition** – https://www.jetbrains.com/idea/download/
- **VS Code** with the Java Extension Pack – https://code.visualstudio.com/

For this course, either works well. IntelliJ is recommended because it's purpose-built for Java.

---

## 4. Your First Java Program

Let's write a program that prints a message to the screen. This is the classic "Hello, World!" program — a tradition when learning any new language.

Create a new file called `HelloWorld.java` and type the following:

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

### Running the Program

If you're using IntelliJ IDEA:
1. Create a new Java project.
2. Create a file named `HelloWorld.java` inside the `src` folder.
3. Paste the code above.
4. Click the green **Run** button (or press `Shift + F10`).

If you're using the terminal:
```bash
javac HelloWorld.java   # Compile
java HelloWorld         # Run
```

**Expected Output:**
```
Hello, World!
```

---

## 5. Understanding the Code

Let's break down each line of the Hello World program.

### `public class HelloWorld`

```java
public class HelloWorld {
```

- In Java, all code lives inside a **class**. Think of a class as a container for your code.
- The class name (`HelloWorld`) must exactly match the filename (`HelloWorld.java`) — capital letters included.
- `public` means this class is accessible from anywhere. You'll learn more about this in Module 5.

### `public static void main(String[] args)`

```java
public static void main(String[] args) {
```

This is the **main method** — the entry point of every Java program. When you run a Java program, the JVM looks for this exact line and starts executing from here.

- `public` – accessible from outside the class.
- `static` – belongs to the class itself, not a specific object. (More on this in Module 3.)
- `void` – this method doesn't return any value.
- `main` – the name Java looks for to start the program.
- `String[] args` – allows you to pass arguments from the command line. You can ignore this for now.

### `System.out.println(...)`

```java
System.out.println("Hello, World!");
```

This line **prints text to the screen**, followed by a new line.

- `System` – a built-in Java class.
- `out` – represents the standard output (your screen).
- `println` – short for "print line". It prints the text and moves to the next line.

To print without a new line, use `System.out.print(...)` instead.

### Semicolons and Curly Braces

- Every **statement** in Java ends with a semicolon `;`.
- **Curly braces** `{ }` mark the beginning and end of blocks of code (like the class body and method body).

---

## 6. Variables and Data Types

A **variable** is a named container that holds a value. Before using a variable in Java, you must declare its **type**.

### Declaring a Variable

```java
int age = 25;
```

This creates a variable named `age`, of type `int` (integer), with the value `25`.

### Common Data Types

| Type      | What it stores              | Example           |
|-----------|-----------------------------|-------------------|
| `int`     | Whole numbers               | `42`, `-7`, `0`   |
| `double`  | Decimal numbers             | `3.14`, `-0.5`    |
| `boolean` | True or false               | `true`, `false`   |
| `char`    | A single character          | `'A'`, `'z'`      |
| `String`  | A sequence of characters    | `"Hello"`         |

> **Note:** `String` starts with a capital letter because it's a class, not a primitive type. You'll learn the difference later.

### Examples

```java
int score = 100;
double price = 9.99;
boolean isStudent = true;
char grade = 'A';
String name = "Maria";
```

### Printing Variables

```java
String name = "Maria";
int age = 20;

System.out.println("Name: " + name);
System.out.println("Age: " + age);
```

Output:
```
Name: Maria
Age: 20
```

The `+` operator is used here to **concatenate** (join) text and variables together.

---

## 7. Basic Input and Output

So far, programs print output but don't receive any input. Let's learn how to read input from the user.

### Using the Scanner Class

Java provides a built-in class called `Scanner` that reads user input. To use it, you need to **import** it at the top of your file.

```java
import java.util.Scanner;

public class GreetUser {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.print("Enter your name: ");
        String name = scanner.nextLine();

        System.out.println("Hello, " + name + "!");

        scanner.close();
    }
}
```

**What's happening here:**

- `import java.util.Scanner;` — brings the Scanner class into your program.
- `new Scanner(System.in)` — creates a Scanner that reads from the keyboard.
- `scanner.nextLine()` — waits for the user to type something and press Enter, then stores it as a String.
- `scanner.close()` — good practice: closes the scanner when you're done.

**Sample Run:**
```
Enter your name: Carlos
Hello, Carlos!
```

### Reading Other Types of Input

| Method                | Reads               |
|-----------------------|---------------------|
| `scanner.nextLine()`  | A full line of text |
| `scanner.nextInt()`   | An integer number   |
| `scanner.nextDouble()`| A decimal number    |

Example with a number:

```java
import java.util.Scanner;

public class AddNumbers {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.print("Enter a number: ");
        int number = scanner.nextInt();

        System.out.println("You entered: " + number);
        System.out.println("Double that is: " + (number * 2));

        scanner.close();
    }
}
```

---

## 8. Comments in Java

Comments are notes in your code that Java ignores when running. They're for humans — to explain what the code does.

### Single-line Comment

```java
// This is a single-line comment
int x = 10; // You can also put comments at the end of a line
```

### Multi-line Comment

```java
/*
  This is a multi-line comment.
  Useful for longer explanations.
*/
int y = 20;
```

### Javadoc Comment

```java
/**
 * This style of comment is used to generate documentation.
 * You'll see it used in professional Java code.
 */
```

Good comments explain **why** something is done, not just what. Code usually makes the "what" clear enough on its own.

---

## 9. Exercises

Work through these exercises to test what you've learned. Try to write the code yourself before looking anything up.

---

### Exercise 1 – Hello, You!

Write a program that:
1. Asks the user for their name.
2. Asks the user for their age.
3. Prints a message like: `Hi Maria! You are 20 years old.`

**Hint:** Use `scanner.nextLine()` for the name and `scanner.nextInt()` for the age.

---

### Exercise 2 – Variable Practice

Declare variables to describe yourself:
- Your name (String)
- Your age (int)
- Your height in meters (double)
- Whether you are a student (boolean)

Print all four values using `System.out.println()`.

---

### Exercise 3 – Simple Calculator

Write a program that:
1. Asks the user to enter two whole numbers.
2. Prints their **sum**, **difference**, **product**, and **quotient**.

**Sample Output:**
```
Enter first number: 10
Enter second number: 4

Sum: 14
Difference: 6
Product: 40
Quotient: 2
```

**Hint:** Use `scanner.nextInt()` twice.

---

### Exercise 4 – Spot the Errors

The following code has **3 mistakes**. Find and fix them.

```java
public class BuggyProgram {
    public static void main(String[] args) {
        int number = 5
        String message = "The number is: "
        System.out.println(message + number)
    }
}
```

---

### Exercise 5 – Explore `print` vs `println`

Rewrite this code using `System.out.print()` instead of `System.out.println()` so that all the output appears on a **single line**:

```java
System.out.println("Java ");
System.out.println("is ");
System.out.println("fun!");
```

Expected single-line output:
```
Java is fun!
```

---

## Module Summary

Here's what you learned in Module 1:

- Java is platform-independent — it compiles to bytecode that runs on any JVM.
- Every Java program needs a `main` method as its starting point.
- Code is organized inside **classes**, which must match the filename.
- Java has several basic **data types**: `int`, `double`, `boolean`, `char`, and `String`.
- Use `System.out.println()` to print output.
- Use the `Scanner` class to read input from the user.
- Statements end with `;` and code blocks are wrapped in `{ }`.
- Comments (`//` and `/* */`) help document your code.

---

**Next up:** Module 2 – Control Structures (if/else, loops, and more)

---

*CS 1102 Programming 1 (Java) — Module 1 of 8*
