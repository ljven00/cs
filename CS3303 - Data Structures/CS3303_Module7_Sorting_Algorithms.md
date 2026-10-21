# CS 3303 — Data Structures
## Module 7: Sorting Algorithms

---

## 7.1 Introduction

Sorting is the most studied problem in computer science. Nearly every large-scale system sorts data at some point — databases sort rows, search engines rank results, operating systems schedule processes. Even binary search (Module 6) requires sorted input.

Understanding sorting algorithms means understanding a spectrum of design philosophies:

- **Comparison-based sorts** — determine order purely by comparing pairs of elements
- **Non-comparison sorts** — exploit the structure of the data itself to sort without comparisons

We will cover eight algorithms, analyze each rigorously, and build the intuition for choosing the right one in practice.

---

## 7.2 Sorting Taxonomy

```
Sorting Algorithms
│
├── Comparison-Based (lower bound: Ω(n log n))
│   ├── Simple / Quadratic
│   │   ├── Bubble Sort ───────── O(n²)
│   │   ├── Selection Sort ─────── O(n²)
│   │   └── Insertion Sort ─────── O(n²) worst, O(n) best
│   │
│   └── Efficient / Linearithmic
│       ├── Merge Sort ──────────── O(n log n) — stable, divide & conquer
│       ├── Quick Sort ──────────── O(n log n) avg — in-place, divide & conquer
│       └── Heap Sort ───────────── O(n log n) — in-place, uses heap
│
└── Non-Comparison Based
    ├── Counting Sort ────────── O(n + k) — integer keys in [0, k]
    ├── Radix Sort ──────────── O(d*(n + k)) — multi-digit integers
    └── Bucket Sort ─────────── O(n) average — uniform distribution
```

**The comparison sort lower bound:** Any algorithm that sorts purely by comparing pairs of elements must make at least Ω(n log n) comparisons. This is proven by the decision tree argument — a decision tree for n elements has n! leaves, requiring height ≥ log₂(n!) ≈ n log n. This means Merge Sort, Heap Sort, and Quick Sort are all *asymptotically optimal* comparison sorts.

---

## 7.3 Bubble Sort

Bubble sort repeatedly steps through the array, comparing adjacent elements and swapping them if they are in the wrong order. Large elements "bubble up" to the end.

```
Pass 1: [5, 3, 8, 1, 2]
        [3, 5, 8, 1, 2]  → swap 5,3
        [3, 5, 8, 1, 2]  → 5<8, no swap
        [3, 5, 1, 8, 2]  → swap 8,1
        [3, 5, 1, 2, 8]  → swap 8,2  → 8 is in its final position

Pass 2: [3, 5, 1, 2, | 8]
        [3, 1, 5, 2, | 8]  → swap 5,1
        [3, 1, 2, 5, | 8]  → swap 5,2
        ...continues
```

```java
/**
 * Bubble Sort — repeatedly bubble the largest unsorted element to the end.
 *
 * Optimization: if no swaps occur in a full pass, the array is already sorted.
 * This gives O(n) best case on already-sorted input.
 *
 * Time:  O(n²) worst/average,  O(n) best (already sorted)
 * Space: O(1) in-place
 * Stable: YES — equal elements never swap
 */
public static void bubbleSort(int[] arr) {
    int n = arr.length;
    for (int i = 0; i < n - 1; i++) {
        boolean swapped = false;

        // After each pass, the last i elements are already in place
        for (int j = 0; j < n - 1 - i; j++) {
            if (arr[j] > arr[j + 1]) {
                int tmp  = arr[j];      // swap adjacent pair
                arr[j]   = arr[j + 1];
                arr[j+1] = tmp;
                swapped  = true;
            }
        }
        if (!swapped) break; // early exit: already sorted
    }
}

// Trace: [5,3,8,1,2] → [3,5,1,2,8] → [3,1,2,5,8] → [1,2,3,5,8]
```

**When to use:** Almost never in production. Its one advantage is detecting nearly-sorted arrays early with the `swapped` optimization. Mostly taught for conceptual clarity.

