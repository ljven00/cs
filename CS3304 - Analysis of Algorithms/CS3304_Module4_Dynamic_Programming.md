# CS 3304 — Analysis of Algorithms
## Module 4: Dynamic Programming Techniques

---

## 4.1 Introduction

In Module 3 we saw that greedy algorithms make irrevocable local choices and work beautifully — but only when the greedy choice property holds. Many important problems lack this property: the locally optimal choice at each step does not guarantee a globally optimal solution.

**Dynamic Programming (DP)** is the technique for these problems. Instead of committing greedily to one choice, DP systematically explores all choices — but does so efficiently by storing and reusing the results of overlapping subproblems rather than recomputing them.

The name "dynamic programming" was coined by Richard Bellman in the 1950s as a deliberate obfuscation to hide the mathematical nature of his research from a skeptical Secretary of Defense. The name stuck. The technique itself is one of the most powerful ideas in all of computer science.

> **Core idea:** Every optimal solution is built from optimal solutions to smaller subproblems. Solve each subproblem exactly once, store its result, and look it up when needed.

By the end of this module you will master:
- The two essential DP properties and how to verify them
- Memoization (top-down) and tabulation (bottom-up) — with trade-off analysis
- How to define a DP state and write the recurrence relation
- Eight classic DP problems with full Java implementations and complexity proofs
- The transition from exponential brute force to polynomial DP

---

## 4.2 The Two DP Properties

A problem is amenable to dynamic programming if and only if it has both:

### Property 1: Optimal Substructure

> An optimal solution to the problem contains optimal solutions to its subproblems.

This means: if you cut out a piece of the optimal solution, that piece must itself be optimal for its own sub-problem. If you could improve a sub-piece, you could improve the whole — contradicting optimality.

**Has optimal substructure:**
- Shortest path (Bellman-Ford, Dijkstra, Floyd-Warshall)
- Knapsack, coin change, LCS, edit distance

**Does NOT have optimal substructure:**
- Longest simple path (cycle-free path) in a graph — the longest path from u to v does not contain the longest path from u to an intermediate w
- Unweighted shortest path is in P, but longest simple path is NP-hard

### Property 2: Overlapping Subproblems

> The recursive solution solves the same subproblems repeatedly.

This is what distinguishes DP from Divide and Conquer. In D&C (Merge Sort, Quick Sort), the subproblems are **independent** — the left half of the array has nothing to do with the right half. In DP, subproblems **share sub-subproblems**.

```
Fibonacci(5)
├── Fibonacci(4)
│   ├── Fibonacci(3)          ← computed AGAIN below
│   │   ├── Fibonacci(2)      ← computed AGAIN below
│   │   └── Fibonacci(1)
│   └── Fibonacci(2)          ← DUPLICATE
└── Fibonacci(3)              ← DUPLICATE
    ├── Fibonacci(2)          ← DUPLICATE
    └── Fibonacci(1)

Without memoization: O(2^n) calls
With memoization:    O(n) calls — each subproblem solved once
```

---

## 4.3 The DP Design Recipe

Every DP solution follows a disciplined process:

```
Step 1: DEFINE THE SUBPROBLEM
  What is dp[i] (or dp[i][j]) computing?
  Be precise: "dp[i] = the maximum value achievable using the first i items"

Step 2: WRITE THE RECURRENCE
  How does dp[i] relate to smaller subproblems?
  Express dp[i] in terms of dp[0], dp[1], ..., dp[i-1]

Step 3: IDENTIFY BASE CASES
  What are the smallest subproblems you can answer directly?
  dp[0] = ?, dp[1] = ?

Step 4: DETERMINE COMPUTATION ORDER
  Top-down (memoization): compute on demand via recursion
  Bottom-up (tabulation):  fill the table from smallest to largest

Step 5: EXTRACT THE ANSWER
  Where in the table is the final answer?
  Is reconstruction (finding the actual solution, not just its value) needed?

Step 6: ANALYZE COMPLEXITY
  Time = (number of subproblems) × (time per subproblem)
  Space = size of memoization table (often reducible)
```

---

## 4.4 Memoization vs. Tabulation

Both approaches compute the same values. The choice affects constant factors, stack usage, and which subproblems are computed.

### Top-Down: Memoization

```java
/**
 * Fibonacci — Top-Down Memoization.
 *
 * Recursion with a cache. Only computes subproblems that are actually needed.
 * Natural: mirrors the mathematical recurrence directly.
 *
 * Time:  O(n) — each subproblem computed once
 * Space: O(n) cache + O(n) call stack
 */
public static long fibMemo(int n, long[] memo) {
    if (n <= 1)       return n;               // base cases
    if (memo[n] != 0) return memo[n];         // cache hit — no recomputation

    memo[n] = fibMemo(n - 1, memo) + fibMemo(n - 2, memo); // compute and cache
    return memo[n];
}

// Usage:
long[] memo = new long[100];
System.out.println(fibMemo(50, memo)); // 12586269025
```

