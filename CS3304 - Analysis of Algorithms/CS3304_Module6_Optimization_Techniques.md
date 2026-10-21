# CS 3304 — Analysis of Algorithms
## Module 6: Optimization Techniques

---

## 6.1 Introduction

The previous four modules covered exact algorithmic paradigms: Divide and Conquer gives exact answers, Greedy gives exact answers on the right problems, Dynamic Programming gives exact optimal answers, and Backtracking finds all solutions. But many real-world optimization problems are either NP-hard (no known polynomial algorithm) or so large that even polynomial algorithms are too slow in practice.

**Optimization techniques** are strategies for finding good — often optimal — solutions to complex problems, spanning:

- **Mathematical optimization frameworks** (Linear Programming, Integer Programming)
- **Network flow algorithms** (max-flow, min-cut, bipartite matching)
- **Metaheuristics** (simulated annealing, genetic algorithms, tabu search) for when exact methods are impractical
- **Branch and Bound** — the classical exact method for combinatorial optimization

This module bridges the gap between the theoretical elegance of exact algorithms and the pragmatic demands of real-world optimization. By the end you will:

- Formulate problems as linear programs and interpret the simplex method
- Understand max-flow / min-cut and apply Ford-Fulkerson
- Solve bipartite matching and assignment problems
- Use Branch and Bound for integer programs
- Apply metaheuristics to NP-hard problems
- Know which technique to reach for in practice

---

## 6.2 Linear Programming (LP)

### What is Linear Programming?

A **Linear Program** seeks to maximize or minimize a **linear objective function** subject to **linear constraints**.

```
General form (Standard Maximization):

  Maximize:    c^T x         (linear objective)
  Subject to:  Ax ≤ b        (linear inequality constraints)
               x ≥ 0         (non-negativity)

Where:
  x ∈ ℝⁿ  — decision variables (what we control)
  c ∈ ℝⁿ  — objective coefficients (what we optimize)
  A ∈ ℝᵐˣⁿ — constraint matrix
  b ∈ ℝᵐ  — constraint bounds
```

### Geometric Intuition

The feasible region defined by linear constraints forms a **convex polytope** (polygon in 2D, polyhedron in 3D). The optimal solution always lies at a **vertex** (corner point) of this polytope.

```
Example:
  Maximize:   3x + 5y
  Subject to: x ≤ 4
              2y ≤ 12
              3x + 5y ≤ 25
              x, y ≥ 0

Feasible region vertices: (0,0), (4,0), (4,13/5), (5/3,6), (0,6)

Evaluate objective at each corner:
  (0,0):    0
  (4,0):    12
  (4,13/5): 12 + 13 = 25
  (5/3,6):  5 + 30  = 35  ← optimal!
  (0,6):    30

Optimal: x=5/3, y=6, value=35
```

### Formulating Problems as LPs

```java
/**
 * LP Formulation — Production Planning Example.
 *
 * A factory makes two products P1 and P2.
 *   P1: profit $5, needs 1 hr machine A + 2 hr machine B
 *   P2: profit $4, needs 2 hr machine A + 1 hr machine B
 *   Machine A available: 8 hours/day
 *   Machine B available: 7 hours/day
 *
 * Variables: x1 = units of P1, x2 = units of P2
 *
 * LP:
 *   Maximize:   5x1 + 4x2
 *   Subject to: 1x1 + 2x2 ≤ 8   (machine A)
 *               2x1 + 1x2 ≤ 7   (machine B)
 *               x1, x2 ≥ 0
 *
 * This demo evaluates corner points (since we cannot call a full simplex solver).
 */
public class LPDemo {

    // Objective function: 5x + 4y
    static double objective(double x, double y) { return 5*x + 4*y; }

    public static void main(String[] args) {
        // Corner points of feasible region
        double[][] corners = {
            {0, 0},          // origin
            {3.5, 0},        // 2x=7 and y=0
            {2, 3},          // intersection of 1x+2y=8 and 2x+y=7
            {0, 4}           // x=0 and 1x+2y=8 → y=4
        };

        double best = Double.NEGATIVE_INFINITY;
        double[] optimal = null;

        for (double[] pt : corners) {
            double val = objective(pt[0], pt[1]);
            System.out.printf("Corner (%.1f, %.1f): objective = %.1f%n",
                              pt[0], pt[1], val);
            if (val > best) { best = val; optimal = pt; }
        }
        System.out.printf("Optimal: x1=%.1f, x2=%.1f, profit=%.1f%n",
                          optimal[0], optimal[1], best);
        // Optimal: x1=2.0, x2=3.0, profit=22.0
    }
}
```

### LP Duality