---

## 7.4 Selection Sort

Selection sort divides the array into a sorted left portion and unsorted right portion. On each pass, it selects the minimum element from the unsorted portion and swaps it into place.

```
[5, 3, 8, 1, 2]
 ↑ find min in [5,3,8,1,2] = 1 at index 3 → swap with index 0
[1, | 3, 8, 5, 2]
     ↑ find min in [3,8,5,2] = 2 at index 4 → swap with index 1
[1, 2, | 8, 5, 3]
         ↑ find min in [8,5,3] = 3 at index 4 → swap with index 2
[1, 2, 3, | 5, 8]
```

```java
/**
 * Selection Sort — find minimum, swap into the sorted prefix.
 *
 * Note: always makes exactly n*(n-1)/2 comparisons regardless of input.
 * Makes at most n-1 swaps — useful when swapping is expensive.
 *
 * Time:  O(n²) always
 * Space: O(1) in-place
 * Stable: NO — swapping can move equal elements past each other
 */
public static void selectionSort(int[] arr) {
    int n = arr.length;
    for (int i = 0; i < n - 1; i++) {
        int minIdx = i;

        // Find the minimum in the unsorted portion arr[i+1..n-1]
        for (int j = i + 1; j < n; j++) {
            if (arr[j] < arr[minIdx]) minIdx = j;
        }

        // Swap minimum into position i
        int tmp      = arr[minIdx];
        arr[minIdx]  = arr[i];
        arr[i]       = tmp;
    }
}
```

**When to use:** When the cost of swapping is very high relative to comparison (e.g., large objects on disk). Otherwise, insertion sort dominates.

---

## 7.5 Insertion Sort

Insertion sort builds a sorted subarray from left to right. For each element, it "inserts" it into the correct position within the already-sorted prefix by shifting larger elements right.

```
Start: [5, 3, 8, 1, 2]

i=1: key=3 → shift 5 right → [3, 5, 8, 1, 2]
i=2: key=8 → 8>5, no shift → [3, 5, 8, 1, 2]
i=3: key=1 → shift 8,5,3 right → [1, 3, 5, 8, 2]
i=4: key=2 → shift 8,5,3 right → [1, 2, 3, 5, 8]
```

```java
/**
 * Insertion Sort — insert each element into its correct position.
 *
 * Excellent for:
 *   - Nearly sorted arrays → O(n) performance
 *   - Small arrays (n < ~20) — low constant overhead
 *   - Online sorting (process data as it arrives)
 *
 * Time:  O(n²) worst (reverse sorted),  O(n) best (already sorted)
 * Space: O(1) in-place
 * Stable: YES
 */
public static void insertionSort(int[] arr) {
    int n = arr.length;
    for (int i = 1; i < n; i++) {
        int key = arr[i]; // element to insert
        int j   = i - 1;

        // Shift elements larger than key one position to the right
        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = key; // insert key into correct position
    }
}
```

**When to use:** For small arrays (n < 20) or nearly-sorted data. Java's `Arrays.sort` for primitive types uses insertion sort as a base case within Timsort for small subarrays.

---

## 7.6 Merge Sort

Merge sort is the archetype of **divide and conquer**. It splits the array in half recursively until subarrays have one element (trivially sorted), then merges them back together in sorted order.

```
Split phase:
[38, 27, 43, 3, 9, 82, 10]
      /                \
[38, 27, 43]        [3, 9, 82, 10]
   /     \             /        \
[38]  [27,43]       [3,9]     [82,10]
       /   \         / \       /   \
     [27] [43]     [3] [9]  [82] [10]

Merge phase (bottom up):
[27,43]  [3,9]  [10,82]
[27,38,43]  [3,9,10,82]
[3,9,10,27,38,43,82]
```

