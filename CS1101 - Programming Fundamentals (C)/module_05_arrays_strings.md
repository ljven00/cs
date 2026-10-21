# Module 5 — Arrays and Strings

## 1. Arrays

An array is a contiguous block of memory holding a fixed number of elements of the same type.
All elements are laid out one after another with no gaps, which makes indexed access fast and
predictable.

### Declaration and initialisation

```c
uint32_t scores[5];                        /* unintialised — do not read before writing   */
uint32_t primes[5] = {2, 3, 5, 7, 11};    /* fully initialised                           */
uint32_t zeros[8]  = {0};                  /* first element 0, rest zero-initialised      */
uint32_t auto_sz[] = {10, 20, 30};         /* compiler infers size: 3                     */
```

Indices run from `0` to `length - 1`. Accessing outside this range is undefined behaviour — C
does not perform bounds checking at runtime.

### Iterating over an array

```c
/* encoding: utf-8 */
#include <stdio.h>
#include <stdint.h>
#include <inttypes.h>

int main(void) {
    uint32_t primes[] = {2, 3, 5, 7, 11};
    uint32_t count    = sizeof(primes) / sizeof(primes[0]);

    for (uint32_t i = 0; i < count; i++) {
        printf("primes[%" PRIu32 "] = %" PRIu32 "\n", i, primes[i]);
    }
    return 0;
}
```

Expected output:

```
primes[0] = 2
primes[1] = 3
primes[2] = 5
primes[3] = 7
primes[4] = 11
```

`sizeof(primes) / sizeof(primes[0])` is the standard C idiom for computing the number of elements
in a stack-allocated array. `sizeof` returns the total byte size of the array; dividing by the
size of one element gives the count. This only works when the array is in scope — once it decays
to a pointer (see section 3), the size information is gone.

---

## 2. 2D Arrays

A two-dimensional array is declared with two size specifiers and stored in *row-major order*:
all elements of row 0 come first in memory, then all of row 1, and so on.

```c
/* encoding: utf-8 */
#include <stdio.h>
#include <stdint.h>
#include <inttypes.h>

int main(void) {
    uint32_t matrix[3][4] = {
        {1,  2,  3,  4},
        {5,  6,  7,  8},
        {9, 10, 11, 12}
    };

    for (uint32_t row = 0; row < 3; row++) {
        for (uint32_t col = 0; col < 4; col++) {
            printf("%3" PRIu32, matrix[row][col]);
        }
        printf("\n");
    }
    return 0;
}
```

Expected output:

```
  1  2  3  4
  5  6  7  8
  9 10 11 12
```

When passing a 2D array to a function, the column count must be specified in the parameter type
because the compiler needs it to calculate row offsets:

```c
void print_matrix(uint32_t m[][4], uint32_t rows);
```

---

## 3. Arrays and Pointers

In most expressions, an array *decays* to a pointer to its first element. This is not a copy —
it is the same memory, accessed through a pointer.

```c
uint32_t data[] = {10, 20, 30};
uint32_t *p     = data;           /* equivalent to &data[0] */

printf("%" PRIu32 "\n", *p);      /* 10 */
printf("%" PRIu32 "\n", *(p+1));  /* 20 */
printf("%" PRIu32 "\n", *(p+2));  /* 30 */
```

`data[i]` and `*(data + i)` are exactly equivalent — the compiler treats them identically.
Pointer arithmetic here is the practical application deferred from Module 4: adding `1` to a
`uint32_t *` advances by 4 bytes, landing on the next element.

### Passing arrays to functions

Because arrays decay to pointers, a function receiving an array receives a pointer to its first
element, not a copy of the array. The function can read and modify the original data. The size
must be passed separately — the function has no way to recover it from the pointer alone.

```c
uint32_t sum(const uint32_t *data, uint32_t count) {
    uint32_t total = 0;
    for (uint32_t i = 0; i < count; i++) {
        total += data[i];
    }
    return total;
}
```

Using `const uint32_t *` signals that the function will not modify the array contents. Apply
`const` to any pointer parameter whose data the function only reads.

---

## 4. Strings in C

C has no built-in string type. A string is a **null-terminated array of `char`**: a sequence of
bytes followed by a byte with value `0` (written `'\0'`). This terminator is what allows string
functions to know where the string ends.

