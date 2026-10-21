# CS 3303 — Data Structures
## Module 3: Stacks and Queues

---

## 3.1 Introduction

Stacks and queues are **restricted linear data structures** — they are lists with rules about where elements can be added and removed. These rules make them simple, predictable, and incredibly powerful for solving a specific class of problems.

They appear everywhere in computing:
- Your browser's back button → **stack**
- A printer job queue → **queue**
- Function call management in every running program → **stack**
- Breadth-first search in graphs → **queue**

Understanding stacks and queues means understanding the shape of computation itself.

---

## 3.2 The Stack

### What is a Stack?

A **stack** is a Last-In, First-Out (LIFO) data structure. The last element added is the first one removed — like a stack of plates.

```
     push →  [  50  ]  ← top
             [  30  ]
             [  20  ]
             [  10  ]
             ────────
              (base)
```

You can only interact with the **top** of the stack. Everything below is inaccessible until the elements above it are removed.

### Core Stack Operations

| Operation    | Description                              | Time Complexity |
|--------------|------------------------------------------|-----------------|
| `push(x)`    | Add element x to the top                | O(1)            |
| `pop()`      | Remove and return the top element        | O(1)            |
| `peek()`     | Return top element without removing it   | O(1)            |
| `isEmpty()`  | Return true if the stack has no elements | O(1)            |
| `size()`     | Return the number of elements            | O(1)            |

Every operation is O(1) — stacks are extremely efficient.

---

## 3.3 Stack — Array-Based Implementation

```java
/**
 * A generic stack backed by a fixed-size array.
 * Uses a "top" index to track the current top position.
 */
public class ArrayStack<T> {

    private Object[] data;  // backing array
    private int top;        // index of the current top element
    private int capacity;

    public ArrayStack(int capacity) {
        this.capacity = capacity;
        this.data = new Object[capacity];
        this.top = -1; // -1 means empty
    }

    // ─────────────────────────────────────
    //  PUSH — O(1)
    // ─────────────────────────────────────
    /**
     * Add an element to the top of the stack.
     * Increment top first, then store the value.
     */
    public void push(T item) {
        if (top == capacity - 1) throw new RuntimeException("Stack overflow");
        data[++top] = item; // pre-increment: top becomes 0 on first push
    }

    // ─────────────────────────────────────
    //  POP — O(1)
    // ─────────────────────────────────────
    /**
     * Remove and return the top element.
     * Decrement top after reading to "erase" the element.
     */
    @SuppressWarnings("unchecked")
    public T pop() {
        if (isEmpty()) throw new RuntimeException("Stack underflow");
        T item = (T) data[top];
        data[top--] = null; // help garbage collection, post-decrement
        return item;
    }

    // ─────────────────────────────────────
    //  PEEK — O(1)
    // ─────────────────────────────────────
    @SuppressWarnings("unchecked")
    public T peek() {
        if (isEmpty()) throw new RuntimeException("Stack is empty");
        return (T) data[top];
    }

    public boolean isEmpty() { return top == -1; }
    public int size()        { return top + 1; }

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder("[");
        for (int i = 0; i <= top; i++) {
            sb.append(data[i]);
            if (i < top) sb.append(", ");
        }
        return sb.append("] ← top").toString();
    }
}
```

### Demo

```java
ArrayStack<Integer> stack = new ArrayStack<>(5);
stack.push(10);
stack.push(20);
stack.push(30);
System.out.println(stack);        // [10, 20, 30] ← top
System.out.println(stack.peek()); // 30
System.out.println(stack.pop());  // 30
System.out.println(stack);        // [10, 20] ← top
System.out.println(stack.size()); // 2
```

---

## 3.4 Stack — Linked List-Based Implementation

