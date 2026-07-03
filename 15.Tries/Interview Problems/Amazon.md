> **Topic:** [Tries](../README.md) · **Interview Problems**

# Amazon — Interview Problems (Tries)

Amazon loves Tries because they map directly onto real product surfaces: search autocomplete, product catalog prefix matching, and spell/word filtering. These three problems are the most commonly asked Trie questions in Amazon loops. See the building blocks in [Trie Fundamentals](../Patterns/Trie%20Fundamentals.md) and [Trie with Search Variants](../Patterns/Trie%20with%20Search%20Variants.md).

---

## (a) Implement Trie — LC 208

**Problem.** Implement a `Trie` (prefix tree) supporting `insert(word)`, `search(word)` (exact match), and `startsWith(prefix)` (any word with this prefix exists).

**Framing it as an autocomplete backend.** When a customer types into the Amazon search box, the backend must answer "is `lapt` a prefix of any product term?" in time proportional to the length of what the user typed, not the size of the catalog. A Trie gives exactly that: `startsWith` is `O(L)` regardless of how many millions of products exist. `search` is the "did they type a complete known term" check, and `insert` is how we load the dictionary of search terms at index-build time.

**Design decision: `TrieNode*[26]` vs `unordered_map<char, TrieNode*>`.**

| Aspect | `TrieNode* children[26]` | `unordered_map<char, TrieNode*>` |
|---|---|---|
| Lookup | O(1) array index, no hashing | O(1) amortized, but with hashing overhead |
| Memory | 26 pointers per node even if sparse | Only allocates entries that exist |
| Alphabet | Fixed lowercase `a–z` | Arbitrary / Unicode |
| Cache behavior | Contiguous, cache friendly | Pointer-chasing |

For this problem the constraint is lowercase English letters, so the array form is preferred: it is faster and simpler, and the `26`-wide fan-out is acceptable. If the alphabet were Unicode or the tree extremely sparse, the unordered_map form would win on memory. **Say this trade-off out loud in the interview** — that is the point of the question.

```cpp
#include <bits/stdc++.h>
using namespace std;

struct TrieNode {
    TrieNode* children[26];
    bool isEnd;
    TrieNode() : isEnd(false) {
        fill(begin(children), end(children), nullptr);
    }
};

class Trie {
    TrieNode* root;

    TrieNode* find(const string& s) {
        TrieNode* node = root;
        for (char ch : s) {
            int idx = ch - 'a';
            if (node->children[idx] == nullptr) {
                return nullptr;
            }
            node = node->children[idx];
        }
        return node;
    }

public:
    Trie() {
        root = new TrieNode();
    }

    void insert(const string& word) {
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
        TrieNode* node = find(word);
        return node != nullptr && node->isEnd;
    }

    bool startsWith(const string& prefix) {
        return find(prefix) != nullptr;
    }
};
```

**Reasoning.** `search` and `startsWith` differ only by the final `isEnd` check; factoring the walk into a private `find` removes duplication. The `isEnd` flag is what lets `"app"` and `"apple"` coexist — without it, inserting `"apple"` would make `search("app")` falsely return true.

**Complexity.** Let `L` be the word/prefix length. `insert`, `search`, `startsWith` are each `O(L)` time. Space is `O(total characters inserted)` in the worst case, i.e. `O(N · L)` for `N` words with no shared prefixes.

**Follow-ups.**
- *Add autocomplete (`vector<string> suggestions(prefix)`):* walk to the prefix node, then DFS collecting words. To bound it, cap at top-k.
- *Make it case-insensitive / support digits:* widen the array or switch to a HashMap.
- *Concurrency:* the read-heavy autocomplete case can use a copy-on-write or an immutable Trie rebuilt at index time.

---

## (b) Word Search II — LC 212

**Problem.** Given an `m x n` board of characters and a list of `words`, return all words on the board. A word is formed from sequentially adjacent (horizontally/vertically) cells, and a cell may not be reused within a single word.

**Key idea.** Instead of running a board search for every word independently, build **one** Trie from all words and DFS the board once, walking the Trie in lockstep with the path. Shared prefixes are explored only once. We store the full word string at the terminal node so that when we reach an `isEnd`, we can collect the answer without reconstructing the path.

