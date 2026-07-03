# Coding Tips — Linked List

> **Topic:** [Linked List](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## C++ Node Definitions

```cpp
#include <bits/stdc++.h>
using namespace std;

// Singly Linked List
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int val) : val(val), next(nullptr) {}
    ListNode(int val, ListNode* next) : val(val), next(next) {}
};

// Doubly Linked List (for LRU/LFU)
struct Node {
    int key, val;
    Node* prev;
    Node* next;
    Node(int k, int v) : key(k), val(v), prev(nullptr), next(nullptr) {}
};
```

---

## The Dummy Node Idiom

Always use a dummy head when the result list's head might change:

```cpp
// BAD — head can change, requires if (head == nullptr) checks everywhere
ListNode* head = nullptr, *tail = nullptr;
if (head == nullptr) head = tail = new ListNode(val);
else { tail->next = new ListNode(val); tail = tail->next; }

// GOOD — dummy head never changes; result is always dummy->next
ListNode* dummy = new ListNode(0), *tail = dummy;
tail->next = new ListNode(val);
tail = tail->next;
return dummy->next;
```

Use dummy head for: merge, partition, remove nodes, reverse sublist.

---

## Pointer Manipulation Order

When relinking pointers, always save references **before** overwriting:

```cpp
// Reverse step — save nxt BEFORE breaking curr->next
ListNode* nxt = curr->next; // 1. save
curr->next = prev;           // 2. relink (curr->next is now gone)
prev = curr;                 // 3. advance prev
curr = nxt;                  // 4. advance curr using saved reference
```

The order `save → relink → advance` prevents losing nodes.

---

## Linked List Length

Count once and store — don't recount:

```cpp
int length = 0;
ListNode* curr = head;
while (curr != nullptr) { length++; curr = curr->next; }
// Reuse length, don't traverse again
```

---

## Avoid Null Pointer in Fast Pointer

Fast pointer skips 2 nodes per step — always check both `fast` and `fast->next`:

```cpp
while (fast != nullptr && fast->next != nullptr) {
    slow = slow->next;
    fast = fast->next->next; // safe because fast->next != nullptr checked above
}
```

If you check only `fast != nullptr`, `fast->next->next` causes undefined behavior (segfault).

---

## Even vs Odd Length Bias in Mid-Finding

The init of `fast` controls which mid you get for even-length lists:

```cpp
// fast = head → second middle (default for most problems)
ListNode* slow = head, *fast = head;
// fast = head->next → first middle (for palindrome: split after first half)
ListNode* slow = head, *fast = head->next;
```

For `[1, 2, 3, 4]`:
- `fast = head`: slow ends at 3 (second middle)
- `fast = head->next`: slow ends at 2 (first middle)

---

## In-Place Tricks

**Delete Node without head reference (LC 237):**
```cpp
// Copy next node's value, then skip next node
node->val = node->next->val;
node->next = node->next->next;
```

**Cycle detection phase 2 — after meeting, both pointers move 1x:**
```cpp
slow = head; // reset to head
while (slow != fast) { slow = slow->next; fast = fast->next; }
```

---

## Recursion vs Iteration Trade-off

| | Iterative | Recursive |
|--|-----------|-----------|
| Space | O(1) | O(n) stack |
| Readability | Lower | Higher |
| Preferred in interviews | ✅ Usually | Only if asked |

Exception: tree-like problems (flatten multilevel DLL) are naturally recursive.

---

## Common C++ APIs

```cpp
#include <bits/stdc++.h>
using namespace std;

// stack<T> (preferred for LIFO)
stack<int> stk;
stk.push(x);  // push
stk.pop();    // pop (no return value)
stk.top();    // peek

// priority_queue for min-heap (Merge K Sorted)
auto cmp = [](ListNode* a, ListNode* b) { return a->val > b->val; };
priority_queue<ListNode*, vector<ListNode*>, decltype(cmp)> pq(cmp);
// Safe comparator (no overflow):
// lambda-based comparator above works fine and avoids subtraction overflow
```

---

## Related Files

- [Common Mistakes](./Common%20Mistakes.md)
- [Dummy Node Technique](./Dummy%20Node%20Technique.md)
- [Complexity Analysis](./Complexity%20Analysis.md)

> **Last Updated:** 2026-06-26
