> **Topic:** [Binary Trees](../README.md) · **Pattern N of M** · **Company 2 of 3**

# Google — Binary Tree Interview Problems

Google binary-tree questions tend to stress **deterministic ordering, coordinate reasoning, and graph-style traversals on tree structures**. The interviewer cares less about whether you can recall an algorithm and more about whether your code is clean, your tie-breaking rules are exact, and you proactively handle edge cases. The three problems below are classic Google rounds.

Standard node definition used throughout:

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

## 1. Vertical Order Traversal of a Binary Tree (LC 987)

### Problem Statement
Assign the root the coordinate `(row=0, col=0)`. A left child is at `(row+1, col-1)` and a right child at `(row+1, col+1)`. The **vertical order traversal** lists nodes grouped by column from leftmost to rightmost. Within the same column, nodes are ordered by row (top to bottom); and crucially, **if two nodes share the same row and column, they are reported in ascending order of value**.

That last tie-break is what makes LC 987 harder than the older LC 314.

### Intuition — The Full Ordering Rule
We need a strict total order over `(col, row, value)`:

1. **Primary:** smaller `col` first (left to right).
2. **Secondary:** smaller `row` first (top to bottom).
3. **Tertiary:** smaller `value` first (the LC 987 tie-break).

### Why BTreeMap, not HashMap
The output must be emitted in **ascending column order**, and within a column in ascending row order. A `HashMap` gives no ordering guarantee, so we'd have to collect keys and sort afterward. A **`BTreeMap` keeps keys sorted automatically**, so iterating it yields columns left-to-right (and rows top-to-bottom) for free, with deterministic behavior. This determinism is exactly what Google interviewers probe — "what happens to ties?" — and choosing `BTreeMap` lets you answer "the structure enforces it."

We model the data as:

```
BTreeMap<i32, BTreeMap<i32, BinaryHeap<Reverse<i32>>>>
   col  ->    row  ->       min-heap of values
```

- Outer `BTreeMap`: columns sorted ascending.
- Inner `BTreeMap`: rows sorted ascending within a column.
- `BinaryHeap<Reverse<i32>>` (min-heap): values sorted ascending within the same `(col,row)` cell, satisfying the tertiary tie-break.

### Full Rust Solution

```rust
use std::collections::{BTreeMap, BinaryHeap};
use std::cmp::Reverse;

struct Solution;

impl Solution {
    pub fn vertical_traversal(root: Option<Box<TreeNode>>) -> Vec<Vec<i32>> {
        // col -> (row -> min-heap of values)
        let mut grid: BTreeMap<i32, BTreeMap<i32, BinaryHeap<Reverse<i32>>>> = BTreeMap::new();

        Self::dfs(root.as_deref(), 0, 0, &mut grid);

        let mut result = Vec::new();
        // Iterating a BTreeMap walks columns in ascending order.
        for (_col, rows) in &mut grid {
            let mut column = Vec::new();
            // Inner BTreeMap walks rows in ascending order.
            for (_row, cell) in rows.iter_mut() {
                // Drain the min-heap: smallest values first.
                while let Some(Reverse(val)) = cell.pop() {
                    column.push(val);
                }
            }
            result.push(column);
        }
        result
    }

    fn dfs(
        node: Option<&TreeNode>,
        row: i32,
        col: i32,
        grid: &mut BTreeMap<i32, BTreeMap<i32, BinaryHeap<Reverse<i32>>>>,
    ) {
        if let Some(n) = node {
            grid.entry(col).or_default().entry(row).or_default().push(Reverse(n.val));
            Self::dfs(n.left.as_deref(), row + 1, col - 1, grid);
            Self::dfs(n.right.as_deref(), row + 1, col + 1, grid);
        }
    }
}
```

### Complexity

| Metric | Value | Reason |
| --- | --- | --- |
| Time | O(n log n) | Each insert into the nested structure / heap costs O(log n); n inserts dominate. |
| Space | O(n) | The grid stores every node once, plus O(h) recursion stack. |

### Note
A DFS or BFS both work for *visiting* nodes; the ordering is enforced entirely by the data structure, not the traversal order. That decoupling — "traverse however; let `BTreeMap`/`BinaryHeap` impose the rule" — is the clean insight to state aloud.

