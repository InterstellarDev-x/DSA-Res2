> **Topic:** [Binary Search Trees](../README.md) · **Pattern 2 of 4**

# BST Validation & Inorder

**Core idea:** the **inorder traversal of a BST is strictly sorted** (left → node → right). A huge
class of problems reduces to "process the keys in sorted order": validate, find the k-th smallest,
detect/repair swapped nodes, accumulate suffix sums (greater tree), and answer range queries. The
two workhorse tools are (a) the **(low, high) bounds** that an ancestor imposes on a subtree, and
(b) an explicit **stack-driven inorder** that can stop early.

```cpp
#include <bits/stdc++.h>
using namespace std;

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int val) : val(val), left(nullptr), right(nullptr) {}
};
```

> **Recognition signals:** "is this a valid BST", "k-th smallest/largest", "two nodes swapped /
> recover", "add all greater keys", "sum of keys in [low, high]", "find a pair summing to k".

---

## 1. Validate Binary Search Tree (LC 98) — Medium

The classic trap: it is **not enough** to check `left < node < right` against the direct parent.
*Every* node must lie within the bounds set by **all** its ancestors. Pass an open interval
`(low, high)` downward and tighten it as you descend.

Use `long` for the bounds so a node with value `INT_MIN` or `INT_MAX` does not
produce a false negative at the boundary.

```cpp
#include <bits/stdc++.h>
using namespace std;

bool valid(TreeNode* node, long low, long high) {
    if (node == nullptr) return true;
    if (node->val <= low || node->val >= high) return false;     // strict: no duplicates allowed
    return valid(node->left,  low, node->val)
        && valid(node->right, node->val, high);
}

bool isValidBST(TreeNode* root) {
    return valid(root, LONG_MIN, LONG_MAX);
}
```

**Alternative — strictly-increasing inorder.** Equivalent and avoids the `long` choice by storing
the previous node reference:

```cpp
#include <bits/stdc++.h>
using namespace std;

TreeNode* prev = nullptr;

bool inorder(TreeNode* node) {
    if (node == nullptr) return true;
    if (!inorder(node->left)) return false;
    if (prev != nullptr && node->val <= prev->val) return false;    // must be strictly increasing
    prev = node;
    return inorder(node->right);
}

bool isValidBSTInorder(TreeNode* root) {
    prev = nullptr;
    return inorder(root);
}
```

**Complexity:** Time O(n); Space O(h).

---

## 2. Kth Smallest Element in a BST (LC 230) — Medium

Inorder visits keys in ascending order, so the k-th visited node is the answer. The **iterative
stack** version is preferred: it stops as soon as the count reaches `k` (early exit) instead of
traversing the whole tree.

```cpp
#include <bits/stdc++.h>
using namespace std;

int kthSmallest(TreeNode* root, int k) {
    stack<TreeNode*> st;
    TreeNode* cur = root;
    while (cur != nullptr || !st.empty()) {
        while (cur != nullptr) {            // dive to the leftmost
            st.push(cur);
            cur = cur->left;
        }
        cur = st.top(); st.pop();           // visit in sorted order
        if (--k == 0) return cur->val;      // k-th smallest reached → stop
        cur = cur->right;
    }
    return -1;   // k out of range
}
```

**Complexity:** Time O(h + k) (dive once, then pop k); Space O(h).

> **Follow-up (frequent modifications):** augment each node with `leftCount` (size of its left
> subtree) so a single k-th query becomes O(h). See [Google Interview](../Interview%20Problems/Google.md).

---

## 3. Two Sum IV - Input is a BST (LC 653) — Easy

Two clean approaches.

**(a) `std::unordered_set` during any traversal** — O(n) time, O(n) space, does not exploit ordering:

```cpp
#include <bits/stdc++.h>
using namespace std;

bool dfs(TreeNode* node, int k, unordered_set<int>& seen) {
    if (node == nullptr) return false;
    if (seen.count(k - node->val)) return true;
    seen.insert(node->val);
    return dfs(node->left, k, seen) || dfs(node->right, k, seen);
}

bool findTarget(TreeNode* root, int k) {
    unordered_set<int> seen;
    return dfs(root, k, seen);
}
```

**(b) Inorder to sorted list + two pointers** — exploits sortedness; O(n) time, O(n) for the list:

```cpp
#include <bits/stdc++.h>
using namespace std;

void inorder(TreeNode* n, vector<int>& out) {
    if (n == nullptr) return;
    inorder(n->left, out);
    out.push_back(n->val);
    inorder(n->right, out);
}

bool findTargetTwoPtr(TreeNode* root, int k) {
    vector<int> sorted;
    inorder(root, sorted);
    int lo = 0, hi = (int)sorted.size() - 1;
    while (lo < hi) {
        int sum = sorted[lo] + sorted[hi];
        if (sum == k) return true;
        if (sum < k) lo++; else hi--;
    }
    return false;
}
```

