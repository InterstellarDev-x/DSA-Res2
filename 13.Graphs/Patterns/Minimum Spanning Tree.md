> **Topic:** [Graphs](../README.md) · **Pattern 5 of 7**

# Minimum Spanning Tree

A **Minimum Spanning Tree (MST)** of a connected, undirected, weighted graph is a subset of edges that connects **all vertices** together, with **no cycles**, and with the **minimum possible total edge weight**.

Key properties:
- A spanning tree of a graph with `V` vertices always has exactly `V - 1` edges.
- If the graph is disconnected, no spanning tree exists (you instead get a *minimum spanning forest*).
- Edge weights need not be distinct, but if they are, the MST is unique.

There are two classic greedy algorithms. Both are correct because of the **cut property**: for any partition of the vertices into two sets, the minimum-weight edge crossing the partition is part of some MST.

---

## Prim's vs Kruskal's

| | Prim's | Kruskal's |
|---|---|---|
| **Strategy** | Grow a single tree outward from a starting vertex | Build a forest by adding globally cheapest edges |
| **Data structure** | `PriorityQueue` (min-heap of candidate edges) | Sorted edge list + **Union-Find (DSU)** |
| **Cycle handling** | Track visited vertices; skip a popped edge if its endpoint is already in the tree | Skip an edge if both endpoints are already in the same DSU component |
| **Best for** | **Dense** graphs (e.g. complete graphs); when edges are given implicitly | **Sparse** graphs; when edges are given as an explicit list |
| **Time** | `O(E log V)` with a binary heap | `O(E log E)` dominated by the sort |
| **Space** | `O(V + E)` | `O(V + E)` |

**Prim's** picks a start vertex, pushes its outgoing edges into a min-heap, then repeatedly pops the cheapest edge that reaches a *new* vertex, adds that vertex to the tree, and pushes its outgoing edges. It finishes when all vertices are in the tree.

**Kruskal's** sorts every edge by weight ascending, then walks the list: for each edge it uses DSU `find` to check whether the two endpoints are already connected. If they are, adding the edge would create a cycle, so it is skipped. Otherwise it `union`s them and counts the edge into the MST. It finishes after `V - 1` edges are accepted.

---

## Prim's — Java Template

```java
import java.util.*;

class PrimMST {
    // adj[u] = list of {v, weight}; vertices are 0..n-1
    public int prim(int n, List<int[]>[] adj) {
        boolean[] inTree = new boolean[n];
        // min-heap ordered by edge weight: each entry is {weight, vertex}
        PriorityQueue<int[]> pq =
            new PriorityQueue<>((a, b) -> Integer.compare(a[0], b[0]));

        pq.offer(new int[]{0, 0}); // start from vertex 0 with cost 0
        int totalWeight = 0;
        int verticesUsed = 0;

        while (!pq.isEmpty() && verticesUsed < n) {
            int[] top = pq.poll();
            int w = top[0], u = top[1];

            if (inTree[u]) continue;   // already connected — stale heap entry
            inTree[u] = true;
            totalWeight += w;
            verticesUsed++;

            for (int[] edge : adj[u]) {
                int v = edge[0], weight = edge[1];
                if (!inTree[v]) {
                    pq.offer(new int[]{weight, v});
                }
            }
        }

        // If verticesUsed < n the graph was disconnected.
        return verticesUsed == n ? totalWeight : -1;
    }
}
```

---

## Kruskal's — Java Template (with inline DSU)

The DSU below uses **path compression** in `find` and **union by rank** in `union`. For a deeper treatment of the structure itself see [Union Find (DSU)](./Union%20Find%20(DSU).md).

```java
import java.util.*;

class KruskalMST {
    // edges[i] = {u, v, weight}
    public int kruskal(int n, int[][] edges) {
        Arrays.sort(edges, (a, b) -> Integer.compare(a[2], b[2]));

        DSU dsu = new DSU(n);
        int totalWeight = 0;
        int edgesUsed = 0;

        for (int[] e : edges) {
            int u = e[0], v = e[1], w = e[2];
            if (dsu.union(u, v)) {   // union returns false if it would form a cycle
                totalWeight += w;
                edgesUsed++;
                if (edgesUsed == n - 1) break; // tree complete
            }
        }

        return edgesUsed == n - 1 ? totalWeight : -1; // -1 if disconnected
    }

    // Inline Disjoint Set Union — path compression + union by rank
    static class DSU {
        int[] parent, rank;

        DSU(int n) {
            parent = new int[n];
            rank = new int[n];
            for (int i = 0; i < n; i++) parent[i] = i;
        }

        int find(int x) {
            if (parent[x] != x) parent[x] = find(parent[x]); // path compression
            return parent[x];
        }

        // returns true if the two were in different sets (a real union happened)
        boolean union(int a, int b) {
            int ra = find(a), rb = find(b);
            if (ra == rb) return false;
            if (rank[ra] < rank[rb]) { int t = ra; ra = rb; rb = t; }
            parent[rb] = ra;
            if (rank[ra] == rank[rb]) rank[ra]++;
            return true;
        }
    }
}
```

