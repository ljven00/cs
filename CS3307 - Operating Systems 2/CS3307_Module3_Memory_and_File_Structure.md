# CS 3307 — Operating Systems 2
## Module 3: Memory and File Structure

---

> **Course Progress:** Module 3 of 8 — CS 3307 (OS 2)

---

## 1. Building on the Foundations

CS 2301 covered the mechanics of paging, virtual memory, and basic file allocation. This module goes deeper — how memory is structured at the hardware level, how the OS bridges hardware and software memory management, and how file systems are architecturally organized on disk.

---

## 2. Physical Memory Architecture

### 2.1 Memory Hierarchy Revisited

Modern systems have several layers of memory between the CPU and main RAM:

```
CPU
 ├── L1 Cache    (~32 KB,   ~1 ns)    <- per core
 ├── L2 Cache    (~256 KB,  ~5 ns)    <- per core
 ├── L3 Cache    (~8-32 MB, ~20 ns)   <- shared across cores
 └── RAM (DRAM)  (~GB,      ~60 ns)
           └── SSD / HDD   (~TB,      ~100 us - 10 ms)
```

Every memory access first checks L1, then L2, then L3, then RAM. A cache miss at L3 forces a RAM access — 60x slower than L1.

### 2.2 NUMA — Non-Uniform Memory Access

On multi-socket servers, each CPU socket has its own local RAM. Accessing local RAM is fast; accessing another socket's RAM crosses an interconnect — slower.

```
Socket 0              Socket 1
+----------+          +----------+
|  CPU 0   |          |  CPU 1   |
|  Cores   |          |  Cores   |
+----------+          +----------+
| Local RAM|---link---| Local RAM|
+----------+          +----------+
```

The OS must be NUMA-aware — schedule threads on cores close to their data, and allocate memory on the same socket as the thread using it.

```bash
numactl --hardware          # show NUMA topology
numactl --cpunodebind=0 ./program   # run on socket 0
```

---

## 3. Virtual Memory in Depth

### 3.1 The Full Address Translation Path

When a process accesses a virtual address, the full translation path is:

```
Virtual Address
      |
TLB (Translation Lookaside Buffer) -- cache of recent translations
      | (if TLB miss)
Page Table Walk (multi-level page table in memory)
      |
Physical Address
      |
CPU Cache (L1 -> L2 -> L3)
      | (if cache miss)
RAM
```

### 3.2 The TLB

The **TLB (Translation Lookaside Buffer)** is a small, very fast hardware cache storing recent virtual-to-physical address mappings.

- TLB **hit**: translation found instantly (~1 cycle)
- TLB **miss**: must walk the page table (~100+ cycles)

On a context switch, the TLB is typically flushed (or tagged with an Address Space ID — ASID — to avoid full flushes). This is one reason context switches have overhead.

### 3.3 Multi-Level Page Tables

A flat page table for a 64-bit address space would be enormous. Modern OSes use multi-level page tables — only the portions of the address space actually in use need page table entries.

```
Virtual Address:  [L1 index | L2 index | L3 index | offset]
                       |
                  L1 Page Table
                       |
                  L2 Page Table
                       |
                  L3 Page Table
                       |
                  Physical Frame
```

Linux x86-64 uses a 4-level (or 5-level on newer hardware) page table.

### 3.4 Huge Pages

Standard pages are 4 KB. **Huge pages** are 2 MB or 1 GB. Benefits:

- Fewer TLB entries needed to cover the same memory
- Fewer page table levels to walk
- Better performance for memory-intensive workloads (databases, VMs)

```bash
# Enable transparent huge pages on Linux
echo always > /sys/kernel/mm/transparent_hugepage/enabled
```

---

## 4. Memory Mapped Files

**Memory mapping** (`mmap`) allows a file to be mapped directly into a process's virtual address space. Reading/writing the mapped memory region reads/writes the file — no explicit `read()`/`write()` calls needed.

```c
#include <sys/mman.h>
#include <fcntl.h>

int fd = open("data.bin", O_RDWR);
fstat(fd, &sb);

// Map the file into virtual memory
char *data = mmap(NULL, sb.st_size, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);

// Access the file like an array
data[0] = 'X';   // writes directly to the file

munmap(data, sb.st_size);
```

### Why Use mmap?

- **Performance** — the OS uses demand paging; only accessed pages are loaded
- **Shared memory** — two processes can map the same file and share data
- **Simplicity** — treat a large file like an array
- Used by: databases, JVM class loading, dynamic linkers (`ld.so`)

---

## 5. File System Architecture

### 5.1 Disk Layout Overview

A disk partition formatted with a file system has this general structure:

