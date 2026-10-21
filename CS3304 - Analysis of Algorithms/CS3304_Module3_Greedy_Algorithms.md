# CS 3304 — Analysis of Algorithms
## Module 3: Advanced Algorithm Techniques Part 2 — Greedy Algorithms

---

## 3.1 Introduction

In Divide and Conquer, we solve a problem by splitting it into independent subproblems and combining their results. In **Greedy algorithms**, we take a fundamentally different approach:

> At each step, make the choice that looks **best right now** — without reconsidering past choices or exploring future consequences.

This sounds dangerously shortsighted. And for many problems, it is — a greedy approach gives the wrong answer. But for a special, important class of problems, the greedy choice is always globally optimal. The intellectual challenge of this module is learning to recognize that class and to **prove** when greediness is safe.

Greedy algorithms are often the fastest possible solution — many run in O(n log n) or O(n). They underpin some of the most important algorithms in computer science: Dijkstra's shortest path, Huffman compression, Prim's and Kruskal's MST, and job scheduling systems.

By the end of this module you will be able to:
- Identify the two properties that make a problem greedy-solvable
- Prove greedy correctness using exchange arguments and induction
- Implement and analyze seven classic greedy algorithms
- Know exactly when greedy fails — and why

---

## 3.2 The Greedy Paradigm

### Structure of a Greedy Algorithm

```
greedy(problem):
    solution = empty
    candidates = all possible choices

    while problem not solved:
        choice = selectBest(candidates)   // greedy selection
        if choice is feasible:
            solution = solution + choice  // irrevocable commitment
            candidates = candidates − {choice}

    return solution
```

Three key properties of a greedy algorithm:
1. **Greedy choice**: always picks the locally optimal option
2. **Irrevocable**: never goes back and reverses a decision
3. **Incremental**: builds the solution piece by piece

### Two Properties Required for Correctness

A greedy algorithm produces an optimal solution if and only if the problem has:

**Property 1 — Greedy Choice Property:**
A globally optimal solution can always be reached by making the locally optimal (greedy) choice at each step. The greedy choice never needs to be revised.

**Property 2 — Optimal Substructure:**
An optimal solution to the problem contains optimal solutions to its subproblems.

If either property fails, greedy will give the wrong answer. Dynamic programming (Module 4) handles problems with optimal substructure but without the greedy choice property.

---

## 3.3 Proving Greedy Correctness

There are two standard proof techniques for greedy algorithms.

### Technique 1: Exchange Argument

Show that any optimal solution can be transformed into the greedy solution without making it worse.

**Template:**
1. Assume an optimal solution OPT that differs from the greedy solution GREEDY
2. Find the first point where they differ
3. Show you can "swap" the OPT choice with the GREEDY choice and the solution doesn't get worse
4. By repeated exchanges, OPT can be transformed into GREEDY — so GREEDY is also optimal

### Technique 2: Greedy Stays Ahead

Show that after each step, the greedy solution is at least as good as any other partial solution.

**Template:**
1. Define a measure of "how good" a partial solution is after k steps
2. Prove by induction that greedy's measure is ≥ any other algorithm's measure after every step
3. At termination, greedy is therefore optimal

We will see both techniques applied concretely in the algorithms below.

---

## 3.4 Activity Selection Problem

**Problem:** Given n activities, each with a start time s[i] and finish time f[i], select the maximum number of non-overlapping activities.

```
Activities: (s, f) = (1,4), (3,5), (0,6), (5,7), (3,9), (5,9), (6,10), (8,11), (8,12), (2,14), (12,16)

After sorting by finish time:
(1,4), (3,5), (0,6), (5,7), (3,9), (5,9), (6,10), (8,11), (8,12), (2,14), (12,16)

Greedy (always pick earliest-finishing compatible activity):
Select (1,4) → next must start ≥ 4
Select (5,7) → next must start ≥ 7
Select (8,11) → next must start ≥ 11
Select (12,16) → done

4 activities selected: optimal!
```

### Correctness Proof (Exchange Argument)

Let A = greedy solution, O = some other optimal solution.
Sort both by finish time. Let activity a_k be the first activity where they differ.
- Greedy chose a_k (earliest finishing at that point)
- OPT chose some o_k where f[o_k] ≥ f[a_k]

We can replace o_k with a_k in OPT:
- a_k doesn't overlap what came before (greedy chose it as compatible)
- a_k finishes no later than o_k, so it doesn't create new conflicts with later activities
- |OPT| doesn't decrease

Repeat until OPT = greedy. Therefore greedy is optimal. ✓

