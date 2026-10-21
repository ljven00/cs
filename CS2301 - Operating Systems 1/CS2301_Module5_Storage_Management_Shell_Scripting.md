# CS 2301 — Operating Systems 1
## Module 5: Storage Management and Shell Scripting

---

> **Course Progress:** Module 5 of 8 — CS 2301 (OS 1)

---

## 1. Storage Hierarchy

Not all storage is equal. The OS manages a hierarchy — faster storage costs more and holds less; slower storage is cheap and large.

```
         Speed / Cost
              ▲
              │   Registers       (bytes, nanoseconds)
              │   CPU Cache       (KB–MB, nanoseconds)
              │   RAM             (GB, nanoseconds–microseconds)
              │   SSD             (GB–TB, microseconds)
              │   HDD             (TB, milliseconds)
              ▼   Tape/Cloud      (PB, seconds–minutes)
         Capacity
```

The OS tries to keep frequently used data as high up the hierarchy as possible (caching).

---

## 2. Disk Structure

A traditional **Hard Disk Drive (HDD)** stores data on spinning magnetic platters.

```
         Spindle
            │
   ┌────────┼────────┐
   │  Track │        │  ← Platter
   │    ┌───┘        │
   │    │  Sector    │
   └────┴────────────┘
         ↑
       Read/Write Head (on an arm)
```

Key terms:

| Term | Meaning |
|------|---------|
| **Platter** | Circular magnetic disk |
| **Track** | Concentric ring on a platter |
| **Sector** | Smallest addressable unit on a track (typically 512B or 4KB) |
| **Cylinder** | Same track across all platters |
| **Seek time** | Time for the arm to move to the right track |
| **Rotational latency** | Time waiting for the sector to rotate under the head |
| **Transfer time** | Time to read/write the data |

**SSDs** have none of these mechanical parts — data is stored in flash memory cells, making them much faster with no seek or rotational latency.

---

## 3. Disk Scheduling Algorithms

The OS queues multiple disk I/O requests. The order in which they are served affects performance significantly.

Imagine the read/write head is at track **50**, and requests are for tracks: **98, 183, 37, 122, 14, 124, 65, 67**

### 3.1 FCFS (First-Come, First-Served)
Serve requests in arrival order.

Movement: 50→98→183→37→122→14→124→65→67

Total head movement: **640 tracks** — not efficient.

---

### 3.2 SSTF (Shortest Seek Time First)
Always serve the closest request next.

From 50: nearest is 65 → 67 → 37 → 14 → 98 → 122 → 124 → 183

Total head movement: **236 tracks** — much better. Risk: starvation of far tracks.

---

### 3.3 SCAN (Elevator Algorithm)
The head moves in one direction, serving all requests along the way, then reverses.

50→65→67→98→122→124→183 → reverse → 37→14

Total head movement: **208 tracks** — efficient and fair.

---

### 3.4 C-SCAN (Circular SCAN)
Like SCAN, but when the head reaches the end, it jumps back to the beginning without serving on the return trip. More uniform wait times.

---

### Comparison

| Algorithm | Head Movement | Starvation Risk | Notes |
|-----------|--------------|-----------------|-------|
| FCFS | High | No | Simple, poor performance |
| SSTF | Low | Yes | Greedy, unfair to outer tracks |
| SCAN | Medium | No | Like an elevator |
| C-SCAN | Medium | No | More uniform response times |

---

## 4. File Systems

A **file system** defines how data is organized and stored on disk. It provides the abstraction of **files** and **directories** on top of raw sectors.

### Responsibilities

- Naming files and organizing them into directories
- Tracking which disk blocks belong to which file
- Managing free space
- Enforcing permissions

### Common File Systems

| File System | Used By |
|-------------|---------|
| FAT32 | USB drives, older Windows |
| NTFS | Modern Windows |
| ext4 | Linux |
| APFS | macOS |
| exFAT | Cross-platform flash drives |

---

## 5. File Allocation Methods

