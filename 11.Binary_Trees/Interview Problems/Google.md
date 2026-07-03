> **Topic:** [Binary Trees](../README.md) · **Pattern N of M** · **Company 2 of 3**

# Google — Binary Tree Interview Problems

Google binary-tree questions tend to stress **deterministic ordering, coordinate reasoning, and graph-style traversals on tree structures**. The interviewer cares less about whether you can recall an algorithm and more about whether your code is clean, your tie-breaking rules are exact, and you proactively handle edge cases. The three problems below are classic Google rounds.

Standard node definition used throughout:

```cpp
#include <bits/stdc++.h>
using namespace std;

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int val) : val(val), left(nullptr), right(nullptr) {}
};
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

### Why std::map, not std::unordered_map
The output must be emitted in **ascending column order**, and within a column in ascending row order. A `std::unordered_map` gives no ordering guarantee, so we'd have to collect keys and sort afterward. A **`std::map` keeps keys sorted automatically**, so iterating it yields columns left-to-right (and rows top-to-bottom) for free, with deterministic behavior. This determinism is exactly what Google interviewers probe — "what happens to ties?" — and choosing `std::map` lets you answer "the structure enforces it."

We model the data as:

```
map<int, map<int, priority_queue<int, vector<int>, greater<int>>>>
   col  ->  row  ->         min-heap of values
```

- Outer `std::map`: columns sorted ascending.
- Inner `std::map`: rows sorted ascending within a column.
- `priority_queue<int, vector<int>, greater<int>>` (min-heap): values sorted ascending within the same `(col,row)` cell, satisfying the tertiary tie-break.

### Full C++ Solution

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    vector<vector<int>> verticalTraversal(TreeNode* root) {
        // col -> (row -> min-heap of values)
        map<int, map<int, priority_queue<int, vector<int>, greater<int>>>> grid;

        dfs(root, 0, 0, grid);

        vector<vector<int>> result;
        // Iterating a map walks columns in ascending order.
        for (auto& [col, rows] : grid) {
            vector<int> column;
            // Inner map walks rows in ascending order.
            for (auto& [row, cell] : rows) {
                // Drain the min-heap: smallest values first.
                while (!cell.empty()) {
                    column.push_back(cell.top());
                    cell.pop();
                }
            }
            result.push_back(column);
        }
        return result;
    }

private:
    void dfs(TreeNode* node, int row, int col,
             map<int, map<int, priority_queue<int, vector<int>, greater<int>>>>& grid) {
        if (node == nullptr) return;

        grid[col][row].push(node->val);

        dfs(node->left,  row + 1, col - 1, grid);
        dfs(node->right, row + 1, col + 1, grid);
    }
};
```

### Complexity

| Metric | Value | Reason |
| --- | --- | --- |
| Time | O(n log n) | Each insert into the nested structure / heap costs O(log n); n inserts dominate. |
| Space | O(n) | The grid stores every node once, plus O(h) recursion stack. |

### Note
A DFS or BFS both work for *visiting* nodes; the ordering is enforced entirely by the data structure, not the traversal order. That decoupling — "traverse however; let `std::map`/`std::priority_queue` impose the rule" — is the clean insight to state aloud.

---

## 2. Lowest Common Ancestor of Deepest Leaves (LC 1123)

### Problem Statement
Return the LCA of the **deepest leaves** of a binary tree. If there is a single deepest leaf, it is its own LCA. Equivalent to LC 865 ("Smallest Subtree with all the Deepest Nodes").

### Intuition — The Two-Return Approach
A single post-order pass can return, for each subtree, a pair: **(the LCA-of-deepest-leaves within this subtree, the depth of those deepest leaves)**. Combine children:

- Compute `left = solve(node->left)` and `right = solve(node->right)`.
- If `left.depth > right.depth`: the deepest leaves all live in the left subtree, so propagate the left answer with depth `left.depth + 1`.
- If `right.depth > left.depth`: symmetric — propagate the right answer.
- If **`left.depth == right.depth`**: the deepest leaves straddle both subtrees, so the **current node is their LCA**; propagate `node` with depth `left.depth + 1`.

Base case: a null node returns `(nullptr, 0)`. A leaf therefore returns `(leaf, 1)` because both children give depth 0 (equal), making the leaf its own LCA.

The "equal depths → current node" rule is the heart of the problem: equal-depth deepest leaves on both sides mean the divergence point is here.

