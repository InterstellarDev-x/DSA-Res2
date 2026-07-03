> **Topic:** [Binary Search Trees](../README.md) · **Pattern 4 of 4**

# BST LCA & Ancestors

**Core idea:** the BST ordering lets you answer ancestor/neighbor queries with a single **O(h)
descent** and **O(1) extra space** — no parent pointers, no full-tree search. You compare the
target(s) against the current node and decide which way to go, updating a candidate as you pass.

```cpp
#include <bits/stdc++.h>
using namespace std;

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
```

> **Recognition signals:** "lowest common ancestor (and it's a BST)", "inorder successor /
> predecessor", "floor / ceil", "closest value ≤ / ≥ x", "the split point".

---

## 1. Lowest Common Ancestor of a BST (LC 235) — Medium

Walk down from the root. If **both** `p` and `q` are smaller than the current node, the LCA is in
the left subtree; if **both** are larger, go right. The first node where they **split** (one on
each side, or one equals the node) is the LCA. No recursion or extra space required.

```cpp
#include <bits/stdc++.h>
using namespace std;

TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
    TreeNode* cur = root;
    while (cur != nullptr) {
        if (p->val < cur->val && q->val < cur->val)      cur = cur->left;   // both smaller
        else if (p->val > cur->val && q->val > cur->val) cur = cur->right;  // both larger
        else return cur;                                                     // split point = LCA
    }
    return nullptr;
}
```

**Complexity:** Time O(h); Space O(1).

> Contrast with the [general binary-tree LCA](../../11.Binary_Trees/Patterns/Tree%20DP.md) (LC 236),
> which has no ordering to exploit and must search both subtrees in O(n).

---

## 2. Inorder Successor in BST (LC 285) — Medium

The inorder successor of `p` is the smallest key strictly greater than `p.val`. Two cases:

- **`p` has a right subtree** → the successor is its **leftmost** node.
- **No right subtree** → the successor is the **lowest ancestor for which `p` is in its left
  subtree**. While descending from the root toward `p`, every time we move *left* we found a
  candidate successor; the last such candidate is the answer.

A single descent handles both cases:

```cpp
#include <bits/stdc++.h>
using namespace std;

TreeNode* inorderSuccessor(TreeNode* root, TreeNode* p) {
    TreeNode* successor = nullptr;
    TreeNode* cur = root;
    while (cur != nullptr) {
        if (p->val < cur->val) {
            successor = cur;        // candidate: we may come back up to it
            cur = cur->left;
        } else {
            cur = cur->right;       // p->val >= cur->val: successor is further right
        }
    }
    return successor;
}
```

> When `p` *does* have a right subtree, the loop naturally descends right past `p` and then keeps
> going left into that subtree, landing on its leftmost node — the same answer the two-case
> reasoning gives. The **predecessor** is the mirror: track a candidate whenever you move *right*.

**Complexity:** Time O(h); Space O(1).

---

## 3. Floor and Ceil in BST — Medium

- **Floor(x)** = the largest key **≤ x**.
- **Ceil(x)**  = the smallest key **≥ x**.

Descend once. When the current value qualifies as a candidate, record it and move toward a
*tighter* candidate.

```cpp
#include <bits/stdc++.h>
using namespace std;

// Largest value <= x, or INT_MIN if none.
int floor(TreeNode* root, int x) {
    int best = INT_MIN;
    TreeNode* cur = root;
    while (cur != nullptr) {
        if (cur->val == x) return x;
        if (cur->val < x) {           // candidate floor; try to find a larger one on the right
            best = cur->val;
            cur = cur->right;
        } else {
            cur = cur->left;          // too big, go smaller
        }
    }
    return best;
}

// Smallest value >= x, or INT_MAX if none.
int ceil(TreeNode* root, int x) {
    int best = INT_MAX;
    TreeNode* cur = root;
    while (cur != nullptr) {
        if (cur->val == x) return x;
        if (cur->val > x) {           // candidate ceil; try to find a smaller one on the left
            best = cur->val;
            cur = cur->left;
        } else {
            cur = cur->right;         // too small, go larger
        }
    }
    return best;
}
```

**Complexity:** Time O(h); Space O(1) each.

> C++'s `std::map`/`std::set` expose exactly these as `lower_bound`, `upper_bound` — all O(log n). See [BST vs Hash vs Heap](../Interview%20Tips/BST%20vs%20Hash%20vs%20Heap.md).

---

## Dry Run — LCA of `p=2`, `q=4`

```
        6
       / \
      2   8
     / \
    0   4
       / \
      3   5
```

1. cur = 6: `2 < 6` and `4 < 6` → both smaller → go **left** to `2`.
2. cur = 2: `p->val == cur->val` → not "both smaller" and not "both larger" → **split / equal**.
   Return `2`.

LCA(2, 4) = **2** (a node can be its own ancestor). The walk visited only 2 nodes — O(h).

---

## Summary

| Problem | Technique | Time | Space |
|---|---|---|---|
| LCA of BST (235) | descend to split point | O(h) | O(1) |
| Inorder Successor (285) | leftmost-of-right or last left-turn ancestor | O(h) | O(1) |
| Floor / Ceil | descend, update tighter candidate | O(h) | O(1) |

> **Last Updated:** 2026-06-26
