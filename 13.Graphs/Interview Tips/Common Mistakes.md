> **Topic:** [Graphs](../README.md) · **Section:** Interview Tips · **Tips 2 of 4**

# Graphs — Common Mistakes

Twelve recurring bugs as wrong/correct pairs. Each costs correctness or a TLE in real interviews. See also [Coding Tips](./Coding%20Tips.md), [Algorithm Selection](./Algorithm%20Selection.md), [Complexity Analysis](./Complexity%20Analysis.md).

---

## 1. Marking visited at dequeue instead of enqueue → TLE / duplicates

```cpp
#include <bits/stdc++.h>
using namespace std;
// WRONG — same node enqueued by many neighbors before it's marked
while (!q.empty()) {
    int u = q.front(); q.pop();
    if (visited[u]) continue;       // marked too late
    visited[u] = true;
    for (int v : adj[u]) q.push(v);
}
// CORRECT — mark the instant you enqueue
visited[start] = true; q.push(start);
while (!q.empty()) {
    int u = q.front(); q.pop();
    for (int v : adj[u])
        if (!visited[v]) { visited[v] = true; q.push(v); }
}
```

## 2. Subtraction in a PQ comparator → integer overflow

```cpp
#include <bits/stdc++.h>
using namespace std;
// WRONG — a[1] - b[1] overflows when distances are large
auto cmp_wrong = [](auto& a, auto& b){ return a[1] - b[1] > 0; };
priority_queue<vector<int>, vector<vector<int>>, decltype(cmp_wrong)> pq_wrong(cmp_wrong);
// CORRECT
auto cmp = [](auto& a, auto& b){ return a[1] > b[1]; }; // min-heap by dist
priority_queue<vector<int>, vector<vector<int>>, decltype(cmp)> pq(cmp);
```

## 3. Forgetting the stale-entry skip in Dijkstra

```cpp
#include <bits/stdc++.h>
using namespace std;
// WRONG — reprocesses finalized nodes, may relax with outdated d
auto top = pq.top(); pq.pop();
for (auto& e : adj[top[0]]) { /* relax using top[1] */ }
// CORRECT
auto top = pq.top(); pq.pop();
int u = top[0], d = top[1];
if (d > dist[u]) continue;          // skip stale entry
for (auto& e : adj[u]) { /* relax using d */ }
```

## 4. Bellman-Ford for K-stops without cloning `dist`

```cpp
#include <bits/stdc++.h>
using namespace std;
// WRONG — one round chains multiple edges → exceeds stop limit
for (int round = 0; round <= k; round++)
    for (auto& f : flights)
        dist[f[1]] = min(dist[f[1]], dist[f[0]] + f[2]);
// CORRECT — freeze previous round's distances
for (int round = 0; round <= k; round++) {
    vector<int> prev = dist;
    for (auto& f : flights)
        if (prev[f[0]] != INT_MAX)
            dist[f[1]] = min(dist[f[1]], prev[f[0]] + f[2]);
}
```

## 5. Adding an undirected edge in only one direction

```cpp
#include <bits/stdc++.h>
using namespace std;
// WRONG — half the graph is missing
adj[u].push_back(v);
// CORRECT
adj[u].push_back(v);
adj[v].push_back(u);
```

## 6. DSU without path compression → near-O(n) find, TLE

```cpp
#include <bits/stdc++.h>
using namespace std;
// WRONG — degenerate chains
int find(int x) { while (parent[x] != x) x = parent[x]; return x; }
// CORRECT — path compression (+ union by rank)
int find(int x) { while (parent[x] != x) { parent[x] = parent[parent[x]]; x = parent[x]; } return x; }
```

## 7. Directed cycle detection via parent-tracking → wrong

Parent-skipping detects cycles in **undirected** graphs only. For directed graphs use 3-color.

```cpp
#include <bits/stdc++.h>
using namespace std;
// WRONG for directed — misses cycles like a->b->a's true direction semantics
bool dfs(int u, int parent) {
    for (int v : adj[u]) {
        if (v == parent) continue;          // undirected idea, wrong here
        if (visited[v] || dfs(v, u)) return true;
    }
    return true;
}
// CORRECT for directed — gray = on current path
bool dfs(int u) {
    color[u] = 1;
    for (int v : adj[u]) {
        if (color[v] == 1) return true;     // back edge
        if (color[v] == 0 && dfs(v)) return true;
    }
    color[u] = 2;
    return false;
}
```

## 8. Kahn's without checking all nodes processed → misses cycle

```cpp
#include <bits/stdc++.h>
using namespace std;
// WRONG — returns a partial "order" even with a cycle
while (!q.empty()) { order.push_back(q.front()); q.pop(); /* ... */ }
return order;
// CORRECT
return (int)order.size() == n ? order : vector<int>{};   // cycle ⇒ empty
```

## 9. Grid bounds check ordered after the array read → out-of-bounds

```cpp
#include <bits/stdc++.h>
using namespace std;
// WRONG — reads grid[r][c] before validating r,c
if (grid[r][c] == '1' && r >= 0 && r < m && c >= 0 && c < n) ...
// CORRECT — bounds FIRST, short-circuit protects the read
if (r >= 0 && r < m && c >= 0 && c < n && grid[r][c] == '1') ...
```

## 10. Modifying the grid during BFS without marking → revisits / infinite loop

```cpp
#include <bits/stdc++.h>
using namespace std;
// WRONG — neighbor added repeatedly; never marked
for (auto& d : dirs) { int r=..., c=...; if (grid[r][c]==1) q.push({r,c}); }
// CORRECT — mark as you enqueue
for (auto& d : dirs) {
    int r=..., c=...;
    if (inBounds(r,c) && grid[r][c]==1) { grid[r][c] = 2; q.push({r,c}); }
}
```

## 11. Alien Dictionary missing the prefix-conflict check

```cpp
#include <bits/stdc++.h>
using namespace std;
// WRONG — ["abc","ab"] should be invalid but slips through
int j = 0; while (j < len && a[j]==b[j]) j++;
if (j < len) addEdge(a[j], b[j]);
// CORRECT
if (j == len) { if (a.length() > b.length()) return ""; }   // longer word before its prefix
else addEdge(a[j], b[j]);
```

## 12. 8-direction vs 4-direction mix-up

Number of Islands / Rotting Oranges are **4-directional**. Using `dirs8` over-connects and inflates/merges islands. Conversely, problems like *Shortest Path in Binary Matrix* are **8-directional** — using 4 dirs gives wrong (too-long or no) paths. Always confirm connectivity from the prompt before choosing `dirs` vs `dirs8`.

```cpp
#include <bits/stdc++.h>
using namespace std;
vector<vector<int>> dirs  = {{1,0},{-1,0},{0,1},{0,-1}};                         // islands, rotting
vector<vector<int>> dirs8 = {{1,0},{-1,0},{0,1},{0,-1},{1,1},{1,-1},{-1,1},{-1,-1}}; // binary matrix path
```

> **Last Updated:** 2026-06-26
