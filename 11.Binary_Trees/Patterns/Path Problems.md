> **Topic:** [Binary Trees](../README.md) · **Pattern 3 of 5**

# Path Problems

A "path" in a binary tree is a sequence of connected nodes. Problems differ by **where the
path may start/end** (root→leaf, any→any), and whether you **collect** paths or **score**
them. Two recurring techniques: **leaf-base-case checks** and **backtracking** for collection.

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

## 1. Path Sum (LC 112) — does a root→leaf path equal target?

The base case is the **leaf check**: a leaf has no children, so the path ends there.

```rust
fn has_path_sum(root: Option<&Box<TreeNode>>, target_sum: i32) -> bool {
    match root {
        None => false,
        Some(node) => {
            if node.left.is_none() && node.right.is_none() {      // leaf
                return node.val == target_sum;
            }
            let remaining = target_sum - node.val;
            has_path_sum(node.left.as_ref(), remaining)
                || has_path_sum(node.right.as_ref(), remaining)
        }
    }
}
```

> Don't use `root == None && target_sum == 0` as the success base case — an empty child of a
> one-child node would falsely succeed. Always check the **leaf** condition explicitly.

---

## 2. Path Sum II (LC 113) — collect all root→leaf paths summing to target

Classic **backtracking**: add the node to the path, recurse, then *remove it* so siblings
start from a clean path.

```rust
fn path_sum(root: Option<&Box<TreeNode>>, target_sum: i32) -> Vec<Vec<i32>> {
    let mut result = Vec::new();
    let mut path = Vec::new();
    backtrack(root, target_sum, &mut path, &mut result);
    result
}

fn backtrack(node: Option<&Box<TreeNode>>, remaining: i32,
             path: &mut Vec<i32>, result: &mut Vec<Vec<i32>>) {
    let node = match node {
        None => return,
        Some(n) => n,
    };
    path.push(node.val);                                  // choose
    if node.left.is_none() && node.right.is_none() && remaining == node.val {
        result.push(path.clone());               // copy! the path is mutable
    } else {
        backtrack(node.left.as_ref(),  remaining - node.val, path, result);
        backtrack(node.right.as_ref(), remaining - node.val, path, result);
    }
    path.pop();                        // un-choose (backtrack)
}
```

> ⚠️ Two non-negotiables: **copy** the path when recording (`result.push(path.clone())`), and
> **remove** the last element on the way out. Forgetting either is the #1 bug here.

---

## 3. Diameter of Binary Tree (LC 543)

The diameter is the longest path *in edges* between any two nodes — it **need not pass
through the root**. Compute each node's depth bottom-up while updating a global max of
`left_depth + right_depth`.

```rust
fn diameter_of_binary_tree(root: Option<&Box<TreeNode>>) -> i32 {
    let mut max_diameter = 0;
    depth(root, &mut max_diameter);
    max_diameter
}

fn depth(node: Option<&Box<TreeNode>>, max_diameter: &mut i32) -> i32 {
    let node = match node {
        None => return 0,
        Some(n) => n,
    };
    let left  = depth(node.left.as_ref(),  max_diameter);
    let right = depth(node.right.as_ref(), max_diameter);
    *max_diameter = (*max_diameter).max(left + right);   // path THROUGH this node
    1 + left.max(right)                                  // depth returned to parent
}
```

**The dual-purpose recursion insight:** the helper *returns* depth to its parent but
*updates a mutable reference* with the through-node path length. The thing you return and the thing you
maximize are different.

---

## 4. Binary Tree Maximum Path Sum (LC 124) — Hard

Same dual-purpose shape as diameter, with two twists:
1. **Prune negative subtrees:** `gain = 0.max(gain)` — a negative subtree contributes
   nothing, so we treat it as 0.
2. The best path **through** a node is `node.val + left_gain + right_gain`, but the value
   **returned** to the parent can only follow one branch:
   `node.val + left_gain.max(right_gain)`.

```rust
fn max_path_sum(root: Option<&Box<TreeNode>>) -> i32 {
    let mut max_sum = i32::MIN;
    gain(root, &mut max_sum);
    max_sum
}

fn gain(node: Option<&Box<TreeNode>>, max_sum: &mut i32) -> i32 {
    let node = match node {
        None => return 0,
        Some(n) => n,
    };
    let left_gain  = 0_i32.max(gain(node.left.as_ref(),  max_sum));    // ignore negative contributions
    let right_gain = 0_i32.max(gain(node.right.as_ref(), max_sum));
    *max_sum = (*max_sum).max(node.val + left_gain + right_gain);  // path through node
    node.val + left_gain.max(right_gain)                            // extendable to parent
}
```

---

## 5. Sum Root to Leaf Numbers (LC 129)

Each root→leaf path spells a number (e.g. `1→2→3` = 123). Carry the running value **down**
the recursion: `current = current * 10 + node.val`.

```rust
fn sum_numbers(root: Option<&Box<TreeNode>>) -> i32 {
    dfs(root, 0)
}

fn dfs(node: Option<&Box<TreeNode>>, current: i32) -> i32 {
    let node = match node {
        None => return 0,
        Some(n) => n,
    };
    let current = current * 10 + node.val;
    if node.left.is_none() && node.right.is_none() { return current; }   // leaf -> the number
    dfs(node.left.as_ref(), current) + dfs(node.right.as_ref(), current)
}
```

This is **top-down** accumulation — state flows down via the parameter, results sum on the
way back up.

---

## 6. Binary Tree Paths (LC 257) — all root→leaf paths as strings

Build a `"a->b->c"` `String` per path. Use a `String` with length-rollback (backtrack),
or simple string concatenation passed down.

```rust
fn binary_tree_paths(root: Option<&Box<TreeNode>>) -> Vec<String> {
    let mut result = Vec::new();
    if root.is_some() {
        let mut path = String::new();
        dfs(root, &mut path, &mut result);
    }
    result
}

fn dfs(node: Option<&Box<TreeNode>>, path: &mut String, result: &mut Vec<String>) {
    let node = match node {
        None => return,
        Some(n) => n,
    };
    let len = path.len();                       // remember length to roll back
    if !path.is_empty() { path.push_str("->"); }
    path.push_str(&node.val.to_string());
    if node.left.is_none() && node.right.is_none() {
        result.push(path.clone());
    } else {
        if node.left.is_some()  { dfs(node.left.as_ref(),  path, result); }
        if node.right.is_some() { dfs(node.right.as_ref(), path, result); }
    }
    path.truncate(len);                           // backtrack the string
}
```

Simpler (less efficient) alternative: pass `path.clone() + "->" + &node.val.to_string()` or
build with a delimiter — fine for interview clarity.

---

## Complexity

| Problem | LC | Time | Space |
|---|---|---|---|
| Path Sum | 112 | O(n) | O(h) |
| Path Sum II | 113 | O(n²) worst (copying paths) | O(n) paths + O(h) |
| Diameter | 543 | O(n) | O(h) |
| Maximum Path Sum | 124 | O(n) | O(h) |
| Sum Root to Leaf | 129 | O(n) | O(h) |
| Binary Tree Paths | 257 | O(n·h) (string build) | O(h) |

*h = height. Path Sum II is O(n²) worst because each of up to O(n) leaves may copy an
O(h)-length path.*

---

### Related
- [Tree DP](Tree%20DP.md) — same bottom-up "return value + global" template applied to DP
- [Tree Traversals](Tree%20Traversals.md) — DFS foundations

> **Last Updated:** 2026-06-26
