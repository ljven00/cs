# CS 2301 — Operating Systems 1
## Module 3: Concurrency

---

> **Course Progress:** Module 3 of 8 — CS 2301 (OS 1)

---

## 1. What is Concurrency?

**Concurrency** is when multiple tasks make progress during overlapping time periods. This doesn't necessarily mean they run at the exact same instant — on a single CPU, the OS rapidly switches between them, creating the *illusion* of simultaneous execution.

On multi-core systems, tasks can run truly simultaneously — this is called **parallelism**.

| Term | Meaning |
|------|---------|
| **Concurrency** | Multiple tasks in progress at overlapping times |
| **Parallelism** | Multiple tasks running at the exact same instant (multi-core) |

Concurrency is a feature — but it introduces problems that don't exist when only one thing runs at a time.

---

## 2. Threads

A **thread** is the smallest unit of execution within a process. A process can have multiple threads, all sharing the same memory space.

```
Process
+----------------------------------+
|  Code | Data | Heap (shared)     |
|----------------------------------|
| Thread 1  | Thread 2  | Thread 3 |
| Stack     | Stack     | Stack    |
| Registers | Registers | Registers|
+----------------------------------+
```

### Process vs. Thread

| | Process | Thread |
|-|---------|--------|
| Memory | Own address space | Shares process memory |
| Creation cost | Heavy | Lightweight |
| Communication | Harder (IPC needed) | Easy (shared memory) |
| Crash impact | Isolated | Can crash whole process |

### Why Use Threads?

A web server handling 1000 users could create 1000 threads — each handles one user — rather than 1000 full processes. Much faster and lighter.

### Thread Example in C (pthreads)

```c
#include <pthread.h>
#include <stdio.h>

void* say_hello(void* arg) {
    printf("Hello from thread!\n");
    return NULL;
}

int main() {
    pthread_t t;
    pthread_create(&t, NULL, say_hello, NULL);
    pthread_join(t, NULL);  // wait for thread to finish
    return 0;
}
```

---

## 3. The Critical Section Problem

When two or more threads share data and at least one modifies it, we have a **race condition** — the result depends on the exact timing of execution.

### Example: Two threads incrementing a counter

```c
int counter = 0;  // shared variable

// Thread 1            // Thread 2
counter++;             counter++;
```

Each `counter++` is actually three steps:
1. Read `counter` from memory
2. Add 1
3. Write back to memory

If both threads interleave these steps, you can lose an update:

```
Thread 1 reads counter = 0
Thread 2 reads counter = 0
Thread 1 writes counter = 1
Thread 2 writes counter = 1   ← should be 2!
```

The section of code that accesses shared data is called the **critical section**.

### Requirements for a Correct Solution

1. **Mutual Exclusion** — only one thread in its critical section at a time
2. **Progress** — if no thread is in the critical section, one waiting thread must eventually enter
3. **Bounded Waiting** — no thread waits forever (no starvation)

---

## 4. Synchronization Tools

### 4.1 Mutex (Mutual Exclusion Lock)

A **mutex** is a lock — a thread acquires it before entering a critical section and releases it when done. If another thread tries to acquire a locked mutex, it blocks.

```c
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;

// Thread 1 and Thread 2 both do this:
pthread_mutex_lock(&lock);
    counter++;          // critical section — safe now
pthread_mutex_unlock(&lock);
```

Only one thread holds the lock at a time. Others wait.

---

### 4.2 Semaphore

A **semaphore** is a more general tool — it's an integer variable with two atomic operations:

- **wait()** (also called P or down): decrement; if result < 0, block
- **signal()** (also called V or up): increment; wake a blocked thread if any

**Binary semaphore** (value 0 or 1) works like a mutex.

**Counting semaphore** can be initialized to N — useful when N threads can access a resource simultaneously.

```c
// Allow up to 3 threads into a resource pool
sem_t pool;
sem_init(&pool, 0, 3);

// Each thread:
sem_wait(&pool);    // enter (decrement)
    // use resource
sem_post(&pool);    // leave (increment)
```

---

### 4.3 Monitor

