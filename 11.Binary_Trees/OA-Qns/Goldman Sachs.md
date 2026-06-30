> **Topic:** [Binary Trees](../README.md) · **Company 4 of 5**

# Goldman Sachs — Binary Trees OA Questions

Goldman Sachs assessments mix CS fundamentals with quantitative reasoning, and tree questions tend to focus on value-aggregation and optimization over structure — maximizing a path sum, choosing non-adjacent nodes, or summing root-to-leaf values. Expect a HackerRank-style timed round where clear dynamic-programming-on-trees thinking pays off.

## Problem Bank

| Problem | LC # | Frequency | Difficulty | Pattern |
|---------|------|-----------|------------|---------|
| Binary Tree Maximum Path Sum | 124 | ⭐⭐⭐⭐ | Hard | [Tree DP](../Patterns/Tree%20DP.md) |
| House Robber III | 337 | ⭐⭐⭐ | Medium | [Tree DP](../Patterns/Tree%20DP.md) |
| Sum Root to Leaf Numbers | 129 | ⭐⭐⭐ | Medium | [Path Problems](../Patterns/Path%20Problems.md) |
| Path Sum | 112 | ⭐⭐⭐ | Easy | [Path Problems](../Patterns/Path%20Problems.md) |
| Diameter of Binary Tree | 543 | ⭐⭐⭐ | Easy | [Tree DP](../Patterns/Tree%20DP.md) |
| Maximum Depth of Binary Tree | 104 | ⭐⭐ | Easy | [Tree Traversals](../Patterns/Tree%20Traversals.md) |
| Lowest Common Ancestor of a Binary Tree | 236 | ⭐⭐ | Medium | [Path Problems](../Patterns/Path%20Problems.md) |
| Binary Tree Level Order Traversal | 102 | ⭐⭐ | Medium | [Level Order](../Patterns/Level%20Order.md) |
| Count Complete Tree Nodes | 222 | ⭐⭐ | Easy | [Tree Traversals](../Patterns/Tree%20Traversals.md) |

## OA Tips

- The GS coding round is typically time-boxed per question with no going back — read all parts before starting and manage the clock strictly.
- Tree DP problems reward a post-order helper that returns a small tuple (e.g., {rob, skip} or {gain, best}); design that return shape first.
- The assessment often pairs coding with aptitude/numerical and logical-reasoning sections — practice mental math separately.
- Favor clean, readable solutions with correct integer overflow handling (use 64-bit sums where path values can be large).

> **Last Updated:** 2026-06-26
