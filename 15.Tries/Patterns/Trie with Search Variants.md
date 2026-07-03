> **Topic:** [Tries](../README.md) · **Pattern 2 of 3**

# Trie with Search Variants

## Core Concept

Once you can build a trie ([Trie Fundamentals](Trie%20Fundamentals.md)), the interesting problems layer a **search strategy** on top of it:

- **Wildcards** (`.` matches any letter) → DFS that branches into *all* children when it hits a `.`.
- **Build-one-char-at-a-time** (Longest Word) → DFS that only descends through nodes whose every prefix is itself a complete word.
- **Root replacement** (Replace Words) → walk each query word until you hit the first `isEnd` (shortest root).
- **Board search** (Word Search II) → build a trie of the target words, then DFS the grid, pruning the trie as words are found.

The unifying idea: the trie is a **shared index**, and DFS/recursion explores it far more cheaply than running each query independently.

**Recognition signals:** wildcard matching, "find all words on a board", autocomplete with patterns, replacing words by shortest root, building words incrementally.

---

## Problem 3 — Add and Search Word (Word Dictionary, `.` wildcard) · LC 211 · Medium

`addWord` is a normal insert. `search` must handle `.` which matches **any** single character. When the DFS encounters `.`, it recurses into every non-null child.

```cpp
#include <bits/stdc++.h>
using namespace std;

struct TrieNode {
    TrieNode* children[26];
    bool isEnd = false;
    TrieNode() { fill(children, children + 26, nullptr); }
};

class WordDictionary {
private:
    TrieNode* root;

    bool dfs(const string& word, int i, TrieNode* node) {
        if (node == nullptr) return false;
        if (i == (int)word.length()) return node->isEnd;   // matched all chars

        char c = word[i];
        if (c == '.') {
            // wildcard: try every existing child
            for (TrieNode* child : node->children) {
                if (child != nullptr && dfs(word, i + 1, child)) {
                    return true;
                }
            }
            return false;
        } else {
            return dfs(word, i + 1, node->children[c - 'a']);
        }
    }

public:
    WordDictionary() : root(new TrieNode()) {}

    void addWord(const string& word) {
        TrieNode* node = root;
        for (char ch : word) {
            int idx = ch - 'a';
            if (node->children[idx] == nullptr) {
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

**Complexity:** `addWord` is **O(L)**. `search` is **O(L)** for a concrete word, but **O(26^k · L)** in the worst case where `k` is the number of `.` characters — each dot can branch into 26 subtrees. Full analysis in [Word Dictionary.md](../Design%20Data%20Structure%20Problems/Word%20Dictionary.md).

---

## Problem 4 — Longest Word in Dictionary · LC 720 · Medium

Find the longest word that can be **built one character at a time**, where every intermediate prefix is also a word in the dictionary. Ties broken by lexicographically smallest.

Insert all words, then DFS: only descend into a child if that child `isEnd` (its prefix is a valid buildable word). Because we iterate children `a → z`, the first/longest valid word we record under lexicographic tie-breaking is correct.

```cpp
#include <bits/stdc++.h>
using namespace std;

struct TrieNode {
    TrieNode* children[26];
    bool isEnd = false;
    TrieNode() { fill(children, children + 26, nullptr); }
};

class Solution {
private:
    string best = "";

    void insert(TrieNode* root, const string& w) {
        TrieNode* node = root;
        for (char ch : w) {
            int idx = ch - 'a';
            if (node->children[idx] == nullptr) node->children[idx] = new TrieNode();
            node = node->children[idx];
        }
        node->isEnd = true;
    }

