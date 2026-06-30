> **Topic:** [Graphs](../README.md) · **Pattern 1 of 7**

# Graph Traversal (BFS / DFS)

Graph traversal is the foundation of almost every graph algorithm. Before you can detect cycles, find shortest paths, topologically sort, or compute connectivity, you must be able to systematically visit every reachable vertex exactly once. The two workhorses are **Breadth-First Search (BFS)** and **Depth-First Search (DFS)**.

---

## 1. Core Concept — BFS vs DFS

Both BFS and DFS explore a graph by starting at a source vertex and visiting reachable vertices, marking each as visited so it is processed only once. The difference is the **order** in which they expand the frontier.

### BFS (Breadth-First Search)
- Explores the graph **level by level** (all nodes at distance 1, then distance 2, ...).
- Uses a **queue (FIFO)**.
- A node is marked visited **the moment it is enqueued** (not when it is dequeued) — this prevents the same node from being added to the queue multiple times.
- Naturally computes the **shortest path in an unweighted graph** because the first time you reach a node is via the fewest edges.

### DFS (Depth-First Search)
- Explores **as deep as possible** along one branch before backtracking.
- Uses the **call stack** (recursion) or an explicit **stack**.
- Excellent for **connectivity questions**, **exhaustive exploration**, cycle detection, topological sort, and flood-fill style problems where you just need to touch every reachable cell.

### When to use which

| Goal | Prefer |
|------|--------|
| Shortest path / minimum steps in an **unweighted** graph | **BFS** |
| Level-order processing (e.g., "minimum number of transformations") | **BFS** |
| Spreading / multi-source expansion (rotting oranges, locks) | **BFS** |
| Just need to know what's connected / count components | **DFS or BFS** (either works) |
| Exhaustive exploration, path reconstruction, backtracking | **DFS** |
| Very deep graphs where recursion could stack-overflow | **BFS or iterative DFS** |

> **Key intuition:** If the question contains the words *"shortest"*, *"minimum number of steps/moves/transformations"*, or *"fewest"* on an **unweighted** graph, reach for **BFS**. If it's about *"can I reach"*, *"how many groups"*, or *"explore everything"*, **DFS** is usually simplest.

---

## 2. Graph Representation

### Building an adjacency list from an edge list

Most graph problems hand you `n` nodes and a list of edges. Convert that into an **adjacency list** — for each node, a list of its neighbors.

```java
import java.util.*;

class GraphBuilder {
    // Build an undirected adjacency list for n nodes (0-indexed) from edges.
    public static List<List<Integer>> buildUndirected(int n, int[][] edges) {
        List<List<Integer>> adj = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            adj.add(new ArrayList<>());
        }
        for (int[] e : edges) {
            int u = e[0], v = e[1];
            adj.get(u).add(v);
            adj.get(v).add(u); // omit this line for a DIRECTED graph
        }
        return adj;
    }
}
```

For a **directed** graph, only add `adj.get(u).add(v)`.

### `boolean[] visited` vs `Set<Integer>`

| Structure | Use when | Cost |
|-----------|----------|------|
| `boolean[] visited` | Nodes are integers in a known range `[0, n)` | O(1) access, O(n) space, cache-friendly |
| `Set<Integer>` (or `Set<String>`) | Node identifiers are sparse, non-integer, or unknown range (strings, coordinates encoded as keys, generic objects) | O(1) average, hashing overhead |

Rule of thumb: if the problem gives you `n` and integer-labeled nodes, use `boolean[] visited`. If nodes are strings (Word Ladder) or arbitrary objects (Clone Graph), use a `Set` / `Map`.

---

## 3. BFS Template (with level tracking)

The **size-loop** idiom is how you process one full level at a time: snapshot `queue.size()` before the inner loop so newly added nodes (the next level) don't get counted in the current level.

```java
import java.util.*;

class BFSTemplate {
    public int bfs(int start, List<List<Integer>> adj, int n) {
        boolean[] visited = new boolean[n];
        Queue<Integer> queue = new ArrayDeque<>();

        queue.offer(start);
        visited[start] = true;   // mark visited when ENQUEUED, not when dequeued
        int level = 0;

        while (!queue.isEmpty()) {
            int size = queue.size();        // snapshot: number of nodes at this level
            for (int i = 0; i < size; i++) {
                int node = queue.poll();
                // ---- process `node` here (it is at distance `level` from start) ----
                for (int next : adj.get(node)) {
                    if (!visited[next]) {
                        visited[next] = true;
                        queue.offer(next);
                    }
                }
            }
            level++;   // finished an entire level
        }
        return level;
    }
}
```

