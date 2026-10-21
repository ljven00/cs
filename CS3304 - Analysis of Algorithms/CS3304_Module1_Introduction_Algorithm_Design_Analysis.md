# CS 3304 — Analysis of Algorithms
## Module 1: Introduction to Algorithm Design & Analysis

---

## 1.1 Introduction

CS 3303 gave you a toolkit of data structures and showed you how to measure the cost of operations with Big-O notation. CS 3304 goes deeper: instead of just *using* algorithms, you will learn to *design* them systematically, *prove* they are correct, and *analyze* them rigorously.

This module establishes the mathematical and conceptual foundations that every subsequent module builds on:

- The full asymptotic notation family (O, Ω, Θ, o, ω)
- Recurrence relations — the language of recursive algorithms
- Amortized analysis — the true cost of operations over time
- Algorithm correctness — loop invariants and formal proofs
- The algorithm design process — from problem to proven solution

These are not abstract formalities. Every real algorithm decision — choosing between two approaches, knowing when a solution is optimal, understanding why a bottleneck exists — rests on this foundation.

---

## 1.2 Review: Asymptotic Notation

Before extending the notation family, let us cement the core idea. Asymptotic notation describes how a function **grows** as its input grows, ignoring constants and lower-order terms.

### Why Ignore Constants?

Constants depend on hardware, compiler, and implementation — they are not intrinsic to the algorithm. What matters is the *growth rate*: an O(n) algorithm will always eventually outperform an O(n²) algorithm, regardless of constants.

```
f(n) = 1000n      → O(n)
g(n) = n² / 1000  → O(n²)

At n = 1,000,001: f = 10⁹  vs  g ≈ 10⁶   (f is already losing)
At n = 2,000,000: f = 2×10⁹ vs g ≈ 4×10⁶ (g is still 500x better)

Wait — at n = 1,000,001, f > g because the constant matters at small n.
But beyond n = 1,000,000 the crossover happens. O analysis captures this.
```

---

## 1.3 The Five Asymptotic Notations

### O (Big-Oh) — Upper Bound

> f(n) = O(g(n)) if there exist constants c > 0 and n₀ ≥ 1 such that:
> **f(n) ≤ c · g(n)** for all n ≥ n₀

Intuition: f grows **no faster than** g. It is a worst-case ceiling.

```
3n² + 5n + 2 = O(n²)   because 3n² + 5n + 2 ≤ 10n² for all n ≥ 1
```

### Ω (Big-Omega) — Lower Bound

> f(n) = Ω(g(n)) if there exist constants c > 0 and n₀ ≥ 1 such that:
> **f(n) ≥ c · g(n)** for all n ≥ n₀

Intuition: f grows **at least as fast as** g. It is a best-case floor.

```
3n² + 5n + 2 = Ω(n²)   because 3n² + 5n + 2 ≥ 3n² for all n ≥ 0
```

### Θ (Theta) — Tight Bound

> f(n) = Θ(g(n)) if f(n) = O(g(n)) **and** f(n) = Ω(g(n))

Intuition: f grows **at exactly the same rate** as g (up to constants).

```
3n² + 5n + 2 = Θ(n²)   because it is both O(n²) and Ω(n²)
```

### o (Little-Oh) — Strict Upper Bound

> f(n) = o(g(n)) if for **every** constant c > 0, there exists n₀ such that:
> **f(n) < c · g(n)** for all n ≥ n₀

Intuition: f grows **strictly slower** than g. The ratio f(n)/g(n) → 0.

```
n = o(n²)   because n/n² = 1/n → 0
n² ≠ o(n²)  because n²/n² = 1 ↛ 0
```

### ω (Little-Omega) — Strict Lower Bound

> f(n) = ω(g(n)) if for **every** constant c > 0, there exists n₀ such that:
> **f(n) > c · g(n)** for all n ≥ n₀

Intuition: f grows **strictly faster** than g. The ratio f(n)/g(n) → ∞.

```
n² = ω(n)   because n²/n = n → ∞
```

### The Analogy to Real Numbers

