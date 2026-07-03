> **Topic:** [Binary Trees](../README.md) · **Pattern 2 of 5**

# Tree Construction

Rebuilding a tree from traversal arrays, and structural rewrites (flatten, width, next
pointers). The unifying idea: **the root is identifiable from one array, and its position in
another array splits the remaining nodes into left and right subtrees.**

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

## 1. Construct from Preorder & Inorder (LC 105)

- **Preorder** = `[Root | Left subtree | Right subtree]` → the root is `preorder[preStart]`.
- **Inorder**  = `[Left subtree | Root | Right subtree]` → finding the root in inorder splits
  left vs right.

Use a `HashMap<val → inorder_index>` for **O(1)** root lookup instead of a linear scan.

**Key invariants:**
```
rootVal  = preorder[preStart]
inIdx    = indexInInorder(rootVal)
leftSize = inIdx - inStart            // count of nodes in the left subtree
left  subtree: preorder[preStart+1 .. preStart+leftSize], inorder[inStart .. inIdx-1]
right subtree: preorder[preStart+leftSize+1 .. ],          inorder[inIdx+1 .. inEnd]
```

```rust
use std::collections::HashMap;

pub fn build_tree(preorder: Vec<i32>, inorder: Vec<i32>) -> Option<Box<TreeNode>> {
    let mut idx: HashMap<i32, i32> = HashMap::new();
    for (i, &val) in inorder.iter().enumerate() {
        idx.insert(val, i as i32);
    }
    let mut pre_index: i32 = 0;
    build(&preorder, 0, inorder.len() as i32 - 1, &idx, &mut pre_index)
}

fn build(
    preorder: &[i32],
    in_start: i32,
    in_end: i32,
    idx: &HashMap<i32, i32>,
    pre_index: &mut i32,
) -> Option<Box<TreeNode>> {
    if in_start > in_end { return None; }
    let root_val = preorder[*pre_index as usize];
    *pre_index += 1;   // preorder consumed front-to-back
    let mut root = Box::new(TreeNode::new(root_val));
    let in_idx = idx[&root_val];
    root.left  = build(preorder, in_start, in_idx - 1, idx, pre_index);  // build LEFT first
    root.right = build(preorder, in_idx + 1, in_end, idx, pre_index);    // matches preorder order
    Some(root)
}
```

> Build the **left** subtree before the right, because the global `pre_index` must advance
> through the left subtree's nodes (which appear first in preorder).

---

## 2. Construct from Inorder & Postorder (LC 106)

**Postorder** = `[Left | Right | Root]` → the root is the **last** element. Reading postorder
**back-to-front** gives Root, then Right subtree, then Left subtree — so build the **right**
subtree before the left.

```rust
use std::collections::HashMap;

pub fn build_tree(inorder: Vec<i32>, postorder: Vec<i32>) -> Option<Box<TreeNode>> {
    let mut idx: HashMap<i32, i32> = HashMap::new();
    for (i, &val) in inorder.iter().enumerate() {
        idx.insert(val, i as i32);
    }
    let n = postorder.len() as i32;
    let mut post_index: i32 = n - 1;   // consume from the back
    build(&postorder, 0, n - 1, &idx, &mut post_index)
}

fn build(
    postorder: &[i32],
    in_start: i32,
    in_end: i32,
    idx: &HashMap<i32, i32>,
    post_index: &mut i32,
) -> Option<Box<TreeNode>> {
    if in_start > in_end { return None; }
    let root_val = postorder[*post_index as usize];
    *post_index -= 1;
    let mut root = Box::new(TreeNode::new(root_val));
    let in_idx = idx[&root_val];
    root.right = build(postorder, in_idx + 1, in_end, idx, post_index);   // RIGHT first (mirror of LC 105)
    root.left  = build(postorder, in_start, in_idx - 1, idx, post_index);
    Some(root)
}
```

---

## 3. Construct from Preorder & Postorder (LC 889)

Without inorder the answer isn't unique, but any valid tree works. Identify the **left
subtree's root** = `preorder[preStart+1]`, find it in postorder to compute the left-subtree
size.

```rust
use std::collections::HashMap;

pub fn construct_from_pre_post(pre: Vec<i32>, post: Vec<i32>) -> Option<Box<TreeNode>> {
    let mut post_idx: HashMap<i32, i32> = HashMap::new();
    for (i, &val) in post.iter().enumerate() {
        post_idx.insert(val, i as i32);
    }
    let mut pre_index: i32 = 0;
    build(&pre, &post, 0, post.len() as i32 - 1, &post_idx, &mut pre_index)
}

fn build(
    pre: &[i32],
    post: &[i32],
    post_start: i32,
    post_end: i32,
    post_idx: &HashMap<i32, i32>,
    pre_index: &mut i32,
) -> Option<Box<TreeNode>> {
    if *pre_index >= pre.len() as i32 || post_start > post_end {
        return None;
    }
    let mut root = Box::new(TreeNode::new(pre[*pre_index as usize]));
    *pre_index += 1;
    if post_start == post_end { return Some(root); }  // leaf
    let left_root_val = pre[*pre_index as usize];     // next in preorder is left child's root
    let left_root_pos = post_idx[&left_root_val];
    root.left  = build(pre, post, post_start, left_root_pos, post_idx, pre_index);
    root.right = build(pre, post, left_root_pos + 1, post_end - 1, post_idx, pre_index);
    Some(root)
}
```

---

## 4. Maximum Width of Binary Tree (LC 662)

