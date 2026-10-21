# CS 1102 – Programming 1 (Java)
## Module 3: Static Methods and Member Variables

---

## Table of Contents

1. [Why Break Code Into Methods?](#1-why-break-code-into-methods)
2. [Defining and Calling a Method](#2-defining-and-calling-a-method)
3. [Parameters and Arguments](#3-parameters-and-arguments)
4. [Return Values](#4-return-values)
5. [The `static` Keyword](#5-the-static-keyword)
6. [Member Variables (Fields)](#6-member-variables-fields)
7. [Static vs Instance Members](#7-static-vs-instance-members)
8. [Method Overloading](#8-method-overloading)
9. [Variable Scope](#9-variable-scope)
10. [Putting It All Together](#10-putting-it-all-together)
11. [Exercises](#11-exercises)

---

## 1. Why Break Code Into Methods?

Imagine writing a program that calculates a restaurant bill three times — once for lunch, once for dinner, once for a takeout order. Without methods, you'd copy and paste the same calculation logic three times. If you later needed to change it (say, to add a new tax rate), you'd have to update it in three places.

**Methods** solve this by letting you write a block of code once and reuse it by name as many times as you like.

Benefits of using methods:

- **Less repetition** — write once, use anywhere.
- **Easier to read** — a well-named method tells you what it does at a glance.
- **Easier to fix** — change the logic in one place, and it's fixed everywhere.
- **Easier to test** — you can test each method independently.

---

## 2. Defining and Calling a Method

### Defining a Method

A method definition has this structure:

```java
static returnType methodName(parameters) {
    // body
}
```

Here's the simplest possible method — one that takes nothing and returns nothing:

```java
static void greet() {
    System.out.println("Hello! Welcome to the program.");
}
```

- `static` — we'll explain this fully in Section 5.
- `void` — means the method doesn't return a value.
- `greet` — the name of the method.
- `()` — the parameter list (empty here).

### Calling a Method

To run a method, you **call** it by name followed by parentheses:

```java
public class MethodDemo {
    static void greet() {
        System.out.println("Hello! Welcome to the program.");
    }

    public static void main(String[] args) {
        greet();         // First call
        greet();         // Second call
        greet();         // Third call
    }
}
```

Output:
```
Hello! Welcome to the program.
Hello! Welcome to the program.
Hello! Welcome to the program.
```

The method runs each time it's called.

---

## 3. Parameters and Arguments

Methods become much more powerful when you can pass information into them.

- **Parameters** are the variables listed in the method definition.
- **Arguments** are the actual values you pass when calling the method.

### Example

```java
static void greetUser(String name) {
    System.out.println("Hello, " + name + "!");
}
```

Here, `name` is a **parameter** — a placeholder. When you call the method:

```java
greetUser("Sofia");   // "Sofia" is the argument
greetUser("James");
```

Output:
```
Hello, Sofia!
Hello, James!
```

### Multiple Parameters

```java
static void describeItem(String item, double price) {
    System.out.println(item + " costs $" + price);
}
```

```java
describeItem("Coffee", 3.50);
describeItem("Sandwich", 7.99);
```

Output:
```
Coffee costs $3.5
Sandwich costs $7.99
```

> **Important:** Arguments must match the parameter types **in order**. Passing a number where a String is expected will cause a compile error.

---

## 4. Return Values

A method can **compute something and send the result back** to whoever called it. Instead of `void`, you declare the type of value the method returns.

### Syntax

```java
static returnType methodName(parameters) {
    // compute something
    return value;
}
```

### Example — Returning an `int`

```java
static int square(int number) {
    return number * number;
}
```

```java
int result = square(5);
System.out.println("5 squared is: " + result);
System.out.println("7 squared is: " + square(7));
```

Output:
```
5 squared is: 25
7 squared is: 49
```

### Example — Returning a `double`

```java
static double calculateBill(double subtotal, double taxRate) {
    double tax = subtotal * taxRate;
    return subtotal + tax;
}
```

```java
double total = calculateBill(50.00, 0.08);
System.out.println("Total bill: $" + total);
```

Output:
```
Total bill: $54.0
```

### The `return` Statement

- `return` immediately exits the method and sends back the value.
- Every execution path in a non-void method must return a value.
- In a `void` method, you can use `return;` (with no value) to exit early.

---

## 5. The `static` Keyword

You've seen `static` in every method so far, including `main`. Let's understand what it means.

In Java, code lives inside classes. A **class** is like a blueprint. You can create many **objects** (instances) from that blueprint — each with its own data.

- A **static** method or variable **belongs to the class itself**.
- A **non-static** (instance) method or variable **belongs to a specific object**.

Because `main` is static, Java can run it without creating an object first. And because we've been calling our helper methods from `main`, those methods also need to be static (at least for now).

Think of it this way:

> A static method is like a function on a calculator — it doesn't remember anything between uses. You give it inputs, it gives you an output.

You'll fully explore the difference once we get to objects in Module 5. For now, mark your helper methods as `static` and you'll be fine.

---

## 6. Member Variables (Fields)

A **member variable** (also called a **field**) is a variable declared inside a class but **outside of any method**. Unlike local variables (which exist only inside a method), member variables are accessible throughout the class.

### Example

```java
public class Counter {
    static int count = 0;   // member variable

    static void increment() {
        count++;
    }

    static void printCount() {
        System.out.println("Count: " + count);
    }

    public static void main(String[] args) {
        increment();
        increment();
        increment();
        printCount();
    }
}
```

Output:
```
Count: 3
```

`count` is declared at the class level, so both `increment()` and `printCount()` can access and modify it. If `count` were declared inside `increment()`, it would be a local variable — gone after the method finishes, and invisible to `printCount()`.

---

## 7. Static vs Instance Members

Here's a clear comparison to build your mental model before you dive into full object-oriented programming in Module 5.

| Feature          | `static` member                        | Instance member (no `static`)        |
|------------------|----------------------------------------|--------------------------------------|
| Belongs to       | The class                              | A specific object                    |
| Shared?          | Yes — one copy for the whole class     | No — each object has its own copy    |
| How to access    | `ClassName.memberName`                 | `objectName.memberName`              |
| Needs an object? | No                                     | Yes                                  |

### Static Member Variable Example

```java
public class AppConfig {
    static String appName = "MyApp";
    static double version = 1.0;

    public static void main(String[] args) {
        System.out.println(AppConfig.appName + " v" + AppConfig.version);
    }
}
```

Output:
```
MyApp v1.0
```

`appName` and `version` are shared — there's only one copy regardless of how many objects you create from this class. This makes `static` useful for constants and shared counters.

---

## 8. Method Overloading

Java allows you to have **multiple methods with the same name**, as long as their **parameter lists are different** (different number or types of parameters). This is called **overloading**.

### Example

```java
static int add(int a, int b) {
    return a + b;
}

static double add(double a, double b) {
    return a + b;
}

static int add(int a, int b, int c) {
    return a + b + c;
}
```

```java
System.out.println(add(3, 4));           // Uses first method → 7
System.out.println(add(2.5, 1.5));       // Uses second method → 4.0
System.out.println(add(1, 2, 3));        // Uses third method → 6
```

Java figures out which version to call based on the arguments you pass. This is decided at **compile time**.

Overloading is useful when you want a method to handle different types of input without forcing the caller to use different names.

> **Note:** Overloading is based on parameters only — you cannot overload by changing just the return type.

---

## 9. Variable Scope

**Scope** refers to where in your code a variable can be accessed.

### Local Variables

Declared inside a method or block. They only exist within that block.

```java
static void demo() {
    int x = 10;   // local variable
    System.out.println(x);
}
// x does not exist here
```

### Block Scope

Variables declared inside `if`, `for`, or `while` blocks exist only within those blocks:

```java
for (int i = 0; i < 3; i++) {
    int doubled = i * 2;   // exists only inside this loop
    System.out.println(doubled);
}
// i and doubled are gone here
```

### Member Variables (Class-Level Scope)

Declared at the class level — accessible in all methods of that class:

```java
public class ScopeExample {
    static int total = 0;   // accessible everywhere in the class

    static void addFive() {
        total += 5;
    }

    static void showTotal() {
        System.out.println("Total: " + total);
    }

    public static void main(String[] args) {
        addFive();
        addFive();
        showTotal();
    }
}
```

Output:
```
Total: 10
```

### Shadowing (a common trap)

If a local variable has the **same name** as a member variable, the local one "shadows" (hides) the member variable within that scope:

```java
public class ShadowDemo {
    static int value = 100;   // member variable

    static void showValue() {
        int value = 42;   // local variable — shadows the member variable
        System.out.println(value);   // prints 42, not 100
    }

    public static void main(String[] args) {
        showValue();
        System.out.println(value);   // prints 100 — back to member variable
    }
}
```

Output:
```
42
100
```

To avoid confusion, give local variables and member variables different names.

---

## 10. Putting It All Together

Here's a complete example that combines methods, parameters, return values, and a member variable:

```java
import java.util.Scanner;

public class BillCalculator {
    static double taxRate = 0.08;   // member variable — shared tax rate

    static double calculateTax(double subtotal) {
        return subtotal * taxRate;
    }

    static double calculateTotal(double subtotal) {
        return subtotal + calculateTax(subtotal);
    }

    static void printReceipt(String item, double price) {
        double tax = calculateTax(price);
        double total = calculateTotal(price);
        System.out.println("Item:     " + item);
        System.out.println("Subtotal: $" + price);
        System.out.println("Tax:      $" + tax);
        System.out.println("Total:    $" + total);
        System.out.println("-------------------");
    }

    public static void main(String[] args) {
        printReceipt("Coffee", 3.50);
        printReceipt("Sandwich", 7.99);
    }
}
```

Output:
```
Item:     Coffee
Subtotal: $3.5
Tax:      $0.27999999999999997
Total:    $3.78
-------------------
Item:     Sandwich
Subtotal: $7.99
Tax:      $0.6392
Total:    $8.6292
-------------------
```

Notice how `taxRate` is defined once and used by multiple methods. If the tax rate changes, you update it in one place.

---

## 11. Exercises

---

### Exercise 1 – Simple Methods

Write a program with three methods:
- `printDivider()` — prints `"-------------------"` with no parameters and no return value.
- `square(int n)` — returns the square of `n`.
- `cube(int n)` — returns the cube of `n`.

In `main`, call `printDivider()`, then print the square and cube of the numbers 2, 4, and 6.

---

### Exercise 2 – Temperature Converter

Write two methods:
- `celsiusToFahrenheit(double celsius)` — converts and returns the Fahrenheit value.
  - Formula: `fahrenheit = (celsius * 9/5) + 32`
- `fahrenheitToCelsius(double fahrenheit)` — converts and returns the Celsius value.
  - Formula: `celsius = (fahrenheit - 32) * 5/9`

In `main`, ask the user for a temperature and whether it's in Celsius or Fahrenheit, then print the converted value.

---

### Exercise 3 – Min and Max

Write two methods:
- `max(int a, int b)` — returns the larger of the two numbers.
- `min(int a, int b)` — returns the smaller of the two numbers.

In `main`, ask the user to enter three numbers. Print the largest and smallest of the three.

**Hint:** You can call `max(max(a, b), c)` to find the max of three numbers.

---

### Exercise 4 – Method Overloading

Write an overloaded method called `describe`:
- `describe(String name)` — prints `"Name: [name]"`
- `describe(String name, int age)` — prints `"Name: [name], Age: [age]"`
- `describe(String name, int age, String city)` — prints `"Name: [name], Age: [age], City: [city]"`

Call all three versions from `main`.

---

### Exercise 5 – Running Total

Declare a **static member variable** called `total` initialized to `0`.

Write a method `addToTotal(int amount)` that adds the given amount to `total`.

In `main`, call `addToTotal` with values 10, 25, and 5. After each call, print the current total.

**Expected Output:**
```
Total after adding 10: 10
Total after adding 25: 35
Total after adding 5: 40
```

---

### Exercise 6 – Is It a Vowel? *(Challenge)*

Write a method `isVowel(char letter)` that returns `true` if the character is a vowel (a, e, i, o, u — upper or lower case) and `false` otherwise.

In `main`, ask the user to enter a word and count how many vowels it contains.

**Hint:** Use a `for` loop to go through each character with `word.charAt(i)`.

---

## Module Summary

Here's what you covered in Module 3:

- **Methods** let you write reusable blocks of code with a name.
- **Parameters** pass data into methods; **return values** send results back.
- `void` methods perform actions without returning a value.
- The `static` keyword means the method/variable belongs to the class, not an object.
- **Member variables** are declared at the class level and accessible throughout the class.
- **Method overloading** allows multiple methods with the same name but different parameters.
- **Scope** determines where a variable can be accessed — local, block, or class-level.

---

**Next up:** Module 4 – Arrays and ArrayLists

---

*CS 1102 Programming 1 (Java) — Module 3 of 8*
