> **Topic:** [Binary Trees](../README.md) · **Pattern 1 of 4**

# Binary Trees — Coding Tips

A focused set of practical coding tips that make binary-tree problems faster to write and harder to get wrong in an interview setting. Each tip pairs a short explanation with a Java snippet you can reproduce from memory.

Throughout these tips we assume the standard node definition:

```java
public class TreeNode {
    int val;
    TreeNode left, right;
    TreeNode(int val) { this.val = val; }
}
```

---

## 1. TreeNode null-check discipline

The single most reliable habit in tree recursion: **the first line of every recursive method handles the `null` node**. This terminates recursion at the "edges" of the tree (below the leaves) and removes the need to null-check children individually before recursing.

If you make this the very first statement every time, you will almost never hit a `NullPointerException`, and your base case is always explicit.

```java
int maxDepth(TreeNode node) {
    if (node == null) return 0;            // first line, always
    return 1 + Math.max(maxDepth(node.left), maxDepth(node.right));
}
```

The return value of the base case is problem-specific (`0` for depth/height, `null` for "no node found", `true` for vacuously-true predicates, etc.), but the *shape* is invariant.

---

## 2. Leaf check pattern

A leaf is a node with **no children**. Many problems (Path Sum, Min Depth, Sum Root to Leaf, Binary Tree Paths) treat leaves specially, so commit the exact condition to muscle memory:

```java
if (node.left == null && node.right == null) {
    // this is a leaf — terminate the path / record the answer here
}
```

Be careful to distinguish this from the `null`-node base case. A common bug in **Min Depth** is treating a node with one child as a leaf — it is not. You only short-circuit on a *true* leaf, and otherwise take the depth of the non-null subtree.

```java
int minDepth(TreeNode node) {
    if (node == null) return 0;
    if (node.left == null) return 1 + minDepth(node.right);  // not a leaf
    if (node.right == null) return 1 + minDepth(node.left);  // not a leaf
    return 1 + Math.min(minDepth(node.left), minDepth(node.right));
}
```

---

## 3. Diameter doesn't pass through the root — track a global max

A frequent misconception is that the diameter (longest path between any two nodes) must run through the root. It does not. The longest path can sit entirely inside a subtree.

The trick: write a helper that **returns the height** of a subtree, but as a side effect **updates a global** with `leftHeight + rightHeight` at every node. The diameter is the maximum of those local sums across the whole tree.

```java
int[] best = new int[1];               // global state, length-1 array

int height(TreeNode node) {
    if (node == null) return 0;
    int left = height(node.left);
    int right = height(node.right);
    best[0] = Math.max(best[0], left + right);   // update at EVERY node
    return 1 + Math.max(left, right);            // return value is the height
}
```

This "return one thing, accumulate another globally" pattern generalizes to Max Path Sum (Tip 7), House Robber III, and more.

---

## 4. BFS size snapshot

Level-order traversal (BFS) needs to know **where one level ends and the next begins**. The clean way is to capture the queue size *before* the inner loop, because the queue grows as you enqueue children.

```java
Queue<TreeNode> queue = new LinkedList<>();
queue.offer(root);
while (!queue.isEmpty()) {
    int size = queue.size();           // SNAPSHOT before the inner loop
    List<Integer> level = new ArrayList<>();
    for (int i = 0; i < size; i++) {   // iterate exactly this level
        TreeNode node = queue.poll();
        level.add(node.val);
        if (node.left != null) queue.offer(node.left);
        if (node.right != null) queue.offer(node.right);
    }
    // `level` now holds one full level
}
```

If you write `for (int i = 0; i < queue.size(); i++)` instead, the loop bound shifts as children are added and levels bleed into each other.

---

## 5. Morris traversal — O(1) space inorder via threading

Recursive and stack-based inorder both use O(h) extra space. **Morris traversal** achieves O(1) space (ignoring output) by temporarily *threading* each node to its inorder successor.

Core idea: for the current node, find its inorder predecessor (rightmost node of its left subtree). Create a temporary right-pointer ("thread") from that predecessor back to the current node so you can climb back up without a stack. On the second visit, remove the thread and emit the value.

