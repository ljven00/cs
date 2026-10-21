# CS 3307 — Operating Systems 2
## Module 2: Parallel Computing and Concurrency

---

> **Course Progress:** Module 2 of 8 — CS 3307 (OS 2)

---

## 1. From Concurrency to Parallelism

In CS 2301 Module 3, you learned the basics of concurrency — multiple tasks making progress in overlapping time, with threads, mutexes, semaphores, and deadlock. This module goes deeper.

Quick recap of the distinction:

| Term | Meaning | Hardware needed |
|------|---------|----------------|
| **Concurrency** | Multiple tasks in progress at overlapping times | Single or multi-core |
| **Parallelism** | Multiple tasks running at the exact same instant | Multi-core required |

Concurrency is about **structure** (how you design the program). Parallelism is about **execution** (how the hardware runs it). A concurrent program may or may not run in parallel depending on the hardware.

---

## 2. Why Parallel Computing?

Single-core CPU clock speeds stopped increasing around 2004 due to heat and power limits. The industry responded by adding more cores. Today:

- A typical laptop CPU has 8–16 cores
- A server CPU has 32–128 cores
- A GPU has thousands of simpler cores

To take advantage of this hardware, software must be written to run in **parallel**. A program that only uses one core runs at the same speed on a 64-core machine as on a 1-core machine.

---

## 3. Amdahl's Law

**Amdahl's Law** quantifies the maximum speedup achievable by parallelizing a program.

```
Speedup = 1 / (S + (1 - S) / N)

Where:
  S = fraction of the program that must run serially (cannot be parallelized)
  N = number of processors
```

### Example

If 20% of a program is serial and you add 4 cores:

```
Speedup = 1 / (0.2 + 0.8 / 4)
        = 1 / (0.2 + 0.2)
        = 1 / 0.4
        = 2.5x speedup
```

Even with infinite cores, the maximum speedup is `1 / S = 1 / 0.2 = 5x`.

### Key Insight

The serial portion of a program is the bottleneck. Even a small serial fraction severely limits parallel speedup. This is why identifying and minimizing serial sections is critical in parallel programming.

---

## 4. Parallel Programming Models

### 4.1 Shared Memory Model

All threads share the same address space. They communicate by reading and writing shared variables.

```
Thread 1  Thread 2  Thread 3
   |         |         |
   +----+----+----+----+
        | Shared Memory |
        +---------------+
```

- Natural extension of single-threaded programming
- Requires synchronization (mutexes, semaphores) to avoid race conditions
- Used by: pthreads, OpenMP, Java threads, C++ std::thread

### 4.2 Message Passing Model

Processes have separate memory spaces. They communicate by explicitly sending and receiving messages.

```
Process 1          Process 2
[Memory A]  --msg--> [Memory B]
            <--msg--
```

- No shared state — no race conditions by design
- Scales to distributed systems (different machines)
- Used by: MPI (Message Passing Interface), Erlang, Go channels, Actor model

### 4.3 Data Parallelism

The same operation is applied to many data elements simultaneously. This is the model GPUs excel at.

```
Array: [1, 2, 3, 4, 5, 6, 7, 8]
         ↓  ↓  ↓  ↓  ↓  ↓  ↓  ↓
       [×2 ×2 ×2 ×2 ×2 ×2 ×2 ×2]  (all in parallel)
Result:[2, 4, 6, 8,10,12,14,16]
```

- Used by: SIMD CPU instructions, CUDA (GPU), OpenCL, NumPy (Python)

### 4.4 Task Parallelism

Different tasks run in parallel, each possibly doing something different.

```
Task A: compress file    ─┐
Task B: send email       ─┤ all running simultaneously
Task C: update database  ─┘
```

---

## 5. Threads in Depth

### User-Level vs. Kernel-Level Threads

| | User-Level Threads | Kernel-Level Threads |
|-|--------------------|----------------------|
| Managed by | User-space library | OS kernel |
| Context switch | Fast (no kernel call) | Slower (kernel involvement) |
| Parallelism | Limited (kernel sees one thread) | True parallelism on multi-core |
| Block on I/O | Blocks all threads in process | Only blocks that thread |
| Example | Green threads, early Java | pthreads, Windows threads |

Modern systems use a **many-to-many** or **one-to-one** mapping. Linux uses one-to-one — each user thread maps to one kernel thread, giving true parallelism.

### Thread Pools

Creating and destroying threads has overhead. A **thread pool** pre-creates a fixed number of threads that pick up tasks from a queue.

```
Task Queue: [task1] [task2] [task3] [task4] [task5]
                ↓        ↓        ↓
           Thread 1  Thread 2  Thread 3  (pool of 3)
```

- Limits the number of concurrent threads (avoids thrashing)
- Reuses threads — no creation/destruction overhead per task
- Used by web servers, database engines, and almost every production system

```c
// Conceptual thread pool usage
ThreadPool pool = create_pool(4);   // 4 worker threads
submit_task(pool, handle_request);  // task picked up by next free thread
```

---

## 6. Advanced Synchronization

