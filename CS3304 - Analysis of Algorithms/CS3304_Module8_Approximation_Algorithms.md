# CS 3304 — Analysis of Algorithms
## Module 8: Approximation Algorithms

---

## 8.1 Introduction

Module 7 established that NP-complete problems almost certainly have no polynomial-time exact algorithms. But these problems — TSP, Knapsack, Set Cover, Vertex Cover — arise constantly in practice. We cannot simply declare them unsolvable and move on.

**Approximation algorithms** offer a principled middle ground:

> Run in **polynomial time** and produce a solution guaranteed to be within a **proven factor** of the optimal solution.

This is fundamentally different from heuristics (which may work well but offer no guarantees) and from metaheuristics (which converge slowly and give no worst-case bound). An approximation algorithm comes with a mathematical certificate of quality.

The **approximation ratio** ρ(n) of an algorithm is the worst-case ratio between the algorithm's solution and the optimal solution:

```
For minimization: ALG(I) ≤ ρ · OPT(I)  for all instances I
For maximization: ALG(I) ≥ (1/ρ) · OPT(I)  for all instances I

ρ = 1      → exact algorithm
ρ = 1.5    → solution is at most 50% worse than optimal
ρ = 2      → solution is at most twice the optimal cost
ρ = ln(n)  → solution grows logarithmically relative to optimal
```

By the end of this module you will:
- Understand the approximation ratio and how to prove it
- Implement and analyze seven classic approximation algorithms
- Understand PTAS, FPTAS, and APX — the approximability hierarchy
- Know which problems admit good approximations and which do not (inapproximability)
- Connect approximation to LP relaxation and randomized rounding

---

## 8.2 Vertex Cover — 2-Approximation

**Problem:** Find the minimum vertex cover — a set of vertices that touches every edge.

**Greedy idea:** Repeatedly pick the vertex with the highest degree. This is a heuristic but has no proven guarantee. Instead, a simple 2-approximation:

### The Matching-Based 2-Approximation

```
Algorithm:
  C = empty set
  while there are uncovered edges:
    pick any uncovered edge (u, v)
    add BOTH u and v to C
    remove all edges incident to u or v

CLAIM: |C| ≤ 2 · |OPT|
```

```java
/**
 * Vertex Cover — 2-Approximation.
 *
 * CORRECTNESS: C covers all edges (by construction — we add both
 *              endpoints of every edge we pick).
 *
 * APPROXIMATION RATIO PROOF:
 *   Let M = the set of edges we picked (a matching — no shared endpoints).
 *   Every vertex cover must include at least one endpoint of each edge in M.
 *   Since M is a matching, these requirements are DISJOINT.
 *   Therefore |OPT| ≥ |M|.
 *   Our algorithm adds exactly 2|M| vertices → |C| = 2|M| ≤ 2·|OPT|. ✓
 *
 * Time:  O(V + E)
 * Space: O(V)
 * Ratio: 2  (tight — examples exist where ratio = 2 - ε for any ε > 0)
 */
public static Set<Integer> vertexCover2Approx(int V, List<int[]> edges) {
    Set<Integer> cover   = new HashSet<>();
    boolean[]   covered  = new boolean[edges.size()];

    for (int i = 0; i < edges.size(); i++) {
        if (covered[i]) continue;          // already covered

        int u = edges.get(i)[0];
        int v = edges.get(i)[1];

        // GREEDY CHOICE: take both endpoints of this uncovered edge
        cover.add(u);
        cover.add(v);

        // Mark all edges incident to u or v as covered
        for (int j = i + 1; j < edges.size(); j++) {
            int a = edges.get(j)[0], b = edges.get(j)[1];
            if (a == u || a == v || b == u || b == v) covered[j] = true;
        }
    }
    return cover;
}

public static void main(String[] args) {
    // Graph: 0-1, 1-2, 2-3, 3-4, 4-0, 0-3
    List<int[]> edges = List.of(
        new int[]{0,1}, new int[]{1,2}, new int[]{2,3},
        new int[]{3,4}, new int[]{4,0}, new int[]{0,3}
    );

    Set<Integer> vc = vertexCover2Approx(5, edges);
    System.out.println("2-Approx Vertex Cover: " + vc);
    System.out.println("Size: " + vc.size() + "  (OPT ≥ size/2)");
}
```

---

## 8.3 Set Cover — O(ln n) Approximation

**Problem:** Given a universe U of n elements and a collection of sets S₁, ..., Sₘ, find the minimum number of sets whose union covers U.

**Greedy:** Always pick the set that covers the most uncovered elements.

```
Example:
U = {1,2,3,4,5,6,7}
Sets: S1={1,2,3,4}, S2={2,4,6,7}, S3={1,4,5}, S4={3,5,6,7}

Iteration 1: S1 covers 4 new elements → pick S1
Iteration 2: S4 covers 3 new elements (3,5,6,7? wait: 3 covered) → {5,6,7} → pick S2 or S4
...
```