### Bottom-Up: Tabulation

```java
/**
 * Fibonacci — Bottom-Up Tabulation.
 *
 * Fill the table from base cases upward. No recursion overhead.
 * Computes ALL subproblems from 0 to n (even unused ones).
 *
 * Time:  O(n)
 * Space: O(n) table — reducible to O(1) with rolling variables
 */
public static long fibTab(int n) {
    if (n <= 1) return n;
    long[] dp = new long[n + 1];
    dp[0] = 0; dp[1] = 1;                    // base cases

    for (int i = 2; i <= n; i++) {
        dp[i] = dp[i-1] + dp[i-2];           // recurrence
    }
    return dp[n];
}

/**
 * Space-optimized: O(1) — only keep last two values
 */
public static long fibO1Space(int n) {
    if (n <= 1) return n;
    long prev2 = 0, prev1 = 1;
    for (int i = 2; i <= n; i++) {
        long curr = prev1 + prev2;
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
```

### Comparison Table

| Feature                  | Top-Down (Memoization)          | Bottom-Up (Tabulation)            |
|--------------------------|---------------------------------|-----------------------------------|
| Implementation style     | Recursive                       | Iterative                         |
| Subproblems computed     | Only needed ones                | All subproblems                   |
| Call stack overhead      | O(depth) — risk of overflow     | O(1)                              |
| Code clarity             | Mirrors recurrence directly     | Requires order-of-filling logic   |
| Space optimization       | Harder                          | Easier (sliding window, etc.)     |
| Best when                | Not all subproblems needed      | All subproblems needed; large n   |

---

## 4.5 0/1 Knapsack

**Problem:** n items, each with weight w[i] and value v[i]. A knapsack of capacity W. Select items (whole only — no fractions) to maximize total value without exceeding W.

### Why Greedy Fails (Recap)

As shown in Module 3, sorting by value density and taking items greedily can miss the optimal combination. We need to explore all subsets — but there are 2^n subsets.

### DP Formulation

```
State:  dp[i][j] = maximum value achievable using items {1..i} with capacity j

Recurrence:
  dp[i][j] = dp[i-1][j]                          if w[i] > j  (can't take item i)
           = max(dp[i-1][j],                      otherwise
                 dp[i-1][j - w[i]] + v[i])
           ↑ skip item i    ↑ take item i

Base case:  dp[0][j] = 0 for all j  (no items → value 0)
            dp[i][0] = 0 for all i  (no capacity → value 0)

Answer:     dp[n][W]
```

```java
/**
 * 0/1 Knapsack — Bottom-Up DP.
 *
 * Time:  O(n * W)   — pseudo-polynomial (polynomial in n and W, not input bits)
 * Space: O(n * W)   — reducible to O(W) with 1D rolling array
 */
public static int knapsack(int[] weights, int[] values, int W) {
    int n = weights.length;
    int[][] dp = new int[n + 1][W + 1];

    // Base cases: dp[0][j] = 0 already (Java default)

    for (int i = 1; i <= n; i++) {
        for (int j = 0; j <= W; j++) {
            // Option 1: skip item i
            dp[i][j] = dp[i-1][j];

            // Option 2: take item i (only if it fits)
            if (weights[i-1] <= j) {
                dp[i][j] = Math.max(dp[i][j],
                                    dp[i-1][j - weights[i-1]] + values[i-1]);
            }
        }
    }

    return dp[n][W];
}

/**
 * Space-optimized: O(W) — use 1D array, iterate j in reverse.
 * Reverse iteration ensures dp[j - w[i]] refers to the PREVIOUS row
 * (i.e., before item i was considered).
 */
public static int knapsackO1Space(int[] weights, int[] values, int W) {
    int[] dp = new int[W + 1];

    for (int i = 0; i < weights.length; i++) {
        // Traverse RIGHT-TO-LEFT to avoid using item i twice
        for (int j = W; j >= weights[i]; j--) {
            dp[j] = Math.max(dp[j], dp[j - weights[i]] + values[i]);
        }
    }
    return dp[W];
}

/**
 * Reconstruct which items were selected.
 */
public static List<Integer> knapsackItems(int[] weights, int[] values, int W) {
    int n   = weights.length;
    int[][] dp = new int[n + 1][W + 1];

    for (int i = 1; i <= n; i++)
        for (int j = 0; j <= W; j++) {
            dp[i][j] = dp[i-1][j];
            if (weights[i-1] <= j)
                dp[i][j] = Math.max(dp[i][j],
                                    dp[i-1][j - weights[i-1]] + values[i-1]);
        }

    // Trace back through table to find selected items
    List<Integer> items = new ArrayList<>();
    int j = W;
    for (int i = n; i > 0; i--) {
        if (dp[i][j] != dp[i-1][j]) {   // item i was taken
            items.add(i);
            j -= weights[i-1];
        }
    }
    Collections.reverse(items);
    return items;
}

public static void main(String[] args) {
    int[] weights = {1, 3, 4, 5};
    int[] values  = {1, 4, 5, 7};
    int   W       = 7;

    System.out.println("Max value: " + knapsack(weights, values, W)); // 9
    System.out.println("Items:     " + knapsackItems(weights, values, W)); // [2, 3]
    // Items 2 (w=3,v=4) + 3 (w=4,v=5) = weight 7, value 9
}
```

