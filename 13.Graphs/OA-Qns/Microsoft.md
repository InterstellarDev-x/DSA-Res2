> **Topic:** [Graphs](../README.md) · **Section:** OA-Qns · **Company:** Microsoft

# Microsoft — Graphs OA Questions

Microsoft graph rounds favor **clean, well-structured implementations** of standard problems: Number of Islands, Course Schedule, Clone Graph, and a single weighted shortest-path (Network Delay Time). They value readable code, correct data-structure choice, and clear complexity analysis over exotic algorithms.

**Related companies:** [Amazon](./Amazon.md) · [Google](./Google.md) · [Goldman Sachs](./Goldman%20Sachs.md) · [Adobe](./Adobe.md)

---

## Question Frequency Table

| Problem | LC | Frequency | Difficulty | Pattern | Source |
|---------|----|-----------|------------|---------|--------|
| Number of Islands | 200 | ⭐⭐⭐⭐⭐ | Medium | [Graph Traversal](../Patterns/Graph%20Traversal%20(BFS%20DFS).md) | OA + onsite |
| Course Schedule | 207 | ⭐⭐⭐⭐ | Medium | [Topological Sort](../Patterns/Topological%20Sort.md) | OA + onsite |
| Clone Graph | 133 | ⭐⭐⭐⭐ | Medium | [Graph Traversal](../Patterns/Graph%20Traversal%20(BFS%20DFS).md) | Onsite |
| Network Delay Time | 743 | ⭐⭐⭐ | Medium | [Shortest Path](../Patterns/Shortest%20Path.md) | Onsite |

---

## Number of Islands ⭐⭐⭐⭐⭐

**Prompt.** Count 4-connected components of `'1'` in a binary grid.

**Approach.** Scan cells; flood-fill (BFS/DFS) each unvisited land cell, marking sunk. In-place marking avoids extra space.

**What interviewers look for** — clean component-counting loop; marking at enqueue; choosing iterative BFS for very large grids (recursion stack risk).

**Follow-ups** — diagonal connectivity (8-dir); count island **perimeter** (LC 463); islands added incrementally → [Union Find (DSU)](../Patterns/Union%20Find%20(DSU).md).

---

## Course Schedule ⭐⭐⭐⭐

**Prompt.** Can all `numCourses` be finished given prerequisite pairs? (Feasibility only.)

**Approach.** Detect a cycle in the directed prerequisite graph: **Kahn's** (process count == numCourses) or **3-color DFS**.

**What interviewers look for** — correct edge direction; cycle = impossible; ability to extend to Course Schedule II (return the order).

**Follow-ups** — Course Schedule II (ordering); minimum semesters with parallelism; [Cycle Detection](../Patterns/Cycle%20Detection.md) on the same graph.

---

## Clone Graph ⭐⭐⭐⭐

**Prompt.** Deep-copy a connected undirected graph.

**Approach.** Traverse with a `Map<Node,Node>` (visited → clone). On first sight create the clone; always wire `clone.neighbors`. Works with BFS or DFS.

**What interviewers look for** — using the map both to dedupe and as the clone store; cycle safety; null input handling.

**Follow-ups** — DFS vs BFS recursion-depth trade-off; copy graph with extra random pointers.

---

## Network Delay Time ⭐⭐⭐

**Prompt.** Directed weighted graph; signal from node `k`. Return time for all nodes to receive it, or `-1`.

**Approach.** **Dijkstra** from `k`; answer is the max finalized distance (or `-1` if any unreachable).

**What interviewers look for**
- `PriorityQueue<int[]>` ordered by distance with **`Integer.compare`** (never subtraction).
- **Stale-entry skipping**: pop, skip if `d > dist[node]`.
- Recognizing non-negative weights → Dijkstra (not Bellman-Ford).

**Follow-ups** — *Cheapest Flights with K Stops* (constraint → Bellman-Ford); *Path With Minimum Effort* (minimize max edge → modified Dijkstra).

---

## What Microsoft Emphasizes

| Dimension | Expectation |
|-----------|-------------|
| Code clarity | Named helpers, `int[][] dirs`, no magic numbers |
| Data structures | Right choice (PQ for Dijkstra, deque for 0-1 BFS) |
| Complexity | State time/space precisely (V+E, (V+E)log V) |
| Testing | Walk through a small example, then edge cases |

> **Last Updated:** 2026-06-26
