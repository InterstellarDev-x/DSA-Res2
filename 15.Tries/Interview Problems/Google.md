> **Topic:** [Tries](../README.md) · **Interview Problems**

# Google — Interview Problems (Tries)

Google Trie questions reward depth: not "can you build a Trie" but "can you reason about *why* a shared structure beats brute force, and analyze the wildcard / bit-search complexity precisely." The three problems below are staples in L4/L5 loops. Background: [Trie Fundamentals](../Patterns/Trie%20Fundamentals.md), [Trie with Search Variants](../Patterns/Trie%20with%20Search%20Variants.md), [Bitwise Trie (XOR)](../Patterns/Bitwise%20Trie%20(XOR).md).

---

## (a) Word Search II — LC 212

**Problem.** Given an `m x n` board and a list of `words`, return every word that can be formed by a path of adjacent (up/down/left/right) cells with no cell reused within a word.

**The full optimization story.** A first instinct is: for each word, run a board DFS. That is `O(W · M·N · 4^L)` — the number of words `W` multiplies everything. The insight Google is probing for is that **the words share prefixes**, and a brute-force per-word search redoes that shared prefix exploration `W` times. Collapsing all words into **one Trie** and walking the board *once* lets the shared prefixes be explored exactly once.

Three concrete optimizations make this efficient:

1. **Store the word at the Trie node.** Instead of `bool isEnd`, store `string word` at terminal nodes. When the board DFS reaches a node whose `!node->word.empty()`, we have a full match and can collect it without reconstructing the path from the board.
2. **Clear the word after it is found** (`node->word = ""`). This deduplicates (the same word reachable via two paths is reported once) and signals the word is consumed.
3. **Remove/prune dead leaves.** After exploring a node, if it has no children left and is not itself a word, it is a dead end; detaching it from its parent shrinks the Trie so future board paths bail out earlier. The `dfs` below carries the parent so it can prune.

Why **one shared Trie beats per-word search**: the per-word approach pays `4^L` *per word*; the shared Trie pays the board-exploration cost once and prunes branches the instant no word continues with the current prefix. The dominant term drops from `O(W · M·N · 4^L)` to `O(M·N · 4^Lmax)`.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
    struct TrieNode {
        TrieNode* children[26] = {};
        string word;   // full word stored at terminal node, else empty
    };

    TrieNode* buildTrie(vector<string>& words) {
        TrieNode* root = new TrieNode();
        for (auto& w : words) {
            TrieNode* node = root;
            for (char ch : w) {
                int idx = ch - 'a';
                if (!node->children[idx]) {
                    node->children[idx] = new TrieNode();
                }
                node = node->children[idx];
            }
            node->word = w;
        }
        return root;
    }

    bool isEmpty(TrieNode* node) {
        for (TrieNode* child : node->children) {
            if (child) return false;
        }
        return true;
    }

    void dfs(vector<vector<char>>& board, int r, int c, TrieNode* parent, vector<string>& result) {
        int m = board.size(), n = board[0].size();
        if (r < 0 || r >= m || c < 0 || c >= n) return;
        char ch = board[r][c];
        if (ch == '#') return;                              // on the current path already
        int idx = ch - 'a';
        TrieNode* node = parent->children[idx];
        if (!node) return;                              // no word continues -> prune
        if (!node->word.empty()) {
            result.push_back(node->word);
            node->word = "";                    // dedup
        }

        board[r][c] = '#';
        dfs(board, r + 1, c, node, result);
        dfs(board, r - 1, c, node, result);
        dfs(board, r, c + 1, node, result);
        dfs(board, r, c - 1, node, result);
        board[r][c] = ch;

        // prune dead leaf: no children left and not a word -> detach
        if (isEmpty(node) && node->word.empty()) {
            parent->children[idx] = nullptr;
        }
    }

public:
    vector<string> findWords(vector<vector<char>>& board, vector<string>& words) {
        TrieNode* root = buildTrie(words);
        vector<string> result;
        int m = board.size(), n = board[0].size();
        for (int r = 0; r < m; r++) {
            for (int c = 0; c < n; c++) {
                dfs(board, r, c, root, result);
            }
        }
        return result;
    }
};
```

**Complexity.** Building the Trie is `O(sum of word lengths)`. The search is `O(M·N · 4^Lmax)` time, where `Lmax` is the longest word, because each cell seeds a DFS bounded by the Trie depth and pruned aggressively. Space is `O(sum of word lengths)` for the Trie, shrinking as leaves are pruned.

**Follow-ups.**
- *Return paths, not just words:* track `(r,c)` on a path list.
- *Words can reuse cells:* drop the `'#'` marking — but then termination requires bounding by max word length.

---

## (b) Add and Search Word / Word Dictionary — LC 211

**Problem.** Design `WordDictionary` supporting `addWord(word)` and `search(word)`, where `search` may contain `'.'` as a wildcard that matches any single letter.

**Wildcard DFS.** A literal character walks deterministically to one child. A `'.'` must try **all 26** children, recursing into each — that branching is what makes wildcards expensive. We recurse from the current Trie node and string index; on `'.'` we loop over every non-null child.

```cpp
#include <bits/stdc++.h>
using namespace std;

class WordDictionary {
    struct TrieNode {
        TrieNode* children[26] = {};
        bool isEnd = false;
    };

    TrieNode* root;