```java
/**
 * Activity Selection — Greedy by earliest finish time.
 *
 * Time:  O(n log n) for sorting + O(n) selection = O(n log n)
 * Space: O(n)
 */
public static List<int[]> activitySelection(int[] start, int[] finish) {
    int n = finish.length;

    // Create and sort activities by finish time
    Integer[] idx = new Integer[n];
    for (int i = 0; i < n; i++) idx[i] = i;
    Arrays.sort(idx, Comparator.comparingInt(i -> finish[i]));

    List<int[]> selected = new ArrayList<>();
    int lastFinish = Integer.MIN_VALUE;

    for (int i : idx) {
        // GREEDY CHOICE: pick this activity if it doesn't overlap
        if (start[i] >= lastFinish) {
            selected.add(new int[]{start[i], finish[i]});
            lastFinish = finish[i]; // irrevocable commitment
        }
    }
    return selected;
}

public static void main(String[] args) {
    int[] s = {1, 3, 0, 5, 3, 5, 6, 8, 8,  2, 12};
    int[] f = {4, 5, 6, 7, 9, 9,10,11,12, 14, 16};

    List<int[]> result = activitySelection(s, f);
    System.out.println("Selected " + result.size() + " activities:");
    for (int[] a : result) System.out.println("  (" + a[0] + ", " + a[1] + ")");
    // Selected 4 activities: (1,4) (5,7) (8,11) (12,16)
}
```

### Why "Sort by Finish Time" and not Start Time?

```
Counter-example for "earliest start":
Activities: (0,10), (1,3), (1,3)

Earliest start picks (0,10) first — only 1 activity selected.
Earliest finish picks (1,3) first — 2 activities selected. ✓

Counter-example for "shortest duration":
Activities: (1,3), (2,4), (3,5)   durations: 2, 2, 2

Shortest duration picks (1,3) and (3,5) — 2 activities. Same as finish-time.
But try: (1,4), (2,3), (3,5)  durations: 3, 1, 2

Shortest: picks (2,3) then... (1,4) conflicts! Only (3,5) left → 2 total.
Finish-time: picks (2,3) then (3,5) → also 2. OK here.
But in general, shortest duration is not always optimal.
```

---

## 3.5 Fractional Knapsack

**Problem:** A thief has a knapsack of capacity W. There are n items, each with weight w[i] and value v[i]. Items can be taken **fractionally**. Maximize total value.

**Greedy strategy:** Always take as much as possible of the item with the highest value-per-unit-weight (v[i]/w[i]).

```
Items: (weight, value) = (10,60), (20,100), (30,120)
Capacity W = 50

Value/weight: 6, 5, 4

Greedy order: item 1 (ratio 6), item 2 (ratio 5), item 3 (ratio 4)
Take all of item 1: weight=10, value=60, remaining=40
Take all of item 2: weight=20, value=100, remaining=20
Take 20/30 of item 3: value=120×(20/30)=80, remaining=0

Total value = 60 + 100 + 80 = 240  ← optimal
```

```java
/**
 * Fractional Knapsack — Greedy by value density.
 *
 * Time:  O(n log n) for sorting
 * Space: O(n)
 *
 * NOTE: This greedy ONLY works for the FRACTIONAL version.
 *       The 0/1 Knapsack (no fractions) requires Dynamic Programming.
 */
public static double fractionalKnapsack(int[] weights, int[] values, int W) {
    int n = weights.length;

    // Create items with value density and sort descending
    double[][] items = new double[n][3]; // {valueDensity, weight, value}
    for (int i = 0; i < n; i++) {
        items[i][0] = (double) values[i]  / weights[i]; // value per unit weight
        items[i][1] = weights[i];
        items[i][2] = values[i];
    }
    Arrays.sort(items, (a, b) -> Double.compare(b[0], a[0])); // descending density

    double totalValue  = 0;
    int    remaining   = W;

    for (double[] item : items) {
        if (remaining <= 0) break;

        double density = item[0];
        int    weight  = (int) item[1];
        int    value   = (int) item[2];

        if (weight <= remaining) {
            // GREEDY CHOICE: take the entire item
            totalValue += value;
            remaining  -= weight;
        } else {
            // Take fractional amount to fill remaining capacity
            totalValue += density * remaining;
            remaining   = 0;
        }
    }
    return totalValue;
}

// weights=[10,20,30], values=[60,100,120], W=50 → 240.0
```

### Why Greedy Fails for 0/1 Knapsack

```
Items: (weight=10, value=60), (weight=20, value=100), (weight=30, value=120)
Capacity W = 50

0/1 Knapsack — greedy by density picks item 1 (density 6):
  Take item 1 (w=10, v=60), remaining=40
  Take item 2 (w=20, v=100), remaining=20
  Item 3 needs w=30 > 20 — can't take fraction → skip
  Total = 160

Optimal for 0/1:
  Take items 2 and 3: 100 + 120 = 220 ← better!

Greedy fails because items cannot be split. This requires DP (Module 4).
```

