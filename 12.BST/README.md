# Binary Search Trees — Step 14

> ← [Binary Trees](../11.Binary_Trees/README.md) | [Graphs](../13.Graphs/README.md) →

A production-quality, interview-focused reference for **Binary Search Trees (BST)**: 16 problems
organized into 4 reusable patterns, plus a design problem (BST Iterator), company question banks,
interview deep dives, and tips. Every solution is full Rust — no pseudocode.

```rust
#[derive(Debug, Clone, PartialEq)]
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

## Mental Model — The One Invariant That Drives Everything

> **The BST invariant:** for *every* node, **all** keys in the left subtree are `< node.val`,
> and **all** keys in the right subtree are `> node.val`. This must hold against *all
> ancestors*, not just the direct parent.

Three consequences power nearly every solution on this page:

1. **Inorder traversal yields a strictly sorted sequence.** Validation, Kth-smallest,
   min-abs-difference, recover, two-sum, and "convert to greater tree" all reduce to "walk the
   sorted order" (left→node→right, or right→node→left for descending).
2. **You can navigate in O(h) without visiting every node.** Search, insert, delete, LCA, floor,
   ceil, and successor *compare the target against the current node and descend one side*. With a
   balanced tree `h = O(log n)`; with a degenerate (skewed) tree `h = O(n)`.
3. **You can prune.** Range queries (Range Sum) and bounded validation skip entire subtrees that
   cannot contain a valid answer.

`h` = tree height. Read it everywhere as **O(log n) balanced / O(n) skewed**. Self-balancing
variants (Red-Black, AVL) — and Rust's `BTreeMap`/`BTreeSet` — guarantee `h = O(log n)`.

---

## All 16 Problems

| # | Problem | LC | Difficulty | Pattern | Companies |
|---|---|---|---|---|---|
| 1 | Search in a BST | 700 | Easy | [Operations](Patterns/BST%20Operations.md) | Adobe |
| 2 | Insert into a BST | 701 | Medium | [Operations](Patterns/BST%20Operations.md) | Adobe |
| 3 | Delete Node in a BST | 450 | Medium | [Operations](Patterns/BST%20Operations.md) | MS |
| 4 | Minimum Absolute Difference in BST | 530 | Easy | [Operations](Patterns/BST%20Operations.md) | Amazon |
| 5 | Validate Binary Search Tree | 98 | Medium | [Validation & Inorder](Patterns/BST%20Validation%20and%20Inorder.md) | Amazon, MS |
| 6 | Kth Smallest Element in a BST | 230 | Medium | [Validation & Inorder](Patterns/BST%20Validation%20and%20Inorder.md) | Amazon, Google |
| 7 | Two Sum IV - Input is a BST | 653 | Easy | [Validation & Inorder](Patterns/BST%20Validation%20and%20Inorder.md) | Adobe |
| 8 | Recover Binary Search Tree | 99 | Hard | [Validation & Inorder](Patterns/BST%20Validation%20and%20Inorder.md) | Google |
| 9 | Convert BST to Greater Tree | 538 | Medium | [Validation & Inorder](Patterns/BST%20Validation%20and%20Inorder.md) | MS |
| 10 | Range Sum of BST | 938 | Easy | [Validation & Inorder](Patterns/BST%20Validation%20and%20Inorder.md) | GS |
| 11 | Convert Sorted Array to BST | 108 | Easy | [Construction](Patterns/BST%20Construction.md) | MS |
| 12 | Construct BST from Preorder Traversal | 1008 | Medium | [Construction](Patterns/BST%20Construction.md) | MS |
| 13 | Balance a Binary Search Tree | 1382 | Medium | [Construction](Patterns/BST%20Construction.md) | Google |
| 14 | Lowest Common Ancestor of a BST | 235 | Medium | [LCA & Ancestors](Patterns/BST%20LCA%20and%20Ancestors.md) | Amazon, GS |
| 15 | Inorder Successor in BST | 285 | Medium | [LCA & Ancestors](Patterns/BST%20LCA%20and%20Ancestors.md) | Google |
| 16 | Floor and Ceil in BST | — | Medium | [LCA & Ancestors](Patterns/BST%20LCA%20and%20Ancestors.md) | Amazon |

*MS = Microsoft, GS = Goldman Sachs.*

---

## Patterns Summary

| Pattern | Problems | Core Idea |
|---|---|---|
| [BST Operations](Patterns/BST%20Operations.md) | 700, 701, 450, 530 | Compare-and-descend in O(h). Search iteratively; insert attaches at a null leaf; delete handles 3 cases (replace two-child node with its inorder successor). Min-abs-diff tracks `prev` during inorder. |
| [BST Validation & Inorder](Patterns/BST%20Validation%20and%20Inorder.md) | 98, 230, 653, 99, 538, 938 | Inorder = sorted. Validate with (low, high) bounds (use `long`) or strictly-increasing inorder; Kth-smallest via stack with early stop; recover finds the two swapped nodes; greater-tree uses *reverse* inorder; range-sum prunes. |
| [BST Construction](Patterns/BST%20Construction.md) | 108, 1008, 1382 | Sorted array → pick mid as root recursively (height-balanced). Construct-from-preorder uses an upper-bound technique in O(n). Balance = inorder to sorted array, then array→BST. |
| [BST LCA & Ancestors](Patterns/BST%20LCA%20and%20Ancestors.md) | 235, 285, Floor/Ceil | O(h) walk using the ordering. LCA descends until the split point; successor = leftmost of right subtree else last left-turn ancestor; floor/ceil update candidates while descending. |

---

## Design Problems

| Problem | LC | File |
|---|---|---|
| Binary Search Tree Iterator | 173 | [BST Iterator](Design%20Data%20Structure%20Problems/BST%20Iterator.md) |

Controlled inorder using an explicit stack — `next()` and `hasNext()` are **amortized O(1)** with
**O(h) space**. Includes the reverse (predecessor) iterator follow-up.

---

## Company Coverage

| Company | OA Bank | Interview Deep Dive | Headline Problems |
|---|---|---|---|
| Amazon | [OA](OA-Qns/Amazon.md) | [Interview](Interview%20Problems/Amazon.md) | Validate BST ⭐⭐⭐⭐⭐, Kth Smallest ⭐⭐⭐⭐, LCA of BST ⭐⭐⭐⭐ |
| Google | [OA](OA-Qns/Google.md) | [Interview](Interview%20Problems/Google.md) | Kth Smallest ⭐⭐⭐⭐⭐, Recover BST ⭐⭐⭐⭐, Inorder Successor ⭐⭐⭐⭐ |
| Microsoft | [OA](OA-Qns/Microsoft.md) | [Interview](Interview%20Problems/Microsoft.md) | Validate BST ⭐⭐⭐⭐, Delete Node ⭐⭐⭐, Convert Sorted Array ⭐⭐⭐ |
| Goldman Sachs | [OA](OA-Qns/Goldman%20Sachs.md) | — | LCA of BST ⭐⭐⭐, Range Sum ⭐⭐⭐ |
| Adobe | [OA](OA-Qns/Adobe.md) | — | Search/Insert ⭐⭐⭐, Two Sum IV ⭐⭐⭐ |

---

## Directory Map

- **[Patterns/](Patterns/)** — the four core pattern playbooks
  - [BST Operations](Patterns/BST%20Operations.md)
  - [BST Validation and Inorder](Patterns/BST%20Validation%20and%20Inorder.md)
  - [BST Construction](Patterns/BST%20Construction.md)
  - [BST LCA and Ancestors](Patterns/BST%20LCA%20and%20Ancestors.md)
- **[Design Data Structure Problems/](Design%20Data%20Structure%20Problems/)**
  - [BST Iterator](Design%20Data%20Structure%20Problems/BST%20Iterator.md)
- **[OA-Qns/](OA-Qns/)** — company online-assessment banks: [Amazon](OA-Qns/Amazon.md) ·
  [Google](OA-Qns/Google.md) · [Microsoft](OA-Qns/Microsoft.md) ·
  [Goldman Sachs](OA-Qns/Goldman%20Sachs.md) · [Adobe](OA-Qns/Adobe.md)
- **[Interview Problems/](Interview%20Problems/)** — deep dives:
  [Amazon](Interview%20Problems/Amazon.md) · [Google](Interview%20Problems/Google.md) ·
  [Microsoft](Interview%20Problems/Microsoft.md)
- **[Interview Tips/](Interview%20Tips/)** — [Coding Tips](Interview%20Tips/Coding%20Tips.md) ·
  [Common Mistakes](Interview%20Tips/Common%20Mistakes.md) ·
  [BST vs Hash vs Heap](Interview%20Tips/BST%20vs%20Hash%20vs%20Heap.md) ·
  [Complexity Analysis](Interview%20Tips/Complexity%20Analysis.md)
- **[Most Recent Questions/](Most%20Recent%20Questions/)** —
  [2024](Most%20Recent%20Questions/2024.md) · [2025](Most%20Recent%20Questions/2025.md) ·
  [2026](Most%20Recent%20Questions/2026.md)

---

## How to Study This Topic
1. Internalize the **invariant** and the fact that **inorder = sorted** — half the problems
   collapse to this.
2. Drill the **compare-and-descend O(h) walk**: Search → LCA → Floor/Ceil → Successor.
3. Master **Validate BST** with the (low, high) bounds technique and the `long` overflow trap —
   the single most-asked BST problem.
4. Learn the **two-children Delete** case (replace with inorder successor) cold.
5. Practice the **BST Iterator** (controlled inorder) — a frequent design follow-up.
6. Cross-reference the general [LCA in a binary tree](../11.Binary_Trees/Patterns/Tree%20DP.md)
   to appreciate why the BST version is dramatically simpler.


> **Last Updated:** 2026-06-26