| Notation | Meaning          | Real number analogy |
|----------|------------------|---------------------|
| f = O(g) | f ≤ g (approx.)  | a ≤ b               |
| f = Ω(g) | f ≥ g (approx.)  | a ≥ b               |
| f = Θ(g) | f = g (approx.)  | a = b               |
| f = o(g) | f < g (strictly) | a < b               |
| f = ω(g) | f > g (strictly) | a > b               |

---

## 1.4 Properties of Asymptotic Notation

These properties let you simplify and combine complexity expressions.

### Transitivity
```
f = O(g) and g = O(h)  →  f = O(h)
(same for Ω, Θ, o, ω)
```

### Reflexivity
```
f = O(f),  f = Ω(f),  f = Θ(f)
```

### Symmetry
```
f = Θ(g)  if and only if  g = Θ(f)
```

### Transpose Symmetry
```
f = O(g)  if and only if  g = Ω(f)
f = o(g)  if and only if  g = ω(f)
```

### Sum Rule
```
O(f(n)) + O(g(n)) = O(max(f(n), g(n)))

Example: O(n²) + O(n) = O(n²)   ← higher-order term dominates
```

### Product Rule
```
O(f(n)) · O(g(n)) = O(f(n) · g(n))

Example: a loop inside a loop: O(n) · O(n) = O(n²)
```

### Common Growth Rate Hierarchy (slowest to fastest)
```
O(1) ⊂ O(log log n) ⊂ O(log n) ⊂ O(√n) ⊂ O(n) ⊂ O(n log n)
   ⊂ O(n²) ⊂ O(n³) ⊂ O(2ⁿ) ⊂ O(n!) ⊂ O(nⁿ)
```

---

## 1.5 Recurrence Relations

Recursive algorithms naturally express their cost as a **recurrence** — a function defined in terms of itself on smaller inputs.

### What is a Recurrence?

```
T(n) = time to solve problem of size n

Merge Sort:   T(n) = 2·T(n/2) + O(n)
              ↑ two recursive calls on half-size  ↑ merge cost

Binary Search: T(n) = T(n/2) + O(1)
               ↑ one recursive call on half-size  ↑ comparison cost

Fibonacci:    T(n) = T(n-1) + T(n-2) + O(1)
```

### Method 1: Substitution Method

Guess the solution, then prove it by induction.

**Example:** Solve T(n) = 2T(n/2) + n

**Guess:** T(n) = O(n log n), i.e., T(n) ≤ cn log n for some c.

**Inductive step:**
```
T(n) = 2T(n/2) + n
     ≤ 2(c·(n/2)·log(n/2)) + n        [apply inductive hypothesis]
     = cn·log(n/2) + n
     = cn·(log n − log 2) + n
     = cn·log n − cn + n
     = cn·log n − (c−1)n
     ≤ cn·log n                         [holds when c ≥ 1]  ✓
```

### Method 2: Recursion Tree Method

Draw the recursion as a tree. Sum costs at each level. Multiply by number of levels.

**Example:** T(n) = 2T(n/2) + n

```
Level 0:         n              cost = n
Level 1:     n/2   n/2          cost = n/2 + n/2 = n
Level 2:   n/4 n/4 n/4 n/4     cost = 4·(n/4) = n
...
Level k:   2^k nodes, each n/2^k  cost = n

Number of levels = log₂(n)
Total cost = n · log₂(n) = O(n log n)
```

```java
/**
 * Recursion tree visualization for T(n) = 2T(n/2) + n
 * Each level costs n work total. There are log(n) levels.
 * Total: O(n log n)
 */
public static void recursionTreeDemo(int n, int level) {
    if (n <= 1) {
        System.out.println("  ".repeat(level) + "Base case (n=1), cost=1");
        return;
    }
    System.out.println("  ".repeat(level) + "Level " + level +
                       ": n=" + n + ", split cost=" + n);
    recursionTreeDemo(n / 2, level + 1); // left subtree
    recursionTreeDemo(n / 2, level + 1); // right subtree
}
```

### Method 3: The Master Theorem

The Master Theorem provides a formula for recurrences of the form:

> **T(n) = a · T(n/b) + f(n)**

where a ≥ 1, b > 1, and f(n) is the cost of non-recursive work.

```
a = number of subproblems
b = factor by which input size shrinks
f(n) = cost of work outside the recursive calls
```

Compare f(n) to n^(log_b(a)) — the "critical exponent":

#### Case 1: f(n) = O(n^(log_b(a) − ε)) for some ε > 0
Recursive work dominates. **T(n) = Θ(n^(log_b(a)))**

#### Case 2: f(n) = Θ(n^(log_b(a)) · log^k(n)) for k ≥ 0
Equal split. **T(n) = Θ(n^(log_b(a)) · log^(k+1)(n))**
Most commonly k=0: **T(n) = Θ(n^(log_b(a)) · log n)**

#### Case 3: f(n) = Ω(n^(log_b(a) + ε)) for some ε > 0 (and regularity condition)
Non-recursive work dominates. **T(n) = Θ(f(n))**

### Master Theorem Examples

```
Merge Sort:    T(n) = 2T(n/2) + n
  a=2, b=2, f(n)=n
  n^(log_2(2)) = n^1 = n
  f(n) = n = Θ(n^1) → Case 2 (k=0)
  T(n) = Θ(n log n)  ✓

Binary Search: T(n) = 1·T(n/2) + O(1)
  a=1, b=2, f(n)=1
  n^(log_2(1)) = n^0 = 1
  f(n) = 1 = Θ(1) → Case 2 (k=0)
  T(n) = Θ(log n)  ✓

Strassen's Matrix Multiply: T(n) = 7T(n/2) + O(n²)
  a=7, b=2, f(n)=n²
  n^(log_2(7)) ≈ n^2.807
  f(n) = n² = O(n^(2.807 − 0.807)) → Case 1
  T(n) = Θ(n^log₂7) ≈ Θ(n^2.807)  ← beats naive O(n³)  ✓

Karatsuba Multiplication: T(n) = 3T(n/2) + O(n)
  a=3, b=2, f(n)=n
  n^(log_2(3)) ≈ n^1.585
  f(n) = n = O(n^(1.585 − 0.585)) → Case 1
  T(n) = Θ(n^log₂3) ≈ Θ(n^1.585)  ← beats naive O(n²)  ✓
```

### Master Theorem in Java Context

```java
/**
 * Concrete examples matching each Master Theorem case.
 */
public class MasterTheoremExamples {

    // Case 1: T(n) = 8T(n/2) + n²  →  T(n) = Θ(n³)
    // log_2(8) = 3,  f(n)=n² = O(n^(3-1))  →  recursive work dominates
    public static int case1Example(int n) {
        if (n <= 1) return 1;
        int sum = 0;
        // 8 recursive calls on n/2
        for (int i = 0; i < 8; i++) sum += case1Example(n / 2);
        // O(n²) non-recursive work
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++) sum++;
        return sum;
    }

    // Case 2: T(n) = 2T(n/2) + n  →  T(n) = Θ(n log n)
    // log_2(2) = 1,  f(n)=n = Θ(n^1)  →  equal split
    public static int case2Example(int[] arr, int l, int r) {
        if (l >= r) return 0;
        int mid  = l + (r - l) / 2;
        int left = case2Example(arr, l, mid);        // T(n/2)
        int right= case2Example(arr, mid + 1, r);    // T(n/2)
        int merge= r - l + 1;                        // O(n) merge work
        return left + right + merge;
    }

    // Case 3: T(n) = 2T(n/2) + n²  →  T(n) = Θ(n²)
    // log_2(2) = 1,  f(n)=n² = Ω(n^(1+1))  →  non-recursive dominates
    public static int case3Example(int n) {
        if (n <= 1) return 1;
        int sum = case3Example(n / 2) + case3Example(n / 2); // 2T(n/2)
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++) sum++;  // O(n²) work
        return sum;
    }
}
```

---

## 1.6 Amortized Analysis

