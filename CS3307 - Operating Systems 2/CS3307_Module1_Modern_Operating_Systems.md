# CS 3307 — Operating Systems 2
## Module 1: Modern Operating Systems

---

> **Course Progress:** Module 1 of 8 — CS 3307 (OS 2)

---

## 1. Welcome to OS 2

CS 2301 covered the foundations — processes, memory, storage, I/O, and security. CS 3307 builds on those foundations and goes deeper into how modern systems actually work at scale.

This module surveys the landscape of modern operating systems: what has changed since the early days, what drives OS design today, and how the major systems compare.

---

## 2. What Has Changed in Modern OS Design

Early OSes were designed for a single user on a single machine with limited resources. Today's OS must handle:

| Then | Now |
|------|-----|
| Single core CPU | Multi-core, many-core CPUs |
| MB of RAM | GB–TB of RAM |
| Local storage only | Network, cloud, distributed storage |
| One user at a time | Thousands of concurrent users (servers) |
| Isolated machine | Always-connected, networked |
| Batch jobs | Real-time, interactive, streaming workloads |
| Physical hardware only | Virtual machines and containers |

The core abstractions (processes, files, memory) are the same — but the scale, complexity, and environment have changed dramatically.

---

## 3. Key Design Goals of Modern OSes

Modern OS design is driven by several competing goals:

### 3.1 Performance
Minimize latency, maximize throughput. Every OS decision — scheduling, memory layout, I/O handling — has a performance cost.

### 3.2 Scalability
The OS should work well whether running on a single core laptop or a 128-core server. Locking strategies, scheduler design, and memory management must all scale.

### 3.3 Reliability and Fault Tolerance
A server OS cannot crash every time a driver fails. Modern OSes use techniques like:
- **Driver isolation** — faulty drivers are contained
- **Journaling file systems** — recover cleanly after a crash
- **Watchdog timers** — detect and restart hung processes automatically

### 3.4 Security
Modern threat models assume attackers are constantly probing. OSes now build in:
- Mandatory access control (SELinux, AppArmor)
- Secure boot
- Memory-safe execution environments
- Hardware-level isolation (Intel TXT, ARM TrustZone)

### 3.5 Energy Efficiency
Mobile and cloud OSes must balance performance with power consumption. Techniques include:
- CPU frequency scaling (run slower when idle)
- Core parking (shut down idle cores entirely)
- Intelligent I/O scheduling to keep disks spun down

---

## 4. Modern Kernel Architectures

You saw these briefly in CS 2301. Here is a deeper look.

### 4.1 Monolithic Kernel

All OS services — scheduling, memory management, file systems, device drivers — run in one large kernel space.

```
+------------------------------------------+
|   Scheduler | Memory Mgr | FS | Drivers  |  <- All in kernel space
+------------------------------------------+
|                 Hardware                  |
+------------------------------------------+
```

- **Pros:** Fast (no context switching between components), mature, well-optimized
- **Cons:** One buggy driver can crash the whole kernel
- **Examples:** Linux, older Unix systems

Linux is monolithic but uses **loadable kernel modules (LKMs)** — drivers and file systems can be loaded/unloaded at runtime without rebooting.

```bash
lsmod          # list loaded kernel modules
modprobe ext4  # load a module
rmmod ext4     # remove a module
```

### 4.2 Microkernel

Only the most essential services run in kernel space (IPC, basic memory management, scheduling). Everything else — file systems, drivers, networking — runs in user space.

```
+-------------+  +----------+  +---------+
|  FS Server  |  |  Driver  |  |  Net    |  <- User space
+-------------+  +----------+  +---------+
+------------------------------------------+
|   Microkernel (IPC + basic memory)       |  <- Kernel space
+------------------------------------------+
|                 Hardware                  |
+------------------------------------------+
```

- **Pros:** Better fault isolation (a crashed driver does not take down the kernel), cleaner design
- **Cons:** More context switches between components — slower
- **Examples:** Minix, QNX (used in cars and medical devices), seL4

