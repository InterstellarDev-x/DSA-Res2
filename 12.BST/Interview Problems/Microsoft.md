> **Topic:** [Binary Search Trees](../README.md) · **Company 3 of 3**

# Microsoft — BST Interview Deep Dives

Microsoft interviews favor **clean implementation of the core operations** and solid **design** sense. The bar is less about an exotic trick and more about: can you write delete correctly (the case everyone fumbles), can you build a tree from a traversal in linear time, and can you design an iterator with the right amortized guarantees? They reward readable code, explicit edge-case handling, and clearly communicated tradeoffs.

The node type used throughout:

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

## 1. Delete Node in a BST (LC 450)

**Prompt.** Given the root of a BST and a key, delete the node with that key (if present) and return the new root, keeping the BST property intact.

**Approach discussion.** First locate the node by the usual BST descent. Deletion then splits into three cases:

1. **Leaf (no children):** detach it — return `None`.
2. **One child:** splice it out — return the non-null child.
3. **Two children:** the hard case. Replace the node's value with its **inorder successor** (the smallest value in the right subtree = the **leftmost node of the right subtree**), then delete that successor node from the right subtree. The successor has at most a right child, so its deletion falls into case 1 or 2 — no infinite recursion.

Why the successor works: it is the next-larger value, so substituting it preserves "everything left < node < everything right." (Symmetrically, the inorder predecessor — rightmost of the left subtree — also works.)

```rust
pub fn delete_node(root: Option<Box<TreeNode>>, key: i32) -> Option<Box<TreeNode>> {
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
                if node.left.is_none() { return node.right; }  // 0 or 1 child (right)
                if node.right.is_none() { return node.left; }  // 1 child (left)

                // two children: find inorder successor = leftmost of right subtree
                let mut succ = node.right.as_ref().unwrap();
                while succ.left.is_some() {
                    succ = succ.left.as_ref().unwrap();
                }
                let succ_val = succ.val;                             // copy successor value up
                node.val = succ_val;
                node.right = delete_node(node.right, succ_val);      // delete the successor copy
                Some(node)
            }
        }
    }
}
```

### Worked dry-run

Delete `50` from:

```
        50
       /  \
      30    70
     / \   /  \
    20 40 60  80
```

1. `key == root.val (50)`. Both children exist → two-children case.
2. Find the inorder successor: go right to `70`, then left as far as possible → `60`. (`60` is the smallest value greater than `50`.)
3. Copy `succ_val = 60` into the root. The tree now has `60` at the top *and* still a `60` in the right subtree (a temporary duplicate).
4. Recursively delete `60` from the right subtree. `60` is found as the left child of `70`; it has no left child, so case "one child / leaf" returns `60.right` (`None`). `70.left` becomes `None`.

Result:

```
        60
       /  \
      30    70
     / \      \
    20 40     80
```

The BST property holds, and `60` (the leftmost-of-right) had at most a right child, so its removal was trivial. **Complexity:** O(h) time, O(h) recursion space. See [BST Operations](../Patterns/BST%20Operations.md).

---

## 2. Construct BST from Preorder Traversal (LC 1008)

**Prompt.** Given the preorder traversal of a BST, reconstruct the tree and return its root.

**Approach discussion.** Preorder visits *root, then left subtree, then right subtree*. The naive approach finds a split point per node (O(n²) worst case). The optimal **O(n) upper-bound technique** processes each value exactly once: maintain a global index `i` and a recursion that is handed an **upper bound** for the current subtree. A value belongs to the current subtree only if it is below that bound; otherwise it belongs to some ancestor's right subtree, and we return without consuming it.

```rust
struct Solution {
    i: usize,
    pre: Vec<i32>,
}

impl Solution {
    fn build(&mut self, bound: i32) -> Option<Box<TreeNode>> {
        if self.i == self.pre.len() || self.pre[self.i] > bound {
            return None;
        }
        let val = self.pre[self.i];
        self.i += 1;                                    // consume this value
        let mut node = Box::new(TreeNode::new(val));
        node.left = self.build(val);                    // left subtree bounded by node
        node.right = self.build(bound);                 // right subtree keeps parent bound
        Some(node)
    }

    pub fn bst_from_preorder(preorder: Vec<i32>) -> Option<Box<TreeNode>> {
        let mut sol = Solution { i: 0, pre: preorder };
        sol.build(i32::MAX)
    }
}
```

**Why each element is consumed once.** The index `i` only ever advances — it increments exactly when a value is turned into a node, and never rewinds. Every recursive call either consumes the current value (creating a node and moving `i` forward) or returns immediately because `pre[i] > bound` (this value belongs higher up the tree). Since `i` goes from `0` to `n` monotonically and each step does O(1) work, the whole construction is O(n) time, O(h) recursion space.

The left child inherits the current node's value as its upper bound (everything left must be smaller); the right child inherits the *parent's* bound (everything right must be smaller than the ancestor that owns this right edge). See [BST Construction](../Patterns/BST%20Construction.md).

---

## 3. BST Iterator (LC 173)

**Prompt.** Implement an iterator over a BST that returns values in ascending (inorder) order. Support `next()` returning the next smallest value and `has_next()` returning whether a next value exists.

**Approach discussion.** This is **controlled inorder traversal** — we pause the walk between calls instead of running it to completion. Keep a stack pre-loaded with the path of left children from the current position. `next()` pops the top (the next smallest), then pushes the left spine of its right child. `has_next()` is just "stack not empty."

```rust
struct BSTIterator<'a> {
    stk: Vec<&'a TreeNode>,
}

impl<'a> BSTIterator<'a> {
    fn push_left(&mut self, node: Option<&'a Box<TreeNode>>) {
        let mut curr = node;
        while let Some(n) = curr {
            self.stk.push(n.as_ref());
            curr = n.left.as_ref();
        }
    }

    pub fn new(root: Option<&'a Box<TreeNode>>) -> Self {
        let mut iter = BSTIterator { stk: Vec::new() };
        iter.push_left(root);
        iter
    }

    pub fn next(&mut self) -> i32 {
        let node = self.stk.pop().unwrap();   // next smallest
        self.push_left(node.right.as_ref());  // queue up its right subtree's left spine
        node.val
    }

    pub fn has_next(&self) -> bool {
        !self.stk.is_empty()
    }
}
```

**Complexity.** `has_next()` is O(1). `next()` is **amortized O(1)**: although a single call may push a long left spine, each node is pushed and popped exactly once across the entire iteration, so n calls do O(n) total work. Space is **O(h)** — the stack never holds more than one root-to-leaf path. This beats flattening the tree into a list up front (which would be O(n) space).

For the full design write-up and variants, see [BST Iterator](../Design%20Data%20Structure%20Problems/BST%20Iterator.md).

---

## What Microsoft Looks For

Clean, idiomatic code that handles the edge cases without being asked — null roots, the two-children delete case, keys not present, and out-of-range indices. They want the *amortized* and *space* analysis stated correctly (not hand-waved), the inorder/BST invariant named explicitly, and tradeoffs communicated plainly: why successor-copy over predecessor, why a stack iterator beats pre-flattening, why the upper-bound construction is O(n). Readability and a calm, correct first implementation matter more here than reaching for the cleverest possible trick.

> **Last Updated:** 2026-06-26