```cpp
#include <bits/stdc++.h>
using namespace std;

struct TrieNode {
    TrieNode* children[26];
    string word; // non-empty only at a terminal node; holds the full word
    TrieNode() {
        fill(begin(children), end(children), nullptr);
    }
};

class Solution {
    TrieNode* buildTrie(const vector<string>& words) {
        TrieNode* root = new TrieNode();
        for (auto& w : words) {
            TrieNode* node = root;
            for (char ch : w) {
                int idx = ch - 'a';
                if (node->children[idx] == nullptr) {
                    node->children[idx] = new TrieNode();
                }
                node = node->children[idx];
            }
            node->word = w;
        }
        return root;
    }

    void dfs(vector<vector<char>>& board, int r, int c, TrieNode* node, vector<string>& result) {
        int m = board.size(), n = board[0].size();
        if (r < 0 || r >= m || c < 0 || c >= n) {
            return;
        }
        char ch = board[r][c];
        if (ch == '#') {
            return;                       // already on current path
        }
        TrieNode* next = node->children[ch - 'a'];
        if (next == nullptr) {
            return;                       // no word continues with this prefix -> prune
        }
        if (!next->word.empty()) {
            result.push_back(next->word);
            next->word = "";              // prune: avoid adding duplicates
        }

        board[r][c] = '#';                // mark visited
        dfs(board, r + 1, c, next, result);
        dfs(board, r - 1, c, next, result);
        dfs(board, r, c + 1, next, result);
        dfs(board, r, c - 1, next, result);
        board[r][c] = ch;                 // restore
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

**The mark-visited trick.** A single word cannot reuse a cell, so during one DFS path we temporarily overwrite `board[r][c]` with `'#'` before recursing and restore the original character afterward. This in-place marking avoids allocating a separate `vector<vector<bool>> visited` per call and is `O(1)` per cell. The check `if (ch == '#') return;` is what enforces "no cell reused on the current path."

**The pruning trick.** Setting `next.word = null` after collecting a word means we never report the same word twice and we let the Trie shrink as words are found. A stronger variant also prunes dead leaf nodes (children that lead nowhere) to keep the search tree small — see the Google deep dive for that optimization.

**Complexity comparison.**

| Approach | Time | Why |
|---|---|---|
| Naive: DFS per word | `O(W · M·N · 4^L)` | For each of `W` words, start a 4-direction DFS from every cell; each DFS branches 4 ways up to depth `L` |
| Trie + shared DFS | `O(M·N · 4^Lmax)` | Board is traversed once; the Trie collapses shared prefixes so each board path is explored a single time regardless of `W` |

Here `W` = number of words, `M·N` = board cells, `L` = word length, `Lmax` = longest word. The win is that the per-word factor `W` disappears: words sharing prefixes (`"oath"`, `"oat"`, `"oats"`) drive the same board exploration only once.

**Reasoning.** Storing `word` at the leaf (rather than `boolean isEnd`) lets us add the answer immediately on arrival — no path reconstruction. The board walk and Trie walk advance together: the Trie tells us "is there any word continuing with this prefix?"; the moment `node.children[ch - 'a']` is null we stop, which is the pruning that makes this fast.

**Follow-ups.**
- *Memory:* prune empty subtrees to let GC reclaim nodes as words are found.
- *Very long words / huge board:* the exponential `4^Lmax` is unavoidable in the worst case, but Trie pruning makes the realistic case dramatically faster.
- *Diagonal moves / wrap-around:* change the neighbor offsets.

---

## (c) Replace Words — LC 648

**Problem.** Given a dictionary of `roots` and a `sentence`, replace every word in the sentence with the **shortest root** that is a prefix of it. If no root applies, keep the word.

**Shortest-root logic.** Build a Trie of the roots. For each word, walk character by character; the **first** time we hit a node with `isEnd == true`, we have found the shortest root (we stop immediately). If we never hit an `isEnd` (or the path breaks), keep the original word. Stopping at the first `isEnd` is what guarantees "shortest" — a longer matching root would only be found if we kept walking, and we deliberately do not.

```cpp
#include <bits/stdc++.h>
using namespace std;

struct TrieNode {
    TrieNode* children[26];
    bool isEnd;
    TrieNode() : isEnd(false) {
        fill(begin(children), end(children), nullptr);
    }
};

class Solution {
    void insert(TrieNode* root, const string& word) {
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

    string shortestRoot(TrieNode* root, const string& word) {
        TrieNode* node = root;
        for (int i = 0; i < (int)word.size(); i++) {
            int idx = word[i] - 'a';
            if (node->children[idx] == nullptr) {
                return word;              // path breaks, no root prefix
            }
            node = node->children[idx];
            if (node->isEnd) {
                return word.substr(0, i + 1);   // first (shortest) root
            }
        }
        return word;                      // word itself never matched a root
    }

public:
    string replaceWords(vector<string>& dictionary, string sentence) {
        TrieNode* root = new TrieNode();
        for (auto& r : dictionary) {
            insert(root, r);
        }

        // split sentence by spaces
        vector<string> words;
        istringstream iss(sentence);
        string token;
        while (iss >> token) words.push_back(token);

        string result;
        for (int i = 0; i < (int)words.size(); i++) {
            if (i > 0) result += ' ';
            result += shortestRoot(root, words[i]);
        }
        return result;
    }
};
```

**Reasoning.** The `return` inside the loop the moment `node.isEnd` is true is the entire "shortest" guarantee. `word.substring(0, i + 1)` slices off the root prefix. If the path through the Trie breaks before any `isEnd`, the word has no root prefix and is returned unchanged.

**Complexity.** Building the Trie is `O(sum of root lengths)`. Replacement is `O(total characters in the sentence)` since each character is walked at most once. Space is `O(sum of root lengths)`.

**Follow-ups.**
- *Longest root instead of shortest:* keep walking, remember the last `isEnd` index, return that prefix.
- *Multiple roots, return all:* collect every `isEnd` along the path.
- *Streaming sentence:* process word-by-word without splitting the whole string.

---

## Amazon Leadership Principles alignment

| Problem / Skill | Leadership Principle | One-line justification |
|---|---|---|
| Implement Trie as autocomplete backend (208) | Customer Obsession | `startsWith` is `O(L)` so search suggestions stay instant no matter how large the catalog grows. |
| Array[26] vs HashMap children trade-off | Invent and Simplify | Choosing the simplest structure that fits the alphabet, and justifying it, instead of over-engineering. |
| Word Search II naive-vs-Trie complexity analysis (212) | Dive Deep | Quantifying `O(W·M·N·4^L)` → `O(M·N·4^Lmax)` shows you understand *why* the optimization works, not just that it does. |
| Mark-visited + prune `node.word = null` (212) | Insist on the Highest Standards | Correct in-place visited handling and dedup pruning produce clean, leak-free, bug-free code under pressure. |
| Shortest-root early return (648) | Bias for Action | Returning at the first `isEnd` is the minimal correct move — no wasted traversal, ship the simplest correct path. |

---

> **Last Updated:** 2026-06-26
