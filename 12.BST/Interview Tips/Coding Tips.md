> **Topic:** [Binary Search Trees](../README.md) · **Tips 1 of 4**

# Coding Tips — BST

Eight tips that turn most BST interview problems into a 5-minute exercise.

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

---

### 1. Inorder = sorted. Reach for it first.
Left → node → right visits keys in **ascending** order. Before writing anything clever, ask "does
this become trivial on the sorted sequence?" — it does for [validate](../Patterns/BST%20Validation%20and%20Inorder.md),
kth-smallest, min-abs-diff, two-sum, and recover.

### 2. Validate with (low, high) bounds, not just a parent compare.
A node can be larger than its parent yet still violate an ancestor's bound. Pass an open interval
down and tighten it:

```rust
fn valid(n: Option<&Box<TreeNode>>, lo: i64, hi: i64) -> bool {
    match n {
        None => true,
        Some(node) => {
            let v = node.val as i64;
            if v <= lo || v >= hi { return false; }
            valid(node.left.as_ref(), lo, v) && valid(node.right.as_ref(), v, hi)
        }
    }
}
```

### 3. Use `i64` for validation bounds.
A node holding `i32::MIN` or `i32::MAX` breaks `i32` bounds at the boundary.
Start from `i64::MIN` / `i64::MAX` (or store the previous `&TreeNode` reference instead
of a primitive).

### 4. O(h) compare-and-descend for LCA, floor, ceil, successor.
Never run a full-tree O(n) search when ordering is available. Compare the target against the
current node and descend one side, updating a candidate as you pass.

```rust
// LCA: descend to the split point
let mut cur = root.as_ref();
while let Some(node) = cur {
    if p_val < node.val && q_val < node.val {
        cur = node.left.as_ref();
    } else if p_val > node.val && q_val > node.val {
        cur = node.right.as_ref();
    } else {
        return cur; // split point is the LCA
    }
}
```

### 5. Use an explicit stack for *controlled* inorder.
When you need to stop early (kth smallest) or pause/resume (BST Iterator), drive inorder with a
`Vec` as a stack:

```rust
let mut stk: Vec<&TreeNode> = Vec::new();
let mut cur = root.as_ref();
loop {
    while let Some(node) = cur {
        stk.push(node);
        cur = node.left.as_ref();
    }
    match stk.pop() {
        None => break,
        Some(node) => {
            // visit node — break here for early stop
            cur = node.right.as_ref();
        }
    }
}
```

### 6. Reverse inorder for "sum of greater" problems.
Right → node → left visits keys **descending**. Carry a running sum to convert a BST to a greater
tree (LC 538) in one pass.

### 7. Two-children delete = replace with the inorder successor.
The successor (leftmost of the right subtree) is the smallest key larger than the node, so
promoting its value keeps the invariant. Then delete that successor copy from the right subtree.

```rust
// find inorder successor (leftmost of right subtree)
let mut succ = root.right.as_ref().unwrap();
while let Some(left) = succ.left.as_ref() {
    succ = left;
}
let succ_val = succ.val;
root.val = succ_val;
root.right = delete_node(root.right.take(), succ_val);
```

### 8. Prune with the BST property on range queries.
If `node.val < low`, the whole left subtree is out of range; if `node.val > high`, the whole right
subtree is. Skip them — that is the entire speedup of [Range Sum](../Patterns/BST%20Validation%20and%20Inorder.md).

---

> **Bonus — comparisons:** use `(a > b) as i32 - (a < b) as i32` rather than `a - b`; subtraction overflows
> for large/negative values. It rarely matters inside a BST descent (which uses `<`/`>`) but
> matters whenever you write a comparator over node values.

> **Last Updated:** 2026-06-26
