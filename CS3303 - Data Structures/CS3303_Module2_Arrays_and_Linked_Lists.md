# CS 3303 — Data Structures
## Module 2: Arrays and Linked Lists

---

## 2.1 Introduction

Arrays and linked lists are the two most fundamental data structures in computer science. Nearly every other structure — stacks, queues, trees, hash tables — is built on top of one of these two. Understanding them deeply means understanding *why* we make design trade-offs in almost every program we write.

Both are **linear** data structures: they store elements in a sequence. But they differ fundamentally in *how* that sequence is stored in memory.

---

## 2.2 Arrays

### What is an Array?

An **array** is a fixed-size, contiguous block of memory that stores elements of the same type, accessed by index.

```
Index:  [0]  [1]  [2]  [3]  [4]
Value:  [ 10] [20] [30] [40] [50]
         ↑
     base address
     each slot is adjacent in memory
```

Because all elements are adjacent in memory, accessing any element by index is a single arithmetic operation:

```
address of arr[i] = base_address + (i × element_size)
```

This is why array access is **O(1)** — it's just arithmetic, no searching needed.

### Declaring Arrays in Java

```java
// Method 1: Declare with a fixed size
int[] scores = new int[5];          // [0, 0, 0, 0, 0]

// Method 2: Declare and initialize
int[] primes = {2, 3, 5, 7, 11};   // size inferred as 5

// Method 3: Declare first, initialize later
String[] names;
names = new String[3];
names[0] = "Alice";
names[1] = "Bob";
names[2] = "Carol";

// 2D Array
int[][] matrix = new int[3][4];     // 3 rows, 4 columns
matrix[1][2] = 99;
```

### Array Operations and Their Complexity

| Operation             | Time Complexity | Notes                                    |
|-----------------------|-----------------|------------------------------------------|
| Access by index       | O(1)            | Direct memory calculation                |
| Search (unsorted)     | O(n)            | Must check every element                 |
| Search (sorted)       | O(log n)        | Binary search possible                   |
| Insert at end         | O(1)*           | *Only if space available                 |
| Insert at middle/front| O(n)            | Must shift elements right                |
| Delete at middle/front| O(n)            | Must shift elements left                 |
| Resize                | O(n)            | Must copy all elements to new array      |

---

## 2.3 Core Array Operations in Java

### Traversal

```java
/**
 * Traverse an array and print each element.
 * Time Complexity: O(n)
 */
public static void traverseArray(int[] arr) {
    // Method 1: index-based
    for (int i = 0; i < arr.length; i++) {
        System.out.print(arr[i] + " ");
    }

    System.out.println();

    // Method 2: enhanced for-loop (when index not needed)
    for (int val : arr) {
        System.out.print(val + " ");
    }
}
```

### Insertion (Shifting Elements)

```java
/**
 * Insert a value at a given index by shifting elements right.
 * The last element is lost if the array is full.
 * Time Complexity: O(n)
 */
public static void insertAt(int[] arr, int index, int value) {
    // Shift elements to the right, starting from the end
    for (int i = arr.length - 1; i > index; i--) {
        arr[i] = arr[i - 1];
    }
    arr[index] = value;
}
```

### Deletion (Shifting Elements)

```java
/**
 * Delete the element at a given index by shifting elements left.
 * The last slot is zeroed out.
 * Time Complexity: O(n)
 */
public static void deleteAt(int[] arr, int index) {
    // Shift elements to the left
    for (int i = index; i < arr.length - 1; i++) {
        arr[i] = arr[i + 1];
    }
    arr[arr.length - 1] = 0; // clear last slot
}
```

### Searching

```java
/**
 * Linear Search — works on any array.
 * Time Complexity: O(n)
 */
public static int linearSearch(int[] arr, int target) {
    for (int i = 0; i < arr.length; i++) {
        if (arr[i] == target) return i;
    }
    return -1; // not found
}

/**
 * Binary Search — requires a SORTED array.
 * Time Complexity: O(log n)
 */
public static int binarySearch(int[] arr, int target) {
    int left = 0, right = arr.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] == target)      return mid;
        else if (arr[mid] < target)  left = mid + 1;
        else                         right = mid - 1;
    }
    return -1;
}
```

---

## 2.4 Dynamic Arrays: ArrayList