Every LP has a **dual** LP. The dual of a maximization problem is a minimization problem.

```
Primal (max):           Dual (min):
  max c^T x               min b^T y
  Ax ≤ b                  A^T y ≥ c
  x ≥ 0                   y ≥ 0

Strong Duality Theorem: If both primal and dual are feasible,
  optimal primal value = optimal dual value.

Significance:
  - The dual solution gives shadow prices (how much the
    objective improves per unit relaxation of each constraint)
  - LP duality underlies the max-flow min-cut theorem
  - Used in algorithm analysis and approximation algorithms
```

---

## 6.3 The Simplex Method

The simplex method is the classical algorithm for solving LPs. It walks along edges of the feasible polytope from vertex to vertex, always improving the objective.

```
Algorithm (conceptual):
1. Start at a feasible vertex (usually origin if feasible)
2. Look at adjacent vertices connected by an edge
3. Move to the adjacent vertex with the greatest objective improvement
4. Repeat until no adjacent vertex improves the objective → optimal

Time complexity: O(2^n) worst case (Klee-Minty examples), but
                 O(polynomial) on virtually all practical problems.
                 Average case is remarkably fast.
```

```java
/**
 * Simplex Method — Tableau form for maximization.
 *
 * Uses the standard tableau (augmented matrix with slack variables).
 * Implements the full pivoting procedure.
 *
 * Format: tableau rows represent constraints + objective,
 *         columns represent variables + slack variables + RHS
 *
 * Time:  Exponential worst case, polynomial in practice
 * Space: O(m * n) for tableau
 */
public class SimplexMethod {

    /**
     * Solve LP: maximize c·x subject to Ax ≤ b, x ≥ 0
     *
     * @param A  constraint matrix (m × n)
     * @param b  RHS of constraints (m)
     * @param c  objective coefficients (n)
     * @return   optimal objective value, or -∞ if infeasible/unbounded
     */
    public static double simplex(double[][] A, double[] b, double[] c) {
        int m = A.length;    // number of constraints
        int n = c.length;    // number of variables

        // Build initial tableau [A | I | b] with slack variables
        // Last row is the objective (-c | 0...0 | 0)
        double[][] tab = new double[m + 1][n + m + 1];

        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) tab[i][j] = A[i][j]; // constraint coeffs
            tab[i][n + i] = 1;                                 // slack variable
            tab[i][n + m]  = b[i];                             // RHS
        }
        for (int j = 0; j < n; j++) tab[m][j] = -c[j]; // objective row (negated)

        int cols = n + m + 1;

        while (true) {
            // Find pivot column: most negative coefficient in objective row
            int pivotCol = -1;
            double minVal = -1e-9;
            for (int j = 0; j < cols - 1; j++) {
                if (tab[m][j] < minVal) { minVal = tab[m][j]; pivotCol = j; }
            }
            if (pivotCol == -1) break; // no negative → optimal

            // Find pivot row: minimum ratio test (b[i] / tab[i][pivotCol])
            int pivotRow = -1;
            double minRatio = Double.POSITIVE_INFINITY;
            for (int i = 0; i < m; i++) {
                if (tab[i][pivotCol] > 1e-9) {
                    double ratio = tab[i][cols - 1] / tab[i][pivotCol];
                    if (ratio < minRatio) { minRatio = ratio; pivotRow = i; }
                }
            }
            if (pivotRow == -1) return Double.POSITIVE_INFINITY; // unbounded

            // Pivot: make tab[pivotRow][pivotCol] = 1, clear rest of column
            double pivotVal = tab[pivotRow][pivotCol];
            for (int j = 0; j < cols; j++) tab[pivotRow][j] /= pivotVal;
            for (int i = 0; i <= m; i++) {
                if (i != pivotRow) {
                    double factor = tab[i][pivotCol];
                    for (int j = 0; j < cols; j++) tab[i][j] -= factor * tab[pivotRow][j];
                }
            }
        }

        return tab[m][cols - 1]; // optimal objective value
    }

    public static void main(String[] args) {
        // Maximize 5x1 + 4x2
        // Subject to: x1 + 2x2 ≤ 8,  2x1 + x2 ≤ 7
        double[][] A = {{1, 2}, {2, 1}};
        double[]   b = {8, 7};
        double[]   c = {5, 4};

        double opt = simplex(A, b, c);
        System.out.printf("Optimal value: %.2f%n", opt); // 22.00
    }
}
```

---

## 6.4 Network Flow — Max-Flow / Min-Cut

### The Maximum Flow Problem

