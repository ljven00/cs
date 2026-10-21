# CS 3303 — Data Structures
## Module 4: Binary Trees and Tree Traversals

---

## 4.1 Introduction

Arrays and linked lists organize data **linearly** — one element after another. But many real-world relationships are **hierarchical**: a file system, a company org chart, the HTML structure of a webpage, a family tree. For these, we need a new shape.

**Trees** are the go-to data structure for hierarchical data. Among all tree types, the **binary tree** is the foundation of the most important structures in computer science — binary search trees, heaps, expression trees, and decision trees all build on it.

In this module you will learn what binary trees are, how to build them in Java, and — critically — how to **traverse** them in four distinct ways, each revealing different information about the tree.

---

## 4.2 Tree Terminology

Before writing code, we need a shared vocabulary.

```
              [A]           ← root (level 0)
             /   \
           [B]   [C]        ← level 1
           / \     \
         [D] [E]   [F]      ← level 2
             /
           [G]              ← level 3
```

| Term            | Definition                                                      |
|-----------------|-----------------------------------------------------------------|
| **Root**        | The topmost node (no parent) — [A] above                        |
| **Node**        | Each element in the tree                                        |
| **Edge**        | A link between a parent and a child                            |
| **Parent**      | A node with children — [B] is parent of [D] and [E]            |
| **Child**       | A node with a parent — [D] and [E] are children of [B]         |
| **Leaf**        | A node with no children — [D], [G], [F]                        |
| **Subtree**     | A node plus all its descendants                                 |
| **Height**      | Longest path from root to a leaf — height above = 3            |
| **Depth/Level** | Distance from root to a node — depth of [G] = 3                |
| **Degree**      | Number of children a node has                                   |
| **Sibling**     | Nodes sharing the same parent                                   |

---

## 4.3 What is a Binary Tree?

A **binary tree** is a tree where each node has **at most two children**: a **left child** and a **right child**.

```
         [10]
        /    \
      [5]    [15]
      / \      \
    [3] [7]   [20]
```

### Types of Binary Trees

| Type                    | Definition                                                                 |
|-------------------------|----------------------------------------------------------------------------|
| **Full**                | Every node has 0 or 2 children (never 1)                                   |
| **Complete**            | All levels filled except possibly the last, which fills left-to-right      |
| **Perfect**             | All internal nodes have 2 children; all leaves are at the same level       |
| **Balanced**            | Height is O(log n); no subtree is dramatically deeper than another         |
| **Degenerate (skewed)** | Every node has only one child — essentially a linked list; height = n      |

### Why Balance Matters

A balanced binary tree of `n` nodes has height ≈ log₂(n). An unbalanced (degenerate) tree has height = n.

```
Balanced (n=7, height=2):      Degenerate (n=7, height=6):
        [4]                    [1]
       /   \                     \
     [2]   [6]                   [2]
     / \   / \                     \
   [1][3][5][7]                    [3]
                                     \
                                     [4] ...
```
Operations on a balanced tree cost O(log n); on a degenerate tree they degrade to O(n).

---

## 4.4 The Node Class and Binary Tree in Java

```java
/**
 * A single node in a binary tree.
 * Holds data plus references to left and right children.
 */
public class TreeNode {
    int data;
    TreeNode left;
    TreeNode right;

    public TreeNode(int data) {
        this.data  = data;
        this.left  = null;
        this.right = null;
    }
}
```

```java
/**
 * Build this tree manually:
 *
 *         [10]
 *        /    \
 *      [5]    [15]
 *      / \      \
 *    [3] [7]   [20]
 */
public class BinaryTree {
    TreeNode root;

    public BinaryTree() {
        root = new TreeNode(10);
        root.left            = new TreeNode(5);
        root.right           = new TreeNode(15);
        root.left.left       = new TreeNode(3);
        root.left.right      = new TreeNode(7);
        root.right.right     = new TreeNode(20);
    }
}
```

---

## 4.5 Tree Traversals — Overview

A **traversal** visits every node in the tree exactly once. Unlike arrays, a tree has no single "natural" order — the order we choose reveals different structure.

There are **four** standard traversals:

| Traversal        | Order                          | Visit root…       |
|------------------|--------------------------------|-------------------|
| Pre-order        | Root → Left → Right            | First (before subtrees) |
| In-order         | Left → Root → Right            | Middle (between subtrees) |
| Post-order       | Left → Right → Root            | Last (after subtrees) |
| Level-order (BFS)| Level by level, left to right  | Uses a queue      |

We'll use this reference tree throughout:

```
           [1]
          /   \
        [2]   [3]
        / \   / \
      [4] [5][6] [7]
```

---

## 4.6 Pre-order Traversal (Root → Left → Right)

**Visit the node first, then recurse left, then recurse right.**

Think of it as: "process me before my children." Useful for copying a tree, printing a directory structure, or serializing a tree.

```
Pre-order result: 1, 2, 4, 5, 3, 6, 7
```

```
Visit [1]  →  go left
  Visit [2]  →  go left
    Visit [4]  →  no children, backtrack
  Back at [2]  →  go right
    Visit [5]  →  no children, backtrack
Back at [1]  →  go right
  Visit [3]  →  go left
    Visit [6]  →  no children, backtrack
  Back at [3]  →  go right
    Visit [7]  →  no children, backtrack
```

```java
/**
 * Pre-order traversal: Root → Left → Right
 * Time: O(n)   Space: O(h) where h = height (recursion stack)
 */
public static void preOrder(TreeNode node) {
    if (node == null) return;      // base case

    System.out.print(node.data + " "); // 1. Visit root
    preOrder(node.left);               // 2. Recurse left
    preOrder(node.right);              // 3. Recurse right
}

// Output: 1 2 4 5 3 6 7
```

### Iterative Pre-order (using a stack)

```java
/**
 * Iterative pre-order using an explicit stack.
 * Push right child first so left is processed first (LIFO).
 */
public static void preOrderIterative(TreeNode root) {
    if (root == null) return;
    Deque<TreeNode> stack = new ArrayDeque<>();
    stack.push(root);

    while (!stack.isEmpty()) {
        TreeNode node = stack.pop();
        System.out.print(node.data + " ");     // visit

        if (node.right != null) stack.push(node.right); // right pushed first
        if (node.left  != null) stack.push(node.left);  // left on top → processed first
    }
}
```

---

## 4.7 In-order Traversal (Left → Root → Right)

**Recurse left first, visit the node, then recurse right.**

The most important property: **in-order traversal of a Binary Search Tree produces elements in sorted order.** This makes it essential for BST operations (covered more in Module 5).

```
In-order result: 4, 2, 5, 1, 6, 3, 7
```

```java
/**
 * In-order traversal: Left → Root → Right
 * Key use: produces sorted output on a BST.
 * Time: O(n)   Space: O(h)
 */
public static void inOrder(TreeNode node) {
    if (node == null) return;

    inOrder(node.left);                    // 1. Recurse left
    System.out.print(node.data + " ");     // 2. Visit root
    inOrder(node.right);                   // 3. Recurse right
}

// Output: 4 2 5 1 6 3 7
```

### Iterative In-order

```java
/**
 * Iterative in-order: go as far left as possible, then process, then go right.
 */
public static void inOrderIterative(TreeNode root) {
    Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode current = root;

    while (current != null || !stack.isEmpty()) {
        // Push all left nodes
        while (current != null) {
            stack.push(current);
            current = current.left;
        }
        // Process node
        current = stack.pop();
        System.out.print(current.data + " ");
        // Move to right subtree
        current = current.right;
    }
}
```

---

## 4.8 Post-order Traversal (Left → Right → Root)

**Recurse left, recurse right, then visit the node last.**

Think of it as: "process children before me." Ideal for deleting a tree (delete children before parent), evaluating expression trees, and calculating directory sizes.

```
Post-order result: 4, 5, 2, 6, 7, 3, 1
```

```java
/**
 * Post-order traversal: Left → Right → Root
 * Key use: delete/free a tree, evaluate expressions.
 * Time: O(n)   Space: O(h)
 */
public static void postOrder(TreeNode node) {
    if (node == null) return;

    postOrder(node.left);                  // 1. Recurse left
    postOrder(node.right);                 // 2. Recurse right
    System.out.print(node.data + " ");     // 3. Visit root
}

// Output: 4 5 2 6 7 3 1
```

