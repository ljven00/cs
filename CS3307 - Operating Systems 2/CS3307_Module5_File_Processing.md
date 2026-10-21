# CS 3307 — Operating Systems 2
## Module 5: File Processing

---

> **Course Progress:** Module 5 of 8 — CS 3307 (OS 2)

---

## 1. What is File Processing?

Module 4 covered how to open, read, write, seek, and close files at the system call level. This module covers what you do **with** the data once you have it — parsing formats, processing text and binary data, handling large files, and working with streams efficiently.

File processing is at the core of almost every real program: compilers read source files, databases read data files, web servers read config files, and log analyzers read gigabytes of text.

---

## 2. Text Files vs. Binary Files

### Text Files

- Contain human-readable characters encoded as bytes (ASCII, UTF-8)
- Lines separated by newline characters (`\n` on Unix, `\r\n` on Windows)
- Can be opened and read in any text editor
- Examples: source code, CSV, JSON, HTML, log files, config files

### Binary Files

- Contain raw bytes — not necessarily printable characters
- Structure is defined by a format, not by line breaks
- Compact and fast to read/write (no parsing overhead)
- Examples: executables, images (JPEG, PNG), compiled data, database files

```
Text file (CSV):           Binary file (same data):
id,name,score              01 00 00 00 41 6C 69 63
1,Alice,95.5               65 00 ... 00 00 00 BF 42
2,Bob,82.0                 02 00 00 00 42 6F 62 00
```

The binary version is smaller and faster to parse — no string-to-number conversion needed.

---

## 3. Processing Text Files

### 3.1 Reading Line by Line (C Standard Library)

```c
#include <stdio.h>

FILE *f = fopen("data.csv", "r");
if (!f) { perror("fopen"); return 1; }

char line[256];
while (fgets(line, sizeof(line), f)) {
    // line contains one line including the \n
    printf("Read: %s", line);
}

fclose(f);
```

- `fgets()` reads one line at a time, up to the buffer size
- Returns `NULL` at end of file or on error
- The `\n` is included in the string — strip it with `line[strcspn(line, "\n")] = 0;`

### 3.2 Parsing CSV

```c
#include <stdio.h>
#include <string.h>

FILE *f = fopen("students.csv", "r");
char line[256];
int id;
char name[50];
float score;

// Skip header line
fgets(line, sizeof(line), f);

while (fgets(line, sizeof(line), f)) {
    // Parse: "1,Alice,95.5"
    sscanf(line, "%d,%49[^,],%f", &id, name, &score);
    printf("ID: %d | Name: %s | Score: %.1f\n", id, name, score);
}

fclose(f);
```

`sscanf()` parses a string using a format — similar to `scanf()` but reads from a string instead of stdin.

### 3.3 Writing Text Files

```c
FILE *f = fopen("output.txt", "w");

fprintf(f, "Student: %s\n", "Alice");
fprintf(f, "Score:   %.1f\n", 95.5);

fclose(f);
```

---

## 4. Processing Binary Files

### 4.1 Writing Binary Records

```c
#include <stdio.h>

typedef struct {
    int   id;
    char  name[50];
    float score;
} Student;

Student s = {1, "Alice", 95.5f};

FILE *f = fopen("students.bin", "wb");   // "wb" = write binary
fwrite(&s, sizeof(Student), 1, f);       // write 1 record
fclose(f);
```

### 4.2 Reading Binary Records

```c
Student s;
FILE *f = fopen("students.bin", "rb");   // "rb" = read binary

while (fread(&s, sizeof(Student), 1, f) == 1) {
    printf("ID: %d | Name: %s | Score: %.1f\n", s.id, s.name, s.score);
}

fclose(f);
```

- `fwrite(ptr, size, count, f)` — writes `count` items of `size` bytes each
- `fread(ptr, size, count, f)` — reads `count` items; returns number actually read
- Returns less than `count` at end of file or on error

### 4.3 Random Access in Binary Files

