# Module 2 — Variables, Data Types, Expressions, and Statements

## 1. Variables and Declarations

A variable is a named region of memory that holds a value of a specific type. In C, every variable
must be declared before it is used, and its type cannot change after declaration.

The general form of a declaration is:

```
type name;
type name = initial_value;
```

C does not zero-initialise local variables automatically. Reading an uninitialised variable is
undefined behaviour — the value could be anything left over in that memory location.

```c
/* encoding: utf-8 */
#include <stdio.h>
#include <stdint.h>
#include <inttypes.h>

int main(void) {
    uint32_t puntos = 0;        /* initialised — safe to read */
    int32_t  temperatura = -5;  /* signed, negative value     */

    printf("Puntos: %" PRIu32 "\n", puntos);
    printf("Temperatura: %" PRId32 " °C\n", temperatura);
    return 0;
}
```

Expected output:

```
Puntos: 0
Temperatura: -5 °C
```

> The `°` character is two bytes in UTF-8 (`\xC2\xB0`). The C compiler passes it through unchanged
> inside the string literal, and a UTF-8 terminal displays it correctly.

---

## 2. Fixed-Width Types in Depth

As established in Module 1, this course uses fixed-width types from `<stdint.h>` exclusively.
Here is how to choose the right one:

| Situation | Preferred type |
|-----------|---------------|
| Non-negative counter or size (≤ 4 billion) | `uint32_t` |
| Non-negative counter or size (> 4 billion) | `uint64_t` |
| Signed arithmetic | `int32_t` |
| Raw byte or ASCII character | `uint8_t` |
| Boolean flag (0 or 1) | `uint8_t` or `_Bool` |
| Bit manipulation / masks | `uint32_t` or `uint64_t` |

### Type limits

`<stdint.h>` defines constants for every type's minimum and maximum value:

```c
#include <stdint.h>

uint32_t max_u32 = UINT32_MAX;   /* 4 294 967 295 */
int32_t  min_i32 = INT32_MIN;    /* -2 147 483 648 */
int32_t  max_i32 = INT32_MAX;    /*  2 147 483 647 */
```

Always check these constants rather than hard-coding magic numbers like `2147483647`.

---

## 3. Arithmetic Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `+` | Addition | `a + b` |
| `-` | Subtraction | `a - b` |
| `*` | Multiplication | `a * b` |
| `/` | Division | `a / b` |
| `%` | Remainder (modulo) | `a % b` |

Two important rules:

**Integer division truncates toward zero.** `7 / 2` produces `3`, not `3.5`.

**The `%` operator is for integers only.** `7 % 2` produces `1`. The result has the same sign as
the left operand in C11.

### Overflow

Unsigned integer overflow is well-defined in C: it wraps around modulo 2ⁿ where n is the bit
width. Signed integer overflow is *undefined behaviour* — the compiler is free to produce any
result or optimise the surrounding code in surprising ways.

```c
uint32_t a = UINT32_MAX;  /* 4 294 967 295 */
uint32_t b = a + 1;       /* wraps to 0 — defined, predictable */

int32_t  c = INT32_MAX;   /* 2 147 483 647 */
int32_t  d = c + 1;       /* UNDEFINED BEHAVIOUR — never do this */
```

---

## 4. Relational and Logical Operators

Relational operators compare two values and produce `1` (true) or `0` (false):

| Operator | Meaning |
|----------|---------|
| `==` | Equal to |
| `!=` | Not equal to |
| `<` | Less than |
| `>` | Greater than |
| `<=` | Less than or equal |
| `>=` | Greater than or equal |

Logical operators combine conditions:

| Operator | Meaning | Short-circuits? |
|----------|---------|----------------|
| `&&` | Logical AND | Yes |
| `\|\|` | Logical OR | Yes |
| `!` | Logical NOT | — |

Short-circuit evaluation means the right operand is not evaluated if the result is already
determined by the left operand. This matters when the right operand has side effects or could
cause an error (such as a null pointer dereference).

---

## 5. Bitwise Operators

Bitwise operators work directly on the binary representation of integers. They are fundamental to
systems programming: setting hardware flags, packing data, and writing efficient low-level code.

