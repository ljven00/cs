# CS 3303 — Data Structures
## Module 1: Fundamentals of Data Structures and Algorithm Analysis

---

## 1.1 What is a Data Structure?

A **data structure** is a way of organizing, storing, and managing data in a computer so that it can be accessed and modified efficiently. Choosing the right data structure is one of the most critical decisions in software engineering — it directly affects the performance and scalability of your program.

Think of data structures as containers with rules:
- An **array** is like a row of numbered lockers — fast to access by number, but fixed in size.
- A **linked list** is like a treasure hunt — each clue points to the next location.
- A **stack** is like a pile of plates — you can only add or remove from the top.

---

## 1.2 What is an Algorithm?

An **algorithm** is a finite, well-defined sequence of steps to solve a problem or accomplish a task. Every program you write is ultimately a collection of algorithms.

**Key properties of a good algorithm:**
1. **Correctness** — It produces the right output for all valid inputs.
2. **Efficiency** — It uses minimal time and memory.
3. **Clarity** — It is easy to understand and maintain.
4. **Finiteness** — It always terminates.

---

## 1.3 Why Algorithm Analysis?

Writing code that *works* is not enough. Consider two algorithms that both sort a list of 1,000,000 numbers:
- Algorithm A takes **2 seconds**
- Algorithm B takes **4 days**

Both are correct. Only one is practical. **Algorithm analysis** gives us tools to measure and compare performance *before* running the code — using mathematical models instead of stopwatches.

---

## 1.4 Big-O Notation

**Big-O notation** describes the *worst-case* growth rate of an algorithm's time or space requirements as the input size `n` grows. It strips away constants and lower-order terms to expose the dominant behavior.

### Common Big-O Classes (from fastest to slowest)

| Notation       | Name             | Example                         |
|----------------|------------------|---------------------------------|
| O(1)           | Constant         | Array index access              |
| O(log n)       | Logarithmic      | Binary search                   |
| O(n)           | Linear           | Linear search                   |
| O(n log n)     | Linearithmic     | Merge sort, heap sort           |
| O(n²)          | Quadratic        | Bubble sort, selection sort     |
| O(2ⁿ)          | Exponential      | Recursive Fibonacci (naïve)     |
| O(n!)          | Factorial        | Brute-force traveling salesman  |

### Big-O Growth Visualization (n = 16)

```
n = 16

O(1)      →    1  operation
O(log n)  →    4  operations
O(n)      →   16  operations
O(n log n)→   64  operations
O(n²)     →  256  operations
O(2ⁿ)     →  65536 operations  ← danger zone
```

### Formal Definition

> f(n) = O(g(n)) if there exist constants c > 0 and n₀ ≥ 1 such that:
> f(n) ≤ c · g(n)   for all n ≥ n₀

In plain English: after a certain input size, `g(n)` (scaled by some constant) is always an upper bound on `f(n)`.

---

## 1.5 Other Asymptotic Notations

| Notation   | Meaning                        | Use case                    |
|------------|--------------------------------|-----------------------------|
| O(f(n))    | Upper bound (worst case)       | Most common in practice     |
| Ω(f(n))    | Lower bound (best case)        | Proves minimum effort needed|
| Θ(f(n))    | Tight bound (exact growth)     | When best = worst case      |

---

## 1.6 Analyzing Time Complexity — Step by Step

### Rule 1: Ignore Constants
```java
// O(3n) → simplified to O(n)
for (int i = 0; i < 3 * n; i++) {
    System.out.println(i);
}
```

### Rule 2: Drop Non-Dominant Terms
```java
// O(n² + n) → simplified to O(n²)
// The n² term dominates as n grows large
```

### Rule 3: Sequential Steps → Add
```java
// O(n) + O(n) = O(2n) → O(n)
for (int i = 0; i < n; i++) { /* step 1 */ }
for (int j = 0; j < n; j++) { /* step 2 */ }
```

### Rule 4: Nested Steps → Multiply
```java
// O(n) × O(n) = O(n²)
for (int i = 0; i < n; i++) {
    for (int j = 0; j < n; j++) {
        /* nested */ 
    }
}
```

---

## 1.7 Java Code Examples

