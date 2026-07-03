> **Topic:** [Binary Trees](../README.md) · **Pattern 1 of 5**

# Tree Traversals

Traversal is the foundation of every binary-tree algorithm. Master the four core orders
(preorder, inorder, postorder, level-order) both recursively and iteratively, plus the two
"hard" traversals interviewers love: **zigzag** and **vertical order**.

```rust
use std::rc::Rc;
use std::cell::RefCell;

#[derive(Debug, PartialEq, Eq)]
pub struct TreeNode {
    pub val: i32,
    pub left: Option<Rc<RefCell<TreeNode>>>,
    pub right: Option<Rc<RefCell<TreeNode>>>,
}

impl TreeNode {
    pub fn new(val: i32) -> Self {
        TreeNode { val, left: None, right: None }
    }
}
```

---

## 1. Recursive DFS (the clean 3-liners)

The recursive shape is identical for all three orders — only the *position* of the visit
statement changes.

```rust
use std::rc::Rc;
use std::cell::RefCell;

type TreeLink = Option<Rc<RefCell<TreeNode>>>;

// Preorder: Root -> Left -> Right
fn preorder(node: &TreeLink, out: &mut Vec<i32>) {
    if let Some(n) = node {
        let n = n.borrow();
        out.push(n.val);              // visit
        preorder(&n.left, out);
        preorder(&n.right, out);
    }
}

// Inorder: Left -> Root -> Right
fn inorder(node: &TreeLink, out: &mut Vec<i32>) {
    if let Some(n) = node {
        let n = n.borrow();
        inorder(&n.left, out);
        out.push(n.val);              // visit
        inorder(&n.right, out);
    }
}

// Postorder: Left -> Right -> Root
fn postorder(node: &TreeLink, out: &mut Vec<i32>) {
    if let Some(n) = node {
        let n = n.borrow();
        postorder(&n.left, out);
        postorder(&n.right, out);
        out.push(n.val);              // visit
    }
}
```

**Mnemonic:** the prefix (pre/in/post) tells you *where the Root is visited* relative to the
two recursive descents.

---

## 2. Iterative Inorder — push-left-first (LC 94)

Walk all the way left pushing onto a stack, then pop (visit), then move to the right child
and repeat.

```rust
use std::rc::Rc;
use std::cell::RefCell;

type TreeLink = Option<Rc<RefCell<TreeNode>>>;

fn inorder_traversal(root: TreeLink) -> Vec<i32> {
    let mut out = Vec::new();
    let mut stk: Vec<Rc<RefCell<TreeNode>>> = Vec::new();
    let mut curr = root;
    while curr.is_some() || !stk.is_empty() {
        while let Some(node) = curr {       // push the entire left spine
            curr = node.borrow().left.clone();
            stk.push(node);
        }
        let node = stk.pop().unwrap();      // leftmost unvisited node
        out.push(node.borrow().val);        // visit
        curr = node.borrow().right.clone(); // then explore its right subtree
    }
    out
}
```

---

## 3. Iterative Preorder — push right, then left (LC 144)

Because a stack is LIFO, push the **right** child first so the **left** child is processed
next (preorder = Root, Left, Right).

```rust
use std::rc::Rc;
use std::cell::RefCell;

type TreeLink = Option<Rc<RefCell<TreeNode>>>;

fn preorder_traversal(root: TreeLink) -> Vec<i32> {
    let mut out = Vec::new();
    if root.is_none() { return out; }
    let mut stk: Vec<Rc<RefCell<TreeNode>>> = Vec::new();
    stk.push(root.unwrap());
    while let Some(node) = stk.pop() {
        out.push(node.borrow().val);                                           // visit root first
        if let Some(right) = node.borrow().right.clone() { stk.push(right); } // right pushed first
        if let Some(left)  = node.borrow().left.clone()  { stk.push(left);  } // left popped first
    }
    out
}
```

---

## 4. Iterative Postorder (LC 145)

### Approach A — two stacks
Do a *modified preorder* (Root, Right, Left) onto `stack2`, then reverse it by popping.

```rust
use std::rc::Rc;
use std::cell::RefCell;

type TreeLink = Option<Rc<RefCell<TreeNode>>>;

fn postorder_traversal(root: TreeLink) -> Vec<i32> {
    let mut out = Vec::new();
    if root.is_none() { return out; }
    let mut stk: Vec<Rc<RefCell<TreeNode>>> = Vec::new();
    stk.push(root.unwrap());
    while let Some(node) = stk.pop() {
        out.push(node.borrow().val);                    // collect Root,Right,Left
        if let Some(left)  = node.borrow().left.clone()  { stk.push(left);  }
        if let Some(right) = node.borrow().right.clone() { stk.push(right); }
    }
    out.reverse(); // ends up Left, Right, Root
    out
}
```

