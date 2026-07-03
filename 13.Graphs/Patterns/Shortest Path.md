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

> **Critical pitfall:** when ordering a `std::priority_queue` with a custom comparator, NEVER use `a[1] - b[1]` as the comparison value. The subtraction can overflow for large weights and silently corrupt the heap order. Always use explicit `<` or `>` comparisons.

---

## Dijkstra Template

Greedy single-source shortest path for **non-negative** weights. Maintain a `dist[]` array, push `{dist, node}` onto a min-heap, and pop the closest unfinished node. Because we may push the same node multiple times, **skip stale entries** (a popped distance larger than the recorded `dist[node]`).

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> dijkstra(int n, vector<vector<pair<int,int>>>& adj, int src) {
    // adj[u] holds entries {v, weight}
    vector<int> dist(n, INT_MAX);
    dist[src] = 0;

    // min-heap ordered by distance; {dist, node}
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<pair<int,int>>> pq;
    pq.push({0, src});

    while (!pq.empty()) {
        auto [d, u] = pq.top(); pq.pop();

        // skip stale entry: a better distance was already finalized
        if (d > dist[u]) continue;

        for (auto& [v, w] : adj[u]) {
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                pq.push({dist[v], v});
            }
        }
    }
    return dist;
}
```

---

## Bellman-Ford Template

Handles **negative** edge weights and detects negative cycles. Relax every edge `V-1` times: after `i` rounds, all shortest paths using at most `i` edges are correct. A `V`-th round that still relaxes something proves a negative cycle.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> bellmanFord(int n, vector<vector<int>>& edges, int src) {
    // edges[i] = {u, v, w}
    vector<int> dist(n, INT_MAX);
    dist[src] = 0;

    // V - 1 relaxation rounds
    for (int i = 0; i < n - 1; i++) {
        for (auto& e : edges) {
            int u = e[0], v = e[1], w = e[2];
            if (dist[u] != INT_MAX && dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
            }
        }
    }

    // optional: negative-cycle check
    for (auto& e : edges) {
        int u = e[0], v = e[1], w = e[2];
        if (dist[u] != INT_MAX && dist[u] + w < dist[v]) {
            // negative cycle reachable from src
        }
    }
    return dist;
}
```

---

## Floyd-Warshall Template

All-pairs shortest path via dynamic programming. `dist[i][j]` is iteratively improved by allowing each vertex `k` as an intermediate. The triple loop is `O(V³)` — the `k` loop **must** be outermost.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<vector<int>> floydWarshall(int n, vector<vector<int>>& edges) {
    const int INF = 1'000'000'000;
    vector<vector<int>> dist(n, vector<int>(n, INF));
    for (int i = 0; i < n; i++) dist[i][i] = 0;

    for (auto& e : edges) {
        dist[e[0]][e[1]] = min(dist[e[0]][e[1]], e[2]);
        dist[e[1]][e[0]] = min(dist[e[1]][e[0]], e[2]); // drop if directed
    }

    // k outermost: "can we route i->j through k?"
    for (int k = 0; k < n; k++) {
        for (int i = 0; i < n; i++) {
            if (dist[i][k] == INF) continue;          // micro-optimization
            for (int j = 0; j < n; j++) {
                if (dist[i][k] + dist[k][j] < dist[i][j]) {
                    dist[i][j] = dist[i][k] + dist[k][j];
                }
            }
        }
    }
    return dist;
}
```

---

## 0-1 BFS Template

When every edge costs **0 or 1**, a plain deque beats Dijkstra's log factor. Push 0-weight relaxations to the **front** (`push_front`) and 1-weight ones to the **back** (`push_back`). The deque stays sorted by distance, so the first time you pop a node its distance is final.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> zeroOneBfs(int n, vector<vector<pair<int,int>>>& adj, int src) {
    // adj[u] holds entries {v, weight in {0,1}}
    vector<int> dist(n, INT_MAX);
    dist[src] = 0;

    deque<pair<int,int>> dq;  // {node, dist}
    dq.push_front({src, 0});

    while (!dq.empty()) {
        auto [u, d] = dq.front(); dq.pop_front();
        if (d > dist[u]) continue;          // stale

        for (auto& [v, w] : adj[u]) {
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                if (w == 0) dq.push_front({v, dist[v]});
                else        dq.push_back({v, dist[v]});
            }
        }
    }
    return dist;
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

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    int networkDelayTime(vector<vector<int>>& times, int n, int k) {
        vector<vector<pair<int,int>>> adj(n + 1);
        for (auto& t : times) adj[t[0]].push_back({t[1], t[2]});

        vector<int> dist(n + 1, INT_MAX);
        dist[k] = 0;

        priority_queue<pair<int,int>, vector<pair<int,int>>, greater<pair<int,int>>> pq;
        pq.push({0, k});

        while (!pq.empty()) {
            auto [d, u] = pq.top(); pq.pop();
            if (d > dist[u]) continue;            // stale entry

            for (auto& [v, w] : adj[u]) {
                if (dist[u] + w < dist[v]) {
                    dist[v] = dist[u] + w;
                    pq.push({dist[v], v});
                }
            }
        }

        int ans = 0;
        for (int i = 1; i <= n; i++) {
            if (dist[i] == INT_MAX) return -1;   // unreachable
            ans = max(ans, dist[i]);
        }
        return ans;
    }
};
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

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    int findCheapestPrice(int n, vector<vector<int>>& flights, int src, int dst, int k) {
        const int INF = INT_MAX;
        vector<int> dist(n, INF);
        dist[src] = 0;

        // K stops => at most K+1 edges => K+1 relaxation rounds
        for (int i = 0; i <= k; i++) {
            vector<int> prev = dist;   // freeze last round's distances
            for (auto& f : flights) {
                int u = f[0], v = f[1], w = f[2];
                if (prev[u] != INF && prev[u] + w < dist[v]) {
                    dist[v] = prev[u] + w;
                }
            }
        }
        return dist[dst] == INF ? -1 : dist[dst];
    }
};
```

