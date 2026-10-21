# CS 2301 — Operating Systems 1
## Module 1: Introduction to Operating Systems

---

> **Course Progress:** Module 1 of 8 — CS 2301 (OS 1)

---

## 1. What is an Operating System?

An **Operating System (OS)** is the software layer that sits between the hardware and the applications you use. Think of it as a manager: it controls who gets to use the CPU, memory, disk, and other resources — and when.

Without an OS, every application would need to directly control hardware, which is impractical and unsafe.

**Core roles of an OS:**

- **Resource Manager** — allocates CPU time, memory, and devices
- **Abstraction Layer** — hides hardware complexity from programs
- **Interface Provider** — gives users and programs a way to interact with the system
- **Enforcer** — prevents programs from interfering with each other

---

## 2. A Brief History

| Era | Key Development |
|-----|----------------|
| 1950s | No OS — programmers ran jobs directly on hardware |
| 1960s | Batch systems (jobs queued and processed one by one) |
| 1970s | Timesharing — multiple users share one machine (UNIX born here) |
| 1980s | Personal computers, MS-DOS, early GUIs |
| 1990s | Windows, Linux, networking-focused OS |
| 2000s–today | Mobile OS (Android, iOS), cloud, virtualization, containers |

---

## 3. Types of Operating Systems

### 3.1 Batch OS
Jobs are collected and executed in groups. No user interaction during execution.

**Example:** Early IBM mainframe jobs submitted on punch cards.

### 3.2 Timesharing OS
The CPU rapidly switches between multiple users, giving the illusion each has their own machine.

**Example:** UNIX on university servers in the 1970s–80s.

### 3.3 Real-Time OS (RTOS)
Designed to respond to events within strict time constraints.

**Example:** Software controlling an airplane autopilot or a pacemaker.

### 3.4 Distributed OS
Manages a group of independent machines and makes them appear as one system.

**Example:** Google's internal cluster management systems.

### 3.5 Mobile OS
Optimized for battery life, touch interfaces, and cellular connectivity.

**Example:** Android, iOS.

---

## 4. OS Structure

Modern operating systems are organized in layers:

```
+---------------------------+
|     User Applications     |
+---------------------------+
|    System Libraries       |
+---------------------------+
|       Kernel              |   <-- Core of the OS
+---------------------------+
|       Hardware            |
+---------------------------+
```

The **kernel** is the heart of the OS. It runs in **privileged mode** (also called kernel mode), meaning it has full access to hardware. Applications run in **user mode**, with restricted access.

### Kernel Types

| Type | Description | Example |
|------|-------------|---------|
| Monolithic | All OS services run in kernel space | Linux |
| Microkernel | Only essential services in kernel; rest in user space | Minix |
| Hybrid | Mix of both approaches | Windows NT, macOS |

---

## 5. System Calls

Applications communicate with the OS through **system calls** — controlled entry points into the kernel.

**Common examples:**

- `open()` — open a file
- `read()` / `write()` — read from or write to a file/device
- `fork()` — create a new process
- `exit()` — terminate a process

### Example: Opening a file in C

```c
#include <fcntl.h>

int fd = open("notes.txt", O_RDONLY);
if (fd == -1) {
    // File could not be opened
}
```

When `open()` is called, execution switches from user mode to kernel mode, the OS handles the request, then returns control to the program.

---

## 6. OS Services

The OS provides several key services to programs and users:

- **Program execution** — load and run programs
- **I/O operations** — manage access to devices (keyboard, disk, screen)
- **File system management** — create, read, write, delete files
- **Communication** — pass data between processes
- **Error detection** — handle hardware/software faults
- **Resource allocation** — divide CPU, memory, devices among multiple users
- **Protection & Security** — control who can access what

---

## 7. Real-World Example: What Happens When You Open a Browser?

1. You double-click the browser icon → the OS receives the request
2. OS allocates **memory** for the browser process
3. OS loads the program from **disk** into memory
4. OS schedules **CPU time** for the browser
5. You click a link → OS handles **network I/O** on behalf of the browser
6. The page renders → OS manages **display output**
7. You close the browser → OS **frees resources** (memory, file handles)

Every step involves the OS working behind the scenes.

---

## 8. Summary

| Concept | Key Takeaway |
|---------|-------------|
| What is an OS | Software that manages hardware and provides services to programs |
| OS roles | Resource manager, abstraction layer, interface, enforcer |
| Kernel | Core of the OS; runs in privileged mode |
| System calls | How applications request services from the OS |
| OS types | Batch, timesharing, real-time, distributed, mobile |

---

## Module 1 Quiz

Test your understanding. Try to answer before checking the explanations.

---

**Question 1**
What is the primary purpose of an operating system?

- A) To run user applications as fast as possible
- B) To manage hardware resources and provide services to programs
- C) To connect computers to the internet
- D) To store files on disk

> **Answer: B** — The OS manages hardware (CPU, memory, devices) and provides a set of services (file system, process management, I/O) that programs rely on.

---

**Question 2**
A factory uses software to control robotic arms that must react within 2 milliseconds. What type of OS is most appropriate?

- A) Batch OS
- B) Distributed OS
- C) Real-Time OS
- D) Mobile OS

> **Answer: C** — A Real-Time OS (RTOS) is designed for systems where tasks must complete within strict deadlines. Robotic control and medical devices are classic RTOS use cases.

---

**Question 3**
What is the difference between user mode and kernel mode?

> **Answer:** User mode is a restricted execution mode for regular programs — they cannot directly access hardware. Kernel mode is a privileged mode where the OS kernel runs and has full hardware access. This separation protects the system from misbehaving programs.

---

**Question 4**
Which of the following is a system call?

- A) `printf()`
- B) `sqrt()`
- C) `fork()`
- D) `strlen()`

> **Answer: C** — `fork()` is a system call that creates a new process, which requires kernel involvement. The others are standard library functions that don't require switching to kernel mode.

---

**Question 5**
A university has one powerful server that 200 students use simultaneously for coding assignments. Which OS type best describes this setup?

- A) Batch OS
- B) Timesharing OS
- C) Real-Time OS
- D) Mobile OS

> **Answer: B** — Timesharing allows multiple users to share a single machine by rapidly switching between their sessions, giving each user the impression of having their own machine.

---

## What's Next

**Module 2: Process Management & CPU Scheduling**

You'll learn what a process is, how the OS creates and manages them, and how the CPU decides which process runs next.

---

*CS 2301 Operating Systems 1 — Module 1 of 8*
