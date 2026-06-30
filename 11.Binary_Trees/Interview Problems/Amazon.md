> **Topic:** [Binary Trees](../README.md) · **Pattern N of M** · **Company 1 of 3**

# Amazon — Binary Tree Interview Problems

Amazon's bar-raiser interviews on binary trees reward candidates who can reason about **recursion contracts** precisely, prune work greedily, and design clean APIs. The three problems below are recurring Amazon favorites because each one hides a subtle "why does this work" that separates a memorized solution from a genuinely understood one.

Throughout, we use the standard node definition:

```java
public class TreeNode {
    int val;
    TreeNode left, right;
    TreeNode(int val) { this.val = val; }
}
```

---

## 1. Lowest Common Ancestor of a Binary Tree (LC 236)

### Problem Statement
Given a binary tree and two nodes `p` and `q` that are guaranteed to exist in the tree, return their **lowest common ancestor (LCA)** — the deepest node that has both `p` and `q` as descendants (a node is allowed to be a descendant of itself).

Unlike the BST variant, there are **no ordering guarantees**, so we cannot use value comparisons to navigate. We must search the structure directly.

### Intuition — The Recursion Contract
The elegance of this solution lives entirely in the **contract** of the recursive function. Define `lowestCommonAncestor(root, p, q)` to return:

- `null` if neither `p` nor `q` exists in the subtree rooted at `root`.
- A non-null node otherwise, with this precise meaning:
  - If `root`'s subtree contains **both** `p` and `q`, it returns the **LCA**.
  - If it contains only one of them, it returns **that node** (acting as a "found" signal bubbling up).

With this contract, the logic at each node becomes mechanical:

1. **Base case:** if `root == null`, return `null` (empty subtree contains nothing).
2. **Short-circuit:** if `root == p || root == q`, return `root`. We stop descending — if the *other* target is somewhere below, this node is necessarily the ancestor, so reporting "found here" is sufficient and correct.
3. **Recurse** into both children to get `left` and `right` results.
4. **Combine:**
   - If **both** `left != null` and `right != null`, then `p` and `q` were found in *different* subtrees, so the **current node is the LCA** — return `root`.
   - Otherwise return whichever side is non-null (propagating the single "found" signal upward), or `null` if neither found anything.

#### Why returning p/q early is correct
This is the crux Amazon interviewers probe. Suppose `root == p`. We return `root` *without even looking* at whether `q` is below. Is that safe?

Yes — because of how the parent combines results. Two cases:
- `q` lies in `p`'s subtree. Then the search for `q` from above will never produce a *second* non-null sibling result, because the entire branch containing `q` flows up through `p` (now reported as non-null). The first ancestor that sees non-null from **both** of its children is exactly the point where the path to `p` and the path to `q` diverge — which is the true LCA. When `q` is *inside* `p`'s subtree, that divergence is `p` itself, and `p` is correctly returned all the way up unopposed.
- `q` lies outside `p`'s subtree. Then somewhere above, one child returns the `p`-signal and the other returns the `q`-signal; that node sees **both sides non-null** and is declared LCA. Correct.

The key insight: **a node returns non-null iff its subtree contains at least one target. The first (lowest) node to receive non-null from BOTH sides is the LCA. A node equal to p or q short-circuits because it can never be "missed" — any deeper match still funnels through it.**

### Full Java Solution

```java
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        // Base case: empty subtree contains neither target.
        if (root == null) return null;

        // Short-circuit: if this node is one of the targets, report it.
        // Any deeper match necessarily passes through here, so this is the
        // ancestor candidate.
        if (root == p || root == q) return root;

        // Search both subtrees.
        TreeNode left = lowestCommonAncestor(root.left, p, q);
        TreeNode right = lowestCommonAncestor(root.right, p, q);

        // Both sides found a target -> p and q diverge here -> LCA.
        if (left != null && right != null) return root;

        // Otherwise propagate the single non-null "found" signal upward
        // (or null if neither side found anything).
        return left != null ? left : right;
    }
}
```

### Complexity

| Metric | Value | Reason |
| --- | --- | --- |
| Time | O(n) | Each node is visited at most once. |
| Space | O(h) | Recursion stack, where `h` is tree height (O(n) worst case for a skewed tree, O(log n) balanced). |

