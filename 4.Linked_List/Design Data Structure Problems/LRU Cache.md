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

1. **HashMap<Integer, Node>** — O(1) lookup by key to the node in the DLL
2. **Doubly Linked List** — O(1) move-to-front and evict-from-back

```
head ↔ [most recent] ↔ ... ↔ [least recent] ↔ tail
```

Sentinel nodes `head` and `tail` eliminate edge cases (no null checks on insert/remove).

```java
class LRUCache {
    private final int capacity;
    private final Map<Integer, Node> map;
    private final Node head, tail; // sentinels

    private static class Node {
        int key, val;
        Node prev, next;
        Node(int k, int v) { key = k; val = v; }
    }

    public LRUCache(int capacity) {
        this.capacity = capacity;
        map = new HashMap<>();
        head = new Node(0, 0);
        tail = new Node(0, 0);
        head.next = tail;
        tail.prev = head;
    }

    public int get(int key) {
        if (!map.containsKey(key)) return -1;
        Node node = map.get(key);
        remove(node);
        insertFront(node);
        return node.val;
    }

    public void put(int key, int value) {
        if (map.containsKey(key)) {
            remove(map.get(key));
        } else if (map.size() == capacity) {
            // Evict LRU: node just before tail
            Node lru = tail.prev;
            remove(lru);
            map.remove(lru.key);
        }
        Node node = new Node(key, value);
        insertFront(node);
        map.put(key, node);
    }

    private void remove(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }

    private void insertFront(Node node) {
        node.next = head.next;
        node.prev = head;
        head.next.prev = node;
        head.next = node;
    }
}
```

---

## Why Doubly Linked List (not singly)?

`remove(node)` needs O(1) access to `node.prev`. Without a back-pointer, you'd need to traverse from head — O(n).

---

## Why Store `key` in the Node?

When evicting `tail.prev`, we need to remove it from the HashMap. Without `key` in the node, there's no way to look it up — we'd have to scan the map (O(n)).

---

## Java LinkedHashMap Shortcut

Java's `LinkedHashMap` with `accessOrder = true` implements LRU internally:

```java
class LRUCache extends LinkedHashMap<Integer, Integer> {
    private final int capacity;

    public LRUCache(int capacity) {
        super(capacity, 0.75f, true); // accessOrder = true
        this.capacity = capacity;
    }

    public int get(int key) { return super.getOrDefault(key, -1); }

    public void put(int key, int value) { super.put(key, value); }

    @Override
    protected boolean removeEldestEntry(Map.Entry<Integer, Integer> eldest) {
        return size() > capacity;
    }
}
```

> **Note for interviews:** Mention this exists but implement from scratch unless told otherwise. Interviewers at Google/Amazon expect the DLL + HashMap design.

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
Wrap with `Collections.synchronizedMap` or use `ConcurrentHashMap` + explicit synchronized block around DLL operations. Or use `ReentrantReadWriteLock`.

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