---

## 4. DFS Templates

### Recursive DFS

```java
import java.util.*;

class DFSRecursive {
    public void dfs(int node, List<List<Integer>> adj, boolean[] visited) {
        visited[node] = true;
        // ---- process `node` here ----
        for (int next : adj.get(node)) {
            if (!visited[next]) {
                dfs(next, adj, visited);
            }
        }
    }
}
```

### Iterative DFS (explicit stack)

Equivalent traversal without recursion — safer for very deep graphs (avoids stack overflow).

```java
import java.util.*;

class DFSIterative {
    public void dfs(int start, List<List<Integer>> adj, int n) {
        boolean[] visited = new boolean[n];
        Deque<Integer> stack = new ArrayDeque<>();
        stack.push(start);

        while (!stack.isEmpty()) {
            int node = stack.pop();
            if (visited[node]) continue;   // a node can be pushed multiple times
            visited[node] = true;
            // ---- process `node` here ----
            for (int next : adj.get(node)) {
                if (!visited[next]) {
                    stack.push(next);
                }
            }
        }
    }
}
```

> Note: in iterative DFS we check `visited` on **pop** (not push) because the same node may be pushed by several neighbors before it is popped. Guarding on pop keeps it correct.

---

## 5. Recognition Signals

| Signal in the problem | Use this |
|-----------------------|----------|
| "shortest path" / "minimum steps" / "fewest moves" on **unweighted** graph | **BFS** |
| "level order" / process layer by layer | **BFS (size loop)** |
| Spread from multiple sources simultaneously | **Multi-source BFS** |
| "number of connected components" / "number of groups/provinces/islands" | **DFS or BFS** flood |
| "is it reachable" / "can we get from A to B" | **DFS or BFS** |
| Grid of cells, expand to 4 (or 8) neighbors | **Grid DFS/BFS with `dirs`** |
| Color nodes / two groups / no two adjacent equal | **Bipartite check (BFS/DFS 2-coloring)** |
| Reconstruct **all** shortest paths | **BFS to build parent map + DFS backtrack** |
| Generic node objects, must duplicate structure | **DFS/BFS with `Map<orig,clone>`** |

---

## 6. The Problems

---

## 1. Number of Provinces — Medium (LC547)

**Idea:** The matrix `isConnected[i][j] == 1` means cities `i` and `j` are directly connected. A *province* is a connected component. Run DFS from each unvisited city; each DFS launch marks one whole province, so count the launches.

```java
class Solution {
    public int findCircleNum(int[][] isConnected) {
        int n = isConnected.length;
        boolean[] visited = new boolean[n];
        int provinces = 0;

        for (int i = 0; i < n; i++) {
            if (!visited[i]) {
                dfs(i, isConnected, visited);
                provinces++;
            }
        }
        return provinces;
    }

    private void dfs(int city, int[][] isConnected, boolean[] visited) {
        visited[city] = true;
        for (int next = 0; next < isConnected.length; next++) {
            if (isConnected[city][next] == 1 && !visited[next]) {
                dfs(next, isConnected, visited);
            }
        }
    }
}
```

**Complexity:** Time O(n²) (we scan the full row for every city), Space O(n) for `visited` + recursion stack.

---

## 2. Number of Islands — Medium (LC200)

**Idea:** Treat the grid as a graph where adjacent `'1'` cells are connected. Each time we find an unvisited `'1'`, we flood-fill (DFS) the entire island, sinking it to `'0'`, and increment the island count.

```java
class Solution {
    int[][] dirs = {{1,0},{-1,0},{0,1},{0,-1}};

    public int numIslands(char[][] grid) {
        int rows = grid.length, cols = grid[0].length;
        int count = 0;

        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                if (grid[r][c] == '1') {
                    dfs(grid, r, c);
                    count++;
                }
            }
        }
        return count;
    }

    private void dfs(char[][] grid, int r, int c) {
        if (r < 0 || r >= grid.length || c < 0 || c >= grid[0].length
                || grid[r][c] != '1') {
            return;
        }
        grid[r][c] = '0';                  // sink this land cell
        for (int[] d : dirs) {
            dfs(grid, r + d[0], c + d[1]);
        }
    }
}
```

