# CS 3303 — Data Structures
## Module 6: Searching and Pattern Matching Techniques

---

## 6.1 Introduction

Searching is one of the most fundamental operations in computing. Every time you use Ctrl+F in a document, query a database, or type in a search bar, a searching algorithm is at work. This module covers two broad categories:

**Searching in collections** — finding a value in an array, list, or tree structure.
**Pattern matching in strings** — finding where a pattern (substring) appears within a larger text.

These two domains share a common theme: the naïve approach works, but clever algorithms exploit structure to achieve dramatically better performance. By the end of this module, you will understand not just *how* each algorithm works, but *why* it is faster and *when* to choose it.

---

## 6.2 Searching in Arrays — A Taxonomy

```
Searching Algorithms
│
├── Unsorted Data
│   └── Linear Search ──────────────── O(n)
│
└── Sorted Data
    ├── Binary Search ───────────────── O(log n)
    ├── Interpolation Search ─────────── O(log log n) average
    └── Exponential Search ───────────── O(log n)
```

---

## 6.3 Linear Search

The simplest search: scan every element until the target is found or the array is exhausted.

```java
/**
 * Linear Search — works on any array, sorted or unsorted.
 *
 * Strategy: check each element one by one.
 *
 * Time:  O(n) worst/average,  O(1) best (first element)
 * Space: O(1)
 */
public static int linearSearch(int[] arr, int target) {
    for (int i = 0; i < arr.length; i++) {
        if (arr[i] == target) return i;   // found at index i
    }
    return -1;                            // not found
}
```

### When to use Linear Search
- Array is **unsorted** and sorting it first would be more expensive than a single search
- Array is **very small** (n < ~20) — constant overhead makes binary search unnecessary
- You are searching a **linked list** (no random access, so binary search is impossible)

---

## 6.4 Binary Search

Binary search is one of the most elegant algorithms in computer science. It works on **sorted** arrays and eliminates half the search space at every step.

```
Target = 7
Array: [1, 3, 4, 6, 7, 8, 10, 13, 14]
        0  1  2  3  4  5   6   7   8

Step 1: left=0, right=8, mid=4  → arr[4]=7 → FOUND at index 4
```

```
Target = 13
Array: [1, 3, 4, 6, 7, 8, 10, 13, 14]

Step 1: left=0, right=8, mid=4  → arr[4]=7  < 13 → go right: left=5
Step 2: left=5, right=8, mid=6  → arr[6]=10 < 13 → go right: left=7
Step 3: left=7, right=8, mid=7  → arr[7]=13 = 13 → FOUND at index 7
```

### Iterative Binary Search

```java
/**
 * Binary Search — requires a sorted array.
 *
 * At each step, compare target with the middle element:
 *   - Equal    → return mid
 *   - Target < mid → search left half
 *   - Target > mid → search right half
 *
 * Time:  O(log n) — halves the space each step
 * Space: O(1)
 *
 * NOTE: mid = left + (right - left) / 2  avoids integer overflow
 *       compared to (left + right) / 2
 */
public static int binarySearch(int[] arr, int target) {
    int left = 0, right = arr.length - 1;

    while (left <= right) {
        int mid = left + (right - left) / 2;

        if      (arr[mid] == target) return mid;        // found
        else if (arr[mid] <  target) left  = mid + 1;   // target in right half
        else                         right = mid - 1;   // target in left half
    }
    return -1; // not found
}
```

### Recursive Binary Search

```java
/**
 * Recursive binary search.
 * Identical logic, but expressed as a recursive function.
 * Time: O(log n)   Space: O(log n) — recursion stack
 */
public static int binarySearchRec(int[] arr, int target, int left, int right) {
    if (left > right) return -1;             // base case: not found

    int mid = left + (right - left) / 2;

    if      (arr[mid] == target) return mid;
    else if (arr[mid] <  target) return binarySearchRec(arr, target, mid + 1, right);
    else                         return binarySearchRec(arr, target, left, mid - 1);
}
```

### Binary Search Variants

Binary search is a template — many problems require finding not just equality but **boundaries**.

