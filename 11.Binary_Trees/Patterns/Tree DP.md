> **Topic:** [Binary Trees](../README.md) · **Pattern 5 of 5**

# Tree DP / Advanced

**Core idea:** a *post-order* DFS where each node **returns aggregated information about its
subtree** to its parent, and (optionally) a **global result** is updated at each node. The
recurrence lives in how a parent combines its children's returns.

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

> The template: `f(node)` returns a *summary* of the subtree rooted at `node`; the parent
> combines `f(node.left)` and `f(node.right)` plus `node.val`. When the answer can occur
> "through" any node (not just the root), keep a global and update it inside `f`.

---

## 1. House Robber III (LC 337)

You can't rob two directly-connected nodes. Each node returns a 2-element state:
`[notRob, rob]` where `notRob` = best when this node is NOT robbed (children free to do
either), `rob` = `node.val + children's notRob`.

```rust
// returns (not_robbing_this_node, robbing_this_node)
fn rob_sub(node: &Option<Box<TreeNode>>) -> (i32, i32) {
    if let Some(n) = node {
        let l = rob_sub(&n.left);
        let r = rob_sub(&n.right);
        let not_rob = l.0.max(l.1) + r.0.max(r.1); // children choose freely
        let rob = n.val + l.0 + r.0;                // children must NOT be robbed
        (not_rob, rob)
    } else {
        (0, 0)
    }
}

fn rob(root: &Option<Box<TreeNode>>) -> i32 {
    let res = rob_sub(root);
    res.0.max(res.1)
}
```

---

## 2. Binary Tree Cameras (LC 968) — Hard

Greedy bottom-up DP with three states:
`0 = NOT_COVERED`, `1 = HAS_CAMERA`, `2 = COVERED (by a child's camera)`.

- A `null` child is treated as **COVERED** (it needs nothing).
- If **either** child is `NOT_COVERED`, this node **must** place a camera → return
  `HAS_CAMERA`, `count++`.
- If **either** child `HAS_CAMERA`, this node is `COVERED`.
- Otherwise both children are `COVERED`, so this node is `NOT_COVERED` (its parent must cover
  it).
- **Root special case:** after recursion, if the root is `NOT_COVERED`, add one camera.

```rust
const NOT_COVERED: i32 = 0;
const HAS_CAMERA: i32 = 1;
const COVERED: i32 = 2;

fn dfs(node: &Option<Box<TreeNode>>, cameras: &mut i32) -> i32 {
    if let Some(n) = node {
        let l = dfs(&n.left, cameras);
        let r = dfs(&n.right, cameras);
        if l == NOT_COVERED || r == NOT_COVERED {
            *cameras += 1;
            return HAS_CAMERA;
        }
        if l == HAS_CAMERA || r == HAS_CAMERA {
            return COVERED;
        }
        NOT_COVERED
    } else {
        COVERED // null is implicitly covered
    }
}

fn min_camera_cover(root: &Option<Box<TreeNode>>) -> i32 {
    let mut cameras = 0;
    if dfs(root, &mut cameras) == NOT_COVERED {
        cameras += 1; // root left uncovered -> place a camera
    }
    cameras
}
```

---

## 3. Distribute Coins in Binary Tree (LC 979)

Each node should end with exactly 1 coin. The number of moves across an edge equals the
**absolute surplus/deficit** flowing through it. At each node compute
`excess = leftExcess + rightExcess + node.val − 1`; add `|excess|` to the answer and return
`excess` to the parent.

```rust
fn excess(node: &Option<Box<TreeNode>>, moves: &mut i32) -> i32 {
    if let Some(n) = node {
        let left  = excess(&n.left,  moves);
        let right = excess(&n.right, moves);
        *moves += left.abs() + right.abs(); // coins crossing both child edges
        n.val + left + right - 1            // surplus(+)/deficit(-) passed up
    } else {
        0
    }
}

fn distribute_coins(root: &Option<Box<TreeNode>>) -> i32 {
    let mut moves = 0;
    excess(root, &mut moves);
    moves
}
```

---

## 4. Lowest Common Ancestor (LC 236)

If the current node is `p` or `q`, return it. Recurse both sides; if **both** return non-null,
the current node is the LCA. Otherwise propagate the non-null side up.

```rust
fn lowest_common_ancestor<'a>(
    root: &'a Option<Box<TreeNode>>,
    p: i32,
    q: i32,
) -> Option<&'a TreeNode> {
    if let Some(node) = root {
        if node.val == p || node.val == q {
            return Some(node); // deref coercion: &Box<TreeNode> -> &TreeNode
        }
        let left  = lowest_common_ancestor(&node.left,  p, q);
        let right = lowest_common_ancestor(&node.right, p, q);
        match (left, right) {
            (Some(_), Some(_)) => Some(node), // p and q on opposite sides -> LCA
            (Some(l), None)    => Some(l),
            (None, Some(r))    => Some(r),
            (None, None)       => None,
        }
    } else {
        None
    }
}
```

**Why early-return works:** a node returns non-null iff its subtree contains `p` or `q`. The
first node to receive non-null from *both* sides is the split point = the LCA. If one of
`p`/`q` is an ancestor of the other, the early `node.val == p` return makes that ancestor the LCA.

---

## 5. LCA of Deepest Leaves (LC 1123)

Return a `(node, depth)` pair upward. Pick the side with greater depth; if both depths are
equal, the **current node** is the LCA of all deepest leaves below it.