```
"hola"  →  ['h', 'o', 'l', 'a', '\0']   — 5 bytes total
```

A string literal in C is a pointer to a null-terminated array stored in read-only memory. You
must not modify it.

### Declaring strings

```c
/* String literal — read-only, do not modify */
const char *greeting = "¡Hola, mundo!";

/* Mutable character array — writable copy */
char buffer[64] = "¡Hola, mundo!";

/* Empty buffer for building strings */
char output[128];
```

`char` is used for string data. When working with raw bytes or ASCII values numerically, use
`uint8_t`; when working with text, use `char` so that the standard string functions apply.

### UTF-8 and char arrays

UTF-8 encodes non-ASCII characters as sequences of two to four bytes. The `char` array holds
those bytes unchanged; `strlen` counts bytes, not Unicode code points. On a UTF-8 terminal this
is transparent for display, but be aware that `strlen("año")` returns 4, not 3 — the `ñ`
occupies two bytes (`\xC3\xB1`).

```c
/* encoding: utf-8 */
#include <stdio.h>
#include <string.h>

int main(void) {
    const char *word = "año";
    printf("Texto:  %s\n",   word);
    printf("Bytes:  %zu\n",  strlen(word));   /* 4 */
    return 0;
}
```

Expected output:

```
Texto:  año
Bytes:  4
```

---

## 5. The Standard String Library (`<string.h>`)

Always `#include <string.h>` when using these functions.

### strlen — string length

```c
size_t strlen(const char *s);
```

Returns the number of bytes before the null terminator. Does not count `'\0'` itself.

### strcpy and strncpy — copying

```c
char *strcpy (char *dest, const char *src);
char *strncpy(char *dest, const char *src, size_t n);
```

`strcpy` copies `src` into `dest` including the null terminator. The destination buffer must be
large enough — `strcpy` does not check. Buffer overflows from `strcpy` are a classic source of
security vulnerabilities.

`strncpy` copies at most `n` bytes. If `src` is shorter than `n`, the remainder of `dest` is
zero-filled. If `src` is `n` bytes or longer, **no null terminator is written** — you must add
it manually:

```c
char dest[16];
strncpy(dest, src, sizeof(dest) - 1);
dest[sizeof(dest) - 1] = '\0';   /* guarantee null termination */
```

### strcmp — comparison

```c
int strcmp (const char *a, const char *b);
int strncmp(const char *a, const char *b, size_t n);
```

Returns `0` if the strings are equal, a negative value if `a < b`, and a positive value if
`a > b`, compared lexicographically. **Never compare strings with `==`** — that compares
addresses, not contents.

```c
if (strcmp(input, "salir") == 0) {
    printf("Saliendo...\n");
}
```

### strcat — concatenation

```c
char *strcat (char *dest, const char *src);
char *strncat(char *dest, const char *src, size_t n);
```

Appends `src` to the end of `dest`. The destination must have enough space for the combined
result plus the null terminator. Prefer `strncat` with an explicit limit for safety.

### strchr and strstr — searching

```c
char *strchr(const char *s, int c);
char *strstr(const char *haystack, const char *needle);
```

`strchr` returns a pointer to the first occurrence of character `c` in `s`, or `NULL` if not
found.

`strstr` returns a pointer to the first occurrence of `needle` in `haystack`, or `NULL` if not
found.

```c
/* encoding: utf-8 */
#include <stdio.h>
#include <string.h>

int main(void) {
    const char *sentence = "El precio es bajo";
    const char *found    = strstr(sentence, "precio");

    if (found != NULL) {
        printf("Encontrado en posición: %td\n", found - sentence);
    }
    return 0;
}
```

Expected output:

```
Encontrado en posición: 3
```

The subtraction of two pointers within the same array yields a `ptrdiff_t` (a signed integer
type), formatted with `%td`.

### strtok — tokenisation

```c
char *strtok(char *str, const char *delimiters);
```

`strtok` splits a string into tokens separated by any character in `delimiters`. It **modifies
the input string** by replacing delimiter characters with `'\0'`. Call it with the string on the
first call and `NULL` on subsequent calls to continue tokenising the same string.

