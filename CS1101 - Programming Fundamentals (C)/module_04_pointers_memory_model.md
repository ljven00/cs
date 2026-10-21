# Module 4 — Pointers and Memory Model

## 1. What Is a Pointer?

Every variable occupies a location in memory. That location has an **address** — a number that
identifies exactly where in RAM the variable lives. A **pointer** is a variable that holds an
address.

This is the concept that makes C unique. Most languages hide addresses from you entirely. C
exposes them, which is why C can talk directly to hardware, implement allocators, and build data
structures that higher-level languages implement behind the scenes.

A pointer type is written as the type it points to, followed by `*`:

```c
uint32_t  value;    /* an ordinary variable            */
uint32_t *ptr;      /* a pointer to a uint32_t         */
```

The `*` is part of the declaration syntax, not the variable name. By convention this course places
it adjacent to the type: `uint32_t *ptr`.

---

## 2. Address-Of and Dereference

Two operators form the core of pointer usage:

| Operator | Name | Meaning |
|----------|------|---------|
| `&x` | Address-of | Produces the address of variable `x` |
| `*p` | Dereference | Accesses the value stored at the address in `p` |

```c
/* encoding: utf-8 */
#include <stdio.h>
#include <stdint.h>
#include <inttypes.h>

int main(void) {
    uint32_t  score  = 42;
    uint32_t *ptr    = &score;   /* ptr holds the address of score */

    printf("Value of score:    %" PRIu32 "\n", score);
    printf("Address of score:  %p\n",          (void *)ptr);
    printf("Value via pointer: %" PRIu32 "\n", *ptr);

    *ptr = 100;   /* write through the pointer */
    printf("score is now:      %" PRIu32 "\n", score);

    return 0;
}
```

Expected output (address will differ on your machine):

```
Value of score:    42
Address of score:  0x7ffd4a2c1e4c
Value via pointer: 42
score is now:      100
```

`*ptr = 100` does not change `ptr` itself — `ptr` still holds the same address. It changes the
value *at* that address, which is `score`.

> Casting to `(void *)` before passing to `%p` is required by the C standard. `%p` expects a
> `void *`; passing any other pointer type without the cast is technically undefined behaviour.

---

## 3. The Memory Model: Stack and Heap

C programs use two distinct regions of memory for runtime data.

### The Stack

The stack is managed automatically by the runtime. When a function is called, a *stack frame* is
pushed containing its local variables and parameters. When the function returns, the frame is
popped and that memory is reclaimed.

Properties:
- Allocation and deallocation are instant (just a pointer increment/decrement).
- Size is limited (typically 1–8 MB on modern systems).
- Lifetime is tied to the function call — a local variable ceases to exist when its function
  returns.

### The Heap

The heap is a large pool of memory managed manually by the programmer via `malloc` and `free`.

Properties:
- Can be much larger than the stack.
- Lifetime is explicit — memory lives until you call `free`.
- Slower to allocate than stack memory.
- Mismanagement (forgetting to free, freeing twice, using after freeing) causes bugs that are
  often hard to track down.

Dynamic memory allocation is the subject of Module 7. This module focuses on pointers to
stack-allocated variables, which is already powerful enough to pass-by-reference and build useful
patterns.

### The dangling pointer

A pointer that holds the address of a variable that no longer exists is called a *dangling pointer*.
Dereferencing it is undefined behaviour.

```c
uint32_t *danger(void) {
    uint32_t local = 5;
    return &local;   /* WRONG: local is destroyed when the function returns */
}
```

The compiler with `-Wall` will warn about this. Never return the address of a local variable.

---

## 4. NULL Pointers

A pointer that does not point to any valid memory location should be set to `NULL`. `NULL` is
defined in `<stddef.h>` (also included transitively by `<stdio.h>` and `<stdint.h>`).

```c
uint32_t *ptr = NULL;
```

Dereferencing a `NULL` pointer is undefined behaviour and almost always causes a segmentation
fault at runtime. Before dereferencing any pointer received from outside your function, check it:

