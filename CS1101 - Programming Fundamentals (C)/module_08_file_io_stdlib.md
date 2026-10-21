# Module 8 — File I/O and the Standard Library

## 1. Streams and File Pointers

C represents every open file as a `FILE *` — an opaque pointer to a structure maintained by the
standard library that tracks the file's position, buffering state, and error flags. You never
interact with that structure directly; you pass the `FILE *` to standard library functions.

Three streams are open automatically at program start:

| Stream | `FILE *` constant | Default connection |
|--------|------------------|--------------------|
| Standard input | `stdin` | Keyboard |
| Standard output | `stdout` | Terminal |
| Standard error | `stderr` | Terminal (unbuffered) |

`printf` writes to `stdout`; `fprintf(stderr, ...)` writes to `stderr`. Error messages should
always go to `stderr` so they are visible even when `stdout` is redirected to a file.

---

## 2. Opening and Closing Files

```c
FILE *fopen(const char *path, const char *mode);
int   fclose(FILE *stream);
```

`fopen` returns `NULL` if the file cannot be opened. Always check. `fclose` flushes any buffered
data and releases the file descriptor — always close every file you open.

### Mode strings

| Mode | Meaning |
|------|---------|
| `"r"` | Read. File must exist. |
| `"w"` | Write. Creates or truncates. |
| `"a"` | Append. Creates if absent; writes at end. |
| `"r+"` | Read and write. File must exist. |
| `"w+"` | Read and write. Creates or truncates. |
| `"rb"`, `"wb"`, `"ab"` | Binary variants of the above. |

```c
/* encoding: utf-8 */
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    FILE *f = fopen("notes.txt", "w");
    if (f == NULL) {
        perror("fopen");
        return 1;
    }

    fprintf(f, "Line one\n");
    fprintf(f, "Line two\n");

    fclose(f);
    return 0;
}
```

`perror` prints the string you pass, followed by a colon and the human-readable description of
the current `errno` value — the standard way to report file errors in one line.

---

## 3. Text Mode vs. Binary Mode

In **text mode** (the default), the standard library may translate newline characters. On Windows,
`\n` is translated to `\r\n` on write and back to `\n` on read. On Unix, no translation occurs.
Text mode is appropriate for human-readable files.

In **binary mode** (`"rb"`, `"wb"`), bytes are written and read exactly as specified — no
translation. Use binary mode whenever you are writing structured data, images, serialised structs,
or any content where exact byte values matter.

```c
FILE *f = fopen("data.bin", "wb");
```

If you write a struct to a file in text mode on Windows and read it on Linux, the byte counts will
differ and the data will be corrupted. Use binary mode for portability.

---

## 4. Reading and Writing Text Files

### Character and line I/O

```c
int   fgetc(FILE *stream);              /* read one character, returns EOF on end/error */
char *fgets(char *buf, int n, FILE *f); /* read one line, at most n-1 chars + '\0'      */
int   fputc(int c,    FILE *stream);    /* write one character                           */
int   fputs(const char *s, FILE *f);    /* write a string (no newline added)             */
```

`fgets` is the safe way to read a line — it always null-terminates and never overflows the
buffer. It retains the `\n` in the buffer if the line fit. Avoid `gets` entirely; it was removed
from C11 because it cannot be used safely.

### Formatted I/O

```c
int fprintf(FILE *f, const char *format, ...);
int fscanf (FILE *f, const char *format, ...);
```

These work exactly like `printf` and `scanf` but target a `FILE *` instead of `stdout`/`stdin`.

```c
/* encoding: utf-8 */
#include <stdio.h>
#include <stdint.h>
#include <inttypes.h>

int main(void) {
    /* Write */
    FILE *f = fopen("scores.txt", "w");
    if (f == NULL) { perror("fopen"); return 1; }

    fprintf(f, "Alice %" PRIu32 "\n", (uint32_t)95);
    fprintf(f, "Bob   %" PRIu32 "\n", (uint32_t)82);
    fclose(f);

    /* Read back */
    f = fopen("scores.txt", "r");
    if (f == NULL) { perror("fopen"); return 1; }

    char     name[32];
    uint32_t score;
    while (fscanf(f, "%31s %" SCNu32, name, &score) == 2) {
        printf("Read: %s scored %" PRIu32 "\n", name, score);
    }

    fclose(f);
    return 0;
}
```

Expected output:

```
Read: Alice scored 95
Read: Bob scored 82
```

`fscanf` returns the number of items successfully matched. Checking against the expected count
(here `2`) is the correct way to detect end-of-file or a malformed line.

---

## 5. Binary File I/O