This is the **reverse-of-modified-preorder trick**: preorder visiting Root→Right→Left,
collected with `push` then `reverse`, yields Left→Right→Root.

### Approach B — single stack with a `last_visited` pointer
Visit a node only after its right child has been fully processed.

```rust
use std::rc::Rc;
use std::cell::RefCell;

type TreeLink = Option<Rc<RefCell<TreeNode>>>;

fn postorder_single_stack(root: TreeLink) -> Vec<i32> {
    let mut out = Vec::new();
    let mut stk: Vec<Rc<RefCell<TreeNode>>> = Vec::new();
    let mut curr = root;
    let mut last_visited: Option<Rc<RefCell<TreeNode>>> = None;
    while curr.is_some() || !stk.is_empty() {
        while let Some(node) = curr {
            curr = node.borrow().left.clone();
            stk.push(node);
        }
        let peek = stk.last().unwrap().clone();
        let peek_right = peek.borrow().right.clone();
        let should_go_right = match (&peek_right, &last_visited) {
            (Some(r), Some(lv)) => !Rc::ptr_eq(r, lv),
            (Some(_), None) => true,
            _ => false,
        };
        if should_go_right {
            curr = peek_right;                    // go right before visiting
        } else {
            out.push(peek.borrow().val);
            last_visited = Some(stk.pop().unwrap());
        }
    }
    out
}
```

---

## 5. Morris Traversal — O(1) space inorder

Threads each node's predecessor's right pointer to itself, removing the need for a stack.
Time O(n), space **O(1)** (mutates then restores pointers).

```rust
use std::rc::Rc;
use std::cell::RefCell;

type TreeLink = Option<Rc<RefCell<TreeNode>>>;

fn morris_inorder(root: TreeLink) -> Vec<i32> {
    let mut out = Vec::new();
    let mut curr = root;
    while let Some(curr_rc) = curr.clone() {
        if curr_rc.borrow().left.is_none() {
            out.push(curr_rc.borrow().val);
            curr = curr_rc.borrow().right.clone();
        } else {
            // find inorder predecessor
            let mut pred_opt = curr_rc.borrow().left.clone();
            loop {
                let next = {
                    let pred = pred_opt.as_ref().unwrap().borrow();
                    match &pred.right {
                        Some(r) if !Rc::ptr_eq(r, &curr_rc) => Some(r.clone()),
                        _ => None,
                    }
                };
                if let Some(n) = next { pred_opt = Some(n); }
                else { break; }
            }
            let pred_rc = pred_opt.unwrap();
            if pred_rc.borrow().right.is_none() {          // create thread
                pred_rc.borrow_mut().right = Some(curr_rc.clone());
                curr = curr_rc.borrow().left.clone();
            } else {                                       // thread exists -> remove it, visit
                pred_rc.borrow_mut().right = None;
                out.push(curr_rc.borrow().val);
                curr = curr_rc.borrow().right.clone();
            }
        }
    }
    out
}
```

---

## 6. Level Order — BFS with size snapshot (LC 102)

The **size-snapshot** is the key idiom: capture `queue.len()` *before* the inner loop so
you process exactly one level at a time.

```rust
use std::rc::Rc;
use std::cell::RefCell;
use std::collections::VecDeque;

type TreeLink = Option<Rc<RefCell<TreeNode>>>;

fn level_order(root: TreeLink) -> Vec<Vec<i32>> {
    let mut result = Vec::new();
    if root.is_none() { return result; }
    let mut q: VecDeque<Rc<RefCell<TreeNode>>> = VecDeque::new();
    q.push_back(root.unwrap());
    while !q.is_empty() {
        let size = q.len();                    // snapshot: nodes on this level
        let mut level = Vec::with_capacity(size);
        for _ in 0..size {
            let node = q.pop_front().unwrap();
            level.push(node.borrow().val);
            if let Some(left)  = node.borrow().left.clone()  { q.push_back(left);  }
            if let Some(right) = node.borrow().right.clone() { q.push_back(right); }
        }
        result.push(level);
    }
    result
}
```

---

## 7. Zigzag Level Order (LC 103)

Run normal BFS but alternate the **insertion direction** within each level using a
`VecDeque` + `push_front`/`push_back`. This keeps each insert O(1) (never inserting at an
arbitrary index).

