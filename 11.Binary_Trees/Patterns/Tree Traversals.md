> **Topic:** [Binary Trees](../README.md) · **Pattern 1 of 5**

# Tree Traversals

Traversal is the foundation of every binary-tree algorithm. Master the four core orders
(preorder, inorder, postorder, level-order) both recursively and iteratively, plus the two
"hard" traversals interviewers love: **zigzag** and **vertical order**.

```cpp
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int val) : val(val), left(nullptr), right(nullptr) {}
};
```

---

## 1. Recursive DFS (the clean 3-liners)

The recursive shape is identical for all three orders — only the *position* of the visit
statement changes.

```cpp
#include <bits/stdc++.h>
using namespace std;

// Preorder: Root -> Left -> Right
void preorder(TreeNode* node, vector<int>& out) {
    if (node == nullptr) return;
    out.push_back(node->val);              // visit
    preorder(node->left, out);
    preorder(node->right, out);
}

// Inorder: Left -> Root -> Right
void inorder(TreeNode* node, vector<int>& out) {
    if (node == nullptr) return;
    inorder(node->left, out);
    out.push_back(node->val);              // visit
    inorder(node->right, out);
}

// Postorder: Left -> Right -> Root
void postorder(TreeNode* node, vector<int>& out) {
    if (node == nullptr) return;
    postorder(node->left, out);
    postorder(node->right, out);
    out.push_back(node->val);              // visit
}
```

**Mnemonic:** the prefix (pre/in/post) tells you *where the Root is visited* relative to the
two recursive descents.

---

## 2. Iterative Inorder — push-left-first (LC 94)

Walk all the way left pushing onto a stack, then pop (visit), then move to the right child
and repeat.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> inorderTraversal(TreeNode* root) {
    vector<int> out;
    stack<TreeNode*> stk;
    TreeNode* curr = root;
    while (curr != nullptr || !stk.empty()) {
        while (curr != nullptr) {      // push the entire left spine
            stk.push(curr);
            curr = curr->left;
        }
        curr = stk.top(); stk.pop();   // leftmost unvisited node
        out.push_back(curr->val);      // visit
        curr = curr->right;            // then explore its right subtree
    }
    return out;
}
```

---

## 3. Iterative Preorder — push right, then left (LC 144)

Because a stack is LIFO, push the **right** child first so the **left** child is processed
next (preorder = Root, Left, Right).

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> preorderTraversal(TreeNode* root) {
    vector<int> out;
    if (root == nullptr) return out;
    stack<TreeNode*> stk;
    stk.push(root);
    while (!stk.empty()) {
        TreeNode* node = stk.top(); stk.pop();
        out.push_back(node->val);                        // visit root first
        if (node->right != nullptr) stk.push(node->right); // right pushed first
        if (node->left  != nullptr) stk.push(node->left);  // left popped first
    }
    return out;
}
```

---

## 4. Iterative Postorder (LC 145)

### Approach A — two stacks
Do a *modified preorder* (Root, Right, Left) onto `stack2`, then reverse it by popping.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> postorderTraversal(TreeNode* root) {
    vector<int> out;
    if (root == nullptr) return out;
    stack<TreeNode*> stk;
    stk.push(root);
    while (!stk.empty()) {
        TreeNode* node = stk.top(); stk.pop();
        out.push_back(node->val);                  // collect Root,Right,Left
        if (node->left  != nullptr) stk.push(node->left);
        if (node->right != nullptr) stk.push(node->right);
    }
    reverse(out.begin(), out.end()); // ends up Left, Right, Root
    return out;
}
```

This is the **reverse-of-modified-preorder trick**: preorder visiting Root→Right→Left,
collected with `push_back` then `reverse`, yields Left→Right→Root.

### Approach B — single stack with a `lastVisited` pointer
Visit a node only after its right child has been fully processed.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> postorderSingleStack(TreeNode* root) {
    vector<int> out;
    stack<TreeNode*> stk;
    TreeNode* curr = root;
    TreeNode* lastVisited = nullptr;
    while (curr != nullptr || !stk.empty()) {
        while (curr != nullptr) { stk.push(curr); curr = curr->left; }
        TreeNode* peek = stk.top();
        if (peek->right != nullptr && lastVisited != peek->right) {
            curr = peek->right;                   // go right before visiting
        } else {
            out.push_back(peek->val);
            lastVisited = stk.top(); stk.pop();
        }
    }
    return out;
}
```

---

## 5. Morris Traversal — O(1) space inorder

