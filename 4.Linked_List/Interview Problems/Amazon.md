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
1. We need O(1) `get` and O(1) `put`. std::unordered_map gives O(1) lookup; we need O(1) eviction.
2. Eviction must always remove the "least recently used" — this is inherently ordered. A doubly linked list maintains this order.
3. But iterating the list to find a key is O(n). So we combine: std::unordered_map for O(1) key→node lookup, DLL for O(1) front-insertion and back-removal.
4. Sentinel head/tail nodes avoid special-casing empty list and boundary nodes.

```cpp
// The 4 key operations on the DLL:
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
```

**On `get`:** find node in map → remove from current position → insertFront → return val.
**On `put`:** if key exists, remove old; if at capacity, evict `tail->prev` (LRU) and remove from map; create new node → insertFront → put in map.

---

## Deep Dive: Copy List with Random Pointer (O(1) Space)

### Two Approaches

**Approach 1 — std::unordered_map (O(n) space):**
```cpp
#include <bits/stdc++.h>
using namespace std;
Node* copyRandomList(Node* head) {
    unordered_map<Node*, Node*> mp;
    Node* curr = head;
    while (curr != nullptr) { mp[curr] = new Node(curr->val); curr = curr->next; }
    curr = head;
    while (curr != nullptr) {
        mp[curr]->next = mp[curr->next];
        mp[curr]->random = mp[curr->random];
        curr = curr->next;
    }
    return mp[head];
}
```

**Approach 2 — Interleaving (O(1) space):**

```cpp
Node* copyRandomList(Node* head) {
    if (head == nullptr) return nullptr;

    // Step 1: Interleave clones — 1 → 1' → 2 → 2' → 3 → 3' → ...
    Node* curr = head;
    while (curr != nullptr) {
        Node* clone = new Node(curr->val);
        clone->next = curr->next;
        curr->next = clone;
        curr = clone->next;
    }

    // Step 2: Set random pointers — curr->next->random = curr->random->next (the clone of random)
    curr = head;
    while (curr != nullptr) {
        if (curr->random != nullptr) curr->next->random = curr->random->next;
        curr = curr->next->next;
    }

    // Step 3: Separate lists
    Node* cloneHead = head->next;
    curr = head;
    while (curr != nullptr) {
        Node* clone = curr->next;
        curr->next = clone->next;
        clone->next = (clone->next != nullptr) ? clone->next->next : nullptr;
        curr = curr->next;
    }
    return cloneHead;
}
```

**Key insight in step 2:** `curr->random->next` is the clone of `curr->random` because we interleaved all clones in step 1.

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