A **flow network** is a directed graph where each edge has a **capacity**. A **flow** assigns a value to each edge (≤ capacity) such that flow is conserved at every internal node (flow in = flow out). The **maximum flow** problem finds the greatest amount of flow from a source s to a sink t.

```
Flow Network:
        10          10
    s ─────→ u ─────→ t
    │    10  ↑  10    ↑
    └────→ v ─────────┘

Capacities: s→u=10, s→v=10, u→v=10, u→t=10, v→t=10

Max flow: 20 (10 along s→u→t and 10 along s→v→t)
```

### Ford-Fulkerson Algorithm

```
IDEA: Find any path from s to t where all edges have remaining capacity.
      Send as much flow along this path as possible.
      Repeat until no such path exists.

The "residual graph" tracks remaining capacity and allows
"undoing" flow along reverse edges.
```

```java
/**
 * Ford-Fulkerson Max Flow using BFS (Edmonds-Karp variant).
 *
 * Time:  O(V * E²) for Edmonds-Karp (BFS-based path finding)
 * Space: O(V + E) for residual graph
 *
 * KEY INSIGHT: The residual graph includes reverse edges.
 * Sending flow along a reverse edge "cancels" previously sent flow,
 * allowing the algorithm to correct suboptimal earlier choices.
 */
public class MaxFlow {

    private int V;
    private int[][] capacity; // residual capacity matrix

    public MaxFlow(int v) {
        this.V        = v;
        this.capacity = new int[v][v];
    }

    public void addEdge(int u, int v, int cap) {
        capacity[u][v] += cap; // forward edge
        // capacity[v][u] += 0  (reverse edge starts at 0 — already 0 by default)
    }

    /**
     * BFS to find an augmenting path from source to sink.
     * Returns true if a path exists; fills parent[] with the path.
     */
    private boolean bfs(int source, int sink, int[] parent) {
        boolean[] visited = new boolean[V];
        Queue<Integer> queue = new LinkedList<>();
        queue.offer(source);
        visited[source] = true;
        parent[source]  = -1;

        while (!queue.isEmpty()) {
            int u = queue.poll();
            for (int v = 0; v < V; v++) {
                if (!visited[v] && capacity[u][v] > 0) { // residual capacity > 0
                    parent[v]  = u;
                    visited[v] = true;
                    if (v == sink) return true; // found path to sink
                    queue.offer(v);
                }
            }
        }
        return false; // no augmenting path
    }

    /**
     * Compute maximum flow from source to sink.
     * Also reveals the min-cut after completion (edges with capacity[u][v] > 0
     * from visited to unvisited nodes in the last BFS form the min-cut).
     */
    public int maxFlow(int source, int sink) {
        int[] parent   = new int[V];
        int   totalFlow = 0;

        // While there is an augmenting path
        while (bfs(source, sink, parent)) {
            // Find minimum residual capacity along the path
            int pathFlow = Integer.MAX_VALUE;
            for (int v = sink; v != source; v = parent[v]) {
                int u = parent[v];
                pathFlow = Math.min(pathFlow, capacity[u][v]);
            }

            // Update residual capacities
            for (int v = sink; v != source; v = parent[v]) {
                int u = parent[v];
                capacity[u][v] -= pathFlow; // reduce forward capacity
                capacity[v][u] += pathFlow; // increase reverse capacity
            }
            totalFlow += pathFlow;
        }
        return totalFlow;
    }

    public static void main(String[] args) {
        //         10      10
        //     s→u→t   s→v→t  u↔v=10
        MaxFlow mf = new MaxFlow(4); // 0=s, 1=u, 2=v, 3=t
        mf.addEdge(0, 1, 10); // s→u
        mf.addEdge(0, 2, 10); // s→v
        mf.addEdge(1, 3, 10); // u→t
        mf.addEdge(2, 3, 10); // v→t
        mf.addEdge(1, 2, 10); // u→v

        System.out.println("Max Flow: " + mf.maxFlow(0, 3)); // 20
    }
}
```

### Max-Flow Min-Cut Theorem

> The maximum flow from s to t equals the minimum capacity cut separating s from t.

A **cut** is a partition of vertices into two sets S (containing s) and T (containing t). The **cut capacity** is the total capacity of edges from S to T.

```
Min-Cut for the example above:
  S = {s},  T = {u, v, t}
  Cut edges: s→u (cap 10) + s→v (cap 10) = 20 = max flow ✓

This duality is a special case of LP duality:
  max-flow LP dual = min-cut LP
```

---

## 6.5 Bipartite Matching

**Problem:** Given a bipartite graph (two disjoint sets with edges only between sets), find the maximum matching — the largest set of edges with no shared endpoints.

Applications: job assignment, medical resident matching, internet ad placement.