    void dfs(TrieNode* node, string& path) {
        // Update best: longer wins; on equal length, smaller lexicographic wins.
        if (path.length() > best.length()
            || (path.length() == best.length() && path < best)) {
            best = path;
        }
        // Iterate a..z so ties naturally favour the smaller letter.
        for (int c = 0; c < 26; c++) {
            TrieNode* child = node->children[c];
            // Only continue if this prefix is itself a complete word.
            if (child != nullptr && child->isEnd) {
                path += (char)('a' + c);
                dfs(child, path);
                path.pop_back();   // backtrack
            }
        }
    }

public:
    string longestWord(vector<string>& words) {
        TrieNode* root = new TrieNode();
        for (auto& w : words) insert(root, w);
        // Root's prefix "" is trivially buildable, so start DFS from root.
        string path = "";
        dfs(root, path);
        return best;
    }
};
```

**Why the `child.isEnd` gate?** A word like `"apple"` is only valid if `"a"`, `"ap"`, `"app"`, `"appl"`, `"apple"` are *all* present. By refusing to descend through a non-`isEnd` node, the DFS structurally guarantees every prefix on the path is a word.

**Complexity:** **O(sum of word lengths)** to build the trie, **O(number of trie nodes)** for the DFS — effectively **O(total characters)**.

---

## Problem 5 — Replace Words (root replacement) · LC 648 · Medium

Given a dictionary of roots and a sentence, replace each word by the **shortest root** that is a prefix of it. Build a trie of roots; for each word, walk the trie and stop at the first node with `isEnd` (shortest matching root). If no root matches, keep the word.

```cpp
#include <bits/stdc++.h>
using namespace std;

struct TrieNode {
    TrieNode* children[26];
    bool isEnd = false;
    TrieNode() { fill(children, children + 26, nullptr); }
};

class Solution {
private:
    TrieNode* root;

    void insert(const string& r) {
        TrieNode* node = this->root;
        for (char ch : r) {
            int idx = ch - 'a';
            if (node->children[idx] == nullptr) node->children[idx] = new TrieNode();
            node = node->children[idx];
        }
        node->isEnd = true;
    }

    // Return the shortest root prefix of `word`, or `word` itself if none.
    string shortestRoot(const string& word) {
        TrieNode* node = root;
        for (int i = 0; i < (int)word.length(); i++) {
            int idx = word[i] - 'a';
            if (node->children[idx] == nullptr) break;     // no further root possible
            node = node->children[idx];
            if (node->isEnd) return word.substr(0, i + 1);  // shortest root found
        }
        return word;
    }

public:
    Solution() : root(new TrieNode()) {}

    string replaceWords(vector<string>& dictionary, const string& sentence) {
        for (auto& r : dictionary) insert(r);

        // Split sentence by spaces
        istringstream iss(sentence);
        string token;
        vector<string> words;
        while (iss >> token) words.push_back(token);

        string sb = "";
        for (int w = 0; w < (int)words.size(); w++) {
            if (w > 0) sb += ' ';
            sb += shortestRoot(words[w]);
        }
        return sb;
    }
};
```

**Key insight:** because we return at the *first* `isEnd` we hit while walking down, we always get the **shortest** root (e.g. given roots `"cat"` and `"cattle"`, the word `"cattle"` is replaced by `"cat"`).

**Complexity:** **O(sum of root lengths)** to build, **O(sum of word lengths)** to replace.

---

## Problem 7 — Word Search II (Trie + DFS on board) · LC 212 · Hard

Find all dictionary words that exist on a 2D board (adjacent horizontally/vertically, no cell reuse within one word). The naive approach runs a separate DFS per word: **O(words · M·N·4^L)**. The trie approach builds **one** trie of all words and DFSes the board once, following the trie in lockstep with the board path.

Two crucial optimizations:
1. **Store the complete word at its leaf node** (`node.word`) so we never build a string during the hot DFS — when `node.word` is non-empty we have found that exact word.
2. **Prune found words** by setting `node.word = ""` after collecting them, so duplicates aren't re-reported and dead branches shrink.

```cpp
#include <bits/stdc++.h>
using namespace std;

struct TrieNode {
    TrieNode* children[26];
    string word = "";   // the full word ending here, or empty string

    TrieNode() { fill(children, children + 26, nullptr); }
};

