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

```java
class DSU {
    int[] parent, rank;

    DSU(int n) {
        parent = new int[n];
        rank = new int[n];
        for (int i = 0; i < n; i++) parent[i] = i;
    }

    int find(int x) {                 // path compression
        if (parent[x] != x) parent[x] = find(parent[x]);
        return parent[x];
    }

    // returns false if x,y already connected (=> cycle); true if merged
    boolean union(int x, int y) {
        int rx = find(x), ry = find(y);
        if (rx == ry) return false;
        if (rank[rx] < rank[ry])      { parent[rx] = ry; }
        else if (rank[rx] > rank[ry]) { parent[ry] = rx; }
        else                          { parent[ry] = rx; rank[rx]++; }
        return true;
    }
}

public class UndirectedCycleDSU {
    // edges given as int[][] where each row is {u, v}, vertices 0..n-1
    public boolean hasCycle(int n, int[][] edges) {
        DSU dsu = new DSU(n);
        for (int[] e : edges) {
            if (!dsu.union(e[0], e[1])) return true; // both share root => cycle
        }
        return false;
    }
}
```

### 2b. Via DFS with Parent Tracking

Walk the graph; if you reach a visited neighbor that isn't the parent, it's a back edge → cycle. Handle disconnected graphs by starting DFS from every unvisited vertex.

```java
import java.util.*;

public class UndirectedCycleDFS {
    public boolean hasCycle(int n, List<List<Integer>> adj) {
        boolean[] visited = new boolean[n];
        for (int i = 0; i < n; i++) {
            if (!visited[i] && dfs(i, -1, adj, visited)) return true;
        }
        return false;
    }

    private boolean dfs(int node, int parent, List<List<Integer>> adj, boolean[] visited) {
        visited[node] = true;
        for (int nei : adj.get(node)) {
            if (!visited[nei]) {
                if (dfs(nei, node, adj, visited)) return true;
            } else if (nei != parent) {   // visited and not the node we came from
                return true;
            }
        }
        return false;
    }
}
```

> **BFS variant note:** The same idea works iteratively with BFS by storing `(node, parent)` pairs in the queue and flagging a cycle when a neighbor is already visited and isn't the parent. With parallel edges, track the edge instead of the parent vertex.

---

## 3. Directed Cycle Detection

### 3a. Via 3-Color DFS

Colors: `0 = white` (unvisited), `1 = gray` (in current recursion stack), `2 = black` (fully processed). A cycle exists exactly when DFS hits a **gray** node.

```java
import java.util.*;

public class DirectedCycle3Color {
    static final int WHITE = 0, GRAY = 1, BLACK = 2;

    public boolean hasCycle(int n, List<List<Integer>> adj) {
        int[] color = new int[n];          // all WHITE initially
        for (int i = 0; i < n; i++) {
            if (color[i] == WHITE && dfs(i, adj, color)) return true;
        }
        return false;
    }

    private boolean dfs(int node, List<List<Integer>> adj, int[] color) {
        color[node] = GRAY;                 // entering recursion stack
        for (int nei : adj.get(node)) {
            if (color[nei] == GRAY) return true;          // back edge => cycle
            if (color[nei] == WHITE && dfs(nei, adj, color)) return true;
            // color[nei] == BLACK => already done, safe to skip
        }
        color[node] = BLACK;                // leaving recursion stack
        return false;
    }
}
```

### 3b. Via Kahn's Algorithm (BFS Topological Sort)

Peel off indegree-0 vertices. If the number processed is less than `n`, the leftovers are stuck in a cycle.

```java
import java.util.*;

public class DirectedCycleKahn {
    public boolean hasCycle(int n, List<List<Integer>> adj) {
        int[] indeg = new int[n];
        for (int u = 0; u < n; u++)
            for (int v : adj.get(u)) indeg[v]++;

        Queue<Integer> q = new ArrayDeque<>();
        for (int i = 0; i < n; i++)
            if (indeg[i] == 0) q.offer(i);

        int processed = 0;
        while (!q.isEmpty()) {
            int u = q.poll();
            processed++;
            for (int v : adj.get(u)) {
                if (--indeg[v] == 0) q.offer(v);
            }
        }
        return processed < n;   // couldn't topologically order everything => cycle
    }
}
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

```java
import java.util.*;

public class DetectCycleUndirected {
    int[] parent, rank;

