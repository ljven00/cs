# CS 3303 — Data Structures
## Module 8: Exploring Graphs and Advanced Graph Algorithms

---

## 8.1 Introduction

Every data structure we have studied so far — arrays, linked lists, trees — imposes a strict shape on data. Trees are hierarchical; lists are linear. But the real world is messier. Road networks, social connections, the internet, airline routes, compiler dependencies — none of these fit neatly into a hierarchy.

**Graphs** are the universal data structure for relationships. A graph is simply a set of **nodes (vertices)** connected by **edges**. With this simple definition, graphs model virtually any network imaginable.

This final module covers:
- Graph representations and their trade-offs
- Graph traversals: DFS and BFS
- Shortest path algorithms: Dijkstra, Bellman-Ford, Floyd-Warshall
- Minimum Spanning Trees: Prim's and Kruskal's
- Advanced topics: topological sort, cycle detection, union-find

---

## 8.2 Graph Terminology

```
      (A)───3──(B)
       |  \      |
       1    5    2
       |      \  |
      (C)───4──(D)───6──(E)
```

| Term                | Definition                                                              |
|---------------------|-------------------------------------------------------------------------|
| **Vertex (Node)**   | A point in the graph — A, B, C, D, E above                             |
| **Edge**            | A connection between two vertices                                       |
| **Weight**          | A value associated with an edge (distance, cost, etc.)                  |
| **Degree**          | Number of edges incident to a vertex                                    |
| **Path**            | A sequence of vertices connected by edges                               |
| **Cycle**           | A path that starts and ends at the same vertex                          |
| **Connected**       | Every vertex is reachable from every other vertex                       |
| **Component**       | A maximal connected subgraph                                            |
| **Directed (Digraph)** | Edges have direction: A→B ≠ B→A                                     |
| **Undirected**      | Edges have no direction: A—B = B—A                                      |
| **Weighted**        | Edges have numeric values                                               |
| **Sparse**          | Few edges relative to vertices: E ≪ V²                                  |
| **Dense**           | Many edges: E ≈ V²                                                      |
| **DAG**             | Directed Acyclic Graph — directed, no cycles                            |

---

## 8.3 Graph Representations

Two primary ways to store a graph in memory, each with different trade-offs.

### Adjacency Matrix

A V×V 2D array where `matrix[i][j] = weight` if there is an edge from i to j, else 0 (or ∞).

```
Graph:  A—B, A—C, B—D, C—D, D—E

     A  B  C  D  E
A  [ 0, 1, 1, 0, 0 ]
B  [ 1, 0, 0, 1, 0 ]
C  [ 1, 0, 0, 1, 0 ]
D  [ 0, 1, 1, 0, 1 ]
E  [ 0, 0, 0, 1, 0 ]
```

```java
/**
 * Adjacency Matrix representation.
 * Space: O(V²) — always allocates V² entries regardless of edge count.
 *
 * Best for: dense graphs, O(1) edge existence check.
 */
public class AdjacencyMatrix {
    private int[][] matrix;
    private int vertices;

    public AdjacencyMatrix(int v) {
        this.vertices = v;
        this.matrix   = new int[v][v];
    }

    // Add undirected edge (use single direction for directed)
    public void addEdge(int u, int v, int weight) {
        matrix[u][v] = weight;
        matrix[v][u] = weight; // remove for directed graph
    }

    public boolean hasEdge(int u, int v) { return matrix[u][v] != 0; } // O(1)

    public void print() {
        for (int[] row : matrix) System.out.println(Arrays.toString(row));
    }
}
```

### Adjacency List

Each vertex stores a list of its neighbors. The standard representation for most graph algorithms.

```
A → [B, C]
B → [A, D]
C → [A, D]
D → [B, C, E]
E → [D]
```

```java
/**
 * Adjacency List representation using HashMap of Lists.
 * Space: O(V + E) — only stores actual edges.
 *
 * Best for: sparse graphs, iterating over neighbors.
 */
public class Graph {
    private Map<Integer, List<int[]>> adjList; // int[] = {neighbor, weight}
    private int vertices;
    private boolean directed;

    public Graph(int v, boolean directed) {
        this.vertices = v;
        this.directed = directed;
        adjList = new HashMap<>();
        for (int i = 0; i < v; i++) adjList.put(i, new ArrayList<>());
    }

    public void addEdge(int u, int v, int weight) {
        adjList.get(u).add(new int[]{v, weight});
        if (!directed) adjList.get(v).add(new int[]{u, weight});
    }

    public List<int[]> neighbors(int u) { return adjList.get(u); }

    public void print() {
        for (int u : adjList.keySet()) {
            System.out.print(u + " → ");
            for (int[] edge : adjList.get(u)) {
                System.out.print("[" + edge[0] + ", w=" + edge[1] + "] ");
            }
            System.out.println();
        }
    }
}
```

### Representation Comparison

