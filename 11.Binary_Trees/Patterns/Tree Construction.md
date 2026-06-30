> **Topic:** [Binary Trees](../README.md) · **Pattern 2 of 5**

# Tree Construction

Rebuilding a tree from traversal arrays, and structural rewrites (flatten, width, next
pointers). The unifying idea: **the root is identifiable from one array, and its position in
another array splits the remaining nodes into left and right subtrees.**

```java
public class TreeNode {
    int val;
    TreeNode left, right;
    TreeNode(int val) { this.val = val; }
}
```

---

## 1. Construct from Preorder & Inorder (LC 105)

- **Preorder** = `[Root | Left subtree | Right subtree]` → the root is `preorder[preStart]`.
- **Inorder**  = `[Left subtree | Root | Right subtree]` → finding the root in inorder splits
  left vs right.

Use a `HashMap<val → inorderIndex>` for **O(1)** root lookup instead of a linear scan.

**Key invariants:**
```
rootVal  = preorder[preStart]
inIdx    = indexInInorder(rootVal)
leftSize = inIdx - inStart            // count of nodes in the left subtree
left  subtree: preorder[preStart+1 .. preStart+leftSize], inorder[inStart .. inIdx-1]
right subtree: preorder[preStart+leftSize+1 .. ],          inorder[inIdx+1 .. inEnd]
```

```java
private Map<Integer, Integer> idx;       // inorder value -> index
private int preIndex;                     // global pointer into preorder

public TreeNode buildTree(int[] preorder, int[] inorder) {
    idx = new HashMap<>();
    for (int i = 0; i < inorder.length; i++) idx.put(inorder[i], i);
    preIndex = 0;
    return build(preorder, 0, inorder.length - 1);
}

private TreeNode build(int[] preorder, int inStart, int inEnd) {
    if (inStart > inEnd) return null;
    int rootVal = preorder[preIndex++];   // preorder consumed front-to-back
    TreeNode root = new TreeNode(rootVal);
    int inIdx = idx.get(rootVal);
    root.left  = build(preorder, inStart, inIdx - 1);   // build LEFT first
    root.right = build(preorder, inIdx + 1, inEnd);     // matches preorder order
    return root;
}
```

> ⚠️ Build the **left** subtree before the right, because the global `preIndex` must advance
> through the left subtree's nodes (which appear first in preorder).

---

## 2. Construct from Inorder & Postorder (LC 106)

**Postorder** = `[Left | Right | Root]` → the root is the **last** element. Reading postorder
**back-to-front** gives Root, then Right subtree, then Left subtree — so build the **right**
subtree before the left.

```java
private Map<Integer, Integer> idx;
private int postIndex;

public TreeNode buildTree(int[] inorder, int[] postorder) {
    idx = new HashMap<>();
    for (int i = 0; i < inorder.length; i++) idx.put(inorder[i], i);
    postIndex = postorder.length - 1;     // consume from the back
    return build(postorder, 0, inorder.length - 1);
}

private TreeNode build(int[] postorder, int inStart, int inEnd) {
    if (inStart > inEnd) return null;
    int rootVal = postorder[postIndex--];
    TreeNode root = new TreeNode(rootVal);
    int inIdx = idx.get(rootVal);
    root.right = build(postorder, inIdx + 1, inEnd);    // RIGHT first (mirror of LC 105)
    root.left  = build(postorder, inStart, inIdx - 1);
    return root;
}
```

---

## 3. Construct from Preorder & Postorder (LC 889)

Without inorder the answer isn't unique, but any valid tree works. Identify the **left
subtree's root** = `preorder[preStart+1]`, find it in postorder to compute the left-subtree
size.

```java
private Map<Integer, Integer> postIdx;
private int preIndex = 0;

public TreeNode constructFromPrePost(int[] pre, int[] post) {
    postIdx = new HashMap<>();
    for (int i = 0; i < post.length; i++) postIdx.put(post[i], i);
    return build(pre, post, 0, post.length - 1);
}

private TreeNode build(int[] pre, int[] post, int postStart, int postEnd) {
    if (preIndex >= pre.length || postStart > postEnd) return null;
    TreeNode root = new TreeNode(pre[preIndex++]);
    if (postStart == postEnd) return root;        // leaf
    int leftRootVal = pre[preIndex];              // next in preorder is left child's root
    int leftRootPos = postIdx.get(leftRootVal);
    root.left  = build(pre, post, postStart, leftRootPos);
    root.right = build(pre, post, leftRootPos + 1, postEnd - 1);
    return root;
}
```

