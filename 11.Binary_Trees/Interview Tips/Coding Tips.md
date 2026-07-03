> **Topic:** [Binary Trees](../README.md) · **Pattern 1 of 4**

# Binary Trees — Coding Tips

A focused set of practical coding tips that make binary-tree problems faster to write and harder to get wrong in an interview setting. Each tip pairs a short explanation with a C++ snippet you can reproduce from memory.

Throughout these tips we assume the standard node definition:

```cpp
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int val) : val(val), left(nullptr), right(nullptr) {}
};
```

---

## 1. TreeNode null-check discipline

The single most reliable habit in tree recursion: **the first line of every recursive function handles the `nullptr` node**. This terminates recursion at the "edges" of the tree (below the leaves) and removes the need to null-check children individually before recursing.

If you make this the very first statement every time, you will almost never hit a null pointer dereference, and your base case is always explicit.

```cpp
int maxDepth(TreeNode* node) {
    if (node == nullptr) return 0;            // first line, always
    return 1 + max(maxDepth(node->left), maxDepth(node->right));
}
```

The return value of the base case is problem-specific (`0` for depth/height, `nullptr` for "no node found", `true` for vacuously-true predicates, etc.), but the *shape* is invariant.

---

## 2. Leaf check pattern

A leaf is a node with **no children**. Many problems (Path Sum, Min Depth, Sum Root to Leaf, Binary Tree Paths) treat leaves specially, so commit the exact condition to muscle memory:

```cpp
if (node->left == nullptr && node->right == nullptr) {
    // this is a leaf — terminate the path / record the answer here
}
```

Be careful to distinguish this from the `nullptr`-node base case. A common bug in **Min Depth** is treating a node with one child as a leaf — it is not. You only short-circuit on a *true* leaf, and otherwise take the depth of the non-null subtree.

```cpp
int minDepth(TreeNode* node) {
    if (node == nullptr) return 0;
    if (node->left == nullptr) return 1 + minDepth(node->right);  // not a leaf
    if (node->right == nullptr) return 1 + minDepth(node->left);  // not a leaf
    return 1 + min(minDepth(node->left), minDepth(node->right));
}
```

---

## 3. Diameter doesn't pass through the root — track a global max

A frequent misconception is that the diameter (longest path between any two nodes) must run through the root. It does not. The longest path can sit entirely inside a subtree.

The trick: write a helper that **returns the height** of a subtree, but as a side effect **updates a global** with `leftHeight + rightHeight` at every node. The diameter is the maximum of those local sums across the whole tree.

```cpp
int best = 0;               // global state

int height(TreeNode* node) {
    if (node == nullptr) return 0;
    int left = height(node->left);
    int right = height(node->right);
    best = max(best, left + right);   // update at EVERY node
    return 1 + max(left, right);            // return value is the height
}
```

This "return one thing, accumulate another globally" pattern generalizes to Max Path Sum (Tip 7), House Robber III, and more.

---

## 4. BFS size snapshot

Level-order traversal (BFS) needs to know **where one level ends and the next begins**. The clean way is to capture the queue size *before* the inner loop, because the queue grows as you enqueue children.

```cpp
#include <bits/stdc++.h>
using namespace std;

queue<TreeNode*> q;
q.push(root);
while (!q.empty()) {
    int size = q.size();           // SNAPSHOT before the inner loop
    vector<int> level;
    for (int i = 0; i < size; i++) {   // iterate exactly this level
        TreeNode* node = q.front(); q.pop();
        level.push_back(node->val);
        if (node->left != nullptr) q.push(node->left);
        if (node->right != nullptr) q.push(node->right);
    }
    // `level` now holds one full level
}
```

If you write `for (int i = 0; i < q.size(); i++)` instead, the loop bound shifts as children are added and levels bleed into each other.

---

## 5. Morris traversal — O(1) space inorder via threading

Recursive and stack-based inorder both use O(h) extra space. **Morris traversal** achieves O(1) space (ignoring output) by temporarily *threading* each node to its inorder successor.

