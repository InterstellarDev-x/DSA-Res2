> **Topic:** [Graphs](../README.md) · **Pattern 2 of 7**

# Grid and Matrix BFS

A 2D grid is one of the most common ways a graph shows up in disguise. Every cell is a node, and edges connect a cell to its orthogonal (or diagonal) neighbors. Because the edges are *implicit* — derived from coordinates rather than stored in an adjacency list — we never build a graph object at all. We just walk neighbors on demand using a **directions array**.

This pattern focuses on **Breadth-First Search (BFS)** over grids, with a heavy emphasis on **multi-source BFS**, a technique that quietly solves a whole family of "spread / shortest-distance-from-nearest-source" problems.

---

## Core Concept: The Grid as an Implicit Graph

In an explicit graph you have `vector<int> adj[]`. In a grid you have nothing but `grid[r][c]`. The "neighbors" of cell `(r, c)` are computed:

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

```cpp
#include <bits/stdc++.h>
using namespace std;

queue<pair<int,int>> q;
// 1. seed every source
for (each source cell s) {
    q.push(s);
    visited[s] = true;        // or write distance 0
}
// 2. expand level by level
int level = 0;
while (!q.empty()) {
    int size = q.size();      // freeze this level's frontier
    for (int i = 0; i < size; i++) {
        auto [r, c] = q.front(); q.pop();
        for (auto& d : dirs) {
            int nr = r + d[0], nc = c + d[1];
            if (inBounds(nr, nc) && !visited[nr][nc] && valid(nr, nc)) {
                visited[nr][nc] = true;
                q.push({nr, nc});
            }
        }
    }
    level++;
}
```

Marking a cell as visited **at the moment you enqueue it** (not when you dequeue it) is essential to avoid the same cell being added multiple times by different sources in the same wave.

---

## Directions Arrays

The single most reused idiom in grid problems. Iterating over a small array keeps the four (or eight) neighbor checks DRY.

**4-directional** (up / down / left / right):

```cpp
vector<vector<int>> dirs = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};
```

**8-directional** (orthogonal + diagonals) — required for *Shortest Path in Binary Matrix*, where diagonal moves are legal:

```cpp
vector<vector<int>> dirs8 = {
    {1, 0}, {-1, 0}, {0, 1}, {0, -1},
    {1, 1}, {1, -1}, {-1, 1}, {-1, -1}
};
```

Usage:

```cpp
for (auto& d : dirs) {
    int nr = r + d[0], nc = c + d[1];
    if (nr < 0 || nr >= rows || nc < 0 || nc >= cols) continue;
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

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    int orangesRotting(vector<vector<int>>& grid) {
        int rows = grid.size(), cols = grid[0].size();
        vector<vector<int>> dirs = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};

        queue<pair<int,int>> q;
        int fresh = 0;

        // Seed all initially rotten oranges (value 2) as sources at minute 0.
        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                if (grid[r][c] == 2) {
                    q.push({r, c});
                } else if (grid[r][c] == 1) {
                    fresh++;
                }
            }
        }

        // No fresh oranges at all -> zero minutes needed.
        if (fresh == 0) return 0;

        int minutes = 0;
        while (!q.empty()) {
            int size = q.size();   // all oranges rotting this minute
            bool rottedThisMinute = false;

            for (int i = 0; i < size; i++) {
                auto [r, c] = q.front(); q.pop();

                for (auto& d : dirs) {
                    int nr = r + d[0], nc = c + d[1];
                    if (nr < 0 || nr >= rows || nc < 0 || nc >= cols) continue;
                    if (grid[nr][nc] != 1) continue;   // skip empty or already rotten

                    grid[nr][nc] = 2;                  // rot it (also marks visited)
                    fresh--;
                    q.push({nr, nc});
                    rottedThisMinute = true;
                }
            }
            if (rottedThisMinute) minutes++;
        }

        // If any fresh orange remains, some were isolated and can never rot.
        return fresh == 0 ? minutes : -1;
    }
};
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

`rottedThisMinute = true` → `minutes = 1`. Grid now:
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

**Minute 5 attempt** — `size = 1`. Pop `(2,2)`: no fresh neighbors. `rottedThisMinute = false`, so `minutes` stays `4`. Queue empties.

`fresh == 0` → return **4**.

---

## 12. 01 Matrix — Medium (LC542)

**Idea:** For each cell, find the distance to the nearest `0`. Run multi-source BFS seeded from **all** `0` cells at once; the BFS level at which a `1` is first reached is its nearest-zero distance.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    vector<vector<int>> updateMatrix(vector<vector<int>>& mat) {
        int rows = mat.size(), cols = mat[0].size();
        vector<vector<int>> dist(rows, vector<int>(cols, 0));
        vector<vector<bool>> visited(rows, vector<bool>(cols, false));
        vector<vector<int>> dirs = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};

        queue<pair<int,int>> q;

        // Seed every 0 as a source (distance 0).
        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                if (mat[r][c] == 0) {
                    q.push({r, c});
                    visited[r][c] = true;
                }
            }
        }

        while (!q.empty()) {
            auto [r, c] = q.front(); q.pop();

            for (auto& d : dirs) {
                int nr = r + d[0], nc = c + d[1];
                if (nr < 0 || nr >= rows || nc < 0 || nc >= cols) continue;
                if (visited[nr][nc]) continue;

                dist[nr][nc] = dist[r][c] + 1;   // one step farther from source
                visited[nr][nc] = true;
                q.push({nr, nc});
            }
        }

        return dist;
    }
};
```

