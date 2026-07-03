> **Topic:** [Graphs](../README.md) · **Pattern 7 of 7**

# Cycle Detection

A **cycle** is a path that starts and ends at the same vertex while traversing at least one edge, without reusing any edge. Detecting cycles is a foundational graph operation: it tells us whether a graph is a tree/forest (acyclic), whether a dependency graph can be scheduled (a DAG), and whether a network has redundant connections.

The technique you reach for depends entirely on whether the graph is **undirected** or **directed** — the two cases have different traps and different toolkits.

---

## 1. Core Concept

| Graph type | Cycle means | Primary tools |
| --- | --- | --- |
| **Undirected** | An edge connects two vertices already in the same connected component | **DSU (Union-Find)** or **DFS with parent tracking** |
| **Directed** | A vertex can reach itself by following edge directions (a back edge to an ancestor on the recursion stack) | **3-color DFS** or **Kahn's algorithm (BFS topological sort)** |

Key intuitions:

- **Undirected, DSU view:** Each undirected edge `(u, v)` tries to *union* two components. If `u` and `v` already share a root before the union, adding the edge closes a loop — that's a cycle.
- **Undirected, DFS view:** During DFS, if we reach an already-visited vertex that is **not** the parent we came from, we found a back edge → cycle. (We must ignore the immediate parent, otherwise the edge we just traversed would falsely look like a cycle.)
- **Directed, 3-color view:** A cycle exists iff DFS encounters a vertex currently **on the recursion stack** (gray). Reaching a fully-finished (black) vertex is fine — that's just a cross/forward edge, not a cycle.
- **Directed, Kahn view:** Repeatedly remove vertices with indegree 0. If you can't remove all `n` vertices (processed count `< n`), the remaining vertices form one or more cycles.

> **Trap:** DSU works only for *undirected* cycle detection. For directed graphs you need ordering-aware methods (3-color DFS or Kahn), because direction matters — `a→b` and `a→c→b` do not form a cycle even though `b` is reached twice.

---

## 2. Undirected Cycle Detection

### 2a. Via DSU (Union-Find)

For each edge, if both endpoints already share a root, we've found a cycle. Otherwise union them.

```cpp
#include <bits/stdc++.h>
using namespace std;

struct DSU {
    vector<int> parent, rank_;

    DSU(int n) : parent(n), rank_(n, 0) {
        iota(parent.begin(), parent.end(), 0);
    }

    int find(int x) {                 // path compression
        if (parent[x] != x) parent[x] = find(parent[x]);
        return parent[x];
    }

    // returns false if x,y already connected (=> cycle); true if merged
    bool unite(int x, int y) {
        int rx = find(x), ry = find(y);
        if (rx == ry) return false;
        if (rank_[rx] < rank_[ry])      { parent[rx] = ry; }
        else if (rank_[rx] > rank_[ry]) { parent[ry] = rx; }
        else                            { parent[ry] = rx; rank_[rx]++; }
        return true;
    }
};

class UndirectedCycleDSU {
public:
    // edges given as vector<vector<int>> where each row is {u, v}, vertices 0..n-1
    bool hasCycle(int n, vector<vector<int>>& edges) {
        DSU dsu(n);
        for (auto& e : edges) {
            if (!dsu.unite(e[0], e[1])) return true; // both share root => cycle
        }
        return false;
    }
};
```

### 2b. Via DFS with Parent Tracking

Walk the graph; if you reach a visited neighbor that isn't the parent, it's a back edge → cycle. Handle disconnected graphs by starting DFS from every unvisited vertex.