```java
/**
 * Merge Sort — divide array, sort halves, merge back.
 *
 * Time:  O(n log n) — always (best, average, worst)
 * Space: O(n) — requires auxiliary array for merging
 * Stable: YES
 *
 * Preferred when:
 *   - Guaranteed O(n log n) is required
 *   - Sorting linked lists (no random access needed)
 *   - Stability is required and data doesn't fit in cache
 */
public static void mergeSort(int[] arr, int left, int right) {
    if (left >= right) return; // base case: single element

    int mid = left + (right - left) / 2;
    mergeSort(arr, left, mid);       // sort left half
    mergeSort(arr, mid + 1, right);  // sort right half
    merge(arr, left, mid, right);    // merge sorted halves
}

/**
 * Merge two sorted subarrays arr[left..mid] and arr[mid+1..right]
 * into a single sorted subarray.
 */
private static void merge(int[] arr, int left, int mid, int right) {
    // Copy both halves into a temporary array
    int[] temp = Arrays.copyOfRange(arr, left, right + 1);

    int i = 0;                 // pointer into left half  (temp)
    int j = mid - left + 1;   // pointer into right half (temp)
    int k = left;              // pointer into original array

    while (i <= mid - left && j <= right - left) {
        if (temp[i] <= temp[j]) arr[k++] = temp[i++]; // take from left
        else                    arr[k++] = temp[j++]; // take from right
    }
    while (i <= mid - left) arr[k++] = temp[i++]; // remaining left
    while (j <= right-left) arr[k++] = temp[j++]; // remaining right
}

// Usage:
int[] arr = {38, 27, 43, 3, 9, 82, 10};
mergeSort(arr, 0, arr.length - 1);
System.out.println(Arrays.toString(arr)); // [3, 9, 10, 27, 38, 43, 82]
```

### Merge Sort on Linked Lists

Merge sort is the **preferred sort for linked lists** because it requires no random access and splitting is O(1) with a pointer.

```java
/**
 * Sort a linked list using merge sort.
 * Find middle, split, sort each half, merge.
 * Time: O(n log n)   Space: O(log n) recursion stack
 */
public static ListNode sortList(ListNode head) {
    if (head == null || head.next == null) return head;

    // Find middle using slow/fast pointers
    ListNode slow = head, fast = head.next;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    ListNode mid = slow.next;
    slow.next = null;              // split the list

    ListNode left  = sortList(head);  // sort left half
    ListNode right = sortList(mid);   // sort right half
    return mergeLists(left, right);
}

private static ListNode mergeLists(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(0), curr = dummy;
    while (l1 != null && l2 != null) {
        if (l1.val <= l2.val) { curr.next = l1; l1 = l1.next; }
        else                  { curr.next = l2; l2 = l2.next; }
        curr = curr.next;
    }
    curr.next = (l1 != null) ? l1 : l2;
    return dummy.next;
}
```

---

## 7.7 Quick Sort

Quick sort is the most widely used sorting algorithm in practice. Like merge sort it uses divide and conquer, but it works by **partitioning** the array around a **pivot** element.

```
Partition around pivot 5:
[3, 6, 8, 10, 1, 2, 5]
                   ↑ pivot

After partition: [3, 1, 2, | 5 | 6, 8, 10]
                  elements    pivot  elements
                  < pivot            > pivot

Now recursively sort left and right of pivot.
Pivot is already in its FINAL position.
```

```java
/**
 * Quick Sort — partition around pivot, recurse on both sides.
 *
 * Time:  O(n log n) average,  O(n²) worst case (bad pivot choices)
 * Space: O(log n) average recursion stack,  O(n) worst case
 * Stable: NO
 *
 * Fastest in practice for in-memory sorting on random data because:
 *   - O(1) extra space (in-place)
 *   - Excellent cache behavior (sequential memory access)
 *   - Small constant factors
 */
public static void quickSort(int[] arr, int low, int high) {
    if (low < high) {
        int pivotIdx = partition(arr, low, high); // pivot is now in final position
        quickSort(arr, low, pivotIdx - 1);        // sort left of pivot
        quickSort(arr, pivotIdx + 1, high);       // sort right of pivot
    }
}

/**
 * Lomuto partition scheme.
 * Places pivot (last element) in its correct sorted position.
 * Returns the pivot's final index.
 */
private static int partition(int[] arr, int low, int high) {
    int pivot = arr[high]; // choose last element as pivot
    int i     = low - 1;  // i tracks boundary of elements ≤ pivot

    for (int j = low; j < high; j++) {
        if (arr[j] <= pivot) {
            i++;
            int tmp = arr[i]; arr[i] = arr[j]; arr[j] = tmp; // swap
        }
    }
    // Place pivot in its final position
    int tmp = arr[i+1]; arr[i+1] = arr[high]; arr[high] = tmp;
    return i + 1;
}

// Usage:
int[] arr = {10, 7, 8, 9, 1, 5};
quickSort(arr, 0, arr.length - 1);
System.out.println(Arrays.toString(arr)); // [1, 5, 7, 8, 9, 10]
```