**Complexity:** Time `O(rows × cols)`. Space `O(rows × cols)` for the queue and visited/distance arrays.

---

## 13. Walls and Gates — Medium (LC286)

**Idea:** Fill each empty room with the distance to its nearest gate. Gates are `0`, walls are `-1`, empty rooms are `INF = 2147483647`. Multi-source BFS from all gates; we never need a separate visited array because an empty room still holding `INF` is "unvisited".

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
    static const int INF = 2147483647;  // INT_MAX

public:
    void wallsAndGates(vector<vector<int>>& rooms) {
        if (rooms.empty()) return;
        int rows = rooms.size(), cols = rooms[0].size();
        vector<vector<int>> dirs = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};

        queue<pair<int,int>> q;

        // Seed every gate (value 0) as a source.
        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                if (rooms[r][c] == 0) {
                    q.push({r, c});
                }
            }
        }

        while (!q.empty()) {
            auto [r, c] = q.front(); q.pop();

            for (auto& d : dirs) {
                int nr = r + d[0], nc = c + d[1];
                if (nr < 0 || nr >= rows || nc < 0 || nc >= cols) continue;
                // Only update untouched empty rooms (still INF). Walls (-1)
                // and already-filled rooms are skipped automatically.
                if (rooms[nr][nc] != INF) continue;

                rooms[nr][nc] = rooms[r][c] + 1;
                q.push({nr, nc});
            }
        }
    }
};
```

**Complexity:** Time `O(rows × cols)`. Space `O(rows × cols)` for the queue worst case (grid full of gates).

---

## 14. Shortest Path in Binary Matrix — Medium (LC1091)

**Idea:** Find the shortest clear path from top-left to bottom-right where you may move in **8 directions** and only step on `0` cells. Single-source BFS with an 8-directional dirs array; path length counts cells visited (start counts as 1).

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    int shortestPathBinaryMatrix(vector<vector<int>>& grid) {
        int n = grid.size();
        if (grid[0][0] != 0 || grid[n - 1][n - 1] != 0) return -1;

        vector<vector<int>> dirs = {
            {1, 0}, {-1, 0}, {0, 1}, {0, -1},
            {1, 1}, {1, -1}, {-1, 1}, {-1, -1}
        };

        queue<pair<int,int>> q;
        q.push({0, 0});
        grid[0][0] = 1;            // mark visited by reusing the grid; start path length = 1
        int pathLen = 1;

        while (!q.empty()) {
            int size = q.size();
            for (int i = 0; i < size; i++) {
                auto [r, c] = q.front(); q.pop();

                if (r == n - 1 && c == n - 1) return pathLen;

                for (auto& d : dirs) {
                    int nr = r + d[0], nc = c + d[1];
                    if (nr < 0 || nr >= n || nc < 0 || nc >= n) continue;
                    if (grid[nr][nc] != 0) continue;   // wall (1) or already visited

                    grid[nr][nc] = 1;                  // mark visited on enqueue
                    q.push({nr, nc});
                }
            }
            pathLen++;
        }

        return -1;
    }
};
```

