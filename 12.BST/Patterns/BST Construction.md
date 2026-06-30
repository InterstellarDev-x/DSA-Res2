> **Topic:** [Binary Search Trees](../README.md) · **Pattern 3 of 4**

# BST Construction

**Core idea:** rebuilding a BST is about **identifying the root and partitioning the rest** into
the left subtree (smaller keys) and right subtree (larger keys), then recursing. From a *sorted
array* the balanced root is the **middle** element. From a *preorder traversal* the **first**
element is the root and an **upper bound** cleanly separates the two subtrees in a single O(n)
pass.

```java
public class TreeNode {
    int val;
    TreeNode left, right;
    TreeNode(int val) { this.val = val; }
}
```

> **Recognition signals:** "build a height-balanced BST", "from a sorted array", "from preorder",
> "rebalance an unbalanced BST", "minimum height".

---

## 1. Convert Sorted Array to BST (LC 108) — Easy

The array is already sorted (= an inorder sequence). Choosing the **middle** element as the root
at every level guarantees a **height-balanced** tree: left half builds the left subtree, right half
the right subtree.

```java
public TreeNode sortedArrayToBST(int[] nums) {
    return build(nums, 0, nums.length - 1);
}

private TreeNode build(int[] nums, int lo, int hi) {
    if (lo > hi) return null;
    int mid = lo + (hi - lo) / 2;            // middle ⇒ balanced; avoids overflow
    TreeNode root = new TreeNode(nums[mid]);
    root.left  = build(nums, lo, mid - 1);
    root.right = build(nums, mid + 1, hi);
    return root;
}
```

**Complexity:** Time O(n); Space O(log n) recursion (balanced). Picking the lower or upper middle
both yield valid balanced trees.

---

## 2. Construct BST from Preorder Traversal (LC 1008) — Medium

In a preorder, the **first** value is the root. Every subsequent value smaller than the root
belongs to the left subtree; the first value larger starts the right subtree. The elegant **O(n)**
solution uses a single shared index and an **upper bound** that says "consume values that still
belong to the current subtree."

```java
private int idx = 0;

public TreeNode bstFromPreorder(int[] preorder) {
    idx = 0;
    return build(preorder, Integer.MAX_VALUE);
}

private TreeNode build(int[] pre, int bound) {
    if (idx == pre.length || pre[idx] > bound) return null;  // nothing more fits here
    TreeNode root = new TreeNode(pre[idx++]);
    root.left  = build(pre, root.val);   // left subtree: values < root.val
    root.right = build(pre, bound);      // right subtree: values < inherited bound
    return root;
}
```

Each element is consumed exactly once → **O(n)**.

A simpler-to-explain (but O(n²) worst case) recursion uses explicit lo/hi value bounds:

```java
private int p = 0;

public TreeNode bstFromPreorderBounds(int[] pre) {
    p = 0;
    return helper(pre, Integer.MIN_VALUE, Integer.MAX_VALUE);
}

private TreeNode helper(int[] pre, int lower, int upper) {
    if (p == pre.length) return null;
    int v = pre[p];
    if (v < lower || v > upper) return null;
    p++;
    TreeNode node = new TreeNode(v);
    node.left  = helper(pre, lower, v);
    node.right = helper(pre, v, upper);
    return node;
}
```

**Complexity:** upper-bound version O(n) time, O(h) space.

---

## 3. Balance a Binary Search Tree (LC 1382) — Medium

Two-step composition of the previous ideas:

1. **Inorder traversal → sorted array** (a BST's inorder is sorted, so no actual sorting needed).
2. **Sorted array → height-balanced BST** using the LC 108 mid-element technique.

```java
public TreeNode balanceBST(TreeNode root) {
    List<Integer> sorted = new ArrayList<>();
    inorder(root, sorted);
    return build(sorted, 0, sorted.size() - 1);
}

private void inorder(TreeNode node, List<Integer> out) {
    if (node == null) return;
    inorder(node.left, out);
    out.add(node.val);
    inorder(node.right, out);
}

private TreeNode build(List<Integer> a, int lo, int hi) {
    if (lo > hi) return null;
    int mid = lo + (hi - lo) / 2;
    TreeNode root = new TreeNode(a.get(mid));
    root.left  = build(a, lo, mid - 1);
    root.right = build(a, mid + 1, hi);
    return root;
}
```

**Complexity:** Time O(n); Space O(n) for the list + O(log n) recursion.

---

## Dry Run — Sorted Array `[1,2,3,4,5,6,7]` → Balanced BST

`build(0,6)`: `mid = 0 + (6-0)/2 = 3` → root `4`.
- left `build(0,2)`: `mid = 1` → `2`; left `build(0,0)` → `1`; right `build(2,2)` → `3`.
- right `build(4,6)`: `mid = 5` → `6`; left `build(4,4)` → `5`; right `build(6,6)` → `7`.

```
        4
       / \
      2   6
     / \ / \
    1  3 5  7
```

Perfectly balanced, height = 2 = ⌊log₂ 7⌋. Inorder reproduces `1 2 3 4 5 6 7`.

---

## Summary

| Problem | Technique | Time | Space |
|---|---|---|---|
| Sorted Array → BST (108) | mid element as root, recurse halves | O(n) | O(log n) |
| Construct from Preorder (1008) | first = root, upper-bound consume | O(n) | O(h) |
| Balance a BST (1382) | inorder → sorted array → LC 108 | O(n) | O(n) |

> See also the binary-tree [construction patterns](../../11.Binary_Trees/Patterns/Tree%20Construction.md)
> (from preorder+inorder) for trees without the BST ordering shortcut.

> **Last Updated:** 2026-06-26