```cpp
#include <bits/stdc++.h>
using namespace std;

class UndirectedCycleDFS {
public:
    bool hasCycle(int n, vector<vector<int>>& adj) {
        vector<bool> visited(n, false);
        for (int i = 0; i < n; i++) {
            if (!visited[i] && dfs(i, -1, adj, visited)) return true;
        }
        return false;
    }

private:
    bool dfs(int node, int parent, vector<vector<int>>& adj, vector<bool>& visited) {
        visited[node] = true;
        for (int nei : adj[node]) {
            if (!visited[nei]) {
                if (dfs(nei, node, adj, visited)) return true;
            } else if (nei != parent) {   // visited and not the node we came from
                return true;
            }
        }
        return false;
    }
};
```

> **BFS variant note:** The same idea works iteratively with BFS by storing `(node, parent)` pairs in the queue and flagging a cycle when a neighbor is already visited and isn't the parent. With parallel edges, track the edge instead of the parent vertex.

---

## 3. Directed Cycle Detection

### 3a. Via 3-Color DFS

Colors: `0 = white` (unvisited), `1 = gray` (in current recursion stack), `2 = black` (fully processed). A cycle exists exactly when DFS hits a **gray** node.

```cpp
#include <bits/stdc++.h>
using namespace std;

class DirectedCycle3Color {
    static const int WHITE = 0, GRAY = 1, BLACK = 2;

public:
    bool hasCycle(int n, vector<vector<int>>& adj) {
        vector<int> color(n, 0);          // all WHITE initially
        for (int i = 0; i < n; i++) {
            if (color[i] == WHITE && dfs(i, adj, color)) return true;
        }
        return false;
    }

private:
    bool dfs(int node, vector<vector<int>>& adj, vector<int>& color) {
        color[node] = GRAY;                 // entering recursion stack
        for (int nei : adj[node]) {
            if (color[nei] == GRAY) return true;          // back edge => cycle
            if (color[nei] == WHITE && dfs(nei, adj, color)) return true;
            // color[nei] == BLACK => already done, safe to skip
        }
        color[node] = BLACK;                // leaving recursion stack
        return false;
    }
};
```

### 3b. Via Kahn's Algorithm (BFS Topological Sort)

Peel off indegree-0 vertices. If the number processed is less than `n`, the leftovers are stuck in a cycle.

```cpp
#include <bits/stdc++.h>
using namespace std;

class DirectedCycleKahn {
public:
    bool hasCycle(int n, vector<vector<int>>& adj) {
        vector<int> indeg(n, 0);
        for (int u = 0; u < n; u++)
            for (int v : adj[u]) indeg[v]++;

        queue<int> q;
        for (int i = 0; i < n; i++)
            if (indeg[i] == 0) q.push(i);

        int processed = 0;
        while (!q.empty()) {
            int u = q.front(); q.pop();
            processed++;
            for (int v : adj[u]) {
                if (--indeg[v] == 0) q.push(v);
            }
        }
        return processed < n;   // couldn't topologically order everything => cycle
    }
};
```

---

## 4. Recognition Signals

| Signal in the problem | Likely approach |
| --- | --- |
| "Undirected graph, is it a tree / valid / has redundant edge?" | DSU (union; cycle when endpoints already connected) |
| "Undirected, detect any loop" | DFS with parent tracking |
| "Directed dependencies, can all tasks finish?" | Kahn or 3-color DFS |
| "Find the edge that, if removed, makes it a tree" | DSU, return the edge that closes a cycle |
| "Directed, a node has two parents (indegree 2)" | Redundant Connection II — combine indegree check + DSU |
| "Edges whose removal disconnects the graph" | Tarjan's bridges (`disc[]`/`low[]`), not plain cycle detection |
| "Which nodes can never reach a cycle / are 'safe'?" | Reverse topological / 3-color DFS (see Topological Sort) |
| "Just check if two nodes are connected" | DSU or BFS/DFS reachability |

---

## 5. Cross-Link

See [Topological Sort](./Topological%20Sort.md) — cycle detection underpins topological ordering: **a directed graph has a valid topological order if and only if it is acyclic (a DAG).** Kahn's algorithm and 3-color DFS are literally the same machinery used in both patterns; problems like *Course Schedule* and *Find Eventual Safe States* are cycle-detection problems framed as ordering problems.