| Operation              | Adjacency Matrix | Adjacency List      |
|------------------------|------------------|---------------------|
| Space                  | O(V²)            | O(V + E)            |
| Check edge (u,v)       | O(1)             | O(degree(u))        |
| Iterate neighbors of u | O(V)             | O(degree(u))        |
| Add edge               | O(1)             | O(1)                |
| Best for               | Dense graphs     | Sparse graphs       |

---

## 8.4 Depth-First Search (DFS)

DFS explores as far as possible along each branch before backtracking. It uses a **stack** (either explicit or the call stack via recursion).

```
Graph:  1—2—5
        |   |
        3—4—6

DFS from 1: 1 → 2 → 5 → 6 → 4 → 3
```

```java
/**
 * DFS — Recursive implementation.
 * Uses the system call stack as the implicit stack.
 *
 * Time:  O(V + E)
 * Space: O(V) for visited set + O(V) recursion depth
 *
 * Applications: cycle detection, topological sort,
 *               connected components, maze solving.
 */
public static void dfsRecursive(Graph graph, int start, int v) {
    boolean[] visited = new boolean[v];
    dfsHelper(graph, start, visited);
}

private static void dfsHelper(Graph graph, int node, boolean[] visited) {
    visited[node] = true;
    System.out.print(node + " ");                       // process node

    for (int[] edge : graph.neighbors(node)) {
        int neighbor = edge[0];
        if (!visited[neighbor]) {
            dfsHelper(graph, neighbor, visited);        // recurse on unvisited
        }
    }
}

/**
 * DFS — Iterative using an explicit stack.
 * Useful for very deep graphs where recursion would cause StackOverflow.
 */
public static void dfsIterative(Graph graph, int start, int v) {
    boolean[] visited = new boolean[v];
    Deque<Integer> stack = new ArrayDeque<>();
    stack.push(start);

    while (!stack.isEmpty()) {
        int node = stack.pop();
        if (visited[node]) continue;
        visited[node] = true;
        System.out.print(node + " ");

        for (int[] edge : graph.neighbors(node)) {
            if (!visited[edge[0]]) stack.push(edge[0]);
        }
    }
}
```

---

## 8.5 Breadth-First Search (BFS)

BFS explores all neighbors at the current depth before going deeper. It uses a **queue** and naturally finds **shortest paths** in unweighted graphs.

```
Graph:  1—2—5
        |   |
        3—4—6

BFS from 1: 1 → 2 → 3 → 5 → 4 → 6
              (level 1) (level 2)  (level 3)
```

```java
/**
 * BFS — uses a queue to explore level by level.
 *
 * Time:  O(V + E)
 * Space: O(V) for visited array + O(V) queue
 *
 * Applications: shortest path (unweighted), level-order traversal,
 *               connected components, bipartite check.
 */
public static void bfs(Graph graph, int start, int v) {
    boolean[] visited = new boolean[v];
    Queue<Integer> queue = new LinkedList<>();

    visited[start] = true;
    queue.offer(start);

    while (!queue.isEmpty()) {
        int node = queue.poll();
        System.out.print(node + " ");                   // process node

        for (int[] edge : graph.neighbors(node)) {
            int neighbor = edge[0];
            if (!visited[neighbor]) {
                visited[neighbor] = true;
                queue.offer(neighbor);                  // enqueue unvisited neighbors
            }
        }
    }
}

/**
 * BFS Shortest Path — finds shortest hop-count path in unweighted graph.
 * Returns the path from start to end, or empty list if unreachable.
 *
 * Time: O(V + E)
 */
public static List<Integer> bfsShortestPath(Graph graph, int start, int end, int v) {
    boolean[] visited = new boolean[v];
    int[] parent = new int[v];          // parent[i] = previous node on shortest path
    Arrays.fill(parent, -1);

    Queue<Integer> queue = new LinkedList<>();
    visited[start] = true;
    queue.offer(start);

    while (!queue.isEmpty()) {
        int node = queue.poll();
        if (node == end) break;

        for (int[] edge : graph.neighbors(node)) {
            int neighbor = edge[0];
            if (!visited[neighbor]) {
                visited[neighbor] = true;
                parent[neighbor] = node;
                queue.offer(neighbor);
            }
        }
    }

    // Reconstruct path from end → start using parent array
    List<Integer> path = new ArrayList<>();
    if (!visited[end]) return path; // unreachable

    for (int at = end; at != -1; at = parent[at]) path.add(0, at);
    return path;
}
```

### DFS vs BFS Summary

| Feature              | DFS                            | BFS                            |
|----------------------|--------------------------------|--------------------------------|
| Data structure       | Stack (implicit or explicit)   | Queue                          |
| Memory               | O(h) — max depth               | O(w) — max width               |
| Finds shortest path? | No (unweighted)                | Yes (unweighted)               |
| Best for             | Cycle detection, topo sort, maze | Shortest path, level order   |
| Complete?            | Yes (finite graphs)            | Yes                            |

---

## 8.6 Dijkstra's Algorithm — Shortest Path (Non-negative Weights)