---

## 3.6 Huffman Coding

**Problem:** Assign variable-length binary codes to characters such that more frequent characters get shorter codes and the total encoded length is minimized.

This is the foundation of lossless compression (ZIP, GZIP, JPEG use Huffman as a component).

### Building the Huffman Tree

```
Greedy strategy: repeatedly merge the two lowest-frequency nodes.

Characters and frequencies:
  a:45, b:13, c:12, d:16, e:9, f:5

Step 1: Min-heap = [(5,f),(9,e),(12,c),(13,b),(16,d),(45,a)]
Step 2: Merge f(5) + e(9) → internal(14):  [(12,c),(13,b),(14,fe),(16,d),(45,a)]
Step 3: Merge c(12) + b(13) → internal(25): [(14,fe),(16,d),(25,cb),(45,a)]
Step 4: Merge fe(14) + d(16) → internal(30): [(25,cb),(30,fed),(45,a)]
Step 5: Merge cb(25) + fed(30) → internal(55): [(45,a),(55,cbfed)]
Step 6: Merge a(45) + cbfed(55) → root(100)

Tree:
              [100]
             /     \
          [45:a]  [55]
                  /    \
               [25]    [30]
               / \     /  \
            [12:c][13:b][14][16:d]
                        / \
                      [5:f][9:e]

Codes: a=0, c=100, b=101, f=1100, e=1101, d=111
```

```java
import java.util.*;

/**
 * Huffman Coding — Greedy via min-heap.
 *
 * Time:  O(n log n) — n-1 merges, each O(log n) heap operation
 * Space: O(n) for the tree
 *
 * OPTIMALITY PROOF (sketch):
 * The two symbols with the lowest frequency must be deepest in any
 * optimal tree (if they were higher up, we could swap them with deeper
 * nodes to reduce total length). The greedy merge of the two smallest
 * frequencies satisfies this — proof by exchange argument.
 */
public class HuffmanCoding {

    static class HuffNode implements Comparable<HuffNode> {
        char   ch;
        int    freq;
        HuffNode left, right;

        HuffNode(char ch, int freq) {
            this.ch = ch; this.freq = freq;
        }
        HuffNode(int freq, HuffNode left, HuffNode right) {
            this.ch = '\0'; this.freq = freq;
            this.left = left; this.right = right;
        }

        @Override public int compareTo(HuffNode o) {
            return this.freq - o.freq; // min-heap by frequency
        }

        boolean isLeaf() { return left == null && right == null; }
    }

    // Build the Huffman tree
    public static HuffNode buildTree(char[] chars, int[] freqs) {
        PriorityQueue<HuffNode> pq = new PriorityQueue<>();
        for (int i = 0; i < chars.length; i++) {
            pq.offer(new HuffNode(chars[i], freqs[i]));
        }

        while (pq.size() > 1) {
            HuffNode left  = pq.poll(); // lowest frequency
            HuffNode right = pq.poll(); // second lowest

            // GREEDY CHOICE: merge two smallest into a new internal node
            pq.offer(new HuffNode(left.freq + right.freq, left, right));
        }
        return pq.poll(); // root of Huffman tree
    }

    // Generate codes by traversing the tree
    // '0' = go left, '1' = go right
    public static Map<Character, String> generateCodes(HuffNode root) {
        Map<Character, String> codes = new HashMap<>();
        generateCodesRec(root, "", codes);
        return codes;
    }

    private static void generateCodesRec(HuffNode node, String code,
                                          Map<Character, String> codes) {
        if (node == null)    return;
        if (node.isLeaf())   { codes.put(node.ch, code.isEmpty() ? "0" : code); return; }

        generateCodesRec(node.left,  code + "0", codes);
        generateCodesRec(node.right, code + "1", codes);
    }

    // Encode a string using the generated codes
    public static String encode(String text, Map<Character, String> codes) {
        StringBuilder sb = new StringBuilder();
        for (char c : text.toCharArray()) sb.append(codes.get(c));
        return sb.toString();
    }

    // Decode a bit string using the Huffman tree
    public static String decode(String bits, HuffNode root) {
        StringBuilder sb = new StringBuilder();
        HuffNode curr = root;
        for (char bit : bits.toCharArray()) {
            curr = (bit == '0') ? curr.left : curr.right;
            if (curr.isLeaf()) { sb.append(curr.ch); curr = root; }
        }
        return sb.toString();
    }

    // Compute compression ratio
    public static void analyzeCompression(char[] chars, int[] freqs,
                                           Map<Character, String> codes) {
        int totalChars  = 0, fixedBits = 0, huffBits = 0;
        int bitsNeeded  = (int) Math.ceil(Math.log(chars.length) / Math.log(2));

        for (int i = 0; i < chars.length; i++) {
            totalChars += freqs[i];
            fixedBits  += freqs[i] * bitsNeeded;
            huffBits   += freqs[i] * codes.get(chars[i]).length();
        }
        System.out.printf("Fixed encoding:   %d bits (%d bits/char)%n",
                          fixedBits, bitsNeeded);
        System.out.printf("Huffman encoding: %d bits%n", huffBits);
        System.out.printf("Compression ratio: %.1f%%%n",
                          100.0 * (1 - (double) huffBits / fixedBits));
    }

    public static void main(String[] args) {
        char[] chars = {'a', 'b', 'c', 'd', 'e', 'f'};
        int[]  freqs = {45,  13,  12,  16,   9,   5};

        HuffNode           root  = buildTree(chars, freqs);
        Map<Character, String> codes = generateCodes(root);

        System.out.println("Huffman Codes:");
        codes.forEach((ch, code) ->
            System.out.printf("  '%c' (freq=%d): %s%n", ch,
                freqs[new String(chars).indexOf(ch)], code));

        String text    = "abacabad";
        String encoded = encode(text, codes);
        String decoded = decode(encoded, root);
        System.out.println("\nOriginal:  " + text);
        System.out.println("Encoded:   " + encoded);
        System.out.println("Decoded:   " + decoded);
        System.out.println("Bits used: " + encoded.length() +
                           " vs fixed: " + text.length() * 3);

        System.out.println();
        analyzeCompression(chars, freqs, codes);
    }
    /*
    Huffman Codes:
      'a' (freq=45): 0
      'b' (freq=13): 101
      'c' (freq=12): 100
      'd' (freq=16): 111
      'e' (freq=9):  1101
      'f' (freq=5):  1100

    Compression ratio: 35.4%
    */
}
```

