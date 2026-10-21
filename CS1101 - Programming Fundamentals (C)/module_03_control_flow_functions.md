# Module 3 — Control Flow and Functions

## 1. Conditionals

### if / else

The most basic branching construct. The condition is any expression that evaluates to an integer —
zero means false, any non-zero value means true.

```c
/* encoding: utf-8 */
#include <stdio.h>
#include <stdint.h>
#include <inttypes.h>

int main(void) {
    int32_t score = 74;

    if (score >= 90) {
        printf("Grade: A\n");
    } else if (score >= 75) {
        printf("Grade: B\n");
    } else {
        printf("Grade: C or below\n");
    }

    return 0;
}
```

Expected output:

```
Grade: C or below
```

Braces around a single statement are technically optional, but omitting them is a common source of
bugs. Always include them.

### switch

`switch` compares a single integer expression against a list of constant values. It is cleaner than
a long `if / else if` chain when testing one variable against many known values.

```c
uint8_t day = 3;

switch (day) {
    case 1:  printf("Monday\n");    break;
    case 2:  printf("Tuesday\n");   break;
    case 3:  printf("Wednesday\n"); break;
    default: printf("Other\n");     break;
}
```

Two rules to remember:

**Always include `break`.** Without it, execution *falls through* to the next case. This is
occasionally useful but almost always a bug when unintentional.

**Always include `default`.** It handles any value not matched by a `case` and makes the intent
explicit.

---

## 2. Loops

### for

Use `for` when the number of iterations is known or driven by an index.

```c
for (uint32_t i = 0; i < 5; i++) {
    printf("i = %" PRIu32 "\n", i);
}
```

The three parts of the header — initialisation, condition, increment — are each optional and can
be left blank, but the semicolons are required.

### while

Use `while` when the loop should continue as long as a condition holds, and the number of
iterations is not known in advance.

```c
uint32_t n = 1;
while (n < 100) {
    n *= 2;
}
printf("First power of 2 >= 100: %" PRIu32 "\n", n);
```

### do / while

Identical to `while`, except the body executes at least once because the condition is checked
*after* each iteration.

```c
uint8_t choice;
do {
    printf("Enter 1 or 2: ");
    scanf("%" SCNu8, &choice);
} while (choice != 1 && choice != 2);
```

### break and continue

`break` exits the nearest enclosing loop or `switch` immediately.

`continue` skips the rest of the current loop body and jumps to the next iteration.

Both should be used sparingly — overusing them makes control flow hard to follow.

---

## 3. Functions

### Declaration vs. definition

A **declaration** (also called a *prototype*) tells the compiler the function's name, return type,
and parameter types. It ends with a semicolon.

A **definition** provides the actual body of the function.

```c
/* Declaration — placed near the top of the file or in a header */
uint32_t add(uint32_t a, uint32_t b);

/* Definition — placed below main, or in a separate .c file */
uint32_t add(uint32_t a, uint32_t b) {
    return a + b;
}
```

C compiles top-to-bottom. If you call a function before its definition appears in the file, you
must have declared it first. Forgetting the declaration causes a compiler warning and, under strict
standards, an error.

### Return types

Every function must declare a return type. Use `void` when the function produces no value.

```c
void print_separator(void) {
    printf("---\n");
}
```

The `return` statement immediately exits the function and optionally sends a value back to the
caller. A `void` function may use `return;` (with no value) to exit early.

### Pass-by-value

C passes all function arguments *by value*: the function receives a copy of each argument.
Modifying a parameter inside the function has no effect on the variable in the caller.

```c
/* encoding: utf-8 */
#include <stdio.h>
#include <stdint.h>
#include <inttypes.h>

void double_it(uint32_t x) {
    x *= 2;   /* modifies the local copy only */
}

int main(void) {
    uint32_t value = 10;
    double_it(value);
    printf("value is still %" PRIu32 "\n", value);   /* prints 10 */
    return 0;
}
```

Expected output:

```
value is still 10
```

To modify a variable in the caller, you must pass a *pointer* to it. That is covered in Module 4.

### Function structure convention

Following the file structure introduced in Module 1, declare all functions before `main` and define
them after:

```c
/* encoding: utf-8 */
#include <stdio.h>
#include <stdint.h>
#include <inttypes.h>

/* --- Declarations --- */
uint32_t clamp(uint32_t value, uint32_t lo, uint32_t hi);
void     print_result(uint32_t value);

/* --- main --- */
int main(void) {
    print_result(clamp(150, 0, 100));
    return 0;
}

/* --- Definitions --- */
uint32_t clamp(uint32_t value, uint32_t lo, uint32_t hi) {
    if (value < lo) return lo;
    if (value > hi) return hi;
    return value;
}

void print_result(uint32_t value) {
    printf("Clamped value: %" PRIu32 "\n", value);
}
```

Expected output:

```
Clamped value: 100
```

---

## 4. Scope and Lifetime

**Scope** is the region of source code where a name is visible. **Lifetime** is the period during
program execution when the variable's storage exists.

### Block scope (local variables)

A variable declared inside a `{...}` block is visible only within that block. Its lifetime begins
when the declaration is reached and ends when the block exits — the memory on the stack is
reclaimed automatically.

