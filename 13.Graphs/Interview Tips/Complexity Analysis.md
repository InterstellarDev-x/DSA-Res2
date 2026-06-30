> **Topic:** [Graphs](../README.md) · **Section:** Interview Tips · **Tips 4 of 4**

# Graphs — Complexity Analysis

Time and space for every algorithm and all 53 problems. `V` = vertices, `E` = edges; for grids `m·n` cells with `O(1)` neighbors each, so `V+E = O(m·n)`. See also [Coding Tips](./Coding%20Tips.md), [Common Mistakes](./Common%20Mistakes.md), [Algorithm Selection](./Algorithm%20Selection.md).

---

## Core algorithm complexities

| Algorithm | Time | Space | Why |
|-----------|------|-------|-----|
| BFS / DFS | O(V+E) | O(V) | Each vertex + edge touched once |
| Multi-source BFS | O(V+E) | O(V) | All sources seeded; same bound |
| 0-1 BFS | O(V+E) | O(V) | Deque, each vertex finalized once |
| Dijkstra (binary heap) | O((V+E) log V) | O(V) | Each edge may push once; log V per pop |
| Bellman-Ford | O(V·E) | O(V) | V−1 (or K+1) relaxation rounds over all edges |
| Floyd-Warshall | O(V³) | O(V²) | Triple loop over all pairs through every intermediate |
| Union-Find (PC + rank) | ~O(α(n)) per op | O(V) | Inverse Ackermann ≈ constant |
| Topological sort (Kahn) | O(V+E) | O(V) | Each node dequeued once, each edge relaxed once |
| Prim (binary heap) | O(E log V) | O(V+E) | Each edge offered; heap ops log V |
| Prim (array, dense) | O(V²) | O(V) | V iterations × V scan |
| Kruskal | O(E log E) | O(V) | Sort dominates; DSU near-constant |
| Tarjan (bridges / SCC) | O(V+E) | O(V) | Single DFS with disc/low |
| BFS + bitmask (visit all) | O(2ⁿ·V²) | O(2ⁿ·V) | State = (node, subset mask) |

---

## All 53 Problems

### Graph Traversal (BFS / DFS) — [pattern](../Patterns/Graph%20Traversal%20(BFS%20DFS).md)

| # | Problem | LC | Time | Space |
|---|---------|----|------|-------|
| 1 | Number of Provinces | 547 | O(V²) | O(V) |
| 2 | Number of Islands | 200 | O(m·n) | O(m·n) |
| 3 | Flood Fill | 733 | O(m·n) | O(m·n) |
| 4 | Clone Graph | 133 | O(V+E) | O(V) |
| 5 | Surrounded Regions | 130 | O(m·n) | O(m·n) |
| 6 | Pacific Atlantic Water Flow | 417 | O(m·n) | O(m·n) |
| 7 | Is Graph Bipartite | 785 | O(V+E) | O(V) |
| 8 | Word Ladder | 127 | O(N·L²) | O(N·L) |
| 9 | Word Ladder II | 126 | O(N·L² + paths) | O(N·L) |
| 10 | Open the Lock | 752 | O(10⁴·8) | O(10⁴) |

### Grid and Matrix BFS — [pattern](../Patterns/Grid%20and%20Matrix%20BFS.md)

| # | Problem | LC | Time | Space |
|---|---------|----|------|-------|
| 11 | Rotting Oranges | 994 | O(m·n) | O(m·n) |
| 12 | 01 Matrix | 542 | O(m·n) | O(m·n) |
| 13 | Walls and Gates | 286 | O(m·n) | O(m·n) |
| 14 | Shortest Path in Binary Matrix | 1091 | O(m·n) | O(m·n) |
| 15 | As Far from Land as Possible | 1162 | O(m·n) | O(m·n) |
| 16 | Shortest Bridge | 934 | O(m·n) | O(m·n) |
| 17 | Number of Distinct Islands | 694 | O(m·n) | O(m·n) |
| 18 | Making A Large Island | 827 | O(m·n) | O(m·n) |

### Topological Sort — [pattern](../Patterns/Topological%20Sort.md)

| # | Problem | LC | Time | Space |
|---|---------|----|------|-------|
| 19 | Course Schedule | 207 | O(V+E) | O(V+E) |
| 20 | Course Schedule II | 210 | O(V+E) | O(V+E) |
| 21 | Alien Dictionary | 269 | O(C) total chars | O(1) alphabet bounded |
| 22 | Minimum Height Trees | 310 | O(V+E) | O(V+E) |
| 23 | Sequence Reconstruction | 444 | O(V+E) | O(V+E) |
| 24 | Parallel Courses | 1136 | O(V+E) | O(V+E) |
| 25 | Find Eventual Safe States | 802 | O(V+E) | O(V+E) |