**Complexity:** Time O(rows × cols) — each cell visited once. Space O(rows × cols) worst-case recursion depth (a grid that is one giant island).

### Dry run — Number of Islands

Grid:
```
1 1 0 0
1 0 0 1
0 0 1 1
0 0 0 0
```

Scan in row-major order:

1. `(0,0) == '1'` → start DFS #1. It sinks `(0,0)`, then `(1,0)` (down), then `(0,1)` (right of origin). All connected land in the top-left becomes `'0'`. `count = 1`.
   - Grid now:
     ```
     0 0 0 0
     0 0 0 1
     0 0 1 1
     0 0 0 0
     ```
2. Continue scanning. Cells `(0,1)`, `(0,2)`... are now `'0'`, skipped.
3. `(1,3) == '1'` → start DFS #2. Sinks `(1,3)`, goes down to `(2,3)`, then left to `(2,2)`. This whole bottom-right blob sinks. `count = 2`.
   - Grid now all `'0'`.
4. Remaining scan finds only `'0'` cells.

**Result: 2 islands.** Notice the diagonal between the top-left blob and `(2,2)` does **not** connect them — only the 4 directions in `dirs` count.

---

## 3. Flood Fill — Easy (LC733)

**Idea:** Standard flood fill. Starting at `(sr, sc)`, repaint every 4-directionally connected cell that shares the original color with `newColor`. Guard against the case `newColor == originalColor` to avoid infinite recursion.

```java
class Solution {
    int[][] dirs = {{1,0},{-1,0},{0,1},{0,-1}};

    public int[][] floodFill(int[][] image, int sr, int sc, int newColor) {
        int original = image[sr][sc];
        if (original == newColor) return image;   // nothing to do; avoids infinite loop
        dfs(image, sr, sc, original, newColor);
        return image;
    }

    private void dfs(int[][] image, int r, int c, int original, int newColor) {
        if (r < 0 || r >= image.length || c < 0 || c >= image[0].length
                || image[r][c] != original) {
            return;
        }
        image[r][c] = newColor;
        for (int[] d : dirs) {
            dfs(image, r + d[0], c + d[1], original, newColor);
        }
    }
}
```

**Complexity:** Time O(rows × cols), Space O(rows × cols) recursion depth.

---

## 4. Clone Graph — Medium (LC133)

**Idea:** Deep-copy a connected undirected graph. Maintain a `HashMap<Node, Node>` mapping each original node to its clone. During DFS, the first time we see a node we create its clone and recurse on neighbors; if we've seen it before, we return the existing clone (this also breaks cycles).

```java
import java.util.*;

// Definition for a Node.
class Node {
    public int val;
    public List<Node> neighbors;
    public Node() {
        val = 0;
        neighbors = new ArrayList<>();
    }
    public Node(int val) {
        this.val = val;
        neighbors = new ArrayList<>();
    }
    public Node(int val, ArrayList<Node> neighbors) {
        this.val = val;
        this.neighbors = neighbors;
    }
}

class Solution {
    private Map<Node, Node> visited = new HashMap<>();

    public Node cloneGraph(Node node) {
        if (node == null) return null;

        // If already cloned, return the stored clone (handles cycles).
        if (visited.containsKey(node)) {
            return visited.get(node);
        }

        Node clone = new Node(node.val);
        visited.put(node, clone);              // record BEFORE recursing

        for (Node neighbor : node.neighbors) {
            clone.neighbors.add(cloneGraph(neighbor));
        }
        return clone;
    }
}
```

**Complexity:** Time O(V + E), Space O(V) for the map + recursion stack.

---

## 5. Surrounded Regions — Medium (LC130)

**Idea:** An `'O'` survives only if it is connected to a border `'O'`. So first, from every border `'O'`, DFS and mark all reachable `'O'`s as safe (temporary `'#'`). Then sweep the board: any remaining `'O'` is surrounded → flip to `'X'`; restore `'#'` back to `'O'`.

