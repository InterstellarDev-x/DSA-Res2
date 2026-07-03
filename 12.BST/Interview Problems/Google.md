> **Topic:** [Binary Search Trees](../README.md) · **Company 2 of 3**

# Google — BST Interview Deep Dives

Google interviews push on **optimal space** and **deep, layered follow-ups**, especially at the L4/L5 level. A working O(n)-space inorder solution is the *opening move*, not the answer — the interviewer will ask you to shave it to O(1), to handle repeated queries efficiently, or to reason about invariants out loud. The signal they hunt for is whether you reach for the harder, tighter technique (Morris traversal, subtree augmentation) on your own and can defend its tradeoffs.

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

## 1. Recover BST (LC 99)

**Prompt.** Exactly two nodes of a BST have been swapped by mistake. Recover the tree by restoring it without changing its structure.

**Approach discussion.** A valid BST has a strictly increasing inorder sequence. Swapping two nodes creates either one or two "descents" (a place where `prev.val > curr.val`):

- **Non-adjacent swap** → two descents. The first offending node is `prev` at the *first* descent; the second offending node is `curr` at the *second* descent.
- **Adjacent swap** → one descent. Both offending nodes come from that single descent: `first = prev`, `second = curr`.

Track three pointers during inorder: `first`, `second`, and a rolling `prev`. At each descent, if `first` is still `None` set `first = prev`; always set `second = curr`. After the walk, swap the two values.

```rust
fn recover_tree(root: &mut Option<Box<TreeNode>>) {
    let mut stk: Vec<*mut TreeNode> = Vec::new();
    let mut curr: *mut TreeNode = root
        .as_deref_mut()
        .map_or(std::ptr::null_mut(), |n| n as *mut TreeNode);
    let mut prev: *mut TreeNode = std::ptr::null_mut();
    let mut first: *mut TreeNode = std::ptr::null_mut();
    let mut second: *mut TreeNode = std::ptr::null_mut();

    // SAFETY: We hold exclusive access to the tree via the &mut root parameter.
    // Raw pointers are derived from Box-managed nodes and never outlive the tree.
    unsafe {
        while !curr.is_null() || !stk.is_empty() {
            while !curr.is_null() {
                stk.push(curr);
                curr = (*curr).left.as_deref_mut()
                    .map_or(std::ptr::null_mut(), |n| n as *mut TreeNode);
            }
            curr = stk.pop().unwrap();
            if !prev.is_null() && (*prev).val > (*curr).val {   // a descent
                if first.is_null() { first = prev; }            // set once
                second = curr;                                  // always update
            }
            prev = curr;
            curr = (*curr).right.as_deref_mut()
                .map_or(std::ptr::null_mut(), |n| n as *mut TreeNode);
        }
        if !first.is_null() && !second.is_null() {
            std::mem::swap(&mut (*first).val, &mut (*second).val);
        }
    }
}
```

This is O(n) time, O(h) space for the stack. See [BST Validation and Inorder](../Patterns/BST%20Validation%20and%20Inorder.md).

### Morris traversal follow-up — O(1) space

**The L5 follow-up:** *"Can you do it in constant space?"* Morris inorder threads the tree: before descending left, it links the **rightmost node of the left subtree** (the inorder predecessor) back to the current node. That temporary thread replaces the stack; we tear it down when we return through it, leaving the tree unchanged.

