# CS 2301 — Operating Systems 1
## Module 8: Advanced Topics and Case Studies

---

> **Course Progress:** Module 8 of 8 — CS 2301 (OS 1) ✓

---

## 1. Bringing It All Together

This final module zooms out. You have learned how an OS manages processes, memory, storage, I/O, and security. Now we look at how these ideas combine in real systems, explore modern extensions like virtualization and containers, and examine two dominant OSes — Linux and Windows — as case studies.

---

## 2. Virtualization

**Virtualization** allows one physical machine to run multiple **virtual machines (VMs)**, each behaving as if it has its own CPU, memory, and devices.

```
+------------------+  +------------------+
|   VM 1           |  |   VM 2           |
|  (Windows 11)    |  |  (Ubuntu Linux)  |
+------------------+  +------------------+
|         Hypervisor (VMware, Hyper-V)   |
+----------------------------------------+
|           Physical Hardware            |
+----------------------------------------+
```

### The Hypervisor

A **hypervisor** (also called a Virtual Machine Monitor — VMM) is the software layer that creates and manages VMs. It sits between the hardware and the guest OSes.

| Type | Description | Example |
|------|-------------|---------|
| **Type 1 (Bare Metal)** | Runs directly on hardware | VMware ESXi, Hyper-V, Xen |
| **Type 2 (Hosted)** | Runs on top of a host OS | VirtualBox, VMware Workstation |

### Why Virtualization Matters

- **Server consolidation** — run 20 VMs on one physical server instead of 20 servers
- **Isolation** — a crash in one VM doesn't affect others
- **Snapshots** — save a VM's state and roll back if something goes wrong
- **Cloud computing** — AWS, Azure, and Google Cloud are built on virtualization; when you rent a server, you're renting a VM

### How the OS Topics Apply to VMs

Every concept from this course applies at two levels in a virtualized environment:

| Topic | In a VM |
|-------|---------|
| Process management | The guest OS manages its processes; the hypervisor manages VMs like processes |
| Memory management | The hypervisor adds a second layer of address translation |
| I/O management | Device drivers communicate with virtual (emulated) hardware |
| Security | VMs are isolated, but a hypervisor vulnerability can compromise all VMs |

---

## 3. Containers

**Containers** are a lighter alternative to VMs. Instead of virtualizing the entire hardware, containers share the host OS kernel but isolate the application's environment.

```
+------------+  +------------+  +------------+
| Container A|  | Container B|  | Container C|
| (App + libs)|  | (App + libs)|  | (App + libs)|
+------------+  +------------+  +------------+
|         Container Runtime (Docker)         |
+--------------------------------------------+
|               Host OS Kernel               |
+--------------------------------------------+
|               Physical Hardware            |
+--------------------------------------------+
```

### VMs vs. Containers

| | Virtual Machine | Container |
|-|----------------|-----------|
| Isolation | Full OS isolation | Process-level isolation |
| Startup time | Minutes | Seconds |
| Size | GBs | MBs |
| OS | Each VM has its own | All share host kernel |
| Use case | Different OSes, strong isolation | Microservices, fast deployment |

### How Containers Work at the OS Level

Containers use two Linux kernel features:

- **Namespaces** — isolate what a process can see (its own process list, network, filesystem)
- **cgroups (control groups)** — limit how much CPU, memory, and I/O a container can use

Docker, Kubernetes, and modern cloud deployments are built entirely on these OS primitives.

---

## 4. Multiprocessor and Multicore Systems

Modern CPUs have multiple **cores** — independent processing units on one chip. The OS must manage them.

```
CPU Chip
+-----------------------------------+
|  Core 1  |  Core 2  |  Core 3  | Core 4 |
+-----------------------------------+
|         Shared L3 Cache           |
+-----------------------------------+
```

### Symmetric Multiprocessing (SMP)

All cores share the same memory and run the same OS. The scheduler can assign any process to any core. Most modern OSes (Linux, Windows) use SMP.

### Challenges

- **Cache coherence** — if Core 1 and Core 2 both cache the same memory location and one modifies it, they must stay in sync
- **Scheduling complexity** — the scheduler must now decide not just *when* a process runs, but *which core* it runs on
- **Lock contention** — synchronization primitives (mutexes, semaphores) become bottlenecks when many cores compete for the same lock

---

## 5. Real-Time Operating Systems (Revisited)

Introduced in Module 1, RTOS deserves a closer look in context.

An RTOS guarantees that tasks complete within a **deadline**. There are two types:

| Type | Deadline Miss Consequence | Example |
|------|--------------------------|---------|
| **Hard real-time** | Catastrophic (system failure) | Aircraft flight control, pacemaker |
| **Soft real-time** | Degraded quality, acceptable | Video streaming, online gaming |

