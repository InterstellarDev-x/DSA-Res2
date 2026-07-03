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

```cpp
#include <bits/stdc++.h>
using namespace std;

class GraphBuilder {
public:
    // Build an undirected adjacency list for n nodes (0-indexed) from edges.
    static vector<vector<int>> buildUndirected(int n, vector<vector<int>>& edges) {
        vector<vector<int>> adj(n);
        for (auto& e : edges) {
            int u = e[0], v = e[1];
            adj[u].push_back(v);
            adj[v].push_back(u); // omit this line for a DIRECTED graph
        }
        return adj;
    }
};
```

For a **directed** graph, only add `adj[u].push_back(v)`.

### `vector<bool> visited` vs `unordered_set<int>`

| Structure | Use when | Cost |
|-----------|----------|------|
| `vector<bool> visited` | Nodes are integers in a known range `[0, n)` | O(1) access, O(n) space, cache-friendly |
| `unordered_set<int>` (or `unordered_set<string>`) | Node identifiers are sparse, non-integer, or unknown range (strings, coordinates encoded as keys, generic objects) | O(1) average, hashing overhead |

Rule of thumb: if the problem gives you `n` and integer-labeled nodes, use `vector<bool> visited`. If nodes are strings (Word Ladder) or arbitrary objects (Clone Graph), use an `unordered_set` / `unordered_map`.

---

## 3. BFS Template (with level tracking)

The **size-loop** idiom is how you process one full level at a time: snapshot `queue.size()` before the inner loop so newly added nodes (the next level) don't get counted in the current level.

```cpp
#include <bits/stdc++.h>
using namespace std;

class BFSTemplate {
public:
    int bfs(int start, vector<vector<int>>& adj, int n) {
        vector<bool> visited(n, false);
        queue<int> q;

        q.push(start);
        visited[start] = true;   // mark visited when ENQUEUED, not when dequeued
        int level = 0;

        while (!q.empty()) {
            int size = q.size();        // snapshot: number of nodes at this level
            for (int i = 0; i < size; i++) {
                int node = q.front(); q.pop();
                // ---- process `node` here (it is at distance `level` from start) ----
                for (int next : adj[node]) {
                    if (!visited[next]) {
                        visited[next] = true;
                        q.push(next);
                    }
                }
            }
            level++;   // finished an entire level
        }
        return level;
    }
};
```

---

## 4. DFS Templates

### Recursive DFS

```cpp
#include <bits/stdc++.h>
using namespace std;

class DFSRecursive {
public:
    void dfs(int node, vector<vector<int>>& adj, vector<bool>& visited) {
        visited[node] = true;
        // ---- process `node` here ----
        for (int next : adj[node]) {
            if (!visited[next]) {
                dfs(next, adj, visited);
            }
        }
    }
};
```

### Iterative DFS (explicit stack)

Equivalent traversal without recursion — safer for very deep graphs (avoids stack overflow).

```cpp
#include <bits/stdc++.h>
using namespace std;

class DFSIterative {
public:
    void dfs(int start, vector<vector<int>>& adj, int n) {
        vector<bool> visited(n, false);
        stack<int> stk;
        stk.push(start);

        while (!stk.empty()) {
            int node = stk.top(); stk.pop();
            if (visited[node]) continue;   // a node can be pushed multiple times
            visited[node] = true;
            // ---- process `node` here ----
            for (int next : adj[node]) {
                if (!visited[next]) {
                    stk.push(next);
                }
            }
        }
    }
};
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
| Generic node objects, must duplicate structure | **DFS/BFS with `unordered_map<orig,clone>`** |

---

## 6. The Problems

---

## 1. Number of Provinces — Medium (LC547)

**Idea:** The matrix `isConnected[i][j] == 1` means cities `i` and `j` are directly connected. A *province* is a connected component. Run DFS from each unvisited city; each DFS launch marks one whole province, so count the launches.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    int findCircleNum(vector<vector<int>>& isConnected) {
        int n = isConnected.size();
        vector<bool> visited(n, false);
        int provinces = 0;

        for (int i = 0; i < n; i++) {
            if (!visited[i]) {
                dfs(i, isConnected, visited);
                provinces++;
            }
        }
        return provinces;
    }

private:
    void dfs(int city, vector<vector<int>>& isConnected, vector<bool>& visited) {
        visited[city] = true;
        for (int next = 0; next < (int)isConnected.size(); next++) {
            if (isConnected[city][next] == 1 && !visited[next]) {
                dfs(next, isConnected, visited);
            }
        }
    }
};
```

