> **Topic:** [Binary Trees](../README.md) · **Pattern 2 of 4**

# Binary Trees — Common Mistakes

Ten bugs that show up over and over in binary-tree interviews. Each one pairs the broken version (❌) with the fix (✅). Node definition used throughout:

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

## 1. Missing null check before accessing `node.left` / `node.right`

Dereferencing a child without first confirming the node exists is the classic crash (panic). Always handle `None` as the first line.

❌ **Wrong**
```rust
fn max_depth(node: Option<&TreeNode>) -> i32 {
    let node = node.unwrap(); // panic when node is None
    let left = max_depth(node.left.as_deref());
    let right = max_depth(node.right.as_deref());
    1 + left.max(right)
}
```

✅ **Correct**
```rust
fn max_depth(node: Option<&TreeNode>) -> i32 {
    let node = match node {
        None => return 0,       // base case first
        Some(n) => n,
    };
    let left = max_depth(node.left.as_deref());
    let right = max_depth(node.right.as_deref());
    1 + left.max(right)
}
```

---

## 2. Diameter: returning the depth/diameter instead of updating a global

The helper must **return the height** but **record the diameter in a global**. Returning the local `left + right` upward corrupts the height calculation of ancestors.

❌ **Wrong**
```rust
fn diameter(node: Option<&TreeNode>) -> i32 {
    let node = match node {
        None => return 0,
        Some(n) => n,
    };
    let left = diameter(node.left.as_deref());
    let right = diameter(node.right.as_deref());
    left + right               // this is NOT a height; ancestors break
}
```

✅ **Correct**
```rust
fn height(node: Option<&TreeNode>, best: &mut i32) -> i32 {
    let node = match node {
        None => return 0,
        Some(n) => n,
    };
    let left = height(node.left.as_deref(), best);
    let right = height(node.right.as_deref(), best);
    *best = (*best).max(left + right);   // diameter recorded via mutable reference
    1 + left.max(right)                  // height returned upward
}
```

---

## 3. Max Path Sum: forgetting `max(0, gain)`

Without clamping, a negative subtree contribution is allowed to drag the path total down, producing wrong answers on trees with negative values.

❌ **Wrong**
```rust
fn gain(node: Option<&TreeNode>, best: &mut i32) -> i32 {
    let node = match node {
        None => return 0,
        Some(n) => n,
    };
    let left = gain(node.left.as_deref(), best);        // may be negative and still used
    let right = gain(node.right.as_deref(), best);
    *best = (*best).max(node.val + left + right);
    node.val + left.max(right)
}
```

✅ **Correct**
```rust
fn gain(node: Option<&TreeNode>, best: &mut i32) -> i32 {
    let node = match node {
        None => return 0,
        Some(n) => n,
    };
    let left  = 0_i32.max(gain(node.left.as_deref(), best));    // prune negatives
    let right = 0_i32.max(gain(node.right.as_deref(), best));
    *best = (*best).max(node.val + left + right);
    node.val + left.max(right)
}
```

---

## 4. Path Sum II: not backtracking (forgetting to remove from the list)

Reusing a shared list without removing the current node after recursion leaves stale entries, so sibling paths inherit nodes that aren't on them.

❌ **Wrong**
```rust
fn dfs(node: Option<&TreeNode>, target: i32, path: &mut Vec<i32>, res: &mut Vec<Vec<i32>>) {
    let node = match node {
        None => return,
        Some(n) => n,
    };
    path.push(node.val);
    if node.left.is_none() && node.right.is_none() && node.val == target {
        res.push(path.clone());
    }
    dfs(node.left.as_deref(),  target - node.val, path, res);
    dfs(node.right.as_deref(), target - node.val, path, res);
    // missing removal — path keeps growing across branches
}
```

✅ **Correct**
```rust
fn dfs(node: Option<&TreeNode>, target: i32, path: &mut Vec<i32>, res: &mut Vec<Vec<i32>>) {
    let node = match node {
        None => return,
        Some(n) => n,
    };
    path.push(node.val);
    if node.left.is_none() && node.right.is_none() && node.val == target {
        res.push(path.clone());
    }
    dfs(node.left.as_deref(),  target - node.val, path, res);
    dfs(node.right.as_deref(), target - node.val, path, res);
    path.pop();      // backtrack on every return
}
```

---

## 5. Level Order: not capturing size before the inner loop

Reading `q.len()` inside the loop condition lets the bound drift as children are enqueued, merging levels together.

❌ **Wrong**
```rust
use std::collections::VecDeque;

while !q.is_empty() {
    let mut level = Vec::new();
    let mut i = 0;
    while i < q.len() {                  // q.len() changes mid-loop
        let node = q.pop_front().unwrap();
        level.push(node.val);
        if let Some(left) = node.left.as_deref() { q.push_back(left); }
        if let Some(right) = node.right.as_deref() { q.push_back(right); }
        i += 1;
    }
    result.push(level);
}
```

✅ **Correct**
```rust
use std::collections::VecDeque;

while !q.is_empty() {
    let size = q.len();                  // snapshot the level boundary
    let mut level = Vec::new();
    for _ in 0..size {
        let node = q.pop_front().unwrap();
        level.push(node.val);
        if let Some(left) = node.left.as_deref() { q.push_back(left); }
        if let Some(right) = node.right.as_deref() { q.push_back(right); }
    }
    result.push(level);
}
```

---

## 6. Construct from traversals: off-by-one in `leftSize`

