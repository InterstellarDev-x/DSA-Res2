> **Topic:** [Graphs](../README.md) · **Section:** OA-Qns · **Company:** Google

# Google — Graphs OA Questions

Google leans toward problems that require **building a graph from non-obvious structure** (Alien Dictionary, Word Ladder) and **advanced algorithms** (Tarjan bridges, MST). Interviewers care about precise reasoning over edge cases, optimal complexity, and clean abstractions — a working answer is necessary but not sufficient.

**Related companies:** [Amazon](./Amazon.md) · [Microsoft](./Microsoft.md) · [Goldman Sachs](./Goldman%20Sachs.md) · [Adobe](./Adobe.md)

---

## Question Frequency Table

| Problem | LC | Frequency | Difficulty | Pattern | Source |
|---------|----|-----------|------------|---------|--------|
| Word Ladder | 127 | ⭐⭐⭐⭐⭐ | Hard | [Graph Traversal](../Patterns/Graph%20Traversal%20(BFS%20DFS).md) | Onsite, very common |
| Alien Dictionary | 269 | ⭐⭐⭐⭐ | Hard | [Topological Sort](../Patterns/Topological%20Sort.md) | Onsite (L4/L5 staple) |
| Pacific Atlantic Water Flow | 417 | ⭐⭐⭐⭐ | Medium | [Graph Traversal](../Patterns/Graph%20Traversal%20(BFS%20DFS).md) | Onsite |
| Critical Connections | 1192 | ⭐⭐⭐ | Hard | [Cycle Detection](../Patterns/Cycle%20Detection.md) | Onsite (L5+) |
| Min Cost to Connect Points | 1584 | ⭐⭐⭐ | Medium | [Minimum Spanning Tree](../Patterns/Minimum%20Spanning%20Tree.md) | Onsite |

---

## Word Ladder ⭐⭐⭐⭐⭐

**Prompt.** Shortest transformation sequence length from `beginWord` to `endWord`, one-letter changes, each step in the dictionary.

**Approach.** Unweighted shortest path → BFS. Use **wildcard buckets** (`*ot`, `h*t`, `ho*`) to precompute adjacency in O(N·L) instead of comparing all pairs. Consider **bidirectional BFS** to cut the search frontier.

**What interviewers look for** — recognizing BFS gives shortest path; the preprocessing optimization; clean visited handling; bidirectional discussion at L5.

**Follow-ups** — *Word Ladder II* (all shortest paths); analyze when bidirectional helps.

---

## Alien Dictionary ⭐⭐⭐⭐

**Prompt.** Given words sorted in an alien language's order, return a valid character order (or `""` if invalid).

**Approach.** Build a graph from **adjacent word pairs** — first differing char gives edge `a → b`. Then topo-sort (Kahn's). Detect a cycle → `""`.

**What interviewers look for**
- The **prefix-conflict edge case**: if `word1` is longer than `word2` but `word2` is a prefix of `word1` (e.g. `["abc","ab"]`) → invalid → return `""`.
- Including all distinct chars as nodes even if they appear in no edge.
- Cycle detection during topo-sort.

**Follow-ups** — multiple valid orders (return any / all); incremental words.

---

## Pacific Atlantic Water Flow ⭐⭐⭐⭐

**Prompt.** Matrix of heights; water flows to lower/equal neighbors. Return cells from which water reaches **both** oceans.

**Approach.** **Reverse the flow**: DFS/BFS *inward* from each ocean's border cells (climbing to ≥ heights). Intersection of the two reachable sets is the answer.

**What interviewers look for** — the reversal insight (don't search from every cell); two visited matrices; `>=` comparison direction.

**Follow-ups** — count cells reaching exactly one ocean; 3D heights.

---

## Critical Connections ⭐⭐⭐

**Prompt.** Find all bridges in an undirected graph (edges whose removal disconnects it).

**Approach.** **Tarjan's bridge-finding**: DFS with `disc[]` (discovery time) and `low[]` (lowest reachable disc). Edge `(u,v)` is a bridge iff `low[v] > disc[u]`.

**What interviewers look for** — correct `low` update (`min` over children and back-edges, skipping the parent edge); single DFS pass O(V+E).

**Follow-ups** — articulation points; Strongly Connected Components (Tarjan SCC).

---

## Min Cost to Connect Points ⭐⭐⭐

**Prompt.** Connect all points (Manhattan distance edges) at minimum total cost.

**Approach.** MST on a **complete graph** → **Prim's** (dense, no edge list needed) with a PQ, or Kruskal's after generating all O(n²) edges.

**What interviewers look for** — Prim-vs-Kruskal reasoning: dense complete graph favors Prim's O(V²) / O(E log V); Kruskal needs all pairs sorted.

**Follow-ups** — *Connecting Cities*, *Optimize Water Distribution* (virtual node trick), *Find Critical & Pseudo-Critical Edges*.

---

## L4 / L5 Level Context

| Signal | L4 (SWE) | L5 (Senior) |
|--------|----------|-------------|
| Algorithm choice | Reach correct standard template | Justify optimal complexity + alternatives |
| Edge cases | Handle the obvious ones | Anticipate prefix-conflict, disconnected, self-loop unprompted |
| Optimization | Get it working | Bidirectional BFS, wildcard buckets, reverse-flow insight |
| Communication | Explain after coding | Narrate trade-offs while designing |

> **Last Updated:** 2026-06-26