### 4.3 Hybrid Kernel

Combines elements of both — some services run in kernel space for performance, others in user space for isolation.

- **Examples:** Windows NT, macOS (XNU — combines Mach microkernel with BSD Unix components)

### 4.4 Exokernel and Unikernel (Research / Emerging)

- **Exokernel:** Exposes hardware directly to applications; OS abstractions live in user-space libraries.
- **Unikernel:** Application + only the OS code it needs, compiled into a single executable. Used in cloud functions and embedded systems.

---

## 5. Major Modern Operating Systems

### 5.1 Linux

The backbone of the modern internet, cloud, and Android ecosystem.

- Kernel: Monolithic + loadable modules
- Scheduler: Completely Fair Scheduler (CFS) + real-time policies
- Used in: Web servers, cloud VMs, Android, supercomputers, embedded devices
- Market share: ~96% of the world's top 1 million web servers

```
Android
   └── Linux Kernel (modified)
           └── ART runtime + Android framework on top
```

### 5.2 Windows

Dominant on desktops; significant in enterprise servers.

- Kernel: Hybrid (NT kernel)
- Key subsystems: Win32, WinRT, WSL (Windows Subsystem for Linux)
- **WSL** — allows running Linux binaries natively on Windows
- Used in: Desktop PCs, enterprise, gaming, Azure cloud

### 5.3 macOS

Apple's desktop OS, built on a Unix foundation.

- Kernel: XNU (hybrid — Mach + BSD)
- POSIX-compliant — Unix programs run natively
- Tight hardware-software integration (especially on Apple Silicon)
- Used in: Developer workstations, creative professionals

### 5.4 Android

The world's most widely used OS by device count.

- Built on a modified Linux kernel
- Applications run in the **ART (Android Runtime)**
- Each app is sandboxed — its own user ID, isolated from other apps
- Used in: Smartphones, tablets, TVs, cars

### 5.5 Real-Time and Embedded OSes

| OS | Used In |
|----|---------|
| FreeRTOS | Microcontrollers, IoT sensors |
| Zephyr | Industrial IoT, wearables |
| QNX | Car infotainment, medical devices |
| VxWorks | Aerospace, defense, Mars rovers |

---

## 6. The POSIX Standard

**POSIX (Portable Operating System Interface)** defines a common API for Unix-like OSes. A program written to the POSIX standard should compile and run on any POSIX-compliant OS.

Key POSIX components:
- Process creation: `fork()`, `exec()`, `wait()`
- File I/O: `open()`, `read()`, `write()`, `close()`
- Signals: `kill()`, `signal()`
- Threads: `pthread_create()`, `pthread_join()`

Linux, macOS, and most Unix systems are POSIX-compliant. Windows is not fully compliant, but WSL provides a POSIX layer.

```c
// This code runs on Linux, macOS, and any POSIX system unchanged
#include <unistd.h>
#include <fcntl.h>

int fd = open("data.txt", O_RDONLY);
char buf[256];
read(fd, buf, 256);
close(fd);
```

---

## 7. System Calls Across OSes

| Operation | Linux | Windows |
|-----------|-------|---------|
| Create process | `fork()` / `exec()` | `CreateProcess()` |
| Open file | `open()` | `CreateFile()` |
| Read file | `read()` | `ReadFile()` |
| Create thread | `clone()` / pthreads | `CreateThread()` |
| Allocate memory | `mmap()` / `brk()` | `VirtualAlloc()` |

This is why programs are not directly portable between Linux and Windows at the binary level.

---

## 8. Modern OS Trends

### Convergence of Desktop and Mobile
macOS and iOS share a kernel. Windows 11 runs Android apps. The line between desktop and mobile is blurring at the OS level.

### Kernel Bypass
High-performance systems bypass the OS kernel entirely for I/O using technologies like **io_uring** — reducing latency to microseconds.

### OS in the Cloud
AWS Firecracker uses a microVM model — booting a minimal Linux kernel in under 125ms for serverless functions.