### Pivot Selection Strategies

Poor pivot choice is Quick Sort's Achilles heel. If the pivot is always the smallest or largest element, it degrades to O(n²).

```java
/**
 * Median-of-Three pivot selection.
 * Look at first, middle, and last elements — pick the median.
 * Makes worst-case O(n²) extremely unlikely on real data.
 */
private static int medianOfThree(int[] arr, int low, int high) {
    int mid = low + (high - low) / 2;

    // Sort low, mid, high
    if (arr[low] > arr[mid])  { int t=arr[low];  arr[low]=arr[mid];  arr[mid]=t; }
    if (arr[low] > arr[high]) { int t=arr[low];  arr[low]=arr[high]; arr[high]=t;}
    if (arr[mid] > arr[high]) { int t=arr[mid];  arr[mid]=arr[high]; arr[high]=t;}

    // Median is now at mid; swap to high-1 for use as pivot
    int t=arr[mid]; arr[mid]=arr[high-1]; arr[high-1]=t;
    return arr[high - 1];
}

/**
 * Randomized pivot: swap a random element with the last element.
 * Makes O(n²) astronomically unlikely regardless of input.
 */
private static void randomPivot(int[] arr, int low, int high) {
    int r = low + (int)(Math.random() * (high - low + 1));
    int t = arr[r]; arr[r] = arr[high]; arr[high] = t;
}
```

### Three-Way Partition (Dutch National Flag)

Handles arrays with many duplicate elements efficiently — all duplicates end up in the middle partition and require no further sorting.

```java
/**
 * Three-Way Quick Sort (Dijkstra's Dutch National Flag).
 * Partitions into: [ < pivot | == pivot | > pivot ]
 * All equal elements are placed in one pass, never re-examined.
 *
 * Time: O(n log n) average,  O(n) when all elements equal
 */
public static void quickSort3Way(int[] arr, int low, int high) {
    if (low >= high) return;

    int pivot = arr[low];
    int lt = low;      // arr[low..lt-1]  < pivot
    int gt = high;     // arr[gt+1..high] > pivot
    int i  = low + 1; // arr[lt..i-1]   == pivot

    while (i <= gt) {
        if      (arr[i] < pivot) { swap(arr, lt++, i++); }
        else if (arr[i] > pivot) { swap(arr, i, gt--);   } // don't increment i
        else                     { i++;                    }
    }

    quickSort3Way(arr, low,  lt - 1); // sort elements < pivot
    quickSort3Way(arr, gt + 1, high); // sort elements > pivot
    // elements == pivot are already in place
}

private static void swap(int[] arr, int i, int j) {
    int t = arr[i]; arr[i] = arr[j]; arr[j] = t;
}
```

---

## 7.8 Heap Sort

Heap Sort combines the O(n log n) guarantee of Merge Sort with the O(1) space of Quick Sort. It builds a max-heap, then repeatedly extracts the maximum to produce sorted order.