Java's built-in array has a fixed size. For a resizable array, Java provides `ArrayList<T>` — a **dynamic array** that grows automatically.

### How Dynamic Arrays Work

When the internal array is full and a new element is added:
1. Allocate a new array, typically **2× the old size**
2. Copy all elements to the new array
3. Insert the new element

This resizing costs O(n), but happens rarely. The **amortized** cost per insertion is still O(1).

```
Initial:  [10 | 20 | 30 | 40]  capacity=4, size=4
Add 50:   [10 | 20 | 30 | 40 | 50 |    |    |    ]  capacity=8, size=5
```

### ArrayList in Java

```java
import java.util.ArrayList;

public class DynamicArrayDemo {
    public static void main(String[] args) {
        ArrayList<Integer> list = new ArrayList<>();

        // Add elements — O(1) amortized
        list.add(10);
        list.add(20);
        list.add(30);

        // Insert at specific index — O(n)
        list.add(1, 15);  // [10, 15, 20, 30]

        // Access by index — O(1)
        System.out.println(list.get(2));  // 20

        // Remove by index — O(n)
        list.remove(1);  // removes 15 → [10, 20, 30]

        // Remove by value — O(n)
        list.remove(Integer.valueOf(20));  // [10, 30]

        // Size
        System.out.println(list.size());  // 2

        // Iterate
        for (int val : list) {
            System.out.println(val);
        }
    }
}
```

---

## 2.5 Linked Lists

### The Problem with Arrays

Arrays are excellent for random access but poor at insertions and deletions in the middle, because elements must be shifted. What if we need a structure that excels at insertion and deletion?

### What is a Linked List?

A **linked list** is a sequence of **nodes**, where each node contains:
1. **Data** — the value stored
2. **Next** — a reference (pointer) to the next node

```
  head
   ↓
[10 | •]──→[20 | •]──→[30 | •]──→[40 | null]
```

Unlike an array, nodes are **not** stored in contiguous memory. They are scattered throughout the heap, connected only by pointers.

### The Node Class

```java
/**
 * A single node in a singly linked list.
 * Generic type T allows any data type.
 */
public class Node<T> {
    T data;       // the value stored in this node
    Node<T> next; // reference to the next node (null if last)

    // Constructor
    public Node(T data) {
        this.data = data;
        this.next = null;
    }
}
```

---

## 2.6 Singly Linked List — Full Implementation

```java
/**
 * A generic singly linked list implementation.
 * Supports add, insert, delete, search, and traversal.
 */
public class LinkedList<T> {

    private Node<T> head; // first node in the list
    private int size;

    public LinkedList() {
        head = null;
        size = 0;
    }

    // ─────────────────────────────────────────────
    //  ADD TO FRONT — O(1)
    // ─────────────────────────────────────────────
    /**
     * Inserts a new node at the beginning of the list.
     * Simply point new node's next to the current head, then update head.
     */
    public void addFirst(T data) {
        Node<T> newNode = new Node<>(data);
        newNode.next = head; // new node points to old head
        head = newNode;      // head now points to new node
        size++;
    }

    // ─────────────────────────────────────────────
    //  ADD TO END — O(n)
    // ─────────────────────────────────────────────
    /**
     * Inserts a new node at the end of the list.
     * Must traverse all n nodes to find the last one.
     */
    public void addLast(T data) {
        Node<T> newNode = new Node<>(data);
        if (head == null) {
            head = newNode;
        } else {
            Node<T> current = head;
            while (current.next != null) { // walk to the last node
                current = current.next;
            }
            current.next = newNode; // attach new node at the end
        }
        size++;
    }

    // ─────────────────────────────────────────────
    //  INSERT AT INDEX — O(n)
    // ─────────────────────────────────────────────
    /**
     * Inserts a new node at the specified index.
     * Index 0 uses addFirst(); otherwise traverse to (index-1).
     */
    public void insertAt(int index, T data) {
        if (index < 0 || index > size) throw new IndexOutOfBoundsException();
        if (index == 0) { addFirst(data); return; }

        Node<T> current = head;
        for (int i = 0; i < index - 1; i++) { // stop one before target
            current = current.next;
        }
        Node<T> newNode = new Node<>(data);
        newNode.next = current.next; // new node points to what was at index
        current.next = newNode;      // previous node points to new node
        size++;
    }

    // ─────────────────────────────────────────────
    //  DELETE BY VALUE — O(n)
    // ─────────────────────────────────────────────
    /**
     * Removes the first node that contains the given value.
     * Key insight: we need the node BEFORE the target to re-link.
     */
    public boolean delete(T data) {
        if (head == null) return false;

        // Special case: target is the head
        if (head.data.equals(data)) {
            head = head.next;
            size--;
            return true;
        }

        // Find node just before the target
        Node<T> current = head;
        while (current.next != null) {
            if (current.next.data.equals(data)) {
                current.next = current.next.next; // skip the target node
                size--;
                return true;
            }
            current = current.next;
        }
        return false; // not found
    }

    // ─────────────────────────────────────────────
    //  SEARCH — O(n)
    // ─────────────────────────────────────────────
    /**
     * Returns the index of the first node containing the value,
     * or -1 if not found.
     */
    public int search(T data) {
        Node<T> current = head;
        int index = 0;
        while (current != null) {
            if (current.data.equals(data)) return index;
            current = current.next;
            index++;
        }
        return -1;
    }

    // ─────────────────────────────────────────────
    //  PRINT — O(n)
    // ─────────────────────────────────────────────
    public void print() {
        Node<T> current = head;
        while (current != null) {
            System.out.print(current.data);
            if (current.next != null) System.out.print(" → ");
            current = current.next;
        }
        System.out.println(" → null");
    }

    public int size() { return size; }
}
```

