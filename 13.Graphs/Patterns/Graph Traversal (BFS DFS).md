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

```rust
struct GraphBuilder;

impl GraphBuilder {
    // Build an undirected adjacency list for n nodes (0-indexed) from edges.
    fn build_undirected(n: usize, edges: &[Vec<i32>]) -> Vec<Vec<usize>> {
        let mut adj = vec![vec![]; n];
        for e in edges {
            let u = e[0] as usize;
            let v = e[1] as usize;
            adj[u].push(v);
            adj[v].push(u); // omit this line for a DIRECTED graph
        }
        adj
    }
}
```

For a **directed** graph, only add `adj[u].push(v)`.

### `Vec<bool> visited` vs `HashSet<i32>`

| Structure | Use when | Cost |
|-----------|----------|------|
| `Vec<bool> visited` | Nodes are integers in a known range `[0, n)` | O(1) access, O(n) space, cache-friendly |
| `HashSet<i32>` (or `HashSet<String>`) | Node identifiers are sparse, non-integer, or unknown range (strings, coordinates encoded as keys, generic objects) | O(1) average, hashing overhead |

Rule of thumb: if the problem gives you `n` and integer-labeled nodes, use `Vec<bool> visited`. If nodes are strings (Word Ladder) or arbitrary objects (Clone Graph), use a `HashSet` / `HashMap`.

---

## 3. BFS Template (with level tracking)

The **size-loop** idiom is how you process one full level at a time: snapshot `queue.len()` before the inner loop so newly added nodes (the next level) don't get counted in the current level.

```rust
use std::collections::VecDeque;

struct BFSTemplate;

impl BFSTemplate {
    fn bfs(start: usize, adj: &[Vec<usize>], n: usize) -> i32 {
        let mut visited = vec![false; n];
        let mut q: VecDeque<usize> = VecDeque::new();

        q.push_back(start);
        visited[start] = true;   // mark visited when ENQUEUED, not when dequeued
        let mut level = 0;

        while !q.is_empty() {
            let size = q.len();        // snapshot: number of nodes at this level
            for _ in 0..size {
                let node = q.pop_front().unwrap();
                // ---- process `node` here (it is at distance `level` from start) ----
                for &next in &adj[node] {
                    if !visited[next] {
                        visited[next] = true;
                        q.push_back(next);
                    }
                }
            }
            level += 1;   // finished an entire level
        }
        level
    }
}
```

---

## 4. DFS Templates

### Recursive DFS

```rust
struct DFSRecursive;

impl DFSRecursive {
    fn dfs(node: usize, adj: &[Vec<usize>], visited: &mut Vec<bool>) {
        visited[node] = true;
        // ---- process `node` here ----
        for &next in &adj[node] {
            if !visited[next] {
                Self::dfs(next, adj, visited);
            }
        }
    }
}
```

### Iterative DFS (explicit stack)

Equivalent traversal without recursion — safer for very deep graphs (avoids stack overflow).

```rust
struct DFSIterative;

impl DFSIterative {
    fn dfs(start: usize, adj: &[Vec<usize>], n: usize) {
        let mut visited = vec![false; n];
        let mut stk: Vec<usize> = Vec::new();
        stk.push(start);

        while let Some(node) = stk.pop() {
            if visited[node] {
                continue;   // a node can be pushed multiple times
            }
            visited[node] = true;
            // ---- process `node` here ----
            for &next in &adj[node] {
                if !visited[next] {
                    stk.push(next);
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
| Generic node objects, must duplicate structure | **DFS/BFS with `HashMap<orig,clone>`** |

---

## 6. The Problems

---

## 1. Number of Provinces — Medium (LC547)

**Idea:** The matrix `isConnected[i][j] == 1` means cities `i` and `j` are directly connected. A *province* is a connected component. Run DFS from each unvisited city; each DFS launch marks one whole province, so count the launches.

```rust
struct Solution;

impl Solution {
    pub fn find_circle_num(is_connected: Vec<Vec<i32>>) -> i32 {
        let n = is_connected.len();
        let mut visited = vec![false; n];
        let mut provinces = 0;

        for i in 0..n {
            if !visited[i] {
                Self::dfs(i, &is_connected, &mut visited);
                provinces += 1;
            }
        }
        provinces
    }