```
Phase 1 — Build max-heap from [4, 10, 3, 5, 1]:
After heapify: [10, 5, 3, 4, 1]

Phase 2 — Extract max repeatedly:
Swap max(10) to end: [1, 5, 3, 4, | 10] → sift down → [5, 4, 3, 1, | 10]
Swap max(5)  to end: [1, 4, 3, | 5, 10] → sift down → [4, 1, 3, | 5, 10]
Swap max(4)  to end: [3, 1, | 4, 5, 10] → sift down → [3, 1, | 4, 5, 10]
Swap max(3)  to end: [1, | 3, 4, 5, 10] → done
Result: [1, 3, 4, 5, 10]
```

```java
/**
 * Heap Sort — build max-heap, repeatedly extract max to end.
 *
 * Time:  O(n log n) always — best, average, worst
 * Space: O(1) in-place — no auxiliary array
 * Stable: NO — heap operations can move equal elements
 *
 * Limitation: poor cache performance due to non-sequential memory access
 * during sift-down (jumps to 2i+1 and 2i+2).
 */
public static void heapSort(int[] arr) {
    int n = arr.length;

    // Phase 1: Build max-heap in O(n)
    // Start from last non-leaf and sift down each node
    for (int i = n / 2 - 1; i >= 0; i--) {
        siftDown(arr, n, i);
    }

    // Phase 2: Extract elements one by one
    for (int i = n - 1; i > 0; i--) {
        // Swap root (max) with last element
        int tmp = arr[0]; arr[0] = arr[i]; arr[i] = tmp;
        // Restore heap on the reduced array arr[0..i-1]
        siftDown(arr, i, 0);
    }
}

private static void siftDown(int[] arr, int n, int i) {
    int largest = i;
    int left    = 2 * i + 1;
    int right   = 2 * i + 2;

    if (left  < n && arr[left]  > arr[largest]) largest = left;
    if (right < n && arr[right] > arr[largest]) largest = right;

    if (largest != i) {
        int tmp = arr[i]; arr[i] = arr[largest]; arr[largest] = tmp;
        siftDown(arr, n, largest); // recursively sift down
    }
}
```

---

## 7.9 Counting Sort

Counting sort breaks the O(n log n) comparison barrier by exploiting the fact that keys are integers in a known range [0, k].

```
Input:  [4, 2, 2, 8, 3, 3, 1]  (values 0–8, so k=8)

Count array: index = value, value = frequency
count = [0, 1, 2, 2, 1, 0, 0, 0, 1]
         0  1  2  3  4  5  6  7  8

Cumulative: [0, 1, 3, 5, 6, 6, 6, 6, 7]
            → each value becomes the starting index in output

Output: [1, 2, 2, 3, 3, 4, 8]
```

```java
/**
 * Counting Sort — count occurrences, reconstruct sorted array.
 *
 * Time:  O(n + k) where k = range of values
 * Space: O(n + k) for count array and output
 * Stable: YES (if implemented with cumulative counts)
 *
 * Best when k = O(n) — otherwise O(k) dominates.
 * Not suitable for floating point, strings, or large value ranges.
 */
public static int[] countingSort(int[] arr, int maxVal) {
    int[] count  = new int[maxVal + 1];
    int[] output = new int[arr.length];

    // Step 1: Count occurrences
    for (int val : arr) count[val]++;

    // Step 2: Cumulative counts — count[i] = # elements ≤ i
    for (int i = 1; i <= maxVal; i++) {
        count[i] += count[i - 1];
    }

    // Step 3: Build output array (traverse input right-to-left for stability)
    for (int i = arr.length - 1; i >= 0; i--) {
        output[--count[arr[i]]] = arr[i];
    }
    return output;
}

// Example:
int[] arr = {4, 2, 2, 8, 3, 3, 1};
int[] sorted = countingSort(arr, 8);
System.out.println(Arrays.toString(sorted)); // [1, 2, 2, 3, 3, 4, 8]
```

---

## 7.10 Radix Sort

Radix sort handles integers with multiple digits by sorting digit by digit from least significant to most significant (LSD), using a stable sort (typically counting sort) at each digit position.

