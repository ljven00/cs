# CS 1102 – Programming 1 (Java)
## Module 2: Control Structures

---

## Table of Contents

1. [What Are Control Structures?](#1-what-are-control-structures)
2. [The if Statement](#2-the-if-statement)
3. [if / else and if / else if](#3-if--else-and-if--else-if)
4. [Comparison and Logical Operators](#4-comparison-and-logical-operators)
5. [The switch Statement](#5-the-switch-statement)
6. [The while Loop](#6-the-while-loop)
7. [The for Loop](#7-the-for-loop)
8. [The do-while Loop](#8-the-do-while-loop)
9. [Controlling Loops: break and continue](#9-controlling-loops-break-and-continue)
10. [Nested Loops](#10-nested-loops)
11. [Exercises](#11-exercises)

---

## 1. What Are Control Structures?

By default, Java runs your code **line by line, from top to bottom**. Control structures let you change that flow. They allow your program to:

- **Make decisions** — run some code only if a condition is true.
- **Repeat actions** — run a block of code multiple times.

There are two categories:

| Category        | Examples                        |
|-----------------|---------------------------------|
| Decision-making | `if`, `if/else`, `switch`       |
| Loops           | `while`, `for`, `do-while`      |

---

## 2. The `if` Statement

The `if` statement runs a block of code **only if a condition is true**.

### Syntax

```java
if (condition) {
    // code to run if condition is true
}
```

### Example

```java
int temperature = 35;

if (temperature > 30) {
    System.out.println("It's a hot day!");
}
```

Output:
```
It's a hot day!
```

If `temperature` were `25`, nothing would be printed — the block is simply skipped.

---

## 3. `if` / `else` and `if` / `else if`

### `if` / `else`

Use `else` to run code when the condition is **false**.

```java
int temperature = 20;

if (temperature > 30) {
    System.out.println("It's a hot day!");
} else {
    System.out.println("It's a cool day.");
}
```

Output:
```
It's a cool day.
```

### `if` / `else if` / `else`

Use `else if` to check **multiple conditions** in sequence.

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
```

Output:
```
Grade: C
```

Java checks each condition from top to bottom and runs the **first** block where the condition is true. Once a match is found, the rest are skipped.

---

## 4. Comparison and Logical Operators

Conditions are built using **comparison operators** and **logical operators**.

### Comparison Operators

| Operator | Meaning                  | Example          |
|----------|--------------------------|------------------|
| `==`     | Equal to                 | `x == 5`         |
| `!=`     | Not equal to             | `x != 5`         |
| `>`      | Greater than             | `x > 10`         |
| `<`      | Less than                | `x < 10`         |
| `>=`     | Greater than or equal to | `x >= 10`        |
| `<=`     | Less than or equal to    | `x <= 10`        |

> ⚠️ Don't confuse `=` (assignment) with `==` (comparison). Writing `if (x = 5)` is a common mistake.

### Logical Operators

Use these to combine multiple conditions:

| Operator | Meaning | Example                        |
|----------|---------|--------------------------------|
| `&&`     | AND     | `age > 18 && hasID == true`    |
| `\|\|`   | OR      | `day == "Sat" \|\| day == "Sun"` |
| `!`      | NOT     | `!isRaining`                   |

### Example

```java
int age = 20;
boolean hasTicket = true;

if (age >= 18 && hasTicket) {
    System.out.println("Entry allowed.");
} else {
    System.out.println("Entry denied.");
}
```

Output:
```
Entry allowed.
```

---

## 5. The `switch` Statement

When you need to compare **one variable against many specific values**, `switch` is often cleaner than a long chain of `else if`.

### Syntax

```java
switch (variable) {
    case value1:
        // code
        break;
    case value2:
        // code
        break;
    default:
        // code if no case matches
}
```

### Example

```java
String day = "Monday";

switch (day) {
    case "Monday":
        System.out.println("Start of the work week.");
        break;
    case "Friday":
        System.out.println("Almost the weekend!");
        break;
    case "Saturday":
    case "Sunday":
        System.out.println("It's the weekend!");
        break;
    default:
        System.out.println("Midweek day.");
}
```

Output:
```
Start of the work week.
```

### Important: `break`

Without `break`, Java continues executing the **next case** even if it doesn't match. This is called **fall-through** and is usually a bug if unintentional. Always include `break` unless you specifically want fall-through behavior.

---

## 6. The `while` Loop

A `while` loop repeats a block of code **as long as a condition is true**.

### Syntax

```java
while (condition) {
    // code to repeat
}
```

### Example

```java
int count = 1;

while (count <= 5) {
    System.out.println("Count: " + count);
    count++; // count = count + 1
}
```

Output:
```
Count: 1
Count: 2
Count: 3
Count: 4
Count: 5
```

### How it works

1. Check the condition (`count <= 5`).
2. If true, run the block.
3. Go back to step 1.
4. When the condition becomes false, exit the loop.

> ⚠️ **Infinite loops:** If the condition never becomes false, the loop runs forever. Always make sure something inside the loop moves toward making the condition false.

---

## 7. The `for` Loop

The `for` loop is ideal when you know **exactly how many times** you want to repeat something.

### Syntax

```java
for (initialization; condition; update) {
    // code to repeat
}
```

### Example

```java
for (int i = 1; i <= 5; i++) {
    System.out.println("Line " + i);
}
```

Output:
```
Line 1
Line 2
Line 3
Line 4
Line 5
```

### Breaking it down

```java
for (int i = 1; i <= 5; i++)
//   ^^^^^^^^^^^  ^^^^^^  ^^^
//   Start at 1   Run     Add 1
//                while   each time
//                i <= 5
```

- **Initialization** (`int i = 1`) — runs once before the loop starts.
- **Condition** (`i <= 5`) — checked before each iteration; loop stops when false.
- **Update** (`i++`) — runs after each iteration.

### Counting Down

```java
for (int i = 5; i >= 1; i--) {
    System.out.println(i);
}
System.out.println("Go!");
```

Output:
```
5
4
3
2
1
Go!
```

---

## 8. The `do-while` Loop

The `do-while` loop is like a `while` loop, but it **always runs at least once** — the condition is checked *after* the block executes.

### Syntax

```java
do {
    // code to run
} while (condition);
```

### Example

```java
import java.util.Scanner;

public class GuessGame {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int number;

        do {
            System.out.print("Enter a number between 1 and 10: ");
            number = scanner.nextInt();
        } while (number < 1 || number > 10);

        System.out.println("You entered: " + number);
        scanner.close();
    }
}
```

This keeps asking until the user enters a valid number. Because we always need to ask at least once, `do-while` fits perfectly here.

---

## 9. Controlling Loops: `break` and `continue`

### `break` — Exit the loop immediately

```java
for (int i = 1; i <= 10; i++) {
    if (i == 5) {
        break; // Stop the loop when i reaches 5
    }
    System.out.println(i);
}
```

Output:
```
1
2
3
4
```

### `continue` — Skip the current iteration

```java
for (int i = 1; i <= 10; i++) {
    if (i % 2 == 0) {
        continue; // Skip even numbers
    }
    System.out.println(i);
}
```

Output:
```
1
3
5
7
9
```

> `i % 2` is the **modulo** (remainder) operator. `i % 2 == 0` means "i is even".

---

## 10. Nested Loops

A loop inside another loop is called a **nested loop**. The inner loop completes all its iterations for every single iteration of the outer loop.

### Example — Multiplication Table

```java
for (int i = 1; i <= 3; i++) {
    for (int j = 1; j <= 3; j++) {
        System.out.print(i * j + "\t"); // \t is a tab character
    }
    System.out.println(); // New line after each row
}
```

Output:
```
1	2	3	
2	4	6	
3	6	9	
```

### Example — Pattern Printing

```java
for (int row = 1; row <= 4; row++) {
    for (int col = 1; col <= row; col++) {
        System.out.print("* ");
    }
    System.out.println();
}
```

Output:
```
* 
* * 
* * * 
* * * * 
```

---

## 11. Exercises

---

### Exercise 1 – Even or Odd

Write a program that:
1. Asks the user to enter a whole number.
2. Prints whether the number is **even** or **odd**.

**Hint:** A number is even if `number % 2 == 0`.

---

### Exercise 2 – Letter Grade

Write a program that:
1. Asks the user for a numeric score (0–100).
2. Prints the corresponding letter grade using this scale:

| Score     | Grade |
|-----------|-------|
| 90 – 100  | A     |
| 80 – 89   | B     |
| 70 – 79   | C     |
| 60 – 69   | D     |
| Below 60  | F     |

---

### Exercise 3 – Countdown

Write a program using a `for` loop that counts down from 10 to 1 and then prints `"Liftoff!"`.

**Expected Output:**
```
10
9
8
...
1
Liftoff!
```

---

### Exercise 4 – Sum of Numbers

Write a program that:
1. Asks the user for a positive integer `n`.
2. Uses a loop to calculate the sum of all numbers from 1 to `n`.
3. Prints the result.

**Example:** If `n = 5`, the output should be `Sum: 15` (because 1+2+3+4+5 = 15).

---

### Exercise 5 – Number Guessing Game

Write a program where:
1. The "secret" number is set to `7` (hardcoded).
2. The user keeps guessing until they get it right.
3. After each wrong guess, print `"Too high!"` or `"Too low!"`.
4. When correct, print `"Correct! You got it in X tries."` where X is the number of attempts.

---

### Exercise 6 – Multiplication Table

Write a program that:
1. Asks the user for a number.
2. Prints its multiplication table from 1 to 10.

**Example output for input `3`:**
```
3 x 1 = 3
3 x 2 = 6
...
3 x 10 = 30
```

---

### Exercise 7 – FizzBuzz *(Challenge)*

Write a program that prints the numbers from 1 to 30, but:
- For multiples of 3, print `"Fizz"` instead of the number.
- For multiples of 5, print `"Buzz"` instead of the number.
- For multiples of **both** 3 and 5, print `"FizzBuzz"`.

**Expected output (first few lines):**
```
1
2
Fizz
4
Buzz
Fizz
7
...
FizzBuzz
```

**Hint:** Check the combined condition (`% 3 == 0 && % 5 == 0`) *first*.

---

## Module Summary

Here's what you learned in Module 2:

- `if / else if / else` — make decisions based on conditions.
- `switch` — cleanly compare one variable to many values.
- **Comparison operators** (`==`, `!=`, `>`, `<`, `>=`, `<=`) build conditions.
- **Logical operators** (`&&`, `||`, `!`) combine conditions.
- `while` — repeat while a condition is true (check before running).
- `for` — repeat a set number of times.
- `do-while` — repeat at least once (check after running).
- `break` exits a loop early; `continue` skips to the next iteration.
- Loops can be nested inside other loops.

---

**Next up:** Module 3 – Static Methods and Member Variables

---

*CS 1102 Programming 1 (Java) — Module 2 of 8*
