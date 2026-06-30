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

```java
class WordDictionary {
    static class TrieNode {
        TrieNode[] children = new TrieNode[26];
        boolean isEnd = false;
    }

    private final TrieNode root = new TrieNode();

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
        if (node == null) return false;
        if (i == word.length()) return node.isEnd;   // matched all chars

        char c = word.charAt(i);
        if (c == '.') {
            // wildcard: try every existing child
            for (TrieNode child : node.children) {
                if (child != null && dfs(word, i + 1, child)) {
                    return true;
                }
            }
            return false;
        } else {
            return dfs(word, i + 1, node.children[c - 'a']);
        }
    }
}
```

**Complexity:** `addWord` is **O(L)**. `search` is **O(L)** for a concrete word, but **O(26^k · L)** in the worst case where `k` is the number of `.` characters — each dot can branch into 26 subtrees. Full analysis in [Word Dictionary.md](../Design%20Data%20Structure%20Problems/Word%20Dictionary.md).

---

## Problem 4 — Longest Word in Dictionary · LC 720 · Medium

Find the longest word that can be **built one character at a time**, where every intermediate prefix is also a word in the dictionary. Ties broken by lexicographically smallest.

Insert all words, then DFS: only descend into a child if that child `isEnd` (its prefix is a valid buildable word). Because we iterate children `a → z`, the first/longest valid word we record under lexicographic tie-breaking is correct.

```java
class Solution {
    static class TrieNode {
        TrieNode[] children = new TrieNode[26];
        boolean isEnd = false;
    }

    private String best = "";

    public String longestWord(String[] words) {
        TrieNode root = new TrieNode();
        for (String w : words) insert(root, w);
        // Root's prefix "" is trivially buildable, so start DFS from root.
        dfs(root, new StringBuilder());
        return best;
    }

    private void insert(TrieNode root, String w) {
        TrieNode node = root;
        for (int i = 0; i < w.length(); i++) {
            int idx = w.charAt(i) - 'a';
            if (node.children[idx] == null) node.children[idx] = new TrieNode();
            node = node.children[idx];
        }
        node.isEnd = true;
    }

    private void dfs(TrieNode node, StringBuilder path) {
        // Update best: longer wins; on equal length, smaller lexicographic wins.
        String cur = path.toString();
        if (cur.length() > best.length()
            || (cur.length() == best.length() && cur.compareTo(best) < 0)) {
            best = cur;
        }
        // Iterate a..z so ties naturally favour the smaller letter.
        for (int c = 0; c < 26; c++) {
            TrieNode child = node.children[c];
            // Only continue if this prefix is itself a complete word.
            if (child != null && child.isEnd) {
                path.append((char) ('a' + c));
                dfs(child, path);
                path.deleteCharAt(path.length() - 1);   // backtrack
            }
        }
    }
}
```

**Why the `child.isEnd` gate?** A word like `"apple"` is only valid if `"a"`, `"ap"`, `"app"`, `"appl"`, `"apple"` are *all* present. By refusing to descend through a non-`isEnd` node, the DFS structurally guarantees every prefix on the path is a word.

**Complexity:** **O(sum of word lengths)** to build the trie, **O(number of trie nodes)** for the DFS — effectively **O(total characters)**.

---

## Problem 5 — Replace Words (root replacement) · LC 648 · Medium

Given a dictionary of roots and a sentence, replace each word by the **shortest root** that is a prefix of it. Build a trie of roots; for each word, walk the trie and stop at the first node with `isEnd` (shortest matching root). If no root matches, keep the word.

```java
class Solution {
    static class TrieNode {
        TrieNode[] children = new TrieNode[26];
        boolean isEnd = false;
    }

    private final TrieNode root = new TrieNode();

    public String replaceWords(List<String> dictionary, String sentence) {
        for (String r : dictionary) insert(r);

        String[] words = sentence.split(" ");
        StringBuilder sb = new StringBuilder();
        for (int w = 0; w < words.length; w++) {
            if (w > 0) sb.append(' ');
            sb.append(shortestRoot(words[w]));
        }
        return sb.toString();
    }

    private void insert(String root) {
        TrieNode node = this.root;
        for (int i = 0; i < root.length(); i++) {
            int idx = root.charAt(i) - 'a';
            if (node.children[idx] == null) node.children[idx] = new TrieNode();
            node = node.children[idx];
        }
        node.isEnd = true;
    }

    // Return the shortest root prefix of `word`, or `word` itself if none.
    private String shortestRoot(String word) {
        TrieNode node = root;
        for (int i = 0; i < word.length(); i++) {
            int idx = word.charAt(i) - 'a';
            if (node.children[idx] == null) break;     // no further root possible
            node = node.children[idx];
            if (node.isEnd) return word.substring(0, i + 1);  // shortest root found
        }
        return word;
    }
}
```