```rust
// returns (lca_node, max_depth_below)
fn dfs_lca<'a>(node: &'a Option<Box<TreeNode>>) -> (Option<&'a TreeNode>, i32) {
    if let Some(n) = node {
        let (l_node, l_depth) = dfs_lca(&n.left);
        let (r_node, r_depth) = dfs_lca(&n.right);
        if l_depth == r_depth {
            (Some(n.as_ref()), l_depth + 1) // balanced -> this node is LCA
        } else if l_depth > r_depth {
            (l_node, l_depth + 1)
        } else {
            (r_node, r_depth + 1)
        }
    } else {
        (None, 0)
    }
}

fn lca_deepest_leaves(root: &Option<Box<TreeNode>>) -> Option<&TreeNode> {
    dfs_lca(root).0
}
```

---

## 6. Delete Nodes and Return Forest (LC 1110)

Put the to-delete values in a `HashSet`. Post-order: a child that survives **becomes a new root**
if its parent was deleted (or if it is the original root). When a node is deleted, its
surviving children are added to the forest; the node returns `None` to detach itself.

```rust
use std::collections::HashSet;

// consumes node; returns Some(node) if it survives, else None (detaches from parent)
fn dfs_delete(
    node: Option<Box<TreeNode>>,
    to_delete: &HashSet<i32>,
    forest: &mut Vec<Box<TreeNode>>,
) -> Option<Box<TreeNode>> {
    let mut n = node?;
    let deleted = to_delete.contains(&n.val);
    n.left  = dfs_delete(n.left,  to_delete, forest); // child becomes root if this node is deleted
    n.right = dfs_delete(n.right, to_delete, forest);
    if deleted {
        if let Some(l) = n.left.take()  { forest.push(l); }
        if let Some(r) = n.right.take() { forest.push(r); }
        return None;
    }
    Some(n)
}

fn del_nodes(root: Option<Box<TreeNode>>, to_delete: Vec<i32>) -> Vec<Box<TreeNode>> {
    let to_delete: HashSet<i32> = to_delete.into_iter().collect();
    let mut forest = Vec::new();
    if let Some(root_node) = dfs_delete(root, &to_delete, &mut forest) {
        forest.push(root_node);
    }
    forest
}
```

(The top-level `root` is added in `del_nodes` if it survives; surviving children of deleted
nodes are added inside `dfs_delete`.)

---

## 7. All Nodes Distance K (LC 863)

Two phases: (1) DFS to build an undirected adjacency map (keyed by node value) so the tree
becomes an undirected graph; (2) BFS from `target`, expanding to all neighbours, until you
reach distance `K`.

```rust
use std::collections::{HashMap, HashSet, VecDeque};

// build undirected adjacency list keyed by node value (values are unique per problem)
fn build_graph(
    node: &Option<Box<TreeNode>>,
    par_val: Option<i32>,
    adj: &mut HashMap<i32, Vec<i32>>,
) {
    if let Some(n) = node {
        let val = n.val;
        if let Some(p) = par_val {
            adj.entry(val).or_insert_with(Vec::new).push(p);
            adj.entry(p).or_insert_with(Vec::new).push(val);
        } else {
            adj.entry(val).or_insert_with(Vec::new); // root has no parent
        }
        build_graph(&n.left,  Some(val), adj);
        build_graph(&n.right, Some(val), adj);
    }
}

fn distance_k(root: &Option<Box<TreeNode>>, target_val: i32, k: i32) -> Vec<i32> {
    let mut adj: HashMap<i32, Vec<i32>> = HashMap::new();
    build_graph(root, None, &mut adj);

    let mut queue: VecDeque<i32> = VecDeque::new();
    let mut visited: HashSet<i32> = HashSet::new();
    queue.push_back(target_val);
    visited.insert(target_val);
    let mut dist = 0;

    while !queue.is_empty() {
        if dist == k {                                // current frontier is exactly distance k
            return queue.into_iter().collect();
        }
        let sz = queue.len();
        for _ in 0..sz {
            if let Some(val) = queue.pop_front() {
                if let Some(neighbors) = adj.get(&val) {
                    for &nei in neighbors {
                        if visited.insert(nei) {
                            queue.push_back(nei);
                        }
                    }
                }
            }
        }
        dist += 1;
    }
    vec![]
}
```

(The adjacency map uses node values as keys since all values are unique in this problem.
`adj.entry(val).or_insert_with(Vec::new).push(p)` adds the parent to the current node's
neighbour list, and the symmetric call adds the current node to the parent's list.)

---

## Complexity

| Problem | LC | Time | Space |
|---|---|---|---|
| House Robber III | 337 | O(n) | O(h) |
| Binary Tree Cameras | 968 | O(n) | O(h) |
| Distribute Coins | 979 | O(n) | O(h) |
| LCA | 236 | O(n) | O(h) |
| LCA of Deepest Leaves | 1123 | O(n) | O(h) |
| Delete Nodes Return Forest | 1110 | O(n) | O(n) set + O(h) |
| All Nodes Distance K | 863 | O(n) | O(n) parent map + queue |

---

### Related
- [Path Problems](Path%20Problems.md) — diameter & max-path-sum share the "return + global" idiom
- [Serialize / Deserialize](../Design%20Data%20Structure%20Problems/Serialize%20Deserialize.md)

> **Last Updated:** 2026-06-26
