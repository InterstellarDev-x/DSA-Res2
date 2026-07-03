> **Topic:** [Graphs](../README.md) · **Section:** Interview Problems · **Company:** Microsoft

# Microsoft — Graphs Interview Deep Dives

Microsoft values clean, correct implementations and precise complexity reasoning. These deep dives cover its most-asked graph problems. See also [Amazon](./Amazon.md) and [Google](./Google.md).

---

## 1. Clone Graph (LC 133)

Deep-copy a connected undirected graph from a node reference.

### The HashMap dual role

A single `HashMap<i32, Vec<i32>>` from **node value → neighbor list** serves two purposes at once: it's the **visited set** (key presence) *and* the **clone registry** (value). When you meet a node, if it's already a key you reuse its clone; otherwise you create one, store it, and recurse/enqueue. This is what makes cycles safe — you never re-create a node, so traversal terminates.

```rust
use std::collections::{HashMap, VecDeque};

// Graph represented as adjacency list: node_val -> list of neighbor vals
// (node values are unique and serve as node IDs)

// BFS
fn clone_graph_bfs(start: i32, graph: &HashMap<i32, Vec<i32>>) -> HashMap<i32, Vec<i32>> {
    if !graph.contains_key(&start) {
        return HashMap::new();
    }
    let mut cloned: HashMap<i32, Vec<i32>> = HashMap::new();
    let mut q: VecDeque<i32> = VecDeque::new();
    q.push_back(start);
    cloned.insert(start, Vec::new());
    while let Some(cur) = q.pop_front() {
        let neighbors = graph[&cur].clone();
        for nb in neighbors {
            if !cloned.contains_key(&nb) {
                cloned.insert(nb, Vec::new());
                q.push_back(nb);
            }
            cloned.get_mut(&cur).unwrap().push(nb);
        }
    }
    cloned
}

// DFS alternative
fn dfs(node: i32, graph: &HashMap<i32, Vec<i32>>, cloned: &mut HashMap<i32, Vec<i32>>) {
    if cloned.contains_key(&node) { return; }
    cloned.insert(node, Vec::new()); // store BEFORE recursing (breaks cycles)
    if let Some(neighbors) = graph.get(&node) {
        let neighbors = neighbors.clone();
        for nb in neighbors {
            dfs(nb, graph, cloned);
            cloned.get_mut(&node).unwrap().push(nb);
        }
    }
}

fn clone_graph_dfs(start: i32, graph: &HashMap<i32, Vec<i32>>) -> HashMap<i32, Vec<i32>> {
    let mut cloned: HashMap<i32, Vec<i32>> = HashMap::new();
    dfs(start, graph, &mut cloned);
    cloned
}
```

**BFS vs DFS here:** identical complexity O(V+E); DFS is shorter but risks deep recursion on long chains, BFS uses an explicit queue. The critical detail in DFS is putting the clone in the map **before** recursing into neighbors — otherwise a cycle re-enters and overflows.

### Edge cases — null input → `None`; single node with no neighbors; self-loop; node referencing itself.

### Follow-ups — clone a graph with random pointers; serialize/deserialize a graph.

---

## 2. Network Delay Time (LC 743)

Time for a signal from node `k` to reach all nodes; `-1` if some unreachable.

### Dijkstra with stale-entry skipping

A binary heap doesn't support decrease-key cheaply, so the standard trick is **lazy deletion**: push a new `(dist, node)` whenever you relax, and when you pop, **skip the entry if its distance is worse than the best known**. This keeps the heap correct without an indexed PQ.

```rust
use std::collections::{HashMap, BinaryHeap};
use std::cmp::Reverse;

fn network_delay_time(times: Vec<Vec<i32>>, n: i32, k: i32) -> i32 {
    let mut adj: HashMap<i32, Vec<(i32, i32)>> = HashMap::new();
    for t in &times {
        adj.entry(t[0]).or_default().push((t[1], t[2]));
    }
    let n = n as usize;
    let mut dist = vec![i32::MAX; n + 1];
    dist[k as usize] = 0;
    // min-heap by distance: (Reverse(dist), node)
    let mut pq: BinaryHeap<(Reverse<i32>, i32)> = BinaryHeap::new();
    pq.push((Reverse(0), k));
    while let Some((Reverse(d), node)) = pq.pop() {
        if d > dist[node as usize] { continue; }            // STALE entry - skip
        if let Some(neighbors) = adj.get(&node) {
            for &(v, w) in neighbors {
                let nd = d + w;
                if nd < dist[v as usize] {
                    dist[v as usize] = nd;
                    pq.push((Reverse(nd), v));
                }
            }
        }
    }
    let mut ans = 0;
    for i in 1..=n {
        if dist[i] == i32::MAX { return -1; }
        ans = ans.max(dist[i]);
    }
    ans
}
```