```java
/**
 * A generic stack backed by a singly linked list.
 * The head of the list serves as the top of the stack.
 * No fixed capacity — grows dynamically.
 */
public class LinkedStack<T> {

    private static class Node<T> {
        T data;
        Node<T> next;
        Node(T data) { this.data = data; }
    }

    private Node<T> top; // the top node (head of linked list)
    private int size;

    // PUSH — O(1): create new node, point to old top, update top
    public void push(T item) {
        Node<T> node = new Node<>(item);
        node.next = top;
        top = node;
        size++;
    }

    // POP — O(1): save top data, advance top to next, return data
    public T pop() {
        if (isEmpty()) throw new RuntimeException("Stack underflow");
        T item = top.data;
        top = top.next;
        size--;
        return item;
    }

    public T peek()      { 
        if (isEmpty()) throw new RuntimeException("Stack is empty");
        return top.data; 
    }
    public boolean isEmpty() { return top == null; }
    public int size()        { return size; }
}
```

### Array-Based vs. Linked List-Based Stack

| Feature            | Array Stack              | Linked Stack               |
|--------------------|--------------------------|----------------------------|
| Memory             | Contiguous (cache-friendly) | Scattered (pointer overhead) |
| Capacity           | Fixed (or must resize)   | Unlimited (heap-bound)     |
| Overhead           | None per element         | One pointer per element    |
| Speed              | Slightly faster (cache)  | Slightly slower (GC)       |
| Best when          | Max size is known        | Size is unpredictable      |

---

## 3.5 Java's Built-in Stack

Java provides `java.util.Stack<T>` (legacy) and the preferred `ArrayDeque<T>` as a stack:

```java
import java.util.ArrayDeque;
import java.util.Deque;

Deque<Integer> stack = new ArrayDeque<>();
stack.push(10);   // push to front
stack.push(20);
stack.push(30);
System.out.println(stack.peek()); // 30
System.out.println(stack.pop());  // 30
System.out.println(stack);        // [20, 10]
```

> **Best Practice:** Prefer `ArrayDeque` over `Stack` in Java. `Stack` extends `Vector` (a legacy synchronized class) and has poor performance in modern code.

---

## 3.6 Classic Stack Applications

### Application 1: Balanced Parentheses Checker

One of the most classic stack problems — verifying that every opening bracket has a matching closing bracket in the correct order.

```java
/**
 * Check if a string of brackets is balanced.
 *
 * Strategy:
 *   - Push every opening bracket onto the stack
 *   - When a closing bracket is seen, pop and verify it matches
 *   - At the end, the stack must be empty
 *
 * Examples:
 *   "({[]})"  → true
 *   "([)]"    → false
 *   "((("     → false
 *
 * Time: O(n)   Space: O(n)
 */
public static boolean isBalanced(String s) {
    Deque<Character> stack = new ArrayDeque<>();

    for (char c : s.toCharArray()) {
        if (c == '(' || c == '{' || c == '[') {
            stack.push(c); // push all openers
        } else if (c == ')' || c == '}' || c == ']') {
            if (stack.isEmpty()) return false; // closer with no opener

            char top = stack.pop();
            if ((c == ')' && top != '(') ||
                (c == '}' && top != '{') ||
                (c == ']' && top != '[')) {
                return false; // mismatched pair
            }
        }
    }
    return stack.isEmpty(); // must have no unmatched openers
}

// Tests:
System.out.println(isBalanced("({[]})"));  // true
System.out.println(isBalanced("([)]"));    // false
System.out.println(isBalanced("{{{"));     // false
```

---

### Application 2: Evaluate Postfix Expressions

In **postfix notation** (Reverse Polish Notation), operators come *after* their operands. `3 4 +` means `3 + 4`. No parentheses needed — and stacks evaluate it naturally.

```
Infix:   (3 + 4) * 2     → Postfix: 3 4 + 2 *
Infix:   5 + 3 * 2       → Postfix: 5 3 2 * +
```

