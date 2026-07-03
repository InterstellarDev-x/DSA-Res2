> **Topic:** [Binary Trees](../README.md) · **Pattern N of M** · **Company 3 of 3**

# Microsoft — Binary Tree Interview Problems

Microsoft binary-tree interviews lean on **reconstruction, careful index/state bookkeeping, and greedy tree DP**. The interviewer typically wants to see you reason out invariants on paper, write clean compilable code, and then *test it yourself* on a small input. The three problems below are recurring Microsoft on-sites.

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

## 1. Construct Binary Tree from Preorder and Inorder Traversal (LC 105)

### Problem Statement
Given two integer arrays `preorder` and `inorder` representing the preorder and inorder traversals of a binary tree (all values distinct), reconstruct and return the tree.

### Intuition — The Index Invariants
Two facts power the whole reconstruction:

1. In **preorder** (root, left, right), the **first** element of any contiguous segment is the **root** of that subtree.
2. In **inorder** (left, root, right), the root **splits** the segment into a left part (everything before the root) and a right part (everything after). The size of the left part tells us how many preorder elements belong to the left subtree.

So for each recursive call over `preorder[pre_start..pre_end]` and `inorder[in_start..in_end]`:

- `root_val = preorder[pre_start]` — the root of this subtree.
- `in_idx` = index of `root_val` inside `inorder` (looked up in O(1) via a `HashMap<i32, usize>`).
- `left_size = in_idx - in_start` — number of nodes in the left subtree.
- **Left subtree** uses preorder range `pre_start+1 .. pre_start+left_size` and inorder range `in_start .. in_idx-1`.
- **Right subtree** uses preorder range `pre_start+left_size+1 .. pre_end` and inorder range `in_idx+1 .. in_end`.

The trickiest line is the preorder split: after the root at `pre_start`, the **next `left_size` elements** are the entire left subtree's preorder, so the right subtree's preorder begins at `pre_start + left_size + 1`.

### Worked Trace
`preorder = [3, 9, 20, 15, 7]`, `inorder = [9, 3, 15, 20, 7]`.

1. `pre_start=0` → `root_val = 3`. In inorder, `in_idx = 1`. `left_size = 1 - 0 = 1`.
   - **Left** child: `preorder[1..1] = [9]`, `inorder[0..0] = [9]`.
   - **Right** child: `preorder[2..4] = [20,15,7]`, `inorder[2..4] = [15,20,7]`.
2. Left call `root_val = 9`, `in_idx = 0`, `left_size = 0` → leaf `9`. Done.
3. Right call `root_val = 20`, in inorder `[15,20,7]` → `in_idx = 3`, `in_start = 2`, `left_size = 3 - 2 = 1`.
   - **Left** child: `preorder[3..3] = [15]`, `inorder[2..2] = [15]` → leaf `15`.
   - **Right** child: `preorder[4..4] = [7]`, `inorder[4..4] = [7]` → leaf `7`.

Reconstructed tree:

```
        3
       / \
      9   20
         /  \
        15   7
```

### Full Rust Solution

```rust
use std::collections::HashMap;

pub struct Solution;

impl Solution {
    pub fn build_tree(preorder: Vec<i32>, inorder: Vec<i32>) -> Option<Box<TreeNode>> {
        let mut inorder_index: HashMap<i32, usize> = HashMap::new();
        for (i, &val) in inorder.iter().enumerate() {
            inorder_index.insert(val, i);
        }
        build(&preorder, 0, 0, inorder.len() as i32 - 1, &inorder_index)
    }
}

// pre_start: start of this subtree in preorder
// in_start, in_end: bounds of this subtree in inorder
fn build(
    preorder: &[i32],
    pre_start: usize,
    in_start: i32,
    in_end: i32,
    inorder_index: &HashMap<i32, usize>,
) -> Option<Box<TreeNode>> {
    if in_start > in_end {
        return None; // empty segment
    }

    let root_val = preorder[pre_start];      // root is first in preorder
    let mut root = Box::new(TreeNode::new(root_val));

    let in_idx = *inorder_index.get(&root_val).unwrap() as i32; // O(1) lookup
    let left_size = (in_idx - in_start) as usize;               // nodes in left subtree

    // Left subtree:  preorder[pre_start+1 .. pre_start+left_size],
    //                inorder[in_start .. in_idx-1]
    root.left = build(preorder, pre_start + 1, in_start, in_idx - 1, inorder_index);

    // Right subtree: preorder[pre_start+left_size+1 .. ],
    //                inorder[in_idx+1 .. in_end]
    root.right = build(preorder, pre_start + left_size + 1, in_idx + 1, in_end, inorder_index);

    Some(root)
}
```

