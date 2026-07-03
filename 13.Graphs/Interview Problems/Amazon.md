> **Topic:** [Graphs](../README.md) · **Section:** Interview Problems · **Company:** Amazon

# Amazon — Graphs Interview Deep Dives

Full-reasoning walkthroughs of the graph problems Amazon asks most. Each includes the decision behind the approach, edge cases, and follow-ups. See also [Google](./Google.md) and [Microsoft](./Microsoft.md).

---

## 1. Number of Islands (LC 200)

Count 4-connected components of `'1'` in an `m x n` grid.

### Approach trade-offs

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| **DFS (recursive)** | O(m·n) | O(m·n) stack worst case | Cleanest; risk of stack overflow on huge single islands |
| **BFS (iterative)** | O(m·n) | O(min(m,n)) queue | Safe for large grids; preferred when depth unbounded |
| **Union-Find** | O(m·n·α) | O(m·n) | Best when land is **added incrementally** (LC 305) |

The key shared idea: **mark visited in place** by setting `grid[r][c] = '0'`, which removes the need for a separate visited matrix.

### BFS solution

```rust
use std::collections::VecDeque;

fn num_islands(grid: &mut Vec<Vec<char>>) -> i32 {
    if grid.is_empty() || grid[0].is_empty() { return 0; }
    let m = grid.len();
    let n = grid[0].len();
    let mut count = 0;
    let dirs: [(i32, i32); 4] = [(1,0),(-1,0),(0,1),(0,-1)];
    let mut q: VecDeque<(usize, usize)> = VecDeque::new();
    for i in 0..m {
        for j in 0..n {
            if grid[i][j] != '1' { continue; }
            count += 1;
            grid[i][j] = '0'; // mark at enqueue
            q.push_back((i, j));
            while let Some((r0, c0)) = q.pop_front() {
                for &(dr, dc) in &dirs {
                    let r = r0 as i32 + dr;
                    let c = c0 as i32 + dc;
                    if r < 0 || r >= m as i32 || c < 0 || c >= n as i32 { continue; }
                    let (r, c) = (r as usize, c as usize);
                    if grid[r][c] != '1' { continue; }
                    grid[r][c] = '0'; // mark BEFORE enqueue → no duplicates
                    q.push_back((r, c));
                }
            }
        }
    }
    count
}
```

**Why mark at enqueue, not dequeue?** If you only mark when popping, the same cell can be enqueued by multiple neighbors before it is processed — duplicate work, inflated queue, and potential double counting. Marking the instant you push guarantees each cell enters the queue once.

### Edge cases
- Empty / null grid → `0`.
- All water → `0`; all land → `1`.
- Single row/column (1-D grid).

### Follow-ups
- **Number of Islands II** (LC 305): land cells added one at a time, return island count after each → [Union Find (DSU)](../Patterns/Union%20Find%20(DSU).md).
- **Making A Large Island** (LC 827): flip one `0`; label each island with a size and sum distinct neighboring labels.
- "Grid doesn't fit in memory" → process row-by-row with DSU, or distributed connected components.

---

## 2. Rotting Oranges (LC 994)

Each minute every rotten orange (`2`) rots 4-adjacent fresh (`1`). Minutes until none fresh, else `-1`.

### Why seed all sources first

This is **multi-source BFS**. All rotten oranges rot their neighbors *simultaneously* each minute. If you BFS from one source at a time, you'd compute the wrong (sequential) timing. Seeding the queue with **every** rotten cell at time 0 makes each BFS level == one minute, so they spread in lockstep.

```rust
use std::collections::VecDeque;

fn oranges_rotting(grid: &mut Vec<Vec<i32>>) -> i32 {
    let m = grid.len();
    let n = grid[0].len();
    let mut fresh = 0;
    let mut minutes = 0;
    let mut q: VecDeque<(usize, usize)> = VecDeque::new();
    for i in 0..m {
        for j in 0..n {
            if grid[i][j] == 2 { q.push_back((i, j)); }
            else if grid[i][j] == 1 { fresh += 1; }
        }
    }
    if fresh == 0 { return 0; } // edge case: nothing to rot
    let dirs: [(i32, i32); 4] = [(1,0),(-1,0),(0,1),(0,-1)];
    while !q.is_empty() && fresh > 0 {
        minutes += 1;
        let size = q.len();
        for _ in 0..size { // process exactly one level
            let (r0, c0) = q.pop_front().unwrap();
            for &(dr, dc) in &dirs {
                let r = r0 as i32 + dr;
                let c = c0 as i32 + dc;
                if r < 0 || r >= m as i32 || c < 0 || c >= n as i32 { continue; }
                let (r, c) = (r as usize, c as usize);
                if grid[r][c] != 1 { continue; }
                grid[r][c] = 2;
                fresh -= 1;
                q.push_back((r, c));
            }
        }
    }
    if fresh == 0 { minutes } else { -1 }
}
```