---

## 4. Maximum Width of Binary Tree (LC 662)

Assign each node a **heap-style index**: a node at index `i` has children `2i` and `2i+1`.
Width of a level = `rightmostIndex − leftmostIndex + 1`. **Normalize** indices each level by
subtracting the level's leftmost index to prevent `int` overflow on deep trees.

```java
public int widthOfBinaryTree(TreeNode root) {
    if (root == null) return 0;
    int maxWidth = 0;
    // pair (node, index)
    Queue<int[]> idxQueue = new LinkedList<>();
    Queue<TreeNode> nodeQueue = new LinkedList<>();
    nodeQueue.offer(root);
    idxQueue.offer(new int[]{0});
    while (!nodeQueue.isEmpty()) {
        int size = nodeQueue.size();
        int leftmost = idxQueue.peek()[0];        // normalize against this
        int first = 0, last = 0;
        for (int i = 0; i < size; i++) {
            TreeNode node = nodeQueue.poll();
            int idx = idxQueue.poll()[0] - leftmost;   // <-- overflow guard
            if (i == 0) first = idx;
            if (i == size - 1) last = idx;
            if (node.left != null) {
                nodeQueue.offer(node.left);
                idxQueue.offer(new int[]{2 * idx});
            }
            if (node.right != null) {
                nodeQueue.offer(node.right);
                idxQueue.offer(new int[]{2 * idx + 1});
            }
        }
        maxWidth = Math.max(maxWidth, last - first + 1);
    }
    return maxWidth;
}
```

---

## 5. Flatten Binary Tree to Linked List (LC 114)

Flatten in-place into a "linked list" using the **right** pointers, in preorder.

### Approach A — Morris-like, O(1) space (right-threading)
For each node with a left child, find the left subtree's rightmost node, and splice the
current right subtree onto it; then move the left subtree to the right.

```java
public void flatten(TreeNode root) {
    TreeNode curr = root;
    while (curr != null) {
        if (curr.left != null) {
            TreeNode pred = curr.left;
            while (pred.right != null) pred = pred.right;  // rightmost of left subtree
            pred.right = curr.right;        // splice original right subtree after it
            curr.right = curr.left;         // move left subtree to the right
            curr.left = null;
        }
        curr = curr.right;
    }
}
```

### Approach B — recursive reverse-postorder (Right → Left → Root)
Visit in reverse preorder, chaining each node onto a `prev` pointer.

```java
private TreeNode prev = null;

public void flattenRecursive(TreeNode root) {
    if (root == null) return;
    flattenRecursive(root.right);   // right first
    flattenRecursive(root.left);
    root.right = prev;              // attach what comes after in preorder
    root.left = null;
    prev = root;
}
```

---

## 6. Populating Next Right Pointers (LC 116 — perfect tree)

In a **perfect** binary tree, use the already-established `next` pointers of the parent level
to connect the child level in **constant space** (no queue).

```java
class Node { int val; Node left, right, next; }

public Node connect(Node root) {
    if (root == null) return root;
    Node leftmost = root;
    while (leftmost.left != null) {           // while there is a next level
        Node head = leftmost;
        while (head != null) {
            head.left.next = head.right;                 // within same parent
            if (head.next != null)
                head.right.next = head.next.left;        // across adjacent parents
            head = head.next;
        }
        leftmost = leftmost.left;
    }
    return root;
}
```

For the **non-perfect** variant (LC 117), see [Level Order](Level%20Order.md) — the
**dummy-head** technique handles missing children.

---

## Complexity

| Problem | LC | Time | Space |
|---|---|---|---|
| Construct from Pre+In | 105 | O(n) | O(n) map + O(h) stack |
| Construct from In+Post | 106 | O(n) | O(n) + O(h) |
| Construct from Pre+Post | 889 | O(n) | O(n) + O(h) |
| Maximum Width | 662 | O(n) | O(w) |
| Flatten (Morris) | 114 | O(n) | **O(1)** |
| Flatten (recursive) | 114 | O(n) | O(h) |
| Populating Next Right | 116 | O(n) | **O(1)** |

---

### Related
- [Tree Traversals](Tree%20Traversals.md) — the arrays we rebuild from
- [Level Order](Level%20Order.md) — Next Right Pointers II, the dummy-head trick

> **Last Updated:** 2026-06-26