### Demo

```java
public class LinkedListDemo {
    public static void main(String[] args) {
        LinkedList<Integer> list = new LinkedList<>();

        list.addLast(10);
        list.addLast(20);
        list.addLast(30);
        list.addFirst(5);
        list.print();             // 5 → 10 → 20 → 30 → null

        list.insertAt(2, 15);
        list.print();             // 5 → 10 → 15 → 20 → 30 → null

        list.delete(15);
        list.print();             // 5 → 10 → 20 → 30 → null

        System.out.println(list.search(20));  // 2
        System.out.println(list.size());       // 4
    }
}
```

---

## 2.7 Doubly Linked List

A **doubly linked list** adds a `prev` pointer to each node, allowing traversal in both directions.

```
null ←[5 | •]⇄[10 | •]⇄[20 | •]⇄[30 | null]
```

```java
/**
 * A node in a doubly linked list.
 */
public class DoublyNode<T> {
    T data;
    DoublyNode<T> next;
    DoublyNode<T> prev; // NEW: pointer to previous node

    public DoublyNode(T data) {
        this.data = data;
        this.next = null;
        this.prev = null;
    }
}
```

### Doubly Linked List — Add and Delete

```java
public class DoublyLinkedList<T> {
    private DoublyNode<T> head;
    private DoublyNode<T> tail; // NEW: direct access to last node

    /**
     * Add to end — O(1) because we track the tail directly
     */
    public void addLast(T data) {
        DoublyNode<T> newNode = new DoublyNode<>(data);
        if (tail == null) {
            head = tail = newNode;
        } else {
            newNode.prev = tail;  // new node's prev points to old tail
            tail.next = newNode;  // old tail's next points to new node
            tail = newNode;       // update tail
        }
    }

    /**
     * Delete a node — O(1) if we already have the node reference!
     * This is the key advantage of doubly linked lists.
     */
    public void deleteNode(DoublyNode<T> node) {
        if (node.prev != null) node.prev.next = node.next;
        else head = node.next;           // node was the head

        if (node.next != null) node.next.prev = node.prev;
        else tail = node.prev;           // node was the tail
    }

    /**
     * Traverse forward
     */
    public void printForward() {
        DoublyNode<T> current = head;
        while (current != null) {
            System.out.print(current.data + " ⇄ ");
            current = current.next;
        }
        System.out.println("null");
    }

    /**
     * Traverse backward (unique to doubly linked lists)
     */
    public void printBackward() {
        DoublyNode<T> current = tail;
        while (current != null) {
            System.out.print(current.data + " ⇄ ");
            current = current.prev;
        }
        System.out.println("null");
    }
}
```

---

## 2.8 Circular Linked List

In a **circular linked list**, the last node's `next` pointer points back to the head instead of `null`.

```
      ┌──────────────────────┐
      ↓                      |
   [10]→[20]→[30]→[40]───────┘
```

