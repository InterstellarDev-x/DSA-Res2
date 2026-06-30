> **Topic:** [Binary Trees](../README.md) · **Company 2 of 5**

# Google — Binary Trees OA Questions

Google's assessments favor problems with subtle correctness requirements and elegant recursive insights over brute force. Tree questions frequently combine traversal with coordinate geometry (vertical order), greedy reasoning (coin distribution), or careful definitions (deepest leaves). Clean, provably-correct O(n) solutions with strong reasoning about invariants are the differentiator.

## Problem Bank

| Problem | LC # | Frequency | Difficulty | Pattern |
|---------|------|-----------|------------|---------|
| Vertical Order Traversal of a Binary Tree | 987 | ⭐⭐⭐⭐⭐ | Hard | [Tree Traversals](../Patterns/Tree%20Traversals.md) |
| Lowest Common Ancestor of Deepest Leaves | 1123 | ⭐⭐⭐⭐ | Medium | [Path Problems](../Patterns/Path%20Problems.md) |
| Distribute Coins in Binary Tree | 979 | ⭐⭐⭐⭐ | Medium | [Tree DP](../Patterns/Tree%20DP.md) |
| Binary Tree Cameras | 968 | ⭐⭐⭐⭐ | Hard | [Tree DP](../Patterns/Tree%20DP.md) |
| Lowest Common Ancestor of a Binary Tree | 236 | ⭐⭐⭐⭐ | Medium | [Path Problems](../Patterns/Path%20Problems.md) |
| All Nodes Distance K in Binary Tree | 863 | ⭐⭐⭐ | Medium | [Path Problems](../Patterns/Path%20Problems.md) |
| Maximum Width of Binary Tree | 662 | ⭐⭐⭐ | Medium | [Level Order](../Patterns/Level%20Order.md) |
| Delete Nodes And Return Forest | 1110 | ⭐⭐⭐ | Medium | [Tree Traversals](../Patterns/Tree%20Traversals.md) |
| Binary Tree Maximum Path Sum | 124 | ⭐⭐⭐ | Hard | [Tree DP](../Patterns/Tree%20DP.md) |
| Serialize and Deserialize Binary Tree | 297 | ⭐⭐⭐ | Hard | [Serialize Deserialize](../Design%20Data%20Structure%20Problems/Serialize%20Deserialize.md) |

## OA Tips

- Google weights edge-case correctness heavily — verify column/row tie-breaking in vertical order, and define "deepest leaves" precisely before coding.
- Expect follow-up reasoning: be ready to state and justify your time/space complexity, since onsite rounds revisit OA problems.
- Greedy tree problems (Cameras, Distribute Coins) reward a post-order "return state up" formulation — model node states explicitly.
- Avoid global mutable state where a clean return value works; interviewers value functional, side-effect-light recursion.

> **Last Updated:** 2026-06-26