```java
class Solution {
    int[][] dirs = {{1,0},{-1,0},{0,1},{0,-1}};

    public void solve(char[][] board) {
        if (board == null || board.length == 0) return;
        int rows = board.length, cols = board[0].length;

        // 1. Mark border-connected 'O's as safe ('#').
        for (int r = 0; r < rows; r++) {
            dfs(board, r, 0);
            dfs(board, r, cols - 1);
        }
        for (int c = 0; c < cols; c++) {
            dfs(board, 0, c);
            dfs(board, rows - 1, c);
        }

        // 2. Flip surrounded 'O' -> 'X', restore safe '#' -> 'O'.
        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                if (board[r][c] == 'O') board[r][c] = 'X';
                else if (board[r][c] == '#') board[r][c] = 'O';
            }
        }
    }

    private void dfs(char[][] board, int r, int c) {
        if (r < 0 || r >= board.length || c < 0 || c >= board[0].length
                || board[r][c] != 'O') {
            return;
        }
        board[r][c] = '#';                 // safe marker
        for (int[] d : dirs) {
            dfs(board, r + d[0], c + d[1]);
        }
    }
}
```

**Complexity:** Time O(rows × cols), Space O(rows × cols) recursion depth.

---

## 6. Pacific Atlantic Water Flow — Medium (LC417)

**Idea:** Instead of asking which cells flow to *both* oceans (expensive per-cell), reverse the flow: DFS **inland from each ocean's border**, climbing only to cells of **equal-or-greater** height. Cells reachable from the Pacific border AND the Atlantic border are the answer.

```java
import java.util.*;

class Solution {
    int[][] dirs = {{1,0},{-1,0},{0,1},{0,-1}};

    public List<List<Integer>> pacificAtlantic(int[][] heights) {
        List<List<Integer>> result = new ArrayList<>();
        if (heights == null || heights.length == 0) return result;

        int rows = heights.length, cols = heights[0].length;
        boolean[][] pacific = new boolean[rows][cols];
        boolean[][] atlantic = new boolean[rows][cols];

        // Pacific touches top row and left column.
        // Atlantic touches bottom row and right column.
        for (int r = 0; r < rows; r++) {
            dfs(heights, r, 0, pacific);
            dfs(heights, r, cols - 1, atlantic);
        }
        for (int c = 0; c < cols; c++) {
            dfs(heights, 0, c, pacific);
            dfs(heights, rows - 1, c, atlantic);
        }

        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                if (pacific[r][c] && atlantic[r][c]) {
                    result.add(Arrays.asList(r, c));
                }
            }
        }
        return result;
    }

    private void dfs(int[][] heights, int r, int c, boolean[][] ocean) {
        ocean[r][c] = true;
        for (int[] d : dirs) {
            int nr = r + d[0], nc = c + d[1];
            if (nr < 0 || nr >= heights.length || nc < 0 || nc >= heights[0].length) {
                continue;
            }
            // climb only to equal-or-higher cells (reverse of water flow)
            if (!ocean[nr][nc] && heights[nr][nc] >= heights[r][c]) {
                dfs(heights, nr, nc, ocean);
            }
        }
    }
}
```

**Complexity:** Time O(rows × cols), Space O(rows × cols) for the two visited matrices.

---

## 7. Is Graph Bipartite — Medium (LC785)

**Idea:** A graph is bipartite iff it is 2-colorable. Use `int[] color` initialized to `-1` (uncolored). For each uncolored node, BFS and color neighbors the opposite color. If we ever find a neighbor already colored the **same** as the current node, it's not bipartite.

```java
import java.util.*;

class Solution {
    public boolean isBipartite(int[][] graph) {
        int n = graph.length;
        int[] color = new int[n];
        Arrays.fill(color, -1);              // -1 = uncolored

        for (int start = 0; start < n; start++) {
            if (color[start] != -1) continue;     // already handled component

            Queue<Integer> queue = new ArrayDeque<>();
            queue.offer(start);
            color[start] = 0;

            while (!queue.isEmpty()) {
                int node = queue.poll();
                for (int next : graph[node]) {
                    if (color[next] == -1) {
                        color[next] = 1 - color[node];   // opposite color
                        queue.offer(next);
                    } else if (color[next] == color[node]) {
                        return false;                    // conflict
                    }
                }
            }
        }
        return true;
    }
}
```

**Complexity:** Time O(V + E), Space O(V).

---

## 8. Word Ladder — Hard (LC127)

**Idea:** Each word is a node; two words are connected if they differ by exactly one letter. We want the **shortest transformation length** → BFS. To find neighbors efficiently, pre-build buckets keyed by wildcard patterns (e.g., `h*t`); all words sharing a pattern are one-edit neighbors.

