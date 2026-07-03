> **Topic:** [Tries](../README.md) · **Interview Problems**

# Microsoft — Interview Problems (Tries)

Microsoft interviews emphasize complete, robust implementations — including the operations people skip, like a *physical* delete that reclaims memory. The three problems below cover the full Trie API, wildcard search, and the bitwise-Trie XOR pattern. References: [Trie Fundamentals](../Patterns/Trie%20Fundamentals.md), [Trie with Search Variants](../Patterns/Trie%20with%20Search%20Variants.md), [Bitwise Trie (XOR)](../Patterns/Bitwise%20Trie%20(XOR).md).

---

## (a) Implement Trie with Delete — LC 208 (extended)

**Problem.** Implement a `Trie` supporting `insert`, `search`, `startsWith`, and additionally a **physical** `delete(word)` that not only unmarks the word but also reclaims (detaches) nodes that become unnecessary.

**Design discussion.** There are two flavors of delete:

- **Logical delete:** just set `is_end = false`. Simple, but leaves orphan paths consuming memory; over time a long-lived Trie accumulates dead nodes.
- **Physical delete:** unmark the terminal, then walk back up pruning any node that (1) has no children and (2) is not itself the end of another word. We must *not* prune a node that is a prefix of another stored word (e.g. deleting `"apple"` must keep nodes shared with `"app"`).

The clean way to express physical delete is a recursive function that returns "should my parent detach me?". A node is safe to detach when it has no remaining children and is not a word end. The `is_empty()` helper checks "no children." Importantly we still recurse into the deepest node first, unmark `is_end` there, and prune on the way back up only while the detach condition holds.

```rust
#[derive(Debug)]
pub struct TrieNode {
    children: [Option<Box<TrieNode>>; 26],
    is_end: bool,
}

impl TrieNode {
    pub fn new() -> Self {
        TrieNode {
            children: Default::default(),
            is_end: false,
        }
    }

    fn is_empty(&self) -> bool {
        self.children.iter().all(|c| c.is_none())
    }
}

pub struct Trie {
    root: Box<TrieNode>,
}

impl Trie {
    pub fn new() -> Self {
        Trie { root: Box::new(TrieNode::new()) }
    }

    fn find(&self, s: &str) -> Option<&TrieNode> {
        let mut node = &*self.root;
        for c in s.chars() {
            let idx = (c as u8 - b'a') as usize;
            match node.children[idx].as_deref() {
                None => return None,
                Some(child) => node = child,
            }
        }
        Some(node)
    }

    // Recursively deletes `word` starting at depth `i` from `node`.
    // Returns true if `node` itself should be detached by its parent
    // (i.e. it has no children and is not the end of any word).
    fn delete_helper(node: &mut TrieNode, word: &[u8], i: usize) -> bool {
        if i == word.len() {
            if !node.is_end { return false; }   // word not present; nothing to remove
            node.is_end = false;               // unmark terminal
            return node.is_empty();            // detach only if it's now a dead leaf
        }

        let idx = (word[i] - b'a') as usize;
        if node.children[idx].is_none() {
            return false;
        }
        let should_detach = {
            let child = node.children[idx].as_deref_mut().unwrap();
            Self::delete_helper(child, word, i + 1)
        };

        if should_detach {
            node.children[idx] = None;    // prune the now-useless child
        }
        // this node may itself be detached if it is now an empty non-word
        node.is_empty() && !node.is_end
    }

    pub fn insert(&mut self, word: &str) {
        let mut node: &mut TrieNode = &mut self.root;
        for c in word.chars() {
            let idx = (c as u8 - b'a') as usize;
            if node.children[idx].is_none() {
                node.children[idx] = Some(Box::new(TrieNode::new()));
            }
            node = node.children[idx].as_deref_mut().unwrap();
        }
        node.is_end = true;
    }

    pub fn search(&self, word: &str) -> bool {
        self.find(word).map_or(false, |n| n.is_end)
    }

    pub fn starts_with(&self, prefix: &str) -> bool {
        self.find(prefix).is_some()
    }

    pub fn delete_word(&mut self, word: &str) {
        Self::delete_helper(&mut self.root, word.as_bytes(), 0);
    }
}
```