---

## 2. Lowest Common Ancestor of Deepest Leaves (LC 1123)

### Problem Statement
Return the LCA of the **deepest leaves** of a binary tree. If there is a single deepest leaf, it is its own LCA. Equivalent to LC 865 ("Smallest Subtree with all the Deepest Nodes").

### Intuition — The Two-Return Approach
A single post-order pass can return, for each subtree, a pair: **(the LCA-of-deepest-leaves within this subtree, the depth of those deepest leaves)**. Combine children:

- Compute `left = solve(node.left)` and `right = solve(node.right)`.
- If `left.depth > right.depth`: the deepest leaves all live in the left subtree, so propagate the left answer with depth `left.depth + 1`.
- If `right.depth > left.depth`: symmetric — propagate the right answer.
- If **`left.depth == right.depth`**: the deepest leaves straddle both subtrees, so the **current node is their LCA**; propagate `node` with depth `left.depth + 1`.

Base case: a null node returns `(None, 0)`. A leaf therefore returns `(leaf, 1)` because both children give depth 0 (equal), making the leaf its own LCA.

The "equal depths → current node" rule is the heart of the problem: equal-depth deepest leaves on both sides mean the divergence point is here.

### Full Rust Solution

```rust
use std::rc::Rc;
use std::cell::RefCell;

// Rc<RefCell<>> is used here to allow returning shared references to interior
// nodes, matching LeetCode's actual Rust TreeNode type.
type Link = Option<Rc<RefCell<TreeNode>>>;

// Small struct carrying the running answer and its depth.
struct LcaResult {
    node: Link, // LCA of deepest leaves within this subtree
    depth: i32, // depth of those deepest leaves (relative to this subtree)
}

struct Solution;

impl Solution {
    pub fn lca_deepest_leaves(root: Link) -> Link {
        Self::solve(root).node
    }

    fn solve(node: Link) -> LcaResult {
        match node {
            None => LcaResult { node: None, depth: 0 },
            Some(n) => {
                let left = Self::solve(n.borrow().left.clone());
                let right = Self::solve(n.borrow().right.clone());

                if left.depth > right.depth {
                    LcaResult { node: left.node, depth: left.depth + 1 }
                } else if right.depth > left.depth {
                    LcaResult { node: right.node, depth: right.depth + 1 }
                } else {
                    // Equal depths: deepest leaves are on both sides -> this node is the LCA.
                    LcaResult { node: Some(n), depth: left.depth + 1 }
                }
            }
        }
    }
}
```

### Complexity

| Metric | Value | Reason |
| --- | --- | --- |
| Time | O(n) | Single post-order traversal. |
| Space | O(h) | Recursion stack; each `LcaResult` is O(1). |

### Note
Avoid the naive "first compute max depth, then run LCA over all deepest leaves" — it works but is two passes and clumsier. The single-pass two-return formulation is the answer Google expects.

---

## 3. All Nodes Distance K in Binary Tree (LC 863)

### Problem Statement
Given the root, a `target` node, and an integer `k`, return the values of all nodes that are **exactly distance `k`** from `target`. Distance counts edges, moving up to parents or down to children freely.

### Intuition — Treat the Tree as an Undirected Graph
The difficulty is that distance can go **upward** (toward parents), but tree nodes only point downward. Two-phase fix:

1. **Build a parent map** with a DFS so every node knows its parent. Now each node effectively has up to three neighbors: left child, right child, and parent.
2. **BFS from `target`**, expanding to all three neighbors, tracking visited nodes so we never bounce back. After exactly `k` BFS levels, every node still in the queue is at distance `k`.

This converts a tree into an undirected graph and reduces "distance k" to a plain breadth-first layer.

### Full Rust Solution

