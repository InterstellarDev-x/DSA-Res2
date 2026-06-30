> **Topic:** [Graphs](../README.md) · **Section:** Interview Tips · **Tips 3 of 4**

# Graphs — Algorithm Selection

The fastest way to lose a graph interview is solving the right problem with the wrong algorithm. This is the master decision guide. See also [Coding Tips](./Coding%20Tips.md), [Common Mistakes](./Common%20Mistakes.md), [Complexity Analysis](./Complexity%20Analysis.md).

---

## Decision Tree

```
Is it a shortest / minimum path problem?
├─ Unweighted (all edges cost 1)? .................... BFS                  O(V+E)
├─ Weights are only 0 or 1? ......................... 0-1 BFS (deque)       O(V+E)
├─ Non-negative weights, single source? ............. Dijkstra             O((V+E) log V)
├─ Negative edges OR "at most K stops/edges"? ....... Bellman-Ford         O(V·E)
└─ Need ALL pairs shortest paths? ................... Floyd-Warshall       O(V³)

Is it about connecting everything cheaply (MST)?
├─ Dense / complete graph? .......................... Prim's (array)       O(V²)
├─ Sparse graph given as edge list? ................. Kruskal's (+DSU)     O(E log E)
└─ Prim with a heap (general) ....................... Prim's (PQ)          O(E log V)

Is it about grouping / dynamic connectivity / "are X and Y connected"?
└─ ........................................... Union-Find (DSU)            ~O(α(n)) per op

Is it about ordering with dependencies (a DAG)?
└─ ........................................... Topological Sort (Kahn/DFS) O(V+E)

Is it about finding cycles / critical structure?
├─ Detect any cycle (directed)? ..................... 3-color DFS / Kahn   O(V+E)
├─ Detect any cycle (undirected)? ................... DFS parent / DSU     O(V+E)
└─ Bridges / articulation points? ................... Tarjan               O(V+E)
```

---

## Selection Table

| Problem shape | Algorithm | Time | Space |
|---------------|-----------|------|-------|
| Shortest path, unweighted | BFS | O(V+E) | O(V) |
| Shortest path, 0/1 weights | 0-1 BFS (deque) | O(V+E) | O(V) |
| Shortest path, non-negative weights | Dijkstra (PQ) | O((V+E) log V) | O(V) |
| Shortest path, negative edges / ≤K hops | Bellman-Ford | O(V·E) | O(V) |
| All-pairs shortest paths | Floyd-Warshall | O(V³) | O(V²) |
| MST, dense/complete | Prim (array) | O(V²) | O(V) |
| MST, general | Prim (heap) | O(E log V) | O(V+E) |
| MST, sparse edge list | Kruskal + DSU | O(E log E) | O(V) |
| Dynamic connectivity / grouping | Union-Find | ~O(α(n))/op | O(V) |
| Dependency ordering (DAG) | Topological sort | O(V+E) | O(V) |
| Directed cycle | 3-color DFS / Kahn | O(V+E) | O(V) |
| Undirected cycle | DFS parent / DSU | O(V+E) | O(V) |
| Bridges / articulation pts | Tarjan | O(V+E) | O(V) |
| Visit ALL nodes shortest | BFS + bitmask | O(2ⁿ·V²) | O(2ⁿ·V) |

---

## Signal Phrases → Algorithm

| Phrase in the prompt | Likely algorithm |
|----------------------|------------------|
| "minimum number of steps / moves", all moves equal | **BFS** |
| "fewest changes" where each costs 0 or 1 | **0-1 BFS** |
| "minimum total weight / cost", positive weights | **Dijkstra** |
| "minimize the maximum edge along a path" | **Dijkstra (minimax relax)** — Path With Min Effort, Swim in Rising Water |
| "at most K stops / K edges", or negative costs | **Bellman-Ford** |
| "shortest distance between every pair" | **Floyd-Warshall** |
| "connect all ... with minimum cost / wire / road" | **MST (Prim/Kruskal)** |
| "are these in the same group", "merge accounts", "redundant connection" | **Union-Find** |
| "order to finish", "prerequisites", "build sequence" | **Topological sort** |
| "can it be finished", "is there a cycle" (directed deps) | **Cycle detection (Kahn/3-color)** |
| "critical / single point of failure connection" | **Tarjan bridges** |
| "spread from multiple origins simultaneously" | **Multi-source BFS** |
| "smallest path through a grid", 8 directions | **BFS (grid)** |

---

## Quick disambiguations interviewers test

- **BFS vs Dijkstra:** equal weights? BFS is simpler and faster. Don't reach for a heap when one isn't needed.
- **Dijkstra vs Bellman-Ford:** any negative edge, or a hop/stop constraint, breaks Dijkstra's greedy correctness → Bellman-Ford.
- **Prim vs Kruskal:** dense/complete → Prim (no edge list); sparse edge list → Kruskal (sort + DSU).
- **DSU vs traversal for components:** static graph → either; **incremental** edges/cells → DSU is the clear win.
- **Kahn vs DFS topo:** need the order *and* cycle detection cleanly → Kahn's "processed == V" check.

> **Last Updated:** 2026-06-26
