# CS 3304 — Analysis of Algorithms
## Module 7: Computational Complexity Theory

---

## 7.1 Introduction

Throughout this course we have designed algorithms and measured their efficiency. But we have mostly asked: *how fast can we solve this problem?* Computational complexity theory asks a deeper question:

> **How hard is this problem, fundamentally?** Is there a polynomial-time algorithm waiting to be discovered, or is the problem inherently intractable?

This distinction is not about our current knowledge — it is about the mathematical nature of the problem itself. Some problems are provably easy. Some are provably hard. And some sit in a mysterious middle ground where we strongly suspect they are hard but cannot yet prove it.

This module develops the mathematical framework for classifying problems by their inherent difficulty:

- **Decision problems** and the language-theoretic view of computation
- **The complexity classes P, NP, co-NP, PSPACE, EXP**
- **Polynomial-time reductions** — the tool for comparing problem difficulty
- **NP-completeness** — the class of hardest problems in NP
- **The P vs. NP question** — the most important open problem in mathematics
- **Proof techniques**: reduction, diagonalization, padding
- **Space complexity** and the relationship between time and space

By the end of this module you will understand why certain problems resist efficient algorithms, how to prove a problem is NP-hard, and what the resolution of P vs. NP would mean for computing.

---

## 7.2 Computational Models and Decision Problems

### The Turing Machine

Complexity theory is grounded in the **Turing Machine (TM)** — a theoretical model of computation consisting of:
- An infinite tape divided into cells
- A read/write head that moves left or right
- A finite set of states and a transition function

```
Tape:  [... | 1 | 0 | 1 | 1 | 0 | ...]
                   ↑
              read/write head
              current state: q3

Transition: δ(q3, 1) = (q5, 0, R)
            → write 0, move right, go to state q5
```

Why Turing Machines? Because:
- They capture **all** computations (Church-Turing Thesis)
- They enable **precise definitions** of time and space complexity
- The **Extended Church-Turing Thesis**: any physically realizable computation can be simulated by a TM with polynomial overhead

### Decision Problems

Complexity theory focuses on **decision problems** — problems with a yes/no answer. This simplifies the framework without losing generality (optimization problems can be converted to decision problems).

```
Optimization problem:  Find the shortest path from s to t
Decision problem:      Is there a path from s to t of length ≤ k?

If we can answer the decision version for all values of k,
we can binary search for the optimum → equivalent in polynomial time.
```

### Languages

A decision problem is formalized as a **language** L:

```
L ⊆ Σ*  (a set of strings over an alphabet Σ)

x ∈ L   ↔  the answer to the decision problem on input x is YES

Example:
  SHORTEST-PATH = {⟨G, s, t, k⟩ : G has a path from s to t of length ≤ k}
  3-SAT = {⟨φ⟩ : Boolean formula φ has a satisfying assignment}
```

A TM **decides** language L if it accepts all x ∈ L and rejects all x ∉ L.

---

## 7.3 Time Complexity Classes

### The Class P

> **P** = the class of decision problems solvable by a deterministic Turing machine in **polynomial time** O(n^k) for some constant k.

```java
/**
 * Problems in P — solvable efficiently.
 *
 * Sorting:        O(n log n)         → In P
 * Shortest path:  O((V+E) log V)     → In P
 * Matrix multiply:O(n^2.807)         → In P
 * Primality test: O(log^6 n)         → In P (AKS algorithm, 2002)
 * Linear program: Poly average       → In P (ellipsoid method, 1979)
 *
 * These all have polynomial-time algorithms.
 */
public class ProblemsInP {

    // REACHABILITY — is vertex t reachable from s? O(V+E)
    public static boolean reachable(List<List<Integer>> graph, int s, int t) {
        boolean[] visited = new boolean[graph.size()];
        Queue<Integer> queue = new LinkedList<>();
        queue.offer(s); visited[s] = true;
        while (!queue.isEmpty()) {
            int u = queue.poll();
            if (u == t) return true;
            for (int v : graph.get(u))
                if (!visited[v]) { visited[v] = true; queue.offer(v); }
        }
        return false;
    }

    // PRIMALITY — is n prime? O(sqrt(n)) trial division
    // (Note: the true P algorithm AKS is O(log^6 n) but complex)
    public static boolean isPrime(long n) {
        if (n < 2) return false;
        for (long i = 2; i * i <= n; i++) if (n % i == 0) return false;
        return true;
    }

    // BIPARTITE — is graph G bipartite? O(V+E)
    public static boolean isBipartite(List<List<Integer>> graph) {
        int n = graph.size();
        int[] color = new int[n];
        Arrays.fill(color, -1);
        for (int start = 0; start < n; start++) {
            if (color[start] != -1) continue;
            Queue<Integer> q = new LinkedList<>();
            q.offer(start); color[start] = 0;
            while (!q.isEmpty()) {
                int u = q.poll();
                for (int v : graph.get(u)) {
                    if (color[v] == -1) { color[v] = 1 - color[u]; q.offer(v); }
                    else if (color[v] == color[u]) return false;
                }
            }
        }
        return true;
    }
}
```

