> **Topic:** [Graphs](../README.md) · **Pattern 4 of 7**

# Shortest Path

Shortest-path problems ask for the minimum-cost way to get from a source to one or more destinations in a graph. The "cost" is usually a sum of edge weights, but it can also be a **minimax** quantity (minimize the largest edge on the path) or a **product** (maximize a probability). The right algorithm depends almost entirely on the *kind* of edge weights you have.

## Core Concept & Algorithm Selection

The single most important skill for this pattern is picking the correct algorithm in the first 30 seconds. Read the constraints, identify the edge-weight model, and the algorithm falls out.

| Edge model | Algorithm | Time | Why |
|---|---|---|---|
| Unweighted (all edges cost 1) | **BFS** | `O(V + E)` | Layer-by-layer expansion already gives shortest hop count. |
| Weighted, **non-negative** | **Dijkstra** (min-heap) | `O(E log V)` | Greedy: once a node is popped, its distance is final. |
| **Negative** edges / **bounded** number of hops | **Bellman-Ford** | `O(V · E)` | Relax all edges `V-1` times; tolerates negatives, detects cycles, naturally limits hops. |
| **All-pairs** shortest path | **Floyd-Warshall** | `O(V³)` | DP over intermediate vertices; great for small dense graphs. |
| Weights are only **0 or 1** | **0-1 BFS** (deque) | `O(V + E)` | Push 0-edges to the front, 1-edges to the back — keeps the deque monotone. |

Quick mental flowchart:

1. Are weights all equal (or absent)? -> **BFS**.
2. Are weights only 0/1? -> **0-1 BFS**.
3. Any negative weights, or a "at most K stops/edges" limit? -> **Bellman-Ford**.
4. Need every pair, and `V` is small (<= ~400)? -> **Floyd-Warshall**.
5. Otherwise (non-negative weights, single source) -> **Dijkstra**.

> **Critical pitfall:** when ordering a `BinaryHeap` with a custom comparator, NEVER use `a[1] - b[1]` as the comparison value. The subtraction can overflow for large weights and silently corrupt the heap order. Always use explicit `<` or `>` comparisons.

---

## Dijkstra Template

Greedy single-source shortest path for **non-negative** weights. Maintain a `dist[]` array, push `(dist, node)` onto a min-heap, and pop the closest unfinished node. Because we may push the same node multiple times, **skip stale entries** (a popped distance larger than the recorded `dist[node]`).

```rust
use std::collections::BinaryHeap;
use std::cmp::Reverse;

fn dijkstra(n: usize, adj: &Vec<Vec<(usize, i32)>>, src: usize) -> Vec<i32> {
    // adj[u] holds entries (v, weight)
    let mut dist = vec![i32::MAX; n];
    dist[src] = 0;

    // min-heap ordered by distance; (dist, node)
    let mut pq: BinaryHeap<Reverse<(i32, usize)>> = BinaryHeap::new();
    pq.push(Reverse((0, src)));

    while let Some(Reverse((d, u))) = pq.pop() {
        // skip stale entry: a better distance was already finalized
        if d > dist[u] { continue; }

        for &(v, w) in &adj[u] {
            if dist[u] + w < dist[v] {
                dist[v] = dist[u] + w;
                pq.push(Reverse((dist[v], v)));
            }
        }
    }
    dist
}
```

---

## Bellman-Ford Template

Handles **negative** edge weights and detects negative cycles. Relax every edge `V-1` times: after `i` rounds, all shortest paths using at most `i` edges are correct. A `V`-th round that still relaxes something proves a negative cycle.

```rust
fn bellman_ford(n: usize, edges: &Vec<Vec<i32>>, src: usize) -> Vec<i32> {
    // edges[i] = [u, v, w]
    let mut dist = vec![i32::MAX; n];
    dist[src] = 0;

    // V - 1 relaxation rounds
    for _ in 0..n - 1 {
        for e in edges {
            let u = e[0] as usize;
            let v = e[1] as usize;
            let w = e[2];
            if dist[u] != i32::MAX && dist[u] + w < dist[v] {
                dist[v] = dist[u] + w;
            }
        }
    }

    // optional: negative-cycle check
    for e in edges {
        let u = e[0] as usize;
        let v = e[1] as usize;
        let w = e[2];
        if dist[u] != i32::MAX && dist[u] + w < dist[v] {
            // negative cycle reachable from src
        }
    }
    dist
}
```

---

