> **Topic:** [Binary Trees](../README.md) · **Pattern 2 of 4**

# Binary Trees — Common Mistakes

Ten bugs that show up over and over in binary-tree interviews. Each one pairs the broken version (❌) with the fix (✅). Node definition used throughout:

```java
public class TreeNode {
    int val;
    TreeNode left, right;
    TreeNode(int val) { this.val = val; }
}
```

---

## 1. Missing null check before accessing `node.left` / `node.right`

Dereferencing a child without first confirming the node exists is the classic `NullPointerException`. Always handle `null` as the first line.

❌ **Wrong**
```java
int maxDepth(TreeNode node) {
    int left = maxDepth(node.left);    // NPE when node == null
    int right = maxDepth(node.right);
    return 1 + Math.max(left, right);
}
```

✅ **Correct**
```java
int maxDepth(TreeNode node) {
    if (node == null) return 0;        // base case first
    int left = maxDepth(node.left);
    int right = maxDepth(node.right);
    return 1 + Math.max(left, right);
}
```

---

## 2. Diameter: returning the depth/diameter instead of updating a global

The helper must **return the height** but **record the diameter in a global**. Returning the local `left + right` upward corrupts the height calculation of ancestors.

❌ **Wrong**
```java
int diameter(TreeNode node) {
    if (node == null) return 0;
    int left = diameter(node.left);
    int right = diameter(node.right);
    return left + right;               // this is NOT a height; ancestors break
}
```

✅ **Correct**
```java
int[] best = new int[1];

int height(TreeNode node) {
    if (node == null) return 0;
    int left = height(node.left);
    int right = height(node.right);
    best[0] = Math.max(best[0], left + right);   // diameter recorded globally
    return 1 + Math.max(left, right);            // height returned upward
}
```

---

## 3. Max Path Sum: forgetting `Math.max(0, gain)`

Without clamping, a negative subtree contribution is allowed to drag the path total down, producing wrong answers on trees with negative values.

❌ **Wrong**
```java
int gain(TreeNode node) {
    if (node == null) return 0;
    int left = gain(node.left);        // may be negative and still used
    int right = gain(node.right);
    best = Math.max(best, node.val + left + right);
    return node.val + Math.max(left, right);
}
```

✅ **Correct**
```java
int gain(TreeNode node) {
    if (node == null) return 0;
    int left  = Math.max(0, gain(node.left));    // prune negatives
    int right = Math.max(0, gain(node.right));
    best = Math.max(best, node.val + left + right);
    return node.val + Math.max(left, right);
}
```

---

## 4. Path Sum II: not backtracking (forgetting to remove from the list)

Reusing a shared list without removing the current node after recursion leaves stale entries, so sibling paths inherit nodes that aren't on them.

❌ **Wrong**
```java
void dfs(TreeNode node, int target, List<Integer> path, List<List<Integer>> res) {
    if (node == null) return;
    path.add(node.val);
    if (node.left == null && node.right == null && node.val == target)
        res.add(new ArrayList<>(path));
    dfs(node.left,  target - node.val, path, res);
    dfs(node.right, target - node.val, path, res);
    // missing removal — path keeps growing across branches
}
```

✅ **Correct**
```java
void dfs(TreeNode node, int target, List<Integer> path, List<List<Integer>> res) {
    if (node == null) return;
    path.add(node.val);
    if (node.left == null && node.right == null && node.val == target)
        res.add(new ArrayList<>(path));
    dfs(node.left,  target - node.val, path, res);
    dfs(node.right, target - node.val, path, res);
    path.remove(path.size() - 1);      // backtrack on every return
}
```

---

## 5. Level Order: not capturing size before the inner loop

Reading `queue.size()` inside the loop condition lets the bound drift as children are enqueued, merging levels together.

❌ **Wrong**
```java
while (!queue.isEmpty()) {
    List<Integer> level = new ArrayList<>();
    for (int i = 0; i < queue.size(); i++) {   // size() changes mid-loop
        TreeNode node = queue.poll();
        level.add(node.val);
        if (node.left != null) queue.offer(node.left);
        if (node.right != null) queue.offer(node.right);
    }
    result.add(level);
}
```

✅ **Correct**
```java
while (!queue.isEmpty()) {
    int size = queue.size();           // snapshot the level boundary
    List<Integer> level = new ArrayList<>();
    for (int i = 0; i < size; i++) {
        TreeNode node = queue.poll();
        level.add(node.val);
        if (node.left != null) queue.offer(node.left);
        if (node.right != null) queue.offer(node.right);
    }
    result.add(level);
}
```

---