```java
/**
 * Evaluate a postfix expression.
 *
 * Algorithm:
 *   - If token is a number → push it
 *   - If token is an operator → pop two operands, apply, push result
 *
 * Time: O(n)   Space: O(n)
 */
public static int evaluatePostfix(String[] tokens) {
    Deque<Integer> stack = new ArrayDeque<>();

    for (String token : tokens) {
        switch (token) {
            case "+" -> stack.push(stack.pop() + stack.pop());
            case "*" -> stack.push(stack.pop() * stack.pop());
            case "-" -> {
                int b = stack.pop();
                int a = stack.pop();
                stack.push(a - b); // order matters for subtraction
            }
            case "/" -> {
                int b = stack.pop();
                int a = stack.pop();
                stack.push(a / b); // order matters for division
            }
            default  -> stack.push(Integer.parseInt(token));
        }
    }
    return stack.pop();
}

// Test: "3 4 + 2 *" = (3+4)*2 = 14
System.out.println(evaluatePostfix(new String[]{"3","4","+","2","*"})); // 14
```

---

### Application 3: Undo/Redo with Two Stacks

```java
/**
 * Simple text editor with undo/redo functionality.
 * Uses two stacks: one for undo history, one for redo history.
 */
public class TextEditor {
    private StringBuilder text = new StringBuilder();
    private Deque<String> undoStack = new ArrayDeque<>();
    private Deque<String> redoStack = new ArrayDeque<>();

    public void type(String s) {
        undoStack.push(text.toString()); // save current state
        redoStack.clear();               // new action clears redo history
        text.append(s);
    }

    public void undo() {
        if (undoStack.isEmpty()) { System.out.println("Nothing to undo"); return; }
        redoStack.push(text.toString()); // save for redo
        text = new StringBuilder(undoStack.pop());
    }

    public void redo() {
        if (redoStack.isEmpty()) { System.out.println("Nothing to redo"); return; }
        undoStack.push(text.toString());
        text = new StringBuilder(redoStack.pop());
    }

    public String getText() { return text.toString(); }
}

// Demo:
TextEditor editor = new TextEditor();
editor.type("Hello");
editor.type(", World");
System.out.println(editor.getText()); // Hello, World
editor.undo();
System.out.println(editor.getText()); // Hello
editor.redo();
System.out.println(editor.getText()); // Hello, World
```

---

## 3.7 The Queue

### What is a Queue?

A **queue** is a First-In, First-Out (FIFO) data structure. The first element added is the first one removed — like a line of people at a ticket booth.

```
enqueue →  [ 10 | 20 | 30 | 40 ]  → dequeue
           rear                front
```

New elements join at the **rear**. Elements leave from the **front**.

### Core Queue Operations

| Operation       | Description                                 | Time Complexity |
|-----------------|---------------------------------------------|-----------------|
| `enqueue(x)`    | Add element x to the rear                   | O(1)            |
| `dequeue()`     | Remove and return the front element         | O(1)            |
| `peek()`        | Return front element without removing it    | O(1)            |
| `isEmpty()`     | Return true if the queue is empty           | O(1)            |
| `size()`        | Return number of elements                   | O(1)            |

---

## 3.8 Queue — Linked List-Based Implementation

```java
/**
 * A generic queue backed by a singly linked list.
 * head = front (dequeue here)
 * tail = rear  (enqueue here)
 *
 * Both operations are O(1) because we maintain pointers to both ends.
 */
public class LinkedQueue<T> {

    private static class Node<T> {
        T data;
        Node<T> next;
        Node(T data) { this.data = data; }
    }

    private Node<T> head; // front — dequeue from here
    private Node<T> tail; // rear  — enqueue here
    private int size;

    // ENQUEUE — O(1): add to tail
    public void enqueue(T item) {
        Node<T> node = new Node<>(item);
        if (tail != null) tail.next = node;
        tail = node;
        if (head == null) head = node; // first element
        size++;
    }

    // DEQUEUE — O(1): remove from head
    public T dequeue() {
        if (isEmpty()) throw new RuntimeException("Queue is empty");
        T item = head.data;
        head = head.next;
        if (head == null) tail = null; // queue is now empty
        size--;
        return item;
    }

    public T peek()          { 
        if (isEmpty()) throw new RuntimeException("Queue is empty");
        return head.data; 
    }
    public boolean isEmpty() { return head == null; }
    public int size()        { return size; }

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder("front → [");
        Node<T> current = head;
        while (current != null) {
            sb.append(current.data);
            if (current.next != null) sb.append(", ");
            current = current.next;
        }
        return sb.append("] ← rear").toString();
    }
}
```

