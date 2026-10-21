# CS 1105 — Module 8: Compiler Processes Overview

---

## 1. What is a Compiler?

A **compiler** is a program that translates **high-level source code** (C, Java, Python) into **low-level target code** (assembly or machine code) that a CPU can execute.

```
Source Code   →   [ Compiler ]   →   Machine Code
  (C, Java)                         (binary / assembly)
```

Unlike an **interpreter** (which executes code line by line), a compiler processes the **entire program** before execution, producing an independent executable.

| Feature         | Compiler                        | Interpreter                   |
|-----------------|---------------------------------|-------------------------------|
| Translation     | Entire program at once          | Line by line at runtime       |
| Speed           | Faster execution                | Slower execution              |
| Error reporting | All errors before running       | Errors found during execution |
| Output          | Standalone executable           | No separate executable        |
| Examples        | GCC (C), javac (Java)           | Python, Ruby, JavaScript      |

---

## 2. The Compilation Pipeline

Compilation is not one step — it's a **pipeline of phases**, each transforming the code into a closer representation of machine instructions.

```
Source Code (.c)
      ↓
  1. Lexical Analysis       → Token stream
      ↓
  2. Syntax Analysis        → Parse tree
      ↓
  3. Semantic Analysis      → Annotated AST
      ↓
  4. Intermediate Code Gen  → IR (e.g., three-address code)
      ↓
  5. Optimization           → Optimized IR
      ↓
  6. Code Generation        → Assembly / Machine code
      ↓
  7. Linking                → Executable
```

---

## 3. Phase 1 — Lexical Analysis (Scanning)

The **lexer** (or scanner) reads raw source text and breaks it into **tokens** — the smallest meaningful units.

### Token Types

| Type        | Examples                          |
|-------------|-----------------------------------|
| Keyword     | `int`, `if`, `while`, `return`    |
| Identifier  | `x`, `total`, `myFunction`        |
| Literal     | `42`, `3.14`, `"hello"`           |
| Operator    | `+`, `-`, `=`, `==`, `&&`         |
| Punctuation | `(`, `)`, `{`, `}`, `;`           |

### Example

```c
int x = a + 5;
```

Tokens produced:

```
[KEYWORD: int] [IDENT: x] [OP: =] [IDENT: a] [OP: +] [LIT: 5] [PUNCT: ;]
```

> Lexical errors: unrecognized characters, malformed literals (e.g., `0xGG`).

---

## 4. Phase 2 — Syntax Analysis (Parsing)

The **parser** takes the token stream and checks it against the language's **grammar rules**, building a **Parse Tree** (or Abstract Syntax Tree — AST).

### Grammar Example (simplified)

```
expression → expression + term
           | term
term       → identifier | number
```

### AST for `x = a + 5`

```
        ASSIGN
       /      \
      x        ADD
              /   \
             a     5
```

> Syntax errors: missing semicolons, mismatched brackets, incorrect statement structure.

---

## 5. Phase 3 — Semantic Analysis

The **semantic analyzer** checks that the program is **meaningful** — not just structurally correct.

### Checks performed:

| Check                  | Example Error                              |
|------------------------|--------------------------------------------|
| Type checking          | `int x = "hello";` — type mismatch        |
| Undeclared variables   | Using `y` before declaring it             |
| Function arity         | Calling `add(1)` when it requires 2 args  |
| Scope rules            | Accessing a variable outside its scope    |
| Return type matching   | `int` function returning a `string`       |

The output is an **annotated AST** — the tree with type and scope information attached.

---

## 6. Phase 4 — Intermediate Code Generation

The compiler converts the AST into **Intermediate Representation (IR)** — a platform-independent, simplified instruction format.

### Three-Address Code (TAC)

Each instruction has at most **one operator and three addresses** (two sources, one destination).

```c
// Source
x = (a + b) * (c - d);

// Three-Address Code
t1 = a + b
t2 = c - d
t3 = t1 * t2
x  = t3
```

> IR is easier to optimize and can target multiple architectures from one front-end.

---

## 7. Phase 5 — Optimization

The optimizer improves the IR to make the final code **faster or smaller** — without changing what it does.

### Common Optimizations

| Technique               | Description                                      | Example                              |
|-------------------------|--------------------------------------------------|--------------------------------------|
| Constant Folding        | Evaluate constant expressions at compile time    | `x = 3 * 4` → `x = 12`              |
| Dead Code Elimination   | Remove code that never executes                  | Code after `return` in a block       |
| Common Subexpression    | Reuse already-computed values                    | Avoid computing `a+b` twice          |
| Loop Invariant Removal  | Move unchanging computation out of loops         | Move `n*2` outside a loop            |
| Inlining                | Replace function call with function body         | Small utility functions              |
| Register Allocation     | Keep frequently used values in CPU registers     | Avoid repeated memory loads          |

---

## 8. Phase 6 — Code Generation

The **code generator** translates optimized IR into **target assembly or machine code**.