```
Left set (workers): A, B, C
Right set (jobs):   1, 2, 3

Edges: A-1, A-2, B-2, B-3, C-3

Maximum matching: A-1, B-2, C-3  (all 3 workers matched)
```

### Bipartite Matching via Max-Flow

```java
/**
 * Bipartite Matching — reduced to Max-Flow.
 *
 * Construction:
 *   - Add super-source s with edge to each left node (capacity 1)
 *   - Add super-sink t with edge from each right node (capacity 1)
 *   - All original bipartite edges have capacity 1
 *
 * Max flow = maximum matching size.
 *
 * Time: O(V * E) — V augmenting paths, each found in O(E) with BFS
 */
public static int bipartiteMatching(boolean[][] adj, int leftSize, int rightSize) {
    int V = leftSize + rightSize + 2;
    int source = V - 2, sink = V - 1;

    MaxFlow mf = new MaxFlow(V);

    // Source → left nodes
    for (int i = 0; i < leftSize; i++) mf.addEdge(source, i, 1);

    // Right nodes → sink
    for (int j = 0; j < rightSize; j++) mf.addEdge(leftSize + j, sink, 1);

    // Bipartite edges (left i → right j)
    for (int i = 0; i < leftSize; i++)
        for (int j = 0; j < rightSize; j++)
            if (adj[i][j]) mf.addEdge(i, leftSize + j, 1);

    return mf.maxFlow(source, sink);
}

/**
 * Hungarian Algorithm — O(n³) for Assignment Problem (minimum cost matching).
 *
 * The Assignment Problem: assign n workers to n jobs with minimum total cost.
 * This is the weighted bipartite matching problem.
 *
 * Key idea: augment a cost matrix with row/column reductions until
 * a zero-cost perfect matching exists.
 *
 * Time: O(n³)
 */
public static int[] hungarianAlgorithm(int[][] cost) {
    int n = cost.length;
    int[] u = new int[n+1], v = new int[n+1];      // potential (dual variables)
    int[] p = new int[n+1], way = new int[n+1];    // p[j]=worker assigned to job j

    for (int i = 1; i <= n; i++) {
        p[0] = i;
        int j0 = 0;
        int[] minVal = new int[n+1];
        boolean[] used = new boolean[n+1];
        Arrays.fill(minVal, Integer.MAX_VALUE);

        do {
            used[j0] = true;
            int i0 = p[j0], delta = Integer.MAX_VALUE, j1 = -1;
            for (int j = 1; j <= n; j++) {
                if (!used[j]) {
                    int cur = cost[i0-1][j-1] - u[i0] - v[j];
                    if (cur < minVal[j]) { minVal[j] = cur; way[j] = j0; }
                    if (minVal[j] < delta) { delta = minVal[j]; j1 = j; }
                }
            }
            for (int j = 0; j <= n; j++) {
                if (used[j]) { u[p[j]] += delta; v[j] -= delta; }
                else          minVal[j] -= delta;
            }
            j0 = j1;
        } while (p[j0] != 0);

        do { int j1 = way[j0]; p[j0] = p[j1]; j0 = j1; } while (j0 != 0);
    }

    int[] assignment = new int[n]; // assignment[i] = job assigned to worker i
    for (int j = 1; j <= n; j++) assignment[p[j]-1] = j-1;
    return assignment;
}
```

---

## 6.6 Branch and Bound

**Branch and Bound** is the standard exact method for integer and combinatorial optimization. It systematically explores the solution space as a tree but uses **bounds** to prune large portions.

```
Branch and Bound for Maximization:

1. RELAX: solve LP relaxation (allow fractional solutions) → upper bound
2. BRANCH: if solution has a fractional variable x=2.7, create two subproblems:
           left: add constraint x ≤ 2
           right: add constraint x ≥ 3
3. BOUND: compute upper bound for each subproblem
4. PRUNE: if upper bound ≤ best known integer solution → prune
5. EXPLORE: recursively branch on the most promising node
```