| Operator | Meaning |
|----------|---------|
| `&` | Bitwise AND |
| `\|` | Bitwise OR |
| `^` | Bitwise XOR |
| `~` | Bitwise NOT (complement) |
| `<<` | Left shift |
| `>>` | Right shift |

> Always apply bitwise operators to *unsigned* types. The behaviour of `>>` on signed negative
> values is implementation-defined in C11.

### AND — clearing bits

`&` keeps a bit only if it is set in *both* operands. It is used to *test* or *clear* specific
bits.

```
  0b00111010   (value)
& 0b00001111   (mask)
= 0b00001010   (only the lower 4 bits survive)
```

### OR — setting bits

`|` sets a bit if it is set in *either* operand. It is used to *set* specific bits.

```
  0b00001010   (value)
| 0b00100000   (mask)
= 0b00101010   (bit 5 is now set)
```

### XOR — toggling bits

`^` sets a bit if the two operands *differ*. It is used to *toggle* specific bits.

```
  0b00101010   (value)
^ 0b00100000   (mask)
= 0b00001010   (bit 5 is toggled off)
```

### NOT — inverting all bits

`~` flips every bit in the operand.

```
~ 0b00001111  =  0b11110000   (for an 8-bit type)
```

### Shifts

`<<` moves all bits to the left by n positions, filling the vacated right bits with zeros.
Shifting left by n is equivalent to multiplying by 2ⁿ (as long as no bits are lost).

`>>` moves all bits to the right by n positions. For unsigned types, vacated left bits are filled
with zeros (logical shift).

```c
uint32_t x = 1u << 4;   /* x = 16  (0b...00010000) */
uint32_t y = x >> 2;    /* y = 4   (0b...00000100) */
```

### Practical pattern: flags and masks

A common systems-programming pattern is to represent a set of independent boolean flags as bits
in a single integer. Each flag occupies one bit, and named constants identify which bit is which.

```c
/* encoding: utf-8 */
#include <stdio.h>
#include <stdint.h>
#include <inttypes.h>

/* Permission flags for a file entry */
#define PERM_LEER     (UINT32_C(1) << 0)   /* bit 0 — read    */
#define PERM_ESCRIBIR (UINT32_C(1) << 1)   /* bit 1 — write   */
#define PERM_EJECUTAR (UINT32_C(1) << 2)   /* bit 2 — execute */

int main(void) {
    uint32_t permisos = 0;

    /* Otorgar permisos de lectura y escritura */
    permisos |= PERM_LEER;
    permisos |= PERM_ESCRIBIR;

    /* Verificar permiso de lectura */
    if (permisos & PERM_LEER) {
        printf("Lectura permitida\n");
    }

    /* Revocar permiso de escritura */
    permisos &= ~PERM_ESCRIBIR;

    printf("Permisos finales: 0x%08" PRIX32 "\n", permisos);
    return 0;
}
```

Expected output:

```
Lectura permitida
Permisos finales: 0x00000001
```

Four operations to memorise:

| Goal | Expression |
|------|-----------|
| Set bit n | `x \|= (UINT32_C(1) << n)` |
| Clear bit n | `x &= ~(UINT32_C(1) << n)` |
| Toggle bit n | `x ^= (UINT32_C(1) << n)` |
| Test bit n | `(x >> n) & UINT32_C(1)` |

> `UINT32_C(1)` is a macro from `<stdint.h>` that produces the constant `1` as a `uint32_t`
> literal, avoiding accidental promotion issues when shifting.

---

## 6. Type Casting

A cast explicitly converts a value from one type to another:

```c
uint32_t a = 300;
uint8_t  b = (uint8_t)a;   /* truncates: b = 44 (300 mod 256) */
```

The cast operator `(type)` has high precedence. Always cast explicitly rather than relying on
implicit conversions — implicit conversions are a common source of bugs, and compilers often warn
about them with `-Wall`.

Important cases to watch:

**Narrowing cast** — converting to a smaller type silently truncates bits. Always verify the value
fits before casting.

**Unsigned to signed** — if the value exceeds `INT32_MAX`, the result is implementation-defined.

