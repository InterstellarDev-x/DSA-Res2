> **Topic:** [Tries](../README.md) · **Interview Problems**

# Amazon — Interview Problems (Tries)

Amazon loves Tries because they map directly onto real product surfaces: search autocomplete, product catalog prefix matching, and spell/word filtering. These three problems are the most commonly asked Trie questions in Amazon loops. See the building blocks in [Trie Fundamentals](../Patterns/Trie%20Fundamentals.md) and [Trie with Search Variants](../Patterns/Trie%20with%20Search%20Variants.md).

---

## (a) Implement Trie — LC 208

**Problem.** Implement a `Trie` (prefix tree) supporting `insert(word)`, `search(word)` (exact match), and `starts_with(prefix)` (any word with this prefix exists).

**Framing it as an autocomplete backend.** When a customer types into the Amazon search box, the backend must answer "is `lapt` a prefix of any product term?" in time proportional to the length of what the user typed, not the size of the catalog. A Trie gives exactly that: `starts_with` is `O(L)` regardless of how many millions of products exist. `search` is the "did they type a complete known term" check, and `insert` is how we load the dictionary of search terms at index-build time.

**Design decision: `children: [i32; 26]` (index-based array) vs `HashMap<char, usize>`.**

| Aspect | `children: [i32; 26]` (index-based array) | `HashMap<char, usize>` |
|---|---|---|
| Lookup | O(1) array index, no hashing | O(1) amortized, but with hashing overhead |
| Memory | 26 integer slots per node even if sparse | Only allocates entries that exist |
| Alphabet | Fixed lowercase `a–z` | Arbitrary / Unicode |
| Cache behavior | Contiguous, cache friendly | Pointer-chasing |

For this problem the constraint is lowercase English letters, so the array form is preferred: it is faster and simpler, and the `26`-wide fan-out is acceptable. If the alphabet were Unicode or the tree extremely sparse, the HashMap form would win on memory. **Say this trade-off out loud in the interview** — that is the point of the question.

```rust
struct TrieNode {
    children: [i32; 26], // -1 means no child
    is_end: bool,
}

impl TrieNode {
    fn new() -> Self {
        TrieNode {
            children: [-1; 26],
            is_end: false,
        }
    }
}

struct Trie {
    nodes: Vec<TrieNode>,
}

impl Trie {
    fn new() -> Self {
        Trie { nodes: vec![TrieNode::new()] } // node 0 is root
    }

    fn find(&self, s: &str) -> Option<usize> {
        let mut idx = 0usize;
        for ch in s.bytes() {
            let ci = (ch - b'a') as usize;
            let next = self.nodes[idx].children[ci];
            if next == -1 {
                return None;
            }
            idx = next as usize;
        }
        Some(idx)
    }

    fn insert(&mut self, word: &str) {
        let mut idx = 0usize;
        for ch in word.bytes() {
            let ci = (ch - b'a') as usize;
            if self.nodes[idx].children[ci] == -1 {
                self.nodes[idx].children[ci] = self.nodes.len() as i32;
                self.nodes.push(TrieNode::new());
            }
            idx = self.nodes[idx].children[ci] as usize;
        }
        self.nodes[idx].is_end = true;
    }

    fn search(&self, word: &str) -> bool {
        self.find(word).map_or(false, |idx| self.nodes[idx].is_end)
    }

    fn starts_with(&self, prefix: &str) -> bool {
        self.find(prefix).is_some()
    }
}
```

**Reasoning.** `search` and `starts_with` differ only by the final `is_end` check; factoring the walk into a private `find` removes duplication. The `is_end` flag is what lets `"app"` and `"apple"` coexist — without it, inserting `"apple"` would make `search("app")` falsely return true.

**Complexity.** Let `L` be the word/prefix length. `insert`, `search`, `starts_with` are each `O(L)` time. Space is `O(total characters inserted)` in the worst case, i.e. `O(N · L)` for `N` words with no shared prefixes.

**Follow-ups.**
- *Add autocomplete (`Vec<String> suggestions(prefix)`):* walk to the prefix node, then DFS collecting words. To bound it, cap at top-k.
- *Make it case-insensitive / support digits:* widen the array or switch to a HashMap.
- *Concurrency:* the read-heavy autocomplete case can use a copy-on-write or an immutable Trie rebuilt at index time.

---

## (b) Word Search II — LC 212