---

## 4.6 Longest Common Subsequence (LCS)

**Problem:** Given two strings X and Y, find the length of their longest common subsequence — a subsequence that appears in both strings (not necessarily contiguous).

```
X = "ABCBDAB"
Y = "BDCAB"

LCS = "BCAB" or "BDAB" — length 4
```

### DP Formulation

```
State:  dp[i][j] = length of LCS of X[0..i-1] and Y[0..j-1]

Recurrence:
  If X[i-1] == Y[j-1]:  dp[i][j] = dp[i-1][j-1] + 1   (extend LCS)
  Else:                  dp[i][j] = max(dp[i-1][j],      (skip X[i-1])
                                        dp[i][j-1])       (skip Y[j-1])

Base case:  dp[0][j] = dp[i][0] = 0

Answer:     dp[m][n]
```

```java
/**
 * Longest Common Subsequence — Bottom-Up DP.
 *
 * Time:  O(m * n)
 * Space: O(m * n) — reducible to O(min(m,n)) with rolling rows
 */
public static int lcs(String X, String Y) {
    int m = X.length(), n = Y.length();
    int[][] dp = new int[m + 1][n + 1];

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (X.charAt(i-1) == Y.charAt(j-1)) {
                dp[i][j] = dp[i-1][j-1] + 1;         // characters match
            } else {
                dp[i][j] = Math.max(dp[i-1][j], dp[i][j-1]); // take best skip
            }
        }
    }
    return dp[m][n];
}

/**
 * Reconstruct the actual LCS string by tracing back through the table.
 */
public static String lcsString(String X, String Y) {
    int m = X.length(), n = Y.length();
    int[][] dp = new int[m + 1][n + 1];

    for (int i = 1; i <= m; i++)
        for (int j = 1; j <= n; j++)
            dp[i][j] = (X.charAt(i-1) == Y.charAt(j-1))
                       ? dp[i-1][j-1] + 1
                       : Math.max(dp[i-1][j], dp[i][j-1]);

    // Trace back
    StringBuilder sb = new StringBuilder();
    int i = m, j = n;
    while (i > 0 && j > 0) {
        if (X.charAt(i-1) == Y.charAt(j-1)) {
            sb.append(X.charAt(i-1));  // this char is in LCS
            i--; j--;
        } else if (dp[i-1][j] > dp[i][j-1]) {
            i--;                        // came from above
        } else {
            j--;                        // came from left
        }
    }
    return sb.reverse().toString();
}

// lcsString("ABCBDAB", "BDCAB") → "BCAB" (length 4)
```

### DP Table Visualization

```
    ""  B  D  C  A  B
""  [0  0  0  0  0  0]
A   [0  0  0  0  1  1]
B   [0  1  1  1  1  2]
C   [0  1  1  2  2  2]
B   [0  1  1  2  2  3]
D   [0  1  2  2  2  3]
A   [0  1  2  2  3  3]
B   [0  1  2  2  3  4]  ← answer: dp[7][5] = 4
```

---

## 4.7 Edit Distance (Levenshtein Distance)

**Problem:** Given two strings, find the minimum number of single-character edits (insert, delete, substitute) to transform one string into the other.

Applications: spell checkers, DNA sequence alignment, diff tools, plagiarism detection.

```
"KITTEN" → "SITTING"

KITTEN → SITTEN  (substitute K→S)
SITTEN → SITTIN  (substitute E→I)
SITTIN → SITTING (insert G)

Edit distance = 3
```

### DP Formulation

```
State:  dp[i][j] = min edits to transform X[0..i-1] into Y[0..j-1]

Recurrence:
  If X[i-1] == Y[j-1]:  dp[i][j] = dp[i-1][j-1]          (no edit needed)
  Else: dp[i][j] = 1 + min(
                    dp[i-1][j],    ← delete X[i-1]
                    dp[i][j-1],    ← insert Y[j-1] into X
                    dp[i-1][j-1]   ← substitute X[i-1] with Y[j-1]
                  )

Base cases: dp[i][0] = i  (delete all i characters)
            dp[0][j] = j  (insert all j characters)
```