### The Class NP

> **NP** (Nondeterministic Polynomial) = the class of decision problems where a YES answer can be **verified** in polynomial time given a **certificate** (witness).

```
Equivalently:
  NP = problems solvable in polynomial time on a
       nondeterministic Turing machine

The NDTM "guesses" a certificate nondeterministically,
then verifies it deterministically in polynomial time.
```

**The key definition — polynomial-time verifier:**

```
Problem L ∈ NP  iff  there exists a polynomial-time algorithm V such that:
  x ∈ L  iff  ∃ certificate c with |c| ≤ poly(|x|) and V(x, c) = YES
```

### NP Examples with Certificates

```java
/**
 * Demonstrating the NP verifier structure.
 * Given a certificate, we can verify it in polynomial time.
 */
public class NPVerifiers {

    /**
     * 3-SAT VERIFIER
     * Problem: Does Boolean formula φ (in CNF with 3 literals/clause) have
     *          a satisfying assignment?
     * Certificate: a truth assignment (one bit per variable)
     * Verification: check each clause has at least one true literal  O(n)
     */
    public static boolean verify3SAT(int[][] clauses, boolean[] assignment) {
        for (int[] clause : clauses) {
            boolean clauseSat = false;
            for (int literal : clause) {
                // literal > 0 means variable (literal-1) is positive
                // literal < 0 means variable (-literal-1) is negated
                boolean val = assignment[Math.abs(literal) - 1];
                if (literal < 0) val = !val;
                if (val) { clauseSat = true; break; }
            }
            if (!clauseSat) return false; // unsatisfied clause
        }
        return true;
    }

    /**
     * HAMILTONIAN CYCLE VERIFIER
     * Problem: Does graph G have a Hamiltonian cycle?
     * Certificate: a sequence of all vertices forming a cycle
     * Verification: check it's a valid cycle visiting all vertices  O(V)
     */
    public static boolean verifyHamCycle(int[][] graph, int[] cycle) {
        int V = graph.length;
        if (cycle.length != V) return false;

        // Check all vertices appear exactly once
        boolean[] seen = new boolean[V];
        for (int v : cycle) {
            if (v < 0 || v >= V || seen[v]) return false;
            seen[v] = true;
        }

        // Check each consecutive pair has an edge, including wrap-around
        for (int i = 0; i < V; i++) {
            int u = cycle[i], v = cycle[(i+1) % V];
            if (graph[u][v] == 0) return false; // no edge
        }
        return true;
    }

    /**
     * SUBSET SUM VERIFIER
     * Problem: Given set S and target T, does any subset of S sum to T?
     * Certificate: the subset itself
     * Verification: check it's a subset and sums to T  O(n)
     */
    public static boolean verifySubsetSum(int[] set, int[] subset, int target) {
        int sum = 0;
        // Check subset elements are valid and count occurrences
        Map<Integer, Long> setCounts   = new HashMap<>();
        Map<Integer, Long> subCounts   = new HashMap<>();
        for (int x : set)    setCounts.merge(x, 1L, Long::sum);
        for (int x : subset) { sum += x; subCounts.merge(x, 1L, Long::sum); }

        // Every element in subset must appear in set (with correct multiplicity)
        for (Map.Entry<Integer, Long> e : subCounts.entrySet())
            if (setCounts.getOrDefault(e.getKey(), 0L) < e.getValue())
                return false;

        return sum == target;
    }
}
```

### P ⊆ NP

Every problem in P is also in NP: if you can *solve* a problem in polynomial time, you can certainly *verify* a solution in polynomial time (just re-solve it and compare).

