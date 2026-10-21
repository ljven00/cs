# Module 6 — Structs, Enums, and User-Defined Types

## 1. Structs

A `struct` groups related variables of different types under a single name. Where an array holds
many values of the *same* type, a struct holds a fixed set of values of *different* types.

### Declaration and definition

```c
struct point {
    int32_t x;
    int32_t y;
};
```

This declares a new type called `struct point`. The variables inside (`x`, `y`) are called
*members* or *fields*. The declaration alone allocates no memory — you must define a variable of
that type to do so.

```c
struct point origin;          /* uninitialised              */
struct point p = {3, -7};    /* initialise by position     */
struct point q = {.x = 3, .y = -7};  /* initialise by name — preferred */
```

Designated initialisers (the `.field = value` form) are preferred because they are immune to
field reordering bugs and make the code self-documenting.

### Accessing members

Use the `.` operator for a struct variable and the `->` operator for a pointer to a struct:

```c
/* encoding: utf-8 */
#include <stdio.h>
#include <stdint.h>
#include <inttypes.h>

struct point {
    int32_t x;
    int32_t y;
};

void print_point(const struct point *p) {
    printf("(%"PRId32", %"PRId32")\n", p->x, p->y);
}

int main(void) {
    struct point a = {.x = 4, .y = -2};
    print_point(&a);

    a.x += 1;          /* direct member access */
    print_point(&a);

    return 0;
}
```

Expected output:

```
(4, -2)
(5, -2)
```

`p->x` is exactly equivalent to `(*p).x`. The arrow operator exists purely for readability.

### Structs are passed by value

Like all other types in C, structs are passed by value — the function receives a full copy.
For any struct larger than a pointer, always pass a pointer instead to avoid the copying overhead.
Mark it `const` if the function only reads the struct.

### Memory layout and padding

The compiler may insert *padding bytes* between fields to satisfy alignment requirements of the
target CPU. Do not assume a struct's size equals the sum of its field sizes.

```c
struct example {
    uint8_t  a;    /* 1 byte  */
    /* 3 bytes padding likely inserted here */
    uint32_t b;    /* 4 bytes */
};
/* sizeof(struct example) is likely 8, not 5 */
```

Use `sizeof` to get the actual size. When serialising structs to disk or a network, never write
the raw struct bytes directly — the padding and byte order may differ on another machine.

---

## 2. typedef

`typedef` creates an alias for an existing type. Its most common use with structs eliminates the
need to write `struct` before the type name every time:

```c
typedef struct {
    int32_t x;
    int32_t y;
} point_t;
```

Now `point_t` can be used directly:

```c
point_t origin = {.x = 0, .y = 0};
```

This course uses the `_t` suffix convention for typedef'd types (matching `<stdint.h>`'s
`uint32_t` style). Avoid reusing names already reserved by POSIX or the standard library.

### Self-referential structs and typedef

A struct that contains a pointer to its own type (needed for linked lists) must use the tagged
form because the typedef name is not yet defined when the struct body is being written:

```c
typedef struct node {
    uint32_t    value;
    struct node *next;   /* must use "struct node", not "node_t" here */
} node_t;
```

---

## 3. Enums

An `enum` defines a set of named integer constants. It makes code that works with a fixed set of
states far more readable than bare integer literals.

```c
typedef enum {
    DIR_NORTH = 0,
    DIR_EAST,
    DIR_SOUTH,
    DIR_WEST
} direction_t;
```

Without explicit values, each constant is one greater than the previous, starting from 0.
Assigning explicit values (as with `DIR_NORTH = 0`) is good practice — it makes the mapping clear
and stable if the list is later extended.

```c
/* encoding: utf-8 */
#include <stdio.h>

typedef enum {
    DIR_NORTH = 0,
    DIR_EAST,
    DIR_SOUTH,
    DIR_WEST
} direction_t;

const char *direction_name(direction_t d) {
    switch (d) {
        case DIR_NORTH: return "North";
        case DIR_EAST:  return "East";
        case DIR_SOUTH: return "South";
        case DIR_WEST:  return "West";
        default:        return "Unknown";
    }
}

int main(void) {
    direction_t heading = DIR_EAST;
    printf("Heading: %s\n", direction_name(heading));
    return 0;
}
```

