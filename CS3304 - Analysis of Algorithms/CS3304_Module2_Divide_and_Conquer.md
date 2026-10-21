# CS 3304 — Analysis of Algorithms
## Module 2: Advanced Algorithm Techniques Part 1 — Divide and Conquer

---

## 2.1 Introduction

Divide and Conquer is arguably the most elegant and widely applicable algorithm design paradigm. It is the intellectual engine behind Merge Sort, Binary Search, Fast Fourier Transform, Strassen's matrix multiplication, and dozens of other foundational algorithms.

The core idea is deceptively simple:

> **Divide** the problem into smaller subproblems of the same type.
> **Conquer** each subproblem recursively.
> **Combine** the subproblem solutions into the solution to the original problem.

What makes this powerful is that many problems have a recursive structure — the solution to a large instance naturally depends on solutions to smaller instances of the same problem. Divide and conquer exploits this structure and, with the right decomposition, transforms O(n²) algorithms into O(n log n) or even O(n^1.585).

By the end of this module you will have mastered:
- The D&C template and how to apply it
- Rigorous recurrence analysis for every algorithm
- Seven classic D&C algorithms with full Java implementations
- The geometric insight behind why D&C beats brute force

---

## 2.2 The Divide and Conquer Template

Every D&C algorithm follows the same three-part structure:

```
solve(problem of size n):
    if n is small enough:
        return base_case_solution     // CONQUER directly

    subproblems = divide(problem)     // DIVIDE into a subproblems of size n/b
    solutions   = [solve(s) for s in subproblems]  // CONQUER recursively
    return combine(solutions)         // COMBINE results
```

The cost of this template gives the recurrence:

```
T(n) = a · T(n/b) + D(n) + C(n)

a    = number of subproblems
n/b  = size of each subproblem
D(n) = cost of dividing
C(n) = cost of combining
f(n) = D(n) + C(n) = total non-recursive work
```

This is exactly the Master Theorem recurrence from Module 1.

### Key Design Questions

When designing a D&C algorithm, ask:

1. **What is the base case?** (When is the problem small enough to solve directly?)
2. **How to divide?** (Into how many pieces? Of what size?)
3. **How to combine?** (What work is needed to merge subproblem solutions?)
4. **What is the recurrence?** (Apply Master Theorem to predict complexity.)

---

## 2.3 Binary Search — The Simplest D&C

Binary search is the purest D&C algorithm: divide the search space in half, conquer exactly one half, combine with zero work.

```
T(n) = T(n/2) + O(1)
     ↑ one subproblem  ↑ comparison (constant combine)

Master Theorem: a=1, b=2, f(n)=O(1)
  n^(log_2 1) = n^0 = 1 = Θ(1) → Case 2 → T(n) = Θ(log n)
```

```java
/**
 * Binary Search — D&C template made explicit.
 *
 * DIVIDE:   Split search space at midpoint
 * CONQUER:  Recurse on exactly one half
 * COMBINE:  No work — just return the result
 *
 * Time:  Θ(log n)
 * Space: Θ(log n) recursive stack — O(1) iterative
 */
public static int binarySearch(int[] arr, int target, int left, int right) {
    // BASE CASE: search space is empty
    if (left > right) return -1;

    // DIVIDE: compute midpoint
    int mid = left + (right - left) / 2;

    // CONQUER one subproblem:
    if      (arr[mid] == target) return mid;            // found
    else if (arr[mid] <  target) return binarySearch(arr, target, mid + 1, right); // right half
    else                         return binarySearch(arr, target, left,    mid - 1); // left half

    // COMBINE: nothing to do — result propagates directly
}
```

---

## 2.4 Merge Sort — The Classic D&C Sort

Merge sort is the canonical D&C sorting algorithm. Its insight: sorting two halves separately and merging is cheaper than sorting the whole.

```
DIVIDE:   Split array at midpoint             O(1)
CONQUER:  Recursively sort each half          2T(n/2)
COMBINE:  Merge two sorted halves             O(n)

T(n) = 2T(n/2) + O(n)
Master Theorem: a=2, b=2, f(n)=n
  n^(log_2 2) = n^1 → Case 2 → T(n) = Θ(n log n)
```

### Why is O(n log n) the best possible comparison sort?

