> **Topic:** [Binary Trees](../README.md) · **Pattern 1 of 4**

# Binary Trees — Coding Tips

A focused set of practical coding tips that make binary-tree problems faster to write and harder to get wrong in an interview setting. Each tip pairs a short explanation with a Rust snippet you can reproduce from memory.

Throughout these tips we assume the standard node definition:

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

## 1. TreeNode null-check discipline

The single most reliable habit in tree recursion: **the first line of every recursive function handles the `None` node**. This terminates recursion at the "edges" of the tree (below the leaves) and removes the need to null-check children individually before recursing.

If you make this the very first statement every time, you will almost never hit a null pointer dereference, and your base case is always explicit.

```rust
fn max_depth(node: &Option<Box<TreeNode>>) -> i32 {
    match node {
        None => 0,  // first line, always
        Some(n) => 1 + max_depth(&n.left).max(max_depth(&n.right)),
    }
}
```

The return value of the base case is problem-specific (`0` for depth/height, `None` for "no node found", `true` for vacuously-true predicates, etc.), but the *shape* is invariant.

---

## 2. Leaf check pattern

A leaf is a node with **no children**. Many problems (Path Sum, Min Depth, Sum Root to Leaf, Binary Tree Paths) treat leaves specially, so commit the exact condition to muscle memory:

```rust
if node.left.is_none() && node.right.is_none() {
    // this is a leaf — terminate the path / record the answer here
}
```

Be careful to distinguish this from the `None`-node base case. A common bug in **Min Depth** is treating a node with one child as a leaf — it is not. You only short-circuit on a *true* leaf, and otherwise take the depth of the non-null subtree.

```rust
fn min_depth(node: &Option<Box<TreeNode>>) -> i32 {
    match node {
        None => 0,
        Some(n) => {
            if n.left.is_none()  { return 1 + min_depth(&n.right); }  // not a leaf
            if n.right.is_none() { return 1 + min_depth(&n.left); }   // not a leaf
            1 + min_depth(&n.left).min(min_depth(&n.right))
        }
    }
}
```

---

## 3. Diameter doesn't pass through the root — track a global max

A frequent misconception is that the diameter (longest path between any two nodes) must run through the root. It does not. The longest path can sit entirely inside a subtree.

The trick: write a helper that **returns the height** of a subtree, but as a side effect **updates a global** with `leftHeight + rightHeight` at every node. The diameter is the maximum of those local sums across the whole tree.

```rust
let mut best: i32 = 0;  // global state (passed as mutable reference)

fn height(node: &Option<Box<TreeNode>>, best: &mut i32) -> i32 {
    match node {
        None => 0,
        Some(n) => {
            let left  = height(&n.left, best);
            let right = height(&n.right, best);
            *best = (*best).max(left + right);  // update at EVERY node
            1 + left.max(right)                  // return value is the height
        }
    }
}
```

This "return one thing, accumulate another globally" pattern generalizes to Max Path Sum (Tip 7), House Robber III, and more.

---

## 4. BFS size snapshot

Level-order traversal (BFS) needs to know **where one level ends and the next begins**. The clean way is to capture the queue size *before* the inner loop, because the queue grows as you enqueue children.

```rust
use std::collections::VecDeque;

let mut q: VecDeque<&TreeNode> = VecDeque::new();
if let Some(root_node) = &root {
    q.push_back(root_node);
}
while !q.is_empty() {
    let size = q.len();           // SNAPSHOT before the inner loop
    let mut level: Vec<i32> = Vec::new();
    for _ in 0..size {            // iterate exactly this level
        let node = q.pop_front().unwrap();
        level.push(node.val);
        if let Some(left)  = &node.left  { q.push_back(left); }
        if let Some(right) = &node.right { q.push_back(right); }
    }
    // `level` now holds one full level
}
```

If you omit the size snapshot and re-evaluate `q.len()` inside the inner loop, the loop bound shifts as children are added and levels bleed into each other.

---

## 5. Morris traversal — O(1) space inorder via threading

Recursive and stack-based inorder both use O(h) extra space. **Morris traversal** achieves O(1) space (ignoring output) by temporarily *threading* each node to its inorder successor.

Core idea: for the current node, find its inorder predecessor (rightmost node of its left subtree). Create a temporary right-pointer ("thread") from that predecessor back to the current node so you can climb back up without a stack. On the second visit, remove the thread and emit the value.

```rust
// Tree represented as Vec<(val, left_idx, right_idx)> with Option<usize> indices.
// Morris traversal creates temporary "thread" pointers — stored here as mutable index slots.
fn morris_inorder(nodes: &mut Vec<(i32, Option<usize>, Option<usize>)>, root: Option<usize>) -> Vec<i32> {
    let mut out: Vec<i32> = Vec::new();
    let mut curr = root;
    while let Some(c) = curr {
        if nodes[c].1.is_none() {
            out.push(nodes[c].0);
            curr = nodes[c].2;              // move right
        } else {
            let mut pred = nodes[c].1.unwrap();
            while nodes[pred].2.is_some() && nodes[pred].2 != Some(c) {
                pred = nodes[pred].2.unwrap();
            }
            if nodes[pred].2.is_none() {
                nodes[pred].2 = Some(c);    // create thread
                curr = nodes[c].1;           // move left
            } else {
                nodes[pred].2 = None;        // remove thread, restore tree
                out.push(nodes[c].0);
                curr = nodes[c].2;           // move right
            }
        }
    }
    out
}
```

