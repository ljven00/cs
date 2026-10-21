# CS 3304 — Analysis of Algorithms
## Module 5: Backtracking Algorithms

---

## 5.1 Introduction

Dynamic Programming solves problems by breaking them into overlapping subproblems and storing results. Greedy algorithms commit to the best local choice. But some problems resist both approaches — there is no obvious greedy rule, and the subproblems are not neatly separable. For these problems, we must explore a space of possibilities systematically.

**Backtracking** is a general algorithm for finding solutions by incrementally building candidates and abandoning (backtracking from) a candidate as soon as it is determined that it cannot lead to a valid solution.

> Backtracking = Depth-First Search on a **decision tree**, with **pruning**.

The key improvement over brute force is **pruning**: when a partial candidate violates the problem's constraints, we cut off that entire branch without exploring its children — potentially eliminating an exponential number of candidates.

Backtracking is the backbone of constraint satisfaction: solving Sudoku, placing queens on a chessboard, finding all subsets with a given sum, generating permutations, solving mazes, and building solvers for logic puzzles.

By the end of this module you will:
- Understand the backtracking template and how it maps to a recursion tree
- Implement pruning strategies that make backtracking practical
- Analyze the time complexity of backtracking algorithms
- Apply backtracking to eight canonical problems
- Know when to choose backtracking over DP or greedy

---

## 5.2 The Backtracking Template

Every backtracking algorithm has the same recursive skeleton:

```
backtrack(state, choices):
    if state is a COMPLETE SOLUTION:
        record/output state
        return

    for each choice in choices:
        if choice is VALID (doesn't violate constraints):
            MAKE the choice  (modify state)
            backtrack(updated_state, remaining_choices)  // recurse
            UNDO the choice  (restore state)  ← this is the "backtrack"
```

The three key operations are:
1. **Choose** — add an element to the partial solution
2. **Explore** — recurse with the updated state
3. **Unchoose** — remove the element, restoring state for the next choice

This choose/explore/unchoose pattern is the hallmark of backtracking.

### The Recursion Tree

A backtracking algorithm implicitly traverses a **state space tree**:

```
                      []                ← root: empty solution
                /    |    \
             [1]    [2]    [3]          ← choose first element
            /  \   /  \   /  \
         [1,2][1,3][2,1][2,3][3,1][3,2] ← choose second element
          ...                           ← prune invalid branches early!
```

The power of backtracking lies in cutting off branches **before** exploring them fully.

---

## 5.3 Generating All Subsets (Power Set)

**Problem:** Generate all 2^n subsets of a set {1, 2, ..., n}.

This is the simplest backtracking problem — there are no constraints to check, so no pruning occurs. It demonstrates the pure choose/explore/unchoose structure.

```
n = 3, elements = [1, 2, 3]

Decision tree (include or exclude each element):
                       []
                   /        \
               [1]            []
            /      \        /    \
         [1,2]    [1]    [2]     []
         / \      / \    / \     / \
      [1,2,3][1,2][1,3][1][2,3][2][3][]
```

```java
/**
 * Generate All Subsets (Power Set).
 *
 * At each position, we make a binary choice: include or exclude.
 *
 * Time:  O(2^n * n) — 2^n subsets, each takes O(n) to copy
 * Space: O(n) recursion depth
 */
public static List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    backtrackSubsets(nums, 0, new ArrayList<>(), result);
    return result;
}

private static void backtrackSubsets(int[] nums, int start,
                                      List<Integer> current,
                                      List<List<Integer>> result) {
    // Every partial state is a valid subset — record it
    result.add(new ArrayList<>(current));       // ADD A COPY — not a reference

    for (int i = start; i < nums.length; i++) {
        current.add(nums[i]);                   // CHOOSE: include nums[i]
        backtrackSubsets(nums, i + 1, current, result); // EXPLORE
        current.remove(current.size() - 1);     // UNCHOOSE: restore state
    }
}

public static void main(String[] args) {
    int[] nums = {1, 2, 3};
    List<List<Integer>> result = subsets(nums);
    System.out.println("All subsets (" + result.size() + "):");
    result.forEach(System.out::println);
    // [], [1], [1,2], [1,2,3], [1,3], [2], [2,3], [3]
}
```

---

## 5.4 Generating All Permutations

**Problem:** Generate all n! permutations of n distinct elements.

```
[1,2,3] → [1,2,3], [1,3,2], [2,1,3], [2,3,1], [3,1,2], [3,2,1]
```