### Follow-ups Amazon may ask
- **Nodes not guaranteed to exist?** Track found-counts and only declare an LCA after verifying both targets were located.
- **With parent pointers?** Walk both nodes up to collect ancestor sets, or use the "two-pointer intersection" technique (like finding the intersection of two linked lists).

---

## 2. Binary Tree Maximum Path Sum (LC 124)

### Problem Statement
A **path** is any sequence of nodes connected by parent-child edges; each node appears at most once and the path need not pass through the root. Return the maximum sum of node values over all possible paths. Node values may be negative.

### Intuition — The Gain Function
Define a helper `gain(node)` that answers a deliberately restricted question:

> *"What is the maximum sum of a downward path that starts at `node` and goes straight down through at most one child?"*

This restriction matters: a value returned to the **parent** can only be extended in one direction (you can't enter `node` from the parent and then branch into *both* children — that would form a fork, not a single path the parent can append to). So the contribution a node offers upward is `node.val + max(leftGain, rightGain)`.

#### Why `gain = Math.max(0, gain)`
For each child we compute its gain, then clamp it: `leftGain = Math.max(0, gain(node.left))`. If a child's best downward path is negative, **including it can only hurt** — we are better off "cutting it off" and treating that direction as contributing 0. The clamp is a local greedy prune that is globally optimal because path sums are additive and a negative prefix never improves a sum.

#### Why the answer uses both children but the return uses only one
At each node we *also* consider a path that **bends** at `node` — entering from the left subtree, passing through `node`, and exiting into the right subtree:

```
priceNewPath = node.val + leftGain + rightGain
```

This is a candidate for the **global maximum**, because such a bent path is a legitimate path (it just can't be extended further up — its endpoints are in the two subtrees). We update a global best with it.

But we **return** `node.val + max(leftGain, rightGain)` to the parent, because the parent needs a path it can *attach* to. A path that already bent at `node` cannot continue upward without revisiting `node`. Hence:

- **Considered locally (for the answer):** `node.val + leftGain + rightGain` (a fork — both children).
- **Returned to parent (for extension):** `node.val + max(leftGain, rightGain)` (a straight line — one child).

This dual role is the entire trick. Interviewers love asking *why* the returned value differs from the value compared against the global max.

### Full Java Solution

```java
class Solution {
    // Instance var as global recursion state (initialized to the smallest int
    // so even an all-negative tree yields its largest single node).
    private int maxSum = Integer.MIN_VALUE;

    public int maxPathSum(TreeNode root) {
        gain(root);
        return maxSum;
    }

    // Returns the max gain of a straight downward path starting at `node`.
    private int gain(TreeNode node) {
        if (node == null) return 0;

        // Prune negative contributions: a child that hurts is treated as 0.
        int leftGain  = Math.max(0, gain(node.left));
        int rightGain = Math.max(0, gain(node.right));

        // Candidate path that BENDS at `node` (uses both children).
        int priceNewPath = node.val + leftGain + rightGain;
        maxSum = Math.max(maxSum, priceNewPath);

        // Return value the PARENT can extend: only one child allowed.
        return node.val + Math.max(leftGain, rightGain);
    }
}
```

### Complexity

| Metric | Value | Reason |
| --- | --- | --- |
| Time | O(n) | One post-order visit per node. |
| Space | O(h) | Recursion depth equals tree height. |

### Edge cases to verbalize
- Single node with negative value: answer is that value (handled because `maxSum` starts at `Integer.MIN_VALUE` and `priceNewPath = node.val + 0 + 0`).
- All-negative tree: the clamp ensures we never *add* a negative child, so the answer is the single least-negative node.

---

## 3. Serialize and Deserialize Binary Tree (LC 297)

### Problem Statement
Design an algorithm to encode a binary tree to a string and decode that string back to the **identical** tree. The exact format is your choice, as long as serialize/deserialize round-trip losslessly.

### Intuition — Preorder DFS with Null Markers
The reason a plain in-order or level-order *value-only* dump fails is that it loses structure: many distinct trees produce the same value sequence once nulls are dropped. The fix is to **record nulls explicitly**. A **preorder traversal (root, left, right) that emits a sentinel for every null pointer** uniquely determines the tree, because preorder fixes the root first and the null markers tell us exactly where each subtree ends.

- **Serialize:** preorder DFS; append `node.val` (or a null marker like `"#"`) followed by a delimiter.
- **Deserialize:** consume tokens left-to-right with the same preorder logic. The first token is the root; recursively build the left subtree, then the right. A null marker terminates a branch.

### Codec API Design and Tradeoffs
LeetCode supplies a `Codec` class with two methods. Design choices worth discussing:

- **Delimiter:** a comma is simple but assumes values don't contain commas. For arbitrary string payloads, length-prefixing is safer. For `int` values a comma/space is fine.
- **Null marker:** `"#"` (or `"null"`) — must be distinguishable from any legal value. With negative ints allowed, avoid using `"-1"` as the marker.
- **Preorder vs. level-order (BFS):** Preorder recursion is the cleanest to write and reason about. Level-order produces output resembling LeetCode's display format but needs explicit queue management and careful null handling. Preorder is preferred for clarity in interviews.
- **Stateful cursor:** during deserialization we walk the token list with a shared index/iterator. Using a `Queue<String>` (or `Deque`) as a consumable cursor avoids passing an index by reference.

### Full Java Solution

```java
public class Codec {

    private static final String NULL = "#";
    private static final String SEP = ",";

    // ---- Serialize: preorder DFS with null markers ----
    public String serialize(TreeNode root) {
        StringBuilder sb = new StringBuilder();
        buildString(root, sb);
        return sb.toString();
    }

    private void buildString(TreeNode node, StringBuilder sb) {
        if (node == null) {
            sb.append(NULL).append(SEP);
            return;
        }
        sb.append(node.val).append(SEP); // root
        buildString(node.left, sb);      // left
        buildString(node.right, sb);     // right
    }

    // ---- Deserialize: consume tokens in preorder ----
    public TreeNode deserialize(String data) {
        // A Deque used as a consumable cursor over the tokens.
        Deque<String> tokens = new ArrayDeque<>(Arrays.asList(data.split(SEP)));
        return buildTree(tokens);
    }

    private TreeNode buildTree(Deque<String> tokens) {
        String token = tokens.poll(); // advance the shared cursor
        if (token == null || token.equals(NULL)) {
            return null;
        }
        TreeNode node = new TreeNode(Integer.parseInt(token)); // root
        node.left  = buildTree(tokens);                        // left
        node.right = buildTree(tokens);                        // right
        return node;
    }
}

// Required imports: java.util.ArrayDeque, java.util.Arrays, java.util.Deque
```

### Complexity

| Operation | Time | Space |
| --- | --- | --- |
| serialize | O(n) | O(n) output string + O(h) recursion |
| deserialize | O(n) | O(n) token list + O(h) recursion |

### Why this is robust
Every node contributes exactly one value token, and every null child contributes exactly one marker — so the token stream length is `2n + 1`, and the preorder consumption order on deserialize mirrors the production order on serialize, guaranteeing a faithful reconstruction.

---

## Amazon Leadership Principles Alignment

Amazon interviews blend technical depth with behavioral signals. Here is how each problem maps to relevant Leadership Principles — useful framing when an interviewer asks you to reflect on your approach.

| Problem | Leadership Principle | How it shows up |
| --- | --- | --- |
| LCA (LC 236) | **Dive Deep** | Articulating the precise recursion contract — *why* the early p/q return is correct — demonstrates going beyond a memorized template to true mechanism-level understanding. |
| LCA (LC 236) | **Are Right, A Lot** | Reasoning through both "q inside p's subtree" and "q outside" cases shows rigorous case analysis rather than hand-waving. |
| Max Path Sum (LC 124) | **Insist on the Highest Standards** | Cleanly separating the *considered* value (fork through both children) from the *returned* value (single child) reflects refusing to ship a subtly-wrong solution. |
| Max Path Sum (LC 124) | **Invent and Simplify** | The `Math.max(0, gain)` prune is a simple, elegant device that collapses many edge cases into one rule. |
| Serialize/Deserialize (LC 297) | **Invent and Simplify** | Choosing preorder + null markers yields the simplest correct codec; discussing delimiter/marker tradeoffs shows pragmatic design. |
| Serialize/Deserialize (LC 297) | **Customer Obsession** | Treating the codec as an API (clear contract, lossless round-trip, robust to negatives) mirrors building reliable interfaces for downstream consumers. |
| All three | **Bias for Action** | Establishing the base case and contract first, then coding decisively, demonstrates structured speed under interview time pressure. |

---

> **Last Updated:** 2026-06-26