```rust
use std::rc::Rc;
use std::cell::RefCell;
use std::collections::VecDeque;

type TreeLink = Option<Rc<RefCell<TreeNode>>>;

fn zigzag_level_order(root: TreeLink) -> Vec<Vec<i32>> {
    let mut result = Vec::new();
    if root.is_none() { return result; }
    let mut q: VecDeque<Rc<RefCell<TreeNode>>> = VecDeque::new();
    q.push_back(root.unwrap());
    let mut left_to_right = true;
    while !q.is_empty() {
        let size = q.len();
        let mut level: VecDeque<i32> = VecDeque::new();
        for _ in 0..size {
            let node = q.pop_front().unwrap();
            if left_to_right { level.push_back(node.borrow().val); }
            else             { level.push_front(node.borrow().val); }
            if let Some(left)  = node.borrow().left.clone()  { q.push_back(left);  }
            if let Some(right) = node.borrow().right.clone() { q.push_back(right); }
        }
        result.push(level.into_iter().collect());
        left_to_right = !left_to_right;
    }
    result
}
```

---

## 8. N-ary Level Order (LC 429)

Identical BFS, but children come from a list (`node.children`).

```rust
use std::rc::Rc;
use std::cell::RefCell;
use std::collections::VecDeque;

#[derive(Debug)]
pub struct Node {
    pub val: i32,
    pub children: Vec<Rc<RefCell<Node>>>,
}

fn level_order_nary(root: Option<Rc<RefCell<Node>>>) -> Vec<Vec<i32>> {
    let mut result = Vec::new();
    if root.is_none() { return result; }
    let mut q: VecDeque<Rc<RefCell<Node>>> = VecDeque::new();
    q.push_back(root.unwrap());
    while !q.is_empty() {
        let size = q.len();
        let mut level = Vec::with_capacity(size);
        for _ in 0..size {
            let node = q.pop_front().unwrap();
            level.push(node.borrow().val);
            let children: Vec<_> = node.borrow().children.clone();
            for child in children { q.push_back(child); }
        }
        result.push(level);
    }
    result
}
```

---

## 9. Vertical Order Traversal (LC 987)

Group nodes by **column** (root = 0, left = col−1, right = col+1). Within a column, order by
**row** (depth), and ties at the same (row, col) by **value**. A
`BTreeMap<col, BTreeMap<row, BinaryHeap<Reverse<val>>>>` makes ordering deterministic without a final
sort — `BTreeMap` is used (not `HashMap`) precisely because we need keys iterated in sorted
order.

```rust
use std::rc::Rc;
use std::cell::RefCell;
use std::collections::{BTreeMap, BinaryHeap};
use std::cmp::Reverse;

type TreeLink = Option<Rc<RefCell<TreeNode>>>;

fn dfs(node: &TreeLink, row: i32, col: i32,
       mp: &mut BTreeMap<i32, BTreeMap<i32, BinaryHeap<Reverse<i32>>>>) {
    if let Some(n) = node {
        let val   = n.borrow().val;
        let left  = n.borrow().left.clone();
        let right = n.borrow().right.clone();
        mp.entry(col).or_default().entry(row).or_default().push(Reverse(val));
        dfs(&left,  row + 1, col - 1, mp);
        dfs(&right, row + 1, col + 1, mp);
    }
}

fn vertical_traversal(root: TreeLink) -> Vec<Vec<i32>> {
    // col -> (row -> min-heap of values)
    let mut mp: BTreeMap<i32, BTreeMap<i32, BinaryHeap<Reverse<i32>>>> = BTreeMap::new();
    dfs(&root, 0, 0, &mut mp);

    let mut result = Vec::new();
    for (_col_key, rows) in &mut mp {
        let mut col = Vec::new();
        for (_row_key, heap) in rows.iter_mut() {
            let mut tmp = std::mem::take(heap);
            while let Some(Reverse(val)) = tmp.pop() { col.push(val); }
        }
        result.push(col);
    }
    result
}
```

---

## Complexity

| Traversal | Approach | Time | Space |
|---|---|---|---|
| Inorder / Preorder / Postorder (recursive) | DFS | O(n) | O(h) stack |
| Inorder (iterative) | explicit stack | O(n) | O(h) |
| Preorder (iterative) | explicit stack | O(n) | O(h) |
| Postorder (iterative) | two-stack / last_visited | O(n) | O(h) |
| Inorder (Morris) | threading | O(n) | **O(1)** |
| Level Order (LC 102) | BFS | O(n) | O(w) queue |
| Zigzag (LC 103) | BFS | O(n) | O(w) |
| N-ary Level Order (LC 429) | BFS | O(n) | O(w) |
| Vertical Order (LC 987) | DFS + BTreeMap | O(n log n) | O(n) |

*h = tree height (O(log n) balanced, O(n) skewed); w = max width of any level.*

---

### Related
- [Level Order](Level%20Order.md) — the full BFS catalog
- [Tree Construction](Tree%20Construction.md) — rebuilding trees from traversal arrays

> **Last Updated:** 2026-06-26
