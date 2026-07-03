# Design: Binary Search Tree (BST)

> **Topic:** [Binary Search](../README.md) · **Category:** Design Data Structure
> **Difficulty:** Medium · **Tags:** `BST` `design` `insertion` `search` `deletion`

---

## Table of Contents

1. [Problem Statement](#problem-statement)
2. [Interview Expectations](#interview-expectations)
3. [Core Operations](#core-operations)
4. [C++ Implementation](#c-implementation)
5. [Complexity Analysis](#complexity-analysis)
6. [Edge Cases](#edge-cases)
7. [Similar Problems](#similar-problems)
8. [Follow-up Questions](#follow-up-questions)
9. [Company Tags](#company-tags)

---

## Problem Statement

Design and implement a Binary Search Tree supporting:

- `void insert(int val)` — insert a value
- `bool search(int val)` — return true if value exists
- `void deleteVal(int val)` — remove a value
- `int floor(int val)` — largest value ≤ val
- `int ceil(int val)` — smallest value ≥ val
- `int kthSmallest(int k)` — kth smallest element

---

## Interview Expectations

| Expectation | Detail |
|-------------|--------|
| Know three deletion cases | Leaf, one child, two children |
| Successor/Predecessor | Used in deletion of two-child node |
| Balanced vs unbalanced | Know when BST degrades to O(n) |
| In-order traversal = sorted | Key BST property |
| Augmentation | How to store subtree size for kth smallest |

---

## Core Operations

### BST Property

```
For every node N:
  - All values in left subtree < N.val
  - All values in right subtree > N.val
```

### Deletion — Three Cases

| Case | Action |
|------|--------|
| Node is a leaf | Simply remove it |
| Node has one child | Replace node with its child |
| Node has two children | Replace with in-order successor (leftmost of right subtree), then delete successor |

---

## C++ Implementation

```cpp
#include <bits/stdc++.h>
using namespace std;

struct TreeNode {
    int val, size; // size for kth-smallest augmentation
    TreeNode* left;
    TreeNode* right;
    TreeNode(int val) : val(val), size(1), left(nullptr), right(nullptr) {}
};

class BST {
private:
    TreeNode* root;

    // ---------- INSERT ----------
    TreeNode* insert(TreeNode* node, int val) {
        if (node == nullptr) return new TreeNode(val);
        if (val < node->val)       node->left  = insert(node->left, val);
        else if (val > node->val)  node->right = insert(node->right, val);
        // if val == node->val: duplicate, ignore (or count)
        node->size = 1 + size(node->left) + size(node->right);
        return node;
    }

    // ---------- SEARCH ----------
    bool searchNode(TreeNode* node, int val) {
        if (node == nullptr) return false;
        if (val == node->val) return true;
        return val < node->val ? searchNode(node->left, val) : searchNode(node->right, val);
    }

    // ---------- DELETE ----------
    TreeNode* deleteNode(TreeNode* node, int val) {
        if (node == nullptr) return nullptr;
        if (val < node->val) {
            node->left = deleteNode(node->left, val);
        } else if (val > node->val) {
            node->right = deleteNode(node->right, val);
        } else {
            // Case 1 & 2: leaf or one child
            if (node->left == nullptr)  return node->right;
            if (node->right == nullptr) return node->left;
            // Case 3: two children — replace with in-order successor
            TreeNode* successor = leftmost(node->right);
            node->val = successor->val;
            node->right = deleteNode(node->right, successor->val);
        }
        node->size = 1 + size(node->left) + size(node->right);
        return node;
    }

    TreeNode* leftmost(TreeNode* node) {
        while (node->left != nullptr) node = node->left;
        return node;
    }

    // ---------- FLOOR ----------
    TreeNode* floorNode(TreeNode* node, int val) {
        if (node == nullptr) return nullptr;
        if (node->val == val) return node;
        if (node->val > val)  return floorNode(node->left, val);
        TreeNode* right = floorNode(node->right, val);
        return right != nullptr ? right : node;
    }

    // ---------- CEIL ----------
    TreeNode* ceilNode(TreeNode* node, int val) {
        if (node == nullptr) return nullptr;
        if (node->val == val) return node;
        if (node->val < val)  return ceilNode(node->right, val);
        TreeNode* left = ceilNode(node->left, val);
        return left != nullptr ? left : node;
    }

    // ---------- KTH SMALLEST (augmented) ----------
    int kthSmallestNode(TreeNode* node, int k) {
        int leftSize = size(node->left);
        if (k == leftSize + 1) return node->val;
        if (k <= leftSize)     return kthSmallestNode(node->left, k);
        return kthSmallestNode(node->right, k - leftSize - 1);
    }

    int size(TreeNode* node) { return node == nullptr ? 0 : node->size; }

public:
    BST() : root(nullptr) {}

    // ---------- INSERT ----------
    void insert(int val) { root = insert(root, val); }

    // ---------- SEARCH ----------
    bool search(int val) { return searchNode(root, val); }

    // ---------- DELETE ----------
    void deleteVal(int val) { root = deleteNode(root, val); }

    // ---------- FLOOR ----------
    int floor(int val) {
        TreeNode* res = floorNode(root, val);
        return res == nullptr ? -1 : res->val;
    }

    // ---------- CEIL ----------
    int ceil(int val) {
        TreeNode* res = ceilNode(root, val);
        return res == nullptr ? -1 : res->val;
    }

    // ---------- KTH SMALLEST (augmented) ----------
    int kthSmallest(int k) { return kthSmallestNode(root, k); }
};
```

---

## Complexity Analysis

| Operation | Average (balanced) | Worst (skewed/sorted input) |
|-----------|-------------------|------------------------------|
| Insert | O(log n) | O(n) |
| Search | O(log n) | O(n) |
| Delete | O(log n) | O(n) |
| Floor / Ceil | O(log n) | O(n) |
| Kth Smallest (augmented) | O(log n) | O(n) |
| Space | O(n) | O(n) |

> **Why worst case O(n)?** Inserting 1, 2, 3, …, n in sorted order creates a right-skewed linked list. Self-balancing trees (AVL, Red-Black) fix this.

---

## Edge Cases

- Deleting a node that doesn't exist → do nothing (no crash)
- `kthSmallest(0)` or `k > size` → throw `std::invalid_argument`
- Inserting duplicate values → define behaviour (ignore, count, allow)
- Floor/Ceil on empty tree → return -1 or throw
- Tree with a single node → deletion returns null root

---

## Similar Problems

| Problem | Link |
|---------|------|
| [Validate Binary Search Tree](https://leetcode.com/problems/validate-binary-search-tree/) | LC 98 |
| [Kth Smallest Element in BST](https://leetcode.com/problems/kth-smallest-element-in-a-bst/) | LC 230 |
| [Insert into BST](https://leetcode.com/problems/insert-into-a-binary-search-tree/) | LC 701 |
| [Delete Node in BST](https://leetcode.com/problems/delete-node-in-a-bst/) | LC 450 |
| [BST to Greater Sum Tree](https://leetcode.com/problems/binary-search-tree-to-greater-sum-tree/) | LC 1038 |

---

## Follow-up Questions

1. **BST degrades to O(n) — how do you fix it?** → Self-balancing: AVL (strict height balance), Red-Black (relaxed, used in `std::map`)
2. **`std::map` vs BST?** → `std::map` is a Red-Black tree. `find/insert/erase` all O(log n) guaranteed.
3. **How does augmentation with `size` help?** → kth smallest in O(log n) vs O(n) with in-order traversal
4. **Thread-safe BST?** → Use concurrent data structures or protect with `std::mutex` for thread-safe sorted access

---

## Company Tags

`Amazon` `Google` `Microsoft` `Adobe` `Goldman Sachs`

---

## Navigation

| Related |
|---------|
| [Binary Search README](../README.md) |
| [Classic Binary Search Pattern](../Patterns/Classic%20Binary%20Search.md) |
| [Lower and Upper Bound Pattern](../Patterns/Lower%20and%20Upper%20Bound.md) |

> **Last Updated:** 2026-06-26
