> **Topic:** [Binary Trees](../README.md) · **Pattern 2 of 4**

# Binary Trees — Common Mistakes

Ten bugs that show up over and over in binary-tree interviews. Each one pairs the broken version (❌) with the fix (✅). Node definition used throughout:

```cpp
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int val) : val(val), left(nullptr), right(nullptr) {}
};
```

---

## 1. Missing null check before accessing `node->left` / `node->right`

Dereferencing a child without first confirming the node exists is the classic crash (segmentation fault). Always handle `nullptr` as the first line.

❌ **Wrong**
```cpp
#include <bits/stdc++.h>
using namespace std;

int maxDepth(TreeNode* node) {
    int left = maxDepth(node->left);    // crash when node == nullptr
    int right = maxDepth(node->right);
    return 1 + max(left, right);
}
```

✅ **Correct**
```cpp
#include <bits/stdc++.h>
using namespace std;

int maxDepth(TreeNode* node) {
    if (node == nullptr) return 0;        // base case first
    int left = maxDepth(node->left);
    int right = maxDepth(node->right);
    return 1 + max(left, right);
}
```

---

## 2. Diameter: returning the depth/diameter instead of updating a global

The helper must **return the height** but **record the diameter in a global**. Returning the local `left + right` upward corrupts the height calculation of ancestors.

❌ **Wrong**
```cpp
int diameter(TreeNode* node) {
    if (node == nullptr) return 0;
    int left = diameter(node->left);
    int right = diameter(node->right);
    return left + right;               // this is NOT a height; ancestors break
}
```

✅ **Correct**
```cpp
#include <bits/stdc++.h>
using namespace std;

int best = 0;

int height(TreeNode* node) {
    if (node == nullptr) return 0;
    int left = height(node->left);
    int right = height(node->right);
    best = max(best, left + right);   // diameter recorded globally
    return 1 + max(left, right);      // height returned upward
}
```

---

## 3. Max Path Sum: forgetting `max(0, gain)`

Without clamping, a negative subtree contribution is allowed to drag the path total down, producing wrong answers on trees with negative values.

❌ **Wrong**
```cpp
#include <bits/stdc++.h>
using namespace std;

int gain(TreeNode* node) {
    if (node == nullptr) return 0;
    int left = gain(node->left);        // may be negative and still used
    int right = gain(node->right);
    best = max(best, node->val + left + right);
    return node->val + max(left, right);
}
```

✅ **Correct**
```cpp
#include <bits/stdc++.h>
using namespace std;

int gain(TreeNode* node) {
    if (node == nullptr) return 0;
    int left  = max(0, gain(node->left));    // prune negatives
    int right = max(0, gain(node->right));
    best = max(best, node->val + left + right);
    return node->val + max(left, right);
}
```

---

## 4. Path Sum II: not backtracking (forgetting to remove from the list)

Reusing a shared list without removing the current node after recursion leaves stale entries, so sibling paths inherit nodes that aren't on them.

❌ **Wrong**
```cpp
#include <bits/stdc++.h>
using namespace std;

void dfs(TreeNode* node, int target, vector<int>& path, vector<vector<int>>& res) {
    if (node == nullptr) return;
    path.push_back(node->val);
    if (node->left == nullptr && node->right == nullptr && node->val == target)
        res.push_back(path);
    dfs(node->left,  target - node->val, path, res);
    dfs(node->right, target - node->val, path, res);
    // missing removal — path keeps growing across branches
}
```

✅ **Correct**
```cpp
#include <bits/stdc++.h>
using namespace std;

void dfs(TreeNode* node, int target, vector<int>& path, vector<vector<int>>& res) {
    if (node == nullptr) return;
    path.push_back(node->val);
    if (node->left == nullptr && node->right == nullptr && node->val == target)
        res.push_back(path);
    dfs(node->left,  target - node->val, path, res);
    dfs(node->right, target - node->val, path, res);
    path.pop_back();      // backtrack on every return
}
```

---

## 5. Level Order: not capturing size before the inner loop

Reading `q.size()` inside the loop condition lets the bound drift as children are enqueued, merging levels together.

❌ **Wrong**
```cpp
while (!q.empty()) {
    vector<int> level;
    for (int i = 0; i < (int)q.size(); i++) {   // size() changes mid-loop
        TreeNode* node = q.front(); q.pop();
        level.push_back(node->val);
        if (node->left != nullptr) q.push(node->left);
        if (node->right != nullptr) q.push(node->right);
    }
    result.push_back(level);
}
```

✅ **Correct**
```cpp
while (!q.empty()) {
    int size = (int)q.size();           // snapshot the level boundary
    vector<int> level;
    for (int i = 0; i < size; i++) {
        TreeNode* node = q.front(); q.pop();
        level.push_back(node->val);
        if (node->left != nullptr) q.push(node->left);
        if (node->right != nullptr) q.push(node->right);
    }
    result.push_back(level);
}
```