**Reasoning.** The recursion descends to the terminal node, clears `is_end`, then unwinds. At each level we detach the child only if the child reported it is a dead leaf, and we report ourselves detachable only if we have become an empty non-word node. This correctly preserves shared prefixes: when deleting `"apple"` while `"app"` is stored, recursion at the `"app"` node finds `is_end == true` (it is `"app"`'s terminal) so `is_empty() && !is_end` is false and pruning stops there. The root is never detached because the public `delete_word` ignores the top-level return value.

**Complexity.** `insert`, `search`, `starts_with` are `O(L)`. `delete_word` is `O(L)` time (one descent, one unwind) and `O(L)` recursion stack. Memory is actively reclaimed, unlike logical delete.

**Follow-ups.**
- *Reference counting:* if duplicate inserts are allowed, store a count instead of a boolean and only unmark when the count hits zero.
- *Iterative delete:* push visited nodes on a stack and prune in reverse to avoid recursion depth limits for very long words.

---

## (b) Add and Search Word / Word Dictionary — LC 211

**Problem.** Design `WordDictionary` with `addWord(word)` and `search(word)`, where `'.'` in a search matches any single letter.

**DFS recursion for the `'.'` wildcard.** `addWord` is a standard Trie insert. `search` is recursive over `(node, index)`:
- If we have consumed the whole query (`index == word.length()`), the answer is whether the current node is a word terminal (`node.is_end`).
- For a normal character, descend to exactly one child (`node.children[c - 'a']`) and recurse; if that child is null the match fails on this branch.
- For `'.'`, we cannot know which letter it stands for, so we try **every** non-null child and succeed if any recursive branch succeeds. This is where the search fans out — a single `'.'` turns one path into up to 26.

```rust
#[derive(Debug)]
pub struct TrieNode {
    children: [Option<Box<TrieNode>>; 26],
    is_end: bool,
}

impl TrieNode {
    pub fn new() -> Self {
        TrieNode {
            children: Default::default(),
            is_end: false,
        }
    }
}

pub struct WordDictionary {
    root: Box<TrieNode>,
}

impl WordDictionary {
    pub fn new() -> Self {
        WordDictionary { root: Box::new(TrieNode::new()) }
    }

    fn dfs(word: &[u8], i: usize, node: &TrieNode) -> bool {
        if i == word.len() { return node.is_end; }
        let c = word[i];
        if c == b'.' {
            for child in node.children.iter() {
                if let Some(child_node) = child.as_deref() {
                    if Self::dfs(word, i + 1, child_node) {
                        return true;  // any matching branch suffices
                    }
                }
            }
            return false;
        }
        let idx = (c - b'a') as usize;
        match node.children[idx].as_deref() {
            None => false,
            Some(child) => Self::dfs(word, i + 1, child),
        }
    }

    pub fn add_word(&mut self, word: &str) {
        let mut node: &mut TrieNode = &mut self.root;
        for c in word.chars() {
            let idx = (c as u8 - b'a') as usize;
            if node.children[idx].is_none() {
                node.children[idx] = Some(Box::new(TrieNode::new()));
            }
            node = node.children[idx].as_deref_mut().unwrap();
        }
        node.is_end = true;
    }

    pub fn search(&self, word: &str) -> bool {
        Self::dfs(word.as_bytes(), 0, &self.root)
    }
}
```

**Reasoning.** The base case `i == word.len()` returning `node.is_end` is what enforces *exact length* matching — reaching a node is not enough; it must terminate a stored word. The `'.'` branch short-circuits (`return true`) on the first success, so we do not explore unnecessary siblings once a match is found.

**Complexity.** `add_word` is `O(L)`. `search` is `O(L)` when there are no wildcards; with `k` dots it is worst case `O(26^k · L)` because each dot branches into up to 26 recursive paths. Space is `O(sum of word lengths)` for the Trie plus `O(L)` recursion depth.

**Follow-ups.**
- *Frequent all-dot queries:* bucket words by length to shrink the search space.
- *Wildcard `'*'` (zero or more):* requires a different recurrence allowing the index to stay or advance.

---

## (c) Maximum XOR of Two Numbers in an Array — LC 421

**Problem.** Given `Vec<i32>`, find the maximum `nums[i] ^ nums[j]`.

**Bitwise Trie construction, step by step.** We build a binary Trie where every node has exactly **2 children**, indexed by bit value `0` or `1`. The construction:

1. Fix a bit width. Values fit in 31 bits, so we iterate from the most significant bit `HIGH_BIT = 30` down to `0`. Processing MSB-first is essential — the XOR is maximized greedily from the highest bit.
2. For each number, start at the root. At bit position `i`, extract `bit = (num >> i) & 1`.
3. If `node.children[bit]` is `None`, create it. Move down: `node = node.children[bit]`.
4. After the lowest bit, the path from root to leaf encodes the number's bit pattern (MSB at depth 1).

So each root-to-leaf path is a number's binary representation. Numbers sharing a high-bit prefix share Trie nodes near the root.

To answer a query for a number `num`, walk the same way but at each bit prefer the **opposite** bit (`want = bit ^ 1`): if such a child exists, that bit of the XOR is `1` (set it with `xor_val |= (1 << i)`); otherwise follow the same bit.

```rust
#[derive(Debug)]
pub struct TrieNode {
    children: [Option<Box<TrieNode>>; 2],
}

impl TrieNode {
    pub fn new() -> Self {
        TrieNode {
            children: [None, None],
        }
    }
}

pub struct Solution;

impl Solution {
    const HIGH_BIT: i32 = 30;

    fn insert(root: &mut TrieNode, num: i32) {
        let mut node: &mut TrieNode = root;
        for i in (0..=Self::HIGH_BIT).rev() {
            let bit = ((num >> i) & 1) as usize;  // MSB-first
            if node.children[bit].is_none() {
                node.children[bit] = Some(Box::new(TrieNode::new()));
            }
            node = node.children[bit].as_deref_mut().unwrap();
        }
    }

    fn max_xor_with(root: &TrieNode, num: i32) -> i32 {
        let mut node = root;
        let mut xor_val = 0i32;
        for i in (0..=Self::HIGH_BIT).rev() {
            let bit = ((num >> i) & 1) as usize;
            let want = bit ^ 1;  // opposite bit maximizes this position
            if node.children[want].is_some() {
                xor_val |= 1 << i;
                node = node.children[want].as_deref().unwrap();
            } else {
                node = node.children[bit].as_deref().unwrap();
            }
        }
        xor_val
    }

    pub fn find_maximum_xor(nums: Vec<i32>) -> i32 {
        let mut root = TrieNode::new();
        for &num in &nums {
            Self::insert(&mut root, num);
        }

        let mut max_val = 0i32;
        for &num in &nums {
            max_val = max_val.max(Self::max_xor_with(&root, num));
        }
        max_val
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