```java
/**
 * Find the FIRST position where arr[i] >= target (lower bound).
 * Useful for: first occurrence of a duplicate, insertion point.
 *
 * Example: arr=[1,3,3,3,5], target=3 → returns index 1 (first 3)
 */
public static int lowerBound(int[] arr, int target) {
    int left = 0, right = arr.length;       // right = length (one past end)

    while (left < right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] < target) left  = mid + 1;
        else                   right = mid;     // keep mid as a candidate
    }
    return left; // insertion point
}

/**
 * Find the LAST position where arr[i] <= target (upper bound).
 * Useful for: last occurrence of a duplicate.
 *
 * Example: arr=[1,3,3,3,5], target=3 → returns index 3 (last 3)
 */
public static int upperBound(int[] arr, int target) {
    int left = 0, right = arr.length;

    while (left < right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] <= target) left  = mid + 1;
        else                    right = mid;
    }
    return left - 1;
}

/**
 * Count occurrences of target in a sorted array.
 * Uses both bounds: O(log n)
 */
public static int countOccurrences(int[] arr, int target) {
    int first = lowerBound(arr, target);
    int last  = upperBound(arr, target);
    if (first > last || (first < arr.length && arr[first] != target)) return 0;
    return last - first + 1;
}
```

---

## 6.5 Interpolation Search

Binary search always checks the **midpoint**. Interpolation search is smarter — it estimates where the target likely is based on its value relative to the range, like how you'd open a dictionary closer to "Z" when looking for "zebra."

```
position = left + ((target - arr[left]) / (arr[right] - arr[left])) * (right - left)
```

```java
/**
 * Interpolation Search — best for uniformly distributed sorted data.
 *
 * Time:  O(log log n) average on uniform data
 *        O(n) worst case (highly skewed data)
 * Space: O(1)
 */
public static int interpolationSearch(int[] arr, int target) {
    int left = 0, right = arr.length - 1;

    while (left <= right && target >= arr[left] && target <= arr[right]) {
        if (left == right) {
            return arr[left] == target ? left : -1;
        }

        // Estimate position using linear interpolation
        int pos = left + ((target - arr[left]) * (right - left))
                        / (arr[right] - arr[left]);

        if      (arr[pos] == target) return pos;
        else if (arr[pos] <  target) left  = pos + 1;
        else                         right = pos - 1;
    }
    return -1;
}
```

---

## 6.6 Exponential Search