```c
size_t fread (void *buf, size_t size, size_t count, FILE *f);
size_t fwrite(const void *buf, size_t size, size_t count, FILE *f);
```

Both functions transfer `count` elements of `size` bytes each. They return the number of elements
successfully transferred, which may be less than `count` on error or end-of-file.

```c
/* encoding: utf-8 */
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <inttypes.h>

typedef struct {
    uint32_t id;
    uint32_t value;
} record_t;

int main(void) {
    record_t records[] = {
        {.id = 1, .value = 100},
        {.id = 2, .value = 200},
        {.id = 3, .value = 300},
    };
    uint32_t count = sizeof(records) / sizeof(records[0]);

    /* Write */
    FILE *f = fopen("records.bin", "wb");
    if (f == NULL) { perror("fopen"); return 1; }
    fwrite(records, sizeof(record_t), count, f);
    fclose(f);

    /* Read back */
    f = fopen("records.bin", "rb");
    if (f == NULL) { perror("fopen"); return 1; }

    record_t r;
    while (fread(&r, sizeof(record_t), 1, f) == 1) {
        printf("id=%" PRIu32 " value=%" PRIu32 "\n", r.id, r.value);
    }

    fclose(f);
    return 0;
}
```

Expected output:

```
id=1 value=100
id=2 value=200
id=3 value=300
```

Remember the padding and endianness caveats from Module 6: binary files written on one machine
may not be readable on a machine with a different architecture or compiler without explicit
serialisation.

---

## 6. File Positioning

```c
int    fseek(FILE *f, long offset, int origin);
long   ftell(FILE *f);
void   rewind(FILE *f);
```

`fseek` moves the file position to `offset` bytes relative to `origin`:

| Origin constant | Meaning |
|----------------|---------|
| `SEEK_SET` | Beginning of file |
| `SEEK_CUR` | Current position |
| `SEEK_END` | End of file |

`ftell` returns the current position in bytes from the start. `rewind` resets the position to the
beginning and clears the error flag.

A common pattern for reading an entire file into a heap buffer:

```c
fseek(f, 0, SEEK_END);
long size = ftell(f);
rewind(f);

char *buf = malloc((size_t)size + 1);
if (buf == NULL) { fclose(f); return NULL; }
fread(buf, 1, (size_t)size, f);
buf[size] = '\0';
```

---

## 7. Error Handling with errno

`<errno.h>` declares the global variable `errno`, which the standard library sets to a non-zero
error code when a system or library call fails. Crucially, a successful call does not reset
`errno` to zero — always check the return value of a function first, and only read `errno` when
the function signals failure.

```c
#include <errno.h>
#include <string.h>   /* strerror */

FILE *f = fopen("missing.txt", "r");
if (f == NULL) {
    fprintf(stderr, "Error %d: %s\n", errno, strerror(errno));
    return 1;
}
```

`strerror(errno)` returns the human-readable description of the error code. `perror(label)` is a
one-call shortcut that prints `label: description\n` to `stderr`.

Common `errno` values in file I/O:

| Constant | Meaning |
|----------|---------|
| `ENOENT` | No such file or directory |
| `EACCES` | Permission denied |
| `EEXIST` | File already exists |
| `ENOMEM` | Out of memory |
| `ENOSPC` | No space left on device |

---

## 8. Standard Library Tour

The headers below are available on any conforming C11 implementation. This section summarises the
most useful functions in each — ones you will reach for repeatedly.

### `<stdio.h>` — Input / Output

Already covered in depth above. Additional functions worth knowing:

```c
int  fflush(FILE *f);          /* flush buffered output to the OS */
int  feof(FILE *f);            /* non-zero if end-of-file reached */
int  ferror(FILE *f);          /* non-zero if an error occurred   */
int  remove(const char *path); /* delete a file                   */
int  rename(const char *old, const char *new);
```

### `<stdlib.h>` — General Utilities

```c
void  exit(int status);          /* terminate the process             */
void  abort(void);               /* terminate with SIGABRT            */
int   abs(int n);                /* absolute value of int             */
long  strtol (const char *s, char **end, int base);  /* string to long   */
unsigned long strtoul(const char *s, char **end, int base);
double strtod(const char *s, char **end);            /* string to double */
int   system(const char *cmd);   /* run a shell command               */
char *getenv(const char *name);  /* read an environment variable      */
void  qsort(void *base, size_t n, size_t size,
            int (*cmp)(const void *, const void *));
void *bsearch(const void *key, const void *base, size_t n,
              size_t size, int (*cmp)(const void *, const void *));
```