Expected output:

```
Heading: East
```

An enum does not create a new numeric type in C — the underlying type is `int`. The compiler will
not prevent you from passing an arbitrary integer where an enum is expected, so the `default`
case in a `switch` over an enum is important.

---

## 4. Unions

A `union` looks like a struct but all its members share the same memory location. The size of a
union is the size of its largest member. Only one member holds a meaningful value at any time.

```c
typedef union {
    uint32_t as_u32;
    uint8_t  bytes[4];
} word_t;
```

Both `as_u32` and `bytes` refer to the same four bytes of memory. Writing through one and reading
through the other is type-punning, and in C (unlike C++) it is explicitly permitted.

```c
/* encoding: utf-8 */
#include <stdio.h>
#include <stdint.h>
#include <inttypes.h>

typedef union {
    uint32_t as_u32;
    uint8_t  bytes[4];
} word_t;

int main(void) {
    word_t w = {.as_u32 = 0x12345678};

    printf("Value:   0x%08" PRIX32 "\n", w.as_u32);
    for (int i = 0; i < 4; i++) {
        printf("byte[%d]: 0x%02X\n", i, w.bytes[i]);
    }
    return 0;
}
```

Expected output on a little-endian system:

```
Value:   0x12345678
byte[0]: 0x78
byte[1]: 0x56
byte[2]: 0x34
byte[3]: 0x12
```

The byte order in the output reveals the machine's *endianness*. On a little-endian system (x86,
ARM in default mode) the least significant byte comes first.

Unions are most often found in embedded systems for register access, in protocol parsers for
interpreting raw bytes, and in tagged union patterns (a struct containing an enum and a union)
to implement sum types.

---

## 5. Pointers to Structs

Passing and returning pointers to structs is the norm in C programs. The pattern appears so
frequently that fluency with `->` is essential.

```c
/* encoding: utf-8 */
#include <stdio.h>
#include <stdint.h>
#include <inttypes.h>
#include <string.h>

typedef struct {
    char     name[32];
    uint32_t score;
} player_t;

void award_points(player_t *p, uint32_t points) {
    p->score += points;
}

int main(void) {
    player_t hero;
    strncpy(hero.name, "Aria", sizeof(hero.name) - 1);
    hero.name[sizeof(hero.name) - 1] = '\0';
    hero.score = 0;

    award_points(&hero, 50);
    award_points(&hero, 30);

    printf("Player: %s — Score: %" PRIu32 "\n", hero.name, hero.score);
    return 0;
}
```

Expected output:

```
Player: Aria — Score: 80
```

---

## 6. Singly Linked List

An array stores elements at fixed, contiguous addresses. A linked list stores elements at arbitrary
addresses and connects them through pointers. This allows efficient insertion and removal at any
position without shifting elements, at the cost of extra memory per node and non-contiguous access.

### Node definition

```c
typedef struct node {
    uint32_t    value;
    struct node *next;
} node_t;
```

Each node holds a value and a pointer to the next node. The last node's `next` pointer is `NULL`.

### Building and traversing a list

```c
/* encoding: utf-8 */
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <inttypes.h>

typedef struct node {
    uint32_t    value;
    struct node *next;
} node_t;

node_t *node_create(uint32_t value) {
    node_t *n = malloc(sizeof(node_t));
    if (n == NULL) return NULL;
    n->value = value;
    n->next  = NULL;
    return n;
}

void list_print(const node_t *head) {
    const node_t *cur = head;
    while (cur != NULL) {
        printf("%" PRIu32, cur->value);
        if (cur->next != NULL) printf(" -> ");
        cur = cur->next;
    }
    printf("\n");
}

void list_free(node_t *head) {
    while (head != NULL) {
        node_t *next = head->next;
        free(head);
        head = next;
    }
}

int main(void) {
    node_t *head = node_create(10);
    head->next   = node_create(20);
    head->next->next = node_create(30);

    list_print(head);
    list_free(head);
    return 0;
}
```

Expected output:

```
10 -> 20 -> 30
```