```java
/**
 * Edit Distance (Levenshtein) — Bottom-Up DP.
 *
 * Time:  O(m * n)
 * Space: O(m * n) — reducible to O(min(m,n))
 */
public static int editDistance(String X, String Y) {
    int m = X.length(), n = Y.length();
    int[][] dp = new int[m + 1][n + 1];

    // Base cases: transforming to/from empty string
    for (int i = 0; i <= m; i++) dp[i][0] = i; // delete all
    for (int j = 0; j <= n; j++) dp[0][j] = j; // insert all

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (X.charAt(i-1) == Y.charAt(j-1)) {
                dp[i][j] = dp[i-1][j-1];      // characters match: free
            } else {
                dp[i][j] = 1 + Math.min(
                    dp[i-1][j],      // delete
                    Math.min(
                        dp[i][j-1],  // insert
                        dp[i-1][j-1] // substitute
                    )
                );
            }
        }
    }
    return dp[m][n];
}

/**
 * Reconstruct the sequence of operations.
 */
public static List<String> editOperations(String X, String Y) {
    int m = X.length(), n = Y.length();
    int[][] dp = new int[m + 1][n + 1];
    for (int i = 0; i <= m; i++) dp[i][0] = i;
    for (int j = 0; j <= n; j++) dp[0][j] = j;
    for (int i = 1; i <= m; i++)
        for (int j = 1; j <= n; j++) {
            if (X.charAt(i-1) == Y.charAt(j-1)) dp[i][j] = dp[i-1][j-1];
            else dp[i][j] = 1 + Math.min(dp[i-1][j],
                                Math.min(dp[i][j-1], dp[i-1][j-1]));
        }

    // Trace back
    List<String> ops = new ArrayList<>();
    int i = m, j = n;
    while (i > 0 || j > 0) {
        if (i > 0 && j > 0 && X.charAt(i-1) == Y.charAt(j-1)) {
            i--; j--;
        } else if (j > 0 && (i == 0 || dp[i][j-1] <= dp[i-1][j] &&
                                        dp[i][j-1] <= dp[i-1][j-1])) {
            ops.add("Insert '" + Y.charAt(j-1) + "' at position " + i);
            j--;
        } else if (i > 0 && (j == 0 || dp[i-1][j] <= dp[i][j-1] &&
                                        dp[i-1][j] <= dp[i-1][j-1])) {
            ops.add("Delete '" + X.charAt(i-1) + "' at position " + i);
            i--;
        } else {
            ops.add("Substitute '" + X.charAt(i-1) + "' → '" +
                    Y.charAt(j-1) + "' at position " + i);
            i--; j--;
        }
    }
    Collections.reverse(ops);
    return ops;
}

// editDistance("kitten", "sitting") → 3
```

---

## 4.8 Longest Increasing Subsequence (LIS)

**Problem:** Find the length of the longest strictly increasing subsequence in an array.

```
arr = [10, 9, 2, 5, 3, 7, 101, 18]
LIS = [2, 3, 7, 18]  or  [2, 5, 7, 18]  or  [2, 5, 7, 101] — length 4
```

### DP Formulation — O(n²)

```
State:  dp[i] = length of LIS ending at index i

Recurrence:
  dp[i] = 1 + max(dp[j]) for all j < i where arr[j] < arr[i]
  dp[i] = 1 if no such j exists

Answer: max(dp[i]) for all i
```

```java
/**
 * Longest Increasing Subsequence — O(n²) DP.
 *
 * Time:  O(n²)
 * Space: O(n)
 */
public static int lisDP(int[] arr) {
    int n  = arr.length;
    int[] dp = new int[n];
    Arrays.fill(dp, 1);    // every element is an LIS of length 1 alone

    for (int i = 1; i < n; i++) {
        for (int j = 0; j < i; j++) {
            if (arr[j] < arr[i]) {
                dp[i] = Math.max(dp[i], dp[j] + 1); // extend LIS ending at j
            }
        }
    }
    return Arrays.stream(dp).max().getAsInt();
}

/**
 * LIS — O(n log n) using patience sorting + binary search.
 * Maintains a "tails" array where tails[i] = smallest tail element
 * of all increasing subsequences of length i+1.
 *
 * Time:  O(n log n)
 * Space: O(n)
 */
public static int lisNLogN(int[] arr) {
    List<Integer> tails = new ArrayList<>();

    for (int num : arr) {
        // Binary search: find leftmost position where tails[pos] >= num
        int lo = 0, hi = tails.size();
        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (tails.get(mid) < num) lo = mid + 1;
            else                      hi = mid;
        }

        if (lo == tails.size()) tails.add(num);  // extend the longest
        else                    tails.set(lo, num); // replace to keep tails small
    }
    return tails.size();
}

// arr=[10,9,2,5,3,7,101,18] → LIS length = 4
```

---

## 4.9 Matrix Chain Multiplication

**Problem:** Given n matrices A₁, A₂, ..., Aₙ where matrix Aᵢ has dimensions p[i-1] × p[i], find the optimal order to multiply them to minimize the total number of scalar multiplications.

