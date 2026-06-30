> **Topic:** [Binary Trees](../README.md) · **Design 1 of 1**

# Serialize and Deserialize Binary Tree (LC 297)

Design a `Codec` that converts a binary tree to a string and back. Any encoding is acceptable
as long as `deserialize(serialize(root))` reproduces the original tree. The two standard
encodings are **BFS (level-order)** and **DFS (preorder)** — both must explicitly mark `null`
children so structure is recoverable.

```java
public class TreeNode {
    int val;
    TreeNode left, right;
    TreeNode(int val) { this.val = val; }
}
```

> **Why null markers matter:** values alone are ambiguous. `[1, 2]` could mean 2 is the left
> or the right child of 1. Recording `null` for absent children removes the ambiguity, so a
> single traversal order fully determines the tree.

---

## Approach 1 — DFS Preorder (recommended)

Write nodes in preorder (Root, Left, Right), emitting `"#"` for `null`. To rebuild, consume
tokens in the same preorder using an index pointer (or a `Queue`/iterator).

```java
public class Codec {
    private static final String NULL = "#";
    private static final String SEP  = ",";

    // ---- serialize ----
    public String serialize(TreeNode root) {
        StringBuilder sb = new StringBuilder();
        buildString(root, sb);
        return sb.toString();
    }

    private void buildString(TreeNode node, StringBuilder sb) {
        if (node == null) {
            sb.append(NULL).append(SEP);
            return;
        }
        sb.append(node.val).append(SEP);
        buildString(node.left, sb);     // preorder: left then right
        buildString(node.right, sb);
    }

    // ---- deserialize ----
    public TreeNode deserialize(String data) {
        Queue<String> tokens = new LinkedList<>(Arrays.asList(data.split(SEP)));
        return buildTree(tokens);
    }

    private TreeNode buildTree(Queue<String> tokens) {
        String val = tokens.poll();          // consume in the SAME preorder
        if (NULL.equals(val)) return null;
        TreeNode node = new TreeNode(Integer.parseInt(val));
        node.left  = buildTree(tokens);
        node.right = buildTree(tokens);
        return node;
    }
}
```

**Why it works:** preorder fixes the order in which nodes (and null markers) are emitted.
Deserialization mirrors that exact order, so each recursive call consumes precisely the tokens
that belong to its subtree. The `Queue` acts as a shared, advancing cursor across all
recursive frames.

---

## Approach 2 — BFS Level Order

Serialize level by level, writing `"#"` for null children encountered in the queue.
Deserialize by reading the root, then for each non-null node, attach the next two tokens as
its children.

```java
public class CodecBFS {
    private static final String NULL = "#";
    private static final String SEP  = ",";

    public String serialize(TreeNode root) {
        if (root == null) return "";
        StringBuilder sb = new StringBuilder();
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        while (!queue.isEmpty()) {
            TreeNode node = queue.poll();
            if (node == null) {
                sb.append(NULL).append(SEP);
            } else {
                sb.append(node.val).append(SEP);
                queue.offer(node.left);     // enqueue nulls too; they'll be written as #
                queue.offer(node.right);
            }
        }
        return sb.toString();
    }

    public TreeNode deserialize(String data) {
        if (data.isEmpty()) return null;
        String[] tokens = data.split(SEP);
        TreeNode root = new TreeNode(Integer.parseInt(tokens[0]));
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        int i = 1;
        while (!queue.isEmpty()) {
            TreeNode node = queue.poll();
            if (!NULL.equals(tokens[i])) {           // left child
                node.left = new TreeNode(Integer.parseInt(tokens[i]));
                queue.offer(node.left);
            }
            i++;
            if (!NULL.equals(tokens[i])) {           // right child
                node.right = new TreeNode(Integer.parseInt(tokens[i]));
                queue.offer(node.right);
            }
            i++;
        }
        return root;
    }
}
```

---

## Follow-ups

### Codec for a BST (LC 449) — no null markers needed
A BST's structure is recoverable from preorder alone (values are unique and ordered), so you
can drop the `"#"` markers and rebuild using an `upperBound` to decide where each value
belongs.

```java
public TreeNode deserializeBST(Queue<Integer> q, int lower, int upper) {
    if (q.isEmpty()) return null;
    int val = q.peek();
    if (val < lower || val > upper) return null;     // not in this subtree's range
    q.poll();
    TreeNode node = new TreeNode(val);
    node.left  = deserializeBST(q, lower, val);
    node.right = deserializeBST(q, val, upper);
    return node;
}
```
This is more compact (no null tokens) and is the reason serializing a **BST** is cheaper than
a general binary tree.

### Codec for an N-ary tree (LC 428)
Emit each node's value followed by its **child count**, then recurse over children. The count
tells deserialization how many children to read — no per-child null markers required.

```java
// serialize: val + "," + children.size() + "," + ...children...
private void encode(Node node, StringBuilder sb) {
    sb.append(node.val).append(",").append(node.children.size()).append(",");
    for (Node child : node.children) encode(child, sb);
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
