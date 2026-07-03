> **Topic:** [Graphs](../README.md) · **Pattern 2 of 7**

# Grid and Matrix BFS

A 2D grid is one of the most common ways a graph shows up in disguise. Every cell is a node, and edges connect a cell to its orthogonal (or diagonal) neighbors. Because the edges are *implicit* — derived from coordinates rather than stored in an adjacency list — we never build a graph object at all. We just walk neighbors on demand using a **directions array**.

This pattern focuses on **Breadth-First Search (BFS)** over grids, with a heavy emphasis on **multi-source BFS**, a technique that quietly solves a whole family of "spread / shortest-distance-from-nearest-source" problems.

---

## Core Concept: The Grid as an Implicit Graph

In an explicit graph you have `Vec<Vec<i32>>` (adjacency list). In a grid you have nothing but `grid[r][c]`. The "neighbors" of cell `(r, c)` are computed:

```
(r+1, c), (r-1, c), (r, c+1), (r, c-1)   // 4-directional
```

A neighbor is valid only if it stays inside bounds and (usually) satisfies some condition (it's land, it's empty, it's unvisited, etc.).

### Single-source BFS

Standard BFS: push one start cell, expand outward level by level. The level number equals the shortest distance (in number of steps) from the start, because every edge has weight 1.

### Multi-source BFS — the key idea

Sometimes you want the shortest distance from **the nearest of many sources**, or you want to know how something **spreads simultaneously** from many origins. Naively running a BFS per source is `O(sources × cells)`. Instead:

> **Seed ALL sources into the queue first (all at distance 0 / minute 0), then run a single BFS.**

Because all sources start at level 0 together, the BFS naturally computes, for every cell, the distance to its *closest* source. The frontier expands like multiple ripples meeting in the middle. This is the engine behind:

- **Rotting Oranges** — all initially rotten oranges are sources; the answer is the number of BFS levels.
- **01 Matrix** — all `0` cells are sources; each `1` gets its distance to the nearest `0`.
- **Walls and Gates** — all gates (`0`) are sources; each empty room gets distance to nearest gate.
- **As Far from Land as Possible** — all land cells are sources; the last water cell reached is the farthest.

The skeleton:

```rust
use std::collections::VecDeque;

let mut q: VecDeque<(i32, i32)> = VecDeque::new();
// 1. seed every source
for s in source_cells {
    q.push_back(s);
    visited[s.0 as usize][s.1 as usize] = true;  // or write distance 0
}
// 2. expand level by level
let mut level = 0;
while !q.is_empty() {
    let size = q.len();      // freeze this level's frontier
    for _ in 0..size {
        let (r, c) = q.pop_front().unwrap();
        for d in &dirs {
            let nr = r + d[0];
            let nc = c + d[1];
            if in_bounds(nr, nc) && !visited[nr as usize][nc as usize] && valid(nr, nc) {
                visited[nr as usize][nc as usize] = true;
                q.push_back((nr, nc));
            }
        }
    }
    level += 1;
}
```

Marking a cell as visited **at the moment you enqueue it** (not when you dequeue it) is essential to avoid the same cell being added multiple times by different sources in the same wave.

---

## Directions Arrays

The single most reused idiom in grid problems. Iterating over a small array keeps the four (or eight) neighbor checks DRY.

**4-directional** (up / down / left / right):

```rust
let dirs = [[1i32, 0], [-1, 0], [0, 1], [0, -1]];
```

**8-directional** (orthogonal + diagonals) — required for *Shortest Path in Binary Matrix*, where diagonal moves are legal:

```rust
let dirs8 = [
    [1i32, 0], [-1, 0], [0, 1], [0, -1],
    [1, 1], [1, -1], [-1, 1], [-1, -1]
];
```

Usage:

```rust
for d in &dirs {
    let nr = r + d[0];
    let nc = c + d[1];
    if nr < 0 || nr >= rows || nc < 0 || nc >= cols { continue; }
    // ... process (nr, nc)
}
```

---

## Recognition Signals

| Signal in the problem | What it suggests |
|---|---|
| "Shortest path / minimum steps" on an unweighted grid | BFS (level = distance) |
| "Distance to nearest X for every cell" | Multi-source BFS from all X |
| "Spreads / rots / infects each minute simultaneously" | Multi-source BFS, answer = #levels |
| "Can move in 8 directions" | 8-directional `dirs` |
| "Count islands / connected regions" | DFS or BFS flood fill |
| "Shape of an island matters" | DFS + normalized shape signature |
| "Connect two regions with fewest changes" | Find one region (DFS) then BFS outward |
| "Flip one cell to maximize a region" | Label regions with ids + size map, test each flip |
| Maximize the minimum distance ("as far as possible") | Multi-source BFS, answer = last level reached |

---

## 11. Rotting Oranges — Medium (LC994)

**Idea:** Every rotten orange is a BFS source. Each minute, all rotten oranges simultaneously rot their fresh neighbors — exactly a multi-source level-order BFS. The answer is the number of levels; return `-1` if any fresh orange is unreachable.

```rust
use std::collections::VecDeque;

struct Solution;

impl Solution {
    pub fn oranges_rotting(mut grid: Vec<Vec<i32>>) -> i32 {
        let rows = grid.len();
        let cols = grid[0].len();
        let dirs = [[1i32, 0], [-1, 0], [0, 1], [0, -1]];

        let mut q: VecDeque<(i32, i32)> = VecDeque::new();
        let mut fresh = 0;

        // Seed all initially rotten oranges (value 2) as sources at minute 0.
        for r in 0..rows {
            for c in 0..cols {
                if grid[r][c] == 2 {
                    q.push_back((r as i32, c as i32));
                } else if grid[r][c] == 1 {
                    fresh += 1;
                }
            }
        }

        // No fresh oranges at all -> zero minutes needed.
        if fresh == 0 { return 0; }

        let mut minutes = 0;
        while !q.is_empty() {
            let size = q.len();   // all oranges rotting this minute
            let mut rotted_this_minute = false;

            for _ in 0..size {
                let (r, c) = q.pop_front().unwrap();

                for d in &dirs {
                    let nr = r + d[0];
                    let nc = c + d[1];
                    if nr < 0 || nr >= rows as i32 || nc < 0 || nc >= cols as i32 { continue; }
                    let (nr, nc) = (nr as usize, nc as usize);
                    if grid[nr][nc] != 1 { continue; }   // skip empty or already rotten

                    grid[nr][nc] = 2;                    // rot it (also marks visited)
                    fresh -= 1;
                    q.push_back((nr as i32, nc as i32));
                    rotted_this_minute = true;
                }
            }
            if rotted_this_minute { minutes += 1; }
        }

        // If any fresh orange remains, some were isolated and can never rot.
        if fresh == 0 { minutes } else { -1 }
    }
}
```

**Complexity:** Time `O(rows × cols)` — every cell is enqueued at most once. Space `O(rows × cols)` for the queue in the worst case (grid full of rotten oranges).

### Detailed dry-run

Grid (`2` = rotten, `1` = fresh, `0` = empty):

```
2 1 1
1 1 0
0 1 1
```

**Seeding:** Only `(0,0)` is rotten. `queue = [(0,0)]`, `fresh = 6`, `minutes = 0`.

**Minute 1** — `size = 1`. Pop `(0,0)`. Neighbors:
- `(1,0)` is `1` → rot to `2`, enqueue, `fresh = 5`.
- `(0,1)` is `1` → rot to `2`, enqueue, `fresh = 4`.

`rotted_this_minute = true` → `minutes = 1`. Grid now:
```
2 2 1
2 1 0
0 1 1
```
`queue = [(1,0), (0,1)]`.

**Minute 2** — `size = 2`. Pop `(1,0)`: neighbor `(1,1)` is `1` → rot, `fresh = 3`. (`(2,0)` is `0`, skip.) Pop `(0,1)`: neighbor `(0,2)` is `1` → rot, `fresh = 2`. (`(1,1)` already `2`.)

`minutes = 2`. Grid:
```
2 2 2
2 2 0
0 1 1
```
`queue = [(1,1), (0,2)]`.

**Minute 3** — `size = 2`. Pop `(1,1)`: neighbor `(2,1)` is `1` → rot, `fresh = 1`. Pop `(0,2)`: no fresh neighbors.

`minutes = 3`. Grid:
```
2 2 2
2 2 0
0 2 1
```
`queue = [(2,1)]`.

**Minute 4** — `size = 1`. Pop `(2,1)`: neighbor `(2,2)` is `1` → rot, `fresh = 0`.

`minutes = 4`. `queue = [(2,2)]`.

**Minute 5 attempt** — `size = 1`. Pop `(2,2)`: no fresh neighbors. `rotted_this_minute = false`, so `minutes` stays `4`. Queue empties.

`fresh == 0` → return **4**.

---

## 12. 01 Matrix — Medium (LC542)

**Idea:** For each cell, find the distance to the nearest `0`. Run multi-source BFS seeded from **all** `0` cells at once; the BFS level at which a `1` is first reached is its nearest-zero distance.

```rust
use std::collections::VecDeque;

struct Solution;

impl Solution {
    pub fn update_matrix(mat: Vec<Vec<i32>>) -> Vec<Vec<i32>> {
        let rows = mat.len();
        let cols = mat[0].len();
        let mut dist = vec![vec![0i32; cols]; rows];
        let mut visited = vec![vec![false; cols]; rows];
        let dirs = [[1i32, 0], [-1, 0], [0, 1], [0, -1]];

        let mut q: VecDeque<(i32, i32)> = VecDeque::new();

        // Seed every 0 as a source (distance 0).
        for r in 0..rows {
            for c in 0..cols {
                if mat[r][c] == 0 {
                    q.push_back((r as i32, c as i32));
                    visited[r][c] = true;
                }
            }
        }

        while !q.is_empty() {
            let (r, c) = q.pop_front().unwrap();

            for d in &dirs {
                let nr = r + d[0];
                let nc = c + d[1];
                if nr < 0 || nr >= rows as i32 || nc < 0 || nc >= cols as i32 { continue; }
                let (nr, nc) = (nr as usize, nc as usize);
                if visited[nr][nc] { continue; }

                dist[nr][nc] = dist[r as usize][c as usize] + 1;   // one step farther from source
                visited[nr][nc] = true;
                q.push_back((nr as i32, nc as i32));
            }
        }

        dist
    }
}
```

**Complexity:** Time `O(rows × cols)`. Space `O(rows × cols)` for the queue and visited/distance arrays.

---

## 13. Walls and Gates — Medium (LC286)

**Idea:** Fill each empty room with the distance to its nearest gate. Gates are `0`, walls are `-1`, empty rooms are `INF = 2147483647`. Multi-source BFS from all gates; we never need a separate visited array because an empty room still holding `INF` is "unvisited".

```rust
use std::collections::VecDeque;

struct Solution;

impl Solution {
    pub fn walls_and_gates(rooms: &mut Vec<Vec<i32>>) {
        if rooms.is_empty() { return; }
        let rows = rooms.len();
        let cols = rooms[0].len();
        let dirs = [[1i32, 0], [-1, 0], [0, 1], [0, -1]];
        const INF: i32 = i32::MAX;  // 2147483647

        let mut q: VecDeque<(i32, i32)> = VecDeque::new();

        // Seed every gate (value 0) as a source.
        for r in 0..rows {
            for c in 0..cols {
                if rooms[r][c] == 0 {
                    q.push_back((r as i32, c as i32));
                }
            }
        }

        while !q.is_empty() {
            let (r, c) = q.pop_front().unwrap();

            for d in &dirs {
                let nr = r + d[0];
                let nc = c + d[1];
                if nr < 0 || nr >= rows as i32 || nc < 0 || nc >= cols as i32 { continue; }
                let (nr, nc) = (nr as usize, nc as usize);
                // Only update untouched empty rooms (still INF). Walls (-1)
                // and already-filled rooms are skipped automatically.
                if rooms[nr][nc] != INF { continue; }

                rooms[nr][nc] = rooms[r as usize][c as usize] + 1;
                q.push_back((nr as i32, nc as i32));
            }
        }
    }
}
```

**Complexity:** Time `O(rows × cols)`. Space `O(rows × cols)` for the queue worst case (grid full of gates).

---

## 14. Shortest Path in Binary Matrix — Medium (LC1091)

**Idea:** Find the shortest clear path from top-left to bottom-right where you may move in **8 directions** and only step on `0` cells. Single-source BFS with an 8-directional dirs array; path length counts cells visited (start counts as 1).

```rust
use std::collections::VecDeque;

struct Solution;

impl Solution {
    pub fn shortest_path_binary_matrix(mut grid: Vec<Vec<i32>>) -> i32 {
        let n = grid.len();
        if grid[0][0] != 0 || grid[n - 1][n - 1] != 0 { return -1; }

        let dirs = [
            [1i32, 0], [-1, 0], [0, 1], [0, -1],
            [1, 1], [1, -1], [-1, 1], [-1, -1]
        ];

        let mut q: VecDeque<(i32, i32)> = VecDeque::new();
        q.push_back((0, 0));
        grid[0][0] = 1;            // mark visited by reusing the grid; start path length = 1
        let mut path_len = 1;

        while !q.is_empty() {
            let size = q.len();
            for _ in 0..size {
                let (r, c) = q.pop_front().unwrap();

                if r == (n - 1) as i32 && c == (n - 1) as i32 { return path_len; }

                for d in &dirs {
                    let nr = r + d[0];
                    let nc = c + d[1];
                    if nr < 0 || nr >= n as i32 || nc < 0 || nc >= n as i32 { continue; }
                    let (nr, nc) = (nr as usize, nc as usize);
                    if grid[nr][nc] != 0 { continue; }   // wall (1) or already visited

                    grid[nr][nc] = 1;                    // mark visited on enqueue
                    q.push_back((nr as i32, nc as i32));
                }
            }
            path_len += 1;
        }

        -1
    }
}
```

**Complexity:** Time `O(n²)` — each cell enqueued once, 8 neighbor checks each. Space `O(n²)` for the queue.

---

## 15. As Far from Land as Possible — Medium (LC1162)

**Idea:** Find the water cell whose distance to the nearest land is maximized. Multi-source BFS seeded from **all land** cells; the BFS expands outward, and the *last* level reached is the maximum land-to-water distance.

```rust
use std::collections::VecDeque;

struct Solution;

impl Solution {
    pub fn max_distance(mut grid: Vec<Vec<i32>>) -> i32 {
        let n = grid.len();
        let dirs = [[1i32, 0], [-1, 0], [0, 1], [0, -1]];

        let mut q: VecDeque<(i32, i32)> = VecDeque::new();
        let mut land_count = 0;
        let mut water_count = 0;

        // Seed all land cells (value 1) as sources.
        for r in 0..n {
            for c in 0..n {
                if grid[r][c] == 1 {
                    q.push_back((r as i32, c as i32));
                    land_count += 1;
                } else {
                    water_count += 1;
                }
            }
        }

        // Edge case: all land or all water -> no valid answer.
        if land_count == 0 || water_count == 0 { return -1; }

        let mut distance = -1;
        while !q.is_empty() {
            let size = q.len();
            distance += 1;   // each full level pushes the frontier one step into the water

            for _ in 0..size {
                let (r, c) = q.pop_front().unwrap();

                for d in &dirs {
                    let nr = r + d[0];
                    let nc = c + d[1];
                    if nr < 0 || nr >= n as i32 || nc < 0 || nc >= n as i32 { continue; }
                    let (nr, nc) = (nr as usize, nc as usize);
                    if grid[nr][nc] != 0 { continue; }   // skip land / already visited

                    grid[nr][nc] = 1;   // mark this water cell as reached
                    q.push_back((nr as i32, nc as i32));
                }
            }
        }

        distance   // last level processed = farthest water from any land
    }
}
```

**Complexity:** Time `O(n²)`. Space `O(n²)` for the queue.

---

## 16. Shortest Bridge — Medium (LC934)

**Idea:** There are exactly two islands. Use **DFS** to find and mark the first island (changing its cells to `2` and seeding them into a queue), then run **multi-source BFS** outward through water until it touches the second island. The number of BFS levels is the shortest bridge length.

```rust
use std::collections::VecDeque;

struct Solution;

impl Solution {
    pub fn shortest_bridge(mut grid: Vec<Vec<i32>>) -> i32 {
        let n = grid.len();
        let dirs = [[1i32, 0i32], [-1, 0], [0, 1], [0, -1]];
        let mut q: VecDeque<(i32, i32)> = VecDeque::new();

        // Step 1: locate the first island, mark it (1 -> 2) and seed its cells.
        'outer: for r in 0..n {
            for c in 0..n {
                if grid[r][c] == 1 {
                    Self::dfs_mark(&mut grid, r as i32, c as i32, n, &dirs, &mut q);
                    break 'outer;
                }
            }
        }

        // Step 2: BFS outward from the entire first island simultaneously.
        let mut steps = 0;
        while !q.is_empty() {
            let size = q.len();
            for _ in 0..size {
                let (r, c) = q.pop_front().unwrap();

                for d in &dirs {
                    let nr = r + d[0];
                    let nc = c + d[1];
                    if nr < 0 || nr >= n as i32 || nc < 0 || nc >= n as i32 { continue; }
                    let (nru, ncu) = (nr as usize, nc as usize);

                    if grid[nru][ncu] == 1 { return steps; }  // reached the second island
                    if grid[nru][ncu] == 0 {                  // water: extend the bridge
                        grid[nru][ncu] = 2;                   // mark visited
                        q.push_back((nr, nc));
                    }
                    // grid[nr][nc] == 2 -> already part of island/visited, skip
                }
            }
            steps += 1;
        }

        -1  // unreachable in valid input
    }

    fn dfs_mark(grid: &mut Vec<Vec<i32>>, r: i32, c: i32, n: usize, dirs: &[[i32; 2]; 4], q: &mut VecDeque<(i32, i32)>) {
        if r < 0 || r >= n as i32 || c < 0 || c >= n as i32 || grid[r as usize][c as usize] != 1 { return; }
        grid[r as usize][c as usize] = 2;  // mark as visited / first island
        q.push_back((r, c));
        for d in dirs {
            Self::dfs_mark(grid, r + d[0], c + d[1], n, dirs, q);
        }
    }
}
```

**Complexity:** Time `O(n²)` — DFS touches the first island once, BFS touches each cell once. Space `O(n²)` for the recursion stack and queue.

---

## 17. Number of Distinct Islands — Medium (LC694)

**Idea:** Count islands that are distinct in **shape** (translations are the same, rotations/reflections are not). DFS each island while recording the **relative path signature** — the sequence of moves taken from the island's entry cell. Two islands with the same signature have the same shape. Store signatures in a `HashSet`.

```rust
use std::collections::HashSet;

struct Solution;

impl Solution {
    pub fn num_distinct_islands(mut grid: Vec<Vec<i32>>) -> i32 {
        let rows = grid.len();
        let cols = grid[0].len();
        let dirs = [[1i32, 0i32], [-1, 0], [0, 1], [0, -1]];
        let dir_char = ['D', 'U', 'R', 'L'];  // aligned with dirs
        let mut shapes: HashSet<String> = HashSet::new();

        for r in 0..rows {
            for c in 0..cols {
                if grid[r][c] == 1 {
                    let mut path = String::new();
                    Self::dfs(&mut grid, r as i32, c as i32, 'S', &mut path, &dirs, &dir_char);   // 'S' = start of this island
                    shapes.insert(path);
                }
            }
        }

        shapes.len() as i32
    }

    fn dfs(grid: &mut Vec<Vec<i32>>, r: i32, c: i32, move_char: char, path: &mut String, dirs: &[[i32; 2]; 4], dir_char: &[char; 4]) {
        let rows = grid.len() as i32;
        let cols = grid[0].len() as i32;
        if r < 0 || r >= rows || c < 0 || c >= cols || grid[r as usize][c as usize] != 1 { return; }

        grid[r as usize][c as usize] = 0;  // mark visited
        path.push(move_char);              // record how we entered this cell

        for i in 0..dirs.len() {
            Self::dfs(grid, r + dirs[i][0], c + dirs[i][1], dir_char[i], path, dirs, dir_char);
        }

        // Append a backtrack marker so different shapes can't collide via concatenation.
        path.push('B');
    }
}
```

The `'B'` backtrack marker is important: without it, shapes that branch differently could produce identical strings. By recording both forward moves and backtracks, the signature uniquely encodes the traversal tree, and identical shapes always yield identical signatures because the DFS visits directions in a fixed order.

**Complexity:** Time `O(rows × cols)` for the traversal. Space `O(rows × cols)` for recursion plus the set of signature strings.

---

## 18. Making A Large Island — Hard (LC827)

**Idea:** You may flip at most one `0` to `1`. Maximize the resulting island size. First, **label every island with a unique id** and record each island's size in a map. Then, for each `0` cell, look at its distinct neighboring island ids and sum their sizes plus one (the flipped cell). The best such sum is the answer. If there are no `0` cells, the whole grid is already one island.

```rust
use std::collections::{HashMap, HashSet};

struct Solution;

impl Solution {
    pub fn largest_island(mut grid: Vec<Vec<i32>>) -> i32 {
        let n = grid.len();
        let dirs = [[1i32, 0i32], [-1, 0], [0, 1], [0, -1]];
        let mut size_by_id: HashMap<i32, i32> = HashMap::new();
        let mut id = 2i32;   // start ids at 2 (0 = water, 1 = unlabeled land)

        // Pass 1: label each island and store its size.
        for r in 0..n {
            for c in 0..n {
                if grid[r][c] == 1 {
                    let size = Self::dfs_label(&mut grid, r as i32, c as i32, n, id, &dirs);
                    size_by_id.insert(id, size);
                    id += 1;
                }
            }
        }

        // If the grid was entirely land, there is no 0 to flip.
        let mut best = size_by_id.values().cloned().max().unwrap_or(0);

        // Pass 2: try flipping each water cell and sum distinct neighbor islands.
        for r in 0..n {
            for c in 0..n {
                if grid[r][c] == 0 {
                    let mut seen: HashSet<i32> = HashSet::new();
                    let mut combined = 1;   // the flipped cell itself
                    for d in &dirs {
                        let nr = r as i32 + d[0];
                        let nc = c as i32 + d[1];
                        if nr < 0 || nr >= n as i32 || nc < 0 || nc >= n as i32 { continue; }
                        let neighbor_id = grid[nr as usize][nc as usize];
                        if neighbor_id > 1 && !seen.contains(&neighbor_id) {
                            seen.insert(neighbor_id);
                            combined += size_by_id.get(&neighbor_id).cloned().unwrap_or(0);
                        }
                    }
                    best = best.max(combined);
                }
            }
        }

        best
    }

    fn dfs_label(grid: &mut Vec<Vec<i32>>, r: i32, c: i32, n: usize, id: i32, dirs: &[[i32; 2]; 4]) -> i32 {
        if r < 0 || r >= n as i32 || c < 0 || c >= n as i32 || grid[r as usize][c as usize] != 1 { return 0; }
        grid[r as usize][c as usize] = id;   // overwrite land with its island id
        let mut size = 1;
        for d in dirs {
            size += Self::dfs_label(grid, r + d[0], c + d[1], n, id, dirs);
        }
        size
    }
}
```

Using a `HashSet` of neighbor ids per `0` cell prevents double-counting when two of the four neighbors belong to the *same* island. Labeling with ids (rather than re-flooding for every candidate flip) is what keeps this efficient.

**Complexity:** Time `O(n²)` — one labeling pass and one flip-evaluation pass, each touching every cell a constant number of times. Space `O(n²)` for the recursion stack and the size map.

> **Last Updated:** 2026-06-26