```java
/**
 * Generate All Permutations.
 *
 * At each position, choose any unused element.
 * Use a "used" boolean array to track which elements are taken.
 *
 * Time:  O(n! * n) — n! permutations, each O(n) to copy
 * Space: O(n) for used array + O(n) recursion depth
 */
public static List<List<Integer>> permutations(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    backtrackPerms(nums, new boolean[nums.length], new ArrayList<>(), result);
    return result;
}

private static void backtrackPerms(int[] nums, boolean[] used,
                                    List<Integer> current,
                                    List<List<Integer>> result) {
    // BASE CASE: permutation is complete
    if (current.size() == nums.length) {
        result.add(new ArrayList<>(current));
        return;
    }

    for (int i = 0; i < nums.length; i++) {
        if (used[i]) continue;                  // PRUNE: already in current path

        used[i] = true;                         // CHOOSE
        current.add(nums[i]);
        backtrackPerms(nums, used, current, result); // EXPLORE
        current.remove(current.size() - 1);     // UNCHOOSE
        used[i] = false;
    }
}

/**
 * Permutations with Duplicates — skip duplicate choices at the same depth.
 *
 * Sort first. If nums[i] == nums[i-1] and nums[i-1] was NOT used in the
 * current path (was skipped), skip nums[i] too — it would produce
 * the same permutation branch.
 */
public static List<List<Integer>> permutationsUnique(int[] nums) {
    Arrays.sort(nums);
    List<List<Integer>> result = new ArrayList<>();
    backtrackPermsUnique(nums, new boolean[nums.length], new ArrayList<>(), result);
    return result;
}

private static void backtrackPermsUnique(int[] nums, boolean[] used,
                                          List<Integer> current,
                                          List<List<Integer>> result) {
    if (current.size() == nums.length) { result.add(new ArrayList<>(current)); return; }

    for (int i = 0; i < nums.length; i++) {
        if (used[i]) continue;
        // PRUNE DUPLICATES: skip if same value as previous and previous was skipped
        if (i > 0 && nums[i] == nums[i-1] && !used[i-1]) continue;

        used[i] = true;
        current.add(nums[i]);
        backtrackPermsUnique(nums, used, current, result);
        current.remove(current.size() - 1);
        used[i] = false;
    }
}
```

---

## 5.5 Combination Sum

**Problem:** Given an array of distinct integers (candidates) and a target, find all unique combinations that sum to the target. Each number may be used multiple times.

```
candidates = [2, 3, 6, 7],  target = 7

Solutions: [2,2,3], [7]
```

```java
/**
 * Combination Sum — numbers may be reused.
 *
 * PRUNING: Sort candidates first. If current candidate > remaining,
 * all subsequent candidates are also too large — break early.
 *
 * Time:  O(n^(T/min)) in the worst case — T=target, min=smallest candidate
 * Space: O(T/min) recursion depth
 */
public static List<List<Integer>> combinationSum(int[] candidates, int target) {
    Arrays.sort(candidates);         // sort for early termination
    List<List<Integer>> result = new ArrayList<>();
    backtrackCombSum(candidates, target, 0, new ArrayList<>(), result);
    return result;
}

private static void backtrackCombSum(int[] cands, int remaining, int start,
                                      List<Integer> current,
                                      List<List<Integer>> result) {
    if (remaining == 0) {
        result.add(new ArrayList<>(current));  // SOLUTION FOUND
        return;
    }

    for (int i = start; i < cands.length; i++) {
        if (cands[i] > remaining) break;       // PRUNE: sorted → rest too large

        current.add(cands[i]);                 // CHOOSE
        // i (not i+1): same element can be reused
        backtrackCombSum(cands, remaining - cands[i], i, current, result);
        current.remove(current.size() - 1);    // UNCHOOSE
    }
}

/**
 * Combination Sum II — each number used at most once, may have duplicates.
 *
 * PRUNING: Skip duplicate values at the same recursion depth.
 */
public static List<List<Integer>> combinationSum2(int[] candidates, int target) {
    Arrays.sort(candidates);
    List<List<Integer>> result = new ArrayList<>();
    backtrackCombSum2(candidates, target, 0, new ArrayList<>(), result);
    return result;
}

private static void backtrackCombSum2(int[] cands, int remaining, int start,
                                       List<Integer> current,
                                       List<List<Integer>> result) {
    if (remaining == 0) { result.add(new ArrayList<>(current)); return; }

    for (int i = start; i < cands.length; i++) {
        if (cands[i] > remaining) break;
        // PRUNE DUPLICATES: skip same value at same depth level
        if (i > start && cands[i] == cands[i-1]) continue;

        current.add(cands[i]);
        backtrackCombSum2(cands, remaining - cands[i], i + 1, current, result);
        current.remove(current.size() - 1);
    }
}
```

