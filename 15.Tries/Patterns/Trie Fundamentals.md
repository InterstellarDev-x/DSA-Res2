> **Topic:** [Tries](../README.md) · **Pattern 1 of 3**

# Trie Fundamentals

## Core Concept

A trie stores strings as paths in a tree. Each node owns up to 26 children (one per lowercase letter), and a boolean `is_end` marks where a complete word terminates. Three operations form the backbone:

- **insert(word):** walk/create one node per character, set `is_end` at the last node.
- **search(word):** walk the path; return `true` only if the path exists **and** the final node has `is_end == true`.
- **starts_with(prefix):** walk the path; return `true` if the path simply exists (no `is_end` check).

Every operation is **O(L)** in the length of the key, completely independent of the number of stored words.

**Recognition signals:** "implement a prefix tree", "support startsWith / hasPrefix", "autocomplete", "count words with a given prefix", "dictionary that grows dynamically".

---

## Problem 1 — Implement Trie (Prefix Tree) · LC 208 · Medium

```rust
#[derive(Debug)]
struct TrieNode {
    children: [Option<Box<TrieNode>>; 26],
    is_end: bool,
}

impl TrieNode {
    fn new() -> Self {
        TrieNode {
            children: Default::default(),
            is_end: false,
        }
    }
}

struct Trie {
    root: TrieNode,
}

impl Trie {
    fn new() -> Self {
        Trie { root: TrieNode::new() }
    }

    fn insert(&mut self, word: &str) {
        let mut node = &mut self.root;
        for c in word.chars() {
            let idx = (c as u8 - b'a') as usize;
            if node.children[idx].is_none() {
                node.children[idx] = Some(Box::new(TrieNode::new()));
            }
            node = node.children[idx].as_mut().unwrap().as_mut();
        }
        node.is_end = true;
    }

    fn search(&self, word: &str) -> bool {
        self.find(word).map_or(false, |n| n.is_end)  // path AND terminal flag
    }

    fn starts_with(&self, prefix: &str) -> bool {
        self.find(prefix).is_some()                   // path is enough
    }

    // Walk the trie following the string; return the final node or None.
    fn find(&self, s: &str) -> Option<&TrieNode> {
        let mut node = &self.root;
        for c in s.chars() {
            let idx = (c as u8 - b'a') as usize;
            node = node.children[idx].as_deref()?;
        }
        Some(node)
    }
}
```

**Complexity:** `insert` / `search` / `starts_with` are all **O(L)** time, **O(1)** auxiliary. Space for the whole structure is **O(26 · total characters inserted)** in the worst case (no shared prefixes).

---

## Problem 2 — Implement Trie II (count prefixes / word count) · Medium

Trie II turns the trie into a **multiset**: the same word can be inserted multiple times, and we must answer "how many times is exactly this word stored?" and "how many stored words have this prefix?". We add two integer counters per node and an `erase` that decrements them.

```rust
#[derive(Debug)]
struct TrieNodeII {
    children: [Option<Box<TrieNodeII>>; 26],
    count_word: i32,    // words ending exactly at this node
    count_prefix: i32,  // words passing through this node (prefix)
}

impl TrieNodeII {
    fn new() -> Self {
        TrieNodeII {
            children: Default::default(),
            count_word: 0,
            count_prefix: 0,
        }
    }
}

struct TrieII {
    root: TrieNodeII,
}

impl TrieII {
    fn new() -> Self {
        TrieII { root: TrieNodeII::new() }
    }

    fn insert(&mut self, word: &str) {
        let mut node = &mut self.root;
        for c in word.chars() {
            let idx = (c as u8 - b'a') as usize;
            if node.children[idx].is_none() {
                node.children[idx] = Some(Box::new(TrieNodeII::new()));
            }
            node = node.children[idx].as_mut().unwrap().as_mut();
            node.count_prefix += 1;           // every node on the path gains a prefix count
        }
        node.count_word += 1;                 // only the terminal node gains a word count
    }

    fn count_words_equal_to(&self, word: &str) -> i32 {
        self.find(word).map_or(0, |n| n.count_word)
    }

    fn count_words_starting_with(&self, prefix: &str) -> i32 {
        self.find(prefix).map_or(0, |n| n.count_prefix)
    }

    // Decrement counters along the path; assumes the word was inserted before.
    fn erase(&mut self, word: &str) {
        if self.count_words_equal_to(word) == 0 { return; }  // guard: nothing to erase
        let mut node = &mut self.root;
        for c in word.chars() {
            let idx = (c as u8 - b'a') as usize;
            node = node.children[idx].as_mut().unwrap().as_mut();
            node.count_prefix -= 1;
        }
        node.count_word -= 1;
    }

    fn find(&self, s: &str) -> Option<&TrieNodeII> {
        let mut node = &self.root;
        for c in s.chars() {
            let idx = (c as u8 - b'a') as usize;
            node = node.children[idx].as_deref()?;
        }
        Some(node)
    }
}
```