**Why clone:** without the snapshot, relaxing edge A then edge B in the same round could chain two new edges, sneaking in an extra stop beyond `K`.

**Complexity:** Time `O(K · E)`, Space `O(V)`.

---

## 28. Path With Minimum Effort — Medium (LC1631)

**Idea:** Dijkstra on the grid where the "distance" of a path is the **maximum** absolute height difference between adjacent cells along it (minimax). Relaxation uses `max(currentEffort, |h - hNeighbor|)`.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    int minimumEffortPath(vector<vector<int>>& heights) {
        int rows = heights.size(), cols = heights[0].size();
        vector<vector<int>> dirs = {{0, 1}, {1, 0}, {0, -1}, {-1, 0}};

        vector<vector<int>> effort(rows, vector<int>(cols, INT_MAX));
        effort[0][0] = 0;

        // {effortSoFar, row, col}
        priority_queue<tuple<int,int,int>, vector<tuple<int,int,int>>, greater<tuple<int,int,int>>> pq;
        pq.push({0, 0, 0});

        while (!pq.empty()) {
            auto [e, r, c] = pq.top(); pq.pop();
            if (e > effort[r][c]) continue;                 // stale
            if (r == rows - 1 && c == cols - 1) return e;    // first pop is optimal

            for (auto& d : dirs) {
                int nr = r + d[0], nc = c + d[1];
                if (nr < 0 || nr >= rows || nc < 0 || nc >= cols) continue;
                int ne = max(e, abs(heights[nr][nc] - heights[r][c]));
                if (ne < effort[nr][nc]) {
                    effort[nr][nc] = ne;
                    pq.push({ne, nr, nc});
                }
            }
        }
        return 0; // single cell grid
    }
};
```

**Complexity:** Time `O(R·C·log(R·C))`, Space `O(R·C)`.

---

## 29. Swim in Rising Water — Hard (LC778)

**Idea:** The time to reach the bottom-right equals the **minimum over all paths of the maximum elevation** on that path — another minimax Dijkstra. The cost of stepping onto a cell is its elevation; the path cost is the max elevation seen.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    int swimInWater(vector<vector<int>>& grid) {
        int n = grid.size();
        vector<vector<int>> dirs = {{0, 1}, {1, 0}, {0, -1}, {-1, 0}};

        vector<vector<bool>> visited(n, vector<bool>(n, false));
        // {maxElevationSoFar, row, col}
        priority_queue<tuple<int,int,int>, vector<tuple<int,int,int>>, greater<tuple<int,int,int>>> pq;
        pq.push({grid[0][0], 0, 0});

        while (!pq.empty()) {
            auto [t, r, c] = pq.top(); pq.pop();
            if (visited[r][c]) continue;
            visited[r][c] = true;
            if (r == n - 1 && c == n - 1) return t;   // first arrival is optimal

            for (auto& d : dirs) {
                int nr = r + d[0], nc = c + d[1];
                if (nr < 0 || nr >= n || nc < 0 || nc >= n || visited[nr][nc]) continue;
                pq.push({max(t, grid[nr][nc]), nr, nc});
            }
        }
        return -1; // unreachable (won't happen on valid input)
    }
};
```

**Complexity:** Time `O(N²·log N)`, Space `O(N²)`.

---

## 30. Path with Maximum Probability — Medium (LC1514)

