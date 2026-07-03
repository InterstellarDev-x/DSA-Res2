> **Topic:** [Graphs](../README.md) · **Section:** Interview Tips · **Tips 1 of 4**

# Graphs — Coding Tips

Ten battle-tested idioms that make graph code shorter, faster, and correct. See also [Common Mistakes](./Common%20Mistakes.md), [Algorithm Selection](./Algorithm%20Selection.md), [Complexity Analysis](./Complexity%20Analysis.md).

---

## 1. Standard adjacency-list build idiom

For a weighted graph use `HashMap<i32, Vec<(i32, i32)>>`; for unweighted, `Vec<Vec<i32>>`. Build once, reuse everywhere.

```rust
use std::collections::HashMap;

let mut adj: Vec<Vec<i32>> = vec![vec![]; n];
for e in &edges {
    adj[e[0] as usize].push(e[1]);
    adj[e[1] as usize].push(e[0]);          // BOTH directions for undirected
}
// weighted:
let mut wadj: HashMap<i32, Vec<(i32, i32)>> = HashMap::new();
for e in &edges {
    wadj.entry(e[0]).or_default().push((e[1], e[2]));
}
```

## 2. Use a direction-tuple array `[(i32, i32)]` for grid neighbors

Replaces four near-identical blocks with one loop. Use 8-dir only when diagonals count.

```rust
let dirs: [(i32, i32); 4] = [(1,0),(-1,0),(0,1),(0,-1)];            // 4-directional
let dirs8: [(i32, i32); 8] = [(1,0),(-1,0),(0,1),(0,-1),(1,1),(1,-1),(-1,1),(-1,-1)]; // 8-dir
for &(dr, dc) in &dirs {
    let r = row as i32 + dr;
    let c = col as i32 + dc;
    if r < 0 || r >= m as i32 || c < 0 || c >= n as i32 { continue; }  // bounds FIRST
    let (r, c) = (r as usize, c as usize);
    // ... use grid[r][c]
}
```

## 3. Mark visited BEFORE enqueue, never after dequeue

The single most important BFS rule. Marking at dequeue lets the same node be pushed by multiple neighbors → duplicates, bloated queue, sometimes wrong counts.

```rust
use std::collections::VecDeque;

visited[start] = true;
let mut q: VecDeque<usize> = VecDeque::new();
q.push_back(start);
while let Some(u) = q.pop_front() {
    for &v in &adj[u] {
        if !visited[v] {
            visited[v] = true;          // mark at the moment you enqueue
            q.push_back(v);
        }
    }
}
```

## 4. Multi-source BFS — seed all sources first

For "spread from many origins simultaneously" (Rotting Oranges, 01 Matrix, Walls and Gates), push every source at time 0, then expand level by level.

```rust
use std::collections::VecDeque;

let mut q: VecDeque<(usize, usize)> = VecDeque::new();
for i in 0..m {
    for j in 0..n {
        if is_source(grid[i][j]) { q.push_back((i, j)); }
    }
}
let mut level = 0;
while !q.is_empty() {
    let size = q.len();            // freeze the layer
    for _ in 0..size { /* expand one cell */ }
    level += 1;
}
```

## 5. Dijkstra — stale-entry skip + direct comparison

Lazy-deletion heap: re-push on relax, skip popped entries that are worse than the recorded best. Always use a proper comparator, never `a[1] - b[1]`.

```rust
use std::collections::BinaryHeap;
use std::cmp::Reverse;

let mut pq: BinaryHeap<Reverse<(i32, usize)>> = BinaryHeap::new();
pq.push(Reverse((0, src)));
while let Some(Reverse((d, u))) = pq.pop() {
    if d > dist[u] { continue; }          // stale — already finalized cheaper
    for &(v, w) in &adj[u] {
        let nd = d + w;
        if nd < dist[v] { dist[v] = nd; pq.push(Reverse((nd, v))); }
    }
}
```

## 6. 0-1 BFS with a `VecDeque`

