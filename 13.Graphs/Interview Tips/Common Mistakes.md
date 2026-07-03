> **Topic:** [Graphs](../README.md) · **Section:** Interview Tips · **Tips 2 of 4**

# Graphs — Common Mistakes

Twelve recurring bugs as wrong/correct pairs. Each costs correctness or a TLE in real interviews. See also [Coding Tips](./Coding%20Tips.md), [Algorithm Selection](./Algorithm%20Selection.md), [Complexity Analysis](./Complexity%20Analysis.md).

---

## 1. Marking visited at dequeue instead of enqueue → TLE / duplicates

```rust
use std::collections::VecDeque;
// WRONG — same node enqueued by many neighbors before it's marked
while let Some(u) = q.pop_front() {
    if visited[u] { continue; }       // marked too late
    visited[u] = true;
    for &v in &adj[u] { q.push_back(v); }
}
// CORRECT — mark the instant you enqueue
visited[start] = true; q.push_back(start);
while let Some(u) = q.pop_front() {
    for &v in &adj[u] {
        if !visited[v] { visited[v] = true; q.push_back(v); }
    }
}
```

## 2. Subtraction in a PQ comparator → integer overflow

```rust
use std::collections::BinaryHeap;
use std::cmp::Reverse;
// WRONG — subtraction overflows when distances are large
// (a[1] - b[1] > 0 as an ordering expression is unsafe for large i32 values)
// CORRECT — min-heap by dist using Reverse, no overflow
let mut pq: BinaryHeap<Reverse<(i32, i32)>> = BinaryHeap::new(); // (dist, node)
// push: pq.push(Reverse((dist, node)));
// pop:  if let Some(Reverse((d, u))) = pq.pop() { ... }
```

## 3. Forgetting the stale-entry skip in Dijkstra

```rust
use std::cmp::Reverse;
// WRONG — reprocesses finalized nodes, may relax with outdated d
if let Some(Reverse((d, u))) = pq.pop() {
    for &(v, w) in &adj[u] { /* relax using d */ }
}
// CORRECT
if let Some(Reverse((d, u))) = pq.pop() {
    if d > dist[u] { continue; }          // skip stale entry
    for &(v, w) in &adj[u] { /* relax using d */ }
}
```

## 4. Bellman-Ford for K-stops without cloning `dist`

```rust
// WRONG — one round chains multiple edges → exceeds stop limit
for _round in 0..=k {
    for f in &flights {
        let (src, dst, cost) = (f[0], f[1], f[2]);
        dist[dst] = dist[dst].min(dist[src] + cost);
    }
}
// CORRECT — freeze previous round's distances
for _round in 0..=k {
    let prev = dist.clone();
    for f in &flights {
        let (src, dst, cost) = (f[0], f[1], f[2]);
        if prev[src] != i32::MAX {
            dist[dst] = dist[dst].min(prev[src] + cost);
        }
    }
}
```

## 5. Adding an undirected edge in only one direction

```rust
// WRONG — half the graph is missing
adj[u].push(v);
// CORRECT
adj[u].push(v);
adj[v].push(u);
```

## 6. DSU without path compression → near-O(n) find, TLE

```rust
// WRONG — degenerate chains
fn find(parent: &[usize], mut x: usize) -> usize {
    while parent[x] != x { x = parent[x]; }
    x
}
// CORRECT — path compression (+ union by rank)
fn find(parent: &mut Vec<usize>, mut x: usize) -> usize {
    while parent[x] != x {
        let gp = parent[parent[x]]; // path halving
        parent[x] = gp;
        x = parent[x];
    }
    x
}
```

## 7. Directed cycle detection via parent-tracking → wrong

Parent-skipping detects cycles in **undirected** graphs only. For directed graphs use 3-color.

```rust
// WRONG for directed — misses cycles like a->b->a's true direction semantics
fn dfs_wrong(u: usize, par: usize, adj: &[Vec<usize>], visited: &mut Vec<bool>) -> bool {
    for &v in &adj[u] {
        if v == par { continue; }                 // undirected idea, wrong here
        if visited[v] || dfs_wrong(v, u, adj, visited) { return true; }
    }
    true
}
// CORRECT for directed — gray = on current path
fn dfs(u: usize, adj: &[Vec<usize>], color: &mut Vec<i32>) -> bool {
    color[u] = 1;
    for &v in &adj[u] {
        if color[v] == 1 { return true; }         // back edge
        if color[v] == 0 && dfs(v, adj, color) { return true; }
    }
    color[u] = 2;
    false
}
```

## 8. Kahn's without checking all nodes processed → misses cycle

```rust
use std::collections::VecDeque;
// WRONG — returns a partial "order" even with a cycle
while let Some(u) = q.pop_front() { order.push(u); /* ... */ }
return order;
// CORRECT
if order.len() == n { order } else { vec![] }   // cycle ⇒ empty
```

## 9. Grid bounds check ordered after the array read → out-of-bounds

```rust
// WRONG — reads grid[r][c] before validating r,c (panics if out of bounds)
if grid[r as usize][c as usize] == b'1' && r >= 0 && r < m && c >= 0 && c < n { ... }
// CORRECT — bounds FIRST, short-circuit protects the read
if r >= 0 && r < m && c >= 0 && c < n && grid[r as usize][c as usize] == b'1' { ... }
```

## 10. Modifying the grid during BFS without marking → revisits / infinite loop

```rust
// WRONG — neighbor added repeatedly; never marked
for &(dr, dc) in &dirs {
    let (r, c) = (...);
    if grid[r][c] == 1 { q.push_back((r, c)); }
}
// CORRECT — mark as you enqueue
for &(dr, dc) in &dirs {
    let (r, c) = (...);
    if in_bounds(r, c) && grid[r][c] == 1 { grid[r][c] = 2; q.push_back((r, c)); }
}
```

## 11. Alien Dictionary missing the prefix-conflict check

```rust
// WRONG — ["abc","ab"] should be invalid but slips through
let j = a.chars().zip(b.chars()).take_while(|(x, y)| x == y).count();
if j < len { add_edge(a_chars[j], b_chars[j]); }
// CORRECT
if j == len {
    if a.len() > b.len() { return String::new(); }   // longer word before its prefix
} else {
    add_edge(a_chars[j], b_chars[j]);
}
```

## 12. 8-direction vs 4-direction mix-up

Number of Islands / Rotting Oranges are **4-directional**. Using `dirs8` over-connects and inflates/merges islands. Conversely, problems like *Shortest Path in Binary Matrix* are **8-directional** — using 4 dirs gives wrong (too-long or no) paths. Always confirm connectivity from the prompt before choosing `dirs` vs `dirs8`.

```rust
let dirs: &[(i32, i32)]  = &[(1,0),(-1,0),(0,1),(0,-1)];                               // islands, rotting
let dirs8: &[(i32, i32)] = &[(1,0),(-1,0),(0,1),(0,-1),(1,1),(1,-1),(-1,1),(-1,-1)];   // binary matrix path
```

> **Last Updated:** 2026-06-26
