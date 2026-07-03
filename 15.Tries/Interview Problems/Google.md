> **Topic:** [Tries](../README.md) · **Interview Problems**

# Google — Interview Problems (Tries)

Google Trie questions reward depth: not "can you build a Trie" but "can you reason about *why* a shared structure beats brute force, and analyze the wildcard / bit-search complexity precisely." The three problems below are staples in L4/L5 loops. Background: [Trie Fundamentals](../Patterns/Trie%20Fundamentals.md), [Trie with Search Variants](../Patterns/Trie%20with%20Search%20Variants.md), [Bitwise Trie (XOR)](../Patterns/Bitwise%20Trie%20(XOR).md).

---

## (a) Word Search II — LC 212

**Problem.** Given an `m x n` board and a list of `words`, return every word that can be formed by a path of adjacent (up/down/left/right) cells with no cell reused within a word.

**The full optimization story.** A first instinct is: for each word, run a board DFS. That is `O(W · M·N · 4^L)` — the number of words `W` multiplies everything. The insight Google is probing for is that **the words share prefixes**, and a brute-force per-word search redoes that shared prefix exploration `W` times. Collapsing all words into **one Trie** and walking the board *once* lets the shared prefixes be explored exactly once.

Three concrete optimizations make this efficient:

1. **Store the word at the Trie node.** Instead of `bool is_end`, store a `String word` at terminal nodes. When the board DFS reaches a node whose `!node.word.is_empty()`, we have a full match and can collect it without reconstructing the path from the board.
2. **Clear the word after it is found** (`node.word.clear()`). This deduplicates (the same word reachable via two paths is reported once) and signals the word is consumed.
3. **Remove/prune dead leaves.** After exploring a node, if it has no children left and is not itself a word, it is a dead end; detaching it from its parent shrinks the Trie so future board paths bail out earlier. The `dfs` below carries the parent so it can prune.

Why **one shared Trie beats per-word search**: the per-word approach pays `4^L` *per word*; the shared Trie pays the board-exploration cost once and prunes branches the instant no word continues with the current prefix. The dominant term drops from `O(W · M·N · 4^L)` to `O(M·N · 4^Lmax)`.

```rust
struct TrieNode {
    children: [Option<usize>; 26],
    word: String,  // full word stored at terminal node, else empty
}

struct Solution;

impl Solution {
    fn build_trie(words: &[String]) -> Vec<TrieNode> {
        let mut nodes: Vec<TrieNode> = vec![TrieNode {
            children: [None; 26],
            word: String::new(),
        }];
        for w in words {
            let mut idx = 0;
            for ch in w.chars() {
                let ci = (ch as u8 - b'a') as usize;
                if nodes[idx].children[ci].is_none() {
                    nodes[idx].children[ci] = Some(nodes.len());
                    nodes.push(TrieNode {
                        children: [None; 26],
                        word: String::new(),
                    });
                }
                idx = nodes[idx].children[ci].unwrap();
            }
            nodes[idx].word = w.clone();
        }
        nodes
    }

    fn dfs(
        board: &mut Vec<Vec<char>>,
        r: i32,
        c: i32,
        parent_idx: usize,
        nodes: &mut Vec<TrieNode>,
        result: &mut Vec<String>,
    ) {
        let m = board.len() as i32;
        let n = board[0].len() as i32;
        if r < 0 || r >= m || c < 0 || c >= n {
            return;
        }
        let ch = board[r as usize][c as usize];
        if ch == '#' {
            return;  // on the current path already
        }
        let ci = (ch as u8 - b'a') as usize;
        let node_idx = match nodes[parent_idx].children[ci] {
            Some(idx) => idx,
            None => return,  // no word continues -> prune
        };

        // collect word if terminal; std::mem::take replaces with empty String (dedup)
        let word = std::mem::take(&mut nodes[node_idx].word);
        if !word.is_empty() {
            result.push(word);
        }

        board[r as usize][c as usize] = '#';
        Self::dfs(board, r + 1, c, node_idx, nodes, result);
        Self::dfs(board, r - 1, c, node_idx, nodes, result);
        Self::dfs(board, r, c + 1, node_idx, nodes, result);
        Self::dfs(board, r, c - 1, node_idx, nodes, result);
        board[r as usize][c as usize] = ch;

        // prune dead leaf: no children left and not a word -> detach
        let is_dead = nodes[node_idx].children.iter().all(|c| c.is_none())
            && nodes[node_idx].word.is_empty();
        if is_dead {
            nodes[parent_idx].children[ci] = None;
        }
    }

    pub fn find_words(board: &mut Vec<Vec<char>>, words: &[String]) -> Vec<String> {
        let mut nodes = Self::build_trie(words);
        let mut result = Vec::new();
        let m = board.len();
        let n = board[0].len();
        for r in 0..m {
            for c in 0..n {
                Self::dfs(board, r as i32, c as i32, 0, &mut nodes, &mut result);
            }
        }
        result
    }
}
```