**Complexity:** Time O(n²) (we scan the full row for every city), Space O(n) for `visited` + recursion stack.

---

## 2. Number of Islands — Medium (LC200)

**Idea:** Treat the grid as a graph where adjacent `'1'` cells are connected. Each time we find an unvisited `'1'`, we flood-fill (DFS) the entire island, sinking it to `'0'`, and increment the island count.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
    int dirs[4][2] = {{1,0},{-1,0},{0,1},{0,-1}};
public:
    int numIslands(vector<vector<char>>& grid) {
        int rows = grid.size(), cols = grid[0].size();
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

private:
    void dfs(vector<vector<char>>& grid, int r, int c) {
        if (r < 0 || r >= (int)grid.size() || c < 0 || c >= (int)grid[0].size()
                || grid[r][c] != '1') {
            return;
        }
        grid[r][c] = '0';                  // sink this land cell
        for (auto& d : dirs) {
            dfs(grid, r + d[0], c + d[1]);
        }
    }
};
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

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
    int dirs[4][2] = {{1,0},{-1,0},{0,1},{0,-1}};
public:
    vector<vector<int>> floodFill(vector<vector<int>>& image, int sr, int sc, int newColor) {
        int original = image[sr][sc];
        if (original == newColor) return image;   // nothing to do; avoids infinite loop
        dfs(image, sr, sc, original, newColor);
        return image;
    }

private:
    void dfs(vector<vector<int>>& image, int r, int c, int original, int newColor) {
        if (r < 0 || r >= (int)image.size() || c < 0 || c >= (int)image[0].size()
                || image[r][c] != original) {
            return;
        }
        image[r][c] = newColor;
        for (auto& d : dirs) {
            dfs(image, r + d[0], c + d[1], original, newColor);
        }
    }
};
```

**Complexity:** Time O(rows × cols), Space O(rows × cols) recursion depth.

---

## 4. Clone Graph — Medium (LC133)

**Idea:** Deep-copy a connected undirected graph. Maintain an `unordered_map<Node*, Node*>` mapping each original node to its clone. During DFS, the first time we see a node we create its clone and recurse on neighbors; if we've seen it before, we return the existing clone (this also breaks cycles).

```cpp
#include <bits/stdc++.h>
using namespace std;

// Definition for a Node.
class Node {
public:
    int val;
    vector<Node*> neighbors;
    Node() {
        val = 0;
        neighbors = {};
    }
    Node(int _val) {
        val = _val;
        neighbors = {};
    }
    Node(int _val, vector<Node*> _neighbors) {
        val = _val;
        neighbors = _neighbors;
    }
};

class Solution {
private:
    unordered_map<Node*, Node*> visited;

public:
    Node* cloneGraph(Node* node) {
        if (node == nullptr) return nullptr;

        // If already cloned, return the stored clone (handles cycles).
        if (visited.count(node)) {
            return visited[node];
        }

        Node* clone = new Node(node->val);
        visited[node] = clone;              // record BEFORE recursing

        for (auto& neighbor : node->neighbors) {
            clone->neighbors.push_back(cloneGraph(neighbor));
        }
        return clone;
    }
};
```

**Complexity:** Time O(V + E), Space O(V) for the map + recursion stack.

---

## 5. Surrounded Regions — Medium (LC130)

**Idea:** An `'O'` survives only if it is connected to a border `'O'`. So first, from every border `'O'`, DFS and mark all reachable `'O'`s as safe (temporary `'#'`). Then sweep the board: any remaining `'O'` is surrounded → flip to `'X'`; restore `'#'` back to `'O'`.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
    int dirs[4][2] = {{1,0},{-1,0},{0,1},{0,-1}};
public:
    void solve(vector<vector<char>>& board) {
        if (board.empty() || board[0].empty()) return;
        int rows = board.size(), cols = board[0].size();

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

private:
    void dfs(vector<vector<char>>& board, int r, int c) {
        if (r < 0 || r >= (int)board.size() || c < 0 || c >= (int)board[0].size()
                || board[r][c] != 'O') {
            return;
        }
        board[r][c] = '#';                 // safe marker
        for (auto& d : dirs) {
            dfs(board, r + d[0], c + d[1]);
        }
    }
};
```

