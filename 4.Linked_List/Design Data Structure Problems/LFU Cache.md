# LFU Cache

> **Topic:** [Linked List](../README.md) · **Section:** Design Data Structure Problems
> **LeetCode:** [460 — LFU Cache](https://leetcode.com/problems/lfu-cache/) | Difficulty: Hard

---

## Problem

Design a **Least Frequently Used** cache. Among ties in frequency, evict the **Least Recently Used**.

- `get(key)` — O(1), increases key's frequency
- `put(key, value)` — O(1), evict LFU item if at capacity

---

## Design

Three data structures:

1. **`key_map: HashMap<i32, usize>`** — O(1) lookup to node (arena index) by key
2. **`freq_map: HashMap<i32, DLL>`** — each frequency bucket is a DLL of nodes (ordered by recency: head = most recent)
3. **`min_freq: i32`** — current minimum frequency in the cache

### Node Structure

```rust
// In Rust, raw pointers are replaced with arena indices (usize)
const NULL: usize = usize::MAX; // represents null pointer

#[derive(Debug, Clone)]
struct Node {
    key: i32,
    val: i32,
    freq: i32,
    prev: usize, // index into arena (NULL = no node)
    next: usize,
}

impl Node {
    fn new(key: i32, val: i32) -> Self {
        Node { key, val, freq: 1, prev: NULL, next: NULL }
    }
    fn sentinel() -> Self {
        Node { key: 0, val: 0, freq: 0, prev: NULL, next: NULL }
    }
}
```

### DoublyLinkedList (per frequency bucket)

Each frequency bucket is an LRU cache internally — most recently used is at head, LRU is at tail.

```rust
// Per-frequency doubly-linked list backed by a shared node arena (Vec<Node>)
// head/tail are sentinel node indices; actual nodes sit between them.
struct DLL {
    head: usize, // sentinel head index in arena
    tail: usize, // sentinel tail index in arena
    size: usize,
}

impl DLL {
    fn new(arena: &mut Vec<Node>) -> Self {
        let head = arena.len();
        arena.push(Node::sentinel());
        let tail = arena.len();
        arena.push(Node::sentinel());
        arena[head].next = tail;
        arena[tail].prev = head;
        DLL { head, tail, size: 0 }
    }

    // Insert node at the front (most-recently-used end)
    fn add_front(&mut self, arena: &mut Vec<Node>, node_idx: usize) {
        let second = arena[self.head].next;
        arena[node_idx].next = second;
        arena[node_idx].prev = self.head;
        arena[second].prev = node_idx;
        arena[self.head].next = node_idx;
        self.size += 1;
    }

    fn remove(arena: &mut Vec<Node>, node_idx: usize) {
        let p = arena[node_idx].prev;
        let n = arena[node_idx].next;
        arena[p].next = n;
        arena[n].prev = p;
    }

    // Remove and return LRU node (tail's predecessor); None if empty
    fn remove_lru(&mut self, arena: &mut Vec<Node>) -> Option<usize> {
        if self.size == 0 { return None; }
        let lru = arena[self.tail].prev;
        Self::remove(arena, lru);
        self.size -= 1;
        Some(lru)
    }
}
```

---

## Full Implementation

```rust
use std::collections::HashMap;

const NULL: usize = usize::MAX; // replaces C++ nullptr

#[derive(Debug, Clone)]
struct Node {
    key: i32,
    val: i32,
    freq: i32,
    prev: usize, // arena index (NULL = no node)
    next: usize,
}

impl Node {
    fn new(key: i32, val: i32) -> Self {
        Node { key, val, freq: 1, prev: NULL, next: NULL }
    }
    fn sentinel() -> Self {
        Node { key: 0, val: 0, freq: 0, prev: NULL, next: NULL }
    }
}

struct DLL {
    head: usize, // sentinel head index in arena
    tail: usize, // sentinel tail index in arena
    size: usize,
}

struct LFUCache {
    capacity: usize,
    min_freq: i32,
    arena:    Vec<Node>,           // node arena (replaces raw pointer heap allocations)
    key_map:  HashMap<i32, usize>, // key -> arena index
    freq_map: HashMap<i32, DLL>,   // freq -> DLL
}

impl LFUCache {
    fn new(capacity: i32) -> Self {
        LFUCache {
            capacity: capacity as usize,
            min_freq: 0,
            arena:    Vec::new(),
            key_map:  HashMap::new(),
            freq_map: HashMap::new(),
        }
    }

    fn alloc_node(&mut self, key: i32, val: i32) -> usize {
        let i = self.arena.len();
        self.arena.push(Node::new(key, val));
        i
    }

    fn alloc_sentinel(&mut self) -> usize {
        let i = self.arena.len();
        self.arena.push(Node::sentinel());
        i
    }

    fn make_dll(&mut self) -> DLL {
        let head = self.alloc_sentinel();
        let tail = self.alloc_sentinel();
        self.arena[head].next = tail;
        self.arena[tail].prev = head;
        DLL { head, tail, size: 0 }
    }

    // Insert node at front of list (most-recently-used end)
    fn list_add_front(&mut self, head: usize, node: usize) {
        let second = self.arena[head].next;
        self.arena[node].prev = head;
        self.arena[node].next = second;
        self.arena[second].prev = node;
        self.arena[head].next = node;
    }

    // Unlink node from wherever it sits in the list
    fn list_remove(&mut self, node: usize) {
        let p = self.arena[node].prev;
        let n = self.arena[node].next;
        self.arena[p].next = n;
        self.arena[n].prev = p;
    }

    // Ensure a frequency bucket exists; returns (head, tail) sentinel indices
    fn ensure_bucket(&mut self, freq: i32) -> (usize, usize) {
        if !self.freq_map.contains_key(&freq) {
            let dll = self.make_dll();
            self.freq_map.insert(freq, dll);
        }
        let dll = &self.freq_map[&freq];
        (dll.head, dll.tail)
    }

    fn get(&mut self, key: i32) -> i32 {
        let Some(&node_idx) = self.key_map.get(&key) else { return -1; };
        let val = self.arena[node_idx].val;
        self.update_freq(node_idx);
        val
    }

    fn put(&mut self, key: i32, value: i32) {
        if self.capacity == 0 { return; }

        if let Some(&node_idx) = self.key_map.get(&key) {
            self.arena[node_idx].val = value;
            self.update_freq(node_idx);
            return;
        }

        // Evict LFU (LRU within minFreq bucket)
        if self.key_map.len() == self.capacity {
            let mf = self.min_freq;
            let (head, tail) = {
                let dll = self.freq_map.get(&mf).unwrap();
                (dll.head, dll.tail)
            };
            let lru = self.arena[tail].prev; // LRU = tail's predecessor
            debug_assert!(lru != head, "minFreq bucket is unexpectedly empty");
            self.list_remove(lru);
            let evicted_key = self.arena[lru].key;
            self.key_map.remove(&evicted_key);
            let dll = self.freq_map.get_mut(&mf).unwrap();
            dll.size -= 1;
            if dll.size == 0 {
                self.freq_map.remove(&mf);
            }
        }

        let node_idx = self.alloc_node(key, value); // freq = 1
        self.key_map.insert(key, node_idx);
        let (head, _) = self.ensure_bucket(1);
        self.list_add_front(head, node_idx);
        self.freq_map.get_mut(&1).unwrap().size += 1;
        self.min_freq = 1; // new node always at freq 1
    }

    fn update_freq(&mut self, node_idx: usize) {
        let old_freq = self.arena[node_idx].freq;

        // Remove from old bucket
        self.list_remove(node_idx);
        let dll = self.freq_map.get_mut(&old_freq).unwrap();
        dll.size -= 1;
        let became_empty = dll.size == 0;
        if became_empty {
            self.freq_map.remove(&old_freq);
            if self.min_freq == old_freq {
                self.min_freq += 1; // only update if min bucket emptied
            }
        }

        // Move to new bucket
        let new_freq = old_freq + 1;
        self.arena[node_idx].freq = new_freq;
        let (new_head, _) = self.ensure_bucket(new_freq);
        self.list_add_front(new_head, node_idx);
        self.freq_map.get_mut(&new_freq).unwrap().size += 1;
    }
}
```

---

## Key Insight: Why `min_freq += 1` Only When `old_freq == min_freq`

When a node's frequency increases from `old_freq` to `old_freq+1`:
- If `old_freq` was `min_freq` AND the bucket is now empty → min shifts to `min_freq+1`
- If `old_freq` > `min_freq`, the minimum doesn't change

On `put` of a new node: reset `min_freq = 1` (new node always at frequency 1).

---

## Comparison: LRU vs LFU

| Aspect | LRU | LFU |
|--------|-----|-----|
| Eviction rule | Least recently used | Least frequently used (LRU as tiebreaker) |
| State tracked | Position in access order | Frequency + position per frequency |
| Data structures | 1 HashMap + 1 DLL | 2 HashMaps + N DLLs |
| When optimal | Temporal locality (e.g., video streaming) | Frequency locality (e.g., database pages) |

---

## Dry Run

`capacity = 2`

```
put(1,1): key_map={1:Node(1,1,f=1)}, freq_map={1:[1]}, min_freq=1
put(2,2): key_map={1,2}, freq_map={1:[2,1]}, min_freq=1
get(1):   freq[1] moves to freq[2]. freq_map={1:[2], 2:[1]}, min_freq=1
put(3,3): evict min_freq=1 bucket LRU = node 2
          key_map={1,3}, freq_map={1:[3], 2:[1]}, min_freq=1
get(2):   return -1
get(3):   freq[3] moves to 2. freq_map={2:[3,1]}, min_freq=2
```

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| `get` | O(1) | — |
| `put` | O(1) | O(capacity) |

---

## Common Pitfalls

| Mistake | Fix |
|---------|-----|
| Incrementing `min_freq` unconditionally on update | Only increment if `old_freq == min_freq` AND that bucket is now empty |
| Not resetting `min_freq = 1` on new `put` | New nodes always go to freq=1; min_freq must reflect this |
| `computeIfAbsent` (Java) / missing bucket init | Use `freq_map.contains_key(&freq)` before inserting; create new DLL if absent |
| Forgetting `if capacity == 0 { return; }` | LFU with capacity 0 should do nothing |

---

## Related Files

- [LRU Cache](./LRU%20Cache.md) — simpler eviction, one DLL
- [Linked List README](../README.md)

---

**Back:** [Linked List README](../README.md)

> **Last Updated:** 2026-06-26
