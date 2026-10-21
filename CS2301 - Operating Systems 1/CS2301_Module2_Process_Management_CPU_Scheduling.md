# CS 2301 — Operating Systems 1
## Module 2: Process Management & CPU Scheduling

---

> **Course Progress:** Module 2 of 8 — CS 2301 (OS 1)

---

## 1. What is a Process?

A **process** is a program in execution. When you run a program (say, a browser or a game), the OS creates a process for it — giving it memory, a program counter, and resources to run.

Key distinction:
- **Program** — a passive file sitting on disk (e.g., `chrome.exe`)
- **Process** — an active, running instance of that program

You can have multiple processes of the same program running at once (e.g., two browser windows = two processes).

---

## 2. Process Components

Every process has:

| Component | Description |
|-----------|-------------|
| **Code (Text)** | The program instructions |
| **Stack** | Temporary data — function calls, local variables |
| **Heap** | Dynamically allocated memory (e.g., `malloc`) |
| **Data** | Global and static variables |
| **Program Counter** | Points to the next instruction to execute |
| **Registers** | CPU register values at the current moment |

```
+-------------------+
|      Stack        |   ← grows downward
|        ↓          |
|                   |
|        ↑          |
|      Heap         |   ← grows upward
+-------------------+
|      Data         |
+-------------------+
|      Code         |
+-------------------+
```

---

## 3. Process States

A process moves through several states during its lifetime:

```
         admit
new  ─────────────► ready
                      │
          dispatch     │    ◄─────── interrupt
                      ▼
                   running
                      │
         I/O wait      │    ◄─────── I/O complete
                      ▼
                   waiting
                      │
          exit         │
                      ▼
                  terminated
```

| State | Meaning |
|-------|---------|
| **New** | Process is being created |
| **Ready** | Waiting for CPU time |
| **Running** | Currently executing on the CPU |
| **Waiting** | Blocked, waiting for I/O or an event |
| **Terminated** | Finished execution |

A process can only be **Running** on one CPU core at a time.

---

## 4. Process Control Block (PCB)

The OS keeps track of every process using a data structure called the **Process Control Block (PCB)**.

It stores:
- Process ID (PID)
- Process state
- Program counter
- CPU registers
- Memory limits
- List of open files
- Scheduling priority

When the OS switches from one process to another (a **context switch**), it saves the current process's PCB and loads the next one.

### Context Switch Cost

Context switches have overhead — saving and restoring state takes time. Too many context switches per second can slow the system down.

---

## 5. Process Creation and Termination

### Creation — `fork()`

In Unix/Linux, a new process is created by **forking** an existing one.

```c
#include <unistd.h>
#include <stdio.h>

int main() {
    pid_t pid = fork();

    if (pid == 0) {
        printf("I am the child process\n");
    } else {
        printf("I am the parent process\n");
    }
    return 0;
}
```

