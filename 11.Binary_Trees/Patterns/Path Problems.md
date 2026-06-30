> **Topic:** [Binary Trees](../README.md) · **Pattern 3 of 5**

# Path Problems

A "path" in a binary tree is a sequence of connected nodes. Problems differ by **where the
path may start/end** (root→leaf, any→any), and whether you **collect** paths or **score**
them. Two recurring techniques: **leaf-base-case checks** and **backtracking** for collection.

```java
public class TreeNode {
    int val;
    TreeNode left, right;
    TreeNode(int val) { this.val = val; }
}
```

---

## 1. Path Sum (LC 112) — does a root→leaf path equal target?

The base case is the **leaf check**: a leaf has no children, so the path ends there.

```java
public boolean hasPathSum(TreeNode root, int targetSum) {
    if (root == null) return false;
    if (root.left == null && root.right == null)      // leaf
        return root.val == targetSum;
    int remaining = targetSum - root.val;
    return hasPathSum(root.left, remaining)
        || hasPathSum(root.right, remaining);
}
```

> Don't use `root == null && targetSum == 0` as the success base case — an empty child of a
> one-child node would falsely succeed. Always check the **leaf** condition explicitly.

---

## 2. Path Sum II (LC 113) — collect all root→leaf paths summing to target

Classic **backtracking**: add the node to the path, recurse, then *remove it* so siblings
start from a clean path.

```java
public List<List<Integer>> pathSum(TreeNode root, int targetSum) {
    List<List<Integer>> result = new ArrayList<>();
    backtrack(root, targetSum, new ArrayList<>(), result);
    return result;
}

private void backtrack(TreeNode node, int remaining,
                       List<Integer> path, List<List<Integer>> result) {
    if (node == null) return;
    path.add(node.val);                                  // choose
    if (node.left == null && node.right == null && remaining == node.val) {
        result.add(new ArrayList<>(path));               // copy! the path is mutable
    } else {
        backtrack(node.left,  remaining - node.val, path, result);
        backtrack(node.right, remaining - node.val, path, result);
    }
    path.remove(path.size() - 1);                        // un-choose (backtrack)
}
```

> ⚠️ Two non-negotiables: **copy** the path when recording (`new ArrayList<>(path)`), and
> **remove** the last element on the way out. Forgetting either is the #1 bug here.

---

## 3. Diameter of Binary Tree (LC 543)

The diameter is the longest path *in edges* between any two nodes — it **need not pass
through the root**. Compute each node's depth bottom-up while updating a global max of
`leftDepth + rightDepth`.

```java
private int maxDiameter = 0;

public int diameterOfBinaryTree(TreeNode root) {
    depth(root);
    return maxDiameter;
}

private int depth(TreeNode node) {
    if (node == null) return 0;
    int left  = depth(node.left);
    int right = depth(node.right);
    maxDiameter = Math.max(maxDiameter, left + right);   // path THROUGH this node
    return 1 + Math.max(left, right);                    // depth returned to parent
}
```

**The dual-purpose recursion insight:** the helper *returns* depth to its parent but
*updates a global* with the through-node path length. The thing you return and the thing you
maximize are different.

---

## 4. Binary Tree Maximum Path Sum (LC 124) — Hard

Same dual-purpose shape as diameter, with two twists:
1. **Prune negative subtrees:** `gain = Math.max(0, gain)` — a negative subtree contributes
   nothing, so we treat it as 0.
2. The best path **through** a node is `node.val + leftGain + rightGain`, but the value
   **returned** to the parent can only follow one branch:
   `node.val + max(leftGain, rightGain)`.

```java
private int maxSum = Integer.MIN_VALUE;

public int maxPathSum(TreeNode root) {
    gain(root);
    return maxSum;
}

private int gain(TreeNode node) {
    if (node == null) return 0;
    int leftGain  = Math.max(0, gain(node.left));    // ignore negative contributions
    int rightGain = Math.max(0, gain(node.right));
    maxSum = Math.max(maxSum, node.val + leftGain + rightGain);  // path through node
    return node.val + Math.max(leftGain, rightGain);             // extendable to parent
}
```

---

## 5. Sum Root to Leaf Numbers (LC 129)

Each root→leaf path spells a number (e.g. `1→2→3` = 123). Carry the running value **down**
the recursion: `current = current * 10 + node.val`.

```java
public int sumNumbers(TreeNode root) {
    return dfs(root, 0);
}

private int dfs(TreeNode node, int current) {
    if (node == null) return 0;
    current = current * 10 + node.val;
    if (node.left == null && node.right == null) return current;   // leaf -> the number
    return dfs(node.left, current) + dfs(node.right, current);
}
```

This is **top-down** accumulation — state flows down via the parameter, results sum on the
way back up.

---

## 6. Binary Tree Paths (LC 257) — all root→leaf paths as strings

Build a `"a->b->c"` string per path. Use a `StringBuilder` with length-rollback (backtrack),
or simple string concatenation passed down.

```java
public List<String> binaryTreePaths(TreeNode root) {
    List<String> result = new ArrayList<>();
    if (root != null) dfs(root, new StringBuilder(), result);
    return result;
}

private void dfs(TreeNode node, StringBuilder sb, List<String> result) {
    int len = sb.length();                       // remember length to roll back
    if (sb.length() > 0) sb.append("->");
    sb.append(node.val);
    if (node.left == null && node.right == null) {
        result.add(sb.toString());
    } else {
        if (node.left  != null) dfs(node.left,  sb, result);
        if (node.right != null) dfs(node.right, sb, result);
    }
    sb.setLength(len);                           // backtrack the StringBuilder
}
```

Simpler (less efficient) alternative: pass `path + "->" + node.val` or
`String.join("->", list)` — fine for interview clarity.

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
