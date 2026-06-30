> **Topic:** [Binary Trees](../README.md) · **Pattern 5 of 5**

# Tree DP / Advanced

**Core idea:** a *post-order* DFS where each node **returns aggregated information about its
subtree** to its parent, and (optionally) a **global result** is updated at each node. The
recurrence lives in how a parent combines its children's returns.

```java
public class TreeNode {
    int val;
    TreeNode left, right;
    TreeNode(int val) { this.val = val; }
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

```java
public int rob(TreeNode root) {
    int[] res = robSub(root);
    return Math.max(res[0], res[1]);
}

// returns {notRobbingThisNode, robbingThisNode}
private int[] robSub(TreeNode node) {
    if (node == null) return new int[]{0, 0};
    int[] l = robSub(node.left);
    int[] r = robSub(node.right);
    int notRob = Math.max(l[0], l[1]) + Math.max(r[0], r[1]);  // children choose freely
    int rob    = node.val + l[0] + r[0];                       // children must NOT be robbed
    return new int[]{notRob, rob};
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

```java
private int cameras = 0;
private static final int NOT_COVERED = 0, HAS_CAMERA = 1, COVERED = 2;

public int minCameraCover(TreeNode root) {
    if (dfs(root) == NOT_COVERED) cameras++;   // root left uncovered -> place a camera
    return cameras;
}

private int dfs(TreeNode node) {
    if (node == null) return COVERED;          // null is implicitly covered
    int l = dfs(node.left);
    int r = dfs(node.right);
    if (l == NOT_COVERED || r == NOT_COVERED) {
        cameras++;
        return HAS_CAMERA;
    }
    if (l == HAS_CAMERA || r == HAS_CAMERA) return COVERED;
    return NOT_COVERED;
}
```

---

## 3. Distribute Coins in Binary Tree (LC 979)

Each node should end with exactly 1 coin. The number of moves across an edge equals the
**absolute surplus/deficit** flowing through it. At each node compute
`excess = leftExcess + rightExcess + node.val − 1`; add `|excess|` to the answer and return
`excess` to the parent.

```java
private int moves = 0;

public int distributeCoins(TreeNode root) {
    excess(root);
    return moves;
}

private int excess(TreeNode node) {
    if (node == null) return 0;
    int left  = excess(node.left);
    int right = excess(node.right);
    moves += Math.abs(left) + Math.abs(right);     // coins crossing both child edges
    return node.val + left + right - 1;            // surplus(+)/deficit(-) passed up
}
```

---

## 4. Lowest Common Ancestor (LC 236)

If the current node is `p` or `q`, return it. Recurse both sides; if **both** return non-null,
the current node is the LCA. Otherwise propagate the non-null side up.

```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    if (root == null || root == p || root == q) return root;
    TreeNode left  = lowestCommonAncestor(root.left,  p, q);
    TreeNode right = lowestCommonAncestor(root.right, p, q);
    if (left != null && right != null) return root;   // p and q on opposite sides -> LCA
    return left != null ? left : right;               // both on one side (or neither)
}
```

**Why early-return works:** a node returns non-null iff its subtree contains `p` or `q`. The
first node to receive non-null from *both* sides is the split point = the LCA. If one of
`p`/`q` is an ancestor of the other, the early `root == p` return makes that ancestor the LCA.

---

## 5. LCA of Deepest Leaves (LC 1123)

Return a `(node, depth)` pair upward. Pick the side with greater depth; if both depths are
equal, the **current node** is the LCA of all deepest leaves below it.

```java
private static class Result {
    TreeNode node; int depth;
    Result(TreeNode node, int depth) { this.node = node; this.depth = depth; }
}

public TreeNode lcaDeepestLeaves(TreeNode root) {
    return dfs(root).node;
}

private Result dfs(TreeNode node) {
    if (node == null) return new Result(null, 0);
    Result l = dfs(node.left);
    Result r = dfs(node.right);
    if (l.depth == r.depth) return new Result(node, l.depth + 1);  // balanced -> this node
    return l.depth > r.depth
        ? new Result(l.node, l.depth + 1)
        : new Result(r.node, r.depth + 1);
}
```

---

## 6. Delete Nodes and Return Forest (LC 1110)

Put the to-delete values in a `Set`. Post-order: a child that survives **becomes a new root**
if its parent was deleted (or if it is the original root). When a node is deleted, its
surviving children are added to the forest; the node returns `null` to detach itself.

```java
public List<TreeNode> delNodes(TreeNode root, int[] to_delete) {
    Set<Integer> toDelete = new HashSet<>();
    for (int d : to_delete) toDelete.add(d);
    List<TreeNode> forest = new ArrayList<>();
    if (dfs(root, true, toDelete, forest) != null) forest.add(root);
    return forest;
}

// returns the node if it survives, else null (so the parent can detach it)
private TreeNode dfs(TreeNode node, boolean isRoot,
                     Set<Integer> toDelete, List<TreeNode> forest) {
    if (node == null) return null;
    boolean deleted = toDelete.contains(node.val);
    if (isRoot && !deleted) { /* will be added by caller */ }
    node.left  = dfs(node.left,  deleted, toDelete, forest);   // child is a root if I'm deleted
    node.right = dfs(node.right, deleted, toDelete, forest);
    if (deleted) {
        if (node.left  != null) forest.add(node.left);
        if (node.right != null) forest.add(node.right);
        return null;
    }
    return node;
}
```

(The top-level `root` is added in `delNodes` if it survives; surviving children of deleted
nodes are added inside `dfs`.)

---

## 7. All Nodes Distance K (LC 863)

Two phases: (1) DFS to build a `child → parent` map so the tree becomes an undirected graph;
(2) BFS from `target`, expanding to left/right/parent, until you reach distance `K`.

```java
public List<Integer> distanceK(TreeNode root, TreeNode target, int k) {
    Map<TreeNode, TreeNode> parent = new HashMap<>();
    buildParents(root, null, parent);

    Queue<TreeNode> queue = new LinkedList<>();
    Set<TreeNode> visited = new HashSet<>();
    queue.offer(target);
    visited.add(target);
    int dist = 0;
    while (!queue.isEmpty()) {
        if (dist == k) {                          // current frontier is exactly distance k
            List<Integer> res = new ArrayList<>();
            for (TreeNode node : queue) res.add(node.val);
            return res;
        }
        int size = queue.size();
        for (int i = 0; i < size; i++) {
            TreeNode node = queue.poll();
            for (TreeNode nei : new TreeNode[]{node.left, node.right, parent.get(node)}) {
                if (nei != null && visited.add(nei)) queue.offer(nei);
            }
        }
        dist++;
    }
    return new ArrayList<>();
}

private void buildParents(TreeNode node, TreeNode par, Map<TreeNode, TreeNode> parent) {
    if (node == null) return;
    parent.put(node, par);
    buildParents(node.left,  node, parent);
    buildParents(node.right, node, parent);
}
```

(`parent.put` is the explicit analogue of `Map.computeIfAbsent` for building the adjacency
relation; here each node has exactly one parent so a direct put suffices.)

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