**Two non-negotiables:** use `BinaryHeap<(Reverse<i32>, i32)>` with `Reverse` for min-heap (subtraction comparators overflow on large weights); and the `if d > dist[node as usize] { continue }` skip avoids reprocessing finalized nodes — without it you can still get the right answer but with wasted work, and it's the detail interviewers probe.

### Edge cases — unreachable node → `-1`; `k` itself counts (distance 0); duplicate edges → keep the cheaper via relaxation.

### Follow-ups — *Path With Minimum Effort* (minimize the max edge → relax on `max(d, edge)`); *Swim in Rising Water* (same minimax Dijkstra).

---

## 3. Cheapest Flights Within K Stops (LC 787)

Cheapest price from `src` to `dst` using at most `k` stops.

### Why Bellman-Ford / per-round dist cloning

Plain Dijkstra fails because the cheapest cost ignoring the stop limit may use too many hops; greedily finalizing a node can lock in a path that violates the `k` constraint. **Bellman-Ford** relaxes all edges exactly `k+1` times — each round = "one more flight allowed" — which naturally bounds the hop count.

The subtlety: within a single round you must relax using the **previous round's** distances, not values updated earlier in the same round. Otherwise one round could chain multiple edges (more than one extra stop). So **clone `dist` at the start of each round** and read from the clone, write to the live array.

```rust
fn find_cheapest_price(n: i32, flights: Vec<Vec<i32>>, src: i32, dst: i32, k: i32) -> i32 {
    let n = n as usize;
    let mut dist = vec![i32::MAX; n];
    dist[src as usize] = 0;
    for _round in 0..=k {                              // k+1 relaxations
        let prev = dist.clone();                       // freeze this round's source
        for f in &flights {
            let (u, v, w) = (f[0] as usize, f[1] as usize, f[2]);
            if prev[u] == i32::MAX { continue; }
            dist[v] = dist[v].min(prev[u] + w);
        }
    }
    if dist[dst as usize] == i32::MAX { -1 } else { dist[dst as usize] }
}
```

**Without the clone**, a single round could relax `src→a→b`, spending two hops while only one should be allowed — undercounting stops and returning illegally cheap prices.

### Edge cases — `src == dst` → 0; no path within k → `-1`; k = 0 → only direct flights.

### Follow-ups — return the actual route; minimize stops then cost; Dijkstra variant keyed by `(node, stopsUsed)` state.

---

## 4. Surrounded Regions (LC 130)

Capture all `'O'` regions not connected to the border (flip them to `'X'`).

### The border-DFS trick

Instead of finding enclosed regions (hard to define directly), **invert the problem**: an `'O'` survives only if it's connected to a border `'O'`. DFS/BFS from every **border** `'O'`, marking the safe ones with a temporary sentinel (`'#'`). Then sweep the grid: remaining `'O'` → `'X'` (captured), `'#'` → `'O'` (restore).

```rust
fn mark(board: &mut Vec<Vec<char>>, r: i32, c: i32) {
    if r < 0 || r >= board.len() as i32 || c < 0 || c >= board[0].len() as i32 {
        return;
    }
    if board[r as usize][c as usize] != 'O' { return; }
    board[r as usize][c as usize] = '#';               // sentinel for border-connected
    mark(board, r + 1, c); mark(board, r - 1, c);
    mark(board, r, c + 1); mark(board, r, c - 1);
}

fn solve(board: &mut Vec<Vec<char>>) {
    if board.is_empty() || board[0].is_empty() { return; }
    let m = board.len();
    let n = board[0].len();
    for i in 0..m {
        mark(board, i as i32, 0);
        mark(board, i as i32, n as i32 - 1);
    }
    for j in 0..n {
        mark(board, 0, j as i32);
        mark(board, m as i32 - 1, j as i32);
    }
    for i in 0..m {
        for j in 0..n {
            if board[i][j] == 'O' { board[i][j] = 'X'; }       // enclosed -> capture
            else if board[i][j] == '#' { board[i][j] = 'O'; }  // border-safe -> restore
        }
    }
}
```

### Edge cases — single row/column (all border, nothing captured); no `'O'`; all `'O'` touching border.

### Follow-ups — *Number of Enclaves* (count, don't flip); *Pacific Atlantic* uses the same reverse-from-border idea.

---

## What Microsoft Emphasizes

| Dimension | Expectation |
|-----------|-------------|
| Correctness | Handle `-1` / `None` / single-cell cases |
| Clarity | `let dirs`, named helpers, no subtraction comparators |
| Complexity | State `O(V+E)`, `O((V+E) log V)`, `O(V·E)` precisely |
| Reasoning | Explain *why* the algorithm fits (negative/limited stops → Bellman-Ford) |

> **Last Updated:** 2026-06-26