The decision tree for sorting n elements has n! leaves (one per permutation). A binary tree with n! leaves needs height ≥ log₂(n!) ≈ n log n (Stirling's approximation). Therefore any comparison sort must make Ω(n log n) comparisons. Merge sort achieves this — it is **asymptotically optimal**.

```java
/**
 * Merge Sort — annotated D&C template.
 *
 * Time:  Θ(n log n) — best, average, and worst
 * Space: Θ(n) auxiliary array
 * Stable: YES
 */
public static void mergeSort(int[] arr, int left, int right) {
    // BASE CASE: single element is trivially sorted
    if (left >= right) return;

    // DIVIDE: split at midpoint
    int mid = left + (right - left) / 2;

    // CONQUER: sort each half recursively
    mergeSort(arr, left,    mid);
    mergeSort(arr, mid + 1, right);

    // COMBINE: merge the two sorted halves
    merge(arr, left, mid, right);
}

/**
 * Merge two sorted subarrays arr[left..mid] and arr[mid+1..right].
 * Uses a temporary array to avoid overwriting elements.
 *
 * LOOP INVARIANT: At each step, arr[k..right] is not yet merged,
 * and arr[left..k-1] contains the k-left smallest elements in sorted order.
 */
private static void merge(int[] arr, int left, int mid, int right) {
    int[] temp = Arrays.copyOfRange(arr, left, right + 1);
    int   i    = 0;               // pointer into left  half of temp
    int   j    = mid - left + 1;  // pointer into right half of temp
    int   k    = left;            // pointer into arr

    while (i <= mid - left && j <= right - left) {
        // Take the smaller element — <= preserves stability
        if (temp[i] <= temp[j]) arr[k++] = temp[i++];
        else                    arr[k++] = temp[j++];
    }
    while (i <= mid - left) arr[k++] = temp[i++]; // remaining left
    while (j <= right-left) arr[k++] = temp[j++]; // remaining right
}
```

### Counting Inversions — A D&C Application

An **inversion** in array A is a pair (i,j) where i < j but A[i] > A[j]. The number of inversions measures how far the array is from sorted. Modify merge sort to count inversions in O(n log n).

```java
/**
 * Count Inversions using modified Merge Sort.
 *
 * KEY INSIGHT: During merge, whenever we take an element from the RIGHT
 * half before exhausting the LEFT half, it forms inversions with ALL
 * remaining elements of the left half.
 *
 * Time: Θ(n log n)   Space: Θ(n)
 */
public static long countInversions(int[] arr, int left, int right) {
    if (left >= right) return 0;

    int  mid         = left + (right - left) / 2;
    long leftInv     = countInversions(arr, left,    mid);   // inversions in left half
    long rightInv    = countInversions(arr, mid + 1, right); // inversions in right half
    long splitInv    = mergeCount(arr, left, mid, right);    // split inversions

    return leftInv + rightInv + splitInv;
}

private static long mergeCount(int[] arr, int left, int mid, int right) {
    int[] temp  = Arrays.copyOfRange(arr, left, right + 1);
    int   i     = 0, j = mid - left + 1, k = left;
    long  count = 0;

    while (i <= mid - left && j <= right - left) {
        if (temp[i] <= temp[j]) {
            arr[k++] = temp[i++];
        } else {
            // temp[i..mid-left] are all greater than temp[j] → inversions!
            count   += (mid - left - i + 1);
            arr[k++] = temp[j++];
        }
    }
    while (i <= mid - left) arr[k++] = temp[i++];
    while (j <= right-left) arr[k++] = temp[j++];
    return count;
}

// Test:
// arr = [2, 4, 1, 3, 5]
// Inversions: (2,1),(4,1),(4,3) → 3
// countInversions(arr, 0, 4) → 3
```

---

## 2.5 Quick Sort — D&C with Partition

Quick sort divides not at the midpoint but around a **pivot** element, such that everything left of the pivot is ≤ pivot and everything right is ≥ pivot. The pivot is then in its final sorted position.

```
DIVIDE:   Partition around pivot              O(n)
CONQUER:  Recursively sort each partition     T(k) + T(n-k-1)
COMBINE:  Nothing — pivot is already placed   O(1)

Best/Average: T(n) = 2T(n/2) + O(n)  → Θ(n log n)
Worst case:   T(n) = T(n-1)  + O(n)  → Θ(n²)  (sorted input, bad pivot)
```

```java
/**
 * Quick Sort — D&C with in-place partitioning.
 *
 * Time:  Θ(n log n) average,  Θ(n²) worst
 * Space: Θ(log n) average stack depth
 * Stable: NO
 *
 * Fastest in practice for in-memory sorting because:
 *   - In-place (no auxiliary array)
 *   - Excellent cache locality (sequential scans)
 *   - Small constant factors
 */
public static void quickSort(int[] arr, int low, int high) {
    if (low < high) {
        // DIVIDE + partial CONQUER: partition and place pivot
        randomPivot(arr, low, high);           // randomize to avoid O(n²) worst case
        int pivotIdx = partition(arr, low, high);

        // CONQUER: sort the two partitions
        quickSort(arr, low,          pivotIdx - 1);
        quickSort(arr, pivotIdx + 1, high);

        // COMBINE: nothing — pivot is already in its final position
    }
}

/**
 * Lomuto Partition Scheme.
 * Pivot = last element. i tracks boundary between ≤pivot and >pivot regions.
 */
private static int partition(int[] arr, int low, int high) {
    int pivot = arr[high];
    int i     = low - 1;

    for (int j = low; j < high; j++) {
        if (arr[j] <= pivot) {
            swap(arr, ++i, j);
        }
    }
    swap(arr, i + 1, high); // place pivot between the two regions
    return i + 1;
}

private static void randomPivot(int[] arr, int low, int high) {
    int r = low + (int)(Math.random() * (high - low + 1));
    swap(arr, r, high);
}

private static void swap(int[] arr, int i, int j) {
    int t = arr[i]; arr[i] = arr[j]; arr[j] = t;
}
```

### Quick Select — Find kth Smallest in O(n)

Quick sort's partition idea can find the kth smallest element **without fully sorting**:

```java
/**
 * Quick Select — find the kth smallest element (0-indexed).
 *
 * INSIGHT: After partition, the pivot is at its final sorted index p.
 *   - If p == k → pivot IS the kth smallest
 *   - If p > k  → kth smallest is in the left partition
 *   - If p < k  → kth smallest is in the right partition
 *
 * Time:  Θ(n) average,  Θ(n²) worst
 * Space: Θ(log n) average stack
 */
public static int quickSelect(int[] arr, int low, int high, int k) {
    if (low == high) return arr[low];

    randomPivot(arr, low, high);
    int p = partition(arr, low, high);

    if      (p == k) return arr[p];             // found
    else if (p >  k) return quickSelect(arr, low,  p - 1, k); // go left
    else             return quickSelect(arr, p + 1, high,  k); // go right
}

// Example: find 3rd smallest (k=2, 0-indexed) in [3,2,1,5,6,4]
// quickSelect(arr, 0, 5, 2) → 3
```

---

## 2.6 Strassen's Matrix Multiplication

Naive matrix multiplication of two n×n matrices requires n³ multiplications — O(n³). Strassen's algorithm uses D&C to reduce this to O(n^2.807).

### Naive Algorithm

```
C[i][j] = Σ(k=0 to n-1) A[i][k] * B[k][j]

Time: Θ(n³)
```

### The D&C Insight

Divide each n×n matrix into four n/2 × n/2 submatrices:

```
A = [A11 A12]    B = [B11 B12]    C = [C11 C12]
    [A21 A22]        [B21 B22]        [C21 C22]

Standard D&C:  C11 = A11*B11 + A12*B21  (8 multiplications)
               T(n) = 8T(n/2) + O(n²)
               → n^(log_2 8) = n³ — no improvement!
```

Strassen's breakthrough: **7 multiplications** instead of 8 (by introducing clever linear combinations that trade multiplications for additions):

```
M1 = (A11 + A22)(B11 + B22)
M2 = (A21 + A22) B11
M3 = A11 (B12 − B22)
M4 = A22 (B21 − B11)
M5 = (A11 + A12) B22
M6 = (A21 − A11)(B11 + B12)
M7 = (A12 − A22)(B21 + B22)

C11 = M1 + M4 − M5 + M7
C12 = M3 + M5
C21 = M2 + M4
C22 = M1 − M2 + M3 + M6
```

```java
/**
 * Strassen's Matrix Multiplication.
 *
 * T(n) = 7T(n/2) + O(n²)
 * Master Theorem: a=7, b=2, f(n)=n²
 *   n^(log_2 7) ≈ n^2.807  >  n² → Case 1
 *   T(n) = Θ(n^log₂7) ≈ Θ(n^2.807)
 *
 * NOTE: In practice, Strassen is used only for very large matrices (n > ~1000)
 * because the large constant from extra additions makes it slower for small n.
 * For pedagogical clarity, this implementation uses 2×2 base case.
 */
public static int[][] strassen(int[][] A, int[][] B) {
    int n = A.length;

    // BASE CASE: 1×1 matrix
    if (n == 1) {
        return new int[][]{{A[0][0] * B[0][0]}};
    }

    int half = n / 2;

    // DIVIDE: extract submatrices
    int[][] A11 = subMatrix(A, 0,    0,    half);
    int[][] A12 = subMatrix(A, 0,    half, half);
    int[][] A21 = subMatrix(A, half, 0,    half);
    int[][] A22 = subMatrix(A, half, half, half);

    int[][] B11 = subMatrix(B, 0,    0,    half);
    int[][] B12 = subMatrix(B, 0,    half, half);
    int[][] B21 = subMatrix(B, half, 0,    half);
    int[][] B22 = subMatrix(B, half, half, half);

    // CONQUER: 7 recursive multiplications
    int[][] M1 = strassen(add(A11, A22), add(B11, B22));
    int[][] M2 = strassen(add(A21, A22), B11);
    int[][] M3 = strassen(A11,           sub(B12, B22));
    int[][] M4 = strassen(A22,           sub(B21, B11));
    int[][] M5 = strassen(add(A11, A12), B22);
    int[][] M6 = strassen(sub(A21, A11), add(B11, B12));
    int[][] M7 = strassen(sub(A12, A22), add(B21, B22));

    // COMBINE: compute result quadrants
    int[][] C11 = add(sub(add(M1, M4), M5), M7);
    int[][] C12 = add(M3, M5);
    int[][] C21 = add(M2, M4);
    int[][] C22 = add(sub(add(M1, M3), M2), M6);

    return assembleMatrix(C11, C12, C21, C22, n);
}

// Matrix utility methods
private static int[][] subMatrix(int[][] M, int row, int col, int size) {
    int[][] S = new int[size][size];
    for (int i = 0; i < size; i++)
        for (int j = 0; j < size; j++)
            S[i][j] = M[row + i][col + j];
    return S;
}

private static int[][] add(int[][] A, int[][] B) {
    int n = A.length;
    int[][] C = new int[n][n];
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++) C[i][j] = A[i][j] + B[i][j];
    return C;
}

private static int[][] sub(int[][] A, int[][] B) {
    int n = A.length;
    int[][] C = new int[n][n];
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++) C[i][j] = A[i][j] - B[i][j];
    return C;
}

private static int[][] assembleMatrix(int[][] C11, int[][] C12,
                                       int[][] C21, int[][] C22, int n) {
    int[][] C = new int[n][n];
    int half  = n / 2;
    for (int i = 0; i < half; i++)
        for (int j = 0; j < half; j++) {
            C[i][j]           = C11[i][j];
            C[i][j + half]    = C12[i][j];
            C[i + half][j]    = C21[i][j];
            C[i + half][j + half] = C22[i][j];
        }
    return C;
}
```

### Complexity Comparison

| Algorithm         | Multiplications | Time Complexity |
|-------------------|-----------------|-----------------|
| Naive             | n³              | Θ(n³)           |
| Naive D&C (8 sub) | n³              | Θ(n³)           |
| Strassen          | n^2.807         | Θ(n^2.807)      |
| Best known (2024) | —               | ~O(n^2.371)     |

---

## 2.7 Karatsuba Multiplication

Multiplying two n-digit integers naively requires n² single-digit multiplications — O(n²). Karatsuba's algorithm reduces this to O(n^1.585) using only **3 recursive multiplications** instead of 4.

### The D&C Setup

Split each n-digit number at the midpoint:
```
X = X_high · 10^(n/2) + X_low
Y = Y_high · 10^(n/2) + Y_low

Naive: X·Y = (X_high · Y_high)·10^n
           + (X_high · Y_low + X_low · Y_high)·10^(n/2)
           + (X_low · Y_low)
         → 4 multiplications: T(n) = 4T(n/2) + O(n) → Θ(n²)
```

Karatsuba's insight — compute the middle term with just ONE extra multiplication:
```
z0 = X_low  · Y_low
z2 = X_high · Y_high
z1 = (X_low + X_high)(Y_low + Y_high) − z0 − z2
   = X_low·Y_high + X_high·Y_low   ← the cross terms, via 1 multiplication!

X·Y = z2·10^n + z1·10^(n/2) + z0
```

```java
/**
 * Karatsuba Multiplication for large integers.
 *
 * T(n) = 3T(n/2) + O(n)
 * Master Theorem: a=3, b=2, f(n)=n
 *   n^(log_2 3) ≈ n^1.585  >  n → Case 1
 *   T(n) = Θ(n^log₂3) ≈ Θ(n^1.585)
 *
 * Uses Java BigInteger for arbitrary-precision arithmetic.
 */
import java.math.BigInteger;

public static BigInteger karatsuba(BigInteger x, BigInteger y) {
    int n = Math.max(x.bitLength(), y.bitLength());

    // BASE CASE: small enough to multiply directly
    if (n <= 64) return x.multiply(y);

    int half = (n + 1) / 2;
    BigInteger base = BigInteger.TWO.pow(half); // 2^(n/2) = the split point

    // DIVIDE: split x and y at half
    BigInteger xLow  = x.mod(base);     // lower half of x
    BigInteger xHigh = x.divide(base);  // upper half of x
    BigInteger yLow  = y.mod(base);
    BigInteger yHigh = y.divide(base);

    // CONQUER: 3 recursive multiplications (instead of 4)
    BigInteger z0 = karatsuba(xLow,             yLow);
    BigInteger z2 = karatsuba(xHigh,            yHigh);
    BigInteger z1 = karatsuba(xLow.add(xHigh),  yLow.add(yHigh))
                              .subtract(z0).subtract(z2);

    // COMBINE: x·y = z2·base² + z1·base + z0
    return z2.multiply(base.pow(2))
             .add(z1.multiply(base))
             .add(z0);
}

// Demo:
// karatsuba(123456789, 987654321) → 121932631112635269
```

---

## 2.8 Finding the Maximum Subarray — Kadane meets D&C

**Problem:** Given an array of integers (possibly negative), find the contiguous subarray with the largest sum.

```
arr = [-2, 1, -3, 4, -1, 2, 1, -5, 4]
Maximum subarray: [4, -1, 2, 1] with sum = 6
```

### D&C Approach

Any maximum subarray lies entirely in:
1. The left half
2. The right half
3. Crossing the midpoint (spans both halves)

Cases 1 and 2 are solved recursively. Case 3 can be found in O(n) by scanning outward from the midpoint.

```java
/**
 * Maximum Subarray via Divide and Conquer.
 *
 * T(n) = 2T(n/2) + O(n)  →  Θ(n log n)
 *
 * (Note: Kadane's algorithm solves this in Θ(n) — covered in Module 4.
 * The D&C version is pedagogically important for showing the paradigm.)
 */
public static int[] maxSubarrayDC(int[] arr, int left, int right) {
    // BASE CASE: single element
    if (left == right) return new int[]{arr[left], left, right};

    // DIVIDE
    int mid = left + (right - left) / 2;

    // CONQUER: find max subarray in each half
    int[] leftResult  = maxSubarrayDC(arr, left,    mid);
    int[] rightResult = maxSubarrayDC(arr, mid + 1, right);

    // COMBINE: find max subarray crossing the midpoint
    int[] crossResult = maxCrossing(arr, left, mid, right);

    // Return whichever has the largest sum
    if (leftResult[0] >= rightResult[0] && leftResult[0] >= crossResult[0])
        return leftResult;
    else if (rightResult[0] >= leftResult[0] && rightResult[0] >= crossResult[0])
        return rightResult;
    else
        return crossResult;
}

/**
 * Find maximum subarray that crosses the midpoint.
 * Scan left from mid, scan right from mid+1, take the best of each.
 * Returns {sum, leftIndex, rightIndex}.
 */
private static int[] maxCrossing(int[] arr, int left, int mid, int right) {
    int leftSum = Integer.MIN_VALUE, sum = 0, maxLeft = mid;
    for (int i = mid; i >= left; i--) {
        sum += arr[i];
        if (sum > leftSum) { leftSum = sum; maxLeft = i; }
    }

    int rightSum = Integer.MIN_VALUE; sum = 0;
    int maxRight = mid + 1;
    for (int i = mid + 1; i <= right; i++) {
        sum += arr[i];
        if (sum > rightSum) { rightSum = sum; maxRight = i; }
    }

    return new int[]{leftSum + rightSum, maxLeft, maxRight};
}

// Demo:
// arr = [-2, 1, -3, 4, -1, 2, 1, -5, 4]
// maxSubarrayDC(arr, 0, 8) → {6, 3, 6}  (sum=6, indices 3..6 = [4,-1,2,1])
```

---

## 2.9 Closest Pair of Points

**Problem:** Given n points in a 2D plane, find the pair with the smallest Euclidean distance.

Brute force: check all pairs — O(n²). D&C achieves O(n log n).

```
Strategy:
1. Sort points by x-coordinate             O(n log n)
2. Divide into left and right halves
3. Recursively find closest pair in each half
4. Check pairs that span the dividing line (the "strip")  ← key insight
5. Return the minimum of the three
```

```java
/**
 * Closest Pair of Points — Divide and Conquer.
 *
 * T(n) = 2T(n/2) + O(n log n)  →  Θ(n log²n)
 * (With a refined strip analysis: Θ(n log n))
 *
 * The critical insight: the strip between the two halves contains at most
 * O(1) = 8 candidate points within distance δ of any given point,
 * bounding the strip scan to O(n) per level.
 */
public static double closestPair(int[][] points) {
    // Sort by x-coordinate once before recursion
    Arrays.sort(points, Comparator.comparingInt(p -> p[0]));
    return closestPairRec(points, 0, points.length - 1);
}

private static double closestPairRec(int[][] pts, int left, int right) {
    // BASE CASE: brute force for ≤ 3 points
    if (right - left <= 2) return bruteForceMin(pts, left, right);

    // DIVIDE
    int   mid  = left + (right - left) / 2;
    int   midX = pts[mid][0];

    // CONQUER
    double dLeft  = closestPairRec(pts, left,    mid);
    double dRight = closestPairRec(pts, mid + 1, right);
    double delta  = Math.min(dLeft, dRight);

    // COMBINE: check strip of width 2δ around the dividing line
    List<int[]> strip = new ArrayList<>();
    for (int i = left; i <= right; i++) {
        if (Math.abs(pts[i][0] - midX) < delta) strip.add(pts[i]);
    }
    strip.sort(Comparator.comparingInt(p -> p[1])); // sort strip by y

    // For each strip point, check only the next 7 points (geometric bound)
    for (int i = 0; i < strip.size(); i++) {
        for (int j = i + 1; j < strip.size() &&
             (strip.get(j)[1] - strip.get(i)[1]) < delta; j++) {
            double d = dist(strip.get(i), strip.get(j));
            if (d < delta) delta = d;
        }
    }
    return delta;
}

private static double dist(int[] p1, int[] p2) {
    double dx = p1[0] - p2[0], dy = p1[1] - p2[1];
    return Math.sqrt(dx*dx + dy*dy);
}

private static double bruteForceMin(int[][] pts, int l, int r) {
    double min = Double.MAX_VALUE;
    for (int i = l; i <= r; i++)
        for (int j = i + 1; j <= r; j++)
            min = Math.min(min, dist(pts[i], pts[j]));
    return min;
}
```

### Why At Most 8 Points in the Strip?

The strip is 2δ wide. Any δ×δ square contains at most 4 points (since any two points in the same half are at least δ apart). The δ×2δ rectangle has at most 8 points. This bounds the inner strip loop to O(1) per point → O(n) total strip work per level.

---

## 2.10 Tower of Hanoi

A beautiful recursive D&C problem that demonstrates that the recurrence precisely captures the algorithm's cost.

**Problem:** Move n disks from peg A to peg C using peg B. Only move one disk at a time. Never place a larger disk on a smaller one.

```
DIVIDE:   Move top n-1 disks from A to B (using C)   T(n-1)
CONQUER:  Move bottom disk from A to C                O(1)
COMBINE:  Move n-1 disks from B to C (using A)        T(n-1)

T(n) = 2T(n-1) + 1
     = 2(2T(n-2) + 1) + 1
     = 4T(n-2) + 3
     = 2^k · T(n-k) + 2^k - 1
     = 2^n · T(0) + 2^n - 1
     = 2^n - 1  = O(2^n)
```

```java
/**
 * Tower of Hanoi.
 *
 * T(n) = 2T(n-1) + 1 = 2^n - 1 moves  →  Θ(2^n)
 *
 * This is OPTIMAL — you cannot solve Hanoi with fewer than 2^n - 1 moves.
 * Proof: the largest disk must move exactly once (from A to C), and before
 * it moves, all n-1 smaller disks must be on B (2^(n-1)-1 moves), and after
 * it moves, all n-1 disks must come from B to C (another 2^(n-1)-1 moves).
 * Total: 2^(n-1)-1 + 1 + 2^(n-1)-1 = 2^n - 1. ✓
 */
public static void hanoi(int n, char from, char to, char aux) {
    // BASE CASE: one disk — move it directly
    if (n == 1) {
        System.out.println("Move disk 1 from " + from + " to " + to);
        return;
    }

    // DIVIDE + CONQUER: move n-1 disks out of the way
    hanoi(n - 1, from, aux, to);  // A → B using C

    // Move the largest disk
    System.out.println("Move disk " + n + " from " + from + " to " + to);

    // COMBINE: move n-1 disks from aux to destination
    hanoi(n - 1, aux, to, from);  // B → C using A
}

// hanoi(3, 'A', 'C', 'B') produces 7 = 2³-1 moves:
// Move disk 1 from A to C
// Move disk 2 from A to B
// Move disk 1 from C to B
// Move disk 3 from A to C
// Move disk 1 from B to A
// Move disk 2 from B to C
// Move disk 1 from A to C
```

---

## 2.11 D&C Algorithm Complexity Summary

| Algorithm              | Recurrence              | Time Complexity    | Space      |
|------------------------|-------------------------|--------------------|------------|
| Binary Search          | T(n) = T(n/2) + O(1)    | Θ(log n)           | Θ(log n)   |
| Merge Sort             | T(n) = 2T(n/2) + O(n)  | Θ(n log n)         | Θ(n)       |
| Quick Sort (avg)       | T(n) = 2T(n/2) + O(n)  | Θ(n log n)         | Θ(log n)   |
| Quick Select (avg)     | T(n) = T(n/2) + O(n)   | Θ(n)               | Θ(log n)   |
| Max Subarray (D&C)     | T(n) = 2T(n/2) + O(n)  | Θ(n log n)         | Θ(log n)   |
| Strassen               | T(n) = 7T(n/2) + O(n²) | Θ(n^2.807)         | Θ(n^2.807) |
| Karatsuba              | T(n) = 3T(n/2) + O(n)  | Θ(n^1.585)         | Θ(n^1.585) |
| Closest Pair           | T(n) = 2T(n/2) + O(n log n) | Θ(n log²n)    | Θ(n)       |
| Tower of Hanoi         | T(n) = 2T(n-1) + O(1)  | Θ(2^n)             | Θ(n)       |
| Count Inversions       | T(n) = 2T(n/2) + O(n)  | Θ(n log n)         | Θ(n)       |

---

## 2.12 When Does D&C Help? Geometric Intuition

D&C wins when the **combine step is cheap** relative to the work saved by splitting. The key is the ratio between subproblem size reduction and combine cost.

```
If you split into a subproblems of size n/b with O(n^c) combine cost:

  a < b^c  → combine dominates  → no gain over brute force
  a = b^c  → balanced           → gain a log factor
  a > b^c  → recursive work dominates → still faster than O(n^c)

This is exactly the three cases of the Master Theorem.
```

### D&C Checklist

Use D&C when:
- The problem has **optimal substructure** (solution to large = function of solutions to small)
- **Subproblems are independent** (unlike DP where they overlap)
- **Combine step is efficient** — cheaper than solving the full problem
- You can reduce the problem to **fewer than n subproblems** of size n/b

---

## 2.13 Complete Worked Example — Merge Sort Analysis

```java
import java.util.Arrays;

public class MergeSortComplete {

    private static long comparisons = 0; // track actual comparisons
    private static long mergeOps    = 0; // track merge operations

    public static void mergeSortInstrumented(int[] arr, int left, int right) {
        if (left >= right) return;

        int mid = left + (right - left) / 2;
        mergeSortInstrumented(arr, left,    mid);
        mergeSortInstrumented(arr, mid + 1, right);
        mergeInstrumented(arr, left, mid, right);
    }

    private static void mergeInstrumented(int[] arr, int left, int mid, int right) {
        int[] temp = Arrays.copyOfRange(arr, left, right + 1);
        int i = 0, j = mid - left + 1, k = left;

        while (i <= mid - left && j <= right - left) {
            comparisons++;
            mergeOps++;
            if (temp[i] <= temp[j]) arr[k++] = temp[i++];
            else                    arr[k++] = temp[j++];
        }
        while (i <= mid - left) { arr[k++] = temp[i++]; mergeOps++; }
        while (j <= right-left) { arr[k++] = temp[j++]; mergeOps++; }
    }

    public static void main(String[] args) {
        int[] sizes = {100, 1000, 10000, 100000};

        System.out.printf("%-10s %-15s %-15s %-15s%n",
                          "n", "Comparisons", "n·log₂(n)", "Ratio");

        for (int n : sizes) {
            int[] arr = new int[n];
            for (int i = 0; i < n; i++) arr[i] = (int)(Math.random() * 10000);
            comparisons = 0; mergeOps = 0;

            mergeSortInstrumented(arr, 0, n - 1);

            double nlogn = n * (Math.log(n) / Math.log(2));
            System.out.printf("%-10d %-15d %-15.0f %-15.3f%n",
                              n, comparisons, nlogn, comparisons / nlogn);
        }
    }
    /* Output (approximate):
       n          Comparisons     n·log₂(n)       Ratio
       100        557             664              0.839
       1000       8706            9966             0.874
       10000      120458          132877           0.907
       100000     1568782         1660964          0.944
       → Ratio converges to ~1 from below, confirming Θ(n log n) ✓
    */
}
```

---

## 2.14 Key Takeaways

| Concept                  | Summary                                                                         |
|--------------------------|---------------------------------------------------------------------------------|
| D&C template             | Divide → Conquer recursively → Combine; gives T(n) = aT(n/b) + f(n)           |
| Master Theorem           | Solves T(n) = aT(n/b) + f(n) via comparison of f(n) and n^(log_b a)           |
| Binary Search            | 1 subproblem, half size, O(1) combine → Θ(log n)                               |
| Merge Sort               | 2 subproblems, half size, O(n) combine → Θ(n log n); optimal comparison sort  |
| Quick Sort               | Partition-based; Θ(n log n) average, Θ(n²) worst; randomize pivot             |
| Quick Select             | Find kth smallest in Θ(n) average — no need to sort fully                      |
| Strassen                 | 7 subproblems instead of 8 → breaks O(n³) to Θ(n^2.807)                       |
| Karatsuba                | 3 multiplications instead of 4 → breaks O(n²) to Θ(n^1.585)                  |
| Count Inversions         | Augment merge sort — count during merge in Θ(n log n)                          |
| Closest Pair             | 2T(n/2) + O(n log n); strip analysis bounds combine to O(n) per level         |
| Tower of Hanoi           | 2T(n-1)+1 = 2^n-1; exponential and provably optimal                           |
| D&C vs DP                | D&C: independent subproblems; DP: overlapping subproblems (Module 4)           |

---

## 2.15 Practice Problems

1. **Recurrence derivation** — Write the exact recurrence for the closest pair algorithm and solve it with the Master Theorem. Then explain why the "strip contains at most 8 points" argument is necessary.

2. **Modified Merge Sort** — Modify merge sort to sort an array in descending order. Then modify it again to sort an array of strings by length (ties broken alphabetically). Analyze the complexity of each.

3. **Median of Two Sorted Arrays** — Given two sorted arrays of sizes m and n, find the median of the combined array in O(log(m+n)) time. *(Hint: binary search on partition points — this is a classic D&C problem.)*

4. **Power function** — Implement a fast integer exponentiation function `power(base, exp)` using D&C. It should run in O(log exp) multiplications. Prove correctness using the recurrence. *(Hint: x^n = x^(n/2) · x^(n/2) if n is even; x · x^(n-1) if n is odd.)*

5. **D&C on a sorted rotated array** — A sorted array was rotated at some unknown pivot. Design a D&C algorithm to search for a target in O(log n). Prove correctness with a loop invariant. How does the recurrence differ from standard binary search?

---

## 2.16 Further Reading

- *Introduction to Algorithms* (CLRS) — Chapters 2, 4, 7 (Merge Sort, Recurrences, Quick Sort), Chapter 33 (Closest Pair)
- *Algorithm Design* — Kleinberg & Tardos, Chapter 5 (Divide and Conquer)
- *The Algorithm Design Manual* — Skiena, Chapter 4
- Strassen's original paper: "Gaussian Elimination is not Optimal" (1969)
- Karatsuba's paper: "The Complexity of Computations" (1962)

---

*CS 3304 Analysis of Algorithms — Module 2 of 8*
*Previous: Module 1 — Introduction to Algorithm Design & Analysis*
*Next: Module 3 — Advanced Algorithm Techniques Part 2: Greedy Algorithms*