How does the OS track which disk blocks belong to a file?

### 5.1 Contiguous Allocation
Each file occupies a consecutive run of blocks.

```
Block:  1   2   3   4   5   6   7   8
File A: [A] [A] [A] [ ] [B] [B] [ ] [C]
```

- Fast sequential access
- External fragmentation over time
- Hard to grow files

### 5.2 Linked Allocation
Each block contains a pointer to the next block in the file.

```
Block 1 → Block 4 → Block 7 → Block 9 → NULL
```

- No external fragmentation
- Slow random access (must follow the chain)
- One bad pointer breaks the whole file

### 5.3 Indexed Allocation
A dedicated **index block** holds pointers to all the file's data blocks.

```
Index Block: [ptr→3, ptr→7, ptr→12, ptr→20]
```

- Good random access
- Wastes space on the index block for small files
- Used by most modern file systems (Unix inodes)

---

## 6. Free Space Management

The OS must track which disk blocks are free.

- **Bitmap** — one bit per block (1 = used, 0 = free). Compact and fast to search.
- **Free list** — linked list of free blocks. Simple but slow to find contiguous space.

---

## 7. Introduction to Shell Scripting

The **shell** is a command-line interface to the OS. A **shell script** is a text file containing a sequence of shell commands, allowing you to automate tasks.

Common shells: `bash` (Linux/macOS default), `zsh`, `sh`

---

## 8. Shell Scripting Basics

### 8.1 Your First Script

```bash
#!/bin/bash
# This is a comment
echo "Hello, Operating Systems!"
```

- The first line `#!/bin/bash` is called the **shebang** — it tells the OS which interpreter to use
- Save as `hello.sh`, then run:

```bash
chmod +x hello.sh   # make it executable
./hello.sh          # run it
```

---

### 8.2 Variables

```bash
#!/bin/bash
name="Alice"
echo "Hello, $name"

# Capture command output into a variable
current_dir=$(pwd)
echo "You are in: $current_dir"
```

---

### 8.3 User Input

```bash
#!/bin/bash
echo "Enter your name:"
read username
echo "Welcome, $username!"
```

---

### 8.4 Conditionals

```bash
#!/bin/bash
echo "Enter a number:"
read num

if [ $num -gt 10 ]; then
    echo "Greater than 10"
elif [ $num -eq 10 ]; then
    echo "Equal to 10"
else
    echo "Less than 10"
fi
```

Common comparison operators:

| Operator | Meaning |
|----------|---------|
| `-eq` | Equal |
| `-ne` | Not equal |
| `-gt` | Greater than |
| `-lt` | Less than |
| `-ge` | Greater than or equal |
| `-le` | Less than or equal |

---

### 8.5 Loops

```bash
#!/bin/bash

# For loop
for i in 1 2 3 4 5; do
    echo "Count: $i"
done

# While loop
count=1
while [ $count -le 5 ]; do
    echo "While count: $count"
    count=$((count + 1))
done
```

---

### 8.6 Functions

```bash
#!/bin/bash

greet() {
    echo "Hello, $1!"   # $1 = first argument
}

greet "Alice"
greet "Bob"
```

---

## 9. Practical Shell Script Example

A script that backs up all `.txt` files in a directory:

```bash
#!/bin/bash

SOURCE_DIR="$HOME/documents"
BACKUP_DIR="$HOME/backup"
DATE=$(date +%Y-%m-%d)

# Create backup directory if it doesn't exist
mkdir -p "$BACKUP_DIR/$DATE"

# Copy all .txt files
for file in "$SOURCE_DIR"/*.txt; do
    if [ -f "$file" ]; then
        cp "$file" "$BACKUP_DIR/$DATE/"
        echo "Backed up: $file"
    fi
done

echo "Backup complete: $BACKUP_DIR/$DATE"
```

This script demonstrates variables, date formatting, directory creation, loops, file checks, and output — all in one practical task.

---

