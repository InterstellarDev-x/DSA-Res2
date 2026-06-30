# Complexity Analysis — Linked List

> **Topic:** [Linked List](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## Pattern-to-Complexity Quick Reference

| Pattern / Problem | Time | Space | Notes |
|-------------------|------|-------|-------|
| [Fast & Slow Pointer](../Patterns/Fast%20and%20Slow%20Pointer.md) — middle | O(n) | O(1) | Slow traverses n/2 steps |
| [Fast & Slow Pointer](../Patterns/Fast%20and%20Slow%20Pointer.md) — cycle detect | O(n) | O(1) | Meet in ≤ c iterations after entry |
| [Fast & Slow Pointer](../Patterns/Fast%20and%20Slow%20Pointer.md) — cycle entry | O(n) | O(1) | Phase 1 + phase 2, both O(n) |
| [Reverse LL](../Patterns/Reverse%20Linked%20List.md) — iterative | O(n) | O(1) | |
| [Reverse LL](../Patterns/Reverse%20Linked%20List.md) — recursive | O(n) | O(n) | n frames on call stack |
| [Reverse K-Group](../Patterns/Reverse%20Linked%20List.md) | O(n) | O(1) | Each node visited twice |
| [Merge Two Sorted](../Patterns/Merge%20Linked%20Lists.md) | O(n+m) | O(1) | Reuses nodes |
| [Merge K Sorted — heap](../Patterns/Merge%20Linked%20Lists.md) | O(N log k) | O(k) | N total nodes, k lists |
| [Merge K Sorted — D&C](../Patterns/Merge%20Linked%20Lists.md) | O(N log k) | O(log k) | Log k levels of recursion |
| [Sort List — top-down](../Patterns/Merge%20Linked%20Lists.md) | O(n log n) | O(log n) | Recursion stack |
| [Sort List — bottom-up](../Patterns/Merge%20Linked%20Lists.md) | O(n log n) | O(1) | Iterative, no stack |
| [Two Pointers — nth from end](../Patterns/Two%20Pointers%20on%20LL.md) | O(n) | O(1) | One pass |
| [Two Pointers — intersection](../Patterns/Two%20Pointers%20on%20LL.md) | O(m+n) | O(1) | Each traverses both lists |
| [Rotate List](../Patterns/Two%20Pointers%20on%20LL.md) | O(n) | O(1) | Length + reconnect |
| [LRU Cache](../Design%20Data%20Structure%20Problems/LRU%20Cache.md) — get/put | O(1) | O(capacity) | HashMap + DLL |
| [LFU Cache](../Design%20Data%20Structure%20Problems/LFU%20Cache.md) — get/put | O(1) | O(capacity) | 2 HashMaps + freq DLLs |
| Copy List with Random (HashMap) | O(n) | O(n) | One map entry per node |
| Copy List with Random (interleave) | O(n) | O(1) | 3-pass in-place |
| Palindrome LL (reverse half) | O(n) | O(1) | |
| Find Duplicate (Floyd's) | O(n) | O(1) | Array as implicit graph |

---

## Space Complexity Traps

| Operation | Appears O(1) | Actually |
|-----------|-------------|---------|
| Recursive reverse | — | O(n) stack frames |
| HashSet cycle detection | — | O(n) set entries |
| Copy List (recursive) | — | O(n) stack |
| Java `LinkedHashMap` LRU | — | O(n) internal DLL + HashMap |
| `new ListNode(0)` dummy | ✅ O(1) | O(1) — single node |

---

## Amortized Analysis: Sort List (Top-Down)

Recurrence: `T(n) = 2T(n/2) + O(n)`

By Master Theorem: a=2, b=2, f(n)=O(n) → Case 2 → T(n) = O(n log n).

Space: recursion depth is O(log n) — each level halves the sublist size.

---

## Amortized Analysis: Merge K Sorted (Heap)

Each of N nodes is inserted into the heap exactly once and extracted exactly once.
- Each insert: O(log k)
- Each extract: O(log k)
- Total: O(N log k)

Space: heap holds at most k nodes at any time → O(k).

---

## How to Explain Floyd's O(n) Bound

> "Let the cycle length be c. When slow first enters the cycle, fast has already been cycling for some number of steps. The gap between them is at most c. Since fast gains 1 step on slow per iteration, they meet within c more iterations. Both the pre-cycle phase (O(F)) and the in-cycle phase (O(c)) are bounded by O(n). Phase 2 resets one pointer and they meet in O(F) steps, also O(n). Total: O(n)."

---

## How to Explain LRU O(1)

> "HashMap gives O(1) key lookup. The DLL gives O(1) insert-at-front and O(1) delete (because we have the node reference directly, not just a position). By combining them, get moves a node to front in O(1): map lookup → DLL remove → DLL insert. Put evicts tail.prev in O(1): DLL remove → map.remove(node.key) (key stored in node) → create new node → DLL insert-front → map.put."

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Common Mistakes](./Common%20Mistakes.md)
- [Dummy Node Technique](./Dummy%20Node%20Technique.md)
- [Linked List README](../README.md)

> **Last Updated:** 2026-06-26