```java
import java.util.*;

class Solution {
    public int ladderLength(String beginWord, String endWord, List<String> wordList) {
        Set<String> dict = new HashSet<>(wordList);
        if (!dict.contains(endWord)) return 0;

        int L = beginWord.length();

        // Build wildcard buckets: "h*t" -> [hot, hat, ...]
        Map<String, List<String>> patterns = new HashMap<>();
        for (String word : dict) {
            for (int i = 0; i < L; i++) {
                String key = word.substring(0, i) + "*" + word.substring(i + 1);
                patterns.computeIfAbsent(key, k -> new ArrayList<>()).add(word);
            }
        }

        Queue<String> queue = new ArrayDeque<>();
        Set<String> visited = new HashSet<>();
        queue.offer(beginWord);
        visited.add(beginWord);
        int level = 1;                        // beginWord counts as the first word

        while (!queue.isEmpty()) {
            int size = queue.size();
            for (int i = 0; i < size; i++) {
                String word = queue.poll();
                if (word.equals(endWord)) return level;

                for (int j = 0; j < L; j++) {
                    String key = word.substring(0, j) + "*" + word.substring(j + 1);
                    for (String neighbor : patterns.getOrDefault(key, Collections.emptyList())) {
                        if (!visited.contains(neighbor)) {
                            visited.add(neighbor);
                            queue.offer(neighbor);
                        }
                    }
                }
            }
            level++;
        }
        return 0;
    }
}
```

**Complexity:** Time O(N × L²) — N words, L word length; building each pattern is O(L) and there are L patterns per word, and substring is O(L). Space O(N × L²) for the bucket map.

### Dry run — Word Ladder

`beginWord = "hit"`, `endWord = "cog"`, `wordList = ["hot","dot","dog","lot","log","cog"]`.

**Build pattern buckets** (only for dict words):
```
*ot -> [hot, dot, lot]
h*t -> [hot]
ho* -> [hot]
d*t -> [dot]
do* -> [dot, dog]
*og -> [dog, log, cog]
d*g -> [dog]
l*t -> [lot]
lo* -> [lot, log]
l*g -> [log]
c*g -> [cog]
co* -> [cog]
```

**BFS** (level = number of words in the sequence so far):

- **Level 1:** queue = `[hit]`. Dequeue `hit`. Patterns of `hit`: `*it` (no bucket), `h*t` → `[hot]`, `hi*` (none). Enqueue `hot`. visited = {hit, hot}. → level becomes 2.
- **Level 2:** queue = `[hot]`. Dequeue `hot`. Patterns: `*ot` → `[hot,dot,lot]`, `h*t` → `[hot]`, `ho*` → `[hot]`. New neighbors: `dot`, `lot`. Enqueue both. visited adds dot, lot. → level 3.
- **Level 3:** queue = `[dot, lot]`.
  - `dot`: `*ot` → dot,hot,lot (seen); `d*t` → dot (seen); `do*` → dot,**dog**(new). Enqueue dog.
  - `lot`: `*ot` (seen); `l*t` → lot (seen); `lo*` → lot,**log**(new). Enqueue log.
  - visited adds dog, log. → level 4.
- **Level 4:** queue = `[dog, log]`.
  - `dog`: `*og` → dog,log,**cog**(new). Enqueue cog.
  - `log`: `*og` → cog already visited now.
  - visited adds cog. → level 5.
- **Level 5:** queue = `[cog]`. Dequeue `cog` → equals `endWord`. **Return 5.**

Path: `hit → hot → dot → dog → cog` (or `hit → hot → lot → log → cog`), length **5 words**.

---

## 9. Word Ladder II — Hard (LC126)

**Idea:** Now we must return **all** shortest transformation sequences. Do BFS level by level recording, for each word, the set of predecessors (`parents`) that first reached it at the shortest distance. Stop BFS at the level where `endWord` appears. Then DFS-backtrack from `endWord` through the parent map to reconstruct every shortest path.