---

## 47. Detect Cycle in Undirected Graph — Medium (LC——)

Given an undirected graph with `n` vertices and an edge list, determine whether it contains a cycle.

**Approach A — DSU.** Process edges; the first edge whose endpoints already share a root proves a cycle.

```cpp
#include <bits/stdc++.h>
using namespace std;

class DetectCycleUndirected {
    vector<int> parent, rank_;

    int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]);
        return parent[x];
    }

    bool unite(int x, int y) {
        int rx = find(x), ry = find(y);
        if (rx == ry) return false;            // already connected => cycle
        if (rank_[rx] < rank_[ry]) parent[rx] = ry;
        else if (rank_[rx] > rank_[ry]) parent[ry] = rx;
        else { parent[ry] = rx; rank_[rx]++; }
        return true;
    }

public:
    bool hasCycle(int n, vector<vector<int>>& edges) {
        parent.resize(n);
        rank_.assign(n, 0);
        iota(parent.begin(), parent.end(), 0);
        for (auto& e : edges) {
            if (!unite(e[0], e[1])) return true;
        }
        return false;
    }

    // Approach B — DFS with parent tracking (handles disconnected components)
    bool hasCycleDFS(int n, vector<vector<int>>& adj) {
        vector<bool> visited(n, false);
        for (int i = 0; i < n; i++)
            if (!visited[i] && dfs(i, -1, adj, visited)) return true;
        return false;
    }

private:
    bool dfs(int node, int par, vector<vector<int>>& adj, vector<bool>& visited) {
        visited[node] = true;
        for (int nei : adj[node]) {
            if (!visited[nei]) {
                if (dfs(nei, node, adj, visited)) return true;
            } else if (nei != par) {
                return true;
            }
        }
        return false;
    }
};
```

**Complexity:** DSU runs in `O(E · α(N))`; DFS in `O(V + E)`.

---

## 48. Detect Cycle in Directed Graph — Medium (LC——)

Given a directed graph, determine whether it contains a directed cycle. Use 3-color DFS: a cycle exists iff DFS reaches a vertex currently on the recursion stack (gray).

```cpp
#include <bits/stdc++.h>
using namespace std;

class DetectCycleDirected {
    static const int WHITE = 0, GRAY = 1, BLACK = 2;

public:
    bool hasCycle(int n, vector<vector<int>>& adj) {
        vector<int> color(n, 0);
        for (int i = 0; i < n; i++) {
            if (color[i] == WHITE && dfs(i, adj, color)) return true;
        }
        return false;
    }

private:
    bool dfs(int node, vector<vector<int>>& adj, vector<int>& color) {
        color[node] = GRAY;
        for (int nei : adj[node]) {
            if (color[nei] == GRAY) return true;                 // back edge
            if (color[nei] == WHITE && dfs(nei, adj, color)) return true;
        }
        color[node] = BLACK;
        return false;
    }
};
```

**Why parent-tracking fails here:** In a directed graph `a→b` and `b→a` is a genuine 2-cycle, so we cannot "excuse the parent." The recursion-stack (gray) check correctly distinguishes a back edge (ancestor on the stack) from a harmless cross edge (already-finished black node).

**Complexity:** `O(V + E)` time, `O(V)` space.

---

## 49. Find Eventual Safe States — Medium (LC802)

> **Cross-reference.** This problem is covered in detail in [Topological Sort](./Topological%20Sort.md). A node is *safe* if every path from it leads to a terminal node — equivalently, if it is **not** part of and cannot reach any cycle. The standard solution is a 3-color DFS (a node is safe iff its DFS subtree contains no gray/back edge) or a reverse-graph Kahn's algorithm. See Topological Sort for the full walkthrough.

Brief snippet (3-color DFS; safe ⟺ no cycle reachable):

