# CS 3303 — Data Structures
## Module 5: Advanced Tree Structures and Recursion Techniques

---

## 5.1 Introduction

Module 4 introduced binary trees and how to traverse them. But a plain binary tree gives us no guarantees about *where* values are placed — finding an element still requires O(n) time in the worst case.

This module introduces trees with **rules** that make operations dramatically faster:

- The **Binary Search Tree (BST)** enforces an ordering property that enables O(log n) search
- **AVL Trees** and **Red-Black Trees** self-balance to keep that O(log n) guarantee even after insertions and deletions
- **Heaps** enforce a priority property and power the priority queue
- **Tries** organize strings character by character for lightning-fast prefix search

Alongside these structures, we deepen our understanding of **recursion** — the natural language of trees — including how to design recursive solutions, trace the call stack, and optimize with memoization.

---

## 5.2 Binary Search Tree (BST)

### The BST Property

A **Binary Search Tree** is a binary tree that enforces one rule at every node:

> For any node N:
> - All values in the **left subtree** are **less than** N's value
> - All values in the **right subtree** are **greater than** N's value

```
           [8]
          /   \
        [3]   [10]
        / \     \
      [1] [6]   [14]
          / \   /
        [4] [7][13]
```

This single rule lets us discard half the tree at every step — exactly like binary search on a sorted array.

### BST Operations and Complexity

| Operation | Average Case | Worst Case (degenerate) |
|-----------|-------------|--------------------------|
| Search    | O(log n)    | O(n)                     |
| Insert    | O(log n)    | O(n)                     |
| Delete    | O(log n)    | O(n)                     |

The worst case occurs when the tree degenerates into a linked list (inserting already-sorted data). This motivates self-balancing trees.

---

## 5.3 BST — Full Java Implementation

```java
public class BinarySearchTree {

    private static class Node {
        int data;
        Node left, right;
        Node(int data) { this.data = data; }
    }

    private Node root;

    // ─────────────────────────────────────────────
    //  INSERT — O(log n) average
    // ─────────────────────────────────────────────
    /**
     * Insert a value into the BST.
     * Navigate left if value < current, right if value > current.
     * Insert at the first null spot we reach.
     */
    public void insert(int value) {
        root = insertRec(root, value);
    }

    private Node insertRec(Node node, int value) {
        if (node == null) return new Node(value); // found the insertion point

        if (value < node.data)
            node.left  = insertRec(node.left,  value);
        else if (value > node.data)
            node.right = insertRec(node.right, value);
        // value == node.data → duplicate, ignore

        return node;
    }

    // ─────────────────────────────────────────────
    //  SEARCH — O(log n) average
    // ─────────────────────────────────────────────
    /**
     * Return true if the value exists in the BST.
     * At each node, go left (smaller) or right (larger).
     */
    public boolean search(int value) {
        return searchRec(root, value);
    }

    private boolean searchRec(Node node, int value) {
        if (node == null)         return false;        // not found
        if (value == node.data)   return true;         // found
        if (value < node.data)    return searchRec(node.left,  value);
        else                      return searchRec(node.right, value);
    }

    // ─────────────────────────────────────────────
    //  DELETE — O(log n) average
    // ─────────────────────────────────────────────
    /**
     * Delete a value from the BST. Three cases:
     *
     * Case 1 — Node has NO children (leaf):
     *   Simply remove it.
     *
     * Case 2 — Node has ONE child:
     *   Replace the node with its only child.
     *
     * Case 3 — Node has TWO children:
     *   Replace the node's value with its in-order successor
     *   (smallest value in the right subtree), then delete
     *   the in-order successor from the right subtree.
     */
    public void delete(int value) {
        root = deleteRec(root, value);
    }

    private Node deleteRec(Node node, int value) {
        if (node == null) return null; // value not found

        if (value < node.data) {
            node.left  = deleteRec(node.left,  value);
        } else if (value > node.data) {
            node.right = deleteRec(node.right, value);
        } else {
            // Found the node to delete

            // Case 1 & 2: zero or one child
            if (node.left  == null) return node.right;
            if (node.right == null) return node.left;

            // Case 3: two children — find in-order successor
            Node successor = findMin(node.right);      // smallest in right subtree
            node.data  = successor.data;               // overwrite with successor value
            node.right = deleteRec(node.right, successor.data); // delete successor
        }
        return node;
    }

    /** Find the leftmost (minimum) node in a subtree. */
    private Node findMin(Node node) {
        while (node.left != null) node = node.left;
        return node;
    }

    // ─────────────────────────────────────────────
    //  IN-ORDER → produces sorted output
    // ─────────────────────────────────────────────
    public void inOrder() {
        inOrderRec(root);
        System.out.println();
    }

    private void inOrderRec(Node node) {
        if (node == null) return;
        inOrderRec(node.left);
        System.out.print(node.data + " ");
        inOrderRec(node.right);
    }

    // ─────────────────────────────────────────────
    //  DEMO
    // ─────────────────────────────────────────────
    public static void main(String[] args) {
        BinarySearchTree bst = new BinarySearchTree();
        int[] values = {8, 3, 10, 1, 6, 14, 4, 7, 13};
        for (int v : values) bst.insert(v);

        bst.inOrder();                            // 1 3 4 6 7 8 10 13 14
        System.out.println(bst.search(6));        // true
        System.out.println(bst.search(99));       // false

        bst.delete(3);   // Case 3: two children
        bst.inOrder();   // 1 4 6 7 8 10 13 14

        bst.delete(1);   // Case 1: leaf
        bst.inOrder();   // 4 6 7 8 10 13 14
    }
}
```