    int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]);
        return parent[x];
    }

    boolean union(int x, int y) {
        int rx = find(x), ry = find(y);
        if (rx == ry) return false;            // already connected => cycle
        if (rank[rx] < rank[ry]) parent[rx] = ry;
        else if (rank[rx] > rank[ry]) parent[ry] = rx;
        else { parent[ry] = rx; rank[rx]++; }
        return true;
    }

    public boolean hasCycle(int n, int[][] edges) {
        parent = new int[n];
        rank = new int[n];
        for (int i = 0; i < n; i++) parent[i] = i;
        for (int[] e : edges) {
            if (!union(e[0], e[1])) return true;
        }
        return false;
    }

    // Approach B — DFS with parent tracking (handles disconnected components)
    public boolean hasCycleDFS(int n, List<List<Integer>> adj) {
        boolean[] visited = new boolean[n];
        for (int i = 0; i < n; i++)
            if (!visited[i] && dfs(i, -1, adj, visited)) return true;
        return false;
    }

    private boolean dfs(int node, int parent, List<List<Integer>> adj, boolean[] visited) {
        visited[node] = true;
        for (int nei : adj.get(node)) {
            if (!visited[nei]) {
                if (dfs(nei, node, adj, visited)) return true;
            } else if (nei != parent) {
                return true;
            }
        }
        return false;
    }
}
```

**Complexity:** DSU runs in `O(E · α(N))`; DFS in `O(V + E)`.

---

## 48. Detect Cycle in Directed Graph — Medium (LC——)

Given a directed graph, determine whether it contains a directed cycle. Use 3-color DFS: a cycle exists iff DFS reaches a vertex currently on the recursion stack (gray).

```java
import java.util.*;

public class DetectCycleDirected {
    static final int WHITE = 0, GRAY = 1, BLACK = 2;

    public boolean hasCycle(int n, List<List<Integer>> adj) {
        int[] color = new int[n];
        for (int i = 0; i < n; i++) {
            if (color[i] == WHITE && dfs(i, adj, color)) return true;
        }
        return false;
    }