Sometimes a single operation is expensive, but it is expensive *rarely*. **Amortized analysis** computes the average cost per operation over a sequence of operations — giving a more accurate picture than worst-case analysis.

> Amortized cost ≠ average case. There is no probability involved.
> It is the **worst-case average** over any sequence of n operations.

### Three Methods of Amortized Analysis

#### Method 1: Aggregate Analysis

Find the total cost of n operations, divide by n.

**Example: Dynamic Array (ArrayList)**

```
Capacity doubles when full: 1 → 2 → 4 → 8 → 16 → ...

Append n elements (starting from capacity 1):
- Most appends cost O(1) (just store element)
- Resize operations cost: 1 + 2 + 4 + 8 + ... + n ≈ 2n

Total cost = n (regular appends) + 2n (all resizes) = 3n = O(n)
Amortized cost per append = O(n) / n = O(1)  ✓
```

#### Method 2: Accounting Method (Banker's Method)

Assign each operation a "charge" — the amortized cost. When actual cost < charge, store the surplus as "credit". When actual cost > charge, spend stored credit.

```
Dynamic Array — charge each append $3:
  - $1 pays for the current insertion
  - $1 saved for moving this element when the next resize happens
  - $1 saved to move the mirror element (from the first half)

When resize happens: all elements have pre-paid for their move.
Amortized cost = $3 = O(1) per append.
```

```java
/**
 * Dynamic array demonstrating amortized O(1) append.
 * The resize operation is O(n) but happens so rarely that
 * the amortized cost per append remains O(1).
 */
public class DynamicArray {
    private int[] data;
    private int size;
    private int capacity;
    private int totalCopies = 0; // track total work for amortized analysis

    public DynamicArray() {
        capacity = 1;
        data = new int[capacity];
        size = 0;
    }

    public void append(int value) {
        if (size == capacity) {
            // Expensive resize: O(n)
            int[] newData = new int[capacity * 2];
            for (int i = 0; i < size; i++) {
                newData[i] = data[i];
                totalCopies++;
            }
            data     = newData;
            capacity *= 2;
        }
        data[size++] = value;
    }

    public void printAmortizedAnalysis() {
        System.out.println("Elements appended: " + size);
        System.out.println("Total copy operations: " + totalCopies);
        System.out.printf("Copies per append: %.2f (amortized O(1))%n",
                          (double) totalCopies / size);
    }

    public static void main(String[] args) {
        DynamicArray da = new DynamicArray();
        for (int i = 0; i < 1000; i++) da.append(i);
        da.printAmortizedAnalysis();
        // Elements appended: 1000
        // Total copy operations: 999
        // Copies per append: 1.00 (amortized O(1))
    }
}
```

#### Method 3: Potential Method

Define a **potential function** Φ that maps the data structure's state to a non-negative real number. The amortized cost of an operation is:

```
â_i = actual cost + Φ(state after) − Φ(state before)
    = actual cost + ΔΦ
```

**Example: Dynamic Array**

Let Φ = 2 × (size − capacity/2) = 2×size − capacity

```
Regular append (no resize):
  actual cost = 1
  ΔΦ = 2(s+1)−c − (2s−c) = 2
  â = 1 + 2 = 3

Resize append (doubles capacity from c to 2c, size = c+1):
  actual cost = c + 1 (copy c elements + store new)
  ΔΦ = 2(c+1)−2c − (2c−c) = 2 − c
  â = (c+1) + (2−c) = 3

Both cases: amortized cost = 3 = O(1)  ✓
```

---

## 1.7 Algorithm Correctness: Loop Invariants

A **loop invariant** is a property that holds:
1. **Before the first iteration** (initialization)
2. **After each iteration, if it held before** (maintenance)
3. **After the loop terminates** (termination — combined with loop condition, gives correctness)

This is a form of mathematical induction applied to loops.

### Example 1: Insertion Sort Correctness