---

## 5.4 AVL Tree (Self-Balancing BST)

### The Problem with Plain BSTs

Insert values 1, 2, 3, 4, 5 into a BST:
```
[1]
  \
  [2]
    \
    [3]
      \
      [4]
        \
        [5]     ← degenerate: height = n, search = O(n)
```

The BST property holds, but performance degrades to O(n). An **AVL tree** fixes this automatically.

### The AVL Property

An **AVL tree** (Adelson-Velsky and Landis, 1962) is a BST that enforces:

> At every node, |height(left subtree) − height(right subtree)| ≤ 1

This **balance factor** constraint guarantees height ≤ 1.44 · log₂(n), ensuring all operations stay O(log n).

### Balance Factor

```
Balance Factor (BF) = height(left) − height(right)

BF =  0 → perfectly balanced
BF =  1 → left-heavy by 1 (OK)
BF = -1 → right-heavy by 1 (OK)
BF =  2 → too left-heavy → rotation needed
BF = -2 → too right-heavy → rotation needed
```

### AVL Rotations

When a node becomes unbalanced after insert/delete, we **rotate** subtrees to restore balance. There are four cases:

#### Case 1: Right Rotation (Left-Left imbalance)

```
    z (BF=2)            y
   /                   / \
  y (BF=1)    →      x    z
 /
x
```

```java
private Node rotateRight(Node z) {
    Node y  = z.left;
    Node T3 = y.right;

    y.right = z;        // y becomes new root
    z.left  = T3;       // T3 moves to z's left

    // Update heights (leaves first)
    z.height = 1 + Math.max(height(z.left), height(z.right));
    y.height = 1 + Math.max(height(y.left), height(y.right));

    return y; // new subtree root
}
```

#### Case 2: Left Rotation (Right-Right imbalance)

```
  z (BF=-2)              y
    \                   / \
     y (BF=-1)  →    z     x
      \
       x
```

```java
private Node rotateLeft(Node z) {
    Node y  = z.right;
    Node T2 = y.left;

    y.left  = z;        // y becomes new root
    z.right = T2;

    z.height = 1 + Math.max(height(z.left), height(z.right));
    y.height = 1 + Math.max(height(y.left), height(y.right));

    return y;
}
```

#### Case 3: Left-Right Rotation
Rotate left on child, then right on current node.

#### Case 4: Right-Left Rotation
Rotate right on child, then left on current node.

### AVL Insert

