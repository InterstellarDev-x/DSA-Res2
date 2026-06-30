> **Topic:** [Binary Search Trees](../README.md) · **Company 1 of 3**

# Amazon — BST Interview Deep Dives

Amazon's BST bar focuses on **correctness under edge cases** and clean **O(h) reasoning**. Interviewers want to see you actually *Dive Deep* into the invariants (what makes a tree a BST, what the inorder sequence guarantees) and *Insist on the Highest Standards* by handling the nasty boundary cases — duplicate values, `Integer.MIN_VALUE`/`MAX_VALUE` at the leaves, skewed trees — without being prompted. Expect a follow-up on every problem; the first solution is rarely the last word.

The node type used throughout:

```java
public class TreeNode {
    int val;
    TreeNode left, right;
    TreeNode(int val) { this.val = val; }
}
```

---

## 1. Validate BST (LC 98)

**Prompt.** Given the root of a binary tree, determine if it is a valid BST. A valid BST requires every node in the left subtree to be strictly less than the node, every node in the right subtree to be strictly greater, and both subtrees themselves to be valid BSTs.

**Approach discussion.** The naive instinct — "left child < node < right child" checked locally — is wrong. A node deep in the right subtree can still violate an ancestor's bound. There are two correct strategies:

- **(low, high) bounds:** recurse carrying the open interval the current node must lie within. Tightens going down.
- **Strictly-increasing inorder:** an inorder walk of a valid BST yields a strictly increasing sequence. Validate by checking each value exceeds the previous.

### 1a. The (low, high) bounds approach — and the overflow trap

The obvious signature is `boolean valid(TreeNode node, int low, int high)` seeded with `Integer.MIN_VALUE` / `Integer.MAX_VALUE`. **This is a trap.** If the tree legitimately contains a node with value `Integer.MIN_VALUE` as a left-most leaf, comparing `node.val > low` with `low == Integer.MIN_VALUE` fails for a valid node (since `MIN_VALUE > MIN_VALUE` is false). The same happens at the high end with `Integer.MAX_VALUE`. Using `int` sentinels means you cannot represent "no bound" distinct from a real extreme value.

**Fix:** use `long` bounds so the sentinels `Long.MIN_VALUE` / `Long.MAX_VALUE` sit strictly outside the `int` range and can never collide with a real node value.

```java
public boolean isValidBST(TreeNode root) {
    return valid(root, Long.MIN_VALUE, Long.MAX_VALUE);
}

private boolean valid(TreeNode node, long low, long high) {
    if (node == null) return true;
    if (node.val <= low || node.val >= high) return false;   // strict bounds
    return valid(node.left, low, node.val)
        && valid(node.right, node.val, high);
}
```

The left subtree inherits the current node as its new *high*; the right subtree inherits it as its new *low*. Strict `<=` / `>=` enforces "no duplicates."

### 1b. The strictly-increasing inorder approach

```java
public boolean isValidBST(TreeNode root) {
    Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode curr = root;
    long prev = Long.MIN_VALUE;     // long avoids the MIN_VALUE leaf trap

    while (curr != null || !stack.isEmpty()) {
        while (curr != null) {
            stack.push(curr);
            curr = curr.left;
        }
        curr = stack.pop();
        if (curr.val <= prev) return false;   // must strictly increase
        prev = curr.val;
        curr = curr.right;
    }
    return true;
}
```

Here too `prev` is a `long` so the very first node can be `Integer.MIN_VALUE` without a false negative.

**When each is preferred.** The bounds approach is the cleaner mental model and is trivially adaptable when you also need to *locate* where a violation occurs. The inorder approach generalizes better — it is the same skeleton you reuse for Kth Smallest and Recover BST, and it makes the "BST ⇔ sorted inorder" invariant explicit, which interviewers like to hear stated out loud.

**Complexity.** Both are O(n) time. Bounds recursion is O(h) stack; iterative inorder is O(h) explicit stack. See [BST Validation and Inorder](../Patterns/BST%20Validation%20and%20Inorder.md).

**Interviewer follow-up.** *"What if duplicates are allowed and should sit in the right subtree?"* — relax the right-bound comparison to allow equality on one side consistently (e.g. `node.val < low` instead of `<=`), and decide which side duplicates go to up front. State the chosen convention explicitly.

---

## 2. Kth Smallest Element (LC 230)

**Prompt.** Given the root of a BST and an integer `k`, return the kth smallest value (1-indexed).

**Approach discussion.** Inorder traversal visits nodes in ascending order, so the kth node popped is the answer. Iterative stack inorder lets us **stop early** the moment we have counted `k` nodes — no need to traverse the rest of the tree.