### Demo

```java
LinkedQueue<String> queue = new LinkedQueue<>();
queue.enqueue("Alice");
queue.enqueue("Bob");
queue.enqueue("Carol");
System.out.println(queue);           // front → [Alice, Bob, Carol] ← rear
System.out.println(queue.dequeue()); // Alice
System.out.println(queue);           // front → [Bob, Carol] ← rear
System.out.println(queue.peek());    // Bob
```

---

## 3.9 Circular Queue (Ring Buffer)

A naive array-based queue wastes space: after many enqueue/dequeue operations, the front index drifts right and the start of the array is unused.

A **circular queue** solves this by treating the array as a ring — when the rear reaches the end, it wraps around to index 0.

```
  [  ] [10] [20] [30] [  ] [  ]
        ↑                ↑
       front            rear

After wrap-around:
  [50] [10] [20] [30] [40] [  ]
        ↑                      ↑
       front                  rear (wrapped to front of array)
```

```java
/**
 * A circular queue using a fixed-size array.
 * Uses modulo arithmetic to wrap indices.
 */
public class CircularQueue<T> {
    private Object[] data;
    private int front, rear, size, capacity;

    public CircularQueue(int capacity) {
        this.capacity = capacity;
        this.data = new Object[capacity];
        this.front = 0;
        this.rear = 0;
        this.size = 0;
    }

    // ENQUEUE — O(1)
    public void enqueue(T item) {
        if (size == capacity) throw new RuntimeException("Queue full");
        data[rear] = item;
        rear = (rear + 1) % capacity; // wrap around using modulo
        size++;
    }

    // DEQUEUE — O(1)
    @SuppressWarnings("unchecked")
    public T dequeue() {
        if (isEmpty()) throw new RuntimeException("Queue empty");
        T item = (T) data[front];
        data[front] = null;
        front = (front + 1) % capacity; // wrap around
        size--;
        return item;
    }

    @SuppressWarnings("unchecked")
    public T peek()          { return (T) data[front]; }
    public boolean isEmpty() { return size == 0; }
    public boolean isFull()  { return size == capacity; }
    public int size()        { return size; }
}
```

---

## 3.10 Deque (Double-Ended Queue)

A **deque** (pronounced "deck") allows insertion and deletion at **both** ends.

```
addFirst →  [ 10 | 20 | 30 | 40 ]  ← addLast
removeFirst → front           rear → removeLast
```

This generalizes both stacks (use only one end) and queues (add to one end, remove from the other).

```java
import java.util.ArrayDeque;
import java.util.Deque;

Deque<Integer> deque = new ArrayDeque<>();

deque.addFirst(20);   // [20]
deque.addFirst(10);   // [10, 20]
deque.addLast(30);    // [10, 20, 30]
deque.addLast(40);    // [10, 20, 30, 40]

System.out.println(deque.removeFirst()); // 10  → [20, 30, 40]
System.out.println(deque.removeLast());  // 40  → [20, 30]
System.out.println(deque.peekFirst());   // 20
System.out.println(deque.peekLast());    // 30
```

---

## 3.11 Priority Queue

A **priority queue** is a queue where each element has a **priority**, and the element with the highest (or lowest) priority is always dequeued first — regardless of insertion order.

```
Enqueue: (task="email", priority=1), (task="fire alarm", priority=10), (task="meeting", priority=3)
Dequeue order: fire alarm(10) → meeting(3) → email(1)
```

Java's `PriorityQueue` is a **min-heap** by default (smallest value = highest priority).