## 6. Construct from traversals: off-by-one in `leftSize`

When splitting inorder around the root, the count of left-subtree nodes is `inIdx - inStart` (a *count* of elements strictly before the root), not `inIdx - inStart + 1` (which wrongly includes the root).

❌ **Wrong**
```java
int inIdx = inMap.get(rootVal);
int leftSize = inIdx - inStart + 1;    // includes the root — too large by one
```

✅ **Correct**
```java
int inIdx = inMap.get(rootVal);
int leftSize = inIdx - inStart;        // nodes strictly left of the root
// use leftSize to slice the preorder/postorder ranges for the left subtree
```

---

## 7. Binary Tree Cameras: missing the root check after recursion

The greedy post-order returns states (0 = not covered, 1 = has camera, 2 = covered). After the recursion completes, if the **root itself** comes back uncovered (state 0), you must add one more camera — easy to forget.

❌ **Wrong**
```java
int cameras = 0;

int minCameraCover(TreeNode root) {
    dfs(root);
    return cameras;                    // ignores an uncovered root → undercount
}
```

✅ **Correct**
```java
int cameras = 0;

int minCameraCover(TreeNode root) {
    if (dfs(root) == 0) cameras++;     // root left uncovered → place a camera
    return cameras;
}

// dfs returns: 0 = needs cover, 1 = has camera, 2 = covered
int dfs(TreeNode node) {
    if (node == null) return 2;
    int left = dfs(node.left);
    int right = dfs(node.right);
    if (left == 0 || right == 0) { cameras++; return 1; }
    if (left == 1 || right == 1) return 2;
    return 0;
}
```

---

## 8. LCA: returning `null` incorrectly for nodes present in the tree

The lowest-common-ancestor base case must return the node when it matches `p` or `q`. Returning `null` for a matched node (or short-circuiting before checking the match) loses the answer.

❌ **Wrong**
```java
TreeNode lca(TreeNode node, TreeNode p, TreeNode q) {
    if (node == null) return null;
    if (node.left == null && node.right == null) return null;  // drops leaf matches
    TreeNode left = lca(node.left, p, q);
    TreeNode right = lca(node.right, p, q);
    if (left != null && right != null) return node;
    return left != null ? left : right;
}
```

✅ **Correct**
```java
TreeNode lca(TreeNode node, TreeNode p, TreeNode q) {
    if (node == null || node == p || node == q) return node;   // match returns self
    TreeNode left = lca(node.left, p, q);
    TreeNode right = lca(node.right, p, q);
    if (left != null && right != null) return node;            // split → this is LCA
    return left != null ? left : right;
}
```

---

## 9. Zigzag: using `result.get(level).add(0, val)` — O(n) — instead of a deque

Inserting at index 0 of an `ArrayList` shifts every element, turning each level into O(n²). Use a `LinkedList` (or `Deque`) and `addFirst` for O(1) front insertion.

❌ **Wrong**
```java
List<Integer> level = new ArrayList<>();
// ... for each node on the level:
if (leftToRight) level.add(node.val);
else             level.add(0, node.val);   // O(n) shift each insert
```

✅ **Correct**
```java
LinkedList<Integer> level = new LinkedList<>();
// ... for each node on the level:
if (leftToRight) level.addLast(node.val);   // O(1)
else             level.addFirst(node.val);  // O(1)
result.add(level);
```

---

## 10. Max Width: integer overflow without index normalization

Heap-style indices (`2*i`, `2*i+1`) blow past `int`/`long` range on deep sparse trees. Normalize each level against its leftmost index so only the differences — which is all the width needs — are preserved.

❌ **Wrong**
```java
for (int i = 0; i < size; i++) {
    TreeNode node = nodes.poll();
    int idx = idxs.poll();                 // grows exponentially → overflow
    if (node.left != null)  { nodes.offer(node.left);  idxs.offer(2 * idx); }
    if (node.right != null) { nodes.offer(node.right); idxs.offer(2 * idx + 1); }
}
```

✅ **Correct**
```java
int leftmost = idxs.peek();                // base for this level
for (int i = 0; i < size; i++) {
    TreeNode node = nodes.poll();
    int idx = idxs.poll() - leftmost;      // NORMALIZE → indices stay small
    if (i == 0) first = idx;
    if (i == size - 1) last = idx;
    if (node.left != null)  { nodes.offer(node.left);  idxs.offer(2 * idx); }
    if (node.right != null) { nodes.offer(node.right); idxs.offer(2 * idx + 1); }
}
max = Math.max(max, last - first + 1);
```

---

> **Last Updated:** 2026-06-26