---

## 5.6 N-Queens Problem

**Problem:** Place n queens on an n×n chessboard so that no two queens attack each other (no two queens in the same row, column, or diagonal).

This is one of the most celebrated backtracking problems. The brute force checks all n^n placements; backtracking prunes to O(n!) and then further with constraint checks.

```
4-Queens solution:
. Q . .      . . Q .
. . . Q      Q . . .
Q . . .      . . . Q
. . Q .      . Q . .

Two solutions for n=4
```

```java
/**
 * N-Queens — place n queens with no conflicts.
 *
 * REPRESENTATION: queens[row] = column where queen in that row is placed.
 * One queen per row is guaranteed by the recursion structure.
 *
 * PRUNING:
 *   - Column conflict:  another queen in same column
 *   - Diagonal conflict: |row1 - row2| == |col1 - col2|
 *
 * Time:  O(n!) in the worst case (much better with pruning in practice)
 * Space: O(n) for queen positions + O(n) recursion depth
 */
public static List<List<String>> solveNQueens(int n) {
    List<List<String>> solutions = new ArrayList<>();
    int[] queens = new int[n]; // queens[i] = column of queen in row i
    Arrays.fill(queens, -1);
    backtrackQueens(n, 0, queens, new HashSet<>(), new HashSet<>(),
                   new HashSet<>(), solutions);
    return solutions;
}

/**
 * cols:    set of occupied columns
 * diag1:   set of occupied (row - col) diagonals  (top-left to bottom-right)
 * diag2:   set of occupied (row + col) diagonals  (top-right to bottom-left)
 *
 * Using hash sets makes conflict checking O(1) instead of O(n).
 */
private static void backtrackQueens(int n, int row, int[] queens,
                                     Set<Integer> cols,
                                     Set<Integer> diag1, Set<Integer> diag2,
                                     List<List<String>> solutions) {
    if (row == n) {
        solutions.add(buildBoard(queens, n)); // complete solution
        return;
    }

    for (int col = 0; col < n; col++) {
        int d1 = row - col;
        int d2 = row + col;

        // PRUNE: skip if column or either diagonal is occupied
        if (cols.contains(col) || diag1.contains(d1) || diag2.contains(d2))
            continue;

        // CHOOSE: place queen
        queens[row] = col;
        cols.add(col); diag1.add(d1); diag2.add(d2);

        backtrackQueens(n, row + 1, queens, cols, diag1, diag2, solutions); // EXPLORE

        // UNCHOOSE: remove queen
        queens[row] = -1;
        cols.remove(col); diag1.remove(d1); diag2.remove(d2);
    }
}

private static List<String> buildBoard(int[] queens, int n) {
    List<String> board = new ArrayList<>();
    for (int row = 0; row < n; row++) {
        char[] line = new char[n];
        Arrays.fill(line, '.');
        line[queens[row]] = 'Q';
        board.add(new String(line));
    }
    return board;
}

public static void main(String[] args) {
    List<List<String>> solutions = solveNQueens(8);
    System.out.println("8-Queens: " + solutions.size() + " solutions"); // 92

    System.out.println("\n4-Queens first solution:");
    solveNQueens(4).get(0).forEach(System.out::println);
    // .Q..
    // ...Q
    // Q...
    // ..Q.
}
```

---

## 5.7 Sudoku Solver

**Problem:** Fill a 9×9 Sudoku grid so that every row, column, and 3×3 box contains digits 1–9 exactly once.

```java
/**
 * Sudoku Solver — Backtracking with constraint propagation.
 *
 * REPRESENTATION:
 *   board[r][c] = digit (1-9) or '.' for empty
 *
 * PRUNING:
 *   Before placing a digit, check:
 *   - Not already in the same row
 *   - Not already in the same column
 *   - Not already in the same 3×3 box
 *
 * Time:  O(9^m) where m = number of empty cells (at most 81)
 *        With effective pruning, typical Sudoku solves in microseconds.
 * Space: O(m) recursion depth
 */
public static boolean solveSudoku(char[][] board) {
    for (int r = 0; r < 9; r++) {
        for (int c = 0; c < 9; c++) {
            if (board[r][c] == '.') {             // find empty cell
                for (char d = '1'; d <= '9'; d++) {
                    if (isValidPlacement(board, r, c, d)) {
                        board[r][c] = d;          // CHOOSE: place digit

                        if (solveSudoku(board))   // EXPLORE
                            return true;          // solution found — propagate

                        board[r][c] = '.';        // UNCHOOSE: backtrack
                    }
                }
                return false;                     // no digit works → backtrack
            }
        }
    }
    return true; // all cells filled → solved
}

private static boolean isValidPlacement(char[][] board, int row, int col, char d) {
    int boxRow = (row / 3) * 3;
    int boxCol = (col / 3) * 3;

    for (int i = 0; i < 9; i++) {
        if (board[row][i]        == d) return false; // row check
        if (board[i][col]        == d) return false; // column check
        if (board[boxRow + i/3][boxCol + i%3] == d)
                                       return false; // box check
    }
    return true;
}

public static void printBoard(char[][] board) {
    for (int r = 0; r < 9; r++) {
        if (r % 3 == 0 && r != 0) System.out.println("------+-------+------");
        for (int c = 0; c < 9; c++) {
            if (c % 3 == 0 && c != 0) System.out.print(" | ");
            System.out.print(board[r][c] + " ");
        }
        System.out.println();
    }
}
```