Dijkstra's algorithm finds the shortest path from a source vertex to all other vertices in a graph with **non-negative edge weights**. It greedily picks the nearest unvisited vertex at each step.

```
Graph (weighted):
     A ──2── B
     |  \    |
     4    6  3
     |      \|
     C ──1── D ──5── E

Dijkstra from A:
dist = {A:0, B:∞, C:∞, D:∞, E:∞}

Step 1: Visit A (dist=0). Update: B=2, C=4, D=6
Step 2: Visit B (dist=2). Update: D=min(6, 2+3)=5
Step 3: Visit C (dist=4). Update: D=min(5, 4+1)=5
Step 4: Visit D (dist=5). Update: E=min(∞, 5+5)=10
Step 5: Visit E (dist=10). Done.

Shortest distances from A: {A:0, B:2, C:4, D:5, E:10}
```

```java
import java.util.*;

/**
 * Dijkstra's Algorithm using a Min-Priority Queue.
 *
 * Strategy: always expand the unvisited vertex with the smallest
 *           known distance (greedy choice).
 *
 * Time:  O((V + E) log V) with binary heap priority queue
 * Space: O(V)
 *
 * IMPORTANT: Does NOT work with negative edge weights.
 *            Use Bellman-Ford for graphs with negative weights.
 */
public static int[] dijkstra(Graph graph, int start, int v) {
    int[] dist    = new int[v];
    boolean[] visited = new boolean[v];
    int[] parent  = new int[v];

    Arrays.fill(dist,   Integer.MAX_VALUE);
    Arrays.fill(parent, -1);
    dist[start] = 0;

    // Min-heap: {distance, vertex}
    PriorityQueue<int[]> pq = new PriorityQueue<>(Comparator.comparingInt(a -> a[0]));
    pq.offer(new int[]{0, start});

    while (!pq.isEmpty()) {
        int[] curr    = pq.poll();
        int currDist  = curr[0];
        int node      = curr[1];

        if (visited[node]) continue;     // skip stale entries
        visited[node] = true;

        for (int[] edge : graph.neighbors(node)) {
            int neighbor = edge[0];
            int weight   = edge[1];

            if (!visited[neighbor] && dist[node] + weight < dist[neighbor]) {
                dist[neighbor]   = dist[node] + weight;  // relax edge
                parent[neighbor] = node;
                pq.offer(new int[]{dist[neighbor], neighbor});
            }
        }
    }
    return dist;
}

/**
 * Reconstruct the shortest path from start to end using the parent array.
 */
public static List<Integer> getPath(int[] parent, int end) {
    List<Integer> path = new ArrayList<>();
    for (int at = end; at != -1; at = parent[at]) path.add(0, at);
    return path;
}
```

---

## 8.7 Bellman-Ford Algorithm — Shortest Path (Negative Weights)

Dijkstra fails if any edge weight is negative. **Bellman-Ford** handles negative weights and also detects **negative cycles** (a cycle whose total weight is negative, making "shortest path" undefined).

```
Strategy: relax ALL edges V-1 times.
After V-1 rounds, shortest paths are guaranteed to be found
(a shortest path can have at most V-1 edges).

If a distance still decreases in round V → negative cycle detected.
```

```java
/**
 * Bellman-Ford Algorithm.
 *
 * Time:  O(V * E) — much slower than Dijkstra
 * Space: O(V)
 *
 * Use when:
 *   - Graph has negative edge weights
 *   - Need to detect negative cycles
 */
public static int[] bellmanFord(int v, int[][] edges, int start) {
    // edges[i] = {u, v, weight}
    int[] dist = new int[v];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[start] = 0;

    // Relax all edges V-1 times
    for (int i = 0; i < v - 1; i++) {
        for (int[] edge : edges) {
            int u = edge[0], nb = edge[1], w = edge[2];
            if (dist[u] != Integer.MAX_VALUE && dist[u] + w < dist[nb]) {
                dist[nb] = dist[u] + w;       // relax edge
            }
        }
    }

    // V-th relaxation pass: if any distance still improves → negative cycle
    for (int[] edge : edges) {
        int u = edge[0], nb = edge[1], w = edge[2];
        if (dist[u] != Integer.MAX_VALUE && dist[u] + w < dist[nb]) {
            System.out.println("Negative cycle detected!");
            return null;
        }
    }
    return dist;
}

/*
Example:
edges = {{0,1,4},{0,2,5},{1,2,-3},{2,3,2}}
start = 0

dist after 3 relaxation passes: [0, 4, 1, 3]
Shortest paths: 0→1=4, 0→1→2=1, 0→1→2→3=3
*/
```

---

## 8.8 Floyd-Warshall Algorithm — All-Pairs Shortest Paths

Floyd-Warshall computes shortest paths between **all pairs** of vertices in a single run. It works with negative weights (but not negative cycles).

