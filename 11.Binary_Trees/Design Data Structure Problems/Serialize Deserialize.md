> **Topic:** [Binary Trees](../README.md) · **Design 1 of 1**

# Serialize and Deserialize Binary Tree (LC 297)

Design a `Codec` that converts a binary tree to a string and back. Any encoding is acceptable
as long as `deserialize(serialize(root))` reproduces the original tree. The two standard
encodings are **BFS (level-order)** and **DFS (preorder)** — both must explicitly mark `null`
children so structure is recoverable.

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

> **Why null markers matter:** values alone are ambiguous. `[1, 2]` could mean 2 is the left
> or the right child of 1. Recording `null` for absent children removes the ambiguity, so a
> single traversal order fully determines the tree.

---

## Approach 1 — DFS Preorder (recommended)

Write nodes in preorder (Root, Left, Right), emitting `"#"` for `None`. To rebuild, consume
tokens in the same preorder using an index pointer (or a `VecDeque`/iterator).

```rust
use std::collections::VecDeque;

struct Codec;

impl Codec {
    const NULLMARK: &'static str = "#";
    const SEP: &'static str = ",";

    // ---- serialize ----
    fn build_string(node: &Option<Box<TreeNode>>, sb: &mut String) {
        match node {
            None => {
                sb.push_str(Self::NULLMARK);
                sb.push_str(Self::SEP);
            }
            Some(n) => {
                sb.push_str(&n.val.to_string());
                sb.push_str(Self::SEP);
                Self::build_string(&n.left, sb);     // preorder: left then right
                Self::build_string(&n.right, sb);
            }
        }
    }

    fn serialize(root: &Option<Box<TreeNode>>) -> String {
        let mut sb = String::new();
        Self::build_string(root, &mut sb);
        sb
    }

    // ---- deserialize ----
    fn build_tree(tokens: &mut VecDeque<String>) -> Option<Box<TreeNode>> {
        let val = tokens.pop_front().unwrap(); // consume in the SAME preorder
        if val == Self::NULLMARK {
            return None;
        }
        let mut node = Box::new(TreeNode::new(val.parse::<i32>().unwrap()));
        node.left  = Self::build_tree(tokens);
        node.right = Self::build_tree(tokens);
        Some(node)
    }

    fn deserialize(data: &str) -> Option<Box<TreeNode>> {
        let mut tokens: VecDeque<String> = data.split(',')
            .filter(|s| !s.is_empty())
            .map(|s| s.to_string())
            .collect();
        Self::build_tree(&mut tokens)
    }
}
```

**Why it works:** preorder fixes the order in which nodes (and null markers) are emitted.
Deserialization mirrors that exact order, so each recursive call consumes precisely the tokens
that belong to its subtree. The `VecDeque` acts as a shared, advancing cursor across all
recursive frames.

---

## Approach 2 — BFS Level Order

Serialize level by level, writing `"#"` for null children encountered in the queue.
Deserialize by reading the root, then for each non-null node, attach the next two tokens as
its children.