```java
/**
 * Branch and Bound for 0/1 Knapsack.
 *
 * LP relaxation: allow fractional items → gives upper bound.
 * Branch on the first fractional item in the relaxed solution.
 *
 * Time:  O(2^n) worst case, much better with effective bounding
 * Space: O(n) recursion depth
 */
public class BranchAndBound {

    static int   n;
    static int[] weights, values;
    static int   capacity;
    static int   bestValue = 0;

    /**
     * Compute upper bound via LP relaxation of remaining items.
     * Sort items by value/weight density; take greedily, allowing fractions.
     */
    static double upperBound(int level, int currentValue,
                              int currentWeight, int[][] sortedItems) {
        double bound  = currentValue;
        int    weight = currentWeight;

        for (int i = level; i < n; i++) {
            int w = sortedItems[i][0], v = sortedItems[i][1];
            if (weight + w <= capacity) {
                weight += w;
                bound  += v;             // take whole item
            } else {
                // Take fraction of remaining item
                bound += (double)(capacity - weight) / w * v;
                break;
            }
        }
        return bound;
    }

    /**
     * Recursive Branch and Bound.
     * At each level, branch on including/excluding current item.
     */
    static void bnb(int level, int currentValue, int currentWeight,
                    int[][] sortedItems) {
        // LEAF: all items considered
        if (level == n) {
            bestValue = Math.max(bestValue, currentValue);
            return;
        }

        // PRUNE: upper bound ≤ best known → no improvement possible
        double ub = upperBound(level, currentValue, currentWeight, sortedItems);
        if (ub <= bestValue) return;

        int w = sortedItems[level][0], v = sortedItems[level][1];

        // BRANCH RIGHT: include item if it fits
        if (currentWeight + w <= capacity) {
            bnb(level + 1, currentValue + v, currentWeight + w, sortedItems);
        }

        // BRANCH LEFT: exclude item
        bnb(level + 1, currentValue, currentWeight, sortedItems);
    }

    public static int solve(int[] w, int[] v, int W) {
        n = w.length; weights = w; values = v; capacity = W;
        bestValue = 0;

        // Sort items by value/weight density for better upper bounds
        int[][] items = new int[n][2];
        for (int i = 0; i < n; i++) { items[i][0] = w[i]; items[i][1] = v[i]; }
        Arrays.sort(items, (a, b) ->
            Double.compare((double)b[1]/b[0], (double)a[1]/a[0])); // descending density

        bnb(0, 0, 0, items);
        return bestValue;
    }

    public static void main(String[] args) {
        int[] weights = {2, 3, 4, 5};
        int[] values  = {3, 4, 5, 6};
        int W = 5;

        System.out.println("B&B Knapsack: " + solve(weights, values, W)); // 7
        // Compare: DP also gives 7
    }
}
```

### Branch and Bound vs. Backtracking

```
Backtracking (Module 5):      Branch and Bound:
  - Uses FEASIBILITY pruning    - Uses OPTIMALITY pruning (bounds)
  - Prunes constraint violations - Prunes suboptimal subtrees
  - Finds all solutions          - Finds optimal solution
  - Best for: CSPs               - Best for: optimization problems
  - No objective function        - Has objective function + bound
```

---

## 6.7 Simulated Annealing

For NP-hard problems that are too large for exact methods, **metaheuristics** find good (not necessarily optimal) solutions in reasonable time.

**Simulated Annealing** is inspired by the physical annealing process (cooling metal slowly to reach low-energy states). It accepts "worse" solutions with a probability that decreases over time — allowing escape from local optima.

```
ALGORITHM:
  current = initial solution
  temperature T = T_max

  while T > T_min:
    neighbor = random small change to current
    ΔE = cost(neighbor) - cost(current)

    if ΔE < 0:                              ← improvement
      current = neighbor  (always accept)
    else:
      accept with probability e^(-ΔE / T)   ← sometimes accept worse

    T = T * cooling_rate                    ← gradually lower temperature
```

