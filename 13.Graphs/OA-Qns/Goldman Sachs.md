> **Topic:** [Graphs](../README.md) · **Section:** OA-Qns · **Company:** Goldman Sachs

# Goldman Sachs — Graphs OA Questions

Goldman Sachs graph questions cluster around **connectivity and grouping** — counting connected components and merging entities. These map directly onto financial/account-domain modeling, so Union-Find and component-counting traversals dominate.

**Related companies:** [Amazon](./Amazon.md) · [Google](./Google.md) · [Microsoft](./Microsoft.md) · [Adobe](./Adobe.md)

---

## Question Frequency Table

| Problem | LC | Frequency | Difficulty | Pattern | Source |
|---------|----|-----------|------------|---------|--------|
| Number of Provinces | 547 | ⭐⭐⭐⭐ | Medium | [Graph Traversal](../Patterns/Graph%20Traversal%20(BFS%20DFS).md) | OA + onsite |
| Accounts Merge | 721 | ⭐⭐⭐ | Medium | [Union Find (DSU)](../Patterns/Union%20Find%20(DSU).md) | Onsite |
| Number of Connected Components (Network Connected) | 323 | ⭐⭐⭐ | Medium | [Union Find (DSU)](../Patterns/Union%20Find%20(DSU).md) | OA |

---

## Number of Provinces ⭐⭐⭐⭐

**Prompt.** `n x n` `isConnected` adjacency matrix; count groups of directly/indirectly connected cities.

**Approach.** Count connected components: DFS/BFS over the matrix, or **Union-Find** merging `i,j` when `isConnected[i][j]==1` and counting roots.

**What interviewers look for** — recognizing it as component counting; iterating the upper triangle to avoid redundant work; DSU vs traversal trade-off.

**Follow-ups** — *Number of Connected Components* (edge-list form); incremental connections (favor DSU); largest province size.

---

## Accounts Merge ⭐⭐⭐

**Prompt.** Merge accounts sharing any email (same owner). Return merged, sorted email lists.

**Approach.** **Union-Find** over emails: union all emails within an account, map each email → owner, then group emails by root and sort.

**What interviewers look for**
- Choosing emails (not account indices) as DSU nodes.
- Maintaining `email → name` for output.
- Final grouping + per-group sort.

**Follow-ups** — *Smallest String With Swaps* (DSU over index pairs); *Redundant Connection* (first cycle edge).

---

## Number of Connected Components (Network Connected) ⭐⭐⭐

**Prompt.** `n` nodes, undirected `edges`; return the count of connected components. (a.k.a. *Number of Operations to Make Network Connected*, LC 1319 — extra edges needed = components − 1 when `edges.length >= n-1`.)

**Approach.** **Union-Find** starting at `n` components, decrement on each successful union. For the "operations" variant, ensure enough cables (`edges.length >= n-1`) then answer `components - 1`.

**What interviewers look for** — DSU with **path compression + union by rank**; the insight that connecting `c` components needs `c-1` edges; checking spare-cable feasibility.

**Follow-ups** — *Graph Valid Tree* (exactly one component AND no cycle); dynamic edge additions.

> **Last Updated:** 2026-06-26
