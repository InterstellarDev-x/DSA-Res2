> **Topic:** [Binary Search Trees](../README.md) · **Company 1 of 3**

# Amazon — BST Interview Deep Dives

Amazon's BST bar focuses on **correctness under edge cases** and clean **O(h) reasoning**. Interviewers want to see you actually *Dive Deep* into the invariants (what makes a tree a BST, what the inorder sequence guarantees) and *Insist on the Highest Standards* by handling the nasty boundary cases — duplicate values, `i32::MIN`/`i32::MAX` at the leaves, skewed trees — without being prompted. Expect a follow-up on every problem; the first solution is rarely the last word.

The node type used throughout:

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

---

## 1. Validate BST (LC 98)

**Prompt.** Given the root of a binary tree, determine if it is a valid BST. A valid BST requires every node in the left subtree to be strictly less than the node, every node in the right subtree to be strictly greater, and both subtrees themselves to be valid BSTs.

**Approach discussion.** The naive instinct — "left child < node < right child" checked locally — is wrong. A node deep in the right subtree can still violate an ancestor's bound. There are two correct strategies:

- **(low, high) bounds:** recurse carrying the open interval the current node must lie within. Tightens going down.
- **Strictly-increasing inorder:** an inorder walk of a valid BST yields a strictly increasing sequence. Validate by checking each value exceeds the previous.

### 1a. The (low, high) bounds approach — and the overflow trap

The obvious signature is `fn valid(node: &Option<Box<TreeNode>>, low: i32, high: i32)` seeded with `i32::MIN` / `i32::MAX`. **This is a trap.** If the tree legitimately contains a node with value `i32::MIN` as a left-most leaf, comparing `n.val > low` with `low == i32::MIN` fails for a valid node (since `i32::MIN > i32::MIN` is false). The same happens at the high end with `i32::MAX`. Using `i32` sentinels means you cannot represent "no bound" distinct from a real extreme value.

**Fix:** use `i64` bounds so the sentinels `i64::MIN` / `i64::MAX` sit strictly outside the `i32` range and can never collide with a real node value.

```rust
fn valid(node: &Option<Box<TreeNode>>, low: i64, high: i64) -> bool {
    match node {
        None => true,
        Some(n) => {
            if (n.val as i64) <= low || (n.val as i64) >= high {
                return false; // strict bounds
            }
            valid(&n.left, low, n.val as i64) && valid(&n.right, n.val as i64, high)
        }
    }
}

fn is_valid_bst(root: &Option<Box<TreeNode>>) -> bool {
    valid(root, i64::MIN, i64::MAX)
}
```

The left subtree inherits the current node as its new *high*; the right subtree inherits it as its new *low*. Strict `<=` / `>=` enforces "no duplicates."

### 1b. The strictly-increasing inorder approach

```rust
fn is_valid_bst(root: &Option<Box<TreeNode>>) -> bool {
    let mut stk: Vec<&TreeNode> = Vec::new();
    let mut curr: Option<&TreeNode> = root.as_deref();
    let mut prev: i64 = i64::MIN; // i64 avoids the MIN_VALUE leaf trap

    loop {
        while let Some(node) = curr {
            stk.push(node);
            curr = node.left.as_deref();
        }
        match stk.pop() {
            None => break,
            Some(node) => {
                if node.val as i64 <= prev {
                    return false; // must strictly increase
                }
                prev = node.val as i64;
                curr = node.right.as_deref();
            }
        }
    }
    true
}
```

Here too `prev` is an `i64` so the very first node can be `i32::MIN` without a false negative.

**When each is preferred.** The bounds approach is the cleaner mental model and is trivially adaptable when you also need to *locate* where a violation occurs. The inorder approach generalizes better — it is the same skeleton you reuse for Kth Smallest and Recover BST, and it makes the "BST ⇔ sorted inorder" invariant explicit, which interviewers like to hear stated out loud.

**Complexity.** Both are O(n) time. Bounds recursion is O(h) stack; iterative inorder is O(h) explicit stack. See [BST Validation and Inorder](../Patterns/BST%20Validation%20and%20Inorder.md).

**Interviewer follow-up.** *"What if duplicates are allowed and should sit in the right subtree?"* — relax the right-bound comparison to allow equality on one side consistently (e.g. `n.val as i64 < low` instead of `<=`), and decide which side duplicates go to up front. State the chosen convention explicitly.

---

## 2. Kth Smallest Element (LC 230)

**Prompt.** Given the root of a BST and an integer `k`, return the kth smallest value (1-indexed).

**Approach discussion.** Inorder traversal visits nodes in ascending order, so the kth node popped is the answer. Iterative stack inorder lets us **stop early** the moment we have counted `k` nodes — no need to traverse the rest of the tree.

```rust
fn kth_smallest(root: &Option<Box<TreeNode>>, mut k: i32) -> i32 {
    let mut stk: Vec<&TreeNode> = Vec::new();
    let mut curr: Option<&TreeNode> = root.as_deref();

    loop {
        while let Some(node) = curr {
            stk.push(node);
            curr = node.left.as_deref();
        }
        match stk.pop() {
            None => break,
            Some(node) => {
                k -= 1;
                if k == 0 {
                    return node.val; // early stop
                }
                curr = node.right.as_deref();
            }
        }
    }
    -1 // k out of range
}
```

