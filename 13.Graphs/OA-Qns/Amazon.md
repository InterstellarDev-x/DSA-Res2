> **Topic:** [Graphs](../README.md) · **Section:** OA-Qns · **Company:** Amazon

# Amazon — Graphs OA Questions

Amazon's Online Assessment (OA) and early-round graph questions are dominated by **grid traversal** (Number of Islands, Rotting Oranges) and **dependency ordering** (Course Schedule II). These map cleanly onto Amazon's "scale" and "operational dependency" framing. Expect a clean, working solution plus at least one follow-up on scaling or edge cases.

**Related companies:** [Google](./Google.md) · [Microsoft](./Microsoft.md) · [Goldman Sachs](./Goldman%20Sachs.md) · [Adobe](./Adobe.md)

---

## Question Frequency Table

| Problem | LC | Frequency | Difficulty | Pattern | Source |
|---------|----|-----------|------------|---------|--------|
| Number of Islands | 200 | ⭐⭐⭐⭐⭐ | Medium | [Graph Traversal](../Patterns/Graph%20Traversal%20(BFS%20DFS).md) | OA + onsite, all teams |
| Rotting Oranges | 994 | ⭐⭐⭐⭐⭐ | Medium | [Grid and Matrix BFS](../Patterns/Grid%20and%20Matrix%20BFS.md) | OA (very common) |
| Course Schedule II | 210 | ⭐⭐⭐⭐ | Medium | [Topological Sort](../Patterns/Topological%20Sort.md) | OA + onsite |
| Word Ladder | 127 | ⭐⭐⭐ | Hard | [Graph Traversal](../Patterns/Graph%20Traversal%20(BFS%20DFS).md) | Onsite (L5+) |
| Clone Graph | 133 | ⭐⭐⭐ | Medium | [Graph Traversal](../Patterns/Graph%20Traversal%20(BFS%20DFS).md) | Onsite |

---

## Number of Islands ⭐⭐⭐⭐⭐

**Prompt.** Given an `m x n` grid of `'1'` (land) and `'0'` (water), count the islands. An island is land connected 4-directionally.

**Approach.** Iterate cells; when you hit unvisited land, run BFS/DFS to sink the whole island, counting one. Mark in-place (`grid[r][c] = '0'`) to avoid a visited array.

**What interviewers look for**
- Marking visited **before / at enqueue**, not after dequeue (prevents duplicate enqueues).
- Bounds check ordering: index validity before reading `grid[r][c]`.
- Discussing BFS vs DFS (recursion depth / stack overflow on large grids → prefer BFS or iterative DFS).

**Follow-ups**
- *Number of Islands II* (LC 305) — land added one at a time → switch to [Union Find (DSU)](../Patterns/Union%20Find%20(DSU).md).
- *Making A Large Island* (LC 827) — flip one `0`; label islands and sum neighbor sizes.
- *Count distinct island shapes* (LC 694) → [Grid and Matrix BFS](../Patterns/Grid%20and%20Matrix%20BFS.md).
- "Grid too large for memory" → stream rows / DSU with rank.

---

## Rotting Oranges ⭐⭐⭐⭐⭐

**Prompt.** Grid of `0` (empty), `1` (fresh), `2` (rotten). Each minute, rotten infects 4-adjacent fresh. Return minutes until none fresh, or `-1`.

**Approach.** **Multi-source BFS**: seed the queue with *all* rotten cells first, count fresh, then expand level by level incrementing the timer. Answer is the last level; if fresh remain, return `-1`.

**What interviewers look for**
- Seeding all sources before expanding (single-source BFS gives wrong timing).
- Tracking `fresh` count to detect unreachable oranges.
- Edge case: zero fresh at start → return `0`.

**Follow-ups**
- *01 Matrix* / *Walls and Gates* — same multi-source template.
- Weighted spread time per cell → [Shortest Path](../Patterns/Shortest%20Path.md) (Dijkstra / 0-1 BFS).

---

## Course Schedule II ⭐⭐⭐⭐

**Prompt.** `numCourses` and prerequisite pairs. Return a valid order, or empty if impossible.

**Approach.** **Kahn's algorithm**: build adjacency + indegree, queue zero-indegree nodes, pop and decrement neighbors. If processed count `< numCourses`, a cycle exists → return `[]`.

**What interviewers look for**
- Cycle detection via "did we output all nodes?".
- Kahn's (BFS) vs DFS postorder reversal — know both.
- Direction of edges: `prereq → course`.

**Follow-ups**
- *Course Schedule I* (LC 207) — only feasibility.
- *Alien Dictionary* (LC 269) — derive the graph then topo-sort.
- *Parallel Courses* — minimum semesters = longest dependency chain (level count).

---

## Word Ladder ⭐⭐⭐

**Prompt.** Transform `beginWord` to `endWord` one letter at a time, each intermediate in the dictionary. Return shortest transformation length.

**Approach.** BFS over implicit graph. Optimize neighbor generation with **wildcard buckets** (`h*t`) to avoid O(26·L) per word against the whole list. Bidirectional BFS for large dictionaries.

**What interviewers look for** — recognizing it as unweighted shortest path → BFS; the wildcard preprocessing; visited set.

**Follow-ups** — *Word Ladder II* (return all paths → BFS layering + parent backtrack); bidirectional BFS optimization.

---

## Clone Graph ⭐⭐⭐

**Prompt.** Deep-copy a connected undirected graph given a node reference.

**Approach.** BFS/DFS with a `Map<Node,Node>` from original → clone; create clone on first visit, then wire neighbors.

**What interviewers look for** — the visited map doubling as the clone registry; handling cycles (the map prevents infinite recursion); null/empty input.

**Follow-ups** — clone a graph with random pointers; serialize/deserialize.

---

## Amazon Leadership Principle Alignment

| LP | How the graph problem signals it |
|----|----------------------------------|
| **Customer Obsession** | Clarify constraints/edge cases before coding (empty grid, disconnected graph). |
| **Dive Deep** | Explain BFS-vs-DFS trade-offs and why visited is marked at enqueue. |
| **Bias for Action** | Get a correct brute-force / standard template down fast, then optimize. |
| **Insist on Highest Standards** | Handle `-1`/empty-order failure cases; test on degenerate inputs. |
| **Invent and Simplify** | Wildcard buckets, in-place marking, multi-source seeding. |

> **Last Updated:** 2026-06-26