```java
/**
 * Set Cover — Greedy O(ln n) Approximation.
 *
 * APPROXIMATION RATIO PROOF:
 *   Let OPT = k (optimal uses k sets).
 *   After any iteration, if m elements remain uncovered, some set in OPT
 *   covers at least m/k of them (pigeonhole). The greedy set covers
 *   at least as many, so after each step the uncovered count decreases
 *   by a factor of at least (1 - 1/k).
 *
 *   After t iterations: uncovered ≤ n·(1 - 1/k)^t ≤ n·e^(-t/k)
 *   Setting t = k·ln(n): uncovered < 1 → all elements covered.
 *
 *   Therefore greedy uses at most k·ln(n) = OPT·ln(n) sets.
 *   Approximation ratio: H(n) = ln(n) + O(1) ≈ ln(n)  ✓
 *
 * INAPPROXIMABILITY:
 *   Unless P = NP, no polynomial algorithm can achieve ratio < (1-ε)·ln(n)
 *   for any ε > 0. So greedy is essentially OPTIMAL among approximation algorithms!
 *
 * Time:  O(n * m * |max set|) per iteration
 * Space: O(n + m)
 */
public static List<Integer> setCoverGreedy(int n, List<Set<Integer>> sets) {
    Set<Integer> uncovered = new HashSet<>();
    for (int i = 1; i <= n; i++) uncovered.add(i); // universe

    List<Integer>  chosen  = new ArrayList<>();
    boolean[]      used    = new boolean[sets.size()];

    while (!uncovered.isEmpty()) {
        // GREEDY CHOICE: find set covering most uncovered elements
        int bestSet   = -1;
        int bestCount = 0;

        for (int i = 0; i < sets.size(); i++) {
            if (used[i]) continue;
            int count = 0;
            for (int elem : sets.get(i)) if (uncovered.contains(elem)) count++;
            if (count > bestCount) { bestCount = count; bestSet = i; }
        }

        if (bestSet == -1) break; // remaining elements uncoverable

        // Commit to this set
        chosen.add(bestSet + 1); // 1-indexed for display
        used[bestSet] = true;
        uncovered.removeAll(sets.get(bestSet));

        System.out.printf("Picked set %d (covers %d new elements)%n",
                          bestSet + 1, bestCount);
    }
    return chosen;
}

/**
 * Verify the approximation ratio guarantee empirically.
 */
public static void demonstrateRatio(int n, List<Set<Integer>> sets, int opt) {
    List<Integer> greedy = setCoverGreedy(n, sets);
    double ratio  = (double) greedy.size() / opt;
    double logN   = Math.log(n);
    System.out.printf("Greedy: %d sets, OPT: %d, ratio: %.2f, ln(%d)=%.2f%n",
                      greedy.size(), opt, ratio, n, logN);
}
```

---

## 8.4 Traveling Salesman Problem (TSP) Approximations

### Metric TSP — 2-Approximation via MST

If edge weights satisfy the **triangle inequality** (d(u,v) ≤ d(u,w) + d(w,v)), we can approximate TSP using a minimum spanning tree.

```
KEY INSIGHT:
  1. The optimal TSP tour has cost ≥ MST cost
     (removing any edge from a tour gives a spanning tree)
  2. A full walk of the MST visits every edge twice → cost = 2·MST
  3. Shortcutting the walk (skipping revisited vertices) never increases
     cost (by triangle inequality)
  4. Therefore: algorithm cost ≤ 2·MST ≤ 2·OPT
```

```java
/**
 * Metric TSP — 2-Approximation (MST-based).
 *
 * ALGORITHM:
 *   1. Build MST of the graph
 *   2. Double all MST edges (creating an Eulerian multigraph)
 *   3. Find Euler circuit of doubled MST
 *   4. Shortcut to Hamiltonian cycle (skip revisited vertices)
 *
 * APPROXIMATION RATIO: 2 (tight for metric TSP)
 *
 * Time:  O(V² log V) or O(E log V) with efficient MST
 */
public static List<Integer> tsp2Approx(double[][] dist) {
    int V = dist.length;

    // Step 1: Build MST using Prim's algorithm
    boolean[] inMST  = new boolean[V];
    double[]  key    = new double[V];
    int[]     parent = new int[V];
    Arrays.fill(key,    Double.MAX_VALUE);
    Arrays.fill(parent, -1);
    key[0] = 0;

    PriorityQueue<int[]> pq = new PriorityQueue<>(Comparator.comparingDouble(a -> key[a[0]]));
    pq.offer(new int[]{0});

    List<List<Integer>> mstAdj = new ArrayList<>();
    for (int i = 0; i < V; i++) mstAdj.add(new ArrayList<>());

    while (!pq.isEmpty()) {
        int u = pq.poll()[0];
        if (inMST[u]) continue;
        inMST[u] = true;

        if (parent[u] != -1) {
            mstAdj.get(parent[u]).add(u);
            mstAdj.get(u).add(parent[u]); // undirected MST
        }

        for (int v = 0; v < V; v++) {
            if (!inMST[v] && dist[u][v] < key[v]) {
                key[v]    = dist[u][v];
                parent[v] = u;
                pq.offer(new int[]{v});
            }
        }
    }

    // Step 2: DFS pre-order traversal of MST → gives shortcut tour
    List<Integer> tour    = new ArrayList<>();
    boolean[]     visited = new boolean[V];

    dfsPreOrder(0, mstAdj, visited, tour);
    tour.add(tour.get(0)); // return to start

    return tour;
}

private static void dfsPreOrder(int u, List<List<Integer>> adj,
                                 boolean[] visited, List<Integer> tour) {
    visited[u] = true;
    tour.add(u);
    for (int v : adj.get(u)) if (!visited[v]) dfsPreOrder(v, adj, visited, tour);
}

static double tourCost(List<Integer> tour, double[][] dist) {
    double cost = 0;
    for (int i = 0; i < tour.size() - 1; i++) cost += dist[tour.get(i)][tour.get(i+1)];
    return cost;
}
```

### Christofides Algorithm — 1.5-Approximation

The best known polynomial-time approximation for metric TSP achieves ratio 1.5:

```
CHRISTOFIDES ALGORITHM:
  1. Build MST T                          (cost ≤ OPT)
  2. Find vertices of ODD DEGREE in T     (must be even number of them)
  3. Find MINIMUM WEIGHT PERFECT MATCHING M on odd-degree vertices
     (cost ≤ OPT/2 — since optimal TSP restricted to odd vertices costs ≤ OPT,
      and matching is half that)
  4. Combine T ∪ M into Eulerian multigraph
  5. Find Euler circuit, shortcut to Hamiltonian cycle

  Total cost ≤ MST + matching ≤ OPT + OPT/2 = 1.5 · OPT

NOTE (2022 breakthrough): Karlin, Klein, and Gharan proved a (1.5 - ε)
approximation exists for some ε > 0, breaking the 1.5 barrier after
47 years. The exact best ratio remains an open problem.
```

```java
/**
 * Christofides Algorithm — 1.5-Approximation for Metric TSP.
 *
 * Full implementation requires minimum weight perfect matching (Blossom algorithm).
 * Here we sketch the structure; matching is called as a subroutine.
 *
 * Time:  O(V³) — dominated by minimum weight perfect matching
 * Ratio: 1.5
 */
public static List<Integer> christofides(double[][] dist) {
    int V = dist.length;

    // Step 1: Build MST (Prim's, O(V²))
    int[] parent = primMST(dist, V);
    List<List<int[]>> mstAdj = buildMSTAdj(parent, dist, V);

    // Step 2: Find odd-degree vertices
    int[] degree = new int[V];
    for (int u = 0; u < V; u++) degree[u] = mstAdj.get(u).size();
    List<Integer> oddVertices = new ArrayList<>();
    for (int u = 0; u < V; u++) if (degree[u] % 2 != 0) oddVertices.add(u);

    System.out.println("Odd-degree vertices: " + oddVertices);

    // Step 3: Min-weight perfect matching on odd vertices
    // (In practice: use Blossom algorithm; here we use greedy matching)
    List<int[]> matching = greedyMatching(oddVertices, dist);

    // Step 4: Combine MST edges + matching edges → Eulerian multigraph
    List<List<Integer>> multiAdj = new ArrayList<>();
    for (int i = 0; i < V; i++) multiAdj.add(new ArrayList<>());
    for (int u = 0; u < V; u++) for (int[] e : mstAdj.get(u)) multiAdj.get(u).add(e[0]);
    for (int[] e : matching) { multiAdj.get(e[0]).add(e[1]); multiAdj.get(e[1]).add(e[0]); }

    // Step 5: Euler circuit → shortcut to Hamiltonian cycle
    List<Integer> euler   = eulerCircuit(multiAdj, V);
    List<Integer> tour    = shortcut(euler, V);
    tour.add(tour.get(0));
    return tour;
}

private static int[] primMST(double[][] dist, int V) {
    double[] key = new double[V]; int[] parent = new int[V]; boolean[] vis = new boolean[V];
    Arrays.fill(key, Double.MAX_VALUE); key[0] = 0; parent[0] = -1;
    for (int iter = 0; iter < V; iter++) {
        int u = -1;
        for (int v = 0; v < V; v++) if (!vis[v] && (u == -1 || key[v] < key[u])) u = v;
        vis[u] = true;
        for (int v = 0; v < V; v++)
            if (!vis[v] && dist[u][v] < key[v]) { key[v] = dist[u][v]; parent[v] = u; }
    }
    return parent;
}

private static List<List<int[]>> buildMSTAdj(int[] parent, double[][] dist, int V) {
    List<List<int[]>> adj = new ArrayList<>();
    for (int i = 0; i < V; i++) adj.add(new ArrayList<>());
    for (int v = 1; v < V; v++) {
        int u = parent[v];
        adj.get(u).add(new int[]{v}); adj.get(v).add(new int[]{u});
    }
    return adj;
}

// Greedy matching (not minimum weight — for illustration)
private static List<int[]> greedyMatching(List<Integer> vertices, double[][] dist) {
    List<int[]> matching = new ArrayList<>();
    boolean[] matched = new boolean[vertices.size()];
    for (int i = 0; i < vertices.size(); i++) {
        if (matched[i]) continue;
        double best = Double.MAX_VALUE; int bestJ = -1;
        for (int j = i+1; j < vertices.size(); j++) {
            if (!matched[j] && dist[vertices.get(i)][vertices.get(j)] < best) {
                best = dist[vertices.get(i)][vertices.get(j)]; bestJ = j;
            }
        }
        if (bestJ != -1) {
            matching.add(new int[]{vertices.get(i), vertices.get(bestJ)});
            matched[i] = matched[bestJ] = true;
        }
    }
    return matching;
}

private static List<Integer> eulerCircuit(List<List<Integer>> adj, int V) {
    List<List<Integer>> copy = new ArrayList<>();
    for (List<Integer> l : adj) copy.add(new ArrayList<>(l));
    List<Integer> circuit = new ArrayList<>();
    Deque<Integer> stack = new ArrayDeque<>();
    stack.push(0);
    while (!stack.isEmpty()) {
        int u = stack.peek();
        if (copy.get(u).isEmpty()) { circuit.add(u); stack.pop(); }
        else { int v = copy.get(u).remove(0); copy.get(v).remove((Integer)u); stack.push(v); }
    }
    Collections.reverse(circuit);
    return circuit;
}

private static List<Integer> shortcut(List<Integer> euler, int V) {
    Set<Integer> seen = new LinkedHashSet<>();
    for (int v : euler) seen.add(v);
    return new ArrayList<>(seen);
}
```

---

## 8.5 Knapsack — FPTAS

**Problem:** 0/1 Knapsack — maximize value subject to weight constraint.

The Knapsack decision problem is NP-complete, but the optimization version admits a **Fully Polynomial-Time Approximation Scheme (FPTAS)**: for any ε > 0, find a solution of value ≥ (1−ε) · OPT in time O(n³/ε).