### Complexity

| Metric | Value | Reason |
| --- | --- | --- |
| Time | O(n) | Each node built once; HashMap makes root lookup O(1). |
| Space | O(n) | HashMap of size n + O(h) recursion stack. |

### Note
Without the `HashMap`, finding `in_idx` by linear scan turns this into O(n²) on a skewed tree — a common follow-up is "how do you avoid the linear search?"

---

## 2. Delete Nodes And Return Forest (LC 1110)

### Problem Statement
Given the root of a tree and a list `to_delete` of values to remove, delete those nodes. The result is a **forest** (a list of disjoint trees). Return the roots of the trees in the remaining forest, in any order.

### Intuition — Orphan Detection in Post-order
Deleting a node can **create new roots**: its surviving children lose their parent and become roots of their own trees. So we need two pieces of information at each node during a **post-order** traversal:

1. Is this node **itself** going to be deleted? (membership in a `HashSet<i32>` of the to-delete values).
2. Is this node currently a **root** of the forest — i.e., is it the original root, or did its parent just get deleted?

The rule: **a surviving node becomes a new root exactly when it is a root candidate** (`is_root == true`) — that happens when it is the original tree root, or when its parent was deleted. We pass an `is_root` flag downward; a child is a root candidate iff the current node is being deleted.

Process post-order so children are resolved first:
- Recurse into left and right, passing `is_root = (this node is deleted)`.
- The recursive call returns the (possibly `None`) surviving child to re-link.
- If the current node is deleted, return `None` to its parent (severing the link); its surviving children were already collected as new roots.
- If the current node survives **and** is a root candidate, add it to the result list.

### Full Rust Solution

```rust
use std::collections::HashSet;

pub struct Solution;

impl Solution {
    pub fn del_nodes(root: Option<Box<TreeNode>>, to_delete: Vec<i32>) -> Vec<Box<TreeNode>> {
        let to_delete_set: HashSet<i32> = to_delete.into_iter().collect();
        let mut forest: Vec<Box<TreeNode>> = Vec::new();
        // The original root is a root candidate.
        dfs(root, true, &to_delete_set, &mut forest);
        forest
    }
}

// Returns the surviving node (or None if this node is deleted),
// for the parent to re-link.
fn dfs(
    node: Option<Box<TreeNode>>,
    is_root: bool,
    to_delete: &HashSet<i32>,
    forest: &mut Vec<Box<TreeNode>>,
) -> Option<Box<TreeNode>> {
    let Some(mut node) = node else { return None };

    let deleted = to_delete.contains(&node.val);

    // Children become root candidates iff THIS node is deleted.
    node.left  = dfs(node.left.take(),  deleted, to_delete, forest);
    node.right = dfs(node.right.take(), deleted, to_delete, forest);

    if deleted {
        // Sever from parent by returning None;
        // surviving children were already added to forest above.
        None
    } else if is_root {
        // Surviving root candidate: transfer ownership into forest.
        forest.push(node);
        None // no parent to return to
    } else {
        // Surviving non-root: return for parent re-linking.
        Some(node)
    }
}
```

### Complexity

| Metric | Value | Reason |
| --- | --- | --- |
| Time | O(n) | Each node visited once; set lookups are O(1). |
| Space | O(n) | The to-delete set + O(h) recursion + output forest. |

### Note
The two flags — *am I deleted?* and *am I a root candidate?* — must be kept distinct. A node can be a root candidate yet deleted (it is then simply dropped, and its children become candidates instead).

---

## 3. Binary Tree Cameras (LC 968)

### Problem Statement
Each node can host a camera that monitors itself, its parent, and its immediate children. Return the **minimum number of cameras** needed so that every node is monitored.