```java
List<Integer> morrisInorder(TreeNode root) {
    List<Integer> out = new ArrayList<>();
    TreeNode curr = root;
    while (curr != null) {
        if (curr.left == null) {
            out.add(curr.val);
            curr = curr.right;
        } else {
            TreeNode pred = curr.left;
            while (pred.right != null && pred.right != curr) pred = pred.right;
            if (pred.right == null) {
                pred.right = curr;       // create thread
                curr = curr.left;
            } else {
                pred.right = null;       // remove thread, restore tree
                out.add(curr.val);
                curr = curr.right;
            }
        }
    }
    return out;
}
```

Mention this when an interviewer asks for **O(1) extra space** traversal — it shows you know the trick beyond the textbook stack approach.

---

## 6. Index normalization for Max Width

To compute the maximum width of a tree, assign each node a **heap-style index**: a node at index `i` has children at `2*i` and `2*i + 1`. The width of a level is `rightmostIdx - leftmostIdx + 1`.

The danger: on a deep, sparse tree these indices grow exponentially and **overflow `int`/`long`**. Normalize every level by subtracting the leftmost index on that level, which keeps indices small without changing the *differences* (which is all width depends on).

```java
int widthOfBinaryTree(TreeNode root) {
    if (root == null) return 0;
    int max = 0;
    Queue<TreeNode> nodes = new LinkedList<>();
    Queue<Integer> idxs = new LinkedList<>();
    nodes.offer(root);
    idxs.offer(0);
    while (!nodes.isEmpty()) {
        int size = nodes.size();
        int leftmost = idxs.peek();          // base for this level
        int first = 0, last = 0;
        for (int i = 0; i < size; i++) {
            TreeNode node = nodes.poll();
            int idx = idxs.poll() - leftmost; // NORMALIZE
            if (i == 0) first = idx;
            if (i == size - 1) last = idx;
            if (node.left != null)  { nodes.offer(node.left);  idxs.offer(2 * idx); }
            if (node.right != null) { nodes.offer(node.right); idxs.offer(2 * idx + 1); }
        }
        max = Math.max(max, last - first + 1);
    }
    return max;
}
```

---

## 7. `gain = Math.max(0, gain)` for Max Path Sum

In Maximum Path Sum, each node reports to its parent the best **single downward path gain** it can contribute. If a subtree's best contribution is negative, it is better to contribute **nothing** (i.e. cut that branch off) than to drag the path down.

Clamp each child's gain at zero before using it:

```java
int best = Integer.MIN_VALUE;            // global (use an instance var or int[1])

int gain(TreeNode node) {
    if (node == null) return 0;
    int left  = Math.max(0, gain(node.left));   // prune negatives
    int right = Math.max(0, gain(node.right));
    best = Math.max(best, node.val + left + right);  // path "through" this node
    return node.val + Math.max(left, right);         // gain handed to parent
}
```

The path that bends through a node uses *both* children (`left + right`), but the gain handed upward can only follow *one* branch (`Math.max(left, right)`), because a path cannot fork.

---

## 8. Parent-map for distance problems

Trees only have child pointers, so "move toward the parent" is impossible by default. For problems like **All Nodes Distance K** or **LCA Deepest Leaves**, first build a `child -> parent` map, then you can BFS outward in all three directions (left child, right child, parent).

```java
Map<TreeNode, TreeNode> parent = new HashMap<>();

void buildParents(TreeNode node, TreeNode par) {
    if (node == null) return;
    parent.put(node, par);
    buildParents(node.left, node);
    buildParents(node.right, node);
}
```

Once built, a standard BFS from the target node — enqueuing `node.left`, `node.right`, and `parent.get(node)` while tracking visited nodes — finds everything at distance K.

---

## 9. Backtrack discipline for path collection

When collecting root-to-leaf paths (Binary Tree Paths, Path Sum II), you reuse a single mutable `path` list and **undo** your change after recursing. The pattern is: add, recurse into both children, then remove the last element.

```java
void dfs(TreeNode node, List<Integer> path, List<List<Integer>> result) {
    if (node == null) return;
    path.add(node.val);                              // choose
    if (node.left == null && node.right == null) {
        result.add(new ArrayList<>(path));           // record a COPY at a leaf
    } else {
        dfs(node.left, path, result);                // explore
        dfs(node.right, path, result);
    }
    path.remove(path.size() - 1);                    // un-choose (backtrack)
}
```

Two non-negotiables: (1) add a **copy** (`new ArrayList<>(path)`) to the result, never the live list; (2) the `path.remove(path.size() - 1)` runs on **every** return, including after recording a leaf, so it must sit outside the leaf branch as shown.

---

> **Last Updated:** 2026-06-26
