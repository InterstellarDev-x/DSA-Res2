> **Topic:** [Graphs](../README.md) · **Section:** OA-Qns · **Company:** Adobe

# Adobe — Graphs OA Questions

Adobe's graph questions stay close to the **fundamentals**: grid flood-fill (Flood Fill, Number of Islands) and basic dependency feasibility (Course Schedule). Clean, correct implementations of the core templates are what matter — exotic algorithms are rare.

**Related companies:** [Amazon](./Amazon.md) · [Google](./Google.md) · [Microsoft](./Microsoft.md) · [Goldman Sachs](./Goldman%20Sachs.md)

---

## Question Frequency Table

| Problem | LC | Frequency | Difficulty | Pattern | Source |
|---------|----|-----------|------------|---------|--------|
| Flood Fill | 733 | ⭐⭐⭐ | Easy | [Graph Traversal](../Patterns/Graph%20Traversal%20(BFS%20DFS).md) | OA |
| Number of Islands | 200 | ⭐⭐⭐ | Medium | [Graph Traversal](../Patterns/Graph%20Traversal%20(BFS%20DFS).md) | OA + onsite |
| Course Schedule | 207 | ⭐⭐⭐ | Medium | [Topological Sort](../Patterns/Topological%20Sort.md) | Onsite |

---

## Flood Fill ⭐⭐⭐

**Prompt.** Replace the color of a starting pixel and all 4-connected same-color pixels with a new color.

**Approach.** DFS/BFS from the start, recoloring same-color neighbors.

**What interviewers look for**
- **The classic trap**: if `newColor == startColor`, return immediately — otherwise infinite recursion (a cell stays "matchable").
- Recording the original color before overwriting.
- Bounds checks before reading a pixel.

**Follow-ups** — Number of Islands (count components instead of recolor); 8-directional fill; very large image → iterative BFS.

---

## Number of Islands ⭐⭐⭐

**Prompt.** Count 4-connected components of `'1'` in a binary grid.

**Approach.** Flood-fill each unvisited land cell (BFS/DFS), marking sunk; increment the count per fill.

**What interviewers look for** — in-place marking; marking at enqueue (not dequeue) to avoid duplicates; iterative traversal for big grids.

**Follow-ups** — island perimeter / max area; diagonal connectivity; incremental land → [Union Find (DSU)](../Patterns/Union%20Find%20(DSU).md).

---

## Course Schedule ⭐⭐⭐

**Prompt.** Determine whether all courses can be finished given prerequisite pairs.

**Approach.** Cycle detection in the directed graph via **Kahn's** (processed == numCourses) or 3-color DFS.

**What interviewers look for** — correct edge direction (`prereq → course`); cycle ⇒ impossible; clean indegree bookkeeping.

**Follow-ups** — Course Schedule II (return order); [Topological Sort](../Patterns/Topological%20Sort.md) variants; [Cycle Detection](../Patterns/Cycle%20Detection.md).

> **Last Updated:** 2026-06-26