```c
// Jump to the Nth record (0-indexed) and update it
int n = 2;  // 3rd record

FILE *f = fopen("students.bin", "r+b");  // r+b = read+write binary

fseek(f, n * sizeof(Student), SEEK_SET);
fread(&s, sizeof(Student), 1, f);

s.score = 99.0f;  // update score

fseek(f, n * sizeof(Student), SEEK_SET);  // seek back to overwrite
fwrite(&s, sizeof(Student), 1, f);

fclose(f);
```

This is the foundation of a simple flat-file database.

---

## 5. File Streams and Buffering Modes

The C standard library (`stdio`) buffers I/O for efficiency. There are three buffering modes:

| Mode | Behavior | Default For |
|------|----------|------------|
| **Full buffering** | Flush when buffer is full | Regular files |
| **Line buffering** | Flush on each newline | stdout when connected to terminal |
| **No buffering** | Flush immediately on every write | stderr |

You can control buffering explicitly:

```c
// Set full buffering with a custom 8KB buffer
char buf[8192];
setvbuf(f, buf, _IOFBF, sizeof(buf));

// Set line buffering
setvbuf(f, NULL, _IOLBF, 0);

// Disable buffering entirely
setvbuf(f, NULL, _IONBF, 0);

// Manually flush the buffer at any time
fflush(f);
fflush(stdout);  // force pending output to appear on screen
```

### Why This Matters

```c
printf("Processing...");
// If stdout is fully buffered, this may not appear immediately!
fflush(stdout);   // force it to appear now
sleep(2);
printf(" done.\n");
```

This is a common source of confusion — output appears to be missing or out of order because it is stuck in a buffer.

---

## 6. Processing Large Files

Some files are too large to fit in memory (multi-GB log files, large datasets). You must process them in **chunks**.

### 6.1 Chunk-Based Processing

```c
#define CHUNK 4096

FILE *f = fopen("bigfile.bin", "rb");
char buf[CHUNK];
size_t n;
long total = 0;

while ((n = fread(buf, 1, CHUNK, f)) > 0) {
    // process buf[0..n-1]
    total += n;
}

printf("Total bytes processed: %ld\n", total);
fclose(f);
```

- Read a fixed chunk at a time
- Process the chunk, then read the next
- Memory usage stays constant regardless of file size

### 6.2 Memory-Mapped Large Files

For random access into a large file, `mmap` (covered in Module 3) is often better than `fseek`:

```c
#include <sys/mman.h>
#include <sys/stat.h>
#include <fcntl.h>

int fd = open("bigfile.bin", O_RDONLY);
struct stat sb;
fstat(fd, &sb);

// Map the entire file into virtual memory
char *data = mmap(NULL, sb.st_size, PROT_READ, MAP_PRIVATE, fd, 0);

// Access any byte directly -- OS loads pages on demand
printf("Byte 1000000: %02X\n", (unsigned char)data[1000000]);

munmap(data, sb.st_size);
close(fd);
```

The OS loads only the pages you actually access — you can "random access" a 50 GB file without loading it all into RAM.

---

## 7. Working with Directories in C

The OS provides system calls to read directory contents programmatically.

```c
#include <dirent.h>
#include <stdio.h>

DIR *dir = opendir("/home/alice");
if (!dir) { perror("opendir"); return 1; }

struct dirent *entry;
while ((entry = readdir(dir)) != NULL) {
    // Skip . and ..
    if (strcmp(entry->d_name, ".") == 0 ||
        strcmp(entry->d_name, "..") == 0) continue;

    printf("%s\n", entry->d_name);
}

closedir(dir);
```

### Getting File Metadata with `stat()`

```c
#include <sys/stat.h>

struct stat info;
stat("/home/alice/notes.txt", &info);

printf("Size:     %ld bytes\n",  info.st_size);
printf("Inode:    %ld\n",        info.st_ino);
printf("UID:      %d\n",         info.st_uid);
printf("Modified: %ld\n",        info.st_mtime);
```

This is exactly what `ls -l` uses — it calls `stat()` on each file to get size, permissions, and timestamps from the inode.