Assign each node a **heap-style index**: a node at index `i` has children `2i` and `2i+1`.
Width of a level = `rightmostIndex − leftmostIndex + 1`. **Normalize** indices each level by
subtracting the level's leftmost index to prevent `i32` overflow on deep trees.

```rust
use std::collections::VecDeque;

pub fn width_of_binary_tree(root: Option<Box<TreeNode>>) -> i32 {
    let root = match root {
        None => return 0,
        Some(r) => r,
    };
    fn bfs(root: &TreeNode) -> i32 {
        let mut max_width: u64 = 0;
        // Queue holds (&node, heap-style index)
        let mut queue: VecDeque<(&TreeNode, u64)> = VecDeque::new();
        queue.push_back((root, 0));
        while !queue.is_empty() {
            let size = queue.len();
            let leftmost = queue.front().unwrap().1;  // normalize against this
            let (mut first, mut last) = (0u64, 0u64);
            for i in 0..size {
                let (node, raw_idx) = queue.pop_front().unwrap();
                let idx = raw_idx - leftmost;   // overflow guard
                if i == 0 { first = idx; }
                if i == size - 1 { last = idx; }
                if let Some(left) = &node.left {
                    queue.push_back((left.as_ref(), 2 * idx));
                }
                if let Some(right) = &node.right {
                    queue.push_back((right.as_ref(), 2 * idx + 1));
                }
            }
            max_width = max_width.max(last - first + 1);
        }
        max_width as i32
    }
    bfs(&root)
}
```

---

## 5. Flatten Binary Tree to Linked List (LC 114)

Flatten in-place into a "linked list" using the **right** pointers, in preorder.

### Approach A — Iterative preorder rebuild, O(n) time

Collect nodes in preorder, then rebuild as a right-linked list. (The C++ Morris O(1)-space
approach requires simultaneous mutable aliasing that Rust's borrow checker disallows safely;
this idiomatic Rust equivalent has the same O(n) time at O(n) extra space.)

```rust
pub fn flatten(root: &mut Option<Box<TreeNode>>) {
    fn preorder(node: Option<Box<TreeNode>>, out: &mut Vec<i32>) {
        if let Some(n) = node {
            out.push(n.val);
            preorder(n.left, out);
            preorder(n.right, out);
        }
    }
    let mut vals = Vec::new();
    preorder(root.take(), &mut vals);
    // Rebuild as right-linked list from the back
    *root = vals.into_iter().rev().fold(None, |next, val| {
        let mut node = Box::new(TreeNode::new(val));
        node.right = next;
        Some(node)
    });
}
```

### Approach B — recursive reverse-postorder (Right → Left → Root)
Visit in reverse preorder, chaining each node onto an accumulated `prev`.

```rust
pub fn flatten_recursive(root: &mut Option<Box<TreeNode>>) {
    // Visit order: right → left → root; thread each node onto accumulated tail
    fn helper(node: Option<Box<TreeNode>>, prev: Option<Box<TreeNode>>) -> Option<Box<TreeNode>> {
        match node {
            None => prev,
            Some(mut n) => {
                let right = n.right.take();
                let left  = n.left.take();
                let prev = helper(right, prev);   // right first
                let prev = helper(left, prev);
                n.right = prev;                   // attach what comes after in preorder
                n.left  = None;
                Some(n)
            }
        }
    }
    *root = helper(root.take(), None);
}
```

---

## 6. Populating Next Right Pointers (LC 116 — perfect tree)

In a **perfect** binary tree, use the already-established `next` pointers of the parent level
to connect the child level in **constant space** (no queue). Uses an index-based arena Vec to
represent the `next` sibling pointer without requiring shared mutable references.

```rust
// Node struct for next-right-pointer problem (index-based arena)
pub struct Node {
    pub val: i32,
    pub left: Option<usize>,   // index into arena Vec
    pub right: Option<usize>,
    pub next: Option<usize>,
}

pub fn connect(root: Option<usize>, nodes: &mut Vec<Node>) -> Option<usize> {
    if root.is_none() { return root; }
    let mut leftmost = root;
    while let Some(lm) = leftmost {
        if nodes[lm].left.is_none() { break; }  // reached leaf level
        let mut head = Some(lm);
        while let Some(h) = head {
            let left  = nodes[h].left.unwrap();
            let right = nodes[h].right.unwrap();
            nodes[left].next = Some(right);              // connect within same parent
            if let Some(nx) = nodes[h].next {
                nodes[right].next = nodes[nx].left;      // connect across adjacent parents
            }
            head = nodes[h].next;
        }
        leftmost = nodes[lm].left;
    }
    root
}
```

For the **non-perfect** variant (LC 117), see [Level Order](Level%20Order.md) — the
**dummy-head** technique handles missing children.

---

## Complexity

| Problem | LC | Time | Space |
|---|---|---|---|
| Construct from Pre+In | 105 | O(n) | O(n) map + O(h) stack |
| Construct from In+Post | 106 | O(n) | O(n) + O(h) |
| Construct from Pre+Post | 889 | O(n) | O(n) + O(h) |
| Maximum Width | 662 | O(n) | O(w) |
| Flatten (iterative) | 114 | O(n) | O(n) |
| Flatten (recursive) | 114 | O(n) | O(h) |
| Populating Next Right | 116 | O(n) | **O(1)** |

---

### Related
- [Tree Traversals](Tree%20Traversals.md) — the arrays we rebuild from
- [Level Order](Level%20Order.md) — Next Right Pointers II, the dummy-head trick

> **Last Updated:** 2026-06-26