```
P ⊆ NP        (proven — trivially)
P ≠ NP?       (unknown — the million-dollar question)
```

---

## 7.4 Polynomial-Time Reductions

A **polynomial-time reduction** from problem A to problem B (written A ≤_p B) is a polynomial-time computable function f such that:

```
x ∈ A  iff  f(x) ∈ B

Meaning: if we can solve B, we can solve A (using f as a preprocessor).
Informally: B is "at least as hard as" A.

If A ≤_p B and B ∈ P, then A ∈ P.
Contrapositive: if A ∉ P, then B ∉ P.
```

Reductions are the fundamental tool for comparing problem hardness.

### Example: 3-COLORING ≤_p 4-COLORING

If we could 4-color any graph, we could also 3-color it: add a special "4th color vertex" connected to nothing — a 4-coloring of the new graph that doesn't use the 4th color on original vertices is a 3-coloring.

### Example: INDEPENDENT-SET ≤_p CLIQUE

```java
/**
 * Reduction: INDEPENDENT-SET to CLIQUE
 *
 * INDEPENDENT-SET: Does graph G have an independent set of size k?
 *   (A set of k vertices with no edges between them)
 *
 * CLIQUE: Does graph G' have a clique of size k?
 *   (A set of k vertices all connected to each other)
 *
 * REDUCTION: Complement the graph!
 *   G' = complement of G (edges become non-edges, non-edges become edges)
 *   G has independent set of size k  iff  G' has clique of size k
 *
 * This shows INDEPENDENT-SET ≤_p CLIQUE (and vice versa — they're equivalent).
 */
public static int[][] complementGraph(int[][] graph) {
    int n = graph.length;
    int[][] comp = new int[n][n];
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++)
            if (i != j) comp[i][j] = 1 - graph[i][j]; // flip edges
    return comp;
}

/**
 * VERTEX-COVER ≤_p INDEPENDENT-SET
 *
 * S is a vertex cover (every edge has at least one endpoint in S)
 * iff V\S is an independent set (no edge between complement vertices)
 *
 * G has vertex cover of size k  iff  G has independent set of size n-k
 */
public static boolean hasIndependentSetOfSize(int[][] graph, int k) {
    // Reduction: check vertex cover of size n-k instead
    int n = graph.length;
    return hasVertexCoverOfSize(graph, n - k); // forward to VC solver
}

private static boolean hasVertexCoverOfSize(int[][] graph, int k) {
    // Placeholder — in practice we'd call a VC algorithm
    return true; // simplified
}
```

---

## 7.5 NP-Completeness

### Definition

A problem L is **NP-complete** if:
1. L ∈ NP  (solutions are verifiable in polynomial time)
2. Every problem in NP reduces to L in polynomial time (L is NP-hard)

```
NP-complete = the hardest problems in NP.

If ANY NP-complete problem is in P, then P = NP.
(Because every NP problem reduces to it, and reductions compose.)
```

### Cook-Levin Theorem (1971)

Stephen Cook (1971) and Leonid Levin (1973) independently proved:

> **3-SAT is NP-complete.**

This was the first NP-complete problem. The proof shows that any NP problem's nondeterministic verification can be encoded as a Boolean formula.

```
Proof sketch (Cook-Levin):
  For any NP language L with verifier V:
  On input x, construct a Boolean formula φ_{x,V} that is satisfiable
  iff V(x, c) accepts for some certificate c.

  φ_{x,V} encodes:
    - The initial configuration of V's tape
    - The transition function of V
    - The accepting configuration
  
  All three can be encoded in polynomial size using CNF clauses.
  A satisfying assignment = a valid computation history of V.
```

### The NP-Completeness Proof Template

To prove problem B is NP-complete, show:

```
Step 1: B ∈ NP
  Describe a polynomial-time verifier:
  "Given certificate c, we can verify it is a valid solution in O(poly) time."

Step 2: B is NP-hard (some known NP-complete A ≤_p B)
  Describe a polynomial-time reduction from A to B:
  "Given any instance x of A, construct instance f(x) of B in polynomial time,
   such that x ∈ A iff f(x) ∈ B."
```

### Classic NP-Complete Problems