### RTOS Scheduling

RTOS schedulers prioritize **deadline** over fairness:

- **Rate Monotonic Scheduling (RMS)** — higher-frequency tasks get higher priority
- **Earliest Deadline First (EDF)** — the task with the nearest deadline runs next

General-purpose OSes like Linux can approximate soft real-time with the `SCHED_DEADLINE` policy, but cannot guarantee hard real-time.

---

## 6. Case Study — Linux

Linux is the dominant OS for servers, cloud, Android (mobile), and embedded systems.

### Architecture

```
+-------------------------------+
|       User Applications       |
+-------------------------------+
|    GNU C Library (glibc)      |
+-------------------------------+
|        Linux Kernel           |
|  ┌──────────────────────────┐ |
|  │ Process | Memory | VFS   │ |
|  │ Sched.  | Mgmt.  | I/O   │ |
|  └──────────────────────────┘ |
+-------------------------------+
|           Hardware            |
+-------------------------------+
```

### Key Linux Characteristics

| Feature | Detail |
|---------|--------|
| **Kernel type** | Monolithic (but modular — drivers load as modules) |
| **Scheduler** | Completely Fair Scheduler (CFS) — proportional CPU time |
| **File systems** | ext4 (default), also Btrfs, XFS, ZFS |
| **Process model** | `fork()` / `exec()` — everything is a child of `init` (PID 1) |
| **Security** | DAC (file permissions) + SELinux/AppArmor for mandatory access control |
| **License** | Open source (GPL) |

### The Linux Process Tree

```
init (PID 1)
├── systemd services
├── login
│   └── bash (your shell)
│       ├── vim
│       └── python script.py
└── cron
    └── backup.sh
```

Every process has a parent. `init` (or `systemd`) is the ancestor of all processes. When a parent exits before its child, the child is **adopted** by init.

### Everything Is a File (Revisited)

```bash
/dev/sda        # Hard drive
/dev/null       # Discard output (black hole)
/dev/random     # Random number generator
/proc/1/status  # Live info about process 1
/sys/class/net  # Network interfaces
```

All of these are "files" you can read/write with standard tools.

---

## 7. Case Study — Windows

Windows is dominant on desktops and is widely used in enterprise environments.

### Architecture

```
+-----------------------------+
|     User Applications       |
+-----------------------------+
|   Win32 / WinRT API Layer   |
+-----------------------------+
|      Windows Subsystem      |
+-----------------------------+
|       NT Kernel (ntoskrnl)  |
|  ┌─────────────────────────┐|
|  │ Proc. | Mem. | I/O Mgr  ││
|  │ Sched.| Mgmt.| Obj. Mgr ││
|  └─────────────────────────┘|
+-----------------------------+
|     Hardware Abstraction    |
|          Layer (HAL)        |
+-----------------------------+
|         Hardware            |
+-----------------------------+
```

### Key Windows Characteristics

