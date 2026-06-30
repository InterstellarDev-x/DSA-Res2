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

> **Critical pitfall:** when ordering a `PriorityQueue<int[]>`, NEVER use `(a, b) -> a[1] - b[1]`. The subtraction can overflow for large weights and silently corrupt the heap order. Always use `Integer.compare(a[1], b[1])`.

---

## Dijkstra Template

Greedy single-source shortest path for **non-negative** weights. Maintain a `dist[]` array, push `{node, dist}` onto a min-heap, and pop the closest unfinished node. Because we may push the same node multiple times, **skip stale entries** (a popped distance larger than the recorded `dist[node]`).

```java
import java.util.*;

int[] dijkstra(int n, List<int[]>[] adj, int src) {
    // adj[u] holds entries {v, weight}
    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[src] = 0;

    // min-heap ordered by distance; {node, dist}
    PriorityQueue<int[]> pq =
        new PriorityQueue<>((a, b) -> Integer.compare(a[1], b[1]));
    pq.offer(new int[]{src, 0});

    while (!pq.isEmpty()) {
        int[] cur = pq.poll();
        int u = cur[0], d = cur[1];

        // skip stale entry: a better distance was already finalized
        if (d > dist[u]) continue;

        for (int[] edge : adj[u]) {
            int v = edge[0], w = edge[1];
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                pq.offer(new int[]{v, dist[v]});
            }
        }
    }
    return dist;
}
```

---

## Bellman-Ford Template

Handles **negative** edge weights and detects negative cycles. Relax every edge `V-1` times: after `i` rounds, all shortest paths using at most `i` edges are correct. A `V`-th round that still relaxes something proves a negative cycle.

```java
import java.util.*;

int[] bellmanFord(int n, int[][] edges, int src) {
    // edges[i] = {u, v, w}
    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[src] = 0;

    // V - 1 relaxation rounds
    for (int i = 0; i < n - 1; i++) {
        for (int[] e : edges) {
            int u = e[0], v = e[1], w = e[2];
            if (dist[u] != Integer.MAX_VALUE && dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
            }
        }
    }

    // optional: negative-cycle check
    for (int[] e : edges) {
        int u = e[0], v = e[1], w = e[2];
        if (dist[u] != Integer.MAX_VALUE && dist[u] + w < dist[v]) {
            // negative cycle reachable from src
        }
    }
    return dist;
}
```

---

## Floyd-Warshall Template

All-pairs shortest path via dynamic programming. `dist[i][j]` is iteratively improved by allowing each vertex `k` as an intermediate. The triple loop is `O(V³)` — the `k` loop **must** be outermost.

```java
import java.util.*;

int[][] floydWarshall(int n, int[][] edges) {
    final int INF = 1_000_000_000;
    int[][] dist = new int[n][n];
    for (int[] row : dist) Arrays.fill(row, INF);
    for (int i = 0; i < n; i++) dist[i][i] = 0;

    for (int[] e : edges) {
        dist[e[0]][e[1]] = Math.min(dist[e[0]][e[1]], e[2]);
        dist[e[1]][e[0]] = Math.min(dist[e[1]][e[0]], e[2]); // drop if directed
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

When every edge costs **0 or 1**, a plain deque beats Dijkstra's log factor. Push 0-weight relaxations to the **front** (`addFirst`) and 1-weight ones to the **back** (`addLast`). The deque stays sorted by distance, so the first time you pop a node its distance is final.

```java
import java.util.*;

