> **Topic:** [Graphs](../README.md) · **Pattern 5 of 7**

# Minimum Spanning Tree

A **Minimum Spanning Tree (MST)** of a connected, undirected, weighted graph is a subset of edges that connects **all vertices** together, with **no cycles**, and with the **minimum possible total edge weight**.

Key properties:
- A spanning tree of a graph with `V` vertices always has exactly `V - 1` edges.
- If the graph is disconnected, no spanning tree exists (you instead get a *minimum spanning forest*).
- Edge weights need not be distinct, but if they are, the MST is unique.

There are two classic greedy algorithms. Both are correct because of the **cut property**: for any partition of the vertices into two sets, the minimum-weight edge crossing the partition is part of some MST.

---

## Prim's vs Kruskal's

| | Prim's | Kruskal's |
|---|---|---|
| **Strategy** | Grow a single tree outward from a starting vertex | Build a forest by adding globally cheapest edges |
| **Data structure** | `BinaryHeap` (min-heap of candidate edges) | Sorted edge list + **Union-Find (DSU)** |
| **Cycle handling** | Track visited vertices; skip a popped edge if its endpoint is already in the tree | Skip an edge if both endpoints are already in the same DSU component |
| **Best for** | **Dense** graphs (e.g. complete graphs); when edges are given implicitly | **Sparse** graphs; when edges are given as an explicit list |
| **Time** | `O(E log V)` with a binary heap | `O(E log E)` dominated by the sort |
| **Space** | `O(V + E)` | `O(V + E)` |

**Prim's** picks a start vertex, pushes its outgoing edges into a min-heap, then repeatedly pops the cheapest edge that reaches a *new* vertex, adds that vertex to the tree, and pushes its outgoing edges. It finishes when all vertices are in the tree.

**Kruskal's** sorts every edge by weight ascending, then walks the list: for each edge it uses DSU `find` to check whether the two endpoints are already connected. If they are, adding the edge would create a cycle, so it is skipped. Otherwise it `union`s them and counts the edge into the MST. It finishes after `V - 1` edges are accepted.

---

## Prim's — Rust Template

```rust
use std::cmp::Reverse;
use std::collections::BinaryHeap;

struct PrimMST;

impl PrimMST {
    // adj[u] = list of (v, weight); vertices are 0..n-1
    fn prim(n: usize, adj: &[Vec<(usize, i32)>]) -> i32 {
        let mut in_tree = vec![false; n];
        // min-heap ordered by edge weight: each entry is (weight, vertex)
        let mut pq: BinaryHeap<Reverse<(i32, usize)>> = BinaryHeap::new();

        pq.push(Reverse((0, 0))); // start from vertex 0 with cost 0
        let mut total_weight = 0i32;
        let mut vertices_used = 0;

        while let Some(Reverse((w, u))) = pq.pop() {
            if vertices_used >= n { break; }
            if in_tree[u] { continue; } // already connected — stale heap entry
            in_tree[u] = true;
            total_weight += w;
            vertices_used += 1;

            for &(v, weight) in &adj[u] {
                if !in_tree[v] {
                    pq.push(Reverse((weight, v)));
                }
            }
        }

        // If vertices_used < n the graph was disconnected.
        if vertices_used == n { total_weight } else { -1 }
    }
}
```

---

## Kruskal's — Rust Template (with inline DSU)

The DSU below uses **path compression** in `find` and **union by rank** in `unite`. For a deeper treatment of the structure itself see [Union Find (DSU)](./Union%20Find%20(DSU).md).

```rust
// Inline Disjoint Set Union — path compression + union by rank
struct DSU {
    parent: Vec<usize>,
    rank: Vec<usize>,
}

impl DSU {
    fn new(n: usize) -> Self {
        DSU {
            parent: (0..n).collect(),
            rank: vec![0; n],
        }
    }

    fn find(&mut self, x: usize) -> usize {
        if self.parent[x] != x {
            let px = self.parent[x];
            self.parent[x] = self.find(px); // path compression
        }
        self.parent[x]
    }

    // returns true if the two were in different sets (a real union happened)
    fn unite(&mut self, a: usize, b: usize) -> bool {
        let ra = self.find(a);
        let rb = self.find(b);
        if ra == rb { return false; }
        let (ra, rb) = if self.rank[ra] < self.rank[rb] { (rb, ra) } else { (ra, rb) };
        self.parent[rb] = ra;
        if self.rank[ra] == self.rank[rb] { self.rank[ra] += 1; }
        true
    }
}

struct KruskalMST;

impl KruskalMST {
    // edges[i] = (u, v, weight)
    fn kruskal(n: usize, edges: &mut Vec<(usize, usize, i32)>) -> i32 {
        edges.sort_by_key(|&(_, _, w)| w);

        let mut dsu = DSU::new(n);
        let mut total_weight = 0i32;
        let mut edges_used = 0;

        for &(u, v, w) in edges.iter() {
            if dsu.unite(u, v) { // unite returns false if it would form a cycle
                total_weight += w;
                edges_used += 1;
                if edges_used == n - 1 { break; } // tree complete
            }
        }

        if edges_used == n - 1 { total_weight } else { -1 } // -1 if disconnected
    }
}
```