```cpp
#include <bits/stdc++.h>
using namespace std;

bool safe(int u, vector<vector<int>>& g, vector<int>& color) {
    if (color[u] != 0) return color[u] == 2; // gray => unsafe, black => safe
    color[u] = 1;
    for (int v : g[u]) if (!safe(v, g, color)) return false;
    color[u] = 2;
    return true;
}

vector<int> eventualSafeNodes(vector<vector<int>>& graph) {
    int n = graph.size();
    vector<int> color(n, 0); // 0 white, 1 gray, 2 black(safe)
    vector<int> res;
    for (int i = 0; i < n; i++) if (safe(i, graph, color)) res.push_back(i);
    return res;
}
```

See [Topological Sort](./Topological%20Sort.md) for the detailed reverse-Kahn treatment.

---

## 50. Course Schedule — Medium (LC207)

> **Cross-reference.** Covered fully in [Topological Sort](./Topological%20Sort.md). `numCourses` with prerequisite pairs `[a, b]` (take `b` before `a`) is solvable iff the dependency graph is a **DAG** — i.e., it has no directed cycle. Detect with Kahn's algorithm or 3-color DFS.

Brief snippet (Kahn's):

```cpp
#include <bits/stdc++.h>
using namespace std;

bool canFinish(int numCourses, vector<vector<int>>& prerequisites) {
    vector<vector<int>> adj(numCourses);
    vector<int> indeg(numCourses, 0);
    for (auto& p : prerequisites) { adj[p[1]].push_back(p[0]); indeg[p[0]]++; }

    queue<int> q;
    for (int i = 0; i < numCourses; i++) if (indeg[i] == 0) q.push(i);
    int done = 0;
    while (!q.empty()) {
        int u = q.front(); q.pop(); done++;
        for (int v : adj[u]) if (--indeg[v] == 0) q.push(v);
    }
    return done == numCourses; // all scheduled => acyclic
}
```

See [Topological Sort](./Topological%20Sort.md) for the full explanation and the DFS variant.

---

## 51. Redundant Connection II — Hard (LC685)

A rooted tree with `n` nodes is given `n` directed edges; exactly one extra edge was added. Return the edge to remove so the result is a valid rooted tree (each node except the root has exactly one parent, and there are no cycles). If multiple answers, return the last one in the input.

**The two failure modes (and they can co-occur):**
1. **A node has two parents** (indegree 2). Two edges point to the same node; one of them must be the culprit.
2. **A cycle** exists.

**Strategy:**
- Scan edges; if some node `v` has two incoming edges, record them as `cand1` (first) and `cand2` (second/later). Temporarily *invalidate* `cand2` (skip it during DSU).
- Run DSU over the remaining edges.
  - If no cycle forms → `cand2` was the redundant one (it was the only problem). Return `cand2`.
  - If a cycle still forms:
    - If there was **no** indegree-2 node, the redundant edge is simply the one that closed the cycle (pure cycle case). Return it.
    - If there **was** an indegree-2 node, then `cand1` is the offender (removing `cand2` didn't fix things, so the other parent edge `cand1` is part of the cycle). Return `cand1`.

```cpp
#include <bits/stdc++.h>
using namespace std;

class RedundantConnectionII {
    vector<int> parent;

    int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]);
        return parent[x];
    }

public:
    vector<int> findRedundantDirectedConnection(vector<vector<int>>& edges) {
        int n = edges.size();
        parent.resize(n + 1);
        vector<int> incomingParent(n + 1, 0); // incomingParent[v] = which edge index gave v its parent

        vector<int> cand1, cand2;      // two edges into the same node
        for (int i = 0; i < n; i++) {
            int u = edges[i][0], v = edges[i][1];
            if (incomingParent[v] != 0) {       // v already has a parent edge
                cand1 = edges[incomingParent[v] - 1]; // earlier edge into v
                cand2 = edges[i];                     // current edge into v
            } else {
                incomingParent[v] = i + 1;      // store 1-based index to keep 0 as "none"
            }
        }

        iota(parent.begin(), parent.end(), 0);

        // Run union over all edges except cand2 (if it exists)
        for (int i = 0; i < n; i++) {
            if (!cand2.empty() && edges[i] == cand2) continue; // skip the second parent edge
            int u = edges[i][0], v = edges[i][1];
            int ru = find(u), rv = find(v);
            if (ru == rv) {
                // a cycle formed without cand2
                if (cand1.empty()) return edges[i]; // pure cycle, no indegree-2 node
                return cand1;                       // indegree-2 case: cand1 is the bad edge
            }
            parent[rv] = ru;
        }
        // No cycle without cand2 => cand2 was the redundant edge
        return cand2;
    }
};
```

**Walking the cases:**
- **Only a cycle, no two-parent node:** `cand1`/`cand2` stay empty; the union loop finds the closing edge and returns it.
- **Only two parents, no cycle:** Skipping `cand2` leaves a clean tree; loop finishes; we return `cand2`.
- **Two parents AND a cycle:** Skipping `cand2` still leaves a cycle (because `cand1` is the one in the loop); we return `cand1`.

**Complexity:** `O(n · α(n))` time, `O(n)` space.

---

## 52. Critical Connections in a Network — Hard (LC1192)

Given `n` servers (`0..n-1`) and undirected `connections`, return all **critical connections** — edges whose removal disconnects part of the network. These are exactly the **bridges** of the graph.

**Tarjan's bridge algorithm.** Do a DFS assigning each node a discovery time `disc[u]` (when first visited) and a low-link `low[u]` (the smallest discovery time reachable from `u`'s DFS subtree, including via at most one back edge). For a tree edge `u→v`:
- After exploring `v`, set `low[u] = min(low[u], low[v])`.
- If `low[v] > disc[u]`, then `v`'s subtree has **no** back edge that reaches `u` or above → edge `(u, v)` is a **bridge**.
- For a back edge `u→v` (where `v` is already visited and isn't the parent), update `low[u] = min(low[u], disc[v])`.

We must skip the immediate parent edge — but to handle **parallel edges** correctly, skip by *edge identity*, not by parent vertex.

```cpp
#include <bits/stdc++.h>
using namespace std;

class CriticalConnections {
    vector<vector<pair<int,int>>> adj;   // adj[u] = list of {neighbor, edgeId}
    vector<int> disc, low;
    vector<bool> visited;
    int timer_ = 0;
    vector<vector<int>> bridges;

public:
    vector<vector<int>> criticalConnections(int n, vector<vector<int>>& connections) {
        adj.assign(n, {});
        int id = 0;
        for (auto& c : connections) {
            int u = c[0], v = c[1];
            adj[u].push_back({v, id});
            adj[v].push_back({u, id});
            id++;
        }
        disc.resize(n);
        low.resize(n);
        visited.assign(n, false);

        for (int i = 0; i < n; i++)
            if (!visited[i]) dfs(i, -1);   // -1 = no incoming edge id
        return bridges;
    }

private:
    void dfs(int u, int parentEdgeId) {
        visited[u] = true;
        disc[u] = low[u] = timer_++;
        for (auto& [v, eid] : adj[u]) {
            if (eid == parentEdgeId) continue;     // don't go back over the same edge
            if (!visited[v]) {
                dfs(v, eid);
                low[u] = min(low[u], low[v]);
                if (low[v] > disc[u]) {            // bridge condition
                    bridges.push_back({u, v});
                }
            } else {
                low[u] = min(low[u], disc[v]); // back edge
            }
        }
    }
};
```

### Detailed Dry-Run

Take `n = 4`, `connections = [[0,1],[1,2],[2,0],[1,3]]`. Edge ids: `0=(0,1)`, `1=(1,2)`, `2=(2,0)`, `3=(1,3)`. Visually: nodes `0,1,2` form a triangle; node `3` hangs off node `1`. The only bridge should be `(1,3)`.

Start `dfs(0, -1)`:

| Step | Action | disc/low updates |
| --- | --- | --- |
| 1 | Enter `0`. `timer=0` | `disc[0]=low[0]=0`, `timer→1` |
| 2 | From `0`, edge to `1` (eid 0, not parent). `1` unvisited → recurse `dfs(1, 0)` | — |
| 3 | Enter `1`. | `disc[1]=low[1]=1`, `timer→2` |
| 4 | From `1`, edge to `0` (eid 0) = parentEdgeId → **skip** | — |
| 5 | From `1`, edge to `2` (eid 1). `2` unvisited → recurse `dfs(2, 1)` | — |
| 6 | Enter `2`. | `disc[2]=low[2]=2`, `timer→3` |
| 7 | From `2`, edge to `1` (eid 1) = parentEdgeId → **skip** | — |
| 8 | From `2`, edge to `0` (eid 2). `0` visited, not parent edge → **back edge** | `low[2]=min(2, disc[0]=0)=0` |
| 9 | `2` done. Return to `1`. | `low[1]=min(1, low[2]=0)=0`; check bridge `low[2]=0 > disc[1]=1`? No |
| 10 | From `1`, edge to `3` (eid 3). `3` unvisited → recurse `dfs(3, 3)` | — |
| 11 | Enter `3`. | `disc[3]=low[3]=3`, `timer→4` |
| 12 | From `3`, edge to `1` (eid 3) = parentEdgeId → **skip**. No other edges. `3` done. | — |
| 13 | Return to `1`. | `low[1]=min(0, low[3]=3)=0`; check bridge `low[3]=3 > disc[1]=1`? **Yes → add (1,3)** |
| 14 | `1` done. Return to `0`. | `low[0]=min(0, low[1]=0)=0`; check bridge `low[1]=0 > disc[0]=0`? No |
| 15 | From `0`, edge to `2` (eid 2). `2` visited, not parent edge → back edge | `low[0]=min(0, disc[2]=2)=0` (no change) |
| 16 | `0` done. | — |

Final: `disc=[0,1,2,3]`, `low=[0,0,0,3]`. Only `(1,3)` satisfied `low[v] > disc[u]`, so **the single bridge is `[1,3]`** — matching intuition, since removing any triangle edge leaves the triangle connected, but removing `(1,3)` isolates node `3`.

**Complexity:** `O(V + E)` time, `O(V + E)` space.

---

## 53. Find if Path Exists in Graph — Easy (LC1971)

Given an undirected graph with `n` vertices and edge list, determine whether a path exists between `source` and `destination`.

**Approach A — DSU.** Union every edge, then `source` and `destination` are connected iff they share a root.

```cpp
#include <bits/stdc++.h>
using namespace std;

class PathExists {
    vector<int> parent;

    int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]);
        return parent[x];
    }

    void unite(int x, int y) {
        parent[find(x)] = find(y);
    }

public:
    bool validPath(int n, vector<vector<int>>& edges, int source, int destination) {
        parent.resize(n);
        iota(parent.begin(), parent.end(), 0);
        for (auto& e : edges) unite(e[0], e[1]);
        return find(source) == find(destination);
    }

    // Approach B — BFS reachability
    bool validPathBFS(int n, vector<vector<int>>& edges, int source, int destination) {
        vector<vector<int>> adj(n);
        for (auto& e : edges) { adj[e[0]].push_back(e[1]); adj[e[1]].push_back(e[0]); }

        vector<bool> visited(n, false);
        queue<int> q;
        q.push(source);
        visited[source] = true;
        while (!q.empty()) {
            int u = q.front(); q.pop();
            if (u == destination) return true;
            for (int v : adj[u]) {
                if (!visited[v]) { visited[v] = true; q.push(v); }
            }
        }
        return false;
    }
};
```

**Complexity:** DSU `O(V + E·α(V))`; BFS `O(V + E)`.

---

> **Last Updated:** 2026-06-26