    private boolean dfs(int node, List<List<Integer>> adj, int[] color) {
        color[node] = GRAY;
        for (int nei : adj.get(node)) {
            if (color[nei] == GRAY) return true;                 // back edge
            if (color[nei] == WHITE && dfs(nei, adj, color)) return true;
        }
        color[node] = BLACK;
        return false;
    }
}
```

**Why parent-tracking fails here:** In a directed graph `a→b` and `b→a` is a genuine 2-cycle, so we cannot "excuse the parent." The recursion-stack (gray) check correctly distinguishes a back edge (ancestor on the stack) from a harmless cross edge (already-finished black node).

**Complexity:** `O(V + E)` time, `O(V)` space.

---

## 49. Find Eventual Safe States — Medium (LC802)

> **Cross-reference.** This problem is covered in detail in [Topological Sort](./Topological%20Sort.md). A node is *safe* if every path from it leads to a terminal node — equivalently, if it is **not** part of and cannot reach any cycle. The standard solution is a 3-color DFS (a node is safe iff its DFS subtree contains no gray/back edge) or a reverse-graph Kahn's algorithm. See Topological Sort for the full walkthrough.

Brief snippet (3-color DFS; safe ⟺ no cycle reachable):

```java
public List<Integer> eventualSafeNodes(int[][] graph) {
    int n = graph.length;
    int[] color = new int[n]; // 0 white, 1 gray, 2 black(safe)
    List<Integer> res = new ArrayList<>();
    for (int i = 0; i < n; i++) if (safe(i, graph, color)) res.add(i);
    return res;
}
private boolean safe(int u, int[][] g, int[] color) {
    if (color[u] != 0) return color[u] == 2; // gray => unsafe, black => safe
    color[u] = 1;
    for (int v : g[u]) if (!safe(v, g, color)) return false;
    color[u] = 2;
    return true;
}
```

See [Topological Sort](./Topological%20Sort.md) for the detailed reverse-Kahn treatment.

---

## 50. Course Schedule — Medium (LC207)

> **Cross-reference.** Covered fully in [Topological Sort](./Topological%20Sort.md). `numCourses` with prerequisite pairs `[a, b]` (take `b` before `a`) is solvable iff the dependency graph is a **DAG** — i.e., it has no directed cycle. Detect with Kahn's algorithm or 3-color DFS.

Brief snippet (Kahn's):

```java
public boolean canFinish(int numCourses, int[][] prerequisites) {
    List<List<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < numCourses; i++) adj.add(new ArrayList<>());
    int[] indeg = new int[numCourses];
    for (int[] p : prerequisites) { adj.get(p[1]).add(p[0]); indeg[p[0]]++; }

    Queue<Integer> q = new ArrayDeque<>();
    for (int i = 0; i < numCourses; i++) if (indeg[i] == 0) q.offer(i);
    int done = 0;
    while (!q.isEmpty()) {
        int u = q.poll(); done++;
        for (int v : adj.get(u)) if (--indeg[v] == 0) q.offer(v);
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

```java
import java.util.*;

public class RedundantConnectionII {
    int[] parent;

    int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]);
        return parent[x];
    }

    public int[] findRedundantDirectedConnection(int[][] edges) {
        int n = edges.length;
        parent = new int[n + 1];
        int[] incomingParent = new int[n + 1]; // incomingParent[v] = which edge index gave v its parent

        int[] cand1 = null, cand2 = null;      // two edges into the same node
        for (int i = 0; i < n; i++) {
            int u = edges[i][0], v = edges[i][1];
            if (incomingParent[v] != 0) {       // v already has a parent edge
                cand1 = edges[incomingParent[v] - 1]; // earlier edge into v
                cand2 = edges[i];                     // current edge into v
            } else {
                incomingParent[v] = i + 1;      // store 1-based index to keep 0 as "none"
            }
        }

        for (int i = 0; i <= n; i++) parent[i] = i;

        // Run union over all edges except cand2 (if it exists)
        for (int i = 0; i < n; i++) {
            if (cand2 != null && edges[i] == cand2) continue; // skip the second parent edge
            int u = edges[i][0], v = edges[i][1];
            int ru = find(u), rv = find(v);
            if (ru == rv) {
                // a cycle formed without cand2
                if (cand1 == null) return edges[i]; // pure cycle, no indegree-2 node
                return cand1;                       // indegree-2 case: cand1 is the bad edge
            }
            parent[rv] = ru;
        }
        // No cycle without cand2 => cand2 was the redundant edge
        return cand2;
    }
}
```

**Walking the cases:**
- **Only a cycle, no two-parent node:** `cand1`/`cand2` stay null; the union loop finds the closing edge and returns it.
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

```java
import java.util.*;

public class CriticalConnections {
    private List<List<int[]>> adj;   // adj.get(u) = list of {neighbor, edgeId}
    private int[] disc, low;
    private boolean[] visited;
    private int timer = 0;
    private List<List<Integer>> bridges = new ArrayList<>();

    public List<List<Integer>> criticalConnections(int n, List<List<Integer>> connections) {
        adj = new ArrayList<>();
        for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
        int id = 0;
        for (List<Integer> c : connections) {
            int u = c.get(0), v = c.get(1);
            adj.get(u).add(new int[]{v, id});
            adj.get(v).add(new int[]{u, id});
            id++;
        }
        disc = new int[n];
        low = new int[n];
        visited = new boolean[n];

        for (int i = 0; i < n; i++)
            if (!visited[i]) dfs(i, -1);   // -1 = no incoming edge id
        return bridges;
    }

    private void dfs(int u, int parentEdgeId) {
        visited[u] = true;
        disc[u] = low[u] = timer++;
        for (int[] edge : adj.get(u)) {
            int v = edge[0], eid = edge[1];
            if (eid == parentEdgeId) continue;     // don't go back over the same edge
            if (!visited[v]) {
                dfs(v, eid);
                low[u] = Math.min(low[u], low[v]);
                if (low[v] > disc[u]) {            // bridge condition
                    bridges.add(Arrays.asList(u, v));
                }
            } else {
                low[u] = Math.min(low[u], disc[v]); // back edge
            }
        }
    }
}
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

```java
import java.util.*;

public class PathExists {
    int[] parent;

    int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]);
        return parent[x];
    }

    void union(int x, int y) {
        parent[find(x)] = find(y);
    }

    public boolean validPath(int n, int[][] edges, int source, int destination) {
        parent = new int[n];
        for (int i = 0; i < n; i++) parent[i] = i;
        for (int[] e : edges) union(e[0], e[1]);
        return find(source) == find(destination);
    }

    // Approach B — BFS reachability
    public boolean validPathBFS(int n, int[][] edges, int source, int destination) {
        List<List<Integer>> adj = new ArrayList<>();
        for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
        for (int[] e : edges) { adj.get(e[0]).add(e[1]); adj.get(e[1]).add(e[0]); }

        boolean[] visited = new boolean[n];
        Queue<Integer> q = new ArrayDeque<>();
        q.offer(source);
        visited[source] = true;
        while (!q.isEmpty()) {
            int u = q.poll();
            if (u == destination) return true;
            for (int v : adj.get(u)) {
                if (!visited[v]) { visited[v] = true; q.offer(v); }
            }
        }
        return false;
    }
}
```

**Complexity:** DSU `O(V + E·α(V))`; BFS `O(V + E)`.

---

> **Last Updated:** 2026-06-26
