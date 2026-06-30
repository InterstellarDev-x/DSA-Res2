> **Topic:** [Binary Search Trees](../README.md) · **Tips 2 of 4**

# Common Mistakes — BST

Ten bugs that fail hidden test cases or get flagged in interviews, each with the wrong and the
correct code.

```java
public class TreeNode {
    int val;
    TreeNode left, right;
    TreeNode(int val) { this.val = val; }
}
```

---

### 1. Validating only against the direct parent
The deepest descendants must respect *every* ancestor's bound, not just the parent.

```java
// WRONG — passes for [5,1,4,null,null,3,6] although 3 < 5
boolean bad(TreeNode n) {
    if (n == null) return true;
    if (n.left  != null && n.left.val  >= n.val) return false;
    if (n.right != null && n.right.val <= n.val) return false;
    return bad(n.left) && bad(n.right);
}
// CORRECT — bounds flow from all ancestors
boolean good(TreeNode n, long lo, long hi) {
    if (n == null) return true;
    if (n.val <= lo || n.val >= hi) return false;
    return good(n.left, lo, n.val) && good(n.right, n.val, hi);
}
```

### 2. Integer overflow in the bounds
```java
// WRONG — fails when a node holds Integer.MIN_VALUE / MAX_VALUE
boolean bad(TreeNode n, int lo, int hi) { ... }
// CORRECT — use long sentinels
boolean good(TreeNode n, long lo, long hi) { ... }   // start with Long.MIN/MAX_VALUE
```

### 3. Kth-smallest off-by-one
```java
// WRONG — decrements then compares, returning the (k+1)-th
if (k-- == 0) return cur.val;
// CORRECT — pre-decrement, compare to 0
if (--k == 0) return cur.val;
```

### 4. Delete not handling the two-children case
```java
// WRONG — only patches leaves / single children, corrupts two-child nodes
if (root.val == key) return null;
// CORRECT — splice single child, else promote inorder successor
if (root.left == null)  return root.right;
if (root.right == null) return root.left;
TreeNode s = root.right; while (s.left != null) s = s.left;
root.val = s.val; root.right = deleteNode(root.right, s.val);
```

### 5. Recover BST: ignoring the adjacent-vs-non-adjacent case
```java
// WRONG — assumes two descents always exist; breaks for an adjacent swap (one descent)
if (descent) { if (first == null) first = prev; else second = node; }
// CORRECT — first only on the first descent; second on EVERY descent
if (descent) { if (first == null) first = prev; second = node; }
```

### 6. Forgetting "strictly" increasing (duplicates)
```java
// WRONG — accepts equal neighbors as valid
if (prev != null && node.val < prev.val) return false;
// CORRECT — a BST inorder must strictly increase
if (prev != null && node.val <= prev.val) return false;
```

### 7. Using the full-tree LCA algorithm on a BST
```java
// WRONG — O(n), searches both subtrees, ignores ordering
TreeNode lca(TreeNode r, TreeNode p, TreeNode q) {
    if (r == null || r == p || r == q) return r;
    TreeNode l = lca(r.left, p, q), rr = lca(r.right, p, q);
    return (l != null && rr != null) ? r : (l != null ? l : rr);
}
// CORRECT — O(h) descent to the split point
while (cur != null) {
    if (p.val < cur.val && q.val < cur.val)      cur = cur.left;
    else if (p.val > cur.val && q.val > cur.val) cur = cur.right;
    else return cur;
}
```

### 8. BST Iterator not pushing the leftmost spine
```java
// WRONG — pushes only the node, so next() can skip its left descendants
private void push(TreeNode n) { if (n != null) stack.push(n); }
// CORRECT — push the node and ALL of its left children
private void pushLeftmost(TreeNode n) { while (n != null) { stack.push(n); n = n.left; } }
```

### 9. Range Sum not pruning
```java
// WRONG — visits every node, ignoring the BST property (O(n) always)
int sum(TreeNode n, int lo, int hi) {
    if (n == null) return 0;
    int s = (n.val >= lo && n.val <= hi) ? n.val : 0;
    return s + sum(n.left, lo, hi) + sum(n.right, lo, hi);
}
// CORRECT — skip whole subtrees out of range
int sum(TreeNode n, int lo, int hi) {
    if (n == null) return 0;
    if (n.val < lo)  return sum(n.right, lo, hi);
    if (n.val > hi)  return sum(n.left, lo, hi);
    return n.val + sum(n.left, lo, hi) + sum(n.right, lo, hi);
}
```

### 10. Greater Tree: wrong traversal direction
```java
// WRONG — normal inorder (ascending) accumulates SMALLER keys
inorder(n.left); sum += n.val; n.val = sum; inorder(n.right);
// CORRECT — reverse inorder (descending) accumulates GREATER keys
reverse(n.right); sum += n.val; n.val = sum; reverse(n.left);
```

---

> **Last Updated:** 2026-06-26
