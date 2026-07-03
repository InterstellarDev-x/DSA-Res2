# Dummy Node Technique

> **Topic:** [Linked List](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## Why Dummy Nodes?

Linked list problems frequently require:
1. Building a new list (merge, partition, sort)
2. Deleting the head node (remove nth, delete duplicates)
3. Inserting at the front (reverse sublist)

Without a dummy node, you write `if (head == nullptr)` or `if (prev == nullptr)` guards throughout. With a dummy node, the head is never the "first real node" — it's always a safe sentinel, and `dummy->next` is the answer.

---

## Pattern 1 — Building a New List

```cpp
ListNode* dummy = new ListNode(0);
ListNode* tail = dummy;

// Append nodes
tail->next = someNode;
tail = tail->next;

return dummy->next; // real answer
```

**Problems:** Merge Two Sorted, Merge K Sorted, Add Two Numbers, Odd Even LL, Partition List.

---

## Pattern 2 — Deleting a Node (May Delete Head)

```cpp
ListNode* dummy = new ListNode(0);
dummy->next = head;
ListNode* prev = dummy;

// Walk until prev->next is the node to delete
prev->next = prev->next->next; // delete

return dummy->next; // head may have changed
```

**Without dummy:** If the head itself needs deletion, `prev` would be `nullptr` — you'd need to special-case `if (prev == nullptr) head = head->next`.

**Problems:** Remove Nth from End, Remove Duplicates (82), Remove Elements.

---

## Pattern 3 — Reverse Sublist (Protect Left Boundary)

```cpp
ListNode* dummy = new ListNode(0);
dummy->next = head;
ListNode* pre = dummy; // node just before the reversal window

// Walk pre to position left-1
for (int i = 0; i < left - 1; i++) pre = pre->next;

// Reverse from pre->next to pre->next + (right-left) nodes
// pre->next will be head of reversed section; pre is its left anchor
```

**Without dummy:** When `left = 1`, `pre` would be `nullptr`.

---

## Pattern 4 — Sentinel Head + Tail (DLL, LRU/LFU)

```cpp
Node* head = new Node(0, 0); // sentinel head
Node* tail = new Node(0, 0); // sentinel tail
head->next = tail;
tail->prev = head;
```

**All operations become uniform:**
```cpp
// Insert at front (most recent)
void insertFront(Node* node) {
    node->next = head->next; // never nullptr (at worst, tail)
    node->prev = head;
    head->next->prev = node;
    head->next = node;
}

// Remove LRU (just before tail)
Node* lru = tail->prev; // never nullptr (at worst, head — but we check size)
remove(lru);
```

**Without sentinels:** `insertFront` must check `if (head->next == nullptr) ...`; `removeLast` must check `if (tail->prev == nullptr) ...`

---

## When NOT to Use Dummy Node

- In-place problems where you return the original head unchanged (e.g., palindrome check, cycle detection)
- When you only traverse, never modify the list structure

---

## Dummy Node Value Convention

Use `new ListNode(0)` — the value doesn't matter since `dummy` is never part of the answer. Some coders use `-1` or `INT_MIN` to distinguish from real values during debugging.

---

## Cheat Sheet

| Scenario | Use Dummy? | Return |
|----------|-----------|--------|
| Building output list | ✅ Yes | `dummy->next` |
| Deleting node (head may change) | ✅ Yes | `dummy->next` |
| Reversing sublist | ✅ Yes | `dummy->next` |
| Just traversing | ❌ No | unchanged `head` |
| DLL with O(1) insert/remove | ✅ Yes (both ends) | `head->next` |

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Common Mistakes](./Common%20Mistakes.md)

> **Last Updated:** 2026-06-26