`strtol` / `strtoul` / `strtod` are the correct way to convert strings to numbers — they report
where parsing stopped and do not invoke undefined behaviour on overflow the way `atoi` does.
Prefer them over `atoi`, `atof`, and `atol` in all new code.

### `<string.h>` — String and Memory

Covered in Module 5. Two memory functions not yet introduced:

```c
void *memcpy (void *dst, const void *src, size_t n); /* copy n bytes, no overlap */
void *memmove(void *dst, const void *src, size_t n); /* copy n bytes, overlap OK */
void *memset (void *dst, int c,           size_t n); /* fill n bytes with c      */
int   memcmp (const void *a, const void *b, size_t n);
```

Use `memmove` whenever the source and destination regions might overlap; `memcpy` is only safe
for non-overlapping regions.

### `<ctype.h>` — Character Classification

All functions take an `int` (the character value) and return non-zero for true:

```c
isalpha(c)   /* letter                    */
isdigit(c)   /* decimal digit             */
isalnum(c)   /* letter or digit           */
isspace(c)   /* whitespace                */
isupper(c)   /* uppercase letter          */
islower(c)   /* lowercase letter          */
toupper(c)   /* convert to uppercase      */
tolower(c)   /* convert to lowercase      */
```

These functions are locale-aware and handle edge cases that manual comparisons like `c >= 'a'`
miss. Always use them for character testing instead of raw comparisons.

### `<math.h>` — Mathematics

Link with `-lm` when using `<math.h>` functions:

```bash
gcc -std=c11 -Wall -Wextra -o program source.c -lm
```

```c
double sqrt(double x);
double pow(double base, double exp);
double fabs(double x);      /* absolute value for doubles */
double floor(double x);
double ceil(double x);
double round(double x);
double log(double x);       /* natural logarithm          */
double log2(double x);
double sin(double x);       /* x in radians               */
double cos(double x);
double fmod(double x, double y);  /* floating-point remainder */
```

When working with `float` instead of `double`, use the `f`-suffixed variants: `sqrtf`, `powf`,
`fabsf`, etc.

### `<limits.h>` — Platform Integer Limits

Defines the minimum and maximum values for the platform's native integer types:

```c
CHAR_BIT    /* bits in a char, always 8 on modern systems */
INT_MIN     /* minimum value of int                       */
INT_MAX     /* maximum value of int                       */
UINT_MAX    /* maximum value of unsigned int              */
LONG_MIN / LONG_MAX
ULONG_MAX
```

These complement `<stdint.h>`'s fixed-width limits (`INT32_MIN`, `UINT32_MAX`, etc.). Prefer
the `<stdint.h>` constants in new code; use `<limits.h>` when interfacing with APIs that use
native types like `int` or `long`.

### `<errno.h>` — Error Codes

Covered in section 7. Include this header whenever you need to inspect `errno` or use the `E*`
constants symbolically rather than by number.

### `<time.h>` — Time and Date

```c
time_t  time(time_t *t);              /* current calendar time as seconds since epoch */
clock_t clock(void);                  /* CPU time used by the process                 */
double  difftime(time_t t1, time_t t0);
struct tm *localtime(const time_t *t);
struct tm *gmtime(const time_t *t);
size_t  strftime(char *buf, size_t max,
                 const char *format, const struct tm *tm);
```

`clock()` is the simplest tool for benchmarking short sections of code:

```c
#include <time.h>

clock_t start = clock();
/* ... code to measure ... */
clock_t end   = clock();
double  elapsed = (double)(end - start) / CLOCKS_PER_SEC;
printf("Elapsed: %.6f s\n", elapsed);
```

---

## 9. Common Mistakes

**Not closing files**  
An unclosed `FILE *` leaks a file descriptor and may leave buffered data unwritten. Every
`fopen` must have a matching `fclose`.

**Checking `feof` before reading**  
`feof` returns true only *after* a read has failed due to end-of-file. The correct pattern is to
check the return value of the read function itself (`fgets`, `fread`, `fscanf`) and call `feof`
only to distinguish EOF from an error.

**Writing raw structs to text-mode files**  
In text mode on Windows, a byte with value `0x0A` (`'\n'`) is translated to `0x0D 0x0A`. If a
struct field contains that byte value the file will be corrupted. Always use binary mode for
non-text data.

**Using `atoi` / `atof` for user input**  
These functions have no error reporting and invoke undefined behaviour on overflow. Always use
`strtol`, `strtoul`, or `strtod`.

**Reading `errno` after a successful call**  
A successful call does not reset `errno`. Read it only immediately after a failing call.

**Forgetting `-lm`**  
On Linux, `<math.h>` functions require explicit linking with `-lm`. The linker error
`undefined reference to 'sqrt'` is almost always caused by a missing `-lm`.