## 10. Useful Shell Commands for Storage

| Command | Purpose |
|---------|---------|
| `df -h` | Show disk space usage (human-readable) |
| `du -sh folder/` | Show size of a folder |
| `ls -lh` | List files with sizes |
| `fdisk -l` | List disk partitions (root needed) |
| `mount` | Show mounted file systems |
| `find / -name "file.txt"` | Search for a file |

---

## 11. Real-World Example: Automated Log Cleanup

System logs grow over time and can fill up a disk. A sysadmin might run this script nightly:

```bash
#!/bin/bash

LOG_DIR="/var/log/myapp"
MAX_DAYS=30

echo "Deleting logs older than $MAX_DAYS days..."
find "$LOG_DIR" -name "*.log" -mtime +$MAX_DAYS -delete
echo "Cleanup complete."
```

This is a real pattern used by every production Linux server. The OS scheduler (cron) can run this script automatically at 2 AM every night:

```bash
0 2 * * * /home/admin/cleanup_logs.sh
```

---

## 12. Summary

| Concept | Key Takeaway |
|---------|-------------|
| Storage hierarchy | Faster = smaller and costlier; OS uses caching |
| Disk components | Platters, tracks, sectors; seek + rotational latency |
| Disk scheduling | FCFS, SSTF, SCAN, C-SCAN — minimize head movement |
| File system | Organizes raw disk blocks into files and directories |
| File allocation | Contiguous (fast), Linked (flexible), Indexed (both) |
| Free space | Bitmap or free list tracks available blocks |
| Shell script | Automate OS tasks with variables, loops, conditionals |
| Shebang | `#!/bin/bash` — declares the script interpreter |

---

## Module 5 Quiz

---

**Question 1**
What are the three components of disk access time, and which one does an SSD eliminate entirely?

> **Answer:** The three components are **seek time** (arm moves to the right track), **rotational latency** (platter spins to the right sector), and **transfer time** (data is read/written). SSDs eliminate all three since they have no moving parts — there are no arms or platters.

---

**Question 2**
A disk head is at track 40. Pending requests are for tracks: 10, 60, 25, 80, 15. Using **SSTF**, what is the order of service?

> **Answer:**
> - From 40: nearest is 25 (distance 15)
> - From 25: nearest is 15 (distance 10)
> - From 15: nearest is 10 (distance 5)
> - From 10: nearest is 60 (distance 50)
> - From 60: nearest is 80 (distance 20)
>
> Order: **40 → 25 → 15 → 10 → 60 → 80**
> Total movement: 15 + 10 + 5 + 50 + 20 = **100 tracks**

---

**Question 3**
What is the difference between contiguous and indexed file allocation?

> **Answer:** Contiguous allocation stores all file blocks in a row on disk — fast to access but causes fragmentation and makes it hard to grow files. Indexed allocation uses a separate index block holding pointers to all the file's data blocks — supports random access and easy growth, but wastes space on the index for small files.

---

**Question 4**
What does this shell script do?

```bash
#!/bin/bash
for file in *.log; do
    if [ -f "$file" ]; then
        echo "Found: $file"
    fi
done
```

> **Answer:** It loops through every `.log` file in the current directory. For each one, it checks whether it is a regular file (not a directory), and if so, prints its name prefixed with "Found:".

---

**Question 5**
Write a shell script that asks the user for a number and prints whether it is even or odd.

> **Answer:**
> ```bash
> #!/bin/bash
> echo "Enter a number:"
> read num
>
> if [ $((num % 2)) -eq 0 ]; then
>     echo "$num is even"
> else
>     echo "$num is odd"
> fi
> ```
> The modulo operator `%` gives the remainder when dividing by 2. If it equals 0, the number is even.

---

## What's Next

**Module 6: Input / Output Management**

You'll learn how the OS communicates with devices — keyboards, disks, network cards — and the techniques it uses to do so efficiently.

---

*CS 2301 Operating Systems 1 — Module 5 of 8*