```c
void print_value(uint32_t *ptr) {
    if (ptr == NULL) {
        printf("Error: null pointer received\n");
        return;
    }
    printf("%" PRIu32 "\n", *ptr);
}
```

**Rule:** initialise every pointer to either a valid address or `NULL`. An uninitialised pointer
holds whatever garbage was on the stack — it looks like a valid address and will silently corrupt
memory when dereferenced.

---

## 5. Passing Pointers to Functions (Pass-by-Reference)

Module 3 showed that C passes arguments by value — the function gets a copy. Passing a pointer to
a variable gives the function the address of the original, letting it read and modify it directly.
This is how C simulates pass-by-reference.

```c
/* encoding: utf-8 */
#include <stdio.h>
#include <stdint.h>
#include <inttypes.h>

void swap(uint32_t *a, uint32_t *b) {
    uint32_t temp = *a;
    *a = *b;
    *b = temp;
}

int main(void) {
    uint32_t x = 10;
    uint32_t y = 20;

    printf("Before: x = %" PRIu32 ", y = %" PRIu32 "\n", x, y);
    swap(&x, &y);
    printf("After:  x = %" PRIu32 ", y = %" PRIu32 "\n", x, y);

    return 0;
}
```

Expected output:

```
Before: x = 10, y = 20
After:  x = 20, y = 10
```

Inside `swap`, `a` and `b` are pointer parameters — they hold the *addresses* of `x` and `y`.
`*a` and `*b` dereference those addresses to read and write the original variables. `a` and `b`
themselves are copies (of addresses), but that is enough — the addresses correctly identify the
original memory.

### Output parameters

A common C idiom is to return multiple values from a function by writing results into pointer
parameters rather than returning a struct. These are called *output parameters* and are
conventionally listed last:

```c
void min_max(const uint32_t *data, uint32_t count,
             uint32_t *out_min, uint32_t *out_max);
```

The `const` qualifier on `data` tells the compiler — and the reader — that the function will not
modify the values pointed to by `data`. Use `const` on any pointer parameter that is read-only.

---

## 6. Pointer Arithmetic

Arithmetic on a pointer moves it by multiples of the *size of the type it points to*, not by
bytes. Adding `1` to a `uint32_t *` advances the address by 4 bytes (the size of `uint32_t`);
adding `1` to a `uint8_t *` advances it by 1 byte.

```c
uint32_t  value = 0xDEADBEEF;
uint32_t *p     = &value;

printf("p   = %p\n", (void *)p);
printf("p+1 = %p\n", (void *)(p + 1));
```

On a 64-bit system the two addresses will differ by exactly 4.

Valid pointer arithmetic is limited to pointers within the same allocated object (an array or a
block from `malloc`). Moving a pointer outside its object's bounds and dereferencing it is
undefined behaviour. Pointer arithmetic inside a single-variable allocation — like the example
above — is valid only for `p` itself; `p + 1` may be formed but must not be dereferenced.

The practical application of pointer arithmetic — iterating through arrays — is covered fully in
Module 5.

---

## 7. Pointers to Pointers

A pointer can point to another pointer. This is written with two `*` symbols:

```c
uint32_t  value  = 7;
uint32_t *ptr    = &value;
uint32_t **pptr  = &ptr;

printf("%" PRIu32 "\n", **pptr);   /* dereference twice to reach value */
```

Pointers to pointers appear frequently when a function needs to change what a pointer points to
(rather than the value it points to), and in the construction of dynamic data structures. The most
common use at this stage is the `char **argv` parameter of `main`, which is an array of string
pointers.

---

## 8. Function Pointers

A function pointer holds the address of a function. This enables selecting behaviour at runtime,
passing functions as arguments, and building dispatch tables — patterns used extensively in
operating system design, event-driven programming, and generic algorithms.

### Declaration syntax

```c
return_type (*pointer_name)(parameter_types);
```

The parentheses around `*pointer_name` are required. Without them, `*` would bind to the return
type, which means something different entirely.