```
IR:                         Assembly (ARM-style):
t1 = a + b      →           LDR R1, [a]
                            LDR R2, [b]
                            ADD R3, R1, R2
                            STR R3, [t1]
```

Tasks at this phase:
- **Instruction selection** — pick the best machine instruction for each IR operation
- **Register allocation** — decide which values live in registers vs. memory
- **Instruction scheduling** — reorder instructions to avoid pipeline stalls (Module 6)

---

## 9. Phase 7 — Linking

The **linker** combines multiple compiled object files and libraries into a single executable.

```
main.o  +  math.o  +  libc.a   →   [ Linker ]   →   program.exe
```

### Linker Tasks

| Task                  | Description                                          |
|-----------------------|------------------------------------------------------|
| Symbol resolution     | Match function calls to their definitions            |
| Relocation            | Adjust addresses now that everything is combined     |
| Library linking       | Static (`.a`) copies code in; Dynamic (`.so`) links at runtime |

---

## 10. Worked Example — Tracing a Full Compilation

**Source (C):**

```c
int add(int a, int b) {
    return a + b;
}
```

**Lexical Analysis → Tokens:**
```
[int] [add] [(] [int] [a] [,] [int] [b] [)] [{] [return] [a] [+] [b] [;] [}]
```

**Parsing → AST:**
```
FunctionDef: add
  Params: (int a, int b)
  Body:
    Return:
      ADD
     /   \
    a     b
```

**Semantic Analysis:**
- `a` and `b` are declared as `int` ✓
- `a + b` produces `int` ✓
- Return type matches `int` ✓

**Intermediate Code:**
```
t1 = a + b
return t1
```

**Optimization:**
```
return a + b    (t1 eliminated — only used once)
```

**Code Generation (ARM):**
```
ADD  R0, R0, R1   ; R0 = a, R1 = b, result in R0
MOV  PC, LR       ; return
```

---

## 📝 Module 8 Quiz

**Q1.** What is the difference between a compiler and an interpreter?

<details>
<summary>Show Answer</summary>

A **compiler** translates the entire source program into machine code before execution, producing a standalone executable. An **interpreter** translates and executes the program line by line at runtime with no separate output file. Compiled programs run faster; interpreted programs are more flexible and easier to debug interactively.
</details>

---

**Q2.** What does the lexer produce, and what does the parser do with it?

<details>
<summary>Show Answer</summary>

The **lexer** scans raw source text and produces a stream of **tokens** — the smallest meaningful units (keywords, identifiers, operators, literals, punctuation).

The **parser** takes that token stream and checks it against the language's grammar rules, building an **Abstract Syntax Tree (AST)** that represents the program's structure hierarchically.
</details>

---

**Q3.** Identify the semantic error in this code snippet:

```c
int x = "hello";
```

<details>
<summary>Show Answer</summary>

This is a **type mismatch** error. The variable `x` is declared as `int`, but the value `"hello"` is a string literal. The semantic analyzer would flag this because the types are incompatible — it is syntactically valid (no grammar violations) but semantically meaningless.
</details>

---

**Q4.** Apply constant folding and dead code elimination to this IR:

```
t1 = 6 * 7
t2 = t1 + 0
x  = t2
y  = 10
return x
```

<details>
<summary>Show Answer</summary>

**Constant folding:** `6 * 7 = 42`, `42 + 0 = 42`

**Dead code elimination:** `y = 10` is assigned but never used before `return`

**Optimized IR:**
```
x = 42
return x
```
</details>

---

**Q5.** What is the difference between static and dynamic linking?

<details>
<summary>Show Answer</summary>

**Static linking** copies all required library code directly into the executable at compile time. The result is a larger but self-contained file — no external dependencies at runtime.

**Dynamic linking** keeps library code in separate shared files (`.so` / `.dll`) and links them at runtime. The executable is smaller and multiple programs can share the same library in memory, but the library must be present on the target system.
</details>

---

## 🔑 Key Takeaways

- A compiler transforms high-level source code into machine code through a structured pipeline of phases.
- Lexical analysis tokenizes source text; syntax analysis builds a parse tree from tokens.
- Semantic analysis enforces type safety, scope rules, and meaning — beyond grammar correctness.
- Intermediate representation (IR) decouples the front-end (language) from the back-end (hardware).
- Optimization improves code speed and size without changing program behavior.
- The linker combines object files and resolves symbols into a final executable.

---

## 🎓 Course Complete!

You have covered all 8 modules of **CS 1105 Digital Electronics & Computer Architecture**:

| Module | Topic                              |
|--------|------------------------------------|
| 1      | Introduction to Digital Design     |
| 2      | Combinational Logic Design         |
| 3      | Sequential Logic Design            |
| 4      | Computer Arithmetic                |
| 5      | Memory and Programmable Logic      |
| 6      | Computer Architecture              |
| 7      | Assembler Language                 |
| 8      | Compiler Processes Overview        |

*Good luck with your studies!*