**Complexity:** Time `O(n²)` — each cell enqueued once, 8 neighbor checks each. Space `O(n²)` for the queue.

---

## 15. As Far from Land as Possible — Medium (LC1162)

**Idea:** Find the water cell whose distance to the nearest land is maximized. Multi-source BFS seeded from **all land** cells; the BFS expands outward, and the *last* level reached is the maximum land-to-water distance.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    int maxDistance(vector<vector<int>>& grid) {
        int n = grid.size();
        vector<vector<int>> dirs = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};

        queue<pair<int,int>> q;
        int landCount = 0, waterCount = 0;

        // Seed all land cells (value 1) as sources.
        for (int r = 0; r < n; r++) {
            for (int c = 0; c < n; c++) {
                if (grid[r][c] == 1) {
                    q.push({r, c});
                    landCount++;
                } else {
                    waterCount++;
                }
            }
        }

        // Edge case: all land or all water -> no valid answer.
        if (landCount == 0 || waterCount == 0) return -1;

        int distance = -1;
        while (!q.empty()) {
            int size = q.size();
            distance++;   // each full level pushes the frontier one step into the water

            for (int i = 0; i < size; i++) {
                auto [r, c] = q.front(); q.pop();

                for (auto& d : dirs) {
                    int nr = r + d[0], nc = c + d[1];
                    if (nr < 0 || nr >= n || nc < 0 || nc >= n) continue;
                    if (grid[nr][nc] != 0) continue;   // skip land / already visited

                    grid[nr][nc] = 1;   // mark this water cell as reached
                    q.push({nr, nc});
                }
            }
        }

        return distance;   // last level processed = farthest water from any land
    }
};
```

**Complexity:** Time `O(n²)`. Space `O(n²)` for the queue.

---

## 16. Shortest Bridge — Medium (LC934)

**Idea:** There are exactly two islands. Use **DFS** to find and mark the first island (changing its cells to `2` and seeding them into a queue), then run **multi-source BFS** outward through water until it touches the second island. The number of BFS levels is the shortest bridge length.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
    vector<vector<int>> dirs = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};

public:
    int shortestBridge(vector<vector<int>>& grid) {
        int n = grid.size();
        queue<pair<int,int>> q;

        // Step 1: locate the first island, mark it (1 -> 2) and seed its cells.
        bool found = false;
        for (int r = 0; r < n && !found; r++) {
            for (int c = 0; c < n && !found; c++) {
                if (grid[r][c] == 1) {
                    dfsMark(grid, r, c, q);
                    found = true;
                }
            }
        }

        // Step 2: BFS outward from the entire first island simultaneously.
        int steps = 0;
        while (!q.empty()) {
            int size = q.size();
            for (int i = 0; i < size; i++) {
                auto [r, c] = q.front(); q.pop();

                for (auto& d : dirs) {
                    int nr = r + d[0], nc = c + d[1];
                    if (nr < 0 || nr >= n || nc < 0 || nc >= n) continue;

                    if (grid[nr][nc] == 1) return steps;  // reached the second island
                    if (grid[nr][nc] == 0) {              // water: extend the bridge
                        grid[nr][nc] = 2;                 // mark visited
                        q.push({nr, nc});
                    }
                    // grid[nr][nc] == 2 -> already part of island/visited, skip
                }
            }
            steps++;
        }

        return -1;  // unreachable in valid input
    }

private:
    void dfsMark(vector<vector<int>>& grid, int r, int c, queue<pair<int,int>>& q) {
        int n = grid.size();
        if (r < 0 || r >= n || c < 0 || c >= n || grid[r][c] != 1) return;
        grid[r][c] = 2;                   // mark as visited / first island
        q.push({r, c});
        for (auto& d : dirs) {
            dfsMark(grid, r + d[0], c + d[1], q);
        }
    }
};
```

**Complexity:** Time `O(n²)` — DFS touches the first island once, BFS touches each cell once. Space `O(n²)` for the recursion stack and queue.

---

## 17. Number of Distinct Islands — Medium (LC694)