```java
/**
 * Insertion Sort with loop invariant documentation.
 *
 * LOOP INVARIANT: At the start of each iteration of the outer loop,
 * arr[0..i-1] contains the same elements as the original arr[0..i-1]
 * but in sorted order.
 */
public static void insertionSort(int[] arr) {
    int n = arr.length;

    // INITIALIZATION: i=1, arr[0..0] is trivially sorted (1 element). ✓

    for (int i = 1; i < n; i++) {
        // INVARIANT HOLDS: arr[0..i-1] is sorted

        int key = arr[i]; // element to insert
        int j   = i - 1;

        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j]; // shift right to make room
            j--;
        }
        arr[j + 1] = key; // insert key into correct position

        // MAINTENANCE: arr[0..i] is now sorted
        // (key was inserted into correct position among arr[0..i-1])
    }

    // TERMINATION: i = n, so arr[0..n-1] = entire array is sorted. ✓
}
```

### Example 2: Binary Search Correctness

```java
/**
 * Binary Search with loop invariant.
 *
 * LOOP INVARIANT: If target exists in arr, it is within arr[left..right].
 */
public static int binarySearch(int[] arr, int target) {
    int left = 0, right = arr.length - 1;

    // INITIALIZATION: entire array is arr[0..n-1] = arr[left..right]. ✓

    while (left <= right) {
        // INVARIANT HOLDS: target is in arr[left..right] (if it exists)

        int mid = left + (right - left) / 2;

        if (arr[mid] == target) return mid;

        else if (arr[mid] < target) {
            left = mid + 1;
            // MAINTENANCE: target > arr[mid], so it can't be in arr[left..mid]
            // New search range arr[mid+1..right] still contains target if present. ✓
        }
        else {
            right = mid - 1;
            // MAINTENANCE: target < arr[mid], so it can't be in arr[mid..right]
            // New search range arr[left..mid-1] still contains target if present. ✓
        }
    }

    // TERMINATION: left > right, empty range, target not found. ✓
    return -1;
}
```

---

## 1.8 Algorithm Design Process

A disciplined process for moving from a problem statement to a proven, efficient solution.

```
Step 1: UNDERSTAND THE PROBLEM
  - What are the inputs? What constraints apply?
  - What are the outputs?
  - What edge cases exist? (empty input, duplicates, negatives)
  - Clarify ambiguities before writing code.

Step 2: EXPLORE EXAMPLES
  - Work through small concrete examples by hand.
  - Include edge cases.
  - Identify patterns.

Step 3: DEVISE AN APPROACH
  - What design paradigm fits? (brute force → greedy → D&C → DP → ...)
  - What data structures support the required operations?
  - What is the target complexity? (Can you do better than brute force?)

Step 4: PROVE CORRECTNESS
  - Loop invariants for iterative algorithms
  - Induction for recursive algorithms
  - Argue that all cases are handled

Step 5: ANALYZE COMPLEXITY
  - Time: count operations as a function of n
  - Space: count auxiliary memory
  - Amortized if applicable

Step 6: IMPLEMENT AND TEST
  - Write clean, well-commented code
  - Test against examples from Step 2
  - Test edge cases
  - Compare against brute-force on small inputs
```

### Worked Design Example — Find Two Sum

**Problem:** Given an array of integers and a target, return indices of two numbers that add up to target. Each input has exactly one solution.