---

## 5.8 Word Search

**Problem:** Given a 2D grid of characters and a target word, determine if the word exists as a path in the grid (horizontally or vertically adjacent, no cell reused).

```
Board:              Word: "ABCCED"
A B C E
S F C S             Path: A(0,0)→B(0,1)→C(0,2)→C(1,2)→E(1,3)→D?
A D E E             → A(0,0)→B(0,1)→C(0,2)→C(1,2)→E(2,2)→D(2,1) ✓
```

```java
/**
 * Word Search — DFS Backtracking on a 2D Grid.
 *
 * PRUNING:
 *   1. Character mismatch → prune immediately
 *   2. Already visited cell → prune (track with temporary marker)
 *
 * Time:  O(rows * cols * 4^len) — each cell can start a DFS of depth len
 * Space: O(len) recursion depth
 */
public static boolean wordSearch(char[][] board, String word) {
    int rows = board.length, cols = board[0].length;

    for (int r = 0; r < rows; r++)
        for (int c = 0; c < cols; c++)
            if (dfsWord(board, word, r, c, 0)) return true;

    return false;
}

private static boolean dfsWord(char[][] board, String word, int r, int c, int idx) {
    if (idx == word.length()) return true;          // BASE CASE: all chars matched

    // PRUNE: out of bounds, wrong character, or already visited
    if (r < 0 || r >= board.length || c < 0 || c >= board[0].length) return false;
    if (board[r][c] != word.charAt(idx)) return false;
    if (board[r][c] == '#') return false;           // visited marker

    char temp  = board[r][c];
    board[r][c] = '#';                              // CHOOSE: mark as visited

    boolean found = dfsWord(board, word, r+1, c, idx+1) // EXPLORE all 4 directions
                 || dfsWord(board, word, r-1, c, idx+1)
                 || dfsWord(board, word, r, c+1, idx+1)
                 || dfsWord(board, word, r, c-1, idx+1);

    board[r][c] = temp;                            // UNCHOOSE: unmark

    return found;
}
```

---

## 5.9 Graph Coloring

**Problem:** Given a graph and k colors, assign a color to each vertex such that no two adjacent vertices share the same color. Find all valid k-colorings or determine if one exists.

This is an NP-complete problem for general graphs, but backtracking finds solutions efficiently for small inputs.

```java
/**
 * Graph k-Coloring — Backtracking.
 *
 * PRUNING: Before assigning color c to vertex v, check that no
 *          neighbor of v already has color c.
 *
 * Time:  O(k^V) worst case — k choices for each of V vertices
 *        With pruning, far fewer branches are explored in practice.
 * Space: O(V) for color assignment + O(V) recursion depth
 */
public static boolean graphColoring(int[][] graph, int k, int[] colors, int vertex) {
    int V = graph.length;

    // BASE CASE: all vertices colored
    if (vertex == V) return true;

    for (int color = 1; color <= k; color++) {
        if (isSafeColor(graph, colors, vertex, color)) {
            colors[vertex] = color;                     // CHOOSE

            if (graphColoring(graph, k, colors, vertex + 1)) // EXPLORE
                return true;                            // found valid coloring

            colors[vertex] = 0;                         // UNCHOOSE
        }
    }
    return false; // no valid color for this vertex → backtrack
}

private static boolean isSafeColor(int[][] graph, int[] colors, int v, int color) {
    for (int u = 0; u < graph.length; u++) {
        if (graph[v][u] == 1 && colors[u] == color)
            return false; // neighbor has same color
    }
    return true;
}

public static void main(String[] args) {
    // Graph: 0-1-2-3-0 (cycle of 4)
    int[][] graph = {
        {0,1,0,1},
        {1,0,1,0},
        {0,1,0,1},
        {1,0,1,0}
    };
    int[] colors = new int[4]; // 0 = uncolored

    if (graphColoring(graph, 3, colors, 0)) {
        System.out.println("3-coloring: " + Arrays.toString(colors));
        // e.g., [1, 2, 1, 2]
    } else {
        System.out.println("No valid 3-coloring exists");
    }
}
```