### The FPTAS Idea

The DP solution has complexity O(n · V_max). If V_max is large, this is pseudo-polynomial. The trick: **scale and round** all values to make V_max small.

```
ROUNDING SCHEME:
  K = ε · V_max / n  (scaling factor)
  v'[i] = floor(v[i] / K)  (rounded values — now at most n/ε each)

  New DP: O(n · (n/ε)) = O(n²/ε) states
  Total time: O(n³/ε) — fully polynomial in n and 1/ε!

  APPROXIMATION GUARANTEE:
  OPT' ≥ floor(OPT/K) · K ≥ OPT - n·K = OPT - ε·V_max ≥ (1-ε)·OPT
```

```java
/**
 * Knapsack FPTAS — (1-ε)-approximation.
 *
 * Time:  O(n² / ε) for the DP, O(n log n) for sorting
 * Space: O(n / ε) for DP table
 * Ratio: (1 - ε) for any ε > 0
 *
 * Key: this is BETTER than a PTAS because running time is polynomial
 * in BOTH n AND 1/ε (not just n for fixed ε).
 */
public static int knapsackFPTAS(int[] weights, int[] values, int W, double epsilon) {
    int n    = weights.length;
    int Vmax = Arrays.stream(values).max().getAsInt();

    // Scaling factor: round values down to reduce range
    double K = epsilon * Vmax / n;

    // Scale values
    int[] scaledValues = new int[n];
    for (int i = 0; i < n; i++) scaledValues[i] = (int)(values[i] / K);

    int scaledMax = Arrays.stream(scaledValues).sum();
    int[] dp = new int[scaledMax + 1]; // dp[v] = min weight to achieve value v
    Arrays.fill(dp, Integer.MAX_VALUE / 2);
    dp[0] = 0;

    // DP on scaled values: dp[v] = min weight to achieve scaled value v
    for (int i = 0; i < n; i++) {
        for (int v = scaledMax; v >= scaledValues[i]; v--) {
            if (dp[v - scaledValues[i]] + weights[i] <= W) {
                dp[v] = Math.min(dp[v], dp[v - scaledValues[i]] + weights[i]);
            }
        }
    }

    // Find maximum achievable scaled value within weight limit
    int bestScaled = 0;
    for (int v = scaledMax; v >= 0; v--) {
        if (dp[v] <= W) { bestScaled = v; break; }
    }

    // Convert back to approximate original value
    return (int)(bestScaled * K);
}

public static void main(String[] args) {
    int[] weights = {2, 3, 4, 5, 9};
    int[] values  = {3, 4, 5, 8, 10};
    int   W       = 10;

    // Exact DP
    int exact = knapsackExact(weights, values, W);

    // FPTAS with ε = 0.1
    int approx10 = knapsackFPTAS(weights, values, W, 0.1);
    // FPTAS with ε = 0.25
    int approx25 = knapsackFPTAS(weights, values, W, 0.25);

    System.out.println("Exact:          " + exact);
    System.out.println("FPTAS (ε=0.10): " + approx10 +
                       " (≥ " + (int)(0.9 * exact) + " guaranteed)");
    System.out.println("FPTAS (ε=0.25): " + approx25 +
                       " (≥ " + (int)(0.75 * exact) + " guaranteed)");
}

static int knapsackExact(int[] w, int[] v, int W) {
    int n = w.length; int[] dp = new int[W+1];
    for (int i = 0; i < n; i++)
        for (int j = W; j >= w[i]; j--)
            dp[j] = Math.max(dp[j], dp[j-w[i]] + v[i]);
    return dp[W];
}
```

---

## 8.6 Bin Packing — First Fit Decreasing (FFD)

**Problem:** Pack n items of sizes s₁, ..., sₙ ∈ (0,1] into the fewest bins of capacity 1.

```java
/**
 * Bin Packing — First Fit Decreasing (FFD) Algorithm.
 *
 * ALGORITHM:
 *   1. Sort items in DECREASING order of size
 *   2. For each item, place it in the FIRST bin it fits in
 *      (open a new bin if no existing bin has room)
 *
 * APPROXIMATION RATIO:
 *   FFD(I) ≤ (11/9)·OPT(I) + 6/9
 *   ≈ 1.222 · OPT  (for large instances)
 *
 * This is tight: there exist instances where FFD = (11/9)·OPT.
 *
 * Time:  O(n log n) + O(n · bins) = O(n²) worst case
 * Space: O(bins)
 */
public static int binPackingFFD(double[] sizes) {
    double[] sorted = sizes.clone();
    Arrays.sort(sorted);
    // Reverse for descending order
    for (int i = 0; i < sorted.length / 2; i++) {
        double t = sorted[i]; sorted[i] = sorted[sorted.length-1-i];
        sorted[sorted.length-1-i] = t;
    }

    List<Double> bins = new ArrayList<>(); // bins[i] = remaining capacity

    for (double item : sorted) {
        boolean placed = false;
        for (int b = 0; b < bins.size(); b++) {
            if (bins.get(b) >= item - 1e-9) {   // fits in existing bin
                bins.set(b, bins.get(b) - item);
                placed = true;
                break;
            }
        }
        if (!placed) {
            bins.add(1.0 - item);               // open new bin
        }
    }

    System.out.printf("FFD uses %d bins (OPT ≥ %.0f)%n",
                      bins.size(), Math.ceil(Arrays.stream(sizes).sum()));
    return bins.size();
}
```

---

## 8.7 MAX-SAT — Randomized 3/4-Approximation

**Problem:** Given a Boolean formula in CNF, find an assignment that satisfies the maximum number of clauses.

This is the **optimization** version of SAT (which is NP-complete). We can approximate it well using randomization.