```c
{
    uint32_t x = 5;
    printf("%" PRIu32 "\n", x);   /* valid here */
}
/* x does not exist here — using it would be a compile error */
```

### Function scope

Parameters and variables declared inside a function are local to that function. Two functions can
each have a variable named `i` without conflict.

### File scope (global variables)

A variable declared outside all functions has file scope — it is visible from its declaration to
the end of the file. Its lifetime is the entire duration of the program.

Global variables are initialised to zero automatically, unlike local variables.

```c
static uint32_t call_count = 0;   /* file-scope, zero-initialised */

void record_call(void) {
    call_count++;
}
```

The `static` keyword here limits visibility to this translation unit (this `.c` file), which is
good practice for globals that are not meant to be shared across files.

### Shadowing

A local variable with the same name as an outer-scope variable *shadows* it — the outer variable
becomes inaccessible within the inner block. This is legal but confusing; avoid it.

---

## 5. Recursion

A function is recursive when it calls itself. Every recursive function needs two things:

1. **A base case** — a condition under which the function returns without making another recursive
   call.
2. **A recursive case** — a call to itself with arguments that move toward the base case.

Without a reachable base case, the function calls itself indefinitely until the stack overflows.

```c
/* encoding: utf-8 */
#include <stdio.h>
#include <stdint.h>
#include <inttypes.h>

uint32_t factorial(uint32_t n) {
    if (n == 0) return 1;          /* base case */
    return n * factorial(n - 1);   /* recursive case */
}

int main(void) {
    for (uint32_t i = 0; i <= 10; i++) {
        printf("%" PRIu32 "! = %" PRIu32 "\n", i, factorial(i));
    }
    return 0;
}
```

Expected output:

```
0! = 1
1! = 1
2! = 2
3! = 6
4! = 24
5! = 120
6! = 720
7! = 5040
8! = 40320
9! = 362880
10! = 3628800
```

### The call stack

Each call to a function creates a *stack frame* — a region of memory that holds the function's
local variables, parameters, and the return address. Recursive calls stack these frames on top of
each other. When the base case is reached, frames are unwound one by one as each call returns.

```
factorial(3)
  └── factorial(2)
        └── factorial(1)
              └── factorial(0)  ← base case, returns 1
              returns 1 * 1 = 1
        returns 2 * 1 = 2
  returns 3 * 2 = 6
```

### When to use recursion

Recursion is natural for problems that have a recursive structure: tree traversal, divide-and-
conquer algorithms, and parsing. For simple counting loops, iteration is faster and uses less
stack space.

C does not guarantee *tail-call optimisation* (the compiler reusing the current stack frame for a
tail-recursive call), so deep recursion on large inputs risks a stack overflow. Know the maximum
depth before choosing recursion over iteration.

---

## 6. Common Mistakes

**Missing `break` in `switch`**  
Fall-through is silent. The compiler will not warn unless you use `-Wimplicit-fallthrough` (add it
to your flags).

**Off-by-one in `for` loops**  
`for (i = 0; i <= n; i++)` iterates `n + 1` times. `for (i = 0; i < n; i++)` iterates exactly
`n` times. The `<` form is almost always correct when indexing from zero.

**Expecting pass-by-value to modify the caller's variable**  
If a function needs to change a variable in the caller, you must pass a pointer. This is one of
the first genuine surprises coming from Python.

**No base case in recursion**  
If the base case is wrong or unreachable, the program will crash with a stack overflow at runtime,
not at compile time.

**Declaring variables mid-block in pre-C99 code**  
C89 requires all variable declarations at the top of a block. With `-std=c11` this is not a
concern, but it explains error messages you may encounter in older codebases.

---

## 7. Practice Exercises

1. Write a function `uint32_t is_prime(uint32_t n)` that returns `1` if `n` is prime and `0`
   otherwise. Use it in `main` to print all prime numbers up to 100.

2. Write an iterative function and a recursive function that both compute the nth Fibonacci number
   (using `uint64_t` to avoid overflow). Verify they produce the same results for n = 0 through 20.

3. Rewrite the `clamp` example from section 3 so that `print_result` accepts a string label as
   well as the value, and prints `"Label: value"`. Use `const char *` for the label parameter
   (type details are covered in Module 5; for now just copy the signature).

4. Write a program using `switch` that maps a numeric month (1–12) to its name and prints the
   number of days in that month. Handle February as always having 28 days for simplicity.

5. Trace through the recursive call stack for `factorial(5)` by hand, writing out each frame and
   its local value of `n`. Then add a `printf` inside `factorial` to verify your trace.

---

## Summary

- `if / else if / else` handles branching; always include braces around the body.
- `switch` is cleaner than long `if / else if` chains; always include `break` and `default`.
- Use `for` when the iteration count is known, `while` for condition-driven loops, and `do/while`
  when at least one iteration is always required.
- Declare functions before use (prototype) and define them after `main`.
- C passes all arguments by value — functions receive copies and cannot modify the caller's
  variables without pointers.
- Local variables have block scope and automatic lifetime; file-scope globals are zero-initialised
  and persist for the program's lifetime.
- Recursive functions require a base case and a recursive case that converges toward it; each call
  consumes stack space, so deep recursion requires care.