---

## 10. Practice Exercises

1. Write a program that reads a text file line by line using `fgets`, counts the number of lines,
   words, and bytes, and prints a summary — a minimal `wc` clone.

2. Write a program that reads a binary file of `record_t` structs (from section 5), sorts them
   by `value` using `qsort`, and writes the sorted records to a new binary file. Read the output
   file back and verify the order.

3. Write a function `char *read_file(const char *path)` that allocates a heap buffer, reads the
   entire file into it, null-terminates it, and returns the pointer. The caller is responsible for
   freeing it. Handle all error cases and verify with Valgrind.

4. Use `<time.h>` to benchmark two sorting algorithms (e.g. bubble sort vs. the `qsort` from
   `<stdlib.h>`) on an array of 100 000 `uint32_t` values. Print the elapsed time for each.

5. Write a program that uses `getenv` to read the `HOME` environment variable and `<time.h>` to
   get the current date. Using `strftime`, write a log entry of the form
   `"[YYYY-MM-DD HH:MM:SS] Session started by /home/user\n"` to a file named `session.log` in
   append mode.

---

## 11. Capstone Project — Student Grade Manager

This project integrates the major concepts from all eight modules: fixed-width types, structs,
dynamic memory, file I/O, string handling, sorting, and error management.

### Specification

Build a command-line program that manages a list of students and their grades. The program must:

**Data model**

Define a `student_t` struct containing:
- `uint32_t id` — unique identifier
- `char name[48]` — UTF-8 student name
- `uint32_t grades[5]` — five subject scores (0–100)
- `float average` — computed from the grades array

**Required operations** (implement each as a separate function)

| Function | Description |
|----------|-------------|
| `student_t *students_load(const char *path, uint32_t *count)` | Read all students from a binary file; allocate and return a heap array; set `*count` |
| `int students_save(const char *path, const student_t *s, uint32_t count)` | Write all students to a binary file; return 0 on success, -1 on error |
| `void students_compute_averages(student_t *s, uint32_t count)` | Fill the `average` field for every student |
| `void students_sort_by_average(student_t *s, uint32_t count)` | Sort descending by `average` using `qsort` |
| `void students_print(const student_t *s, uint32_t count)` | Print a formatted table to `stdout` |
| `student_t *student_find_by_id(student_t *s, uint32_t count, uint32_t id)` | Return a pointer to the matching student or `NULL` |

**Program flow**

1. If the data file exists, load it with `students_load`.
2. If it does not exist (check `errno == ENOENT`), start with a small hard-coded set of students.
3. Compute averages, sort by average, and print the table.
4. Save the current state to the file with `students_save`.
5. Demonstrate `student_find_by_id` by looking up one student and printing their full record.

**Requirements**

- All heap allocations must be checked for `NULL`.
- Every opened file must be closed in all code paths, including error paths.
- The program must exit cleanly with zero errors and zero leaks under
  `valgrind --leak-check=full`.
- Use `fprintf(stderr, ...)` for all error messages.
- All integer fields use fixed-width types; no bare `int` or `long`.

**Expected terminal output (example)**

```
Rank  ID    Name                             Average
----  ----  ------                           -------
   1    3   Sofia Reyes                        91.4
   2    1   James Carter                       87.2
   3    2   Mei Lin                            79.6

Student ID 2: Mei Lin — grades: 80 75 85 72 86 — avg: 79.60
Data saved to students.bin
```

---

## Course Summary

You have now covered the full foundation of C programming:

| Module | Topic |
|--------|-------|
| 1 | Environment, compilation pipeline, GCC, UTF-8, fixed-width types |
| 2 | Variables, data types, arithmetic, bitwise operators, undefined behaviour |
| 3 | Control flow, functions, pass-by-value, scope, recursion |
| 4 | Pointers, memory model, NULL, pass-by-reference, function pointers |
| 5 | Arrays, strings, array-pointer relationship, `<string.h>` |
| 6 | Structs, enums, unions, typedef, linked lists |
| 7 | Dynamic memory: malloc, calloc, realloc, free, Valgrind |
| 8 | File I/O, error handling, standard library tour |

The concepts that separate C from higher-level languages — explicit memory management, direct
pointer manipulation, manual resource ownership — are now part of your toolkit. The capstone
project brings them together into a program that a systems programmer would recognise as
idiomatic C.

Where to go from here: multi-file projects and makefiles, the POSIX API (sockets, processes,
threads), data structures (hash tables, trees, graphs), and ultimately operating systems
internals or embedded systems programming — all built on exactly the foundation covered in this
course.
