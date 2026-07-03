> **Topic:** [Graphs](../README.md) · **Pattern 7 of 7**

# Cycle Detection

A **cycle** is a path that starts and ends at the same vertex while traversing at least one edge, without reusing any edge. Detecting cycles is a foundational graph operation: it tells us whether a graph is a tree/forest (acyclic), whether a dependency graph can be scheduled (a DAG), and whether a network has redundant connections.

The technique you reach for depends entirely on whether the graph is **undirected** or **directed** — the two cases have different traps and different toolkits.

---

## 1. Core Concept

| Graph type | Cycle means | Primary tools |
| --- | --- | --- |
| **Undirected** | An edge connects two vertices already in the same connected component | **DSU (Union-Find)** or **DFS with parent tracking** |
| **Directed** | A vertex can reach itself by following edge directions (a back edge to an ancestor on the recursion stack) | **3-color DFS** or **Kahn's algorithm (BFS topological sort)** |

Key intuitions:

- **Undirected, DSU view:** Each undirected edge `(u, v)` tries to *union* two components. If `u` and `v` already share a root before the union, adding the edge closes a loop — that's a cycle.
- **Undirected, DFS view:** During DFS, if we reach an already-visited vertex that is **not** the parent we came from, we found a back edge → cycle. (We must ignore the immediate parent, otherwise the edge we just traversed would falsely look like a cycle.)
- **Directed, 3-color view:** A cycle exists iff DFS encounters a vertex currently **on the recursion stack** (gray). Reaching a fully-finished (black) vertex is fine — that's just a cross/forward edge, not a cycle.
- **Directed, Kahn view:** Repeatedly remove vertices with indegree 0. If you can't remove all `n` vertices (processed count `< n`), the remaining vertices form one or more cycles.

> **Trap:** DSU works only for *undirected* cycle detection. For directed graphs you need ordering-aware methods (3-color DFS or Kahn), because direction matters — `a→b` and `a→c→b` do not form a cycle even though `b` is reached twice.

---

## 2. Undirected Cycle Detection

### 2a. Via DSU (Union-Find)

For each edge, if both endpoints already share a root, we've found a cycle. Otherwise union them.

```rust
struct DSU {
    parent: Vec<usize>,
    rank_: Vec<usize>,
}

impl DSU {
    fn new(n: usize) -> Self {
        DSU {
            parent: (0..n).collect(),
            rank_: vec![0; n],
        }
    }

    fn find(&mut self, mut x: usize) -> usize { // path halving
        while self.parent[x] != x {
            let pp = self.parent[self.parent[x]];
            self.parent[x] = pp;
            x = self.parent[x];
        }
        x
    }

    // returns false if x,y already connected (=> cycle); true if merged
    fn unite(&mut self, x: usize, y: usize) -> bool {
        let rx = self.find(x);
        let ry = self.find(y);
        if rx == ry { return false; }
        if self.rank_[rx] < self.rank_[ry]      { self.parent[rx] = ry; }
        else if self.rank_[rx] > self.rank_[ry] { self.parent[ry] = rx; }
        else                                     { self.parent[ry] = rx; self.rank_[rx] += 1; }
        true
    }
}

struct UndirectedCycleDSU;

impl UndirectedCycleDSU {
    // edges given as Vec<Vec<i32>> where each row is [u, v], vertices 0..n-1
    fn has_cycle(n: usize, edges: &[Vec<i32>]) -> bool {
        let mut dsu = DSU::new(n);
        for e in edges {
            if !dsu.unite(e[0] as usize, e[1] as usize) { return true; } // both share root => cycle
        }
        false
    }
}
```

### 2b. Via DFS with Parent Tracking

Walk the graph; if you reach a visited neighbor that isn't the parent, it's a back edge → cycle. Handle disconnected graphs by starting DFS from every unvisited vertex.

```rust
struct UndirectedCycleDFS;

impl UndirectedCycleDFS {
    fn has_cycle(n: usize, adj: &[Vec<i32>]) -> bool {
        let mut visited = vec![false; n];
        for i in 0..n {
            if !visited[i] && Self::dfs(i, None, adj, &mut visited) { return true; }
        }
        false
    }

    fn dfs(node: usize, parent: Option<usize>, adj: &[Vec<i32>], visited: &mut Vec<bool>) -> bool {
        visited[node] = true;
        for &nei in &adj[node] {
            let nei = nei as usize;
            if !visited[nei] {
                if Self::dfs(nei, Some(node), adj, visited) { return true; }
            } else if Some(nei) != parent {   // visited and not the node we came from
                return true;
            }
        }
        false
    }
}
```