```
RANDOMIZED ALGORITHM:
  Assign each variable TRUE or FALSE independently with probability 1/2.

ANALYSIS (k-clause with k distinct literals):
  P(clause satisfied) = 1 - (1/2)^k

  For k=1: 1/2   For k=2: 3/4   For k=3: 7/8

  For 3-CNF: E[clauses satisfied] ≥ (7/8) · m   where m = total clauses
  → This is a 7/8-approximation for MAX-3-SAT!

  INAPPROXIMABILITY: MAX-3-SAT cannot be approximated better than
  7/8 + ε for any ε > 0 unless P = NP (PCP Theorem consequence).
  So this randomized algorithm is OPTIMAL!
```

```java
/**
 * MAX-SAT Randomized Approximation.
 *
 * Time:  O(n * m) per trial — n variables, m clauses
 * Expected ratio: 7/8 for 3-CNF, (1 - 1/2^k) for k-CNF
 *
 * Derandomization: the method of conditional expectations gives
 * a DETERMINISTIC (1 - 1/2^k)-approximation.
 */
public static int maxSatRandom(int[][] clauses, int numVars, int trials) {
    Random rng  = new Random(42);
    int    best = 0;

    for (int t = 0; t < trials; t++) {
        // Random assignment
        boolean[] assignment = new boolean[numVars + 1];
        for (int i = 1; i <= numVars; i++) assignment[i] = rng.nextBoolean();

        // Count satisfied clauses
        int satisfied = 0;
        for (int[] clause : clauses) {
            for (int literal : clause) {
                boolean val = assignment[Math.abs(literal)];
                if (literal < 0) val = !val;
                if (val) { satisfied++; break; }
            }
        }
        best = Math.max(best, satisfied);
    }
    return best;
}

/**
 * Derandomized MAX-SAT via Method of Conditional Expectations.
 * Set each variable to the value that maximizes the conditional
 * expected number of satisfied clauses.
 *
 * Time:  O(n * m)   Ratio: (1 - 1/2^k) deterministically
 */
public static int maxSatDeterministic(int[][] clauses, int numVars) {
    boolean[] assignment = new boolean[numVars + 1];

    for (int var = 1; var <= numVars; var++) {
        // Try setting var = TRUE vs FALSE; pick whichever satisfies more clauses
        int trueCount  = countSatisfied(clauses, assignment, var, true);
        int falseCount = countSatisfied(clauses, assignment, var, false);
        assignment[var] = (trueCount >= falseCount);
    }
    return countSatisfied(clauses, assignment, -1, false);
}

private static int countSatisfied(int[][] clauses, boolean[] assignment,
                                   int fixVar, boolean fixVal) {
    int count = 0;
    outer:
    for (int[] clause : clauses) {
        for (int literal : clause) {
            int var = Math.abs(literal);
            boolean val;
            if (var == fixVar)     val = fixVal;
            else if (var < fixVar) val = assignment[var];
            else continue; // unset variable — treat as potentially satisfying
            if (literal < 0) val = !val;
            if (val) { count++; continue outer; }
        }
        // If we get here with no definite TRUE literal and some unset vars,
        // count conservatively (clause might still be satisfied)
        for (int literal : clause)
            if (Math.abs(literal) > fixVar) { count++; continue outer; }
    }
    return count;
}
```

---

## 8.8 Approximation Hierarchy

Problems can be classified by their **best achievable approximation ratio**:

### PTAS (Polynomial-Time Approximation Scheme)

> For every fixed ε > 0, a (1+ε)-approximation runs in polynomial time in n.
> (Time may depend exponentially on 1/ε.)

```
Examples:
  Euclidean TSP: PTAS with ratio (1+ε), running time O(n · (1/ε)^O(1/ε))
  Knapsack:      PTAS via scaling (becomes FPTAS)
  Bin Packing:   PTAS exists, no FPTAS known
```

### FPTAS (Fully Polynomial-Time Approximation Scheme)

> For every ε > 0, a (1+ε)-approximation runs in time polynomial in BOTH n AND 1/ε.

```
Examples:
  Knapsack: O(n³/ε) — shown in Section 8.5
  Subset Sum: O(n/ε) approximation

  If a problem is NP-hard AND has FPTAS, the DP is pseudo-polynomial.
  (Knapsack DP is O(nW) — polynomial in n and W, so FPTAS is possible.)
```

### APX

> The class of problems with a **constant-factor** polynomial approximation.

```
Examples:
  Vertex Cover: 2-APX
  MAX-3-SAT:    7/8-APX
  Metric TSP:   1.5-APX (Christofides)
  Set Cover:    ln(n)-APX  ← NOT in APX (ratio grows with n)
```

### Inapproximability

Some problems cannot be approximated beyond a certain ratio unless P = NP:

```
Problem         Best Achievable Ratio    Hardness Result
─────────────────────────────────────────────────────────────
Vertex Cover    2 - ε                    Unique Games Conjecture
Set Cover       (1-ε)·ln(n)             P ≠ NP (Feige 1998)
Max-3-SAT       7/8 + ε                 PCP Theorem (Håstad 2001)
TSP (general)   No constant factor      P ≠ NP
Clique          n^(1-ε)                 P ≠ NP (Zuckerman 2007)
Chromatic #     n^(1/2-ε)              P ≠ NP
```

```
The PCP Theorem (Probabilistically Checkable Proofs, 1992):
  Every NP proof can be verified by a probabilistic verifier that
  reads only O(log n) bits and makes O(1) random choices.

  Consequence: optimal approximation ratios exist for many NP-hard problems,
  and they are often achieved by simple greedy or rounding algorithms.
```

---

## 8.9 LP Relaxation and Randomized Rounding

