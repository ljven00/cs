# CS 1102 – Programming 1 (Java)
## Module 4: Arrays and ArrayLists

---

## Table of Contents

1. [Why Do We Need Arrays?](#1-why-do-we-need-arrays)
2. [Declaring and Creating Arrays](#2-declaring-and-creating-arrays)
3. [Accessing and Modifying Array Elements](#3-accessing-and-modifying-array-elements)
4. [Looping Through Arrays](#4-looping-through-arrays)
5. [Common Array Operations](#5-common-array-operations)
6. [Multidimensional Arrays](#6-multidimensional-arrays)
7. [Limitations of Arrays](#7-limitations-of-arrays)
8. [Introducing ArrayList](#8-introducing-arraylist)
9. [Common ArrayList Operations](#9-common-arraylist-operations)
10. [Looping Through an ArrayList](#10-looping-through-an-arraylist)
11. [Array vs ArrayList — When to Use Which](#11-array-vs-arraylist--when-to-use-which)
12. [Exercises](#12-exercises)

---

## 1. Why Do We Need Arrays?

Suppose you want to store the scores of 5 students. Without arrays, you'd write:

```java
int score1 = 85;
int score2 = 92;
int score3 = 78;
int score4 = 95;
int score5 = 88;
```

That's manageable for 5 students. But what about 100? Or 1000?

An **array** lets you store many values of the same type under a single name, and access each one by its position number.

```java
int[] scores = {85, 92, 78, 95, 88};
```

Clean, compact, and easy to loop through.

---

## 2. Declaring and Creating Arrays

### Method 1 — Declare and initialize at the same time

```java
int[] scores = {85, 92, 78, 95, 88};
String[] names = {"Alice", "Bob", "Carlos"};
double[] prices = {1.99, 4.50, 12.00};
```

### Method 2 — Declare the size first, fill in values later

```java
int[] scores = new int[5];   // creates an array with 5 slots, all set to 0
```

Default values when you create an empty array:

| Type      | Default value |
|-----------|---------------|
| `int`     | `0`           |
| `double`  | `0.0`         |
| `boolean` | `false`       |
| `String`  | `null`        |

### Array Length

Every array has a `.length` property that tells you how many elements it holds:

```java
int[] scores = {85, 92, 78, 95, 88};
System.out.println(scores.length);   // 5
```

> **Note:** `.length` is a property, not a method — there are no parentheses after it.

---

## 3. Accessing and Modifying Array Elements

Each element in an array has an **index** — a number that identifies its position. Java arrays are **zero-indexed**, meaning the first element is at index `0`.

```
scores:  { 85,  92,  78,  95,  88 }
index:     0    1    2    3    4
```

### Reading a value

```java
int[] scores = {85, 92, 78, 95, 88};

System.out.println(scores[0]);   // 85
System.out.println(scores[2]);   // 78
System.out.println(scores[4]);   // 88
```

### Modifying a value

```java
scores[2] = 100;   // change the value at index 2
System.out.println(scores[2]);   // 100
```

### Out of Bounds Error

Accessing an index that doesn't exist causes an `ArrayIndexOutOfBoundsException` — one of the most common errors in Java:

```java
int[] scores = {85, 92, 78};
System.out.println(scores[5]);   // ERROR — index 5 doesn't exist
```

Always make sure your index is between `0` and `array.length - 1`.

---

## 4. Looping Through Arrays

Loops and arrays are a natural pair. The most common pattern is using a `for` loop with the array's index.

### Using a `for` loop

```java
int[] scores = {85, 92, 78, 95, 88};

for (int i = 0; i < scores.length; i++) {
    System.out.println("Score " + (i + 1) + ": " + scores[i]);
}
```

Output:
```
Score 1: 85
Score 2: 92
Score 3: 78
Score 4: 95
Score 5: 88
```

### Using a for-each loop

Java has a cleaner loop for when you just need the values (not the index):

```java
for (int score : scores) {
    System.out.println(score);
}
```

Read this as: *"for each `score` in `scores`"*. It's simpler, but you lose access to the index. Use it when the position doesn't matter.

---

## 5. Common Array Operations

### Finding the Maximum Value

```java
int[] scores = {85, 92, 78, 95, 88};
int max = scores[0];   // assume first is the largest to start

for (int i = 1; i < scores.length; i++) {
    if (scores[i] > max) {
        max = scores[i];
    }
}

System.out.println("Highest score: " + max);   // 95
```

### Calculating the Average

```java
int[] scores = {85, 92, 78, 95, 88};
int sum = 0;

for (int score : scores) {
    sum += score;
}

double average = (double) sum / scores.length;
System.out.println("Average: " + average);   // 87.6
```

> `(double) sum` is a **cast** — it temporarily treats `sum` as a decimal so the division gives a decimal result instead of a whole number.

### Filling an Array with User Input

```java
import java.util.Scanner;

public class FillArray {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int[] numbers = new int[3];

        for (int i = 0; i < numbers.length; i++) {
            System.out.print("Enter number " + (i + 1) + ": ");
            numbers[i] = scanner.nextInt();
        }

        System.out.println("You entered:");
        for (int number : numbers) {
            System.out.println(number);
        }

        scanner.close();
    }
}
```

### Sorting an Array

Java's built-in `Arrays` utility class can sort an array for you:

```java
import java.util.Arrays;

int[] scores = {85, 92, 78, 95, 88};
Arrays.sort(scores);

System.out.println(Arrays.toString(scores));   // [78, 85, 88, 92, 95]
```

- `Arrays.sort()` sorts the array **in place** (modifies the original).
- `Arrays.toString()` converts an array to a readable string like `[78, 85, 88, 92, 95]`.

---

## 6. Multidimensional Arrays

A **2D array** is like a table with rows and columns — useful for grids, matrices, and game boards.

### Declaring a 2D Array

```java
int[][] grid = {
    {1, 2, 3},
    {4, 5, 6},
    {7, 8, 9}
};
```

Think of it as an array of arrays:

```
grid[0] → {1, 2, 3}
grid[1] → {4, 5, 6}
grid[2] → {7, 8, 9}
```

### Accessing Elements

```java
System.out.println(grid[0][0]);   // 1  (row 0, column 0)
System.out.println(grid[1][2]);   // 6  (row 1, column 2)
System.out.println(grid[2][1]);   // 8  (row 2, column 1)
```

### Looping Through a 2D Array

```java
for (int row = 0; row < grid.length; row++) {
    for (int col = 0; col < grid[row].length; col++) {
        System.out.print(grid[row][col] + " ");
    }
    System.out.println();
}
```

Output:
```
1 2 3 
4 5 6 
7 8 9 
```

---

## 7. Limitations of Arrays

Arrays are fast and simple, but they have one major limitation: **their size is fixed when created**. You cannot add or remove elements after creation.

```java
int[] scores = new int[5];
// You're stuck with exactly 5 slots — no more, no less
```

If you need a collection that can **grow and shrink dynamically**, use an `ArrayList`.

---

## 8. Introducing ArrayList

An `ArrayList` works like an array but can change size automatically. To use it, you need to import it:

```java
import java.util.ArrayList;
```

### Creating an ArrayList

```java
ArrayList<String> names = new ArrayList<>();
```

The type in angle brackets (`<String>`) tells Java what kind of items the list holds. This is called a **generic type** — you'll learn more about generics in CS 1103.

You can create ArrayLists of any type:

```java
ArrayList<Integer> scores = new ArrayList<>();
ArrayList<Double> prices = new ArrayList<>();
ArrayList<String> cities = new ArrayList<>();
```

> **Note:** ArrayList cannot hold primitive types directly. Use `Integer` instead of `int`, `Double` instead of `double`, etc. Java automatically converts between them — this is called **autoboxing**.

### Adding Elements

```java
ArrayList<String> names = new ArrayList<>();
names.add("Alice");
names.add("Bob");
names.add("Carlos");

System.out.println(names);   // [Alice, Bob, Carlos]
```

---

## 9. Common ArrayList Operations

### Getting the Size

```java
System.out.println(names.size());   // 3
```

Use `.size()` for ArrayList (not `.length` like arrays).

### Accessing an Element

```java
System.out.println(names.get(0));   // Alice
System.out.println(names.get(2));   // Carlos
```

### Modifying an Element

```java
names.set(1, "Barbara");   // replaces index 1
System.out.println(names);   // [Alice, Barbara, Carlos]
```

### Removing an Element

```java
names.remove("Barbara");        // remove by value
System.out.println(names);      // [Alice, Carlos]

names.remove(0);                // remove by index
System.out.println(names);      // [Carlos]
```

### Checking if an Element Exists

```java
ArrayList<String> names = new ArrayList<>();
names.add("Alice");
names.add("Bob");

System.out.println(names.contains("Alice"));   // true
System.out.println(names.contains("Zara"));    // false
```

### Clearing the List

```java
names.clear();
System.out.println(names.size());   // 0
```

### Quick Reference Table

| Operation          | Array                  | ArrayList                   |
|--------------------|------------------------|-----------------------------|
| Get element        | `arr[i]`               | `list.get(i)`               |
| Set element        | `arr[i] = value`       | `list.set(i, value)`        |
| Get size/length    | `arr.length`           | `list.size()`               |
| Add element        | Not possible           | `list.add(value)`           |
| Remove element     | Not possible           | `list.remove(value/index)`  |
| Check if contains  | Manual loop            | `list.contains(value)`      |

---

## 10. Looping Through an ArrayList

### Using a regular `for` loop

```java
ArrayList<String> names = new ArrayList<>();
names.add("Alice");
names.add("Bob");
names.add("Carlos");

for (int i = 0; i < names.size(); i++) {
    System.out.println(i + ": " + names.get(i));
}
```

Output:
```
0: Alice
1: Bob
2: Carlos
```

### Using a for-each loop

```java
for (String name : names) {
    System.out.println("Hello, " + name + "!");
}
```

Output:
```
Hello, Alice!
Hello, Bob!
Hello, Carlos!
```

---

## 11. Array vs ArrayList — When to Use Which

| Use an `Array` when...                        | Use an `ArrayList` when...                     |
|-----------------------------------------------|------------------------------------------------|
| You know the exact number of items upfront    | The number of items can change                 |
| You need maximum performance                  | You need to add or remove items easily         |
| You're working with primitive types directly  | Convenience matters more than raw speed        |
| You're working with a fixed set (e.g. days of the week) | You're building a dynamic list (e.g. a shopping cart) |

In practice, `ArrayList` is used far more often in everyday Java programming because of its flexibility.

---

## 12. Exercises

---

### Exercise 1 – Array Basics

Write a program that:
1. Creates an array of 5 integers entered by the user.
2. Prints the array values.
3. Prints the **sum** and **average** of the values.

---

### Exercise 2 – Find Min and Max

Write a program that:
1. Creates an array: `{34, 17, 82, 5, 61, 43, 29}`.
2. Finds and prints the **minimum** and **maximum** values without using `Arrays.sort()`.

---

### Exercise 3 – Reverse an Array

Write a program that prints an array **in reverse order**.

Start with: `{10, 20, 30, 40, 50}`

Expected output:
```
50 40 30 20 10
```

**Hint:** Loop from `array.length - 1` down to `0`.

---

### Exercise 4 – ArrayList Shopping Cart

Write a program that simulates a simple shopping cart:
1. Start with an empty `ArrayList<String>`.
2. Add at least 4 items to it.
3. Print all items.
4. Remove one item by name.
5. Print the updated list and the total item count.

---

### Exercise 5 – Frequency Count

Write a program that:
1. Creates this array: `{3, 7, 3, 2, 7, 3, 8, 2, 7, 7}`.
2. Counts how many times `3`, `7`, and `2` appear.
3. Prints the result.

**Expected Output:**
```
3 appears 3 times
7 appears 4 times
2 appears 2 times
```

---

### Exercise 6 – Gradebook

Write a program that:
1. Asks the user how many students are in the class.
2. Reads a name and score for each student into two separate arrays.
3. Prints a formatted report showing each name and score.
4. Prints the class average at the end.

**Sample Output:**
```
Student Grades:
Alice   - 88
Bob     - 74
Carlos  - 95

Class Average: 85.67
```

---

### Exercise 7 – 2D Grid *(Challenge)*

Write a program that creates a 3x3 integer grid and:
1. Fills it with values 1 through 9.
2. Prints the grid in table format.
3. Prints the sum of each row.

**Expected Output:**
```
1 2 3   Row sum: 6
4 5 6   Row sum: 15
7 8 9   Row sum: 24
```

---

## Module Summary

Here's what you covered in Module 4:

- **Arrays** store multiple values of the same type under one name, with a fixed size.
- Array elements are accessed by **index**, starting at `0`.
- Use `array.length` to get the number of elements.
- Loop through arrays with a `for` loop (when index matters) or a **for-each** loop (when it doesn't).
- Common operations: finding max/min, summing, averaging, sorting with `Arrays.sort()`.
- **2D arrays** represent tables with rows and columns.
- **ArrayLists** are like resizable arrays — use `add()`, `get()`, `set()`, `remove()`, and `size()`.
- ArrayLists require wrapper types (`Integer`, `Double`) instead of primitives.
- Choose arrays for fixed-size, performance-critical data; choose ArrayLists for flexible, dynamic lists.

---

**Next up:** Module 5 – Objects and Classes

---

*CS 1102 Programming 1 (Java) — Module 4 of 8*
