> **Topic:** [Binary Trees](../README.md) · **Pattern N of M** · **Company 2 of 3**

# Google — Binary Tree Interview Problems

Google binary-tree questions tend to stress **deterministic ordering, coordinate reasoning, and graph-style traversals on tree structures**. The interviewer cares less about whether you can recall an algorithm and more about whether your code is clean, your tie-breaking rules are exact, and you proactively handle edge cases. The three problems below are classic Google rounds.

Standard node definition used throughout:

```java
public class TreeNode {
    int val;
    TreeNode left, right;
    TreeNode(int val) { this.val = val; }
}
```

---

## 1. Vertical Order Traversal of a Binary Tree (LC 987)

### Problem Statement
Assign the root the coordinate `(row=0, col=0)`. A left child is at `(row+1, col-1)` and a right child at `(row+1, col+1)`. The **vertical order traversal** lists nodes grouped by column from leftmost to rightmost. Within the same column, nodes are ordered by row (top to bottom); and crucially, **if two nodes share the same row and column, they are reported in ascending order of value**.

That last tie-break is what makes LC 987 harder than the older LC 314.

### Intuition — The Full Ordering Rule
We need a strict total order over `(col, row, value)`:

1. **Primary:** smaller `col` first (left to right).
2. **Secondary:** smaller `row` first (top to bottom).
3. **Tertiary:** smaller `value` first (the LC 987 tie-break).

### Why TreeMap, not HashMap
The output must be emitted in **ascending column order**, and within a column in ascending row order. A `HashMap` gives no ordering guarantee, so we'd have to collect keys and sort afterward. A **`TreeMap` keeps keys sorted automatically**, so iterating it yields columns left-to-right (and rows top-to-bottom) for free, with deterministic behavior. This determinism is exactly what Google interviewers probe — "what happens to ties?" — and choosing `TreeMap` lets you answer "the structure enforces it."

We model the data as:

```
TreeMap<Integer, TreeMap<Integer, PriorityQueue<Integer>>>
   col  ->        row  ->        min-heap of values
```

- Outer `TreeMap`: columns sorted ascending.
- Inner `TreeMap`: rows sorted ascending within a column.
- `PriorityQueue<Integer>` (min-heap): values sorted ascending within the same `(col,row)` cell, satisfying the tertiary tie-break.

### Full Java Solution

```java
class Solution {
    public List<List<Integer>> verticalTraversal(TreeNode root) {
        // col -> (row -> min-heap of values)
        TreeMap<Integer, TreeMap<Integer, PriorityQueue<Integer>>> grid =
                new TreeMap<>();

        dfs(root, 0, 0, grid);

        List<List<Integer>> result = new ArrayList<>();
        // Iterating a TreeMap walks columns in ascending order.
        for (TreeMap<Integer, PriorityQueue<Integer>> rows : grid.values()) {
            List<Integer> column = new ArrayList<>();
            // Inner TreeMap walks rows in ascending order.
            for (PriorityQueue<Integer> cell : rows.values()) {
                // Drain the min-heap: smallest values first.
                while (!cell.isEmpty()) {
                    column.add(cell.poll());
                }
            }
            result.add(column);
        }
        return result;
    }

    private void dfs(TreeNode node, int row, int col,
                     TreeMap<Integer, TreeMap<Integer, PriorityQueue<Integer>>> grid) {
        if (node == null) return;

        grid.computeIfAbsent(col, c -> new TreeMap<>())
            .computeIfAbsent(row, r -> new PriorityQueue<>())
            .add(node.val);

        dfs(node.left,  row + 1, col - 1, grid);
        dfs(node.right, row + 1, col + 1, grid);
    }
}

// Required imports: java.util.ArrayList, java.util.List, java.util.PriorityQueue,
// java.util.TreeMap
```

### Complexity

| Metric | Value | Reason |
| --- | --- | --- |
| Time | O(n log n) | Each insert into the nested structure / heap costs O(log n); n inserts dominate. |
| Space | O(n) | The grid stores every node once, plus O(h) recursion stack. |

### Note
A DFS or BFS both work for *visiting* nodes; the ordering is enforced entirely by the data structure, not the traversal order. That decoupling — "traverse however; let `TreeMap`/`PriorityQueue` impose the rule" — is the clean insight to state aloud.

---

## 2. Lowest Common Ancestor of Deepest Leaves (LC 1123)

### Problem Statement
Return the LCA of the **deepest leaves** of a binary tree. If there is a single deepest leaf, it is its own LCA. Equivalent to LC 865 ("Smallest Subtree with all the Deepest Nodes").

### Intuition — The Two-Return Approach
A single post-order pass can return, for each subtree, a pair: **(the LCA-of-deepest-leaves within this subtree, the depth of those deepest leaves)**. Combine children:

- Compute `left = solve(node.left)` and `right = solve(node.right)`.
- If `left.depth > right.depth`: the deepest leaves all live in the left subtree, so propagate the left answer with depth `left.depth + 1`.
- If `right.depth > left.depth`: symmetric — propagate the right answer.
- If **`left.depth == right.depth`**: the deepest leaves straddle both subtrees, so the **current node is their LCA**; propagate `node` with depth `left.depth + 1`.

Base case: a null node returns `(null, 0)`. A leaf therefore returns `(leaf, 1)` because both children give depth 0 (equal), making the leaf its own LCA.

The "equal depths → current node" rule is the heart of the problem: equal-depth deepest leaves on both sides mean the divergence point is here.

### Full Java Solution