**Problem.** Given an `m x n` board of characters and a list of `words`, return all words on the board. A word is formed from sequentially adjacent (horizontally/vertically) cells, and a cell may not be reused within a single word.

**Key idea.** Instead of running a board search for every word independently, build **one** Trie from all words and DFS the board once, walking the Trie in lockstep with the path. Shared prefixes are explored only once. We store the full word string at the terminal node so that when we reach an `is_end`, we can collect the answer without reconstructing the path.

```rust
struct TrieNode {
    children: [i32; 26], // -1 means no child
    word: String,        // non-empty only at a terminal node; holds the full word
}

impl TrieNode {
    fn new() -> Self {
        TrieNode {
            children: [-1; 26],
            word: String::new(),
        }
    }
}

struct Solution;

impl Solution {
    fn build_trie(words: &[String]) -> Vec<TrieNode> {
        let mut nodes = vec![TrieNode::new()]; // node 0 is root
        for w in words {
            let mut idx = 0usize;
            for ch in w.bytes() {
                let ci = (ch - b'a') as usize;
                if nodes[idx].children[ci] == -1 {
                    nodes[idx].children[ci] = nodes.len() as i32;
                    nodes.push(TrieNode::new());
                }
                idx = nodes[idx].children[ci] as usize;
            }
            nodes[idx].word = w.clone();
        }
        nodes
    }

    fn dfs(
        board: &mut Vec<Vec<char>>,
        r: i32,
        c: i32,
        node_idx: usize,
        trie: &mut Vec<TrieNode>,
        result: &mut Vec<String>,
    ) {
        let m = board.len() as i32;
        let n = board[0].len() as i32;
        if r < 0 || r >= m || c < 0 || c >= n {
            return;
        }
        let ch = board[r as usize][c as usize];
        if ch == '#' {
            return; // already on current path
        }
        let ci = (ch as u8 - b'a') as usize;
        let next_idx = trie[node_idx].children[ci];
        if next_idx == -1 {
            return; // no word continues with this prefix -> prune
        }
        let next_idx = next_idx as usize;
        if !trie[next_idx].word.is_empty() {
            let word = std::mem::take(&mut trie[next_idx].word);
            result.push(word); // prune: avoid adding duplicates
        }

        board[r as usize][c as usize] = '#'; // mark visited
        for (dr, dc) in [(1i32, 0i32), (-1, 0), (0, 1), (0, -1)] {
            Self::dfs(board, r + dr, c + dc, next_idx, trie, result);
        }
        board[r as usize][c as usize] = ch; // restore
    }

    pub fn find_words(board: &mut Vec<Vec<char>>, words: &[String]) -> Vec<String> {
        let mut trie = Self::build_trie(words);
        let mut result = Vec::new();
        let m = board.len();
        let n = board[0].len();
        for r in 0..m {
            for c in 0..n {
                Self::dfs(board, r as i32, c as i32, 0, &mut trie, &mut result);
            }
        }
        result
    }
}
```

**The mark-visited trick.** A single word cannot reuse a cell, so during one DFS path we temporarily overwrite `board[r][c]` with `'#'` before recursing and restore the original character afterward. This in-place marking avoids allocating a separate `Vec<Vec<bool>> visited` per call and is `O(1)` per cell. The check `if ch == '#' { return; }` is what enforces "no cell reused on the current path."

**The pruning trick.** Taking `next.word` via `std::mem::take` after collecting a word means we never report the same word twice and we let the Trie shrink as words are found. A stronger variant also prunes dead leaf nodes (children that lead nowhere) to keep the search tree small — see the Google deep dive for that optimization.

**Complexity comparison.**

| Approach | Time | Why |
|---|---|---|
| Naive: DFS per word | `O(W · M·N · 4^L)` | For each of `W` words, start a 4-direction DFS from every cell; each DFS branches 4 ways up to depth `L` |
| Trie + shared DFS | `O(M·N · 4^Lmax)` | Board is traversed once; the Trie collapses shared prefixes so each board path is explored a single time regardless of `W` |

Here `W` = number of words, `M·N` = board cells, `L` = word length, `Lmax` = longest word. The win is that the per-word factor `W` disappears: words sharing prefixes (`"oath"`, `"oat"`, `"oats"`) drive the same board exploration only once.

**Reasoning.** Storing `word` at the leaf (rather than `bool is_end`) lets us add the answer immediately on arrival — no path reconstruction. The board walk and Trie walk advance together: the Trie tells us "is there any word continuing with this prefix?"; the moment `trie[node_idx].children[ci]` is `-1` we stop, which is the pruning that makes this fast.