```
Input: [170, 45, 75, 90, 802, 24, 2, 66]

Sort by units digit:
[170, 90, 802, 2, 24, 45, 75, 66]

Sort by tens digit:
[802, 2, 24, 45, 66, 170, 75, 90]

Sort by hundreds digit:
[2, 24, 45, 66, 75, 90, 170, 802]  ← sorted!
```

```java
/**
 * Radix Sort (LSD — Least Significant Digit first).
 * Uses counting sort as the stable subroutine for each digit.
 *
 * Time:  O(d * (n + b)) where d = number of digits, b = base (10)
 *        For fixed-width integers: O(n)
 * Space: O(n + b)
 * Stable: YES
 */
public static void radixSort(int[] arr) {
    int max = Arrays.stream(arr).max().getAsInt(); // find max to determine digit count

    // Sort by each digit position: 1s, 10s, 100s, ...
    for (int exp = 1; max / exp > 0; exp *= 10) {
        countingSortByDigit(arr, exp);
    }
}

private static void countingSortByDigit(int[] arr, int exp) {
    int n       = arr.length;
    int[] output = new int[n];
    int[] count  = new int[10]; // digits 0–9

    // Count occurrences of each digit at position exp
    for (int val : arr) count[(val / exp) % 10]++;

    // Cumulative count
    for (int i = 1; i < 10; i++) count[i] += count[i - 1];

    // Build output (right-to-left for stability)
    for (int i = n - 1; i >= 0; i--) {
        int digit = (arr[i] / exp) % 10;
        output[--count[digit]] = arr[i];
    }

    System.arraycopy(output, 0, arr, 0, n);
}

// Example:
int[] arr = {170, 45, 75, 90, 802, 24, 2, 66};
radixSort(arr);
System.out.println(Arrays.toString(arr)); // [2, 24, 45, 66, 75, 90, 170, 802]
```

---

## 7.11 Bucket Sort

Bucket sort distributes elements into a number of buckets, sorts each bucket individually, then concatenates them. It achieves O(n) average time when input is uniformly distributed.

```
Input (floats in [0, 1)): [0.78, 0.17, 0.39, 0.26, 0.72, 0.94, 0.21, 0.12]
n = 8 buckets

Bucket 0 [0.0–0.1): []
Bucket 1 [0.1–0.2): [0.17, 0.12]
Bucket 2 [0.2–0.3): [0.26, 0.21]
Bucket 3 [0.3–0.4): [0.39]
...
Bucket 7 [0.7–0.8): [0.78, 0.72]
Bucket 9 [0.9–1.0): [0.94]

Sort each bucket (insertion sort), concatenate:
[0.12, 0.17, 0.21, 0.26, 0.39, 0.72, 0.78, 0.94]
```

```java
/**
 * Bucket Sort for floating-point values in [0, 1).
 *
 * Time:  O(n) average (uniform distribution),  O(n²) worst case
 * Space: O(n)
 * Stable: YES (if bucket sort is stable)
 */
public static double[] bucketSort(double[] arr) {
    int n = arr.length;
    List<List<Double>> buckets = new ArrayList<>();
    for (int i = 0; i < n; i++) buckets.add(new ArrayList<>());

    // Distribute into buckets
    for (double val : arr) {
        int bucketIdx = (int)(val * n); // map value to bucket index
        buckets.get(bucketIdx).add(val);
    }

    // Sort each bucket and concatenate
    int k = 0;
    for (List<Double> bucket : buckets) {
        Collections.sort(bucket);       // insertion sort for small buckets
        for (double val : bucket) arr[k++] = val;
    }
    return arr;
}
```

---

## 7.12 Complete Comparison