---

## 5.10 Rat in a Maze (Path Finding)

**Problem:** A rat starts at position (0,0) in an n×n maze. Find all paths to (n-1, n-1). The rat can move Up, Down, Left, Right. Cells with value 1 are open; 0 are blocked.

```java
/**
 * Rat in a Maze — Backtracking path finder.
 *
 * PRUNING:
 *   1. Out-of-bounds → prune
 *   2. Blocked cell (maze[r][c] == 0) → prune
 *   3. Already visited → prune
 *
 * Time:  O(4^(n²)) worst case — 4 directions at each of n² cells
 * Space: O(n²) for visited array + O(n²) recursion depth
 */
public static List<String> findPaths(int[][] maze) {
    int n = maze.length;
    boolean[][] visited = new boolean[n][n];
    List<String> paths  = new ArrayList<>();

    if (maze[0][0] == 1)
        backtrackMaze(maze, 0, 0, n, visited, "", paths);

    return paths;
}

private static void backtrackMaze(int[][] maze, int r, int c, int n,
                                   boolean[][] visited,
                                   String path, List<String> paths) {
    // BASE CASE: reached destination
    if (r == n-1 && c == n-1) {
        paths.add(path);
        return;
    }

    // Directions: Down, Left, Right, Up (alphabetical for consistent output)
    int[][] dirs    = {{1,0},{0,-1},{0,1},{-1,0}};
    char[]  dirChar = {'D','L','R','U'};

    visited[r][c] = true; // CHOOSE: mark current cell

    for (int d = 0; d < 4; d++) {
        int nr = r + dirs[d][0];
        int nc = c + dirs[d][1];

        // PRUNE: bounds, blocked, visited
        if (nr >= 0 && nr < n && nc >= 0 && nc < n
                && maze[nr][nc] == 1 && !visited[nr][nc]) {
            backtrackMaze(maze, nr, nc, n, visited,
                          path + dirChar[d], paths); // EXPLORE
        }
    }

    visited[r][c] = false; // UNCHOOSE: unmark
}

public static void main(String[] args) {
    int[][] maze = {
        {1, 0, 0, 0},
        {1, 1, 0, 1},
        {1, 1, 0, 0},
        {0, 1, 1, 1}
    };
    List<String> paths = findPaths(maze);
    System.out.println("Paths found: " + paths.size());
    paths.forEach(System.out::println);
    // DDRDRR
    // DDRRD (etc.)
}
```

---

## 5.11 Hamiltonian Path and Cycle

**Problem:** A Hamiltonian path visits every vertex exactly once. A Hamiltonian cycle additionally returns to the starting vertex. Determining existence is NP-complete in general.

```java
/**
 * Hamiltonian Cycle — Backtracking.
 *
 * PRUNING:
 *   1. No edge from current vertex to next → prune
 *   2. Next vertex already visited → prune
 *   3. Final vertex: must have edge back to start → check before recording
 *
 * Time:  O(V!) worst case — permutations of vertices
 * Space: O(V) path array + O(V) recursion depth
 */
public static boolean hamiltonianCycle(int[][] graph, int[] path, int pos) {
    int V = graph.length;

    // BASE CASE: all vertices included
    if (pos == V) {
        // Check if there is an edge from last vertex back to start
        return graph[path[pos-1]][path[0]] == 1;
    }

    for (int v = 1; v < V; v++) { // vertex 0 is always the starting point
        if (isSafeHamiltonian(graph, path, pos, v)) {
            path[pos] = v;                              // CHOOSE

            if (hamiltonianCycle(graph, path, pos + 1)) // EXPLORE
                return true;

            path[pos] = -1;                             // UNCHOOSE
        }
    }
    return false;
}

private static boolean isSafeHamiltonian(int[][] graph, int[] path, int pos, int v) {
    // No edge from previous vertex to v
    if (graph[path[pos-1]][v] == 0) return false;
    // Already in path
    for (int i = 0; i < pos; i++) if (path[i] == v) return false;
    return true;
}

public static void main(String[] args) {
    int[][] graph = {
        {0,1,0,1,0},
        {1,0,1,1,1},
        {0,1,0,0,1},
        {1,1,0,0,1},
        {0,1,1,1,0}
    };
    int V     = graph.length;
    int[] path = new int[V];
    Arrays.fill(path, -1);
    path[0] = 0;    // always start from vertex 0

    if (hamiltonianCycle(graph, path, 1)) {
        System.out.println("Hamiltonian Cycle: " + Arrays.toString(path));
        // e.g., [0, 1, 2, 4, 3]
    } else {
        System.out.println("No Hamiltonian Cycle exists.");
    }
}
```