**Follow-ups.**
- *Memory:* prune empty subtrees to let the allocator reclaim nodes as words are found.
- *Very long words / huge board:* the exponential `4^Lmax` is unavoidable in the worst case, but Trie pruning makes the realistic case dramatically faster.
- *Diagonal moves / wrap-around:* change the neighbor offsets.

---

## (c) Replace Words — LC 648

**Problem.** Given a dictionary of `roots` and a `sentence`, replace every word in the sentence with the **shortest root** that is a prefix of it. If no root applies, keep the word.

**Shortest-root logic.** Build a Trie of the roots. For each word, walk character by character; the **first** time we hit a node with `is_end == true`, we have found the shortest root (we stop immediately). If we never hit an `is_end` (or the path breaks), keep the original word. Stopping at the first `is_end` is what guarantees "shortest" — a longer matching root would only be found if we kept walking, and we deliberately do not.

```rust
struct TrieNode {
    children: [i32; 26], // -1 means no child
    is_end: bool,
}

impl TrieNode {
    fn new() -> Self {
        TrieNode {
            children: [-1; 26],
            is_end: false,
        }
    }
}

struct Solution;

impl Solution {
    fn insert(nodes: &mut Vec<TrieNode>, word: &str) {
        let mut idx = 0usize;
        for ch in word.bytes() {
            let ci = (ch - b'a') as usize;
            if nodes[idx].children[ci] == -1 {
                nodes[idx].children[ci] = nodes.len() as i32;
                nodes.push(TrieNode::new());
            }
            idx = nodes[idx].children[ci] as usize;
        }
        nodes[idx].is_end = true;
    }

    fn shortest_root(nodes: &[TrieNode], word: &str) -> String {
        let mut idx = 0usize;
        for (i, ch) in word.bytes().enumerate() {
            let ci = (ch - b'a') as usize;
            let next = nodes[idx].children[ci];
            if next == -1 {
                return word.to_string(); // path breaks, no root prefix
            }
            idx = next as usize;
            if nodes[idx].is_end {
                return word[..i + 1].to_string(); // first (shortest) root
            }
        }
        word.to_string() // word itself never matched a root
    }

    pub fn replace_words(dictionary: &[String], sentence: &str) -> String {
        let mut nodes = vec![TrieNode::new()]; // node 0 is root
        for r in dictionary {
            Self::insert(&mut nodes, r);
        }

        // split sentence by spaces
        sentence
            .split_whitespace()
            .map(|w| Self::shortest_root(&nodes, w))
            .collect::<Vec<_>>()
            .join(" ")
    }
}
```

**Reasoning.** The `return` inside the loop the moment `nodes[idx].is_end` is true is the entire "shortest" guarantee. `word[..i + 1].to_string()` slices off the root prefix. If the path through the Trie breaks before any `is_end`, the word has no root prefix and is returned unchanged.

**Complexity.** Building the Trie is `O(sum of root lengths)`. Replacement is `O(total characters in the sentence)` since each character is walked at most once. Space is `O(sum of root lengths)`.

**Follow-ups.**
- *Longest root instead of shortest:* keep walking, remember the last `is_end` index, return that prefix.
- *Multiple roots, return all:* collect every `is_end` along the path.
- *Streaming sentence:* process word-by-word without splitting the whole string.

---

## Amazon Leadership Principles alignment

| Problem / Skill | Leadership Principle | One-line justification |
|---|---|---|
| Implement Trie as autocomplete backend (208) | Customer Obsession | `starts_with` is `O(L)` so search suggestions stay instant no matter how large the catalog grows. |
| Array[26] vs HashMap children trade-off | Invent and Simplify | Choosing the simplest structure that fits the alphabet, and justifying it, instead of over-engineering. |
| Word Search II naive-vs-Trie complexity analysis (212) | Dive Deep | Quantifying `O(W·M·N·4^L)` → `O(M·N·4^Lmax)` shows you understand *why* the optimization works, not just that it does. |
| Mark-visited + clear `node.word` via `std::mem::take` (212) | Insist on the Highest Standards | Correct in-place visited handling and dedup pruning produce clean, leak-free, bug-free code under pressure. |
| Shortest-root early return (648) | Bias for Action | Returning at the first `is_end` is the minimal correct move — no wasted traversal, ship the simplest correct path. |

---

> **Last Updated:** 2026-06-26
