> **Topic:** [Binary Search Trees](../README.md) · **Pattern 3 of 4**

# BST Construction

**Core idea:** rebuilding a BST is about **identifying the root and partitioning the rest** into
the left subtree (smaller keys) and right subtree (larger keys), then recursing. From a *sorted
array* the balanced root is the **middle** element. From a *preorder traversal* the **first**
element is the root and an **upper bound** cleanly separates the two subtrees in a single O(n)
pass.

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

> **Recognition signals:** "build a height-balanced BST", "from a sorted array", "from preorder",
> "rebalance an unbalanced BST", "minimum height".

---

## 1. Convert Sorted Array to BST (LC 108) — Easy

The array is already sorted (= an inorder sequence). Choosing the **middle** element as the root
at every level guarantees a **height-balanced** tree: left half builds the left subtree, right half
the right subtree.

```rust
fn sorted_array_to_bst(nums: &[i32]) -> Option<Box<TreeNode>> {
    build(nums, 0, nums.len() as i32 - 1)
}

fn build(nums: &[i32], lo: i32, hi: i32) -> Option<Box<TreeNode>> {
    if lo > hi { return None; }
    let mid = lo + (hi - lo) / 2;            // middle => balanced; avoids overflow
    let mut root = Box::new(TreeNode::new(nums[mid as usize]));
    root.left  = build(nums, lo, mid - 1);
    root.right = build(nums, mid + 1, hi);
    Some(root)
}
```

**Complexity:** Time O(n); Space O(log n) recursion (balanced). Picking the lower or upper middle
both yield valid balanced trees.

---

## 2. Construct BST from Preorder Traversal (LC 1008) — Medium

In a preorder, the **first** value is the root. Every subsequent value smaller than the root
belongs to the left subtree; the first value larger starts the right subtree. The elegant **O(n)**
solution uses a single shared index and an **upper bound** that says "consume values that still
belong to the current subtree."

```rust
fn bst_from_preorder(preorder: &[i32]) -> Option<Box<TreeNode>> {
    let mut idx = 0;
    build(preorder, &mut idx, i32::MAX)
}

fn build(pre: &[i32], idx: &mut usize, bound: i32) -> Option<Box<TreeNode>> {
    if *idx == pre.len() || pre[*idx] > bound { return None; }  // nothing more fits here
    let val = pre[*idx];
    *idx += 1;
    let mut root = Box::new(TreeNode::new(val));
    root.left  = build(pre, idx, val);    // left subtree: values < val
    root.right = build(pre, idx, bound);  // right subtree: values < inherited bound
    Some(root)
}
```

Each element is consumed exactly once → **O(n)**.

A simpler-to-explain (but O(n²) worst case) recursion uses explicit lo/hi value bounds:

```rust
fn bst_from_preorder_bounds(pre: &[i32]) -> Option<Box<TreeNode>> {
    let mut p = 0;
    helper(pre, &mut p, i32::MIN, i32::MAX)
}

fn helper(pre: &[i32], p: &mut usize, lower: i32, upper: i32) -> Option<Box<TreeNode>> {
    if *p == pre.len() { return None; }
    let v = pre[*p];
    if v < lower || v > upper { return None; }
    *p += 1;
    let mut node = Box::new(TreeNode::new(v));
    node.left  = helper(pre, p, lower, v);
    node.right = helper(pre, p, v, upper);
    Some(node)
}
```

**Complexity:** upper-bound version O(n) time, O(h) space.

---

## 3. Balance a Binary Search Tree (LC 1382) — Medium

Two-step composition of the previous ideas:

1. **Inorder traversal → sorted array** (a BST's inorder is sorted, so no actual sorting needed).
2. **Sorted array → height-balanced BST** using the LC 108 mid-element technique.

```rust
fn balance_bst(root: Option<Box<TreeNode>>) -> Option<Box<TreeNode>> {
    let mut sorted = Vec::new();
    inorder(&root, &mut sorted);
    build(&sorted, 0, sorted.len() as i32 - 1)
}

fn inorder(node: &Option<Box<TreeNode>>, out: &mut Vec<i32>) {
    if let Some(n) = node {
        inorder(&n.left, out);
        out.push(n.val);
        inorder(&n.right, out);
    }
}

fn build(a: &[i32], lo: i32, hi: i32) -> Option<Box<TreeNode>> {
    if lo > hi { return None; }
    let mid = lo + (hi - lo) / 2;
    let mut root = Box::new(TreeNode::new(a[mid as usize]));
    root.left  = build(a, lo, mid - 1);
    root.right = build(a, mid + 1, hi);
    Some(root)
}
```

**Complexity:** Time O(n); Space O(n) for the list + O(log n) recursion.

---

## Dry Run — Sorted Array `[1,2,3,4,5,6,7]` → Balanced BST

`build(0,6)`: `mid = 0 + (6-0)/2 = 3` → root `4`.
- left `build(0,2)`: `mid = 1` → `2`; left `build(0,0)` → `1`; right `build(2,2)` → `3`.
- right `build(4,6)`: `mid = 5` → `6`; left `build(4,4)` → `5`; right `build(6,6)` → `7`.

```
        4
       / \
      2   6
     / \ / \
    1  3 5  7
```

Perfectly balanced, height = 2 = ⌊log₂ 7⌋. Inorder reproduces `1 2 3 4 5 6 7`.

---

## Summary

| Problem | Technique | Time | Space |
|---|---|---|---|
| Sorted Array → BST (108) | mid element as root, recurse halves | O(n) | O(log n) |
| Construct from Preorder (1008) | first = root, upper-bound consume | O(n) | O(h) |
| Balance a BST (1382) | inorder → sorted array → LC 108 | O(n) | O(n) |

> See also the binary-tree [construction patterns](../../11.Binary_Trees/Patterns/Tree%20Construction.md)
> (from preorder+inorder) for trees without the BST ordering shortcut.

> **Last Updated:** 2026-06-26