**Idea:** Dijkstra with a **max-heap**, maximizing the **product** of edge probabilities. Since probabilities are in `[0,1]`, products only shrink, so the greedy "pop the largest first" property holds just like minimizing a sum.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    double maxProbability(int n, vector<vector<int>>& edges, vector<double>& succProb,
                          int start, int end) {
        vector<vector<pair<int,double>>> adj(n);
        for (int i = 0; i < (int)edges.size(); i++) {
            int u = edges[i][0], v = edges[i][1];
            double p = succProb[i];
            adj[u].push_back({v, p});
            adj[v].push_back({u, p});
        }

        vector<double> prob(n, 0.0);
        prob[start] = 1.0;

        // max-heap by probability; {prob, node}
        priority_queue<pair<double,int>> pq;
        pq.push({1.0, start});

        while (!pq.empty()) {
            auto [p, u] = pq.top(); pq.pop();
            if (p < prob[u]) continue;          // stale
            if (u == end) return p;             // first pop of end is optimal

            for (auto& [v, ep] : adj[u]) {
                double np = p * ep;
                if (np > prob[v]) {
                    prob[v] = np;
                    pq.push({np, v});
                }
            }
        }
        return 0.0; // end unreachable
    }
};
```

**Complexity:** Time `O(E log V)`, Space `O(V + E)`.

---

## 31. Find the City With Smallest Number of Neighbors at a Threshold Distance — Medium (LC1334)

**Idea:** Compute all-pairs shortest paths with Floyd-Warshall, then for each city count how many others lie within `distanceThreshold`; pick the city with the fewest such neighbors, breaking ties by the **largest** index.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    int findTheCity(int n, vector<vector<int>>& edges, int distanceThreshold) {
        const int INF = 1'000'000'000;
        vector<vector<int>> dist(n, vector<int>(n, INF));
        for (int i = 0; i < n; i++) dist[i][i] = 0;

        for (auto& e : edges) {
            dist[e[0]][e[1]] = e[2];
            dist[e[1]][e[0]] = e[2];   // undirected
        }

        for (int k = 0; k < n; k++)
            for (int i = 0; i < n; i++) {
                if (dist[i][k] == INF) continue;
                for (int j = 0; j < n; j++)
                    if (dist[i][k] + dist[k][j] < dist[i][j])
                        dist[i][j] = dist[i][k] + dist[k][j];
            }

        int bestCity = -1, fewest = INT_MAX;
        for (int i = 0; i < n; i++) {
            int count = 0;
            for (int j = 0; j < n; j++)
                if (i != j && dist[i][j] <= distanceThreshold) count++;
            // <= so that on ties we keep the larger index
            if (count <= fewest) {
                fewest = count;
                bestCity = i;
            }
        }
        return bestCity;
    }
};
```

**Complexity:** Time `O(V³)`, Space `O(V²)`.

---

## 32. Minimum Cost to Make at Least One Valid Path in a Grid — Hard (LC1368)

**Idea:** 0-1 BFS. Each cell points a direction; **following** the arrow costs `0`, **changing** it (moving any other direction) costs `1`. Find the minimum-cost path from top-left to bottom-right.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    int minCost(vector<vector<int>>& grid) {
        int rows = grid.size(), cols = grid[0].size();
        // grid value 1=right, 2=left, 3=down, 4=up -> matching dirs index
        vector<vector<int>> dirs = {{0, 1}, {0, -1}, {1, 0}, {-1, 0}};

        vector<vector<int>> dist(rows, vector<int>(cols, INT_MAX));
        dist[0][0] = 0;

        deque<tuple<int,int,int>> dq;   // {row, col, cost}
        dq.push_front({0, 0, 0});

        while (!dq.empty()) {
            auto [r, c, cost] = dq.front(); dq.pop_front();
            if (cost > dist[r][c]) continue;            // stale

            for (int dir = 0; dir < 4; dir++) {
                int nr = r + dirs[dir][0], nc = c + dirs[dir][1];
                if (nr < 0 || nr >= rows || nc < 0 || nc >= cols) continue;
                // grid is 1-indexed direction; arrow already points dir -> cost 0
                int w = (grid[r][c] == dir + 1) ? 0 : 1;
                if (cost + w < dist[nr][nc]) {
                    dist[nr][nc] = cost + w;
                    if (w == 0) dq.push_front({nr, nc, cost});
                    else        dq.push_back({nr, nc, cost + 1});
                }
            }
        }
        return dist[rows - 1][cols - 1];
    }
};
```

**Complexity:** Time `O(R·C)`, Space `O(R·C)`.

---

## 33. Shortest Path Visiting All Nodes — Hard (LC847)

**Idea:** Unweighted shortest path over a **state graph** `(node, visitedMask)`. BFS layer by layer; the goal is any node whose mask equals all-ones (`(1<<n)-1`). Because `n <= 12`, the total state count `n · 2^n` is small.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    int shortestPathLength(vector<vector<int>>& graph) {
        int n = graph.size();
        if (n == 1) return 0;
        int full = (1 << n) - 1;

        vector<vector<bool>> visited(n, vector<bool>(1 << n, false));
        deque<pair<int,int>> queue;   // {node, mask}

        // can start from any node
        for (int i = 0; i < n; i++) {
            queue.push_back({i, 1 << i});
            visited[i][1 << i] = true;
        }

        int steps = 0;
        while (!queue.empty()) {
            int size = queue.size();
            for (int s = 0; s < size; s++) {
                auto [node, mask] = queue.front(); queue.pop_front();
                if (mask == full) return steps;     // all nodes visited

                for (int next : graph[node]) {
                    int nextMask = mask | (1 << next);
                    if (!visited[next][nextMask]) {
                        visited[next][nextMask] = true;
                        queue.push_back({next, nextMask});
                    }
                }
            }
            steps++;
        }
        return -1; // graph guaranteed connected, so unreachable
    }
};
```

**Complexity:** Time `O(n² · 2ⁿ)`, Space `O(n · 2ⁿ)`.

---

> **Last Updated:** 2026-06-26