This is useful for round-robin scheduling, playlists, and buffering.

```java
/**
 * Traverse a circular linked list.
 * Must stop when we return to the head — NOT when we see null.
 */
public void printCircular(Node<Integer> head) {
    if (head == null) return;
    Node<Integer> current = head;
    do {
        System.out.print(current.data + " → ");
        current = current.next;
    } while (current != head); // stop when we've gone full circle
    System.out.println("(back to head)");
}
```

---

## 2.9 Arrays vs. Linked Lists — Side-by-Side Comparison

| Feature                | Array                          | Linked List                     |
|------------------------|--------------------------------|---------------------------------|
| Memory layout          | Contiguous                     | Non-contiguous (scattered)      |
| Access by index        | O(1) — direct                  | O(n) — must traverse            |
| Search                 | O(n) / O(log n) sorted         | O(n)                            |
| Insert at front        | O(n) — shift needed            | O(1) — update head pointer      |
| Insert at middle       | O(n) — shift needed            | O(n) to find position, O(1) link|
| Insert at end          | O(1) amortized (ArrayList)     | O(1) with tail pointer          |
| Delete at front        | O(n) — shift needed            | O(1) — update head pointer      |
| Delete at middle       | O(n) — shift needed            | O(n) to find, O(1) relink       |
| Memory overhead        | None (just data)               | Extra pointer per node          |
| Cache performance      | Excellent (contiguous)         | Poor (pointer chasing)          |
| Size flexibility       | Fixed (or expensive to resize) | Dynamic — grows naturally       |

### When to Use Each

**Use an Array (or ArrayList) when:**
- You need frequent random access by index
- Data size is known or changes rarely
- Cache performance matters (tight loops, numerical processing)
- Memory is constrained (no pointer overhead)

**Use a Linked List when:**
- You frequently insert/delete at the front
- You build stacks and queues (covered in Module 3)
- Data size is unpredictable and changes frequently
- You need O(1) insertion when you already have a reference to the position

---

## 2.10 Classic Interview Problems

### Problem 1: Reverse a Linked List

```java
/**
 * Reverse a singly linked list in-place.
 * Strategy: walk forward, reversing each pointer.
 *
 * Before: 1 → 2 → 3 → 4 → null
 * After:  4 → 3 → 2 → 1 → null
 *
 * Time: O(n)   Space: O(1)
 */
public static Node<Integer> reverse(Node<Integer> head) {
    Node<Integer> prev = null;
    Node<Integer> current = head;

    while (current != null) {
        Node<Integer> next = current.next; // save next before overwriting
        current.next = prev;              // reverse the pointer
        prev = current;                   // move prev forward
        current = next;                   // move current forward
    }
    return prev; // prev is now the new head
}
```

**Trace through `1 → 2 → 3 → null`:**

```
Step 1: prev=null, curr=1 → curr.next=null, prev=1, curr=2
Step 2: prev=1,    curr=2 → curr.next=1,    prev=2, curr=3
Step 3: prev=2,    curr=3 → curr.next=2,    prev=3, curr=null
Return: prev=3 → new head
Result: 3 → 2 → 1 → null ✓
```

---

### Problem 2: Detect a Cycle (Floyd's Algorithm)

```java
/**
 * Detect if a linked list has a cycle using Floyd's Tortoise and Hare.
 *
 * Two pointers move at different speeds:
 *   - slow moves 1 step at a time
 *   - fast moves 2 steps at a time
 *
 * If there's a cycle, fast will eventually lap slow and they'll meet.
 * If there's no cycle, fast reaches null first.
 *
 * Time: O(n)   Space: O(1)
 */
public static boolean hasCycle(Node<Integer> head) {
    Node<Integer> slow = head;
    Node<Integer> fast = head;

    while (fast != null && fast.next != null) {
        slow = slow.next;       // move 1 step
        fast = fast.next.next;  // move 2 steps
        if (slow == fast) return true; // they met → cycle exists
    }
    return false; // fast reached end → no cycle
}
```

---

### Problem 3: Find the Middle Node

