# Design: Binary Search Tree (BST)

> **Topic:** [Binary Search](../README.md) · **Category:** Design Data Structure
> **Difficulty:** Medium · **Tags:** `BST` `design` `insertion` `search` `deletion`

---

## Table of Contents

1. [Problem Statement](#problem-statement)
2. [Interview Expectations](#interview-expectations)
3. [Core Operations](#core-operations)
4. [Rust Implementation](#rust-implementation)
5. [Complexity Analysis](#complexity-analysis)
6. [Edge Cases](#edge-cases)
7. [Similar Problems](#similar-problems)
8. [Follow-up Questions](#follow-up-questions)
9. [Company Tags](#company-tags)

---

## Problem Statement

Design and implement a Binary Search Tree supporting:

- `fn insert(&mut self, val: i32)` — insert a value
- `fn search(&self, val: i32) -> bool` — return true if value exists
- `fn delete_val(&mut self, val: i32)` — remove a value
- `fn floor(&self, val: i32) -> i32` — largest value ≤ val
- `fn ceil(&self, val: i32) -> i32` — smallest value ≥ val
- `fn kth_smallest(&self, k: usize) -> i32` — kth smallest element

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

## Rust Implementation

```rust
#[derive(Debug)]
pub struct TreeNode {
    pub val: i32,
    pub size: usize, // size for kth-smallest augmentation
    pub left: Option<Box<TreeNode>>,
    pub right: Option<Box<TreeNode>>,
}

impl TreeNode {
    pub fn new(val: i32) -> Self {
        TreeNode { val, size: 1, left: None, right: None }
    }
}

pub struct BST {
    root: Option<Box<TreeNode>>,
}

fn node_size(node: &Option<Box<TreeNode>>) -> usize {
    node.as_ref().map_or(0, |n| n.size)
}

// ---------- INSERT ----------
fn insert_node(node: Option<Box<TreeNode>>, val: i32) -> Box<TreeNode> {
    match node {
        None => Box::new(TreeNode::new(val)),
        Some(mut n) => {
            if val < n.val {
                n.left = Some(insert_node(n.left, val));
            } else if val > n.val {
                n.right = Some(insert_node(n.right, val));
            }
            // if val == n.val: duplicate, ignore (or count)
            n.size = 1 + node_size(&n.left) + node_size(&n.right);
            n
        }
    }
}

// ---------- SEARCH ----------
fn search_node(node: &Option<Box<TreeNode>>, val: i32) -> bool {
    match node {
        None => false,
        Some(n) => {
            if val == n.val { true }
            else if val < n.val { search_node(&n.left, val) }
            else { search_node(&n.right, val) }
        }
    }
}

fn leftmost_val(node: &Box<TreeNode>) -> i32 {
    let mut curr = node;
    while let Some(ref left) = curr.left {
        curr = left;
    }
    curr.val
}

// ---------- DELETE ----------
fn delete_node(node: Option<Box<TreeNode>>, val: i32) -> Option<Box<TreeNode>> {
    match node {
        None => None,
        Some(mut n) => {
            if val < n.val {
                n.left = delete_node(n.left, val);
            } else if val > n.val {
                n.right = delete_node(n.right, val);
            } else {
                // Case 1 & 2: leaf or one child
                if n.left.is_none() { return n.right; }
                if n.right.is_none() { return n.left; }
                // Case 3: two children — replace with in-order successor
                let successor_val = leftmost_val(n.right.as_ref().unwrap());
                n.val = successor_val;
                n.right = delete_node(n.right, successor_val);
            }
            n.size = 1 + node_size(&n.left) + node_size(&n.right);
            Some(n)
        }
    }
}

// ---------- FLOOR ----------
fn floor_node(node: &Option<Box<TreeNode>>, val: i32) -> Option<i32> {
    match node {
        None => None,
        Some(n) => {
            if n.val == val { Some(n.val) }
            else if n.val > val { floor_node(&n.left, val) }
            else { floor_node(&n.right, val).or(Some(n.val)) }
        }
    }
}

// ---------- CEIL ----------
fn ceil_node(node: &Option<Box<TreeNode>>, val: i32) -> Option<i32> {
    match node {
        None => None,
        Some(n) => {
            if n.val == val { Some(n.val) }
            else if n.val < val { ceil_node(&n.right, val) }
            else { ceil_node(&n.left, val).or(Some(n.val)) }
        }
    }
}

// ---------- KTH SMALLEST (augmented) ----------
fn kth_smallest_node(node: &Option<Box<TreeNode>>, k: usize) -> i32 {
    let n = node.as_ref().unwrap();
    let left_size = node_size(&n.left);
    if k == left_size + 1 { n.val }
    else if k <= left_size { kth_smallest_node(&n.left, k) }
    else { kth_smallest_node(&n.right, k - left_size - 1) }
}

impl BST {
    pub fn new() -> Self {
        BST { root: None }
    }

    // ---------- INSERT ----------
    pub fn insert(&mut self, val: i32) {
        self.root = Some(insert_node(self.root.take(), val));
    }

    // ---------- SEARCH ----------
    pub fn search(&self, val: i32) -> bool {
        search_node(&self.root, val)
    }

    // ---------- DELETE ----------
    pub fn delete_val(&mut self, val: i32) {
        self.root = delete_node(self.root.take(), val);
    }

    // ---------- FLOOR ----------
    pub fn floor(&self, val: i32) -> i32 {
        floor_node(&self.root, val).unwrap_or(-1)
    }

    // ---------- CEIL ----------
    pub fn ceil(&self, val: i32) -> i32 {
        ceil_node(&self.root, val).unwrap_or(-1)
    }

    // ---------- KTH SMALLEST (augmented) ----------
    pub fn kth_smallest(&self, k: usize) -> i32 {
        kth_smallest_node(&self.root, k)
    }
}
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
- `kth_smallest(0)` or `k > size` → panic or return sentinel
- Inserting duplicate values → define behaviour (ignore, count, allow)
- Floor/Ceil on empty tree → return -1 or use `Option`
- Tree with a single node → deletion returns `None` root

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

1. **BST degrades to O(n) — how do you fix it?** → Self-balancing: AVL (strict height balance), Red-Black (relaxed, used in `BTreeMap`)
2. **`BTreeMap` vs BST?** → `BTreeMap` is a B-tree. `get/insert/remove` all O(log n) guaranteed.
3. **How does augmentation with `size` help?** → kth smallest in O(log n) vs O(n) with in-order traversal
4. **Thread-safe BST?** → Use concurrent data structures or protect with `std::sync::Mutex` for thread-safe sorted access

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
