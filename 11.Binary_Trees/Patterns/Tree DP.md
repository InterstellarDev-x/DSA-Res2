> **Topic:** [Binary Trees](../README.md) · **Pattern 5 of 5**

# Tree DP / Advanced

**Core idea:** a *post-order* DFS where each node **returns aggregated information about its
subtree** to its parent, and (optionally) a **global result** is updated at each node. The
recurrence lives in how a parent combines its children's returns.

```cpp
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int val) : val(val), left(nullptr), right(nullptr) {}
};
```

> The template: `f(node)` returns a *summary* of the subtree rooted at `node`; the parent
> combines `f(node.left)` and `f(node.right)` plus `node.val`. When the answer can occur
> "through" any node (not just the root), keep a global and update it inside `f`.

---

## 1. House Robber III (LC 337)

You can't rob two directly-connected nodes. Each node returns a 2-element state:
`[notRob, rob]` where `notRob` = best when this node is NOT robbed (children free to do
either), `rob` = `node.val + children's notRob`.

```cpp
#include <bits/stdc++.h>
using namespace std;

// returns {notRobbingThisNode, robbingThisNode}
vector<int> robSub(TreeNode* node) {
    if (node == nullptr) return {0, 0};
    auto l = robSub(node->left);
    auto r = robSub(node->right);
    int notRob = max(l[0], l[1]) + max(r[0], r[1]);  // children choose freely
    int rob    = node->val + l[0] + r[0];             // children must NOT be robbed
    return {notRob, rob};
}

int rob(TreeNode* root) {
    auto res = robSub(root);
    return max(res[0], res[1]);
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

```cpp
int cameras = 0;
const int NOT_COVERED = 0, HAS_CAMERA = 1, COVERED = 2;

int dfs(TreeNode* node) {
    if (node == nullptr) return COVERED;          // null is implicitly covered
    int l = dfs(node->left);
    int r = dfs(node->right);
    if (l == NOT_COVERED || r == NOT_COVERED) {
        cameras++;
        return HAS_CAMERA;
    }
    if (l == HAS_CAMERA || r == HAS_CAMERA) return COVERED;
    return NOT_COVERED;
}

int minCameraCover(TreeNode* root) {
    if (dfs(root) == NOT_COVERED) cameras++;   // root left uncovered -> place a camera
    return cameras;
}
```

---

## 3. Distribute Coins in Binary Tree (LC 979)

Each node should end with exactly 1 coin. The number of moves across an edge equals the
**absolute surplus/deficit** flowing through it. At each node compute
`excess = leftExcess + rightExcess + node.val − 1`; add `|excess|` to the answer and return
`excess` to the parent.

```cpp
#include <bits/stdc++.h>
using namespace std;

int moves = 0;

int excess(TreeNode* node) {
    if (node == nullptr) return 0;
    int left  = excess(node->left);
    int right = excess(node->right);
    moves += abs(left) + abs(right);     // coins crossing both child edges
    return node->val + left + right - 1; // surplus(+)/deficit(-) passed up
}

int distributeCoins(TreeNode* root) {
    excess(root);
    return moves;
}
```

---

## 4. Lowest Common Ancestor (LC 236)

If the current node is `p` or `q`, return it. Recurse both sides; if **both** return non-null,
the current node is the LCA. Otherwise propagate the non-null side up.

```cpp
TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
    if (root == nullptr || root == p || root == q) return root;
    TreeNode* left  = lowestCommonAncestor(root->left,  p, q);
    TreeNode* right = lowestCommonAncestor(root->right, p, q);
    if (left != nullptr && right != nullptr) return root;   // p and q on opposite sides -> LCA
    return left != nullptr ? left : right;                  // both on one side (or neither)
}
```

**Why early-return works:** a node returns non-null iff its subtree contains `p` or `q`. The
first node to receive non-null from *both* sides is the split point = the LCA. If one of
`p`/`q` is an ancestor of the other, the early `root == p` return makes that ancestor the LCA.

---

## 5. LCA of Deepest Leaves (LC 1123)

Return a `(node, depth)` pair upward. Pick the side with greater depth; if both depths are
equal, the **current node** is the LCA of all deepest leaves below it.

```cpp
struct Result {
    TreeNode* node;
    int depth;
    Result(TreeNode* node, int depth) : node(node), depth(depth) {}
};

