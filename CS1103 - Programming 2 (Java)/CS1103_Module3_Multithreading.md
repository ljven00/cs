# CS 1103 – Programming 2 (Java)
## Module 3: Multithreading

---

## Table of Contents

1. [What Is Multithreading?](#1-what-is-multithreading)
2. [Processes vs Threads](#2-processes-vs-threads)
3. [Creating Threads — Two Approaches](#3-creating-threads--two-approaches)
4. [Thread Lifecycle](#4-thread-lifecycle)
5. [Thread Methods](#5-thread-methods)
6. [The Problem with Shared Data](#6-the-problem-with-shared-data)
7. [Synchronization](#7-synchronization)
8. [Inter-Thread Communication](#8-inter-thread-communication)
9. [The Executor Framework](#9-the-executor-framework)
10. [Callable and Future](#10-callable-and-future)
11. [Common Multithreading Pitfalls](#11-common-multithreading-pitfalls)
12. [Putting It All Together](#12-putting-it-all-together)
13. [Exercises](#13-exercises)

---

## 1. What Is Multithreading?

Every program you've written so far runs as a single sequence of instructions — one thing happens, then the next, then the next. This is called **single-threaded** execution.

**Multithreading** allows a program to do several things at the same time by running multiple **threads** concurrently. Each thread is an independent path of execution within the same program.

Real-world examples where multithreading is essential:

- A **web browser** loads a page, plays a video, and responds to your clicks — all at once.
- A **web server** handles hundreds of client requests simultaneously.
- A **download manager** downloads multiple files in parallel.
- A **GUI application** keeps the interface responsive while a slow operation runs in the background.

Without multithreading, a GUI app would freeze completely while loading a file — the user couldn't even move the window. With it, the background work runs on a separate thread while the UI thread stays responsive.

---

## 2. Processes vs Threads

These terms are often confused:

| | Process | Thread |
|---|---|---|
| What it is | A running program | A unit of execution within a process |
| Memory | Has its own memory space | Shares memory with other threads in the same process |
| Creation cost | Expensive — the OS allocates resources | Lightweight — threads share the process's resources |
| Communication | Complex — must use IPC (inter-process communication) | Simple — threads can read/write shared variables |
| Example | Chrome browser (one process) | Each browser tab runs in its own thread |

A Java program is a **process**. When it starts, it has one thread — the **main thread** — which runs your `main()` method. You can create additional threads to run other code in parallel.

---

## 3. Creating Threads — Two Approaches

Java gives you two ways to create a thread.

### Approach 1 — Extend the `Thread` Class

Create a class that extends `Thread` and override its `run()` method. The `run()` method is what the thread will execute.

```java
public class CounterThread extends Thread {

    private String name;
    private int limit;

    CounterThread(String name, int limit) {
        this.name  = name;
        this.limit = limit;
    }

    @Override
    public void run() {
        for (int i = 1; i <= limit; i++) {
            System.out.println(name + " → " + i);
        }
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        CounterThread t1 = new CounterThread("Thread A", 5);
        CounterThread t2 = new CounterThread("Thread B", 5);

        t1.start();   // start() creates a new thread and calls run()
        t2.start();
    }
}
```

Possible output (order is not guaranteed — threads run concurrently):
```
Thread A → 1
Thread B → 1
Thread A → 2
Thread B → 2
Thread B → 3
Thread A → 3
Thread B → 4
Thread B → 5
Thread A → 4
Thread A → 5
```

> **Critical:** Always call `start()`, never `run()` directly. Calling `run()` directly just runs the method on the current thread — no new thread is created.

### Approach 2 — Implement the `Runnable` Interface

The preferred approach. `Runnable` is a functional interface with a single `run()` method. Your class implements `Runnable`, and you pass it to a `Thread` object.

```java
public class PrintTask implements Runnable {

    private String message;
    private int times;

    PrintTask(String message, int times) {
        this.message = message;
        this.times   = times;
    }

    @Override
    public void run() {
        for (int i = 0; i < times; i++) {
            System.out.println(message);
        }
    }
}
```

```java
Thread t1 = new Thread(new PrintTask("Hello from Task 1", 3));
Thread t2 = new Thread(new PrintTask("Hello from Task 2", 3));

t1.start();
t2.start();
```

### Why Prefer Runnable?

- Java allows only **single inheritance** — if your class extends `Thread`, it can't extend anything else.
- `Runnable` separates the **task** (what to do) from the **thread** (how to run it). You can reuse the same `Runnable` with different thread configurations.
- `Runnable` works seamlessly with Java's Executor framework (covered later).

### Lambda Shortcut

Since `Runnable` is a functional interface, you can use a lambda:

```java
Thread t1 = new Thread(() -> {
    for (int i = 1; i <= 5; i++) {
        System.out.println("Lambda thread: " + i);
    }
});

t1.start();
```

---

## 4. Thread Lifecycle

A thread passes through several states during its life:

```
NEW → RUNNABLE → (BLOCKED/WAITING/TIMED_WAITING) → TERMINATED
```

| State | Description |
|---|---|
| **NEW** | Thread created but `start()` not yet called |
| **RUNNABLE** | Ready to run or currently running on the CPU |
| **BLOCKED** | Waiting to acquire a lock (for synchronized access) |
| **WAITING** | Paused indefinitely until another thread notifies it |
| **TIMED_WAITING** | Paused for a specific duration (e.g. `sleep()`) |
| **TERMINATED** | `run()` has finished — the thread is done |

You can check a thread's state with `thread.getState()`, but in practice you rarely need to — you just call the right methods and let Java manage transitions.

---

## 5. Thread Methods

### `sleep()` — Pause a Thread

`Thread.sleep(milliseconds)` pauses the current thread for the specified time. It throws `InterruptedException`, so you must handle it:

```java
Thread t = new Thread(() -> {
    for (int i = 1; i <= 5; i++) {
        System.out.println("Tick " + i);
        try {
            Thread.sleep(1000);   // pause 1 second between ticks
        } catch (InterruptedException e) {
            System.out.println("Thread interrupted.");
        }
    }
});

t.start();
```

Output (with 1-second gaps):
```
Tick 1
Tick 2
Tick 3
Tick 4
Tick 5
```

### `join()` — Wait for a Thread to Finish

`join()` makes the calling thread wait until the target thread finishes. This is how you ensure one thread's results are ready before proceeding:

```java
Thread t = new Thread(() -> {
    System.out.println("Worker thread running...");
    try { Thread.sleep(2000); } catch (InterruptedException e) {}
    System.out.println("Worker thread done.");
});

t.start();

System.out.println("Main thread waiting for worker...");
t.join();   // main thread pauses here until t finishes
System.out.println("Main thread continuing.");
```

Output:
```
Main thread waiting for worker...
Worker thread running...
Worker thread done.
Main thread continuing.
```

### `setName()` and `getName()`

```java
Thread t = new Thread(() -> {
    System.out.println("Running: " + Thread.currentThread().getName());
});

t.setName("MyWorkerThread");
t.start();
```

### `setPriority()` — Hint at Scheduling

Threads can have a priority from 1 (`MIN_PRIORITY`) to 10 (`MAX_PRIORITY`), default 5 (`NORM_PRIORITY`). Higher priority threads are more likely to be scheduled first — but this is only a **hint** to the JVM. Don't rely on it for correctness.

```java
t.setPriority(Thread.MAX_PRIORITY);   // 10
```

### `setDaemon()` — Background Threads

A **daemon thread** runs in the background and is automatically killed when all non-daemon (user) threads finish. Useful for background tasks like garbage collection or monitoring:

```java
Thread monitor = new Thread(() -> {
    while (true) {
        System.out.println("Monitoring...");
        try { Thread.sleep(1000); } catch (InterruptedException e) { break; }
    }
});

monitor.setDaemon(true);   // must be called BEFORE start()
monitor.start();
```

---

## 6. The Problem with Shared Data

When multiple threads access and modify the same data, things can go wrong in unexpected ways.

### Race Condition

A **race condition** happens when the final result depends on the unpredictable timing of thread execution.

```java
public class Counter {
    int count = 0;

    void increment() {
        count++;   // looks atomic, but it's actually three steps:
                   // 1. read count
                   // 2. add 1
                   // 3. write back
    }
}
```

```java
Counter counter = new Counter();

Thread t1 = new Thread(() -> {
    for (int i = 0; i < 1000; i++) counter.increment();
});

Thread t2 = new Thread(() -> {
    for (int i = 0; i < 1000; i++) counter.increment();
});

t1.start();
t2.start();
t1.join();
t2.join();

System.out.println("Final count: " + counter.count);
```

Expected: `2000`. Actual: something less — perhaps `1743` or `1891` — different every run.

**Why?** Two threads can both read `count` (say, `500`) before either writes back. Both compute `501` and write it. One increment is effectively lost.

This is called a **data race** — and it's the core challenge of multithreading.

---

## 7. Synchronization

**Synchronization** prevents race conditions by ensuring only one thread executes a critical section at a time.

### The `synchronized` Keyword

Add `synchronized` to a method to make it a **monitor** — only one thread can execute it at a time. Others must wait.

```java
public class SafeCounter {
    private int count = 0;

    public synchronized void increment() {
        count++;   // only one thread runs this at a time
    }

    public synchronized int getCount() {
        return count;
    }
}
```

```java
SafeCounter counter = new SafeCounter();

Thread t1 = new Thread(() -> {
    for (int i = 0; i < 1000; i++) counter.increment();
});

Thread t2 = new Thread(() -> {
    for (int i = 0; i < 1000; i++) counter.increment();
});

t1.start();
t2.start();
t1.join();
t2.join();

System.out.println("Final count: " + counter.getCount());   // always 2000
```

### Synchronized Block

If you only need to synchronize part of a method (for better performance), use a synchronized block:

```java
public void increment() {
    // non-critical code can run freely
    synchronized (this) {
        count++;   // only this part is locked
    }
    // more non-critical code
}
```

### The `volatile` Keyword

For simple flags that are read/written by multiple threads, `volatile` ensures every thread sees the latest value — changes are immediately visible across threads:

```java
public class Worker implements Runnable {
    private volatile boolean running = true;

    public void stop() {
        running = false;   // change visible to the worker thread immediately
    }

    @Override
    public void run() {
        while (running) {
            System.out.println("Working...");
            try { Thread.sleep(500); } catch (InterruptedException e) { break; }
        }
        System.out.println("Worker stopped.");
    }
}
```

```java
Worker worker = new Worker();
Thread t = new Thread(worker);
t.start();

Thread.sleep(2000);
worker.stop();   // signals the worker to stop
```

`volatile` is lighter than `synchronized` but only safe for simple reads and writes — not for compound operations like `count++`.

### Atomic Classes

For counters and flags, `java.util.concurrent.atomic` provides thread-safe classes without needing `synchronized`:

```java
import java.util.concurrent.atomic.AtomicInteger;

public class AtomicCounter {
    private AtomicInteger count = new AtomicInteger(0);

    public void increment() {
        count.incrementAndGet();   // atomic — thread-safe without synchronized
    }

    public int getCount() {
        return count.get();
    }
}
```

---

## 8. Inter-Thread Communication

Sometimes threads need to coordinate — one thread produces data, another consumes it. Java provides `wait()`, `notify()`, and `notifyAll()` for this.

These methods are called on an object that acts as a shared monitor:

| Method | What it does |
|---|---|
| `wait()` | Releases the lock and pauses until notified |
| `notify()` | Wakes up one waiting thread |
| `notifyAll()` | Wakes up all waiting threads |

All three must be called from within a `synchronized` block.

### Producer-Consumer Example

```java
import java.util.LinkedList;
import java.util.Queue;

public class SharedBuffer {

    private Queue<Integer> buffer = new LinkedList<>();
    private int capacity;

    SharedBuffer(int capacity) {
        this.capacity = capacity;
    }

    public synchronized void produce(int item) throws InterruptedException {
        while (buffer.size() == capacity) {
            System.out.println("Buffer full — producer waiting...");
            wait();   // wait until consumer removes an item
        }
        buffer.add(item);
        System.out.println("Produced: " + item + " | Buffer: " + buffer);
        notifyAll();   // wake up any waiting consumers
    }

    public synchronized int consume() throws InterruptedException {
        while (buffer.isEmpty()) {
            System.out.println("Buffer empty — consumer waiting...");
            wait();   // wait until producer adds an item
        }
        int item = buffer.poll();
        System.out.println("Consumed: " + item + " | Buffer: " + buffer);
        notifyAll();   // wake up any waiting producers
        return item;
    }
}
```

```java
SharedBuffer buffer = new SharedBuffer(3);

Thread producer = new Thread(() -> {
    for (int i = 1; i <= 7; i++) {
        try {
            buffer.produce(i);
            Thread.sleep(300);
        } catch (InterruptedException e) {}
    }
});

Thread consumer = new Thread(() -> {
    for (int i = 1; i <= 7; i++) {
        try {
            buffer.consume();
            Thread.sleep(700);
        } catch (InterruptedException e) {}
    }
});

producer.start();
consumer.start();
```

The producer adds items; the consumer removes them. When the buffer is full, the producer waits. When empty, the consumer waits. `notifyAll()` wakes the other side when the situation changes.

---

## 9. The Executor Framework

Creating threads manually works for simple cases, but for real applications the `java.util.concurrent` **Executor framework** is the preferred approach. It manages a **pool** of reusable threads, avoiding the overhead of creating and destroying threads repeatedly.

### ExecutorService and Thread Pools

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

ExecutorService executor = Executors.newFixedThreadPool(3);   // pool of 3 threads
```

Common pool types:

| Factory method | Description |
|---|---|
| `newFixedThreadPool(n)` | Exactly `n` threads — tasks queue if all are busy |
| `newCachedThreadPool()` | Creates threads as needed, reuses idle ones |
| `newSingleThreadExecutor()` | Exactly 1 thread — tasks run one at a time |
| `newScheduledThreadPool(n)` | Runs tasks after a delay or on a schedule |

### Submitting Tasks

```java
ExecutorService executor = Executors.newFixedThreadPool(3);

for (int i = 1; i <= 6; i++) {
    final int taskId = i;
    executor.submit(() -> {
        System.out.println("Task " + taskId + " running on "
            + Thread.currentThread().getName());
        try { Thread.sleep(500); } catch (InterruptedException e) {}
        System.out.println("Task " + taskId + " done.");
    });
}

executor.shutdown();   // stop accepting new tasks; finish existing ones
```

Output (order varies — only 3 threads handle 6 tasks):
```
Task 1 running on pool-1-thread-1
Task 2 running on pool-1-thread-2
Task 3 running on pool-1-thread-3
Task 1 done.
Task 4 running on pool-1-thread-1
Task 2 done.
Task 5 running on pool-1-thread-2
...
```

### Shutdown Methods

| Method | Behaviour |
|---|---|
| `shutdown()` | Stops accepting new tasks; waits for running tasks to finish |
| `shutdownNow()` | Attempts to stop all running tasks immediately |
| `awaitTermination(time, unit)` | Blocks until all tasks finish or timeout expires |

---

## 10. Callable and Future

`Runnable` runs a task but can't return a result or throw a checked exception. **`Callable<V>`** fixes both:

```java
import java.util.concurrent.*;

Callable<Integer> task = () -> {
    Thread.sleep(1000);
    return 42;   // returns a result
};

ExecutorService executor = Executors.newSingleThreadExecutor();
Future<Integer> future = executor.submit(task);

System.out.println("Task submitted. Doing other work...");

int result = future.get();   // blocks here until the result is ready
System.out.println("Result: " + result);   // Result: 42

executor.shutdown();
```

### Future Methods

| Method | What it does |
|---|---|
| `get()` | Blocks and returns the result when ready |
| `get(timeout, unit)` | Blocks up to a timeout, then throws `TimeoutException` |
| `isDone()` | Returns `true` if the task has finished |
| `isCancelled()` | Returns `true` if the task was cancelled |
| `cancel(true)` | Attempts to cancel the task |

### Multiple Futures

```java
ExecutorService executor = Executors.newFixedThreadPool(4);
List<Future<Integer>> futures = new ArrayList<>();

for (int i = 1; i <= 4; i++) {
    final int n = i * 10;
    futures.add(executor.submit(() -> {
        Thread.sleep(500);
        return n * n;   // compute square
    }));
}

for (Future<Integer> future : futures) {
    System.out.println("Square: " + future.get());
}

executor.shutdown();
```

Output:
```
Square: 100
Square: 400
Square: 900
Square: 1600
```

---

## 11. Common Multithreading Pitfalls

### Deadlock

A **deadlock** happens when two threads each hold a lock the other needs — both wait forever and the program freezes.

```
Thread 1 holds Lock A, waiting for Lock B
Thread 2 holds Lock B, waiting for Lock A
→ Neither can proceed
```

```java
Object lockA = new Object();
Object lockB = new Object();

Thread t1 = new Thread(() -> {
    synchronized (lockA) {
        System.out.println("T1 has A, waiting for B...");
        try { Thread.sleep(100); } catch (InterruptedException e) {}
        synchronized (lockB) {
            System.out.println("T1 has both locks.");
        }
    }
});

Thread t2 = new Thread(() -> {
    synchronized (lockB) {   // t2 acquires B first (opposite order)
        System.out.println("T2 has B, waiting for A...");
        try { Thread.sleep(100); } catch (InterruptedException e) {}
        synchronized (lockA) {
            System.out.println("T2 has both locks.");
        }
    }
});

t1.start();
t2.start();
// Program hangs — classic deadlock
```

**Prevention:** Always acquire locks in the same order across all threads.

### Starvation

A thread never gets CPU time because other threads are always prioritized. Avoid using extreme thread priorities.

### Over-Synchronization

Synchronizing too much eliminates parallelism — threads queue up and wait. Synchronize only the minimum necessary code.

---

## 12. Putting It All Together

Here's a realistic simulation of a bank with concurrent transactions:

```java
import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicInteger;

public class BankSimulation {

    static class BankAccount {
        private final String owner;
        private double balance;

        BankAccount(String owner, double initialBalance) {
            this.owner   = owner;
            this.balance = initialBalance;
        }

        public synchronized void deposit(double amount) {
            balance += amount;
        }

        public synchronized boolean withdraw(double amount) {
            if (balance >= amount) {
                balance -= amount;
                return true;
            }
            return false;
        }

        public synchronized double getBalance() { return balance; }
        public String getOwner()                { return owner; }
    }

    static AtomicInteger successfulTransactions = new AtomicInteger(0);
    static AtomicInteger failedTransactions     = new AtomicInteger(0);

    public static void main(String[] args) throws InterruptedException {

        BankAccount alice = new BankAccount("Alice", 1000.00);
        BankAccount bob   = new BankAccount("Bob",   1000.00);

        ExecutorService executor = Executors.newFixedThreadPool(5);

        // Submit 20 random transactions
        for (int i = 0; i < 20; i++) {
            final int txId = i + 1;
            executor.submit(() -> {
                double amount = (int)(Math.random() * 200 + 50);   // $50–$250
                boolean aliceToBob = Math.random() > 0.5;

                BankAccount from = aliceToBob ? alice : bob;
                BankAccount to   = aliceToBob ? bob   : alice;

                boolean success = from.withdraw(amount);
                if (success) {
                    to.deposit(amount);
                    successfulTransactions.incrementAndGet();
                    System.out.printf("TX %2d: %-6s → %-6s $%.2f ✓%n",
                        txId, from.getOwner(), to.getOwner(), amount);
                } else {
                    failedTransactions.incrementAndGet();
                    System.out.printf("TX %2d: %-6s → %-6s $%.2f ✗ (insufficient funds)%n",
                        txId, from.getOwner(), to.getOwner(), amount);
                }
            });
        }

        executor.shutdown();
        executor.awaitTermination(10, TimeUnit.SECONDS);

        System.out.println("\n=== Final Balances ===");
        System.out.printf("Alice: $%.2f%n", alice.getBalance());
        System.out.printf("Bob:   $%.2f%n", bob.getBalance());
        System.out.printf("Total: $%.2f (should be $2000.00)%n",
            alice.getBalance() + bob.getBalance());
        System.out.println("Successful: " + successfulTransactions.get());
        System.out.println("Failed:     " + failedTransactions.get());
    }
}
```

The total balance always stays at $2000.00 because `deposit` and `withdraw` are synchronized — no money is ever created or lost, even with 5 threads running concurrently.

---

## 13. Exercises

---

### Exercise 1 – Parallel Countdown

Create two threads using lambda expressions:
- Thread 1 counts up from 1 to 10, sleeping 200ms between each number.
- Thread 2 counts down from 10 to 1, sleeping 300ms between each number.

Start both at the same time. Observe how their output interleaves.

---

### Exercise 2 – join() Practice

Write a program that:
1. Creates 3 threads, each computing the sum of a range of numbers:
   - Thread 1: sum of 1 to 100
   - Thread 2: sum of 101 to 200
   - Thread 3: sum of 201 to 300
2. Uses `join()` to wait for all three to finish.
3. Prints the grand total after all threads complete.

Store each thread's result in a shared array (each thread writes to its own index — no synchronization needed since they never share the same slot).

---

### Exercise 3 – Thread-Safe Counter

Create an `UnsafeCounter` class using a plain `int` field and a `SafeCounter` class using `AtomicInteger`.

For each:
- Launch 10 threads, each calling `increment()` 1000 times.
- Use `join()` to wait for all threads.
- Print the final count.

Run both and compare results. Document what you observe.

---

### Exercise 4 – Thread Pool Task Runner

Write a program that uses `Executors.newFixedThreadPool(4)` to process 10 tasks. Each task should:
1. Print its task number and which thread is running it.
2. Sleep for a random time between 200ms and 800ms.
3. Print when it's done.

Shutdown the executor cleanly and print "All tasks complete" after everything finishes.

---

### Exercise 5 – Callable and Future

Write a program that uses a thread pool and `Callable` to compute:
- The sum of all numbers from 1 to 1,000,000 (in one callable).
- The count of prime numbers from 1 to 100,000 (in another callable).
- The factorial of 15 (in a third callable).

Submit all three to the executor at the same time. Print each result as it becomes available using `future.get()`.

---

### Exercise 6 – Producer-Consumer *(Challenge)*

Implement a producer-consumer system:

**`BoundedBuffer`** — a synchronized buffer with a fixed capacity of 5.
- `put(int item)` — adds an item; waits if full.
- `take()` — removes and returns an item; waits if empty.

**Producer thread** — produces 15 items (numbered 1–15), sleeping 200ms between each.

**Consumer thread** — consumes 15 items, sleeping 500ms between each.

Print each produce and consume action. After all 15 items are consumed, print "All items processed."

---

### Exercise 7 – Deadlock Detection *(Challenge)*

Write a program that intentionally creates a deadlock between two threads (as shown in Section 11).

Then **fix** it by ensuring both threads always acquire the locks in the same order. Verify the fixed version runs to completion.

Document in a comment why the original caused a deadlock and why the fix works.

---

## Module Summary

Here's what you covered in Module 3:

- A **thread** is an independent path of execution within a process; threads share the same memory.
- Create threads by **extending `Thread`** or **implementing `Runnable`** (preferred); lambdas work for `Runnable`.
- Always call `start()` — never `run()` — to launch a new thread.
- `sleep()` pauses a thread; `join()` makes the caller wait for a thread to finish.
- **Race conditions** occur when multiple threads access shared data without coordination — results become unpredictable.
- `synchronized` methods and blocks ensure only one thread executes a critical section at a time.
- `volatile` ensures visibility of simple reads/writes; `AtomicInteger` (and friends) provide thread-safe atomic operations.
- `wait()`, `notify()`, and `notifyAll()` coordinate threads that need to signal each other.
- The **Executor framework** manages thread pools — prefer it over manual thread creation.
- **`Callable<V>`** and **`Future<V>`** allow tasks to return results.
- **Deadlock** occurs when threads wait for each other's locks — prevent it by acquiring locks in a consistent order.

---

**Next up:** Module 4 – I/O and Applets

---

*CS 1103 Programming 2 (Java) — Module 3 of 8*