### Full C++ Solution

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
    // Small struct carrying the running answer and its depth.
    struct Result {
        TreeNode* node; // LCA of deepest leaves within this subtree
        int depth;      // depth of those deepest leaves (relative to this subtree)
        Result(TreeNode* node, int depth) : node(node), depth(depth) {}
    };

public:
    TreeNode* lcaDeepestLeaves(TreeNode* root) {
        return solve(root).node;
    }

private:
    Result solve(TreeNode* node) {
        if (node == nullptr) return Result(nullptr, 0);

        Result left = solve(node->left);
        Result right = solve(node->right);

        if (left.depth > right.depth) {
            return Result(left.node, left.depth + 1);
        }
        if (right.depth > left.depth) {
            return Result(right.node, right.depth + 1);
        }
        // Equal depths: deepest leaves are on both sides -> this node is the LCA.
        return Result(node, left.depth + 1);
    }
};
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

### Full C++ Solution

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    vector<int> distanceK(TreeNode* root, TreeNode* target, int k) {
        // Phase 1: map each node to its parent.
        unordered_map<TreeNode*, TreeNode*> parent;
        buildParents(root, nullptr, parent);

        // Phase 2: BFS outward from target through up-to-three neighbors.
        vector<int> result;
        deque<TreeNode*> bfsQueue;
        unordered_set<TreeNode*> visited;

        bfsQueue.push_back(target);
        visited.insert(target);
        int dist = 0;

        while (!bfsQueue.empty()) {
            int size = bfsQueue.size();
            if (dist == k) {
                // Everything currently in the queue is exactly k edges away.
                for (auto* node : bfsQueue) {
                    result.push_back(node->val);
                }
                return result;
            }
            for (int i = 0; i < size; i++) {
                TreeNode* node = bfsQueue.front();
                bfsQueue.pop_front();
                // Three potential neighbors: left, right, parent.
                addNeighbor(node->left, visited, bfsQueue);
                addNeighbor(node->right, visited, bfsQueue);
                addNeighbor(parent.count(node) ? parent[node] : nullptr, visited, bfsQueue);
            }
            dist++;
        }
        return result; // k is deeper than any reachable node.
    }

private:
    void addNeighbor(TreeNode* neighbor, unordered_set<TreeNode*>& visited,
                     deque<TreeNode*>& bfsQueue) {
        if (neighbor != nullptr && visited.insert(neighbor).second) {
            bfsQueue.push_back(neighbor);
        }
    }

    void buildParents(TreeNode* node, TreeNode* par,
                      unordered_map<TreeNode*, TreeNode*>& parent) {
        if (node == nullptr) return;
        parent[node] = par;
        buildParents(node->left, node, parent);
        buildParents(node->right, node, parent);
    }
};
```

### Complexity

| Metric | Value | Reason |
| --- | --- | --- |
| Time | O(n) | One DFS to build parents + one BFS, each visiting nodes once. |
| Space | O(n) | Parent map, visited set, and queue each hold up to n nodes. |

### Note
`visited.insert(neighbor).second` returns `false` if the node was already present, giving a clean one-line "add only if unseen." Without `visited`, BFS would oscillate between a node and its parent forever.

---

## Google Interview Level Context (L4 / L5)

Google calibrates these problems differently by level — knowing the expectation helps you allocate time:

- **L4 (SWE II):** Expected to reach a correct, clean solution with good naming and structure. You should **enumerate edge cases unprompted** — empty tree, single node, `target` is the root, `k = 0`, `k` larger than the tree's reach. Stating time/space complexity precisely (and *why*) is mandatory, not optional.
- **L5 (Senior SWE):** Beyond correctness, demonstrate **design judgment**: choosing `std::map` for determinism (LC 987), recognizing the single-pass two-return formulation (LC 1123), and reframing a tree as a graph (LC 863). Expect **follow-ups** — "what if the tree is enormous and doesn't fit in memory?", "can you do LC 863 without a parent map?" (yes: a single DFS that returns the subtree distance to `target` and collects downward nodes as it unwinds), "how would you stream LC 987 output?".

General Google bar across levels:
- Talk through the **contract/invariant** before coding; don't dive into syntax first.
- Write **compilable** C++ with full method signatures and includes — Googlers dislike pseudocode.
- **Test your own code** on a small example by hand and walk the interviewer through it.
- Discuss **complexity trade-offs** and at least one **follow-up extension** proactively.

---

> **Last Updated:** 2026-06-26