### Edge cases
- No fresh oranges at start → `0` (don't return 1).
- A fresh orange with no path to any rotten → `-1` (tracked via `fresh` counter).
- Processing one level at a time with `size = q.len()` is essential to count minutes.

### Follow-ups
- **01 Matrix / Walls and Gates** — identical multi-source seeding.
- Different rot speeds per cell → weighted → [Shortest Path](../Patterns/Shortest%20Path.md) (Dijkstra / 0-1 BFS).

---

## 3. Course Schedule II (LC 210)

Return a valid course order, or `[]` if a cycle exists.

### Kahn's vs DFS

| | Kahn's (BFS indegree) | DFS postorder |
|--|----------------------|---------------|
| Cycle detection | `processed < numCourses` | back-edge (gray node) in 3-color |
| Output order | natural forward order | reverse of postorder |
| Intuition | "remove zero-dependency nodes" | "finish dependents first" |

Kahn's is usually cleaner for returning the order:

```rust
use std::collections::VecDeque;

fn find_order(num_courses: i32, prerequisites: Vec<Vec<i32>>) -> Vec<i32> {
    let n = num_courses as usize;
    let mut adj: Vec<Vec<usize>> = vec![vec![]; n];
    let mut indegree = vec![0usize; n];
    for p in &prerequisites { // edge prereq p[1] -> course p[0]
        let (a, b) = (p[0] as usize, p[1] as usize);
        adj[b].push(a);
        indegree[a] += 1;
    }
    let mut q: VecDeque<usize> = VecDeque::new();
    for i in 0..n {
        if indegree[i] == 0 { q.push_back(i); }
    }
    let mut order: Vec<i32> = Vec::new();
    while let Some(node) = q.pop_front() {
        order.push(node as i32);
        for &next in &adj[node] {
            indegree[next] -= 1;
            if indegree[next] == 0 { q.push_back(next); }
        }
    }
    if order.len() == n { order } else { vec![] } // cycle ⇒ not all processed
}
```

### Edge cases
- No prerequisites → any order (here: `0..n-1`).
- Self-loop `[i,i]` → cycle → `[]`.
- Disconnected DAG components → all still get ordered.

### Follow-ups
- **Course Schedule I** — feasibility only.
- **Alien Dictionary** — build the graph first, then this exact topo-sort.
- **Parallel Courses** — minimum semesters = number of BFS levels.

---

## 4. Word Ladder (LC 127)

Shortest transformation length from `beginWord` to `endWord`.

### Wildcard bucket optimization

Naively, generating neighbors of a word means trying all 26 letters at each of L positions and checking membership — fine, but pairwise comparison across the list is O(N²·L). Instead **bucket by wildcard pattern**: `hot → *ot, h*t, ho*`. Words sharing a bucket differ by exactly one letter, giving O(N·L) adjacency.

```rust
use std::collections::{HashMap, HashSet, VecDeque};

fn ladder_length(begin_word: String, end_word: String, word_list: Vec<String>) -> i32 {
    let dict: HashSet<String> = word_list.into_iter().collect();
    if !dict.contains(&end_word) { return 0; }
    let mut buckets: HashMap<String, Vec<String>> = HashMap::new();
    for w in &dict {
        for i in 0..w.len() {
            let pattern = format!("{}*{}", &w[..i], &w[i+1..]);
            buckets.entry(pattern).or_default().push(w.clone());
        }
    }
    let mut q: VecDeque<String> = VecDeque::new();
    let mut visited: HashSet<String> = HashSet::new();
    q.push_back(begin_word.clone());
    visited.insert(begin_word);
    let mut steps = 1;
    while !q.is_empty() {
        let size = q.len();
        for _ in 0..size {
            let word = q.pop_front().unwrap();
            if word == end_word { return steps; }
            for i in 0..word.len() {
                let pattern = format!("{}*{}", &word[..i], &word[i+1..]);
                if let Some(neighbors) = buckets.get(&pattern) {
                    for next in neighbors.clone() {
                        if !visited.contains(&next) {
                            visited.insert(next.clone());
                            q.push_back(next); // visited at enqueue
                        }
                    }
                }
            }
        }
        steps += 1;
    }
    0
}
```

### Edge cases
- `endWord` not in dictionary → `0`.
- `beginWord` need not be in the dictionary.
- Words all distinct length assumed (LC guarantees equal length).

### Follow-ups
- **Word Ladder II** — return all shortest paths (BFS layers + parent backtrack); see [Google deep dive](./Google.md).
- **Bidirectional BFS** — search from both ends, swap to the smaller frontier; large dictionaries see big speedups.

---

## Amazon Leadership Principle Alignment

| LP | Demonstrated by |
|----|-----------------|
| **Customer Obsession** | Clarifying input format and failure semantics (`-1`, `[]`) up front |
| **Dive Deep** | Explaining enqueue-time marking, multi-source seeding, bucket adjacency |
| **Bias for Action** | Reaching a correct standard template quickly |
| **Insist on Highest Standards** | Handling degenerate grids, self-loops, unreachable cells |
| **Are Right, A Lot** | Choosing BFS vs DFS vs DSU with justification |
| **Invent and Simplify** | Wildcard buckets, in-place marking |

> **Last Updated:** 2026-06-26