```rust
use std::collections::VecDeque;

struct CodecBFS;

impl CodecBFS {
    const NULLMARK: &'static str = "#";
    const SEP: &'static str = ",";

    fn serialize(root: &Option<Box<TreeNode>>) -> String {
        if root.is_none() {
            return String::new();
        }
        let mut sb = String::new();
        let mut q: VecDeque<Option<&TreeNode>> = VecDeque::new();
        q.push_back(root.as_deref());
        while let Some(node_opt) = q.pop_front() {
            match node_opt {
                None => {
                    sb.push_str(Self::NULLMARK);
                    sb.push_str(Self::SEP);
                }
                Some(node) => {
                    sb.push_str(&node.val.to_string());
                    sb.push_str(Self::SEP);
                    q.push_back(node.left.as_deref());     // enqueue nulls too; they'll be written as #
                    q.push_back(node.right.as_deref());
                }
            }
        }
        sb
    }

    fn deserialize(data: &str) -> Option<Box<TreeNode>> {
        if data.is_empty() {
            return None;
        }
        let tokens: Vec<&str> = data.split(',')
            .filter(|s| !s.is_empty())
            .collect();
        if tokens.is_empty() || tokens[0] == Self::NULLMARK {
            return None;
        }
        // Index-based BFS: track node values and child links in parallel arrays
        let mut vals: Vec<i32> = vec![tokens[0].parse::<i32>().unwrap()];
        let mut lefts: Vec<Option<usize>> = vec![None];
        let mut rights: Vec<Option<usize>> = vec![None];
        let mut queue: VecDeque<usize> = VecDeque::new();
        queue.push_back(0);
        let mut i = 1;
        while let Some(idx) = queue.pop_front() {
            if i < tokens.len() {
                if tokens[i] != Self::NULLMARK {           // left child
                    let child = vals.len();
                    vals.push(tokens[i].parse::<i32>().unwrap());
                    lefts.push(None);
                    rights.push(None);
                    lefts[idx] = Some(child);
                    queue.push_back(child);
                }
                i += 1;
            }
            if i < tokens.len() {
                if tokens[i] != Self::NULLMARK {           // right child
                    let child = vals.len();
                    vals.push(tokens[i].parse::<i32>().unwrap());
                    lefts.push(None);
                    rights.push(None);
                    rights[idx] = Some(child);
                    queue.push_back(child);
                }
                i += 1;
            }
        }
        // Assemble Box<TreeNode> bottom-up from flat arrays
        let n = vals.len();
        let mut nodes: Vec<Option<Box<TreeNode>>> = vals.iter()
            .map(|&v| Some(Box::new(TreeNode::new(v))))
            .collect();
        for idx in (0..n).rev() {
            let left_node = lefts[idx].and_then(|li| nodes[li].take());
            let right_node = rights[idx].and_then(|ri| nodes[ri].take());
            if let Some(ref mut node) = nodes[idx] {
                node.left = left_node;
                node.right = right_node;
            }
        }
        nodes[0].take()
    }
}
```

---

## Follow-ups

### Codec for a BST (LC 449) — no null markers needed
A BST's structure is recoverable from preorder alone (values are unique and ordered), so you
can drop the `"#"` markers and rebuild using an `upperBound` to decide where each value
belongs.

```rust
use std::collections::VecDeque;

fn deserialize_bst(q: &mut VecDeque<i32>, lower: i32, upper: i32) -> Option<Box<TreeNode>> {
    if q.is_empty() {
        return None;
    }
    let val = *q.front().unwrap();
    if val < lower || val > upper {     // not in this subtree's range
        return None;
    }
    q.pop_front();
    let mut node = Box::new(TreeNode::new(val));
    node.left  = deserialize_bst(q, lower, val);
    node.right = deserialize_bst(q, val, upper);
    Some(node)
}
```
This is more compact (no null tokens) and is the reason serializing a **BST** is cheaper than
a general binary tree.

### Codec for an N-ary tree (LC 428)
Emit each node's value followed by its **child count**, then recurse over children. The count
tells deserialization how many children to read — no per-child null markers required.

```rust
// serialize: val + "," + children.len() + "," + ...children...
fn encode(node: &Node, sb: &mut String) {
    sb.push_str(&node.val.to_string());
    sb.push(',');
    sb.push_str(&node.children.len().to_string());
    sb.push(',');
    for child in &node.children {
        encode(child, sb);
    }
}
```

---

## Complexity

| Approach | serialize Time | deserialize Time | Space |
|---|---|---|---|
| DFS Preorder | O(n) | O(n) | O(n) string + O(h) stack |
| BFS Level Order | O(n) | O(n) | O(n) string + O(w) queue |
| BST (no markers) | O(n) | O(n) | O(n) + O(h) |
| N-ary (child count) | O(n) | O(n) | O(n) + O(h) |

Both core approaches visit every node a constant number of times → **O(n)** time, and the
serialized string is **O(n)**. DFS uses recursion stack O(h); BFS uses a queue O(w).

---

## API Design Notes
- Keep `serialize`/`deserialize` **symmetric**: the same traversal order on both sides is what
  guarantees correctness.
- Choose a **separator** that can't appear in a value (here `","`); for arbitrary string data,
  length-prefix each token instead.
- Make the codec **stateless** between calls (no leftover index/queue), so one instance can be
  reused safely.

---

### Related
- [Tree Traversals](../Patterns/Tree%20Traversals.md) — the DFS/BFS orders this builds on
- [Tree Construction](../Patterns/Tree%20Construction.md) — rebuilding trees from sequences

> **Last Updated:** 2026-06-26