`malloc` and `free` are introduced properly in Module 7. For now, understand that `malloc`
allocates memory on the heap that persists until `free` is called, and that every node allocated
with `malloc` must eventually be freed to avoid a memory leak. `list_free` walks the list and
frees each node, saving `next` before freeing the current node — accessing `head->next` after
`free(head)` would be undefined behaviour.

### Prepending a node

Inserting at the head of a singly linked list is O(1):

```c
node_t *list_prepend(node_t *head, uint32_t value) {
    node_t *n = node_create(value);
    if (n == NULL) return head;
    n->next = head;
    return n;
}
```

The caller must update its `head` pointer with the returned value:

```c
head = list_prepend(head, 5);
```

---

## 7. Doubly Linked List (Overview)

A doubly linked list adds a `prev` pointer to each node, allowing traversal in both directions
and O(1) removal of any node given only its pointer (no need to walk from the head to find the
predecessor).

```c
typedef struct dnode {
    uint32_t     value;
    struct dnode *next;
    struct dnode *prev;
} dnode_t;
```

The tradeoff is extra memory per node and more pointer updates on every insert and remove
operation. Doubly linked lists underpin many standard library containers and operating system
scheduler queues.

A full implementation — including insert, remove, and free — is left as a practice exercise.

---

## 8. Common Mistakes

**Forgetting `->` when accessing through a pointer**  
`p.x` when `p` is a `node_t *` is a compile error. Use `p->x` or `(*p).x`.

**Accessing a freed node**  
In `list_free`, saving `next = head->next` before `free(head)` is mandatory. After `free`, the
memory may be reused and `head->next` could return garbage.

**Assuming struct size equals the sum of field sizes**  
Padding makes this unreliable. Always use `sizeof`.

**Using an enum value outside its defined range**  
The compiler will not stop you from casting an arbitrary integer to an enum type. The `default`
case in a `switch` guards against this.

**Reading a union member that was not the last one written**  
While type-punning through unions is defined in C, reading a member without first writing it
produces an indeterminate value.

**String fields in structs**  
A `char name[32]` field is a fixed-size buffer. Never `strcpy` into it without bounding the copy
with `strncpy` and manually null-terminating.

---

## 9. Practice Exercises

1. Define a `rectangle_t` struct with `width` and `height` fields of type `uint32_t`. Write
   functions to compute its area and perimeter, both taking a `const rectangle_t *`. Test with
   three different rectangles.

2. Add a `list_append` function to the singly linked list example that inserts a node at the
   tail. Measure how its performance compares to `list_prepend` as the list grows, and explain
   the difference.

3. Write a `list_length` function and a `list_reverse` function for the singly linked list. The
   reverse must operate in place by relinking the existing nodes — do not allocate new nodes.

4. Implement a full doubly linked list with `insert_after`, `remove_node`, and `free_list`
   functions. Verify with Valgrind (covered in Module 7) that no memory is leaked.

5. Define a tagged union for a simple value type that can hold either a `uint32_t` integer, a
   `float`, or a boolean (`uint8_t`). Use an enum for the tag. Write a `print_value` function
   that dispatches on the tag and prints the correct member.

6. Define a `direction_t` enum and a `point_t` struct. Write a function
   `point_t move(point_t p, direction_t d, uint32_t steps)` that returns a new point displaced
   in the given direction.

---

## Summary

- A `struct` groups fields of different types; use designated initialisers for clarity and safety.
- Access struct fields with `.` on a value and `->` on a pointer; `->` is shorthand for `(*p).`.
- Always pass large structs by pointer, marking the parameter `const` when the function only reads.
- `typedef` removes the `struct` keyword from type names; the `_t` suffix is conventional.
- `enum` defines named integer constants; always include a `default` case when switching on one.
- A `union` overlays all its members at the same address; it is permitted in C to type-pun through
  a union, but only the last-written member holds a defined value.
- A singly linked list connects heap-allocated nodes through `next` pointers; every node from
  `malloc` must be freed to avoid leaks.
- A doubly linked list adds `prev` pointers for bidirectional traversal and O(1) removal by
  pointer, at the cost of extra memory and more complex pointer management.