---

## 8. File Format Patterns

### 8.1 Header + Data Pattern

Many binary formats start with a **header** describing the structure of the rest of the file.

```c
typedef struct {
    char  magic[4];    // "MYDB" -- identifies the file type
    int   version;     // format version
    int   record_count;
    int   record_size;
} FileHeader;

typedef struct {
    int   id;
    char  name[50];
    float value;
} Record;

// Reading:
FILE *f = fopen("data.mydb", "rb");
FileHeader hdr;
fread(&hdr, sizeof(FileHeader), 1, f);

// Validate magic bytes
if (memcmp(hdr.magic, "MYDB", 4) != 0) {
    fprintf(stderr, "Not a valid MYDB file\n");
    fclose(f); return 1;
}

// Read all records
Record r;
for (int i = 0; i < hdr.record_count; i++) {
    fread(&r, sizeof(Record), 1, f);
    printf("%d: %s = %.2f\n", r.id, r.name, r.value);
}
fclose(f);
```

**Magic bytes** (like `"MYDB"`, `"\x89PNG"`, `"PK"` for ZIP) are the first bytes of a file that identify its format. The `file` command on Linux reads magic bytes to determine file types.

### 8.2 Delimited Text Format (CSV)

```
id,name,score        <- header row
1,Alice,95.5
2,Bob,82.0
3,Carol,91.0
```

Rules:
- Fields separated by a delimiter (comma, tab, pipe)
- First row often contains column names
- Quoted fields can contain the delimiter: `"Smith, John"`

### 8.3 Key-Value Config Format

```ini
# config.ini
[server]
host = localhost
port = 8080
max_connections = 100

[database]
path = /var/db/myapp.db
```

Parsed by reading line by line, splitting on `=`, trimming whitespace.

---

## 9. Error Handling in File Processing

Robust file processing checks every operation:

```c
FILE *f = fopen("data.bin", "rb");
if (!f) {
    perror("fopen");       // prints: "fopen: No such file or directory"
    return EXIT_FAILURE;
}

size_t n = fread(buf, 1, sizeof(buf), f);
if (n < sizeof(buf)) {
    if (feof(f)) {
        // Normal: reached end of file before filling buffer
    } else if (ferror(f)) {
        perror("fread");   // actual I/O error
        fclose(f);
        return EXIT_FAILURE;
    }
}

fclose(f);
```

| Function | Purpose |
|----------|---------|
| `perror(msg)` | Print `msg: error description` using `errno` |
| `feof(f)` | True if end of file was reached |
| `ferror(f)` | True if an I/O error occurred |
| `clearerr(f)` | Reset the error and EOF flags |
| `strerror(errno)` | Return error description as a string |

---

## 10. Real-World Example: Processing a Log File

Server log files can be gigabytes large. This program counts how many lines contain "ERROR":

```c
#include <stdio.h>
#include <string.h>

int main(int argc, char *argv[]) {
    if (argc < 2) {
        fprintf(stderr, "Usage: %s <logfile>\n", argv[0]);
        return 1;
    }

    FILE *f = fopen(argv[1], "r");
    if (!f) { perror("fopen"); return 1; }

    char line[1024];
    long error_count = 0;
    long line_count  = 0;

    while (fgets(line, sizeof(line), f)) {
        line_count++;
        if (strstr(line, "ERROR")) {
            error_count++;
        }
    }

    if (ferror(f)) perror("fread");

    printf("Lines:  %ld\n", line_count);
    printf("Errors: %ld\n", error_count);

    fclose(f);
    return 0;
}
```

This processes the file line by line — only one line is in memory at a time, so it works on a 100 GB log file just as well as a 1 KB one.

---

## 11. Summary