### Security by Default
Modern OSes ship with Secure Boot, encrypted storage, sandboxed apps, and hardware-backed key storage (TPM chips) as standard.

---

## 9. Real-World Example: Choosing an OS for a Project

| Scenario | Best OS Choice | Why |
|----------|---------------|-----|
| Web server hosting 10,000 users | Linux (Ubuntu Server) | Performance, free, dominant ecosystem |
| Developer laptop | macOS | POSIX-compliant, Unix tools, great hardware |
| Corporate office PCs | Windows | Active Directory, enterprise software |
| Car infotainment system | QNX | Hard real-time, safety-certified |
| IoT temperature sensor | FreeRTOS | Fits in KB of RAM, ultra low power |
| Cloud serverless function | Linux (Firecracker microVM) | Sub-second boot, minimal footprint |

---

## 10. Summary

| Concept | Key Takeaway |
|---------|-------------|
| Modern OS challenges | Scale, multicore, networking, security, energy efficiency |
| Monolithic kernel | All services in kernel space; fast but fragile |
| Microkernel | Minimal kernel; services in user space; safer but slower |
| Hybrid kernel | Mix of both; used by Windows and macOS |
| POSIX | Common API standard for Unix-like OSes |
| Linux | Open source monolithic kernel; dominates servers and cloud |
| Windows | Hybrid kernel; dominates desktop and enterprise |
| macOS | XNU (Mach + BSD); POSIX-compliant |
| Android | Linux kernel + ART runtime; world's most used OS |
| Embedded OSes | FreeRTOS, QNX — tiny, real-time, safety-critical |

---

## Module 1 Quiz

---

**Question 1**
What are three key design goals of a modern OS, and how do they sometimes conflict?

> **Answer:** Three key goals are **performance**, **security**, and **energy efficiency**. They conflict in several ways: security mechanisms (encryption, access checks, sandboxing) add overhead that reduces performance. Energy efficiency may require slowing the CPU, which reduces performance. High-performance computing keeps all cores at full speed, which drains power. Every OS makes trade-offs depending on its target use case.

---

**Question 2**
What is the main reliability advantage of a microkernel over a monolithic kernel?

> **Answer:** In a microkernel, device drivers and file system code run in user space. If a driver crashes, it crashes as a user-space process and can be restarted without bringing down the OS. In a monolithic kernel, a buggy driver runs in kernel space — one crash can take down the whole system (kernel panic / blue screen).

---

**Question 3**
What is POSIX and why does it matter for software portability?

> **Answer:** POSIX is a set of standards defining a common OS API — system calls for file I/O, processes, threads, and signals. Software written to POSIX can compile and run on any compliant OS (Linux, macOS, most Unix systems) with little modification. Without POSIX, developers would rewrite OS-level code for every platform. Windows is not fully POSIX-compliant, which is why Unix/Linux programs need WSL to run on Windows.

---

**Question 4**
Android is based on Linux, but Android apps don't run directly on the Linux kernel. What sits between them?

> **Answer:** The **ART (Android Runtime)**. Android apps are written in Java or Kotlin and compiled to bytecode. ART executes this bytecode on top of the Linux kernel. The Linux kernel handles low-level hardware management; ART handles running the apps in isolated sandboxes.

---

**Question 5**
A startup is building a medical infusion pump. Should they use Linux, Windows, or QNX? Justify your answer.

> **Answer:** They should use **QNX** (or a similar certified RTOS). A medical infusion pump is a **hard real-time** system — missing a deadline can be life-threatening. QNX is a microkernel RTOS certified for safety-critical use. It guarantees task deadlines, has a minimal attack surface, and its microkernel design means a component crash does not bring down the whole system. Linux and Windows are not designed for hard real-time guarantees and are not certified for life-critical medical use.

---

## What's Next

**Module 2: Parallel Computing and Concurrency**

You will go deeper than CS 2301's introduction — exploring parallel programming models, advanced synchronization, lock-free data structures, and how modern multi-core hardware affects concurrent software.

---

*CS 3307 Operating Systems 2 — Module 1 of 8*
