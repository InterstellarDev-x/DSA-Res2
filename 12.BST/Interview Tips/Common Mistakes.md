> **Topic:** [Binary Search Trees](../README.md) · **Tips 2 of 4**

# Common Mistakes — BST

Ten bugs that fail hidden test cases or get flagged in interviews, each with the wrong and the
correct code.

```cpp
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int val) : val(val), left(nullptr), right(nullptr) {}
};
```

---

### 1. Validating only against the direct parent
The deepest descendants must respect *every* ancestor's bound, not just the parent.

```cpp
// WRONG — passes for [5,1,4,null,null,3,6] although 3 < 5
bool bad(TreeNode* n) {
    if (n == nullptr) return true;
    if (n->left  != nullptr && n->left->val  >= n->val) return false;
    if (n->right != nullptr && n->right->val <= n->val) return false;
    return bad(n->left) && bad(n->right);
}
// CORRECT — bounds flow from all ancestors
bool good(TreeNode* n, long lo, long hi) {
    if (n == nullptr) return true;
    if (n->val <= lo || n->val >= hi) return false;
    return good(n->left, lo, n->val) && good(n->right, n->val, hi);
}
```

### 2. Integer overflow in the bounds
```cpp
// WRONG — fails when a node holds INT_MIN / INT_MAX
bool bad(TreeNode* n, int lo, int hi) { ... }
// CORRECT — use long sentinels
bool good(TreeNode* n, long lo, long hi) { ... }   // start with LONG_MIN/LONG_MAX
```

### 3. Kth-smallest off-by-one
```cpp
// WRONG — decrements then compares, returning the (k+1)-th
if (k-- == 0) return cur->val;
// CORRECT — pre-decrement, compare to 0
if (--k == 0) return cur->val;
```

### 4. Delete not handling the two-children case
```cpp
// WRONG — only patches leaves / single children, corrupts two-child nodes
if (root->val == key) return nullptr;
// CORRECT — splice single child, else promote inorder successor
if (root->left == nullptr)  return root->right;
if (root->right == nullptr) return root->left;
TreeNode* s = root->right; while (s->left != nullptr) s = s->left;
root->val = s->val; root->right = deleteNode(root->right, s->val);
```

### 5. Recover BST: ignoring the adjacent-vs-non-adjacent case
```cpp
// WRONG — assumes two descents always exist; breaks for an adjacent swap (one descent)
if (descent) { if (first == nullptr) first = prev; else second = node; }
// CORRECT — first only on the first descent; second on EVERY descent
if (descent) { if (first == nullptr) first = prev; second = node; }
```

### 6. Forgetting "strictly" increasing (duplicates)
```cpp
// WRONG — accepts equal neighbors as valid
if (prev != nullptr && node->val < prev->val) return false;
// CORRECT — a BST inorder must strictly increase
if (prev != nullptr && node->val <= prev->val) return false;
```

### 7. Using the full-tree LCA algorithm on a BST
```cpp
// WRONG — O(n), searches both subtrees, ignores ordering
TreeNode* lca(TreeNode* r, TreeNode* p, TreeNode* q) {
    if (r == nullptr || r == p || r == q) return r;
    TreeNode* l = lca(r->left, p, q), *rr = lca(r->right, p, q);
    return (l != nullptr && rr != nullptr) ? r : (l != nullptr ? l : rr);
}
// CORRECT — O(h) descent to the split point
while (cur != nullptr) {
    if (p->val < cur->val && q->val < cur->val)      cur = cur->left;
    else if (p->val > cur->val && q->val > cur->val) cur = cur->right;
    else return cur;
}
```

### 8. BST Iterator not pushing the leftmost spine
```cpp
// WRONG — pushes only the node, so next() can skip its left descendants
void push(TreeNode* n) { if (n != nullptr) st.push(n); }
// CORRECT — push the node and ALL of its left children
void pushLeftmost(TreeNode* n) { while (n != nullptr) { st.push(n); n = n->left; } }
```

### 9. Range Sum not pruning
```cpp
// WRONG — visits every node, ignoring the BST property (O(n) always)
int sum(TreeNode* n, int lo, int hi) {
    if (n == nullptr) return 0;
    int s = (n->val >= lo && n->val <= hi) ? n->val : 0;
    return s + sum(n->left, lo, hi) + sum(n->right, lo, hi);
}
// CORRECT — skip whole subtrees out of range
int sum(TreeNode* n, int lo, int hi) {
    if (n == nullptr) return 0;
    if (n->val < lo)  return sum(n->right, lo, hi);
    if (n->val > hi)  return sum(n->left, lo, hi);
    return n->val + sum(n->left, lo, hi) + sum(n->right, lo, hi);
}
```

### 10. Greater Tree: wrong traversal direction
```cpp
// WRONG — normal inorder (ascending) accumulates SMALLER keys
inorder(n->left); sum += n->val; n->val = sum; inorder(n->right);
// CORRECT — reverse inorder (descending) accumulates GREATER keys
reverse(n->right); sum += n->val; n->val = sum; reverse(n->left);
```

---

> **Last Updated:** 2026-06-26