| Concept | Key Takeaway |
|---------|-------------|
| Text vs. binary files | Text is human-readable; binary is compact and faster to parse |
| `fgets()` | Read one line at a time from a text file |
| `sscanf()` | Parse a formatted string (like a CSV line) |
| `fread()` / `fwrite()` | Read/write binary records of fixed size |
| `fseek()` | Random access in files via stdio |
| Buffering modes | Full, line, none — control when data is flushed |
| `fflush()` | Manually flush the stdio buffer |
| Chunk processing | Handle large files without loading them into memory |
| `mmap` for large files | Random access to huge files via virtual memory |
| `opendir()` / `readdir()` | Enumerate directory contents programmatically |
| `stat()` | Get file metadata (size, inode, timestamps, permissions) |
| Magic bytes | First bytes that identify a binary file's format |
| `feof()` / `ferror()` | Distinguish normal end-of-file from actual I/O errors |

---

## Module 5 Quiz

---

**Question 1**
What is the difference between `fread()` and `read()`? When would you use each?

> **Answer:** `fread()` is part of the C standard library (stdio) — it uses a user-space buffer, batching I/O into fewer system calls automatically. It works with `FILE *` handles and is portable across platforms. `read()` is a raw POSIX system call — every call goes directly to the kernel with no buffering. Use `fread()` for most file processing (simpler, faster due to buffering). Use `read()` when you need direct control — sockets, pipes, devices, or when you need to interleave with other system calls like `select()` or `poll()`.

---

**Question 2**
A program writes progress messages with `printf()` but nothing appears on screen until the program exits. What is the likely cause and how do you fix it?

> **Answer:** The likely cause is **full buffering**. When stdout is redirected to a file or pipe, the C library switches from line buffering to full buffering — output is only flushed when the buffer fills up or the program exits. Fix: call `fflush(stdout)` after each progress message, or use `setvbuf(stdout, NULL, _IOLBF, 0)` at the start of the program to switch stdout to line buffering, so each `\n` triggers a flush.

---

**Question 3**
A binary file stores 200 fixed-size records of 64 bytes each. Write the `fseek()` call to position the stream at the start of the 50th record (1-indexed), then read it.

> **Answer:**
> ```c
> typedef struct { /* 64 bytes total */ } Record;
> Record r;
>
> FILE *f = fopen("data.bin", "rb");
>
> // 50th record is at index 49 (0-indexed)
> fseek(f, 49 * sizeof(Record), SEEK_SET);
> // = fseek(f, 49 * 64, SEEK_SET) = fseek(f, 3136, SEEK_SET)
>
> fread(&r, sizeof(Record), 1, f);
> fclose(f);
> ```

---

**Question 4**
What are magic bytes and why are they useful in binary file formats?

> **Answer:** Magic bytes are a fixed sequence of bytes at the very beginning of a binary file that identify its format. For example, PNG files start with `\x89PNG`, ZIP files start with `PK`, and ELF executables start with `\x7fELF`. They are useful because: (1) the OS and tools can identify a file's type without relying on its extension; (2) a program can validate that a file is the correct format before processing it, avoiding crashes or corruption from reading the wrong format; (3) they survive renaming. The Linux `file` command reads magic bytes to identify file types.

---

**Question 5**
You need to find all lines containing the word "TIMEOUT" in a 50 GB server log file. Describe the correct approach and explain why loading the whole file into memory first would be wrong.

> **Answer:** The correct approach is **chunk/line-based streaming**:
> ```c
> FILE *f = fopen("server.log", "r");
> char line[2048];
> while (fgets(line, sizeof(line), f)) {
>     if (strstr(line, "TIMEOUT")) {
>         printf("%s", line);
>     }
> }
> fclose(f);
> ```
> Only one line is in memory at a time — memory usage is constant (a few KB) regardless of file size. Loading the entire 50 GB file first would require 50 GB of RAM, which most machines don't have, and would cause the OS to start swapping to disk — making the process slower than streaming and potentially crashing the program or the system.

---

## What's Next

**Module 6: Command Shells and Control**

You will go deeper into shell scripting — control flow, functions, signal handling, process control, pipes, and writing production-quality scripts that interact with the OS.

---

*CS 3307 Operating Systems 2 — Module 5 of 8*
