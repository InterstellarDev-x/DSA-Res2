# LRU Cache

> **Topic:** [Linked List](../README.md) · **Section:** Design Data Structure Problems
> **LeetCode:** [146 — LRU Cache](https://leetcode.com/problems/lru-cache/) | Difficulty: Hard

---

## Problem

Design a data structure that follows the **Least Recently Used** (LRU) cache eviction policy.

- `get(key)` — return value if exists, else -1. Mark as recently used.
- `put(key, value)` — insert or update. If at capacity, evict LRU item first.

**Constraint:** Both operations must be O(1).

---

## Design

Two data structures combined:

1. **HashMap<i32, usize>** — O(1) lookup by key to the node index in the DLL
2. **Doubly Linked List** — O(1) move-to-front and evict-from-back

```
head ↔ [most recent] ↔ ... ↔ [least recent] ↔ tail
```

Sentinel nodes `head` and `tail` eliminate edge cases (no null checks on insert/remove).

```rust
use std::collections::HashMap;

struct Node {
    key: i32,
    val: i32,
    prev: usize,
    next: usize,
}

struct LRUCache {
    capacity: usize,
    map: HashMap<i32, usize>, // key -> index in nodes Vec
    nodes: Vec<Node>,
    // nodes[0] = head sentinel, nodes[1] = tail sentinel
}

impl LRUCache {
    fn new(capacity: i32) -> Self {
        let nodes = vec![
            Node { key: 0, val: 0, prev: 0, next: 1 }, // head: next=tail(1)
            Node { key: 0, val: 0, prev: 0, next: 1 }, // tail: prev=head(0)
        ];
        LRUCache {
            capacity: capacity as usize,
            map: HashMap::new(),
            nodes,
        }
    }

    fn remove(&mut self, idx: usize) {
        let prev = self.nodes[idx].prev;
        let next = self.nodes[idx].next;
        self.nodes[prev].next = next;
        self.nodes[next].prev = prev;
    }

    fn insert_front(&mut self, idx: usize) {
        let head_next = self.nodes[0].next;
        self.nodes[idx].next = head_next;
        self.nodes[idx].prev = 0;
        self.nodes[head_next].prev = idx;
        self.nodes[0].next = idx;
    }

    fn get(&mut self, key: i32) -> i32 {
        if let Some(&idx) = self.map.get(&key) {
            self.remove(idx);
            self.insert_front(idx);
            self.nodes[idx].val
        } else {
            -1
        }
    }

    fn put(&mut self, key: i32, value: i32) {
        if let Some(&idx) = self.map.get(&key) {
            self.remove(idx);
            self.nodes[idx].val = value;
            self.insert_front(idx);
        } else if self.map.len() == self.capacity {
            // Evict LRU: node just before tail (index 1)
            let lru_idx = self.nodes[1].prev;
            let lru_key = self.nodes[lru_idx].key;
            self.remove(lru_idx);
            self.map.remove(&lru_key);
            // Reuse the evicted slot for the new node
            self.nodes[lru_idx].key = key;
            self.nodes[lru_idx].val = value;
            self.insert_front(lru_idx);
            self.map.insert(key, lru_idx);
        } else {
            let idx = self.nodes.len();
            self.nodes.push(Node { key, val: value, prev: 0, next: 0 });
            self.insert_front(idx);
            self.map.insert(key, idx);
        }
    }
}
```

---

## Why Doubly Linked List (not singly)?

`remove(node)` needs O(1) access to `node.prev`. Without a back-pointer, you'd need to traverse from head — O(n).

---

## Why Store `key` in the Node?

When evicting `tail->prev`, we need to remove it from the `HashMap`. Without `key` in the node, there's no way to look it up — we'd have to scan the map (O(n)).

---

## Rust Index-Based DLL Alternative

Rust cannot store `LinkedList` node handles in a `HashMap` due to ownership rules. The idiomatic Rust approach uses an index-based DLL (`Vec` with `usize` "pointers") paired with a `HashMap` storing `Vec` indices, enabling O(1) splice (move-to-front) without pointer rewiring:

```rust
use std::collections::HashMap;

// Rust idiomatic LRU using index-based DLL + HashMap
// Note: Rust's ownership model prevents storing LinkedList node handles in a HashMap.
// Vec indices serve as "iterators" — enabling O(1) splice (move-to-front).
struct Node {
    key: i32,
    val: i32,
    prev: usize,
    next: usize,
}

struct LRUCache {
    capacity: usize,
    lru_list: Vec<Node>,        // index-based DLL; 0=head sentinel, 1=tail sentinel
    map: HashMap<i32, usize>,  // key -> Vec index (replaces list iterator)
}

impl LRUCache {
    fn new(capacity: i32) -> Self {
        let lru_list = vec![
            Node { key: 0, val: 0, prev: 0, next: 1 }, // head sentinel
            Node { key: 0, val: 0, prev: 0, next: 1 }, // tail sentinel
        ];
        LRUCache { capacity: capacity as usize, lru_list, map: HashMap::new() }
    }

    // Equivalent of lruList.splice(lruList.begin(), lruList, it)
    fn splice_front(&mut self, idx: usize) {
        let prev = self.lru_list[idx].prev;
        let next = self.lru_list[idx].next;
        self.lru_list[prev].next = next;
        self.lru_list[next].prev = prev;
        let head_next = self.lru_list[0].next;
        self.lru_list[idx].next = head_next;
        self.lru_list[idx].prev = 0;
        self.lru_list[head_next].prev = idx;
        self.lru_list[0].next = idx;
    }

    fn get(&mut self, key: i32) -> i32 {
        if let Some(&idx) = self.map.get(&key) {
            self.splice_front(idx);
            self.lru_list[idx].val
        } else {
            -1
        }
    }

    fn put(&mut self, key: i32, value: i32) {
        if let Some(&idx) = self.map.get(&key) {
            self.lru_list[idx].val = value;
            self.splice_front(idx);
        } else if self.map.len() == self.capacity {
            let lru_idx = self.lru_list[1].prev; // back of list = least recently used
            let lru_key = self.lru_list[lru_idx].key;
            self.map.remove(&lru_key);
            // Remove from back
            let prev = self.lru_list[lru_idx].prev;
            let next = self.lru_list[lru_idx].next;
            self.lru_list[prev].next = next;
            self.lru_list[next].prev = prev;
            // Reuse slot, insert at front
            self.lru_list[lru_idx].key = key;
            self.lru_list[lru_idx].val = value;
            let head_next = self.lru_list[0].next;
            self.lru_list[lru_idx].next = head_next;
            self.lru_list[lru_idx].prev = 0;
            self.lru_list[head_next].prev = lru_idx;
            self.lru_list[0].next = lru_idx;
            self.map.insert(key, lru_idx);
        } else {
            let idx = self.lru_list.len();
            self.lru_list.push(Node { key, val: value, prev: 0, next: 0 });
            let head_next = self.lru_list[0].next;
            self.lru_list[idx].next = head_next;
            self.lru_list[idx].prev = 0;
            self.lru_list[head_next].prev = idx;
            self.lru_list[0].next = idx;
            self.map.insert(key, idx);
        }
    }
}
```

> **Note for interviews:** Mention the index-based DLL variant exists but implement from scratch (DLL + `HashMap`) unless told otherwise. Interviewers at Google/Amazon expect the DLL + HashMap design.

---

## Dry Run

`capacity = 2`

```
put(1,1) → head ↔ [1:1] ↔ tail
put(2,2) → head ↔ [2:2] ↔ [1:1] ↔ tail
get(1)   → head ↔ [1:1] ↔ [2:2] ↔ tail  (1 moved to front)
put(3,3) → evict 2 (tail.prev), head ↔ [3:3] ↔ [1:1] ↔ tail
get(2)   → return -1 (evicted)
```

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| `get` | O(1) | — |
| `put` | O(1) | O(capacity) |

---

## Follow-up Questions

**Q: How would you make this thread-safe?**
Wrap operations with `Mutex<LRUCache>` (exclusive lock) from `std::sync`, or use `RwLock<LRUCache>` with `write()` guards for puts and `read()` guards for gets. This is equivalent to Java's `ReentrantReadWriteLock`.

**Q: What if we want O(1) get/put but also O(1) `getMin`?**
That's the [LFU Cache](./LFU%20Cache.md) variant for frequency, or add a parallel min-heap (but breaks O(1) for updates).

**Q: How would you implement a distributed LRU?**
Redis uses an approximation — sample N random keys on eviction, pick the one with oldest last-access timestamp. True LRU is expensive in distributed settings.

---

## Related Files

- [LFU Cache](./LFU%20Cache.md) — frequency-based eviction, O(1) with freq-bucketed DLL
- [Linked List README](../README.md)

---

**Back:** [Linked List README](../README.md)

> **Last Updated:** 2026-06-26