```
Key idea (dynamic programming):
dist[i][j] = shortest path from i to j using only vertices {0..k} as intermediates

dist[i][j] = min(dist[i][j],  dist[i][k] + dist[k][j])
             current best      route through intermediate k
```

```java
/**
 * Floyd-Warshall Algorithm.
 *
 * Time:  O(V³)
 * Space: O(V²)
 *
 * Use when: need ALL-PAIRS shortest paths; graph is dense; V is small.
 */
public static int[][] floydWarshall(int[][] graph) {
    int v    = graph.length;
    int[][] dist = new int[v][v];
    int INF = Integer.MAX_VALUE / 2; // use /2 to avoid overflow on addition

    // Initialize distance matrix
    for (int i = 0; i < v; i++)
        for (int j = 0; j < v; j++)
            dist[i][j] = (i == j) ? 0 : (graph[i][j] == 0 ? INF : graph[i][j]);

    // Try each vertex k as an intermediate
    for (int k = 0; k < v; k++) {
        for (int i = 0; i < v; i++) {
            for (int j = 0; j < v; j++) {
                if (dist[i][k] + dist[k][j] < dist[i][j]) {
                    dist[i][j] = dist[i][k] + dist[k][j]; // shorter via k
                }
            }
        }
    }

    // Check for negative cycles: negative value on diagonal means cycle
    for (int i = 0; i < v; i++) {
        if (dist[i][i] < 0) {
            System.out.println("Negative cycle exists!");
            return null;
        }
    }
    return dist;
}
```

### Shortest Path Algorithm Comparison

| Algorithm       | Weights        | Complexity     | Use Case                              |
|-----------------|----------------|----------------|---------------------------------------|
| BFS             | Unweighted     | O(V + E)       | Hop-count shortest path               |
| Dijkstra        | Non-negative   | O((V+E) log V) | Single-source, non-negative weights   |
| Bellman-Ford    | Any (neg OK)   | O(V * E)       | Negative weights, cycle detection     |
| Floyd-Warshall  | Any (neg OK)   | O(V³)          | All-pairs shortest paths, dense graph |

---

## 8.9 Minimum Spanning Tree (MST)

A **Minimum Spanning Tree** of a connected, weighted, undirected graph is a subgraph that:
- Connects all V vertices
- Contains exactly V-1 edges
- Has minimum total edge weight
- Contains no cycles

```
Original graph:          MST (total weight = 11):
  A—4—B—2—C               A—4—B—2—C
  |\ 2  /|                        |
  8  \ / 3                        3
  |   X  |                        |
  D—5—E—1—F               D—5—E—1—F
```

MSTs model problems like: minimum cable to connect all offices, cheapest network connecting all cities, minimum cost circuit wiring.

### Prim's Algorithm

Prim's grows the MST one vertex at a time, always adding the cheapest edge that connects the growing tree to a new vertex — similar in structure to Dijkstra.

```java
/**
 * Prim's Algorithm using a Min-Priority Queue.
 *
 * Time:  O((V + E) log V)
 * Space: O(V)
 *
 * Strategy: start from any vertex, greedily add the minimum-weight
 *           edge crossing the cut between tree and non-tree vertices.
 */
public static int primMST(Graph graph, int v) {
    int[]     key     = new int[v];      // minimum edge weight to include vertex
    boolean[] inMST   = new boolean[v];
    int[]     parent  = new int[v];

    Arrays.fill(key,    Integer.MAX_VALUE);
    Arrays.fill(parent, -1);
    key[0] = 0; // start from vertex 0

    // Min-heap: {key, vertex}
    PriorityQueue<int[]> pq = new PriorityQueue<>(Comparator.comparingInt(a -> a[0]));
    pq.offer(new int[]{0, 0});

    int totalWeight = 0;

    while (!pq.isEmpty()) {
        int[] curr = pq.poll();
        int u      = curr[1];

        if (inMST[u]) continue;
        inMST[u]    = true;
        totalWeight += key[u];

        for (int[] edge : graph.neighbors(u)) {
            int neighbor = edge[0];
            int weight   = edge[1];

            if (!inMST[neighbor] && weight < key[neighbor]) {
                key[neighbor]    = weight;
                parent[neighbor] = u;
                pq.offer(new int[]{weight, neighbor});
            }
        }
    }

    // Print MST edges
    System.out.println("MST Edges:");
    for (int i = 1; i < v; i++) {
        System.out.println(parent[i] + " — " + i + " (weight " + key[i] + ")");
    }
    return totalWeight;
}
```

### Kruskal's Algorithm

Kruskal's builds the MST by sorting all edges by weight and adding them one by one, skipping any edge that would create a cycle. It uses a **Union-Find** (Disjoint Set Union) data structure to detect cycles efficiently.

