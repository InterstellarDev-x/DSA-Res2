> **Topic:** [Binary Trees](../README.md) · **Pattern N of M** · **Company 3 of 3**

# Microsoft — Binary Tree Interview Problems

Microsoft binary-tree interviews lean on **reconstruction, careful index/state bookkeeping, and greedy tree DP**. The interviewer typically wants to see you reason out invariants on paper, write clean compilable code, and then *test it yourself* on a small input. The three problems below are recurring Microsoft on-sites.

Standard node definition used throughout:

```java
public class TreeNode {
    int val;
    TreeNode left, right;
    TreeNode(int val) { this.val = val; }
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

So for each recursive call over `preorder[preStart..preEnd]` and `inorder[inStart..inEnd]`:

- `rootVal = preorder[preStart]` — the root of this subtree.
- `inIdx` = index of `rootVal` inside `inorder` (looked up in O(1) via a `HashMap<value, index>`).
- `leftSize = inIdx - inStart` — number of nodes in the left subtree.
- **Left subtree** uses preorder range `preStart+1 .. preStart+leftSize` and inorder range `inStart .. inIdx-1`.
- **Right subtree** uses preorder range `preStart+leftSize+1 .. preEnd` and inorder range `inIdx+1 .. inEnd`.

The trickiest line is the preorder split: after the root at `preStart`, the **next `leftSize` elements** are the entire left subtree's preorder, so the right subtree's preorder begins at `preStart + leftSize + 1`.

### Worked Trace
`preorder = [3, 9, 20, 15, 7]`, `inorder = [9, 3, 15, 20, 7]`.

1. `preStart=0` → `rootVal = 3`. In inorder, `inIdx = 1`. `leftSize = 1 - 0 = 1`.
   - **Left** child: `preorder[1..1] = [9]`, `inorder[0..0] = [9]`.
   - **Right** child: `preorder[2..4] = [20,15,7]`, `inorder[2..4] = [15,20,7]`.
2. Left call `rootVal = 9`, `inIdx = 0`, `leftSize = 0` → leaf `9`. Done.
3. Right call `rootVal = 20`, in inorder `[15,20,7]` → `inIdx = 3`, `inStart = 2`, `leftSize = 3 - 2 = 1`.
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

### Full Java Solution

```java
class Solution {
    private Map<Integer, Integer> inorderIndex; // value -> index in inorder
    private int[] preorder;

    public TreeNode buildTree(int[] preorder, int[] inorder) {
        this.preorder = preorder;
        this.inorderIndex = new HashMap<>();
        for (int i = 0; i < inorder.length; i++) {
            inorderIndex.put(inorder[i], i);
        }
        return build(0, 0, inorder.length - 1);
    }

    // preStart: start of this subtree in preorder
    // inStart, inEnd: bounds of this subtree in inorder
    private TreeNode build(int preStart, int inStart, int inEnd) {
        if (inStart > inEnd) return null; // empty segment

        int rootVal = preorder[preStart];      // root is first in preorder
        TreeNode root = new TreeNode(rootVal);

        int inIdx = inorderIndex.get(rootVal); // O(1) lookup
        int leftSize = inIdx - inStart;        // nodes in left subtree

        // Left subtree:  preorder[preStart+1 .. preStart+leftSize],
        //                inorder[inStart .. inIdx-1]
        root.left = build(preStart + 1, inStart, inIdx - 1);

        // Right subtree: preorder[preStart+leftSize+1 .. ],
        //                inorder[inIdx+1 .. inEnd]
        root.right = build(preStart + leftSize + 1, inIdx + 1, inEnd);

        return root;
    }
}

// Required imports: java.util.HashMap, java.util.Map
```

### Complexity

| Metric | Value | Reason |
| --- | --- | --- |
| Time | O(n) | Each node built once; HashMap makes root lookup O(1). |
| Space | O(n) | HashMap of size n + O(h) recursion stack. |

### Note
Without the `HashMap`, finding `inIdx` by linear scan turns this into O(n²) on a skewed tree — a common follow-up is "how do you avoid the linear search?"

---

## 2. Delete Nodes And Return Forest (LC 1110)

### Problem Statement
Given the root of a tree and a list `to_delete` of values to remove, delete those nodes. The result is a **forest** (a list of disjoint trees). Return the roots of the trees in the remaining forest, in any order.

### Intuition — Orphan Detection in Post-order
Deleting a node can **create new roots**: its surviving children lose their parent and become roots of their own trees. So we need two pieces of information at each node during a **post-order** traversal:

1. Is this node **itself** going to be deleted? (membership in a `Set<Integer>` of the to-delete values).
2. Is this node currently a **root** of the forest — i.e., is it the original root, or did its parent just get deleted?

The rule: **a surviving node becomes a new root exactly when it is a root candidate** (`isRoot == true`) — that happens when it is the original tree root, or when its parent was deleted. We pass an `isRoot` flag downward; a child is a root candidate iff the current node is being deleted.

Process post-order so children are resolved first:
- Recurse into left and right, passing `isRoot = (this node is deleted)`.
- The recursive call returns the (possibly null) surviving child to re-link.
- If the current node is deleted, return `null` to its parent (severing the link); its surviving children were already collected as new roots.
- If the current node survives **and** is a root candidate, add it to the result list.

### Full Java Solution

```java
class Solution {
    public List<TreeNode> delNodes(TreeNode root, int[] to_delete) {
        Set<Integer> toDelete = new HashSet<>();
        for (int v : to_delete) toDelete.add(v);

        List<TreeNode> forest = new ArrayList<>();
        // The original root is a root candidate.
        dfs(root, true, toDelete, forest);
        return forest;
    }

    // Returns the surviving node (or null if this node is deleted),
    // for the parent to re-link.
    private TreeNode dfs(TreeNode node, boolean isRoot,
                         Set<Integer> toDelete, List<TreeNode> forest) {
        if (node == null) return null;

        boolean deleted = toDelete.contains(node.val);

        // A surviving node that is a root candidate joins the forest.
        if (!deleted && isRoot) {
            forest.add(node);
        }

        // Children become root candidates iff THIS node is deleted.
        node.left  = dfs(node.left,  deleted, toDelete, forest);
        node.right = dfs(node.right, deleted, toDelete, forest);

        // If deleted, sever from parent by returning null;
        // otherwise return self for re-linking.
        return deleted ? null : node;
    }
}

// Required imports: java.util.ArrayList, java.util.HashSet, java.util.List,
// java.util.Set
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

### Full Java Solution

```java
class Solution {
    private static final int NOT_COVERED = 0;
    private static final int HAS_CAMERA  = 1;
    private static final int COVERED     = 2;

    private int cameras = 0; // instance var as global count

    public int minCameraCover(TreeNode root) {
        // If the root ends up uncovered, it has no parent to cover it.
        if (dfs(root) == NOT_COVERED) {
            cameras++;
        }
        return cameras;
    }

    private int dfs(TreeNode node) {
        // A null child imposes no demand -> treated as covered.
        if (node == null) return COVERED;

        int left  = dfs(node.left);
        int right = dfs(node.right);

        // A child needs coverage -> we must place a camera here.
        if (left == NOT_COVERED || right == NOT_COVERED) {
            cameras++;
            return HAS_CAMERA;
        }

        // A child has a camera -> this node is monitored.
        if (left == HAS_CAMERA || right == HAS_CAMERA) {
            return COVERED;
        }

        // Both children covered but none has a camera here:
        // defer the camera to the parent.
        return NOT_COVERED;
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