| Algorithm       | Best       | Average    | Worst      | Space   | Stable | Notes                              |
|-----------------|------------|------------|------------|---------|--------|------------------------------------|
| Bubble Sort     | O(n)       | O(n²)      | O(n²)      | O(1)    | Yes    | Only useful as a teaching tool     |
| Selection Sort  | O(n²)      | O(n²)      | O(n²)      | O(1)    | No     | Min swaps (n-1); bad otherwise     |
| Insertion Sort  | O(n)       | O(n²)      | O(n²)      | O(1)    | Yes    | Best for small/nearly-sorted data  |
| Merge Sort      | O(n log n) | O(n log n) | O(n log n) | O(n)    | Yes    | Preferred for linked lists         |
| Quick Sort      | O(n log n) | O(n log n) | O(n²)      | O(log n)| No     | Fastest in practice (arrays)       |
| Heap Sort       | O(n log n) | O(n log n) | O(n log n) | O(1)    | No     | Guaranteed O(n log n), poor cache  |
| Counting Sort   | O(n + k)   | O(n + k)   | O(n + k)   | O(k)    | Yes    | Integers only, small range         |
| Radix Sort      | O(d·n)     | O(d·n)     | O(d·n)     | O(n+b)  | Yes    | Fixed-width integers; beats O(n log n)|
| Bucket Sort     | O(n)       | O(n)       | O(n²)      | O(n)    | Yes    | Uniform float distribution         |

---

## 7.13 Java's Built-in Sorting

Java uses different algorithms depending on what you're sorting:

```java
import java.util.Arrays;
import java.util.Collections;

// Primitive arrays → Dual-Pivot Quicksort (Java 7+)
// Average O(n log n), better constants than classical quicksort
int[]    ints   = {5, 2, 8, 1, 9};
double[] doubles = {3.14, 1.41, 2.71};
Arrays.sort(ints);    // [1, 2, 5, 8, 9]
Arrays.sort(doubles); // [1.41, 2.71, 3.14]

// Object arrays → Timsort (hybrid merge + insertion sort)
// Stable, O(n log n) worst, O(n) on nearly-sorted data
String[]   words = {"banana", "apple", "cherry"};
Integer[]  nums  = {5, 2, 8, 1, 9};
Arrays.sort(words); // [apple, banana, cherry]
Arrays.sort(nums);  // [1, 2, 5, 8, 9]

// Sort a range
Arrays.sort(ints, 1, 4); // sort only indices 1–3

// Collections.sort → also Timsort
List<Integer> list = Arrays.asList(5, 2, 8, 1, 9);
Collections.sort(list);            // ascending
list.sort(Collections.reverseOrder()); // descending

// Custom comparator — sort by string length, then alphabetically
List<String> strs = Arrays.asList("banana","fig","apple","kiwi");
strs.sort(Comparator.comparingInt(String::length)
                    .thenComparing(Comparator.naturalOrder()));
System.out.println(strs); // [fig, kiwi, apple, banana]
```

### Timsort — The Real-World Champion

Java's `Arrays.sort` for objects uses **Timsort**, a hybrid algorithm developed by Tim Peters (originally for Python). Key properties:

- Detects naturally sorted **runs** in the data
- Uses insertion sort for small runs (< 64 elements)
- Merges runs using merge sort logic
- Achieves O(n) on nearly-sorted data, O(n log n) worst case
- Stable — preserves relative order of equal elements

---

## 7.14 Choosing the Right Sorting Algorithm

```
Is the data nearly sorted?
  → YES: Insertion Sort (O(n)), Timsort
  → NO ↓

Is extra memory available?
  → NO:  Quick Sort (fastest in practice), Heap Sort (guaranteed O(n log n))
  → YES ↓

Are keys small integers in a bounded range?
  → YES: Counting Sort or Radix Sort (beats O(n log n))
  → NO ↓

Is stability required?
  → YES: Merge Sort, Timsort
  → NO:  Quick Sort (fastest), Heap Sort (guaranteed)

Are you sorting a linked list?
  → YES: Merge Sort (no random access needed)
  → NO:  Quick Sort or Timsort for arrays
```

---

## 7.15 Complete Demo — All Algorithms

