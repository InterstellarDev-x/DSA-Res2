> **Topic:** [Binary Search Trees](../README.md) · **Tips 4 of 4**

# Complexity Analysis — BST

The single most important fact: most BST operations are **O(h)** where `h` is the height. That `h`
is **O(log n)** for a balanced tree and **O(n)** for a degenerate (skewed) one. Inorder-based
problems that must visit every node are **O(n)** regardless.

---

## All 16 Problems

| # | Problem | LC | Time | Space | Notes |
|---|---|---|---|---|---|
| 1 | Search in a BST | 700 | O(h) | O(1) iter / O(h) rec | compare-and-descend |
| 2 | Insert into a BST | 701 | O(h) | O(1) iter / O(h) rec | attach at null leaf |
| 3 | Delete Node in a BST | 450 | O(h) | O(h) | two-children → successor |
| 4 | Min Absolute Difference | 530 | O(n) | O(h) | inorder, track `prev` |
| 5 | Validate BST | 98 | O(n) | O(h) | bounds or strict inorder |
| 6 | Kth Smallest | 230 | O(h + k) | O(h) | stack inorder, early stop |
| 7 | Two Sum IV | 653 | O(n) | O(n) | HashSet or inorder + 2-ptr |
| 8 | Recover BST | 99 | O(n) | O(h), O(1) Morris | find 2 swapped nodes |
| 9 | Convert to Greater Tree | 538 | O(n) | O(h) | reverse inorder |
| 10 | Range Sum of BST | 938 | O(n) worst, pruned avg | O(h) | skip out-of-range subtrees |
| 11 | Sorted Array → BST | 108 | O(n) | O(log n) | mid as root |
| 12 | Construct from Preorder | 1008 | O(n) | O(h) | upper-bound, each elem once |
| 13 | Balance a BST | 1382 | O(n) | O(n) | inorder list + array→BST |
| 14 | LCA of BST | 235 | O(h) | O(1) | descend to split point |
| 15 | Inorder Successor | 285 | O(h) | O(1) | leftmost-of-right / left-turn |
| 16 | Floor and Ceil | — | O(h) | O(1) | descend, tighten candidate |

`k` = the rank requested; `h` = tree height; `n` = node count.

---

## Reading `O(h)`

```
Balanced BST (h = log n)        Skewed BST (h = n)
        4                        1
       / \                        \
      2   6      h = 2             2          h = n-1
     /\   /\                        \
    1 3  5 7                         3  ...
```

- **Balanced:** `h = ⌊log₂ n⌋`, so O(h) = **O(log n)**. Search/insert/delete/LCA/floor/ceil are all
  logarithmic — the headline selling point of a BST.
- **Skewed (degenerate):** inserting sorted data into a plain BST produces a linked list,
  `h = n - 1`, so every O(h) operation degrades to **O(n)**. This is the trap a self-balancing tree
  removes.

---

## Self-Balancing & Why `TreeMap` Guarantees O(log n)

A plain BST does **not** guarantee balance. Self-balancing variants do:

- **Red-Black tree** — each node is colored red/black; coloring invariants keep the longest
  root-to-leaf path at most twice the shortest, so `h ≤ 2·log₂(n+1)` = **O(log n)**. Rebalancing on
  insert/delete is O(log n) with O(1) amortized rotations.
- **AVL tree** — stricter height balance (subtree heights differ by ≤ 1); slightly faster lookups,
  slightly more rotations on update.

Java's **`TreeMap` and `TreeSet` are Red-Black trees**, which is why `get`, `put`, `remove`,
`floorKey`, `ceilingKey`, `higherKey`, `lowerKey`, and `subMap` are all **guaranteed O(log n)** —
the worst-case skew of a hand-rolled BST cannot happen. See
[BST vs Hash vs Heap](BST%20vs%20Hash%20vs%20Heap.md).

---

## Space Notes

- **Iterative** search/insert/LCA/floor/ceil/successor use **O(1)** extra space.
- **Recursive** versions and stack-driven inorder use **O(h)** for the call stack / explicit stack.
- **Morris traversal** rewires `null` right pointers temporarily to achieve **O(1)** space inorder
  (used in the Recover BST follow-up) at the cost of trickier code.
- Building a list (Two Sum IV variant, Balance a BST) is **O(n)** auxiliary.

> **Last Updated:** 2026-06-26