Mention this when an interviewer asks for **O(1) extra space** traversal — it shows you know the trick beyond the textbook stack approach.

---

## 6. Index normalization for Max Width

To compute the maximum width of a tree, assign each node a **heap-style index**: a node at index `i` has children at `2*i` and `2*i + 1`. The width of a level is `rightmostIdx - leftmostIdx + 1`.

The danger: on a deep, sparse tree these indices grow exponentially and **overflow `i32`/`i64`**. Normalize every level by subtracting the leftmost index on that level, which keeps indices small without changing the *differences* (which is all width depends on).

```rust
use std::collections::VecDeque;

fn width_of_binary_tree(root: &Option<Box<TreeNode>>) -> i32 {
    let root_node = match root {
        None => return 0,
        Some(n) => n.as_ref(),
    };
    let mut max_w: i32 = 0;
    let mut nodes: VecDeque<&TreeNode> = VecDeque::new();
    let mut idxs: VecDeque<i64> = VecDeque::new();
    nodes.push_back(root_node);
    idxs.push_back(0);
    while !nodes.is_empty() {
        let size = nodes.len();
        let leftmost = *idxs.front().unwrap();  // base for this level
        let mut first = 0_i64;
        let mut last = 0_i64;
        for i in 0..size {
            let node = nodes.pop_front().unwrap();
            let idx = idxs.pop_front().unwrap() - leftmost; // NORMALIZE
            if i == 0        { first = idx; }
            if i == size - 1 { last = idx; }
            if let Some(left)  = &node.left  { nodes.push_back(left);  idxs.push_back(2 * idx); }
            if let Some(right) = &node.right { nodes.push_back(right); idxs.push_back(2 * idx + 1); }
        }
        max_w = max_w.max((last - first + 1) as i32);
    }
    max_w
}
```

---

## 7. `gain = max(0, gain)` for Max Path Sum

In Maximum Path Sum, each node reports to its parent the best **single downward path gain** it can contribute. If a subtree's best contribution is negative, it is better to contribute **nothing** (i.e. cut that branch off) than to drag the path down.

Clamp each child's gain at zero before using it:

```rust
let mut best = i32::MIN;  // global (passed by mutable reference)

fn gain(node: &Option<Box<TreeNode>>, best: &mut i32) -> i32 {
    match node {
        None => 0,
        Some(n) => {
            let left  = 0_i32.max(gain(&n.left, best));   // prune negatives
            let right = 0_i32.max(gain(&n.right, best));
            *best = (*best).max(n.val + left + right);  // path "through" this node
            n.val + left.max(right)                      // gain handed to parent
        }
    }
}
```

The path that bends through a node uses *both* children (`left + right`), but the gain handed upward can only follow *one* branch (`left.max(right)`), because a path cannot fork.

---

## 8. Parent-map for distance problems

Trees only have child pointers, so "move toward the parent" is impossible by default. For problems like **All Nodes Distance K** or **LCA Deepest Leaves**, first build a `child -> parent` map, then you can BFS outward in all three directions (left child, right child, parent).

```rust
use std::collections::HashMap;

// parent_map: maps node index → parent index (None for root)
fn build_parents(
    nodes: &[(i32, Option<usize>, Option<usize>)],
    idx: Option<usize>,
    par: Option<usize>,
    parent: &mut HashMap<usize, Option<usize>>,
) {
    if let Some(i) = idx {
        parent.insert(i, par);
        build_parents(nodes, nodes[i].1, Some(i), parent);
        build_parents(nodes, nodes[i].2, Some(i), parent);
    }
}
```

Once built, a standard BFS from the target node — enqueuing the left child index, right child index, and `parent[&node_idx]` while tracking visited indices — finds everything at distance K.

---

## 9. Backtrack discipline for path collection

When collecting root-to-leaf paths (Binary Tree Paths, Path Sum II), you reuse a single mutable `path` Vec and **undo** your change after recursing. The pattern is: add, recurse into both children, then remove the last element.

```rust
fn dfs(node: &Option<Box<TreeNode>>, path: &mut Vec<i32>, result: &mut Vec<Vec<i32>>) {
    if let Some(n) = node {
        path.push(n.val);                                    // choose
        if n.left.is_none() && n.right.is_none() {
            result.push(path.clone());    // record a COPY at a leaf
        } else {
            dfs(&n.left, path, result);                   // explore
            dfs(&n.right, path, result);
        }
        path.pop();                       // un-choose (backtrack)
    }
}
```

Two non-negotiables: (1) `result.push(path.clone())` explicitly clones the Vec, so the stored snapshot is independent of future mutations; (2) the `path.pop()` runs on **every** return, including after recording a leaf, so it must sit outside the leaf branch as shown.

---

> **Last Updated:** 2026-06-26