**Idea:** Count islands that are distinct in **shape** (translations are the same, rotations/reflections are not). DFS each island while recording the **relative path signature** — the sequence of moves taken from the island's entry cell. Two islands with the same signature have the same shape. Store signatures in an `unordered_set`.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
    vector<vector<int>> dirs = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};
    string dirChar = "DURL";  // aligned with dirs

public:
    int numDistinctIslands(vector<vector<int>>& grid) {
        int rows = grid.size(), cols = grid[0].size();
        unordered_set<string> shapes;

        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                if (grid[r][c] == 1) {
                    string path;
                    dfs(grid, r, c, 'S', path);   // 'S' = start of this island
                    shapes.insert(path);
                }
            }
        }

        return shapes.size();
    }

private:
    void dfs(vector<vector<int>>& grid, int r, int c, char move, string& path) {
        int rows = grid.size(), cols = grid[0].size();
        if (r < 0 || r >= rows || c < 0 || c >= cols || grid[r][c] != 1) return;

        grid[r][c] = 0;          // mark visited
        path += move;            // record how we entered this cell

        for (int i = 0; i < (int)dirs.size(); i++) {
            dfs(grid, r + dirs[i][0], c + dirs[i][1], dirChar[i], path);
        }

        // Append a backtrack marker so different shapes can't collide via concatenation.
        path += 'B';
    }
};
```

The `'B'` backtrack marker is important: without it, shapes that branch differently could produce identical strings. By recording both forward moves and backtracks, the signature uniquely encodes the traversal tree, and identical shapes always yield identical signatures because the DFS visits directions in a fixed order.

**Complexity:** Time `O(rows × cols)` for the traversal. Space `O(rows × cols)` for recursion plus the set of signature strings.

---

## 18. Making A Large Island — Hard (LC827)

**Idea:** You may flip at most one `0` to `1`. Maximize the resulting island size. First, **label every island with a unique id** and record each island's size in a map. Then, for each `0` cell, look at its distinct neighboring island ids and sum their sizes plus one (the flipped cell). The best such sum is the answer. If there are no `0` cells, the whole grid is already one island.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
    vector<vector<int>> dirs = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};

public:
    int largestIsland(vector<vector<int>>& grid) {
        int n = grid.size();
        unordered_map<int, int> sizeById;
        int id = 2;   // start ids at 2 (0 = water, 1 = unlabeled land)

        // Pass 1: label each island and store its size.
        for (int r = 0; r < n; r++) {
            for (int c = 0; c < n; c++) {
                if (grid[r][c] == 1) {
                    int size = dfsLabel(grid, r, c, id);
                    sizeById[id] = size;
                    id++;
                }
            }
        }

        // If the grid was entirely land, there is no 0 to flip.
        int best = 0;
        for (auto& [k, v] : sizeById) best = max(best, v);

        // Pass 2: try flipping each water cell and sum distinct neighbor islands.
        for (int r = 0; r < n; r++) {
            for (int c = 0; c < n; c++) {
                if (grid[r][c] == 0) {
                    unordered_set<int> seen;
                    int combined = 1;   // the flipped cell itself
                    for (auto& d : dirs) {
                        int nr = r + d[0], nc = c + d[1];
                        if (nr < 0 || nr >= n || nc < 0 || nc >= n) continue;
                        int neighborId = grid[nr][nc];
                        if (neighborId > 1 && seen.find(neighborId) == seen.end()) {
                            seen.insert(neighborId);
                            combined += sizeById[neighborId];
                        }
                    }
                    best = max(best, combined);
                }
            }
        }

        return best;
    }

private:
    int dfsLabel(vector<vector<int>>& grid, int r, int c, int id) {
        int n = grid.size();
        if (r < 0 || r >= n || c < 0 || c >= n || grid[r][c] != 1) return 0;
        grid[r][c] = id;   // overwrite land with its island id
        int size = 1;
        for (auto& d : dirs) {
            size += dfsLabel(grid, r + d[0], c + d[1], id);
        }
        return size;
    }
};
```

Using an `unordered_set` of neighbor ids per `0` cell prevents double-counting when two of the four neighbors belong to the *same* island. Labeling with ids (rather than re-flooding for every candidate flip) is what keeps this efficient.

**Complexity:** Time `O(n²)` — one labeling pass and one flip-evaluation pass, each touching every cell a constant number of times. Space `O(n²)` for the recursion stack and the size map.

> **Last Updated:** 2026-06-26