Result dfs(TreeNode* node) {
    if (node == nullptr) return Result(nullptr, 0);
    Result l = dfs(node->left);
    Result r = dfs(node->right);
    if (l.depth == r.depth) return Result(node, l.depth + 1);  // balanced -> this node
    return l.depth > r.depth
        ? Result(l.node, l.depth + 1)
        : Result(r.node, r.depth + 1);
}

TreeNode* lcaDeepestLeaves(TreeNode* root) {
    return dfs(root).node;
}
```

---

## 6. Delete Nodes and Return Forest (LC 1110)

Put the to-delete values in an `unordered_set`. Post-order: a child that survives **becomes a new root**
if its parent was deleted (or if it is the original root). When a node is deleted, its
surviving children are added to the forest; the node returns `nullptr` to detach itself.

```cpp
#include <bits/stdc++.h>
using namespace std;

// returns the node if it survives, else nullptr (so the parent can detach it)
TreeNode* dfs(TreeNode* node, bool isRoot,
              unordered_set<int>& toDelete, vector<TreeNode*>& forest) {
    if (node == nullptr) return nullptr;
    bool deleted = toDelete.count(node->val) > 0;
    node->left  = dfs(node->left,  deleted, toDelete, forest);   // child is a root if I'm deleted
    node->right = dfs(node->right, deleted, toDelete, forest);
    if (deleted) {
        if (node->left  != nullptr) forest.push_back(node->left);
        if (node->right != nullptr) forest.push_back(node->right);
        return nullptr;
    }
    return node;
}

vector<TreeNode*> delNodes(TreeNode* root, vector<int>& to_delete) {
    unordered_set<int> toDelete(to_delete.begin(), to_delete.end());
    vector<TreeNode*> forest;
    if (dfs(root, true, toDelete, forest) != nullptr) forest.push_back(root);
    return forest;
}
```

(The top-level `root` is added in `delNodes` if it survives; surviving children of deleted
nodes are added inside `dfs`.)

---

## 7. All Nodes Distance K (LC 863)

Two phases: (1) DFS to build a `child → parent` map so the tree becomes an undirected graph;
(2) BFS from `target`, expanding to left/right/parent, until you reach distance `K`.

```cpp
#include <bits/stdc++.h>
using namespace std;

void buildParents(TreeNode* node, TreeNode* par,
                  unordered_map<TreeNode*, TreeNode*>& parent) {
    if (node == nullptr) return;
    parent[node] = par;
    buildParents(node->left,  node, parent);
    buildParents(node->right, node, parent);
}

vector<int> distanceK(TreeNode* root, TreeNode* target, int k) {
    unordered_map<TreeNode*, TreeNode*> parent;
    buildParents(root, nullptr, parent);

    queue<TreeNode*> q;
    unordered_set<TreeNode*> visited;
    q.push(target);
    visited.insert(target);
    int dist = 0;
    while (!q.empty()) {
        if (dist == k) {                          // current frontier is exactly distance k
            vector<int> res;
            while (!q.empty()) {
                res.push_back(q.front()->val);
                q.pop();
            }
            return res;
        }
        int sz = q.size();
        for (int i = 0; i < sz; i++) {
            TreeNode* node = q.front(); q.pop();
            for (TreeNode* nei : {node->left, node->right, parent[node]}) {
                if (nei != nullptr && visited.insert(nei).second) q.push(nei);
            }
        }
        dist++;
    }
    return {};
}
```

(`parent[node] = par` is the direct map insertion for building the adjacency relation; here
each node has exactly one parent so a direct assignment suffices.)

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