```
The most important NP-complete problems, with the reductions that prove them:

3-SAT (Cook-Levin: fundamental)
  ↓
INDEPENDENT-SET (reduction from 3-SAT)
  ↓                    ↓
CLIQUE              VERTEX-COVER
  ↓
GRAPH-COLORING (3-coloring)

3-SAT
  ↓
SUBSET-SUM
  ↓
0/1-KNAPSACK

HAMILTONIAN-PATH
  ↓
TRAVELING-SALESMAN (decision: TSP ≤ k?)
```

### Reduction: 3-SAT → INDEPENDENT-SET

This is one of the most elegant reductions in all of computer science.

```
Given 3-SAT instance φ with m clauses and n variables:
  Construct graph G = (V, E) as follows:

  VERTICES: one vertex per literal occurrence (3m vertices total)
  EDGES: connect two vertices if:
    (a) They are in the SAME CLAUSE (clause conflict edges), OR
    (b) They are COMPLEMENTARY literals (x_i and ¬x_i)

  CLAIM: φ is satisfiable  iff  G has an independent set of size m.

  WHY: An independent set of size m must pick exactly one vertex per clause
       (clauses are triangles, IS can have at most one per triangle).
       The selected vertex = the TRUE literal in each clause.
       No two selected vertices can be complementary (edge between them).
       So selected literals are consistent → valid assignment. ✓
```

```java
/**
 * Reduction: 3-SAT → INDEPENDENT-SET
 *
 * clauses: array of clauses, each clause is array of literals
 *          positive literal i means variable i
 *          negative literal -i means ¬variable_i
 * n: number of variables
 *
 * Returns adjacency matrix for the constructed graph G.
 * G has independent set of size clauses.length iff formula is satisfiable.
 */
public static int[][] reduceSATtoIS(int[][] clauses, int n) {
    int m       = clauses.length;
    int numVerts = 3 * m; // 3 literals per clause
    int[][] graph = new int[numVerts][numVerts];

    // Each literal at position (clause i, position j) is vertex 3*i + j

    for (int i = 0; i < m; i++) {
        for (int j = 0; j < 3; j++) {
            for (int k = 0; k < 3; k++) {
                if (j != k) {
                    // CLAUSE CONFLICT: two literals in the same clause
                    graph[3*i + j][3*i + k] = 1;
                }
            }
        }
    }

    // COMPLEMENTARY CONFLICT: x_i and ¬x_i in different clauses
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < 3; j++) {
            for (int i2 = 0; i2 < m; i2++) {
                if (i == i2) continue;
                for (int j2 = 0; j2 < 3; j2++) {
                    // If one literal is negation of the other
                    if (clauses[i][j] == -clauses[i2][j2]) {
                        graph[3*i + j][3*i2 + j2] = 1;
                    }
                }
            }
        }
    }
    return graph;
}
```

---

## 7.6 The Complexity Landscape

### The Major Classes

```
                    ┌────────────────────────────────────────┐
                    │              PSPACE                     │
                    │    ┌──────────────────────────────┐    │
                    │    │       NP ∪ co-NP              │    │
                    │    │  ┌──────────┐ ┌────────────┐  │    │
                    │    │  │    NP    │ │   co-NP    │  │    │
                    │    │  │  ┌────┐  │ │  ┌──────┐  │  │    │
                    │    │  │  │    │  │ │  │      │  │  │    │
                    │    │  │  │ P  │  │ │  │  P   │  │  │    │
                    │    │  │  │    │  │ │  │      │  │  │    │
                    │    │  │  └────┘  │ │  └──────┘  │  │    │
                    │    │  └──────────┘ └────────────┘  │    │
                    │    └──────────────────────────────────┘  │
                    └────────────────────────────────────────┘
                    (assuming P ≠ NP and NP ≠ co-NP)
```

### co-NP

> **co-NP** = the class of problems whose complement is in NP.

```
If L ∈ NP, then the complement L̄ ∈ co-NP.

NP: easy to verify YES instances
co-NP: easy to verify NO instances

Example:
  TAUTOLOGY ∈ co-NP: "Is every assignment to φ a satisfying one?"
    A NO certificate: one falsifying assignment (easy to verify)
    But verifying YES requires checking all 2^n assignments → hard

  PRIME (is n prime?): surprisingly, PRIME ∈ NP ∩ co-NP
    YES certificate: small witness (Pratt certificate)
    NO certificate: a non-trivial factor
```

### PSPACE

> **PSPACE** = problems solvable using polynomial space (any amount of time).

