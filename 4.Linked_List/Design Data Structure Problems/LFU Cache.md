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

1. **`keyMap: HashMap<Integer, Node>`** — O(1) lookup to node by key
2. **`freqMap: HashMap<Integer, DoublyLinkedList>`** — each frequency bucket is a DLL of nodes (ordered by recency: head = most recent)
3. **`minFreq: int`** — current minimum frequency in the cache

### Node Structure

```java
private static class Node {
    int key, val, freq;
    Node prev, next;
    Node(int k, int v) { key = k; val = v; freq = 1; }
}
```

### DoublyLinkedList (per frequency bucket)

Each frequency bucket is an LRU cache internally — most recently used is at head, LRU is at tail.

```java
private static class DLL {
    Node head, tail; // sentinels
    int size;

    DLL() {
        head = new Node(0, 0);
        tail = new Node(0, 0);
        head.next = tail;
        tail.prev = head;
    }

    void addFront(Node node) {
        node.next = head.next;
        node.prev = head;
        head.next.prev = node;
        head.next = node;
        size++;
    }

    void remove(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
        size--;
    }

    Node removeLast() { // remove LRU from this bucket
        if (size == 0) return null;
        Node node = tail.prev;
        remove(node);
        return node;
    }
}
```

---

## Full Implementation

```java
class LFUCache {
    private final int capacity;
    private int minFreq;
    private final Map<Integer, Node> keyMap;
    private final Map<Integer, DLL> freqMap;

    public LFUCache(int capacity) {
        this.capacity = capacity;
        this.minFreq = 0;
        keyMap = new HashMap<>();
        freqMap = new HashMap<>();
    }

    public int get(int key) {
        if (!keyMap.containsKey(key)) return -1;
        Node node = keyMap.get(key);
        updateFreq(node);
        return node.val;
    }

    public void put(int key, int value) {
        if (capacity == 0) return;

        if (keyMap.containsKey(key)) {
            Node node = keyMap.get(key);
            node.val = value;
            updateFreq(node);
            return;
        }

        if (keyMap.size() == capacity) {
            // Evict LFU (LRU within minFreq bucket)
            DLL minBucket = freqMap.get(minFreq);
            Node evicted = minBucket.removeLast();
            keyMap.remove(evicted.key);
        }

        Node node = new Node(key, value); // freq = 1
        keyMap.put(key, node);
        freqMap.computeIfAbsent(1, k -> new DLL()).addFront(node);
        minFreq = 1; // new node always has freq 1
    }

    private void updateFreq(Node node) {
        int oldFreq = node.freq;
        DLL oldBucket = freqMap.get(oldFreq);
        oldBucket.remove(node);

        if (oldBucket.size == 0) {
            freqMap.remove(oldFreq);
            if (minFreq == oldFreq) minFreq++; // only increment if we removed min
        }

        node.freq++;
        freqMap.computeIfAbsent(node.freq, k -> new DLL()).addFront(node);
    }
}
```

---

## Key Insight: Why `minFreq++` Only When `oldFreq == minFreq`

When a node's frequency increases from `oldFreq` to `oldFreq+1`:
- If `oldFreq` was `minFreq` AND the bucket is now empty → min shifts to `minFreq+1`
- If `oldFreq` > `minFreq`, the minimum doesn't change

On `put` of a new node: reset `minFreq = 1` (new node always at frequency 1).

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
put(1,1): keyMap={1:Node(1,1,f=1)}, freqMap={1:[1]}, minFreq=1
put(2,2): keyMap={1,2}, freqMap={1:[2,1]}, minFreq=1
get(1):   freq[1] moves to freq[2]. freqMap={1:[2], 2:[1]}, minFreq=1
put(3,3): evict minFreq=1 bucket LRU = node 2
          keyMap={1,3}, freqMap={1:[3], 2:[1]}, minFreq=1
get(2):   return -1
get(3):   freq[3] moves to 2. freqMap={2:[3,1]}, minFreq=2
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
| Incrementing `minFreq` unconditionally on update | Only increment if `oldFreq == minFreq` AND that bucket is now empty |
| Not resetting `minFreq = 1` on new `put` | New nodes always go to freq=1; minFreq must reflect this |
| `computeIfAbsent` returning stale reference | `computeIfAbsent` returns the value — use it directly |
| Forgetting `if (capacity == 0) return` | LFU with capacity 0 should do nothing |

---

## Related Files

- [LRU Cache](./LRU%20Cache.md) — simpler eviction, one DLL
- [Linked List README](../README.md)

---

**Back:** [Linked List README](../README.md)

> **Last Updated:** 2026-06-26
