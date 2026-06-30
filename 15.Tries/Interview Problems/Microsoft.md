> **Topic:** [Tries](../README.md) · **Interview Problems**

# Microsoft — Interview Problems (Tries)

Microsoft interviews emphasize complete, robust implementations — including the operations people skip, like a *physical* delete that reclaims memory. The three problems below cover the full Trie API, wildcard search, and the bitwise-Trie XOR pattern. References: [Trie Fundamentals](../Patterns/Trie%20Fundamentals.md), [Trie with Search Variants](../Patterns/Trie%20with%20Search%20Variants.md), [Bitwise Trie (XOR)](../Patterns/Bitwise%20Trie%20(XOR).md).

---

## (a) Implement Trie with Delete — LC 208 (extended)

**Problem.** Implement a `Trie` supporting `insert`, `search`, `startsWith`, and additionally a **physical** `delete(word)` that not only unmarks the word but also reclaims (detaches) nodes that become unnecessary.

**Design discussion.** There are two flavors of delete:

- **Logical delete:** just set `isEnd = false`. Simple, but leaves orphan paths consuming memory; over time a long-lived Trie accumulates dead nodes.
- **Physical delete:** unmark the terminal, then walk back up pruning any node that (1) has no children and (2) is not itself the end of another word. We must *not* prune a node that is a prefix of another stored word (e.g. deleting `"apple"` must keep nodes shared with `"app"`).

The clean way to express physical delete is a recursive function that returns "should my parent detach me?". A node is safe to detach when it has no remaining children and is not a word end. The `isEmpty(node)` helper checks "no children." Importantly we still recurse into the deepest node first, unmark `isEnd` there, and prune on the way back up only while the detach condition holds.

```java
class Trie {
    static class TrieNode {
        TrieNode[] children = new TrieNode[26];
        boolean isEnd;
    }

    private final TrieNode root;

    public Trie() {
        root = new TrieNode();
    }

    public void insert(String word) {
        TrieNode node = root;
        for (int i = 0; i < word.length(); i++) {
            int idx = word.charAt(i) - 'a';
            if (node.children[idx] == null) {
                node.children[idx] = new TrieNode();
            }
            node = node.children[idx];
        }
        node.isEnd = true;
    }

    public boolean search(String word) {
        TrieNode node = find(word);
        return node != null && node.isEnd;
    }

    public boolean startsWith(String prefix) {
        return find(prefix) != null;
    }

    public void delete(String word) {
        delete(root, word, 0);
    }

    /**
     * Recursively deletes `word` starting at depth `i` from `node`.
     * Returns true if `node` itself should be detached by its parent
     * (i.e. it has no children and is not the end of any word).
     */
    private boolean delete(TrieNode node, String word, int i) {
        if (node == null) {
            return false;
        }
        if (i == word.length()) {
            if (!node.isEnd) {
                return false;             // word not present; nothing to remove
            }
            node.isEnd = false;           // unmark terminal
            return isEmpty(node);         // detach only if it's now a dead leaf
        }

        int idx = word.charAt(i) - 'a';
        TrieNode child = node.children[idx];
        boolean shouldDetachChild = delete(child, word, i + 1);

        if (shouldDetachChild) {
            node.children[idx] = null;    // prune the now-useless child
        }
        // this node may itself be detached if it is now an empty non-word
        return isEmpty(node) && !node.isEnd;
    }

    private boolean isEmpty(TrieNode node) {
        for (TrieNode child : node.children) {
            if (child != null) {
                return false;
            }
        }
        return true;
    }

    private TrieNode find(String s) {
        TrieNode node = root;
        for (int i = 0; i < s.length(); i++) {
            int idx = s.charAt(i) - 'a';
            if (node.children[idx] == null) {
                return null;
            }
            node = node.children[idx];
        }
        return node;
    }
}
```

