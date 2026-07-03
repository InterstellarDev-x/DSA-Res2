> **Topic:** [Binary Search Trees](../README.md) · **Pattern 2 of 4**

# BST Validation & Inorder

**Core idea:** the **inorder traversal of a BST is strictly sorted** (left → node → right). A huge
class of problems reduces to "process the keys in sorted order": validate, find the k-th smallest,
detect/repair swapped nodes, accumulate suffix sums (greater tree), and answer range queries. The
two workhorse tools are (a) the **(low, high) bounds** that an ancestor imposes on a subtree, and
(b) an explicit **stack-driven inorder** that can stop early.

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

> **Recognition signals:** "is this a valid BST", "k-th smallest/largest", "two nodes swapped /
> recover", "add all greater keys", "sum of keys in [low, high]", "find a pair summing to k".

---

## 1. Validate Binary Search Tree (LC 98) — Medium

The classic trap: it is **not enough** to check `left < node < right` against the direct parent.
*Every* node must lie within the bounds set by **all** its ancestors. Pass an open interval
`(low, high)` downward and tighten it as you descend.

Use `i64` for the bounds so a node with value `i32::MIN` or `i32::MAX` does not
produce a false negative at the boundary.

```rust
fn valid(node: &Option<Box<TreeNode>>, low: i64, high: i64) -> bool {
    match node {
        None => true,
        Some(n) => {
            if (n.val as i64) <= low || (n.val as i64) >= high {
                return false;  // strict: no duplicates allowed
            }
            valid(&n.left, low, n.val as i64) && valid(&n.right, n.val as i64, high)
        }
    }
}

fn is_valid_bst(root: &Option<Box<TreeNode>>) -> bool {
    valid(root, i64::MIN, i64::MAX)
}
```

**Alternative — strictly-increasing inorder.** Equivalent and avoids the `i64` choice by storing
the previous node reference:

```rust
fn inorder(node: &Option<Box<TreeNode>>, prev: &mut Option<i32>) -> bool {
    match node {
        None => true,
        Some(n) => {
            if !inorder(&n.left, prev) { return false; }
            if let Some(p) = *prev {
                if n.val <= p { return false; }  // must be strictly increasing
            }
            *prev = Some(n.val);
            inorder(&n.right, prev)
        }
    }
}

fn is_valid_bst_inorder(root: &Option<Box<TreeNode>>) -> bool {
    let mut prev = None;
    inorder(root, &mut prev)
}
```

**Complexity:** Time O(n); Space O(h).

---

## 2. Kth Smallest Element in a BST (LC 230) — Medium

Inorder visits keys in ascending order, so the k-th visited node is the answer. The **iterative
stack** version is preferred: it stops as soon as the count reaches `k` (early exit) instead of
traversing the whole tree.

```rust
fn kth_smallest(root: &Option<Box<TreeNode>>, k: i32) -> i32 {
    let mut st: Vec<&TreeNode> = Vec::new();
    let mut cur: Option<&TreeNode> = root.as_deref();
    let mut k = k;
    while cur.is_some() || !st.is_empty() {
        while let Some(node) = cur {  // dive to the leftmost
            st.push(node);
            cur = node.left.as_deref();
        }
        let node = st.pop().unwrap();  // visit in sorted order
        k -= 1;
        if k == 0 { return node.val; }  // k-th smallest reached → stop
        cur = node.right.as_deref();
    }
    -1  // k out of range
}
```

**Complexity:** Time O(h + k) (dive once, then pop k); Space O(h).

> **Follow-up (frequent modifications):** augment each node with `leftCount` (size of its left
> subtree) so a single k-th query becomes O(h). See [Google Interview](../Interview%20Problems/Google.md).

---

## 3. Two Sum IV - Input is a BST (LC 653) — Easy

Two clean approaches.

**(a) `HashSet` during any traversal** — O(n) time, O(n) space, does not exploit ordering:

```rust
use std::collections::HashSet;

fn dfs(node: &Option<Box<TreeNode>>, k: i32, seen: &mut HashSet<i32>) -> bool {
    match node {
        None => false,
        Some(n) => {
            if seen.contains(&(k - n.val)) { return true; }
            seen.insert(n.val);
            dfs(&n.left, k, seen) || dfs(&n.right, k, seen)
        }
    }
}

fn find_target(root: &Option<Box<TreeNode>>, k: i32) -> bool {
    let mut seen = HashSet::new();
    dfs(root, k, &mut seen)
}
```

**(b) Inorder to sorted list + two pointers** — exploits sortedness; O(n) time, O(n) for the list:

```rust
fn inorder_collect(n: &Option<Box<TreeNode>>, out: &mut Vec<i32>) {
    if let Some(node) = n {
        inorder_collect(&node.left, out);
        out.push(node.val);
        inorder_collect(&node.right, out);
    }
}

fn find_target_two_ptr(root: &Option<Box<TreeNode>>, k: i32) -> bool {
    let mut sorted = Vec::new();
    inorder_collect(root, &mut sorted);
    let mut lo = 0usize;
    let mut hi = sorted.len().saturating_sub(1);
    while lo < hi {
        let sum = sorted[lo] + sorted[hi];
        if sum == k { return true; }
        if sum < k { lo += 1; } else { hi -= 1; }
    }
    false
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

```rust
fn inorder_find_swapped(
    node: &Option<Box<TreeNode>>,
    prev_val: &mut i64,
    first_val: &mut Option<i32>,
    second_val: &mut Option<i32>,
) {
    if let Some(n) = node {
        inorder_find_swapped(&n.left, prev_val, first_val, second_val);
        if *prev_val != i64::MIN && *prev_val > n.val as i64 {  // a descent (violation)
            if first_val.is_none() { *first_val = Some(*prev_val as i32); }  // first violation: take the larger (prev)
            *second_val = Some(n.val);                                        // always update: take the smaller (cur)
        }
        *prev_val = n.val as i64;
        inorder_find_swapped(&n.right, prev_val, first_val, second_val);
    }
}

fn inorder_fix_swapped(node: &mut Option<Box<TreeNode>>, first_val: i32, second_val: i32) {
    if let Some(n) = node.as_mut() {
        if n.val == first_val { n.val = second_val; }
        else if n.val == second_val { n.val = first_val; }
        inorder_fix_swapped(&mut n.left, first_val, second_val);
        inorder_fix_swapped(&mut n.right, first_val, second_val);
    }
}

fn recover_tree(root: &mut Option<Box<TreeNode>>) {
    let mut prev_val = i64::MIN;
    let mut first_val = None;
    let mut second_val = None;
    inorder_find_swapped(root, &mut prev_val, &mut first_val, &mut second_val);
    if let (Some(fv), Some(sv)) = (first_val, second_val) {
        inorder_fix_swapped(root, fv, sv);  // swap values back
    }
}
```

**Complexity:** Time O(n); Space O(h). A Morris-traversal variant achieves **O(1)** space — see
[Google Interview](../Interview%20Problems/Google.md).

---

## 5. Convert BST to Greater Tree (LC 538) — Medium

Each node's new value = its old value + sum of all keys **greater** than it. Process keys in
**descending** order via a **reverse inorder** (right → node → left) and carry a running sum.

```rust
fn reverse_inorder(node: &mut Option<Box<TreeNode>>, running_sum: &mut i32) {
    if let Some(n) = node.as_mut() {
        reverse_inorder(&mut n.right, running_sum);  // larger keys first
        *running_sum += n.val;
        n.val = *running_sum;                         // node + everything greater
        reverse_inorder(&mut n.left, running_sum);
    }
}

fn convert_bst(root: &mut Option<Box<TreeNode>>) {
    let mut running_sum = 0;
    reverse_inorder(root, &mut running_sum);
}
```

**Complexity:** Time O(n); Space O(h). (LC 1038 is identical.)

---

## 6. Range Sum of BST (LC 938) — Easy

Sum keys in `[low, high]`. Use the BST property to **prune**: if `node->val < low` the entire left
subtree is out of range (skip it); if `node->val > high` the entire right subtree is out of range.

```rust
fn range_sum_bst(root: &Option<Box<TreeNode>>, low: i32, high: i32) -> i32 {
    match root {
        None => 0,
        Some(n) => {
            if n.val < low  { return range_sum_bst(&n.right, low, high); }  // prune left
            if n.val > high { return range_sum_bst(&n.left,  low, high); }  // prune right
            n.val
                + range_sum_bst(&n.left,  low, high)
                + range_sum_bst(&n.right, low, high)
        }
    }
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
| Validate BST (98) | (low, high) bounds with `i64`, or strict inorder | O(n) | O(h) |
| Kth Smallest (230) | iterative inorder, early stop | O(h + k) | O(h) |
| Two Sum IV (653) | `HashSet`, or inorder + two pointers | O(n) | O(n) |
| Recover BST (99) | inorder finds 2 swapped nodes | O(n) | O(h) / O(1) Morris |
| Greater Tree (538) | reverse inorder + running sum | O(n) | O(h) |
| Range Sum (938) | prune by BST property | O(n) | O(h) |

> **Last Updated:** 2026-06-26