---

## Recognition Signals

| Signal in the problem | What it suggests |
|---|---|
| "Connect all points / cities / nodes at minimum total cost" | MST |
| "Every node must be reachable" + minimize sum of chosen edge weights | MST (not shortest path) |
| Weighted **undirected** graph, choose a subset of edges | MST |
| Complete graph / coordinates with implicit edge weights (Manhattan, Euclidean) | MST, prefer **Prim's** (dense) |
| Explicit edge list, possibly sparse | MST, prefer **Kruskal's** |
| "Each node has a build cost OR a connection cost" | Add a **virtual node**, then MST |
| "Return -1 if it cannot all be connected" | MST with a connectivity check |
| "Which edges are essential / which are interchangeable" | MST + critical/pseudo-critical analysis |

Note: if the goal were "minimize distance from a source to every node," that is **shortest path (Dijkstra)**, not MST. MST minimizes the *total* edge weight of the connecting tree, not path lengths.

---

## 34. Min Cost to Connect All Points — Medium (LC1584)

**Idea:** The points form a complete graph where the weight between two points is their Manhattan distance; this dense graph is a perfect fit for **Prim's** (no need to materialize all `O(n^2)` edges into a sorted list).

```java
import java.util.*;

class Solution {
    public int minCostConnectPoints(int[][] points) {
        int n = points.length;
        if (n <= 1) return 0;

        boolean[] inTree = new boolean[n];
        // min-heap entries: {cost, pointIndex}
        PriorityQueue<int[]> pq =
            new PriorityQueue<>((a, b) -> Integer.compare(a[0], b[0]));

        pq.offer(new int[]{0, 0}); // start at point 0
        int total = 0;
        int used = 0;

        while (!pq.isEmpty() && used < n) {
            int[] top = pq.poll();
            int cost = top[0], u = top[1];
            if (inTree[u]) continue;

            inTree[u] = true;
            total += cost;
            used++;

            // push edges to every not-yet-connected point (implicit complete graph)
            for (int v = 0; v < n; v++) {
                if (!inTree[v]) {
                    int dist = Math.abs(points[u][0] - points[v][0])
                             + Math.abs(points[u][1] - points[v][1]);
                    pq.offer(new int[]{dist, v});
                }
            }
        }

        return total;
    }
}
```

**Complexity:** Time `O(n^2 log n)` (each of `n` expansions pushes up to `n` heap entries). Space `O(n^2)` for the heap in the worst case.

**Kruskal alternative:** Build all `n(n-1)/2` edges as `{i, j, manhattanDist}`, sort them, and run the Kruskal template above. This is `O(n^2 log n)` for the sort and works fine, but it allocates the full edge list, so Prim's is the cleaner primary choice for this dense graph.

**Dry run (Kruskal) on `points = [[0,0],[2,2],[3,10],[5,2],[7,0]]`** (label points P0..P4):

First, the relevant Manhattan distances (`|dx| + |dy|`):

| Edge | Distance |
|---|---|
| P0–P1 | \|0-2\|+\|0-2\| = 4 |
| P0–P3 | \|0-5\|+\|0-2\| = 7 |
| P0–P4 | \|0-7\|+\|0-0\| = 7 |
| P1–P3 | \|2-5\|+\|2-2\| = 3 |
| P1–P2 | \|2-3\|+\|2-10\| = 9 |
| P3–P4 | \|5-7\|+\|2-0\| = 4 |
| P2–P3 | \|3-5\|+\|10-2\| = 10 |
| P2–P4 | \|3-7\|+\|10-0\| = 14 |
| P0–P2 | \|0-3\|+\|0-10\| = 13 |
| P1–P4 | \|2-7\|+\|2-0\| = 7 |