Threads each node's predecessor's right pointer to itself, removing the need for a stack.
Time O(n), space **O(1)** (mutates then restores pointers).

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
            if (pred->right == nullptr) {            // create thread
                pred->right = curr;
                curr = curr->left;
            } else {                             // thread exists -> remove it, visit
                pred->right = nullptr;
                out.push_back(curr->val);
                curr = curr->right;
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

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<vector<int>> levelOrder(TreeNode* root) {
    vector<vector<int>> result;
    if (root == nullptr) return result;
    queue<TreeNode*> q;
    q.push(root);
    while (!q.empty()) {
        int size = q.size();                 // snapshot: nodes on this level
        vector<int> level;
        level.reserve(size);
        for (int i = 0; i < size; i++) {
            TreeNode* node = q.front(); q.pop();
            level.push_back(node->val);
            if (node->left  != nullptr) q.push(node->left);
            if (node->right != nullptr) q.push(node->right);
        }
        result.push_back(level);
    }
    return result;
}
```

---

## 7. Zigzag Level Order (LC 103)

Run normal BFS but alternate the **insertion direction** within each level using a
`deque` + `push_front`/`push_back`. This keeps each insert O(1) (never inserting at an
arbitrary index).

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<vector<int>> zigzagLevelOrder(TreeNode* root) {
    vector<vector<int>> result;
    if (root == nullptr) return result;
    queue<TreeNode*> q;
    q.push(root);
    bool leftToRight = true;
    while (!q.empty()) {
        int size = q.size();
        deque<int> level;
        for (int i = 0; i < size; i++) {
            TreeNode* node = q.front(); q.pop();
            if (leftToRight) level.push_back(node->val);
            else             level.push_front(node->val);
            if (node->left  != nullptr) q.push(node->left);
            if (node->right != nullptr) q.push(node->right);
        }
        result.push_back(vector<int>(level.begin(), level.end()));
        leftToRight = !leftToRight;
    }
    return result;
}
```

---

## 8. N-ary Level Order (LC 429)

Identical BFS, but children come from a list (`node.children`).

```cpp
#include <bits/stdc++.h>
using namespace std;

struct Node { int val; vector<Node*> children; };

vector<vector<int>> levelOrder(Node* root) {
    vector<vector<int>> result;
    if (root == nullptr) return result;
    queue<Node*> q;
    q.push(root);
    while (!q.empty()) {
        int size = q.size();
        vector<int> level;
        level.reserve(size);
        for (int i = 0; i < size; i++) {
            Node* node = q.front(); q.pop();
            level.push_back(node->val);
            for (auto& child : node->children) q.push(child);
        }
        result.push_back(level);
    }
    return result;
}
```

---

## 9. Vertical Order Traversal (LC 987)

Group nodes by **column** (root = 0, left = col−1, right = col+1). Within a column, order by
**row** (depth), and ties at the same (row, col) by **value**. A
`map<col, map<row, priority_queue<val>>>` makes ordering deterministic without a final
sort — `std::map` is used (not `std::unordered_map`) precisely because we need keys iterated in sorted
order.

```cpp
#include <bits/stdc++.h>
using namespace std;

void dfs(TreeNode* node, int row, int col,
         map<int, map<int, priority_queue<int, vector<int>, greater<int>>>>& mp) {
    if (node == nullptr) return;
    mp[col][row].push(node->val);
    dfs(node->left,  row + 1, col - 1, mp);
    dfs(node->right, row + 1, col + 1, mp);
}

vector<vector<int>> verticalTraversal(TreeNode* root) {
    // col -> (row -> min-heap of values)
    map<int, map<int, priority_queue<int, vector<int>, greater<int>>>> mp;
    dfs(root, 0, 0, mp);

    vector<vector<int>> result;
    for (auto& [colKey, rows] : mp) {
        vector<int> col;
        for (auto& [rowKey, pq] : rows) {
            auto tmp = pq; // copy so we can pop
            while (!tmp.empty()) { col.push_back(tmp.top()); tmp.pop(); }
        }
        result.push_back(col);
    }
    return result;
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
| Vertical Order (LC 987) | DFS + std::map | O(n log n) | O(n) |

*h = tree height (O(log n) balanced, O(n) skewed); w = max width of any level.*

---

### Related
- [Level Order](Level%20Order.md) — the full BFS catalog
- [Tree Construction](Tree%20Construction.md) — rebuilding trees from traversal arrays

> **Last Updated:** 2026-06-26