A powerful general technique for approximation: formulate as an Integer LP, relax the integrality constraint, solve the LP, then round to integers.

```
GENERAL TEMPLATE:
  1. Write the problem as Integer LP (ILP)
  2. Relax: allow variables to be fractional (LP)
  3. Solve LP in polynomial time (Simplex or Interior Point)
  4. Round fractional solution to integer solution
  5. Prove rounding doesn't lose more than factor ρ

  Ratio ρ = (integer optimal) / (LP optimal)
          ≤ (rounded LP solution) / (LP optimal)
```

### LP Rounding for Weighted Vertex Cover

```java
/**
 * Weighted Vertex Cover via LP Relaxation + Rounding.
 *
 * ILP:
 *   Minimize Σ w[v] · x[v]
 *   Subject to: x[u] + x[v] ≥ 1  for each edge (u,v)
 *               x[v] ∈ {0, 1}
 *
 * LP RELAXATION: allow x[v] ∈ [0, 1]
 *
 * LP OPTIMAL: set x[v] = 1/2 for all v if graph has any edges.
 *             (This is optimal for the unweighted case.)
 *
 * ROUNDING: include vertex v if x*[v] ≥ 1/2.
 *
 * RATIO PROOF:
 *   For each edge (u,v): LP requires x*[u] + x*[v] ≥ 1,
 *   so at least one of them has x*[v] ≥ 1/2 → included → edge covered.
 *   Rounding cost ≤ 2 · LP optimal ≤ 2 · ILP optimal = 2 · OPT.  ✓
 *
 * This gives the same 2-approximation as the matching method,
 * but via LP duality and rounding.
 */
public static Set<Integer> weightedVCRounding(int V, List<int[]> edges, int[] weights) {
    // LP solution: for unweighted case, x*[v] = 0.5 for all vertices
    // Round: include vertex if x*[v] >= 0.5
    // For unweighted: include ALL vertices → trivially 2-approx
    // For weighted: solve LP with an LP solver and round

    // Here we demonstrate the rounding on a simplified case
    double[] lpSolution = new double[V];
    Arrays.fill(lpSolution, 0.5); // optimal LP for unweighted VC

    Set<Integer> cover = new HashSet<>();
    for (int v = 0; v < V; v++) {
        if (lpSolution[v] >= 0.5) cover.add(v); // ROUND UP
    }
    return cover;
}

/**
 * Randomized Rounding for MAX-SAT.
 *
 * Formulate as LP:
 *   Maximize Σ zⱼ (fraction of clause j satisfied)
 *   Subject to: LP constraints on xᵢ (truth prob of variable i)
 *
 * Solve LP, get fractional solution x*.
 * ROUND: set variable i = TRUE with probability x*[i].
 *
 * This gives a (1 - 1/e) ≈ 0.632 approximation for general MAX-SAT,
 * which combined with the pure random algorithm (1/2 approx) gives
 * a 3/4-approximation (take best of both).
 */
public static double randomizedRoundingExpected(double[] lpVars, int[][] clauses) {
    // Expected fraction of clauses satisfied by rounding
    double expected = 0;
    for (int[] clause : clauses) {
        // P(clause not satisfied) = product of P(each literal false)
        double probFalse = 1.0;
        for (int lit : clause) {
            double p = lpVars[Math.abs(lit) - 1]; // P(variable = TRUE)
            probFalse *= (lit > 0) ? (1 - p) : p;
        }
        expected += (1 - probFalse);
    }
    return expected;
}
```

---

## 8.10 Complete Approximation Summary

| Problem          | Best Ratio       | Algorithm                    | Inapproximable?          |
|------------------|------------------|------------------------------|--------------------------|
| Vertex Cover     | 2                | Matching / LP rounding       | 1.36 unless UGC fails   |
| Set Cover        | ln(n) + 1        | Greedy                       | (1-ε)ln(n) unless P=NP  |
| Metric TSP       | 1.5              | Christofides (1976)          | 123/122 unless P=NP     |
| General TSP      | None constant    | —                            | No constant unless P=NP |
| MAX-3-SAT        | 7/8              | Random / derandomized        | 7/8 + ε unless P=NP     |
| MAX-CUT          | 0.878            | Goemans-Williamson (SDP)     | 16/17 + ε unless UGC    |
| Knapsack         | 1 + ε (FPTAS)    | Scaling + DP                 | FPTAS is optimal         |
| Bin Packing      | 11/9             | First Fit Decreasing         | Tight                    |
| Clique           | n^(1-ε)          | None better known            | n^(1-ε) unless P=NP     |
| Graph Coloring   | O(n/log n)       | —                            | n^(1/4-ε) unless P=NP   |
| Euclidean TSP    | 1 + ε (PTAS)     | Arora's PTAS (1998)          | PTAS is optimal          |

---

## 8.11 When Can Problems Be Approximated? Key Insights

```
APPROXIMABLE with ratio < 2:
  → Metric TSP (1.5), MAX-3-SAT (7/8), MAX-CUT (0.878)

APPROXIMABLE with ratio 2:
  → Vertex Cover, k-CENTER, Metric k-Median (varies)

APPROXIMABLE with ratio O(log n):
  → Set Cover, Dominating Set, Group Steiner Tree

APPROXIMABLE with PTAS (1+ε for any ε):
  → Knapsack, Euclidean TSP, Bin Packing (APTAS)
  → Many geometric problems

NOT APPROXIMABLE with any constant factor (unless P=NP):
  → General TSP
  → Maximum Clique (only n^(1-ε) possible)
  → Chromatic Number

THE KEY STRUCTURAL DIFFERENCE:
  Problems with rich LP/SDP relaxations tend to be well-approximable.
  Problems where optimal and near-optimal have very different structures
  are hard to approximate.
```