### Shortest Path — [pattern](../Patterns/Shortest%20Path.md)

| # | Problem | LC | Time | Space |
|---|---------|----|------|-------|
| 26 | Network Delay Time | 743 | O((V+E) log V) | O(V+E) |
| 27 | Cheapest Flights Within K Stops | 787 | O(K·E) | O(V) |
| 28 | Path With Minimum Effort | 1631 | O(m·n log(m·n)) | O(m·n) |
| 29 | Swim in Rising Water | 778 | O(n² log n) | O(n²) |
| 30 | Path With Maximum Probability | 1514 | O((V+E) log V) | O(V+E) |
| 31 | Find the City (smallest neighbors) | 1334 | O(V³) | O(V²) |
| 32 | Min Cost to Make Valid Path | 1368 | O(m·n) | O(m·n) |
| 33 | Shortest Path Visiting All Nodes | 847 | O(2ⁿ·n²) | O(2ⁿ·n) |

### Minimum Spanning Tree — [pattern](../Patterns/Minimum%20Spanning%20Tree.md)

| # | Problem | LC | Time | Space |
|---|---------|----|------|-------|
| 34 | Min Cost to Connect Points | 1584 | O(V²) Prim | O(V) |
| 35 | Connecting Cities With Min Cost | 1135 | O(E log E) Kruskal | O(V) |
| 36 | Optimize Water Distribution | 1168 | O(E log E) | O(V) |
| 37 | Find Critical & Pseudo-Critical Edges | 1489 | O(E²·α) | O(V) |

### Union Find (DSU) — [pattern](../Patterns/Union%20Find%20(DSU).md)

| # | Problem | LC | Time | Space |
|---|---------|----|------|-------|
| 38 | Redundant Connection | 684 | O(N·α) | O(N) |
| 39 | Accounts Merge | 721 | O(N·K·α + sort) | O(N·K) |
| 40 | Number of Connected Components | 323 | O(E·α) | O(V) |
| 41 | Satisfiability of Equality Equations | 990 | O(N·α) | O(1) (26) |
| 42 | Most Stones Removed | 947 | O(N·α) | O(N) |
| 43 | Number of Islands II | 305 | O(K·α) | O(m·n) |
| 44 | Regions Cut By Slashes | 959 | O(n²·α) | O(n²) |
| 45 | Smallest String With Swaps | 1202 | O(N log N·α) | O(N) |
| 46 | Graph Valid Tree | 261 | O(E·α) | O(V) |

### Cycle Detection — [pattern](../Patterns/Cycle%20Detection.md)

| # | Problem | LC | Time | Space |
|---|---------|----|------|-------|
| 47 | Detect Cycle in Undirected Graph | — | O(V+E) | O(V) |
| 48 | Detect Cycle in Directed Graph | — | O(V+E) | O(V) |
| 49 | Redundant Connection II | 685 | O(N·α) | O(N) |
| 50 | Critical Connections (Tarjan) | 1192 | O(V+E) | O(V+E) |
| 51 | Find if Path Exists in Graph | 1971 | O(V+E) | O(V) |

### Additional traversal/grid

| # | Problem | LC | Time | Space |
|---|---------|----|------|-------|
| 52 | Bipartite check variants | 886/785 | O(V+E) | O(V) |
| 53 | Open the Lock / state-space BFS | 752 | O(states·branch) | O(states) |

---

## Notes on the tricky bounds

- **Word Ladder O(N·L²):** for each of N words, generating L wildcard patterns costs O(L) each (substring) → O(N·L²) to build buckets; BFS visits each once.
- **Cheapest Flights O(K·E):** Bellman-Ford bounded to K+1 rounds, not the full V−1.
- **Floyd-Warshall O(V³):** only viable when V is small (≤ ~400); used for *Find the City*.
- **Bitmask BFS O(2ⁿ·n²):** state space 2ⁿ subsets × n nodes; n is tiny (≤ 12) by constraint.
- **DSU α(n):** inverse Ackermann — effectively ≤ 4 for any input that fits in memory; treat as constant.
- **Grid problems O(m·n):** each cell enqueued once, O(1) (4 or 8) neighbors → linear in cells.

> **Last Updated:** 2026-06-26
