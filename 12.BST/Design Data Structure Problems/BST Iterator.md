> **Topic:** [Binary Search Trees](../README.md) · **Design Data Structure**

# BST Iterator (LC 173)

Implement an iterator over a BST that returns keys in **ascending (inorder)** order:

- `BSTIterator(TreeNode root)` — initialize.
- `int next()` — return the next-smallest key.
- `boolean hasNext()` — whether a next key exists.

**Both `next()` and `hasNext()` must be amortized O(1), using at most O(h) memory.**

```java
public class TreeNode {
    int val;
    TreeNode left, right;
    TreeNode(int val) { this.val = val; }
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

```java
import java.util.ArrayDeque;
import java.util.Deque;

class BSTIterator {
    private final Deque<TreeNode> stack = new ArrayDeque<>();

    public BSTIterator(TreeNode root) {
        pushLeftmost(root);              // prime: leftmost (smallest) ends up on top
    }

    public int next() {
        TreeNode node = stack.pop();     // next-smallest is always on top
        if (node.right != null) {
            pushLeftmost(node.right);    // its successor lives at the leftmost of its right subtree
        }
        return node.val;
    }

    public boolean hasNext() {
        return !stack.isEmpty();
    }

    private void pushLeftmost(TreeNode node) {
        while (node != null) {
            stack.push(node);
            node = node.left;
        }
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

```java
class BSTIteratorReverse {
    private final Deque<TreeNode> stack = new ArrayDeque<>();

    public BSTIteratorReverse(TreeNode root) {
        pushRightmost(root);             // largest ends up on top
    }

    public int prev() {                  // returns next-largest
        TreeNode node = stack.pop();
        if (node.left != null) {
            pushRightmost(node.left);    // predecessor = rightmost of left subtree
        }
        return node.val;
    }

    public boolean hasPrev() {
        return !stack.isEmpty();
    }

    private void pushRightmost(TreeNode node) {
        while (node != null) {
            stack.push(node);
            node = node.right;
        }
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
