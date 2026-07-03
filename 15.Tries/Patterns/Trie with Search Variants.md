> **Topic:** [Tries](../README.md) · **Pattern 2 of 3**

# Trie with Search Variants

## Core Concept

Once you can build a trie ([Trie Fundamentals](Trie%20Fundamentals.md)), the interesting problems layer a **search strategy** on top of it:

- **Wildcards** (`.` matches any letter) → DFS that branches into *all* children when it hits a `.`.
- **Build-one-char-at-a-time** (Longest Word) → DFS that only descends through nodes whose every prefix is itself a complete word.
- **Root replacement** (Replace Words) → walk each query word until you hit the first `is_end` (shortest root).
- **Board search** (Word Search II) → build a trie of the target words, then DFS the grid, pruning the trie as words are found.

The unifying idea: the trie is a **shared index**, and DFS/recursion explores it far more cheaply than running each query independently.

**Recognition signals:** wildcard matching, "find all words on a board", autocomplete with patterns, replacing words by shortest root, building words incrementally.

---

## Problem 3 — Add and Search Word (Word Dictionary, `.` wildcard) · LC 211 · Medium

`add_word` is a normal insert. `search` must handle `.` which matches **any** single character. When the DFS encounters `.`, it recurses into every non-null child.

```rust
struct TrieNode {
    children: [Option<usize>; 26],
    is_end: bool,
}

impl TrieNode {
    fn new() -> Self {
        TrieNode {
            children: [None; 26],
            is_end: false,
        }
    }
}

struct WordDictionary {
    nodes: Vec<TrieNode>,
}

impl WordDictionary {
    fn new() -> Self {
        WordDictionary {
            nodes: vec![TrieNode::new()], // root at index 0
        }
    }

    fn add_word(&mut self, word: String) {
        let mut idx = 0;
        for ch in word.bytes() {
            let c = (ch - b'a') as usize;
            if self.nodes[idx].children[c].is_none() {
                self.nodes.push(TrieNode::new());
                let new_idx = self.nodes.len() - 1;
                self.nodes[idx].children[c] = Some(new_idx);
            }
            idx = self.nodes[idx].children[c].unwrap();
        }
        self.nodes[idx].is_end = true;
    }

    fn search(&self, word: String) -> bool {
        self.dfs(word.as_bytes(), 0, 0)
    }

    fn dfs(&self, word: &[u8], i: usize, node_idx: usize) -> bool {
        if i == word.len() {
            return self.nodes[node_idx].is_end;   // matched all chars
        }
        let c = word[i];
        if c == b'.' {
            // wildcard: try every existing child
            for child_opt in &self.nodes[node_idx].children {
                if let Some(child_idx) = child_opt {
                    if self.dfs(word, i + 1, *child_idx) {
                        return true;
                    }
                }
            }
            false
        } else {
            let idx = (c - b'a') as usize;
            match self.nodes[node_idx].children[idx] {
                Some(child_idx) => self.dfs(word, i + 1, child_idx),
                None => false,
            }
        }
    }
}
```

**Complexity:** `add_word` is **O(L)**. `search` is **O(L)** for a concrete word, but **O(26^k · L)** in the worst case where `k` is the number of `.` characters — each dot can branch into 26 subtrees. Full analysis in [Word Dictionary.md](../Design%20Data%20Structure%20Problems/Word%20Dictionary.md).

---

## Problem 4 — Longest Word in Dictionary · LC 720 · Medium

Find the longest word that can be **built one character at a time**, where every intermediate prefix is also a word in the dictionary. Ties broken by lexicographically smallest.

Insert all words, then DFS: only descend into a child if that child `is_end` (its prefix is a valid buildable word). Because we iterate children `a → z`, the first/longest valid word we record under lexicographic tie-breaking is correct.