- `fork()` creates an exact copy (child) of the calling process (parent)
- Both processes continue from the point of the fork
- `pid == 0` in the child; `pid > 0` (child's PID) in the parent

### Termination — `exit()`

A process ends by calling `exit()` or when its main function returns. The OS reclaims its resources.

---

## 6. CPU Scheduling

The OS needs to decide: **which ready process gets the CPU next?**

This is the job of the **CPU scheduler** (also called the short-term scheduler).

### Goals of Scheduling

- **Maximize CPU utilization** — keep the CPU busy
- **Maximize throughput** — complete as many processes as possible
- **Minimize turnaround time** — time from submission to completion
- **Minimize waiting time** — time spent in the ready queue
- **Minimize response time** — time until the first response

---

## 7. Scheduling Algorithms

### 7.1 First-Come, First-Served (FCFS)

Processes are served in the order they arrive. Simple, but can cause the **convoy effect** — short processes stuck waiting behind a long one.

**Example:**

| Process | Burst Time |
|---------|-----------|
| P1 | 24 ms |
| P2 | 3 ms |
| P3 | 3 ms |

Order: P1 → P2 → P3

- Average waiting time: (0 + 24 + 27) / 3 = **17 ms**

If order were P2 → P3 → P1: average waiting time = (0 + 3 + 6) / 3 = **3 ms** — much better!

---

### 7.2 Shortest Job First (SJF)

The process with the **shortest burst time** runs next. Optimal for minimizing average waiting time, but requires knowing burst time in advance (often not possible).

---

### 7.3 Round Robin (RR)

Each process gets a fixed **time quantum** (e.g., 10 ms). If it doesn't finish, it goes back to the end of the ready queue.

**Example (quantum = 4 ms):**

| Process | Burst Time |
|---------|-----------|
| P1 | 10 ms |
| P2 | 4 ms |
| P3 | 5 ms |

Execution order: P1(4) → P2(4) → P3(4) → P1(4) → P3(1) → P1(2)

- Fair — everyone gets CPU time
- Good for interactive systems
- Too small a quantum → too many context switches; too large → behaves like FCFS

---

### 7.4 Priority Scheduling

Each process gets a priority number. The highest priority process runs next.

- Can be **preemptive** (a higher-priority arrival interrupts the running process) or **non-preemptive**
- Risk: **starvation** — low-priority processes may never run
- Fix: **aging** — gradually increase priority of waiting processes

---

### 7.5 Multilevel Queue Scheduling

Processes are divided into groups (e.g., interactive vs. batch), each with its own queue and scheduling algorithm. Higher-priority queues are served first.

---

## 8. Scheduling Algorithm Comparison

| Algorithm | Preemptive? | Starvation? | Best For |
|-----------|-------------|-------------|----------|
| FCFS | No | No | Simple batch systems |
| SJF | Optional | Yes | Minimizing average wait |
| Round Robin | Yes | No | Interactive systems |
| Priority | Optional | Yes | Systems with task priority |
| Multilevel Queue | Yes | Possible | Mixed workloads |

---

## 9. Real-World Example: Your Computer Right Now

Open Task Manager (Windows) or Activity Monitor (macOS). You'll see dozens of processes in various states:

- Your browser — **Running** (or **Ready**)
- A background updater — **Waiting** (asleep until a timer fires)
- A finished installer — **Terminated** (about to be cleaned up)

The OS scheduler is juggling all of these, typically using a variant of **priority + round robin**, hundreds of times per second.

---

## 10. Summary

| Concept | Key Takeaway |
|---------|-------------|
| Process | A running program with its own memory and state |
| PCB | Data structure the OS uses to track each process |
| Process states | New → Ready → Running → Waiting → Terminated |
| Context switch | Saving/loading process state when switching CPUs |
| Scheduling | Deciding which ready process runs next |
| FCFS | Simple, but convoy effect hurts short jobs |
| SJF | Optimal average wait, but needs burst time prediction |
| Round Robin | Fair, good for interactive use |
| Priority | Fast for important jobs; risks starvation |

---

## Module 2 Quiz

---

**Question 1**
What is the difference between a program and a process?

> **Answer:** A program is a passive file stored on disk. A process is an active instance of that program currently running in memory, with its own state, memory space, and resources.

---

**Question 2**
A process is waiting for data to be read from a hard drive. What state is it in?

- A) Running
- B) Ready
- C) Waiting
- D) Terminated

> **Answer: C** — The process is blocked on an I/O operation and moves to the Waiting state until the data is ready.

---

**Question 3**
Three processes arrive at the same time with burst times: P1 = 8ms, P2 = 2ms, P3 = 5ms.
Using **SJF**, what is the execution order and average waiting time?

> **Answer:**
> Order: P2 → P3 → P1
> - P2 waits: 0 ms
> - P3 waits: 2 ms
> - P1 waits: 7 ms
> - Average waiting time: (0 + 2 + 7) / 3 = **3 ms**

---

**Question 4**
What problem does **aging** solve in priority scheduling?

> **Answer:** Aging solves starvation. Low-priority processes may never get CPU time if higher-priority processes keep arriving. Aging gradually increases the priority of waiting processes over time, ensuring they eventually run.

---

**Question 5**
Round Robin scheduling uses a time quantum of 5 ms. A process needs 18 ms total to complete. How many times will it be placed back on the ready queue before finishing?

- A) 2
- B) 3
- C) 4
- D) 5

> **Answer: B** — The process runs for 5ms (→ back to queue), 5ms (→ back to queue), 5ms (→ back to queue), then finishes with its remaining 3ms. It is placed back on the queue **3 times** and completes on its 4th turn.

---

## What's Next

**Module 3: Concurrency**

You'll learn how multiple processes (and threads) can run simultaneously, why that creates tricky problems like race conditions, and how the OS uses tools like semaphores and mutexes to keep things orderly.

---

*CS 2301 Operating Systems 1 — Module 2 of 8*