When splitting inorder around the root, the count of left-subtree nodes is `in_idx - in_start` (a *count* of elements strictly before the root), not `in_idx - in_start + 1` (which wrongly includes the root).

❌ **Wrong**
```rust
let in_idx = in_map[&root_val];
let left_size = in_idx - in_start + 1;    // includes the root — too large by one
```

✅ **Correct**
```rust
let in_idx = in_map[&root_val];
let left_size = in_idx - in_start;        // nodes strictly left of the root
// use left_size to slice the preorder/postorder ranges for the left subtree
```

---

## 7. Binary Tree Cameras: missing the root check after recursion

The greedy post-order returns states (0 = not covered, 1 = has camera, 2 = covered). After the recursion completes, if the **root itself** comes back uncovered (state 0), you must add one more camera — easy to forget.

❌ **Wrong**
```rust
fn min_camera_cover(root: Option<Box<TreeNode>>) -> i32 {
    let mut cameras = 0_i32;
    dfs(root.as_deref(), &mut cameras);
    cameras                    // ignores an uncovered root → undercount
}
```

✅ **Correct**
```rust
fn min_camera_cover(root: Option<Box<TreeNode>>) -> i32 {
    let mut cameras = 0_i32;
    if dfs(root.as_deref(), &mut cameras) == 0 {
        cameras += 1;          // root left uncovered → place a camera
    }
    cameras
}

// dfs returns: 0 = needs cover, 1 = has camera, 2 = covered
fn dfs(node: Option<&TreeNode>, cameras: &mut i32) -> i32 {
    let node = match node {
        None => return 2,
        Some(n) => n,
    };
    let left = dfs(node.left.as_deref(), cameras);
    let right = dfs(node.right.as_deref(), cameras);
    if left == 0 || right == 0 { *cameras += 1; return 1; }
    if left == 1 || right == 1 { return 2; }
    0
}
```

---

## 8. LCA: returning `None` incorrectly for nodes present in the tree

The lowest-common-ancestor base case must return the node when it matches `p` or `q`. Returning `None` for a matched node (or short-circuiting before checking the match) loses the answer.

❌ **Wrong**
```rust
fn lca<'a>(node: Option<&'a TreeNode>, p: &TreeNode, q: &TreeNode) -> Option<&'a TreeNode> {
    if node.is_none() { return None; }
    let n = node.unwrap();
    if n.left.is_none() && n.right.is_none() { return None; }  // drops leaf matches
    let left = lca(n.left.as_deref(), p, q);
    let right = lca(n.right.as_deref(), p, q);
    if left.is_some() && right.is_some() { return Some(n); }
    if left.is_some() { left } else { right }
}
```

✅ **Correct**
```rust
fn lca<'a>(node: Option<&'a TreeNode>, p: &TreeNode, q: &TreeNode) -> Option<&'a TreeNode> {
    let n = node?;
    if std::ptr::eq(n, p) || std::ptr::eq(n, q) { return Some(n); }  // match returns self
    let left = lca(n.left.as_deref(), p, q);
    let right = lca(n.right.as_deref(), p, q);
    if left.is_some() && right.is_some() { return Some(n); }  // split → this is LCA
    if left.is_some() { left } else { right }
}
```

---

## 9. Zigzag: using `result[level].insert(0, val)` — O(n) — instead of a VecDeque

Inserting at index 0 of a `Vec` shifts every element, turning each level into O(n²). Use a `VecDeque` and `push_front` for O(1) front insertion.

❌ **Wrong**
```rust
use std::collections::VecDeque;

let mut level: Vec<i32> = Vec::new();
// ... for each node on the level:
if left_to_right { level.push(node.val); }
else             { level.insert(0, node.val); }   // O(n) shift each insert
```

✅ **Correct**
```rust
use std::collections::VecDeque;

let mut level: VecDeque<i32> = VecDeque::new();
// ... for each node on the level:
if left_to_right { level.push_back(node.val); }   // O(1)
else             { level.push_front(node.val); }   // O(1)
result.push(Vec::from(level));
```

---

## 10. Max Width: integer overflow without index normalization

Heap-style indices (`2*i`, `2*i+1`) blow past `i32`/`i64` range on deep sparse trees. Normalize each level against its leftmost index so only the differences — which is all the width needs — are preserved.

❌ **Wrong**
```rust
for _ in 0..size {
    let node = nodes.pop_front().unwrap();
    let idx = idxs.pop_front().unwrap();                  // grows exponentially → overflow
    if let Some(left) = node.left.as_deref() {
        nodes.push_back(left);
        idxs.push_back(2 * idx);
    }
    if let Some(right) = node.right.as_deref() {
        nodes.push_back(right);
        idxs.push_back(2 * idx + 1);
    }
}
```

✅ **Correct**
```rust
let leftmost = *idxs.front().unwrap();                    // base for this level
let (mut first, mut last) = (0_usize, 0_usize);
for i in 0..size {
    let node = nodes.pop_front().unwrap();
    let idx = idxs.pop_front().unwrap() - leftmost;       // NORMALIZE → indices stay small
    if i == 0 { first = idx; }
    if i == size - 1 { last = idx; }
    if let Some(left) = node.left.as_deref() {
        nodes.push_back(left);
        idxs.push_back(2 * idx);
    }
    if let Some(right) = node.right.as_deref() {
        nodes.push_back(right);
        idxs.push_back(2 * idx + 1);
    }
}
max_width = max_width.max(last - first + 1);
```

---

> **Last Updated:** 2026-06-26