```java
import java.util.PriorityQueue;

// Min-priority queue (default) — smallest value dequeued first
PriorityQueue<Integer> minPQ = new PriorityQueue<>();
minPQ.offer(30);
minPQ.offer(10);
minPQ.offer(20);
System.out.println(minPQ.poll()); // 10 (smallest first)
System.out.println(minPQ.poll()); // 20
System.out.println(minPQ.poll()); // 30

// Max-priority queue — largest value dequeued first
PriorityQueue<Integer> maxPQ = new PriorityQueue<>(java.util.Collections.reverseOrder());
maxPQ.offer(30);
maxPQ.offer(10);
maxPQ.offer(20);
System.out.println(maxPQ.poll()); // 30 (largest first)

// Custom priority with Comparator (e.g., task scheduling)
PriorityQueue<int[]> taskQueue = new PriorityQueue<>((a, b) -> b[1] - a[1]); // higher priority first
taskQueue.offer(new int[]{1, 3}); // task 1, priority 3
taskQueue.offer(new int[]{2, 7}); // task 2, priority 7
taskQueue.offer(new int[]{3, 1}); // task 3, priority 1
int[] next = taskQueue.poll();
System.out.println("Next task: " + next[0]); // task 2 (priority 7)
```

> Priority queues are implemented with **heaps** under the hood — covered in depth in Module 5.

---

## 3.12 Classic Queue Applications

### Application 1: Level-Order Tree Traversal (BFS)

```java
/**
 * Print all nodes of a binary tree level by level.
 * A queue naturally processes nodes in the order they were discovered.
 *
 * Time: O(n)   Space: O(n)
 */
public static void levelOrder(TreeNode root) {
    if (root == null) return;
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);

    while (!queue.isEmpty()) {
        int levelSize = queue.size(); // number of nodes at current level
        for (int i = 0; i < levelSize; i++) {
            TreeNode node = queue.poll();
            System.out.print(node.val + " ");
            if (node.left  != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        System.out.println(); // newline between levels
    }
}
```

### Application 2: Hot Potato (Round-Robin Simulation)

```java
/**
 * Simulate the "hot potato" game using a queue.
 * Players pass a potato around; after k passes, the holder is eliminated.
 * Last player remaining wins.
 *
 * Time: O(n * k)
 */
public static String hotPotato(String[] names, int k) {
    Queue<String> queue = new LinkedList<>(java.util.Arrays.asList(names));

    while (queue.size() > 1) {
        // Pass the potato k times
        for (int i = 0; i < k; i++) {
            queue.offer(queue.poll()); // move front to back
        }
        System.out.println(queue.poll() + " is eliminated!"); // k-th person out
    }
    return queue.poll(); // last person standing
}

// Test:
String winner = hotPotato(new String[]{"Alice","Bob","Carol","Dave","Eve"}, 3);
System.out.println("Winner: " + winner);
```

---

## 3.13 Implementing a Queue Using Two Stacks

A classic interview problem — implement a queue using only two stacks.

```java
/**
 * Queue implemented with two stacks.
 *
 * Idea:
 *   stack1 = inbox  (all new elements go here)
 *   stack2 = outbox (elements come out from here)
 *
 *   When outbox is empty and we need to dequeue:
 *     pour everything from inbox into outbox (reverses the order)
 *
 * Amortized O(1) per operation — each element moves between stacks at most once.
 */
public class QueueFromTwoStacks<T> {
    private Deque<T> inbox  = new ArrayDeque<>(); // for enqueue
    private Deque<T> outbox = new ArrayDeque<>(); // for dequeue

    // ENQUEUE — always push to inbox
    public void enqueue(T item) {
        inbox.push(item);
    }

    // DEQUEUE — pull from outbox; refill from inbox when outbox is empty
    public T dequeue() {
        if (outbox.isEmpty()) {
            while (!inbox.isEmpty()) {
                outbox.push(inbox.pop()); // pour inbox → outbox (reverses order)
            }
        }
        if (outbox.isEmpty()) throw new RuntimeException("Queue is empty");
        return outbox.pop();
    }

    public T peek() {
        if (outbox.isEmpty()) {
            while (!inbox.isEmpty()) outbox.push(inbox.pop());
        }
        return outbox.peek();
    }

    public boolean isEmpty() { return inbox.isEmpty() && outbox.isEmpty(); }
}

// Demo:
QueueFromTwoStacks<Integer> q = new QueueFromTwoStacks<>();
q.enqueue(1); q.enqueue(2); q.enqueue(3);
System.out.println(q.dequeue()); // 1 (FIFO)
q.enqueue(4);
System.out.println(q.dequeue()); // 2
System.out.println(q.dequeue()); // 3
System.out.println(q.dequeue()); // 4
```