**Floating point to integer** — the fractional part is discarded (truncated toward zero), not
rounded.

```c
float   f = 3.9f;
int32_t i = (int32_t)f;   /* i = 3, not 4 */
```

---

## 7. Compound Assignment Operators

These are shorthand forms that apply an operator to a variable and store the result back:

| Operator | Equivalent to |
|----------|--------------|
| `a += b` | `a = a + b` |
| `a -= b` | `a = a - b` |
| `a *= b` | `a = a * b` |
| `a /= b` | `a = a / b` |
| `a %= b` | `a = a % b` |
| `a &= b` | `a = a & b` |
| `a \|= b` | `a = a \| b` |
| `a ^= b` | `a = a ^ b` |
| `a <<= n` | `a = a << n` |
| `a >>= n` | `a = a >> n` |

---

## 8. Operator Precedence

Precedence determines which operators bind more tightly when there is no parentheses. The table
below lists the most relevant levels, from highest to lowest:

| Precedence | Operators |
|-----------|-----------|
| Highest | `~` `!` (unary) |
| | `*` `/` `%` |
| | `+` `-` |
| | `<<` `>>` |
| | `<` `<=` `>` `>=` |
| | `==` `!=` |
| | `&` |
| | `^` |
| | `\|` |
| | `&&` |
| | `\|\|` |
| Lowest | `=` `+=` `-=` … (assignment) |

When in doubt, use parentheses. They cost nothing and prevent misreading. A common mistake is
writing `x & MASK == 0` when `==` has higher precedence than `&`, making it equivalent to
`x & (MASK == 0)` — almost certainly not what was intended.

---

## 9. Undefined Behaviour

Undefined behaviour (UB) means the C standard places no requirement on what the program does.
The compiler is allowed to assume UB never occurs, which can lead to optimisations that make the
program appear to work differently than expected — silently producing wrong results, crashing, or
doing nothing at all.

The most common sources of UB at this stage:

- Reading an uninitialised variable.
- Signed integer overflow.
- Shifting by a negative amount or by an amount ≥ the bit width of the type.
- Casting a pointer to an incompatible type and dereferencing it.

The `-Wall -Wextra` flags catch many of these at compile time. For runtime detection, tools like
AddressSanitizer (`-fsanitize=address,undefined`) are invaluable and will be introduced later.

---

## 10. Practice Exercises

1. Declare variables of each signed and unsigned fixed-width type. Print the value of the
   corresponding `MIN` and `MAX` constant for each type using the correct format macro.

2. Write a program that reads a `uint32_t` from the user (using `scanf` with `%" SCNu32 "`) and
   prints whether it is even or odd, using only the bitwise AND operator (no modulo).

3. Extend the flags example from section 5 to add a fourth permission flag `PERM_HEREDAR` (bit 3).
   Write code that sets all four permissions, then revokes `PERM_EJECUTAR` and `PERM_HEREDAR`, and
   prints the final bitmask.

4. Predict the result of each expression below, then verify with a small program:
   - `(uint8_t)300`
   - `(int32_t)3.9f`
   - `UINT32_MAX + 1` (store in a `uint32_t`)
   - `1 << 31` vs `UINT32_C(1) << 31`

5. Fix the precedence bug: write a program where `x & MASK == 0` produces a wrong result, then
   correct it with parentheses.

---

## Summary

- Always initialise variables at the point of declaration.
- Use fixed-width types from `<stdint.h>` and their limit constants (`UINT32_MAX`, `INT32_MIN`,
  etc.) instead of magic numbers.
- Integer division truncates; unsigned overflow wraps; signed overflow is undefined behaviour.
- Bitwise operators (`&`, `|`, `^`, `~`, `<<`, `>>`) operate on unsigned types only.
- The flags-and-masks pattern — using individual bits to represent independent boolean states — is
  a core systems-programming idiom.
- Use `UINT32_C(1) << n` when constructing bit masks to avoid type promotion surprises.
- Explicit casts make intent clear; always verify that narrowing casts do not lose significant bits.
- When operator precedence is unclear, add parentheses — especially around bitwise operators mixed
  with comparisons.