---

## Recognition Signals

| Signal in the problem | What it suggests |
|---|---|
| "Connect all points / cities / nodes at minimum total cost" | MST |
| "Every node must be reachable" + minimize sum of chosen edge weights | MST (not shortest path) |
| Weighted **undirected** graph, choose a subset of edges | MST |
| Complete graph / coordinates with implicit edge weights (Manhattan, Euclidean) | MST, prefer **Prim's** (dense) |
| Explicit edge list, possibly sparse | MST, prefer **Kruskal's** |
| "Each node has a build cost OR a connection cost" | Add a **virtual node**, then MST |
| "Return -1 if it cannot all be connected" | MST with a connectivity check |
| "Which edges are essential / which are interchangeable" | MST + critical/pseudo-critical analysis |

Note: if the goal were "minimize distance from a source to every node," that is **shortest path (Dijkstra)**, not MST. MST minimizes the *total* edge weight of the connecting tree, not path lengths.

---

## 34. Min Cost to Connect All Points — Medium (LC1584)

**Idea:** The points form a complete graph where the weight between two points is their Manhattan distance; this dense graph is a perfect fit for **Prim's** (no need to materialize all `O(n^2)` edges into a sorted list).

```rust
use std::cmp::Reverse;
use std::collections::BinaryHeap;

struct Solution;

impl Solution {
    pub fn min_cost_connect_points(points: Vec<Vec<i32>>) -> i32 {
        let n = points.len();
        if n <= 1 { return 0; }

        let mut in_tree = vec![false; n];
        // min-heap entries: (cost, point_index)
        let mut pq: BinaryHeap<Reverse<(i32, usize)>> = BinaryHeap::new();

        pq.push(Reverse((0, 0))); // start at point 0
        let mut total = 0i32;
        let mut used = 0;

        while let Some(Reverse((cost, u))) = pq.pop() {
            if used >= n { break; }
            if in_tree[u] { continue; }

            in_tree[u] = true;
            total += cost;
            used += 1;

            // push edges to every not-yet-connected point (implicit complete graph)
            for v in 0..n {
                if !in_tree[v] {
                    let dist = (points[u][0] - points[v][0]).abs()
                             + (points[u][1] - points[v][1]).abs();
                    pq.push(Reverse((dist, v)));
                }
            }
        }

        total
    }
}
```

**Complexity:** Time `O(n^2 log n)` (each of `n` expansions pushes up to `n` heap entries). Space `O(n^2)` for the heap in the worst case.

**Kruskal alternative:** Build all `n(n-1)/2` edges as `(i, j, manhattanDist)`, sort them, and run the Kruskal template above. This is `O(n^2 log n)` for the sort and works fine, but it allocates the full edge list, so Prim's is the cleaner primary choice for this dense graph.

**Dry run (Kruskal) on `points = [[0,0],[2,2],[3,10],[5,2],[7,0]]`** (label points P0..P4):

First, the relevant Manhattan distances (`|dx| + |dy|`):

| Edge | Distance |
|---|---|
| P0–P1 | \|0-2\|+\|0-2\| = 4 |
| P0–P3 | \|0-5\|+\|0-2\| = 7 |
| P0–P4 | \|0-7\|+\|0-0\| = 7 |
| P1–P3 | \|2-5\|+\|2-2\| = 3 |
| P1–P2 | \|2-3\|+\|2-10\| = 9 |
| P3–P4 | \|5-7\|+\|2-0\| = 4 |
| P2–P3 | \|3-5\|+\|10-2\| = 10 |
| P2–P4 | \|3-7\|+\|10-0\| = 14 |
| P0–P2 | \|0-3\|+\|0-10\| = 13 |
| P1–P4 | \|2-7\|+\|2-0\| = 7 |

Sort ascending and process (DSU starts as 5 singletons `{P0}{P1}{P2}{P3}{P4}`):