```
Three matrices: A(10×30), B(30×5), C(5×60)

(AB)C: (10×30×5) + (10×5×60) = 1500 + 3000 = 4500 multiplications
A(BC): (30×5×60) + (10×30×60) = 9000 + 18000 = 27000 multiplications

Order matters enormously!
```

### DP Formulation

```
State:  dp[i][j] = minimum multiplications to compute A[i] * A[i+1] * ... * A[j]

Recurrence: try every split point k between i and j:
  dp[i][j] = min over k in [i, j-1] of:
              dp[i][k] + dp[k+1][j] + p[i-1] * p[k] * p[j]
              ↑ left chain  ↑ right chain  ↑ cost to multiply results

Base case:  dp[i][i] = 0  (single matrix, no multiplication needed)

Computation order: fill by increasing chain length (l = 2, 3, ..., n)
```

```java
/**
 * Matrix Chain Multiplication — Bottom-Up DP.
 *
 * Time:  O(n³)
 * Space: O(n²)
 *
 * n = number of matrices
 * p = dimension array of size n+1 (matrix i has dimensions p[i-1] × p[i])
 */
public static int matrixChain(int[] p) {
    int n   = p.length - 1;     // number of matrices
    int[][] dp    = new int[n][n];
    int[][] split = new int[n][n]; // optimal split point for reconstruction

    // dp[i][i] = 0 (already initialized to 0 by Java)

    // Fill by chain length: l = 2 means 2 consecutive matrices
    for (int l = 2; l <= n; l++) {
        for (int i = 0; i <= n - l; i++) {
            int j = i + l - 1;
            dp[i][j] = Integer.MAX_VALUE;

            // Try every split point k in [i, j-1]
            for (int k = i; k < j; k++) {
                int cost = dp[i][k] + dp[k+1][j] + p[i] * p[k+1] * p[j+1];
                if (cost < dp[i][j]) {
                    dp[i][j]    = cost;
                    split[i][j] = k;     // remember best split
                }
            }
        }
    }

    System.out.println("Optimal parenthesization: " +
                       optimalParens(split, 0, n-1));
    return dp[0][n-1];
}

/**
 * Reconstruct the optimal parenthesization using the split table.
 */
private static String optimalParens(int[][] split, int i, int j) {
    if (i == j) return "A" + (i + 1);
    return "(" + optimalParens(split, i, split[i][j]) +
           " × " +
           optimalParens(split, split[i][j] + 1, j) + ")";
}

// p = [10, 30, 5, 60] → 3 matrices: A1(10×30), A2(30×5), A3(5×60)
// matrixChain(p) → 4500, parenthesization: ((A1 × A2) × A3)
```

---

## 4.10 Coin Change

**Problem:** Given coin denominations and an amount, find the **minimum number of coins** to make the amount.

```
coins = [1, 5, 6, 9],  amount = 11

Greedy fails: 9 + 1 + 1 = 3 coins
DP optimal:  6 + 5     = 2 coins
```

### DP Formulation

```
State:  dp[a] = minimum coins needed to make amount a

Recurrence:
  dp[a] = min over all coins c where c ≤ a of:
           1 + dp[a - c]

Base case:  dp[0] = 0  (0 coins needed for amount 0)
            dp[a] = ∞  initially

Answer:     dp[amount]
```

```java
/**
 * Coin Change — Minimum Coins — Bottom-Up DP.
 *
 * Time:  O(amount * n) where n = number of coin denominations
 * Space: O(amount)
 */
public static int coinChangeMin(int[] coins, int amount) {
    int[] dp = new int[amount + 1];
    Arrays.fill(dp, amount + 1); // initialize to "infinity"
    dp[0] = 0;

    for (int a = 1; a <= amount; a++) {
        for (int coin : coins) {
            if (coin <= a) {
                dp[a] = Math.min(dp[a], 1 + dp[a - coin]); // try this coin
            }
        }
    }
    return dp[amount] > amount ? -1 : dp[amount]; // -1 if impossible
}

/**
 * Coin Change — Count ALL ways to make the amount.
 *
 * State:  dp[a] = number of ways to make amount a
 * This is the "unbounded knapsack" variant.
 *
 * NOTE: Outer loop over coins, inner over amounts → counts combinations
 *       (not permutations: [1,2] and [2,1] count as one way)
 * Time:  O(amount * n)
 */
public static long coinChangeWays(int[] coins, int amount) {
    long[] dp = new long[amount + 1];
    dp[0] = 1; // one way to make 0: use no coins

    for (int coin : coins) {            // outer: coins (avoid counting order)
        for (int a = coin; a <= amount; a++) {
            dp[a] += dp[a - coin];
        }
    }
    return dp[amount];
}

public static void main(String[] args) {
    int[] coins  = {1, 5, 6, 9};
    int   amount = 11;

    System.out.println("Min coins: " + coinChangeMin(coins, amount));  // 2
    System.out.println("Ways:      " + coinChangeWays(coins, amount)); // number of combos
}
```

---

## 4.11 Maximum Subarray — Kadane's Algorithm