CS 2301 covered mutexes and semaphores. Here are more advanced tools used in real systems.

### 6.1 Read-Write Locks

A **read-write lock** allows multiple readers OR one writer at a time — never both.

```
State: UNLOCKED
  → Any number of readers can lock simultaneously (read lock)
  → Only one writer can lock, and only when no readers hold the lock (write lock)
```

Why? In many systems, reads are far more frequent than writes (e.g., a config file read thousands of times per second, updated once per hour). A plain mutex would unnecessarily serialize all those reads.

```c
pthread_rwlock_t rwlock = PTHREAD_RWLOCK_INITIALIZER;

// Reader thread
pthread_rwlock_rdlock(&rwlock);
    // read shared data
pthread_rwlock_unlock(&rwlock);

// Writer thread
pthread_rwlock_wrlock(&rwlock);
    // modify shared data
pthread_rwlock_unlock(&rwlock);
```

### 6.2 Condition Variables

A **condition variable** lets a thread sleep until a specific condition becomes true, without wasting CPU in a busy-wait loop.

```c
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t  cond = PTHREAD_COND_INITIALIZER;
int ready = 0;

// Consumer thread — waits until ready
pthread_mutex_lock(&lock);
while (!ready) {
    pthread_cond_wait(&cond, &lock);  // sleeps, releases lock
}
// do work with data
pthread_mutex_unlock(&lock);

// Producer thread — signals when ready
pthread_mutex_lock(&lock);
ready = 1;
pthread_cond_signal(&cond);  // wake up the consumer
pthread_mutex_unlock(&lock);
```

`pthread_cond_wait` atomically releases the lock and sleeps — no window for a missed signal.

### 6.3 Barriers

A **barrier** makes all threads in a group wait until every thread has reached the barrier point, then they all continue together.

```
Thread 1: ----[work]----BARRIER--|
Thread 2: --------[work]--BARRIER--|  → all proceed together
Thread 3: --[work]--------BARRIER--|
```

Used in parallel algorithms where all threads must complete Phase 1 before any begins Phase 2.

```c
pthread_barrier_t barrier;
pthread_barrier_init(&barrier, NULL, 3);  // 3 threads must arrive

// Each thread calls:
pthread_barrier_wait(&barrier);  // blocks until all 3 arrive
```

---

## 7. Lock-Free and Wait-Free Programming

Locks (mutexes) have problems at scale:
- A thread holding a lock can be preempted, blocking all other threads
- Lock contention becomes a bottleneck with many cores
- Deadlock is always a risk

**Lock-free** data structures use **atomic operations** — hardware-guaranteed single-step operations that cannot be interrupted.

### Compare-And-Swap (CAS)

The foundation of lock-free programming. CAS atomically does:

```
if (memory[addr] == expected) {
    memory[addr] = new_value;
    return true;
} else {
    return false;  // someone else changed it first; retry
}
```

### Lock-Free Counter Example

```c
#include <stdatomic.h>

atomic_int counter = 0;

// Thread-safe increment — no mutex needed
atomic_fetch_add(&counter, 1);
```

The CPU guarantees `atomic_fetch_add` is indivisible — no other thread can observe an intermediate state.

### Lock-Free vs. Wait-Free

| | Lock-Free | Wait-Free |
|-|-----------|-----------|
| Progress guarantee | At least one thread always makes progress | Every thread makes progress in bounded steps |
| Complexity | Moderate | Very high |
| Used in | Queues, stacks, counters | Real-time systems, OS internals |

---

## 8. The Memory Consistency Problem

On multi-core systems, each core has its own cache. When one core writes to memory, other cores may not see the change immediately.

```
Core 1 cache: x = 1
Core 2 cache: x = 0   ← stale!
```

This is the **cache coherence** problem. Hardware protocols (like MESI) keep caches consistent, but the OS and programmer must also use the right memory ordering.

### Memory Barriers (Fences)

A **memory barrier** instruction tells the CPU: "don't reorder memory operations past this point."

```c
// Without barrier: CPU may reorder writes for performance
flag = 1;      // CPU might reorder these
data = 42;

// With barrier: guaranteed order
data = 42;
__sync_synchronize();  // memory barrier
flag = 1;              // other cores see data before flag
```

In practice, mutexes and atomic operations include implicit memory barriers — you only need explicit barriers in very low-level code.

---

## 9. Common Parallel Patterns

### 9.1 Map-Reduce

Split data into chunks, process each chunk in parallel (map), then combine results (reduce).

```
Data: [1, 2, 3, 4, 5, 6, 7, 8]
         ↓ split
Chunk A: [1,2,3,4]  → sum = 10  ─┐
Chunk B: [5,6,7,8]  → sum = 26  ─┤ reduce → total = 36
```

Used by: Hadoop, Spark, and distributed data processing at massive scale.

### 9.2 Producer-Consumer Pipeline

Stages of processing run in parallel, passing data through a queue.

```
[Producer] → Queue → [Stage 1] → Queue → [Stage 2] → Queue → [Consumer]
```

Each stage runs in its own thread. While Stage 1 processes item N, Stage 2 processes item N-1. Like an assembly line.

