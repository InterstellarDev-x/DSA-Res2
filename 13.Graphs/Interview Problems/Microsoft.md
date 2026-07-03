> **Topic:** [Graphs](../README.md) · **Section:** Interview Problems · **Company:** Microsoft

# Microsoft — Graphs Interview Deep Dives

Microsoft values clean, correct implementations and precise complexity reasoning. These deep dives cover its most-asked graph problems. See also [Amazon](./Amazon.md) and [Google](./Google.md).

---

## 1. Clone Graph (LC 133)

Deep-copy a connected undirected graph from a node reference.

### The unordered_map dual role

A single `unordered_map<Node*, Node*>` from **original → clone** serves two purposes at once: it's the **visited set** (key presence) *and* the **clone registry** (value). When you meet a node, if it's already a key you reuse its clone; otherwise you create one, store it, and recurse/enqueue. This is what makes cycles safe — you never re-create a node, so traversal terminates.

```cpp
#include <bits/stdc++.h>
using namespace std;

// BFS
Node* cloneGraph(Node* node) {
    if (node == nullptr) return nullptr;
    unordered_map<Node*, Node*> clones;
    clones[node] = new Node(node->val);
    queue<Node*> q;
    q.push(node);
    while (!q.empty()) {
        Node* cur = q.front(); q.pop();
        for (auto& nb : cur->neighbors) {
            if (!clones.count(nb)) {
                clones[nb] = new Node(nb->val);
                q.push(nb);
            }
            clones[cur]->neighbors.push_back(clones[nb]);
        }
    }
    return clones[node];
}

// DFS alternative
Node* dfs(Node* node, unordered_map<Node*, Node*>& clones) {
    if (node == nullptr) return nullptr;
    if (clones.count(node)) return clones[node];
    Node* copy = new Node(node->val);
    clones[node] = copy;                 // store BEFORE recursing (breaks cycles)
    for (auto& nb : node->neighbors)
        copy->neighbors.push_back(dfs(nb, clones));
    return copy;
}
Node* cloneGraphDfs(Node* node) {
    unordered_map<Node*, Node*> clones;
    return dfs(node, clones);
}
```

**BFS vs DFS here:** identical complexity O(V+E); DFS is shorter but risks deep recursion on long chains, BFS uses an explicit queue. The critical detail in DFS is putting the clone in the map **before** recursing into neighbors — otherwise a cycle re-enters and overflows.

### Edge cases — null input → nullptr; single node with no neighbors; self-loop; node referencing itself.

### Follow-ups — clone a graph with random pointers; serialize/deserialize a graph.

---

## 2. Network Delay Time (LC 743)

Time for a signal from node `k` to reach all nodes; `-1` if some unreachable.

### Dijkstra with stale-entry skipping

A binary heap doesn't support decrease-key cheaply, so the standard trick is **lazy deletion**: push a new `(dist, node)` whenever you relax, and when you pop, **skip the entry if its distance is worse than the best known**. This keeps the heap correct without an indexed PQ.

```cpp
#include <bits/stdc++.h>
using namespace std;

int networkDelayTime(vector<vector<int>>& times, int n, int k) {
    unordered_map<int, vector<pair<int,int>>> adj;
    for (auto& t : times)
        adj[t[0]].push_back({t[1], t[2]});
    vector<int> dist(n + 1, INT_MAX);
    dist[k] = 0;
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<pair<int,int>>> pq; // min-heap by distance
    pq.push({0, k});
    while (!pq.empty()) {
        auto [d, node] = pq.top(); pq.pop();
        if (d > dist[node]) continue;            // STALE entry — skip
        if (adj.count(node)) {
            for (auto& [v, w] : adj[node]) {
                int nd = d + w;
                if (nd < dist[v]) {
                    dist[v] = nd;
                    pq.push({nd, v});
                }
            }
        }
    }
    int ans = 0;
    for (int i = 1; i <= n; i++) {
        if (dist[i] == INT_MAX) return -1;
        ans = max(ans, dist[i]);
    }
    return ans;
}
```