    fn dfs(city: usize, is_connected: &[Vec<i32>], visited: &mut Vec<bool>) {
        visited[city] = true;
        for next in 0..is_connected.len() {
            if is_connected[city][next] == 1 && !visited[next] {
                Self::dfs(next, is_connected, visited);
            }
        }
    }
}
```

**Complexity:** Time O(n²) (we scan the full row for every city), Space O(n) for `visited` + recursion stack.

---

## 2. Number of Islands — Medium (LC200)

**Idea:** Treat the grid as a graph where adjacent `'1'` cells are connected. Each time we find an unvisited `'1'`, we flood-fill (DFS) the entire island, sinking it to `'0'`, and increment the island count.

```rust
struct Solution;

impl Solution {
    pub fn num_islands(mut grid: Vec<Vec<char>>) -> i32 {
        let rows = grid.len();
        let cols = grid[0].len();
        let mut count = 0;

        for r in 0..rows {
            for c in 0..cols {
                if grid[r][c] == '1' {
                    Self::dfs(&mut grid, r as i32, c as i32);
                    count += 1;
                }
            }
        }
        count
    }

    fn dfs(grid: &mut Vec<Vec<char>>, r: i32, c: i32) {
        let rows = grid.len() as i32;
        let cols = grid[0].len() as i32;
        if r < 0 || r >= rows || c < 0 || c >= cols || grid[r as usize][c as usize] != '1' {
            return;
        }
        grid[r as usize][c as usize] = '0';                  // sink this land cell
        let dirs = [(1i32, 0i32), (-1, 0), (0, 1), (0, -1)];
        for &(dr, dc) in &dirs {
            Self::dfs(grid, r + dr, c + dc);
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

```rust
struct Solution;

impl Solution {
    pub fn flood_fill(mut image: Vec<Vec<i32>>, sr: i32, sc: i32, new_color: i32) -> Vec<Vec<i32>> {
        let original = image[sr as usize][sc as usize];
        if original == new_color {
            return image;   // nothing to do; avoids infinite loop
        }
        Self::dfs(&mut image, sr, sc, original, new_color);
        image
    }

    fn dfs(image: &mut Vec<Vec<i32>>, r: i32, c: i32, original: i32, new_color: i32) {
        let rows = image.len() as i32;
        let cols = image[0].len() as i32;
        if r < 0 || r >= rows || c < 0 || c >= cols || image[r as usize][c as usize] != original {
            return;
        }
        image[r as usize][c as usize] = new_color;
        let dirs = [(1i32, 0i32), (-1, 0), (0, 1), (0, -1)];
        for &(dr, dc) in &dirs {
            Self::dfs(image, r + dr, c + dc, original, new_color);
        }
    }
}
```

**Complexity:** Time O(rows × cols), Space O(rows × cols) recursion depth.

---

## 4. Clone Graph — Medium (LC133)

**Idea:** Deep-copy a connected undirected graph. Maintain a `HashMap<i32, Rc<RefCell<Node>>>` mapping each original node's value to its clone. During DFS, the first time we see a node we create its clone and recurse on neighbors; if we've seen it before, we return the existing clone (this also breaks cycles).

```rust
use std::collections::HashMap;
use std::cell::RefCell;
use std::rc::Rc;

// Definition for a Node.
#[derive(Debug)]
pub struct Node {
    pub val: i32,
    pub neighbors: Vec<Option<Rc<RefCell<Node>>>>,
}

impl Node {
    pub fn new(val: i32) -> Self {
        Node { val, neighbors: vec![] }
    }
}

struct Solution;

impl Solution {
    pub fn clone_graph(node: Option<Rc<RefCell<Node>>>) -> Option<Rc<RefCell<Node>>> {
        let mut visited: HashMap<i32, Rc<RefCell<Node>>> = HashMap::new();
        Self::dfs(node, &mut visited)
    }

    fn dfs(
        node: Option<Rc<RefCell<Node>>>,
        visited: &mut HashMap<i32, Rc<RefCell<Node>>>,
    ) -> Option<Rc<RefCell<Node>>> {
        let node = node?;
        let val = node.borrow().val;

        // If already cloned, return the stored clone (handles cycles).
        if let Some(clone) = visited.get(&val) {
            return Some(clone.clone());
        }

        let clone = Rc::new(RefCell::new(Node::new(val)));
        visited.insert(val, clone.clone());  // record BEFORE recursing

        let neighbors: Vec<_> = node.borrow().neighbors.clone();
        for neighbor in neighbors {
            let cloned_neighbor = Self::dfs(neighbor, visited);
            clone.borrow_mut().neighbors.push(cloned_neighbor);
        }
        Some(clone)
    }
}
```

**Complexity:** Time O(V + E), Space O(V) for the map + recursion stack.

---

## 5. Surrounded Regions — Medium (LC130)

**Idea:** An `'O'` survives only if it is connected to a border `'O'`. So first, from every border `'O'`, DFS and mark all reachable `'O'`s as safe (temporary `'#'`). Then sweep the board: any remaining `'O'` is surrounded → flip to `'X'`; restore `'#'` back to `'O'`.

```rust
struct Solution;

impl Solution {
    pub fn solve(board: &mut Vec<Vec<char>>) {
        if board.is_empty() || board[0].is_empty() {
            return;
        }
        let rows = board.len();
        let cols = board[0].len();

        // 1. Mark border-connected 'O's as safe ('#').
        for r in 0..rows {
            Self::dfs(board, r as i32, 0);
            Self::dfs(board, r as i32, (cols - 1) as i32);
        }
        for c in 0..cols {
            Self::dfs(board, 0, c as i32);
            Self::dfs(board, (rows - 1) as i32, c as i32);
        }

        // 2. Flip surrounded 'O' -> 'X', restore safe '#' -> 'O'.
        for r in 0..rows {
            for c in 0..cols {
                if board[r][c] == 'O' {
                    board[r][c] = 'X';
                } else if board[r][c] == '#' {
                    board[r][c] = 'O';
                }
            }
        }
    }

    fn dfs(board: &mut Vec<Vec<char>>, r: i32, c: i32) {
        let rows = board.len() as i32;
        let cols = board[0].len() as i32;
        if r < 0 || r >= rows || c < 0 || c >= cols || board[r as usize][c as usize] != 'O' {
            return;
        }
        board[r as usize][c as usize] = '#';                 // safe marker
        let dirs = [(1i32, 0i32), (-1, 0), (0, 1), (0, -1)];
        for &(dr, dc) in &dirs {
            Self::dfs(board, r + dr, c + dc);
        }
    }
}
```

**Complexity:** Time O(rows × cols), Space O(rows × cols) recursion depth.

---

## 6. Pacific Atlantic Water Flow — Medium (LC417)

**Idea:** Instead of asking which cells flow to *both* oceans (expensive per-cell), reverse the flow: DFS **inland from each ocean's border**, climbing only to cells of **equal-or-greater** height. Cells reachable from the Pacific border AND the Atlantic border are the answer.

```rust
struct Solution;

impl Solution {
    pub fn pacific_atlantic(heights: Vec<Vec<i32>>) -> Vec<Vec<i32>> {
        let mut result = Vec::new();
        if heights.is_empty() || heights[0].is_empty() {
            return result;
        }

        let rows = heights.len();
        let cols = heights[0].len();
        let mut pacific = vec![vec![false; cols]; rows];
        let mut atlantic = vec![vec![false; cols]; rows];

        // Pacific touches top row and left column.
        // Atlantic touches bottom row and right column.
        for r in 0..rows {
            Self::dfs(&heights, r as i32, 0, &mut pacific);
            Self::dfs(&heights, r as i32, (cols - 1) as i32, &mut atlantic);
        }
        for c in 0..cols {
            Self::dfs(&heights, 0, c as i32, &mut pacific);
            Self::dfs(&heights, (rows - 1) as i32, c as i32, &mut atlantic);
        }

        for r in 0..rows {
            for c in 0..cols {
                if pacific[r][c] && atlantic[r][c] {
                    result.push(vec![r as i32, c as i32]);
                }
            }
        }
        result
    }

    fn dfs(heights: &[Vec<i32>], r: i32, c: i32, ocean: &mut Vec<Vec<bool>>) {
        let rows = heights.len() as i32;
        let cols = heights[0].len() as i32;
        ocean[r as usize][c as usize] = true;
        let dirs = [(1i32, 0i32), (-1, 0), (0, 1), (0, -1)];
        for &(dr, dc) in &dirs {
            let nr = r + dr;
            let nc = c + dc;
            if nr < 0 || nr >= rows || nc < 0 || nc >= cols {
                continue;
            }
            // climb only to equal-or-higher cells (reverse of water flow)
            if !ocean[nr as usize][nc as usize]
                && heights[nr as usize][nc as usize] >= heights[r as usize][c as usize]
            {
                Self::dfs(heights, nr, nc, ocean);
            }
        }
    }
}
```

**Complexity:** Time O(rows × cols), Space O(rows × cols) for the two visited matrices.

---

## 7. Is Graph Bipartite — Medium (LC785)

**Idea:** A graph is bipartite iff it is 2-colorable. Use `Vec<i32> color` initialized to `-1` (uncolored). For each uncolored node, BFS and color neighbors the opposite color. If we ever find a neighbor already colored the **same** as the current node, it's not bipartite.

```rust
use std::collections::VecDeque;

struct Solution;

impl Solution {
    pub fn is_bipartite(graph: Vec<Vec<i32>>) -> bool {
        let n = graph.len();
        let mut color = vec![-1i32; n];              // -1 = uncolored

        for start in 0..n {
            if color[start] != -1 {
                continue;     // already handled component
            }

            let mut q: VecDeque<usize> = VecDeque::new();
            q.push_back(start);
            color[start] = 0;

            while let Some(node) = q.pop_front() {
                for &next in &graph[node] {
                    let next = next as usize;
                    if color[next] == -1 {
                        color[next] = 1 - color[node];   // opposite color
                        q.push_back(next);
                    } else if color[next] == color[node] {
                        return false;                    // conflict
                    }
                }
            }
        }
        true
    }
}
```

**Complexity:** Time O(V + E), Space O(V).

---

## 8. Word Ladder — Hard (LC127)

**Idea:** Each word is a node; two words are connected if they differ by exactly one letter. We want the **shortest transformation length** → BFS. To find neighbors efficiently, pre-build buckets keyed by wildcard patterns (e.g., `h*t`); all words sharing a pattern are one-edit neighbors.

```rust
use std::collections::{HashMap, HashSet, VecDeque};

struct Solution;

impl Solution {
    pub fn ladder_length(begin_word: String, end_word: String, word_list: Vec<String>) -> i32 {
        let dict: HashSet<String> = word_list.into_iter().collect();
        if !dict.contains(&end_word) {
            return 0;
        }

        let l = begin_word.len();

        // Build wildcard buckets: "h*t" -> [hot, hat, ...]
        let mut patterns: HashMap<String, Vec<String>> = HashMap::new();
        for word in &dict {
            for i in 0..l {
                let key = format!("{}*{}", &word[..i], &word[i+1..]);
                patterns.entry(key).or_default().push(word.clone());
            }
        }

        let mut q: VecDeque<String> = VecDeque::new();
        let mut visited: HashSet<String> = HashSet::new();
        q.push_back(begin_word.clone());
        visited.insert(begin_word.clone());
        let mut level = 1;                        // beginWord counts as the first word

        while !q.is_empty() {
            let size = q.len();
            for _ in 0..size {
                let word = q.pop_front().unwrap();
                if word == end_word {
                    return level;
                }

                for j in 0..l {
                    let key = format!("{}*{}", &word[..j], &word[j+1..]);
                    if let Some(neighbors) = patterns.get(&key).cloned() {
                        for neighbor in neighbors {
                            if !visited.contains(&neighbor) {
                                visited.insert(neighbor.clone());
                                q.push_back(neighbor);
                            }
                        }
                    }
                }
            }
            level += 1;
        }
        0
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

```rust
use std::collections::{HashMap, HashSet};

struct Solution;

impl Solution {
    pub fn find_ladders(
        begin_word: String,
        end_word: String,
        word_list: Vec<String>,
    ) -> Vec<Vec<String>> {
        let mut result: Vec<Vec<String>> = Vec::new();
        let mut dict: HashSet<String> = word_list.into_iter().collect();
        if !dict.contains(&end_word) {
            return result;
        }

        let l = begin_word.len();

        // parents[w] = words that reached w on the shortest path
        let mut parents: HashMap<String, Vec<String>> = HashMap::new();
        let mut current: HashSet<String> = HashSet::new();
        current.insert(begin_word.clone());
        let mut visited: HashSet<String> = HashSet::new();
        visited.insert(begin_word.clone());
        let mut found = false;

        while !current.is_empty() && !found {
            // Remove this level's words from dict so we don't revisit earlier levels.
            for w in &current {
                dict.remove(w);
            }
            let mut next: HashSet<String> = HashSet::new();

            for word in &current {
                for i in 0..l {
                    let old = word.as_bytes()[i] as char;
                    for c in 'a'..='z' {
                        if c == old {
                            continue;
                        }
                        let cand = format!("{}{}{}", &word[..i], c, &word[i+1..]);
                        if dict.contains(&cand) {
                            if !visited.contains(&cand) {      // first time at this BFS depth
                                visited.insert(cand.clone());
                                next.insert(cand.clone());
                            }
                            // record predecessor regardless (same level reach)
                            if next.contains(&cand) || cand == end_word {
                                parents.entry(cand.clone()).or_default().push(word.clone());
                            }
                            if cand == end_word {
                                found = true;
                            }
                        }
                    }
                }
            }
            current = next;
        }

        if found {
            let mut path = vec![end_word.clone()];
            Self::backtrack(&end_word, &begin_word, &parents, &mut path, &mut result);
        }
        result
    }

    fn backtrack(
        word: &str,
        begin_word: &str,
        parents: &HashMap<String, Vec<String>>,
        path: &mut Vec<String>,
        result: &mut Vec<Vec<String>>,
    ) {
        if word == begin_word {
            let mut reversed = path.clone();
            reversed.reverse();
            result.push(reversed);
            return;
        }
        if let Some(preds) = parents.get(word).cloned() {
            for parent in preds {
                path.push(parent.clone());
                Self::backtrack(&parent, begin_word, parents, path, result);
                path.pop();
            }
        }
    }
}
```

**Complexity:** Time O(N × L × 26 + P) where P is the cost of enumerating all shortest paths; Space O(N × L) for the parent map plus output.

---

## 10. Open the Lock — Medium (LC752)

**Idea:** Each 4-digit state is a node; turning one wheel one notch (up or down, with wraparound 0↔9) gives 8 neighbors. We want the **minimum number of moves** from `"0000"` to `target` → BFS. Treat each deadend as already-visited so BFS never expands it.

```rust
use std::collections::{HashSet, VecDeque};

struct Solution;

impl Solution {
    pub fn open_lock(deadends: Vec<String>, target: String) -> i32 {
        let dead: HashSet<String> = deadends.into_iter().collect();
        if dead.contains("0000") {
            return -1;
        }
        if target == "0000" {
            return 0;
        }

        let mut visited: HashSet<String> = dead.clone();   // deadends count as visited
        let mut q: VecDeque<String> = VecDeque::new();
        q.push_back("0000".to_string());
        visited.insert("0000".to_string());
        let mut moves = 0;

        while !q.is_empty() {
            let size = q.len();
            for _ in 0..size {
                let state = q.pop_front().unwrap();
                if state == target {
                    return moves;
                }

                for next in Self::neighbors(&state) {
                    if !visited.contains(&next) {
                        visited.insert(next.clone());
                        q.push_back(next);
                    }
                }
            }
            moves += 1;
        }
        -1
    }

    fn neighbors(state: &str) -> Vec<String> {
        let mut res = Vec::new();
        let bytes = state.as_bytes();
        for i in 0..4 {
            let old = bytes[i];

            let up = if old == b'9' { b'0' } else { old + 1 };   // turn up
            let mut s = bytes.to_vec();
            s[i] = up;
            res.push(String::from_utf8(s).unwrap());

            let down = if old == b'0' { b'9' } else { old - 1 };   // turn down
            let mut s = bytes.to_vec();
            s[i] = down;
            res.push(String::from_utf8(s).unwrap());
        }
        res
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