```
P ⊆ NP ⊆ PSPACE ⊆ EXP

PSPACE-complete examples:
  - Quantified Boolean Formulas (QBF): ∃x∀y∃z φ(x,y,z) = TRUE?
  - TQBF is the canonical PSPACE-complete problem
  - Game-tree evaluation (2-player perfect information games)
  - Generalized Chess/Go/Othello → PSPACE-complete

Space vs. Time: Savitch's Theorem:
  NSPACE(f(n)) ⊆ DSPACE(f(n)²)
  → Nondeterministic space can be simulated by deterministic space
    with only a quadratic blow-up (much better than the exponential
    blow-up for time)
```

### EXP and Beyond

```
EXP  = solvable in exponential time O(2^poly(n))

Known separations:
  P ⊊ EXP           (proven by time hierarchy theorem)
  NP ⊆ EXP          (trivially — try all certificates)
  P ≠ NP or NP ≠ EXP (at least one must hold — but which?)

Undecidable problems (no TM can decide them):
  HALTING PROBLEM: "Does TM M halt on input x?"
  MEMBERSHIP: "Does TM M accept input x?"
  These are outside all complexity classes — no algorithm, period.
```

---

## 7.7 The P vs. NP Question

This is the central open question of computer science and mathematics, listed as one of the Clay Millennium Prize Problems (worth $1,000,000).

### What P = NP Would Mean

```
IF P = NP:
  ✓ Every problem whose solution can be verified quickly
    can also be SOLVED quickly.

  Consequences:
  + Perfect optimization: scheduling, logistics, routing → polynomial time
  + Cryptography BROKEN: RSA, AES → factoring in polynomial time
  + Drug discovery: protein folding, molecular design → polynomial time
  + Mathematical proofs: automated theorem proving → polynomial time
  + Artificial intelligence: planning, learning → polynomial time

  Most computer scientists believe P ≠ NP.
  The "naturalness" of NP-complete problems' apparent hardness
  is strong (though non-rigorous) evidence.
```

### What P ≠ NP Would Mean

```
IF P ≠ NP (widely believed):
  ✓ No polynomial-time algorithm for NP-complete problems exists.
  ✓ Cryptography remains secure (computationally).
  ✓ Many optimization problems are provably hard — use approximation.

  But proving this seems extraordinarily difficult:
  We would need to show that NO polynomial-time algorithm can solve
  NP-complete problems — ruling out all possible algorithms.

  Current barrier: "Natural Proofs" (Razborov-Rudich 1994)
  Most known proof techniques are insufficient.
```

### The Relativization Barrier

```
Baker-Gill-Solovay (1975):
  There exists an oracle A such that P^A = NP^A.
  There exists an oracle B such that P^B ≠ NP^B.

Interpretation: "Relativizing" proof techniques cannot resolve P vs. NP.
Any proof must use non-relativizing arguments.

Known non-relativizing results: IP = PSPACE (interactive proofs)
```

---

## 7.8 Proving NP-Completeness: Worked Examples

### Example 1: VERTEX-COVER is NP-Complete

**Problem:** Does graph G have a vertex cover of size ≤ k? (A set of ≤ k vertices that touches every edge.)

```
Step 1: VERTEX-COVER ∈ NP
  Certificate: the set S of vertices (size ≤ k)
  Verifier: check |S| ≤ k and every edge has at least one endpoint in S
  Time: O(k + |E|) = polynomial ✓

Step 2: INDEPENDENT-SET ≤_p VERTEX-COVER
  (We already know INDEPENDENT-SET is NP-hard via 3-SAT reduction)

  KEY FACT: S is a vertex cover iff V\S is an independent set.
  PROOF:
    → If S covers all edges, then no edge has BOTH endpoints in V\S.
      (Otherwise that edge would not be covered.) So V\S is an IS.
    ← If V\S is an IS, every edge has at least one endpoint NOT in V\S,
      meaning at least one endpoint in S. So S is a VC.

  REDUCTION f: ⟨G, k⟩ ∈ INDEPENDENT-SET
           → ⟨G, n-k⟩ ∈ VERTEX-COVER
  (Same graph, complement target size)

  ⟨G,k⟩ ∈ IS  iff  G has IS of size k
               iff  G has VC of size n-k
               iff  ⟨G, n-k⟩ ∈ VERTEX-COVER  ✓
```