```java
public class TwoSumDesign {

    // ── Step 1: Understand ────────────────────────────────────────
    // Input:  int[] nums, int target
    // Output: int[2] with the two indices
    // Constraints: exactly one solution, cannot use same element twice

    // ── Step 2: Explore examples ──────────────────────────────────
    // nums=[2,7,11,15], target=9  → [0,1] (2+7=9)
    // nums=[3,2,4],     target=6  → [1,2] (2+4=6)
    // nums=[3,3],       target=6  → [0,1] (3+3=6)

    // ── Step 3: Approaches ────────────────────────────────────────

    /**
     * Approach A: Brute Force
     * Try every pair. O(n²) time, O(1) space.
     * Correctness: trivially checks all pairs.
     */
    public static int[] twoSumBrute(int[] nums, int target) {
        for (int i = 0; i < nums.length; i++)
            for (int j = i + 1; j < nums.length; j++)
                if (nums[i] + nums[j] == target) return new int[]{i, j};
        return new int[]{};
    }

    /**
     * Approach B: Hash Map — One Pass
     * For each num, check if (target - num) was already seen.
     * O(n) time, O(n) space.
     *
     * CORRECTNESS ARGUMENT:
     * For each element nums[i], we look up complement = target - nums[i].
     * - If complement is in map → we found a valid pair (map stores earlier index)
     * - If not → store nums[i] with its index for future lookups
     * Since there is exactly one solution, we will always find it.
     */
    public static int[] twoSumOptimal(int[] nums, int target) {
        Map<Integer, Integer> seen = new HashMap<>(); // value → index

        for (int i = 0; i < nums.length; i++) {
            int complement = target - nums[i];
            if (seen.containsKey(complement)) {
                return new int[]{seen.get(complement), i};
            }
            seen.put(nums[i], i);
        }
        return new int[]{}; // guaranteed not reached per problem constraints
    }

    // ── Step 5: Complexity ────────────────────────────────────────
    // Brute:   Time O(n²),  Space O(1)
    // Optimal: Time O(n),   Space O(n)
    // Trade-off: linear space for linear time. The problem guarantees one
    //            solution, so no correctness risk from the single-pass approach.

    public static void main(String[] args) {
        int[] nums   = {2, 7, 11, 15};
        int   target = 9;
        System.out.println(Arrays.toString(twoSumBrute(nums, target)));   // [0, 1]
        System.out.println(Arrays.toString(twoSumOptimal(nums, target)));  // [0, 1]
    }
}
```

---

## 1.9 Complexity Classes Preview

We will explore these formally in Module 7 (Computational Complexity Theory), but here is the landscape:

```
P   — problems solvable in polynomial time O(nᵏ)
      Examples: sorting, shortest path, MST

NP  — problems whose solutions can be VERIFIED in polynomial time
      Examples: traveling salesman, graph coloring, satisfiability

NP-Complete — hardest problems in NP
               if any NP-complete problem is in P, then P = NP
      Examples: 3-SAT, Hamiltonian Cycle, Subset Sum

NP-Hard — at least as hard as NP-complete (may not be in NP)
      Examples: Halting Problem, Optimal TSP

The P vs NP question: Is every problem whose solution can be verified
quickly also solvable quickly?
Answer: Unknown. Most computer scientists believe P ≠ NP.
```

```
          ┌─────────────────────────────┐
          │          NP-Hard            │
          │   ┌─────────────────────┐   │
          │   │    NP-Complete      │   │
          │   │   ┌─────────────┐   │   │
          │   │   │     NP      │   │   │
          │   │   │  ┌───────┐  │   │   │
          │   │   │  │   P   │  │   │   │
          │   │   │  └───────┘  │   │   │
          │   │   └─────────────┘   │   │
          │   └─────────────────────┘   │
          └─────────────────────────────┘
          (assuming P ≠ NP)
```

---

## 1.10 Complete Analysis Example — Selection Sort

Let us apply the full process from Section 1.8 to selection sort as a concrete worked example.