## Floyd-Warshall Template

All-pairs shortest path via dynamic programming. `dist[i][j]` is iteratively improved by allowing each vertex `k` as an intermediate. The triple loop is `O(V³)` — the `k` loop **must** be outermost.

```rust
fn floyd_warshall(n: usize, edges: &Vec<Vec<i32>>) -> Vec<Vec<i32>> {
    const INF: i32 = 1_000_000_000;
    let mut dist = vec![vec![INF; n]; n];
    for i in 0..n { dist[i][i] = 0; }

    for e in edges {
        let (i, j, w) = (e[0] as usize, e[1] as usize, e[2]);
        dist[i][j] = dist[i][j].min(w);
        dist[j][i] = dist[j][i].min(w); // drop if directed
    }

    // k outermost: "can we route i->j through k?"
    for k in 0..n {
        for i in 0..n {
            if dist[i][k] == INF { continue; }          // micro-optimization
            for j in 0..n {
                if dist[i][k] + dist[k][j] < dist[i][j] {
                    dist[i][j] = dist[i][k] + dist[k][j];
                }
            }
        }
    }
    dist
}
```

---

## 0-1 BFS Template

When every edge costs **0 or 1**, a plain deque beats Dijkstra's log factor. Push 0-weight relaxations to the **front** (`push_front`) and 1-weight ones to the **back** (`push_back`). The deque stays sorted by distance, so the first time you pop a node its distance is final.

```rust
use std::collections::VecDeque;

fn zero_one_bfs(n: usize, adj: &Vec<Vec<(usize, i32)>>, src: usize) -> Vec<i32> {
    // adj[u] holds entries (v, weight in {0,1})
    let mut dist = vec![i32::MAX; n];
    dist[src] = 0;

    let mut dq: VecDeque<(usize, i32)> = VecDeque::new();  // (node, dist)
    dq.push_front((src, 0));

    while let Some((u, d)) = dq.pop_front() {
        if d > dist[u] { continue; }  // stale

        for &(v, w) in &adj[u] {
            if dist[u] + w < dist[v] {
                dist[v] = dist[u] + w;
                if w == 0 { dq.push_front((v, dist[v])); }
                else       { dq.push_back((v, dist[v])); }
            }
        }
    }
    dist
}
```

---

## Recognition Signals

| Signal in the problem statement | Likely algorithm |
|---|---|
| "minimum time / cost / distance from a single source", non-negative weights | Dijkstra |
| Grid where the path cost is the **max** (or min) single step (minimax) | Dijkstra / binary search on a grid |
| "at most K stops", "within K moves", or negative prices | Bellman-Ford |
| "for every pair", "for each city", small `V` | Floyd-Warshall |
| Each move is free in one direction, costs 1 otherwise | 0-1 BFS |
| Unweighted graph, "fewest steps / moves" | BFS |
| Path must **visit all nodes**, small `V` (<= ~12) | BFS + bitmask state |
| "maximum probability", multiply weights | Dijkstra with a max-heap |

---

## 26. Network Delay Time — Medium (LC743)

**Idea:** Run Dijkstra from source `k`; the answer is the maximum finalized distance (the time for the *last* node to receive the signal), or `-1` if any node is unreachable.

```rust
use std::collections::BinaryHeap;
use std::cmp::Reverse;

struct Solution;

impl Solution {
    pub fn network_delay_time(times: Vec<Vec<i32>>, n: i32, k: i32) -> i32 {
        let n = n as usize;
        let k = k as usize;
        let mut adj: Vec<Vec<(usize, i32)>> = vec![vec![]; n + 1];
        for t in &times {
            adj[t[0] as usize].push((t[1] as usize, t[2]));
        }

        let mut dist = vec![i32::MAX; n + 1];
        dist[k] = 0;

        let mut pq: BinaryHeap<Reverse<(i32, usize)>> = BinaryHeap::new();
        pq.push(Reverse((0, k)));

        while let Some(Reverse((d, u))) = pq.pop() {
            if d > dist[u] { continue; }  // stale entry

            for &(v, w) in &adj[u] {
                if dist[u] + w < dist[v] {
                    dist[v] = dist[u] + w;
                    pq.push(Reverse((dist[v], v)));
                }
            }
        }

        let ans = dist[1..=n].iter().copied().max().unwrap_or(i32::MAX);
        if ans == i32::MAX { -1 } else { ans }
    }
}
```