```java
/**
 * Kruskal's Algorithm.
 *
 * Time:  O(E log E) — dominated by sorting edges
 * Space: O(V) for union-find
 *
 * Strategy: sort edges by weight, add if it doesn't create a cycle.
 * Better than Prim's for sparse graphs.
 */
public static int kruskalMST(int v, int[][] edges) {
    // Sort edges by weight
    Arrays.sort(edges, Comparator.comparingInt(e -> e[2]));

    UnionFind uf  = new UnionFind(v);
    int totalWeight = 0;
    int edgesAdded  = 0;

    System.out.println("MST Edges:");
    for (int[] edge : edges) {
        int u = edge[0], nb = edge[1], w = edge[2];

        if (uf.find(u) != uf.find(nb)) {    // different components → safe to add
            uf.union(u, nb);
            totalWeight += w;
            edgesAdded++;
            System.out.println(u + " — " + nb + " (weight " + w + ")");
            if (edgesAdded == v - 1) break; // MST complete
        }
    }
    return totalWeight;
}
```

---

## 8.10 Union-Find (Disjoint Set Union)

Union-Find is a data structure that tracks which elements belong to the same **connected component**. It supports two operations in near O(1) time:

- `find(x)` — which component does x belong to?
- `union(x, y)` — merge the components containing x and y

```java
/**
 * Union-Find with Path Compression and Union by Rank.
 *
 * Path Compression: during find(), point every node directly to root.
 * Union by Rank:    always attach smaller tree under larger tree.
 *
 * Amortized time per operation: O(α(n)) ≈ O(1)
 * where α is the inverse Ackermann function (effectively constant).
 */
public class UnionFind {
    private int[] parent;
    private int[] rank;
    private int components;

    public UnionFind(int n) {
        parent     = new int[n];
        rank       = new int[n];
        components = n;
        for (int i = 0; i < n; i++) parent[i] = i; // each node is its own root
    }

    // Find root with path compression
    public int find(int x) {
        if (parent[x] != x)
            parent[x] = find(parent[x]); // compress: point directly to root
        return parent[x];
    }

    // Union by rank — attach smaller tree to larger
    public boolean union(int x, int y) {
        int rootX = find(x), rootY = find(y);
        if (rootX == rootY) return false; // already in same component

        if      (rank[rootX] < rank[rootY]) parent[rootX] = rootY;
        else if (rank[rootX] > rank[rootY]) parent[rootY] = rootX;
        else { parent[rootY] = rootX; rank[rootX]++; }

        components--;
        return true;
    }

    public boolean connected(int x, int y) { return find(x) == find(y); }
    public int components() { return components; }
}
```

### Union-Find Applications

```java
// Check if adding an edge creates a cycle (used in Kruskal's)
UnionFind uf = new UnionFind(5);
uf.union(0, 1);
uf.union(1, 2);
System.out.println(uf.connected(0, 2)); // true — 0 and 2 in same component
System.out.println(uf.connected(0, 3)); // false — 3 not yet connected

// Would adding edge (0,2) create a cycle?
System.out.println(uf.find(0) == uf.find(2)); // true → YES, would create cycle
```

---

## 8.11 Topological Sort

A **topological sort** of a Directed Acyclic Graph (DAG) is a linear ordering of vertices such that for every directed edge u→v, u appears before v in the ordering. This models dependency resolution: task scheduling, build systems, course prerequisites.

```
DAG: Course prerequisites
   CS101 → CS201 → CS301
              ↓
           CS202 → CS401

Topological order: CS101, CS201, CS202, CS301, CS401
(or CS101, CS201, CS301, CS202, CS401 — multiple valid orderings)
```

### Kahn's Algorithm (BFS-based)

```java
/**
 * Topological Sort using Kahn's Algorithm (BFS).
 *
 * Strategy: repeatedly remove vertices with in-degree 0.
 * If the result has fewer than V vertices → cycle exists (not a DAG).
 *
 * Time:  O(V + E)
 * Space: O(V)
 */
public static List<Integer> topologicalSortKahn(int v, List<List<Integer>> adj) {
    int[] inDegree = new int[v];

    // Count incoming edges for each vertex
    for (int u = 0; u < v; u++)
        for (int nb : adj.get(u)) inDegree[nb]++;

    // Start with all vertices that have no prerequisites (in-degree = 0)
    Queue<Integer> queue = new LinkedList<>();
    for (int i = 0; i < v; i++)
        if (inDegree[i] == 0) queue.offer(i);

    List<Integer> order = new ArrayList<>();

    while (!queue.isEmpty()) {
        int node = queue.poll();
        order.add(node);

        // "Remove" this node: reduce in-degree of its neighbors
        for (int neighbor : adj.get(node)) {
            inDegree[neighbor]--;
            if (inDegree[neighbor] == 0) queue.offer(neighbor); // no more prerequisites
        }
    }

    if (order.size() != v) {
        System.out.println("Cycle detected — not a DAG!");
        return new ArrayList<>();
    }
    return order;
}
```

### DFS-based Topological Sort