### Iterative Post-order (two-stack approach)

```java
/**
 * Iterative post-order using two stacks.
 * Stack 1 processes nodes in "root-right-left" order;
 * Stack 2 reverses that to give "left-right-root".
 */
public static void postOrderIterative(TreeNode root) {
    if (root == null) return;
    Deque<TreeNode> stack1 = new ArrayDeque<>();
    Deque<TreeNode> stack2 = new ArrayDeque<>();

    stack1.push(root);
    while (!stack1.isEmpty()) {
        TreeNode node = stack1.pop();
        stack2.push(node);                          // collect in reverse
        if (node.left  != null) stack1.push(node.left);
        if (node.right != null) stack1.push(node.right);
    }
    while (!stack2.isEmpty()) {
        System.out.print(stack2.pop().data + " ");  // print in post-order
    }
}
```

---

## 4.9 Level-order Traversal / BFS (Level by Level)

**Visit all nodes at depth 0, then depth 1, then depth 2, etc.**

Uses a **queue** — enqueue a node, then enqueue its children before moving on. This mirrors how BFS explores a graph.

```
Level-order result: 1, 2, 3, 4, 5, 6, 7
```

```java
/**
 * Level-order (BFS) traversal using a queue.
 * Visits all nodes at each level before going deeper.
 * Time: O(n)   Space: O(w) where w = max width of tree
 */
public static void levelOrder(TreeNode root) {
    if (root == null) return;
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);

    while (!queue.isEmpty()) {
        TreeNode node = queue.poll();              // dequeue front
        System.out.print(node.data + " ");         // visit

        if (node.left  != null) queue.offer(node.left);  // enqueue children
        if (node.right != null) queue.offer(node.right);
    }
}

// Output: 1 2 3 4 5 6 7
```

### Level-order with Level Separation

```java
/**
 * Level-order that prints each level on its own line.
 * Snapshot the queue size at the start of each level to
 * know exactly how many nodes belong to that level.
 */
public static void levelOrderByLevel(TreeNode root) {
    if (root == null) return;
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    int level = 0;

    while (!queue.isEmpty()) {
        int levelSize = queue.size();       // nodes at this level
        System.out.print("Level " + level + ": ");

        for (int i = 0; i < levelSize; i++) {
            TreeNode node = queue.poll();
            System.out.print(node.data + " ");
            if (node.left  != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        System.out.println();
        level++;
    }
}
/*
Level 0: 1
Level 1: 2 3
Level 2: 4 5 6 7
*/
```

---

## 4.10 Traversal Summary

Using our reference tree:
```
           [1]
          /   \
        [2]   [3]
        / \   / \
      [4] [5][6] [7]
```

| Traversal   | Output          | Memory used | Primary use case                    |
|-------------|-----------------|-------------|--------------------------------------|
| Pre-order   | 1 2 4 5 3 6 7   | O(h) stack  | Copy tree, serialize, directory print |
| In-order    | 4 2 5 1 6 3 7   | O(h) stack  | Sorted output from BST               |
| Post-order  | 4 5 2 6 7 3 1   | O(h) stack  | Delete tree, evaluate expressions    |
| Level-order | 1 2 3 4 5 6 7   | O(w) queue  | BFS, shortest path, level printing   |

*(h = height of tree, w = maximum width of tree)*

---

## 4.11 Key Tree Properties and Algorithms

### Height of a Tree

```java
/**
 * Compute the height of a binary tree recursively.
 * Height = 1 + max(height of left subtree, height of right subtree)
 * An empty tree has height -1 (or 0, depending on convention — we use -1 here).
 *
 * Time: O(n)   Space: O(h)
 */
public static int height(TreeNode node) {
    if (node == null) return -1;                          // base case
    int leftHeight  = height(node.left);
    int rightHeight = height(node.right);
    return 1 + Math.max(leftHeight, rightHeight);
}
```

### Count Nodes

```java
/**
 * Count the total number of nodes in the tree.
 * Time: O(n)
 */
public static int countNodes(TreeNode node) {
    if (node == null) return 0;
    return 1 + countNodes(node.left) + countNodes(node.right);
}
```

### Count Leaves

