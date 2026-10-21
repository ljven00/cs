# CS 2301 — Operating Systems 1
## Module 6: Input / Output Management

---

> **Course Progress:** Module 6 of 8 — CS 2301 (OS 1)

---

## 1. What is I/O Management?

Every time your program reads a file, prints to the screen, receives a keystroke, or sends data over a network — that is **I/O (Input/Output)**. The OS is responsible for managing all communication between the CPU/memory and external devices.

Without proper I/O management:
- Programs would have to talk directly to hardware (complex and unsafe)
- One slow device would stall the entire CPU
- Different devices would need completely different code in every program

The OS solves all three problems.

---

## 2. I/O Hardware Overview

I/O devices connect to the CPU via a **bus** — a communication pathway. Each device has a **controller** — a small chip that operates the device and exposes registers the OS can read/write.

```
+-------+       +--------+       +-----------+
|  CPU  | ←───→ |  Bus   | ←───→ | Device    |
+-------+       +--------+       | Controller|
                                 +-----------+
                                       │
                                 +-----------+
                                 |   Device  |
                                 | (disk,    |
                                 |  keyboard)|
                                 +-----------+
```

### Device Categories

| Category | Examples |
|----------|---------|
| **Block devices** | Hard drives, SSDs, USB drives — transfer data in blocks |
| **Character devices** | Keyboards, mice, serial ports — transfer data one character at a time |
| **Network devices** | Ethernet, Wi-Fi — send/receive packets |

---

## 3. I/O Techniques

How does the CPU interact with a device? There are three main techniques, each with different tradeoffs.

### 3.1 Programmed I/O (Polling)

The CPU directly controls the device and keeps checking (polling) whether the operation is done.

```
CPU: "Are you done yet?"  → Device: "No"
CPU: "Are you done yet?"  → Device: "No"
CPU: "Are you done yet?"  → Device: "Yes! Here's your data."
```

- Simple to implement
- Wastes CPU cycles — the CPU is stuck in a loop doing nothing useful
- Fine for very fast devices; terrible for slow ones (e.g., waiting for a disk)

---

### 3.2 Interrupt-Driven I/O

The CPU starts the I/O operation, then goes back to doing other work. When the device finishes, it sends an **interrupt** signal to the CPU.

```
CPU: "Start reading, notify me when done." → Device: starts working
CPU: (runs other processes)
Device: "Done!" → sends interrupt
CPU: pauses current work, runs interrupt handler, resumes
```

- CPU is free while the device works — much more efficient
- Requires an **interrupt handler** (Interrupt Service Routine — ISR)
- Good for most I/O, but too many interrupts can still overwhelm the CPU

---

### 3.3 Direct Memory Access (DMA)

For large data transfers (e.g., reading a big file), even interrupt-driven I/O has overhead because the CPU must copy data byte-by-byte from the device to memory. **DMA** offloads this.

A dedicated **DMA controller** handles the data transfer directly between the device and RAM, without involving the CPU for every byte. The CPU is only interrupted once — when the entire transfer is complete.

```
CPU: "DMA controller: move 4KB from disk to address 0x5000. Tell me when done."
DMA controller: transfers all 4KB directly to RAM
DMA controller: interrupts CPU once
CPU: (was free the whole time the transfer happened)
```

- Best for large, bulk data transfers
- Used by almost all modern systems (disk, network, GPU transfers)

### I/O Technique Comparison

| Technique | CPU Involvement | Best For |
|-----------|----------------|----------|
| Polling | High (busy-waiting) | Simple, fast devices |
| Interrupt-driven | Low (handles interrupts) | Most general I/O |
| DMA | Minimal (notified once) | Large bulk transfers |

---

## 4. Device Drivers

A **device driver** is software that translates generic OS I/O requests into device-specific commands.

```
Application: read("file.txt")
     ↓
OS File System
     ↓
OS I/O Subsystem
     ↓
Device Driver  ← talks the device's language
     ↓
Hardware Controller
     ↓
Physical Device (disk, keyboard, etc.)
```

- Drivers are usually provided by the hardware manufacturer
- They run in kernel mode (trusted, privileged)
- A bad driver can crash the entire OS — which is why driver bugs are a major cause of system crashes

---

## 5. I/O Software Layers

The OS organizes I/O into clean layers, each hiding complexity from the layer above:

```
+-------------------------------+
|     User-Level I/O Library    |  printf(), fread(), fwrite()
+-------------------------------+
|     Kernel I/O Subsystem      |  Buffering, caching, scheduling
+-------------------------------+
|       Device Drivers          |  Device-specific code
+-------------------------------+
|    Interrupt Handlers         |  Respond to hardware signals
+-------------------------------+
|        Hardware               |  Actual devices
+-------------------------------+
```

---

## 6. Buffering

A **buffer** is a temporary memory area that holds data in transit between a device and a program.

Why buffer?
- Devices and programs often work at different speeds
- A keyboard produces characters one at a time; a program reads lines at a time
- A network receives packets in bursts; an application reads a stream

### Types of Buffering

**Single buffering:** One buffer. While the program processes buffer A, the device fills buffer A. They must take turns — one waits for the other.

**Double buffering:** Two buffers. While the program processes buffer A, the device fills buffer B. Then they swap. Overlap is possible — much more efficient.

```
Single:  [Device fills A] → [Program reads A] → [Device fills A] → ...
Double:  [Device fills B] while [Program reads A] → swap → repeat
```

---

## 7. Caching

A **cache** stores recently or frequently used data in faster storage so future requests are served quickly without going back to the slow device.

```
Program requests file block 42
     ↓
Check cache → Hit? → Return immediately (fast!)
           → Miss? → Read from disk, store in cache, return
```

