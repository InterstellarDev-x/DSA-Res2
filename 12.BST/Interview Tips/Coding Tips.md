> **Topic:** [Binary Search Trees](../README.md) · **Tips 1 of 4**

# Coding Tips — BST

Eight tips that turn most BST interview problems into a 5-minute exercise.

```cpp
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int val) : val(val), left(nullptr), right(nullptr) {}
};
```

---

### 1. Inorder = sorted. Reach for it first.
Left → node → right visits keys in **ascending** order. Before writing anything clever, ask "does
this become trivial on the sorted sequence?" — it does for [validate](../Patterns/BST%20Validation%20and%20Inorder.md),
kth-smallest, min-abs-diff, two-sum, and recover.

### 2. Validate with (low, high) bounds, not just a parent compare.
A node can be larger than its parent yet still violate an ancestor's bound. Pass an open interval
down and tighten it:

```cpp
bool valid(TreeNode* n, long lo, long hi) {
    if (n == nullptr) return true;
    if (n->val <= lo || n->val >= hi) return false;
    return valid(n->left, lo, n->val) && valid(n->right, n->val, hi);
}
```

### 3. Use `long` for validation bounds.
A node holding `INT_MIN` or `INT_MAX` breaks `int` bounds at the boundary.
Start from `LONG_MIN` / `LONG_MAX` (or store the previous `TreeNode*` reference instead
of a primitive).

### 4. O(h) compare-and-descend for LCA, floor, ceil, successor.
Never run a full-tree O(n) search when ordering is available. Compare the target against the
current node and descend one side, updating a candidate as you pass.

```cpp
// LCA: descend to the split point
while (cur != nullptr) {
    if (p->val < cur->val && q->val < cur->val)      cur = cur->left;
    else if (p->val > cur->val && q->val > cur->val) cur = cur->right;
    else return cur;
}
```

### 5. Use an explicit stack for *controlled* inorder.
When you need to stop early (kth smallest) or pause/resume (BST Iterator), drive inorder with a
`stack`:

```cpp
#include <bits/stdc++.h>
using namespace std;

stack<TreeNode*> stk;
TreeNode* cur = root;
while (cur != nullptr || !stk.empty()) {
    while (cur != nullptr) { stk.push(cur); cur = cur->left; }
    cur = stk.top(); stk.pop();
    // visit cur — break here for early stop
    cur = cur->right;
}
```

### 6. Reverse inorder for "sum of greater" problems.
Right → node → left visits keys **descending**. Carry a running sum to convert a BST to a greater
tree (LC 538) in one pass.

### 7. Two-children delete = replace with the inorder successor.
The successor (leftmost of the right subtree) is the smallest key larger than the node, so
promoting its value keeps the invariant. Then delete that successor copy from the right subtree.

```cpp
TreeNode* succ = root->right;
while (succ->left != nullptr) succ = succ->left;
root->val = succ->val;
root->right = deleteNode(root->right, succ->val);
```

### 8. Prune with the BST property on range queries.
If `node.val < low`, the whole left subtree is out of range; if `node.val > high`, the whole right
subtree is. Skip them — that is the entire speedup of [Range Sum](../Patterns/BST%20Validation%20and%20Inorder.md).

---

> **Bonus — comparisons:** use `(a > b) - (a < b)` rather than `a - b`; subtraction overflows
> for large/negative values. It rarely matters inside a BST descent (which uses `<`/`>`) but
> matters whenever you write a comparator over node values.

> **Last Updated:** 2026-06-26