---

## 5.12 Pruning Strategies

The performance of a backtracking algorithm depends almost entirely on the quality of its pruning. Here are the standard strategies:

### Strategy 1: Constraint Checking (Feasibility Pruning)

Check all constraints as early as possible — before recursing, not after.

```java
// POOR: check constraint AFTER recursing
for (int choice : choices) {
    make(choice);
    recurse();     // recurse first, then check — wastes work
    if (!valid()) undo(choice);
}

// GOOD: check constraint BEFORE recursing (prune early)
for (int choice : choices) {
    if (isValid(choice)) {   // check FIRST
        make(choice);
        recurse();
        undo(choice);
    }
}
```

### Strategy 2: Forward Checking

After making a choice, check which future choices are still available. If any future variable has no valid choice remaining, prune immediately.

```java
/**
 * Forward checking for constraint satisfaction.
 * After assigning variable x, update domains of unassigned variables.
 * If any domain becomes empty → backtrack immediately.
 */
private static boolean forwardCheck(int[] assignment, int variable,
                                     int value, int[][] domains) {
    // Remove 'value' from domains of all unassigned neighbors
    for (int neighbor : getNeighbors(variable)) {
        if (assignment[neighbor] == -1) { // unassigned
            // Remove conflicting value from domain
            domains[neighbor] = Arrays.stream(domains[neighbor])
                                      .filter(v -> v != value).toArray();
            if (domains[neighbor].length == 0) return false; // domain wipeout
        }
    }
    return true;
}
```

### Strategy 3: Ordering Heuristics

The order in which variables and values are tried dramatically affects performance.

**Fail-First (MRV — Minimum Remaining Values):** Choose the variable with the smallest remaining domain first. If a variable will fail, fail it early rather than deep in the search tree.

**Least Constraining Value:** Among values for a chosen variable, try the one that eliminates the fewest choices from other variables' domains.

```java
/**
 * Minimum Remaining Values (MRV) heuristic.
 * Always pick the unassigned variable with the fewest legal values.
 * Finds failures (dead ends) earlier → better pruning.
 */
private static int selectVariableMRV(int[][] domains, boolean[] assigned) {
    int minSize  = Integer.MAX_VALUE;
    int selected = -1;

    for (int i = 0; i < domains.length; i++) {
        if (!assigned[i] && domains[i].length < minSize) {
            minSize  = domains[i].length;
            selected = i;
        }
    }
    return selected;
}
```

### Strategy 4: Symmetry Breaking

Many problems have equivalent solutions related by symmetry. Pruning symmetric solutions avoids redundant work.

```java
// N-Queens symmetry: the first queen in row 0 can be limited to columns 0..n/2.
// Each solution in the left half has a mirror image in the right half.
for (int col = 0; col <= n/2; col++) {  // only half the columns
    // ... place queen in column col
    // solutions found here can be mirrored for the other half
}
```

---

## 5.13 Complexity Analysis of Backtracking

Backtracking explores a state space tree. Its complexity depends on:
- **b** — branching factor (choices at each step)
- **d** — depth of the solution (length of candidate)
- **Pruning effectiveness** — what fraction of branches are cut

```
Worst case (no pruning):   O(b^d)    — exponential
With perfect pruning:      O(d)      — linear (direct path to solution)
Typical practice:          between these extremes
```

### Complexity of Classic Problems

| Problem              | Worst Case         | With Pruning         |
|----------------------|--------------------|----------------------|
| Subsets              | O(2^n)             | O(2^n) — no pruning possible |
| Permutations         | O(n!)              | O(n!) — all permutations |
| N-Queens             | O(n!)              | O(n! / k) in practice |
| Sudoku               | O(9^81)            | O(9^m) where m = empty cells |
| Graph Coloring       | O(k^V)             | Heavily pruned in practice |
| Hamiltonian Cycle    | O(V!)              | O(V!) worst, better average |
| Combination Sum      | O(n^(T/min))       | O(n^(T/min)) with sorting |
| Word Search          | O(rows×cols×4^L)   | O(rows×cols×4^L) with mismatch prune |

