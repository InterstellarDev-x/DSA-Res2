> **Topic:** [Binary Trees](../README.md) · **Pattern 4 of 5**

# Level Order / BFS

Every level-order problem is the same BFS skeleton with a small per-level twist. Memorize the
template once; then "take the last node" → right side view, "average" → averages, "max" →
largest values, and so on.

```java
public class TreeNode {
    int val;
    TreeNode left, right;
    TreeNode(int val) { this.val = val; }
}
```

---

## The BFS Template (size-snapshot)

```java
Queue<TreeNode> queue = new LinkedList<>();
queue.offer(root);                          // assume root != null (guard separately)
while (!queue.isEmpty()) {
    int size = queue.size();                // SNAPSHOT before the inner loop
    for (int i = 0; i < size; i++) {
        TreeNode node = queue.poll();
        // ... per-node work (often using i == 0 or i == size-1) ...
        if (node.left  != null) queue.offer(node.left);
        if (node.right != null) queue.offer(node.right);
    }
    // ... per-level aggregation ...
}
```

> ⚠️ The single most common BFS bug: reading `queue.size()` *inside* the loop after you've
> already enqueued children. **Snapshot it first.**

---

## 1. Right Side View (LC 199) — last node per level

```java
public List<Integer> rightSideView(TreeNode root) {
    List<Integer> result = new ArrayList<>();
    if (root == null) return result;
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    while (!queue.isEmpty()) {
        int size = queue.size();
        for (int i = 0; i < size; i++) {
            TreeNode node = queue.poll();
            if (i == size - 1) result.add(node.val);   // rightmost node on this level
            if (node.left  != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
    }
    return result;
}
```

---

## 2. Average of Levels (LC 637) — sum / size per level

Use a `double`/`long` accumulator to avoid `int` overflow when summing.

```java
public List<Double> averageOfLevels(TreeNode root) {
    List<Double> result = new ArrayList<>();
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    while (!queue.isEmpty()) {
        int size = queue.size();
        long sum = 0;
        for (int i = 0; i < size; i++) {
            TreeNode node = queue.poll();
            sum += node.val;
            if (node.left  != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        result.add((double) sum / size);
    }
    return result;
}
```

---

## 3. Find Largest Value in Each Row (LC 515) — max per level

```java
public List<Integer> largestValues(TreeNode root) {
    List<Integer> result = new ArrayList<>();
    if (root == null) return result;
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    while (!queue.isEmpty()) {
        int size = queue.size();
        int max = Integer.MIN_VALUE;
        for (int i = 0; i < size; i++) {
            TreeNode node = queue.poll();
            max = Math.max(max, node.val);
            if (node.left  != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        result.add(max);
    }
    return result;
}
```

---

## 4. Level Order Traversal II (LC 107) — bottom-up

Same BFS; prepend each level so the result is bottom-to-top. Use `LinkedList.addFirst` (O(1))
or `Collections.reverse` at the end.

```java
public List<List<Integer>> levelOrderBottom(TreeNode root) {
    LinkedList<List<Integer>> result = new LinkedList<>();
    if (root == null) return result;
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    while (!queue.isEmpty()) {
        int size = queue.size();
        List<Integer> level = new ArrayList<>(size);
        for (int i = 0; i < size; i++) {
            TreeNode node = queue.poll();
            level.add(node.val);
            if (node.left  != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        result.addFirst(level);              // prepend -> bottom-up order, O(1)
    }
    return result;
}
```

---

## 5. N-ary Level Order (LC 429)

Children come from `node.children` instead of `left`/`right` — otherwise identical.

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

## 6. Populating Next Right Pointers II (LC 117) — non-perfect tree

For an arbitrary tree, children may be missing, so the perfect-tree trick (LC 116) breaks. The
**dummy-head** technique builds the next level's `next` chain as we walk the current level: a
sentinel `dummy` precedes the next level, and `tail` extends the chain. This is **O(1) extra
space** (no queue).

```java
class Node { int val; Node left, right, next; }

public Node connect(Node root) {
    Node curr = root;
    while (curr != null) {
        Node dummy = new Node();         // sentinel head of the NEXT level
        Node tail = dummy;
        while (curr != null) {           // walk current level via next pointers
            if (curr.left != null)  { tail.next = curr.left;  tail = tail.next; }
            if (curr.right != null) { tail.next = curr.right; tail = tail.next; }
            curr = curr.next;
        }
        curr = dummy.next;               // descend to the next level's head
    }
    return root;
}
```

The same dummy-head chaining idea handles the perfect-tree case too, so this one solution
covers both LC 116 and LC 117.

---

## Complexity

| Problem | LC | Time | Space |
|---|---|---|---|
| Level Order | 102 | O(n) | O(w) |
| Right Side View | 199 | O(n) | O(w) |
| Average of Levels | 637 | O(n) | O(w) |
| Largest Value Each Row | 515 | O(n) | O(w) |
| Level Order II | 107 | O(n) | O(w) |
| N-ary Level Order | 429 | O(n) | O(w) |
| Populating Next Right II | 117 | O(n) | **O(1)** (dummy-head) |

*w = maximum width of any level (up to n/2 for the bottom level of a full tree).*

---

### Related
- [Tree Traversals](Tree%20Traversals.md) — zigzag & vertical order live there
- [Tree Construction](Tree%20Construction.md) — Maximum Width uses indexed BFS; LC 116

> **Last Updated:** 2026-06-26