---

## 3.7 Job Scheduling — Minimize Lateness

**Problem:** n jobs, each with a deadline d[i] and processing time t[i]. Schedule all jobs on a single machine to minimize **maximum lateness** (where lateness = max(0, completion_time - deadline)).

**Greedy strategy: Earliest Deadline First (EDF)** — schedule jobs in order of increasing deadline.

```
Jobs: (processing time, deadline)
  Job 1: (3, 6)
  Job 2: (2, 9)
  Job 3: (1, 8)
  Job 4: (4, 9)
  Job 5: (3, 14)
  Job 6: (2, 15)

EDF order by deadline: Job1(d=6), Job3(d=8), Job2(d=9), Job4(d=9), Job5(d=14), Job6(d=15)

Schedule:
  Job1: [0, 3]   finish=3,  deadline=6,  lateness = max(0, 3-6)  = 0
  Job3: [3, 4]   finish=4,  deadline=8,  lateness = max(0, 4-8)  = 0
  Job2: [4, 6]   finish=6,  deadline=9,  lateness = max(0, 6-9)  = 0
  Job4: [6,10]   finish=10, deadline=9,  lateness = max(0, 10-9) = 1
  Job5: [10,13]  finish=13, deadline=14, lateness = max(0, 13-14)= 0
  Job6: [13,15]  finish=15, deadline=15, lateness = max(0, 15-15)= 0

Maximum lateness = 1  (optimal!)
```

```java
/**
 * Job Scheduling to Minimize Maximum Lateness.
 * Greedy: Earliest Deadline First (EDF).
 *
 * Time:  O(n log n) for sorting
 * Space: O(n)
 *
 * CORRECTNESS (Exchange Argument):
 * An "inversion" occurs when job i is scheduled before j but d[j] < d[i].
 * Swapping an adjacent inverted pair never increases maximum lateness.
 * (If both meet their deadlines after swap, max lateness unchanged.
 *  If j was late before the swap, swapping makes j earlier — lateness ≤.)
 * EDF has no inversions → it is optimal. ✓
 */
public static int minimizeLateness(int[] processingTime, int[] deadline) {
    int n = deadline.length;

    // Create and sort jobs by deadline (EDF)
    Integer[] jobs = new Integer[n];
    for (int i = 0; i < n; i++) jobs[i] = i;
    Arrays.sort(jobs, Comparator.comparingInt(i -> deadline[i]));

    int maxLateness = 0;
    int currentTime = 0;

    System.out.println("Schedule (EDF):");
    for (int j : jobs) {
        int start  = currentTime;
        int finish = currentTime + processingTime[j];
        int late   = Math.max(0, finish - deadline[j]);

        System.out.printf("  Job %d: [%2d, %2d]  deadline=%d  lateness=%d%n",
                          j+1, start, finish, deadline[j], late);

        maxLateness  = Math.max(maxLateness, late);
        currentTime += processingTime[j]; // GREEDY: commit to this job
    }

    return maxLateness;
}
```

