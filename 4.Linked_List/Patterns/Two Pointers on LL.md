# Two Pointers on Linked List

> **Topic:** [Linked List](../README.md) · **Section:** Patterns
> **Applies to:** Nth from end, intersection, rotate, partition

---

## Core Idea

Unlike arrays, linked lists have no random access — so "two pointers" on a LL means:

1. **Gap pointer:** Advance one pointer N steps ahead, then move both at 1x until the leader hits `nullptr`. Follower is now N steps from end.
2. **Intersection / sync:** Two pointers start at different heads. When each exhausts its list, it restarts from the **other** head. They meet at intersection (or both hit `nullptr` if none).

---

## Template 1 — Remove Nth Node from End (LC 19)

```cpp
#include <bits/stdc++.h>
using namespace std;

struct ListNode { int val; ListNode* next; ListNode(int x): val(x), next(nullptr){} };

ListNode* removeNthFromEnd(ListNode* head, int n) {
    ListNode* dummy = new ListNode(0);
    dummy->next = head;
    ListNode* fast = dummy, *slow = dummy;

    // Advance fast n+1 steps (so slow lands one BEFORE target)
    for (int i = 0; i <= n; i++) fast = fast->next;

    while (fast != nullptr) {
        slow = slow->next;
        fast = fast->next;
    }

    slow->next = slow->next->next; // delete target
    return dummy->next;
}
```

**Why n+1 steps:** We want `slow` to be the node before the target. Advancing `fast` one extra step creates the needed gap.

---

## Template 2 — Intersection of Two Linked Lists (LC 160)

```cpp
#include <bits/stdc++.h>
using namespace std;

ListNode* getIntersectionNode(ListNode* headA, ListNode* headB) {
    ListNode* a = headA, *b = headB;
    while (a != b) {
        a = (a != nullptr) ? a->next : headB;
        b = (b != nullptr) ? b->next : headA;
    }
    return a; // nullptr if no intersection (both reach nullptr simultaneously)
}
```

**Why it works:** If list A has length `la` and list B has length `lb`, pointer `a` traverses `la + lb` nodes (its own list then B) and so does pointer `b`. They synchronize at the intersection node (or at `nullptr` if no intersection).

**Important:** Check `a != b` with reference equality (pointer equality), not value equality.

---

## Template 3 — Rotate List (LC 61)

```cpp
#include <bits/stdc++.h>
using namespace std;

ListNode* rotateRight(ListNode* head, int k) {
    if (head == nullptr || head->next == nullptr || k == 0) return head;

    // Find length and tail
    int length = 1;
    ListNode* tail = head;
    while (tail->next != nullptr) { tail = tail->next; length++; }

    k = k % length;
    if (k == 0) return head;

    // Find new tail (length - k - 1 steps from head)
    ListNode* newTail = head;
    for (int i = 0; i < length - k - 1; i++) newTail = newTail->next;

    ListNode* newHead = newTail->next;
    newTail->next = nullptr;
    tail->next = head; // connect old tail to old head
    return newHead;
}
```

**Key:** Always compute `k % length` first — rotating by `length` is a no-op.

---

## Template 4 — Partition List (LC 86)

Partition list around value `x`: all nodes < x before nodes >= x, preserving relative order.

```cpp
#include <bits/stdc++.h>
using namespace std;

ListNode* partition(ListNode* head, int x) {
    ListNode* lessHead = new ListNode(0), *greaterHead = new ListNode(0);
    ListNode* less = lessHead, *greater = greaterHead;

    while (head != nullptr) {
        if (head->val < x) { less->next = head; less = less->next; }
        else               { greater->next = head; greater = greater->next; }
        head = head->next;
    }

    greater->next = nullptr; // CRITICAL: terminate — avoids a cycle if original tail < x
    less->next = greaterHead->next;
    return lessHead->next;
}
```

**Critical:** Set `greater->next = nullptr` before connecting. If the last node was already in `less`, `greater->next` still points into the original list, creating a cycle.

---

## Template 5 — Swapping Nodes (LC 1721)

Swap the kth node from beginning and kth from end (by value, not relinking):

```cpp
#include <bits/stdc++.h>
using namespace std;

ListNode* swapNodes(ListNode* head, int k) {
    ListNode* first = head, *second = head, *curr = head;

    // Advance curr k-1 steps to kth node from beginning
    for (int i = 1; i < k; i++) curr = curr->next;
    first = curr; // kth from beginning

    // Find kth from end: advance curr to end, second follows
    while (curr->next != nullptr) {
        curr = curr->next;
        second = second->next;
    }
    // second is now kth from end

    int tmp = first->val;
    first->val = second->val;
    second->val = tmp;
    return head;
}
```

---

## Template 6 — Linked List Cycle II via Two-Pointer Length

Alternative to Floyd's for cycle entry: find cycle length `c`, then use two pointers with gap `c`.

```cpp
#include <bits/stdc++.h>
using namespace std;

ListNode* detectCycle(ListNode* head) {
    int cycleLen = getCycleLength(head);
    if (cycleLen == 0) return nullptr;

    ListNode* p1 = head, *p2 = head;
    for (int i = 0; i < cycleLen; i++) p1 = p1->next; // advance p1 by cycleLen

    while (p1 != p2) {
        p1 = p1->next;
        p2 = p2->next;
    }
    return p1;
}
```

---

## Pointer Advancement Cheatsheet

| Goal | Steps to advance fast pointer |
|------|------------------------------|
| Find middle | n/2 (handled by fast = 2x speed) |
| k-th from end | k steps ahead of slow |
| (k-1)-th from end (node before target) | k+1 steps ahead |
| Intersection | restart from other head when null |

---

## Common Pitfalls

| Mistake | Fix |
|---------|-----|
| Off-by-one: slow lands on target, not before it | Advance fast `n+1` times for "node before" |
| Intersection: using `headA` vs `headB` for restart incorrectly | When `a` is null, redirect to `headB` (not `headA`) |
| Rotate: not handling `k % length == 0` | Return `head` immediately if no rotation needed |
| Partition: `greater->next` not nulled — creates cycle | Always null-terminate the greater list |

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Nth from end | O(n) | O(1) |
| Intersection | O(m + n) | O(1) |
| Rotate | O(n) | O(1) |
| Partition | O(n) | O(1) |
| Swap kth nodes | O(n) | O(1) |

---

## Related Patterns

- [Fast & Slow Pointer](./Fast%20and%20Slow%20Pointer.md) — different two-pointer flavor (speed-based)
- [Cycle Detection](./Cycle%20Detection.md) — Floyd's for cycle entry
- [Arrays — Two Pointers](../../1.Arrays/Patterns/Two%20Pointers.md) — array analogue

---

**Back:** [Linked List README](../README.md) | **Prev:** [Cycle Detection](./Cycle%20Detection.md)

> **Last Updated:** 2026-06-26
