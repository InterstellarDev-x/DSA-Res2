> **Topic:** [Binary Trees](../README.md) · **Pattern 1 of 5**

# Tree Traversals

Traversal is the foundation of every binary-tree algorithm. Master the four core orders
(preorder, inorder, postorder, level-order) both recursively and iteratively, plus the two
"hard" traversals interviewers love: **zigzag** and **vertical order**.

```java
public class TreeNode {
    int val;
    TreeNode left, right;
    TreeNode(int val) { this.val = val; }
}
```

---

## 1. Recursive DFS (the clean 3-liners)

The recursive shape is identical for all three orders — only the *position* of the visit
statement changes.

```java
// Preorder: Root -> Left -> Right
void preorder(TreeNode node, List<Integer> out) {
    if (node == null) return;
    out.add(node.val);              // visit
    preorder(node.left, out);
    preorder(node.right, out);
}

// Inorder: Left -> Root -> Right
void inorder(TreeNode node, List<Integer> out) {
    if (node == null) return;
    inorder(node.left, out);
    out.add(node.val);              // visit
    inorder(node.right, out);
}

// Postorder: Left -> Right -> Root
void postorder(TreeNode node, List<Integer> out) {
    if (node == null) return;
    postorder(node.left, out);
    postorder(node.right, out);
    out.add(node.val);              // visit
}
```

**Mnemonic:** the prefix (pre/in/post) tells you *where the Root is visited* relative to the
two recursive descents.

---

## 2. Iterative Inorder — push-left-first (LC 94)

Walk all the way left pushing onto a stack, then pop (visit), then move to the right child
and repeat.

```java
public List<Integer> inorderTraversal(TreeNode root) {
    List<Integer> out = new ArrayList<>();
    Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode curr = root;
    while (curr != null || !stack.isEmpty()) {
        while (curr != null) {      // push the entire left spine
            stack.push(curr);
            curr = curr.left;
        }
        curr = stack.pop();         // leftmost unvisited node
        out.add(curr.val);          // visit
        curr = curr.right;          // then explore its right subtree
    }
    return out;
}
```

---

## 3. Iterative Preorder — push right, then left (LC 144)

Because a stack is LIFO, push the **right** child first so the **left** child is processed
next (preorder = Root, Left, Right).

```java
public List<Integer> preorderTraversal(TreeNode root) {
    List<Integer> out = new ArrayList<>();
    if (root == null) return out;
    Deque<TreeNode> stack = new ArrayDeque<>();
    stack.push(root);
    while (!stack.isEmpty()) {
        TreeNode node = stack.pop();
        out.add(node.val);                       // visit root first
        if (node.right != null) stack.push(node.right); // right pushed first
        if (node.left  != null) stack.push(node.left);  // left popped first
    }
    return out;
}
```

---

## 4. Iterative Postorder (LC 145)

### Approach A — two stacks
Do a *modified preorder* (Root, Right, Left) onto `stack2`, then reverse it by popping.

```java
public List<Integer> postorderTraversal(TreeNode root) {
    LinkedList<Integer> out = new LinkedList<>();
    if (root == null) return out;
    Deque<TreeNode> stack = new ArrayDeque<>();
    stack.push(root);
    while (!stack.isEmpty()) {
        TreeNode node = stack.pop();
        out.addFirst(node.val);                  // prepend => reverses Root,Right,Left
        if (node.left  != null) stack.push(node.left);
        if (node.right != null) stack.push(node.right);
    }
    return out; // ends up Left, Right, Root
}
```

This is the **reverse-of-modified-preorder trick**: preorder visiting Root→Right→Left,
collected with `addFirst`, yields Left→Right→Root.

### Approach B — single stack with a `lastVisited` pointer
Visit a node only after its right child has been fully processed.

```java
public List<Integer> postorderSingleStack(TreeNode root) {
    List<Integer> out = new ArrayList<>();
    Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode curr = root, lastVisited = null;
    while (curr != null || !stack.isEmpty()) {
        while (curr != null) { stack.push(curr); curr = curr.left; }
        TreeNode peek = stack.peek();
        if (peek.right != null && lastVisited != peek.right) {
            curr = peek.right;                   // go right before visiting
        } else {
            out.add(peek.val);
            lastVisited = stack.pop();
        }
    }
    return out;
}
```

---

## 5. Morris Traversal — O(1) space inorder

Threads each node's predecessor's right pointer to itself, removing the need for a stack.
Time O(n), space **O(1)** (mutates then restores pointers).

```java
public List<Integer> morrisInorder(TreeNode root) {
    List<Integer> out = new ArrayList<>();
    TreeNode curr = root;
    while (curr != null) {
        if (curr.left == null) {
            out.add(curr.val);
            curr = curr.right;
        } else {
            TreeNode pred = curr.left;
            while (pred.right != null && pred.right != curr) pred = pred.right;
            if (pred.right == null) {            // create thread
                pred.right = curr;
                curr = curr.left;
            } else {                             // thread exists -> remove it, visit
                pred.right = null;
                out.add(curr.val);
                curr = curr.right;
            }
        }
    }
    return out;
}
```

