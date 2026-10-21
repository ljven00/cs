# CS 3307 — Operating Systems 2
## Module 4: File Handling

---

> **Course Progress:** Module 4 of 8 — CS 3307 (OS 2)

---

## 1. What is File Handling?

Module 3 covered how files are structured on disk — inodes, blocks, directories. This module covers how programs actually **interact** with files at runtime — opening, reading, writing, seeking, locking, and closing them through the OS.

Every file operation a program performs goes through the OS via system calls. Understanding this layer is essential for writing correct, efficient programs.

---

## 2. File Descriptors

When a program opens a file, the OS returns a **file descriptor (fd)** — a small non-negative integer that represents the open file for that process.

```
Process File Descriptor Table:
  fd 0  -> stdin  (keyboard)
  fd 1  -> stdout (screen)
  fd 2  -> stderr (screen, errors)
  fd 3  -> notes.txt       <- opened by your program
  fd 4  -> database.db     <- another open file
```

- Every process starts with fd 0, 1, and 2 pre-opened
- `open()` returns the lowest available fd number
- The fd is just an index into the process's **open file table**

### The Open File Table

The OS maintains two levels of tables:

```
Process A fd table          Kernel Open File Table        Inode Table
  fd 3 ──────────────────► [offset=0, flags=RDONLY] ────► inode 108
  fd 4 ──────────────────► [offset=512, flags=RDWR] ────► inode 214

Process B fd table
  fd 3 ──────────────────► [offset=200, flags=RDONLY] ──► inode 108
```

- Multiple processes can open the same file — each gets its own entry in the kernel open file table with its own **offset**
- The kernel open file table points to the inode (the on-disk file metadata)
- This is why two processes reading the same file don't interfere with each other's position

---

## 3. Core File System Calls

### 3.1 `open()` — Open a File

```c
#include <fcntl.h>

int fd = open("notes.txt", O_RDONLY);           // open for reading
int fd = open("log.txt",   O_WRONLY | O_CREAT | O_TRUNC, 0644); // create/overwrite
int fd = open("data.txt",  O_RDWR);             // open for reading and writing
```

Common flags:

| Flag | Meaning |
|------|---------|
| `O_RDONLY` | Open for reading only |
| `O_WRONLY` | Open for writing only |
| `O_RDWR` | Open for reading and writing |
| `O_CREAT` | Create file if it does not exist |
| `O_TRUNC` | Truncate file to zero length on open |
| `O_APPEND` | All writes go to end of file |
| `O_EXCL` | Fail if file already exists (used with O_CREAT) |

The third argument to `open()` (e.g., `0644`) sets the **permissions** when creating a new file — owner read/write, group read, others read.

---

### 3.2 `read()` — Read from a File

```c
#include <unistd.h>

char buffer[256];
ssize_t bytes_read = read(fd, buffer, sizeof(buffer));

if (bytes_read == -1) {
    perror("read failed");
} else if (bytes_read == 0) {
    // End of file reached
}
```

- Reads up to `count` bytes from the file at the current offset
- Returns the number of bytes actually read (may be less than requested)
- Returns 0 at end of file, -1 on error
- **Advances the file offset** by the number of bytes read

---

### 3.3 `write()` — Write to a File

```c
const char *msg = "Hello, file!\n";
ssize_t bytes_written = write(fd, msg, strlen(msg));

if (bytes_written == -1) {
    perror("write failed");
}
```

- Writes `count` bytes to the file at the current offset
- Returns the number of bytes actually written
- **Advances the file offset** by the number of bytes written
- Data may sit in the OS **page cache** before being flushed to disk

---

### 3.4 `lseek()` — Move the File Offset

```c
#include <unistd.h>

// Seek to byte 100 from the start
lseek(fd, 100, SEEK_SET);

// Seek 50 bytes forward from current position
lseek(fd, 50, SEEK_CUR);

// Seek to 10 bytes before the end of file
lseek(fd, -10, SEEK_END);

// Get current offset
off_t pos = lseek(fd, 0, SEEK_CUR);
```

| Whence | Meaning |
|--------|---------|
| `SEEK_SET` | Offset from beginning of file |
| `SEEK_CUR` | Offset from current position |
| `SEEK_END` | Offset from end of file |

`lseek()` allows **random access** — jump to any byte in the file without reading everything before it.

---

### 3.5 `close()` — Close a File

```c
close(fd);
```

- Releases the file descriptor back to the OS
- Flushes any pending writes (OS may buffer them)
- Always close files when done — file descriptors are a limited resource
- Failing to close files causes **fd leaks** — eventually the process runs out

---

### 3.6 `fsync()` — Flush to Disk

```c
fsync(fd);   // force all buffered writes to disk
```

`write()` returns as soon as data is in the OS page cache — it may not be on disk yet. `fsync()` blocks until the OS guarantees the data is physically written. Critical for databases and any application where data loss on crash is unacceptable.

---

## 4. Buffered vs. Unbuffered I/O

### Unbuffered I/O (system calls directly)