**Complexity.** Building the Trie is `O(sum of word lengths)`. The search is `O(M·N · 4^Lmax)` time, where `Lmax` is the longest word, because each cell seeds a DFS bounded by the Trie depth and pruned aggressively. Space is `O(sum of word lengths)` for the Trie, shrinking as leaves are pruned.

**Follow-ups.**
- *Return paths, not just words:* track `(r,c)` on a path list.
- *Words can reuse cells:* drop the `'#'` marking — but then termination requires bounding by max word length.

---

## (b) Add and Search Word / Word Dictionary — LC 211

**Problem.** Design `WordDictionary` supporting `addWord(word)` and `search(word)`, where `search` may contain `'.'` as a wildcard that matches any single letter.

**Wildcard DFS.** A literal character walks deterministically to one child. A `'.'` must try **all 26** children, recursing into each — that branching is what makes wildcards expensive. We recurse from the current Trie node and string index; on `'.'` we loop over every non-null child.

```rust
struct TrieNode {
    children: [Option<usize>; 26],
    is_end: bool,
}

struct WordDictionary {
    nodes: Vec<TrieNode>,
}

impl WordDictionary {
    fn new() -> Self {
        WordDictionary {
            nodes: vec![TrieNode {
                children: [None; 26],
                is_end: false,
            }],
        }
    }

    fn add_word(&mut self, word: &str) {
        let mut idx = 0;
        for ch in word.chars() {
            let ci = (ch as u8 - b'a') as usize;
            if self.nodes[idx].children[ci].is_none() {
                self.nodes[idx].children[ci] = Some(self.nodes.len());
                self.nodes.push(TrieNode {
                    children: [None; 26],
                    is_end: false,
                });
            }
            idx = self.nodes[idx].children[ci].unwrap();
        }
        self.nodes[idx].is_end = true;
    }

    fn dfs(&self, word: &[u8], i: usize, node_idx: usize) -> bool {
        if i == word.len() {
            return self.nodes[node_idx].is_end;
        }
        let c = word[i] as char;
        if c == '.' {
            for ci in 0..26 {
                if let Some(child_idx) = self.nodes[node_idx].children[ci] {
                    if self.dfs(word, i + 1, child_idx) {
                        return true;
                    }
                }
            }
            false
        } else {
            let ci = (c as u8 - b'a') as usize;
            match self.nodes[node_idx].children[ci] {
                Some(child_idx) => self.dfs(word, i + 1, child_idx),
                None => false,
            }
        }
    }

    fn search(&self, word: &str) -> bool {
        self.dfs(word.as_bytes(), 0, 0)
    }
}
```

**Wildcard complexity.** `add_word` is `O(L)`. For `search`, let `L` be the query length and `k` the number of `'.'` characters. Each `'.'` branches into up to 26 recursive calls, and the wildcards compound multiplicatively, giving a worst case of **`O(26^k · L)`**. Intuition: a query like `".....`" (all dots) forces exploration of every length-`L` path in the Trie, of which there can be up to `26^L`; with `k` dots the branching factor is `26` raised to the number of wildcards, and the `L` factor accounts for walking the depth. With no dots (`k = 0`) it collapses to `O(L)`.

**Follow-ups.**
- *Bound the wildcard cost:* if many queries are all-dots, group words by length and store per-length Tries.
- *Leading wildcards / suffix queries:* add a reversed Trie or a suffix structure.

---

## (c) Maximum XOR of Two Numbers in an Array — LC 421

**Problem.** Given `nums: &[i32]`, return the maximum value of `nums[i] ^ nums[j]`.

