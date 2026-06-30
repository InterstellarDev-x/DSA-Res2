> **Topic:** [Tries](../README.md) · **Interview Problems**

# Amazon — Interview Problems (Tries)

Amazon loves Tries because they map directly onto real product surfaces: search autocomplete, product catalog prefix matching, and spell/word filtering. These three problems are the most commonly asked Trie questions in Amazon loops. See the building blocks in [Trie Fundamentals](../Patterns/Trie%20Fundamentals.md) and [Trie with Search Variants](../Patterns/Trie%20with%20Search%20Variants.md).

---

## (a) Implement Trie — LC 208

**Problem.** Implement a `Trie` (prefix tree) supporting `insert(word)`, `search(word)` (exact match), and `startsWith(prefix)` (any word with this prefix exists).

**Framing it as an autocomplete backend.** When a customer types into the Amazon search box, the backend must answer "is `lapt` a prefix of any product term?" in time proportional to the length of what the user typed, not the size of the catalog. A Trie gives exactly that: `startsWith` is `O(L)` regardless of how many millions of products exist. `search` is the "did they type a complete known term" check, and `insert` is how we load the dictionary of search terms at index-build time.

**Design decision: `TrieNode[26]` vs `HashMap<Character, TrieNode>`.**

| Aspect | `TrieNode[] children = new TrieNode[26]` | `HashMap<Character, TrieNode>` |
|---|---|---|
| Lookup | O(1) array index, no hashing | O(1) amortized, but with hashing + boxing overhead |
| Memory | 26 references per node even if sparse | Only allocates entries that exist |
| Alphabet | Fixed lowercase `a–z` | Arbitrary / Unicode |
| Cache behavior | Contiguous, cache friendly | Pointer-chasing |

For this problem the constraint is lowercase English letters, so the array form is preferred: it is faster and simpler, and the `26`-wide fan-out is acceptable. If the alphabet were Unicode or the tree extremely sparse, the HashMap form would win on memory. **Say this trade-off out loud in the interview** — that is the point of the question.

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

**Reasoning.** `search` and `startsWith` differ only by the final `isEnd` check; factoring the walk into a private `find` removes duplication. The `isEnd` flag is what lets `"app"` and `"apple"` coexist — without it, inserting `"apple"` would make `search("app")` falsely return true.

**Complexity.** Let `L` be the word/prefix length. `insert`, `search`, `startsWith` are each `O(L)` time. Space is `O(total characters inserted)` in the worst case, i.e. `O(N · L)` for `N` words with no shared prefixes.

**Follow-ups.**
- *Add autocomplete (`List<String> suggestions(prefix)`):* walk to the prefix node, then DFS collecting words. To bound it, cap at top-k.
- *Make it case-insensitive / support digits:* widen the array or switch to a HashMap.
- *Concurrency:* the read-heavy autocomplete case can use a copy-on-write or an immutable Trie rebuilt at index time.

---

## (b) Word Search II — LC 212

**Problem.** Given an `m x n` board of characters and a list of `words`, return all words on the board. A word is formed from sequentially adjacent (horizontally/vertically) cells, and a cell may not be reused within a single word.

**Key idea.** Instead of running a board search for every word independently, build **one** Trie from all words and DFS the board once, walking the Trie in lockstep with the path. Shared prefixes are explored only once. We store the full word string at the terminal node so that when we reach an `isEnd`, we can collect the answer without reconstructing the path.

```java
class Solution {
    static class TrieNode {
        TrieNode[] children = new TrieNode[26];
        String word;   // non-null only at a terminal node; holds the full word
    }

    public List<String> findWords(char[][] board, String[] words) {
        TrieNode root = buildTrie(words);
        List<String> result = new ArrayList<>();
        int m = board.length, n = board[0].length;
        for (int r = 0; r < m; r++) {
            for (int c = 0; c < n; c++) {
                dfs(board, r, c, root, result);
            }
        }
        return result;
    }

    private TrieNode buildTrie(String[] words) {
        TrieNode root = new TrieNode();
        for (String w : words) {
            TrieNode node = root;
            for (int i = 0; i < w.length(); i++) {
                int idx = w.charAt(i) - 'a';
                if (node.children[idx] == null) {
                    node.children[idx] = new TrieNode();
                }
                node = node.children[idx];
            }
            node.word = w;
        }
        return root;
    }

    private void dfs(char[][] board, int r, int c, TrieNode node, List<String> result) {
        int m = board.length, n = board[0].length;
        if (r < 0 || r >= m || c < 0 || c >= n) {
            return;
        }
        char ch = board[r][c];
        if (ch == '#') {
            return;                       // already on current path
        }
        TrieNode next = node.children[ch - 'a'];
        if (next == null) {
            return;                       // no word continues with this prefix -> prune
        }
        if (next.word != null) {
            result.add(next.word);
            next.word = null;             // prune: avoid adding duplicates
        }

        board[r][c] = '#';                // mark visited
        dfs(board, r + 1, c, next, result);
        dfs(board, r - 1, c, next, result);
        dfs(board, r, c + 1, next, result);
        dfs(board, r, c - 1, next, result);
        board[r][c] = ch;                 // restore
    }
}
```

**The mark-visited trick.** A single word cannot reuse a cell, so during one DFS path we temporarily overwrite `board[r][c]` with `'#'` before recursing and restore the original character afterward. This in-place marking avoids allocating a separate `boolean[][] visited` per call and is `O(1)` per cell. The check `if (ch == '#') return;` is what enforces "no cell reused on the current path."

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

```java
class Solution {
    static class TrieNode {
        TrieNode[] children = new TrieNode[26];
        boolean isEnd;
    }

    public String replaceWords(List<String> dictionary, String sentence) {
        TrieNode root = new TrieNode();
        for (String r : dictionary) {
            insert(root, r);
        }

        String[] words = sentence.split(" ");
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < words.length; i++) {
            if (i > 0) {
                sb.append(' ');
            }
            sb.append(shortestRoot(root, words[i]));
        }
        return sb.toString();
    }

    private void insert(TrieNode root, String word) {
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

    private String shortestRoot(TrieNode root, String word) {
        TrieNode node = root;
        for (int i = 0; i < word.length(); i++) {
            int idx = word.charAt(i) - 'a';
            if (node.children[idx] == null) {
                return word;              // path breaks, no root prefix
            }
            node = node.children[idx];
            if (node.isEnd) {
                return word.substring(0, i + 1);   // first (shortest) root
            }
        }
        return word;                      // word itself never matched a root
    }
}
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