```java
import java.util.Arrays;

public class SortingDemo {

    public static void main(String[] args) {
        int[][] inputs = {
            {64, 34, 25, 12, 22, 11, 90},
            {64, 34, 25, 12, 22, 11, 90},
            {64, 34, 25, 12, 22, 11, 90},
            {64, 34, 25, 12, 22, 11, 90},
            {64, 34, 25, 12, 22, 11, 90},
            {64, 34, 25, 12, 22, 11, 90}
        };

        bubbleSort(inputs[0]);
        System.out.println("Bubble:    " + Arrays.toString(inputs[0]));

        selectionSort(inputs[1]);
        System.out.println("Selection: " + Arrays.toString(inputs[1]));

        insertionSort(inputs[2]);
        System.out.println("Insertion: " + Arrays.toString(inputs[2]));

        mergeSort(inputs[3], 0, inputs[3].length - 1);
        System.out.println("Merge:     " + Arrays.toString(inputs[3]));

        quickSort(inputs[4], 0, inputs[4].length - 1);
        System.out.println("Quick:     " + Arrays.toString(inputs[4]));

        heapSort(inputs[5]);
        System.out.println("Heap:      " + Arrays.toString(inputs[5]));

        // All output: [11, 12, 22, 25, 34, 64, 90]
    }
}
```

---

## 7.16 Key Takeaways

| Concept                 | Summary                                                                         |
|-------------------------|---------------------------------------------------------------------------------|
| Comparison sort bound   | Any comparison-based sort needs Ω(n log n) comparisons                         |
| Bubble Sort             | O(n²); simple; early exit on sorted input; rarely used in practice             |
| Selection Sort          | O(n²); minimum number of swaps; no early exit                                  |
| Insertion Sort          | O(n²) worst, O(n) best; ideal for small/nearly-sorted arrays                   |
| Merge Sort              | O(n log n) always; stable; O(n) space; best for linked lists                   |
| Quick Sort              | O(n log n) avg, O(n²) worst; fastest in practice; pivot choice critical        |
| Heap Sort               | O(n log n) always; O(1) space; poor cache behavior                              |
| Counting / Radix        | Break O(n log n) barrier for integer keys; O(n) with appropriate constraints   |
| Timsort                 | Java's default for objects; hybrid merge+insertion; O(n) on nearly-sorted      |
| Stable sort             | Preserves relative order of equal elements — critical for multi-key sorting    |

---

## 7.17 Practice Problems

1. **Sort Colors (Dutch National Flag)** — Given an array containing only 0s, 1s, and 2s, sort it in O(n) time and O(1) space without using counting sort. *(Hint: three-way partition with pivot=1.)*

2. **Kth Largest Element** — Find the kth largest element in an unsorted array in O(n) average time. *(Hint: use Quick Select — a variant of Quick Sort's partition that only recurses on one side.)*

3. **Merge K Sorted Arrays** — Given k sorted arrays, merge them into one sorted array. *(Hint: use a min-heap of size k — O(n log k).)*

4. **Count Inversions** — An inversion is a pair (i, j) where i < j but arr[i] > arr[j]. Count all inversions in O(n log n). *(Hint: modify Merge Sort — count inversions during the merge step.)*

5. **External Sort** — You have 10 GB of integers on disk but only 1 GB of RAM. Describe (in pseudocode or prose) how you would sort this data. *(Hint: external merge sort — sort chunks that fit in memory, then merge them.)*

---

## 7.18 Further Reading

- *Introduction to Algorithms* (CLRS) — Chapters 6–8 (Heapsort, Quicksort, Sorting in Linear Time)
- *Algorithms* — Sedgewick & Wayne, Chapter 2 (Sorting)
- Sorting visualizer: [https://visualgo.net/en/sorting](https://visualgo.net/en/sorting)
- Timsort paper: [https://bugs.python.org/file4451/timsort.txt](https://bugs.python.org/file4451/timsort.txt)
- Java dual-pivot quicksort: [https://docs.oracle.com/en/java/se/17/docs/api/java.base/java/util/Arrays.html](https://docs.oracle.com/en/java/se/17/docs/api/java.base/java/util/Arrays.html)

---

*CS 3303 Data Structures — Module 7 of 8*
*Previous: Module 6 — Searching and Pattern Matching Techniques*
*Next: Module 8 — Exploring Graphs and Advanced Graph Algorithms*