Sort ascending and process (DSU starts as 5 singletons `{P0}{P1}{P2}{P3}{P4}`):

1. **P1–P3 (3):** different sets → **accept**. union → `{P1,P3}`. total = 3, edges = 1.
2. **P0–P1 (4):** P0 and P1 in different sets → **accept**. union → `{P0,P1,P3}`. total = 7, edges = 2.
3. **P3–P4 (4):** P3 in `{P0,P1,P3}`, P4 alone → **accept**. union → `{P0,P1,P3,P4}`. total = 11, edges = 3.
4. **P0–P3 (7):** both already in `{P0,P1,P3,P4}` → **cycle, skip**.
5. **P0–P4 (7):** both in same set → **cycle, skip**.
6. **P1–P4 (7):** both in same set → **cycle, skip**.
7. **P1–P2 (9):** P1 in big set, P2 alone → **accept**. union → all 5 connected. total = 20, edges = 4.

We have accepted `V - 1 = 4` edges, so we stop. **MST cost = 20**, which matches the expected answer for this input.

---

## 35. Connecting Cities With Minimum Cost — Medium (LC1135)

**Idea:** Classic **Kruskal's** on the explicit connection list; after processing, return the total only if all `n` cities ended up in one component, else `-1`.

```java
import java.util.*;

class Solution {
    public int minimumCost(int n, int[][] connections) {
        // cities are 1..n — DSU sized n+1, index 0 unused
        Arrays.sort(connections, (a, b) -> Integer.compare(a[2], b[2]));

        DSU dsu = new DSU(n + 1);
        int total = 0;
        int edgesUsed = 0;

        for (int[] c : connections) {
            int u = c[0], v = c[1], cost = c[2];
            if (dsu.union(u, v)) {
                total += cost;
                edgesUsed++;
                if (edgesUsed == n - 1) break;
            }
        }

        return edgesUsed == n - 1 ? total : -1;
    }

    static class DSU {
        int[] parent, rank;

        DSU(int n) {
            parent = new int[n];
            rank = new int[n];
            for (int i = 0; i < n; i++) parent[i] = i;
        }

        int find(int x) {
            if (parent[x] != x) parent[x] = find(parent[x]);
            return parent[x];
        }

        boolean union(int a, int b) {
            int ra = find(a), rb = find(b);
            if (ra == rb) return false;
            if (rank[ra] < rank[rb]) { int t = ra; ra = rb; rb = t; }
            parent[rb] = ra;
            if (rank[ra] == rank[rb]) rank[ra]++;
            return true;
        }
    }
}
```

**Complexity:** Time `O(E log E)` for the sort (E = number of connections), with near-`O(1)` amortized DSU operations. Space `O(n)`.

---

## 36. Optimize Water Distribution in a Village — Hard (LC1168)