```rust
struct TrieNode {
    children: [Option<usize>; 26],
    is_end: bool,
}

impl TrieNode {
    fn new() -> Self {
        TrieNode {
            children: [None; 26],
            is_end: false,
        }
    }
}

struct Solution {
    nodes: Vec<TrieNode>,
    best: String,
}

impl Solution {
    fn new() -> Self {
        Solution {
            nodes: vec![TrieNode::new()], // root at index 0
            best: String::new(),
        }
    }

    fn insert(&mut self, w: &str) {
        let mut idx = 0;
        for ch in w.bytes() {
            let c = (ch - b'a') as usize;
            if self.nodes[idx].children[c].is_none() {
                self.nodes.push(TrieNode::new());
                let new_idx = self.nodes.len() - 1;
                self.nodes[idx].children[c] = Some(new_idx);
            }
            idx = self.nodes[idx].children[c].unwrap();
        }
        self.nodes[idx].is_end = true;
    }

    fn dfs(&mut self, node_idx: usize, path: &mut String) {
        // Update best: longer wins; on equal length, smaller lexicographic wins.
        if path.len() > self.best.len()
            || (path.len() == self.best.len() && path.as_str() < self.best.as_str())
        {
            self.best = path.clone();
        }
        // Iterate a..z so ties naturally favour the smaller letter.
        for c in 0..26usize {
            // Copy indices out to avoid holding a borrow across the recursive call.
            let child_opt = self.nodes[node_idx].children[c];
            if let Some(child_idx) = child_opt {
                let child_is_end = self.nodes[child_idx].is_end;
                // Only continue if this prefix is itself a complete word.
                if child_is_end {
                    path.push((b'a' + c as u8) as char);
                    self.dfs(child_idx, path);
                    path.pop(); // backtrack
                }
            }
        }
    }

    fn longest_word(&mut self, words: Vec<String>) -> String {
        for w in &words {
            self.insert(w);
        }
        // Root's prefix "" is trivially buildable, so start DFS from root.
        let mut path = String::new();
        self.dfs(0, &mut path);
        self.best.clone()
    }
}
```

**Why the `child_is_end` gate?** A word like `"apple"` is only valid if `"a"`, `"ap"`, `"app"`, `"appl"`, `"apple"` are *all* present. By refusing to descend through a non-`is_end` node, the DFS structurally guarantees every prefix on the path is a word.

**Complexity:** **O(sum of word lengths)** to build the trie, **O(number of trie nodes)** for the DFS — effectively **O(total characters)**.

---

## Problem 5 — Replace Words (root replacement) · LC 648 · Medium

Given a dictionary of roots and a sentence, replace each word by the **shortest root** that is a prefix of it. Build a trie of roots; for each word, walk the trie and stop at the first node with `is_end` (shortest matching root). If no root matches, keep the word.

```rust
struct TrieNode {
    children: [Option<usize>; 26],
    is_end: bool,
}

impl TrieNode {
    fn new() -> Self {
        TrieNode {
            children: [None; 26],
            is_end: false,
        }
    }
}

struct Solution {
    nodes: Vec<TrieNode>,
}

impl Solution {
    fn new() -> Self {
        Solution {
            nodes: vec![TrieNode::new()], // root at index 0
        }
    }

    fn insert(&mut self, r: &str) {
        let mut idx = 0;
        for ch in r.bytes() {
            let c = (ch - b'a') as usize;
            if self.nodes[idx].children[c].is_none() {
                self.nodes.push(TrieNode::new());
                let new_idx = self.nodes.len() - 1;
                self.nodes[idx].children[c] = Some(new_idx);
            }
            idx = self.nodes[idx].children[c].unwrap();
        }
        self.nodes[idx].is_end = true;
    }

    // Return the shortest root prefix of `word`, or `word` itself if none.
    fn shortest_root<'a>(&self, word: &'a str) -> &'a str {
        let bytes = word.as_bytes();
        let mut idx = 0;
        for i in 0..bytes.len() {
            let c = (bytes[i] - b'a') as usize;
            match self.nodes[idx].children[c] {
                None => break,     // no further root possible
                Some(child_idx) => {
                    idx = child_idx;
                    if self.nodes[idx].is_end {
                        return &word[..i + 1];  // shortest root found
                    }
                }
            }
        }
        word
    }

    fn replace_words(&mut self, dictionary: Vec<String>, sentence: String) -> String {
        for r in &dictionary {
            self.insert(r);
        }
        // Split sentence by spaces and replace each word.
        sentence
            .split_whitespace()
            .map(|w| self.shortest_root(w).to_string())
            .collect::<Vec<_>>()
            .join(" ")
    }
}
```

**Key insight:** because we return at the *first* `is_end` we hit while walking down, we always get the **shortest** root (e.g. given roots `"cat"` and `"cattle"`, the word `"cattle"` is replaced by `"cat"`).

**Complexity:** **O(sum of root lengths)** to build, **O(sum of word lengths)** to replace.

---

## Problem 7 — Word Search II (Trie + DFS on board) · LC 212 · Hard

Find all dictionary words that exist on a 2D board (adjacent horizontally/vertically, no cell reuse within one word). The naive approach runs a separate DFS per word: **O(words · M·N·4^L)**. The trie approach builds **one** trie of all words and DFSes the board once, following the trie in lockstep with the board path.