**Two non-negotiables:** comparator uses `greater<pair<int,int>>` for min-heap (subtraction overflows on large weights); and the `if (d > dist[node]) continue` skip avoids reprocessing finalized nodes — without it you can still get the right answer but with wasted work, and it's the detail interviewers probe.

### Edge cases — unreachable node → `-1`; `k` itself counts (distance 0); duplicate edges → keep the cheaper via relaxation.

### Follow-ups — *Path With Minimum Effort* (minimize the max edge → relax on `max(d, edge)`); *Swim in Rising Water* (same minimax Dijkstra).

---

## 3. Cheapest Flights Within K Stops (LC 787)

Cheapest price from `src` to `dst` using at most `k` stops.

### Why Bellman-Ford / per-round dist cloning

Plain Dijkstra fails because the cheapest cost ignoring the stop limit may use too many hops; greedily finalizing a node can lock in a path that violates the `k` constraint. **Bellman-Ford** relaxes all edges exactly `k+1` times — each round = "one more flight allowed" — which naturally bounds the hop count.

The subtlety: within a single round you must relax using the **previous round's** distances, not values updated earlier in the same round. Otherwise one round could chain multiple edges (more than one extra stop). So **clone `dist` at the start of each round** and read from the clone, write to the live array.

```cpp
#include <bits/stdc++.h>
using namespace std;

int findCheapestPrice(int n, vector<vector<int>>& flights, int src, int dst, int k) {
    vector<int> dist(n, INT_MAX);
    dist[src] = 0;
    for (int round = 0; round <= k; round++) {       // k+1 relaxations
        vector<int> prev = dist;                     // freeze this round's source
        for (auto& f : flights) {
            int u = f[0], v = f[1], w = f[2];
            if (prev[u] == INT_MAX) continue;
            dist[v] = min(dist[v], prev[u] + w);
        }
    }
    return dist[dst] == INT_MAX ? -1 : dist[dst];
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

```cpp
#include <bits/stdc++.h>
using namespace std;

void mark(vector<vector<char>>& board, int r, int c) {
    if (r < 0 || r >= (int)board.size() || c < 0 || c >= (int)board[0].size() || board[r][c] != 'O') return;
    board[r][c] = '#';                       // sentinel for border-connected
    mark(board, r + 1, c); mark(board, r - 1, c);
    mark(board, r, c + 1); mark(board, r, c - 1);
}

void solve(vector<vector<char>>& board) {
    if (board.empty() || board[0].empty()) return;
    int m = board.size(), n = board[0].size();
    for (int i = 0; i < m; i++) { mark(board, i, 0); mark(board, i, n - 1); }
    for (int j = 0; j < n; j++) { mark(board, 0, j); mark(board, m - 1, j); }
    for (int i = 0; i < m; i++)
        for (int j = 0; j < n; j++) {
            if (board[i][j] == 'O') board[i][j] = 'X';    // enclosed → capture
            else if (board[i][j] == '#') board[i][j] = 'O'; // border-safe → restore
        }
}
```

### Edge cases — single row/column (all border, nothing captured); no `'O'`; all `'O'` touching border.

### Follow-ups — *Number of Enclaves* (count, don't flip); *Pacific Atlantic* uses the same reverse-from-border idea.

---

## What Microsoft Emphasizes

| Dimension | Expectation |
|-----------|-------------|
| Correctness | Handle `-1` / nullptr / single-cell cases |
| Clarity | `int[][] dirs`, named helpers, no subtraction comparators |
| Complexity | State `O(V+E)`, `O((V+E) log V)`, `O(V·E)` precisely |
| Reasoning | Explain *why* the algorithm fits (negative/limited stops → Bellman-Ford) |

> **Last Updated:** 2026-06-26