---

## 3.8 Dijkstra's Algorithm — Greedy Shortest Path

Dijkstra's algorithm (Module 8 of CS 3303) is a quintessential greedy algorithm. At each step it greedily selects the unvisited vertex with the smallest known distance — and this greedy choice is provably correct.

```
GREEDY CHOICE: "The unvisited vertex with the smallest tentative distance
                has its shortest distance finalized."

WHY IT'S CORRECT:
Any path to u that doesn't go through the greedy vertex v must pass through
some other unvisited vertex w with dist[w] ≥ dist[v]. Since all edge weights
are non-negative, that path is ≥ dist[v]. So the greedy choice is optimal.

(This is precisely why Dijkstra fails with negative edges —
 the greedy choice property breaks when weights can be negative.)
```

```java
/**
 * Dijkstra's Algorithm — Greedy via min-priority queue.
 *
 * Time:  O((V + E) log V)
 * Space: O(V)
 *
 * GREEDY PROOF (Greedy Stays Ahead):
 * Claim: When vertex u is extracted from the PQ, dist[u] is the true
 *        shortest distance from source.
 *
 * Proof by induction on the number of extracted vertices:
 * Base: source extracted with dist=0. Correct. ✓
 * Step: Assume all previously extracted vertices have correct distances.
 *       Vertex u is next. Suppose there's a shorter path P to u.
 *       P must pass through some unvisited vertex w.
 *       dist[w] ≥ dist[u] (u was chosen as minimum).
 *       Length(P) ≥ dist[w] ≥ dist[u] (non-negative weights).
 *       Contradiction. So dist[u] is correct. ✓
 */
public static int[] dijkstra(int[][] graph, int src) {
    int V    = graph.length;
    int[] dist    = new int[V];
    boolean[] vis = new boolean[V];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[src] = 0;

    // Min-heap: {distance, vertex}
    PriorityQueue<int[]> pq = new PriorityQueue<>(Comparator.comparingInt(a -> a[0]));
    pq.offer(new int[]{0, src});

    while (!pq.isEmpty()) {
        int[] curr = pq.poll();
        int u      = curr[1];
        if (vis[u]) continue;
        vis[u] = true; // GREEDY CHOICE: finalize this vertex

        for (int v = 0; v < V; v++) {
            if (!vis[v] && graph[u][v] != 0 &&
                dist[u] + graph[u][v] < dist[v]) {
                dist[v] = dist[u] + graph[u][v]; // relax edge
                pq.offer(new int[]{dist[v], v});
            }
        }
    }
    return dist;
}
```

---

## 3.9 Prim's and Kruskal's — Greedy MST

Both MST algorithms are greedy. Their proofs rely on the **Cut Property**:

> **Cut Property:** For any cut (S, V−S) of a graph, the minimum weight edge crossing the cut belongs to some MST.

### Prim's — Greedy by Cheapest Edge Crossing the Cut

```
GREEDY CHOICE: Always add the minimum-weight edge that
               connects the growing tree to a new vertex.

WHY CORRECT (Cut Property):
At each step, S = vertices in tree, V−S = remaining vertices.
The minimum edge crossing (S, V−S) is safe to add by the Cut Property.
```

```java
/**
 * Prim's MST — Greedy via min-priority queue.
 *
 * Time:  O((V + E) log V)
 * Space: O(V)
 */
public static int primMST(int[][] graph) {
    int V      = graph.length;
    int[]     key     = new int[V];    // min edge weight to include vertex
    boolean[] inMST   = new boolean[V];
    int[]     parent  = new int[V];

    Arrays.fill(key,    Integer.MAX_VALUE);
    Arrays.fill(parent, -1);
    key[0] = 0;

    PriorityQueue<int[]> pq = new PriorityQueue<>(Comparator.comparingInt(a -> a[0]));
    pq.offer(new int[]{0, 0});

    int totalWeight = 0;

    while (!pq.isEmpty()) {
        int u = pq.poll()[1];
        if (inMST[u]) continue;
        inMST[u]    = true;           // GREEDY CHOICE: commit to this vertex
        totalWeight += key[u];

        for (int v = 0; v < V; v++) {
            if (graph[u][v] != 0 && !inMST[v] && graph[u][v] < key[v]) {
                key[v]    = graph[u][v];
                parent[v] = u;
                pq.offer(new int[]{key[v], v});
            }
        }
    }

    System.out.println("Prim's MST edges:");
    for (int i = 1; i < V; i++)
        System.out.printf("  %d — %d (weight %d)%n", parent[i], i, key[i]);

    return totalWeight;
}
```