```java
/**
 * Demonstrating the IS ↔ VC relationship.
 */
public class IStoVC {

    // Check if S is an independent set
    public static boolean isIndependentSet(int[][] graph, Set<Integer> S) {
        for (int u : S)
            for (int v : S)
                if (u != v && graph[u][v] == 1) return false; // edge within S
        return true;
    }

    // Check if S is a vertex cover
    public static boolean isVertexCover(int[][] graph, Set<Integer> S) {
        int n = graph.length;
        for (int u = 0; u < n; u++)
            for (int v = u+1; v < n; v++)
                if (graph[u][v] == 1 && !S.contains(u) && !S.contains(v))
                    return false; // uncovered edge
        return true;
    }

    // Reduction: IS of size k ↔ VC of size n-k
    public static boolean hasCoverViaIS(int[][] graph, int k) {
        int n = graph.length;
        // Find IS of size n-k (using any IS algorithm)
        // If exists, complement is a VC of size k
        return hasISOfSize(graph, n - k); // call IS solver
    }

    private static boolean hasISOfSize(int[][] graph, int k) {
        // Placeholder for IS solver (NP-hard in general)
        return false;
    }
}
```

### Example 2: SUBSET-SUM is NP-Complete

**Problem:** Given integers S = {s₁, ..., sₙ} and target T, does any subset sum to T?

```
Step 1: SUBSET-SUM ∈ NP
  Certificate: the subset itself
  Verifier: check elements are in S and sum to T  O(n) ✓

Step 2: 3-SAT ≤_p SUBSET-SUM
  (More complex reduction — key ideas:)

  For each variable xᵢ: create two numbers vᵢ (for xᵢ = TRUE)
                                        and v̄ᵢ (for xᵢ = FALSE)

  For each clause cⱼ: create two "slack" numbers sⱼ and s̄ⱼ

  Target T: constructed so that:
    - Selecting exactly one of vᵢ or v̄ᵢ for each variable
    - Plus appropriate slack variables
    sums to T  iff  the assignment satisfies all clauses.

  (Full reduction is in CLRS Chapter 34.5.4)
```

---

## 7.9 Space Complexity

### PSPACE and NPSPACE

```
PSPACE  = problems decidable in O(poly(n)) space, any time
NPSPACE = problems decidable in O(poly(n)) space nondeterministically

Savitch's Theorem: NPSPACE = PSPACE
  (Space is more "powerful" than time in this sense — nondeterminism
   gives only quadratic overhead for space, vs. potentially exponential for time)
```

### Space Hierarchy Theorem

```
For space-constructible functions f and g where f(n) = o(g(n)):
  DSPACE(f(n)) ⊊ DSPACE(g(n))

This is proven by diagonalization — the same technique as the Time Hierarchy Theorem.
```

### Savitch's Algorithm (STCON in PSPACE)

```
STCON (s-t connectivity): Is vertex t reachable from s in directed graph G?

Trivially O(V+E) time and O(V) space.
But Savitch's result is about GENERAL PSPACE problems:
  any NPSPACE problem can be solved in DSPACE(f(n)²).

For STCON specifically: O(log²n) space is achievable (STCON ∈ L²).
```

---

## 7.10 The Time Hierarchy Theorem

The Time Hierarchy Theorem proves that more time genuinely enables more computation — there are problems that require super-polynomial time.

```
Theorem (Time Hierarchy):
  For time-constructible functions t₁ and t₂ where t₁(n) log t₁(n) = o(t₂(n)):
  DTIME(t₁(n)) ⊊ DTIME(t₂(n))

  In particular: P ⊊ EXP  (proven!)

Proof technique: DIAGONALIZATION
  Construct a language L that "diagonalizes" against all t₁(n)-time machines:
  For each machine Mᵢ, make L disagree with Mᵢ on input i.
  L is in DTIME(t₂(n)) but not DTIME(t₁(n)).

  This is the same diagonal argument Cantor used to show the reals
  are uncountable, and Gödel used in his incompleteness theorems.
```