1. **P1–P3 (3):** different sets → **accept**. union → `{P1,P3}`. total = 3, edges = 1.
2. **P0–P1 (4):** P0 and P1 in different sets → **accept**. union → `{P0,P1,P3}`. total = 7, edges = 2.
3. **P3–P4 (4):** P3 in `{P0,P1,P3}`, P4 alone → **accept**. union → `{P0,P1,P3,P4}`. total = 11, edges = 3.
4. **P0–P3 (7):** both already in `{P0,P1,P3,P4}` → **cycle, skip**.
5. **P0–P4 (7):** both in same set → **cycle, skip**.
6. **P1–P4 (7):** both in same set → **cycle, skip**.
7. **P1–P2 (9):** P1 in big set, P2 alone → **accept**. union → all 5 connected. total = 20, edges = 4.

We have accepted `V - 1 = 4` edges, so we stop. **MST cost = 20**, which matches the expected answer for this input.

---

## 35. Connecting Cities With Minimum Cost — Medium (LC1135)

**Idea:** Classic **Kruskal's** on the explicit connection list; after processing, return the total only if all `n` cities ended up in one component, else `-1`.

```rust
struct DSU {
    parent: Vec<usize>,
    rank: Vec<usize>,
}

impl DSU {
    fn new(n: usize) -> Self {
        DSU {
            parent: (0..n).collect(),
            rank: vec![0; n],
        }
    }

    fn find(&mut self, x: usize) -> usize {
        if self.parent[x] != x {
            let px = self.parent[x];
            self.parent[x] = self.find(px);
        }
        self.parent[x]
    }

    fn unite(&mut self, a: usize, b: usize) -> bool {
        let ra = self.find(a);
        let rb = self.find(b);
        if ra == rb { return false; }
        let (ra, rb) = if self.rank[ra] < self.rank[rb] { (rb, ra) } else { (ra, rb) };
        self.parent[rb] = ra;
        if self.rank[ra] == self.rank[rb] { self.rank[ra] += 1; }
        true
    }
}

struct Solution;

impl Solution {
    pub fn minimum_cost(n: i32, connections: Vec<Vec<i32>>) -> i32 {
        let n = n as usize;
        // cities are 1..n — DSU sized n+1, index 0 unused
        let mut connections = connections;
        connections.sort_by_key(|c| c[2]);

        let mut dsu = DSU::new(n + 1);
        let mut total = 0i32;
        let mut edges_used = 0;

        for c in &connections {
            let u = c[0] as usize;
            let v = c[1] as usize;
            let cost = c[2];
            if dsu.unite(u, v) {
                total += cost;
                edges_used += 1;
                if edges_used == n - 1 { break; }
            }
        }

        if edges_used == n - 1 { total } else { -1 }
    }
}
```

**Complexity:** Time `O(E log E)` for the sort (E = number of connections), with near-`O(1)` amortized DSU operations. Space `O(n)`.

---

## 36. Optimize Water Distribution in a Village — Hard (LC1168)