```java
/**
 * Topological Sort using DFS.
 * After fully exploring all descendants of a node, push it to a stack.
 * Pop the stack at the end for topological order.
 *
 * Time: O(V + E)
 */
public static List<Integer> topologicalSortDFS(int v, List<List<Integer>> adj) {
    boolean[] visited = new boolean[v];
    Deque<Integer> stack = new ArrayDeque<>();

    for (int i = 0; i < v; i++)
        if (!visited[i]) dfsTopoHelper(i, adj, visited, stack);

    List<Integer> order = new ArrayList<>();
    while (!stack.isEmpty()) order.add(stack.pop());
    return order;
}

private static void dfsTopoHelper(int node, List<List<Integer>> adj,
                                   boolean[] visited, Deque<Integer> stack) {
    visited[node] = true;
    for (int neighbor : adj.get(node))
        if (!visited[neighbor]) dfsTopoHelper(neighbor, adj, visited, stack);
    stack.push(node); // push AFTER visiting all descendants
}
```

---

## 8.12 Cycle Detection

### Cycle Detection in Undirected Graphs (Union-Find)

```java
/**
 * Detect cycle in undirected graph using Union-Find.
 * If both endpoints of an edge are already in the same component → cycle.
 *
 * Time: O(E * α(V)) ≈ O(E)
 */
public static boolean hasCycleUndirected(int v, int[][] edges) {
    UnionFind uf = new UnionFind(v);
    for (int[] edge : edges) {
        if (uf.find(edge[0]) == uf.find(edge[1])) return true; // cycle!
        uf.union(edge[0], edge[1]);
    }
    return false;
}
```

### Cycle Detection in Directed Graphs (DFS with Color Marking)

```java
/**
 * Detect cycle in directed graph using DFS with 3-color marking.
 *
 * WHITE (0): not yet visited
 * GRAY  (1): currently being processed (in current DFS path)
 * BLACK (2): fully processed
 *
 * If we reach a GRAY node → we've found a back edge → cycle!
 *
 * Time: O(V + E)
 */
public static boolean hasCycleDirected(int v, List<List<Integer>> adj) {
    int[] color = new int[v]; // 0=WHITE, 1=GRAY, 2=BLACK

    for (int i = 0; i < v; i++)
        if (color[i] == 0 && dfsColorCycle(i, adj, color)) return true;

    return false;
}

private static boolean dfsColorCycle(int node, List<List<Integer>> adj, int[] color) {
    color[node] = 1; // GRAY — currently visiting

    for (int neighbor : adj.get(node)) {
        if (color[neighbor] == 1) return true;   // back edge → cycle
        if (color[neighbor] == 0 && dfsColorCycle(neighbor, adj, color)) return true;
    }

    color[node] = 2; // BLACK — fully processed
    return false;
}
```

---

## 8.13 Strongly Connected Components — Kosaraju's Algorithm

In a directed graph, a **Strongly Connected Component (SCC)** is a maximal set of vertices where every vertex is reachable from every other vertex within the set.

```
Graph:    0→1→2→0   (cycle: SCC = {0,1,2})
          2→3       (3 reaches nothing back: SCC = {3})

SCCs: {0, 1, 2} and {3}
```

```java
/**
 * Kosaraju's Algorithm for Strongly Connected Components.
 *
 * Step 1: DFS on original graph, push vertices to stack in finish order
 * Step 2: Transpose (reverse) the graph
 * Step 3: DFS on transposed graph in stack order — each DFS tree = one SCC
 *
 * Time:  O(V + E)
 * Space: O(V + E)
 */
public static List<List<Integer>> kosarajuSCC(int v, List<List<Integer>> adj) {
    // Step 1: DFS and record finish order
    boolean[] visited = new boolean[v];
    Deque<Integer> stack = new ArrayDeque<>();
    for (int i = 0; i < v; i++)
        if (!visited[i]) dfsFinishOrder(i, adj, visited, stack);

    // Step 2: Transpose the graph
    List<List<Integer>> transposed = new ArrayList<>();
    for (int i = 0; i < v; i++) transposed.add(new ArrayList<>());
    for (int u = 0; u < v; u++)
        for (int nb : adj.get(u)) transposed.get(nb).add(u); // reverse edge

    // Step 3: DFS on transposed graph in finish order
    Arrays.fill(visited, false);
    List<List<Integer>> sccs = new ArrayList<>();

    while (!stack.isEmpty()) {
        int node = stack.pop();
        if (!visited[node]) {
            List<Integer> scc = new ArrayList<>();
            dfsCollect(node, transposed, visited, scc);
            sccs.add(scc);
        }
    }
    return sccs;
}

private static void dfsFinishOrder(int node, List<List<Integer>> adj,
                                    boolean[] visited, Deque<Integer> stack) {
    visited[node] = true;
    for (int nb : adj.get(node))
        if (!visited[nb]) dfsFinishOrder(nb, adj, visited, stack);
    stack.push(node);
}

private static void dfsCollect(int node, List<List<Integer>> adj,
                                boolean[] visited, List<Integer> scc) {
    visited[node] = true;
    scc.add(node);
    for (int nb : adj.get(node))
        if (!visited[nb]) dfsCollect(nb, adj, visited, scc);
}
```

