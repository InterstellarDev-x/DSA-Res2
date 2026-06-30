> **Topic:** [Binary Trees](../README.md) · **Company 1 of 5**

# Amazon — Binary Trees OA Questions

Amazon's online assessments lean heavily on tree traversal fundamentals and "find the optimal path/value" style problems. Expect questions framed around package routing, organizational hierarchies, and warehouse zone layouts that reduce to standard binary-tree algorithms. Recursion depth, clean base cases, and O(n) single-pass solutions are what the auto-grader rewards.

## Problem Bank

| Problem | LC # | Frequency | Difficulty | Pattern |
|---------|------|-----------|------------|---------|
| Lowest Common Ancestor of a Binary Tree | 236 | ⭐⭐⭐⭐⭐ | Medium | [Path Problems](../Patterns/Path%20Problems.md) |
| Binary Tree Maximum Path Sum | 124 | ⭐⭐⭐⭐⭐ | Hard | [Tree DP](../Patterns/Tree%20DP.md) |
| Binary Tree Level Order Traversal | 102 | ⭐⭐⭐⭐ | Medium | [Level Order](../Patterns/Level%20Order.md) |
| Serialize and Deserialize Binary Tree | 297 | ⭐⭐⭐⭐ | Hard | [Serialize Deserialize](../Design%20Data%20Structure%20Problems/Serialize%20Deserialize.md) |
| Diameter of Binary Tree | 543 | ⭐⭐⭐⭐ | Easy | [Tree DP](../Patterns/Tree%20DP.md) |
| Binary Tree Right Side View | 199 | ⭐⭐⭐⭐ | Medium | [Level Order](../Patterns/Level%20Order.md) |
| All Nodes Distance K in Binary Tree | 863 | ⭐⭐⭐ | Medium | [Path Problems](../Patterns/Path%20Problems.md) |
| Zigzag Level Order Traversal | 103 | ⭐⭐⭐ | Medium | [Level Order](../Patterns/Level%20Order.md) |
| Construct Binary Tree from Preorder and Inorder | 105 | ⭐⭐⭐ | Medium | [Tree Construction](../Patterns/Tree%20Construction.md) |
| Path Sum II | 113 | ⭐⭐⭐ | Medium | [Path Problems](../Patterns/Path%20Problems.md) |

## OA Tips

- Amazon OAs are usually 2 coding problems in ~70-90 minutes, auto-graded on hidden test cases — optimize for correctness and edge cases (empty tree, single node, skewed tree) over micro-optimizations.
- Iterative traversals are safer than deep recursion when input sizes are large; watch for stack-overflow risk on skewed inputs.
- Many tree questions are disguised as "org chart" or "delivery network" word problems — translate to nodes/edges quickly.
- A behavioral/Leadership-Principles section often follows the coding round; budget time accordingly and don't over-engineer.

> **Last Updated:** 2026-06-26
