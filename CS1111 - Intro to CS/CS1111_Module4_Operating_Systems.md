# CS 1111 – Introduction to Computer Science
## Module 4: Operating Systems

---

## Table of Contents
1. [What is an Operating System?](#1-what-is-an-operating-system)
2. [History and Evolution of Operating Systems](#2-history-and-evolution-of-operating-systems)
3. [Functions of an Operating System](#3-functions-of-an-operating-system)
4. [Process Management](#4-process-management)
5. [Memory Management](#5-memory-management)
6. [File System Management](#6-file-system-management)
7. [Device Management](#7-device-management)
8. [User Interface](#8-user-interface)
9. [Types of Operating Systems](#9-types-of-operating-systems)
10. [Popular Operating Systems](#10-popular-operating-systems)
11. [OS Security](#11-os-security)
12. [Module Summary](#12-module-summary)
13. [Module Quiz](#13-module-quiz)

---

## 1. What is an Operating System?

An **Operating System (OS)** is the most fundamental piece of software on a computer. It acts as an **intermediary** between the user, application software, and the hardware, managing all the computer's resources and providing a stable, consistent environment in which programs can run.

Without an OS, every application developer would have to write their own code to control the CPU, manage memory, talk to the keyboard, draw pixels on the screen — everything. The OS handles all of that so that application developers can focus on building useful programs.

### The OS as a Manager

Think of the OS as the **manager of a busy office building**:
- The **building** is the hardware (CPU, RAM, storage, devices).
- The **tenants** are the applications (browser, word processor, games).
- The **manager (OS)** allocates office space (memory), schedules meeting rooms (CPU time), controls who can access which files (security), and handles communication between departments (I/O).
- The tenants never deal with the building's plumbing or electricity directly — the manager handles all of that.

### The OS in the Software Stack

```
┌──────────────────────────────────────┐
│           User Applications          │  ← Chrome, Word, Games
├──────────────────────────────────────┤
│         Operating System             │  ← Windows, macOS, Linux
│   (Kernel + System Services + UI)    │
├──────────────────────────────────────┤
│              Hardware                │  ← CPU, RAM, Disk, GPU
└──────────────────────────────────────┘
```

The OS sits between hardware and applications, serving both. Applications request services from the OS via **system calls**, and the OS communicates directly with hardware using **device drivers**.

---

## 2. History and Evolution of Operating Systems

### 2.1 The 1940s–1950s: No Operating System

Early computers had no OS at all. Programmers operated the machine directly, loading programs via punch cards and flipping physical switches. Only one program could run at a time, and if it crashed, the entire machine stopped.

### 2.2 The 1950s–1960s: Batch Processing Systems

The first primitive OS emerged as **batch processing** systems. Programs (called **jobs**) were collected in batches and executed one after another without user interaction. A human **operator** would load decks of punch cards and the computer would process them in sequence.

- **Advantage:** The CPU was kept busy more consistently.
- **Limitation:** No interactivity; users submitted jobs and came back hours later for results.

> **Example:** IBM's early OS/360 introduced batch processing to business computing.

### 2.3 The 1960s–1970s: Time-Sharing and Multiprogramming

Researchers realized that the CPU sat idle a lot while waiting for slow I/O operations (reading a disk, waiting for a printer). **Multiprogramming** kept multiple jobs in memory and switched between them when one was waiting for I/O.

**Time-sharing** (a form of multitasking) took this further — giving each user a very short **time slice** of CPU time in rapid rotation, creating the illusion that multiple users were using the computer simultaneously.

> **Example:** UNIX (1969, Bell Labs) — one of the most influential operating systems ever created. It introduced time-sharing, a hierarchical file system, and the philosophy of small, composable tools.

### 2.4 The 1970s–1980s: Personal Computer OS

With the invention of microprocessors came personal computers, which needed simple, affordable operating systems.

- **CP/M (1974):** The first widely used OS for microcomputers.
- **MS-DOS (1981):** Microsoft's Disk Operating System for the IBM PC — text-based, single-user, single-tasking. Users typed commands at a prompt.
- **Apple DOS / ProDOS:** For the Apple II family.

### 2.5 The 1980s–1990s: Graphical User Interfaces

The introduction of the **GUI** transformed how people interacted with computers — replacing typed commands with visual windows, icons, and menus controlled by a mouse.

- **Macintosh OS (1984):** First commercially successful GUI-based OS.
- **Windows 1.0 (1985):** Microsoft's GUI layer over MS-DOS.
- **Windows 95 (1995):** A landmark release — integrated GUI, 32-bit support, plug-and-play hardware, and the iconic Start Menu.

### 2.6 The 2000s–Present: Modern, Networked, Mobile OS

- **Windows XP/7/10/11:** Consumer dominance on the desktop.
- **Linux (1991, Linus Torvalds):** Open-source Unix-like OS that now powers most servers, supercomputers, and Android devices.
- **macOS X (2001):** Unix-based with a polished GUI.
- **iOS (2007) / Android (2008):** Mobile operating systems for smartphones and tablets — touchscreen-first, app-centric, battery-aware.
- **Cloud OS / Hypervisors:** Operating systems designed to manage virtual machines in data centers (VMware ESXi, Microsoft Hyper-V).

---

## 3. Functions of an Operating System

The OS performs six core functions. We will explore each in depth in the sections that follow.

| Function | What it Does |
|---|---|
| **Process Management** | Creates, schedules, and terminates processes; manages multitasking |
| **Memory Management** | Allocates and deallocates RAM; manages virtual memory |
| **File System Management** | Organizes, stores, retrieves, and protects files on storage |
| **Device Management** | Controls I/O devices via drivers; handles interrupts |
| **User Interface** | Provides CLI or GUI for user interaction |
| **Security & Access Control** | Authenticates users; enforces permissions; protects resources |

At the heart of all these functions is the **kernel** — the core component of the OS that has direct, privileged access to hardware. Everything else in the OS runs on top of it.

### The Kernel

The **kernel** is the always-running core of the OS. It operates in **kernel mode** — a privileged CPU mode with unrestricted hardware access. User applications run in **user mode**, with restricted access, and must request services from the kernel via **system calls**.

```
User Mode:     [ App A ] [ App B ] [ App C ]
                   │          │         │
              System Calls (API boundary)
                   │
Kernel Mode:  [ Kernel: Process Mgr | Memory Mgr | File System | Drivers ]
                   │
              [ Hardware: CPU | RAM | Disk | Network ]
```

This separation protects the OS from buggy or malicious applications.

---

## 4. Process Management

### 4.1 What is a Process?

A **program** is a static set of instructions stored on disk (e.g., `chrome.exe`). A **process** is a program that has been loaded into memory and is actively running. One program can spawn multiple processes.

Each process has:
- A unique **Process ID (PID)**
- Its own allocated **memory space**
- A **state** (running, waiting, etc.)
- A **Program Counter** (which instruction it is executing)
- CPU registers, open file handles, and other resources

> **Example:** When you open three tabs in Chrome, the browser may create three separate processes — one per tab — so that a crash in one tab doesn't kill the others.

### 4.2 Process States

A process moves through several states during its lifetime:

```
         ┌─────────────────────────────────────────────┐
         │                                             │
       [New] ──→ [Ready] ──→ [Running] ──→ [Terminated]
                    ↑              │
                    │         (I/O wait)
                    │              ↓
                    └────── [Waiting/Blocked]
```

| State | Description |
|---|---|
| **New** | Process is being created |
| **Ready** | Loaded in memory, waiting for CPU time |
| **Running** | Currently executing on the CPU |
| **Waiting/Blocked** | Waiting for an I/O event (disk read, keyboard input) |
| **Terminated** | Process has finished or been killed |

### 4.3 CPU Scheduling

Since a CPU core can only run **one process at a time**, the OS must decide which ready process gets CPU time next. This is done by the **scheduler**, using a **scheduling algorithm**.

#### Common Scheduling Algorithms

**1. First-Come, First-Served (FCFS)**
Processes are executed in the order they arrive. Simple but can cause the "convoy effect" where short jobs wait behind a long one.

```
Jobs: A(10ms) → B(2ms) → C(5ms)
Order: A → B → C
Total wait time: A=0, B=10ms, C=12ms
```

**2. Shortest Job First (SJF)**
The process with the shortest estimated run time is executed next. Minimizes average wait time but requires knowing job length in advance.

```
Jobs: A(10ms), B(2ms), C(5ms)
Order: B → C → A
Total wait time: B=0, C=2ms, A=7ms  (much better!)
```

**3. Round Robin (RR)**
Each process gets a fixed **time quantum** (e.g., 10ms) in rotation. When the quantum expires, the process is preempted and the next one runs. This is the most common algorithm for interactive systems.

```
Quantum = 3ms, Jobs: A(10ms), B(5ms), C(4ms)
Timeline: A(3) → B(3) → C(3) → A(3) → B(2) → C(1) → A(3) → A(1)
```

**4. Priority Scheduling**
Each process is assigned a priority. The highest-priority process runs first. Lower-priority processes may suffer **starvation** (never get CPU time) if high-priority processes keep arriving.

#### Context Switching

When the OS switches from running one process to another, it must save the current process's state (registers, program counter) into its **Process Control Block (PCB)** and load the state of the next process. This is called a **context switch**. It has a small overhead cost but enables multitasking.

### 4.4 Threads

A **thread** is the smallest unit of execution within a process. A process can have multiple threads sharing the same memory space — this is called **multithreading**.

| | Process | Thread |
|---|---|---|
| Memory | Own isolated memory space | Shares memory with other threads in same process |
| Creation cost | Heavy (slow) | Lightweight (fast) |
| Communication | Via OS (slow) | Directly via shared memory (fast) |
| Crash impact | Isolated | One bad thread can crash whole process |

> **Example:** A web server handles each incoming request on a separate thread. All threads share the same server code and database connection pool (shared memory), but process requests concurrently.

---

## 5. Memory Management

### 5.1 Why Memory Management is Needed

Multiple processes run simultaneously, all needing RAM. The OS must:
- Allocate memory to processes when they start.
- Ensure one process cannot access another's memory (protection).
- Reclaim memory when processes finish.
- Handle the situation where total demand exceeds physical RAM.

### 5.2 Memory Allocation

When a process starts, the OS allocates it a **contiguous block** of memory (in simple systems) or scattered pages (in modern systems).

**Memory layout of a typical process:**

```
High address ┌─────────────────┐
             │      Stack      │  ← Local variables, function calls (grows down)
             │        ↓        │
             │   (free space)  │
             │        ↑        │
             │      Heap       │  ← Dynamically allocated memory (grows up)
             ├─────────────────┤
             │   Data Segment  │  ← Global and static variables
             ├─────────────────┤
Low address  │  Code Segment   │  ← Program instructions
             └─────────────────┘
```

### 5.3 Virtual Memory

Physical RAM is limited. **Virtual memory** is a technique that gives each process the illusion of having a large, private memory space — larger than actual RAM — by using disk space as an extension of RAM.

The OS divides memory into fixed-size chunks called **pages** (typically 4KB). When a page is needed but not in RAM, the OS loads it from disk (a **page fault**). Less-used pages are temporarily swapped to a special area on disk called the **swap space** (Windows) or **swap partition** (Linux).

```
Process sees:  [ Virtual Address Space — huge, private ]
                           ↓  (page table translation)
Physical RAM:  [ Page 1 | Page 3 | Page 7 | ... ]
                           ↓  (if not in RAM)
Disk Swap:     [ Page 2 | Page 4 | Page 5 | ... ]
```

**Benefits of virtual memory:**
- Programs can be larger than physical RAM.
- Processes are isolated — they cannot accidentally (or maliciously) read each other's memory.
- Memory can be shared between processes (e.g., shared libraries loaded once into RAM, used by many processes).

**Downside:** Excessive swapping (called **thrashing**) causes severe slowdowns, as disk access is much slower than RAM.

### 5.4 Memory Protection

The OS uses the CPU's **Memory Management Unit (MMU)** hardware to enforce that each process can only access its own memory. Any unauthorized access causes a **segmentation fault** (Linux/macOS) or **access violation** (Windows) and the offending process is terminated.

---

## 6. File System Management

### 6.1 What is a File System?

A **file system** is the method the OS uses to organize, store, retrieve, name, and protect data on storage devices. Without a file system, a disk is just a flat sequence of bits with no structure.

The file system provides:
- **Files:** Named collections of data.
- **Directories (Folders):** Containers that organize files hierarchically.
- **Metadata:** Information about each file (name, size, type, permissions, timestamps).

### 6.2 File System Hierarchy

Most modern OSes use a **hierarchical (tree) directory structure**:

**Windows:**
```
C:\
├── Program Files\
│   ├── Google\
│   │   └── Chrome\
│   └── Microsoft Office\
├── Users\
│   └── Ana\
│       ├── Documents\
│       ├── Downloads\
│       └── Desktop\
└── Windows\
    └── System32\
```

**Linux / macOS:**
```
/
├── home/
│   └── ana/
│       ├── Documents/
│       └── Downloads/
├── usr/
│   └── bin/         ← Executable programs
├── etc/             ← Configuration files
├── var/             ← Log files, variable data
└── dev/             ← Device files
```

> **Key difference:** Windows uses drive letters (`C:\`, `D:\`) as separate trees. Linux/macOS uses a single root `/` — all drives are **mounted** as directories within this single tree.

### 6.3 File Attributes and Metadata

Each file has associated **metadata** stored by the file system:

| Attribute | Description |
|---|---|
| **Name** | Human-readable identifier |
| **Type/Extension** | `.txt`, `.mp4`, `.exe`, `.pdf` |
| **Size** | In bytes |
| **Location** | Physical address(es) on disk |
| **Permissions** | Who can read/write/execute |
| **Timestamps** | Created, modified, last accessed |
| **Owner** | Which user/group owns the file |

### 6.4 File Permissions

Operating systems use **permissions** to control who can do what with a file.

**Linux/macOS (Unix-style permissions):**

Permissions are set for three categories: **Owner**, **Group**, and **Others**.
Each category has three bits: **Read (r)**, **Write (w)**, **Execute (x)**.

```
-rwxr-xr--
 │││││││└── Others: read only
 │││││└──── Others: no write
 ││││└───── Others: no execute
 │││└────── Group: no execute... wait, let's read properly:

-  rwx  r-x  r--
│   │    │    └── Others: read only
│   │    └─────── Group: read + execute
│   └──────────── Owner: read + write + execute
└──────────────── File type (- = regular file)
```

Using **octal notation** (from Module 2!):
- `rwx` = 111₂ = 7
- `r-x` = 101₂ = 5
- `r--` = 100₂ = 4
- `chmod 754 file.txt` sets owner=7(rwx), group=5(r-x), others=4(r--)

### 6.5 Common File Systems

| File System | OS | Key Features |
|---|---|---|
| **FAT32** | Windows/cross-platform | Simple; max file size 4GB; used in USB drives |
| **NTFS** | Windows | Journaling, permissions, large files, encryption |
| **ext4** | Linux | Journaling, large files, widely used |
| **APFS** | macOS/iOS | Optimized for SSDs, snapshots, encryption |
| **exFAT** | Cross-platform | For flash drives; no 4GB file limit |

### 6.6 Journaling

A **journaled file system** (NTFS, ext4, APFS) keeps a **journal (log)** of changes before committing them. If the computer crashes mid-write, the journal allows the OS to recover and maintain file system consistency — preventing corruption.

> **Analogy:** Like a chef who writes down the recipe steps before cooking — if interrupted, they can refer back to the log to know exactly where they left off.

---

## 7. Device Management

### 7.1 Device Drivers

Hardware devices (keyboard, mouse, printer, GPU, network card) all communicate differently. The OS uses **device drivers** — specialized software that translates between the OS's generic I/O commands and a specific hardware device's communication protocol.

```
Application: "Print this document"
     ↓
OS I/O System: "Write this data to printer device"
     ↓
Device Driver (HP LaserJet Driver): [device-specific commands]
     ↓
Hardware: Printer physically prints
```

> **Why this matters:** When you install a new printer, you install its **driver**. Without the driver, the OS doesn't know how to talk to that specific hardware model.

### 7.2 Interrupts

Instead of the CPU constantly checking whether a device needs attention (which wastes CPU cycles — called **polling**), modern OSes use **interrupts**:

1. The CPU is executing a process normally.
2. A hardware event occurs (key pressed, disk read complete, network packet arrived).
3. The device sends an **interrupt signal** to the CPU.
4. The CPU pauses its current work, saves its state, and runs an **Interrupt Service Routine (ISR)** — a small piece of OS code that handles the event.
5. The CPU resumes its previous work.

> **Analogy:** Instead of you checking your phone every second for messages (polling), your phone vibrates when a message arrives (interrupt). You briefly check the message and return to what you were doing.

### 7.3 I/O Buffering

A **buffer** is a temporary memory area used to hold data during I/O transfers. It bridges the speed gap between fast CPUs and slow devices.

**Example:** When streaming a video, the player pre-loads (buffers) several seconds of video into RAM. Even if the network briefly slows, playback continues smoothly from the buffer.

---

## 8. User Interface

The OS provides two primary ways for users to interact with the system:

### 8.1 Command-Line Interface (CLI)

The user types text commands, and the OS executes them and returns text output.

**Examples:**
- **Windows:** Command Prompt (`cmd`), PowerShell
- **Linux/macOS:** Bash, Zsh, Fish (Terminal)

```bash
# Common Linux/macOS CLI commands
ls -la              # List files with details
cd Documents        # Change directory
mkdir project       # Create a new folder
cp file.txt backup/ # Copy a file
rm old_file.txt     # Delete a file
chmod 755 script.sh # Change file permissions
ps aux              # List running processes
top                 # Live process monitor
```

**Advantages of CLI:**
- Fast and efficient for experienced users.
- Scriptable — commands can be combined and automated in **shell scripts**.
- Available even without a graphical environment (e.g., remote server access via SSH).
- Uses minimal system resources.

**Disadvantages:**
- Steep learning curve for beginners.
- Easy to make destructive mistakes (e.g., `rm -rf /` on Linux deletes everything).

### 8.2 Graphical User Interface (GUI)

The user interacts through visual elements: windows, icons, menus, buttons, and a pointer (mouse/touchpad/touchscreen).

**Components of a typical GUI OS:**
- **Desktop:** The visual workspace.
- **Window Manager:** Controls size, position, and layering of windows.
- **Taskbar/Dock:** Quick access to running applications and system functions.
- **File Explorer/Finder:** Graphical tool to browse files and folders.
- **Control Panel/Settings:** Interface to configure system preferences.

**Advantages of GUI:**
- Intuitive and easy to learn.
- Visual feedback (progress bars, icons, previews).
- Easier for file management and creative tasks.

**Disadvantages:**
- Consumes more system resources (RAM, GPU).
- Can be slower for repetitive tasks compared to scripting.

### 8.3 CLI vs GUI Comparison

| Feature | CLI | GUI |
|---|---|---|
| Ease of use | Hard for beginners | Easy for beginners |
| Speed (expert user) | Faster | Slower |
| Automation | Easily scriptable | Harder to automate |
| Resource use | Minimal | Higher |
| Remote access | Ideal (SSH) | Possible but slower |
| Error risk | Higher (typos = damage) | Lower (visual confirmation) |

---

## 9. Types of Operating Systems

### 9.1 Batch Operating System

Executes jobs in batches without user interaction. Jobs are queued and run one after another.
- **Use case:** Payroll processing, bank statement generation, scientific simulations.
- **Example:** IBM mainframe batch systems.

### 9.2 Time-Sharing (Multitasking) Operating System

Multiple users or processes share CPU time via rapid switching, creating the illusion of simultaneous execution.
- **Use case:** General-purpose desktop and server computing.
- **Example:** Unix, Linux, Windows, macOS.

### 9.3 Real-Time Operating System (RTOS)

Guarantees that critical tasks are completed within a **strict time deadline**. Missing a deadline can have catastrophic consequences.

Two types:
- **Hard RTOS:** Missing deadline = system failure (e.g., airbag deployment system — must fire within milliseconds).
- **Soft RTOS:** Missing deadline degrades performance but isn't catastrophic (e.g., video streaming).

**Examples:** VxWorks (spacecraft, medical devices), FreeRTOS (microcontrollers), QNX (automotive systems).

### 9.4 Distributed Operating System

Manages a group of independent computers networked together, making them appear as a single system to the user.
- **Use case:** Cloud computing, large-scale web services.
- **Example:** Google's internal systems, Apache Hadoop.

### 9.5 Embedded Operating System

A lightweight OS designed to run on resource-constrained embedded hardware with a specific dedicated function.
- **Use case:** Smart TVs, ATMs, washing machines, medical devices, routers.
- **Examples:** FreeRTOS, Embedded Linux, Windows IoT.

### 9.6 Mobile Operating System

Designed for battery-powered touchscreen devices. Optimized for power efficiency, app ecosystems, and wireless connectivity.
- **Examples:** Android (Linux-based, open-source), iOS (Unix-based, Apple-exclusive).

### 9.7 Network Operating System (NOS)

Designed to manage network resources — file sharing, printer sharing, user authentication across a network.
- **Examples:** Windows Server, Novell NetWare, Linux with Samba.

---

## 10. Popular Operating Systems

### 10.1 Microsoft Windows

- **Market share:** Dominant on desktops/laptops (~70%).
- **Kernel:** Windows NT kernel (hybrid kernel).
- **File system:** NTFS.
- **Key features:** Wide software compatibility, Active Directory for enterprises, Xbox ecosystem, WSL (Windows Subsystem for Linux).
- **Latest:** Windows 11.

### 10.2 macOS

- **Developer:** Apple Inc.
- **Kernel:** XNU (hybrid — based on Mach microkernel + BSD Unix).
- **File system:** APFS.
- **Key features:** Tight hardware-software integration (Apple Silicon), Unix terminal, iCloud ecosystem, Final Cut Pro, developer-friendly.
- **Used on:** MacBook, iMac, Mac Mini, Mac Pro.

### 10.3 Linux

- **Developer:** Community-driven (started by Linus Torvalds in 1991).
- **Kernel:** Monolithic Linux kernel.
- **File system:** ext4, Btrfs, XFS.
- **Key features:** Open-source (free), highly customizable, extremely stable, powers ~96% of the world's top 500 supercomputers, most web servers, and Android.
- **Distributions (Distros):** Ubuntu, Fedora, Debian, Arch Linux, CentOS/RHEL (enterprise).

### 10.4 Android

- **Developer:** Google (Linux-based).
- **Market share:** ~72% of global mobile OS.
- **Key features:** Open-source, Google Play Store, vast hardware variety, customizable.

### 10.5 iOS

- **Developer:** Apple Inc. (Unix-based).
- **Key features:** Closed ecosystem (Apple hardware only), highly secure, smooth performance, App Store, tight integration with macOS/iPad OS.

### 10.6 Comparison Table

| Feature | Windows | macOS | Linux | Android | iOS |
|---|---|---|---|---|---|
| Cost | Paid | Paid (bundled) | Free | Free | Bundled |
| Open Source | No | Partially | Yes | Partially | No |
| Primary Use | Desktop | Desktop | Server/Desktop | Mobile | Mobile |
| Customization | Moderate | Limited | Extensive | Moderate | Very Limited |
| Security | Moderate | High | High | Moderate | Very High |
| Market Share | ~70% desktop | ~15% desktop | ~96% servers | ~72% mobile | ~27% mobile |

---

## 11. OS Security

The OS is the first line of defense in computer security. It implements multiple mechanisms to protect the system and its data.

### 11.1 User Authentication

Before granting access to the system, the OS verifies the user's identity via:
- **Passwords / PINs**
- **Biometrics** (fingerprint, Face ID)
- **Multi-Factor Authentication (MFA):** Requires two or more verification factors (password + phone OTP).

### 11.2 Access Control

The OS enforces **who can do what** with system resources:

- **Discretionary Access Control (DAC):** The owner of a resource sets permissions (Unix file permissions, Windows ACLs).
- **Mandatory Access Control (MAC):** The OS enforces security policies that even the owner cannot override (used in military systems, SELinux).
- **Role-Based Access Control (RBAC):** Permissions are granted based on a user's role (admin, standard user, guest).

### 11.3 The Principle of Least Privilege

A cornerstone security principle: every user, process, and program should have the **minimum level of access** necessary to perform its function — nothing more.

> **Example:** A media player application needs to read audio files but has no need to access your banking documents or modify system settings. Under least privilege, it is only granted read access to the media folder.

### 11.4 User Account Control (UAC) and sudo

- **Windows UAC:** Prompts for administrator confirmation before allowing system-level changes.
- **Linux sudo:** Allows a trusted user to execute specific commands with elevated (root) privileges temporarily.

Both prevent malicious programs from silently making system-wide changes.

### 11.5 Memory Protection and Process Isolation

As discussed in Section 5, the OS uses the MMU to ensure processes cannot access each other's memory. This limits the damage a compromised process can do.

### 11.6 Security Vulnerabilities in OS

Common OS-level vulnerabilities:
- **Privilege escalation:** A non-admin process exploits a bug to gain admin rights.
- **Buffer overflow:** Writing more data than a buffer can hold overwrites adjacent memory, potentially hijacking execution.
- **Kernel exploits:** Bugs in the kernel itself — the most dangerous class of vulnerability.

> **Real-world example:** The **EternalBlue** exploit (2017) targeted a vulnerability in the Windows SMB protocol (part of the OS networking stack). It was used in the WannaCry ransomware attack that infected over 200,000 computers across 150 countries.

---

## 12. Module Summary

Key takeaways from Module 4:

- An **Operating System** is software that acts as an intermediary between users/applications and hardware, managing all computer resources.
- The **kernel** is the privileged core of the OS with direct hardware access; applications communicate with it via **system calls**.
- OS evolution went from **no OS** (1940s) → **batch processing** → **time-sharing** → **GUI** → **mobile/cloud** OS.
- **Process management** involves creating, scheduling, and terminating processes. The scheduler uses algorithms (FCFS, SJF, Round Robin, Priority) to allocate CPU time. A **context switch** saves and restores process state.
- **Memory management** uses allocation, protection, and **virtual memory** (paging + swap) to give each process an isolated, expandable memory space.
- **File system management** organizes data into files and directories with metadata and permissions. Common file systems include NTFS, ext4, and APFS.
- **Device management** uses **drivers** to communicate with hardware and **interrupts** to handle device events efficiently.
- The OS provides either a **CLI** (text commands, scriptable, efficient) or a **GUI** (visual, intuitive) — or both.
- Types of OS include **batch, time-sharing, real-time, distributed, embedded, mobile**, and **network** operating systems.
- OS **security** mechanisms include authentication, access control, least privilege, memory protection, and UAC/sudo.

---

## 13. Module Quiz

Test your understanding of Module 4.

---

**Question 1**
What is the primary role of an Operating System?

- A) To provide a programming language for developers
- B) To manage hardware resources and provide services to applications
- C) To store user files and documents permanently
- D) To connect the computer to the internet

---

**Question 2**
A process that is waiting for a disk read operation to complete is in which state?

- A) Running
- B) New
- C) Ready
- D) Waiting/Blocked

---

**Question 3**
Which CPU scheduling algorithm gives each process an equal, fixed time slot in rotation, making it most suitable for interactive systems?

- A) First-Come, First-Served (FCFS)
- B) Shortest Job First (SJF)
- C) Round Robin
- D) Priority Scheduling

---

**Question 4**
What problem does **virtual memory** primarily solve?

- A) The CPU being too slow to handle multiple tasks
- B) Running programs that require more memory than physically available RAM
- C) Files becoming corrupted when the computer crashes
- D) Preventing unauthorized users from logging in

---

**Question 5**
On a Linux system, a file has the permissions `-rwxr-x---`. Which statement is CORRECT?

- A) All users can read, write, and execute the file
- B) The owner can read, write, and execute; the group can read and execute; others have no access
- C) The owner can only read and execute; the group can write
- D) The file is a directory with full permissions for everyone

---

**Question 6**
What is the purpose of a **device driver**?

- A) To speed up the CPU when running heavy applications
- B) To provide extra RAM when the system is low on memory
- C) To translate generic OS I/O commands into device-specific communication
- D) To encrypt files stored on disk

---

**Question 7**
An aircraft autopilot control system must respond to sensor data within 5 milliseconds — failure to do so could be catastrophic. What type of OS is most appropriate?

- A) Batch Operating System
- B) Distributed Operating System
- C) Mobile Operating System
- D) Hard Real-Time Operating System (Hard RTOS)

---

**Question 8**
Which of the following best describes the **Principle of Least Privilege**?

- A) Admin users should always use the most powerful account for efficiency
- B) Every user and process should have only the minimum access necessary to do its job
- C) New users should be given no privileges until they request them manually
- D) Privileged processes should never be terminated by the OS

---

**Question 9**
What is a **context switch** in operating systems?

- A) Switching a computer from CLI to GUI mode
- B) Transferring data from RAM to the hard disk
- C) The process of saving the state of one process and loading the state of another to enable multitasking
- D) Changing file permissions from one user to another

---

**Question 10** *(Short Answer)*
Explain the difference between a **process** and a **thread**. Describe a real-world scenario where using **multiple threads** within a single process would be more beneficial than using multiple separate processes.

---

### Answer Key

| Question | Answer | Explanation |
|---|---|---|
| 1 | **B** | The OS manages all hardware resources (CPU, memory, I/O) and provides a stable platform and services for applications to run on. |
| 2 | **D** | A process waiting for an I/O event (like a disk read) is in the **Waiting/Blocked** state — it cannot use the CPU until the event completes. |
| 3 | **C** | **Round Robin** allocates a fixed time quantum to each process in turn. This fairness and responsiveness makes it ideal for interactive systems. |
| 4 | **B** | Virtual memory allows programs to use more memory than physically available RAM by using disk space as an extension, managed through paging. |
| 5 | **B** | `-rwxr-x---`: owner=`rwx`(7), group=`r-x`(5), others=`---`(0). Owner can read/write/execute; group can read/execute; others have no access. |
| 6 | **C** | A device driver is a translator — it converts the OS's standard I/O requests into the specific commands that a particular hardware device understands. |
| 7 | **D** | A **Hard RTOS** guarantees task completion within strict deadlines. In safety-critical systems like aircraft, missing a deadline is catastrophic — only Hard RTOS is acceptable. |
| 8 | **B** | Least Privilege means granting the minimum necessary access. This limits the damage any compromised or buggy component can cause. |
| 9 | **C** | A context switch saves the current process's state (registers, PC) into its PCB and loads another process's saved state — the mechanism behind multitasking. |
| 10 | *(See below)* | |

**Question 10 – Sample Answer:**
> A **process** is an independent, running instance of a program with its own private memory space, resources, and Process ID. Processes are isolated from each other — one process cannot directly access another's memory.
>
> A **thread** is a unit of execution *within* a process. Multiple threads share the same process memory, code, and resources, but each has its own stack and program counter. Threads are lighter to create and can communicate faster through shared memory.
>
> **Real-world scenario — A web browser:**
> A modern browser (like Chrome) uses multiple threads within a single renderer process for a webpage. Consider loading a complex website:
> - **Thread 1** handles parsing and rendering the HTML/CSS.
> - **Thread 2** executes JavaScript code.
> - **Thread 3** decodes and plays embedded video.
> - **Thread 4** handles network requests for images.
>
> Using threads here is better than separate processes because all threads need constant access to the same page's DOM (Document Object Model) in shared memory. With separate processes, sharing this data would require expensive inter-process communication (IPC). Threads can share it directly and instantly, making the page load and run much faster. The threads also all benefit from being part of the same process lifecycle — when you close the tab, all threads are cleaned up together.

---

*End of Module 4 — Up next: **Module 5: Overview of Database Management Systems***