**Complexity:** Time O(rows × cols), Space O(rows × cols) recursion depth.

---

## 6. Pacific Atlantic Water Flow — Medium (LC417)

**Idea:** Instead of asking which cells flow to *both* oceans (expensive per-cell), reverse the flow: DFS **inland from each ocean's border**, climbing only to cells of **equal-or-greater** height. Cells reachable from the Pacific border AND the Atlantic border are the answer.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
    int dirs[4][2] = {{1,0},{-1,0},{0,1},{0,-1}};
public:
    vector<vector<int>> pacificAtlantic(vector<vector<int>>& heights) {
        vector<vector<int>> result;
        if (heights.empty() || heights[0].empty()) return result;

        int rows = heights.size(), cols = heights[0].size();
        vector<vector<bool>> pacific(rows, vector<bool>(cols, false));
        vector<vector<bool>> atlantic(rows, vector<bool>(cols, false));

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
                    result.push_back({r, c});
                }
            }
        }
        return result;
    }

private:
    void dfs(vector<vector<int>>& heights, int r, int c, vector<vector<bool>>& ocean) {
        ocean[r][c] = true;
        for (auto& d : dirs) {
            int nr = r + d[0], nc = c + d[1];
            if (nr < 0 || nr >= (int)heights.size() || nc < 0 || nc >= (int)heights[0].size()) {
                continue;
            }
            // climb only to equal-or-higher cells (reverse of water flow)
            if (!ocean[nr][nc] && heights[nr][nc] >= heights[r][c]) {
                dfs(heights, nr, nc, ocean);
            }
        }
    }
};
```

**Complexity:** Time O(rows × cols), Space O(rows × cols) for the two visited matrices.

---

## 7. Is Graph Bipartite — Medium (LC785)

**Idea:** A graph is bipartite iff it is 2-colorable. Use `vector<int> color` initialized to `-1` (uncolored). For each uncolored node, BFS and color neighbors the opposite color. If we ever find a neighbor already colored the **same** as the current node, it's not bipartite.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    bool isBipartite(vector<vector<int>>& graph) {
        int n = graph.size();
        vector<int> color(n, -1);              // -1 = uncolored

        for (int start = 0; start < n; start++) {
            if (color[start] != -1) continue;     // already handled component

            queue<int> q;
            q.push(start);
            color[start] = 0;

            while (!q.empty()) {
                int node = q.front(); q.pop();
                for (int next : graph[node]) {
                    if (color[next] == -1) {
                        color[next] = 1 - color[node];   // opposite color
                        q.push(next);
                    } else if (color[next] == color[node]) {
                        return false;                    // conflict
                    }
                }
            }
        }
        return true;
    }
};
```

**Complexity:** Time O(V + E), Space O(V).

---

## 8. Word Ladder — Hard (LC127)