**Why two counters?** `count_word` distinguishes the multiset multiplicity of an *exact* word; `count_prefix` answers prefix queries in O(L) without traversing subtrees. `erase` simply walks the same path decrementing both — note we keep the nodes (lazy deletion) rather than physically unlinking them, which is correct and far simpler. A physical delete is a follow-up (see [Implement Trie.md](../Design%20Data%20Structure%20Problems/Implement%20Trie.md)).

**Complexity:** all four operations are **O(L)**.

---

## HashMap-based children variant

When the alphabet is large, sparse, or non-Latin (Unicode, mixed case, arbitrary symbols), a fixed `[26]` array either wastes memory or simply cannot index the character. Swap it for a map:

```rust
use std::collections::HashMap;

#[derive(Debug)]
struct TrieNodeMap {
    children: HashMap<char, Box<TrieNodeMap>>,
    is_end: bool,
}

impl TrieNodeMap {
    fn new() -> Self {
        TrieNodeMap {
            children: HashMap::new(),
            is_end: false,
        }
    }
}

struct TrieMap {
    root: TrieNodeMap,
}

impl TrieMap {
    fn new() -> Self {
        TrieMap { root: TrieNodeMap::new() }
    }

    fn insert(&mut self, word: &str) {
        let mut node = &mut self.root;
        for c in word.chars() {
            node = node.children.entry(c).or_insert_with(|| Box::new(TrieNodeMap::new()));
        }
        node.is_end = true;
    }

    fn search(&self, word: &str) -> bool {
        self.find(word).map_or(false, |n| n.is_end)
    }

    fn starts_with(&self, prefix: &str) -> bool {
        self.find(prefix).is_some()
    }

    fn find(&self, s: &str) -> Option<&TrieNodeMap> {
        let mut node = &self.root;
        for c in s.chars() {
            node = node.children.get(&c)?;
        }
        Some(node)
    }
}
```

`entry(c).or_insert_with(...)` is the idiomatic "get-or-create child" for the `HashMap` variant. The trade-off table lives in [Implement Trie.md](../Design%20Data%20Structure%20Problems/Implement%20Trie.md) and the [README anatomy section](../README.md#triechildren-array26-vs-hashmap).

---

## Dry Run — insert "app", "apple"; search "app", "appl", "apple"

Start: `root` with all children null.

**insert("app"):**
```
root --a--> n1 --p--> n2 --p--> n3(is_end=true)
```
**insert("apple"):** reuses `a-p-p`, extends with `l-e`:
```
root --a--> n1 --p--> n2 --p--> n3(is_end=true) --l--> n4 --e--> n5(is_end=true)
```

**search("app"):** walk `a→p→p` reaches `n3`; `n3.is_end == true` → **true**.

**search("appl"):** walk `a→p→p→l` reaches `n4`; `n4.is_end == false` → **false** (it's a prefix, not a stored word — this is exactly why `is_end` exists).

**search("apple"):** walk to `n5`; `n5.is_end == true` → **true**.

**starts_with("appl"):** path `a→p→p→l` exists → **true** (no `is_end` check).

---

## Recognition Signals Recap

- "Prefix", "starts with", "autocomplete", "dictionary".
- Need *both* exact-word and prefix queries fast → trie with `is_end` (+ counters for II).
- Many words sharing prefixes → trie compresses shared work; a `HashSet` cannot answer prefix queries.

See also: [Trie with Search Variants](Trie%20with%20Search%20Variants.md), [Bitwise Trie (XOR)](Bitwise%20Trie%20(XOR).md).

---

> **Last Updated:** 2026-06-26