---

## 8.14 Complete Example — Graph Problem Showcase

```java
import java.util.*;

public class GraphShowcase {

    public static void main(String[] args) {

        // ── Build a weighted undirected graph ─────────────────────
        //   0 ──1── 1
        //   |       |
        //   4       2
        //   |       |
        //   3 ──1── 2
        //    \     /
        //      3  (diagonal)

        int V = 4;
        Graph g = new Graph(V, false);
        g.addEdge(0, 1, 1);
        g.addEdge(1, 2, 2);
        g.addEdge(2, 3, 1);
        g.addEdge(0, 3, 4);
        g.addEdge(0, 2, 3); // cross edge

        // ── Traversals ────────────────────────────────────────────
        System.out.print("DFS from 0: ");
        dfsRecursive(g, 0, V);            // 0 1 2 3

        System.out.print("\nBFS from 0: ");
        bfs(g, 0, V);                     // 0 1 3 2

        // ── Shortest paths ────────────────────────────────────────
        int[] dist = dijkstra(g, 0, V);
        System.out.println("\nDijkstra from 0: " + Arrays.toString(dist));
        // [0, 1, 3, 4]

        // ── MST ───────────────────────────────────────────────────
        int[][] edges = {{0,1,1},{1,2,2},{2,3,1},{0,3,4},{0,2,3}};
        System.out.println("\nKruskal's MST:");
        int mstWeight = kruskalMST(V, edges);
        System.out.println("Total MST weight: " + mstWeight); // 4

        // ── Topological sort ──────────────────────────────────────
        // DAG: 5→2, 5→0, 4→0, 4→1, 2→3, 3→1
        List<List<Integer>> dag = new ArrayList<>();
        for (int i = 0; i < 6; i++) dag.add(new ArrayList<>());
        dag.get(5).add(2); dag.get(5).add(0);
        dag.get(4).add(0); dag.get(4).add(1);
        dag.get(2).add(3); dag.get(3).add(1);

        List<Integer> topo = topologicalSortKahn(6, dag);
        System.out.println("\nTopological order: " + topo);
        // [4, 5, 0, 2, 3, 1] or similar valid ordering

        // ── Cycle detection ──────────────────────────────────────
        int[][] noCycle  = {{0,1},{1,2},{2,3}};
        int[][] hasCycle = {{0,1},{1,2},{2,0}};
        System.out.println("\nHas cycle (no):  " + hasCycleUndirected(4, noCycle));  // false
        System.out.println("Has cycle (yes): " + hasCycleUndirected(3, hasCycle));   // true
    }
}
```

---

## 8.15 Advanced Graph Problems — Quick Reference

### Bipartite Graph Check

```java
/**
 * A graph is bipartite if vertices can be 2-colored such that
 * no two adjacent vertices share the same color.
 * Equivalent to: graph has no odd-length cycles.
 * Uses BFS with 2-coloring.
 * Time: O(V + E)
 */
public static boolean isBipartite(int v, List<List<Integer>> adj) {
    int[] color = new int[v]; // 0=uncolored, 1=red, 2=blue
    for (int start = 0; start < v; start++) {
        if (color[start] != 0) continue;
        Queue<Integer> queue = new LinkedList<>();
        queue.offer(start);
        color[start] = 1;
        while (!queue.isEmpty()) {
            int node = queue.poll();
            for (int nb : adj.get(node)) {
                if (color[nb] == 0) {
                    color[nb] = 3 - color[node]; // alternate: 1↔2
                    queue.offer(nb);
                } else if (color[nb] == color[node]) {
                    return false; // same color adjacent → not bipartite
                }
            }
        }
    }
    return true;
}
```

### Number of Islands (Grid as Graph)

```java
/**
 * Count connected components of '1's in a 2D grid.
 * Treat each cell as a graph node; edges connect adjacent '1' cells.
 * Uses DFS to flood-fill each island.
 * Time: O(rows * cols)
 */
public static int numIslands(char[][] grid) {
    int islands = 0;
    for (int r = 0; r < grid.length; r++) {
        for (int c = 0; c < grid[0].length; c++) {
            if (grid[r][c] == '1') {
                islands++;
                floodFill(grid, r, c); // sink the entire island
            }
        }
    }
    return islands;
}

private static void floodFill(char[][] grid, int r, int c) {
    if (r < 0 || r >= grid.length || c < 0 || c >= grid[0].length
            || grid[r][c] != '1') return;
    grid[r][c] = '0';                  // mark visited
    floodFill(grid, r+1, c);
    floodFill(grid, r-1, c);
    floodFill(grid, r, c+1);
    floodFill(grid, r, c-1);
}
```

---

## 8.16 Graph Algorithm Complexity Summary