```java
/**
 * Simulated Annealing for Traveling Salesman Problem (TSP).
 *
 * Find a short route visiting all cities exactly once and returning to start.
 * TSP is NP-hard — no known polynomial exact algorithm for general case.
 *
 * Neighbor generation: swap two cities in the tour (2-opt move)
 *
 * Time:  O(iterations * n) per run
 * Space: O(n) for tour representation
 */
public class SimulatedAnnealing {

    static double[][] dist;   // dist[i][j] = distance between cities i and j
    static int n;             // number of cities

    static double tourLength(int[] tour) {
        double len = 0;
        for (int i = 0; i < n; i++) len += dist[tour[i]][tour[(i+1) % n]];
        return len;
    }

    /**
     * Generate neighbor: reverse a random segment of the tour (2-opt).
     * This is the most effective local move for TSP.
     */
    static int[] twoOptSwap(int[] tour, int i, int k) {
        int[] newTour = tour.clone();
        // Reverse segment from i to k
        while (i < k) {
            int tmp = newTour[i]; newTour[i] = newTour[k]; newTour[k] = tmp;
            i++; k--;
        }
        return newTour;
    }

    public static int[] solve(double[][] distances, int iterations) {
        dist = distances;
        n    = distances.length;

        // Initial solution: cities in order 0, 1, 2, ..., n-1
        int[] current = new int[n];
        for (int i = 0; i < n; i++) current[i] = i;

        int[]  best       = current.clone();
        double bestLen    = tourLength(best);
        double currentLen = bestLen;

        double T    = 10000.0;   // initial temperature
        double Tmin = 0.001;     // stopping temperature
        double cool = 0.9995;    // cooling rate

        Random rng = new Random(42);

        for (int iter = 0; iter < iterations && T > Tmin; iter++) {
            // Generate neighbor: random 2-opt swap
            int i = rng.nextInt(n);
            int k = rng.nextInt(n);
            if (i > k) { int tmp = i; i = k; k = tmp; }

            int[]  neighbor    = twoOptSwap(current, i, k);
            double neighborLen = tourLength(neighbor);
            double delta       = neighborLen - currentLen;

            // Accept if better, or with probability e^(-delta/T) if worse
            if (delta < 0 || Math.random() < Math.exp(-delta / T)) {
                current    = neighbor;
                currentLen = neighborLen;
                if (currentLen < bestLen) {
                    best    = current.clone();
                    bestLen = currentLen;
                }
            }
            T *= cool; // cool down
        }

        System.out.printf("Best tour length found: %.2f%n", bestLen);
        return best;
    }

    public static void main(String[] args) {
        // 5-city TSP
        double[][] dist = {
            {0, 2, 9, 10, 5},
            {2, 0, 6,  4, 8},
            {9, 6, 0,  3, 7},
            {10,4, 3,  0, 6},
            {5, 8, 7,  6, 0}
        };
        int[] tour = solve(dist, 100000);
        System.out.println("Tour: " + Arrays.toString(tour));
    }
}
```

---

## 6.8 Genetic Algorithms

**Genetic Algorithms (GAs)** are population-based metaheuristics inspired by natural selection. A population of candidate solutions evolves through selection, crossover, and mutation.

```
ALGORITHM:
  population = random initial solutions

  for each generation:
    fitness   = evaluate(population)           ← objective function
    parents   = select(population, fitness)    ← favor better solutions
    offspring = crossover(parents)             ← combine parent solutions
    offspring = mutate(offspring)              ← random small changes
    population = offspring                     ← next generation
```

```java
/**
 * Genetic Algorithm for the Knapsack Problem.
 *
 * Chromosome: binary string of length n (1 = item included, 0 = excluded)
 * Fitness: total value of included items (0 if weight exceeds capacity)
 * Crossover: single-point crossover
 * Mutation: flip a random bit
 *
 * Time:  O(generations * population_size * n)
 * Space: O(population_size * n)
 */
public class GeneticAlgorithm {

    static int[] weights, values;
    static int   capacity, n;
    static Random rng = new Random(42);

    // Fitness function: total value (0 if over capacity)
    static int fitness(int[] chromosome) {
        int totalWeight = 0, totalValue = 0;
        for (int i = 0; i < n; i++) {
            if (chromosome[i] == 1) {
                totalWeight += weights[i];
                totalValue  += values[i];
            }
        }
        return (totalWeight <= capacity) ? totalValue : 0;
    }

    // Tournament selection: pick 2 random individuals, return the better one
    static int[] select(int[][] population) {
        int a = rng.nextInt(population.length);
        int b = rng.nextInt(population.length);
        return fitness(population[a]) >= fitness(population[b])
               ? population[a] : population[b];
    }

    // Single-point crossover
    static int[][] crossover(int[] parent1, int[] parent2) {
        int point = rng.nextInt(n);
        int[] child1 = new int[n], child2 = new int[n];
        for (int i = 0; i < n; i++) {
            child1[i] = (i < point) ? parent1[i] : parent2[i];
            child2[i] = (i < point) ? parent2[i] : parent1[i];
        }
        return new int[][]{child1, child2};
    }

    // Mutation: flip each bit with probability mutRate
    static void mutate(int[] chromosome, double mutRate) {
        for (int i = 0; i < n; i++)
            if (rng.nextDouble() < mutRate) chromosome[i] ^= 1;
    }

    public static int solve(int[] w, int[] v, int W,
                             int popSize, int generations) {
        weights = w; values = v; capacity = W; n = w.length;

        // Initialize random population
        int[][] population = new int[popSize][n];
        for (int[] chrom : population)
            for (int i = 0; i < n; i++) chrom[i] = rng.nextInt(2);

        int bestFitness = 0;

        for (int gen = 0; gen < generations; gen++) {
            int[][] next = new int[popSize][n];
            for (int i = 0; i < popSize; i += 2) {
                int[] p1 = select(population);
                int[] p2 = select(population);
                int[][] children = crossover(p1, p2);
                mutate(children[0], 0.01);
                mutate(children[1], 0.01);
                next[i]   = children[0];
                next[i+1] = children[1];
            }
            population = next;

            // Track best fitness this generation
            for (int[] chrom : population)
                bestFitness = Math.max(bestFitness, fitness(chrom));
        }
        return bestFitness;
    }

    public static void main(String[] args) {
        int[] weights = {2, 3, 4, 5, 9};
        int[] values  = {3, 4, 5, 8, 10};
        int W = 10;

        System.out.println("GA Knapsack: " + solve(weights, values, W, 100, 500));
        // Typically finds near-optimal or optimal solution
    }
}
```