### Kruskal's — Greedy by Global Cheapest Safe Edge

```
GREEDY CHOICE: Sort all edges by weight. Add the cheapest edge that
               doesn't create a cycle (connecting two different components).

WHY CORRECT (Cut Property):
When we consider edge (u,v), the cut S = component containing u,
V−S = everything else. Edge (u,v) is the lightest crossing edge
(we process by sorted weight and it doesn't create a cycle).
By the Cut Property, it belongs to some MST.
```

```java
/**
 * Kruskal's MST — Greedy with Union-Find.
 *
 * Time:  O(E log E) for sorting
 * Space: O(V) for Union-Find
 */
public static int kruskalMST(int V, int[][] edges) {
    // Sort edges by weight
    Arrays.sort(edges, Comparator.comparingInt(e -> e[2]));

    UnionFind uf    = new UnionFind(V);
    int totalWeight = 0;
    int edgesAdded  = 0;

    System.out.println("Kruskal's MST edges:");
    for (int[] edge : edges) {
        int u = edge[0], v = edge[1], w = edge[2];

        if (uf.find(u) != uf.find(v)) { // different components — safe edge
            uf.union(u, v);              // GREEDY CHOICE: add this edge
            totalWeight += w;
            edgesAdded++;
            System.out.printf("  %d — %d (weight %d)%n", u, v, w);
            if (edgesAdded == V - 1) break; // MST complete
        }
    }
    return totalWeight;
}
```

---

## 3.10 Coin Change — Greedy vs. Dynamic Programming

The coin change problem beautifully illustrates **when greedy works and when it doesn't**.

**Problem:** Make change for amount n using the fewest coins from denominations {d₁, d₂, ..., dₖ}.

**Greedy strategy:** Always use the largest denomination that fits.

```java
/**
 * Coin Change — Greedy (works for canonical coin systems only).
 *
 * Greedy WORKS for US coins: {1, 5, 10, 25}
 * Greedy FAILS for arbitrary denominations.
 */
public static Map<Integer, Integer> coinChangeGreedy(int[] coins, int amount) {
    Arrays.sort(coins);
    Map<Integer, Integer> used = new LinkedHashMap<>();
    int remaining = amount;

    // Try each denomination from largest to smallest
    for (int i = coins.length - 1; i >= 0 && remaining > 0; i--) {
        int count = remaining / coins[i]; // GREEDY: use as many as possible
        if (count > 0) {
            used.put(coins[i], count);
            remaining -= count * coins[i];
        }
    }

    if (remaining != 0) System.out.println("Cannot make exact change!");
    return used;
}

public static void main(String[] args) {
    // ✓ Greedy works: US coins
    int[] usCoin = {1, 5, 10, 25};
    System.out.println("US coins for 41¢: " + coinChangeGreedy(usCoin, 41));
    // {25=1, 10=1, 5=1, 1=1} → 4 coins (optimal: 25+10+5+1)

    // ✗ Greedy FAILS: coins = {1, 3, 4}, amount = 6
    int[] badCoins = {1, 3, 4};
    System.out.println("Greedy for 6 with {1,3,4}: " +
                       coinChangeGreedy(badCoins, 6));
    // Greedy: {4=1, 1=2} → 3 coins
    // Optimal: {3=2} → 2 coins  ← greedy gives wrong answer!
}
```

**Why greedy fails for {1, 3, 4}, amount = 6:**
- Greedy picks 4, then 1+1 → 3 coins
- Optimal is 3+3 → 2 coins
- The greedy choice property **does not hold**: taking the largest coin (4) prevents reaching the true optimum

This is exactly the problem that Dynamic Programming (Module 4) is designed to solve.

---

## 3.11 Task Scheduling on Parallel Machines

**Problem:** Schedule n tasks with weights (priorities) w[i] and processing times t[i] on a single machine to minimize the **weighted completion time** Σ w[i] · C[i].

**Greedy strategy:** Schedule in decreasing order of the ratio w[i]/t[i] (weight-to-processing-time ratio).