```c
/* encoding: utf-8 */
#include <stdio.h>
#include <stdint.h>
#include <inttypes.h>

int32_t add(int32_t a, int32_t b) { return a + b; }
int32_t sub(int32_t a, int32_t b) { return a - b; }
int32_t mul(int32_t a, int32_t b) { return a * b; }

int main(void) {
    int32_t (*op)(int32_t, int32_t);   /* declare a function pointer */

    op = add;
    printf("add: %" PRId32 "\n", op(10, 3));

    op = sub;
    printf("sub: %" PRId32 "\n", op(10, 3));

    op = mul;
    printf("mul: %" PRId32 "\n", op(10, 3));

    return 0;
}
```

Expected output:

```
add: 13
sub: 7
mul: 30
```

A function's name without parentheses is automatically its address. `op = add` and `op = &add` are
equivalent; most C programmers omit the `&`.

### Passing a function pointer as an argument

```c
void apply(int32_t a, int32_t b, int32_t (*fn)(int32_t, int32_t)) {
    printf("Result: %" PRId32 "\n", fn(a, b));
}

apply(6, 7, mul);   /* prints: Result: 42 */
```

### Simplifying with typedef

Function pointer declarations become difficult to read quickly. A `typedef` gives the type a
readable name:

```c
typedef int32_t (*binary_op_t)(int32_t, int32_t);

binary_op_t op = add;
```

Using `typedef` for function pointers is strongly recommended whenever they appear in function
signatures or structs.

---

## 9. Common Mistakes

**Confusing `*` in a declaration with `*` as dereference**  
`uint32_t *p = &x;` — the `*` here means "p is a pointer". `*p = 5;` — the `*` here means
"write to the address in p". They look the same but mean different things.

**Dereferencing an uninitialised pointer**  
`uint32_t *p; *p = 5;` is undefined behaviour. Always initialise to `NULL` or a valid address
before use.

**Returning the address of a local variable**  
The local variable is gone when the function returns. The address is dangling and must not be used.

**Forgetting `&` when passing a variable to an output-parameter function**  
`scanf("%" SCNu32, value)` instead of `scanf("%" SCNu32, &value)` is a very common bug. `scanf`
needs the address, not the value.

**Wrong function pointer signature**  
Calling a function through a pointer of a mismatched type is undefined behaviour. The parameter
types and return type must match exactly.

---

## 10. Practice Exercises

1. Write a function `void increment(uint32_t *n)` that adds 1 to the variable pointed to by `n`.
   Call it five times on the same variable and print the result after each call.

2. Write a function `void clamp_inplace(int32_t *value, int32_t lo, int32_t hi)` that clamps
   `*value` to the range `[lo, hi]` without returning anything. Test it with values below, inside,
   and above the range.

3. Demonstrate the dangling pointer problem: write a function that returns a pointer to a local
   variable, print the compiler warning produced by `-Wall`, and explain in a comment why the
   program's behaviour is undefined.

4. Create a dispatch table: declare an array of four function pointers, each pointing to a
   different arithmetic operation (`add`, `sub`, `mul`, `div_safe`). Write a loop that calls each
   one with the same two operands and prints the result.

5. Write a function `void sort2(uint32_t *a, uint32_t *b)` that ensures `*a <= *b` after it
   returns (swap if necessary). Then write `void sort3(uint32_t *a, uint32_t *b, uint32_t *c)`
   that sorts three values using only calls to `sort2`.

---

## Summary

- A pointer holds the memory address of another variable. Declare with `type *name`; take an
  address with `&x`; dereference with `*p`.
- The stack holds local variables with automatic lifetime; the heap holds dynamically allocated
  memory with explicit lifetime (Module 7).
- Always initialise pointers to `NULL` or a valid address. Never dereference `NULL` or a dangling
  pointer.
- Passing a pointer to a function gives it access to the original variable — C's mechanism for
  pass-by-reference.
- Use `const type *` for pointer parameters that the function should not modify.
- Pointer arithmetic moves by multiples of the pointed-to type's size; stay within the bounds of
  the allocated object.
- Function pointers hold the address of a function and enable runtime dispatch; use `typedef` to
  keep their declarations readable.