```java
/**
 * Find the middle node of a linked list in one pass.
 * Uses the same two-pointer idea: when fast reaches the end,
 * slow is at the middle.
 *
 * Time: O(n)   Space: O(1)
 */
public static Node<Integer> findMiddle(Node<Integer> head) {
    Node<Integer> slow = head;
    Node<Integer> fast = head;

    while (fast != null && fast.next != null) {
        slow = slow.next;       // 1 step
        fast = fast.next.next;  // 2 steps
    }
    return slow; // slow is at the middle
}
```

---

### Problem 4: Merge Two Sorted Arrays

```java
/**
 * Merge two sorted arrays into a single sorted array.
 * Classic two-pointer technique.
 *
 * Time: O(n + m)   Space: O(n + m)
 */
public static int[] mergeSorted(int[] a, int[] b) {
    int[] result = new int[a.length + b.length];
    int i = 0, j = 0, k = 0;

    while (i < a.length && j < b.length) {
        if (a[i] <= b[j]) result[k++] = a[i++];
        else               result[k++] = b[j++];
    }

    // Copy remaining elements from whichever array isn't exhausted
    while (i < a.length) result[k++] = a[i++];
    while (j < b.length) result[k++] = b[j++];

    return result;
}

// Test:
// mergeSorted([1,3,5], [2,4,6]) → [1,2,3,4,5,6]
```

---

## 2.11 Java's Built-in List Options

| Class              | Backed by         | Best for                              |
|--------------------|-------------------|---------------------------------------|
| `ArrayList<T>`     | Dynamic array     | Random access, iteration              |
| `LinkedList<T>`    | Doubly linked list| Frequent insert/delete at ends        |
| `ArrayDeque<T>`    | Circular array    | Stack/Queue operations (fastest)      |

```java
import java.util.LinkedList;

LinkedList<String> ll = new LinkedList<>();
ll.addFirst("A");   // O(1)
ll.addLast("B");    // O(1)
ll.addLast("C");    // O(1)
ll.removeFirst();   // O(1) → removes "A"
System.out.println(ll);  // [B, C]
```

---

## 2.12 Key Takeaways

| Concept                      | Summary                                                                 |
|------------------------------|-------------------------------------------------------------------------|
| Array                        | Fixed-size, contiguous, O(1) access by index                           |
| ArrayList                    | Resizable array; O(1) amortized append, O(n) insert/delete at middle   |
| Singly Linked List           | Nodes with `next` pointer; O(1) insert at front, O(n) access by index  |
| Doubly Linked List           | Nodes with `prev` and `next`; O(1) delete if you have the node         |
| Circular Linked List         | Last node points to head; useful for cyclic tasks                       |
| Array vs. Linked List        | Arrays win on access; lists win on insert/delete at front               |
| Floyd's Algorithm            | Detects cycles in O(n) with two pointers                                |
| Two-Pointer Technique        | Powerful pattern for middle, merge, and cycle problems                  |

---

## 2.13 Practice Problems

1. **Rotate an array** — Write a method that rotates an array of size `n` to the right by `k` positions in O(n) time and O(1) space.

2. **Remove duplicates** — Given a sorted linked list, remove all duplicate values so each value appears only once. (`1→1→2→3→3` becomes `1→2→3`)

3. **Nth from end** — Find the nth node from the end of a linked list in a single pass (without knowing the length). *(Hint: two-pointer)*

4. **Palindrome check** — Determine if a linked list is a palindrome in O(n) time and O(1) space.

5. **Array vs. List** — Given a use case where 10 million records are loaded once and then queried by index 99% of the time, justify which structure to use and why.

---

## 2.14 Further Reading

- *Data Structures and Algorithm Analysis in Java* — Weiss, Chapter 3
- Java LinkedList API: [https://docs.oracle.com/en/java/se/17/docs/api/java.base/java/util/LinkedList.html](https://docs.oracle.com/en/java/se/17/docs/api/java.base/java/util/LinkedList.html)
- Java ArrayList API: [https://docs.oracle.com/en/java/se/17/docs/api/java.base/java/util/ArrayList.html](https://docs.oracle.com/en/java/se/17/docs/api/java.base/java/util/ArrayList.html)
- Visualizer for linked list operations: [https://visualgo.net/en/list](https://visualgo.net/en/list)

---

*CS 3303 Data Structures — Module 2 of 8*
*Previous: Module 1 — Fundamentals of Data Structures and Algorithm Analysis*
*Next: Module 3 — Stacks and Queues*