When edge weights are only 0 or 1, replace Dijkstra's heap with a `VecDeque`: push 0-weight moves to the **front**, 1-weight to the **back**. O(V+E).

```rust
use std::collections::VecDeque;

let mut dq: VecDeque<(usize, i32)> = VecDeque::new();
dq.push_front((src, 0));
while let Some((u, d)) = dq.pop_front() {
    if d > dist[u] { continue; }
    for &(v, w) in &adj[u] {
        let nd = d + w;              // w is 0 or 1
        if nd < dist[v] {
            dist[v] = nd;
            if w == 0 { dq.push_front((v, nd)); }
            else       { dq.push_back((v, nd)); }
        }
    }
}
```

## 7. DSU with path compression + union by rank

The canonical near-O(α(n)) disjoint-set. Memorize it; it appears in Union-Find, MST (Kruskal), and cycle detection.

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
        let mut x = x;
        while self.parent[x] != x {
            self.parent[x] = self.parent[self.parent[x]]; // compression
            x = self.parent[x];
        }
        x
    }
    fn unite(&mut self, a: usize, b: usize) -> bool {
        let (mut ra, mut rb) = (self.find(a), self.find(b));
        if ra == rb { return false; }     // already connected (cycle for undirected)
        if self.rank[ra] < self.rank[rb] { std::mem::swap(&mut ra, &mut rb); }
        self.parent[rb] = ra;
        if self.rank[ra] == self.rank[rb] { self.rank[ra] += 1; }
        true
    }
}
```

## 8. 3-color DFS for directed cycle detection

WHITE (unvisited) / GRAY (on the current DFS stack) / BLACK (done). A GRAY→GRAY edge is a back edge = cycle. Do **not** use parent-tracking for directed graphs (that's the undirected technique).

```rust
// color: Vec<u8> initialized to 0 (0 = white, 1 = gray, 2 = black)
fn has_cycle(u: usize, adj: &Vec<Vec<usize>>, color: &mut Vec<u8>) -> bool {
    color[u] = 1;
    for &v in &adj[u] {
        if color[v] == 1 { return true; }        // back edge → cycle
        if color[v] == 0 && has_cycle(v, adj, color) { return true; }
    }
    color[u] = 2;
    false
}
```

## 9. Kahn's indegree topological sort

BFS over zero-indegree nodes. If you can't process all nodes, the graph has a cycle.

```rust
use std::collections::VecDeque;

let mut indegree = vec![0usize; n];
for e in &edges { indegree[e[1]] += 1; }        // edge u->v
let mut q: VecDeque<usize> = VecDeque::new();
for i in 0..n { if indegree[i] == 0 { q.push_back(i); } }
let mut order: Vec<usize> = Vec::new();
while let Some(u) = q.pop_front() {
    order.push(u);
    for &v in &adj[u] {
        indegree[v] -= 1;
        if indegree[v] == 0 { q.push_back(v); }
    }
}
let acyclic = order.len() == n;           // else cycle
```

## 10. Bitmask state for "visit all nodes"

When the answer depends on *which subset* of nodes you've visited (Shortest Path Visiting All Nodes), make the BFS state `(node, mask)`. Goal: `mask == (1 << n) - 1`.

```rust
use std::collections::VecDeque;

let full = (1usize << n) - 1;
let mut q: VecDeque<(usize, usize)> = VecDeque::new();           // (node, mask)
let mut seen = vec![vec![false; 1 << n]; n];
for i in 0..n { q.push_back((i, 1 << i)); seen[i][1 << i] = true; }
let mut steps = 0;
while !q.is_empty() {
    let size = q.len();
    for _ in 0..size {
        let (node, mask) = q.pop_front().unwrap();
        if mask == full { return steps; }
        for &nb in &adj[node] {
            let nmask = mask | (1 << nb);
            if !seen[nb][nmask] { seen[nb][nmask] = true; q.push_back((nb, nmask)); }
        }
    }
    steps += 1;
}
```

> **Last Updated:** 2026-06-26