---

## 3.14 Stacks vs. Queues — Summary Comparison

| Feature             | Stack (LIFO)                   | Queue (FIFO)                      |
|---------------------|--------------------------------|-----------------------------------|
| Order               | Last in, first out             | First in, first out               |
| Add element         | `push` to top                  | `enqueue` to rear                 |
| Remove element      | `pop` from top                 | `dequeue` from front              |
| Real-world analogy  | Stack of plates                | Line at a ticket booth            |
| Common uses         | Undo, parsing, DFS, call stack | BFS, scheduling, buffering        |
| Java class          | `ArrayDeque` (as stack)        | `ArrayDeque` or `LinkedList`      |
| All operations      | O(1)                           | O(1)                              |

---

## 3.15 Key Takeaways

| Concept              | Summary                                                                  |
|----------------------|--------------------------------------------------------------------------|
| Stack (LIFO)         | Add/remove only from top; O(1) push, pop, peek                          |
| Queue (FIFO)         | Add to rear, remove from front; O(1) enqueue, dequeue, peek             |
| Array Stack          | Fixed capacity, cache-friendly                                           |
| Linked Stack/Queue   | Dynamic size, slight pointer overhead                                    |
| Circular Queue       | Array-based queue using modulo to avoid wasted space                     |
| Deque                | Double-ended; generalizes both stack and queue                           |
| Priority Queue       | Dequeues by priority (min or max); backed by a heap                     |
| Balanced Parentheses | Classic stack application — O(n) with one pass                          |
| Postfix Evaluation   | Stack processes operands and operators naturally                         |
| Queue from 2 Stacks  | Classic interview problem; amortized O(1)                               |

---

## 3.16 Practice Problems

1. **Min Stack** — Design a stack that supports `push`, `pop`, `peek`, and `getMin` (returns the minimum element in the stack) — all in O(1). *(Hint: use a second stack to track minimums.)*

2. **Valid brackets** — Extend the balanced parentheses checker to also handle HTML-style tags like `<div>` and `</div>`.

3. **Stack sort** — Sort a stack using only one additional stack (no arrays). The sorted stack should have the smallest element on top.

4. **Sliding window maximum** — Given an array and window size `k`, find the maximum in each window of size `k` as it slides across the array. Use a deque to achieve O(n). *(Hint: maintain a deque of indices in decreasing order of value.)*

5. **Implement a stack using two queues** — The inverse of Section 3.13. Which operations suffer in this direction, and why?

---

## 3.17 Further Reading

- *Data Structures and Algorithm Analysis in Java* — Weiss, Chapter 3
- Java Deque API: [https://docs.oracle.com/en/java/se/17/docs/api/java.base/java/util/Deque.html](https://docs.oracle.com/en/java/se/17/docs/api/java.base/java/util/Deque.html)
- Java PriorityQueue API: [https://docs.oracle.com/en/java/se/17/docs/api/java.base/java/util/PriorityQueue.html](https://docs.oracle.com/en/java/se/17/docs/api/java.base/java/util/PriorityQueue.html)
- Visualizer: [https://visualgo.net/en/list](https://visualgo.net/en/list)

---

*CS 3303 Data Structures — Module 3 of 8*
*Previous: Module 2 — Arrays and Linked Lists*
*Next: Module 4 — Binary Trees and Tree Traversals*
