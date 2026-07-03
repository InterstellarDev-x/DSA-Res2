# Binary Trees — Step 13

> ← [Greedy Algorithms](../Greedy%20Algorithms/README.md) | [Binary Search Trees](../Binary%20Search%20Trees/README.md) →

A production-quality, interview-focused reference for **Binary Trees**: 38 problems organized
into 5 reusable patterns, plus a design problem, company question banks, interview deep dives,
and tips. Every solution is full Rust — no pseudocode.

```rust
#[derive(Debug)]
pub struct TreeNode {
    pub val: i32,
    pub left: Option<Box<TreeNode>>,
    pub right: Option<Box<TreeNode>>,
}

impl TreeNode {
    pub fn new(val: i32) -> Self {
        TreeNode { val, left: None, right: None }
    }
}
```

---

## Mental Model — Recursion vs BFS vs Iterative DFS

| Use… | When | Signal phrases |
|---|---|---|
| **Recursion (DFS)** | The answer composes from subtree answers (depth, sum, balance, LCA, DP). Default choice. | "for every node…", "subtree…", "path from root" |
| **BFS (queue, level snapshot)** | You need per-**level** information or the **shortest** depth. | "level order", "right side view", "minimum depth", "by row" |
| **Iterative DFS (explicit stack)** | Recursion is conceptually right but you must avoid stack overflow on deep/skewed trees, or the interviewer asks for non-recursive. | "without recursion", "very large tree", "O(1) / iterative" |
| **Morris (threaded)** | True **O(1)** space inorder/flatten is required. | "constant space traversal" |

**The two recursion shapes to internalize:**
1. **Top-down:** push state *down* via parameters (e.g. running path sum, current depth).
2. **Bottom-up:** return a *summary* of each subtree *up*, and optionally update a **global**
   (diameter, max path sum, House Robber III, Cameras). The thing you *return* to the parent
   and the thing you *maximize globally* are often different.

---

## All 38 Problems

