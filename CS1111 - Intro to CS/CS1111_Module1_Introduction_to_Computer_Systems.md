# CS 1111 – Introduction to Computer Science
## Module 1: Introduction to Computer Systems

---

## Table of Contents
1. [What is a Computer System?](#1-what-is-a-computer-system)
2. [History and Evolution of Computers](#2-history-and-evolution-of-computers)
3. [Components of a Computer System](#3-components-of-a-computer-system)
4. [How a Computer Processes Data](#4-how-a-computer-processes-data)
5. [Types of Computers](#5-types-of-computers)
6. [Software: The Invisible Engine](#6-software-the-invisible-engine)
7. [Module Summary](#7-module-summary)
8. [Module Quiz](#8-module-quiz)

---

## 1. What is a Computer System?

A **computer system** is an integrated set of hardware and software components that work together to accept data (input), process it, store it, and produce meaningful results (output).

Think of a computer system like a kitchen:
- The **chef** (CPU) does the actual work.
- The **counter space** (RAM) is where active ingredients are placed while cooking.
- The **refrigerator and pantry** (storage) hold ingredients not currently in use.
- The **recipe book** (software) tells the chef what to do.
- The **raw ingredients** (input) come in, and the **finished dish** (output) goes out.

A computer system is not just the physical machine — it includes all the programs, data, people, and procedures that make it function.

### Key Characteristics of a Computer System

| Characteristic | Description |
|---|---|
| **Speed** | Performs millions of operations per second |
| **Accuracy** | Produces precise results when given correct instructions |
| **Automation** | Can carry out tasks without human intervention once programmed |
| **Storage** | Can store vast amounts of data for long periods |
| **Versatility** | Can perform many different types of tasks |

---

## 2. History and Evolution of Computers

Computers have evolved dramatically over the decades. Their evolution is commonly categorized into **five generations**.

### Generation 1 (1940s – 1950s): Vacuum Tubes
- Used **vacuum tubes** for circuitry and **magnetic drums** for memory.
- Enormous in size — some filled entire rooms.
- Generated a lot of heat and were prone to failure.
- **Example:** ENIAC (1945), one of the first general-purpose electronic computers. It weighed about 30 tons and occupied 1,800 square feet.

### Generation 2 (1950s – 1960s): Transistors
- **Transistors** replaced vacuum tubes — smaller, faster, more reliable, and energy-efficient.
- Used **magnetic core memory**.
- Machine language gave way to **assembly language**, then early high-level languages like FORTRAN and COBOL.
- **Example:** IBM 1401.

### Generation 3 (1960s – 1970s): Integrated Circuits (ICs)
- Multiple transistors were miniaturized onto a single **silicon chip** (Integrated Circuit).
- Computers became even smaller and much more affordable.
- **Operating systems** emerged, enabling multitasking.
- **Example:** IBM System/360.

### Generation 4 (1970s – present): Microprocessors
- The **microprocessor** (entire CPU on a single chip) was invented.
- Led to the birth of **personal computers (PCs)**.
- GUIs (Graphical User Interfaces), the internet, and the World Wide Web emerged.
- **Example:** Intel 4004 (1971) – the first commercial microprocessor. Apple II and IBM PC followed.

### Generation 5 (Present – Future): Artificial Intelligence
- Computers are designed to learn, reason, and make decisions.
- Technologies include **machine learning**, **natural language processing**, and **quantum computing**.
- **Example:** Voice assistants (Siri, Alexa), recommendation systems, self-driving cars.

---

## 3. Components of a Computer System

A computer system can be broken down into two fundamental parts: **Hardware** and **Software**.

---

### 3.1 Hardware

Hardware refers to all the **physical components** you can see and touch.

#### 3.1.1 The Central Processing Unit (CPU)

The CPU is the **brain of the computer**. It executes instructions from programs and coordinates all other components.

It has three main parts:

- **Arithmetic Logic Unit (ALU):** Performs mathematical calculations (addition, subtraction, multiplication) and logical comparisons (greater than, equal to, etc.).
- **Control Unit (CU):** Fetches instructions from memory, decodes them, and directs all other components to execute them.
- **Registers:** Tiny, ultra-fast storage locations inside the CPU used to hold data being actively processed.

> **Example:** When you type `5 + 3` in a calculator app, the ALU performs the addition. The Control Unit coordinates where the numbers come from and where the result goes.

---

#### 3.1.2 Memory

Memory is where a computer holds data and instructions. There are two main types:

**Primary Memory (Main Memory)**

| Type | Full Name | Characteristics |
|---|---|---|
| **RAM** | Random Access Memory | Volatile (data lost when power off); fast; used for active programs |
| **ROM** | Read-Only Memory | Non-volatile; stores permanent firmware (e.g., BIOS); cannot normally be written to |
| **Cache** | Cache Memory | Extremely fast; sits between CPU and RAM; stores frequently used data |

> **Analogy:** RAM is like your work desk — you keep current projects on it. ROM is like instructions printed on the wall — permanent and always visible. Cache is like sticky notes on your monitor — the most immediately needed information right at hand.

**Secondary Memory (Storage)**

Used for long-term data storage. Non-volatile (retains data without power).

- **HDD (Hard Disk Drive):** Mechanical spinning disk; larger capacity, slower speed.
- **SSD (Solid-State Drive):** Uses flash memory chips; faster, more durable, but more expensive per GB.
- **USB Flash Drives / Memory Cards:** Portable, compact storage.
- **Optical Discs:** CDs, DVDs, Blu-rays.

---

#### 3.1.3 Input Devices

Input devices allow users to **send data or commands** to the computer.

| Device | Purpose |
|---|---|
| Keyboard | Entering text and commands |
| Mouse | Pointing and clicking on a GUI |
| Scanner | Converting physical documents to digital |
| Microphone | Capturing audio input |
| Webcam | Capturing video/image input |
| Touchscreen | Combined input/output; accepts touch gestures |

---

#### 3.1.4 Output Devices

Output devices **present processed results** to the user.

| Device | Purpose |
|---|---|
| Monitor | Displays visual output (text, images, video) |
| Printer | Produces physical (hard copy) output |
| Speakers | Produces audio output |
| Projector | Displays visuals on a large surface |

---

#### 3.1.5 The Motherboard

The motherboard is the **main circuit board** that connects and allows communication between all hardware components — the CPU, RAM, storage devices, and peripheral connectors (USB, HDMI, etc.). Think of it as the highway system of the computer.

---

### 3.2 Software

Software is the **collection of programs and data** that tells the hardware what to do.

| Category | Description | Examples |
|---|---|---|
| **System Software** | Manages hardware and provides a platform for other software | Windows, macOS, Linux, device drivers |
| **Application Software** | Performs specific tasks for the user | Microsoft Word, Chrome, Photoshop, games |
| **Utility Software** | Supports and maintains the computer | Antivirus, disk cleaner, file compression tools |

> Software will be explored in much greater detail in **Module 4: Operating Systems**.

---

## 4. How a Computer Processes Data

The flow of data through a computer follows the **IPO Model**:

```
INPUT  →  PROCESSING  →  OUTPUT
               ↑
           STORAGE
```

### Step-by-step Example: Searching Google

1. **Input:** You type "weather in Monterrey" into the keyboard and press Enter.
2. **Processing:** The CPU runs the browser's code, sends a network request to Google's servers, and receives data back.
3. **Storage:** The web page data is temporarily stored in RAM. Cached pages may also be stored on your SSD.
4. **Output:** The monitor displays the current weather in Monterrey.

### The Fetch-Decode-Execute Cycle

At the hardware level, CPUs process instructions using a fundamental cycle:

1. **Fetch:** The Control Unit retrieves the next instruction from RAM (using the **Program Counter** register to track position).
2. **Decode:** The instruction is interpreted — what operation is being requested?
3. **Execute:** The ALU or relevant component carries out the instruction.
4. **Store:** The result is written back to a register or to memory.

This cycle repeats **billions of times per second** in modern CPUs. CPU speed is measured in **GHz (gigahertz)** — a 3 GHz CPU can complete approximately 3 billion cycles per second.

---

## 5. Types of Computers

Computers are designed for different purposes and users:

| Type | Description | Examples |
|---|---|---|
| **Supercomputer** | Extremely powerful; used for complex scientific simulations | IBM Summit, Fugaku |
| **Mainframe** | Large, powerful systems used by organizations for bulk data processing | IBM z-Series (used in banking) |
| **Minicomputer (Midrange)** | Mid-sized servers used in businesses | IBM AS/400 |
| **Personal Computer (PC)** | General-purpose desktop or laptop for individuals | Dell, HP, MacBook |
| **Workstation** | High-performance desktop for specialized tasks | Used in 3D animation, engineering |
| **Embedded Computer** | Small computer built into another device | Smart TV, washing machine, ATM |
| **Smartphone / Tablet** | Portable, touch-based computer | iPhone, Samsung Galaxy, iPad |
| **Wearable Computer** | Worn on the body | Smartwatch, fitness tracker |

---

## 6. Software: The Invisible Engine

Without software, hardware is just an expensive piece of metal. Software gives instructions that make hardware useful.

### How Software Works (Simplified)

A programmer writes code in a **high-level language** like Python:

```python
name = input("What is your name? ")
print("Hello, " + name + "!")
```

This code is then translated into **machine language** (binary: 0s and 1s) that the CPU can directly execute. This translation is done by a **compiler** or **interpreter**.

The CPU doesn't understand Python — it only understands electrical signals representing 0s and 1s.

### The Relationship Between Hardware and Software

```
User
  ↓
Application Software (e.g., Chrome, Word)
  ↓
Operating System (e.g., Windows, macOS)
  ↓
Hardware (CPU, RAM, Storage, etc.)
```

Each layer depends on the one below it. The OS sits in between, acting as a **translator and manager** between applications and hardware.

---

## 7. Module Summary

Here are the key takeaways from Module 1:

- A **computer system** is a combination of hardware and software that processes data to produce useful information.
- Computers have evolved through **five generations**, from room-sized vacuum tube machines to AI-powered systems.
- The **hardware** of a computer includes the CPU (ALU + Control Unit + Registers), memory (RAM, ROM, Cache, Storage), input devices, output devices, and the motherboard.
- Data flows through a computer via the **IPO model** (Input → Processing → Output), and the CPU uses the **Fetch-Decode-Execute cycle** to process instructions.
- **Software** (system, application, and utility) provides the instructions that make hardware useful.
- Computers come in many types, from **supercomputers** to **embedded computers**, each designed for different needs.

---

## 8. Module Quiz

Test your understanding of Module 1. Try answering each question before checking the answers.

---

**Question 1**
What are the three main components of a CPU?

- A) ALU, Hard Drive, Control Unit
- B) ALU, Control Unit, Registers
- C) RAM, ROM, Cache
- D) Motherboard, CPU, GPU

---

**Question 2**
Which type of memory is **volatile** (loses its contents when the computer is turned off)?

- A) ROM
- B) SSD
- C) RAM
- D) HDD