**Idea:** Each house can either dig its own well (a cost) or connect to a neighbor via a pipe. Model the well option by adding a **virtual node 0** with an edge of weight `wells[i]` to each house `i`; then the answer is simply the **MST** over houses `1..n` plus node `0` (Kruskal's).

```java
import java.util.*;

class Solution {
    public int minCostToSupplyWater(int n, int[] wells, int[][] pipes) {
        // Build edge list. Houses are 1..n; node 0 is the virtual "water source".
        List<int[]> edges = new ArrayList<>();

        // well[i] becomes an edge from virtual node 0 to house (i+1)
        for (int i = 0; i < n; i++) {
            edges.add(new int[]{0, i + 1, wells[i]});
        }
        for (int[] p : pipes) {
            edges.add(new int[]{p[0], p[1], p[2]});
        }

        edges.sort((a, b) -> Integer.compare(a[2], b[2]));

        DSU dsu = new DSU(n + 1);      // indices 0..n
        int total = 0;
        int edgesUsed = 0;

        for (int[] e : edges) {
            if (dsu.union(e[0], e[1])) {
                total += e[2];
                edgesUsed++;
                if (edgesUsed == n) break; // n+1 nodes => MST has n edges
            }
        }

        return total;
    }

    static class DSU {
        int[] parent, rank;

        DSU(int n) {
            parent = new int[n];
            rank = new int[n];
            for (int i = 0; i < n; i++) parent[i] = i;
        }

        int find(int x) {
            if (parent[x] != x) parent[x] = find(parent[x]);
            return parent[x];
        }

        boolean union(int a, int b) {
            int ra = find(a), rb = find(b);
            if (ra == rb) return false;
            if (rank[ra] < rank[rb]) { int t = ra; ra = rb; rb = t; }
            parent[rb] = ra;
            if (rank[ra] == rank[rb]) rank[ra]++;
            return true;
        }
    }
}
```

**Why the virtual node works:** "Build a well at house `i`" is equivalent to "connect house `i` to a single shared source." Adding node `0` makes every house's well a real edge, so the cheapest way to supply water to every house is exactly the minimum spanning tree of the `n + 1` nodes.

**Complexity:** Time `O((n + P) log (n + P))` where `P = pipes.length`, dominated by the sort. Space `O(n + P)`.

---

## 37. Find Critical and Pseudo-Critical Edges in MST — Hard (LC1489)

**Idea:** Compute the **base MST weight** with Kruskal's. For each edge: it is **critical** if removing it (running Kruskal without it) raises the MST weight or disconnects the graph; it is **pseudo-critical** if forcing it in first and then completing the MST still yields the base weight (but it is not critical).

```java
import java.util.*;

class Solution {
    public List<List<Integer>> findCriticalAndPseudoCriticalEdges(int n, int[][] edges) {
        int m = edges.length;
        // Track original indices since we will sort.
        int[][] indexed = new int[m][4];
        for (int i = 0; i < m; i++) {
            indexed[i][0] = edges[i][0];
            indexed[i][1] = edges[i][1];
            indexed[i][2] = edges[i][2];
            indexed[i][3] = i; // original index
        }
        Arrays.sort(indexed, (a, b) -> Integer.compare(a[2], b[2]));

        int baseWeight = buildMST(n, indexed, -1, -1);

        List<Integer> critical = new ArrayList<>();
        List<Integer> pseudo = new ArrayList<>();

        for (int i = 0; i < m; i++) {
            // 1) Exclude edge i: if MST weight grows or graph disconnects -> critical.
            int without = buildMST(n, indexed, i, -1);
            if (without > baseWeight) {
                critical.add(indexed[i][3]);
                continue;
            }
            // 2) Force edge i in: if total still equals base -> pseudo-critical.
            int with = buildMST(n, indexed, -1, i);
            if (with == baseWeight) {
                pseudo.add(indexed[i][3]);
            }
        }

        return Arrays.asList(critical, pseudo);
    }

    // Runs Kruskal on the pre-sorted edges.
    // skip: index (in sorted array) to exclude, or -1.
    // force: index (in sorted array) to add first, or -1.
    // Returns total weight, or Integer.MAX_VALUE if the graph stays disconnected.
    private int buildMST(int n, int[][] sorted, int skip, int force) {
        DSU dsu = new DSU(n);
        int weight = 0;
        int count = 0;

        if (force != -1) {
            dsu.union(sorted[force][0], sorted[force][1]);
            weight += sorted[force][2];
            count++;
        }

        for (int i = 0; i < sorted.length; i++) {
            if (i == skip) continue;
            if (dsu.union(sorted[i][0], sorted[i][1])) {
                weight += sorted[i][2];
                count++;
            }
        }

        return count == n - 1 ? weight : Integer.MAX_VALUE; // disconnected
    }

    static class DSU {
        int[] parent, rank;

        DSU(int n) {
            parent = new int[n];
            rank = new int[n];
            for (int i = 0; i < n; i++) parent[i] = i;
        }

        int find(int x) {
            if (parent[x] != x) parent[x] = find(parent[x]);
            return parent[x];
        }

        boolean union(int a, int b) {
            int ra = find(a), rb = find(b);
            if (ra == rb) return false;
            if (rank[ra] < rank[rb]) { int t = ra; ra = rb; rb = t; }
            parent[rb] = ra;
            if (rank[ra] == rank[rb]) rank[ra]++;
            return true;
        }
    }
}
```

**Reasoning recap:**
- **Critical edge:** every MST must include it. Detected because excluding it forces a costlier alternative (weight increases) or makes the graph disconnected (`Integer.MAX_VALUE > baseWeight`).
- **Pseudo-critical edge:** appears in *some* MST but not all. Detected because forcing it in still reaches the base weight, yet it was not flagged critical.
- An edge is neither if forcing it in raises the total above the base weight.

**Complexity:** Time `O(E^2 · α(V))` — for each of `E` edges we run a near-linear Kruskal pass (DSU operations are amortized `α(V)`, effectively constant). Space `O(E + V)`.

---

> **Last Updated:** 2026-06-26
