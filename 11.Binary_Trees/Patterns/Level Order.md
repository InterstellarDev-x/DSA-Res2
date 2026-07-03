> **Topic:** [Binary Trees](../README.md) · **Pattern 4 of 5**

# Level Order / BFS

Every level-order problem is the same BFS skeleton with a small per-level twist. Memorize the
template once; then "take the last node" → right side view, "average" → averages, "max" →
largest values, and so on.

```cpp
#include <bits/stdc++.h>
using namespace std;

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
```

---

## The BFS Template (size-snapshot)

```cpp
#include <bits/stdc++.h>
using namespace std;

queue<TreeNode*> q;
q.push(root);                               // assume root != nullptr (guard separately)
while (!q.empty()) {
    int size = q.size();                    // SNAPSHOT before the inner loop
    for (int i = 0; i < size; i++) {
        TreeNode* node = q.front(); q.pop();
        // ... per-node work (often using i == 0 or i == size-1) ...
        if (node->left  != nullptr) q.push(node->left);
        if (node->right != nullptr) q.push(node->right);
    }
    // ... per-level aggregation ...
}
```

> ⚠️ The single most common BFS bug: reading `q.size()` *inside* the loop after you've
> already enqueued children. **Snapshot it first.**

---

## 1. Right Side View (LC 199) — last node per level

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> rightSideView(TreeNode* root) {
    vector<int> result;
    if (root == nullptr) return result;
    queue<TreeNode*> q;
    q.push(root);
    while (!q.empty()) {
        int size = q.size();
        for (int i = 0; i < size; i++) {
            TreeNode* node = q.front(); q.pop();
            if (i == size - 1) result.push_back(node->val);   // rightmost node on this level
            if (node->left  != nullptr) q.push(node->left);
            if (node->right != nullptr) q.push(node->right);
        }
    }
    return result;
}
```

---

## 2. Average of Levels (LC 637) — sum / size per level

Use a `double`/`long long` accumulator to avoid `int` overflow when summing.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<double> averageOfLevels(TreeNode* root) {
    vector<double> result;
    queue<TreeNode*> q;
    q.push(root);
    while (!q.empty()) {
        int size = q.size();
        long long sum = 0;
        for (int i = 0; i < size; i++) {
            TreeNode* node = q.front(); q.pop();
            sum += node->val;
            if (node->left  != nullptr) q.push(node->left);
            if (node->right != nullptr) q.push(node->right);
        }
        result.push_back((double) sum / size);
    }
    return result;
}
```

---

## 3. Find Largest Value in Each Row (LC 515) — max per level

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> largestValues(TreeNode* root) {
    vector<int> result;
    if (root == nullptr) return result;
    queue<TreeNode*> q;
    q.push(root);
    while (!q.empty()) {
        int size = q.size();
        int maxVal = INT_MIN;
        for (int i = 0; i < size; i++) {
            TreeNode* node = q.front(); q.pop();
            maxVal = max(maxVal, node->val);
            if (node->left  != nullptr) q.push(node->left);
            if (node->right != nullptr) q.push(node->right);
        }
        result.push_back(maxVal);
    }
    return result;
}
```

---

## 4. Level Order Traversal II (LC 107) — bottom-up

Same BFS; prepend each level so the result is bottom-to-top. Use `deque::push_front` (O(1))
or `std::reverse` at the end.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<vector<int>> levelOrderBottom(TreeNode* root) {
    deque<vector<int>> dq;
    if (root == nullptr) return {};
    queue<TreeNode*> q;
    q.push(root);
    while (!q.empty()) {
        int size = q.size();
        vector<int> level;
        for (int i = 0; i < size; i++) {
            TreeNode* node = q.front(); q.pop();
            level.push_back(node->val);
            if (node->left  != nullptr) q.push(node->left);
            if (node->right != nullptr) q.push(node->right);
        }
        dq.push_front(level);               // prepend -> bottom-up order, O(1)
    }
    return vector<vector<int>>(dq.begin(), dq.end());
}
```

---

## 5. N-ary Level Order (LC 429)

Children come from `node->children` instead of `left`/`right` — otherwise identical.

```cpp
#include <bits/stdc++.h>
using namespace std;

struct Node {
    int val;
    vector<Node*> children;
};

vector<vector<int>> levelOrder(Node* root) {
    vector<vector<int>> result;
    if (root == nullptr) return result;
    queue<Node*> q;
    q.push(root);
    while (!q.empty()) {
        int size = q.size();
        vector<int> level;
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

## 6. Populating Next Right Pointers II (LC 117) — non-perfect tree

For an arbitrary tree, children may be missing, so the perfect-tree trick (LC 116) breaks. The
**dummy-head** technique builds the next level's `next` chain as we walk the current level: a
sentinel `dummy` precedes the next level, and `tail` extends the chain. This is **O(1) extra
space** (no queue).

```cpp
#include <bits/stdc++.h>
using namespace std;

struct Node {
    int val;
    Node* left;
    Node* right;
    Node* next;
    Node() : val(0), left(nullptr), right(nullptr), next(nullptr) {}
};

Node* connect(Node* root) {
    Node* curr = root;
    while (curr != nullptr) {
        Node dummy;                      // sentinel head of the NEXT level
        Node* tail = &dummy;
        while (curr != nullptr) {        // walk current level via next pointers
            if (curr->left != nullptr)  { tail->next = curr->left;  tail = tail->next; }
            if (curr->right != nullptr) { tail->next = curr->right; tail = tail->next; }
            curr = curr->next;
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
