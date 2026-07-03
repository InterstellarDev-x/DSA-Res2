> **Topic:** [Binary Search Trees](../README.md) · **Pattern 1 of 4**

# BST Operations — Search, Insert, Delete, Min-Abs-Diff

**Core idea:** every BST operation is a **compare-and-descend** walk. At each node, compare the
target with `node.val` and move *left* (target smaller) or *right* (target larger). This visits
one node per level → **O(h)** time, where `h = O(log n)` for a balanced tree and `O(n)` for a
skewed one. No extra structure is needed; the ordering *is* the index.

```cpp
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int val) : val(val), left(nullptr), right(nullptr) {}
};
```

> **Recognition signals:** "find/insert/remove a key", "the tree is a BST", "in O(h)/O(log n)",
> "smallest difference between any two nodes", "successor of the deleted node".

---

## 1. Search in a BST (LC 700) — Easy

Compare and descend. Iterative is the cleanest and uses **O(1)** space.

```cpp
TreeNode* searchBST(TreeNode* root, int val) {
    TreeNode* cur = root;
    while (cur != nullptr && cur->val != val) {
        cur = val < cur->val ? cur->left : cur->right;
    }
    return cur;   // nullptr if not found
}
```

Recursive form (O(h) stack):

```cpp
TreeNode* searchBSTrec(TreeNode* root, int val) {
    if (root == nullptr || root->val == val) return root;
    return val < root->val ? searchBSTrec(root->left, val)
                           : searchBSTrec(root->right, val);
}
```

**Complexity:** Time O(h); Space O(1) iterative, O(h) recursive.

---

## 2. Insert into a BST (LC 701) — Medium

Walk down as if searching; when you fall off the tree (hit `null`), that empty slot is exactly
where the key belongs. The recursive version "re-attaches" subtrees on the way back up.

```cpp
TreeNode* insertIntoBST(TreeNode* root, int val) {
    if (root == nullptr) return new TreeNode(val);   // attach at the null leaf
    if (val < root->val) root->left  = insertIntoBST(root->left, val);
    else                 root->right = insertIntoBST(root->right, val);
    return root;
}
```

Iterative (track the parent, no recursion stack):

```cpp
TreeNode* insertIterative(TreeNode* root, int val) {
    if (root == nullptr) return new TreeNode(val);
    TreeNode* cur = root;
    while (true) {
        if (val < cur->val) {
            if (cur->left == nullptr) { cur->left = new TreeNode(val); break; }
            cur = cur->left;
        } else {
            if (cur->right == nullptr) { cur->right = new TreeNode(val); break; }
            cur = cur->right;
        }
    }
    return root;
}
```

**Complexity:** Time O(h); Space O(h) recursive / O(1) iterative. Problem guarantees `val` is new.

---

## 3. Delete Node in a BST (LC 450) — Medium

The hardest core operation. First *find* the node (compare-and-descend), then handle **three
cases** for the node to remove:

| Case | Action |
|---|---|
| **No child** (leaf) | Return `nullptr` to the parent. |
| **One child** | Return the non-null child to the parent (splice it in). |
| **Two children** | Replace the node's value with its **inorder successor** (the *leftmost* node of the right subtree), then recursively delete that successor from the right subtree. |

The inorder successor is the smallest key larger than the node — it preserves the BST invariant
when promoted.

```cpp
TreeNode* deleteNode(TreeNode* root, int key) {
    if (root == nullptr) return nullptr;
    if (key < root->val) {
        root->left = deleteNode(root->left, key);
    } else if (key > root->val) {
        root->right = deleteNode(root->right, key);
    } else {
        // found the node to delete
        if (root->left == nullptr)  return root->right;   // 0 or 1 child (right)
        if (root->right == nullptr) return root->left;    // 1 child (left)
        // two children: replace with inorder successor = leftmost of right subtree
        TreeNode* succ = root->right;
        while (succ->left != nullptr) succ = succ->left;
        root->val = succ->val;
        root->right = deleteNode(root->right, succ->val); // delete the successor copy
    }
    return root;
}
```

**Complexity:** Time O(h) (the recursive successor deletion is also bounded by the height);
Space O(h).

> Symmetric alternative: replace with the inorder **predecessor** (rightmost of the *left*
> subtree). Either is correct; pick one and be consistent.

---

## 4. Minimum Absolute Difference in BST (LC 530) — Easy

Since inorder yields a **sorted** sequence, the minimum absolute difference can only occur between
**adjacent** values in that order. Walk inorder, keep the previously-visited value in `prev_val`, and
minimize `cur - prev_val`.

```cpp
#include <bits/stdc++.h>
using namespace std;

int prev_val;
int minDiff;

void inorder(TreeNode* node) {
    if (node == nullptr) return;
    inorder(node->left);
    if (prev_val != -1) minDiff = min(minDiff, node->val - prev_val); // sorted ⇒ node->val ≥ prev_val
    prev_val = node->val;
    inorder(node->right);
}

int getMinimumDifference(TreeNode* root) {
    prev_val = -1;
    minDiff = INT_MAX;
    inorder(root);
    return minDiff;
}
```

**Complexity:** Time O(n) (must touch every node); Space O(h). Same idea solves LC 783.

---

## Dry Run — Delete 5 from a two-children node

Tree (key to delete = `5`, which has two children):

```
        5
       / \
      3   7
     / \   \
    2   4   8
```

1. `deleteNode(root=5, key=5)` → match. Both children non-null → **two-children case**.
2. Find inorder successor: start at `root->right = 7`, walk left while possible → `7` has no left
   child, so `succ = 7`.
3. Copy `succ->val (7)` into the node: root value becomes `7`.
4. Delete the successor copy from the right subtree: `deleteNode(node 7, key 7)`. That `7` has no
   left child → return its right child `8`.

Result (still a valid BST, sorted inorder `2 3 4 7 8`):

```
        7
       / \
      3   8
     / \
    2   4
```

---

## Summary

| Operation | Technique | Time | Space |
|---|---|---|---|
| Search (700) | compare-and-descend | O(h) | O(1) iter |
| Insert (701) | attach at null leaf | O(h) | O(1) iter |
| Delete (450) | 3 cases; two-children → successor | O(h) | O(h) |
| Min Abs Diff (530) | inorder + track `prev_val` | O(n) | O(h) |

> **Last Updated:** 2026-06-26