| Algorithm               | Time Complexity        | Space      | Notes                              |
|-------------------------|------------------------|------------|------------------------------------|
| DFS                     | O(V + E)               | O(V)       | Stack / recursion                  |
| BFS                     | O(V + E)               | O(V)       | Queue; shortest path (unweighted)  |
| Dijkstra                | O((V+E) log V)         | O(V)       | Non-negative weights only          |
| Bellman-Ford            | O(V * E)               | O(V)       | Handles negative weights/cycles    |
| Floyd-Warshall          | O(V³)                  | O(V²)      | All-pairs; dense graphs            |
| Prim's MST              | O((V+E) log V)         | O(V)       | Dense graphs                       |
| Kruskal's MST           | O(E log E)             | O(V)       | Sparse graphs                      |
| Union-Find ops          | O(α(V)) ≈ O(1)         | O(V)       | Path compression + rank            |
| Topological Sort        | O(V + E)               | O(V)       | DAGs only                          |
| Kosaraju's SCC          | O(V + E)               | O(V + E)   | Two DFS passes                     |
| Cycle Detection         | O(V + E)               | O(V)       | Union-Find or DFS coloring         |

---

## 8.17 Key Takeaways

| Concept                 | Summary                                                                          |
|-------------------------|----------------------------------------------------------------------------------|
| Graph                   | Vertices connected by edges; models any network relationship                     |
| Adjacency List          | O(V+E) space; preferred for sparse graphs and most algorithms                   |
| Adjacency Matrix        | O(V²) space; O(1) edge lookup; preferred for dense graphs                       |
| DFS                     | Stack-based; explores depth-first; cycle detection, topological sort            |
| BFS                     | Queue-based; explores width-first; shortest path in unweighted graphs           |
| Dijkstra                | Greedy shortest path; O((V+E) log V); non-negative weights only                 |
| Bellman-Ford            | DP shortest path; O(VE); handles negative weights, detects negative cycles      |
| Floyd-Warshall          | All-pairs DP; O(V³); best when you need every pair's shortest path              |
| Prim's / Kruskal's      | Both find MST in O((V+E) log V); Kruskal's simpler for sparse graphs            |
| Union-Find              | Near O(1) connected-component queries; backbone of Kruskal's                    |
| Topological Sort        | Linear ordering of DAG vertices; Kahn's (BFS) or DFS-stack approach            |
| Cycle Detection         | Union-Find for undirected; DFS color marking for directed                       |

---

## 8.18 Practice Problems

1. **Course Schedule** — Given `n` courses and a list of prerequisite pairs `[a, b]` meaning "must take b before a", determine if it is possible to finish all courses. *(Hint: topological sort or cycle detection in a DAG.)*

2. **Network Delay Time** — There are `n` network nodes and weighted directed edges representing transmission times. Find the minimum time for a signal sent from node `k` to reach all nodes, or -1 if not all nodes are reachable. *(Hint: Dijkstra.)*

3. **Cheapest Flights Within K Stops** — Find the cheapest flight from `src` to `dst` with at most `k` stops. *(Hint: modified Bellman-Ford with k+1 relaxation rounds.)*

4. **Redundant Connection** — Given an undirected graph that started as a tree and had one extra edge added, find and return that extra edge. *(Hint: Union-Find — the first edge connecting two already-connected vertices is the answer.)*

5. **Critical Connections (Bridges)** — Find all edges whose removal disconnects the graph. *(Hint: Tarjan's bridge-finding algorithm using DFS discovery times and low values.)*

---

## 8.19 Course Complete — What's Next?

Congratulations on completing CS 3303 Data Structures! You have now mastered:

| Module | Topic                                    |
|--------|------------------------------------------|
| 1      | Fundamentals and Algorithm Analysis      |
| 2      | Arrays and Linked Lists                  |
| 3      | Stacks and Queues                        |
| 4      | Binary Trees and Tree Traversals         |
| 5      | Advanced Trees and Recursion             |
| 6      | Searching and Pattern Matching           |
| 7      | Sorting Algorithms                       |
| 8      | Graphs and Advanced Graph Algorithms     |

**Recommended next courses:**
- **Algorithm Design** — dynamic programming, greedy algorithms, divide and conquer
- **Database Systems** — B-trees, indexing, query optimization
- **Operating Systems** — scheduling algorithms, memory management
- **Networks** — routing protocols (OSPF uses Dijkstra), spanning trees (STP uses MST)

---

## 8.20 Further Reading

- *Introduction to Algorithms* (CLRS) — Chapters 22–25 (Graph algorithms)
- *Algorithms* — Sedgewick & Wayne, Chapter 4 (Graphs)
- Graph visualizer: [https://visualgo.net/en/graphds](https://visualgo.net/en/graphds)
- Dijkstra visualizer: [https://visualgo.net/en/sssp](https://visualgo.net/en/sssp)
- LeetCode graph problems: [https://leetcode.com/tag/graph/](https://leetcode.com/tag/graph/)

---

*CS 3303 Data Structures — Module 8 of 8 — Course Complete*
*Previous: Module 7 — Sorting Algorithms*