---

## 6. Construct from traversals: off-by-one in `leftSize`

When splitting inorder around the root, the count of left-subtree nodes is `inIdx - inStart` (a *count* of elements strictly before the root), not `inIdx - inStart + 1` (which wrongly includes the root).

❌ **Wrong**
```cpp
int inIdx = inMap[rootVal];
int leftSize = inIdx - inStart + 1;    // includes the root — too large by one
```

✅ **Correct**
```cpp
int inIdx = inMap[rootVal];
int leftSize = inIdx - inStart;        // nodes strictly left of the root
// use leftSize to slice the preorder/postorder ranges for the left subtree
```

---

## 7. Binary Tree Cameras: missing the root check after recursion

The greedy post-order returns states (0 = not covered, 1 = has camera, 2 = covered). After the recursion completes, if the **root itself** comes back uncovered (state 0), you must add one more camera — easy to forget.

❌ **Wrong**
```cpp
int cameras = 0;

int minCameraCover(TreeNode* root) {
    dfs(root);
    return cameras;                    // ignores an uncovered root → undercount
}
```

✅ **Correct**
```cpp
int cameras = 0;

int minCameraCover(TreeNode* root) {
    if (dfs(root) == 0) cameras++;     // root left uncovered → place a camera
    return cameras;
}

// dfs returns: 0 = needs cover, 1 = has camera, 2 = covered
int dfs(TreeNode* node) {
    if (node == nullptr) return 2;
    int left = dfs(node->left);
    int right = dfs(node->right);
    if (left == 0 || right == 0) { cameras++; return 1; }
    if (left == 1 || right == 1) return 2;
    return 0;
}
```

---

## 8. LCA: returning `nullptr` incorrectly for nodes present in the tree

The lowest-common-ancestor base case must return the node when it matches `p` or `q`. Returning `nullptr` for a matched node (or short-circuiting before checking the match) loses the answer.

❌ **Wrong**
```cpp
TreeNode* lca(TreeNode* node, TreeNode* p, TreeNode* q) {
    if (node == nullptr) return nullptr;
    if (node->left == nullptr && node->right == nullptr) return nullptr;  // drops leaf matches
    TreeNode* left = lca(node->left, p, q);
    TreeNode* right = lca(node->right, p, q);
    if (left != nullptr && right != nullptr) return node;
    return left != nullptr ? left : right;
}
```

✅ **Correct**
```cpp
TreeNode* lca(TreeNode* node, TreeNode* p, TreeNode* q) {
    if (node == nullptr || node == p || node == q) return node;   // match returns self
    TreeNode* left = lca(node->left, p, q);
    TreeNode* right = lca(node->right, p, q);
    if (left != nullptr && right != nullptr) return node;            // split → this is LCA
    return left != nullptr ? left : right;
}
```

---

## 9. Zigzag: using `result[level].insert(result[level].begin(), val)` — O(n) — instead of a deque

Inserting at index 0 of a `vector` shifts every element, turning each level into O(n²). Use a `deque` and `push_front` for O(1) front insertion.

❌ **Wrong**
```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> level;
// ... for each node on the level:
if (leftToRight) level.push_back(node->val);
else             level.insert(level.begin(), node->val);   // O(n) shift each insert
```

✅ **Correct**
```cpp
#include <bits/stdc++.h>
using namespace std;

deque<int> level;
// ... for each node on the level:
if (leftToRight) level.push_back(node->val);    // O(1)
else             level.push_front(node->val);   // O(1)
result.push_back(vector<int>(level.begin(), level.end()));
```

---

## 10. Max Width: integer overflow without index normalization

Heap-style indices (`2*i`, `2*i+1`) blow past `int`/`long` range on deep sparse trees. Normalize each level against its leftmost index so only the differences — which is all the width needs — are preserved.

❌ **Wrong**
```cpp
for (int i = 0; i < size; i++) {
    TreeNode* node = nodes.front(); nodes.pop();
    int idx = idxs.front(); idxs.pop();                 // grows exponentially → overflow
    if (node->left != nullptr)  { nodes.push(node->left);  idxs.push(2 * idx); }
    if (node->right != nullptr) { nodes.push(node->right); idxs.push(2 * idx + 1); }
}
```

✅ **Correct**
```cpp
int leftmost = idxs.front();                // base for this level
for (int i = 0; i < size; i++) {
    TreeNode* node = nodes.front(); nodes.pop();
    int idx = idxs.front() - leftmost; idxs.pop();      // NORMALIZE → indices stay small
    if (i == 0) first = idx;
    if (i == size - 1) last = idx;
    if (node->left != nullptr)  { nodes.push(node->left);  idxs.push(2 * idx); }
    if (node->right != nullptr) { nodes.push(node->right); idxs.push(2 * idx + 1); }
}
maxWidth = max(maxWidth, last - first + 1);
```

---

> **Last Updated:** 2026-06-26
