# Common Mistakes — Linked List

> **Topic:** [Linked List](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## 1. Fast/Slow Pointer — Wrong Loop Termination

```cpp
// BUG: only checks fast
while (fast != nullptr) {
    slow = slow->next;
    fast = fast->next->next; // crash if fast->next is nullptr
}

// FIX
while (fast != nullptr && fast->next != nullptr) { ... }
```

---

## 2. Reverse — Forgetting to Save Next

```cpp
// BUG: curr->next is lost
curr->next = prev;
prev = curr;
curr = curr->next; // curr->next is now prev, not the original next!

// FIX: save before relinking
ListNode* nxt = curr->next;
curr->next = prev;
prev = curr;
curr = nxt;
```

---

## 3. K-Group Reverse — Wrong groupPrev Advancement

```cpp
// BUG: advancing groupPrev to kth (new head of group) instead of old head (new tail)
groupPrev = kth; // wrong — skips to new head

// FIX: tmp = groupPrev->next (old first = new tail of reversed group)
ListNode* tmp = groupPrev->next;
groupPrev->next = kth; // kth is new head
groupPrev = tmp;       // advance to new tail
```

---

## 4. Partition List — Not Null-Terminating Greater List

```cpp
// BUG: greater->next may still point into the original list → cycle
less->next = greaterHead->next;
return lessHead->next;

// FIX: terminate before connecting
greater->next = nullptr; // MUST do this first
less->next = greaterHead->next;
return lessHead->next;
```

---

## 5. Remove Nth from End — Off-by-One on Gap

```cpp
// BUG: advance fast n times → slow lands ON the target, not before it
for (int i = 0; i < n; i++) fast = fast->next;

// FIX: advance n+1 times → slow lands one before target
for (int i = 0; i <= n; i++) fast = fast->next;
// Also: use dummy head so removing the first node works uniformly
```

---

## 6. LRU Cache — Not Storing Key in Node

```cpp
// BUG: can't evict LRU from unordered_map without knowing its key
Node* lru = tail->prev;
map.erase(???); // don't know the key!

// FIX: store key in Node
struct Node { int key, val; ... };
map.erase(lru->key); // works
```

---

## 7. Floyd's Phase 2 — Moving Fast at 2x

```cpp
// BUG: in phase 2, fast still moves 2x
slow = head;
while (slow != fast) {
    slow = slow->next;
    fast = fast->next->next; // wrong — phase 2 needs 1x speed
}

// FIX: both move 1x in phase 2
while (slow != fast) {
    slow = slow->next;
    fast = fast->next;
}
```

---

## 8. Intersecting Lists — Wrong Head Assignment on Restart

```cpp
// BUG: a restarts to headA, b restarts to headB (no progress)
a = (a != nullptr) ? a->next : headA;
b = (b != nullptr) ? b->next : headB;

// FIX: each restarts to the OTHER list's head
a = (a != nullptr) ? a->next : headB;
b = (b != nullptr) ? b->next : headA;
```

---

## 9. Cycle Detection — Checking Before Advancing

```cpp
// BUG: slow == fast is true at start (both at head) → false positive no-cycle
if (slow == fast) return true; // before any movement!
while (fast != nullptr && fast->next != nullptr) { ... }

// FIX: check AFTER advancing
while (fast != nullptr && fast->next != nullptr) {
    slow = slow->next;
    fast = fast->next->next;
    if (slow == fast) return true; // check after movement
}
```

---

## 10. Recursion Stack Overflow

Recursive reverse / recursive DFS on large inputs (n = 10^5) will cause a stack overflow.

```cpp
// RISKY for large input
ListNode* reverseList(ListNode* head) {
    if (head == nullptr || head->next == nullptr) return head;
    ListNode* rest = reverseList(head->next); // O(n) stack depth
    ...
}

// SAFE
ListNode* reverseList(ListNode* head) {
    ListNode* prev = nullptr, *curr = head; // iterative O(1) space
    while (curr != nullptr) { ... }
    return prev;
}
```

Default system stack size is typically 1–8 MB. A list of length 10^5 with deep recursion will overflow.

---

## 11. Heap Comparator Integer Overflow

```cpp
// BUG: a->val - b->val overflows for large negative/positive pairs
auto bad_cmp = [](Node* a, Node* b) { return a->val - b->val; };

// FIX: use a proper boolean comparator (min-heap by val)
auto cmp = [](Node* a, Node* b) { return a->val > b->val; };
priority_queue<Node*, vector<Node*>, decltype(cmp)> pq(cmp);
```

---

## Quick Checklist Before Submitting

- [ ] Dummy head used where head might change?
- [ ] `fast != nullptr && fast->next != nullptr` (both conditions)?
- [ ] `nxt` saved before `curr->next = prev`?
- [ ] Greater list null-terminated in partition?
- [ ] Phase 2 of Floyd's uses 1x speed?
- [ ] Key stored in LRU/LFU node?

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Dummy Node Technique](./Dummy%20Node%20Technique.md)

> **Last Updated:** 2026-06-26
