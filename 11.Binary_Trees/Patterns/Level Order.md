> **Topic:** [Binary Trees](../README.md) · **Pattern 4 of 5**

# Level Order / BFS

Every level-order problem is the same BFS skeleton with a small per-level twist. Memorize the
template once; then "take the last node" → right side view, "average" → averages, "max" →
largest values, and so on.

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

## The BFS Template (size-snapshot)

```rust
use std::collections::VecDeque;

let mut q: VecDeque<Box<TreeNode>> = VecDeque::new();
q.push_back(root); // assume root is Some(...) (guard separately)
while !q.is_empty() {
    let size = q.len();                     // SNAPSHOT before the inner loop
    for i in 0..size {
        let node = q.pop_front().unwrap();
        // ... per-node work (often using i == 0 or i == size-1) ...
        if let Some(left) = node.left  { q.push_back(left); }
        if let Some(right) = node.right { q.push_back(right); }
    }
    // ... per-level aggregation ...
}
```

> ⚠️ The single most common BFS bug: reading `q.len()` *inside* the loop after you've
> already enqueued children. **Snapshot it first.**

---

## 1. Right Side View (LC 199) — last node per level

```rust
use std::collections::VecDeque;

fn right_side_view(root: Option<Box<TreeNode>>) -> Vec<i32> {
    let mut result = Vec::new();
    let root = match root {
        None => return result,
        Some(r) => r,
    };
    let mut q: VecDeque<Box<TreeNode>> = VecDeque::new();
    q.push_back(root);
    while !q.is_empty() {
        let size = q.len();
        for i in 0..size {
            let node = q.pop_front().unwrap();
            if i == size - 1 { result.push(node.val); }  // rightmost node on this level
            if let Some(left) = node.left  { q.push_back(left); }
            if let Some(right) = node.right { q.push_back(right); }
        }
    }
    result
}
```

---

## 2. Average of Levels (LC 637) — sum / size per level

Use a `f64`/`i64` accumulator to avoid `i32` overflow when summing.

```rust
use std::collections::VecDeque;

fn average_of_levels(root: Option<Box<TreeNode>>) -> Vec<f64> {
    let mut result = Vec::new();
    if root.is_none() { return result; }
    let mut q: VecDeque<Box<TreeNode>> = VecDeque::new();
    q.push_back(root.unwrap());
    while !q.is_empty() {
        let size = q.len();
        let mut sum: i64 = 0;
        for _ in 0..size {
            let node = q.pop_front().unwrap();
            sum += node.val as i64;
            if let Some(left) = node.left  { q.push_back(left); }
            if let Some(right) = node.right { q.push_back(right); }
        }
        result.push(sum as f64 / size as f64);
    }
    result
}
```

---

## 3. Find Largest Value in Each Row (LC 515) — max per level

```rust
use std::collections::VecDeque;

fn largest_values(root: Option<Box<TreeNode>>) -> Vec<i32> {
    let mut result = Vec::new();
    if root.is_none() { return result; }
    let mut q: VecDeque<Box<TreeNode>> = VecDeque::new();
    q.push_back(root.unwrap());
    while !q.is_empty() {
        let size = q.len();
        let mut max_val = i32::MIN;
        for _ in 0..size {
            let node = q.pop_front().unwrap();
            max_val = max_val.max(node.val);
            if let Some(left) = node.left  { q.push_back(left); }
            if let Some(right) = node.right { q.push_back(right); }
        }
        result.push(max_val);
    }
    result
}
```

---

## 4. Level Order Traversal II (LC 107) — bottom-up

Same BFS; prepend each level so the result is bottom-to-top. Use `VecDeque::push_front` (O(1))
or `.reverse()` at the end.