**Complexity.** O(h + k) time — we descend to the smallest leaf (O(h)) then pop k nodes — and O(h) space.

**Interviewer follow-up.** *"What if the BST is modified often (frequent inserts/deletes) and kth queries are also frequent?"* Re-running an O(h + k) inorder per query is wasteful. **Augment each node with the size of its left subtree** (equivalently, total subtree size) so each query becomes a pure O(h) descent — no traversal of the whole prefix.

Decision at each node, given target rank `k`:
- Let `left_count` = size of the left subtree.
- If `k == left_count + 1`, this node is the answer.
- If `k <= left_count`, recurse left with the same `k`.
- Otherwise recurse right with `k - left_count - 1`.

```rust
#[derive(Debug, Clone)]
pub struct CountedNode {
    pub val: i32,
    pub left_count: i32,         // number of nodes in the left subtree
    pub left: Option<Box<CountedNode>>,
    pub right: Option<Box<CountedNode>>,
}

impl CountedNode {
    pub fn new(val: i32) -> Self {
        CountedNode { val, left_count: 0, left: None, right: None }
    }
}

fn kth_smallest_counted(root: &Option<Box<CountedNode>>, mut k: i32) -> i32 {
    let mut curr: Option<&CountedNode> = root.as_deref();
    while let Some(node) = curr {
        let left_size = node.left_count;
        if k == left_size + 1 {
            return node.val;
        }
        if k <= left_size {
            curr = node.left.as_deref();
        } else {
            k -= left_size + 1;
            curr = node.right.as_deref();
        }
    }
    -1
}
```

**Maintaining the counts on insert/delete.** On the way *down* an insert, every time you step into a node's **left** child you increment that node's `left_count` (a new node will land in its left subtree):

```rust
fn insert(root: Option<Box<CountedNode>>, val: i32) -> Option<Box<CountedNode>> {
    match root {
        None => Some(Box::new(CountedNode::new(val))),
        Some(mut node) => {
            if val < node.val {
                node.left_count += 1; // a node is being added to the left subtree
                node.left = insert(node.left.take(), val);
            } else {
                node.right = insert(node.right.take(), val);
            }
            Some(node)
        }
    }
}
```

On **delete**, mirror it: decrement `left_count` on every node whose left subtree shrinks (i.e. when the target is found in the left subtree). Both operations stay O(h) because the count update is a single arithmetic op per level. This is the classic *order-statistic tree* augmentation; pair it with a self-balancing tree (AVL/Red-Black) to guarantee O(log n). See [BST Operations](../Patterns/BST%20Operations.md).

---

## 3. Lowest Common Ancestor of a BST (LC 235)

**Prompt.** Given a BST and two nodes `p` and `q` present in it, return their lowest common ancestor.

**Approach discussion.** In a *general* binary tree, LCA requires searching both subtrees and is O(n) (see [BST LCA and Ancestors](../Patterns/BST%20LCA%20and%20Ancestors.md)). In a **BST** we exploit ordering: the LCA is the first node where `p` and `q` fall on opposite sides (or one of them equals the node). Walk down once — when both values are smaller, go left; both larger, go right; otherwise we are at the split point. This is O(h) time and O(1) space, no recursion needed.

```rust
fn lowest_common_ancestor<'a>(
    root: &'a Option<Box<TreeNode>>,
    p: &TreeNode,
    q: &TreeNode,
) -> Option<&'a TreeNode> {
    let mut curr: Option<&TreeNode> = root.as_deref();
    while let Some(node) = curr {
        if p.val < node.val && q.val < node.val {
            curr = node.left.as_deref();
        } else if p.val > node.val && q.val > node.val {
            curr = node.right.as_deref();
        } else {
            return Some(node); // split point: p and q diverge here, or one is curr
        }
    }
    None
}
```

**Contrast.** The general-tree LCA must potentially visit every node because there is no ordering to prune subtrees, giving O(n) time and O(h) recursion stack. The BST version prunes half the tree at each step, so it never exceeds the height of the tree and uses constant extra space.

**Complexity.** O(h) time, O(1) space.

---

## Leadership Principle Alignment

| Leadership Principle | How it shows up in a BST interview |
| --- | --- |
| **Dive Deep** | Stating the BST invariant precisely, recognizing the `i32::MIN` bounds trap, deriving the order-statistic augmentation rather than re-traversing. |
| **Customer Obsession** | Choosing the data-structure variant that matches the *real* query pattern (augmented counts when kth queries are hot) instead of the textbook default. |
| **Insist on the Highest Standards** | Strict vs non-strict comparisons handled deliberately, duplicate-value policy declared, early-stop in Kth Smallest, no off-by-one in rank arithmetic. |
| **Bias for Action** | Reaching the clean O(h) iterative LCA quickly, then iterating to the follow-up solution without stalling. |

> **Last Updated:** 2026-06-26
