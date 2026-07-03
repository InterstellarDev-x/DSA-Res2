# Amazon Interview Problems — Linked List

> **Topic:** [Linked List](../README.md) · **Section:** Interview Problems
> **Last Updated:** 2026-06-26

---

## Question Bank

| # | Problem | Level | Difficulty | Frequency | Pattern | LeetCode |
|---|---------|-------|-----------|----------|---------|---------|
| 1 | LRU Cache | SDE II+ | Hard | ⭐⭐⭐⭐⭐ | [Design](../Design%20Data%20Structure%20Problems/LRU%20Cache.md) | [146](https://leetcode.com/problems/lru-cache/) |
| 2 | Merge K Sorted Lists | SDE II | Hard | ⭐⭐⭐⭐ | [Merge + Heap](../Patterns/Merge%20Linked%20Lists.md) | [23](https://leetcode.com/problems/merge-k-sorted-lists/) |
| 3 | Reverse Nodes in K-Group | SDE II | Hard | ⭐⭐⭐⭐ | [Reverse](../Patterns/Reverse%20Linked%20List.md) | [25](https://leetcode.com/problems/reverse-nodes-in-k-group/) |
| 4 | Copy List with Random Pointer | SDE I/II | Medium | ⭐⭐⭐⭐ | HashMap / Interleave | [138](https://leetcode.com/problems/copy-list-with-random-pointer/) |
| 5 | Reorder List | SDE I | Medium | ⭐⭐⭐ | Fast&Slow + Reverse + Merge | [143](https://leetcode.com/problems/reorder-list/) |

---

## Deep Dive: LRU Cache (Most Asked)

### Q: Walk me through the O(1) LRU Cache design.

**Answer framework:**
1. We need O(1) `get` and O(1) `put`. HashMap gives O(1) lookup; we need O(1) eviction.
2. Eviction must always remove the "least recently used" — this is inherently ordered. A doubly linked list maintains this order.
3. But iterating the list to find a key is O(n). So we combine: HashMap for O(1) key→node lookup, DLL for O(1) front-insertion and back-removal.
4. Sentinel head/tail nodes avoid special-casing empty list and boundary nodes.

```rust
#[derive(Debug, Clone)]
struct Node {
    val: i32,
    key: i32,
    prev: usize,
    next: usize,
}

// The 4 key operations on the DLL (index-based; sentinel head=0, tail=1):
fn remove(nodes: &mut Vec<Node>, idx: usize) {
    let prev = nodes[idx].prev;
    let next = nodes[idx].next;
    nodes[prev].next = next;
    nodes[next].prev = prev;
}

fn insert_front(nodes: &mut Vec<Node>, head: usize, idx: usize) {
    let front = nodes[head].next;
    nodes[idx].next = front;
    nodes[idx].prev = head;
    nodes[front].prev = idx;
    nodes[head].next = idx;
}
```

**On `get`:** find node in map → remove from current position → insert_front → return val.
**On `put`:** if key exists, remove old; if at capacity, evict `nodes[tail].prev` (LRU) and remove from map; create new node → insert_front → put in map.

---

## Deep Dive: Copy List with Random Pointer (O(1) Space)

### Two Approaches

**Approach 1 — HashMap (O(n) space):**
```rust
use std::collections::HashMap;

#[derive(Debug, Clone)]
struct Node {
    val: i32,
    next: Option<usize>,    // index into arena, None = null
    random: Option<usize>,  // index into arena, None = null
}

fn copy_random_list(nodes: &[Node]) -> Vec<Node> {
    let n = nodes.len();
    let mut mp: HashMap<usize, usize> = HashMap::new();
    let mut clones: Vec<Node> = Vec::with_capacity(n);

    // First pass: create all clone nodes
    for (i, node) in nodes.iter().enumerate() {
        clones.push(Node { val: node.val, next: None, random: None });
        mp.insert(i, i);
    }
    // Second pass: wire next and random using the map
    for (i, node) in nodes.iter().enumerate() {
        clones[i].next = node.next.and_then(|j| mp.get(&j).copied());
        clones[i].random = node.random.and_then(|j| mp.get(&j).copied());
    }
    clones
}
```

**Approach 2 — Interleaving (O(1) space):**

```rust
fn copy_random_list_interleave(nodes: &[Node]) -> Vec<Node> {
    let n = nodes.len();
    if n == 0 {
        return vec![];
    }

    // Step 1: Interleave clones — original[i] at 0..n, clone[i] at n..2n
    // original[i].next = clone[i], clone[i].next = original[i+1]
    let mut aug: Vec<Node> = nodes.to_vec();
    aug.extend(nodes.iter().map(|nd| Node { val: nd.val, next: None, random: None }));
    for i in 0..n {
        aug[i].next = Some(n + i);       // original[i] -> clone[i]
        aug[n + i].next = nodes[i].next; // clone[i] -> original[i+1]
    }

    // Step 2: Set random pointers — clone[i].random = clone of original[i].random
    // aug[r].next = Some(n + r), which is the clone of node r
    for i in 0..n {
        if let Some(r) = nodes[i].random {
            aug[n + i].random = aug[r].next; // points to n+r (the clone of r)
        }
    }

    // Step 3: Separate lists — extract clone list (n..2n), reindex to 0..n
    (0..n)
        .map(|i| Node {
            val: aug[n + i].val,
            next: aug[n + i].next, // original index == clone index after reindexing
            random: aug[n + i].random.map(|j| j - n), // j = n+r => j-n = r
        })
        .collect()
}
```

**Key insight in step 2:** `aug[r].next` is `Some(n + r)` — the clone of node `r` — because we interleaved all clones in step 1.

---

## Amazon LP + Coding Tips

| Problem | LP Principle |
|---------|-------------|
| LRU Cache | Invent and Simplify — clean DLL design over `LinkedHashMap` shortcut |
| Merge K Sorted | Deliver Results — pick correct O(N log k) approach, not O(Nk) naive |
| Reorder List | Dive Deep — trace through fast/slow split, then reverse, then merge |
| Copy List | Are Right, A Lot — proactively mention both approaches, explain tradeoffs |

---

## Related Files

- [Amazon OA-Qns](../OA-Qns/Amazon.md)
- [Linked List README](../README.md)

> **Last Updated:** 2026-06-26