---

## 8.12 Complete Example — Comparing Approaches on Metric TSP

```java
import java.util.*;

public class TSPComparison {

    // Generate random metric (satisfies triangle inequality)
    static double[][] randomMetric(int n, long seed) {
        Random rng = new Random(seed);
        double[] x = new double[n], y = new double[n];
        for (int i = 0; i < n; i++) { x[i] = rng.nextDouble(); y[i] = rng.nextDouble(); }
        double[][] dist = new double[n][n];
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                dist[i][j] = Math.sqrt((x[i]-x[j])*(x[i]-x[j]) + (y[i]-y[j])*(y[i]-y[j]));
        return dist;
    }

    // Nearest neighbor heuristic (no ratio guarantee)
    static double nearestNeighbor(double[][] dist) {
        int n = dist.length;
        boolean[] vis = new boolean[n];
        int cur = 0; vis[0] = true;
        double total = 0;
        for (int step = 1; step < n; step++) {
            int next = -1; double best = Double.MAX_VALUE;
            for (int j = 0; j < n; j++)
                if (!vis[j] && dist[cur][j] < best) { best = dist[cur][j]; next = j; }
            total += best; vis[next] = true; cur = next;
        }
        return total + dist[cur][0];
    }

    // MST lower bound (OPT ≥ MST cost)
    static double mstCost(double[][] dist) {
        int n = dist.length;
        boolean[] inMST = new boolean[n]; double[] key = new double[n];
        Arrays.fill(key, Double.MAX_VALUE); key[0] = 0;
        double total = 0;
        for (int iter = 0; iter < n; iter++) {
            int u = -1;
            for (int v = 0; v < n; v++) if (!inMST[v] && (u==-1 || key[v]<key[u])) u = v;
            inMST[u] = true; total += key[u];
            for (int v = 0; v < n; v++) if (!inMST[v] && dist[u][v] < key[v]) key[v] = dist[u][v];
        }
        return total;
    }

    public static void main(String[] args) {
        int n = 20;
        double[][] dist = randomMetric(n, 42);

        double mst = mstCost(dist);
        double nn  = nearestNeighbor(dist);

        // 2-approximation via MST
        List<Integer> tour2 = tsp2Approx(dist);
        double cost2 = tourCost(tour2, dist);

        System.out.printf("n = %d cities%n", n);
        System.out.printf("MST cost (lower bound for OPT): %.4f%n", mst);
        System.out.printf("Nearest Neighbor (no guarantee): %.4f  ratio ≥ %.2f%n",
                          nn, nn/mst);
        System.out.printf("2-Approximation (MST-based):     %.4f  ratio ≤ 2.00  (actual: %.2f)%n",
                          cost2, cost2/mst);
        System.out.printf("%nGuaranteed: 2-Approx tour ≤ 2 × OPT ≤ 2 × %.4f = %.4f%n",
                          mst, 2*mst);
    }

    // Re-use tsp2Approx and tourCost from Section 8.4
    static List<Integer> tsp2Approx(double[][] dist) {
        int V = dist.length;
        boolean[] inMST = new boolean[V]; double[] key = new double[V]; int[] parent = new int[V];
        Arrays.fill(key, Double.MAX_VALUE); Arrays.fill(parent, -1); key[0] = 0;
        List<List<Integer>> mstAdj = new ArrayList<>();
        for (int i = 0; i < V; i++) mstAdj.add(new ArrayList<>());
        for (int iter = 0; iter < V; iter++) {
            int u = -1;
            for (int v = 0; v < V; v++) if (!inMST[v] && (u==-1 || key[v]<key[u])) u = v;
            inMST[u] = true;
            if (parent[u]!=-1) { mstAdj.get(parent[u]).add(u); mstAdj.get(u).add(parent[u]); }
            for (int v = 0; v < V; v++) if (!inMST[v] && dist[u][v]<key[v]) { key[v]=dist[u][v]; parent[v]=u; }
        }
        List<Integer> tour = new ArrayList<>();
        boolean[] vis = new boolean[V];
        Deque<Integer> stack = new ArrayDeque<>(); stack.push(0);
        while (!stack.isEmpty()) {
            int u = stack.pop();
            if (vis[u]) continue; vis[u]=true; tour.add(u);
            for (int v : mstAdj.get(u)) if (!vis[v]) stack.push(v);
        }
        tour.add(0);
        return tour;
    }

    static double tourCost(List<Integer> tour, double[][] dist) {
        double c = 0;
        for (int i = 0; i < tour.size()-1; i++) c += dist[tour.get(i)][tour.get(i+1)];
        return c;
    }
}
```

---

## 8.13 Course Conclusion — The Algorithm Designer's Toolkit

```
PROBLEM ANALYSIS FLOWCHART:

Is the problem in P?
  YES → Use the known polynomial algorithm (Modules 2–3 techniques)
  NO  ↓

Is n small (≤ 20)?
  YES → Exact: Backtracking (Module 5) or Branch & Bound (Module 6)
  NO  ↓

Does it have optimal substructure + overlapping subproblems?
  YES → Dynamic Programming (Module 4)
  NO  ↓

Does it have the Greedy Choice Property?
  YES → Greedy Algorithm (Module 3)
  NO  ↓

Is there a constant-factor approximation?
  YES → Use it (Module 8): Vertex Cover, TSP, MAX-SAT, etc.
  NO  ↓

Is a (1+ε)-approximation good enough?
  YES → PTAS or FPTAS (Module 8): Knapsack, Euclidean TSP
  NO  ↓

Use Metaheuristics (Module 6): Simulated Annealing, Genetic Algorithms
(No quality guarantee, but often excellent in practice)
```