**Problem:** Find the contiguous subarray with the maximum sum.

In Module 2 we solved this in O(n log n) with D&C. Kadane's algorithm achieves O(n) with DP.

```
arr = [-2, 1, -3, 4, -1, 2, 1, -5, 4]
Max subarray = [4, -1, 2, 1] → sum = 6
```

### DP Formulation

```
State:  dp[i] = maximum subarray sum ending at index i

Recurrence:
  dp[i] = max(arr[i],           ← start a new subarray here
              dp[i-1] + arr[i]) ← extend the best subarray ending at i-1

Intuition: either start fresh at arr[i], or extend whatever was best before.

Answer: max(dp[i]) for all i
```

```java
/**
 * Kadane's Algorithm — O(n) DP for Maximum Subarray Sum.
 *
 * Time:  O(n)
 * Space: O(1) — only need the previous dp value
 *
 * Note: dp[i] depends only on dp[i-1] → no array needed.
 */
public static int maxSubarray(int[] arr) {
    int maxEndingHere = arr[0]; // dp[i]: best sum ending at current position
    int maxSoFar      = arr[0]; // global maximum

    for (int i = 1; i < arr.length; i++) {
        maxEndingHere = Math.max(arr[i], maxEndingHere + arr[i]); // recurrence
        maxSoFar      = Math.max(maxSoFar, maxEndingHere);         // track global max
    }
    return maxSoFar;
}

/**
 * Extended Kadane's — also return the subarray indices.
 */
public static int[] maxSubarrayWithIndices(int[] arr) {
    int maxEndingHere = arr[0], maxSoFar = arr[0];
    int start = 0, end = 0, tempStart = 0;

    for (int i = 1; i < arr.length; i++) {
        if (arr[i] > maxEndingHere + arr[i]) {
            maxEndingHere = arr[i];
            tempStart     = i;           // new subarray starts here
        } else {
            maxEndingHere += arr[i];
        }
        if (maxEndingHere > maxSoFar) {
            maxSoFar = maxEndingHere;
            start    = tempStart;
            end      = i;
        }
    }
    return new int[]{maxSoFar, start, end};
}

// arr=[-2,1,-3,4,-1,2,1,-5,4] → {6, 3, 6} (sum=6, indices 3..6)
```

---

## 4.12 Rod Cutting

**Problem:** A rod of length n. Given prices p[i] for rods of each length i, find the maximum revenue from cutting the rod.

```
Length:  1   2   3   4   5   6   7   8   9   10
Price:   1   5   8   9  10  17  17  20  24  30

Rod of length 4:
  Options: (4), (1+3), (2+2), (1+1+2), (1+1+1+1)
  Revenue:  9    1+8=9  5+5=10  1+1+5=7   4
  Optimal:  cut into 2+2 → revenue 10
```

### DP Formulation

```
State:  dp[n] = max revenue from rod of length n

Recurrence:
  dp[n] = max over i in [1,n] of:
           price[i] + dp[n-i]     ← cut first piece of length i, recurse on rest

Base case:  dp[0] = 0
```

```java
/**
 * Rod Cutting — Bottom-Up DP.
 *
 * Time:  O(n²)
 * Space: O(n)
 */
public static int rodCutting(int[] price, int n) {
    int[] dp = new int[n + 1];
    // dp[0] = 0 (base case)

    for (int len = 1; len <= n; len++) {
        int maxVal = Integer.MIN_VALUE;
        for (int cut = 1; cut <= len; cut++) {
            maxVal = Math.max(maxVal, price[cut - 1] + dp[len - cut]);
        }
        dp[len] = maxVal;
    }
    return dp[n];
}

/**
 * Reconstruct optimal cutting strategy.
 */
public static List<Integer> rodCutSolution(int[] price, int n) {
    int[]  dp    = new int[n + 1];
    int[]  firstCut = new int[n + 1]; // stores optimal first cut for each length

    for (int len = 1; len <= n; len++) {
        dp[len] = Integer.MIN_VALUE;
        for (int cut = 1; cut <= len; cut++) {
            if (price[cut-1] + dp[len-cut] > dp[len]) {
                dp[len]       = price[cut-1] + dp[len-cut];
                firstCut[len] = cut;
            }
        }
    }

    List<Integer> cuts = new ArrayList<>();
    while (n > 0) {
        cuts.add(firstCut[n]);
        n -= firstCut[n];
    }
    return cuts;
}

// price=[1,5,8,9,10,17,17,20,24,30], n=4 → revenue=10, cuts=[2,2]
```

---

## 4.13 DP on Intervals — Burst Balloons

**Problem:** Given n balloons with values nums[i], burst them one at a time. Bursting balloon i earns nums[i-1] * nums[i] * nums[i+1] coins. Maximize total coins.

This problem has a non-obvious DP formulation: instead of thinking about which balloon to burst **first**, think about which balloon to burst **last** in each interval.