---

## 6. Level Order — BFS with size snapshot (LC 102)

The **size-snapshot** is the key idiom: capture `queue.size()` *before* the inner loop so
you process exactly one level at a time.

```java
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();
    if (root == null) return result;
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    while (!queue.isEmpty()) {
        int size = queue.size();                 // snapshot: nodes on this level
        List<Integer> level = new ArrayList<>(size);
        for (int i = 0; i < size; i++) {
            TreeNode node = queue.poll();
            level.add(node.val);
            if (node.left  != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        result.add(level);
    }
    return result;
}
```

---

## 7. Zigzag Level Order (LC 103)

Run normal BFS but alternate the **insertion direction** within each level using a
`LinkedList` + `addFirst`/`addLast`. This keeps each insert O(1) (never `add(0, x)`).

```java
public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();
    if (root == null) return result;
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    boolean leftToRight = true;
    while (!queue.isEmpty()) {
        int size = queue.size();
        LinkedList<Integer> level = new LinkedList<>();
        for (int i = 0; i < size; i++) {
            TreeNode node = queue.poll();
            if (leftToRight) level.addLast(node.val);
            else             level.addFirst(node.val);
            if (node.left  != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        result.add(level);
        leftToRight = !leftToRight;
    }
    return result;
}
```

---

## 8. N-ary Level Order (LC 429)

Identical BFS, but children come from a list (`node.children`).

```java
class Node { public int val; public List<Node> children; }

public List<List<Integer>> levelOrder(Node root) {
    List<List<Integer>> result = new ArrayList<>();
    if (root == null) return result;
    Queue<Node> queue = new LinkedList<>();
    queue.offer(root);
    while (!queue.isEmpty()) {
        int size = queue.size();
        List<Integer> level = new ArrayList<>(size);
        for (int i = 0; i < size; i++) {
            Node node = queue.poll();
            level.add(node.val);
            for (Node child : node.children) queue.offer(child);
        }
        result.add(level);
    }
    return result;
}
```

---

## 9. Vertical Order Traversal (LC 987)

Group nodes by **column** (root = 0, left = col−1, right = col+1). Within a column, order by
**row** (depth), and ties at the same (row, col) by **value**. A
`TreeMap<col, TreeMap<row, PriorityQueue<val>>>` makes ordering deterministic without a final
sort — `TreeMap` is used (not `HashMap`) precisely because we need keys iterated in sorted
order.

```java
public List<List<Integer>> verticalTraversal(TreeNode root) {
    // col -> (row -> min-heap of values)
    TreeMap<Integer, TreeMap<Integer, PriorityQueue<Integer>>> map = new TreeMap<>();
    dfs(root, 0, 0, map);

    List<List<Integer>> result = new ArrayList<>();
    for (TreeMap<Integer, PriorityQueue<Integer>> rows : map.values()) {
        List<Integer> col = new ArrayList<>();
        for (PriorityQueue<Integer> pq : rows.values())
            while (!pq.isEmpty()) col.add(pq.poll());
        result.add(col);
    }
    return result;
}

private void dfs(TreeNode node, int row, int col,
                 TreeMap<Integer, TreeMap<Integer, PriorityQueue<Integer>>> map) {
    if (node == null) return;
    map.computeIfAbsent(col, k -> new TreeMap<>())
       .computeIfAbsent(row, k -> new PriorityQueue<>())
       .offer(node.val);
    dfs(node.left,  row + 1, col - 1, map);
    dfs(node.right, row + 1, col + 1, map);
}
```

---

## Complexity

| Traversal | Approach | Time | Space |
|---|---|---|---|
| Inorder / Preorder / Postorder (recursive) | DFS | O(n) | O(h) stack |
| Inorder (iterative) | explicit stack | O(n) | O(h) |
| Preorder (iterative) | explicit stack | O(n) | O(h) |
| Postorder (iterative) | two-stack / lastVisited | O(n) | O(h) |
| Inorder (Morris) | threading | O(n) | **O(1)** |
| Level Order (LC 102) | BFS | O(n) | O(w) queue |
| Zigzag (LC 103) | BFS | O(n) | O(w) |
| N-ary Level Order (LC 429) | BFS | O(n) | O(w) |
| Vertical Order (LC 987) | DFS + TreeMap | O(n log n) | O(n) |

*h = tree height (O(log n) balanced, O(n) skewed); w = max width of any level.*

---

### Related
- [Level Order](Level%20Order.md) — the full BFS catalog
- [Tree Construction](Tree%20Construction.md) — rebuilding trees from traversal arrays

> **Last Updated:** 2026-06-26