A **monitor** is a higher-level construct (common in Java, C#) that bundles shared data with the procedures that operate on it, and enforces mutual exclusion automatically.

```java
class Counter {
    private int count = 0;

    public synchronized void increment() {
        count++;   // synchronized = only one thread at a time
    }

    public synchronized int get() {
        return count;
    }
}
```

The `synchronized` keyword in Java creates a monitor — the programmer doesn't manage locks manually.

---

## 5. Classic Synchronization Problems

These are canonical problems used to test whether a synchronization solution works correctly.

### 5.1 Producer-Consumer Problem

- A **producer** generates data and puts it in a shared buffer
- A **consumer** takes data from the buffer
- Problem: producer must not add to a full buffer; consumer must not take from an empty one

**Solution:** Use two semaphores — one tracking empty slots, one tracking full slots.

---

### 5.2 Readers-Writers Problem

- Multiple readers can read shared data simultaneously
- Writers need exclusive access
- Problem: don't starve writers if readers keep arriving

---

### 5.3 Dining Philosophers Problem

Five philosophers sit at a round table. Each needs two forks to eat, but there's only one fork between each pair.

```
        [P1]
    🍴       🍴
[P5]           [P2]
    🍴       🍴
       [P4]--[P3]
           🍴
```

- If each philosopher picks up their left fork simultaneously, nobody can pick up their right fork → **deadlock**
- This models resource contention where processes hold some resources and wait for others

---

## 6. Deadlock

A **deadlock** occurs when a set of processes are each waiting for a resource held by another — and none can proceed.

### The Four Conditions (all must hold for deadlock)

1. **Mutual Exclusion** — resources can't be shared
2. **Hold and Wait** — a process holds one resource and waits for another
3. **No Preemption** — resources can't be forcibly taken away
4. **Circular Wait** — P1 waits for P2, P2 waits for P3, P3 waits for P1

### Deadlock Strategies

| Strategy | Approach |
|----------|----------|
| **Prevention** | Remove one of the four conditions |
| **Avoidance** | Use the Banker's Algorithm to never enter unsafe states |
| **Detection & Recovery** | Allow deadlocks, detect them, then break them |
| **Ignore** | Assume deadlocks are rare (used in many real OSes!) |

### Banker's Algorithm (Avoidance)

Before granting a resource request, the OS simulates whether fulfilling it could lead to deadlock. If it could, the request is denied until it's safe.

---

## 7. Real-World Example: A Database with Multiple Users

Imagine two users trying to transfer money between accounts simultaneously:

```
User A: Transfer from Account 1 → Account 2
User B: Transfer from Account 2 → Account 1
```

Without synchronization:
- User A locks Account 1, waits for Account 2
- User B locks Account 2, waits for Account 1
- **Deadlock** — both wait forever

A real database uses transactions with careful locking strategies to prevent this.

---

## 8. Summary

| Concept | Key Takeaway |
|---------|-------------|
| Concurrency | Multiple tasks overlapping in time |
| Thread | Lightweight execution unit within a process |
| Race condition | Bug caused by unsynchronized access to shared data |
| Critical section | Code that accesses shared resources |
| Mutex | Lock ensuring one thread at a time |
| Semaphore | Generalized signaling tool |
| Monitor | High-level synchronized object |
| Deadlock | Circular waiting — all processes stuck |
| Deadlock conditions | Mutual exclusion, hold & wait, no preemption, circular wait |

---

## Module 3 Quiz

---

**Question 1**
What is a race condition and when does it occur?

> **Answer:** A race condition occurs when two or more threads access shared data concurrently and at least one modifies it, and the final result depends on the order/timing of execution. It leads to unpredictable, incorrect behavior.

---

**Question 2**
What are the three requirements a correct critical section solution must satisfy?

> **Answer:**
> 1. **Mutual Exclusion** — only one thread in its critical section at a time
> 2. **Progress** — a waiting thread must eventually be allowed in
> 3. **Bounded Waiting** — no thread waits forever (no starvation)

---

**Question 3**
What is the difference between a mutex and a counting semaphore?

> **Answer:** A mutex is a binary lock — one thread holds it, all others block. A counting semaphore holds an integer value and can allow up to N threads into a section simultaneously. A mutex is essentially a semaphore initialized to 1.

---

**Question 4**
Which of the following is NOT one of the four conditions required for deadlock?

- A) Circular Wait
- B) Hold and Wait
- C) Preemption
- D) Mutual Exclusion

> **Answer: C** — The four deadlock conditions are mutual exclusion, hold and wait, **no preemption**, and circular wait. Preemption (the ability to forcibly take resources) actually *prevents* deadlock — its absence is the condition, not preemption itself.

---

**Question 5**
Two processes are running:
- Process A holds a printer and is waiting for a scanner
- Process B holds a scanner and is waiting for a printer

Is this a deadlock? Which of the four conditions are present?

> **Answer:** Yes, this is a deadlock. All four conditions are present:
> - **Mutual Exclusion** — printer and scanner can only be used by one process at a time
> - **Hold and Wait** — each process holds one resource and waits for another
> - **No Preemption** — neither resource is taken away forcibly
> - **Circular Wait** — A waits for B's resource, B waits for A's resource

---

## What's Next

**Module 4: Memory Management**

You'll learn how the OS allocates and manages RAM — including how it handles more programs than memory can fit using techniques like paging and virtual memory.

---

*CS 2301 Operating Systems 1 — Module 3 of 8*