```java
import java.util.*;

class Solution {
    public List<List<String>> findLadders(String beginWord, String endWord, List<String> wordList) {
        List<List<String>> result = new ArrayList<>();
        Set<String> dict = new HashSet<>(wordList);
        if (!dict.contains(endWord)) return result;

        int L = beginWord.length();

        // parents.get(w) = words that reached w on the shortest path
        Map<String, List<String>> parents = new HashMap<>();
        Set<String> current = new HashSet<>();
        current.add(beginWord);
        Set<String> visited = new HashSet<>();
        visited.add(beginWord);
        boolean found = false;

        while (!current.isEmpty() && !found) {
            // Remove this level's words from dict so we don't revisit earlier levels.
            dict.removeAll(current);
            Set<String> next = new HashSet<>();

            for (String word : current) {
                char[] chars = word.toCharArray();
                for (int i = 0; i < L; i++) {
                    char old = chars[i];
                    for (char c = 'a'; c <= 'z'; c++) {
                        if (c == old) continue;
                        chars[i] = c;
                        String cand = new String(chars);
                        if (dict.contains(cand)) {
                            if (visited.add(cand)) {      // first time at this BFS depth
                                next.add(cand);
                            }
                            // record predecessor regardless (same level reach)
                            if (next.contains(cand) || cand.equals(endWord)) {
                                parents.computeIfAbsent(cand, k -> new ArrayList<>()).add(word);
                            }
                            if (cand.equals(endWord)) found = true;
                        }
                    }
                    chars[i] = old;
                }
            }
            current = next;
        }

        if (found) {
            LinkedList<String> path = new LinkedList<>();
            path.add(endWord);
            backtrack(endWord, beginWord, parents, path, result);
        }
        return result;
    }

    private void backtrack(String word, String beginWord,
                           Map<String, List<String>> parents,
                           LinkedList<String> path, List<List<String>> result) {
        if (word.equals(beginWord)) {
            result.add(new ArrayList<>(path));
            return;
        }
        for (String parent : parents.getOrDefault(word, Collections.emptyList())) {
            path.addFirst(parent);
            backtrack(parent, beginWord, parents, path, result);
            path.removeFirst();
        }
    }
}
```

**Complexity:** Time O(N × L × 26 + P) where P is the cost of enumerating all shortest paths; Space O(N × L) for the parent map plus output.

---

## 10. Open the Lock — Medium (LC752)

**Idea:** Each 4-digit state is a node; turning one wheel one notch (up or down, with wraparound 0↔9) gives 8 neighbors. We want the **minimum number of moves** from `"0000"` to `target` → BFS. Treat each deadend as already-visited so BFS never expands it.

```java
import java.util.*;

class Solution {
    public int openLock(String[] deadends, String target) {
        Set<String> dead = new HashSet<>(Arrays.asList(deadends));
        if (dead.contains("0000")) return -1;
        if (target.equals("0000")) return 0;

        Set<String> visited = new HashSet<>(dead);   // deadends count as visited
        Queue<String> queue = new ArrayDeque<>();
        queue.offer("0000");
        visited.add("0000");
        int moves = 0;

        while (!queue.isEmpty()) {
            int size = queue.size();
            for (int i = 0; i < size; i++) {
                String state = queue.poll();
                if (state.equals(target)) return moves;

                for (String next : neighbors(state)) {
                    if (!visited.contains(next)) {
                        visited.add(next);
                        queue.offer(next);
                    }
                }
            }
            moves++;
        }
        return -1;
    }

    private List<String> neighbors(String state) {
        List<String> res = new ArrayList<>();
        char[] arr = state.toCharArray();
        for (int i = 0; i < 4; i++) {
            char old = arr[i];

            arr[i] = (old == '9') ? '0' : (char) (old + 1);   // turn up
            res.add(new String(arr));

            arr[i] = (old == '0') ? '9' : (char) (old - 1);   // turn down
            res.add(new String(arr));

            arr[i] = old;
        }
        return res;
    }
}
```

**Complexity:** Time O(10⁴ × 8) ≈ O(N) over the bounded state space (10000 states, 8 neighbors each), Space O(10⁴) for the visited set.

---

## Summary

- **BFS** = queue, mark-on-enqueue, size-loop for levels → shortest path / minimum steps on unweighted graphs.
- **DFS** = recursion or explicit stack, mark-on-pop (iterative) → connectivity, flood fill, exhaustive exploration.
- **Grid problems** reduce to graph traversal using the 4-direction `dirs` array.
- **Reverse-thinking** (Surrounded Regions, Pacific Atlantic) often turns an expensive per-cell query into a few border-launched traversals.
- **Parent maps** let BFS reconstruct *all* shortest paths (Word Ladder II).

> **Last Updated:** 2026-06-26