### Example 1: O(1) — Constant Time
```java
/**
 * Returns the first element of an array.
 * No matter how large the array, this always takes one step.
 * Time Complexity: O(1)
 */
public static int getFirst(int[] arr) {
    return arr[0]; // single operation, regardless of array size
}
```

---

### Example 2: O(n) — Linear Time
```java
/**
 * Finds the maximum value in an array by examining every element.
 * As the array doubles in size, the work also doubles.
 * Time Complexity: O(n)
 */
public static int findMax(int[] arr) {
    int max = arr[0];
    for (int i = 1; i < arr.length; i++) {  // n iterations
        if (arr[i] > max) {
            max = arr[i];
        }
    }
    return max;
}
```

---

### Example 3: O(n²) — Quadratic Time
```java
/**
 * Checks if any two elements in the array are duplicates.
 * For each element, we scan the entire remaining array.
 * Time Complexity: O(n²)
 */
public static boolean hasDuplicate(int[] arr) {
    for (int i = 0; i < arr.length; i++) {          // n iterations
        for (int j = i + 1; j < arr.length; j++) {  // up to n iterations
            if (arr[i] == arr[j]) {
                return true;
            }
        }
    }
    return false;
}
```

---

### Example 4: O(log n) — Logarithmic Time
```java
/**
 * Binary Search: finds a target in a SORTED array.
 * Each iteration cuts the search space in half.
 * For n=1,000,000 → only ~20 iterations needed!
 * Time Complexity: O(log n)
 */
public static int binarySearch(int[] arr, int target) {
    int left = 0;
    int right = arr.length - 1;

    while (left <= right) {
        int mid = left + (right - left) / 2;  // avoids integer overflow

        if (arr[mid] == target) {
            return mid;           // found it
        } else if (arr[mid] < target) {
            left = mid + 1;       // target is in the right half
        } else {
            right = mid - 1;      // target is in the left half
        }
    }
    return -1; // not found
}
```

---

### Example 5: O(2ⁿ) — Exponential Time
```java
/**
 * Naïve recursive Fibonacci.
 * Each call spawns two more calls — the tree of calls grows exponentially.
 * fib(40) requires over 300 million calls!
 * Time Complexity: O(2ⁿ)  ← AVOID for large n
 */
public static int fibNaive(int n) {
    if (n <= 1) return n;
    return fibNaive(n - 1) + fibNaive(n - 2);  // two recursive calls
}

/**
 * Optimized Fibonacci using memoization.
 * Each subproblem is solved only once.
 * Time Complexity: O(n)  ← MUCH better
 */
public static int fibMemo(int n, int[] memo) {
    if (n <= 1) return n;
    if (memo[n] != 0) return memo[n];         // already computed
    memo[n] = fibMemo(n - 1, memo) + fibMemo(n - 2, memo);
    return memo[n];
}
```

---

## 1.8 Space Complexity

Time is not the only resource that matters. **Space complexity** measures how much extra memory an algorithm uses relative to input size.

```java
/**
 * O(1) space — uses a fixed number of variables, regardless of input size.
 */
public static int sumArray(int[] arr) {
    int sum = 0;          // one variable
    for (int x : arr) {
        sum += x;
    }
    return sum;
}

/**
 * O(n) space — creates a new array proportional to input size.
 */
public static int[] doubleArray(int[] arr) {
    int[] result = new int[arr.length];  // n extra slots
    for (int i = 0; i < arr.length; i++) {
        result[i] = arr[i] * 2;
    }
    return result;
}
```

---

## 1.9 Complete Worked Example — Complexity Analysis

