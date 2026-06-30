> **Topic:** [Binary Search Trees](../README.md) · **Company 2 of 3**

# Google — BST Interview Deep Dives

Google interviews push on **optimal space** and **deep, layered follow-ups**, especially at the L4/L5 level. A working O(n)-space inorder solution is the *opening move*, not the answer — the interviewer will ask you to shave it to O(1), to handle repeated queries efficiently, or to reason about invariants out loud. The signal they hunt for is whether you reach for the harder, tighter technique (Morris traversal, subtree augmentation) on your own and can defend its tradeoffs.

The node type used throughout:

```java
public class TreeNode {
    int val;
    TreeNode left, right;
    TreeNode(int val) { this.val = val; }
}
```

---

## 1. Recover BST (LC 99)

**Prompt.** Exactly two nodes of a BST have been swapped by mistake. Recover the tree by restoring it without changing its structure.

**Approach discussion.** A valid BST has a strictly increasing inorder sequence. Swapping two nodes creates either one or two "descents" (a place where `prev.val > curr.val`):

- **Non-adjacent swap** → two descents. The first offending node is `prev` at the *first* descent; the second offending node is `curr` at the *second* descent.
- **Adjacent swap** → one descent. Both offending nodes come from that single descent: `first = prev`, `second = curr`.

Track three pointers during inorder: `first`, `second`, and a rolling `prev`. At each descent, if `first` is still null set `first = prev`; always set `second = curr`. After the walk, swap the two values.

```java
public void recoverTree(TreeNode root) {
    Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode curr = root, prev = null;
    TreeNode first = null, second = null;

    while (curr != null || !stack.isEmpty()) {
        while (curr != null) {
            stack.push(curr);
            curr = curr.left;
        }
        curr = stack.pop();
        if (prev != null && prev.val > curr.val) {   // a descent
            if (first == null) first = prev;          // set once
            second = curr;                            // always update
        }
        prev = curr;
        curr = curr.right;
    }
    if (first != null && second != null) {
        int t = first.val; first.val = second.val; second.val = t;
    }
}
```

This is O(n) time, O(h) space for the stack. See [BST Validation and Inorder](../Patterns/BST%20Validation%20and%20Inorder.md).

### Morris traversal follow-up — O(1) space

**The L5 follow-up:** *"Can you do it in constant space?"* Morris inorder threads the tree: before descending left, it links the **rightmost node of the left subtree** (the inorder predecessor) back to the current node. That temporary thread replaces the stack; we tear it down when we return through it, leaving the tree unchanged.

```java
public void recoverTree(TreeNode root) {
    TreeNode curr = root, prev = null;          // prev = last visited in inorder
    TreeNode first = null, second = null;

    while (curr != null) {
        if (curr.left == null) {
            // visit curr
            if (prev != null && prev.val > curr.val) {
                if (first == null) first = prev;
                second = curr;
            }
            prev = curr;
            curr = curr.right;
        } else {
            // find inorder predecessor: rightmost node of left subtree
            TreeNode pred = curr.left;
            while (pred.right != null && pred.right != curr) {
                pred = pred.right;
            }
            if (pred.right == null) {
                pred.right = curr;      // create thread, then go left
                curr = curr.left;
            } else {
                pred.right = null;      // thread already there: tear it down
                // visit curr
                if (prev != null && prev.val > curr.val) {
                    if (first == null) first = prev;
                    second = curr;
                }
                prev = curr;
                curr = curr.right;
            }
        }
    }
    if (first != null && second != null) {
        int t = first.val; first.val = second.val; second.val = t;
    }
}
```

**Why it's O(1) space.** No stack, no recursion — only the threaded `right` pointers, each created and then removed exactly once, so the tree is structurally identical at the end. Time is still O(n): each edge is traversed at most twice (once to set the thread, once to remove it).

---

## 2. Kth Smallest with Subtree-Size Augmentation (LC 230 deep dive)

**Prompt / framing.** Return the kth smallest, but assume kth queries are *frequent* and the tree may also change. The plain O(h + k) inorder is fine once; under repeated queries it is wasteful. The L4/L5 expectation is to **design for repeated queries** by augmenting each node with the size of its subtree (or just its left subtree).

Store `count` = total number of nodes in the subtree rooted at this node. The left subtree's size is then `size(node.left)`, and rank decisions are pure arithmetic.