---

## 6.9 Optimization Technique Selection Guide

```
PROBLEM TYPE                        RECOMMENDED TECHNIQUE
─────────────────────────────────────────────────────────────────
Continuous variables, linear        Linear Programming (Simplex)
Integer variables, small n          Branch and Bound
Integer variables, large n          B&B + LP relaxation, Metaheuristics
Network flow                        Ford-Fulkerson / Edmonds-Karp
Bipartite matching (unweighted)     Max-flow reduction
Bipartite matching (weighted)       Hungarian Algorithm O(n³)
TSP, scheduling (exact, small)      Branch and Bound
TSP, scheduling (heuristic, large)  Simulated Annealing, Genetic Alg.
Multi-objective optimization        Genetic Algorithms (Pareto frontier)
Black-box optimization              Simulated Annealing
Online optimization (stream)        Competitive analysis algorithms

DECISION TREE:
  Is the problem continuous?
    YES → Linear Programming
    NO  ↓
  Is n ≤ ~20 (combinatorial)?
    YES → Branch and Bound (exact)
    NO  ↓
  Is there special structure?
    Network: Max-Flow / Min-Cut
    Bipartite: Hungarian
    NO  ↓
  Use metaheuristics:
    Single solution: Simulated Annealing
    Population:      Genetic Algorithm
```

---

## 6.10 Complexity Summary

| Technique              | Time Complexity          | Optimality    | Best For                         |
|------------------------|--------------------------|---------------|----------------------------------|
| Simplex Method         | O(2^n) worst, poly avg   | Exact         | Continuous LP                    |
| Ford-Fulkerson         | O(E * maxFlow)           | Exact         | Network flow                     |
| Edmonds-Karp           | O(V * E²)                | Exact         | Network flow (all edge weights)  |
| Hungarian Algorithm    | O(n³)                    | Exact         | Weighted bipartite matching      |
| Branch and Bound       | O(2^n) worst, much less  | Exact         | Integer programming              |
| Simulated Annealing    | O(iterations * n)        | Heuristic     | NP-hard continuous/discrete      |
| Genetic Algorithm      | O(gen * pop * n)         | Heuristic     | Multi-modal, multi-objective     |

---

## 6.11 Complete Example — Shortest Path as LP

```java
/**
 * Shortest Path as a Linear Program.
 *
 * The shortest path problem from s to t can be formulated as:
 *
 * Variables: f[e] = flow on edge e ∈ {0, 1}
 *
 * Minimize:  Σ w[e] * f[e]
 * Subject to:
 *   Σ f[e leaving v] - Σ f[e entering v] = { +1 if v=s, -1 if v=t, 0 otherwise }
 *   0 ≤ f[e] ≤ 1
 *
 * The LP relaxation (allowing 0 ≤ f[e] ≤ 1) automatically gives integer
 * solutions for shortest paths on graphs — a special case of total unimodularity.
 *
 * This demo shows the connection conceptually using Dijkstra as the "LP solver".
 */
public class ShortestPathLP {

    public static int[] dijkstra(int[][] graph, int src) {
        int V    = graph.length;
        int[] dist = new int[V];
        boolean[] vis = new boolean[V];
        Arrays.fill(dist, Integer.MAX_VALUE);
        dist[src] = 0;

        PriorityQueue<int[]> pq = new PriorityQueue<>(Comparator.comparingInt(a -> a[0]));
        pq.offer(new int[]{0, src});

        while (!pq.isEmpty()) {
            int[] cur = pq.poll();
            int u = cur[1];
            if (vis[u]) continue;
            vis[u] = true;

            for (int v = 0; v < V; v++) {
                if (!vis[v] && graph[u][v] > 0 && dist[u] + graph[u][v] < dist[v]) {
                    dist[v] = dist[u] + graph[u][v];
                    pq.offer(new int[]{dist[v], v});
                }
            }
        }
        return dist;
    }

    public static void main(String[] args) {
        // Graph as adjacency matrix (0 = no edge)
        int[][] graph = {
            {0, 4, 0, 0, 0, 0, 0, 8, 0},
            {4, 0, 8, 0, 0, 0, 0,11, 0},
            {0, 8, 0, 7, 0, 4, 0, 0, 2},
            {0, 0, 7, 0, 9,14, 0, 0, 0},
            {0, 0, 0, 9, 0,10, 0, 0, 0},
            {0, 0, 4,14,10, 0, 2, 0, 0},
            {0, 0, 0, 0, 0, 2, 0, 1, 6},
            {8,11, 0, 0, 0, 0, 1, 0, 7},
            {0, 0, 2, 0, 0, 0, 6, 7, 0}
        };

        int[] distances = dijkstra(graph, 0);
        System.out.println("Shortest distances from vertex 0:");
        for (int i = 0; i < distances.length; i++)
            System.out.printf("  To vertex %d: %d%n", i, distances[i]);
        // The LP interpretation: each edge flow variable is 0 or 1,
        // and the optimal LP solution gives the shortest path.
    }
}
```