```java
/**
 * Selection Sort — Complete Analysis
 */
public class SelectionSortAnalysis {

    /**
     * ALGORITHM:
     *   For i = 0 to n-2:
     *     Find minimum element in arr[i..n-1]
     *     Swap it with arr[i]
     *
     * LOOP INVARIANT:
     *   At the start of each iteration i, arr[0..i-1] contains the
     *   i smallest elements of the original array, in sorted order.
     *
     * INITIALIZATION: i=0, arr[0..-1] is empty — trivially true. ✓
     *
     * MAINTENANCE: We find the minimum of arr[i..n-1] and swap to arr[i].
     *   arr[0..i] now contains the i+1 smallest elements, sorted. ✓
     *
     * TERMINATION: i=n-1, arr[0..n-2] contains n-1 smallest elements sorted,
     *   and arr[n-1] must be the largest. So arr[0..n-1] is sorted. ✓
     *
     * TIME COMPLEXITY:
     *   Outer loop: n-1 iterations
     *   Inner loop: n-1-i iterations (for outer loop value i)
     *   Total comparisons: Σ(i=0 to n-2) (n-1-i) = (n-1) + (n-2) + ... + 1
     *                    = n(n-1)/2 = Θ(n²)
     *   Total swaps: exactly n-1 (one per outer iteration)
     *
     * SPACE COMPLEXITY: O(1) — only constant extra variables
     *
     * NOTE ON SWAPS:
     *   Selection sort makes FEWER swaps than bubble sort (n-1 vs up to n²).
     *   Useful when swapping is expensive (e.g., large records on disk).
     */
    public static void selectionSort(int[] arr) {
        int n = arr.length;
        for (int i = 0; i < n - 1; i++) {
            int minIdx = i;
            for (int j = i + 1; j < n; j++) {
                if (arr[j] < arr[minIdx]) minIdx = j;
            }
            int tmp = arr[i]; arr[i] = arr[minIdx]; arr[minIdx] = tmp;
        }
    }

    public static void main(String[] args) {
        int[] arr = {64, 25, 12, 22, 11};
        selectionSort(arr);
        System.out.println(Arrays.toString(arr)); // [11, 12, 22, 25, 64]
    }
}
```

---

## 1.11 Key Takeaways

| Concept                 | Summary                                                                          |
|-------------------------|----------------------------------------------------------------------------------|
| O(f(n))                 | Upper bound — algorithm uses at most this many resources                         |
| Ω(f(n))                 | Lower bound — algorithm uses at least this many resources                        |
| Θ(f(n))                 | Tight bound — exact characterization of growth rate                              |
| o / ω                   | Strict upper/lower bounds; ratio approaches 0 or ∞                              |
| Recurrence relation     | Expresses recursive algorithm cost in terms of smaller subproblems               |
| Substitution method     | Guess + prove by induction                                                       |
| Recursion tree          | Visualize cost at each level; sum across all levels                              |
| Master Theorem          | Direct formula for T(n) = aT(n/b) + f(n); three cases                          |
| Amortized analysis      | True per-operation cost averaged over a sequence — not probabilistic             |
| Aggregate method        | Total cost / n operations                                                        |
| Accounting method       | Charge operations, store/spend credit                                            |
| Potential method        | Assign potential to data structure state; amortized = actual + ΔΦ               |
| Loop invariant          | Property true before, during, and after loop — proves correctness                |
| Algorithm design steps  | Understand → Example → Approach → Prove → Analyze → Implement → Test            |

---

## 1.12 Practice Problems

1. **Notation proofs** — Prove formally (using the definition) that 5n³ + 3n² + 100 = Θ(n³). Find explicit constants c₁, c₂, and n₀.

2. **Master Theorem** — Apply the Master Theorem to solve each recurrence:
   - T(n) = 4T(n/2) + n
   - T(n) = 4T(n/2) + n²
   - T(n) = 4T(n/2) + n³
   (Note how the dominant term shifts across the three cases.)

3. **Amortized analysis** — A stack supports `push`, `pop`, and `multipop(k)` which pops min(k, size) elements. Show that the amortized cost of any sequence of n operations is O(n). *(Hint: use accounting method — each element pays for its own push and pop.)*

4. **Loop invariant** — Write a function that reverses an array in-place, and prove its correctness using a loop invariant.

5. **Recursion tree** — Use the recursion tree method (not the Master Theorem) to solve T(n) = 3T(n/3) + n. Draw the first three levels of the tree, compute cost per level, and sum across all levels.

---

## 1.13 Further Reading

- *Introduction to Algorithms* (CLRS) — Chapters 1–5 (foundations, asymptotic notation, recurrences, probabilistic analysis)
- *The Algorithm Design Manual* — Skiena, Chapters 1–2
- *Algorithms* — Sedgewick & Wayne, Chapter 1
- Master Theorem proof: CLRS Chapter 4
- Amortized analysis: CLRS Chapter 17

---

*CS 3304 Analysis of Algorithms — Module 1 of 8*
*Next: Module 2 — Advanced Algorithm Techniques Part 1: Divide and Conquer*