**Reasoning.** The recursion descends to the terminal node, clears `isEnd`, then unwinds. At each level we detach the child only if the child reported it is a dead leaf, and we report ourselves detachable only if we have become an empty non-word node. This correctly preserves shared prefixes: when deleting `"apple"` while `"app"` is stored, recursion at the `"app"` node finds `isEnd == true` (it is `"app"`'s terminal) so `isEmpty(node) && !node.isEnd` is false and pruning stops there. The root is never detached because the public `delete` ignores the top-level return value.

**Complexity.** `insert`, `search`, `startsWith` are `O(L)`. `delete` is `O(L)` time (one descent, one unwind) and `O(L)` recursion stack. Memory is actively reclaimed, unlike logical delete.

**Follow-ups.**
- *Reference counting:* if duplicate inserts are allowed, store a count instead of a boolean and only unmark when the count hits zero.
- *Iterative delete:* push visited nodes on a stack and prune in reverse to avoid recursion depth limits for very long words.

---

## (b) Add and Search Word / Word Dictionary — LC 211

**Problem.** Design `WordDictionary` with `addWord(word)` and `search(word)`, where `'.'` in a search matches any single letter.

**DFS recursion for the `'.'` wildcard.** `addWord` is a standard Trie insert. `search` is recursive over `(node, index)`:
- If we have consumed the whole query (`index == word.length()`), the answer is whether the current node is a word terminal (`node.isEnd`).
- For a normal character, descend to exactly one child (`node.children[c - 'a']`) and recurse; if that child is null the match fails on this branch.
- For `'.'`, we cannot know which letter it stands for, so we try **every** non-null child and succeed if any recursive branch succeeds. This is where the search fans out — a single `'.'` turns one path into up to 26.

```java
class WordDictionary {
    static class TrieNode {
        TrieNode[] children = new TrieNode[26];
        boolean isEnd;
    }

    private final TrieNode root;

    public WordDictionary() {
        root = new TrieNode();
    }

    public void addWord(String word) {
        TrieNode node = root;
        for (int i = 0; i < word.length(); i++) {
            int idx = word.charAt(i) - 'a';
            if (node.children[idx] == null) {
                node.children[idx] = new TrieNode();
            }
            node = node.children[idx];
        }
        node.isEnd = true;
    }

    public boolean search(String word) {
        return dfs(word, 0, root);
    }

    private boolean dfs(String word, int i, TrieNode node) {
        if (node == null) {
            return false;
        }
        if (i == word.length()) {
            return node.isEnd;
        }
        char c = word.charAt(i);
        if (c == '.') {
            for (TrieNode child : node.children) {
                if (child != null && dfs(word, i + 1, child)) {
                    return true;          // any matching branch suffices
                }
            }
            return false;
        }
        return dfs(word, i + 1, node.children[c - 'a']);
    }
}
```

**Reasoning.** The base case `i == word.length()` returning `node.isEnd` is what enforces *exact length* matching — reaching a node is not enough; it must terminate a stored word. The `'.'` branch short-circuits (`return true`) on the first success, so we do not explore unnecessary siblings once a match is found.

**Complexity.** `addWord` is `O(L)`. `search` is `O(L)` when there are no wildcards; with `k` dots it is worst case `O(26^k · L)` because each dot branches into up to 26 recursive paths. Space is `O(sum of word lengths)` for the Trie plus `O(L)` recursion depth.

**Follow-ups.**
- *Frequent all-dot queries:* bucket words by length to shrink the search space.
- *Wildcard `'*'` (zero or more):* requires a different recurrence allowing the index to stay or advance.

---

## (c) Maximum XOR of Two Numbers in an Array — LC 421

**Problem.** Given `int[] nums`, find the maximum `nums[i] ^ nums[j]`.

**Bitwise Trie construction, step by step.** We build a binary Trie where every node has exactly **2 children**, indexed by bit value `0` or `1`. The construction:

1. Fix a bit width. Values fit in 31 bits, so we iterate from the most significant bit `HIGH_BIT = 30` down to `0`. Processing MSB-first is essential — the XOR is maximized greedily from the highest bit.
2. For each number, start at the root. At bit position `i`, extract `bit = (num >> i) & 1`.
3. If `node.children[bit]` is null, create it. Move down: `node = node.children[bit]`.
4. After the lowest bit, the path from root to leaf encodes the number's bit pattern (MSB at depth 1).

So each root-to-leaf path is a number's binary representation. Numbers sharing a high-bit prefix share Trie nodes near the root.

To answer a query for a number `num`, walk the same way but at each bit prefer the **opposite** bit (`want = bit ^ 1`): if such a child exists, that bit of the XOR is `1` (set it with `xor |= (1 << i)`); otherwise follow the same bit.

```java
class Solution {
    static final int HIGH_BIT = 30;

    static class TrieNode {
        TrieNode[] children = new TrieNode[2];
    }

    public int findMaximumXOR(int[] nums) {
        TrieNode root = new TrieNode();
        for (int num : nums) {
            insert(root, num);
        }

        int max = 0;
        for (int num : nums) {
            max = Math.max(max, maxXorWith(root, num));
        }
        return max;
    }

    private void insert(TrieNode root, int num) {
        TrieNode node = root;
        for (int i = HIGH_BIT; i >= 0; i--) {
            int bit = (num >> i) & 1;          // MSB-first
            if (node.children[bit] == null) {
                node.children[bit] = new TrieNode();
            }
            node = node.children[bit];
        }
    }

    private int maxXorWith(TrieNode root, int num) {
        TrieNode node = root;
        int xor = 0;
        for (int i = HIGH_BIT; i >= 0; i--) {
            int bit = (num >> i) & 1;
            int want = bit ^ 1;                // opposite bit maximizes this position
            if (node.children[want] != null) {
                xor |= (1 << i);
                node = node.children[want];
            } else {
                node = node.children[bit];
            }
        }
        return xor;
    }
}
```

**Reasoning.** Building the Trie once and querying every number against it converts the naive `O(n^2)` all-pairs scan into `O(n · B)`. The greedy works because a higher set bit in the XOR dominates all lower bits combined, so taking the opposite bit whenever possible is always optimal.

**Complexity.** With `B = 31` bits and `n` numbers: construction `O(n · B)`, all queries `O(n · B)`, total `O(n · B)` time and `O(n · B)` space. See the cross-cutting pattern in [Bitwise Trie (XOR)](../Patterns/Bitwise%20Trie%20(XOR).md).

**Follow-ups.**
- *Stream of numbers:* the Trie supports incremental inserts, so you can maintain a running max XOR online.
- *Delete support:* add per-node counts so numbers can be removed (mirrors the count-based delete from part (a)).
- *Constrained variant (LC 1707):* answer "max XOR with `nums[j] <= m`" by inserting in sorted order and processing queries offline.

---

> **Last Updated:** 2026-06-26