**Complexity:** Time `O(E log V)`, Space `O(V + E)`.

### Dry Run — Dijkstra on Network Delay Time

Input: `times = [[2,1,1],[2,3,1],[3,4,1]]`, `n = 4`, `k = 2`.

Adjacency: `2 -> {(1,1),(3,1)}`, `3 -> {(4,1)}`, `1 -> {}`, `4 -> {}`.
Initial `dist = [_, INF, 0, INF, INF]` (index 0 unused), `pq = [(2,0)]`.

| Step | Pop `(u,d)` | Stale? | Relaxations | `dist` after (1..4) | `pq` after |
|---|---|---|---|---|---|
| 1 | `(2,0)` | no | `dist[1]=1` push `(1,1)`; `dist[3]=1` push `(3,1)` | `1, 0, 1, INF` | `(1,1),(3,1)` |
| 2 | `(1,1)` | no | node 1 has no edges | `1, 0, 1, INF` | `(3,1)` |
| 3 | `(3,1)` | no | `dist[4]=2` push `(4,2)` | `1, 0, 1, 2` | `(4,2)` |
| 4 | `(4,2)` | no | node 4 has no edges | `1, 0, 1, 2` | empty |

Final `dist[1..4] = [1, 0, 1, 2]`. No `INF` remains, so the answer is `max(1,0,1,2) = 2`. The signal reaches every node, and the slowest arrival (node 4) defines the total delay.

Note how the heap ordering by distance guarantees node 2's neighbors are settled before node 3's neighbor — each node is popped exactly once at its final distance.

---

## 27. Cheapest Flights Within K Stops — Medium (LC787)

**Idea:** Bellman-Ford limited to `K+1` edge-relaxation rounds; **clone** the `dist` array each round so a single round can only extend paths from the *previous* round (preventing more than the allowed number of stops).

```rust
struct Solution;

impl Solution {
    pub fn find_cheapest_price(n: i32, flights: Vec<Vec<i32>>, src: i32, dst: i32, k: i32) -> i32 {
        let n = n as usize;
        let src = src as usize;
        let dst = dst as usize;
        let mut dist = vec![i32::MAX; n];
        dist[src] = 0;

        // K stops => at most K+1 edges => K+1 relaxation rounds
        for _ in 0..=k {
            let prev = dist.clone();  // freeze last round's distances
            for f in &flights {
                let u = f[0] as usize;
                let v = f[1] as usize;
                let w = f[2];
                if prev[u] != i32::MAX && prev[u] + w < dist[v] {
                    dist[v] = prev[u] + w;
                }
            }
        }
        if dist[dst] == i32::MAX { -1 } else { dist[dst] }
    }
}
```

**Why clone:** without the snapshot, relaxing edge A then edge B in the same round could chain two new edges, sneaking in an extra stop beyond `K`.

**Complexity:** Time `O(K · E)`, Space `O(V)`.

---

## 28. Path With Minimum Effort — Medium (LC1631)

**Idea:** Dijkstra on the grid where the "distance" of a path is the **maximum** absolute height difference between adjacent cells along it (minimax). Relaxation uses `max(currentEffort, |h - hNeighbor|)`.

```rust
use std::collections::BinaryHeap;
use std::cmp::Reverse;

struct Solution;

impl Solution {
    pub fn minimum_effort_path(heights: Vec<Vec<i32>>) -> i32 {
        let rows = heights.len();
        let cols = heights[0].len();
        let dirs: [(i32, i32); 4] = [(0, 1), (1, 0), (0, -1), (-1, 0)];

        let mut effort = vec![vec![i32::MAX; cols]; rows];
        effort[0][0] = 0;

        // (effortSoFar, row, col)
        let mut pq: BinaryHeap<Reverse<(i32, usize, usize)>> = BinaryHeap::new();
        pq.push(Reverse((0, 0, 0)));

        while let Some(Reverse((e, r, c))) = pq.pop() {
            if e > effort[r][c] { continue; }  // stale
            if r == rows - 1 && c == cols - 1 { return e; }  // first pop is optimal

            for &(dr, dc) in &dirs {
                let nr = r as i32 + dr;
                let nc = c as i32 + dc;
                if nr < 0 || nr >= rows as i32 || nc < 0 || nc >= cols as i32 { continue; }
                let (nr, nc) = (nr as usize, nc as usize);
                let ne = e.max((heights[nr][nc] - heights[r][c]).abs());
                if ne < effort[nr][nc] {
                    effort[nr][nc] = ne;
                    pq.push(Reverse((ne, nr, nc)));
                }
            }
        }
        0 // single cell grid
    }
}
```

