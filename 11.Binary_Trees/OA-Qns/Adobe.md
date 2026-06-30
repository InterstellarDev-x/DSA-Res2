> **Topic:** [Binary Trees](../README.md) · **Company 5 of 5**

# Adobe — Binary Trees OA Questions

Adobe online assessments cover a balanced spread of tree problems — measuring tree shape (diameter, depth), enumerating root-to-leaf paths, and standard traversals. Questions are generally Medium difficulty with an emphasis on correct recursion and complete path enumeration. Clear logic and handling of all branches are what the grader checks.

## Problem Bank

| Problem | LC # | Frequency | Difficulty | Pattern |
|---------|------|-----------|------------|---------|
| Diameter of Binary Tree | 543 | ⭐⭐⭐⭐ | Easy | [Tree DP](../Patterns/Tree%20DP.md) |
| Path Sum II | 113 | ⭐⭐⭐ | Medium | [Path Problems](../Patterns/Path%20Problems.md) |
| Binary Tree Paths | 257 | ⭐⭐⭐ | Easy | [Path Problems](../Patterns/Path%20Problems.md) |
| Maximum Depth of Binary Tree | 104 | ⭐⭐⭐ | Easy | [Tree Traversals](../Patterns/Tree%20Traversals.md) |
| Binary Tree Level Order Traversal | 102 | ⭐⭐⭐ | Medium | [Level Order](../Patterns/Level%20Order.md) |
| Invert Binary Tree | 226 | ⭐⭐ | Easy | [Tree Traversals](../Patterns/Tree%20Traversals.md) |
| Symmetric Tree | 101 | ⭐⭐ | Easy | [Tree Traversals](../Patterns/Tree%20Traversals.md) |
| Flatten Binary Tree to Linked List | 114 | ⭐⭐ | Medium | [Tree Traversals](../Patterns/Tree%20Traversals.md) |
| Lowest Common Ancestor of a Binary Tree | 236 | ⭐⭐ | Medium | [Path Problems](../Patterns/Path%20Problems.md) |

## OA Tips

- Adobe OAs are commonly hosted on HackerRank with a mix of MCQs (aptitude, output prediction) plus 1-2 coding problems — pace the coding section so MCQs don't get squeezed.
- Path-enumeration problems require backtracking: remember to pop the current node after recursing into both children.
- Diameter and depth questions share a single-pass recursive template — reuse the helper that returns subtree height while updating a global best.
- Watch language-specific output formatting; some Adobe graders match printed output exactly, so follow the required format precisely.

> **Last Updated:** 2026-06-26
