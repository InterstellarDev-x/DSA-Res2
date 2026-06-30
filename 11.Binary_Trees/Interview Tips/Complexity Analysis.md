> **Topic:** [Binary Trees](../README.md) · **Pattern 4 of 4**

# Binary Trees — Complexity Analysis

Most binary-tree problems share the same complexity DNA. Once you understand *why* traversal is O(n) time and *why* space is O(h), you can read off the complexity of nearly any tree problem on sight. This page explains the reasoning, the worst-case behavior, and then gives a full per-problem table for all 38 problems in this step.

Node definition referenced throughout:

```java
public class TreeNode {
    int val;
    TreeNode left, right;
    TreeNode(int val) { this.val = val; }
}
```

---

## Why traversal is O(n) time

A traversal (inorder, preorder, postorder, level-order, or any single DFS/BFS) **visits each of the `n` nodes exactly once** and does a constant amount of work per node — read the value, follow two child pointers, push/pop one queue or stack entry. With `n` nodes and O(1) work each, the total is **O(n)** time. Almost every problem here that "looks at the whole tree once" is O(n): traversals, depth/height, sum, invert, symmetric, same-tree, diameter, path sums, and so on.

The exceptions are problems that do *more* than constant work per node (e.g. inserting at index 0 of a list, or running a sub-traversal at each node), which we call out in the table.

## Why space is O(h), and what `h` is

For recursive solutions, the extra space is dominated by the **recursion call stack**, and the maximum stack depth equals the **height `h`** of the tree — because at the deepest point of a DFS, you have one stack frame per node along a single root-to-leaf path.

- **Balanced tree:** `h = O(log n)`, so recursion uses **O(log n)** space.
- **Skewed / degenerate tree** (each node has one child — effectively a linked list): `h = n`, so recursion uses **O(n)** space.

So when we write space as **O(h)**, that's `O(log n)` in the best/balanced case and `O(n)` in the worst/skewed case. The same logic applies to an explicit `Deque<TreeNode> stack` — it holds at most one path's worth of nodes.

For **BFS / level-order**, space is governed instead by the **maximum number of nodes on any single level**, which can be up to `n/2` (the bottom level of a complete tree). That is **O(n)** space regardless of height.

## Skewed-tree worst case

Whenever you see **O(h)** space, remember the worst case is a fully skewed tree where `h = n`. A method that is "O(log n) space on a balanced tree" is genuinely **O(n) space in the worst case**. This also matters for *correctness*, not just cost: a skewed tree of ~10⁵ nodes will overflow the JVM call stack in a recursive solution — see the iterative/Morris discussion in *Recursive Thinking*.

## Special case: Count Complete Tree Nodes (LC 222)

A naive count is O(n). But a **complete** binary tree lets you do better. By computing the left-spine height and the right-spine height of a subtree:
- if they are equal, the subtree is a *perfect* tree and its size is `2^h - 1` in O(h) — no recursion needed;
- otherwise recurse into both children.

Each level triggers an O(log n) height check, and only O(log n) subtrees are recursed into, giving **O(log² n)** time (and O(log n) space). This is the one problem in the set that beats O(n).

---

## Full complexity table — all 38 problems

`n` = number of nodes, `h` = height (`O(log n)` balanced, `O(n)` skewed). For DFS solutions, "Space O(h)" includes the recursion stack.