```java
/**
 * Weighted Job Scheduling — minimize Σ w[i] * C[i].
 * Greedy: sort by w[i]/t[i] descending (ratio rule).
 *
 * Time:  O(n log n)
 * Space: O(n)
 *
 * CORRECTNESS (Exchange Argument):
 * Suppose jobs i and j are adjacent with w[i]/t[i] < w[j]/t[j].
 * If i comes before j: contribution = w[i](S + t[i]) + w[j](S + t[i] + t[j])
 *                    = wS + w[i]t[i] + w[j]t[i] + w[j]t[j]
 * If j comes before i: contribution = w[j](S+t[j]) + w[i](S+t[j]+t[i])
 *                    = wS + w[j]t[j] + w[i]t[j] + w[i]t[i]
 * j before i is better iff w[i]t[j] < w[j]t[i] iff w[i]/t[i] < w[j]/t[j] ✓
 */
public static long weightedCompletion(int[] weights, int[] times) {
    int n = weights.length;

    Integer[] jobs = new Integer[n];
    for (int i = 0; i < n; i++) jobs[i] = i;

    // Sort by weight/time ratio descending
    Arrays.sort(jobs, (a, b) ->
        Double.compare((double) weights[b] / times[b],
                       (double) weights[a] / times[a]));

    long totalWeightedCompletion = 0;
    int  currentTime             = 0;

    System.out.println("Optimal weighted schedule:");
    for (int j : jobs) {
        currentTime             += times[j];
        long contribution        = (long) weights[j] * currentTime;
        totalWeightedCompletion += contribution;
        System.out.printf("  Job %d (w=%d, t=%d): completion=%d, contribution=%d%n",
                          j+1, weights[j], times[j], currentTime, contribution);
    }
    return totalWeightedCompletion;
}

// Example:
// weights=[3,2,1], times=[1,2,3]
// Ratios: 3,1,0.33 → order: job1, job2, job3
// C1=1, C2=3, C3=6
// Total = 3×1 + 2×3 + 1×6 = 3 + 6 + 6 = 15 (optimal)
```

---

## 3.12 Greedy Algorithm Design Checklist

Before committing to a greedy approach, verify:

```
✓ STEP 1: Identify the greedy choice
  What is the "obvious" locally optimal choice?
  Can you articulate it precisely as a selection criterion?

✓ STEP 2: Verify Greedy Choice Property
  Does making the greedy choice at each step lead to a globally optimal solution?
  PROVE IT — usually via exchange argument or greedy stays ahead.

✓ STEP 3: Verify Optimal Substructure
  After making the greedy choice, is the remaining subproblem of the same form?
  Is the optimal solution to the original = greedy choice + optimal solution to subproblem?

✓ STEP 4: Check for known failure modes
  Are there negative-weight edges? (breaks Dijkstra)
  Are items indivisible? (breaks fractional knapsack)
  Are coin denominations non-canonical? (breaks coin change)
  If YES to any → consider Dynamic Programming instead.

✓ STEP 5: Find a counter-example
  Try to construct an input where greedy fails.
  If you can → greedy is wrong. If you can't → more evidence it's right.
```

---

## 3.13 Greedy vs. Dynamic Programming

| Aspect              | Greedy                           | Dynamic Programming                |
|---------------------|----------------------------------|------------------------------------|
| Subproblem structure| Independent                      | Overlapping                        |
| Choice made         | Once, locally                    | After solving all subproblems      |
| Revisions           | Never — irrevocable              | Table filled bottom-up             |
| Correctness proof   | Exchange argument / stays ahead  | Optimal substructure + recurrence  |
| Time complexity     | Usually O(n log n) or O(n)       | Usually O(n²) or O(nW)             |
| Space complexity    | Usually O(1) or O(n)             | Usually O(n) or O(nW)              |
| When to use         | Greedy choice property holds     | No greedy choice property          |
| Example             | Activity selection, Huffman      | 0/1 Knapsack, Coin Change          |

---

## 3.14 Complete Example — Interval Scheduling with Weights