```java
/**
 * Diagonalization — conceptual illustration.
 *
 * The Halting Problem is undecidable by diagonalization:
 * Suppose HALT(M, x) decides whether M halts on x.
 * Construct D(M): if HALT(M, M) = YES → loop forever
 *                 if HALT(M, M) = NO  → halt
 * 
 * Now ask: does D halt on D?
 *   If D halts on D → HALT(D,D) = YES → D loops → contradiction
 *   If D loops on D → HALT(D,D) = NO  → D halts → contradiction
 *
 * This contradiction proves HALT cannot exist.
 * The same idea proves the Time Hierarchy Theorem.
 */
public class DiagonalizationIllustration {

    // A 2-second time limit verifier — accepts if computation finishes in 2 steps
    static boolean simulateWithTimeLimit(String description, int steps) {
        // In theory: simulate TM on its own description, limited to `steps` steps
        // If it accepts within the limit → accept; else reject (diagonalize)
        return description.length() < steps; // simplified illustration
    }
}
```

---

## 7.11 NP-Intermediate Problems

If P ≠ NP, there might be problems that are in NP but neither in P nor NP-complete. Ladner's Theorem (1975) proves this:

```
Ladner's Theorem: If P ≠ NP, then there exist problems in NP \ P
                  that are not NP-complete.

These are called NP-intermediate problems.

Candidate NP-intermediate problems:
  FACTORING: Given n, does n have a factor in [2, k]?
    - In NP: certificate is the factor
    - Not known to be in P (no poly-time classical algorithm known)
    - Not known to be NP-complete (factoring NP-hard would imply
      coNP = NP, which is widely disbelieved)
    → Widely believed to be NP-intermediate
    → Quantum: Shor's algorithm solves it in O(log³ n) — P on quantum computers

  GRAPH ISOMORPHISM: Are graphs G₁ and G₂ isomorphic?
    - In NP: certificate is the isomorphism mapping
    - Not known to be in P
    - Not known to be NP-complete
    → Quasipolynomial algorithm exists: O(n^polylog(n)) [Babai 2015]
    → Widely believed to be NP-intermediate
```

---

## 7.12 Complexity Classes Summary

| Class    | Informal Definition                          | Complete Problem      | Known Relations              |
|----------|----------------------------------------------|-----------------------|------------------------------|
| P        | Solvable in polynomial time                  | Circuit Value Problem | P ⊆ NP ⊆ PSPACE ⊆ EXP      |
| NP       | Verifiable in polynomial time                | 3-SAT                 | P ⊆ NP (P = NP?)            |
| co-NP    | Complement is in NP                          | TAUTOLOGY             | NP ∩ co-NP ⊇ P              |
| PSPACE   | Solvable in polynomial space                 | TQBF                  | NP ⊆ PSPACE (NP = PSPACE?)  |
| EXP      | Solvable in exponential time                 | —                     | P ⊊ EXP (proven!)           |
| NEXP     | Verifiable in exponential time               | —                     | NP ⊆ NEXP                   |
| L        | Solvable in logarithmic space                | STCON (undirected)    | L ⊆ NL ⊆ P                  |
| NL       | Nondeterministic log space                   | STCON (directed)      | NL = co-NL (Immerman-Szelepcsényi) |

---

## 7.13 Reductions Catalogue

```
Known NP-complete reductions (a partial family tree):

CIRCUIT-SAT (Cook-Levin basis)
      ↓
   3-SAT
  /  |   \   \
IS   VC   3-COLOR  SUBSET-SUM
|    |              |
CLIQUE  HAM-PATH   KNAPSACK
         |
       HAM-CYCLE
         |
        TSP
```

```java
/**
 * Summary: Recognizing NP-complete problems in the wild.
 *
 * When you encounter a new problem, ask:
 * 1. Does it look like "find the best subset/arrangement"?
 * 2. Is the search space exponential (2^n, n!, ...)?
 * 3. Can solutions be verified quickly?
 *
 * If YES to all: likely NP-hard. Look for:
 *   a) Exact algorithm (B&B, DP) — for small n
 *   b) Approximation algorithm — for large n (Module 8)
 *   c) Special structure that makes it easier (e.g., fixed parameter tractability)
 */
public class RecognizeNPHard {

    // These problems are NP-complete:
    static String[] npComplete = {
        "0/1 Knapsack (decision: value ≥ V?)",
        "3-SAT",
        "Graph 3-Coloring",
        "Hamiltonian Cycle",
        "Traveling Salesman (decision: tour ≤ k?)",
        "Vertex Cover (size ≤ k?)",
        "Independent Set (size ≥ k?)",
        "Clique (size ≥ k?)",
        "Subset Sum",
        "Bin Packing",
        "Job Shop Scheduling",
        "Set Cover"
    };

    // These related problems ARE in P:
    static String[] inP = {
        "Fractional Knapsack → Greedy O(n log n)",
        "2-SAT → Strongly Connected Components O(n+m)",
        "Graph 2-Coloring (Bipartite check) → BFS O(V+E)",
        "Eulerian Cycle → O(V+E)",
        "Shortest Path (single source) → Dijkstra O((V+E)log V)",
        "Min Spanning Tree → Kruskal/Prim O(E log V)",
        "Maximum Matching (bipartite) → Max-flow O(VE)",
        "Min Vertex Cover (bipartite) → König's theorem O(VE)"
    };
}
```