```java
public int kthSmallest(TreeNode root, int k) {
    Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode curr = root;

    while (curr != null || !stack.isEmpty()) {
        while (curr != null) {
            stack.push(curr);
            curr = curr.left;
        }
        curr = stack.pop();
        if (--k == 0) return curr.val;   // early stop
        curr = curr.right;
    }
    return -1;   // k out of range
}
```

**Complexity.** O(h + k) time — we descend to the smallest leaf (O(h)) then pop k nodes — and O(h) space.

**Interviewer follow-up.** *"What if the BST is modified often (frequent inserts/deletes) and kth queries are also frequent?"* Re-running an O(h + k) inorder per query is wasteful. **Augment each node with the size of its left subtree** (equivalently, total subtree size) so each query becomes a pure O(h) descent — no traversal of the whole prefix.

Decision at each node, given target rank `k`:
- Let `leftCount` = size of the left subtree.
- If `k == leftCount + 1`, this node is the answer.
- If `k <= leftCount`, recurse left with the same `k`.
- Otherwise recurse right with `k - leftCount - 1`.

```java
class CountedNode {
    int val;
    int leftCount;          // number of nodes in the left subtree
    CountedNode left, right;
    CountedNode(int val) { this.val = val; }
}

public int kthSmallest(CountedNode root, int k) {
    CountedNode curr = root;
    while (curr != null) {
        int leftSize = curr.leftCount;
        if (k == leftSize + 1) return curr.val;
        if (k <= leftSize) {
            curr = curr.left;
        } else {
            k -= leftSize + 1;
            curr = curr.right;
        }
    }
    return -1;
}
```

**Maintaining the counts on insert/delete.** On the way *down* an insert, every time you step into a node's **left** child you increment that node's `leftCount` (a new node will land in its left subtree):

```java
public CountedNode insert(CountedNode root, int val) {
    if (root == null) return new CountedNode(val);
    if (val < root.val) {
        root.leftCount++;            // a node is being added to the left subtree
        root.left = insert(root.left, val);
    } else {
        root.right = insert(root.right, val);
    }
    return root;
}
```

On **delete**, mirror it: decrement `leftCount` on every node whose left subtree shrinks (i.e. when the target is found in the left subtree). Both operations stay O(h) because the count update is a single arithmetic op per level. This is the classic *order-statistic tree* augmentation; pair it with a self-balancing tree (AVL/Red-Black) to guarantee O(log n). See [BST Operations](../Patterns/BST%20Operations.md).

---

## 3. Lowest Common Ancestor of a BST (LC 235)

**Prompt.** Given a BST and two nodes `p` and `q` present in it, return their lowest common ancestor.

**Approach discussion.** In a *general* binary tree, LCA requires searching both subtrees and is O(n) (see [BST LCA and Ancestors](../Patterns/BST%20LCA%20and%20Ancestors.md)). In a **BST** we exploit ordering: the LCA is the first node where `p` and `q` fall on opposite sides (or one of them equals the node). Walk down once — when both values are smaller, go left; both larger, go right; otherwise we are at the split point. This is O(h) time and O(1) space, no recursion needed.

```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    TreeNode curr = root;
    while (curr != null) {
        if (p.val < curr.val && q.val < curr.val) {
            curr = curr.left;
        } else if (p.val > curr.val && q.val > curr.val) {
            curr = curr.right;
        } else {
            return curr;     // split point: p and q diverge here, or one is curr
        }
    }
    return null;
}
```

**Contrast.** The general-tree LCA must potentially visit every node because there is no ordering to prune subtrees, giving O(n) time and O(h) recursion stack. The BST version prunes half the tree at each step, so it never exceeds the height of the tree and uses constant extra space.

**Complexity.** O(h) time, O(1) space.

---

## Leadership Principle Alignment

| Leadership Principle | How it shows up in a BST interview |
| --- | --- |
| **Dive Deep** | Stating the BST invariant precisely, recognizing the `Integer.MIN_VALUE` bounds trap, deriving the order-statistic augmentation rather than re-traversing. |
| **Customer Obsession** | Choosing the data-structure variant that matches the *real* query pattern (augmented counts when kth queries are hot) instead of the textbook default. |
| **Insist on the Highest Standards** | Strict vs non-strict comparisons handled deliberately, duplicate-value policy declared, early-stop in Kth Smallest, no off-by-one in rank arithmetic. |
| **Bias for Action** | Reaching the clean O(h) iterative LCA quickly, then iterating to the follow-up solution without stalling. |

> **Last Updated:** 2026-06-26
