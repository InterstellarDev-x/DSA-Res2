> **Topic:** [Binary Search Trees](../README.md) · **Pattern 1 of 4**

# BST Operations — Search, Insert, Delete, Min-Abs-Diff

**Core idea:** every BST operation is a **compare-and-descend** walk. At each node, compare the
target with `node.val` and move *left* (target smaller) or *right* (target larger). This visits
one node per level → **O(h)** time, where `h = O(log n)` for a balanced tree and `O(n)` for a
skewed one. No extra structure is needed; the ordering *is* the index.

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

> **Recognition signals:** "find/insert/remove a key", "the tree is a BST", "in O(h)/O(log n)",
> "smallest difference between any two nodes", "successor of the deleted node".

---

## 1. Search in a BST (LC 700) — Easy

Compare and descend. Iterative is the cleanest and uses **O(1)** space.

```rust
fn search_bst(root: &Option<Box<TreeNode>>, val: i32) -> &Option<Box<TreeNode>> {
    let mut cur = root;
    loop {
        match cur {
            None => return cur,
            Some(node) if node.val == val => return cur,
            Some(node) => {
                cur = if val < node.val { &node.left } else { &node.right };
            }
        }
    }   // None if not found
}
```

Recursive form (O(h) stack):

```rust
fn search_bst_rec(root: &Option<Box<TreeNode>>, val: i32) -> &Option<Box<TreeNode>> {
    match root {
        None => root,
        Some(node) if node.val == val => root,
        Some(node) => {
            if val < node.val {
                search_bst_rec(&node.left, val)
            } else {
                search_bst_rec(&node.right, val)
            }
        }
    }
}
```

**Complexity:** Time O(h); Space O(1) iterative, O(h) recursive.

---

## 2. Insert into a BST (LC 701) — Medium

Walk down as if searching; when you fall off the tree (hit `None`), that empty slot is exactly
where the key belongs. The recursive version "re-attaches" subtrees on the way back up.

```rust
fn insert_into_bst(root: Option<Box<TreeNode>>, val: i32) -> Option<Box<TreeNode>> {
    match root {
        None => Some(Box::new(TreeNode::new(val))),   // attach at the None leaf
        Some(mut node) => {
            if val < node.val {
                node.left = insert_into_bst(node.left, val);
            } else {
                node.right = insert_into_bst(node.right, val);
            }
            Some(node)
        }
    }
}
```

Iterative (track the parent, no recursion stack):

```rust
fn insert_iterative(root: Option<Box<TreeNode>>, val: i32) -> Option<Box<TreeNode>> {
    if root.is_none() {
        return Some(Box::new(TreeNode::new(val)));
    }
    let mut root = root;
    let mut cur = root.as_mut().unwrap();
    loop {
        if val < cur.val {
            if cur.left.is_none() { cur.left = Some(Box::new(TreeNode::new(val))); break; }
            cur = cur.left.as_mut().unwrap();
        } else {
            if cur.right.is_none() { cur.right = Some(Box::new(TreeNode::new(val))); break; }
            cur = cur.right.as_mut().unwrap();
        }
    }
    root
}
```

**Complexity:** Time O(h); Space O(h) recursive / O(1) iterative. Problem guarantees `val` is new.

---

## 3. Delete Node in a BST (LC 450) — Medium

The hardest core operation. First *find* the node (compare-and-descend), then handle **three
cases** for the node to remove:

| Case | Action |
|---|---|
| **No child** (leaf) | Return `None` to the parent. |
| **One child** | Return the non-null child to the parent (splice it in). |
| **Two children** | Replace the node's value with its **inorder successor** (the *leftmost* node of the right subtree), then recursively delete that successor from the right subtree. |

The inorder successor is the smallest key larger than the node — it preserves the BST invariant
when promoted.

```rust
fn delete_node(root: Option<Box<TreeNode>>, key: i32) -> Option<Box<TreeNode>> {
    match root {
        None => None,
        Some(mut node) => {
            if key < node.val {
                node.left = delete_node(node.left, key);
                Some(node)
            } else if key > node.val {
                node.right = delete_node(node.right, key);
                Some(node)
            } else {
                // found the node to delete
                if node.left.is_none() { return node.right; }   // 0 or 1 child (right)
                if node.right.is_none() { return node.left; }   // 1 child (left)
                // two children: replace with inorder successor = leftmost of right subtree
                let succ_val = {
                    let mut succ = node.right.as_ref().unwrap();
                    while let Some(ref left) = succ.left {
                        succ = left;
                    }
                    succ.val
                };
                node.val = succ_val;
                node.right = delete_node(node.right, succ_val); // delete the successor copy
                Some(node)
            }
        }
    }
}
```

**Complexity:** Time O(h) (the recursive successor deletion is also bounded by the height);
Space O(h).

> Symmetric alternative: replace with the inorder **predecessor** (rightmost of the *left*
> subtree). Either is correct; pick one and be consistent.

---

## 4. Minimum Absolute Difference in BST (LC 530) — Easy

Since inorder yields a **sorted** sequence, the minimum absolute difference can only occur between
**adjacent** values in that order. Walk inorder, keep the previously-visited value in `prev_val`, and
minimize `cur - prev_val`.

```rust
fn inorder(node: &Option<Box<TreeNode>>, prev_val: &mut Option<i32>, min_diff: &mut i32) {
    if let Some(n) = node {
        inorder(&n.left, prev_val, min_diff);
        if let Some(prev) = *prev_val {
            *min_diff = (*min_diff).min(n.val - prev); // sorted => n.val >= prev
        }
        *prev_val = Some(n.val);
        inorder(&n.right, prev_val, min_diff);
    }
}

fn get_minimum_difference(root: &Option<Box<TreeNode>>) -> i32 {
    let mut prev_val: Option<i32> = None;
    let mut min_diff = i32::MAX;
    inorder(root, &mut prev_val, &mut min_diff);
    min_diff
}
```

**Complexity:** Time O(n) (must touch every node); Space O(h). Same idea solves LC 783.

---

## Dry Run — Delete 5 from a two-children node

Tree (key to delete = `5`, which has two children):

```
        5
       / \
      3   7
     / \   \
    2   4   8
```

1. `delete_node(root=5, key=5)` → match. Both children non-null → **two-children case**.
2. Find inorder successor: start at `root.right = 7`, walk left while possible → `7` has no left
   child, so `succ = 7`.
3. Copy `succ.val (7)` into the node: root value becomes `7`.
4. Delete the successor copy from the right subtree: `delete_node(node 7, key 7)`. That `7` has no
   left child → return its right child `8`.

Result (still a valid BST, sorted inorder `2 3 4 7 8`):

```
        7
       / \
      3   8
     / \
    2   4
```

---

## Summary

| Operation | Technique | Time | Space |
|---|---|---|---|
| Search (700) | compare-and-descend | O(h) | O(1) iter |
| Insert (701) | attach at None leaf | O(h) | O(1) iter |
| Delete (450) | 3 cases; two-children → successor | O(h) | O(h) |
| Min Abs Diff (530) | inorder + track `prev_val` | O(n) | O(h) |

> **Last Updated:** 2026-06-26