```
+------------+----------+------------+------------------+
| Boot Block | Superblock| Inode Table |   Data Blocks    |
+------------+----------+------------+------------------+
```

| Region | Purpose |
|--------|---------|
| **Boot Block** | Code to start the OS (only in the first partition) |
| **Superblock** | Metadata about the file system — size, block size, free block count |
| **Inode Table** | Array of inodes — one per file/directory |
| **Data Blocks** | Actual file and directory content |

### 5.2 The Inode

An **inode (index node)** stores all metadata about a file — except its name.

An inode contains:
- File type (regular, directory, symlink, device)
- Owner (UID) and group (GID)
- Permissions (rwxrwxrwx)
- File size
- Timestamps (created, modified, accessed)
- Link count (how many directory entries point to this inode)
- Pointers to data blocks

```
Inode #42:
  type:        regular file
  owner:       alice (UID 1001)
  permissions: rw-r--r--
  size:        4096 bytes
  modified:    2026-05-01 09:00
  blocks:      [ptr->block 7, ptr->block 23, ptr->block 51]
```

The file's **name** lives in a directory entry, not the inode. This is why you can rename a file without modifying its inode.

### 5.3 Block Pointers — Direct, Indirect, Double Indirect

For small files, the inode holds direct pointers to data blocks. For larger files:

```
Inode
 |-- Direct pointers (x12)          -> Data blocks
 |-- Single indirect pointer        -> Block of pointers -> Data blocks
 |-- Double indirect pointer        -> Block of pointers -> Blocks of pointers -> Data
 +-- Triple indirect pointer        -> (one more level)
```

With 4 KB blocks and 8-byte pointers:
- 12 direct blocks = 48 KB
- 1 indirect block = 512 pointers x 4 KB = 2 MB
- 1 double indirect = 512 x 512 x 4 KB = 1 GB
- 1 triple indirect = 512^3 x 4 KB = 512 GB

This structure efficiently handles both tiny and massive files.

---

## 6. Directories

A **directory** is a special file whose data blocks contain a list of (name -> inode number) mappings.

```
Directory: /home/alice/
  Entry: "."         -> inode 42   (this directory itself)
  Entry: ".."        -> inode 5    (parent directory)
  Entry: "notes.txt" -> inode 108
  Entry: "projects"  -> inode 214
```

When you access `/home/alice/notes.txt`, the OS:
1. Starts at the root inode (`/`)
2. Reads root's directory — finds `home` -> inode N
3. Reads inode N's directory — finds `alice` -> inode 42
4. Reads inode 42's directory — finds `notes.txt` -> inode 108
5. Reads inode 108 to get the file's data block locations
6. Reads the data blocks

This is called **path resolution** or **path traversal**.

---

## 7. Hard Links and Symbolic Links

### Hard Links

A hard link is another directory entry pointing to the same inode.

```bash
ln notes.txt notes_backup.txt
# Both names point to inode 108 -- same data, same inode
```

- Deleting one name decrements the inode's link count
- Data is only deleted when link count reaches 0
- Cannot cross file system boundaries

### Symbolic (Soft) Links

A symbolic link is a special file containing a path string pointing to another file.

```bash
ln -s /home/alice/notes.txt /tmp/notes_link
```

- Can cross file systems and point to directories
- If the target is deleted, the symlink becomes a **dangling link**

```
Hard link:   name_A --+
                       +--> inode 108 --> data
             name_B --+

Symlink:     name_C --> "path/to/file" --> (resolved at access time)
```

---

## 8. Journaling File Systems

What happens if the system crashes mid-write? Without protection, the file system can be left inconsistent.

A **journaling file system** writes changes to a **journal (log)** before applying them. If a crash occurs:
- Complete journal entry -> replay it to finish the operation
- Incomplete journal entry -> discard it, as if the operation never happened

```
Write operation:
  1. Write intent to journal        <- fast, sequential write
  2. Apply changes to file system   <- actual update
  3. Mark journal entry as complete <- commit
```

| File System | Journal Type | OS |
|-------------|-------------|-----|
| ext4 | Ordered journaling | Linux |
| NTFS | Log-structured | Windows |
| APFS | Copy-on-write | macOS |
| XFS | Metadata journaling | Linux (high-performance) |
| Btrfs | Copy-on-write | Linux |

---

## 9. Copy-on-Write (CoW) File Systems

Instead of modifying data in place, CoW writes modified data to a new location and atomically updates the pointer.

```
Before write:
  Inode -> Block A (data = "hello")

CoW write of "world":
  1. Write "world" to new Block B
  2. Atomically update Inode -> Block B
  3. Block A is now free

After:
  Inode -> Block B (data = "world")
```