```java
class CountedNode {
    int val;
    int count;                 // size of the subtree rooted here (incl. this node)
    CountedNode left, right;
    CountedNode(int val) { this.val = val; this.count = 1; }
}

private int size(CountedNode n) { return n == null ? 0 : n.count; }
```

### kthSmallest using counts — O(h)

```java
public int kthSmallest(CountedNode root, int k) {
    CountedNode curr = root;
    while (curr != null) {
        int leftSize = size(curr.left);
        if (k == leftSize + 1) return curr.val;     // this node has rank leftSize+1
        if (k <= leftSize) {
            curr = curr.left;                       // answer is in the left subtree
        } else {
            k -= leftSize + 1;                      // skip left subtree + this node
            curr = curr.right;
        }
    }
    return -1;     // k out of range
}
```

### Insert maintaining `count` — O(h)

Every node on the insertion path gains one descendant, so increment `count` as you unwind (or on the way down). Doing it on the recursive return keeps it clean:

```java
public CountedNode insert(CountedNode root, int val) {
    if (root == null) return new CountedNode(val);
    if (val < root.val) {
        root.left = insert(root.left, val);
    } else {
        root.right = insert(root.right, val);
    }
    root.count = 1 + size(root.left) + size(root.right);   // recompute from children
    return root;
}
```

Recomputing `count` from the children's sizes is the most robust pattern — it stays correct even if you later add rotations for balancing. Each query and each update is O(h); with a balanced tree that is O(log n). See [BST Operations](../Patterns/BST%20Operations.md).

**Tradeoff to articulate.** Augmentation adds one `int` per node and a small constant of bookkeeping on every mutation. You only take that on if reads dominate or both reads and writes are frequent. If kth is queried once on a static tree, the plain inorder wins on simplicity.

---

## 3. Inorder Successor in a BST (LC 285)

**Prompt.** Given a BST root and a node `p`, return the node with the smallest value greater than `p.val`, or null if none.

**Approach discussion.** Two cases, both resolved in a single O(h) descent:

- **`p` has a right subtree** → the successor is the **leftmost node of that right subtree**.
- **`p` has no right subtree** → the successor is the **lowest ancestor for which `p` is in the left subtree** — i.e. the last node where we turned *left* while searching for `p`. We capture this candidate during the descent.

The elegant single-descent version handles both without needing parent pointers: walk down from the root toward `p`, and every time we move *left* we record the current node as the best successor candidate so far.

```java
public TreeNode inorderSuccessor(TreeNode root, TreeNode p) {
    TreeNode succ = null;
    TreeNode curr = root;
    while (curr != null) {
        if (p.val < curr.val) {
            succ = curr;            // candidate: we turned left here
            curr = curr.left;
        } else {
            curr = curr.right;      // p.val >= curr.val: successor is further right
        }
    }
    return succ;
}
```

When `p` has a right subtree, the descent naturally moves right at `p` and the last left-turn captured is the correct ancestor (or the leftmost-of-right is reached); when it has none, the last recorded left-turn ancestor is returned. This unifies both cases in O(h) time, O(1) space.

**Parent-pointer variant (LC 510), briefly.** If each node carries a `parent` pointer and you are *given the node itself* (not the root): if it has a right child, return the leftmost descendant of that right child; otherwise climb parents until you come up from a *left* child, and return that parent (or null at the root). Same O(h), still O(1) extra space.

---

## L4 / L5 Signal

What separates a strong senior candidate from a mid-level one on these problems:

- **Reaches for Morris unprompted.** Mid-level: solves Recover BST with a stack and stops. Senior: anticipates "constant space?" and produces the threaded traversal, correctly tearing down threads so the tree is left intact.
- **Reasons about augmentation tradeoffs.** Senior candidates do not just bolt on `count` — they explain the per-mutation cost, the read/write ratio that justifies it, and that recomputing from children sizes survives future rebalancing. They connect it to order-statistic trees.
- **States invariants cleanly.** "Inorder of a BST is strictly increasing" is named explicitly and used to justify both Validate, Recover, and Kth Smallest. The number of descents (one vs two) in Recover is reasoned about, not guessed.
- **Unifies cases.** The single-descent successor that handles both the right-subtree and no-right-subtree cases — rather than two branches stitched together — is a strong L5 signal of comfort with the structure.
- **Clean boundary handling.** `long` bounds, strict comparisons, null roots, and k out of range are addressed before being asked.

> **Last Updated:** 2026-06-26