```
State:  dp[i][j] = max coins from bursting all balloons in interval (i,j) exclusive
                   (balloons i and j are NOT burst — they are boundary sentinels)

Recurrence:
  dp[i][j] = max over k in (i, j) of:
              dp[i][k] + nums[i]*nums[k]*nums[j] + dp[k][j]
              ↑ burst left   ↑ k is last to burst in (i,j)  ↑ burst right

Key insight: k is the LAST balloon burst in interval (i,j).
When k is burst, i and j are still present as its neighbors.

Base case: dp[i][j] = 0 when j <= i+1 (empty interval)
Answer: dp[0][n+1]
```

```java
/**
 * Burst Balloons — Interval DP.
 *
 * Time:  O(n³)
 * Space: O(n²)
 *
 * This is a classic example of "think about what happens LAST"
 * instead of what happens first — a powerful DP trick for interval problems.
 */
public static int maxCoins(int[] nums) {
    int n = nums.length;

    // Add boundary sentinels (value 1) at both ends
    int[] balloons = new int[n + 2];
    balloons[0] = balloons[n + 1] = 1;
    for (int i = 0; i < n; i++) balloons[i + 1] = nums[i];

    int m    = n + 2;
    int[][] dp = new int[m][m];

    // Fill by interval length
    for (int len = 2; len < m; len++) {
        for (int i = 0; i < m - len; i++) {
            int j = i + len;
            // Try every k as the last balloon burst in (i, j)
            for (int k = i + 1; k < j; k++) {
                int coins = balloons[i] * balloons[k] * balloons[j]
                          + dp[i][k] + dp[k][j];
                dp[i][j] = Math.max(dp[i][j], coins);
            }
        }
    }
    return dp[0][m - 1];
}

// nums=[3,1,5,8] → 167
// Optimal order: burst 1(coins=3×1×5=15), burst 5(3×5×8=120), burst 3(1×3×8=24), burst 8(1×8×1=8)
// Total = 167
```

---

## 4.14 DP Complexity Patterns

Most DP problems fall into one of these templates. Recognizing the template tells you the expected complexity immediately.

| Template           | State           | Recurrence                        | Time      | Space   | Examples                        |
|--------------------|-----------------|-----------------------------------|-----------|---------|---------------------------------|
| 1D linear          | dp[i]           | dp[i] = f(dp[i-1], dp[i-2], ...) | O(n)      | O(n)→O(1)| Fibonacci, Kadane, Rod Cutting |
| 1D with inner loop | dp[i]           | dp[i] = max/min over j < i        | O(n²)     | O(n)    | LIS, Coin Change               |
| 2D strings         | dp[i][j]        | f(dp[i-1][j], dp[i][j-1], dp[i-1][j-1]) | O(mn) | O(mn)→O(n) | LCS, Edit Distance        |
| 2D knapsack        | dp[i][j]        | f(dp[i-1][j], dp[i-1][j-w])      | O(nW)     | O(nW)→O(W)| 0/1 Knapsack               |
| Interval           | dp[i][j]        | max over split k in (i,j)         | O(n³)     | O(n²)   | Matrix Chain, Burst Balloons    |
| Bitmask            | dp[mask][i]     | f(dp[mask without i])             | O(2^n * n)| O(2^n * n)| TSP, Assignment              |
| Tree DP            | dp[node][state] | f(dp[children])                   | O(n)      | O(n)    | Tree diameter, root selection   |

---

## 4.15 Common DP Pitfalls and Debugging Tips

```
PITFALL 1: Wrong state definition
  Bad:  dp[i] = "something about position i"  ← too vague
  Good: dp[i] = "maximum value achievable using the first i items"  ← precise

PITFALL 2: Missing base cases
  Always ask: what happens when i=0? j=0? empty input?
  Java initializes arrays to 0 — verify this is the correct base case value.

PITFALL 3: Wrong traversal order
  For 0/1 knapsack: j must go RIGHT-TO-LEFT in the 1D optimization
  For coin change (unbounded): j goes LEFT-TO-RIGHT (items can be reused)

PITFALL 4: Integer overflow
  dp values representing counts can grow exponentially
  Use long instead of int for counting problems.

PITFALL 5: Off-by-one in indexing
  Most string DP uses dp[i][j] for the first i characters of X and j of Y
  Access X.charAt(i-1) — not X.charAt(i) — inside the loop.

DEBUGGING APPROACH:
  1. Verify your recurrence on a tiny example (n ≤ 4) by hand
  2. Print the full DP table and inspect values
  3. Trace the reconstruction path and verify it matches the table
  4. Compare against brute-force on small random inputs
```

---

## 4.16 Complete Benchmark — DP vs. Brute Force