```java
public class AVLTree {

    private static class Node {
        int data, height;
        Node left, right;
        Node(int data) { this.data = data; this.height = 1; }
    }

    private Node root;

    private int height(Node n)  { return n == null ? 0 : n.height; }
    private int balanceFactor(Node n) {
        return n == null ? 0 : height(n.left) - height(n.right);
    }

    public void insert(int value) { root = insertRec(root, value); }

    private Node insertRec(Node node, int value) {
        // 1. Standard BST insert
        if (node == null) return new Node(value);
        if (value < node.data)      node.left  = insertRec(node.left,  value);
        else if (value > node.data) node.right = insertRec(node.right, value);
        else                        return node; // duplicate

        // 2. Update height
        node.height = 1 + Math.max(height(node.left), height(node.right));

        // 3. Check balance and rotate if needed
        int bf = balanceFactor(node);

        // Left-Left
        if (bf > 1  && value < node.left.data)
            return rotateRight(node);

        // Right-Right
        if (bf < -1 && value > node.right.data)
            return rotateLeft(node);

        // Left-Right
        if (bf > 1  && value > node.left.data) {
            node.left = rotateLeft(node.left);
            return rotateRight(node);
        }

        // Right-Left
        if (bf < -1 && value < node.right.data) {
            node.right = rotateRight(node.right);
            return rotateLeft(node);
        }

        return node; // balanced, no rotation needed
    }

    private Node rotateRight(Node z) {
        Node y = z.left; Node T3 = y.right;
        y.right = z; z.left = T3;
        z.height = 1 + Math.max(height(z.left), height(z.right));
        y.height = 1 + Math.max(height(y.left), height(y.right));
        return y;
    }

    private Node rotateLeft(Node z) {
        Node y = z.right; Node T2 = y.left;
        y.left = z; z.right = T2;
        z.height = 1 + Math.max(height(z.left), height(z.right));
        y.height = 1 + Math.max(height(y.left), height(y.right));
        return y;
    }
}
```

---

## 5.5 Red-Black Tree — Concepts

A **Red-Black tree** is another self-balancing BST that colors each node red or black and enforces four rules:

1. Every node is **red** or **black**
2. The **root** is always black
3. No two **consecutive red** nodes (a red node's parent and children must be black)
4. Every path from root to a null leaf has the **same number of black nodes**

```
         [8 B]
        /     \
    [3 R]     [10 B]
    /   \         \
 [1 B] [6 B]    [14 R]
```

These rules guarantee height ≤ 2·log₂(n+1), giving O(log n) operations. Red-Black trees are used in Java's `TreeMap` and `TreeSet` because they rebalance with fewer rotations than AVL trees (better for frequent insertions/deletions).

| Feature              | AVL Tree                     | Red-Black Tree               |
|----------------------|------------------------------|------------------------------|
| Balance guarantee    | Stricter (BF ≤ 1)            | Looser (height ≤ 2 log n)    |
| Lookup speed         | Slightly faster              | Slightly slower              |
| Insert/Delete speed  | More rotations               | Fewer rotations              |
| Best for             | Read-heavy workloads         | Write-heavy workloads        |
| Java uses it in      | —                            | `TreeMap`, `TreeSet`         |

---

## 5.6 Heap

### What is a Heap?

A **heap** is a **complete binary tree** that satisfies the **heap property**:

- **Max-Heap**: Every parent is ≥ all its children. Root = maximum element.
- **Min-Heap**: Every parent is ≤ all its children. Root = minimum element.

```
Max-Heap:              Min-Heap:
      [90]                  [1]
     /    \                /   \
  [75]   [80]           [3]   [5]
  /  \   /  \           / \   / \
[60][65][70][50]       [4][8][9][6]
```

### Array Representation

A heap is stored in an array — no pointers needed! For a node at index `i`:

```
Left child  = 2i + 1
Right child = 2i + 2
Parent      = (i - 1) / 2

Max-Heap array: [90, 75, 80, 60, 65, 70, 50]
Index:           [0]  [1] [2] [3] [4] [5] [6]
```

### Heap Operations

```java
public class MaxHeap {
    private int[] data;
    private int size;
    private int capacity;

    public MaxHeap(int capacity) {
        this.capacity = capacity;
        this.data = new int[capacity];
        this.size = 0;
    }

    // ─────────────────────────────────────────────
    //  INSERT (sift up) — O(log n)
    // ─────────────────────────────────────────────
    /**
     * Add element at the end, then "sift up" until heap property restored.
     * Swap with parent while new element > parent.
     */
    public void insert(int value) {
        if (size == capacity) throw new RuntimeException("Heap full");
        data[size] = value;
        siftUp(size);
        size++;
    }

    private void siftUp(int i) {
        while (i > 0) {
            int parent = (i - 1) / 2;
            if (data[i] > data[parent]) {
                swap(i, parent);
                i = parent;
            } else break; // heap property satisfied
        }
    }

    // ─────────────────────────────────────────────
    //  EXTRACT MAX (sift down) — O(log n)
    // ─────────────────────────────────────────────
    /**
     * Remove and return the root (max).
     * Replace root with last element, then "sift down" to restore heap.
     */
    public int extractMax() {
        if (size == 0) throw new RuntimeException("Heap empty");
        int max = data[0];
        data[0] = data[--size]; // move last element to root
        siftDown(0);
        return max;
    }

    private void siftDown(int i) {
        while (true) {
            int left    = 2 * i + 1;
            int right   = 2 * i + 2;
            int largest = i;

            if (left  < size && data[left]  > data[largest]) largest = left;
            if (right < size && data[right] > data[largest]) largest = right;

            if (largest != i) {
                swap(i, largest);
                i = largest;
            } else break;
        }
    }

    // ─────────────────────────────────────────────
    //  HEAPIFY — Build heap from array in O(n)
    // ─────────────────────────────────────────────
    /**
     * Convert an arbitrary array into a valid heap.
     * Start from the last non-leaf node and sift down each node.
     * Surprisingly O(n) — not O(n log n).
     */
    public static int[] heapify(int[] arr) {
        int n = arr.length;
        for (int i = n / 2 - 1; i >= 0; i--) { // start at last non-leaf
            siftDownArr(arr, n, i);
        }
        return arr;
    }

    private static void siftDownArr(int[] arr, int n, int i) {
        int largest = i;
        int left = 2*i+1, right = 2*i+2;
        if (left  < n && arr[left]  > arr[largest]) largest = left;
        if (right < n && arr[right] > arr[largest]) largest = right;
        if (largest != i) {
            int tmp = arr[i]; arr[i] = arr[largest]; arr[largest] = tmp;
            siftDownArr(arr, n, largest);
        }
    }

    private void swap(int i, int j) {
        int tmp = data[i]; data[i] = data[j]; data[j] = tmp;
    }

    public int peek() { return data[0]; }
    public boolean isEmpty() { return size == 0; }
}
```

### Heap Sort

```java
/**
 * Heap Sort: build a max-heap, then repeatedly extract the max.
 * Each extraction places the max at the end of the array.
 *
 * Time: O(n log n)   Space: O(1)  ← in-place!
 */
public static void heapSort(int[] arr) {
    int n = arr.length;

    // Phase 1: Build max-heap in O(n)
    for (int i = n / 2 - 1; i >= 0; i--)
        siftDownArr(arr, n, i);

    // Phase 2: Extract elements one by one
    for (int i = n - 1; i > 0; i--) {
        int tmp = arr[0]; arr[0] = arr[i]; arr[i] = tmp; // swap max to end
        siftDownArr(arr, i, 0);  // restore heap on reduced array
    }
}
// [64,34,25,12,22,11,90] → [11,12,22,25,34,64,90]
```

---

## 5.7 Trie (Prefix Tree)

### What is a Trie?

A **Trie** (from re**trie**val) is a tree where each path from root to a node spells out a string prefix. Each node represents a character, and words are stored by their characters top-down.

```
Words: "cat", "car", "card", "care", "bat"

          [root]
          /    \
        [c]    [b]
         |      |
        [a]    [a]
        / \     |
      [t] [r]  [t*]    (* = end of word)
           |
          [d*]
           |
          [e*]
```

### Trie Node and Implementation

```java
public class Trie {

    private static class TrieNode {
        TrieNode[] children = new TrieNode[26]; // one slot per letter a-z
        boolean isEndOfWord = false;
    }

    private TrieNode root = new TrieNode();

    // ─────────────────────────────────────────────
    //  INSERT — O(m) where m = word length
    // ─────────────────────────────────────────────
    public void insert(String word) {
        TrieNode current = root;
        for (char c : word.toCharArray()) {
            int idx = c - 'a';                          // 'a'=0, 'b'=1, ...
            if (current.children[idx] == null)
                current.children[idx] = new TrieNode(); // create node if missing
            current = current.children[idx];
        }
        current.isEndOfWord = true; // mark end of word
    }

    // ─────────────────────────────────────────────
    //  SEARCH — O(m)
    // ─────────────────────────────────────────────
    public boolean search(String word) {
        TrieNode node = findNode(word);
        return node != null && node.isEndOfWord;
    }

    // ─────────────────────────────────────────────
    //  PREFIX CHECK — O(m)
    // ─────────────────────────────────────────────
    /** Returns true if any word in the trie starts with the given prefix. */
    public boolean startsWith(String prefix) {
        return findNode(prefix) != null;
    }

    private TrieNode findNode(String s) {
        TrieNode current = root;
        for (char c : s.toCharArray()) {
            int idx = c - 'a';
            if (current.children[idx] == null) return null; // not found
            current = current.children[idx];
        }
        return current;
    }

    // ─────────────────────────────────────────────
    //  AUTOCOMPLETE — O(m + output)
    // ─────────────────────────────────────────────
    /** Return all words in the trie that start with the given prefix. */
    public List<String> autocomplete(String prefix) {
        List<String> results = new ArrayList<>();
        TrieNode node = findNode(prefix);
        if (node != null) collectWords(node, new StringBuilder(prefix), results);
        return results;
    }

    private void collectWords(TrieNode node, StringBuilder current, List<String> results) {
        if (node.isEndOfWord) results.add(current.toString());
        for (int i = 0; i < 26; i++) {
            if (node.children[i] != null) {
                current.append((char)('a' + i));       // add letter
                collectWords(node.children[i], current, results);
                current.deleteCharAt(current.length() - 1); // backtrack
            }
        }
    }

    // Demo
    public static void main(String[] args) {
        Trie trie = new Trie();
        String[] words = {"cat","car","card","care","bat","ball"};
        for (String w : words) trie.insert(w);

        System.out.println(trie.search("car"));          // true
        System.out.println(trie.search("ca"));           // false (not a full word)
        System.out.println(trie.startsWith("ca"));       // true
        System.out.println(trie.autocomplete("car"));    // [car, card, care]
    }
}
```

---

## 5.8 Recursion Deep Dive

Trees and recursion are inseparable. Here we crystallize the principles that make recursive solutions work.

### The Three Laws of Recursion

1. **Base case** — A condition where the problem is solved directly (no more recursion)
2. **Recursive case** — The function calls itself on a *smaller* version of the problem
3. **Progress** — Each recursive call must move toward the base case

```java
// All three laws in one simple example:
public static int factorial(int n) {
    if (n == 0) return 1;          // Law 1: base case
    return n * factorial(n - 1);  // Laws 2 & 3: smaller problem (n-1)
}
```

### Visualizing the Call Stack

```
factorial(4)
  → 4 * factorial(3)
         → 3 * factorial(2)
                → 2 * factorial(1)
                       → 1 * factorial(0)
                                → return 1   ← base case
                       ← return 1*1 = 1
                ← return 2*1 = 2
         ← return 3*2 = 6
  ← return 4*6 = 24
```

Each call is pushed onto the **call stack**. When the base case is reached, results bubble back up. Maximum stack depth = recursion depth. For deep recursion this can cause a **StackOverflowError**.

### Recursion Patterns on Trees

```java
// Pattern 1: Compute a value bottom-up
public static int sumTree(TreeNode node) {
    if (node == null) return 0;
    return node.data + sumTree(node.left) + sumTree(node.right);
}

// Pattern 2: Check a property at every node
public static boolean allPositive(TreeNode node) {
    if (node == null)    return true;             // vacuously true
    if (node.data <= 0)  return false;            // found a violation
    return allPositive(node.left) && allPositive(node.right);
}

// Pattern 3: Transform the tree (return modified nodes)
public static TreeNode doubleValues(TreeNode node) {
    if (node == null) return null;
    node.data  *= 2;
    node.left   = doubleValues(node.left);
    node.right  = doubleValues(node.right);
    return node;
}
```

---

## 5.9 Memoization and Dynamic Programming on Trees

### The Exponential Cost of Naive Recursion

```java
// Fibonacci — naïve: O(2ⁿ) calls, massive redundant work
public static int fib(int n) {
    if (n <= 1) return n;
    return fib(n-1) + fib(n-2); // fib(3) computed many times!
}
```

**Call tree for fib(5):**
```
                    fib(5)
                  /        \
              fib(4)        fib(3)
             /     \        /   \
         fib(3)  fib(2)  fib(2) fib(1)
         /   \
     fib(2) fib(1)    ← fib(2) computed 3 times, fib(3) computed 2 times
```

### Memoization — Top-Down DP

```java
/**
 * Memoized Fibonacci: cache results to avoid recomputation.
 * Each subproblem computed exactly once → O(n) time, O(n) space.
 */
public static int fibMemo(int n, Map<Integer, Integer> memo) {
    if (n <= 1) return n;
    if (memo.containsKey(n)) return memo.get(n); // cache hit!

    int result = fibMemo(n-1, memo) + fibMemo(n-2, memo);
    memo.put(n, result);  // store before returning
    return result;
}

// Usage:
Map<Integer, Integer> cache = new HashMap<>();
System.out.println(fibMemo(40, cache)); // 102334155 — instant
```

### Tabulation — Bottom-Up DP

```java
/**
 * Bottom-up Fibonacci: fill a table from base cases upward.
 * No recursion, no stack overflow risk.
 * Time: O(n)   Space: O(n)  (reducible to O(1) with two variables)
 */
public static int fibTab(int n) {
    if (n <= 1) return n;
    int[] dp = new int[n + 1];
    dp[0] = 0;
    dp[1] = 1;
    for (int i = 2; i <= n; i++) {
        dp[i] = dp[i-1] + dp[i-2];
    }
    return dp[n];
}
```

### Tree DP — Diameter of a Binary Tree

The diameter is the longest path between any two nodes (may not pass through root).

```java
/**
 * Find the diameter of a binary tree.
 *
 * At each node, the longest path through that node =
 *   height(left) + height(right) + 2
 *
 * We track a global max as we recurse.
 * Time: O(n)   Space: O(h)
 */
public static int diameterOfBinaryTree(TreeNode root) {
    int[] maxDiameter = {0}; // use array to pass by reference in lambda
    heightForDiameter(root, maxDiameter);
    return maxDiameter[0];
}

private static int heightForDiameter(TreeNode node, int[] maxDiameter) {
    if (node == null) return -1;
    int leftH  = heightForDiameter(node.left,  maxDiameter);
    int rightH = heightForDiameter(node.right, maxDiameter);

    int diameter = leftH + rightH + 2;              // path through this node
    maxDiameter[0] = Math.max(maxDiameter[0], diameter);

    return 1 + Math.max(leftH, rightH);             // height of this subtree
}
```

---

## 5.10 Backtracking on Trees

**Backtracking** is a recursion technique where we explore a path, and if it doesn't lead to a solution, we "undo" the last step and try another.

### All Root-to-Leaf Paths

```java
/**
 * Print every root-to-leaf path in the tree.
 * Build the path as we go down; remove the last element on the way back up.
 *
 * Time: O(n)   Space: O(h) for the call stack + O(h) for path storage
 */
public static void allPaths(TreeNode node, List<Integer> path) {
    if (node == null) return;

    path.add(node.data);                         // choose: add to path

    if (node.left == null && node.right == null) {
        System.out.println(path);                // reached a leaf: print path
    } else {
        allPaths(node.left,  path);              // explore left
        allPaths(node.right, path);              // explore right
    }

    path.remove(path.size() - 1);               // unchoose: backtrack
}

// For tree [1]->[2,3]->[4,5,6,7]:
// [1, 2, 4]
// [1, 2, 5]
// [1, 3, 6]
// [1, 3, 7]
```

### Path Sum II — All Paths Summing to Target

```java
/**
 * Find all root-to-leaf paths that sum to a target value.
 * Classic backtracking: add node, recurse, remove node.
 *
 * Time: O(n²) worst case (copying paths)
 */
public static List<List<Integer>> pathSumAll(TreeNode node, int target) {
    List<List<Integer>> result = new ArrayList<>();
    backtrack(node, target, new ArrayList<>(), result);
    return result;
}

private static void backtrack(TreeNode node, int remaining,
                               List<Integer> path,
                               List<List<Integer>> result) {
    if (node == null) return;

    path.add(node.data);
    remaining -= node.data;

    if (node.left == null && node.right == null && remaining == 0) {
        result.add(new ArrayList<>(path)); // found a valid path — copy it
    } else {
        backtrack(node.left,  remaining, path, result);
        backtrack(node.right, remaining, path, result);
    }

    path.remove(path.size() - 1); // backtrack
}
```

---

## 5.11 Advanced Tree Structures — Comparison

| Structure      | Ordering Property      | Balance         | Insert/Delete | Search   | Best For                          |
|----------------|------------------------|-----------------|---------------|----------|-----------------------------------|
| BST            | Left < Node < Right    | None (varies)   | O(log n) avg  | O(log n) avg | Simple ordered storage        |
| AVL Tree       | BST + BF ≤ 1           | Strict          | O(log n)      | O(log n) | Read-heavy, fast lookup           |
| Red-Black Tree | BST + color rules      | Loose           | O(log n)      | O(log n) | Write-heavy (`TreeMap`/`TreeSet`) |
| Max/Min Heap   | Parent ≥/≤ children    | Complete        | O(log n)      | O(n)*    | Priority queues, heap sort        |
| Trie           | Characters along path  | N/A             | O(m)          | O(m)     | String prefix search, autocomplete|

*Heap search is O(n) because only root is guaranteed to be max/min.

---

## 5.12 Key Takeaways

| Concept             | Summary                                                                       |
|---------------------|-------------------------------------------------------------------------------|
| BST                 | BST property enables O(log n) search; degenerates to O(n) when unbalanced    |
| AVL Tree            | Self-balancing BST with strict BF ≤ 1; uses rotations; best for lookups      |
| Red-Black Tree      | Self-balancing with color rules; fewer rotations; Java's `TreeMap`/`TreeSet` |
| Max-Heap            | Complete tree, parent ≥ children; O(log n) insert/extract; powers PriorityQueue |
| Heap Sort           | O(n log n), in-place; build heap then repeatedly extract max                 |
| Trie                | Character-indexed tree; O(m) insert/search; ideal for prefix/autocomplete     |
| Memoization         | Cache recursive results top-down; turns O(2ⁿ) into O(n)                     |
| Tabulation          | Build DP table bottom-up; no recursion overhead                              |
| Backtracking        | Explore + undo; used for path enumeration and constraint satisfaction         |
| Tree DP             | Compute subtree properties bottom-up (diameter, max path sum)                |

---

## 5.13 Practice Problems

1. **BST Validation** — Write a function `isValidBST(TreeNode root)` that verifies a binary tree satisfies the BST property. *(Hint: pass min/max bounds down the recursion.)*

2. **Kth Smallest in BST** — Find the kth smallest element in a BST without converting to an array. *(Hint: in-order traversal counts nodes.)*

3. **Top K Frequent Elements** — Given an array of integers, return the `k` most frequent elements using a min-heap of size `k`.

4. **Word Search with Trie** — Given a board of characters and a list of words, find all words that can be formed by a path on the board. Use a trie to prune dead-end searches early.

5. **Longest Univalue Path** — Find the length of the longest path in a binary tree where every node along the path has the same value. Use tree DP (return useful info bottom-up).

---

## 5.14 Further Reading

- *Introduction to Algorithms* (CLRS) — Chapters 12–13 (BST, Red-Black Trees)
- *Data Structures and Algorithm Analysis in Java* — Weiss, Chapters 4–6
- AVL Tree visualizer: [https://visualgo.net/en/bst](https://visualgo.net/en/bst)
- Heap visualizer: [https://visualgo.net/en/heap](https://visualgo.net/en/heap)
- Java `TreeMap` source (Red-Black): [https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/TreeMap.java](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/TreeMap.java)

---

*CS 3303 Data Structures — Module 5 of 8*
*Previous: Module 4 — Binary Trees and Tree Traversals*
*Next: Module 6 — Searching and Pattern Matching Techniques*
