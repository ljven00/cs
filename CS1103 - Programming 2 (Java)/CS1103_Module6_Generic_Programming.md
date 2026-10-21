# CS 1103 – Programming 2 (Java)
## Module 6: Generic Programming

---

## Table of Contents

1. [What Is Generic Programming?](#1-what-is-generic-programming)
2. [The Problem Generics Solve](#2-the-problem-generics-solve)
3. [Generic Classes](#3-generic-classes)
4. [Generic Methods](#4-generic-methods)
5. [Multiple Type Parameters](#5-multiple-type-parameters)
6. [Bounded Type Parameters](#6-bounded-type-parameters)
7. [Wildcards](#7-wildcards)
8. [Generic Interfaces](#8-generic-interfaces)
9. [Type Erasure — How Generics Work Under the Hood](#9-type-erasure--how-generics-work-under-the-hood)
10. [Generics and Collections](#10-generics-and-collections)
11. [Comparable and Comparator](#11-comparable-and-comparator)
12. [Putting It All Together](#12-putting-it-all-together)
13. [Exercises](#13-exercises)

---

## 1. What Is Generic Programming?

**Generic programming** is writing code that works correctly for many different types — without sacrificing type safety or duplicating logic.

You have already been using generics since Module 4 of CS 1102:

```java
ArrayList<String>  names  = new ArrayList<>();
ArrayList<Integer> scores = new ArrayList<>();
HashMap<String, Double> prices = new HashMap<>();
```

The `<String>`, `<Integer>`, and `<String, Double>` parts are **type parameters** — they tell Java what kind of data the collection holds. The collection's internal logic is the same regardless of the type, but Java ensures you only put `String`s in a `String` list.

In this module you'll learn how to **write** your own generic classes, methods, and interfaces — not just use existing ones.

---

## 2. The Problem Generics Solve

Imagine you want a simple container that holds a single value. Without generics, you'd have two bad options:

### Option 1 — Write a separate class for each type

```java
class IntBox    { private int value;    ... }
class StringBox { private String value; ... }
class DoubleBox { private double value; ... }
```

This works but creates massive duplication. Every time you need a new type, you write another class.

### Option 2 — Use `Object` for everything

```java
class Box {
    private Object value;

    public void set(Object value) { this.value = value; }
    public Object get()           { return value; }
}
```

```java
Box box = new Box();
box.set("Hello");
String s = (String) box.get();   // must cast — and it can go wrong

box.set(42);
String oops = (String) box.get();   // ClassCastException at runtime!
```

This compiles fine but explodes at runtime. The compiler can't protect you because it has no idea what type is stored.

### Option 3 — Generics (the right way)

```java
class Box<T> {
    private T value;

    public void set(T value) { this.value = value; }
    public T get()           { return value; }
}
```

```java
Box<String> stringBox = new Box<>();
stringBox.set("Hello");
String s = stringBox.get();   // no cast needed — type is known

Box<Integer> intBox = new Box<>();
intBox.set(42);
// intBox.set("oops");   ← compile error! Java catches this at compile time
```

Generics give you:
- **One class** for all types (no duplication).
- **Compile-time type checking** (no surprise `ClassCastException`).
- **No casting** when retrieving values.

---

## 3. Generic Classes

### Declaring a Generic Class

Place the **type parameter** in angle brackets after the class name. By convention, single uppercase letters are used:

| Letter | Common meaning |
|---|---|
| `T` | Type (general purpose) |
| `E` | Element (used in collections) |
| `K` | Key |
| `V` | Value |
| `N` | Number |

```java
public class Box<T> {

    private T content;

    public Box(T content) {
        this.content = content;
    }

    public T getContent() {
        return content;
    }

    public void setContent(T content) {
        this.content = content;
    }

    public boolean isEmpty() {
        return content == null;
    }

    @Override
    public String toString() {
        return "Box[" + content + "]";
    }
}
```

### Using the Generic Class

```java
Box<String>  letterBox = new Box<>("Hello, Generics!");
Box<Integer> numberBox = new Box<>(42);
Box<Double>  priceBox  = new Box<>(9.99);

System.out.println(letterBox.getContent());        // Hello, Generics!
System.out.println(numberBox.getContent() * 2);    // 84
System.out.println(priceBox);                      // Box[9.99]
```

### A Generic Stack

Here's a more useful example — a stack data structure that works with any type:

```java
import java.util.ArrayList;

public class Stack<T> {

    private ArrayList<T> elements = new ArrayList<>();

    public void push(T item) {
        elements.add(item);
    }

    public T pop() {
        if (isEmpty()) {
            throw new RuntimeException("Stack is empty.");
        }
        return elements.remove(elements.size() - 1);
    }

    public T peek() {
        if (isEmpty()) {
            throw new RuntimeException("Stack is empty.");
        }
        return elements.get(elements.size() - 1);
    }

    public boolean isEmpty() {
        return elements.isEmpty();
    }

    public int size() {
        return elements.size();
    }

    @Override
    public String toString() {
        return elements.toString();
    }
}
```

```java
Stack<String> wordStack = new Stack<>();
wordStack.push("Java");
wordStack.push("is");
wordStack.push("fun");

System.out.println(wordStack.peek());   // fun (top of stack)
System.out.println(wordStack.pop());    // fun
System.out.println(wordStack.pop());    // is
System.out.println(wordStack);          // [Java]

Stack<Integer> numStack = new Stack<>();
numStack.push(10);
numStack.push(20);
numStack.push(30);
System.out.println(numStack);   // [10, 20, 30]
```

One class, works for `String`, `Integer`, any type.

---

## 4. Generic Methods

A method can have its own type parameter, independent of the class it belongs to. This is useful for static utility methods that don't belong to a specific class.

### Syntax

The type parameter `<T>` goes **before the return type**:

```java
public static <T> void printArray(T[] array) {
    System.out.print("[");
    for (int i = 0; i < array.length; i++) {
        System.out.print(array[i]);
        if (i < array.length - 1) System.out.print(", ");
    }
    System.out.println("]");
}
```

```java
Integer[] ints    = {1, 2, 3, 4, 5};
String[]  words   = {"apple", "banana", "cherry"};
Double[]  doubles = {1.1, 2.2, 3.3};

printArray(ints);     // [1, 2, 3, 4, 5]
printArray(words);    // [apple, banana, cherry]
printArray(doubles);  // [1.1, 2.2, 3.3]
```

Java **infers** the type from the argument — you don't need to specify `<Integer>` or `<String>` explicitly.

### Returning a Generic Value

```java
public static <T> T getFirst(T[] array) {
    if (array == null || array.length == 0) {
        throw new IllegalArgumentException("Array is empty.");
    }
    return array[0];
}
```

```java
String first = getFirst(new String[]{"Alice", "Bob", "Carlos"});
Integer num  = getFirst(new Integer[]{10, 20, 30});

System.out.println(first);   // Alice
System.out.println(num);     // 10
```

### Swapping Array Elements

```java
public static <T> void swap(T[] array, int i, int j) {
    T temp    = array[i];
    array[i]  = array[j];
    array[j]  = temp;
}
```

```java
String[] names = {"Alice", "Bob", "Carlos"};
swap(names, 0, 2);
System.out.println(Arrays.toString(names));   // [Carlos, Bob, Alice]
```

---

## 5. Multiple Type Parameters

A class or method can have more than one type parameter. The `HashMap<K, V>` you already know uses exactly this:

```java
public class Pair<A, B> {

    private A first;
    private B second;

    public Pair(A first, B second) {
        this.first  = first;
        this.second = second;
    }

    public A getFirst()  { return first; }
    public B getSecond() { return second; }

    @Override
    public String toString() {
        return "(" + first + ", " + second + ")";
    }
}
```

```java
Pair<String, Integer> nameAge   = new Pair<>("Alice", 30);
Pair<String, Double>  namePrice = new Pair<>("Laptop", 999.99);
Pair<Integer, Boolean> idActive = new Pair<>(1001, true);

System.out.println(nameAge);     // (Alice, 30)
System.out.println(namePrice);   // (Laptop, 999.99)
System.out.println(idActive);    // (1001, true)

String name = nameAge.getFirst();    // Alice
int    age  = nameAge.getSecond();   // 30
```

### Generic Method with Two Parameters

```java
public static <K, V> void printPair(K key, V value) {
    System.out.println(key + " → " + value);
}

printPair("Name", "Alice");
printPair("Score", 95);
printPair("Price", 9.99);
```

Output:
```
Name → Alice
Score → 95
Price → 9.99
```

---

## 6. Bounded Type Parameters

Sometimes you want a generic type that is restricted to a certain family of types. For example, a method that finds the maximum of two values only makes sense for types that can be compared.

### Upper Bound — `extends`

`<T extends SomeType>` means T must be `SomeType` or a subclass of it:

```java
public static <T extends Comparable<T>> T max(T a, T b) {
    return a.compareTo(b) >= 0 ? a : b;
}
```

```java
System.out.println(max(10, 25));           // 25
System.out.println(max("apple", "mango")); // mango (alphabetical)
System.out.println(max(3.14, 2.71));       // 3.14
```

`Integer`, `String`, and `Double` all implement `Comparable`, so they're valid. Passing a type that doesn't implement `Comparable` would cause a **compile error** — the bound enforces the constraint.

### Bounding to a Custom Type

```java
public class NumberBox<T extends Number> {
    private T value;

    NumberBox(T value) { this.value = value; }

    public double doubleValue() {
        return value.doubleValue();   // Number has doubleValue()
    }
}
```

```java
NumberBox<Integer> intBox    = new NumberBox<>(42);
NumberBox<Double>  doubleBox = new NumberBox<>(3.14);
// NumberBox<String> strBox  = new NumberBox<>("hi");  ← compile error

System.out.println(intBox.doubleValue());     // 42.0
System.out.println(doubleBox.doubleValue());  // 3.14
```

### Multiple Bounds

A type parameter can have multiple bounds with `&`:

```java
public static <T extends Comparable<T> & Cloneable> T clampedMax(T a, T b) {
    return a.compareTo(b) >= 0 ? a : b;
}
```

When combining a class and interfaces, the class must come first.

---

## 7. Wildcards

A **wildcard** `?` represents an unknown type. Wildcards are used when working with generic types as method parameters — when you want to accept a family of parameterized types.

### Unbounded Wildcard — `<?>`

Accepts a generic collection of any type:

```java
public static void printList(List<?> list) {
    for (Object item : list) {
        System.out.print(item + " ");
    }
    System.out.println();
}
```

```java
printList(List.of(1, 2, 3));              // 1 2 3
printList(List.of("a", "b", "c"));        // a b c
printList(List.of(1.1, 2.2, 3.3));        // 1.1 2.2 3.3
```

### Upper Bounded Wildcard — `<? extends Type>`

Accepts a collection of `Type` or any subclass. Use when you only need to **read** from the collection:

```java
public static double sumList(List<? extends Number> list) {
    double total = 0;
    for (Number n : list) {
        total += n.doubleValue();
    }
    return total;
}
```

```java
System.out.println(sumList(List.of(1, 2, 3)));           // 6.0
System.out.println(sumList(List.of(1.5, 2.5, 3.0)));     // 7.0
System.out.println(sumList(List.of(10L, 20L, 30L)));      // 60.0
```

### Lower Bounded Wildcard — `<? super Type>`

Accepts a collection of `Type` or any superclass. Use when you only need to **write** to the collection:

```java
public static void addNumbers(List<? super Integer> list) {
    list.add(1);
    list.add(2);
    list.add(3);
}
```

```java
List<Number> numbers = new ArrayList<>();
addNumbers(numbers);
System.out.println(numbers);   // [1, 2, 3]
```

### The PECS Rule

A helpful mnemonic for when to use each wildcard:

> **P**roducer **E**xtends, **C**onsumer **S**uper

- If the collection **produces** (you read from it) → use `<? extends T>`
- If the collection **consumes** (you write to it) → use `<? super T>`
- If both → use explicit type parameter `<T>`

---

## 8. Generic Interfaces

Interfaces can also be generic. You've already used `Comparable<T>` and `Iterable<T>` — both are generic interfaces.

### Defining a Generic Interface

```java
public interface Repository<T, ID> {
    void save(T item);
    T findById(ID id);
    List<T> findAll();
    boolean delete(ID id);
    int count();
}
```

### Implementing with a Concrete Type

```java
import java.util.*;

public class StudentRepository implements Repository<Student, Integer> {

    private Map<Integer, Student> store = new HashMap<>();
    private int nextId = 1;

    @Override
    public void save(Student student) {
        store.put(nextId++, student);
    }

    @Override
    public Student findById(Integer id) {
        return store.get(id);
    }

    @Override
    public List<Student> findAll() {
        return new ArrayList<>(store.values());
    }

    @Override
    public boolean delete(Integer id) {
        return store.remove(id) != null;
    }

    @Override
    public int count() {
        return store.size();
    }
}
```

### Implementing with Another Generic Type

A class can stay generic when implementing a generic interface:

```java
public class InMemoryRepository<T> implements Repository<T, Integer> {

    private Map<Integer, T> store = new HashMap<>();
    private int nextId = 1;

    @Override
    public void save(T item) {
        store.put(nextId++, item);
    }

    @Override
    public T findById(Integer id) {
        return store.get(id);
    }

    @Override
    public List<T> findAll() {
        return new ArrayList<>(store.values());
    }

    @Override
    public boolean delete(Integer id) {
        return store.remove(id) != null;
    }

    @Override
    public int count() {
        return store.size();
    }
}
```

```java
InMemoryRepository<String> repo = new InMemoryRepository<>();
repo.save("Alice");
repo.save("Bob");
repo.save("Carlos");

System.out.println(repo.findById(2));   // Bob
System.out.println(repo.count());       // 3
repo.delete(1);
System.out.println(repo.findAll());     // [Bob, Carlos]
```

---

## 9. Type Erasure — How Generics Work Under the Hood

Java generics are a **compile-time feature**. At runtime, type parameters are erased — all generic types become their bounds (or `Object` if unbounded). This is called **type erasure**.

```java
// What you write
Box<String> box = new Box<>("Hello");

// What the compiler actually produces (simplified)
Box box = new Box("Hello");
// plus automatic cast: String s = (String) box.getContent();
```

### Practical Consequences

**You cannot use primitives as type parameters** (use wrapper classes):

```java
Box<int>    intBox = new Box<>(5);    // compile error
Box<Integer> intBox = new Box<>(5);   // correct — Integer is the wrapper
```

**You cannot check the type parameter at runtime:**

```java
if (box instanceof Box<String>) { }   // compile error — type erased
if (box instanceof Box<?>) { }        // this works — wildcard
```

**You cannot create an array of a generic type:**

```java
T[] array = new T[10];   // compile error
// Workaround:
@SuppressWarnings("unchecked")
T[] array = (T[]) new Object[10];
```

**Generic type information is not available via reflection** (in most cases).

These are not bugs — they're deliberate design decisions that preserve backward compatibility with pre-generics Java code.

---

## 10. Generics and Collections

The entire Collection Framework is generic. Understanding this lets you write much more flexible code.

### Generic Utility Method for Collections

```java
import java.util.*;

public class CollectionUtils {

    // Find the maximum element in any List of Comparable items
    public static <T extends Comparable<T>> T max(List<T> list) {
        if (list == null || list.isEmpty())
            throw new IllegalArgumentException("List is empty.");
        T max = list.get(0);
        for (T item : list) {
            if (item.compareTo(max) > 0) max = item;
        }
        return max;
    }

    // Reverse any List in place
    public static <T> void reverse(List<T> list) {
        int left = 0, right = list.size() - 1;
        while (left < right) {
            T temp = list.get(left);
            list.set(left, list.get(right));
            list.set(right, temp);
            left++;
            right--;
        }
    }

    // Count elements matching a predicate
    public static <T> int countIf(List<T> list, java.util.function.Predicate<T> predicate) {
        int count = 0;
        for (T item : list) {
            if (predicate.test(item)) count++;
        }
        return count;
    }

    // Convert a List to a Map using a key extractor
    public static <T, K> Map<K, T> toMap(List<T> list,
                                          java.util.function.Function<T, K> keyExtractor) {
        Map<K, T> map = new HashMap<>();
        for (T item : list) {
            map.put(keyExtractor.apply(item), item);
        }
        return map;
    }
}
```

```java
List<Integer> nums   = new ArrayList<>(List.of(3, 1, 4, 1, 5, 9, 2, 6));
List<String>  words  = new ArrayList<>(List.of("banana", "apple", "cherry"));

System.out.println(CollectionUtils.max(nums));    // 9
System.out.println(CollectionUtils.max(words));   // cherry

CollectionUtils.reverse(nums);
System.out.println(nums);   // [6, 2, 9, 5, 1, 4, 1, 3]

int evenCount = CollectionUtils.countIf(nums, n -> n % 2 == 0);
System.out.println("Even numbers: " + evenCount);   // 3
```

---

## 11. `Comparable` and `Comparator`

These two interfaces are fundamental to sorting in Java. Both are generic.

### `Comparable<T>` — Natural Ordering

A class implements `Comparable<T>` to define its **natural ordering** — how objects of that type compare to each other by default.

```java
public class Student implements Comparable<Student> {

    private String name;
    private double gpa;

    public Student(String name, double gpa) {
        this.name = name;
        this.gpa  = gpa;
    }

    public String getName() { return name; }
    public double getGpa()  { return gpa; }

    @Override
    public int compareTo(Student other) {
        // Sort by GPA descending (higher GPA first)
        return Double.compare(other.gpa, this.gpa);
    }

    @Override
    public String toString() {
        return String.format("%-10s %.2f", name, gpa);
    }
}
```

```java
List<Student> students = new ArrayList<>(List.of(
    new Student("Alice",  3.85),
    new Student("Bob",    3.42),
    new Student("Carlos", 3.91),
    new Student("Diana",  3.67)
));

Collections.sort(students);   // uses compareTo
students.forEach(System.out::println);
```

Output:
```
Carlos     3.91
Alice      3.85
Diana      3.67
Bob        3.42
```

The `compareTo` contract:
- Returns **negative** if `this` comes before `other`
- Returns **zero** if they are equal
- Returns **positive** if `this` comes after `other`

### `Comparator<T>` — Custom Ordering

`Comparator<T>` defines an ordering **outside** the class. Use it when:
- You don't control the class (e.g. `String`).
- You need multiple different orderings.

```java
// Sort by name alphabetically
Comparator<Student> byName = Comparator.comparing(Student::getName);

// Sort by GPA ascending
Comparator<Student> byGpaAsc = Comparator.comparingDouble(Student::getGpa);

// Sort by GPA descending
Comparator<Student> byGpaDesc = Comparator.comparingDouble(Student::getGpa).reversed();

// Sort by name, then by GPA if names are equal
Comparator<Student> byNameThenGpa = Comparator.comparing(Student::getName)
    .thenComparingDouble(Student::getGpa);
```

```java
students.sort(byName);
System.out.println("By name:");
students.forEach(System.out::println);

students.sort(byGpaDesc);
System.out.println("\nBy GPA (desc):");
students.forEach(System.out::println);
```

Output:
```
By name:
Alice      3.85
Bob        3.42
Carlos     3.91
Diana      3.67

By GPA (desc):
Carlos     3.91
Alice      3.85
Diana      3.67
Bob        3.42
```

### In a Stream

```java
students.stream()
    .sorted(Comparator.comparing(Student::getName))
    .forEach(System.out::println);
```

---

## 12. Putting It All Together

Here's a complete generic data pipeline — a reusable `DataProcessor<T>` that can filter, transform, and summarize any list:

```java
import java.util.*;
import java.util.function.*;
import java.util.stream.*;

public class DataProcessor<T> {

    private final List<T> data;

    public DataProcessor(List<T> data) {
        this.data = new ArrayList<>(data);
    }

    // Filter — keep items matching a predicate
    public DataProcessor<T> filter(Predicate<T> predicate) {
        List<T> filtered = data.stream()
            .filter(predicate)
            .collect(Collectors.toList());
        return new DataProcessor<>(filtered);
    }

    // Sort — sort by a comparator
    public DataProcessor<T> sort(Comparator<T> comparator) {
        List<T> sorted = data.stream()
            .sorted(comparator)
            .collect(Collectors.toList());
        return new DataProcessor<>(sorted);
    }

    // Map — transform each element into a different type
    public <R> DataProcessor<R> map(Function<T, R> mapper) {
        List<R> mapped = data.stream()
            .map(mapper)
            .collect(Collectors.toList());
        return new DataProcessor<>(mapped);
    }

    // Print all elements
    public DataProcessor<T> print(String label) {
        System.out.println("--- " + label + " ---");
        data.forEach(System.out::println);
        System.out.println("Count: " + data.size());
        return this;
    }

    // Get results
    public List<T> toList() {
        return Collections.unmodifiableList(data);
    }

    // Statistics for numeric data
    public static <N extends Number> void stats(List<N> numbers) {
        DoubleSummaryStatistics stats = numbers.stream()
            .mapToDouble(Number::doubleValue)
            .summaryStatistics();
        System.out.printf("Count: %d  |  Min: %.2f  |  Max: %.2f  |  Avg: %.2f%n",
            stats.getCount(), stats.getMin(), stats.getMax(), stats.getAverage());
    }
}
```

```java
public class Main {
    public static void main(String[] args) {

        List<Student> students = List.of(
            new Student("Alice",  3.85),
            new Student("Bob",    2.95),
            new Student("Carlos", 3.91),
            new Student("Diana",  3.67),
            new Student("Ethan",  2.60),
            new Student("Fiona",  3.50)
        );

        // Chain operations — like a mini query engine
        new DataProcessor<>(students)
            .filter(s -> s.getGpa() >= 3.5)
            .sort(Comparator.comparingDouble(Student::getGpa).reversed())
            .print("Honor Roll (GPA ≥ 3.5)");

        // Extract GPAs and compute statistics
        List<Double> gpas = new DataProcessor<>(students)
            .map(Student::getGpa)
            .toList();

        System.out.println("\n--- GPA Statistics ---");
        DataProcessor.stats(new ArrayList<>(gpas));

        // Extract and sort names
        new DataProcessor<>(students)
            .map(Student::getName)
            .sort(Comparator.naturalOrder())
            .print("\nAll Names (Alphabetical)");
    }
}
```

Output:
```
--- Honor Roll (GPA ≥ 3.5) ---
Carlos     3.91
Alice      3.85
Diana      3.67
Fiona      3.50
Count: 4

--- GPA Statistics ---
Count: 6  |  Min: 2.60  |  Max: 3.91  |  Avg: 3.41

--- All Names (Alphabetical) ---
Alice
Bob
Carlos
Diana
Ethan
Fiona
Count: 6
```

---

## 13. Exercises

---

### Exercise 1 – Generic Box

Create a generic class `Box<T>` with:
- A private field `content` of type `T`.
- A constructor, `get()`, `set()`, `isEmpty()`, and `toString()`.

Also add a **static generic method** `swap(Box<T> a, Box<T> b)` that swaps the contents of two boxes.

In `main`, create `Box<String>`, `Box<Integer>`, and `Box<Double>` instances. Test `swap()` on two `Box<String>` objects.

---

### Exercise 2 – Generic Stack

Implement a `Stack<T>` class backed by an `ArrayList` with:
- `push(T item)`, `pop()` → T, `peek()` → T, `isEmpty()`, `size()`, `toString()`.
- `pop()` and `peek()` should throw `EmptyStackException` if the stack is empty.

In `main`, demonstrate a `Stack<String>` used to **reverse a sentence** word by word, and a `Stack<Integer>` used to **check if a sequence of brackets is balanced** (push on `(`, pop on `)`).

---

### Exercise 3 – Bounded Generic Method

Write the following static generic methods in a `MathUtils` class:

- `<T extends Comparable<T>> T min(T a, T b, T c)` — returns the smallest of three values.
- `<T extends Number> double average(List<T> list)` — returns the average of a list of numbers.
- `<T extends Comparable<T>> List<T> clamp(List<T> list, T low, T high)` — returns a new list with all values clamped between `low` and `high`.

Test with `Integer`, `Double`, and `String` where applicable.

---

### Exercise 4 – Generic Pair and Triple

Create:
- A `Pair<A, B>` class with two fields, getters, and `toString()`.
- A `Triple<A, B, C>` class with three fields, getters, and `toString()`.
- A static method `<T> Pair<T, T> fromArray(T[] array, int i, int j)` that creates a `Pair` from two array elements.

In `main`, use `Pair` to represent (name, score) and (city, country), and `Triple` for (name, age, gpa).

---

### Exercise 5 – Comparable Student Ranking

Create a `Student` class that implements `Comparable<Student>` ordering by GPA descending.

Create a list of 6 students. Then:
1. Sort using `Collections.sort()` (uses `compareTo`).
2. Sort by name using a `Comparator`.
3. Sort by name length using a `Comparator`.
4. Find the student with the highest GPA using `Collections.max()`.

Print results after each sort.

---

### Exercise 6 – Generic Repository *(Challenge)*

Define a generic interface:

```java
public interface Storable<T, ID> {
    void add(T item);
    Optional<T> get(ID id);
    List<T> getAll();
    boolean remove(ID id);
    int size();
}
```

Implement `InMemoryStore<T>` that implements `Storable<T, Integer>` using a `HashMap<Integer, T>`. Auto-generate integer IDs starting from 1.

Add a generic method `<T extends Comparable<T>> List<T> getSorted(InMemoryStore<T> store)` that returns all items in sorted order.

Test by creating an `InMemoryStore<String>` and an `InMemoryStore<Student>`.

---

### Exercise 7 – DataProcessor Pipeline *(Challenge)*

Extend the `DataProcessor<T>` class from the module with:
- `limit(int n)` — keeps only the first `n` elements.
- `distinct()` — removes duplicate elements.
- `count()` — returns the number of elements.
- `findFirst(Predicate<T> predicate)` — returns an `Optional<T>` for the first match.
- `groupBy(Function<T, K> classifier)` — returns a `Map<K, List<T>>`.

Test the full pipeline with a list of products (name, category, price):
- Filter to products under $500.
- Group by category.
- For each category, find the cheapest item.

---

## Module Summary

Here's what you covered in Module 6:

- **Generics** let you write one class or method that works safely with any type.
- A **generic class** declares a type parameter in `<T>` after the class name.
- A **generic method** declares `<T>` before the return type and infers the type from arguments.
- **Multiple type parameters** (`<A, B>`) allow relationships between different types.
- **Bounded type parameters** (`<T extends Number>`) restrict which types are valid.
- **Wildcards** (`<?>`, `<? extends T>`, `<? super T>`) allow flexibility in method parameters — use PECS to remember when.
- **Generic interfaces** define contracts that can be implemented for any type.
- **Type erasure** means generics are a compile-time feature — type information is removed at runtime.
- `Comparable<T>` defines natural ordering within a class; `Comparator<T>` defines external, custom ordering.
- `Comparator.comparing()`, `thenComparing()`, and `reversed()` make complex sort orders readable.

---

**Next up:** Module 7 – Files and Networking

---

*CS 1103 Programming 2 (Java) — Module 6 of 8*