```c
/* encoding: utf-8 */
#include <stdio.h>
#include <string.h>

int main(void) {
    char csv[] = "manzana,naranja,uva,pera";
    char *token = strtok(csv, ",");

    while (token != NULL) {
        printf("Fruta: %s\n", token);
        token = strtok(NULL, ",");
    }
    return 0;
}
```

Expected output:

```
Fruta: manzana
Fruta: naranja
Fruta: uva
Fruta: pera
```

Important caveats: `strtok` is not re-entrant (it uses internal static state), so it cannot be
used in two nested loops or across threads. For those cases, use `strtok_r` (POSIX) instead.

---

## 6. sprintf and snprintf — Building Strings

`sprintf` and `snprintf` write formatted output into a character buffer instead of to the
terminal. Always prefer `snprintf`, which accepts a maximum byte count and prevents buffer
overflows:

```c
char mensaje[64];
snprintf(mensaje, sizeof(mensaje), "Puntuación: %" PRIu32, score);
```

`snprintf` always null-terminates the result as long as the buffer size is at least 1.

---

## 7. Common Mistakes

**Out-of-bounds access**  
C does not check array bounds. Writing past the end of an array silently corrupts adjacent memory.
Use `sizeof` / element size to compute the count and always keep indices within `[0, count - 1]`.

**Losing array size after decay**  
Once an array decays to a pointer, `sizeof` on that pointer gives the pointer size (8 bytes on a
64-bit system), not the array size. Always pass the count as a separate parameter.

**Forgetting the null terminator**  
`char buf[5] = "hello"` has no room for `'\0'` — the array holds exactly `{'h','e','l','l','o'}`.
Always allocate `strlen(s) + 1` bytes, or use a size large enough to include `'\0'`.

**Modifying a string literal**  
`char *s = "hola"; s[0] = 'H';` is undefined behaviour. String literals are read-only. Use a
`char` array initialised from a literal when mutation is needed.

**Using `==` to compare strings**  
`if (a == b)` compares two addresses, not the string contents. Always use `strcmp`.

**strtok modifying the input**  
If the original string must be preserved, work on a copy.

---

## 8. Practice Exercises

1. Write a function `uint32_t count_char(const char *s, char c)` that counts how many times
   character `c` appears in string `s`. Test it with a Spanish sentence containing repeated
   letters.

2. Write a function `void reverse_array(uint32_t *data, uint32_t count)` that reverses an array
   in place using pointer arithmetic (no index variables allowed — use only `*p` and `*(q--)`
   style access).

3. Write `void str_toupper(char *s)` that converts all lowercase ASCII letters in `s` to
   uppercase in place. Do not use any library function other than `strlen`.

4. Using `strstr` and pointer arithmetic, write a function that counts how many times a substring
   appears in a larger string. Test with `"banana"` and the substring `"an"`.

5. Write a program that reads a comma-separated list of `uint32_t` values from a string (e.g.
   `"10,200,33,4"`), parses them using `strtok` and `strtoul`, stores them in an array, and prints
   the minimum, maximum, and sum.

6. Declare a `3×3` identity matrix as a 2D `uint32_t` array and write a function that prints it
   in grid form. Then write a second function that multiplies it by a scalar value passed as a
   parameter.

---

## Summary

- Arrays are contiguous blocks of fixed size; indices run from `0` to `count - 1`; C performs no
  bounds checking.
- Use `sizeof(arr) / sizeof(arr[0])` to get the element count of a stack-allocated array; this
  idiom breaks once the array decays to a pointer.
- An array decays to a pointer to its first element in most expressions; `data[i]` and
  `*(data + i)` are identical.
- Always pass the array length as a separate parameter to functions, and mark read-only pointer
  parameters with `const`.
- A C string is a null-terminated `char` array; `strlen` counts bytes, not Unicode code points —
  multi-byte UTF-8 characters occupy more than one byte.
- Never compare strings with `==`; always use `strcmp`.
- Prefer `strncpy`, `strncat`, and `snprintf` over their unbounded counterparts to avoid buffer
  overflows.
- `strtok` modifies its input and maintains internal state; use a copy of the string and `strtok_r`
  when re-entrancy matters.