```java
public class DPBenchmark {

    // Brute-force LCS: O(2^(m+n)) — try all subsequences
    public static int lcsBrute(String X, String Y, int i, int j) {
        if (i == 0 || j == 0) return 0;
        if (X.charAt(i-1) == Y.charAt(j-1))
            return 1 + lcsBrute(X, Y, i-1, j-1);
        return Math.max(lcsBrute(X, Y, i-1, j), lcsBrute(X, Y, i, j-1));
    }

    public static void main(String[] args) {
        String X = "ABCBDAB", Y = "BDCABA";

        long t1, t2;

        t1 = System.nanoTime();
        int bruteResult = lcsBrute(X, Y, X.length(), Y.length());
        t2 = System.nanoTime();
        System.out.printf("Brute-force LCS: %d  time: %d µs%n",
                          bruteResult, (t2-t1)/1000);

        t1 = System.nanoTime();
        int dpResult = lcs(X, Y);
        t2 = System.nanoTime();
        System.out.printf("DP LCS:          %d  time: %d µs%n",
                          dpResult, (t2-t1)/1000);

        // For larger inputs the difference is dramatic:
        String longX = "AGGTAB".repeat(10); // length 60
        String longY = "GXTXAYB".repeat(8); // length 56
        t1 = System.nanoTime();
        System.out.printf("DP LCS (long):   %d  time: %d µs%n",
                          lcs(longX, longY), (System.nanoTime()-t1)/1000);
        // Brute force would take astronomical time on this input
    }
}
```

---

## 4.17 Key Takeaways

| Concept                  | Summary                                                                           |
|--------------------------|-----------------------------------------------------------------------------------|
| Optimal substructure     | Optimal solution contains optimal sub-solutions — the foundation of DP           |
| Overlapping subproblems  | Same subproblems recur — memoization or tabulation avoids recomputation           |
| Memoization (top-down)   | Recursive + cache; natural code; risk of stack overflow on large inputs           |
| Tabulation (bottom-up)   | Iterative table filling; no stack overhead; easier space optimization             |
| 0/1 Knapsack             | dp[i][j]; O(nW); iterate j right-to-left in 1D; reconstruct by backtracking      |
| LCS                      | dp[i][j]; O(mn); match or take max skip; foundation of diff algorithms           |
| Edit Distance            | dp[i][j]; O(mn); three operations: insert, delete, substitute                    |
| LIS                      | O(n²) DP or O(n log n) with patience sorting + binary search                    |
| Matrix Chain             | Interval DP; O(n³); fill by chain length; track split points                    |
| Coin Change              | dp[amount]; O(amount×n); distinguish min-coins vs count-ways formulations        |
| Kadane's Algorithm       | O(n) DP for max subarray; recurrence: extend or start fresh                      |
| Interval DP trick        | Think about last operation, not first — unlocks Burst Balloons, Matrix Chain     |
| DP vs. Greedy            | DP: overlapping subproblems, explores all choices; Greedy: independent, commits  |
| Space optimization       | 1D rolling array when dp[i] only depends on dp[i-1]; significant in practice     |

---

## 4.18 Practice Problems

1. **Longest Palindromic Subsequence** — Given a string s, find the length of the longest subsequence that is a palindrome. *(Hint: this is equivalent to LCS(s, reverse(s)). Define the DP state and recurrence.)*

2. **Word Break** — Given a string s and a dictionary of words, determine if s can be segmented into space-separated dictionary words. *(Hint: dp[i] = can s[0..i-1] be segmented? dp[i] = true if dp[j] is true and s[j..i-1] is in dictionary for some j.)*

3. **Optimal BST** — Given keys with search probabilities p[i] and "dummy" probabilities q[i] for gaps between keys, find the BST that minimizes expected search cost. Formulate the DP recurrence and analyze the complexity. *(This is CLRS Section 15.5.)*

4. **Egg Drop Problem** — You have k eggs and n floors. Find the minimum number of trials needed to determine the critical floor (the highest floor from which an egg won't break) in the worst case. *(Hint: dp[t][k] = maximum number of floors testable with t trials and k eggs.)*

5. **Minimum Cost to Cut a Stick** — A stick of length n must be cut at positions given in a sorted array cuts[]. The cost of a cut is the current length of the stick being cut. Find the minimum total cost. *(Hint: interval DP — same pattern as Matrix Chain and Burst Balloons.)*

---

## 4.19 Further Reading

- *Introduction to Algorithms* (CLRS) — Chapters 14–15 (Dynamic Programming)
- *Algorithm Design* — Kleinberg & Tardos, Chapter 6 (Dynamic Programming)
- *The Algorithm Design Manual* — Skiena, Chapter 10
- Bellman, R. (1957). *Dynamic Programming*. Princeton University Press. (the original)
- LeetCode DP Study Plan: [https://leetcode.com/studyplan/dynamic-programming/](https://leetcode.com/studyplan/dynamic-programming/)

---

*CS 3304 Analysis of Algorithms — Module 4 of 8*
*Previous: Module 3 — Greedy Algorithms*
*Next: Module 5 — Backtracking Algorithms*