**Binary (bitwise) Trie + greedy.** Build a Trie where each node has exactly **2 children** (bit `0` and bit `1`), inserting each number from its most significant bit down. To maximize XOR for a given number, walk the Trie from the MSB and at each bit **greedily prefer the opposite bit** (`want = bit ^ 1`): if a number with the opposite bit exists down this path, taking it sets that XOR bit to 1, which is worth more than every lower bit combined. Constraints fit in 31 bits, so we start at `HIGH_BIT = 30`.

```rust
struct TrieNode {
    children: [Option<usize>; 2],
}

struct Solution;

impl Solution {
    const HIGH_BIT: i32 = 30;

    fn insert(nodes: &mut Vec<TrieNode>, num: i32) {
        let mut idx = 0;
        for i in (0..=Self::HIGH_BIT).rev() {
            let bit = ((num >> i) & 1) as usize;
            if nodes[idx].children[bit].is_none() {
                nodes[idx].children[bit] = Some(nodes.len());
                nodes.push(TrieNode { children: [None; 2] });
            }
            idx = nodes[idx].children[bit].unwrap();
        }
    }

    fn max_xor_with(nodes: &[TrieNode], num: i32) -> i32 {
        let mut idx = 0;
        let mut xor_val = 0;
        for i in (0..=Self::HIGH_BIT).rev() {
            let bit = ((num >> i) & 1) as usize;
            let want = bit ^ 1;  // prefer the opposite bit
            if nodes[idx].children[want].is_some() {
                xor_val |= 1 << i;  // this bit of the XOR becomes 1
                idx = nodes[idx].children[want].unwrap();
            } else {
                idx = nodes[idx].children[bit].unwrap();  // forced to take the same bit
            }
        }
        xor_val
    }

    pub fn find_maximum_xor(nums: &[i32]) -> i32 {
        let mut nodes: Vec<TrieNode> = vec![TrieNode { children: [None; 2] }];
        for &num in nums {
            Self::insert(&mut nodes, num);
        }
        nums.iter()
            .map(|&num| Self::max_xor_with(&nodes, num))
            .max()
            .unwrap_or(0)
    }
}
```

**Why greedy is correct.** XOR is maximized lexicographically by bit significance: a 1 in a higher bit outweighs any combination of lower bits (`2^i > 2^i - 1 = sum of all lower bits`). So at each level, if we *can* make the current XOR bit 1 by choosing a stored number with the opposite bit, we must — no future choice could compensate for giving up a higher bit. The Trie lets us check "does a number with the opposite bit exist on this prefix path?" in `O(1)` per level, and the greedy choice is always locally and globally optimal.

**Complexity.** Let `B = 31` be the bit width and `n = nums.len()`. Insert is `O(B)` per number, search is `O(B)` per number, so total `O(n · B)` time and `O(n · B)` space for the Trie. This beats the naive `O(n^2)` pairwise comparison.

Cross-link: this is the same machinery used in [Bit Search and Trie](../../6.Bit_Manipulation/Patterns/Bit%20Search%20and%20Trie.md).

**Follow-ups.**
- *Maximum XOR with a query upper bound (`nums[j] <= limit`)* — LC 1707; add insertion-ordering / offline sorting.
- *Maximum XOR in a subarray range* — persistent / segment-indexed bit Tries.
- *There is also an `O(n · B)` `HashMap`-prefix solution* that avoids building an explicit Trie; be ready to compare.

---

## L4 vs L5 expectations

- **L4 (mid):** Get to a correct, clean Trie solution. For 212, recognize the shared-Trie idea and implement `node.word` + mark-visited correctly. For 211, implement the wildcard DFS without bugs. For 421, derive the greedy bit-Trie and explain the MSB-first reasoning. State the complexity correctly.
- **L5 (senior):** Drive the *trade-off* conversation unprompted. For 212, propose and implement leaf pruning, discuss memory reclamation and how `Lmax` (not `W`) bounds the exponent. For 211, quantify `O(26^k · L)` and propose mitigations (per-length Tries) for skewed wildcard workloads. For 421, compare the Trie vs `HashMap`-prefix approaches, prove greedy optimality from the bit-significance argument, and discuss the constrained/range variants. L5 is judged on communicating *why*, edge-case rigor, and scaling beyond the literal prompt.

---

> **Last Updated:** 2026-06-26