| Feature | Detail |
|---------|--------|
| **Kernel type** | Hybrid (microkernel-inspired, but much runs in kernel space) |
| **Scheduler** | Priority-based preemptive, with priority boost for foreground windows |
| **File system** | NTFS (primary), ReFS (resilient, for servers) |
| **Process model** | `CreateProcess()` — no fork(); each process starts fresh |
| **Security** | ACLs on every object; UAC (User Account Control) for privilege elevation |
| **Registry** | Centralized configuration database (unlike Linux's text files) |

### Windows vs. Linux — Side by Side

| | Linux | Windows |
|-|-------|---------|
| Kernel | Monolithic + modules | Hybrid |
| Config storage | Text files (`/etc`) | Registry |
| Process creation | `fork()` + `exec()` | `CreateProcess()` |
| Shell | bash, zsh | PowerShell, cmd |
| Package management | apt, dnf, pacman | winget, Microsoft Store |
| Primary use | Servers, cloud, embedded | Desktop, enterprise |
| License | Open source (GPL) | Proprietary |

---

## 8. Emerging Topics

### 8.1 Operating Systems for the Cloud

Cloud OSes are optimized for rapid VM/container startup, elastic scaling, and multi-tenant isolation. Examples: Amazon's custom Linux variants for EC2, Google's gVisor (container sandbox).

### 8.2 Unikernels

A **unikernel** compiles the application and only the OS components it needs into a single, tiny executable. No shell, no unused drivers, no extra processes — minimal attack surface and very fast boot.

### 8.3 Exokernels

An **exokernel** pushes most OS abstractions out of the kernel and into user-space libraries, giving applications more direct control over hardware. Mostly a research concept, but influential in OS design thinking.

### 8.4 Operating Systems for IoT

Devices like thermostats, sensors, and industrial controllers need ultra-lightweight OSes. Examples: FreeRTOS, Zephyr. They must fit in kilobytes of memory, run on low-power hardware, and often have hard real-time requirements.

---

## 9. CS 2301 — Full Course Review

| Module | Topic | Core Idea |
|--------|-------|-----------|
| 1 | Introduction | OS as resource manager and abstraction layer |
| 2 | Process Management | Processes, states, PCB, CPU scheduling algorithms |
| 3 | Concurrency | Threads, race conditions, mutexes, deadlock |
| 4 | Memory Management | Paging, virtual memory, page replacement |
| 5 | Storage & Shell | Disk scheduling, file systems, shell scripting |
| 6 | I/O Management | Polling, interrupts, DMA, buffering, caching |
| 7 | Security | CIA triad, authentication, ACL, encryption |
| 8 | Advanced Topics | Virtualization, containers, Linux, Windows |

---

## Module 8 Quiz

---

**Question 1**
What is the difference between a Type 1 and Type 2 hypervisor? Give a real example of each.

> **Answer:** A Type 1 (bare-metal) hypervisor runs directly on the physical hardware — there is no host OS underneath it. It is faster and more efficient. Example: VMware ESXi, Microsoft Hyper-V. A Type 2 (hosted) hypervisor runs as an application on top of an existing OS. Easier to set up but adds overhead. Example: VirtualBox, VMware Workstation.

---

**Question 2**
Containers share the host OS kernel, but VMs do not. What are the security implications of each approach?

> **Answer:** VMs provide stronger isolation — a vulnerability in one VM's guest OS does not directly affect others or the host. The attack surface between VMs is limited to the hypervisor. Containers share the kernel, so a kernel vulnerability could potentially allow one container to escape and affect others or the host. Containers are faster and lighter, but the trade-off is reduced isolation. For workloads requiring strong security boundaries (e.g., running untrusted code), VMs are preferred.

---

**Question 3**
Linux uses `fork()` to create processes; Windows uses `CreateProcess()`. What is the fundamental difference?

> **Answer:** `fork()` creates a near-identical copy of the calling process — the child starts with the parent's memory, open files, and state. The child then typically calls `exec()` to replace itself with a new program. `CreateProcess()` starts fresh — it creates a new process and loads a specified program directly, without copying the parent. Linux's model is more flexible for shell pipelines; Windows's model avoids the overhead of copying then replacing.

---

**Question 4**
A company runs a web server in a Docker container limited to 2 CPU cores and 512 MB of RAM. Which two Linux kernel features make these limits possible?

> **Answer:** **cgroups (control groups)** enforce resource limits — they cap how much CPU, memory, and I/O a container can consume. **Namespaces** provide isolation — the container sees only its own processes, network interfaces, and filesystem. Together they are the OS-level foundation that Docker and all container runtimes are built upon.

---

**Question 5**
You have finished CS 2301. Describe in your own words what an operating system does, using at least four concepts from the course.

> **Answer (example):** An operating system is the software that manages a computer's hardware and provides services to running programs. It uses **process management** to create, schedule, and terminate processes — deciding which gets CPU time using algorithms like Round Robin or Priority Scheduling. It uses **memory management** techniques like paging and virtual memory to give each process its own address space while efficiently using physical RAM. It handles **I/O management** — communicating with disks, keyboards, and networks using interrupts and DMA so the CPU stays free during slow operations. It enforces **security and protection** through file permissions, authentication, and dual-mode operation, ensuring processes cannot interfere with each other or with the OS itself.

---

## Congratulations — CS 2301 Complete!

You have covered the full Operating Systems 1 curriculum:

```
✓ Module 1 — Introduction to Operating Systems
✓ Module 2 — Process Management & CPU Scheduling
✓ Module 3 — Concurrency
✓ Module 4 — Memory Management
✓ Module 5 — Storage Management and Shell Scripting
✓ Module 6 — Input / Output Management
✓ Module 7 — Security and Protection
✓ Module 8 — Advanced Topics and Case Studies
```

---

## What's Next — CS 3307 Operating Systems 2

| Module | Topic |
|--------|-------|
| 1 | Modern Operating Systems |
| 2 | Parallel Computing and Concurrency |
| 3 | Memory and File Structure |
| 4 | File Handling |
| 5 | File Processing |
| 6 | Command Shells and Control |
| 7 | Operating System Utilities |
| 8 | Graphical User Interfaces |

OS 2 builds directly on this foundation — going deeper into parallelism, advanced file handling, shell mastery, and how modern GUIs interact with the OS.

---

*CS 2301 Operating Systems 1 — Module 8 of 8 — Course Complete*
