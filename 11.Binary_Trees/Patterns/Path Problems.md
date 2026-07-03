> **Topic:** [Binary Trees](../README.md) · **Pattern 3 of 5**

# Path Problems

A "path" in a binary tree is a sequence of connected nodes. Problems differ by **where the
path may start/end** (root→leaf, any→any), and whether you **collect** paths or **score**
them. Two recurring techniques: **leaf-base-case checks** and **backtracking** for collection.

```cpp
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int val) : val(val), left(nullptr), right(nullptr) {}
};
```

---

## 1. Path Sum (LC 112) — does a root→leaf path equal target?

The base case is the **leaf check**: a leaf has no children, so the path ends there.

```cpp
bool hasPathSum(TreeNode* root, int targetSum) {
    if (root == nullptr) return false;
    if (root->left == nullptr && root->right == nullptr)      // leaf
        return root->val == targetSum;
    int remaining = targetSum - root->val;
    return hasPathSum(root->left, remaining)
        || hasPathSum(root->right, remaining);
}
```

> Don't use `root == nullptr && targetSum == 0` as the success base case — an empty child of a
> one-child node would falsely succeed. Always check the **leaf** condition explicitly.

---

## 2. Path Sum II (LC 113) — collect all root→leaf paths summing to target

Classic **backtracking**: add the node to the path, recurse, then *remove it* so siblings
start from a clean path.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<vector<int>> pathSum(TreeNode* root, int targetSum) {
    vector<vector<int>> result;
    vector<int> path;
    backtrack(root, targetSum, path, result);
    return result;
}

void backtrack(TreeNode* node, int remaining,
               vector<int>& path, vector<vector<int>>& result) {
    if (node == nullptr) return;
    path.push_back(node->val);                                  // choose
    if (node->left == nullptr && node->right == nullptr && remaining == node->val) {
        result.push_back(path);               // copy! the path is mutable
    } else {
        backtrack(node->left,  remaining - node->val, path, result);
        backtrack(node->right, remaining - node->val, path, result);
    }
    path.pop_back();                        // un-choose (backtrack)
}
```

> ⚠️ Two non-negotiables: **copy** the path when recording (`result.push_back(path)`), and
> **remove** the last element on the way out. Forgetting either is the #1 bug here.

---

## 3. Diameter of Binary Tree (LC 543)

The diameter is the longest path *in edges* between any two nodes — it **need not pass
through the root**. Compute each node's depth bottom-up while updating a global max of
`leftDepth + rightDepth`.

```cpp
int maxDiameter = 0;

int diameterOfBinaryTree(TreeNode* root) {
    depth(root);
    return maxDiameter;
}

int depth(TreeNode* node) {
    if (node == nullptr) return 0;
    int left  = depth(node->left);
    int right = depth(node->right);
    maxDiameter = max(maxDiameter, left + right);   // path THROUGH this node
    return 1 + max(left, right);                    // depth returned to parent
}
```

**The dual-purpose recursion insight:** the helper *returns* depth to its parent but
*updates a global* with the through-node path length. The thing you return and the thing you
maximize are different.

---

## 4. Binary Tree Maximum Path Sum (LC 124) — Hard

Same dual-purpose shape as diameter, with two twists:
1. **Prune negative subtrees:** `gain = max(0, gain)` — a negative subtree contributes
   nothing, so we treat it as 0.
2. The best path **through** a node is `node->val + leftGain + rightGain`, but the value
   **returned** to the parent can only follow one branch:
   `node->val + max(leftGain, rightGain)`.

```cpp
#include <bits/stdc++.h>
using namespace std;

int maxSum = INT_MIN;

int maxPathSum(TreeNode* root) {
    gain(root);
    return maxSum;
}

int gain(TreeNode* node) {
    if (node == nullptr) return 0;
    int leftGain  = max(0, gain(node->left));    // ignore negative contributions
    int rightGain = max(0, gain(node->right));
    maxSum = max(maxSum, node->val + leftGain + rightGain);  // path through node
    return node->val + max(leftGain, rightGain);             // extendable to parent
}
```

---

## 5. Sum Root to Leaf Numbers (LC 129)

Each root→leaf path spells a number (e.g. `1→2→3` = 123). Carry the running value **down**
the recursion: `current = current * 10 + node->val`.

```cpp
int sumNumbers(TreeNode* root) {
    return dfs(root, 0);
}

int dfs(TreeNode* node, int current) {
    if (node == nullptr) return 0;
    current = current * 10 + node->val;
    if (node->left == nullptr && node->right == nullptr) return current;   // leaf -> the number
    return dfs(node->left, current) + dfs(node->right, current);
}
```

This is **top-down** accumulation — state flows down via the parameter, results sum on the
way back up.

---

## 6. Binary Tree Paths (LC 257) — all root→leaf paths as strings

Build a `"a->b->c"` string per path. Use a `string` with length-rollback (backtrack),
or simple string concatenation passed down.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<string> binaryTreePaths(TreeNode* root) {
    vector<string> result;
    if (root != nullptr) dfs(root, "", result);
    return result;
}

void dfs(TreeNode* node, string path, vector<string>& result) {
    int len = path.length();                       // remember length to roll back
    if (path.length() > 0) path += "->";
    path += to_string(node->val);
    if (node->left == nullptr && node->right == nullptr) {
        result.push_back(path);
    } else {
        if (node->left  != nullptr) dfs(node->left,  path, result);
        if (node->right != nullptr) dfs(node->right, path, result);
    }
    path.resize(len);                           // backtrack the string
}
```

Simpler (less efficient) alternative: pass `path + "->" + to_string(node->val)` or
build with a delimiter — fine for interview clarity.

---

## Complexity

| Problem | LC | Time | Space |
|---|---|---|---|
| Path Sum | 112 | O(n) | O(h) |
| Path Sum II | 113 | O(n²) worst (copying paths) | O(n) paths + O(h) |
| Diameter | 543 | O(n) | O(h) |
| Maximum Path Sum | 124 | O(n) | O(h) |
| Sum Root to Leaf | 129 | O(n) | O(h) |
| Binary Tree Paths | 257 | O(n·h) (string build) | O(h) |

*h = height. Path Sum II is O(n²) worst because each of up to O(n) leaves may copy an
O(h)-length path.*

---

### Related
- [Tree DP](Tree%20DP.md) — same bottom-up "return value + global" template applied to DP
- [Tree Traversals](Tree%20Traversals.md) — DFS foundations

> **Last Updated:** 2026-06-26