```java
/**
 * Given an array of integers, print all pairs whose sum equals a target.
 */
public class PairSum {

    // Approach 1: Brute Force — O(n²) time, O(1) space
    public static void findPairsBrute(int[] arr, int target) {
        for (int i = 0; i < arr.length; i++) {
            for (int j = i + 1; j < arr.length; j++) {
                if (arr[i] + arr[j] == target) {
                    System.out.println("(" + arr[i] + ", " + arr[j] + ")");
                }
            }
        }
    }

    // Approach 2: HashSet — O(n) time, O(n) space
    public static void findPairsOptimal(int[] arr, int target) {
        java.util.Set<Integer> seen = new java.util.HashSet<>();
        for (int num : arr) {
            int complement = target - num;
            if (seen.contains(complement)) {
                System.out.println("(" + complement + ", " + num + ")");
            }
            seen.add(num);
        }
    }

    public static void main(String[] args) {
        int[] arr = {2, 7, 4, 1, 3, 6};
        int target = 9;

        System.out.println("Brute Force O(n²):");
        findPairsBrute(arr, target);

        System.out.println("\nOptimal O(n):");
        findPairsOptimal(arr, target);
    }
}
```

**Output:**
```
Brute Force O(n²):
(2, 7)
(3, 6)

Optimal O(n):
(2, 7)
(3, 6)
```

**Trade-off analysis:**
- Brute force: no extra memory, but slow for large inputs.
- Optimal: uses a HashSet (extra memory), but runs in linear time.
- This is the classic **time-space trade-off** in algorithm design.

---

## 1.10 Abstract Data Types (ADT)

An **Abstract Data Type (ADT)** defines *what* operations a data structure supports — not *how* they are implemented. This separation of interface from implementation is a core principle of computer science.

| ADT        | Core Operations                      | Possible Implementations   |
|------------|--------------------------------------|----------------------------|
| List       | add, remove, get, size               | Array, Linked List          |
| Stack      | push, pop, peek, isEmpty             | Array, Linked List          |
| Queue      | enqueue, dequeue, peek, isEmpty      | Array, Linked List          |
| Map        | put, get, remove, containsKey        | Hash Table, BST             |
| Set        | add, remove, contains                | Hash Table, BST             |

In Java, ADTs are typically expressed as **interfaces**:

```java
/**
 * ADT: Stack
 * Defines the contract — any class implementing this must provide these operations.
 */
public interface Stack<T> {
    void push(T item);    // Add item to top
    T pop();              // Remove and return top item
    T peek();             // Return top item without removing
    boolean isEmpty();    // Check if stack is empty
    int size();           // Return number of items
}
```

The *implementation* details (array-based vs. linked-list-based) are hidden from the user. This is **encapsulation** — a cornerstone of good software design.

---

## 1.11 Key Takeaways

| Concept              | Summary                                                                 |
|----------------------|-------------------------------------------------------------------------|
| Data Structure       | A way to organize data for efficient access and modification            |
| Algorithm            | A finite sequence of steps to solve a problem                           |
| Big-O Notation       | Describes worst-case growth rate as input size increases                |
| O(1)                 | Constant — does not depend on input size                                |
| O(log n)             | Logarithmic — extremely efficient (binary search)                       |
| O(n)                 | Linear — grows proportionally with input                                |
| O(n²)                | Quadratic — nested loops; avoid for large inputs                        |
| Space Complexity     | Measures extra memory used by an algorithm                              |
| ADT                  | Defines *what* a structure does, not *how*                              |
| Time-Space Trade-off | Faster algorithms often use more memory, and vice versa                 |

---

## 1.12 Practice Problems

1. **Identify the complexity** — What is the Big-O time complexity of the following?
   ```java
   for (int i = 1; i < n; i *= 2) {
       System.out.println(i);
   }
   ```

2. **Compare algorithms** — Write two versions of a function that checks whether a string is a palindrome: one O(n) and one O(n²). Explain the difference.

3. **Space analysis** — Write a method that reverses an array (a) using O(n) extra space and (b) in-place using O(1) extra space.

4. **ADT design** — Design a `Queue` ADT interface in Java with at least 4 operations. Do not implement it — only define the interface.

---

## 1.13 Further Reading

- *Introduction to Algorithms* — Cormen, Leiserson, Rivest, Stein (CLRS)
- *Data Structures and Algorithm Analysis in Java* — Mark Allen Weiss
- Oracle Java Documentation: [https://docs.oracle.com/en/java/](https://docs.oracle.com/en/java/)
- Big-O Cheat Sheet: [https://www.bigocheatsheet.com](https://www.bigocheatsheet.com)

---

*CS 3303 Data Structures — Module 1 of 8*
*Next: Module 2 — Arrays and Linked Lists*