---

**Question 3**
A hospital uses a powerful computer to process millions of patient records and run critical billing systems for thousands of users simultaneously. What type of computer is most likely being used?

- A) Personal Computer
- B) Embedded Computer
- C) Mainframe
- D) Wearable Computer

---

**Question 4**
Which generation of computers introduced the **microprocessor** and led to the development of personal computers?

- A) First Generation
- B) Second Generation
- C) Third Generation
- D) Fourth Generation

---

**Question 5**
Put the following steps of the CPU's instruction cycle in the correct order:

1. Execute
2. Decode
3. Store
4. Fetch

- A) 4 → 2 → 1 → 3
- B) 1 → 2 → 3 → 4
- C) 4 → 1 → 2 → 3
- D) 2 → 4 → 1 → 3

---

**Question 6**
Which of the following is an example of **system software**?

- A) Microsoft Word
- B) Google Chrome
- C) Windows 11
- D) Adobe Photoshop

---

**Question 7**
A smart thermostat in a home uses a small computer chip to control heating and cooling. What type of computer is this an example of?

- A) Supercomputer
- B) Mainframe
- C) Workstation
- D) Embedded Computer

---

**Question 8** *(Short Answer)*
Explain the **IPO Model** in your own words and provide one real-life example of it in action.

---

### Answer Key