| # | Problem | LC | Difficulty | Pattern | Companies |
|---|---|---|---|---|---|
| 1 | Binary Tree Inorder Traversal | 94 | Easy | [Traversals](Patterns/Tree%20Traversals.md) | MS, Adobe |
| 2 | Binary Tree Preorder Traversal | 144 | Easy | [Traversals](Patterns/Tree%20Traversals.md) | MS |
| 3 | Binary Tree Postorder Traversal | 145 | Easy | [Traversals](Patterns/Tree%20Traversals.md) | MS |
| 4 | Binary Tree Level Order Traversal | 102 | Medium | [Level Order](Patterns/Level%20Order.md) | Amazon, MS |
| 5 | Binary Tree Zigzag Level Order | 103 | Medium | [Traversals](Patterns/Tree%20Traversals.md) | Amazon, MS |
| 6 | N-ary Tree Level Order Traversal | 429 | Medium | [Level Order](Patterns/Level%20Order.md) | Google |
| 7 | Vertical Order Traversal | 987 | Hard | [Traversals](Patterns/Tree%20Traversals.md) | Google |
| 8 | Construct from Preorder & Inorder | 105 | Medium | [Construction](Patterns/Tree%20Construction.md) | Amazon, MS |
| 9 | Construct from Inorder & Postorder | 106 | Medium | [Construction](Patterns/Tree%20Construction.md) | MS |
| 10 | Construct from Preorder & Postorder | 889 | Medium | [Construction](Patterns/Tree%20Construction.md) | Google |
| 11 | Maximum Width of Binary Tree | 662 | Medium | [Construction](Patterns/Tree%20Construction.md) | Amazon |
| 12 | Flatten Binary Tree to Linked List | 114 | Medium | [Construction](Patterns/Tree%20Construction.md) | Amazon, MS |
| 13 | Populating Next Right Pointers | 116 | Medium | [Construction](Patterns/Tree%20Construction.md) | Amazon |
| 14 | Binary Tree Maximum Path Sum | 124 | Hard | [Path](Patterns/Path%20Problems.md) | Amazon, GS |
| 15 | Path Sum | 112 | Easy | [Path](Patterns/Path%20Problems.md) | Amazon |
| 16 | Path Sum II | 113 | Medium | [Path](Patterns/Path%20Problems.md) | Adobe |
| 17 | Sum Root to Leaf Numbers | 129 | Medium | [Path](Patterns/Path%20Problems.md) | Amazon |
| 18 | Binary Tree Paths | 257 | Easy | [Path](Patterns/Path%20Problems.md) | Google |
| 19 | Diameter of Binary Tree | 543 | Easy | [Path](Patterns/Path%20Problems.md) | Adobe, Amazon |
| 20 | Maximum Depth of Binary Tree | 104 | Easy | [Path](Patterns/Path%20Problems.md) | All |
| 21 | Minimum Depth of Binary Tree | 111 | Easy | [Level Order](Patterns/Level%20Order.md) | MS |
| 22 | Balanced Binary Tree | 110 | Easy | [Path](Patterns/Path%20Problems.md) | Amazon |
| 23 | Symmetric Binary Tree | 101 | Easy | [Traversals](Patterns/Tree%20Traversals.md) | MS |
| 24 | Same Tree | 100 | Easy | [Traversals](Patterns/Tree%20Traversals.md) | MS |
| 25 | Invert Binary Tree | 226 | Easy | [Traversals](Patterns/Tree%20Traversals.md) | MS, Google |
| 26 | Count Complete Tree Nodes | 222 | Medium | [Construction](Patterns/Tree%20Construction.md) | Google |
| 27 | Binary Tree Right Side View | 199 | Medium | [Level Order](Patterns/Level%20Order.md) | Amazon, MS |
| 28 | Average of Levels | 637 | Easy | [Level Order](Patterns/Level%20Order.md) | Adobe |
| 29 | Largest Value in Each Tree Row | 515 | Medium | [Level Order](Patterns/Level%20Order.md) | MS |
| 30 | Populating Next Right Pointers II | 117 | Medium | [Level Order](Patterns/Level%20Order.md) | Amazon |
| 31 | Level Order Traversal II (bottom-up) | 107 | Medium | [Level Order](Patterns/Level%20Order.md) | Amazon |
| 32 | House Robber III | 337 | Medium | [Tree DP](Patterns/Tree%20DP.md) | GS, Amazon |
| 33 | Binary Tree Cameras | 968 | Hard | [Tree DP](Patterns/Tree%20DP.md) | Google, MS |
| 34 | Distribute Coins in Binary Tree | 979 | Medium | [Tree DP](Patterns/Tree%20DP.md) | Google |
| 35 | Lowest Common Ancestor | 236 | Medium | [Tree DP](Patterns/Tree%20DP.md) | Amazon, MS |
| 36 | LCA of Deepest Leaves | 1123 | Medium | [Tree DP](Patterns/Tree%20DP.md) | Google |
| 37 | Delete Nodes and Return Forest | 1110 | Medium | [Tree DP](Patterns/Tree%20DP.md) | MS |
| 38 | All Nodes Distance K | 863 | Medium | [Tree DP](Patterns/Tree%20DP.md) | Google, Amazon |

*MS = Microsoft, GS = Goldman Sachs.*

---

## Patterns Summary

| Pattern | Problems | Core Idea |
|---|---|---|
| [Tree Traversals](Patterns/Tree%20Traversals.md) | 94, 144, 145, 102, 103, 429, 987, 101, 100, 226 | Pre/in/post & level orders; recursive 3-liners, stack-based iteration, Morris O(1), zigzag, vertical BTreeMap. |
| [Tree Construction](Patterns/Tree%20Construction.md) | 105, 106, 889, 662, 114, 116, 222 | Root identification + index splitting (HashMap lookup); in-place rewrites; indexed BFS with overflow normalization. |
| [Path Problems](Patterns/Path%20Problems.md) | 112, 113, 124, 129, 257, 543, 104, 110 | Leaf-base-case checks; backtracking for collection; "return depth, update global" for diameter/max-path-sum. |
| [Level Order](Patterns/Level%20Order.md) | 102, 199, 637, 515, 107, 429, 117, 111 | One BFS skeleton with a size snapshot; per-level aggregation; dummy-head for next pointers. |
| [Tree DP / Advanced](Patterns/Tree%20DP.md) | 337, 968, 979, 236, 1123, 1110, 863 | Post-order DFS returning subtree summaries; global result; greedy DP states; parent-map + BFS. |