```java
/**
 * Count all leaf nodes (nodes with no children).
 * Time: O(n)
 */
public static int countLeaves(TreeNode node) {
    if (node == null) return 0;
    if (node.left == null && node.right == null) return 1; // it's a leaf
    return countLeaves(node.left) + countLeaves(node.right);
}
```

### Check if a Tree is Balanced

```java
/**
 * A tree is balanced if, at every node, the height difference
 * between left and right subtrees is at most 1.
 *
 * Efficient approach: return -1 as a sentinel for "unbalanced"
 * so we short-circuit without extra traversals.
 *
 * Time: O(n)   Space: O(h)
 */
public static boolean isBalanced(TreeNode root) {
    return checkHeight(root) != -1;
}

private static int checkHeight(TreeNode node) {
    if (node == null) return 0;

    int leftH  = checkHeight(node.left);
    if (leftH == -1) return -1;  // already unbalanced below

    int rightH = checkHeight(node.right);
    if (rightH == -1) return -1;

    if (Math.abs(leftH - rightH) > 1) return -1; // unbalanced here

    return 1 + Math.max(leftH, rightH);
}
```

### Mirror / Invert a Tree

```java
/**
 * Invert (mirror) a binary tree.
 * Swap left and right subtrees at every node, recursively.
 *
 * Before:        After:
 *     [1]            [1]
 *    /   \          /   \
 *  [2]   [3]      [3]   [2]
 *
 * Time: O(n)   Space: O(h)
 */
public static TreeNode invertTree(TreeNode node) {
    if (node == null) return null;

    TreeNode temp   = node.left;       // swap children
    node.left       = node.right;
    node.right      = temp;

    invertTree(node.left);             // recurse on swapped children
    invertTree(node.right);
    return node;
}
```

---

## 4.12 Expression Trees

An **expression tree** is a binary tree where:
- **Leaves** hold operands (numbers or variables)
- **Internal nodes** hold operators (+, −, ×, ÷)

```
     Expression: (3 + 4) * (5 - 2)

             [*]
            /   \
          [+]   [-]
          / \   / \
        [3] [4][5] [2]
```

The three traversals naturally produce three notation forms:

```java
/**
 * In-order  → Infix:   ((3 + 4) * (5 - 2))   ← human-readable
 * Pre-order → Prefix:  * + 3 4 - 5 2          ← operator first (LISP-style)
 * Post-order→ Postfix: 3 4 + 5 2 - *          ← stack-evaluable (RPN)
 */
public static int evaluateExpTree(TreeNode node) {
    // Base case: leaf node is a number
    if (node.left == null && node.right == null) {
        return Integer.parseInt(node.data.toString());
    }

    int leftVal  = evaluateExpTree(node.left);
    int rightVal = evaluateExpTree(node.right);

    return switch (node.data.toString()) {
        case "+" -> leftVal + rightVal;
        case "-" -> leftVal - rightVal;
        case "*" -> leftVal * rightVal;
        case "/" -> leftVal / rightVal;
        default  -> throw new IllegalArgumentException("Unknown operator");
    };
}

// evaluateExpTree(root) for the tree above → (3+4)*(5-2) = 7*3 = 21
```

---

## 4.13 Complete Example — Tree Utilities Class