> **BFS variant note:** The same idea works iteratively with BFS by storing `(node, parent)` pairs in the queue and flagging a cycle when a neighbor is already visited and isn't the parent. With parallel edges, track the edge instead of the parent vertex.

---

## 3. Directed Cycle Detection

### 3a. Via 3-Color DFS

Colors: `0 = white` (unvisited), `1 = gray` (in current recursion stack), `2 = black` (fully processed). A cycle exists exactly when DFS hits a **gray** node.

```rust
struct DirectedCycle3Color;

impl DirectedCycle3Color {
    const WHITE: i32 = 0;
    const GRAY: i32 = 1;
    const BLACK: i32 = 2;

    fn has_cycle(n: usize, adj: &[Vec<i32>]) -> bool {
        let mut color = vec![0i32; n]; // all WHITE initially
        for i in 0..n {
            if color[i] == Self::WHITE && Self::dfs(i, adj, &mut color) { return true; }
        }
        false
    }

    fn dfs(node: usize, adj: &[Vec<i32>], color: &mut Vec<i32>) -> bool {
        color[node] = Self::GRAY;               // entering recursion stack
        for &nei in &adj[node] {
            let nei = nei as usize;
            if color[nei] == Self::GRAY { return true; }          // back edge => cycle
            if color[nei] == Self::WHITE && Self::dfs(nei, adj, color) { return true; }
            // color[nei] == BLACK => already done, safe to skip
        }
        color[node] = Self::BLACK;              // leaving recursion stack
        false
    }
}
```

### 3b. Via Kahn's Algorithm (BFS Topological Sort)

Peel off indegree-0 vertices. If the number processed is less than `n`, the leftovers are stuck in a cycle.

```rust
use std::collections::VecDeque;

struct DirectedCycleKahn;

impl DirectedCycleKahn {
    fn has_cycle(n: usize, adj: &[Vec<i32>]) -> bool {
        let mut indeg = vec![0i32; n];
        for u in 0..n {
            for &v in &adj[u] { indeg[v as usize] += 1; }
        }

        let mut q: VecDeque<usize> = VecDeque::new();
        for i in 0..n {
            if indeg[i] == 0 { q.push_back(i); }
        }

        let mut processed = 0;
        while let Some(u) = q.pop_front() {
            processed += 1;
            for &v in &adj[u] {
                indeg[v as usize] -= 1;
                if indeg[v as usize] == 0 { q.push_back(v as usize); }
            }
        }
        processed < n   // couldn't topologically order everything => cycle
    }
}
```

---

## 4. Recognition Signals

| Signal in the problem | Likely approach |
| --- | --- |
| "Undirected graph, is it a tree / valid / has redundant edge?" | DSU (union; cycle when endpoints already connected) |
| "Undirected, detect any loop" | DFS with parent tracking |
| "Directed dependencies, can all tasks finish?" | Kahn or 3-color DFS |
| "Find the edge that, if removed, makes it a tree" | DSU, return the edge that closes a cycle |
| "Directed, a node has two parents (indegree 2)" | Redundant Connection II — combine indegree check + DSU |
| "Edges whose removal disconnects the graph" | Tarjan's bridges (`disc[]`/`low[]`), not plain cycle detection |
| "Which nodes can never reach a cycle / are 'safe'?" | Reverse topological / 3-color DFS (see Topological Sort) |
| "Just check if two nodes are connected" | DSU or BFS/DFS reachability |

---

## 5. Cross-Link

See [Topological Sort](./Topological%20Sort.md) — cycle detection underpins topological ordering: **a directed graph has a valid topological order if and only if it is acyclic (a DAG).** Kahn's algorithm and 3-color DFS are literally the same machinery used in both patterns; problems like *Course Schedule* and *Find Eventual Safe States* are cycle-detection problems framed as ordering problems.

---

## 47. Detect Cycle in Undirected Graph — Medium (LC——)

Given an undirected graph with `n` vertices and an edge list, determine whether it contains a cycle.

**Approach A — DSU.** Process edges; the first edge whose endpoints already share a root proves a cycle.