Core idea: for the current node, find its inorder predecessor (rightmost node of its left subtree). Create a temporary right-pointer ("thread") from that predecessor back to the current node so you can climb back up without a stack. On the second visit, remove the thread and emit the value.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> morrisInorder(TreeNode* root) {
    vector<int> out;
    TreeNode* curr = root;
    while (curr != nullptr) {
        if (curr->left == nullptr) {
            out.push_back(curr->val);
            curr = curr->right;
        } else {
            TreeNode* pred = curr->left;
            while (pred->right != nullptr && pred->right != curr) pred = pred->right;
            if (pred->right == nullptr) {
                pred->right = curr;       // create thread
                curr = curr->left;
            } else {
                pred->right = nullptr;       // remove thread, restore tree
                out.push_back(curr->val);
                curr = curr->right;
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

```cpp
#include <bits/stdc++.h>
using namespace std;

int widthOfBinaryTree(TreeNode* root) {
    if (root == nullptr) return 0;
    int maxW = 0;
    queue<TreeNode*> nodes;
    queue<int> idxs;
    nodes.push(root);
    idxs.push(0);
    while (!nodes.empty()) {
        int size = nodes.size();
        int leftmost = idxs.front();          // base for this level
        int first = 0, last = 0;
        for (int i = 0; i < size; i++) {
            TreeNode* node = nodes.front(); nodes.pop();
            int idx = idxs.front() - leftmost; idxs.pop(); // NORMALIZE
            if (i == 0) first = idx;
            if (i == size - 1) last = idx;
            if (node->left != nullptr)  { nodes.push(node->left);  idxs.push(2 * idx); }
            if (node->right != nullptr) { nodes.push(node->right); idxs.push(2 * idx + 1); }
        }
        maxW = max(maxW, last - first + 1);
    }
    return maxW;
}
```

---

## 7. `gain = max(0, gain)` for Max Path Sum

In Maximum Path Sum, each node reports to its parent the best **single downward path gain** it can contribute. If a subtree's best contribution is negative, it is better to contribute **nothing** (i.e. cut that branch off) than to drag the path down.

Clamp each child's gain at zero before using it:

```cpp
int best = INT_MIN;            // global (use an instance var or reference)

int gain(TreeNode* node) {
    if (node == nullptr) return 0;
    int left  = max(0, gain(node->left));   // prune negatives
    int right = max(0, gain(node->right));
    best = max(best, node->val + left + right);  // path "through" this node
    return node->val + max(left, right);         // gain handed to parent
}
```

The path that bends through a node uses *both* children (`left + right`), but the gain handed upward can only follow *one* branch (`max(left, right)`), because a path cannot fork.

---

## 8. Parent-map for distance problems

Trees only have child pointers, so "move toward the parent" is impossible by default. For problems like **All Nodes Distance K** or **LCA Deepest Leaves**, first build a `child -> parent` map, then you can BFS outward in all three directions (left child, right child, parent).

```cpp
#include <bits/stdc++.h>
using namespace std;

unordered_map<TreeNode*, TreeNode*> parent;

void buildParents(TreeNode* node, TreeNode* par) {
    if (node == nullptr) return;
    parent[node] = par;
    buildParents(node->left, node);
    buildParents(node->right, node);
}
```

Once built, a standard BFS from the target node — enqueuing `node->left`, `node->right`, and `parent[node]` while tracking visited nodes — finds everything at distance K.

---

## 9. Backtrack discipline for path collection

When collecting root-to-leaf paths (Binary Tree Paths, Path Sum II), you reuse a single mutable `path` vector and **undo** your change after recursing. The pattern is: add, recurse into both children, then remove the last element.

```cpp
void dfs(TreeNode* node, vector<int>& path, vector<vector<int>>& result) {
    if (node == nullptr) return;
    path.push_back(node->val);                              // choose
    if (node->left == nullptr && node->right == nullptr) {
        result.push_back(path);           // record a COPY at a leaf
    } else {
        dfs(node->left, path, result);                // explore
        dfs(node->right, path, result);
    }
    path.pop_back();                    // un-choose (backtrack)
}
```

Two non-negotiables: (1) `result.push_back(path)` copies the vector by value automatically in C++, so you never store a dangling reference; (2) the `path.pop_back()` runs on **every** return, including after recording a leaf, so it must sit outside the leaf branch as shown.

---

> **Last Updated:** 2026-06-26