- The OS maintains a **page cache** (also called buffer cache) in RAM for disk data
- Reading the same file twice — second read is nearly instant (from cache)
- Cache must be kept consistent with disk (**write-back** or **write-through** policies)

---

## 8. Spooling

**Spooling** (Simultaneous Peripheral Operations On-Line) is used for devices that can only serve one job at a time — like a printer.

Instead of making programs wait, the OS intercepts print requests and stores them in a **spool queue** on disk. A background process feeds jobs to the printer one at a time.

```
Program A → print job → |              |
Program B → print job → | Spool Queue  | → Printer (one at a time)
Program C → print job → |              |
```

- Programs return immediately after submitting to the queue
- The printer processes jobs in order without programs interfering with each other

---

## 9. I/O Scheduling

Just like CPU scheduling decides which process runs next, **I/O scheduling** decides which I/O request is served next.

For disks, this is the disk scheduling from Module 5 (SCAN, SSTF, etc.). For other devices, the OS typically uses priority queues — urgent requests (e.g., from interactive programs) jump ahead of background tasks.

---

## 10. Error Handling

Devices fail. The OS I/O subsystem must handle errors gracefully:

| Error Type | Example | OS Response |
|------------|---------|-------------|
| Transient error | Network packet lost | Retry automatically |
| Permanent error | Disk sector unreadable | Report to application |
| Device not found | USB unplugged | Notify, clean up |
| Timeout | Device stopped responding | Abort and report |

The goal is that a device failure should never silently corrupt data or crash the whole system — the OS contains the damage.

---

## 11. Unix I/O Philosophy: Everything is a File

Unix and Linux treat almost every I/O resource as a **file**. Disks, keyboards, network sockets, pipes, and even hardware sensors are accessed through the same `open()`, `read()`, `write()`, `close()` interface.

```c
// Reading from a file
int fd = open("data.txt", O_RDONLY);
read(fd, buffer, 100);
close(fd);

// Reading from a keyboard (stdin is also a file descriptor)
read(0, buffer, 100);   // 0 = stdin

// Writing to a network socket (also a file descriptor)
write(sockfd, "Hello", 5);
```

This uniformity is powerful — programs don't need to know what kind of device they're talking to.

---

## 12. Real-World Example: Typing and Submitting a Form

When you type into a web form and hit Submit, here's what happens at the I/O level:

1. **Keyboard** generates an interrupt for each keypress
2. The **interrupt handler** reads the character from the controller
3. Characters are placed in a **buffer** until you press Enter
4. The browser reads the buffered input
5. Browser sends data over the **network** (another I/O device)
6. The OS uses **DMA** to move network packet data to/from RAM efficiently
7. The server receives it, reads from its **socket** (a file descriptor in Unix)
8. Server writes a response — OS **caches** it, sends it back via DMA

All of this happens in milliseconds, coordinated entirely by the OS I/O subsystem.

---

## 13. Summary

| Concept | Key Takeaway |
|---------|-------------|
| I/O devices | Block, character, and network — each behaves differently |
| Polling | CPU busy-waits — simple but wasteful |
| Interrupt-driven | CPU freed while device works; interrupted on completion |
| DMA | Device-to-memory transfer without CPU; one interrupt total |
| Device driver | Translates OS requests into device-specific commands |
| Buffering | Smooths speed mismatches between devices and programs |
| Caching | Stores recent data in fast memory to avoid repeated slow reads |
| Spooling | Queues jobs for exclusive devices (like printers) |
| Unix file model | Everything (disk, keyboard, socket) accessed via same interface |

---

## Module 6 Quiz

---

**Question 1**
What is the main disadvantage of Programmed I/O (Polling)?

> **Answer:** The CPU wastes cycles in a busy-wait loop continuously checking whether the device has finished. During this time the CPU cannot do other useful work. This is highly inefficient for slow devices like disks or networks.

---

**Question 2**
Explain the difference between interrupt-driven I/O and DMA. When would you prefer DMA?

> **Answer:** With interrupt-driven I/O, the CPU is interrupted for every unit of data transferred (e.g., each byte or word), which adds up for large transfers. With DMA, a dedicated DMA controller moves the entire block of data directly between the device and RAM; the CPU is only interrupted once when the whole transfer is complete. DMA is preferred for large bulk transfers like reading big files from disk or receiving large network packets.

---

**Question 3**
What is the purpose of a device driver, and why does it run in kernel mode?

> **Answer:** A device driver translates generic OS I/O requests (like "read block 42") into the specific commands a hardware device understands. It runs in kernel mode because it needs direct, privileged access to hardware registers and memory. Running it in user mode would be too restrictive and too slow.

---

**Question 4**
A print server receives simultaneous print jobs from 10 users. What I/O technique ensures all users can submit jobs without waiting for the printer to finish?

> **Answer:** **Spooling.** Each print job is stored in a spool queue on disk. All 10 users submit their jobs immediately and continue working. A background spooler process feeds jobs to the printer one at a time in order.

---

**Question 5**
In Unix, the following code opens a file and reads from it. What small change would let the exact same code read from standard input (the keyboard) instead?

```c
int fd = open("data.txt", O_RDONLY);
read(fd, buffer, 100);
close(fd);
```

> **Answer:** Replace the `open()` call with file descriptor `0` (stdin), and remove the `close()`:
>
> ```c
> int fd = 0;           // 0 = stdin (keyboard)
> read(fd, buffer, 100);
> ```
>
> This works because Unix treats everything as a file — stdin, files, and devices all share the same `read()` interface. File descriptor 0 is always stdin.

---

## What's Next

**Module 7: Security and Protection**

You'll learn how the OS keeps users, processes, and data safe — covering authentication, access control, and common attack vectors.

---

*CS 2301 Operating Systems 1 — Module 6 of 8*