---

## 8.14 Key Takeaways

| Concept                   | Summary                                                                           |
|---------------------------|-----------------------------------------------------------------------------------|
| Approximation ratio ρ     | ALG ≤ ρ·OPT (min) or ALG ≥ OPT/ρ (max); measures quality guarantee             |
| Vertex Cover 2-approx     | Pick both endpoints of any uncovered edge; proof via maximal matching            |
| Set Cover ln(n)-approx    | Greedy picks most-covering set; tight: no (1-ε)ln(n) unless P=NP               |
| Metric TSP 2-approx       | MST walk + shortcutting; OPT ≥ MST cost guarantees ratio                        |
| Christofides 1.5-approx   | MST + min matching on odd-degree vertices; 47-year-old best known ratio          |
| Knapsack FPTAS            | Scale and round values; O(n³/ε); both n and 1/ε polynomial                      |
| Bin Packing FFD           | Sort decreasing + first fit; ratio ≈ 11/9                                        |
| MAX-3-SAT 7/8-approx      | Random assignment; optimal by PCP Theorem                                        |
| PTAS                      | (1+ε)-approx for any ε; time poly in n, possibly exp in 1/ε                     |
| FPTAS                     | (1+ε)-approx; time poly in both n and 1/ε; strictly stronger than PTAS          |
| APX                       | Constant-factor approximable; subset of NP-hard problems                         |
| LP relaxation + rounding  | Solve fractional LP, round to integer; ratio bounded by integrality gap          |
| PCP Theorem               | Enables inapproximability proofs; MAX-3-SAT is 7/8-hard                         |
| Inapproximability         | General TSP: no constant factor; Clique: only n^(1-ε); unless P=NP             |

---

## 8.15 Practice Problems

1. **Vertex Cover analysis** — Construct a graph where the 2-approximation matching algorithm gives a solution of size exactly 2·OPT. What structure does this graph have? Does this show the ratio 2 is tight?

2. **Set Cover bound** — Prove the approximation ratio of greedy set cover is H(n) = 1 + 1/2 + 1/3 + ... + 1/n ≈ ln(n). Use the potential function argument: after choosing t sets, show the remaining uncovered elements is at most n · (1 - 1/OPT)^t.

3. **Knapsack FPTAS** — For the instance weights=[1,2,3,4,5], values=[4,5,3,7,6], capacity=8, run the FPTAS with ε=0.5. Show the scaling factor K, the scaled values, the DP table on scaled values, and the final approximation. Compare to the exact DP answer.

4. **LP rounding for weighted vertex cover** — Formulate weighted vertex cover as an ILP. Describe the LP relaxation. Prove that rounding all x*[v] ≥ 1/2 gives a 2-approximation by comparing the rounded cost to the LP optimal. Why does this give ratio 2 rather than something larger?

5. **Inapproximability** — General TSP (without the metric constraint) cannot be approximated to any constant factor c unless P=NP. Prove this by reduction from HAMILTONIAN-CYCLE: given a graph G, construct a TSP instance where Hamiltonian cycles have cost n and all other tours have cost > c·n.

---

## 8.16 CS 3304 Course Complete

Congratulations on completing CS 3304 Analysis of Algorithms!

### Full Course Summary

| Module | Topic                                   | Core Techniques                                         |
|--------|-----------------------------------------|---------------------------------------------------------|
| 1      | Algorithm Design & Analysis             | Asymptotic notation, recurrences, amortized analysis    |
| 2      | Divide and Conquer                      | Master Theorem, Merge Sort, Strassen, Karatsuba         |
| 3      | Greedy Algorithms                       | Exchange argument, Activity Selection, Huffman, MST     |
| 4      | Dynamic Programming                     | Memoization, tabulation, Knapsack, LCS, Edit Distance   |
| 5      | Backtracking                            | State space tree, pruning, N-Queens, Sudoku, CSPs       |
| 6      | Optimization Techniques                 | LP, Simplex, Max-Flow, Branch & Bound, Metaheuristics   |
| 7      | Computational Complexity                | P, NP, NP-completeness, reductions, P vs. NP           |
| 8      | Approximation Algorithms                | Approximation ratios, PTAS, FPTAS, LP rounding, PCP    |

### The Algorithm Designer's Hierarchy

```
EXACT + POLYNOMIAL    → P algorithms (Modules 1–3)
OPTIMAL SUBSTRUCTURE  → Dynamic Programming (Module 4)
GREEDY CHOICE PROP.   → Greedy Algorithms (Module 3)
COMBINATORIAL EXACT   → Backtracking / B&B (Modules 5–6)
HARD OPTIMIZATION     → Approximation (Module 8)
VERY LARGE HARD PROB  → Metaheuristics (Module 6)
FUNDAMENTAL LIMITS    → Complexity Theory (Module 7)
```

---

## 8.17 Further Reading

- *Introduction to Algorithms* (CLRS) — Chapter 35 (Approximation Algorithms)
- *Approximation Algorithms* — Vazirani (the comprehensive reference)
- *The Design of Approximation Algorithms* — Williamson & Shmoys (free online)
- Christofides' original paper: "Worst-case analysis of a new heuristic for the travelling salesman problem" (1976)
- PCP Theorem survey: [https://eccc.weizmann.ac.il/report/1998/069/](https://eccc.weizmann.ac.il/report/1998/069/)
- Arora's PTAS for Euclidean TSP (1998): [https://doi.org/10.1145/290179.290180](https://doi.org/10.1145/290179.290180)

---

*CS 3304 Analysis of Algorithms — Module 8 of 8 — Course Complete*
*Previous: Module 7 — Computational Complexity Theory*