---

## 6.12 Key Takeaways

| Concept                  | Summary                                                                          |
|--------------------------|----------------------------------------------------------------------------------|
| Linear Programming       | Optimize a linear objective over linear constraints; feasible region is a polytope|
| Simplex Method           | Walk along polytope vertices; exponential worst case, fast in practice           |
| LP Duality               | Every LP has a dual; strong duality: primal optimal = dual optimal               |
| Max-Flow                 | Maximum flow = minimum cut (Max-Flow Min-Cut Theorem)                            |
| Ford-Fulkerson           | Augment along residual paths; O(E * maxFlow)                                     |
| Edmonds-Karp             | BFS-based Ford-Fulkerson; O(V * E²) — polynomial                                |
| Bipartite Matching       | Reduce to max-flow; O(V * E)                                                     |
| Hungarian Algorithm      | Optimal assignment in O(n³); based on LP duality / potential augmentation        |
| Branch and Bound         | Exact combinatorial optimization via LP relaxation bounds + tree search          |
| Simulated Annealing      | Escape local optima by accepting worse solutions probabilistically               |
| Genetic Algorithm        | Population-based search via selection, crossover, mutation                       |
| Technique selection      | LP for continuous; B&B for small integer; metaheuristics for large NP-hard       |

---

## 6.13 Practice Problems

1. **LP Formulation** — A farmer has 100 acres and $10,000. Corn yields $200/acre profit and costs $80/acre. Wheat yields $150/acre profit and costs $50/acre. The farmer can grow at most 60 acres of corn and at most 80 acres of wheat. Formulate this as a LP, find all corner points of the feasible region, and solve graphically.

2. **Max-Flow trace** — For the following flow network, trace the Ford-Fulkerson algorithm step by step, showing the residual graph after each augmenting path: s→a=3, s→b=2, a→b=1, a→t=2, b→t=3. What is the max flow? Identify the min-cut.

3. **Bipartite Matching** — Five students (A–E) can cover the following topics: A:{ML,DB}, B:{OS,DB}, C:{ML,Networks}, D:{OS,Algorithms}, E:{Networks,Algorithms}. Find the maximum matching that assigns each student a unique topic. Show the max-flow construction.

4. **Branch and Bound trace** — Apply Branch and Bound to the 0/1 Knapsack: items (w=2,v=6), (w=2,v=5), (w=6,v=8), (w=5,v=9), (w=4,v=6), capacity W=10. Draw the search tree, showing the upper bound computed at each node and which nodes are pruned.

5. **Simulated Annealing parameter tuning** — Implement simulated annealing for the following variant of TSP: 10 cities placed uniformly at random in a unit square. Run with 3 different cooling schedules (fast: 0.99, medium: 0.9995, slow: 0.99999) and compare tour quality vs. runtime. What cooling rate achieves the best quality/time trade-off?

---

## 6.14 Further Reading

- *Introduction to Algorithms* (CLRS) — Chapters 24–26 (Shortest Paths, Max-Flow), Chapter 29 (Linear Programming)
- *Algorithm Design* — Kleinberg & Tardos, Chapters 7–8 (Network Flow, NP)
- *Introduction to Linear Optimization* — Bertsimas & Tsitsiklis (LP theory)
- *Combinatorial Optimization* — Papadimitriou & Steiglitz (Branch and Bound, Integer Programming)
- Metaheuristics in Operations Research: [https://www.metaheuristics.net](https://www.metaheuristics.net)

---

*CS 3304 Analysis of Algorithms — Module 6 of 8*
*Previous: Module 5 — Backtracking Algorithms*
*Next: Module 7 — Computational Complexity Theory*
