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

1. **unordered_map<int, Node\*>** — O(1) lookup by key to the node in the DLL
2. **Doubly Linked List** — O(1) move-to-front and evict-from-back

```
head ↔ [most recent] ↔ ... ↔ [least recent] ↔ tail
```

Sentinel nodes `head` and `tail` eliminate edge cases (no null checks on insert/remove).

```cpp
#include <bits/stdc++.h>
using namespace std;

class LRUCache {
    struct Node {
        int key, val;
        Node* prev;
        Node* next;
        Node(int k, int v) : key(k), val(v), prev(nullptr), next(nullptr) {}
    };

    int capacity;
    unordered_map<int, Node*> map;
    Node* head;
    Node* tail; // sentinels

    void remove(Node* node) {
        node->prev->next = node->next;
        node->next->prev = node->prev;
    }

    void insertFront(Node* node) {
        node->next = head->next;
        node->prev = head;
        head->next->prev = node;
        head->next = node;
    }

public:
    LRUCache(int capacity) : capacity(capacity) {
        head = new Node(0, 0);
        tail = new Node(0, 0);
        head->next = tail;
        tail->prev = head;
    }

    int get(int key) {
        if (!map.count(key)) return -1;
        Node* node = map[key];
        remove(node);
        insertFront(node);
        return node->val;
    }

    void put(int key, int value) {
        if (map.count(key)) {
            remove(map[key]);
        } else if ((int)map.size() == capacity) {
            // Evict LRU: node just before tail
            Node* lru = tail->prev;
            remove(lru);
            map.erase(lru->key);
            delete lru;
        }
        Node* node = new Node(key, value);
        insertFront(node);
        map[key] = node;
    }
};
```

---

## Why Doubly Linked List (not singly)?

`remove(node)` needs O(1) access to `node.prev`. Without a back-pointer, you'd need to traverse from head — O(n).

---

## Why Store `key` in the Node?

When evicting `tail->prev`, we need to remove it from the `unordered_map`. Without `key` in the node, there's no way to look it up — we'd have to scan the map (O(n)).

---

## C++ std::list Shortcut

C++ has no direct equivalent of Java's `LinkedHashMap` with `accessOrder`. The idiomatic C++ approach uses `std::list` paired with `std::unordered_map` storing list iterators, enabling O(1) splice (move-to-front) without pointer rewiring:

```cpp
#include <bits/stdc++.h>
using namespace std;

// C++ idiomatic LRU using std::list + unordered_map
class LRUCache {
    int capacity;
    list<pair<int,int>> lruList; // {key, value}, front = most recent
    unordered_map<int, list<pair<int,int>>::iterator> map;

public:
    LRUCache(int capacity) : capacity(capacity) {}

    int get(int key) {
        if (!map.count(key)) return -1;
        lruList.splice(lruList.begin(), lruList, map[key]);
        return map[key]->second;
    }

    void put(int key, int value) {
        if (map.count(key)) {
            lruList.erase(map[key]);
        } else if ((int)lruList.size() == capacity) {
            map.erase(lruList.back().first);
            lruList.pop_back();
        }
        lruList.push_front({key, value});
        map[key] = lruList.begin();
    }
};
```

> **Note for interviews:** Mention the `std::list` variant exists but implement from scratch (DLL + `unordered_map`) unless told otherwise. Interviewers at Google/Amazon expect the DLL + HashMap design.

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
Wrap operations with `std::mutex` (exclusive lock) or use `std::shared_mutex` with `std::unique_lock` for writes and `std::shared_lock` for reads. Or use a `std::shared_mutex` as a reader-writer lock equivalent to Java's `ReentrantReadWriteLock`.

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