### Key Insight: Pruning Changes Everything

```
N-Queens, n=8:
  No pruning:   8^8    = 16,777,216 possibilities to check
  With pruning:         ≈ 15,720 nodes explored
  Speedup:      1000×

Sudoku (typical puzzle, ~50 empty cells):
  No pruning:   9^50   ≈ 5 × 10^47 possibilities
  With pruning: typically < 1000 nodes explored
  Speedup:      astronomical
```

---

## 5.14 Backtracking vs. Other Paradigms

| Feature              | Backtracking              | Dynamic Programming         | Greedy                   |
|----------------------|---------------------------|-----------------------------|--------------------------|
| Explores             | All valid paths (with pruning) | All subproblems         | One path (local best)    |
| Revisions            | Yes — undoes bad choices  | No — builds table          | Never                    |
| Subproblem overlap   | No (or ignored)           | Yes — essential            | No                       |
| Optimality           | Finds optimal if exhaustive | Always optimal            | Optimal only if GCP holds|
| Completeness         | Finds all solutions       | Finds one optimal solution  | Finds one solution       |
| Time complexity      | Exponential (pruned)      | Polynomial                  | Usually O(n log n)       |
| Best for             | Constraint satisfaction   | Optimization with overlap   | Optimization with GCP    |
| Examples             | Sudoku, N-Queens, CSPs    | Knapsack, LCS, Edit Dist.   | Activity Sel., Huffman   |

---

## 5.15 Complete Example — Solving a Constraint Satisfaction Problem

```java
import java.util.*;

/**
 * General Constraint Satisfaction Problem (CSP) Solver.
 *
 * Demonstrates how backtracking, forward checking, and MRV
 * combine into a general-purpose CSP solver.
 *
 * Applied to Map Coloring: color regions of a map so no two
 * adjacent regions share the same color.
 */
public class CSPSolver {

    static final int UNASSIGNED = -1;

    /**
     * Solve a CSP with:
     *   variables:   n variables (e.g., regions)
     *   domains:     domains[i] = list of valid values for variable i
     *   constraints: adjacency matrix (constraints[i][j] = true → i,j must differ)
     */
    public static int[] solve(int n, int[][] domains, boolean[][] constraints) {
        int[] assignment = new int[n];
        Arrays.fill(assignment, UNASSIGNED);
        return backtrackCSP(assignment, domains, constraints, n) ? assignment : null;
    }

    private static boolean backtrackCSP(int[] assignment, int[][] domains,
                                         boolean[][] constraints, int n) {
        // BASE CASE: all variables assigned
        int var = selectUnassigned(assignment, domains, n); // MRV heuristic
        if (var == -1) return true;

        for (int value : domains[var]) {
            if (isConsistent(var, value, assignment, constraints)) {
                assignment[var] = value;                        // CHOOSE

                // Forward check: ensure remaining variables still have options
                int[][] prunedDomains = pruneDomains(var, value, domains,
                                                     assignment, constraints);

                if (prunedDomains != null) {                    // no domain wipeout
                    if (backtrackCSP(assignment, prunedDomains, constraints, n))
                        return true;
                }

                assignment[var] = UNASSIGNED;                   // UNCHOOSE
            }
        }
        return false;
    }

    // MRV: pick unassigned variable with fewest remaining values
    private static int selectUnassigned(int[] assignment, int[][] domains, int n) {
        int minSize = Integer.MAX_VALUE, selected = -1;
        for (int i = 0; i < n; i++) {
            if (assignment[i] == UNASSIGNED && domains[i].length < minSize) {
                minSize = domains[i].length;
                selected = i;
            }
        }
        return selected;
    }

    // Check current assignment is consistent
    private static boolean isConsistent(int var, int value, int[] assignment,
                                         boolean[][] constraints) {
        for (int i = 0; i < assignment.length; i++) {
            if (assignment[i] != UNASSIGNED && constraints[var][i]
                    && assignment[i] == value) return false;
        }
        return true;
    }

    // Forward checking: remove value from neighbors' domains
    private static int[][] pruneDomains(int var, int value, int[][] domains,
                                         int[] assignment, boolean[][] constraints) {
        int[][] newDomains = Arrays.copyOf(domains, domains.length);
        for (int i = 0; i < domains.length; i++) {
            if (assignment[i] == UNASSIGNED && constraints[var][i]) {
                final int val = value;
                newDomains[i] = Arrays.stream(domains[i])
                                      .filter(v -> v != val).toArray();
                if (newDomains[i].length == 0) return null; // domain wipeout
            }
        }
        return newDomains;
    }

    public static void main(String[] args) {
        // Australia map coloring:
        // WA, NT, SA, Q, NSW, V, T (7 regions, 3 colors)
        String[] regions = {"WA","NT","SA","Q","NSW","V","T"};
        int n = regions.length;

        // Adjacency (constraint: adjacent regions must differ)
        boolean[][] adj = new boolean[n][n];
        int[][] edges = {{0,1},{0,2},{1,2},{1,3},{2,3},{2,4},{2,5},{3,4},{4,5}};
        for (int[] e : edges) { adj[e[0]][e[1]] = adj[e[1]][e[0]] = true; }

        // 3 colors: 0=Red, 1=Green, 2=Blue
        int[][] domains = new int[n][];
        Arrays.fill(domains, new int[]{0, 1, 2});

        int[] solution = solve(n, domains, adj);

        String[] colorNames = {"Red","Green","Blue"};
        System.out.println("Australia Map Coloring:");
        for (int i = 0; i < n; i++) {
            System.out.printf("  %s: %s%n", regions[i], colorNames[solution[i]]);
        }
        // WA: Red, NT: Green, SA: Blue, Q: Red, NSW: Green, V: Red, T: Red
    }
}
```