    bool dfs(const string& word, int i, TrieNode* node) {
        if (!node) return false;
        if (i == (int)word.size()) return node->isEnd;
        char c = word[i];
        if (c == '.') {
            for (TrieNode* child : node->children) {
                if (child && dfs(word, i + 1, child)) return true;
            }
            return false;
        }
        return dfs(word, i + 1, node->children[c - 'a']);
    }

public:
    WordDictionary() {
        root = new TrieNode();
    }

    void addWord(const string& word) {
        TrieNode* node = root;
        for (char ch : word) {
            int idx = ch - 'a';
            if (!node->children[idx]) {
                node->children[idx] = new TrieNode();
            }
            node = node->children[idx];
        }
        node->isEnd = true;
    }

    bool search(const string& word) {
        return dfs(word, 0, root);
    }
};
```

**Wildcard complexity.** `addWord` is `O(L)`. For `search`, let `L` be the query length and `k` the number of `'.'` characters. Each `'.'` branches into up to 26 recursive calls, and the wildcards compound multiplicatively, giving a worst case of **`O(26^k · L)`**. Intuition: a query like `".....`" (all dots) forces exploration of every length-`L` path in the Trie, of which there can be up to `26^L`; with `k` dots the branching factor is `26` raised to the number of wildcards, and the `L` factor accounts for walking the depth. With no dots (`k = 0`) it collapses to `O(L)`.

**Follow-ups.**
- *Bound the wildcard cost:* if many queries are all-dots, group words by length and store per-length Tries.
- *Leading wildcards / suffix queries:* add a reversed Trie or a suffix structure.

---

## (c) Maximum XOR of Two Numbers in an Array — LC 421

**Problem.** Given `vector<int>& nums`, return the maximum value of `nums[i] ^ nums[j]`.

**Binary (bitwise) Trie + greedy.** Build a Trie where each node has exactly **2 children** (bit `0` and bit `1`), inserting each number from its most significant bit down. To maximize XOR for a given number, walk the Trie from the MSB and at each bit **greedily prefer the opposite bit** (`want = bit ^ 1`): if a number with the opposite bit exists down this path, taking it sets that XOR bit to 1, which is worth more than every lower bit combined. Constraints fit in 31 bits, so we start at `HIGH_BIT = 30`.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
    static const int HIGH_BIT = 30;

    struct TrieNode {
        TrieNode* children[2] = {};
    };

    void insert(TrieNode* root, int num) {
        TrieNode* node = root;
        for (int i = HIGH_BIT; i >= 0; i--) {
            int bit = (num >> i) & 1;
            if (!node->children[bit]) {
                node->children[bit] = new TrieNode();
            }
            node = node->children[bit];
        }
    }

    int maxXorWith(TrieNode* root, int num) {
        TrieNode* node = root;
        int xorVal = 0;
        for (int i = HIGH_BIT; i >= 0; i--) {
            int bit = (num >> i) & 1;
            int want = bit ^ 1;                 // prefer the opposite bit
            if (node->children[want]) {
                xorVal |= (1 << i);                // this bit of the XOR becomes 1
                node = node->children[want];
            } else {
                node = node->children[bit];      // forced to take the same bit
            }
        }
        return xorVal;
    }

public:
    int findMaximumXOR(vector<int>& nums) {
        TrieNode* root = new TrieNode();
        for (int num : nums) {
            insert(root, num);
        }

        int maxVal = 0;
        for (int num : nums) {
            maxVal = max(maxVal, maxXorWith(root, num));
        }
        return maxVal;
    }
};
```

**Why greedy is correct.** XOR is maximized lexicographically by bit significance: a 1 in a higher bit outweighs any combination of lower bits (`2^i > 2^i - 1 = sum of all lower bits`). So at each level, if we *can* make the current XOR bit 1 by choosing a stored number with the opposite bit, we must — no future choice could compensate for giving up a higher bit. The Trie lets us check "does a number with the opposite bit exist on this prefix path?" in `O(1)` per level, and the greedy choice is always locally and globally optimal.

**Complexity.** Let `B = 31` be the bit width and `n = nums.size()`. Insert is `O(B)` per number, search is `O(B)` per number, so total `O(n · B)` time and `O(n · B)` space for the Trie. This beats the naive `O(n^2)` pairwise comparison.

Cross-link: this is the same machinery used in [Bit Search and Trie](../../6.Bit_Manipulation/Patterns/Bit%20Search%20and%20Trie.md).

**Follow-ups.**
- *Maximum XOR with a query upper bound (`nums[j] <= limit`)* — LC 1707; add insertion-ordering / offline sorting.
- *Maximum XOR in a subarray range* — persistent / segment-indexed bit Tries.
- *There is also an `O(n · B)` `unordered_map`-prefix solution* that avoids building an explicit Trie; be ready to compare.

---

## L4 vs L5 expectations

- **L4 (mid):** Get to a correct, clean Trie solution. For 212, recognize the shared-Trie idea and implement `node->word` + mark-visited correctly. For 211, implement the wildcard DFS without bugs. For 421, derive the greedy bit-Trie and explain the MSB-first reasoning. State the complexity correctly.
- **L5 (senior):** Drive the *trade-off* conversation unprompted. For 212, propose and implement leaf pruning, discuss memory reclamation and how `Lmax` (not `W`) bounds the exponent. For 211, quantify `O(26^k · L)` and propose mitigations (per-length Tries) for skewed wildcard workloads. For 421, compare the Trie vs `unordered_map`-prefix approaches, prove greedy optimality from the bit-significance argument, and discuss the constrained/range variants. L5 is judged on communicating *why*, edge-case rigor, and scaling beyond the literal prompt.

---

> **Last Updated:** 2026-06-26
