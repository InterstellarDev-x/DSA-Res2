[← Binary Search Trees](../Binary%20Search%20Trees/README.md) · **Graphs (Step 15)** · [Dynamic Programming →](../14.Dynamic_Programming/README.md)

# Graphs

A complete, production-quality reference for the **Graphs** topic — 53 problems across 7 core patterns, each with full C++ solutions, complexity analysis, dry-runs, and recognition signals. Graphs unify a huge slice of interview questions: connectivity, ordering with dependencies, shortest paths, and spanning structures all reduce to a handful of reusable templates.

---

## Patterns

| # | Pattern | Core idea | Link |
|---|---------|-----------|------|
| 1 | Graph Traversal (BFS / DFS) | Explore reachable nodes; BFS for shortest unweighted distance / levels, DFS for connectivity & exhaustive search. | [Graph Traversal (BFS DFS)](./Patterns/Graph%20Traversal%20(BFS%20DFS).md) |
| 2 | Grid and Matrix BFS | Treat a grid as an implicit graph; multi-source BFS seeds all sources at once and expands level by level. | [Grid and Matrix BFS](./Patterns/Grid%20and%20Matrix%20BFS.md) |
| 3 | Topological Sort | Linear ordering of a DAG respecting dependencies; Kahn's (indegree BFS) or DFS postorder; detects cycles. | [Topological Sort](./Patterns/Topological%20Sort.md) |
| 4 | Shortest Path | Choose the algorithm by edge weights: BFS / Dijkstra / Bellman-Ford / Floyd-Warshall / 0-1 BFS. | [Shortest Path](./Patterns/Shortest%20Path.md) |
| 5 | Minimum Spanning Tree | Cheapest set of edges connecting all vertices; Prim's (grow from a vertex) or Kruskal's (sort + DSU). | [Minimum Spanning Tree](./Patterns/Minimum%20Spanning%20Tree.md) |
| 6 | Union Find (DSU) | Dynamic connectivity & grouping in near O(1) amortized via path compression + union by rank. | [Union Find (DSU)](./Patterns/Union%20Find%20(DSU).md) |
| 7 | Cycle Detection | Find cycles in directed (3-color DFS / Kahn's) and undirected (DSU / DFS parent) graphs; Tarjan bridges. | [Cycle Detection](./Patterns/Cycle%20Detection.md) |

---

## All 53 Problems

| # | Problem | LC | Difficulty | Pattern | Companies |
|---|---------|----|------------|---------|-----------|
| 1 | Number of Provinces | 547 | Medium | Traversal | Amazon, Microsoft |
| 2 | Number of Islands | 200 | Medium | Traversal | Amazon, Google, Microsoft, Adobe |
| 3 | Flood Fill | 733 | Easy | Traversal | Amazon, Microsoft |
| 4 | Clone Graph | 133 | Medium | Traversal | Google, Amazon, Adobe |
| 5 | Surrounded Regions | 130 | Medium | Traversal | Google, Amazon |
| 6 | Pacific Atlantic Water Flow | 417 | Medium | Traversal | Google, Amazon |
| 7 | Is Graph Bipartite | 785 | Medium | Traversal | Amazon, Google |
| 8 | Word Ladder | 127 | Hard | Traversal | Amazon, Google, Microsoft |
| 9 | Word Ladder II | 126 | Hard | Traversal | Amazon, Google |
| 10 | Open the Lock | 752 | Medium | Traversal | Google, Microsoft |
| 11 | Rotting Oranges | 994 | Medium | Grid BFS | Amazon, Microsoft, Google |
| 12 | 01 Matrix | 542 | Medium | Grid BFS | Google, Amazon |
| 13 | Walls and Gates | 286 | Medium | Grid BFS | Google, Amazon, Microsoft |
| 14 | Shortest Path in Binary Matrix | 1091 | Medium | Grid BFS | Amazon, Goldman Sachs |
| 15 | As Far from Land as Possible | 1162 | Medium | Grid BFS | Amazon, Google |
| 16 | Shortest Bridge | 934 | Medium | Grid BFS | Google, Amazon |
| 17 | Number of Distinct Islands | 694 | Medium | Grid BFS | Amazon, Google |
| 18 | Making A Large Island | 827 | Hard | Grid BFS | Google, Amazon |
| 19 | Course Schedule | 207 | Medium | Topo Sort | Amazon, Google, Microsoft, Adobe |
| 20 | Course Schedule II | 210 | Medium | Topo Sort | Amazon, Google, Microsoft |
| 21 | Alien Dictionary | 269 | Hard | Topo Sort | Amazon, Google, Adobe |
| 22 | Minimum Height Trees | 310 | Medium | Topo Sort | Amazon, Google |
| 23 | Sequence Reconstruction | 444 | Medium | Topo Sort | Google, Amazon |
| 24 | Parallel Courses | 1136 | Medium | Topo Sort | Amazon, Microsoft |
| 25 | Find Eventual Safe States | 802 | Medium | Topo Sort / Cycle | Google, Amazon |
| 26 | Network Delay Time | 743 | Medium | Shortest Path (Dijkstra) | Amazon, Google, Goldman Sachs |
| 27 | Cheapest Flights Within K Stops | 787 | Medium | Shortest Path (Bellman-Ford) | Amazon, Google, Adobe |
| 28 | Path With Minimum Effort | 1631 | Medium | Shortest Path (Dijkstra) | Amazon, Google |
| 29 | Swim in Rising Water | 778 | Hard | Shortest Path (Dijkstra) | Google, Amazon |
| 30 | Path with Maximum Probability | 1514 | Medium | Shortest Path (Dijkstra) | Amazon, Microsoft |
| 31 | Find the City With Smallest Number of Neighbors | 1334 | Medium | Shortest Path (Floyd-Warshall) | Amazon, Goldman Sachs |
| 32 | Minimum Cost to Make Valid Path | 1368 | Hard | Shortest Path (0-1 BFS) | Google, Amazon |
| 33 | Shortest Path Visiting All Nodes | 847 | Hard | Shortest Path (BFS + bitmask) | Google, Amazon |
| 34 | Min Cost to Connect All Points | 1584 | Medium | MST | Amazon, Microsoft, Goldman Sachs |
| 35 | Connecting Cities With Minimum Cost | 1135 | Medium | MST | Amazon, Google |
| 36 | Optimize Water Distribution | 1168 | Hard | MST | Google, Amazon |
| 37 | Find Critical and Pseudo-Critical Edges | 1489 | Hard | MST | Google, Amazon |
| 38 | Redundant Connection | 684 | Medium | Union Find | Amazon, Google, Microsoft |
| 39 | Accounts Merge | 721 | Medium | Union Find | Amazon, Google, Adobe |
| 40 | Number of Operations to Make Network Connected | 1319 | Medium | Union Find | Amazon, Microsoft |
| 41 | Satisfiability of Equality Equations | 990 | Medium | Union Find | Google, Amazon |
| 42 | Most Stones Removed with Same Row or Column | 947 | Medium | Union Find | Amazon, Google |
| 43 | Number of Islands II | 305 | Hard | Union Find | Google, Amazon, Goldman Sachs |
| 44 | Regions Cut By Slashes | 959 | Medium | Union Find | Google, Amazon |
| 45 | Smallest String With Swaps | 1202 | Medium | Union Find | Amazon, Google |
| 46 | Graph Valid Tree | 261 | Medium | Union Find | Amazon, Google, Microsoft |
| 47 | Detect Cycle in Undirected Graph | — | Medium | Cycle Detection | Amazon, Microsoft |
| 48 | Detect Cycle in Directed Graph | — | Medium | Cycle Detection | Amazon, Google |
| 49 | Find Eventual Safe States (cross-ref) | 802 | Medium | Cycle Detection | Google, Amazon |
| 50 | Course Schedule (cross-ref) | 207 | Medium | Cycle Detection | Amazon, Google, Microsoft |
| 51 | Redundant Connection II | 685 | Hard | Cycle Detection | Amazon, Google |
| 52 | Critical Connections (Tarjan bridges) | 1192 | Hard | Cycle Detection | Amazon, Google, Goldman Sachs |
| 53 | Find if Path Exists in Graph | 1971 | Easy | Cycle Detection | Amazon, Microsoft |

---

## Graph Representation

How you store the graph drives both correctness and performance. Pick the representation that matches the operations you do most.

### Adjacency list (default for sparse graphs)

```cpp
#include <bits/stdc++.h>
using namespace std;

// Unweighted, nodes 0..n-1
vector<vector<int>> adj(n);
for (auto& e : edges) {
    adj[e[0]].push_back(e[1]);
    adj[e[1]].push_back(e[0]); // omit for directed graphs
}

// Weighted: store {neighbor, weight}
unordered_map<int, vector<pair<int,int>>> wadj;
for (auto& e : edges) {
    wadj[e[0]].push_back({e[1], e[2]});
}
```

- Space `O(V + E)`. Iterating a node's neighbors is `O(degree)`.
- Use for almost everything: traversal, topo sort, Dijkstra, MST, cycle detection.

### Adjacency matrix

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<vector<int>> g(n, vector<int>(n, 0)); // g[i][j] = weight, or 0/INT_MAX for "no edge"
```

- Space `O(V^2)`. Edge existence lookup `O(1)`.
- Use when the graph is **dense**, given as a matrix already (e.g. Number of Provinces), or for **Floyd-Warshall** all-pairs shortest path.

### When to use matrix vs list

| Situation | Representation |
|-----------|----------------|
| Sparse graph (`E ≪ V^2`) | Adjacency list |
| Dense graph / all-pairs distances | Adjacency matrix |
| Input already a matrix (provinces, grids) | Matrix / treat grid cell as node |
| Need fast "is there an edge u→v?" lookups | Adjacency matrix |
| Weighted edges for Dijkstra / Prim | `unordered_map<int, vector<pair<int,int>>>` list |

---

## Algorithm Selection

The single most important graph skill is mapping the problem to the right algorithm. Read the edge weights and the question being asked.

| Need | Algorithm | Why | Complexity |
|------|-----------|-----|------------|
| Shortest path, **unweighted** (or all weights equal) | **BFS** | First time you reach a node is the shortest distance | `O(V + E)` |
| Shortest path, weights are **0 or 1** | **0-1 BFS** (deque) | addFirst for 0-edges, addLast for 1-edges | `O(V + E)` |
| Shortest path, **non-negative** weights | **Dijkstra** (PQ) | Greedy: settle nearest unsettled node | `O(E log V)` |
| Shortest path with **negative** edges / bounded hops | **Bellman-Ford** | Relax all edges V-1 times; handles negatives | `O(V·E)` |
| **All-pairs** shortest paths | **Floyd-Warshall** | DP over intermediate vertices | `O(V^3)` |
| **Minimum Spanning Tree** | **Prim's / Kruskal's** | Cheapest connecting edge set | `O(E log V)` |
| **Connectivity / grouping / dynamic union** | **Union-Find (DSU)** | Near O(1) amortized merge & query | `O(α(n))` per op |
| **Ordering with dependencies** (DAG) | **Topological Sort** | Kahn's indegree BFS or DFS postorder | `O(V + E)` |
| **Cycle detection** (directed) | **3-color DFS / Kahn's** | Gray node revisit = back edge = cycle | `O(V + E)` |
| **Cycle detection** (undirected) | **DSU / DFS parent** | Edge to already-connected node = cycle | `O(V + E)` |
| **Bridges / critical edges** | **Tarjan** | `low[v] > disc[u]` ⇒ bridge | `O(V + E)` |

---

## Design Data Structure Problems

| Problem | Focus | Link |
|---------|-------|------|
| Graph DSU Design | Production-quality `UnionFind` (path compression + union by rank), generic `UnionFind<T>`, weighted DSU, rollback/persistent DSU | [Graph DSU Design](./Design%20Data%20Structure%20Problems/Graph%20DSU%20Design.md) |

---

## Company Coverage

| Company | Representative problems |
|---------|-------------------------|
| **Amazon** | Number of Islands, Course Schedule I/II, Word Ladder, Rotting Oranges, Network Delay Time, Min Cost to Connect All Points, Accounts Merge, Critical Connections |
| **Google** | Clone Graph, Surrounded Regions, Pacific Atlantic, Alien Dictionary, Cheapest Flights, Swim in Rising Water, Making A Large Island, Redundant Connection II |
| **Microsoft** | Number of Provinces, Flood Fill, Course Schedule, Walls and Gates, Path with Maximum Probability, Graph Valid Tree, Find if Path Exists |
| **Goldman Sachs** | Shortest Path in Binary Matrix, Network Delay Time, Find the City (Floyd-Warshall), Min Cost to Connect All Points, Number of Islands II, Critical Connections |
| **Adobe** | Number of Islands, Clone Graph, Course Schedule, Alien Dictionary, Cheapest Flights, Accounts Merge |

---

## How to Use This Repository

1. Start with **Graph Traversal** and **Grid and Matrix BFS** — they build the mental model of nodes, edges, visited sets, and queues.
2. Layer on **Topological Sort** and **Cycle Detection** (closely related — both reason about the order of DFS/dependencies).
3. Master the **Shortest Path** decision table; most "minimum cost / minimum time" problems map to one of its five algorithms.
4. Learn **Union Find** thoroughly — it powers MST (Kruskal), connectivity, and grouping problems, and reappears in the design section.
5. For each problem: read the idea, study the C++ solution, then re-derive the recognition signal so you can spot the pattern under interview pressure.

---

> **Last Updated:** 2026-06-26