**Complexity:** Time `O(R·C·log(R·C))`, Space `O(R·C)`.

---

## 29. Swim in Rising Water — Hard (LC778)

**Idea:** The time to reach the bottom-right equals the **minimum over all paths of the maximum elevation** on that path — another minimax Dijkstra. The cost of stepping onto a cell is its elevation; the path cost is the max elevation seen.

```rust
use std::collections::BinaryHeap;
use std::cmp::Reverse;

struct Solution;

impl Solution {
    pub fn swim_in_water(grid: Vec<Vec<i32>>) -> i32 {
        let n = grid.len();
        let dirs: [(i32, i32); 4] = [(0, 1), (1, 0), (0, -1), (-1, 0)];

        let mut visited = vec![vec![false; n]; n];
        // (maxElevationSoFar, row, col)
        let mut pq: BinaryHeap<Reverse<(i32, usize, usize)>> = BinaryHeap::new();
        pq.push(Reverse((grid[0][0], 0, 0)));

        while let Some(Reverse((t, r, c))) = pq.pop() {
            if visited[r][c] { continue; }
            visited[r][c] = true;
            if r == n - 1 && c == n - 1 { return t; }  // first arrival is optimal

            for &(dr, dc) in &dirs {
                let nr = r as i32 + dr;
                let nc = c as i32 + dc;
                if nr < 0 || nr >= n as i32 || nc < 0 || nc >= n as i32 { continue; }
                let (nr, nc) = (nr as usize, nc as usize);
                if visited[nr][nc] { continue; }
                pq.push(Reverse((t.max(grid[nr][nc]), nr, nc)));
            }
        }
        -1 // unreachable (won't happen on valid input)
    }
}
```

**Complexity:** Time `O(N²·log N)`, Space `O(N²)`.

---

## 30. Path with Maximum Probability — Medium (LC1514)

**Idea:** Dijkstra with a **max-heap**, maximizing the **product** of edge probabilities. Since probabilities are in `[0,1]`, products only shrink, so the greedy "pop the largest first" property holds just like minimizing a sum.

```rust
use std::collections::BinaryHeap;

struct Solution;

impl Solution {
    pub fn max_probability(n: i32, edges: Vec<Vec<i32>>, succ_prob: Vec<f64>, start: i32, end: i32) -> f64 {
        let n = n as usize;
        let start = start as usize;
        let end = end as usize;
        let mut adj: Vec<Vec<(usize, f64)>> = vec![vec![]; n];
        for i in 0..edges.len() {
            let u = edges[i][0] as usize;
            let v = edges[i][1] as usize;
            let p = succ_prob[i];
            adj[u].push((v, p));
            adj[v].push((u, p));
        }

        let mut prob = vec![0.0f64; n];
        prob[start] = 1.0;

        // max-heap by probability; encode f64 as bits for ordering (valid for non-negative values)
        let mut pq: BinaryHeap<(u64, usize)> = BinaryHeap::new();
        pq.push((1.0f64.to_bits(), start));

        while let Some((p_bits, u)) = pq.pop() {
            let p = f64::from_bits(p_bits);
            if p < prob[u] { continue; }  // stale
            if u == end { return p; }     // first pop of end is optimal

            for &(v, ep) in &adj[u] {
                let np = p * ep;
                if np > prob[v] {
                    prob[v] = np;
                    pq.push((np.to_bits(), v));
                }
            }
        }
        0.0 // end unreachable
    }
}
```

**Complexity:** Time `O(E log V)`, Space `O(V + E)`.

---

## 31. Find the City With Smallest Number of Neighbors at a Threshold Distance — Medium (LC1334)

**Idea:** Compute all-pairs shortest paths with Floyd-Warshall, then for each city count how many others lie within `distanceThreshold`; pick the city with the fewest such neighbors, breaking ties by the **largest** index.