### 9.3 Fork-Join

A main thread forks into multiple worker threads, each does a portion of the work, then all join back to the main thread.

```
main ──fork──► worker 1 ──┐
         └──► worker 2 ──┤ join ──► main continues
         └──► worker 3 ──┘
```

Used by: Java ForkJoinPool, OpenMP parallel for loops.

---

## 10. Real-World Example: A Parallel Web Server

A web server receives thousands of requests per second. Here is how parallelism is applied:

```
Incoming requests
       ↓
  [Acceptor Thread]        ← single thread accepts connections
       ↓
  [Thread Pool: 32 threads] ← each handles one request in parallel
   ├── Thread 1: GET /home
   ├── Thread 2: POST /login
   ├── Thread 3: GET /api/data
   └── ...
```

Each worker thread:
1. Reads the HTTP request (I/O — uses async or blocking read)
2. Queries the database (uses connection pool with read-write locks)
3. Formats and sends the response

A read-write lock protects the database connection pool — many threads can read simultaneously, but only one modifies the pool at a time. Condition variables wake threads when a connection becomes available.

---

## 11. Summary

| Concept | Key Takeaway |
|---------|-------------|
| Concurrency vs. Parallelism | Structure vs. simultaneous execution |
| Amdahl's Law | Serial fraction limits parallel speedup |
| Shared memory model | Threads share address space; needs synchronization |
| Message passing model | Separate memory; communicate via messages; scales to distributed systems |
| Thread pool | Pre-created threads pick up tasks; avoids creation overhead |
| Read-write lock | Multiple readers OR one writer; better than mutex for read-heavy workloads |
| Condition variable | Sleep until a condition is true; avoids busy-waiting |
| Barrier | All threads wait until everyone arrives |
| Lock-free / CAS | Atomic hardware operations; no locks needed |
| Memory barrier | Prevents CPU from reordering memory operations |
| Map-Reduce | Split, process in parallel, combine results |
| Fork-Join | Split work into subtasks, rejoin when all complete |

---

## Module 2 Quiz

---

**Question 1**
A program is 30% serial and 70% parallelizable. Using Amdahl's Law, what is the maximum speedup achievable with any number of cores?

> **Answer:**
> Maximum speedup = 1 / S = 1 / 0.30 = **3.33x**
>
> No matter how many cores you add, the 30% serial portion means you can never exceed a 3.33x speedup. With 4 cores: 1 / (0.3 + 0.7/4) = 1 / 0.475 = **2.1x**. This shows how quickly returns diminish as the serial fraction dominates.

---

**Question 2**
Why is a read-write lock more efficient than a plain mutex for a configuration object that is read 10,000 times per second but updated once per minute?

> **Answer:** A plain mutex allows only one thread at a time — even readers block each other, serializing all 10,000 reads per second unnecessarily. A read-write lock allows all concurrent readers to hold the lock simultaneously, since reads don't conflict with each other. Only the rare write requires exclusive access. This eliminates almost all lock contention in a read-heavy workload.

---

**Question 3**
What is the difference between lock-free and wait-free programming?

> **Answer:** Lock-free guarantees that at least one thread in the system always makes forward progress — some thread completes its operation even if others are delayed. Wait-free is stronger: every thread individually completes its operation in a bounded number of steps, regardless of what other threads do. Wait-free is harder to implement but necessary for hard real-time systems where no thread can be indefinitely delayed.

---

**Question 4**
A parallel program has 4 threads that each compute a partial sum of an array. They must all finish their partial sums before any thread combines the results. Which synchronization primitive should be used at the combination point, and why?

> **Answer:** A **barrier**. A barrier makes all threads wait until every thread has arrived at that point. Once all 4 threads have completed their partial sums and reached the barrier, they all proceed together to the combination step. A mutex alone would not be appropriate here since you don't want mutual exclusion — you want synchronization of all threads at a specific point in time.

---

**Question 5**
Explain why the following code is unsafe in a multithreaded program, and how you would fix it.

```c
int counter = 0;

void increment() {
    counter++;   // called by multiple threads simultaneously
}
```

> **Answer:** `counter++` is not atomic — it compiles to three instructions: read, add, write. Two threads can interleave these steps and lose updates (race condition). There are two correct fixes:
>
> **Fix 1 — Mutex:**
> ```c
> pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
> void increment() {
>     pthread_mutex_lock(&lock);
>     counter++;
>     pthread_mutex_unlock(&lock);
> }
> ```
>
> **Fix 2 — Atomic operation (lock-free):**
> ```c
> #include <stdatomic.h>
> atomic_int counter = 0;
> void increment() {
>     atomic_fetch_add(&counter, 1);
> }
> ```
>
> The atomic version is faster at high contention since it avoids the overhead of kernel-level locking.

---

## What's Next

**Module 3: Memory and File Structure**

You will go deeper into how memory is structured at the hardware and OS level, and how file systems organize data on disk — including inodes, directory structures, and journaling.

---

*CS 3307 Operating Systems 2 — Module 2 of 8*