```rust
struct DetectCycleUndirected {
    parent: Vec<usize>,
    rank_: Vec<usize>,
}

impl DetectCycleUndirected {
    fn new() -> Self {
        DetectCycleUndirected { parent: Vec::new(), rank_: Vec::new() }
    }

    fn find(&mut self, mut x: usize) -> usize {
        while self.parent[x] != x {
            let pp = self.parent[self.parent[x]];
            self.parent[x] = pp;
            x = self.parent[x];
        }
        x
    }

    fn unite(&mut self, x: usize, y: usize) -> bool {
        let rx = self.find(x);
        let ry = self.find(y);
        if rx == ry { return false; }            // already connected => cycle
        if self.rank_[rx] < self.rank_[ry]      { self.parent[rx] = ry; }
        else if self.rank_[rx] > self.rank_[ry] { self.parent[ry] = rx; }
        else                                     { self.parent[ry] = rx; self.rank_[rx] += 1; }
        true
    }

    fn has_cycle(&mut self, n: usize, edges: &[Vec<i32>]) -> bool {
        self.parent = (0..n).collect();
        self.rank_ = vec![0; n];
        for e in edges {
            if !self.unite(e[0] as usize, e[1] as usize) { return true; }
        }
        false
    }

    // Approach B — DFS with parent tracking (handles disconnected components)
    fn has_cycle_dfs(n: usize, adj: &[Vec<i32>]) -> bool {
        let mut visited = vec![false; n];
        for i in 0..n {
            if !visited[i] && Self::dfs(i, None, adj, &mut visited) { return true; }
        }
        false
    }

    fn dfs(node: usize, par: Option<usize>, adj: &[Vec<i32>], visited: &mut Vec<bool>) -> bool {
        visited[node] = true;
        for &nei in &adj[node] {
            let nei = nei as usize;
            if !visited[nei] {
                if Self::dfs(nei, Some(node), adj, visited) { return true; }
            } else if Some(nei) != par {
                return true;
            }
        }
        false
    }
}
```

**Complexity:** DSU runs in `O(E · α(N))`; DFS in `O(V + E)`.

---

## 48. Detect Cycle in Directed Graph — Medium (LC——)

Given a directed graph, determine whether it contains a directed cycle. Use 3-color DFS: a cycle exists iff DFS reaches a vertex currently on the recursion stack (gray).

```rust
struct DetectCycleDirected;

impl DetectCycleDirected {
    const WHITE: i32 = 0;
    const GRAY: i32 = 1;
    const BLACK: i32 = 2;

    fn has_cycle(n: usize, adj: &[Vec<i32>]) -> bool {
        let mut color = vec![0i32; n];
        for i in 0..n {
            if color[i] == Self::WHITE && Self::dfs(i, adj, &mut color) { return true; }
        }
        false
    }

    fn dfs(node: usize, adj: &[Vec<i32>], color: &mut Vec<i32>) -> bool {
        color[node] = Self::GRAY;
        for &nei in &adj[node] {
            let nei = nei as usize;
            if color[nei] == Self::GRAY { return true; }                 // back edge
            if color[nei] == Self::WHITE && Self::dfs(nei, adj, color) { return true; }
        }
        color[node] = Self::BLACK;
        false
    }
}
```

**Why parent-tracking fails here:** In a directed graph `a→b` and `b→a` is a genuine 2-cycle, so we cannot "excuse the parent." The recursion-stack (gray) check correctly distinguishes a back edge (ancestor on the stack) from a harmless cross edge (already-finished black node).

**Complexity:** `O(V + E)` time, `O(V)` space.

---

## 49. Find Eventual Safe States — Medium (LC802)

> **Cross-reference.** This problem is covered in detail in [Topological Sort](./Topological%20Sort.md). A node is *safe* if every path from it leads to a terminal node — equivalently, if it is **not** part of and cannot reach any cycle. The standard solution is a 3-color DFS (a node is safe iff its DFS subtree contains no gray/back edge) or a reverse-graph Kahn's algorithm. See Topological Sort for the full walkthrough.

Brief snippet (3-color DFS; safe ⟺ no cycle reachable):