```rust
// Morris traversal requires temporarily creating back-pointer "threads" in the tree,
// which requires unsafe code in Rust. We use raw *mut TreeNode pointers derived from
// the Box-managed nodes. Box::from_raw / Box::into_raw are used in pairs to store
// threads in Option<Box<TreeNode>> fields without triggering memory deallocation.
fn recover_tree_morris(root: &mut Option<Box<TreeNode>>) {
    let mut curr: *mut TreeNode = root
        .as_deref_mut()
        .map_or(std::ptr::null_mut(), |n| n as *mut TreeNode);
    let mut prev: *mut TreeNode = std::ptr::null_mut();  // prev = last visited in inorder
    let mut first: *mut TreeNode = std::ptr::null_mut();
    let mut second: *mut TreeNode = std::ptr::null_mut();

    // SAFETY: We hold exclusive access via &mut root. All raw pointers are derived
    // from the same Box tree. Box::from_raw / Box::into_raw are used in pairs to
    // temporarily thread pointers; no double-free occurs because we reclaim with
    // Box::into_raw before the original owner could drop.
    unsafe {
        while !curr.is_null() {
            if (*curr).left.is_none() {
                // visit curr
                if !prev.is_null() && (*prev).val > (*curr).val {
                    if first.is_null() { first = prev; }
                    second = curr;
                }
                prev = curr;
                curr = (*curr).right.as_deref_mut()
                    .map_or(std::ptr::null_mut(), |n| n as *mut TreeNode);
            } else {
                // find inorder predecessor: rightmost node of left subtree
                let mut pred: *mut TreeNode = (*curr).left.as_deref_mut()
                    .map_or(std::ptr::null_mut(), |n| n as *mut TreeNode);
                loop {
                    let r = (*pred).right.as_deref_mut()
                        .map_or(std::ptr::null_mut(), |n| n as *mut TreeNode);
                    if r.is_null() || r == curr { break; }
                    pred = r;
                }
                let thread_exists = (*pred).right.as_deref()
                    .map(|n| n as *const TreeNode == curr as *const TreeNode)
                    .unwrap_or(false);
                if !thread_exists {
                    (*pred).right = Some(Box::from_raw(curr));  // create thread, then go left
                    curr = (*curr).left.as_deref_mut()
                        .map_or(std::ptr::null_mut(), |n| n as *mut TreeNode);
                } else {
                    // thread already there: tear it down
                    if let Some(b) = (*pred).right.take() {
                        let _ = Box::into_raw(b);               // reclaim raw ptr, no free
                    }
                    // visit curr
                    if !prev.is_null() && (*prev).val > (*curr).val {
                        if first.is_null() { first = prev; }
                        second = curr;
                    }
                    prev = curr;
                    curr = (*curr).right.as_deref_mut()
                        .map_or(std::ptr::null_mut(), |n| n as *mut TreeNode);
                }
            }
        }
        if !first.is_null() && !second.is_null() {
            std::mem::swap(&mut (*first).val, &mut (*second).val);
        }
    }
}
```

**Why it's O(1) space.** No stack, no recursion — only the threaded `right` pointers, each created and then removed exactly once, so the tree is structurally identical at the end. Time is still O(n): each edge is traversed at most twice (once to set the thread, once to remove it).

---

## 2. Kth Smallest with Subtree-Size Augmentation (LC 230 deep dive)

**Prompt / framing.** Return the kth smallest, but assume kth queries are *frequent* and the tree may also change. The plain O(h + k) inorder is fine once; under repeated queries it is wasteful. The L4/L5 expectation is to **design for repeated queries** by augmenting each node with the size of its subtree (or just its left subtree).

Store `count` = total number of nodes in the subtree rooted at this node. The left subtree's size is then `size(node.left.as_deref())`, and rank decisions are pure arithmetic.

```rust
#[derive(Debug)]
pub struct CountedNode {
    pub val: i32,
    pub count: i32,                // size of the subtree rooted here (incl. this node)
    pub left: Option<Box<CountedNode>>,
    pub right: Option<Box<CountedNode>>,
}

impl CountedNode {
    pub fn new(val: i32) -> Self {
        CountedNode { val, count: 1, left: None, right: None }
    }
}

fn size(n: Option<&CountedNode>) -> i32 {
    n.map_or(0, |node| node.count)
}
```

### kthSmallest using counts — O(h)

```rust
fn kth_smallest(root: Option<&CountedNode>, mut k: i32) -> i32 {
    let mut curr = root;
    while let Some(node) = curr {
        let left_size = node.left.as_deref().map_or(0, |n| n.count);
        if k == left_size + 1 { return node.val; }     // this node has rank leftSize+1
        if k <= left_size {
            curr = node.left.as_deref();               // answer is in the left subtree
        } else {
            k -= left_size + 1;                        // skip left subtree + this node
            curr = node.right.as_deref();
        }
    }
    -1     // k out of range
}
```