| Question | Answer | Explanation |
|---|---|---|
| 1 | **B** | The CPU's three parts are the Arithmetic Logic Unit (ALU), Control Unit (CU), and Registers. |
| 2 | **C** | RAM is volatile — it requires power to retain data. ROM, SSDs, and HDDs are non-volatile. |
| 3 | **C** | Mainframes are designed for large-scale, high-volume data processing for many users simultaneously. |
| 4 | **D** | The Fourth Generation introduced the microprocessor (Intel 4004, 1971), enabling personal computers. |
| 5 | **A** | The correct order is Fetch → Decode → Execute → Store. |
| 6 | **C** | Windows 11 is an operating system — a type of system software. The others are application software. |
| 7 | **D** | A thermostat with a built-in chip is an embedded computer, designed for one specific task. |
| 8 | *(See below)* | |

**Question 8 – Sample Answer:**
> The IPO Model describes how a computer handles data. **Input** is any data or instruction provided to the computer (e.g., pressing keys on a keyboard). **Processing** is when the CPU performs operations on that data according to a program's instructions. **Output** is the result delivered to the user (e.g., text appearing on the screen). Storage can also hold data before or after processing. *Real-life example:* When you use a calculator app to multiply 12 × 8: you type the numbers and the × operator (Input), the CPU multiplies them (Processing), and the result "96" appears on the screen (Output).

---

*End of Module 1 — Up next: **Module 2: Number Systems and Codes***