**Idea:** Each house can either dig its own well (a cost) or connect to a neighbor via a pipe. Model the well option by adding a **virtual node 0** with an edge of weight `wells[i]` to each house `i`; then the answer is simply the **MST** over houses `1..n` plus node `0` (Kruskal's).

```rust
struct DSU {
    parent: Vec<usize>,
    rank: Vec<usize>,
}

impl DSU {
    fn new(n: usize) -> Self {
        DSU {
            parent: (0..n).collect(),
            rank: vec![0; n],
        }
    }

    fn find(&mut self, x: usize) -> usize {
        if self.parent[x] != x {
            let px = self.parent[x];
            self.parent[x] = self.find(px);
        }
        self.parent[x]
    }

    fn unite(&mut self, a: usize, b: usize) -> bool {
        let ra = self.find(a);
        let rb = self.find(b);
        if ra == rb { return false; }
        let (ra, rb) = if self.rank[ra] < self.rank[rb] { (rb, ra) } else { (ra, rb) };
        self.parent[rb] = ra;
        if self.rank[ra] == self.rank[rb] { self.rank[ra] += 1; }
        true
    }
}

struct Solution;

impl Solution {
    pub fn min_cost_to_supply_water(n: i32, wells: Vec<i32>, pipes: Vec<Vec<i32>>) -> i32 {
        let n = n as usize;
        // Build edge list. Houses are 1..n; node 0 is the virtual "water source".
        let mut edges: Vec<(usize, usize, i32)> = Vec::new();

        // well[i] becomes an edge from virtual node 0 to house (i+1)
        for i in 0..n {
            edges.push((0, i + 1, wells[i]));
        }
        for p in &pipes {
            edges.push((p[0] as usize, p[1] as usize, p[2]));
        }

        edges.sort_by_key(|&(_, _, w)| w);

        let mut dsu = DSU::new(n + 1); // indices 0..n
        let mut total = 0i32;
        let mut edges_used = 0;

        for &(u, v, w) in &edges {
            if dsu.unite(u, v) {
                total += w;
                edges_used += 1;
                if edges_used == n { break; } // n+1 nodes => MST has n edges
            }
        }

        total
    }
}
```

**Why the virtual node works:** "Build a well at house `i`" is equivalent to "connect house `i` to a single shared source." Adding node `0` makes every house's well a real edge, so the cheapest way to supply water to every house is exactly the minimum spanning tree of the `n + 1` nodes.

**Complexity:** Time `O((n + P) log (n + P))` where `P = pipes.length`, dominated by the sort. Space `O(n + P)`.

---

## 37. Find Critical and Pseudo-Critical Edges in MST — Hard (LC1489)

**Idea:** Compute the **base MST weight** with Kruskal's. For each edge: it is **critical** if removing it (running Kruskal without it) raises the MST weight or disconnects the graph; it is **pseudo-critical** if forcing it in first and then completing the MST still yields the base weight (but it is not critical).

```rust
struct DSU {
    parent: Vec<usize>,
    rank: Vec<usize>,
}

impl DSU {
    fn new(n: usize) -> Self {
        DSU {
            parent: (0..n).collect(),
            rank: vec![0; n],
        }
    }

    fn find(&mut self, x: usize) -> usize {
        if self.parent[x] != x {
            let px = self.parent[x];
            self.parent[x] = self.find(px);
        }
        self.parent[x]
    }

    fn unite(&mut self, a: usize, b: usize) -> bool {
        let ra = self.find(a);
        let rb = self.find(b);
        if ra == rb { return false; }
        let (ra, rb) = if self.rank[ra] < self.rank[rb] { (rb, ra) } else { (ra, rb) };
        self.parent[rb] = ra;
        if self.rank[ra] == self.rank[rb] { self.rank[ra] += 1; }
        true
    }
}

struct Solution;

impl Solution {
    // Runs Kruskal on the pre-sorted edges.
    // skip: index (in sorted array) to exclude, or None.
    // force: index (in sorted array) to add first, or None.
    // Returns total weight, or i32::MAX if the graph stays disconnected.
    fn build_mst(
        n: usize,
        sorted: &[(usize, usize, i32, usize)],
        skip: Option<usize>,
        force: Option<usize>,
    ) -> i32 {
        let mut dsu = DSU::new(n);
        let mut weight = 0i32;
        let mut count = 0;

        if let Some(f) = force {
            dsu.unite(sorted[f].0, sorted[f].1);
            weight += sorted[f].2;
            count += 1;
        }

        for i in 0..sorted.len() {
            if Some(i) == skip { continue; }
            if dsu.unite(sorted[i].0, sorted[i].1) {
                weight += sorted[i].2;
                count += 1;
            }
        }

        if count == n - 1 { weight } else { i32::MAX } // disconnected
    }

    pub fn find_critical_and_pseudo_critical_edges(n: i32, edges: Vec<Vec<i32>>) -> Vec<Vec<i32>> {
        let n = n as usize;
        let m = edges.len();
        // Track original indices since we will sort.
        let mut indexed: Vec<(usize, usize, i32, usize)> = edges
            .iter()
            .enumerate()
            .map(|(i, e)| (e[0] as usize, e[1] as usize, e[2], i))
            .collect();
        indexed.sort_by_key(|&(_, _, w, _)| w);

        let base_weight = Self::build_mst(n, &indexed, None, None);

        let mut critical: Vec<i32> = Vec::new();
        let mut pseudo: Vec<i32> = Vec::new();

        for i in 0..m {
            // 1) Exclude edge i: if MST weight grows or graph disconnects -> critical.
            let without = Self::build_mst(n, &indexed, Some(i), None);
            if without > base_weight {
                critical.push(indexed[i].3 as i32);
                continue;
            }
            // 2) Force edge i in: if total still equals base -> pseudo-critical.
            let with_forced = Self::build_mst(n, &indexed, None, Some(i));
            if with_forced == base_weight {
                pseudo.push(indexed[i].3 as i32);
            }
        }

        vec![critical, pseudo]
    }
}
```

**Reasoning recap:**
- **Critical edge:** every MST must include it. Detected because excluding it forces a costlier alternative (weight increases) or makes the graph disconnected (`i32::MAX > base_weight`).
- **Pseudo-critical edge:** appears in *some* MST but not all. Detected because forcing it in still reaches the base weight, yet it was not flagged critical.
- An edge is neither if forcing it in raises the total above the base weight.

**Complexity:** Time `O(E^2 · α(V))` — for each of `E` edges we run a near-linear Kruskal pass (DSU operations are amortized `α(V)`, effectively constant). Space `O(E + V)`.

---

> **Last Updated:** 2026-06-26