```rust
fn safe(u: usize, g: &[Vec<i32>], color: &mut Vec<i32>) -> bool {
    if color[u] != 0 { return color[u] == 2; } // gray => unsafe, black => safe
    color[u] = 1;
    for &v in &g[u] {
        if !safe(v as usize, g, color) { return false; }
    }
    color[u] = 2;
    true
}

fn eventual_safe_nodes(graph: &[Vec<i32>]) -> Vec<i32> {
    let n = graph.len();
    let mut color = vec![0i32; n]; // 0 white, 1 gray, 2 black(safe)
    let mut res = Vec::new();
    for i in 0..n {
        if safe(i, graph, &mut color) { res.push(i as i32); }
    }
    res
}
```

See [Topological Sort](./Topological%20Sort.md) for the detailed reverse-Kahn treatment.

---

## 50. Course Schedule — Medium (LC207)

> **Cross-reference.** Covered fully in [Topological Sort](./Topological%20Sort.md). `numCourses` with prerequisite pairs `[a, b]` (take `b` before `a`) is solvable iff the dependency graph is a **DAG** — i.e., it has no directed cycle. Detect with Kahn's algorithm or 3-color DFS.

Brief snippet (Kahn's):

```rust
use std::collections::VecDeque;

fn can_finish(num_courses: usize, prerequisites: &[Vec<i32>]) -> bool {
    let mut adj = vec![Vec::<usize>::new(); num_courses];
    let mut indeg = vec![0i32; num_courses];
    for p in prerequisites {
        adj[p[1] as usize].push(p[0] as usize);
        indeg[p[0] as usize] += 1;
    }

    let mut q: VecDeque<usize> = VecDeque::new();
    for i in 0..num_courses {
        if indeg[i] == 0 { q.push_back(i); }
    }
    let mut done = 0;
    while let Some(u) = q.pop_front() {
        done += 1;
        for &v in &adj[u] {
            indeg[v] -= 1;
            if indeg[v] == 0 { q.push_back(v); }
        }
    }
    done == num_courses // all scheduled => acyclic
}
```

See [Topological Sort](./Topological%20Sort.md) for the full explanation and the DFS variant.

---

## 51. Redundant Connection II — Hard (LC685)

A rooted tree with `n` nodes is given `n` directed edges; exactly one extra edge was added. Return the edge to remove so the result is a valid rooted tree (each node except the root has exactly one parent, and there are no cycles). If multiple answers, return the last one in the input.

**The two failure modes (and they can co-occur):**
1. **A node has two parents** (indegree 2). Two edges point to the same node; one of them must be the culprit.
2. **A cycle** exists.

**Strategy:**
- Scan edges; if some node `v` has two incoming edges, record them as `cand1` (first) and `cand2` (second/later). Temporarily *invalidate* `cand2` (skip it during DSU).
- Run DSU over the remaining edges.
  - If no cycle forms → `cand2` was the redundant one (it was the only problem). Return `cand2`.
  - If a cycle still forms:
    - If there was **no** indegree-2 node, the redundant edge is simply the one that closed the cycle (pure cycle case). Return it.
    - If there **was** an indegree-2 node, then `cand1` is the offender (removing `cand2` didn't fix things, so the other parent edge `cand1` is part of the cycle). Return `cand1`.

```rust
struct RedundantConnectionII {
    parent: Vec<usize>,
}

impl RedundantConnectionII {
    fn new() -> Self {
        RedundantConnectionII { parent: Vec::new() }
    }

    fn find(&mut self, mut x: usize) -> usize {
        while self.parent[x] != x {
            let pp = self.parent[self.parent[x]];
            self.parent[x] = pp;
            x = self.parent[x];
        }
        x
    }

    fn find_redundant_directed_connection(&mut self, edges: &[Vec<i32>]) -> Vec<i32> {
        let n = edges.len();
        self.parent = (0..=n).collect();
        let mut incoming_parent = vec![0usize; n + 1]; // incoming_parent[v] = which edge index gave v its parent

        let mut cand1: Vec<i32> = Vec::new();
        let mut cand2: Vec<i32> = Vec::new(); // two edges into the same node
        for i in 0..n {
            let v = edges[i][1] as usize;
            if incoming_parent[v] != 0 {              // v already has a parent edge
                cand1 = edges[incoming_parent[v] - 1].clone(); // earlier edge into v
                cand2 = edges[i].clone();                       // current edge into v
            } else {
                incoming_parent[v] = i + 1;           // store 1-based index to keep 0 as "none"
            }
        }

        self.parent = (0..=n).collect();

        // Run union over all edges except cand2 (if it exists)
        for i in 0..n {
            if !cand2.is_empty() && edges[i] == cand2 { continue; } // skip the second parent edge
            let u = edges[i][0] as usize;
            let v = edges[i][1] as usize;
            let ru = self.find(u);
            let rv = self.find(v);
            if ru == rv {
                // a cycle formed without cand2
                if cand1.is_empty() { return edges[i].clone(); } // pure cycle, no indegree-2 node
                return cand1.clone();                             // indegree-2 case: cand1 is the bad edge
            }
            self.parent[rv] = ru;
        }
        // No cycle without cand2 => cand2 was the redundant edge
        cand2
    }
}
```

**Walking the cases:**
- **Only a cycle, no two-parent node:** `cand1`/`cand2` stay empty; the union loop finds the closing edge and returns it.
- **Only two parents, no cycle:** Skipping `cand2` leaves a clean tree; loop finishes; we return `cand2`.
- **Two parents AND a cycle:** Skipping `cand2` still leaves a cycle (because `cand1` is the one in the loop); we return `cand1`.

**Complexity:** `O(n · α(n))` time, `O(n)` space.

---

## 52. Critical Connections in a Network — Hard (LC1192)

Given `n` servers (`0..n-1`) and undirected `connections`, return all **critical connections** — edges whose removal disconnects part of the network. These are exactly the **bridges** of the graph.

**Tarjan's bridge algorithm.** Do a DFS assigning each node a discovery time `disc[u]` (when first visited) and a low-link `low[u]` (the smallest discovery time reachable from `u`'s DFS subtree, including via at most one back edge). For a tree edge `u→v`:
- After exploring `v`, set `low[u] = min(low[u], low[v])`.
- If `low[v] > disc[u]`, then `v`'s subtree has **no** back edge that reaches `u` or above → edge `(u, v)` is a **bridge**.
- For a back edge `u→v` (where `v` is already visited and isn't the parent), update `low[u] = min(low[u], disc[v])`.

We must skip the immediate parent edge — but to handle **parallel edges** correctly, skip by *edge identity*, not by parent vertex.

```rust
struct CriticalConnections {
    adj: Vec<Vec<(usize, usize)>>, // adj[u] = list of (neighbor, edgeId)
    disc: Vec<i32>,
    low: Vec<i32>,
    visited: Vec<bool>,
    timer_: i32,
    bridges: Vec<Vec<i32>>,
}

impl CriticalConnections {
    fn new() -> Self {
        CriticalConnections {
            adj: Vec::new(),
            disc: Vec::new(),
            low: Vec::new(),
            visited: Vec::new(),
            timer_: 0,
            bridges: Vec::new(),
        }
    }

    fn critical_connections(&mut self, n: usize, connections: &[Vec<i32>]) -> Vec<Vec<i32>> {
        self.adj = vec![Vec::new(); n];
        let mut id = 0usize;
        for c in connections {
            let u = c[0] as usize;
            let v = c[1] as usize;
            self.adj[u].push((v, id));
            self.adj[v].push((u, id));
            id += 1;
        }
        self.disc = vec![0; n];
        self.low = vec![0; n];
        self.visited = vec![false; n];

        for i in 0..n {
            if !self.visited[i] { self.dfs(i, usize::MAX); } // usize::MAX = no incoming edge id
        }
        self.bridges.clone()
    }

    fn dfs(&mut self, u: usize, parent_edge_id: usize) {
        self.visited[u] = true;
        self.disc[u] = self.timer_;
        self.low[u] = self.timer_;
        self.timer_ += 1;
        let neighbors = self.adj[u].clone();
        for (v, eid) in neighbors {
            if eid == parent_edge_id { continue; }     // don't go back over the same edge
            if !self.visited[v] {
                self.dfs(v, eid);
                self.low[u] = self.low[u].min(self.low[v]);
                if self.low[v] > self.disc[u] {        // bridge condition
                    self.bridges.push(vec![u as i32, v as i32]);
                }
            } else {
                self.low[u] = self.low[u].min(self.disc[v]); // back edge
            }
        }
    }
}
```

### Detailed Dry-Run

Take `n = 4`, `connections = [[0,1],[1,2],[2,0],[1,3]]`. Edge ids: `0=(0,1)`, `1=(1,2)`, `2=(2,0)`, `3=(1,3)`. Visually: nodes `0,1,2` form a triangle; node `3` hangs off node `1`. The only bridge should be `(1,3)`.

Start `dfs(0, usize::MAX)`:

| Step | Action | disc/low updates |
| --- | --- | --- |
| 1 | Enter `0`. `timer=0` | `disc[0]=low[0]=0`, `timer→1` |
| 2 | From `0`, edge to `1` (eid 0, not parent). `1` unvisited → recurse `dfs(1, 0)` | — |
| 3 | Enter `1`. | `disc[1]=low[1]=1`, `timer→2` |
| 4 | From `1`, edge to `0` (eid 0) = parentEdgeId → **skip** | — |
| 5 | From `1`, edge to `2` (eid 1). `2` unvisited → recurse `dfs(2, 1)` | — |
| 6 | Enter `2`. | `disc[2]=low[2]=2`, `timer→3` |
| 7 | From `2`, edge to `1` (eid 1) = parentEdgeId → **skip** | — |
| 8 | From `2`, edge to `0` (eid 2). `0` visited, not parent edge → **back edge** | `low[2]=min(2, disc[0]=0)=0` |
| 9 | `2` done. Return to `1`. | `low[1]=min(1, low[2]=0)=0`; check bridge `low[2]=0 > disc[1]=1`? No |
| 10 | From `1`, edge to `3` (eid 3). `3` unvisited → recurse `dfs(3, 3)` | — |
| 11 | Enter `3`. | `disc[3]=low[3]=3`, `timer→4` |
| 12 | From `3`, edge to `1` (eid 3) = parentEdgeId → **skip**. No other edges. `3` done. | — |
| 13 | Return to `1`. | `low[1]=min(0, low[3]=3)=0`; check bridge `low[3]=3 > disc[1]=1`? **Yes → add (1,3)** |
| 14 | `1` done. Return to `0`. | `low[0]=min(0, low[1]=0)=0`; check bridge `low[1]=0 > disc[0]=0`? No |
| 15 | From `0`, edge to `2` (eid 2). `2` visited, not parent edge → back edge | `low[0]=min(0, disc[2]=2)=0` (no change) |
| 16 | `0` done. | — |

Final: `disc=[0,1,2,3]`, `low=[0,0,0,3]`. Only `(1,3)` satisfied `low[v] > disc[u]`, so **the single bridge is `[1,3]`** — matching intuition, since removing any triangle edge leaves the triangle connected, but removing `(1,3)` isolates node `3`.

**Complexity:** `O(V + E)` time, `O(V + E)` space.

---

## 53. Find if Path Exists in Graph — Easy (LC1971)

Given an undirected graph with `n` vertices and edge list, determine whether a path exists between `source` and `destination`.

**Approach A — DSU.** Union every edge, then `source` and `destination` are connected iff they share a root.

```rust
use std::collections::VecDeque;

struct PathExists {
    parent: Vec<usize>,
}

impl PathExists {
    fn new() -> Self {
        PathExists { parent: Vec::new() }
    }

    fn find(&mut self, mut x: usize) -> usize {
        while self.parent[x] != x {
            let pp = self.parent[self.parent[x]];
            self.parent[x] = pp;
            x = self.parent[x];
        }
        x
    }

    fn unite(&mut self, x: usize, y: usize) {
        let fx = self.find(x);
        let fy = self.find(y);
        self.parent[fx] = fy;
    }

    fn valid_path(&mut self, n: usize, edges: &[Vec<i32>], source: usize, destination: usize) -> bool {
        self.parent = (0..n).collect();
        for e in edges { self.unite(e[0] as usize, e[1] as usize); }
        self.find(source) == self.find(destination)
    }

    // Approach B — BFS reachability
    fn valid_path_bfs(n: usize, edges: &[Vec<i32>], source: usize, destination: usize) -> bool {
        let mut adj = vec![Vec::<usize>::new(); n];
        for e in edges {
            adj[e[0] as usize].push(e[1] as usize);
            adj[e[1] as usize].push(e[0] as usize);
        }

        let mut visited = vec![false; n];
        let mut q: VecDeque<usize> = VecDeque::new();
        q.push_back(source);
        visited[source] = true;
        while let Some(u) = q.pop_front() {
            if u == destination { return true; }
            for &v in &adj[u] {
                if !visited[v] { visited[v] = true; q.push_back(v); }
            }
        }
        false
    }
}
```

**Complexity:** DSU `O(V + E·α(V))`; BFS `O(V + E)`.

---

> **Last Updated:** 2026-06-26