class Solution {
private:
    TrieNode* buildTrie(vector<string>& words) {
        TrieNode* root = new TrieNode();
        for (auto& w : words) {
            TrieNode* node = root;
            for (char ch : w) {
                int idx = ch - 'a';
                if (node->children[idx] == nullptr) node->children[idx] = new TrieNode();
                node = node->children[idx];
            }
            node->word = w;        // store full word at leaf — no string building needed
        }
        return root;
    }

    void dfs(vector<vector<char>>& board, int r, int c, TrieNode* node, vector<string>& result) {
        char ch = board[r][c];
        if (ch == '#') return;                       // already visited on this path
        TrieNode* next = node->children[ch - 'a'];
        if (next == nullptr) return;                 // prune: no word continues here

        if (!next->word.empty()) {
            result.push_back(next->word);
            next->word = "";                         // prune: don't report twice
        }

        board[r][c] = '#';                           // mark visited
        int dirs[4][2] = {{1,0},{-1,0},{0,1},{0,-1}};
        for (auto& d : dirs) {
            int nr = r + d[0], nc = c + d[1];
            if (nr >= 0 && nr < (int)board.size() && nc >= 0 && nc < (int)board[0].size()) {
                dfs(board, nr, nc, next, result);
            }
        }
        board[r][c] = ch;                            // restore (backtrack)
    }

public:
    vector<string> findWords(vector<vector<char>>& board, vector<string>& words) {
        TrieNode* root = buildTrie(words);
        vector<string> result;
        for (int r = 0; r < (int)board.size(); r++) {
            for (int c = 0; c < (int)board[0].size(); c++) {
                dfs(board, r, c, root, result);
            }
        }
        return result;
    }
};
```

**Why this crushes the naive approach:** all words share the trie, so common prefixes are explored once. The board DFS is bounded by the trie depth (longest word) and dead-ends are cut immediately when `node->children[ch - 'a'] == nullptr`. See the complexity comparison in [Interview Problems/Amazon.md](../Interview%20Problems/Amazon.md).

**Complexity:** **O(M · N · 4^Lmax)** worst case (`Lmax` = longest word length, the `4` = branching after the first step is actually `3`, but `4` for the bound). Building the trie is **O(total characters in words)**.

---

## Dry Run — Word Search II pruning

Board:
```
o a a n
e t a e
i h k r
i f l v
```
Words: `["oath", "pea", "eat", "rain"]`. Trie leaves carry: `oath`, `pea`, `eat`, `rain`.

- Start DFS at `(0,0)='o'`: trie has child `o` → follow. Mark `(0,0)='#'`.
- Neighbor `(1,0)='e'`? trie node after `o` has no `e` child → return (prune).
- Neighbor `(0,1)='a'`: node after `o` has child `a` → follow `oa`.
  - From `(0,1)`, neighbor `(1,1)='t'`: node after `oa` has child `t` → `oat`.
    - From `(1,1)`, neighbor `(2,1)='h'`: node after `oat` has child `h` → `oath`. `node->word == "oath"` → **add "oath"**, then set `node->word = ""` (pruned).
- `"pea"` has no `p` anywhere reachable → never explored beyond the first cell mismatch.
- `"eat"`: starts at `(1,0)='e'`. DFS reaches `eat` and adds it.
- `"rain"`: `r` at `(2,3)`, but no path spells `rain` on adjacent cells → trie walk dead-ends → pruned early.

Result: `["oath", "eat"]`. The `'#'` marking prevents reusing a cell; restoring it on backtrack keeps the board valid for other start cells; setting `node->word = ""` stops `"oath"` from being added again if reachable another way.

---

## Recognition Signals Recap

- `.` / `?` wildcard in a dictionary search → DFS branching all children (Problem 3).
- "build word one letter at a time, every prefix valid" → `isEnd`-gated DFS (Problem 4).
- "replace by shortest/longest root prefix" → walk to first `isEnd` (Problem 5).
- "find all dictionary words on a grid" → trie + board DFS with pruning (Problem 7).

See also: [Trie Fundamentals](Trie%20Fundamentals.md), [Word Dictionary design](../Design%20Data%20Structure%20Problems/Word%20Dictionary.md).

---

> **Last Updated:** 2026-06-26
