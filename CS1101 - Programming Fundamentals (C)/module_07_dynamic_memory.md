# Module 7 — Dynamic Memory Management

## 1. Why Dynamic Memory?

Stack-allocated variables have two limitations: their size must be known at compile time, and their
lifetime is tied to the function that declares them. Neither constraint holds for many real
problems — reading a file of unknown length, building a list that grows as the user adds items,
or allocating a struct that must outlive the function that created it.

The heap solves both problems. Memory on the heap is allocated at runtime, can be any size the
system allows, and lives until you explicitly release it. The tradeoff is that the programmer is
fully responsible for that release — the runtime will not do it automatically.

The four heap-management functions all live in `<stdlib.h>`:

| Function | Purpose |
|----------|---------|
| `malloc` | Allocate an uninitialised block |
| `calloc` | Allocate a zero-initialised block |
| `realloc` | Resize an existing block |
| `free` | Release a block back to the heap |

---

## 2. malloc

```c
void *malloc(size_t size);
```

Allocates `size` bytes on the heap and returns a pointer to the first byte. The contents are
uninitialised — reading them before writing is undefined behaviour.

Returns `NULL` if the allocation fails (out of memory or size is 0). Always check the return
value before using the pointer.

```c
/* encoding: utf-8 */
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <inttypes.h>

int main(void) {
    uint32_t count = 5;
    uint32_t *data = malloc(count * sizeof(uint32_t));
    if (data == NULL) {
        fprintf(stderr, "Allocation failed\n");
        return 1;
    }

    for (uint32_t i = 0; i < count; i++) {
        data[i] = (i + 1) * 10;
    }

    for (uint32_t i = 0; i < count; i++) {
        printf("data[%" PRIu32 "] = %" PRIu32 "\n", i, data[i]);
    }

    free(data);
    return 0;
}
```

Expected output:

```
data[0] = 10
data[1] = 20
data[2] = 30
data[3] = 40
data[4] = 50
```

Two idioms to adopt from the start:

**Always write `malloc(count * sizeof(type))`**, never a hard-coded byte count. If the type
changes, the allocation size updates automatically.

**Set the pointer to `NULL` after freeing** to prevent accidental use-after-free:

```c
free(data);
data = NULL;
```

---

## 3. calloc

```c
void *calloc(size_t count, size_t size);
```

Allocates memory for `count` elements of `size` bytes each — equivalent to
`malloc(count * size)` — but also **zero-initialises every byte**. Use `calloc` when you need
the memory to start at zero, which is common for counters, flags, and struct arrays.

```c
uint32_t *counters = calloc(8, sizeof(uint32_t));
if (counters == NULL) { /* handle error */ }
/* All 8 elements are guaranteed to be 0 */
```

`calloc` also has a slight safety advantage: internally it checks for integer overflow in
`count * size`, whereas `malloc(count * size)` can silently overflow if both values are large.

---

## 4. realloc

```c
void *realloc(void *ptr, size_t new_size);
```

Resizes a previously allocated block to `new_size` bytes. If the block can be extended in place,
the same pointer is returned. If not, a new block is allocated, the old data is copied, the old
block is freed, and the new pointer is returned.

Returns `NULL` on failure — and critically, **the original block is not freed** when `realloc`
fails. This leads to a classic memory leak if the result is assigned back to the same pointer:

```c
/* WRONG — leaks memory if realloc fails */
data = realloc(data, new_size);

/* CORRECT — preserve the original pointer until success is confirmed */
uint32_t *tmp = realloc(data, new_size);
if (tmp == NULL) {
    /* data is still valid; handle the error */
    free(data);
    return NULL;
}
data = tmp;
```

### Dynamic array concept

The canonical use of `realloc` is a growable buffer: start with a small allocation and double the
capacity each time it is exhausted. Doubling keeps the amortised cost of appending at O(1).

```c
/* encoding: utf-8 */
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <inttypes.h>

int main(void) {
    uint32_t  capacity = 2;
    uint32_t  length   = 0;
    uint32_t *items    = malloc(capacity * sizeof(uint32_t));
    if (items == NULL) return 1;

    /* Append values 1..8 */
    for (uint32_t v = 1; v <= 8; v++) {
        if (length == capacity) {
            capacity *= 2;
            uint32_t *tmp = realloc(items, capacity * sizeof(uint32_t));
            if (tmp == NULL) { free(items); return 1; }
            items = tmp;
        }
        items[length++] = v;
    }

    for (uint32_t i = 0; i < length; i++) {
        printf("%" PRIu32 " ", items[i]);
    }
    printf("\n");

    free(items);
    return 0;
}
```