```java
import java.util.*;

public class BinaryTreeUtils {

    // ── Build example tree ─────────────────────────────────────────
    //           [1]
    //          /   \
    //        [2]   [3]
    //        / \   / \
    //      [4] [5][6] [7]
    public static TreeNode buildSampleTree() {
        TreeNode root       = new TreeNode(1);
        root.left           = new TreeNode(2);
        root.right          = new TreeNode(3);
        root.left.left      = new TreeNode(4);
        root.left.right     = new TreeNode(5);
        root.right.left     = new TreeNode(6);
        root.right.right    = new TreeNode(7);
        return root;
    }

    // ── All four traversals ────────────────────────────────────────
    public static List<Integer> preOrder(TreeNode node) {
        List<Integer> result = new ArrayList<>();
        preOrderHelper(node, result);
        return result;
    }
    private static void preOrderHelper(TreeNode node, List<Integer> result) {
        if (node == null) return;
        result.add(node.data);
        preOrderHelper(node.left, result);
        preOrderHelper(node.right, result);
    }

    public static List<Integer> inOrder(TreeNode node) {
        List<Integer> result = new ArrayList<>();
        inOrderHelper(node, result);
        return result;
    }
    private static void inOrderHelper(TreeNode node, List<Integer> result) {
        if (node == null) return;
        inOrderHelper(node.left, result);
        result.add(node.data);
        inOrderHelper(node.right, result);
    }

    public static List<Integer> postOrder(TreeNode node) {
        List<Integer> result = new ArrayList<>();
        postOrderHelper(node, result);
        return result;
    }
    private static void postOrderHelper(TreeNode node, List<Integer> result) {
        if (node == null) return;
        postOrderHelper(node.left, result);
        postOrderHelper(node.right, result);
        result.add(node.data);
    }

    public static List<Integer> levelOrder(TreeNode root) {
        List<Integer> result = new ArrayList<>();
        if (root == null) return result;
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        while (!queue.isEmpty()) {
            TreeNode node = queue.poll();
            result.add(node.data);
            if (node.left  != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        return result;
    }

    // ── Driver ─────────────────────────────────────────────────────
    public static void main(String[] args) {
        TreeNode root = buildSampleTree();

        System.out.println("Pre-order  : " + preOrder(root));   // [1, 2, 4, 5, 3, 6, 7]
        System.out.println("In-order   : " + inOrder(root));    // [4, 2, 5, 1, 6, 3, 7]
        System.out.println("Post-order : " + postOrder(root));  // [4, 5, 2, 6, 7, 3, 1]
        System.out.println("Level-order: " + levelOrder(root)); // [1, 2, 3, 4, 5, 6, 7]

        System.out.println("Height     : " + height(root));     // 2
        System.out.println("Node count : " + countNodes(root)); // 7
        System.out.println("Leaf count : " + countLeaves(root));// 4
        System.out.println("Balanced?  : " + isBalanced(root)); // true
    }
}
```

---

## 4.14 Key Takeaways

| Concept           | Summary                                                                    |
|-------------------|----------------------------------------------------------------------------|
| Binary Tree       | Each node has at most 2 children (left and right)                          |
| Height            | Longest root-to-leaf path; O(log n) for balanced, O(n) for degenerate      |
| Pre-order         | Root → Left → Right; used for copying, serializing trees                   |
| In-order          | Left → Root → Right; produces sorted output on a BST                       |
| Post-order        | Left → Right → Root; used for deletion, expression evaluation              |
| Level-order       | BFS using a queue; visits nodes level by level                             |
| Recursion + trees | Most tree algorithms are elegantly expressed recursively                   |
| Expression tree   | Leaves = operands, internal nodes = operators; traversal = notation form   |
| Balanced check    | Use sentinel (-1) to short-circuit and avoid redundant traversals          |

---

## 4.15 Practice Problems

1. **Path sum** — Given a binary tree and a target integer, return `true` if there exists a root-to-leaf path where all node values sum to the target.

2. **Maximum depth** — Find the maximum depth (height) of a binary tree. Then find the **minimum** depth (shortest path from root to any leaf).

3. **Symmetric tree** — Determine if a binary tree is a mirror of itself (symmetric around its center).

4. **Zigzag level order** — Perform level-order traversal but alternate direction each level: left-to-right on even levels, right-to-left on odd levels. *(Hint: use a deque.)*

5. **Serialize and deserialize** — Design an algorithm to convert a binary tree to a string (serialize) and reconstruct the tree from that string (deserialize). Pre-order traversal with null markers is a clean approach.

---

## 4.16 Further Reading

- *Data Structures and Algorithm Analysis in Java* — Weiss, Chapter 4
- *Introduction to Algorithms* (CLRS) — Chapter 12 (Binary Search Trees)
- Tree visualizer: [https://visualgo.net/en/bst](https://visualgo.net/en/bst)
- LeetCode tree problems (tag: Tree): [https://leetcode.com/tag/tree/](https://leetcode.com/tag/tree/)

---

*CS 3303 Data Structures — Module 4 of 8*
*Previous: Module 3 — Stacks and Queues*
*Next: Module 5 — Advanced Tree Structures and Recursion Techniques*
