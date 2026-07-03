> **Topic:** [Tries](../README.md) · **Design Data Structure**

# Implement Trie (Prefix Tree)

LeetCode [208](https://leetcode.com/problems/implement-trie-prefix-tree/) · Medium · Pattern: [Trie Fundamentals](../Patterns/Trie%20Fundamentals.md)

## Problem

Design a trie supporting:
- `void insert(string word)`
- `bool search(string word)` — exact word present?
- `bool startsWith(string prefix)` — any word with this prefix?

## Full Solution (LC 208)

```rust
#[derive(Debug)]
struct TrieNode {
    children: [Option<Box<TrieNode>>; 26],
    is_end: bool,
}

impl TrieNode {
    fn new() -> Self {
        TrieNode {
            children: std::array::from_fn(|_| None),
            is_end: false,
        }
    }
}

struct Trie {
    root: Box<TrieNode>,
}

impl Trie {
    fn new() -> Self {
        Trie {
            root: Box::new(TrieNode::new()),
        }
    }

    fn find(&self, s: &str) -> Option<&TrieNode> {
        let mut node = &*self.root;
        for c in s.chars() {
            let idx = (c as u8 - b'a') as usize;
            match &node.children[idx] {
                None => return None,
                Some(child) => node = child,
            }
        }
        Some(node)
    }

    fn insert(&mut self, word: &str) {
        let mut node = &mut *self.root;
        for c in word.chars() {
            let idx = (c as u8 - b'a') as usize;
            if node.children[idx].is_none() {
                node.children[idx] = Some(Box::new(TrieNode::new()));
            }
            node = node.children[idx].as_mut().unwrap();
        }
        node.is_end = true;
    }

    fn search(&self, word: &str) -> bool {
        self.find(word).map_or(false, |node| node.is_end)
    }

    fn starts_with(&self, prefix: &str) -> bool {
        self.find(prefix).is_some()
    }
}
```

The only difference between `search` and `starts_with` is the `is_end` check: a word must *terminate* at the final node, a prefix only needs the path to *exist*.

## Trie II Extension — count words / prefixes, erase

Many follow-ups ask for multiset semantics: how many times was a word inserted, how many words share a prefix, and remove an instance.

```rust
#[derive(Debug)]
struct TrieNode {
    children: [Option<Box<TrieNode>>; 26],
    count_word: i32,    // words ending exactly here
    count_prefix: i32,  // words passing through here
}

impl TrieNode {
    fn new() -> Self {
        TrieNode {
            children: std::array::from_fn(|_| None),
            count_word: 0,
            count_prefix: 0,
        }
    }
}

struct TrieII {
    root: Box<TrieNode>,
}

impl TrieII {
    fn new() -> Self {
        TrieII {
            root: Box::new(TrieNode::new()),
        }
    }

    fn find(&self, s: &str) -> Option<&TrieNode> {
        let mut node = &*self.root;
        for c in s.chars() {
            let idx = (c as u8 - b'a') as usize;
            match &node.children[idx] {
                None => return None,
                Some(child) => node = child,
            }
        }
        Some(node)
    }

    fn insert(&mut self, word: &str) {
        let mut node = &mut *self.root;
        for c in word.chars() {
            let idx = (c as u8 - b'a') as usize;
            if node.children[idx].is_none() {
                node.children[idx] = Some(Box::new(TrieNode::new()));
            }
            node = node.children[idx].as_mut().unwrap();
            node.count_prefix += 1;
        }
        node.count_word += 1;
    }

    fn count_words_equal_to(&self, word: &str) -> i32 {
        self.find(word).map_or(0, |n| n.count_word)
    }

    fn count_words_starting_with(&self, prefix: &str) -> i32 {
        self.find(prefix).map_or(0, |n| n.count_prefix)
    }

    fn erase(&mut self, word: &str) {
        if self.count_words_equal_to(word) == 0 {
            return;
        }
        let mut node = &mut *self.root;
        for c in word.chars() {
            let idx = (c as u8 - b'a') as usize;
            node = node.children[idx].as_mut().unwrap();
            node.count_prefix -= 1;
        }
        node.count_word -= 1;
    }
}
```

`erase` here is **lazy deletion**: it decrements counters but leaves nodes in place. This is correct (queries read the counters) and simple. Physically removing nodes is the next follow-up.

## Array vs HashMap children — trade-off

| Aspect | `[Option<Box<TrieNode>>; 26]` array | `HashMap<char, Box<TrieNode>>` |
|--------|----------------------|-----------------------------|
| Child lookup | O(1) array index `c - 'a'` | O(1) amortized + hashing constant |
| Memory / node | 26 pointers always (sparse waste) | only existing children stored |
| Alphabet | fixed small (a–z) | arbitrary, Unicode, large |
| Ordered traversal | free (index order = lexicographic) | needs sorting |
| Constant factor | smallest — best for hot loops (Word Search II) | larger; more overhead |
| Cache behaviour | contiguous, cache-friendly | scattered allocations |

**Pick the array** for the standard 26-lowercase LeetCode setting and any performance-critical board DFS. **Pick the map** for large/sparse/Unicode alphabets or when memory per node matters more than constant factor.

## Complexity

| Operation | Time | Space (auxiliary) |
|-----------|------|-------------------|
| `insert` | O(L) | O(L) new nodes worst case |
| `search` | O(L) | O(1) |
| `startsWith` | O(L) | O(1) |
| Trie II counters | O(L) | O(1) |

**Whole-structure space:** **O(ALPHABET · N · L)** worst case — `N` words of length `L` with no shared prefixes, each node carrying a 26-slot array. With heavy prefix sharing the real footprint is far smaller.

## Follow-ups

1. **Delete operation (physical):** to actually unlink, recurse to the word's terminal node, clear `is_end`, and on the way back up remove any child that has no children *and* is not itself a word end. Must be careful not to delete nodes that are prefixes of other words.

   ```rust
   fn is_empty(node: &TrieNode) -> bool {
       node.children.iter().all(|c| c.is_none())
   }

   fn delete_word(node: &mut TrieNode, word: &[u8], i: usize) -> bool {
       if i == word.len() {
           if !node.is_end {
               return false; // word not present
           }
           node.is_end = false;
           return is_empty(node); // can prune if no children
       }
       let idx = (word[i] - b'a') as usize;
       if node.children[idx].is_none() {
           return false;
       }
       let should_delete = {
           let child = node.children[idx].as_mut().unwrap();
           delete_word(child, word, i + 1)
       };
       if should_delete {
           node.children[idx] = None; // prune dead child
           return !node.is_end && is_empty(node);
       }
       false
   }
   ```

2. **Memory optimization — compressed / radix trie (Patricia trie):** collapse chains of single-child nodes into one edge labelled with a substring. Dramatically reduces node count for sparse dictionaries (e.g. URLs, long unique suffixes), at the cost of more complex insert/split logic.

3. **Ternary search tree:** an alternative balancing memory and speed for large alphabets.

See also: [Trie Fundamentals](../Patterns/Trie%20Fundamentals.md), [Word Dictionary.md](Word%20Dictionary.md).

---

> **Last Updated:** 2026-06-26