Expected output:

```
1 2 3 4 5 6 7 8
```

A production implementation wraps this pattern in a struct (`data`, `length`, `capacity`) with
dedicated push, get, and free functions. That is left as a practice exercise.

---

## 5. free

```c
void free(void *ptr);
```

Releases a heap block back to the allocator. After `free`:

- The pointer value is invalid — do not dereference it.
- The memory may be reallocated immediately — its contents are unpredictable.
- Calling `free` again on the same pointer is undefined behaviour (double-free).

Rules that prevent the three most common heap errors:

| Rule | Error it prevents |
|------|------------------|
| Always `free` every `malloc`/`calloc`/`realloc` | Memory leak |
| Set pointer to `NULL` after `free` | Use-after-free, double-free |
| Only `free` pointers returned by the allocator | Heap corruption |

`free(NULL)` is defined to do nothing and is always safe. This means you can safely call `free`
on a `NULL`-initialised pointer without checking first.

---

## 6. Allocating Structs

Structs are commonly heap-allocated and accessed through pointers. The constructor/destructor
pattern keeps allocation and initialisation together:

```c
/* encoding: utf-8 */
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <inttypes.h>
#include <string.h>

typedef struct {
    char     name[32];
    uint32_t health;
    uint32_t attack;
} enemy_t;

enemy_t *enemy_create(const char *name, uint32_t health, uint32_t attack) {
    enemy_t *e = malloc(sizeof(enemy_t));
    if (e == NULL) return NULL;

    strncpy(e->name, name, sizeof(e->name) - 1);
    e->name[sizeof(e->name) - 1] = '\0';
    e->health = health;
    e->attack = attack;
    return e;
}

void enemy_destroy(enemy_t **e) {
    free(*e);
    *e = NULL;   /* null out the caller's pointer */
}

void enemy_print(const enemy_t *e) {
    printf("Enemy: %-16s HP: %" PRIu32 "  ATK: %" PRIu32 "\n",
           e->name, e->health, e->attack);
}

int main(void) {
    enemy_t *goblin = enemy_create("Goblin",  30, 5);
    enemy_t *troll  = enemy_create("Troll",  120, 18);

    if (goblin == NULL || troll == NULL) {
        fprintf(stderr, "Allocation failed\n");
        return 1;
    }

    enemy_print(goblin);
    enemy_print(troll);

    enemy_destroy(&goblin);
    enemy_destroy(&troll);
    return 0;
}
```

Expected output:

```
Enemy: Goblin           HP: 30   ATK: 5
Enemy: Troll            HP: 120  ATK: 18
```

`enemy_destroy` takes a `enemy_t **` so it can set the caller's pointer to `NULL` after freeing.
This is the defensive pattern introduced in section 5.

---

## 7. Memory Errors

### Memory leak

A leak occurs when allocated memory is never freed. The heap shrinks silently; in long-running
programs this eventually exhausts available memory.

```c
void leaky(void) {
    uint32_t *p = malloc(64 * sizeof(uint32_t));
    /* p is never freed — 256 bytes leaked every call */
}
```

### Use-after-free

Accessing memory after calling `free` on it is undefined behaviour. The allocator may have
reused the block; the bytes could hold anything.

```c
uint32_t *p = malloc(sizeof(uint32_t));
*p = 42;
free(p);
printf("%" PRIu32 "\n", *p);   /* undefined behaviour */
```

### Double-free

Calling `free` twice on the same pointer is undefined behaviour and typically corrupts the
allocator's internal data structures, causing crashes elsewhere in the program — often far from
the actual bug.

```c
free(p);
free(p);   /* undefined behaviour */
```

Setting `p = NULL` after the first `free` makes the second `free(NULL)`, which is safe and a
no-op.

### Buffer overflow on the heap

Writing beyond the end of a heap block corrupts adjacent heap metadata or other allocations.
The crash, if any, typically occurs later and at an unrelated location, making these bugs
particularly hard to diagnose without tooling.

---

## 8. Valgrind