```c
// Every write() goes directly to the OS kernel
write(fd, "hello", 5);
write(fd, "world", 5);
// Two separate system calls -- two kernel transitions
```

### Buffered I/O (C standard library)

```c
#include <stdio.h>

FILE *f = fopen("notes.txt", "w");
fprintf(f, "hello");   // goes into a library buffer
fprintf(f, "world");   // still in library buffer
fclose(f);             // now one write() system call is made
```

The C standard library (`stdio`) maintains a **user-space buffer**. Writes accumulate in the buffer and are flushed to the OS in larger chunks — far fewer system calls.

| | Buffered (stdio) | Unbuffered (syscalls) |
|-|-----------------|----------------------|
| Functions | `fopen`, `fread`, `fwrite`, `fclose` | `open`, `read`, `write`, `close` |
| Buffer location | User space | None (or OS page cache) |
| System calls | Fewer (batched) | One per operation |
| Control | Less (library decides when to flush) | More (you control every write) |
| Use case | General text/data files | Sockets, devices, low-level control |

---

## 5. File Positioning and Random Access

### Reading a Binary File Record by Record

```c
typedef struct {
    int id;
    char name[50];
    float score;
} Student;

Student s;
int fd = open("students.bin", O_RDONLY);

// Read the 5th student record (0-indexed)
lseek(fd, 4 * sizeof(Student), SEEK_SET);
read(fd, &s, sizeof(Student));

printf("Name: %s, Score: %.1f\n", s.name, s.score);
close(fd);
```

This pattern — fixed-size records + `lseek` — is the basis of simple binary databases and is far faster than scanning the whole file.

---

## 6. File Locking

When multiple processes access the same file simultaneously, locking prevents corruption.

### 6.1 Advisory Locking — `flock()`

```c
#include <sys/file.h>

int fd = open("shared.txt", O_RDWR);

flock(fd, LOCK_EX);   // acquire exclusive (write) lock -- blocks if another holds it
    // ... modify the file safely ...
flock(fd, LOCK_UN);   // release lock

// For a shared (read) lock:
flock(fd, LOCK_SH);
```

| Lock Type | Meaning |
|-----------|---------|
| `LOCK_SH` | Shared lock — multiple readers allowed |
| `LOCK_EX` | Exclusive lock — only one writer |
| `LOCK_UN` | Unlock |
| `LOCK_NB` | Non-blocking — return immediately if lock unavailable |

`flock()` is **advisory** — it only works if all processes agree to use it. A process that ignores `flock()` can still read/write freely.

### 6.2 POSIX Record Locking — `fcntl()`

`fcntl()` locking is more powerful — it can lock **byte ranges** within a file, not just the whole file.

```c
#include <fcntl.h>

struct flock fl;
fl.l_type   = F_WRLCK;    // write lock
fl.l_whence = SEEK_SET;
fl.l_start  = 0;          // lock from byte 0
fl.l_len    = 100;         // lock 100 bytes

fcntl(fd, F_SETLKW, &fl); // set lock (W = wait if blocked)
    // ... modify bytes 0-99 safely ...

fl.l_type = F_UNLCK;
fcntl(fd, F_SETLK, &fl);  // unlock
```

Used by databases to lock individual records (rows) within a large file.

---

## 7. Duplicating File Descriptors

### `dup()` and `dup2()`

`dup()` creates a copy of a file descriptor — both point to the same open file table entry (same offset, same file).

```c
int fd2 = dup(fd);     // fd2 is a copy of fd

// dup2 redirects a specific fd number
dup2(fd, 1);           // make stdout (fd 1) point to fd's file
                       // now printf() writes to the file!
```

This is how shells implement **output redirection**:

```bash
ls -l > output.txt
```

The shell does:
1. Opens `output.txt` → gets fd 3
2. `dup2(3, 1)` — redirects stdout to the file
3. `exec("ls")` — ls writes to stdout, which now goes to the file

---

## 8. The `/proc` Filesystem

On Linux, `/proc` is a **virtual file system** — it looks like files on disk, but the data is generated live by the kernel.

```bash
cat /proc/1234/fd          # list open file descriptors for process 1234
cat /proc/1234/maps        # virtual memory map of process 1234
cat /proc/meminfo          # current memory usage
cat /proc/cpuinfo          # CPU details
cat /proc/self/status      # info about the current process
```

This means the OS exposes its own internals as files — consistent with the Unix "everything is a file" philosophy. Tools like `top`, `ps`, and `htop` read from `/proc` to get process information.

---

## 9. Complete Example: Safe File Copy

This example brings together `open`, `read`, `write`, `close`, and error handling:

```c
#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>

int copy_file(const char *src, const char *dst) {
    char buf[4096];
    ssize_t n;

    int in  = open(src, O_RDONLY);
    if (in == -1) { perror("open source"); return -1; }

    int out = open(dst, O_WRONLY | O_CREAT | O_TRUNC, 0644);
    if (out == -1) { perror("open dest"); close(in); return -1; }

    while ((n = read(in, buf, sizeof(buf))) > 0) {
        if (write(out, buf, n) != n) {
            perror("write"); close(in); close(out); return -1;
        }
    }

    if (n == -1) perror("read");

    fsync(out);    // ensure data is on disk before we return
    close(in);
    close(out);
    return 0;
}
```

