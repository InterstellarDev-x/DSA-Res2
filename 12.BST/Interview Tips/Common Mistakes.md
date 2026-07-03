> **Topic:** [Binary Search Trees](../README.md) · **Tips 2 of 4**

# Common Mistakes — BST

Ten bugs that fail hidden test cases or get flagged in interviews, each with the wrong and the
correct code.

```rust
#[derive(Debug, Clone, PartialEq)]
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

---

### 1. Validating only against the direct parent
The deepest descendants must respect *every* ancestor's bound, not just the parent.

```rust
// WRONG — passes for [5,1,4,null,null,3,6] although 3 < 5
fn bad(n: &Option<Box<TreeNode>>) -> bool {
    match n {
        None => true,
        Some(node) => {
            if let Some(left) = &node.left {
                if left.val >= node.val { return false; }
            }
            if let Some(right) = &node.right {
                if right.val <= node.val { return false; }
            }
            bad(&node.left) && bad(&node.right)
        }
    }
}
// CORRECT — bounds flow from all ancestors
fn good(n: &Option<Box<TreeNode>>, lo: i64, hi: i64) -> bool {
    match n {
        None => true,
        Some(node) => {
            let v = node.val as i64;
            if v <= lo || v >= hi { return false; }
            good(&node.left, lo, v) && good(&node.right, v, hi)
        }
    }
}
```

### 2. Integer overflow in the bounds
```rust
// WRONG — fails when a node holds i32::MIN / i32::MAX
fn bad(n: &Option<Box<TreeNode>>, lo: i32, hi: i32) { /* ... */ }
// CORRECT — use i64 sentinels
fn good(n: &Option<Box<TreeNode>>, lo: i64, hi: i64) { /* ... */ }   // start with i64::MIN/i64::MAX
```

### 3. Kth-smallest off-by-one
```rust
// WRONG — checks before decrementing, returning the (k+1)-th
if *k == 0 { return cur.val; } *k -= 1;
// CORRECT — pre-decrement, compare to 0
*k -= 1; if *k == 0 { return cur.val; }
```

### 4. Delete not handling the two-children case
```rust
// WRONG — only patches leaves / single children, corrupts two-child nodes
if root.val == key { return None; }
// CORRECT — splice single child, else promote inorder successor
if root.left.is_none()  { return root.right.take(); }
if root.right.is_none() { return root.left.take(); }
// find inorder successor (leftmost in right subtree)
let mut s = root.right.as_ref().unwrap();
while s.left.is_some() { s = s.left.as_ref().unwrap(); }
let succ_val = s.val;
root.val = succ_val;
root.right = delete_node(root.right.take(), succ_val);
```

### 5. Recover BST: ignoring the adjacent-vs-non-adjacent case
```rust
// WRONG — assumes two descents always exist; breaks for an adjacent swap (one descent)
if descent { if first.is_none() { first = prev; } else { second = node; } }
// CORRECT — first only on the first descent; second on EVERY descent
if descent { if first.is_none() { first = prev; } second = node; }
```

### 6. Forgetting "strictly" increasing (duplicates)
```rust
// WRONG — accepts equal neighbors as valid
if prev.is_some() && node.val < prev.unwrap() { return false; }
// CORRECT — a BST inorder must strictly increase
if prev.is_some() && node.val <= prev.unwrap() { return false; }
```

### 7. Using the full-tree LCA algorithm on a BST
```rust
// WRONG — O(n), searches both subtrees, ignores ordering
fn lca<'a>(r: &'a Option<Box<TreeNode>>, p: i32, q: i32) -> Option<&'a Box<TreeNode>> {
    match r {
        None => None,
        Some(node) if node.val == p || node.val == q => Some(node),
        Some(node) => {
            let l = lca(&node.left, p, q);
            let rr = lca(&node.right, p, q);
            match (l, rr) {
                (Some(_), Some(_)) => Some(node),
                (Some(x), None) => Some(x),
                (None, Some(x)) => Some(x),
                _ => None,
            }
        }
    }
}
// CORRECT — O(h) descent to the split point
let mut cur = root.as_ref();
while let Some(node) = cur {
    if p < node.val && q < node.val      { cur = node.left.as_ref(); }
    else if p > node.val && q > node.val { cur = node.right.as_ref(); }
    else { return Some(node); }
}
```

### 8. BST Iterator not pushing the leftmost spine
```rust
// WRONG — pushes only the node, so next() can skip its left descendants
fn push(st: &mut Vec<&TreeNode>, n: Option<&TreeNode>) {
    if let Some(node) = n { st.push(node); }
}
// CORRECT — push the node and ALL of its left children
fn push_leftmost(st: &mut Vec<&TreeNode>, mut n: Option<&TreeNode>) {
    while let Some(node) = n { st.push(node); n = node.left.as_deref(); }
}
```

### 9. Range Sum not pruning
```rust
// WRONG — visits every node, ignoring the BST property (O(n) always)
fn sum(n: &Option<Box<TreeNode>>, lo: i32, hi: i32) -> i32 {
    match n {
        None => 0,
        Some(node) => {
            let s = if node.val >= lo && node.val <= hi { node.val } else { 0 };
            s + sum(&node.left, lo, hi) + sum(&node.right, lo, hi)
        }
    }
}
// CORRECT — skip whole subtrees out of range
fn sum(n: &Option<Box<TreeNode>>, lo: i32, hi: i32) -> i32 {
    match n {
        None => 0,
        Some(node) if node.val < lo => sum(&node.right, lo, hi),
        Some(node) if node.val > hi => sum(&node.left, lo, hi),
        Some(node) => node.val + sum(&node.left, lo, hi) + sum(&node.right, lo, hi),
    }
}
```

### 10. Greater Tree: wrong traversal direction
```rust
// WRONG — normal inorder (ascending) accumulates SMALLER keys
inorder(&mut node.left, sum); *sum += node.val; inorder(&mut node.right, sum);
// CORRECT — reverse inorder (descending) accumulates GREATER keys
reverse_inorder(&mut node.right, sum); *sum += node.val; reverse_inorder(&mut node.left, sum);
```

---

> **Last Updated:** 2026-06-26