---

## Design Problems

| Problem | LC | File |
|---|---|---|
| Serialize and Deserialize Binary Tree | 297 | [Serialize / Deserialize](Design%20Data%20Structure%20Problems/Serialize%20Deserialize.md) |

Covers DFS-preorder and BFS-level-order codecs, plus follow-ups for BST (LC 449) and N-ary
trees (LC 428).

---

## Company Coverage

| Company | OA Bank | Interview Deep Dive | Headline Problems |
|---|---|---|---|
| Amazon | [OA](OA-Qns/Amazon.md) | [Interview](Interview%20Problems/Amazon.md) | LCA ⭐⭐⭐⭐⭐, Max Path Sum ⭐⭐⭐⭐⭐, Serialize/Deserialize |
| Google | [OA](OA-Qns/Google.md) | [Interview](Interview%20Problems/Google.md) | Vertical Order ⭐⭐⭐⭐⭐, LCA Deepest Leaves, All Nodes Distance K |
| Microsoft | [OA](OA-Qns/Microsoft.md) | [Interview](Interview%20Problems/Microsoft.md) | Symmetric, Construct Pre+In, Delete Nodes Forest, Cameras |
| Goldman Sachs | [OA](OA-Qns/Goldman%20Sachs.md) | — | Max Path Sum ⭐⭐⭐⭐, House Robber III |
| Adobe | [OA](OA-Qns/Adobe.md) | — | Diameter ⭐⭐⭐⭐, Path Sum II |

---

## Directory Map

- **[Patterns/](Patterns/)** — the five core pattern playbooks
  - [Tree Traversals](Patterns/Tree%20Traversals.md)
  - [Tree Construction](Patterns/Tree%20Construction.md)
  - [Path Problems](Patterns/Path%20Problems.md)
  - [Level Order](Patterns/Level%20Order.md)
  - [Tree DP](Patterns/Tree%20DP.md)
- **[Design Data Structure Problems/](Design%20Data%20Structure%20Problems/)**
  - [Serialize Deserialize](Design%20Data%20Structure%20Problems/Serialize%20Deserialize.md)
- **[OA-Qns/](OA-Qns/)** — company online-assessment banks: [Amazon](OA-Qns/Amazon.md) ·
  [Google](OA-Qns/Google.md) · [Microsoft](OA-Qns/Microsoft.md) ·
  [Goldman Sachs](OA-Qns/Goldman%20Sachs.md) · [Adobe](OA-Qns/Adobe.md)
- **[Interview Problems/](Interview%20Problems/)** — deep dives:
  [Amazon](Interview%20Problems/Amazon.md) · [Google](Interview%20Problems/Google.md) ·
  [Microsoft](Interview%20Problems/Microsoft.md)
- **[Interview Tips/](Interview%20Tips/)** — [Coding Tips](Interview%20Tips/Coding%20Tips.md) ·
  [Common Mistakes](Interview%20Tips/Common%20Mistakes.md) ·
  [Recursive Thinking](Interview%20Tips/Recursive%20Thinking.md) ·
  [Complexity Analysis](Interview%20Tips/Complexity%20Analysis.md)
- **[Most Recent Questions/](Most%20Recent%20Questions/)** —
  [2024](Most%20Recent%20Questions/2024.md) · [2025](Most%20Recent%20Questions/2025.md) ·
  [2026](Most%20Recent%20Questions/2026.md)

---

## How to Study This Topic
1. Drill the **traversal** template (recursive + iterative + BFS) until it's automatic.
2. Internalize the **bottom-up "return + global"** recursion shape via Diameter → Max Path
   Sum → House Robber III.
3. Master the **BFS size-snapshot** and all its per-level variants.
4. Learn **LCA** and its variants — the single most-asked binary-tree problem in interviews.
5. Practice the **Serialize/Deserialize** codec — a frequent design follow-up.

> **Last Updated:** 2026-06-26