---

## 4. Recover Binary Search Tree (LC 99) — Hard

Exactly **two** nodes of a valid BST were swapped. In the inorder sequence this shows up as one or
two "descents" (a place where `prev->val > cur->val`):

- **First descent** → record `first = prev` (the larger, earlier node).
- Record `second = cur` at **every** descent (the smaller, later node), so it ends up at the
  **last** violation.

If the two swapped nodes are **adjacent** in inorder there is only **one** descent (so
`second = cur` from that single descent). If they are **non-adjacent** there are **two** descents.
Tracking `first` only on the first descent and `second` on every descent handles both cases. Swap
the two values at the end.

```cpp
#include <bits/stdc++.h>
using namespace std;

TreeNode* first_node;
TreeNode* second_node;
TreeNode* prev_node;

void inorder(TreeNode* node) {
    if (node == nullptr) return;
    inorder(node->left);
    if (prev_node != nullptr && prev_node->val > node->val) {   // a descent (violation)
        if (first_node == nullptr) first_node = prev_node;      // first violation: take the larger (prev)
        second_node = node;                                     // always update: take the smaller (cur)
    }
    prev_node = node;
    inorder(node->right);
}

void recoverTree(TreeNode* root) {
    first_node = second_node = prev_node = nullptr;
    inorder(root);
    swap(first_node->val, second_node->val);  // swap values back
}
```

**Complexity:** Time O(n); Space O(h). A Morris-traversal variant achieves **O(1)** space — see
[Google Interview](../Interview%20Problems/Google.md).

---

## 5. Convert BST to Greater Tree (LC 538) — Medium

Each node's new value = its old value + sum of all keys **greater** than it. Process keys in
**descending** order via a **reverse inorder** (right → node → left) and carry a running sum.

```cpp
#include <bits/stdc++.h>
using namespace std;

int runningSum = 0;

void reverseInorder(TreeNode* node) {
    if (node == nullptr) return;
    reverseInorder(node->right);          // larger keys first
    runningSum += node->val;
    node->val = runningSum;               // node + everything greater
    reverseInorder(node->left);
}

TreeNode* convertBST(TreeNode* root) {
    runningSum = 0;
    reverseInorder(root);
    return root;
}
```

**Complexity:** Time O(n); Space O(h). (LC 1038 is identical.)

---

## 6. Range Sum of BST (LC 938) — Easy

Sum keys in `[low, high]`. Use the BST property to **prune**: if `node->val < low` the entire left
subtree is out of range (skip it); if `node->val > high` the entire right subtree is out of range.

```cpp
#include <bits/stdc++.h>
using namespace std;

int rangeSumBST(TreeNode* root, int low, int high) {
    if (root == nullptr) return 0;
    if (root->val < low)  return rangeSumBST(root->right, low, high);  // prune left
    if (root->val > high) return rangeSumBST(root->left,  low, high);  // prune right
    return root->val
         + rangeSumBST(root->left,  low, high)
         + rangeSumBST(root->right, low, high);
}
```

**Complexity:** Time O(n) worst case, but typically far less due to pruning; Space O(h).

---

## Dry Run — Recover BST (non-adjacent swap)

Valid BST is `1 2 3 4 5`; nodes `2` and `5` got swapped, giving inorder `1 5 3 4 2`.

| Step (visit cur) | prev | descent? (prev>cur) | first | second |
|---|---|---|---|---|
| 1 | — | no | null | null |
| 5 | 1 | no | null | null |
| 3 | 5 | **yes** | 5 (first descent) | 3 |
| 4 | 3 | no | 5 | 3 |
| 2 | 4 | **yes** | 5 (kept) | 2 (updated) |

End: `first = 5`, `second = 2`. Swap their values → inorder becomes `1 2 3 4 5`. Correct.

---

## Summary

| Problem | Technique | Time | Space |
|---|---|---|---|
| Validate BST (98) | (low, high) bounds with `long`, or strict inorder | O(n) | O(h) |
| Kth Smallest (230) | iterative inorder, early stop | O(h + k) | O(h) |
| Two Sum IV (653) | `std::unordered_set`, or inorder + two pointers | O(n) | O(n) |
| Recover BST (99) | inorder finds 2 swapped nodes | O(n) | O(h) / O(1) Morris |
| Greater Tree (538) | reverse inorder + running sum | O(n) | O(h) |
| Range Sum (938) | prune by BST property | O(n) | O(h) |

> **Last Updated:** 2026-06-26