Two crucial optimizations:
1. **Store the complete word at its leaf node** (`node.word`) so we never build a string during the hot DFS — when `node.word` is non-empty we have found that exact word.
2. **Prune found words** by calling `node.word.clear()` after collecting them, so duplicates aren't re-reported and dead branches shrink.

```rust
struct TrieNode {
    children: [Option<usize>; 26],
    word: String,   // the full word ending here, or empty string
}

impl TrieNode {
    fn new() -> Self {
        TrieNode {
            children: [None; 26],
            word: String::new(),
        }
    }
}

struct Solution;

impl Solution {
    fn build_trie(words: &[String]) -> Vec<TrieNode> {
        let mut nodes = vec![TrieNode::new()];
        for w in words {
            let mut idx = 0;
            for ch in w.bytes() {
                let c = (ch - b'a') as usize;
                if nodes[idx].children[c].is_none() {
                    nodes.push(TrieNode::new());
                    let new_idx = nodes.len() - 1;
                    nodes[idx].children[c] = Some(new_idx);
                }
                idx = nodes[idx].children[c].unwrap();
            }
            nodes[idx].word = w.clone();    // store full word at leaf — no string building needed
        }
        nodes
    }

    fn dfs(
        board: &mut Vec<Vec<char>>,
        r: usize,
        col: usize,
        node_idx: usize,
        nodes: &mut Vec<TrieNode>,
        result: &mut Vec<String>,
    ) {
        let ch = board[r][col];
        if ch == '#' { return; }                        // already visited on this path
        let c = (ch as u8 - b'a') as usize;
        let next_opt = nodes[node_idx].children[c];
        let next_idx = match next_opt {
            None => return,                             // prune: no word continues here
            Some(idx) => idx,
        };

        if !nodes[next_idx].word.is_empty() {
            result.push(nodes[next_idx].word.clone());
            nodes[next_idx].word.clear();               // prune: don't report twice
        }

        board[r][col] = '#';                            // mark visited
        let rows = board.len();
        let cols = board[0].len();
        let dirs: [(i32, i32); 4] = [(1, 0), (-1, 0), (0, 1), (0, -1)];
        for (dr, dc) in dirs {
            let nr = r as i32 + dr;
            let nc = col as i32 + dc;
            if nr >= 0 && nr < rows as i32 && nc >= 0 && nc < cols as i32 {
                Self::dfs(board, nr as usize, nc as usize, next_idx, nodes, result);
            }
        }
        board[r][col] = ch;                             // restore (backtrack)
    }

    fn find_words(mut board: Vec<Vec<char>>, words: Vec<String>) -> Vec<String> {
        let mut nodes = Self::build_trie(&words);
        let mut result = Vec::new();
        let rows = board.len();
        let cols = board[0].len();
        for r in 0..rows {
            for col in 0..cols {
                Self::dfs(&mut board, r, col, 0, &mut nodes, &mut result);
            }
        }
        result
    }
}
```

**Why this crushes the naive approach:** all words share the trie, so common prefixes are explored once. The board DFS is bounded by the trie depth (longest word) and dead-ends are cut immediately when `nodes[node_idx].children[c].is_none()`. See the complexity comparison in [Interview Problems/Amazon.md](../Interview%20Problems/Amazon.md).

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
    - From `(1,1)`, neighbor `(2,1)='h'`: node after `oat` has child `h` → `oath`. `!nodes[next_idx].word.is_empty()` → **add "oath"**, then `nodes[next_idx].word.clear()` (pruned).
- `"pea"` has no `p` anywhere reachable → never explored beyond the first cell mismatch.
- `"eat"`: starts at `(1,0)='e'`. DFS reaches `eat` and adds it.
- `"rain"`: `r` at `(2,3)`, but no path spells `rain` on adjacent cells → trie walk dead-ends → pruned early.

Result: `["oath", "eat"]`. The `'#'` marking prevents reusing a cell; restoring it on backtrack keeps the board valid for other start cells; `nodes[next_idx].word.clear()` stops `"oath"` from being added again if reachable another way.

---

## Recognition Signals Recap

- `.` / `?` wildcard in a dictionary search → DFS branching all children (Problem 3).
- "build word one letter at a time, every prefix valid" → `is_end`-gated DFS (Problem 4).
- "replace by shortest/longest root prefix" → walk to first `is_end` (Problem 5).
- "find all dictionary words on a grid" → trie + board DFS with pruning (Problem 7).

See also: [Trie Fundamentals](Trie%20Fundamentals.md), [Word Dictionary design](../Design%20Data%20Structure%20Problems/Word%20Dictionary.md).

---

> **Last Updated:** 2026-06-26
