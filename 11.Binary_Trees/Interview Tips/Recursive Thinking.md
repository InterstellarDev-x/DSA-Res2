> **Topic:** [Binary Trees](../README.md) · **Pattern 3 of 4**

# Binary Trees — Recursive Thinking

Binary trees are *the* canonical place to build recursive intuition: a tree is literally defined recursively (a node, a left subtree, a right subtree), so almost every problem has a clean recursive solution. This page builds the mental model you need to write that recursion correctly on the first try.

Node definition used throughout:

```cpp
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int val) : val(val), left(nullptr), right(nullptr) {}
};
```

---

## The mental model: the recursion contract

Before writing a single line, answer two questions about your recursive method:

1. **What does the function RETURN?** Define precisely what value comes back for a given subtree. "Returns the height of the subtree rooted here." "Returns the LCA of `p` and `q` if both live below here, else whichever one does, else `nullptr`." "Returns the best downward path gain starting at this node."
2. **What does it PROMISE its callers?** This is the *contract*. If `height(node)` promises "the number of nodes on the longest root-to-leaf path of this subtree," then every caller can rely on that, and you only have to make the promise true for one node — assuming the children already keep theirs.

If you can state the contract in one sentence, the body almost writes itself: handle the empty subtree (base case), call yourself on the children (trusting the contract), and combine. If you *can't* state it, you don't understand the problem yet — that's the signal to slow down.

---

## Top-down vs bottom-up recursion

There are two fundamentally different ways information flows through tree recursion.

### Bottom-up — compute results up via the return value

Each node computes its answer **from its children's answers**. Information flows from the leaves upward. This is the natural fit for "size/height/sum of this subtree" style questions.

```cpp
// Bottom-up maxDepth: each call RETURNS the height of its subtree.
int maxDepthBottomUp(TreeNode* node) {
    if (node == nullptr) return 0;
    int left = maxDepthBottomUp(node->left);
    int right = maxDepthBottomUp(node->right);
    return 1 + max(left, right);          // combine children's results
}
```

### Top-down — pass state down via parameters

Each node receives **context from above** through method parameters, and "answers" are recorded as it descends (often into a global, or by reaching a leaf). Information flows from the root downward.

```cpp
// Top-down maxDepth: the current depth is PASSED DOWN as a parameter.
int best = 0;

void maxDepthTopDown(TreeNode* node, int depth) {
    if (node == nullptr) return;
    if (node->left == nullptr && node->right == nullptr)
        best = max(best, depth);          // record at the leaf
    maxDepthTopDown(node->left,  depth + 1);    // hand context to children
    maxDepthTopDown(node->right, depth + 1);
}
// call as: maxDepthTopDown(root, 1);
```

Same problem, two flows. Rule of thumb: if a node's answer **depends on its descendants**, go bottom-up (return values). If it depends on its **ancestors / the path so far** (accumulated sum, current depth, the path list), go top-down (parameters). Many problems — Path Sum, Sum Root to Leaf, Binary Tree Paths — are naturally top-down because they care about the path *from the root*.

---

## Global state vs return value — and when you need both

Some problems can't be expressed with a return value alone. **Diameter** and **Max Path Sum** are the textbook cases: the value a node hands its parent is *not* the same as the answer being tracked.

- The **return value** is what an ancestor can legally extend (a single downward chain: a height, or a one-branch gain).
- The **global** captures answers that "bend" at a node and therefore can't be passed upward (a path using *both* children).

```cpp
int best = 0;   // mutable global state

int height(TreeNode* node) {
    if (node == nullptr) return 0;
    int left = height(node->left);
    int right = height(node->right);
    best = max(best, left + right); // GLOBAL: path bending here
    return 1 + max(left, right);          // RETURN: extendable height
}
```

Use a reference parameter or a global/member variable so the value survives across recursive frames. A local `int` would reset on every call. When the return value alone fully answers the question (height, count, sum), skip the global — it's just noise.

---

## "Trust the recursion" mindset

The hardest mental hurdle is resisting the urge to trace the recursion all the way down. Don't. **Assume the recursive call is already correct for the subtrees**, and focus only on combining those (assumed-correct) results for the current node.

When you write `int left = height(node->left);`, treat `left` as a finished, trustworthy fact — *the height of the left subtree* — exactly as the contract promises. Your only job is to make the contract true for *this* node given correct children. Induction does the rest: base case is correct, the combine step preserves correctness, therefore the whole thing is correct.

```cpp
bool isSameTree(TreeNode* a, TreeNode* b) {
    if (a == nullptr && b == nullptr) return true;        // base case
    if (a == nullptr || b == nullptr) return false;
    // TRUST that the two recursive calls correctly compare the subtrees:
    return a->val == b->val
        && isSameTree(a->left,  b->left)
        && isSameTree(a->right, b->right);
}
```

If you find yourself mentally unrolling three levels deep, stop — that's a sign you don't trust the contract yet, not that the code is wrong.

---

## When iterative is strictly needed

Recursion is elegant but not always allowed or safe:

- **Stack overflow on deep / skewed trees.** Recursion depth equals tree height. A degenerate (linked-list-shaped) tree of, say, 100,000 nodes recurses 100,000 frames deep and blows the call stack. Production code over untrusted input often must use an explicit stack instead.
- **Explicit O(1)-space requirements.** If the interviewer asks for constant extra space inorder traversal, recursion (O(h) stack) is disqualified — you need **Morris traversal**, which threads the tree instead of using a stack.
- **The interviewer explicitly asks for non-recursive / iterative.** A common follow-up after you give the recursive answer.

Here is the iterative inorder with an explicit stack — the standard "convert recursion to a stack" pattern:

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> inorderIterative(TreeNode* root) {
    vector<int> out;
    stack<TreeNode*> stk;
    TreeNode* curr = root;
    while (curr != nullptr || !stk.empty()) {
        while (curr != nullptr) {            // go all the way left
            stk.push(curr);
            curr = curr->left;
        }
        curr = stk.top(); stk.pop();               // visit
        out.push_back(curr->val);
        curr = curr->right;                // then go right
    }
    return out;
}
```

The explicit `stack<TreeNode*> stk` mirrors exactly what the call stack was doing for you. It has the same O(h) space, but you control it (and it won't overflow the call stack on a skewed tree because heap space is far larger). For truly O(1) space, fall back to Morris traversal.

---

## Putting it together

A reliable recipe for any tree recursion:

1. State the contract in one sentence (what it returns / promises).
2. Decide the information flow — bottom-up (return values) or top-down (parameters), or both with a global.
3. Write the base case first (`if (node == nullptr) ...`).
4. Recurse on `left` and `right`, **trusting** they are correct.
5. Combine, update any global, and return per the contract.
6. Ask: could this overflow on a skewed tree, or is O(1) space required? If so, convert to an explicit stack or Morris.

---

> **Last Updated:** 2026-06-26