```rust
use std::collections::{HashMap, HashSet, VecDeque};

// Index-based tree representation avoids parent-pointer aliasing issues.
// nodes[i] = (value, left_child_index, right_child_index)
// Root is always at index 0.
struct TreeArr {
    nodes: Vec<(i32, Option<usize>, Option<usize>)>,
}

struct Solution;

impl Solution {
    // target_idx: index of the target node; root is at index 0.
    pub fn distance_k(tree: &TreeArr, target_idx: usize, k: usize) -> Vec<i32> {
        // Phase 1: map each node index to its parent index.
        let mut parent: HashMap<usize, usize> = HashMap::new();
        Self::build_parents(tree, 0, None, &mut parent);

        // Phase 2: BFS outward from target through up-to-three neighbors.
        let mut result = Vec::new();
        let mut bfs_queue: VecDeque<usize> = VecDeque::new();
        let mut visited: HashSet<usize> = HashSet::new();

        bfs_queue.push_back(target_idx);
        visited.insert(target_idx);
        let mut dist = 0;

        while !bfs_queue.is_empty() {
            let size = bfs_queue.len();
            if dist == k {
                // Everything currently in the queue is exactly k edges away.
                for &idx in &bfs_queue {
                    result.push(tree.nodes[idx].0);
                }
                return result;
            }
            for _ in 0..size {
                let idx = bfs_queue.pop_front().unwrap();
                let (_, left, right) = tree.nodes[idx];
                let par = parent.get(&idx).copied();
                // Three potential neighbors: left child, right child, parent.
                Self::add_neighbor(left, &mut visited, &mut bfs_queue);
                Self::add_neighbor(right, &mut visited, &mut bfs_queue);
                Self::add_neighbor(par, &mut visited, &mut bfs_queue);
            }
            dist += 1;
        }
        result
    }

    fn add_neighbor(
        neighbor: Option<usize>,
        visited: &mut HashSet<usize>,
        bfs_queue: &mut VecDeque<usize>,
    ) {
        if let Some(idx) = neighbor {
            if visited.insert(idx) {
                bfs_queue.push_back(idx);
            }
        }
    }

    fn build_parents(
        tree: &TreeArr,
        idx: usize,
        par: Option<usize>,
        parent: &mut HashMap<usize, usize>,
    ) {
        if let Some(p) = par {
            parent.insert(idx, p);
        }
        let (_, left, right) = tree.nodes[idx];
        if let Some(l) = left {
            Self::build_parents(tree, l, Some(idx), parent);
        }
        if let Some(r) = right {
            Self::build_parents(tree, r, Some(idx), parent);
        }
    }
}
```

### Complexity

| Metric | Value | Reason |
| --- | --- | --- |
| Time | O(n) | One DFS to build parents + one BFS, each visiting nodes once. |
| Space | O(n) | Parent map, visited set, and queue each hold up to n nodes. |

### Note
In Rust, `visited.insert(idx)` returns `true` if the index was not already present — "add only if unseen" — the same guarantee as the C++ `visited.insert(neighbor).second` pattern. Without `visited`, BFS would oscillate between a node and its parent forever.

---

## Google Interview Level Context (L4 / L5)

Google calibrates these problems differently by level — knowing the expectation helps you allocate time:

- **L4 (SWE II):** Expected to reach a correct, clean solution with good naming and structure. You should **enumerate edge cases unprompted** — empty tree, single node, `target` is the root, `k = 0`, `k` larger than the tree's reach. Stating time/space complexity precisely (and *why*) is mandatory, not optional.
- **L5 (Senior SWE):** Beyond correctness, demonstrate **design judgment**: choosing `BTreeMap` for determinism (LC 987), recognizing the single-pass two-return formulation (LC 1123), and reframing a tree as a graph (LC 863). Expect **follow-ups** — "what if the tree is enormous and doesn't fit in memory?", "can you do LC 863 without a parent map?" (yes: a single DFS that returns the subtree distance to `target` and collects downward nodes as it unwinds), "how would you stream LC 987 output?".

General Google bar across levels:
- Talk through the **contract/invariant** before coding; don't dive into syntax first.
- Write **compilable** Rust with full method signatures and `use` statements — Googlers dislike pseudocode.
- **Test your own code** on a small example by hand and walk the interviewer through it.
- Discuss **complexity trade-offs** and at least one **follow-up extension** proactively.

---

> **Last Updated:** 2026-06-26
