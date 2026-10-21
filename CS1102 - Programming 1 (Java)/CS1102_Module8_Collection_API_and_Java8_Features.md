# CS 1102 – Programming 1 (Java)
## Module 8: Collection API and Java 8 Features

---

## Table of Contents

1. [The Collection Framework](#1-the-collection-framework)
2. [List — Ordered and Indexed](#2-list--ordered-and-indexed)
3. [Set — No Duplicates](#3-set--no-duplicates)
4. [Map — Key-Value Pairs](#4-map--key-value-pairs)
5. [The Collections Utility Class](#5-the-collections-utility-class)
6. [Choosing the Right Collection](#6-choosing-the-right-collection)
7. [Java 8 — Lambda Expressions](#7-java-8--lambda-expressions)
8. [Java 8 — The Stream API](#8-java-8--the-stream-api)
9. [Java 8 — The Optional Class](#9-java-8--the-optional-class)
10. [Java 8 — New Date and Time API](#10-java-8--new-date-and-time-api)
11. [Putting It All Together](#11-putting-it-all-together)
12. [Exercises](#12-exercises)
13. [Course Wrap-Up](#13-course-wrap-up)

---

## 1. The Collection Framework

In Module 4 you learned about `ArrayList` — a resizable list. The **Java Collection Framework** is a whole family of data structures that go much further. Each one is designed to store groups of objects efficiently, but with different characteristics.

The framework is built around a set of **interfaces** (remember those from Module 6?). The main ones are:

```
Collection
├── List      → Ordered, allows duplicates, accessed by index
├── Set       → No duplicates, no guaranteed order
└── Queue     → Elements processed in order (first in, first out)

Map           → Key-value pairs (not a Collection, but part of the framework)
```

The most commonly used implementations:

| Interface | Common Class | Characteristics |
|---|---|---|
| `List` | `ArrayList` | Fast to read by index, slow to insert in the middle |
| `List` | `LinkedList` | Fast to insert/delete anywhere, slower random access |
| `Set` | `HashSet` | Fast lookups, no duplicates, no guaranteed order |
| `Set` | `TreeSet` | No duplicates, always sorted |
| `Map` | `HashMap` | Key-value pairs, fast lookups, no guaranteed order |
| `Map` | `TreeMap` | Key-value pairs, keys always sorted |

You already know `ArrayList` well. This module focuses on `Set`, `Map`, and then the powerful Java 8 features that work beautifully with all of them.

---

## 2. List — Ordered and Indexed

You know `ArrayList` from Module 4. Here's a quick recap of the interface that unites all lists, plus a look at `LinkedList`.

### Programming to the Interface

Rather than declaring your variable as `ArrayList`, it's a best practice to use the `List` interface type:

```java
List<String> names = new ArrayList<>();
```

This means you could swap `ArrayList` for `LinkedList` later without changing any other code — only that one line. This is the benefit of programming to an interface.

### LinkedList

`LinkedList` stores elements as a chain of nodes, where each node points to the next. It is ideal when you frequently insert or delete elements in the middle of a list.

```java
import java.util.LinkedList;
import java.util.List;

List<String> queue = new LinkedList<>();
queue.add("Alice");
queue.add("Bob");
queue.add("Carlos");

queue.remove(0);              // fast removal from front
System.out.println(queue);    // [Bob, Carlos]
```

For most everyday uses, `ArrayList` is the right choice. Reach for `LinkedList` only when you need frequent insertions or deletions at arbitrary positions.

---

## 3. Set — No Duplicates

A **Set** is a collection that **does not allow duplicate elements**. It is the right tool when you care about whether something exists in a group, not how many times or in what order.

### HashSet

The most common Set implementation. Order is not guaranteed — elements may appear in any sequence.

```java
import java.util.HashSet;
import java.util.Set;

Set<String> countries = new HashSet<>();
countries.add("Mexico");
countries.add("Canada");
countries.add("Brazil");
countries.add("Mexico");   // duplicate — silently ignored

System.out.println(countries);        // [Brazil, Canada, Mexico] (order may vary)
System.out.println(countries.size()); // 3, not 4
```

### Checking Membership

`contains()` on a `HashSet` is extremely fast — much faster than searching through a list:

```java
System.out.println(countries.contains("Mexico"));   // true
System.out.println(countries.contains("France"));   // false
```

### TreeSet — Sorted Set

If you need elements stored in sorted order, use `TreeSet`:

```java
import java.util.TreeSet;

Set<Integer> numbers = new TreeSet<>();
numbers.add(42);
numbers.add(7);
numbers.add(19);
numbers.add(7);   // duplicate — ignored

System.out.println(numbers);   // [7, 19, 42] — always sorted
```

### Set Operations

Sets support classic mathematical set operations:

```java
Set<String> setA = new HashSet<>(List.of("apple", "banana", "cherry"));
Set<String> setB = new HashSet<>(List.of("banana", "cherry", "date"));

// Intersection — elements in both
Set<String> intersection = new HashSet<>(setA);
intersection.retainAll(setB);
System.out.println(intersection);   // [banana, cherry]

// Union — all elements from both
Set<String> union = new HashSet<>(setA);
union.addAll(setB);
System.out.println(union);   // [apple, banana, cherry, date]

// Difference — in A but not B
Set<String> difference = new HashSet<>(setA);
difference.removeAll(setB);
System.out.println(difference);   // [apple]
```

---

## 4. Map — Key-Value Pairs

A **Map** stores pairs of keys and values. Every key is unique — looking up a key gives you its associated value instantly. Think of it like a dictionary: you look up a word (key) and get its definition (value).

### HashMap

```java
import java.util.HashMap;
import java.util.Map;

Map<String, Integer> scores = new HashMap<>();
scores.put("Alice", 95);
scores.put("Bob", 78);
scores.put("Carlos", 88);

System.out.println(scores.get("Alice"));    // 95
System.out.println(scores.get("Bob"));      // 78
System.out.println(scores.get("Zara"));     // null — key doesn't exist
```

### Common Map Methods

| Method | What it does |
|---|---|
| `put(key, value)` | Adds or updates a key-value pair |
| `get(key)` | Returns the value for the key, or `null` |
| `getOrDefault(key, default)` | Returns the value, or a default if key is absent |
| `containsKey(key)` | Returns `true` if the key exists |
| `containsValue(value)` | Returns `true` if the value exists |
| `remove(key)` | Removes the entry for that key |
| `size()` | Number of key-value pairs |
| `keySet()` | Returns all keys as a `Set` |
| `values()` | Returns all values as a `Collection` |
| `entrySet()` | Returns all key-value pairs as a `Set` |

### Iterating Over a Map

```java
// Loop through all key-value pairs
for (Map.Entry<String, Integer> entry : scores.entrySet()) {
    System.out.println(entry.getKey() + " → " + entry.getValue());
}
```

Output:
```
Alice → 95
Bob → 78
Carlos → 88
```

### getOrDefault

Avoids a `null` check when a key might not exist:

```java
int zara = scores.getOrDefault("Zara", 0);
System.out.println(zara);   // 0
```

### Word Frequency Counter

A Map is perfect for counting how many times each word appears:

```java
String sentence = "the cat sat on the mat the cat";
String[] words = sentence.split(" ");

Map<String, Integer> frequency = new HashMap<>();

for (String word : words) {
    int count = frequency.getOrDefault(word, 0);
    frequency.put(word, count + 1);
}

System.out.println(frequency);
// {the=3, cat=2, sat=1, on=1, mat=1}
```

### TreeMap — Sorted by Key

```java
Map<String, Integer> sorted = new TreeMap<>(scores);
System.out.println(sorted);   // {Alice=95, Bob=78, Carlos=88} — sorted by key
```

---

## 5. The Collections Utility Class

`java.util.Collections` (note the 's') is a utility class full of static methods that operate on collections. It's different from the `Collection` interface.

```java
import java.util.Collections;
import java.util.List;
import java.util.ArrayList;

List<Integer> numbers = new ArrayList<>(List.of(3, 1, 4, 1, 5, 9, 2, 6));

Collections.sort(numbers);
System.out.println(numbers);               // [1, 1, 2, 3, 4, 5, 6, 9]

System.out.println(Collections.max(numbers));  // 9
System.out.println(Collections.min(numbers));  // 1

Collections.reverse(numbers);
System.out.println(numbers);               // [9, 6, 5, 4, 3, 2, 1, 1]

Collections.shuffle(numbers);
System.out.println(numbers);               // random order

Collections.sort(numbers, Collections.reverseOrder());
System.out.println(numbers);               // [9, 6, 5, 4, 3, 2, 1, 1]
```

---

## 6. Choosing the Right Collection

| Question | Use |
|---|---|
| Do I need indexed access (get by position)? | `ArrayList` |
| Do I need frequent inserts/deletes in the middle? | `LinkedList` |
| Do I just need to check if something exists? | `HashSet` |
| Do I need unique elements in sorted order? | `TreeSet` |
| Do I need to look up values by a key? | `HashMap` |
| Do I need key-value pairs sorted by key? | `TreeMap` |

---

## 7. Java 8 — Lambda Expressions

You briefly saw lambda expressions in Module 7 with button listeners. Let's understand them fully.

A **lambda expression** is a short, anonymous function — a method without a name that you can pass around as a value.

### The Problem They Solve

Before lambdas, to pass a small piece of behavior (like a sort rule) you had to write a full anonymous inner class:

```java
// Old way — verbose
Collections.sort(names, new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return a.compareTo(b);
    }
});
```

With a lambda:

```java
// New way — concise
Collections.sort(names, (a, b) -> a.compareTo(b));
```

### Lambda Syntax

```java
(parameters) -> expression
(parameters) -> { statements; }
```

Examples:

```java
// No parameters
() -> System.out.println("Hello!")

// One parameter (parentheses optional for single param)
name -> System.out.println("Hello, " + name)

// Two parameters
(a, b) -> a + b

// Multi-line body
(a, b) -> {
    int sum = a + b;
    return sum * 2;
}
```

### Functional Interfaces

Lambdas work with **functional interfaces** — interfaces with exactly one abstract method. Java 8 provides several built-in ones in `java.util.function`:

| Interface | Method | Takes | Returns | Use for |
|---|---|---|---|---|
| `Runnable` | `run()` | Nothing | Nothing | Running a task |
| `Consumer<T>` | `accept(T)` | One value | Nothing | Doing something with a value |
| `Supplier<T>` | `get()` | Nothing | A value | Producing a value |
| `Function<T,R>` | `apply(T)` | One value | Another value | Transforming a value |
| `Predicate<T>` | `test(T)` | One value | `boolean` | Testing a condition |

```java
import java.util.function.*;

Predicate<Integer> isEven = n -> n % 2 == 0;
System.out.println(isEven.test(4));    // true
System.out.println(isEven.test(7));    // false

Function<String, Integer> strLength = s -> s.length();
System.out.println(strLength.apply("Hello"));    // 5

Consumer<String> printer = s -> System.out.println(">> " + s);
printer.accept("Java 8 is great");    // >> Java 8 is great
```

### forEach with Lambda

Every collection now has a `forEach` method that takes a `Consumer`:

```java
List<String> names = List.of("Alice", "Bob", "Carlos");
names.forEach(name -> System.out.println("Hello, " + name));
```

Output:
```
Hello, Alice
Hello, Bob
Hello, Carlos
```

---

## 8. Java 8 — The Stream API

The **Stream API** is one of Java 8's most powerful features. A **stream** is a sequence of elements you can process with a pipeline of operations — filtering, transforming, sorting, and collecting — in a clean, readable style.

A stream does **not** modify the original collection. It produces a new result.

### Creating a Stream

```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
numbers.stream()   // create a stream from the list
```

### The Three Parts of a Stream Pipeline

```
Source → Intermediate Operations → Terminal Operation
```

- **Source**: where the data comes from (`list.stream()`).
- **Intermediate operations**: transform the stream — `filter`, `map`, `sorted`, `distinct`, `limit`. These are **lazy** — they do nothing until a terminal operation is called.
- **Terminal operation**: triggers processing and produces a result — `collect`, `forEach`, `count`, `sum`, `findFirst`.

### filter — Keep elements that match a condition

```java
List<Integer> evens = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());

System.out.println(evens);   // [2, 4, 6, 8, 10]
```

### map — Transform each element

```java
List<String> names = List.of("alice", "bob", "carlos");

List<String> upper = names.stream()
    .map(name -> name.toUpperCase())
    .collect(Collectors.toList());

System.out.println(upper);   // [ALICE, BOB, CARLOS]
```

### sorted — Sort elements

```java
List<Integer> sorted = List.of(5, 2, 8, 1, 9, 3).stream()
    .sorted()
    .collect(Collectors.toList());

System.out.println(sorted);   // [1, 2, 3, 5, 8, 9]
```

### Chaining Operations

The real power is chaining multiple operations:

```java
List<String> names = List.of("Alice", "Bob", "Anna", "Carlos", "Amy");

List<String> result = names.stream()
    .filter(name -> name.startsWith("A"))   // keep names starting with A
    .sorted()                                // sort alphabetically
    .collect(Collectors.toList());

System.out.println(result);   // [Alice, Amy, Anna]
```

### count, sum, average — Aggregate results

```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

long count = numbers.stream()
    .filter(n -> n > 5)
    .count();

System.out.println(count);   // 5

int sum = numbers.stream()
    .mapToInt(Integer::intValue)
    .sum();

System.out.println(sum);     // 55

double avg = numbers.stream()
    .mapToInt(Integer::intValue)
    .average()
    .orElse(0);

System.out.println(avg);     // 5.5
```

### forEach on a Stream

```java
names.stream()
    .filter(name -> name.length() > 3)
    .forEach(name -> System.out.println(name));
```

### Method References — An Even Shorter Lambda

When a lambda just calls an existing method, you can use a **method reference** with `::`:

```java
names.forEach(System.out::println);           // instead of name -> System.out.println(name)
names.stream().map(String::toUpperCase)...    // instead of s -> s.toUpperCase()
```

---

## 9. Java 8 — The Optional Class

Calling `.get("Zara")` on a Map that has no "Zara" key returns `null`. Using that `null` without checking causes a `NullPointerException` — one of the most common Java errors.

**`Optional<T>`** is a container that either holds a value or is explicitly empty. It forces you to think about the "no value" case.

```java
import java.util.Optional;

Optional<String> found = Optional.of("Hello");
Optional<String> empty = Optional.empty();

System.out.println(found.isPresent());        // true
System.out.println(empty.isPresent());        // false

System.out.println(found.get());              // Hello
System.out.println(empty.orElse("Default"));  // Default
```

### Optional with Streams

Stream operations like `findFirst()` return an `Optional`:

```java
List<String> names = List.of("Alice", "Bob", "Carlos");

Optional<String> first = names.stream()
    .filter(name -> name.startsWith("C"))
    .findFirst();

String result = first.orElse("No match found");
System.out.println(result);   // Carlos
```

---

## 10. Java 8 — New Date and Time API

Before Java 8, working with dates was notoriously awkward. Java 8 introduced a clean new API in `java.time`.

### Key Classes

| Class | Use |
|---|---|
| `LocalDate` | A date (year, month, day) — no time, no timezone |
| `LocalTime` | A time (hour, minute, second) — no date |
| `LocalDateTime` | Date and time together |
| `Period` | A span of time in days/months/years |

### LocalDate

```java
import java.time.LocalDate;

LocalDate today = LocalDate.now();
System.out.println(today);              // 2026-05-14 (today's date)

LocalDate birthday = LocalDate.of(1995, 8, 20);
System.out.println(birthday);          // 1995-08-20

System.out.println(today.getYear());   // 2026
System.out.println(today.getMonth());  // MAY
System.out.println(today.getDayOfWeek()); // WEDNESDAY
```

### Date Arithmetic

```java
LocalDate nextWeek     = today.plusDays(7);
LocalDate lastMonth    = today.minusMonths(1);
LocalDate nextYear     = today.plusYears(1);

System.out.println(nextWeek);    // 2026-05-21
System.out.println(lastMonth);   // 2026-04-14
```

### Comparing Dates

```java
LocalDate date1 = LocalDate.of(2025, 1, 1);
LocalDate date2 = LocalDate.of(2026, 1, 1);

System.out.println(date1.isBefore(date2));   // true
System.out.println(date1.isAfter(date2));    // false
System.out.println(date1.isEqual(date2));    // false
```

### Period — Duration Between Two Dates

```java
import java.time.Period;

LocalDate start = LocalDate.of(2000, 6, 15);
LocalDate end   = LocalDate.now();

Period age = Period.between(start, end);
System.out.println("Age: " + age.getYears() + " years, "
                           + age.getMonths() + " months, "
                           + age.getDays() + " days");
```

---

## 11. Putting It All Together

Here's a realistic example combining collections, lambdas, and streams — a simple product catalogue filter:

```java
import java.util.*;
import java.util.stream.*;

public class ProductCatalogue {

    static class Product {
        String name;
        String category;
        double price;

        Product(String name, String category, double price) {
            this.name = name;
            this.category = category;
            this.price = price;
        }

        @Override
        public String toString() {
            return String.format("%-15s %-10s $%.2f", name, category, price);
        }
    }

    public static void main(String[] args) {
        List<Product> catalogue = List.of(
            new Product("Laptop",     "Electronics", 999.99),
            new Product("Phone",      "Electronics", 699.99),
            new Product("Desk",       "Furniture",   249.99),
            new Product("Chair",      "Furniture",   149.99),
            new Product("Headphones", "Electronics", 149.99),
            new Product("Bookshelf",  "Furniture",   199.99)
        );

        System.out.println("=== Electronics under $700 ===");
        catalogue.stream()
            .filter(p -> p.category.equals("Electronics"))
            .filter(p -> p.price < 700)
            .forEach(System.out::println);

        System.out.println("\n=== All products sorted by price ===");
        catalogue.stream()
            .sorted((a, b) -> Double.compare(a.price, b.price))
            .forEach(System.out::println);

        double avgPrice = catalogue.stream()
            .mapToDouble(p -> p.price)
            .average()
            .orElse(0);
        System.out.printf("%n=== Average price: $%.2f ===%n", avgPrice);

        Map<String, List<Product>> byCategory = catalogue.stream()
            .collect(Collectors.groupingBy(p -> p.category));
        System.out.println("\n=== Products by category ===");
        byCategory.forEach((cat, products) -> {
            System.out.println(cat + ":");
            products.forEach(p -> System.out.println("  " + p));
        });
    }
}
```

Output:
```
=== Electronics under $700 ===
Phone           Electronics $699.99
Headphones      Electronics $149.99

=== All products sorted by price ===
Chair           Furniture   $149.99
Headphones      Electronics $149.99
Bookshelf       Furniture   $199.99
Desk            Furniture   $249.99
Phone           Electronics $699.99
Laptop          Electronics $999.99

=== Average price: $408.32 ===

=== Products by category ===
Electronics:
  Laptop          Electronics $999.99
  Phone           Electronics $699.99
  Headphones      Electronics $149.99
Furniture:
  Desk            Furniture   $249.99
  Chair           Furniture   $149.99
  Bookshelf       Furniture   $199.99
```

---

## 12. Exercises

---

### Exercise 1 – Set Basics

Write a program that:
1. Asks the user to enter 8 words (some can be duplicates).
2. Stores them in a `HashSet`.
3. Prints how many **unique** words were entered.
4. Prints the words in alphabetical order using a `TreeSet`.

---

### Exercise 2 – Phone Book

Build a simple phone book using a `HashMap<String, String>` (name → number):
1. Add at least 5 contacts.
2. Ask the user for a name and look up their number. Print a friendly message if not found (`getOrDefault`).
3. Allow the user to add a new contact.
4. Print all contacts in alphabetical order by name (use a `TreeMap`).

---

### Exercise 3 – Stream Filtering

Given this list of integers: `{4, 15, 8, 23, 16, 7, 42, 3, 19, 11, 30, 6}`, use streams to:
1. Print all numbers greater than 10.
2. Print the sum of all even numbers.
3. Print the three smallest numbers.
4. Print whether any number is greater than 40 (use `anyMatch`).

---

### Exercise 4 – Student Report with Streams

Create a `Student` class with `name` (String) and `gpa` (double).

Create a list of at least 6 students. Using streams:
1. Print all students with a GPA above 3.0, sorted by GPA descending.
2. Print the name of the student with the highest GPA.
3. Calculate and print the average GPA.
4. Count how many students are on the honor roll (GPA ≥ 3.5).

---

### Exercise 5 – Word Frequency

Write a program that:
1. Takes a sentence as input from the user.
2. Splits it into words.
3. Uses a `HashMap` to count how many times each word appears.
4. Uses a stream to print only words that appear more than once, sorted alphabetically.

---

### Exercise 6 – Age Calculator

Using the `java.time` API:
1. Ask the user for their birth year, month, and day.
2. Calculate their exact age using `Period.between()`.
3. Print their age in years, months, and days.
4. Print the day of the week they were born.
5. Print how many days until their next birthday.

---

### Exercise 7 – Full Collection App *(Challenge)*

Build a student grade management system:

**Data:** Use a `Map<String, List<Double>>` where each key is a student name and each value is a list of their assignment scores.

**Features (all using streams):**
1. Add students and their scores.
2. Print each student's average score.
3. Print the top-performing student (highest average).
4. Print students whose average is below 60 (at risk).
5. Print all students and averages sorted from highest to lowest.

---

## Module Summary

Here's what you covered in Module 8:

- The **Collection Framework** provides ready-made data structures: `List`, `Set`, `Map`.
- **`HashSet`** stores unique elements with fast lookup; **`TreeSet`** keeps them sorted.
- **`HashMap`** maps keys to values for instant lookups; **`TreeMap`** keeps keys sorted.
- The **`Collections`** utility class offers `sort`, `reverse`, `shuffle`, `max`, `min`.
- Choose your collection based on what you need: order, uniqueness, or key-value lookup.
- **Lambda expressions** are concise anonymous functions used with functional interfaces.
- The **Stream API** lets you process collections with readable pipelines: `filter`, `map`, `sorted`, `collect`, `count`, `average`.
- **`Optional`** handles the absence of a value safely, avoiding `NullPointerException`.
- The **`java.time`** API (`LocalDate`, `LocalDateTime`, `Period`) makes date handling clean and intuitive.

---

## 13. Course Wrap-Up

Congratulations — you've completed **CS 1102 Programming 1 (Java)**! 🎉

Here's a summary of everything you've learned across all 8 modules:

| Module | Topic | Key Skills |
|---|---|---|
| 1 | Introduction to Java | JVM, data types, Scanner, output |
| 2 | Control Structures | if/else, switch, for, while, do-while |
| 3 | Static Methods & Member Variables | Methods, parameters, return values, scope |
| 4 | Arrays & ArrayLists | Fixed arrays, dynamic lists, iteration |
| 5 | Objects & Classes | Constructors, encapsulation, getters/setters |
| 6 | OOP Paradigms | Inheritance, polymorphism, abstract classes, interfaces |
| 7 | GUI & Event Handling | Swing, layout managers, listeners, lambdas |
| 8 | Collection API & Java 8 | Set, Map, streams, Optional, java.time |

### What's Next — CS 1103 Programming 2

In the second course you'll go deeper into professional Java development:

- **Exception Handling** — dealing gracefully with errors at runtime.
- **Multithreading** — running multiple tasks at the same time.
- **File I/O** — reading from and writing to files.
- **Networking** — communicating between programs over a network.
- **Database Connectivity (JDBC)** — connecting Java to a database.
- **Generic Programming** — writing flexible, type-safe code.
- **Advanced GUI** — building more sophisticated interfaces.

The foundation you've built here — OOP, collections, streams, lambdas — is exactly what CS 1103 builds on. You're well prepared.

---

*CS 1102 Programming 1 (Java) — Module 8 of 8 — Course Complete*