```rust
struct Solution;

impl Solution {
    pub fn find_the_city(n: i32, edges: Vec<Vec<i32>>, distance_threshold: i32) -> i32 {
        let n = n as usize;
        const INF: i32 = 1_000_000_000;
        let mut dist = vec![vec![INF; n]; n];
        for i in 0..n { dist[i][i] = 0; }

        for e in &edges {
            let (u, v, w) = (e[0] as usize, e[1] as usize, e[2]);
            dist[u][v] = w;
            dist[v][u] = w;  // undirected
        }

        for k in 0..n {
            for i in 0..n {
                if dist[i][k] == INF { continue; }
                for j in 0..n {
                    if dist[i][k] + dist[k][j] < dist[i][j] {
                        dist[i][j] = dist[i][k] + dist[k][j];
                    }
                }
            }
        }

        let mut best_city = -1i32;
        let mut fewest = i32::MAX;
        for i in 0..n {
            let count = (0..n)
                .filter(|&j| i != j && dist[i][j] <= distance_threshold)
                .count() as i32;
            // <= so that on ties we keep the larger index
            if count <= fewest {
                fewest = count;
                best_city = i as i32;
            }
        }
        best_city
    }
}
```

**Complexity:** Time `O(V³)`, Space `O(V²)`.

---

## 32. Minimum Cost to Make at Least One Valid Path in a Grid — Hard (LC1368)

**Idea:** 0-1 BFS. Each cell points a direction; **following** the arrow costs `0`, **changing** it (moving any other direction) costs `1`. Find the minimum-cost path from top-left to bottom-right.

```rust
use std::collections::VecDeque;

struct Solution;

impl Solution {
    pub fn min_cost(grid: Vec<Vec<i32>>) -> i32 {
        let rows = grid.len();
        let cols = grid[0].len();
        // grid value 1=right, 2=left, 3=down, 4=up -> matching dirs index
        let dirs: [(i32, i32); 4] = [(0, 1), (0, -1), (1, 0), (-1, 0)];

        let mut dist = vec![vec![i32::MAX; cols]; rows];
        dist[0][0] = 0;

        let mut dq: VecDeque<(usize, usize, i32)> = VecDeque::new();  // (row, col, cost)
        dq.push_front((0, 0, 0));

        while let Some((r, c, cost)) = dq.pop_front() {
            if cost > dist[r][c] { continue; }  // stale

            for dir in 0..4usize {
                let nr = r as i32 + dirs[dir].0;
                let nc = c as i32 + dirs[dir].1;
                if nr < 0 || nr >= rows as i32 || nc < 0 || nc >= cols as i32 { continue; }
                let (nr, nc) = (nr as usize, nc as usize);
                // grid is 1-indexed direction; arrow already points dir -> cost 0
                let w = if grid[r][c] == dir as i32 + 1 { 0 } else { 1 };
                if cost + w < dist[nr][nc] {
                    dist[nr][nc] = cost + w;
                    if w == 0 { dq.push_front((nr, nc, cost)); }
                    else       { dq.push_back((nr, nc, cost + 1)); }
                }
            }
        }
        dist[rows - 1][cols - 1]
    }
}
```

**Complexity:** Time `O(R·C)`, Space `O(R·C)`.

---

## 33. Shortest Path Visiting All Nodes — Hard (LC847)

**Idea:** Unweighted shortest path over a **state graph** `(node, visitedMask)`. BFS layer by layer; the goal is any node whose mask equals all-ones (`(1<<n)-1`). Because `n <= 12`, the total state count `n · 2^n` is small.

```rust
use std::collections::VecDeque;

struct Solution;

impl Solution {
    pub fn shortest_path_length(graph: Vec<Vec<i32>>) -> i32 {
        let n = graph.len();
        if n == 1 { return 0; }
        let full = (1 << n) - 1;

        let mut visited = vec![vec![false; 1 << n]; n];
        let mut queue: VecDeque<(usize, usize)> = VecDeque::new();  // (node, mask)

        // can start from any node
        for i in 0..n {
            queue.push_back((i, 1 << i));
            visited[i][1 << i] = true;
        }

        let mut steps = 0i32;
        while !queue.is_empty() {
            let size = queue.len();
            for _ in 0..size {
                let (node, mask) = queue.pop_front().unwrap();
                if mask == full { return steps; }  // all nodes visited

                for &next in &graph[node] {
                    let next = next as usize;
                    let next_mask = mask | (1 << next);
                    if !visited[next][next_mask] {
                        visited[next][next_mask] = true;
                        queue.push_back((next, next_mask));
                    }
                }
            }
            steps += 1;
        }
        -1 // graph guaranteed connected, so unreachable
    }
}
```

**Complexity:** Time `O(n² · 2ⁿ)`, Space `O(n · 2ⁿ)`.

---

> **Last Updated:** 2026-06-26