Key practices shown:
- Check every system call return value
- Close all file descriptors on every error path
- Use a buffer to read/write in chunks (not byte by byte)
- Call `fsync()` before closing to guarantee durability

---

## 10. Real-World Example: How a Text Editor Saves a File

When you hit Save in a text editor, here is what happens at the file handling level:

1. `open("document.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644)` — open/create the file
2. `write(fd, content, length)` — write the document content (may go to OS page cache first)
3. `fsync(fd)` — flush to disk (a crash-safe editor does this before reporting success)
4. `close(fd)` — release the file descriptor

A safer approach used by many editors (vim, emacs, most IDEs):

1. Write to a **temporary file** (`document.txt.tmp`)
2. `fsync()` the temp file
3. `rename("document.txt.tmp", "document.txt")` — atomic on most file systems
4. If a crash happens before step 3, the original file is untouched

`rename()` is atomic — it either fully completes or does nothing. This guarantees you never end up with a half-written save file.

---

## 11. Summary

| Concept | Key Takeaway |
|---------|-------------|
| File descriptor | Integer handle to an open file; 0=stdin, 1=stdout, 2=stderr |
| Open file table | Per-process + kernel tables tracking offset and flags |
| `open()` | Opens a file and returns a file descriptor |
| `read()` / `write()` | Transfer bytes; advance the file offset |
| `lseek()` | Move the file offset for random access |
| `close()` | Release the file descriptor; always do this |
| `fsync()` | Force buffered data to physical disk |
| Buffered I/O | stdio batches writes in user space; fewer system calls |
| `flock()` | Advisory whole-file locking |
| `fcntl()` | POSIX byte-range locking for fine-grained control |
| `dup2()` | Redirect a file descriptor; used by shells for redirection |
| `/proc` | Virtual filesystem exposing live OS internals as files |
| Atomic rename | Safe save pattern — write temp file, rename atomically |

---

## Module 4 Quiz

---

**Question 1**
A process opens the same file twice with two separate `open()` calls. Does each `open()` return a different file descriptor with its own independent offset, or do they share one offset?

> **Answer:** Each `open()` call creates a **separate entry** in the kernel open file table, each with its own independent offset. So the process gets two different file descriptors (e.g., fd 3 and fd 4), and reading through fd 3 does not advance the offset of fd 4. Both entries point to the same inode (same file on disk), but their read/write positions are completely independent.

---

**Question 2**
What is the difference between `write()` returning successfully and the data actually being on disk? How do you guarantee the data is on disk?

> **Answer:** `write()` returns as soon as the data is placed in the OS **page cache** (kernel memory buffer) — it does not wait for the data to be physically written to disk. The OS writes to disk asynchronously in the background. To guarantee the data is on disk, you must call `fsync(fd)` after writing. `fsync()` blocks until the OS confirms the data has been physically written. This is critical for databases, log files, and any write where data loss on crash is unacceptable.

---

**Question 3**
A binary file stores fixed-size employee records of 128 bytes each. Write the `lseek()` call to position the file offset at the start of the 10th record (1-indexed).

> **Answer:**
> ```c
> // Records are 0-indexed, so the 10th record is at index 9
> lseek(fd, 9 * 128, SEEK_SET);
> // = lseek(fd, 1152, SEEK_SET)
> ```
> After this call, a `read(fd, &record, 128)` reads exactly the 10th record.

---

**Question 4**
Explain how `dup2()` is used to implement output redirection in a shell command like `ls > output.txt`.

> **Answer:** The shell:
> 1. Calls `fork()` to create a child process
> 2. In the child: opens `output.txt` with `O_WRONLY | O_CREAT | O_TRUNC` — gets fd 3
> 3. Calls `dup2(3, 1)` — makes stdout (fd 1) point to the same open file table entry as fd 3
> 4. Calls `close(3)` — the original fd 3 is no longer needed
> 5. Calls `exec("ls")` — ls writes to fd 1 (stdout) as normal, but stdout now goes to the file
>
> The `ls` program never knows its output was redirected — it just writes to stdout.

---

**Question 5**
Why do many text editors write to a temporary file and then rename it, instead of writing directly to the target file?

> **Answer:** Writing directly to the target file risks leaving it partially written if the process crashes or the system loses power mid-write — the file is corrupted and the original content is gone. The safe pattern is: write the new content to a temporary file, call `fsync()` to ensure it is on disk, then call `rename()` to atomically replace the original. `rename()` is atomic on most file systems — it either fully completes or does nothing. If a crash occurs before the rename, the original file is completely untouched. If it crashes after, the new file is fully written. There is no window where a half-written file exists as the visible document.

---

## What's Next

**Module 5: File Processing**

You will learn how to process file contents efficiently — parsing structured data, working with text and binary formats, using streams, and handling large files that do not fit in memory.

---

*CS 3307 Operating Systems 2 — Module 4 of 8*