```rust
use std::collections::VecDeque;

fn level_order_bottom(root: Option<Box<TreeNode>>) -> Vec<Vec<i32>> {
    if root.is_none() { return vec![]; }
    let mut dq: VecDeque<Vec<i32>> = VecDeque::new();
    let mut q: VecDeque<Box<TreeNode>> = VecDeque::new();
    q.push_back(root.unwrap());
    while !q.is_empty() {
        let size = q.len();
        let mut level = Vec::new();
        for _ in 0..size {
            let node = q.pop_front().unwrap();
            level.push(node.val);
            if let Some(left) = node.left  { q.push_back(left); }
            if let Some(right) = node.right { q.push_back(right); }
        }
        dq.push_front(level);               // prepend -> bottom-up order, O(1)
    }
    dq.into_iter().collect()
}
```

---

## 5. N-ary Level Order (LC 429)

Children come from `node.children` instead of `left`/`right` — otherwise identical.

```rust
use std::collections::VecDeque;

#[derive(Debug)]
pub struct Node {
    pub val: i32,
    pub children: Vec<Box<Node>>,
}

fn level_order(root: Option<Box<Node>>) -> Vec<Vec<i32>> {
    let mut result = Vec::new();
    if root.is_none() { return result; }
    let mut q: VecDeque<Box<Node>> = VecDeque::new();
    q.push_back(root.unwrap());
    while !q.is_empty() {
        let size = q.len();
        let mut level = Vec::new();
        for _ in 0..size {
            let node = q.pop_front().unwrap();
            level.push(node.val);
            for child in node.children { q.push_back(child); }
        }
        result.push(level);
    }
    result
}
```

---

## 6. Populating Next Right Pointers II (LC 117) — non-perfect tree

For an arbitrary tree, children may be missing, so the perfect-tree trick (LC 116) breaks. The
**dummy-head** technique builds the next level's `next` chain as we walk the current level: a
sentinel `dummy` precedes the next level, and `tail` extends the chain. This is **O(1) extra
space** (no queue).

Index-based representation is used here to avoid cyclic reference issues with `next` pointers.

```rust
// Node stores indices (usize) into a flat Vec for tree structure
struct Node {
    val: i32,
    left: Option<usize>,
    right: Option<usize>,
    next: Option<usize>,
}

fn connect(nodes: &mut Vec<Node>, root: Option<usize>) -> Option<usize> {
    let mut curr = root;
    while let Some(curr_idx) = curr {
        let mut dummy_next: Option<usize> = None;  // sentinel head of the NEXT level
        let mut tail: Option<usize> = None;
        let mut c = Some(curr_idx);
        while let Some(ci) = c {
            // copy fields to locals to satisfy borrow checker
            let left  = nodes[ci].left;
            let right = nodes[ci].right;
            let next  = nodes[ci].next;
            for child_opt in [left, right] {
                if let Some(child_idx) = child_opt {
                    if let Some(ti) = tail {
                        nodes[ti].next = Some(child_idx);
                    } else {
                        dummy_next = Some(child_idx);
                    }
                    tail = Some(child_idx);
                }
            }
            c = next;
        }
        curr = dummy_next;               // descend to the next level's head
    }
    root
}
```

The same dummy-head chaining idea handles the perfect-tree case too, so this one solution
covers both LC 116 and LC 117.

---

## Complexity

| Problem | LC | Time | Space |
|---|---|---|---|
| Level Order | 102 | O(n) | O(w) |
| Right Side View | 199 | O(n) | O(w) |
| Average of Levels | 637 | O(n) | O(w) |
| Largest Value Each Row | 515 | O(n) | O(w) |
| Level Order II | 107 | O(n) | O(w) |
| N-ary Level Order | 429 | O(n) | O(w) |
| Populating Next Right II | 117 | O(n) | **O(1)** (dummy-head) |

*w = maximum width of any level (up to n/2 for the bottom level of a full tree).*

---

### Related
- [Tree Traversals](Tree%20Traversals.md) — zigzag & vertical order live there
- [Tree Construction](Tree%20Construction.md) — Maximum Width uses indexed BFS; LC 116

> **Last Updated:** 2026-06-26