int[] zeroOneBfs(int n, List<int[]>[] adj, int src) {
    // adj[u] holds entries {v, weight in {0,1}}
    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[src] = 0;

    Deque<int[]> dq = new ArrayDeque<>();  // {node, dist}
    dq.offerFirst(new int[]{src, 0});

    while (!dq.isEmpty()) {
        int[] cur = dq.pollFirst();
        int u = cur[0], d = cur[1];
        if (d > dist[u]) continue;          // stale

        for (int[] edge : adj[u]) {
            int v = edge[0], w = edge[1];
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                if (w == 0) dq.offerFirst(new int[]{v, dist[v]});
                else        dq.offerLast(new int[]{v, dist[v]});
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

```java
import java.util.*;

class Solution {
    public int networkDelayTime(int[][] times, int n, int k) {
        List<int[]>[] adj = new List[n + 1];
        for (int i = 1; i <= n; i++) adj[i] = new ArrayList<>();
        for (int[] t : times) adj[t[0]].add(new int[]{t[1], t[2]});

        int[] dist = new int[n + 1];
        Arrays.fill(dist, Integer.MAX_VALUE);
        dist[k] = 0;

        PriorityQueue<int[]> pq =
            new PriorityQueue<>((a, b) -> Integer.compare(a[1], b[1]));
        pq.offer(new int[]{k, 0});

        while (!pq.isEmpty()) {
            int[] cur = pq.poll();
            int u = cur[0], d = cur[1];
            if (d > dist[u]) continue;            // stale entry

            for (int[] e : adj[u]) {
                int v = e[0], w = e[1];
                if (dist[u] + w < dist[v]) {
                    dist[v] = dist[u] + w;
                    pq.offer(new int[]{v, dist[v]});
                }
            }
        }

        int ans = 0;
        for (int i = 1; i <= n; i++) {
            if (dist[i] == Integer.MAX_VALUE) return -1;   // unreachable
            ans = Math.max(ans, dist[i]);
        }
        return ans;
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

```java
import java.util.*;

class Solution {
    public int findCheapestPrice(int n, int[][] flights, int src, int dst, int k) {
        final int INF = Integer.MAX_VALUE;
        int[] dist = new int[n];
        Arrays.fill(dist, INF);
        dist[src] = 0;

        // K stops => at most K+1 edges => K+1 relaxation rounds
        for (int i = 0; i <= k; i++) {
            int[] prev = dist.clone();   // freeze last round's distances
            for (int[] f : flights) {
                int u = f[0], v = f[1], w = f[2];
                if (prev[u] != INF && prev[u] + w < dist[v]) {
                    dist[v] = prev[u] + w;
                }
            }
        }
        return dist[dst] == INF ? -1 : dist[dst];
    }
}
```

**Why clone:** without the snapshot, relaxing edge A then edge B in the same round could chain two new edges, sneaking in an extra stop beyond `K`.

**Complexity:** Time `O(K · E)`, Space `O(V)`.

---

## 28. Path With Minimum Effort — Medium (LC1631)

**Idea:** Dijkstra on the grid where the "distance" of a path is the **maximum** absolute height difference between adjacent cells along it (minimax). Relaxation uses `max(currentEffort, |h - hNeighbor|)`.

```java
import java.util.*;

class Solution {
    public int minimumEffortPath(int[][] heights) {
        int rows = heights.length, cols = heights[0].length;
        int[][] dirs = {{0, 1}, {1, 0}, {0, -1}, {-1, 0}};

        int[][] effort = new int[rows][cols];
        for (int[] row : effort) Arrays.fill(row, Integer.MAX_VALUE);
        effort[0][0] = 0;

        // {row, col, effortSoFar}
        PriorityQueue<int[]> pq =
            new PriorityQueue<>((a, b) -> Integer.compare(a[2], b[2]));
        pq.offer(new int[]{0, 0, 0});

        while (!pq.isEmpty()) {
            int[] cur = pq.poll();
            int r = cur[0], c = cur[1], e = cur[2];
            if (e > effort[r][c]) continue;                 // stale
            if (r == rows - 1 && c == cols - 1) return e;    // first pop is optimal

            for (int[] d : dirs) {
                int nr = r + d[0], nc = c + d[1];
                if (nr < 0 || nr >= rows || nc < 0 || nc >= cols) continue;
                int ne = Math.max(e, Math.abs(heights[nr][nc] - heights[r][c]));
                if (ne < effort[nr][nc]) {
                    effort[nr][nc] = ne;
                    pq.offer(new int[]{nr, nc, ne});
                }
            }
        }
        return 0; // single cell grid
    }
}
```

**Complexity:** Time `O(R·C·log(R·C))`, Space `O(R·C)`.

---

## 29. Swim in Rising Water — Hard (LC778)

**Idea:** The time to reach the bottom-right equals the **minimum over all paths of the maximum elevation** on that path — another minimax Dijkstra. The cost of stepping onto a cell is its elevation; the path cost is the max elevation seen.

```java
import java.util.*;

class Solution {
    public int swimInWater(int[][] grid) {
        int n = grid.length;
        int[][] dirs = {{0, 1}, {1, 0}, {0, -1}, {-1, 0}};

        boolean[][] visited = new boolean[n][n];
        // {row, col, maxElevationSoFar}
        PriorityQueue<int[]> pq =
            new PriorityQueue<>((a, b) -> Integer.compare(a[2], b[2]));
        pq.offer(new int[]{0, 0, grid[0][0]});

        while (!pq.isEmpty()) {
            int[] cur = pq.poll();
            int r = cur[0], c = cur[1], t = cur[2];
            if (visited[r][c]) continue;
            visited[r][c] = true;
            if (r == n - 1 && c == n - 1) return t;   // first arrival is optimal

            for (int[] d : dirs) {
                int nr = r + d[0], nc = c + d[1];
                if (nr < 0 || nr >= n || nc < 0 || nc >= n || visited[nr][nc]) continue;
                pq.offer(new int[]{nr, nc, Math.max(t, grid[nr][nc])});
            }
        }
        return -1; // unreachable (won't happen on valid input)
    }
}
```

**Complexity:** Time `O(N²·log N)`, Space `O(N²)`.

---

## 30. Path with Maximum Probability — Medium (LC1514)

**Idea:** Dijkstra with a **max-heap**, maximizing the **product** of edge probabilities. Since probabilities are in `[0,1]`, products only shrink, so the greedy "pop the largest first" property holds just like minimizing a sum.

```java
import java.util.*;

class Solution {
    public double maxProbability(int n, int[][] edges, double[] succProb,
                                 int start, int end) {
        List<double[]>[] adj = new List[n];
        for (int i = 0; i < n; i++) adj[i] = new ArrayList<>();
        for (int i = 0; i < edges.length; i++) {
            int u = edges[i][0], v = edges[i][1];
            double p = succProb[i];
            adj[u].add(new double[]{v, p});
            adj[v].add(new double[]{u, p});
        }

        double[] prob = new double[n];
        prob[start] = 1.0;

        // max-heap by probability; {node, prob}
        PriorityQueue<double[]> pq =
            new PriorityQueue<>((a, b) -> Double.compare(b[1], a[1]));
        pq.offer(new double[]{start, 1.0});

        while (!pq.isEmpty()) {
            double[] cur = pq.poll();
            int u = (int) cur[0];
            double p = cur[1];
            if (p < prob[u]) continue;          // stale
            if (u == end) return p;             // first pop of end is optimal

            for (double[] e : adj[u]) {
                int v = (int) e[0];
                double np = p * e[1];
                if (np > prob[v]) {
                    prob[v] = np;
                    pq.offer(new double[]{v, np});
                }
            }
        }
        return 0.0; // end unreachable
    }
}
```

**Complexity:** Time `O(E log V)`, Space `O(V + E)`.

---

## 31. Find the City With Smallest Number of Neighbors at a Threshold Distance — Medium (LC1334)

**Idea:** Compute all-pairs shortest paths with Floyd-Warshall, then for each city count how many others lie within `distanceThreshold`; pick the city with the fewest such neighbors, breaking ties by the **largest** index.

```java
import java.util.*;

class Solution {
    public int findTheCity(int n, int[][] edges, int distanceThreshold) {
        final int INF = 1_000_000_000;
        int[][] dist = new int[n][n];
        for (int[] row : dist) Arrays.fill(row, INF);
        for (int i = 0; i < n; i++) dist[i][i] = 0;

        for (int[] e : edges) {
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

        int bestCity = -1, fewest = Integer.MAX_VALUE;
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
}
```

**Complexity:** Time `O(V³)`, Space `O(V²)`.

---

## 32. Minimum Cost to Make at Least One Valid Path in a Grid — Hard (LC1368)

**Idea:** 0-1 BFS. Each cell points a direction; **following** the arrow costs `0`, **changing** it (moving any other direction) costs `1`. Find the minimum-cost path from top-left to bottom-right.

```java
import java.util.*;

class Solution {
    public int minCost(int[][] grid) {
        int rows = grid.length, cols = grid[0].length;
        // grid value 1=right, 2=left, 3=down, 4=up -> matching dirs index
        int[][] dirs = {{0, 1}, {0, -1}, {1, 0}, {-1, 0}};

        int[][] dist = new int[rows][cols];
        for (int[] row : dist) Arrays.fill(row, Integer.MAX_VALUE);
        dist[0][0] = 0;

        Deque<int[]> dq = new ArrayDeque<>();   // {row, col, cost}
        dq.offerFirst(new int[]{0, 0, 0});

        while (!dq.isEmpty()) {
            int[] cur = dq.pollFirst();
            int r = cur[0], c = cur[1], cost = cur[2];
            if (cost > dist[r][c]) continue;            // stale

            for (int dir = 0; dir < 4; dir++) {
                int nr = r + dirs[dir][0], nc = c + dirs[dir][1];
                if (nr < 0 || nr >= rows || nc < 0 || nc >= cols) continue;
                // grid is 1-indexed direction; arrow already points dir -> cost 0
                int w = (grid[r][c] == dir + 1) ? 0 : 1;
                if (cost + w < dist[nr][nc]) {
                    dist[nr][nc] = cost + w;
                    if (w == 0) dq.offerFirst(new int[]{nr, nc, cost});
                    else        dq.offerLast(new int[]{nr, nc, cost + 1});
                }
            }
        }
        return dist[rows - 1][cols - 1];
    }
}
```

**Complexity:** Time `O(R·C)`, Space `O(R·C)`.

---

## 33. Shortest Path Visiting All Nodes — Hard (LC847)

**Idea:** Unweighted shortest path over a **state graph** `(node, visitedMask)`. BFS layer by layer; the goal is any node whose mask equals all-ones (`(1<<n)-1`). Because `n <= 12`, the total state count `n · 2^n` is small.

```java
import java.util.*;

class Solution {
    public int shortestPathLength(int[][] graph) {
        int n = graph.length;
        if (n == 1) return 0;
        int full = (1 << n) - 1;

        boolean[][] visited = new boolean[n][1 << n];
        Deque<int[]> queue = new ArrayDeque<>();   // {node, mask}

        // can start from any node
        for (int i = 0; i < n; i++) {
            queue.offer(new int[]{i, 1 << i});
            visited[i][1 << i] = true;
        }

        int steps = 0;
        while (!queue.isEmpty()) {
            int size = queue.size();
            for (int s = 0; s < size; s++) {
                int[] cur = queue.poll();
                int node = cur[0], mask = cur[1];
                if (mask == full) return steps;     // all nodes visited

                for (int next : graph[node]) {
                    int nextMask = mask | (1 << next);
                    if (!visited[next][nextMask]) {
                        visited[next][nextMask] = true;
                        queue.offer(new int[]{next, nextMask});
                    }
                }
            }
            steps++;
        }
        return -1; // graph guaranteed connected, so unreachable
    }
}
```

**Complexity:** Time `O(n² · 2ⁿ)`, Space `O(n · 2ⁿ)`.

---

> **Last Updated:** 2026-06-26