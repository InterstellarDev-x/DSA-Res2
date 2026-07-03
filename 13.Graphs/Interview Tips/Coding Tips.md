> **Topic:** [Graphs](../README.md) · **Section:** Interview Tips · **Tips 1 of 4**

# Graphs — Coding Tips

Ten battle-tested idioms that make graph code shorter, faster, and correct. See also [Common Mistakes](./Common%20Mistakes.md), [Algorithm Selection](./Algorithm%20Selection.md), [Complexity Analysis](./Complexity%20Analysis.md).

---

## 1. Standard adjacency-list build idiom

For a weighted graph use `unordered_map<int, vector<pair<int,int>>>`; for unweighted, `vector<vector<int>>`. Build once, reuse everywhere.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<vector<int>> adj(n);
for (auto& e : edges) {
    adj[e[0]].push_back(e[1]);
    adj[e[1]].push_back(e[0]);          // BOTH directions for undirected
}
// weighted:
unordered_map<int, vector<pair<int,int>>> wadj;
for (auto& e : edges)
    wadj[e[0]].push_back({e[1], e[2]});
```

## 2. Use a `vector<vector<int>> dirs` array for grid neighbors

Replaces four near-identical blocks with one loop. Use 8-dir only when diagonals count.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<vector<int>> dirs = {{1,0},{-1,0},{0,1},{0,-1}};            // 4-directional
vector<vector<int>> dirs8 = {{1,0},{-1,0},{0,1},{0,-1},{1,1},{1,-1},{-1,1},{-1,-1}}; // 8-dir
for (auto& d : dirs) {
    int r = row + d[0], c = col + d[1];
    if (r < 0 || r >= m || c < 0 || c >= n) continue;  // bounds FIRST
    // ... use grid[r][c]
}
```

## 3. Mark visited BEFORE enqueue, never after dequeue

The single most important BFS rule. Marking at dequeue lets the same node be pushed by multiple neighbors → duplicates, bloated queue, sometimes wrong counts.

```cpp
#include <bits/stdc++.h>
using namespace std;

visited[start] = true;
q.push(start);
while (!q.empty()) {
    int u = q.front(); q.pop();
    for (int v : adj[u])
        if (!visited[v]) {
            visited[v] = true;          // mark at the moment you enqueue
            q.push(v);
        }
}
```

## 4. Multi-source BFS — seed all sources first

For "spread from many origins simultaneously" (Rotting Oranges, 01 Matrix, Walls and Gates), push every source at time 0, then expand level by level.

```cpp
#include <bits/stdc++.h>
using namespace std;

for (int i = 0; i < m; i++)
    for (int j = 0; j < n; j++)
        if (isSource(grid[i][j])) q.push({i, j});
int level = 0;
while (!q.empty()) {
    int size = q.size();            // freeze the layer
    for (int s = 0; s < size; s++) { /* expand one cell */ }
    level++;
}
```

## 5. Dijkstra — stale-entry skip + direct comparison

Lazy-deletion heap: re-push on relax, skip popped entries that are worse than the recorded best. Always use a proper comparator, never `a[1] - b[1]`.

```cpp
#include <bits/stdc++.h>
using namespace std;

priority_queue<pair<int,int>, vector<pair<int,int>>, greater<pair<int,int>>> pq;
pq.push({0, src});
while (!pq.empty()) {
    auto [d, u] = pq.top(); pq.pop();
    if (d > dist[u]) continue;          // stale — already finalized cheaper
    for (auto& [v, w] : adj[u]) {
        int nd = d + w;
        if (nd < dist[v]) { dist[v] = nd; pq.push({nd, v}); }
    }
}
```

## 6. 0-1 BFS with a `deque`

When edge weights are only 0 or 1, replace Dijkstra's heap with a deque: push 0-weight moves to the **front**, 1-weight to the **back**. O(V+E).

```cpp
#include <bits/stdc++.h>
using namespace std;

deque<pair<int,int>> dq;
dq.push_front({src, 0});
while (!dq.empty()) {
    auto [u, d] = dq.front(); dq.pop_front();
    if (d > dist[u]) continue;
    for (auto& [v, w] : adj[u]) {
        int nd = d + w;              // w is 0 or 1
        if (nd < dist[v]) {
            dist[v] = nd;
            if (w == 0) dq.push_front({v, nd});
            else        dq.push_back({v, nd});
        }
    }
}
```

## 7. DSU with path compression + union by rank

The canonical near-O(α(n)) disjoint-set. Memorize it; it appears in Union-Find, MST (Kruskal), and cycle detection.

```cpp
#include <bits/stdc++.h>
using namespace std;

struct DSU {
    vector<int> parent, rank_;
    DSU(int n) : parent(n), rank_(n, 0) {
        iota(parent.begin(), parent.end(), 0);
    }
    int find(int x) {
        while (parent[x] != x) { parent[x] = parent[parent[x]]; x = parent[x]; } // compression
        return x;
    }
    bool unite(int a, int b) {
        int ra = find(a), rb = find(b);
        if (ra == rb) return false;     // already connected (cycle for undirected)
        if (rank_[ra] < rank_[rb]) swap(ra, rb);
        parent[rb] = ra;
        if (rank_[ra] == rank_[rb]) rank_[ra]++;
        return true;
    }
};
```

## 8. 3-color DFS for directed cycle detection

WHITE (unvisited) / GRAY (on the current DFS stack) / BLACK (done). A GRAY→GRAY edge is a back edge = cycle. Do **not** use parent-tracking for directed graphs (that's the undirected technique).

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> color;   // 0 white, 1 gray, 2 black
bool hasCycle(int u, vector<vector<int>>& adj) {
    color[u] = 1;
    for (int v : adj[u]) {
        if (color[v] == 1) return true;        // back edge → cycle
        if (color[v] == 0 && hasCycle(v, adj)) return true;
    }
    color[u] = 2;
    return false;
}
```

## 9. Kahn's indegree topological sort

BFS over zero-indegree nodes. If you can't process all nodes, the graph has a cycle.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> indegree(n, 0);
for (auto& e : edges) indegree[e[1]]++;        // edge u->v
queue<int> q;
for (int i = 0; i < n; i++) if (indegree[i] == 0) q.push(i);
vector<int> order;
while (!q.empty()) {
    int u = q.front(); q.pop();
    order.push_back(u);
    for (int v : adj[u]) if (--indegree[v] == 0) q.push(v);
}
bool acyclic = (int)order.size() == n;           // else cycle
```

## 10. Bitmask state for "visit all nodes"

When the answer depends on *which subset* of nodes you've visited (Shortest Path Visiting All Nodes), make the BFS state `(node, mask)`. Goal: `mask == (1 << n) - 1`.

```cpp
#include <bits/stdc++.h>
using namespace std;

int full = (1 << n) - 1;
queue<pair<int,int>> q;           // {node, mask}
vector<vector<bool>> seen(n, vector<bool>(1 << n, false));
for (int i = 0; i < n; i++) { q.push({i, 1 << i}); seen[i][1 << i] = true; }
int steps = 0;
while (!q.empty()) {
    int size = q.size();
    for (int s = 0; s < size; s++) {
        auto [node, mask] = q.front(); q.pop();
        if (mask == full) return steps;
        for (int nb : adj[node]) {
            int nmask = mask | (1 << nb);
            if (!seen[nb][nmask]) { seen[nb][nmask] = true; q.push({nb, nmask}); }
        }
    }
    steps++;
}
```

> **Last Updated:** 2026-06-26