| Problem | LC # | Time | Space | Notes |
|---|---|---|---|---|
| Binary Tree Inorder Traversal | 94 | O(n) | O(h) | Recursion/stack O(h); Morris gives O(1) extra space |
| Binary Tree Preorder Traversal | 144 | O(n) | O(h) | Explicit `Deque` stack mirrors recursion depth |
| Binary Tree Postorder Traversal | 145 | O(n) | O(h) | Two-stack or reversed-preorder trick |
| Binary Tree Level Order Traversal | 102 | O(n) | O(n) | BFS; queue holds up to a full level (~n/2) |
| Zigzag Level Order Traversal | 103 | O(n) | O(n) | Use `LinkedList.addFirst` (O(1)) to avoid O(n) inserts |
| N-ary Tree Level Order Traversal | 429 | O(n) | O(n) | BFS over children list per node |
| Vertical Order Traversal | 987 | O(n log n) | O(n) | Sort by (col, row, val) dominates; map keyed by column |
| Construct from Preorder + Inorder | 105 | O(n) | O(n) | HashMap of inorder indices avoids O(n) scans |
| Construct from Inorder + Postorder | 106 | O(n) | O(n) | Same idea; consume postorder right-to-left |
| Construct from Preorder + Postorder | 889 | O(n) | O(n) | Index map for postorder positions |
| Maximum Width of Binary Tree | 662 | O(n) | O(n) | BFS with normalized heap indices to avoid overflow |
| Flatten Binary Tree to Linked List | 114 | O(n) | O(h) | Morris-style threading gives O(1) extra space |
| Populating Next Right Pointers (perfect) | 116 | O(n) | O(1) | Use established `next` pointers; no queue needed |
| Binary Tree Maximum Path Sum | 124 | O(n) | O(h) | Global best + clamped child gains `Math.max(0, gain)` |
| Path Sum | 112 | O(n) | O(h) | Subtract along path; check at leaves |
| Path Sum II | 113 | O(n²) | O(n) | Copying each root-to-leaf path can cost O(n) each |
| Sum Root to Leaf Numbers | 129 | O(n) | O(h) | Accumulate `cur*10 + val` down the path |
| Binary Tree Paths | 257 | O(n²) | O(n) | Building each path string is O(h); many paths |
| Diameter of Binary Tree | 543 | O(n) | O(h) | Return height, update global `left + right` |
| Maximum Depth | 104 | O(n) | O(h) | Bottom-up height |
| Minimum Depth | 111 | O(n) | O(h) | BFS can early-exit at first leaf; treat one-child nodes carefully |
| Balanced Binary Tree | 110 | O(n) | O(h) | Bottom-up height with `-1` sentinel for imbalance |
| Symmetric Tree | 101 | O(n) | O(h) | Mirror-compare left vs right |
| Same Tree | 100 | O(n) | O(h) | Structural + value equality |
| Invert Binary Tree | 226 | O(n) | O(h) | Swap children at every node |
| Count Complete Tree Nodes | 222 | O(log² n) | O(log n) | Special: perfect-subtree height trick beats O(n) |
| Binary Tree Right Side View | 199 | O(n) | O(n) | BFS: last node per level (or DFS right-first) |
| Average of Levels | 637 | O(n) | O(n) | BFS sum per level; use `long`/`double` to avoid overflow |
| Largest Value in Each Tree Row | 515 | O(n) | O(n) | BFS max per level |
| Populating Next Right Pointers II (any tree) | 117 | O(n) | O(1) | Dummy-head trick walks level by level via `next` |
| Level Order Traversal II (bottom-up) | 107 | O(n) | O(n) | BFS then reverse, or `addFirst` into a `LinkedList` |
| House Robber III | 337 | O(n) | O(h) | Return `{rob, skip}` pair per node (tree DP) |
| Binary Tree Cameras | 968 | O(n) | O(h) | Greedy post-order states; check root after recursion |
| Distribute Coins in Binary Tree | 979 | O(n) | O(h) | Sum `abs(balance)` of moves bubbling up |
| Lowest Common Ancestor | 236 | O(n) | O(h) | Return self on match; split point is the LCA |
| LCA of Deepest Leaves | 1123 | O(n) | O(h) | Return (depth, lca) pair bottom-up |
| Delete Nodes And Return Forest | 1110 | O(n) | O(n) | HashSet of to-delete; collect orphaned roots |
| All Nodes Distance K | 863 | O(n) | O(n) | Build child→parent map, then BFS outward |
| Serialize and Deserialize Binary Tree | 297 | O(n) | O(n) | Preorder with null markers; queue-driven rebuild |

### Notes on the entries that are *not* O(n) time

- **Vertical Order Traversal (987):** O(n log n) because nodes within and across columns must be sorted by (column, row, value).
- **Path Sum II (113)** and **Binary Tree Paths (257):** the *traversal* is O(n), but **emitting** each root-to-leaf path costs up to O(h) per path, and there can be O(n) leaves — worst case O(n·h) which is O(n²) on a skewed/bushy tree.
- **Count Complete Tree Nodes (222):** O(log² n) thanks to the perfect-subtree height shortcut described above.

### Notes on the O(1)-space entries

- **Populating Next Right Pointers I & II (116 / 117):** these achieve **O(1)** extra space by reusing the `next` pointers already being built to walk each level — no queue or recursion stack required (output pointers don't count as extra space).

---

> **Last Updated:** 2026-06-26
