# Module 1 — Introduction and the C Environment

## 1. Why C?

C was created in the early 1970s by Dennis Ritchie at Bell Labs, primarily to rewrite the Unix
operating system. What made it revolutionary was a simple idea: a language close enough to the
hardware to be fast and predictable, yet expressive enough to write large programs without losing
your mind.

Decades later, that tradeoff still matters. Operating system kernels, embedded firmware, database
engines, and language runtimes are routinely written in C. Learning C teaches you things that
higher-level languages deliberately hide: how memory is laid out, what a pointer actually is, why
integer overflow is dangerous, and what really happens when your program runs.

Python removes those concerns so you can move fast. C forces you to confront them — and that
confrontation is exactly the point of this course.

---

## 2. The Compilation Pipeline

When you write Python, the interpreter reads and executes your source file directly. C does not
work that way. Your source code must be *translated* into machine code before it can run. This
translation happens in four stages:

```
source.c → [Preprocessor] → [Compiler] → [Assembler] → [Linker] → executable
```

**Preprocessor** — Handles lines that start with `#`. It performs textual substitutions: expanding
macros, inserting the contents of header files (`#include`), and conditionally including or
excluding blocks of code. The output is still C source, just with all shortcuts resolved.

**Compiler** — Translates the preprocessed C source into assembly language specific to your CPU
architecture.

**Assembler** — Converts assembly into *object code*: binary machine instructions, but not yet a
runnable program.

**Linker** — Combines one or more object files with library code (such as the C standard library)
to produce a final executable. References to functions defined in other files get resolved here.

In practice you rarely invoke these stages separately. The `gcc` command runs the entire pipeline
for you.

---

## 3. The GCC Toolchain

GCC (GNU Compiler Collection) is the standard compiler used in this course.

### Compiling a file

```bash
gcc -std=c11 -Wall -Wextra -o program source.c
```

| Flag | Meaning |
|------|---------|
| `-std=c11` | Use the C11 standard |
| `-Wall` | Enable common warnings |
| `-Wextra` | Enable additional warnings |
| `-o program` | Name the output executable `program` |

Run the result:

```bash
./program
```

### Inspecting intermediate stages

These are not needed day-to-day, but useful when you want to understand what the compiler is doing
under the hood:

```bash
gcc -E source.c   # Stop after preprocessing — prints expanded source to stdout
gcc -S source.c   # Stop after compiling — produces source.s (assembly)
gcc -c source.c   # Stop after assembling — produces source.o (object file)
```

---

## 4. Source File Encoding

All source files in this course are saved as **UTF-8**. UTF-8 is the dominant encoding on the web
and on modern Unix systems, and it is compatible with ASCII for the basic characters used in C
syntax.

Practical rules:

- Your editor must save `.c` files as UTF-8 (this is the default in VS Code, Vim, and most modern
  editors).
- UTF-8 encoded string literals work correctly with `printf` on any UTF-8 terminal.
- C identifiers (variable and function names) must still use ASCII characters — UTF-8 is for string
  content, not for naming things in code.

---

## 5. Fixed-Width Integer Types

C's built-in types like `int` and `long` have sizes that vary by platform. A `long` is 32 bits on
Windows and 64 bits on Linux. This ambiguity causes subtle bugs when code is moved between systems.

The header `<stdint.h>` (part of C99 and later) solves this by providing types whose sizes are
guaranteed:

| Type | Width | Range |
|------|-------|-------|
| `uint8_t` | 8 bits, unsigned | 0 to 255 |
| `int8_t` | 8 bits, signed | −128 to 127 |
| `uint16_t` | 16 bits, unsigned | 0 to 65 535 |
| `int16_t` | 16 bits, signed | −32 768 to 32 767 |
| `uint32_t` | 32 bits, unsigned | 0 to 4 294 967 295 |
| `int32_t` | 32 bits, signed | −2 147 483 648 to 2 147 483 647 |
| `uint64_t` | 64 bits, unsigned | 0 to 18 446 744 073 709 551 615 |
| `int64_t` | 64 bits, signed | −9 223 372 036 854 775 808 to 9 223 372 036 854 775 807 |

**Rule of thumb used in this course:** always reach for a fixed-width type first. Use `uint32_t`
for non-negative counts and sizes that fit in 32 bits, `int32_t` for signed arithmetic, and
`uint8_t` when working with raw bytes or characters.

### printf format specifiers for fixed-width types

Because the underlying type of `uint32_t` can differ between compilers, `<inttypes.h>` provides
portable format macros:

| Type | Format macro | Expands to (typically) |
|------|-------------|------------------------|
| `uint32_t` | `PRIu32` | `"u"` |
| `int32_t` | `PRId32` | `"d"` |
| `uint64_t` | `PRIu64` | `"lu"` or `"llu"` |
| `int64_t` | `PRId64` | `"ld"` or `"lld"` |

Usage:

```c
printf("value: %" PRIu32 "\n", some_uint32);
```

The `%` is outside the macro, and the macro string is concatenated with the surrounding literal by
the compiler — this is standard C string literal concatenation.

---

## 6. Your First C Program

```c
/* encoding: utf-8 */
#include <stdio.h>
#include <stdint.h>
#include <inttypes.h>

int main(void) {
    uint32_t year = 1972;
    printf("C foi criado em %" PRIu32 "\n", year);
    return 0;
}
```

> The string literal above contains Portuguese and is valid because the file is saved as UTF-8 and
> the terminal is configured to display UTF-8. The C compiler treats string literals as raw byte
> sequences and passes them through unchanged.

Compile and run:

```bash
gcc -std=c11 -Wall -Wextra -o first first.c
./first
```

Expected output:

```
C foi criado em 1972
```

### Line-by-line breakdown

**`/* encoding: utf-8 */`**  
A block comment. Comments are ignored by the compiler. Starting each file with an encoding note is
a good habit — it tells editors and readers what encoding to expect.

**`#include <stdio.h>`**  
Inserts the standard I/O header, which declares `printf`.

**`#include <stdint.h>`**  
Declares the fixed-width integer types (`uint32_t`, etc.).

**`#include <inttypes.h>`**  
Declares the `PRI*` format macros. It also includes `<stdint.h>`, but being explicit about both is
cleaner.

**`int main(void)`**  
Every C program begins execution at `main`. The `int` return type is the *exit status* sent back to
the operating system. `void` means the function takes no arguments. A non-zero return value
conventionally signals an error.

**`uint32_t year = 1972;`**  
Declares a variable named `year` of type `uint32_t`, initialized to `1972`. Note the semicolon —
every statement in C ends with one.

**`printf("C foi criado em %" PRIu32 "\n", year);`**  
Prints formatted output. `\n` is the newline escape sequence.

**`return 0;`**  
Returns exit status 0 (success) to the operating system.

---

## 7. Structure of a C Source File

A well-organised C file follows this order:

```
1. File-level comment (encoding, author, description)
2. #include directives
3. #define constants and macros
4. Type definitions
5. Function declarations (prototypes)
6. Function definitions
7. main() — entry point
```

This order is a convention, not a language requirement, but following it makes code easier to read
and avoids common issues with declaration order.

---

## 8. Common Beginner Mistakes

**Forgetting the semicolon**  
Every statement ends with `;`. Missing one produces a compiler error on the *following* line, which
can be confusing at first.

**Using `int` instead of a fixed-width type**  
`int` is at least 16 bits but often 32. Its exact size is unspecified. Prefer `int32_t` or
`uint32_t` when the size matters.

**Wrong format specifier**  
Using `%d` with a `uint32_t` is technically undefined behaviour. Always match the type to its
correct `PRI*` macro or specifier.

**No `return` in `main`**  
In C99 and later, falling off the end of `main` implicitly returns 0, but being explicit with
`return 0;` is clearer and good practice.

---

## 9. Practice Exercises

1. Write a program that declares variables of types `uint8_t`, `uint16_t`, `uint32_t`, and
   `uint64_t`, assigns each a value of your choice, and prints all four on separate lines using the
   correct `PRI*` format macros.

2. Modify the first program to print a greeting that includes a non-ASCII character (for example,
   an accented letter or a character from your native language). Confirm the output displays
   correctly in your terminal.

3. Run `gcc -E first.c` and examine the preprocessor output. Find where `uint32_t` is ultimately
   defined and note which system header file it comes from.

4. Intentionally introduce three separate compiler errors one at a time (wrong type, missing
   semicolon, undeclared variable), read the error messages, then fix each one. Learning to read
   compiler errors early saves significant time later.

---

## Summary

- C source code goes through four stages before running: preprocessor → compiler → assembler →
  linker.
- `gcc -std=c11 -Wall -Wextra -o program source.c` is the standard compile command for this course.
- All source files are saved as **UTF-8**; C identifiers must still be ASCII.
- Always use fixed-width types from `<stdint.h>` (`uint32_t`, `int32_t`, etc.) instead of
  platform-dependent types like `int` or `long`.
- Use `<inttypes.h>` format macros (`PRIu32`, `PRId64`, etc.) to print fixed-width types correctly.
- Every C program starts at `main`, which returns an integer exit status to the operating system.