**Idea:** Each word is a node; two words are connected if they differ by exactly one letter. We want the **shortest transformation length** → BFS. To find neighbors efficiently, pre-build buckets keyed by wildcard patterns (e.g., `h*t`); all words sharing a pattern are one-edit neighbors.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    int ladderLength(string beginWord, string endWord, vector<string>& wordList) {
        unordered_set<string> dict(wordList.begin(), wordList.end());
        if (!dict.count(endWord)) return 0;

        int L = beginWord.length();

        // Build wildcard buckets: "h*t" -> [hot, hat, ...]
        unordered_map<string, vector<string>> patterns;
        for (auto& word : dict) {
            for (int i = 0; i < L; i++) {
                string key = word.substr(0, i) + "*" + word.substr(i + 1);
                patterns[key].push_back(word);
            }
        }

        queue<string> q;
        unordered_set<string> visited;
        q.push(beginWord);
        visited.insert(beginWord);
        int level = 1;                        // beginWord counts as the first word

        while (!q.empty()) {
            int size = q.size();
            for (int i = 0; i < size; i++) {
                string word = q.front(); q.pop();
                if (word == endWord) return level;

                for (int j = 0; j < L; j++) {
                    string key = word.substr(0, j) + "*" + word.substr(j + 1);
                    auto it = patterns.find(key);
                    if (it != patterns.end()) {
                        for (auto& neighbor : it->second) {
                            if (!visited.count(neighbor)) {
                                visited.insert(neighbor);
                                q.push(neighbor);
                            }
                        }
                    }
                }
            }
            level++;
        }
        return 0;
    }
};
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

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    vector<vector<string>> findLadders(string beginWord, string endWord, vector<string>& wordList) {
        vector<vector<string>> result;
        unordered_set<string> dict(wordList.begin(), wordList.end());
        if (!dict.count(endWord)) return result;

        int L = beginWord.length();

        // parents[w] = words that reached w on the shortest path
        unordered_map<string, vector<string>> parents;
        unordered_set<string> current;
        current.insert(beginWord);
        unordered_set<string> visited;
        visited.insert(beginWord);
        bool found = false;

        while (!current.empty() && !found) {
            // Remove this level's words from dict so we don't revisit earlier levels.
            for (auto& w : current) dict.erase(w);
            unordered_set<string> next;

            for (auto& word : current) {
                string chars = word;
                for (int i = 0; i < L; i++) {
                    char old = chars[i];
                    for (char c = 'a'; c <= 'z'; c++) {
                        if (c == old) continue;
                        chars[i] = c;
                        string cand = chars;
                        if (dict.count(cand)) {
                            if (!visited.count(cand)) {      // first time at this BFS depth
                                visited.insert(cand);
                                next.insert(cand);
                            }
                            // record predecessor regardless (same level reach)
                            if (next.count(cand) || cand == endWord) {
                                parents[cand].push_back(word);
                            }
                            if (cand == endWord) found = true;
                        }
                    }
                    chars[i] = old;
                }
            }
            current = next;
        }

        if (found) {
            vector<string> path = {endWord};
            backtrack(endWord, beginWord, parents, path, result);
        }
        return result;
    }

private:
    void backtrack(string word, string& beginWord,
                   unordered_map<string, vector<string>>& parents,
                   vector<string>& path, vector<vector<string>>& result) {
        if (word == beginWord) {
            result.push_back(vector<string>(path.rbegin(), path.rend()));
            return;
        }
        auto it = parents.find(word);
        if (it == parents.end()) return;
        for (auto& parent : it->second) {
            path.push_back(parent);
            backtrack(parent, beginWord, parents, path, result);
            path.pop_back();
        }
    }
};
```

**Complexity:** Time O(N × L × 26 + P) where P is the cost of enumerating all shortest paths; Space O(N × L) for the parent map plus output.

---

## 10. Open the Lock — Medium (LC752)

**Idea:** Each 4-digit state is a node; turning one wheel one notch (up or down, with wraparound 0↔9) gives 8 neighbors. We want the **minimum number of moves** from `"0000"` to `target` → BFS. Treat each deadend as already-visited so BFS never expands it.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    int openLock(vector<string>& deadends, string target) {
        unordered_set<string> dead(deadends.begin(), deadends.end());
        if (dead.count("0000")) return -1;
        if (target == "0000") return 0;

        unordered_set<string> visited(dead);   // deadends count as visited
        queue<string> q;
        q.push("0000");
        visited.insert("0000");
        int moves = 0;

        while (!q.empty()) {
            int size = q.size();
            for (int i = 0; i < size; i++) {
                string state = q.front(); q.pop();
                if (state == target) return moves;

                for (auto& next : neighbors(state)) {
                    if (!visited.count(next)) {
                        visited.insert(next);
                        q.push(next);
                    }
                }
            }
            moves++;
        }
        return -1;
    }

private:
    vector<string> neighbors(string state) {
        vector<string> res;
        for (int i = 0; i < 4; i++) {
            char old = state[i];

            state[i] = (old == '9') ? '0' : (char)(old + 1);   // turn up
            res.push_back(state);

            state[i] = (old == '0') ? '9' : (char)(old - 1);   // turn down
            res.push_back(state);

            state[i] = old;
        }
        return res;
    }
};
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