---

## 5.16 Key Takeaways

| Concept                    | Summary                                                                       |
|----------------------------|-------------------------------------------------------------------------------|
| Backtracking               | DFS on a state space tree with pruning of invalid branches                   |
| Choose/Explore/Unchoose    | The three steps: commit, recurse, restore — always in this order             |
| Pruning                    | Cut branches before exploring them — transforms exponential into practical   |
| Constraint checking        | Check validity BEFORE recursing, not after                                   |
| Forward checking           | After a choice, verify remaining variables still have valid options          |
| MRV heuristic              | Choose the most constrained variable first — fail fast                       |
| Subsets                    | O(2^n); every partial state is valid — record at every node                 |
| Permutations               | O(n!); use used[] array; skip duplicates with sort + i > start check        |
| N-Queens                   | O(n!); O(1) conflict check with col/diag sets; 92 solutions for n=8         |
| Sudoku                     | O(9^m); in practice microseconds with effective pruning                      |
| Combination Sum            | Sort + break when candidate > remaining — powerful O(1) prune per branch    |
| Graph Coloring             | O(k^V) worst; NP-complete in general                                         |
| Hamiltonian Cycle          | O(V!); NP-complete; backtracking practical for small V                       |
| Backtracking vs. DP        | Backtracking: all solutions, exponential; DP: one optimal, polynomial        |

---

## 5.17 Practice Problems

1. **Letter Combinations of a Phone Number** — Given a string of digits 2–9, return all possible letter combinations (like a phone keypad). For "23", return all combinations of {abc} × {def}. Analyze the time complexity in terms of the number of digits and average letters per digit.

2. **Palindrome Partitioning** — Given a string s, partition it so that every substring of the partition is a palindrome. Return all possible palindrome partitionings. *(Hint: at each position, try every prefix of the remaining string. Prune if the prefix is not a palindrome. Precompute palindrome status with DP for efficiency.)*

3. **N-Queens Count** — Modify the N-Queens solver to only count the number of valid solutions (not store them). Optimize for n=12 (which has 14,200 solutions). Measure the speedup from using the column/diagonal set data structures vs. checking the board array.

4. **Cryptarithmetic** — Solve SEND + MORE = MONEY. Assign distinct digits 0–9 to letters {S,E,N,D,M,O,R,Y} such that the addition holds. *(Hint: assign digits to letters one at a time; prune when partial assignment already violates the column equations.)*

5. **Subset Sum with Memoization** — Given a set of integers and a target, find if any subset sums to the target. Implement both pure backtracking and backtracking with memoization. Compare their performance and explain when memoization converts backtracking to DP.

---

## 5.18 Further Reading

- *Introduction to Algorithms* (CLRS) — Appendix (Backtracking references in NP chapter)
- *Algorithm Design* — Kleinberg & Tardos, Chapter 5 (also covers backtracking via exhaustive search)
- *The Algorithm Design Manual* — Skiena, Chapter 9 (Combinatorial Search)
- *Artificial Intelligence: A Modern Approach* — Russell & Norvig, Chapter 6 (CSPs — constraint propagation, arc consistency, MRV)
- N-Queens statistics: [https://oeis.org/A000170](https://oeis.org/A000170)

---

*CS 3304 Analysis of Algorithms — Module 5 of 8*
*Previous: Module 4 — Dynamic Programming Techniques*
*Next: Module 6 — Optimization Techniques*