Valgrind is a dynamic analysis tool that instruments your program at runtime and reports memory
errors: leaks, use-after-free, double-free, uninitialised reads, and heap buffer overflows.

### Installation

```bash
# Debian / Ubuntu
sudo apt install valgrind

# macOS (via Homebrew — support is limited on recent macOS)
brew install valgrind
```

### Basic usage

Compile with debug symbols (`-g`), then run under Valgrind:

```bash
gcc -std=c11 -Wall -Wextra -g -o program source.c
valgrind --leak-check=full --track-origins=yes ./program
```

| Flag | Effect |
|------|--------|
| `--leak-check=full` | Report every leaked block with its allocation site |
| `--track-origins=yes` | Report where uninitialised values were created |

### Reading Valgrind output

A clean run ends with:

```
LEAK SUMMARY:
   definitely lost: 0 bytes in 0 blocks
   ...
ERROR SUMMARY: 0 errors from 0 contexts
```

A leak report looks like:

```
definitely lost: 256 bytes in 1 blocks
   at 0x...: malloc (vg_replace_malloc.c:...)
   by 0x...: leaky (source.c:4)
   by 0x...: main  (source.c:10)
```

The call stack points directly to the `malloc` that was never freed. Fix the code, re-run
Valgrind, and repeat until the summary shows zero errors.

Valgrind slows execution by roughly 10–50×, so use it during development and testing, not in
production builds.

---

## 9. Common Mistakes

**Not checking `malloc`/`calloc` return value**  
On memory-constrained systems or with large allocations, these functions can and do return `NULL`.
Dereferencing `NULL` immediately crashes; propagate the error instead.

**Assigning `realloc` directly back to the original pointer**  
If `realloc` returns `NULL`, the original block is still allocated but the pointer is now `NULL`
and the block is permanently leaked. Always use a temporary pointer.

**`sizeof` a pointer instead of the type**  
`malloc(sizeof(data))` when `data` is a `uint32_t *` allocates 8 bytes (the pointer size), not
enough for any `uint32_t`. Write `malloc(count * sizeof(uint32_t))` or
`malloc(count * sizeof(*data))` — the latter automatically tracks the pointed-to type.

**Freeing stack memory**  
Only pointers returned by `malloc`, `calloc`, or `realloc` should be passed to `free`. Passing
the address of a local variable causes heap corruption.

**Forgetting to free in all code paths**  
A function that returns early on error must free everything it allocated before returning. Draw
the allocation/free pairs on paper when the code is complex.

---

## 10. Practice Exercises

1. Write a program that intentionally leaks memory, then run it under Valgrind. Read the output,
   fix the leak, and confirm Valgrind reports zero errors.

2. Wrap the dynamic array from section 4 into a proper `dynarray_t` struct with `push`,
   `get`, `length`, and `free` functions. All functions should check for `NULL` inputs and
   return an error code where appropriate.

3. Write a function `char *str_duplicate(const char *s)` that allocates a copy of `s` on the
   heap (like the POSIX `strdup`). The caller is responsible for freeing the result. Test it and
   verify with Valgrind.

4. Demonstrate use-after-free: allocate a `uint32_t`, free it, then write to it. Observe the
   behaviour. Run under Valgrind to see it detected. Then fix it.

5. Extend the `enemy_t` example to manage a heap-allocated array of enemies. Write
   `enemy_t **enemies_create(uint32_t count)` and `void enemies_destroy(enemy_t **enemies, uint32_t count)`.
   Verify with Valgrind that all memory is freed correctly.

---

## Summary

- `malloc(n)` allocates `n` uninitialised bytes; `calloc(count, size)` allocates and
  zero-initialises; `realloc(ptr, n)` resizes; `free(ptr)` releases.
- Always check allocator return values for `NULL` before using the pointer.
- Use `malloc(count * sizeof(type))` or `malloc(count * sizeof(*ptr))` — never hard-code byte
  counts.
- When using `realloc`, assign to a temporary pointer first; only update the original on success.
- After `free`, set the pointer to `NULL` to prevent use-after-free and double-free bugs.
- The three cardinal heap errors are: memory leak (forgot to free), use-after-free (used after
  freeing), and double-free (freed twice). All are undefined behaviour.
- Run every program under `valgrind --leak-check=full --track-origins=yes` during development
  until the summary shows zero errors and zero leaked bytes.