```java
/**
 * Weighted Interval Scheduling (using greedy for the unweighted version,
 * then showing why the weighted version requires DP).
 *
 * This demo contrasts greedy (for unweighted) with the need for DP (weighted).
 */
public class IntervalSchedulingDemo {

    // ── Unweighted: Greedy works perfectly ───────────────────────────────
    public static int maxActivities(int[][] intervals) {
        Arrays.sort(intervals, Comparator.comparingInt(a -> a[1])); // by finish time
        int count = 0, lastFinish = Integer.MIN_VALUE;
        for (int[] interval : intervals) {
            if (interval[0] >= lastFinish) {
                count++;
                lastFinish = interval[1];
            }
        }
        return count;
    }

    // ── Weighted: Greedy fails; need DP (preview of Module 4) ────────────
    // This is included to illustrate the transition point.
    public static int maxWeightedActivities(int[][] intervals, int[] weights) {
        int n = intervals.length;
        // Sort by finish time
        Integer[] idx = new Integer[n];
        for (int i = 0; i < n; i++) idx[i] = i;
        Arrays.sort(idx, Comparator.comparingInt(i -> intervals[i][1]));

        int[] dp = new int[n + 1]; // dp[i] = max weight using first i intervals

        for (int i = 1; i <= n; i++) {
            int curr = idx[i - 1];
            // Option 1: exclude current interval
            dp[i] = dp[i - 1];
            // Option 2: include current interval — find latest non-conflicting
            int p = findLastNonConflicting(intervals, idx, i - 1);
            dp[i] = Math.max(dp[i], dp[p + 1] + weights[curr]);
        }
        return dp[n];
    }

    private static int findLastNonConflicting(int[][] intervals,
                                               Integer[] idx, int i) {
        int start = intervals[idx[i]][0];
        for (int j = i - 1; j >= 0; j--) {
            if (intervals[idx[j]][1] <= start) return j;
        }
        return -1;
    }

    public static void main(String[] args) {
        int[][] intervals = {{1,4},{3,5},{0,6},{5,7},{3,9},{5,9},{6,10},{8,11}};
        int[]   weights   = {2,    4,    6,    3,    1,    8,    5,     7};

        System.out.println("Max unweighted activities: " +
                            maxActivities(intervals));
        // Greedy gives optimal: 4

        System.out.println("Max weight (DP):           " +
                            maxWeightedActivities(intervals, weights));
        // Greedy would fail here; DP gives correct answer
    }
}
```

---

## 3.15 Key Takeaways

| Concept                    | Summary                                                                        |
|----------------------------|--------------------------------------------------------------------------------|
| Greedy paradigm            | Make the locally optimal choice at each step; never revise                    |
| Greedy choice property     | Local optimum leads to global optimum — must be proven                         |
| Optimal substructure       | Optimal solution contains optimal solutions to subproblems                    |
| Exchange argument          | Any optimal solution can be transformed into greedy without worsening it      |
| Greedy stays ahead         | After each step, greedy is at least as good as any other partial solution     |
| Activity selection         | Greedy by earliest finish time → maximum non-overlapping activities           |
| Fractional knapsack        | Greedy by value/weight density → optimal (fails for 0/1 knapsack)            |
| Huffman coding             | Greedy merge of lowest frequencies → optimal prefix-free code                 |
| Job scheduling (EDF)       | Earliest deadline first → minimizes maximum lateness                          |
| Dijkstra                   | Greedy choice: finalize nearest unvisited vertex; fails with negative weights |
| MST (Prim's/Kruskal's)     | Both proven via Cut Property — minimum crossing edge is always safe           |
| Coin change                | Greedy works for canonical systems; fails for arbitrary denominations         |
| Greedy vs. DP              | Greedy: independent subproblems, O(n log n); DP: overlapping, O(n²)          |

---

## 3.16 Practice Problems

1. **Interval partitioning** — You have n lectures, each with a start and end time. Schedule all lectures using the fewest classrooms (no two lectures in the same room can overlap). Prove that the greedy algorithm (assign to any available room; open a new room if all are busy) uses the minimum number of rooms. *(Hint: the number of rooms needed equals the maximum number of overlapping lectures at any moment — "depth" of the schedule.)*

2. **Greedy proof** — Prove or disprove: for the activity selection problem, sorting by **shortest duration** (instead of earliest finish) always gives an optimal solution. Provide either a proof or a counter-example.

3. **Huffman analysis** — Given character frequencies {a:1, b:1, c:2, d:3, e:5, f:8, g:13, h:21}, build the Huffman tree and compute the expected bits per character. What pattern do you notice about the codes and frequencies?

4. **Gas station problem** — There are n gas stations arranged in a circle, each with gas[i] gallons. To travel from station i to i+1 costs cost[i] gallons. Starting with an empty tank, find the starting station (if any) from which you can complete the circuit. Prove your greedy solution is correct. *(Hint: if total gas ≥ total cost, a solution always exists.)*

5. **Scheduling to minimize weighted lateness** — Given jobs with processing times, deadlines, and weights, minimize the **total weighted lateness** Σ w[i] · max(0, C[i] − d[i]). Does a simple greedy rule work here? If yes, derive and prove it. If no, explain why this requires a more powerful technique.

---

## 3.17 Further Reading

- *Introduction to Algorithms* (CLRS) — Chapter 16 (Greedy Algorithms)
- *Algorithm Design* — Kleinberg & Tardos, Chapter 4 (Greedy Algorithms)
- *The Algorithm Design Manual* — Skiena, Chapter 8
- Huffman's original paper: "A Method for the Construction of Minimum Redundancy Codes" (1952)
- Proof of Huffman optimality: CLRS Theorem 16.4

---

*CS 3304 Analysis of Algorithms — Module 3 of 8*
*Previous: Module 2 — Divide and Conquer*
*Next: Module 4 — Dynamic Programming Techniques*