### Insert maintaining `count` — O(h)

Every node on the insertion path gains one descendant, so increment `count` as you unwind (or on the way down). Doing it on the recursive return keeps it clean:

```rust
fn insert(root: Option<Box<CountedNode>>, val: i32) -> Option<Box<CountedNode>> {
    match root {
        None => Some(Box::new(CountedNode::new(val))),
        Some(mut node) => {
            if val < node.val {
                node.left = insert(node.left.take(), val);
            } else {
                node.right = insert(node.right.take(), val);
            }
            node.count = 1 + node.left.as_deref().map_or(0, |n| n.count)
                           + node.right.as_deref().map_or(0, |n| n.count);  // recompute from children
            Some(node)
        }
    }
}
```

Recomputing `count` from the children's sizes is the most robust pattern — it stays correct even if you later add rotations for balancing. Each query and each update is O(h); with a balanced tree that is O(log n). See [BST Operations](../Patterns/BST%20Operations.md).

**Tradeoff to articulate.** Augmentation adds one `i32` per node and a small constant of bookkeeping on every mutation. You only take that on if reads dominate or both reads and writes are frequent. If kth is queried once on a static tree, the plain inorder wins on simplicity.

---

## 3. Inorder Successor in a BST (LC 285)

**Prompt.** Given a BST root and a node `p`, return the node with the smallest value greater than `p.val`, or `None` if none.

**Approach discussion.** Two cases, both resolved in a single O(h) descent:

- **`p` has a right subtree** → the successor is the **leftmost node of that right subtree**.
- **`p` has no right subtree** → the successor is the **lowest ancestor for which `p` is in the left subtree** — i.e. the last node where we turned *left* while searching for `p`. We capture this candidate during the descent.

The elegant single-descent version handles both without needing parent pointers: walk down from the root toward `p`, and every time we move *left* we record the current node as the best successor candidate so far.

```rust
fn inorder_successor<'a>(root: Option<&'a TreeNode>, p: &TreeNode) -> Option<&'a TreeNode> {
    let mut succ: Option<&TreeNode> = None;
    let mut curr = root;
    while let Some(node) = curr {
        if p.val < node.val {
            succ = Some(node);              // candidate: we turned left here
            curr = node.left.as_deref();
        } else {
            curr = node.right.as_deref();   // p.val >= node.val: successor is further right
        }
    }
    succ
}
```

When `p` has a right subtree, the descent naturally moves right at `p` and the last left-turn captured is the correct ancestor (or the leftmost-of-right is reached); when it has none, the last recorded left-turn ancestor is returned. This unifies both cases in O(h) time, O(1) space.

**Parent-pointer variant (LC 510), briefly.** If each node carries a `parent` pointer and you are *given the node itself* (not the root): if it has a right child, return the leftmost descendant of that right child; otherwise climb parents until you come up from a *left* child, and return that parent (or `None` at the root). Same O(h), still O(1) extra space.

---

## L4 / L5 Signal

What separates a strong senior candidate from a mid-level one on these problems:

- **Reaches for Morris unprompted.** Mid-level: solves Recover BST with a stack and stops. Senior: anticipates "constant space?" and produces the threaded traversal, correctly tearing down threads so the tree is left intact.
- **Reasons about augmentation tradeoffs.** Senior candidates do not just bolt on `count` — they explain the per-mutation cost, the read/write ratio that justifies it, and that recomputing from children sizes survives future rebalancing. They connect it to order-statistic trees.
- **States invariants cleanly.** "Inorder of a BST is strictly increasing" is named explicitly and used to justify both Validate, Recover, and Kth Smallest. The number of descents (one vs two) in Recover is reasoned about, not guessed.
- **Unifies cases.** The single-descent successor that handles both the right-subtree and no-right-subtree cases — rather than two branches stitched together — is a strong L5 signal of comfort with the structure.
- **Clean boundary handling.** `i64` bounds, strict comparisons, null roots, and k out of range are addressed before being asked.

> **Last Updated:** 2026-06-26