```java
class Solution {
    // Small pair class carrying the running answer and its depth.
    private static class Result {
        TreeNode node; // LCA of deepest leaves within this subtree
        int depth;     // depth of those deepest leaves (relative to this subtree)
        Result(TreeNode node, int depth) {
            this.node = node;
            this.depth = depth;
        }
    }

    public TreeNode lcaDeepestLeaves(TreeNode root) {
        return solve(root).node;
    }

    private Result solve(TreeNode node) {
        if (node == null) return new Result(null, 0);

        Result left = solve(node.left);
        Result right = solve(node.right);

        if (left.depth > right.depth) {
            return new Result(left.node, left.depth + 1);
        }
        if (right.depth > left.depth) {
            return new Result(right.node, right.depth + 1);
        }
        // Equal depths: deepest leaves are on both sides -> this node is the LCA.
        return new Result(node, left.depth + 1);
    }
}
```

### Complexity

| Metric | Value | Reason |
| --- | --- | --- |
| Time | O(n) | Single post-order traversal. |
| Space | O(h) | Recursion stack; each `Result` is O(1). |

### Note
Avoid the naive "first compute max depth, then run LCA over all deepest leaves" — it works but is two passes and clumsier. The single-pass two-return formulation is the answer Google expects.

---

## 3. All Nodes Distance K in Binary Tree (LC 863)

### Problem Statement
Given the root, a `target` node, and an integer `k`, return the values of all nodes that are **exactly distance `k`** from `target`. Distance counts edges, moving up to parents or down to children freely.

### Intuition — Treat the Tree as an Undirected Graph
The difficulty is that distance can go **upward** (toward parents), but tree nodes only point downward. Two-phase fix:

1. **Build a parent map** with a DFS so every node knows its parent. Now each node effectively has up to three neighbors: left child, right child, and parent.
2. **BFS from `target`**, expanding to all three neighbors, tracking visited nodes so we never bounce back. After exactly `k` BFS levels, every node still in the queue is at distance `k`.

This converts a tree into an undirected graph and reduces "distance k" to a plain breadth-first layer.

### Full Java Solution

```java
class Solution {
    public List<Integer> distanceK(TreeNode root, TreeNode target, int k) {
        // Phase 1: map each node to its parent.
        Map<TreeNode, TreeNode> parent = new HashMap<>();
        buildParents(root, null, parent);

        // Phase 2: BFS outward from target through up-to-three neighbors.
        List<Integer> result = new ArrayList<>();
        Queue<TreeNode> queue = new LinkedList<>();
        Set<TreeNode> visited = new HashSet<>();

        queue.offer(target);
        visited.add(target);
        int dist = 0;

        while (!queue.isEmpty()) {
            int size = queue.size();
            if (dist == k) {
                // Everything currently in the queue is exactly k edges away.
                for (TreeNode node : queue) {
                    result.add(node.val);
                }
                return result;
            }
            for (int i = 0; i < size; i++) {
                TreeNode node = queue.poll();
                // Three potential neighbors: left, right, parent.
                addNeighbor(node.left, visited, queue);
                addNeighbor(node.right, visited, queue);
                addNeighbor(parent.get(node), visited, queue);
            }
            dist++;
        }
        return result; // k is deeper than any reachable node.
    }

    private void addNeighbor(TreeNode neighbor, Set<TreeNode> visited,
                             Queue<TreeNode> queue) {
        if (neighbor != null && visited.add(neighbor)) {
            queue.offer(neighbor);
        }
    }

    private void buildParents(TreeNode node, TreeNode par,
                              Map<TreeNode, TreeNode> parent) {
        if (node == null) return;
        parent.computeIfAbsent(node, n -> par);
        buildParents(node.left, node, parent);
        buildParents(node.right, node, parent);
    }
}

// Required imports: java.util.*  (ArrayList, HashMap, HashSet, LinkedList,
// List, Map, Queue, Set)
```

### Complexity

| Metric | Value | Reason |
| --- | --- | --- |
| Time | O(n) | One DFS to build parents + one BFS, each visiting nodes once. |
| Space | O(n) | Parent map, visited set, and queue each hold up to n nodes. |

### Note
`visited.add(neighbor)` returns `false` if the node was already present, giving a clean one-line "add only if unseen." Without `visited`, BFS would oscillate between a node and its parent forever.

---

## Google Interview Level Context (L4 / L5)

Google calibrates these problems differently by level — knowing the expectation helps you allocate time:

- **L4 (SWE II):** Expected to reach a correct, clean solution with good naming and structure. You should **enumerate edge cases unprompted** — empty tree, single node, `target` is the root, `k = 0`, `k` larger than the tree's reach. Stating time/space complexity precisely (and *why*) is mandatory, not optional.
- **L5 (Senior SWE):** Beyond correctness, demonstrate **design judgment**: choosing `TreeMap` for determinism (LC 987), recognizing the single-pass two-return formulation (LC 1123), and reframing a tree as a graph (LC 863). Expect **follow-ups** — "what if the tree is enormous and doesn't fit in memory?", "can you do LC 863 without a parent map?" (yes: a single DFS that returns the subtree distance to `target` and collects downward nodes as it unwinds), "how would you stream LC 987 output?".

General Google bar across levels:
- Talk through the **contract/invariant** before coding; don't dive into syntax first.
- Write **compilable** Java with full method signatures and imports — Googlers dislike pseudocode.
- **Test your own code** on a small example by hand and walk the interviewer through it.
- Discuss **complexity trade-offs** and at least one **follow-up extension** proactively.

---

> **Last Updated:** 2026-06-26