---

## 7.14 Key Takeaways

| Concept                  | Summary                                                                          |
|--------------------------|----------------------------------------------------------------------------------|
| Decision problem         | Yes/no question; formalized as a language L ⊆ Σ*                               |
| P                        | Solvable in polynomial time; problems we consider "efficiently solvable"         |
| NP                       | Solutions verifiable in polynomial time; contains P                             |
| co-NP                    | Complement of NP; NO instances verifiable; P ⊆ NP ∩ co-NP                      |
| Polynomial reduction A≤_pB | If B ∈ P then A ∈ P; B is "at least as hard as" A                           |
| NP-hard                  | Every NP problem reduces to it; at least as hard as hardest NP problems         |
| NP-complete              | NP-hard AND in NP; the "hardest" problems in NP                                 |
| Cook-Levin Theorem       | 3-SAT is the first and canonical NP-complete problem                            |
| P vs. NP                 | Greatest open question; if P=NP, all NP problems efficiently solvable           |
| PSPACE                   | Polynomial space; NP ⊆ PSPACE; Savitch: NPSPACE = PSPACE                       |
| Time Hierarchy Theorem   | P ⊊ EXP — more time gives strictly more power (proven by diagonalization)       |
| Ladner's Theorem         | If P≠NP, NP-intermediate problems exist (e.g., possibly FACTORING, GI)          |
| NP-completeness proof    | Show ∈ NP + show NP-hard via reduction from known NP-complete problem            |

---

## 7.15 Practice Problems

1. **Prove NP membership** — Formally prove that GRAPH 3-COLORING ∈ NP by describing an explicit polynomial-time verifier. What is the certificate? What does the verifier check? What is its time complexity?

2. **Reduction design** — Prove that CLIQUE ≤_p INDEPENDENT-SET by describing the reduction function f, proving correctness (x ∈ CLIQUE iff f(x) ∈ IS), and analyzing its time complexity.

3. **3-SAT instance** — For the formula φ = (x₁ ∨ ¬x₂ ∨ x₃) ∧ (¬x₁ ∨ x₂ ∨ x₄) ∧ (x₂ ∨ ¬x₃ ∨ ¬x₄):
   a) Construct the graph G used in the 3-SAT → INDEPENDENT-SET reduction.
   b) Find an independent set of size 3 in G.
   c) Read off the satisfying assignment from the independent set.

4. **Complexity classification** — For each of the following problems, classify as P or NP-complete and justify: (a) Find the minimum spanning tree. (b) Find the minimum-weight Hamiltonian cycle. (c) Determine if a graph is bipartite. (d) Determine if a graph has a clique of size n/2.

5. **PSPACE vs. NP** — The problem QBF-2 is: given a quantified Boolean formula ∃x₁∀x₂ φ(x₁,x₂), is it TRUE? Prove QBF-2 ∈ PSPACE by describing an algorithm that uses polynomial space. Can you also bound the time? Does QBF-2 ∈ NP? Explain.

---

## 7.16 Further Reading

- *Introduction to Algorithms* (CLRS) — Chapter 34 (NP-Completeness)
- *Computational Complexity: A Modern Approach* — Arora & Barak (comprehensive)
- *Introduction to the Theory of Computation* — Sipser (accessible introduction)
- *Computers and Intractability* — Garey & Johnson (the classic NP-completeness reference, 318 NP-complete problems listed)
- Scott Aaronson's blog (Shtetl-Optimized): complexity theory insights
- Complexity Zoo: [https://complexityzoo.net](https://complexityzoo.net) — complete catalogue of complexity classes

---

*CS 3304 Analysis of Algorithms — Module 7 of 8*
*Previous: Module 6 — Optimization Techniques*
*Next: Module 8 — Approximation Algorithms*