Benefits:
- No inconsistency possible — the pointer update is atomic
- Snapshots are free — just keep an old pointer
- Used by APFS (macOS), Btrfs, ZFS

---

## 10. Real-World Example: What `ls -l` Actually Does

```bash
$ ls -l /home/alice/notes.txt
-rw-r--r-- 1 alice staff 1024 May 1 09:00 notes.txt
```

Here is what the OS did:

1. Resolved the path through directory traversal
2. Found the inode number for `notes.txt` in alice's directory
3. Read the inode — extracted permissions, link count, owner, group, size, and modification time
4. Printed all of that — the filename came from the directory entry, everything else from the inode

The actual file data (content) was never read — `ls -l` only touches the inode.

---

## 11. Summary

| Concept | Key Takeaway |
|---------|-------------|
| TLB | Cache of virtual->physical translations; miss is expensive |
| Multi-level page tables | Efficient for sparse 64-bit address spaces |
| Huge pages | Fewer TLB entries; better for large memory workloads |
| mmap | Map files into virtual memory; used for shared memory and performance |
| Superblock | File system metadata — size, block count, free space |
| Inode | Per-file metadata (permissions, size, block pointers) — not the name |
| Direct/indirect pointers | Inode structure that scales from tiny to huge files |
| Directory | Special file mapping names to inode numbers |
| Hard link | Multiple names -> same inode; data deleted when count reaches 0 |
| Symbolic link | A file containing a path string; can cross file systems |
| Journaling | Write-ahead log prevents corruption on crash |
| Copy-on-write | Write to new location, atomically update pointer; enables snapshots |

---

## Module 3 Quiz

---

**Question 1**
What is a TLB and what happens on a TLB miss? Why does a context switch affect TLB performance?

> **Answer:** The TLB (Translation Lookaside Buffer) is a small hardware cache storing recent virtual-to-physical address mappings. A TLB hit resolves a translation in ~1 cycle. A TLB miss requires walking the multi-level page table — ~100+ cycles. On a context switch, the new process has a completely different address space, so existing TLB entries are invalid. The TLB is flushed (or tagged with a new ASID), meaning the new process starts cold and suffers TLB misses until its working set is re-cached — this is part of context switch overhead.

---

**Question 2**
What information is stored in an inode? What is notably NOT stored there?

> **Answer:** An inode stores file type, owner (UID), group (GID), permissions, size, timestamps, link count, and pointers to data blocks. What is NOT stored is the **file name**. The name lives in a directory entry that maps the name to an inode number. This separation is why renaming a file is fast (just update the directory entry) and why hard links work (multiple names, one inode).

---

**Question 3**
A file is 10 MB. An inode has 12 direct block pointers and one single indirect pointer. Block size is 4 KB and each pointer is 8 bytes. Is 10 MB reachable with just direct and single indirect pointers?

> **Answer:**
> - 12 direct blocks = 12 x 4 KB = 48 KB
> - Single indirect: 4096 / 8 = 512 pointers x 4 KB = 2 MB
> - Total reachable = ~2.09 MB
>
> A 10 MB file is NOT reachable. The double indirect pointer would be needed: 512 x 512 x 4 KB = 1 GB, which easily covers 10 MB.

---

**Question 4**
What is the difference between a hard link and a symbolic link? Give one scenario where you would prefer each.

> **Answer:** A hard link is a directory entry pointing directly to an inode — another name for the same file data. Deleting one name does not delete the data until all links are gone. A symbolic link is a special file containing a path string — it can cross file system boundaries and point to directories, but becomes invalid if the target is deleted.
>
> Use a hard link when you want a true backup name for a file on the same file system. Use a symbolic link when you need to reference a file across partitions or create a convenient alias, such as `/usr/bin/python` pointing to `/usr/bin/python3.12`.

---

**Question 5**
Explain how journaling prevents corruption after a crash. What is the key difference between journaling and copy-on-write?

> **Answer:** A journaling file system writes intended changes to a sequential journal before applying them. On crash recovery, complete journal entries are replayed; incomplete ones are discarded — the file system is always consistent. Copy-on-write never overwrites existing data. New data goes to a fresh location, then a single atomic pointer update switches the file system to the new version. If the system crashes before the pointer update, the old data is still valid — no replay needed. CoW also makes snapshots trivial by preserving old pointers. APFS and Btrfs use CoW; ext4 and NTFS use journaling.

---

## What's Next

**Module 4: File Handling**

You will learn how programs open, read, write, and manage files through the OS — file descriptors, buffering, seeking, locking, and the system calls that power all file operations.

---

*CS 3307 Operating Systems 2 — Module 3 of 8*