### Intuition — Greedy DP with Three States
A clean greedy emerges if every node reports one of three states to its parent:

- **`0 = NOT_COVERED`** — this node has no camera and is not yet monitored; it *needs* its parent to cover it.
- **`1 = HAS_CAMERA`** — this node holds a camera.
- **`2 = COVERED`** — this node is monitored (by a child's camera) but has none itself.

The greedy principle is to **place cameras as high as possible** — specifically at *parents of uncovered nodes*, never on the leaves themselves. Leaves are cheap to cover from above and expensive to equip, so we let parents pay.

Post-order rules at a node with children states `left` and `right`:

1. **Null child** returns `COVERED` (a non-existent node imposes no demand, so it never forces a camera).
2. If **either child is `NOT_COVERED`**, we *must* place a camera here to cover that child → increment count, return `HAS_CAMERA`.
3. If **either child `HAS_CAMERA`**, this node is monitored by that child → return `COVERED`.
4. Otherwise (both children `COVERED`), this node is **not** monitored by anyone yet → return `NOT_COVERED`, deferring the camera to its parent.

**Root special case:** after the traversal, if the root itself comes back `NOT_COVERED`, there is no parent to cover it, so we add one final camera.

### Full Rust Solution

```rust
pub struct Solution;

impl Solution {
    const NOT_COVERED: i32 = 0;
    const HAS_CAMERA: i32  = 1;
    const COVERED: i32     = 2;

    pub fn min_camera_cover(root: Option<Box<TreeNode>>) -> i32 {
        let mut cameras = 0i32;
        // If the root ends up uncovered, it has no parent to cover it.
        if Self::dfs_camera(&root, &mut cameras) == Self::NOT_COVERED {
            cameras += 1;
        }
        cameras
    }

    fn dfs_camera(node: &Option<Box<TreeNode>>, cameras: &mut i32) -> i32 {
        // A null child imposes no demand -> treated as covered.
        let Some(node) = node else { return Self::COVERED };

        let left  = Self::dfs_camera(&node.left, cameras);
        let right = Self::dfs_camera(&node.right, cameras);

        // A child needs coverage -> we must place a camera here.
        if left == Self::NOT_COVERED || right == Self::NOT_COVERED {
            *cameras += 1;
            return Self::HAS_CAMERA;
        }

        // A child has a camera -> this node is monitored.
        if left == Self::HAS_CAMERA || right == Self::HAS_CAMERA {
            return Self::COVERED;
        }

        // Both children covered but none has a camera here:
        // defer the camera to the parent.
        Self::NOT_COVERED
    }
}
```

### Complexity

| Metric | Value | Reason |
| --- | --- | --- |
| Time | O(n) | Single post-order pass. |
| Space | O(h) | Recursion stack. |

### Note
The ordering of the two `if` checks matters: check `NOT_COVERED` **before** `HAS_CAMERA`. An uncovered child is an unconditional demand and must win, even if the other child already has a camera.

---

## Microsoft Interview Style

Microsoft rounds are typically **collaborative and conversational** rather than adversarial. To do well:

- **Clarify first.** Confirm constraints (are values distinct? can the tree be empty? are the to-delete values guaranteed present?) before writing code. Interviewers reward candidates who surface ambiguity.
- **Think out loud and invite input.** Microsoft interviewers often nudge you toward the next idea; treat it as pair-programming, not a test you must complete in silence.
- **Prioritize clarity over cleverness.** Readable code with named states (`NOT_COVERED`, `HAS_CAMERA`, `COVERED`) and clear invariants scores higher than terse tricks.
- **Test your own code.** After writing, walk through a small example by hand — e.g., trace LC 105 on `[3,9,20,15,7]`, or LC 968 on a single-node tree (answer: 1 camera) and a path of three nodes (answer: 1, camera on the middle). Catching your own bug before the interviewer does is a strong positive signal.
- **Discuss complexity and follow-ups.** Be ready for "what if values aren't distinct?" (LC 105 breaks — you'd need positional encoding) and "can you do LC 968 with explicit DP returning min-cameras per state?" (yes, a 3-value DP per node).

---

> **Last Updated:** 2026-06-26
