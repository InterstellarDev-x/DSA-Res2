> **Topic:** [Binary Trees](../README.md) · **Design 1 of 1**

# Serialize and Deserialize Binary Tree (LC 297)

Design a `Codec` that converts a binary tree to a string and back. Any encoding is acceptable
as long as `deserialize(serialize(root))` reproduces the original tree. The two standard
encodings are **BFS (level-order)** and **DFS (preorder)** — both must explicitly mark `null`
children so structure is recoverable.

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

> **Why null markers matter:** values alone are ambiguous. `[1, 2]` could mean 2 is the left
> or the right child of 1. Recording `null` for absent children removes the ambiguity, so a
> single traversal order fully determines the tree.

---

## Approach 1 — DFS Preorder (recommended)

Write nodes in preorder (Root, Left, Right), emitting `"#"` for `nullptr`. To rebuild, consume
tokens in the same preorder using an index pointer (or a `queue`/iterator).

```cpp
#include <bits/stdc++.h>
using namespace std;

class Codec {
    const string NULLMARK = "#";
    const string SEP = ",";

    // ---- serialize ----
    void buildString(TreeNode* node, string& sb) {
        if (node == nullptr) {
            sb += NULLMARK + SEP;
            return;
        }
        sb += to_string(node->val) + SEP;
        buildString(node->left, sb);     // preorder: left then right
        buildString(node->right, sb);
    }

public:
    string serialize(TreeNode* root) {
        string sb;
        buildString(root, sb);
        return sb;
    }

    // ---- deserialize ----
    TreeNode* buildTree(queue<string>& tokens) {
        string val = tokens.front(); tokens.pop();  // consume in the SAME preorder
        if (val == NULLMARK) return nullptr;
        TreeNode* node = new TreeNode(stoi(val));
        node->left  = buildTree(tokens);
        node->right = buildTree(tokens);
        return node;
    }

    TreeNode* deserialize(string data) {
        queue<string> tokens;
        stringstream ss(data);
        string token;
        while (getline(ss, token, ',')) tokens.push(token);
        return buildTree(tokens);
    }
};
```

**Why it works:** preorder fixes the order in which nodes (and null markers) are emitted.
Deserialization mirrors that exact order, so each recursive call consumes precisely the tokens
that belong to its subtree. The `queue` acts as a shared, advancing cursor across all
recursive frames.

---

## Approach 2 — BFS Level Order

Serialize level by level, writing `"#"` for null children encountered in the queue.
Deserialize by reading the root, then for each non-null node, attach the next two tokens as
its children.

```cpp
#include <bits/stdc++.h>
using namespace std;

class CodecBFS {
    const string NULLMARK = "#";
    const string SEP = ",";

public:
    string serialize(TreeNode* root) {
        if (root == nullptr) return "";
        string sb;
        queue<TreeNode*> q;
        q.push(root);
        while (!q.empty()) {
            TreeNode* node = q.front(); q.pop();
            if (node == nullptr) {
                sb += NULLMARK + SEP;
            } else {
                sb += to_string(node->val) + SEP;
                q.push(node->left);     // enqueue nulls too; they'll be written as #
                q.push(node->right);
            }
        }
        return sb;
    }

    TreeNode* deserialize(string data) {
        if (data.empty()) return nullptr;
        vector<string> tokens;
        stringstream ss(data);
        string token;
        while (getline(ss, token, ',')) tokens.push_back(token);
        TreeNode* root = new TreeNode(stoi(tokens[0]));
        queue<TreeNode*> q;
        q.push(root);
        int i = 1;
        while (!q.empty()) {
            TreeNode* node = q.front(); q.pop();
            if (tokens[i] != NULLMARK) {           // left child
                node->left = new TreeNode(stoi(tokens[i]));
                q.push(node->left);
            }
            i++;
            if (tokens[i] != NULLMARK) {           // right child
                node->right = new TreeNode(stoi(tokens[i]));
                q.push(node->right);
            }
            i++;
        }
        return root;
    }
};
```

---

## Follow-ups

### Codec for a BST (LC 449) — no null markers needed
A BST's structure is recoverable from preorder alone (values are unique and ordered), so you
can drop the `"#"` markers and rebuild using an `upperBound` to decide where each value
belongs.

```cpp
#include <bits/stdc++.h>
using namespace std;

TreeNode* deserializeBST(queue<int>& q, int lower, int upper) {
    if (q.empty()) return nullptr;
    int val = q.front();
    if (val < lower || val > upper) return nullptr;     // not in this subtree's range
    q.pop();
    TreeNode* node = new TreeNode(val);
    node->left  = deserializeBST(q, lower, val);
    node->right = deserializeBST(q, val, upper);
    return node;
}
```
This is more compact (no null tokens) and is the reason serializing a **BST** is cheaper than
a general binary tree.

### Codec for an N-ary tree (LC 428)
Emit each node's value followed by its **child count**, then recurse over children. The count
tells deserialization how many children to read — no per-child null markers required.

```cpp
// serialize: val + "," + children.size() + "," + ...children...
void encode(Node* node, string& sb) {
    sb += to_string(node->val) + "," + to_string(node->children.size()) + ",";
    for (auto& child : node->children) encode(child, sb);
}
```

---

## Complexity

| Approach | serialize Time | deserialize Time | Space |
|---|---|---|---|
| DFS Preorder | O(n) | O(n) | O(n) string + O(h) stack |
| BFS Level Order | O(n) | O(n) | O(n) string + O(w) queue |
| BST (no markers) | O(n) | O(n) | O(n) + O(h) |
| N-ary (child count) | O(n) | O(n) | O(n) + O(h) |

Both core approaches visit every node a constant number of times → **O(n)** time, and the
serialized string is **O(n)**. DFS uses recursion stack O(h); BFS uses a queue O(w).

---

## API Design Notes
- Keep `serialize`/`deserialize` **symmetric**: the same traversal order on both sides is what
  guarantees correctness.
- Choose a **separator** that can't appear in a value (here `","`); for arbitrary string data,
  length-prefix each token instead.
- Make the codec **stateless** between calls (no leftover index/queue), so one instance can be
  reused safely.

---

### Related
- [Tree Traversals](../Patterns/Tree%20Traversals.md) — the DFS/BFS orders this builds on
- [Tree Construction](../Patterns/Tree%20Construction.md) — rebuilding trees from sequences

> **Last Updated:** 2026-06-26
