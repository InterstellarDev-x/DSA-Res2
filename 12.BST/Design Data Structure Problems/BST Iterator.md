> **Topic:** [Binary Search Trees](../README.md) · **Design Data Structure**

# BST Iterator (LC 173)

Implement an iterator over a BST that returns keys in **ascending (inorder)** order:

- `BSTIterator(TreeNode root)` — initialize.
- `int next()` — return the next-smallest key.
- `boolean hasNext()` — whether a next key exists.

**Both `next()` and `hasNext()` must be amortized O(1), using at most O(h) memory.**

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

## Idea — A Paused Inorder Traversal on a Stack

A recursive inorder dives all the way left, visits, then goes right. We make that traversal
**pausable** by keeping its call stack *explicitly*. The stack always holds the chain of nodes
"whose left side is done but who haven't been visited yet" — i.e. the next node to visit is always
on top.

The helper `pushLeftmost(node)` pushes a node and all of its left descendants. Calling it on `root`
in the constructor primes the stack so the top is the global minimum.

```rust
pub struct BSTIterator {
    stk: Vec<Box<TreeNode>>,
}

impl BSTIterator {
    fn push_leftmost(stk: &mut Vec<Box<TreeNode>>, mut node: Option<Box<TreeNode>>) {
        while let Some(mut n) = node {
            node = n.left.take(); // detach left child so we can push n
            stk.push(n);
        }
    }

    pub fn new(root: Option<Box<TreeNode>>) -> Self {
        let mut iter = BSTIterator { stk: Vec::new() };
        Self::push_leftmost(&mut iter.stk, root); // prime: leftmost (smallest) ends up on top
        iter
    }

    pub fn next(&mut self) -> i32 {
        let node = *self.stk.pop().unwrap(); // next-smallest is always on top
        let right = node.right;
        Self::push_leftmost(&mut self.stk, right); // its successor lives at the leftmost of its right subtree
        node.val
    }

    pub fn has_next(&self) -> bool {
        !self.stk.is_empty()
    }
}
```

---

## Why amortized O(1) and O(h) space

**Space:** the stack only ever holds a single root-to-leaf-ish path (a node plus its pending left
ancestors), never more than the tree's height → **O(h)**. For a balanced tree that is O(log n).

**Time — amortized O(1) for `next()`:** a single `next()` can do up to O(h) work (when it pushes a
long left spine after stepping right). But **each node is pushed exactly once and popped exactly
once** over the entire iteration. Across `n` calls to `next()` the total work is O(n) pushes +
O(n) pops = **O(n)**, so the **amortized** cost per `next()` is O(n)/n = **O(1)**. `hasNext()` is a
single emptiness check — true O(1).

---

## Dry Run

```
        7
       / \
      3   15
         /  \
        9    20
```

- Constructor `pushLeftmost(7)`: push 7, then 3 (3 has no left). Stack (top→bottom): `[3, 7]`.
- `next()` → pop **3**; 3 has no right → nothing pushed. Stack `[7]`.
- `next()` → pop **7**; 7.right = 15 → `pushLeftmost(15)`: push 15, then 9. Stack `[9, 15]`.
- `next()` → pop **9**; no right. Stack `[15]`.
- `next()` → pop **15**; 15.right = 20 → push 20. Stack `[20]`.
- `next()` → pop **20**; no right. Stack `[]`.
- `hasNext()` → false.

Emitted: `3, 7, 9, 15, 20` — correct sorted order.

---

## Follow-up 1 — Prove the O(h) space bound rigorously

At any moment the stack contents are exactly the unvisited ancestors-on-the-left of the cursor:
when we push a node we immediately push its entire left spine, and we never push a node's left
children again after popping it (we only ever push a *right* subtree's left spine). The deepest the
stack can get is one full left spine, whose length ≤ tree height `h`. Hence **O(h)**, independent of
`n`. A naive design that flattens the whole tree into a list in the constructor would be O(n)
space — strictly worse and the reason the follow-up is asked.

---

## Follow-up 2 — Reverse iterator (`before()` / `prev()`)

A descending iterator mirrors everything: push the **rightmost** spine, and on each step move into
the popped node's **left** subtree.

```rust
pub struct BSTIteratorReverse {
    stk: Vec<Box<TreeNode>>,
}

impl BSTIteratorReverse {
    fn push_rightmost(stk: &mut Vec<Box<TreeNode>>, mut node: Option<Box<TreeNode>>) {
        while let Some(mut n) = node {
            node = n.right.take(); // detach right child so we can push n
            stk.push(n);
        }
    }

    pub fn new(root: Option<Box<TreeNode>>) -> Self {
        let mut iter = BSTIteratorReverse { stk: Vec::new() };
        Self::push_rightmost(&mut iter.stk, root); // largest ends up on top
        iter
    }

    pub fn prev(&mut self) -> i32 {                // returns next-largest
        let node = *self.stk.pop().unwrap();
        let left = node.left;
        Self::push_rightmost(&mut self.stk, left); // predecessor = rightmost of left subtree
        node.val
    }

    pub fn has_prev(&self) -> bool {
        !self.stk.is_empty()
    }
}
```

> **Bidirectional variant (LC 1586):** maintain both directions by re-priming the appropriate
> stack relative to the last-returned value, or by storing the inorder index. The forward/reverse
> stack templates above are the building blocks.

---

## Summary

| Operation | Time | Space |
|---|---|---|
| Constructor | O(h) (prime leftmost spine) | — |
| `next()` | amortized O(1) (worst-case O(h)) | — |
| `hasNext()` | O(1) | — |
| Overall | O(n) for full iteration | O(h) |

> **Last Updated:** 2026-06-26