**Key insight:** because we return at the *first* `isEnd` we hit while walking down, we always get the **shortest** root (e.g. given roots `"cat"` and `"cattle"`, the word `"cattle"` is replaced by `"cat"`).

**Complexity:** **O(sum of root lengths)** to build, **O(sum of word lengths)** to replace.

---

## Problem 7 — Word Search II (Trie + DFS on board) · LC 212 · Hard

Find all dictionary words that exist on a 2D board (adjacent horizontally/vertically, no cell reuse within one word). The naive approach runs a separate DFS per word: **O(words · M·N·4^L)**. The trie approach builds **one** trie of all words and DFSes the board once, following the trie in lockstep with the board path.

Two crucial optimizations:
1. **Store the complete word at its leaf node** (`node.word`) so we never build a `StringBuilder` during the hot DFS — when `node.word != null` we have found that exact word.
2. **Prune found words** by setting `node.word = null` after collecting them, so duplicates aren't re-reported and dead branches shrink.

```java
class Solution {
    static class TrieNode {
        TrieNode[] children = new TrieNode[26];
        String word = null;   // the full word ending here, or null
    }

    public List<String> findWords(char[][] board, String[] words) {
        TrieNode root = buildTrie(words);
        List<String> result = new ArrayList<>();
        for (int r = 0; r < board.length; r++) {
            for (int c = 0; c < board[0].length; c++) {
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
                if (node.children[idx] == null) node.children[idx] = new TrieNode();
                node = node.children[idx];
            }
            node.word = w;        // store full word at leaf — no StringBuilder needed
        }
        return root;
    }

    private void dfs(char[][] board, int r, int c, TrieNode node, List<String> result) {
        char ch = board[r][c];
        if (ch == '#') return;                       // already visited on this path
        TrieNode next = node.children[ch - 'a'];
        if (next == null) return;                    // prune: no word continues here

        if (next.word != null) {
            result.add(next.word);
            next.word = null;                        // prune: don't report twice
        }

        board[r][c] = '#';                           // mark visited
        int[][] dirs = {{1,0},{-1,0},{0,1},{0,-1}};
        for (int[] d : dirs) {
            int nr = r + d[0], nc = c + d[1];
            if (nr >= 0 && nr < board.length && nc >= 0 && nc < board[0].length) {
                dfs(board, nr, nc, next, result);
            }
        }
        board[r][c] = ch;                            // restore (backtrack)
    }
}
```

**Why this crushes the naive approach:** all words share the trie, so common prefixes are explored once. The board DFS is bounded by the trie depth (longest word) and dead-ends are cut immediately when `node.children[ch - 'a'] == null`. See the complexity comparison in [Interview Problems/Amazon.md](../Interview%20Problems/Amazon.md).

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
    - From `(1,1)`, neighbor `(2,1)='h'`: node after `oat` has child `h` → `oath`. `node.word == "oath"` → **add "oath"**, then set `node.word = null` (pruned).
- `"pea"` has no `p` anywhere reachable → never explored beyond the first cell mismatch.
- `"eat"`: starts at `(1,0)='e'`. DFS reaches `eat` and adds it.
- `"rain"`: `r` at `(2,3)`, but no path spells `rain` on adjacent cells → trie walk dead-ends → pruned early.

Result: `["oath", "eat"]`. The `'#'` marking prevents reusing a cell; restoring it on backtrack keeps the board valid for other start cells; nulling `node.word` stops `"oath"` from being added again if reachable another way.

---

## Recognition Signals Recap

- `.` / `?` wildcard in a dictionary search → DFS branching all children (Problem 3).
- "build word one letter at a time, every prefix valid" → `isEnd`-gated DFS (Problem 4).
- "replace by shortest/longest root prefix" → walk to first `isEnd` (Problem 5).
- "find all dictionary words on a grid" → trie + board DFS with pruning (Problem 7).

See also: [Trie Fundamentals](Trie%20Fundamentals.md), [Word Dictionary design](../Design%20Data%20Structure%20Problems/Word%20Dictionary.md).

---

> **Last Updated:** 2026-06-26