Exponential search is useful when the array is **unbounded** (you don't know the size) or when the target is likely near the beginning.

```java
/**
 * Exponential Search:
 * 1. Find a range [2^i, 2^(i+1)] where target might lie, by doubling i
 * 2. Run binary search within that range
 *
 * Time:  O(log n)
 * Space: O(1)
 */
public static int exponentialSearch(int[] arr, int target) {
    if (arr[0] == target) return 0;

    // Double the index until we exceed target or array bounds
    int i = 1;
    while (i < arr.length && arr[i] <= target) i *= 2;

    // Binary search in the found range
    int left  = i / 2;
    int right = Math.min(i, arr.length - 1);
    return binarySearchRec(arr, target, left, right);
}
```

---

## 6.7 Searching in Other Structures

### Searching in a Hash Map — O(1) Average

```java
import java.util.HashMap;

/**
 * Hash map lookup: the fastest practical search.
 * Uses hashing to map keys directly to buckets.
 *
 * Time: O(1) average,  O(n) worst case (all keys collide)
 */
HashMap<String, Integer> phoneBook = new HashMap<>();
phoneBook.put("Alice", 5551234);
phoneBook.put("Bob",   5555678);

int number = phoneBook.get("Alice"); // O(1)
boolean exists = phoneBook.containsKey("Carol"); // false
```

### Searching in a BST — O(log n) Average

```java
/**
 * BST search — navigate left or right based on comparison.
 * O(log n) on balanced tree, O(n) on degenerate tree.
 */
public boolean searchBST(TreeNode node, int target) {
    if (node == null)           return false;
    if (node.data == target)    return true;
    if (target < node.data)     return searchBST(node.left, target);
    else                        return searchBST(node.right, target);
}
```

### Structure Search Comparison

| Structure         | Search Time     | Requirement                   |
|-------------------|-----------------|-------------------------------|
| Unsorted array    | O(n)            | None                          |
| Sorted array      | O(log n)        | Must be sorted                |
| Linked list       | O(n)            | None (no random access)       |
| Hash table        | O(1) average    | Hashable keys                 |
| BST (balanced)    | O(log n)        | Ordered, balanced             |
| Trie              | O(m)            | String keys (m = key length)  |

---

## 6.8 String Pattern Matching — Introduction

Pattern matching answers the question: **does pattern P appear in text T, and if so, where?**

```
Text:    "the cat sat on the mat"
Pattern: "cat"

Match found at index 4
```

Naïve approach: try every position. For text length n and pattern length m, this is O(n·m). We can do much better.

```
Algorithms covered:
│
├── Brute Force ──────────────── O(n·m)
├── Knuth-Morris-Pratt (KMP) ─── O(n + m)
├── Boyer-Moore ──────────────── O(n/m) best case
└── Rabin-Karp ───────────────── O(n + m) average
```

---

## 6.9 Brute Force String Search

The baseline: slide the pattern across the text one position at a time, checking for a match at each position.

```java
/**
 * Brute Force Pattern Matching.
 *
 * For each position i in text, check if pattern matches starting at i.
 * On mismatch, restart pattern from beginning and advance text by 1.
 *
 * Time:  O(n * m) worst case (e.g. text="aaaa", pattern="aaab")
 * Space: O(1)
 */
public static List<Integer> bruteForce(String text, String pattern) {
    List<Integer> matches = new ArrayList<>();
    int n = text.length(), m = pattern.length();

    for (int i = 0; i <= n - m; i++) {         // try every start position
        int j = 0;
        while (j < m && text.charAt(i + j) == pattern.charAt(j)) {
            j++;                                // characters match, advance
        }
        if (j == m) matches.add(i);            // full pattern matched
    }
    return matches;
}

// Example:
// text="AABAACAADAABAABA", pattern="AABA"
// Matches at: [0, 9, 12]
```

### Why It's Slow

```
Text:    A A A A A A A B
Pattern: A A A A B

Step 1: A A A A [B≠A] → mismatch at position 4, restart
Step 2:   A A A A [B≠A] → mismatch at position 4, restart
Step 3:     A A A A [B≠A] → ...

We wasted all the work we did matching "AAAA" each time.
```

KMP solves this by never re-examining text characters we've already matched.

---

## 6.10 Knuth-Morris-Pratt (KMP) Algorithm

KMP is one of the most celebrated string algorithms. Its key insight:

> When a mismatch occurs, we already know part of the text we just scanned. We can use that knowledge to skip ahead in the pattern — without moving backward in the text.

### The Failure Function (LPS Array)

Before searching, KMP preprocesses the pattern to build a **Longest Proper Prefix which is also Suffix (LPS)** array.

```
Pattern: A A B A A C A A B
Index:   0 1 2 3 4 5 6 7 8

LPS[i] = length of the longest proper prefix of pattern[0..i]
         that is also a suffix of pattern[0..i]

LPS:     0 1 0 1 2 0 1 2 3
```

How to read it: LPS[4] = 2 means that in `"AABAA"`, the prefix `"AA"` equals the suffix `"AA"` — length 2. When a mismatch happens after matching 5 characters, we can restart the pattern comparison from position LPS[4] = 2 instead of 0.

```java
/**
 * Build the LPS (Longest Proper Prefix-Suffix) array.
 * This is the preprocessing step of KMP.
 * Time: O(m)   Space: O(m)
 */
private static int[] buildLPS(String pattern) {
    int m = pattern.length();
    int[] lps = new int[m];
    lps[0] = 0;           // first entry is always 0

    int len = 0;          // length of previous longest prefix-suffix
    int i = 1;

    while (i < m) {
        if (pattern.charAt(i) == pattern.charAt(len)) {
            len++;
            lps[i] = len;
            i++;
        } else {
            if (len != 0) {
                len = lps[len - 1]; // fall back (don't increment i)
            } else {
                lps[i] = 0;
                i++;
            }
        }
    }
    return lps;
}
```

### KMP Search

```java
/**
 * KMP Pattern Matching.
 *
 * Two pointers — i for text, j for pattern.
 * On mismatch: don't move i back; instead, use LPS to jump j.
 * Text characters are NEVER re-examined.
 *
 * Time:  O(n + m)  — O(m) preprocess + O(n) search
 * Space: O(m) for the LPS array
 */
public static List<Integer> kmpSearch(String text, String pattern) {
    List<Integer> matches = new ArrayList<>();
    int n = text.length(), m = pattern.length();
    int[] lps = buildLPS(pattern);

    int i = 0; // index into text
    int j = 0; // index into pattern

    while (i < n) {
        if (text.charAt(i) == pattern.charAt(j)) {
            i++; j++;                               // characters match
        }

        if (j == m) {
            matches.add(i - j);                     // full match found at i-j
            j = lps[j - 1];                         // look for next match using LPS
        } else if (i < n && text.charAt(i) != pattern.charAt(j)) {
            if (j != 0) j = lps[j - 1];            // use LPS to skip
            else        i++;                         // no prefix to fall back on
        }
    }
    return matches;
}
```

### KMP Step-by-Step Trace

```
Text:    A A B A A C A A B A A B
Pattern: A A B
LPS:     [0, 1, 0]

i=0,j=0: T[0]=A = P[0]=A → i=1,j=1
i=1,j=1: T[1]=A = P[1]=A → i=2,j=2
i=2,j=2: T[2]=B = P[2]=B → i=3,j=3  → j==m → MATCH at 0, j=lps[2]=0
i=3,j=0: T[3]=A = P[0]=A → i=4,j=1
i=4,j=1: T[4]=A = P[1]=A → i=5,j=2
i=5,j=2: T[5]=C ≠ P[2]=B → j=lps[1]=1
i=5,j=1: T[5]=C ≠ P[1]=A → j=lps[0]=0
i=5,j=0: T[5]=C ≠ P[0]=A → i=6
...continues, finding matches at 0, 6, 9
```

Notice: `i` never decreases. Every text character is visited at most once.

---

## 6.11 Boyer-Moore Algorithm

Boyer-Moore is arguably the fastest string search algorithm in practice — it can skip large portions of the text entirely. It scans the pattern **right to left** and uses two heuristics to decide how far to jump.

### Bad Character Heuristic

When a mismatch occurs at text position `i` with pattern position `j`:
- Find the rightmost occurrence of `text[i]` in the pattern to the left of `j`
- Shift the pattern so that character aligns with the mismatch in the text

```java
/**
 * Build the Bad Character table.
 * For each character, store its rightmost position in the pattern.
 * Any character not in the pattern gets -1.
 */
private static int[] buildBadChar(String pattern) {
    int[] table = new int[256]; // ASCII characters
    Arrays.fill(table, -1);
    for (int i = 0; i < pattern.length(); i++) {
        table[pattern.charAt(i)] = i;  // rightmost position of each char
    }
    return table;
}
```

### Boyer-Moore Search (Bad Character Only)

```java
/**
 * Boyer-Moore with Bad Character heuristic.
 *
 * Match pattern against text right-to-left.
 * On mismatch, shift pattern so bad character aligns (or skip past it).
 *
 * Time:  O(n/m) best case — can skip entire pattern widths
 *        O(n*m) worst case (bad character alone, e.g. all same chars)
 * Space: O(alphabet_size) for bad character table
 *
 * Full Boyer-Moore with Good Suffix heuristic achieves O(n) worst case.
 */
public static List<Integer> boyerMoore(String text, String pattern) {
    List<Integer> matches = new ArrayList<>();
    int n = text.length(), m = pattern.length();
    int[] badChar = buildBadChar(pattern);

    int shift = 0; // how far the pattern is shifted relative to text

    while (shift <= n - m) {
        int j = m - 1; // start matching from RIGHT end of pattern

        // Move left while characters match
        while (j >= 0 && pattern.charAt(j) == text.charAt(shift + j)) {
            j--;
        }

        if (j < 0) {
            matches.add(shift);                         // full match
            // Shift so next character in text aligns with last occurrence in pattern
            shift += (shift + m < n) ? m - badChar[text.charAt(shift + m)] : 1;
        } else {
            // Shift pattern so bad character in text aligns with its
            // rightmost occurrence in pattern (at least shift by 1)
            int badShift = j - badChar[text.charAt(shift + j)];
            shift += Math.max(1, badShift);
        }
    }
    return matches;
}
```

### Boyer-Moore Intuition

```
Text:    T E S T I N G   P A T T E R N S
Pattern: T E R N

Align at shift=0:
  Text:    T E S T I N G ...
  Pattern: T E R N
           ↑ ↑ ↑ ↑
  Match right-to-left: N≠T (text[3]), bad char T is at pattern[0]
  Shift = 3 - 0 = 3

Align at shift=3:
  Text:    ... T I N G ...
  Pattern:     T E R N
  ...continues, skipping large sections
```

---

## 6.12 Rabin-Karp Algorithm

Rabin-Karp uses **hashing**: compute the hash of the pattern, then slide a window across the text, recomputing the hash at each position. A hash match triggers a full character-by-character verification.

The key innovation is **rolling hash** — updating the window hash in O(1) instead of O(m).

### Rolling Hash Formula

```
hash(s) = s[0]*b^(m-1) + s[1]*b^(m-2) + ... + s[m-1]*b^0   (mod prime)

Rolling: remove leftmost char, add new rightmost char:
newHash = (oldHash - s[0]*b^(m-1)) * b + s[m]   (mod prime)
```

```java
/**
 * Rabin-Karp Pattern Matching.
 *
 * Time:  O(n + m) average (rare hash collisions)
 *        O(n * m) worst case (all hashes collide — pathological)
 * Space: O(1)
 *
 * Particularly powerful for:
 *   - Multiple pattern search (compute all pattern hashes once)
 *   - 2D pattern matching
 */
public static List<Integer> rabinKarp(String text, String pattern) {
    List<Integer> matches = new ArrayList<>();
    int n = text.length(), m = pattern.length();
    if (m > n) return matches;

    final int BASE  = 31;
    final int MOD   = 1_000_000_007; // large prime to reduce collisions

    // Precompute BASE^(m-1) mod MOD
    long highPow = 1;
    for (int i = 0; i < m - 1; i++) highPow = (highPow * BASE) % MOD;

    // Compute hash of pattern and first window of text
    long patternHash = 0, windowHash = 0;
    for (int i = 0; i < m; i++) {
        patternHash = (patternHash * BASE + pattern.charAt(i)) % MOD;
        windowHash  = (windowHash  * BASE + text.charAt(i))    % MOD;
    }

    // Slide the window
    for (int i = 0; i <= n - m; i++) {
        if (windowHash == patternHash) {
            // Hash match — verify character by character (avoid false positives)
            if (text.substring(i, i + m).equals(pattern)) {
                matches.add(i);
            }
        }

        // Roll the hash forward (remove leftmost char, add new rightmost char)
        if (i < n - m) {
            windowHash = (windowHash - text.charAt(i) * highPow % MOD + MOD) % MOD;
            windowHash = (windowHash * BASE + text.charAt(i + m)) % MOD;
        }
    }
    return matches;
}
```

### Multi-Pattern Search with Rabin-Karp

```java
/**
 * Search for MULTIPLE patterns simultaneously in one pass.
 * Compute all pattern hashes once, then check each window against all of them.
 *
 * Time: O(n + k*m) where k = number of patterns
 */
public static Map<String, List<Integer>> multiPatternSearch(
        String text, String[] patterns) {

    Map<String, List<Integer>> results = new HashMap<>();
    Map<Long, String> hashToPattern = new HashMap<>();

    final int BASE = 31, MOD = 1_000_000_007;
    int m = patterns[0].length(); // assume all same length for simplicity

    // Hash each pattern
    for (String p : patterns) {
        long h = 0;
        for (char c : p.toCharArray()) h = (h * BASE + c) % MOD;
        hashToPattern.put(h, p);
        results.put(p, new ArrayList<>());
    }

    // Slide window and check
    long windowHash = 0;
    long highPow = 1;
    for (int i = 0; i < m - 1; i++) highPow = (highPow * BASE) % MOD;
    for (int i = 0; i < m; i++) windowHash = (windowHash * BASE + text.charAt(i)) % MOD;

    for (int i = 0; i <= text.length() - m; i++) {
        String candidate = hashToPattern.get(windowHash);
        if (candidate != null && text.substring(i, i + m).equals(candidate)) {
            results.get(candidate).add(i);
        }
        if (i < text.length() - m) {
            windowHash = (windowHash - text.charAt(i) * highPow % MOD + MOD) % MOD;
            windowHash = (windowHash * BASE + text.charAt(i + m)) % MOD;
        }
    }
    return results;
}
```

---

## 6.13 Algorithm Comparison

### Array Searching

| Algorithm           | Time (avg)     | Time (worst) | Requirement     | Best For                          |
|---------------------|----------------|--------------|-----------------|-----------------------------------|
| Linear Search       | O(n)           | O(n)         | None            | Unsorted or small arrays          |
| Binary Search       | O(log n)       | O(log n)     | Sorted          | General sorted search             |
| Interpolation Search| O(log log n)   | O(n)         | Sorted, uniform | Uniformly distributed data        |
| Exponential Search  | O(log n)       | O(log n)     | Sorted          | Unknown size / front-loaded       |

### String Pattern Matching

| Algorithm    | Preprocessing | Search      | Space  | Best For                           |
|--------------|---------------|-------------|--------|------------------------------------|
| Brute Force  | None          | O(n·m)      | O(1)   | Short text/pattern, simple cases   |
| KMP          | O(m)          | O(n)        | O(m)   | General pattern matching           |
| Boyer-Moore  | O(m + σ)      | O(n/m) best | O(σ)   | Long patterns, large alphabet      |
| Rabin-Karp   | O(m)          | O(n) avg    | O(1)   | Multiple patterns, 2D matching     |

*(σ = alphabet size)*

---

## 6.14 Practical String Search in Java

Java's built-in string methods use optimized variants of these algorithms:

```java
String text    = "the quick brown fox jumps over the lazy dog";
String pattern = "fox";

// String.indexOf — Boyer-Moore-Horspool variant internally
int idx = text.indexOf(pattern);               // 16
System.out.println(idx);

// All occurrences using indexOf loop
int start = 0;
while ((start = text.indexOf(pattern, start)) != -1) {
    System.out.println("Found at: " + start);
    start++;
}

// Java Regex — full pattern language (uses NFA/DFA internally)
import java.util.regex.*;
Pattern p = Pattern.compile("\\b\\w{4}\\b");  // 4-letter words
Matcher m = p.matcher(text);
while (m.find()) {
    System.out.println("'" + m.group() + "' at " + m.start());
}
// 'over' at 27, 'lazy' at 35

// String.contains — simple membership test
System.out.println(text.contains("fox")); // true

// String.matches — full string match against regex
System.out.println("hello123".matches("[a-z]+\\d+"));  // true
```

---

## 6.15 Complete Example — Search Benchmark

```java
import java.util.*;

public class SearchBenchmark {

    public static void main(String[] args) {
        // ── Array search comparison ───────────────────────────────
        int n = 1_000_000;
        int[] sorted = new int[n];
        for (int i = 0; i < n; i++) sorted[i] = i * 2; // 0, 2, 4, ..., 1999998
        int target = 999_998; // near the end

        long t1 = System.nanoTime();
        int li = linearSearch(sorted, target);
        long linearTime = System.nanoTime() - t1;

        t1 = System.nanoTime();
        int bi = binarySearch(sorted, target);
        long binaryTime = System.nanoTime() - t1;

        System.out.printf("Linear search: index=%d, time=%d ns%n",   li, linearTime);
        System.out.printf("Binary search: index=%d, time=%d ns%n%n", bi, binaryTime);

        // ── String pattern search comparison ─────────────────────
        // Build a large text with a pattern hidden near the end
        String text    = "a".repeat(100_000) + "AABABC" + "a".repeat(100_000);
        String pattern = "AABABC";

        t1 = System.nanoTime();
        List<Integer> bf = bruteForce(text, pattern);
        long bfTime = System.nanoTime() - t1;

        t1 = System.nanoTime();
        List<Integer> kmp = kmpSearch(text, pattern);
        long kmpTime = System.nanoTime() - t1;

        t1 = System.nanoTime();
        List<Integer> rk = rabinKarp(text, pattern);
        long rkTime = System.nanoTime() - t1;

        System.out.printf("Brute Force:  matches=%s, time=%d ms%n",
                          bf,  bfTime  / 1_000_000);
        System.out.printf("KMP:          matches=%s, time=%d ms%n",
                          kmp, kmpTime / 1_000_000);
        System.out.printf("Rabin-Karp:   matches=%s, time=%d ms%n",
                          rk,  rkTime  / 1_000_000);
    }

    // (include linearSearch, binarySearch, bruteForce, kmpSearch, rabinKarp from above)
}
```

---

## 6.16 Key Takeaways

| Concept                 | Summary                                                                          |
|-------------------------|----------------------------------------------------------------------------------|
| Linear Search           | O(n); works on any data; only choice for unsorted/linked structures              |
| Binary Search           | O(log n); requires sorted array; use `left + (right-left)/2` for safety         |
| Lower / Upper Bound     | Binary search variants for first/last occurrence; essential for duplicates       |
| Interpolation Search    | Smarter midpoint estimate; O(log log n) on uniform data                          |
| Brute Force String      | O(n·m); simple but wastes work on mismatches                                     |
| KMP                     | O(n+m); LPS array lets us skip ahead in pattern without backtracking in text     |
| Boyer-Moore             | O(n/m) best case; matches right-to-left; extremely fast for long patterns        |
| Rabin-Karp              | O(n+m) average; rolling hash; ideal for multiple patterns simultaneously         |
| Java built-ins          | `String.indexOf` uses Boyer-Moore variant; `Pattern`/`Matcher` uses regex NFA    |

---

## 6.17 Practice Problems

1. **Search in rotated sorted array** — A sorted array was rotated at an unknown pivot (e.g., `[4,5,6,7,0,1,2]`). Find a target value in O(log n). *(Hint: one half is always sorted — use that to decide which half to search.)*

2. **Find peak element** — A peak is an element greater than its neighbors. Find any peak in O(log n). The array may have multiple peaks.

3. **KMP construction** — Manually build the LPS array for the pattern `"ABCABCABC"`. Trace through the KMP algorithm to find all occurrences in the text `"ABCABCABCABC"`.

4. **Longest repeated substring** — Given a string, find the longest substring that appears at least twice. *(Hint: binary search on length + Rabin-Karp hashing.)*

5. **Anagram search** — Find all start indices in string `s` where an anagram of `p` begins. *(Hint: sliding window with character frequency counts — O(n).)*

---

## 6.18 Further Reading

- *Introduction to Algorithms* (CLRS) — Chapter 32 (String Matching)
- *Algorithms* — Sedgewick & Wayne, Chapter 5 (Strings)
- KMP visualizer: [https://visualgo.net/en/suffixarray](https://visualgo.net/en/suffixarray)
- Regex tester (Java flavor): [https://regex101.com](https://regex101.com)
- Java Pattern API: [https://docs.oracle.com/en/java/se/17/docs/api/java.base/java/util/regex/Pattern.html](https://docs.oracle.com/en/java/se/17/docs/api/java.base/java/util/regex/Pattern.html)

---

*CS 3303 Data Structures — Module 6 of 8*
*Previous: Module 5 — Advanced Tree Structures and Recursion Techniques*
*Next: Module 7 — Sorting Algorithms*
