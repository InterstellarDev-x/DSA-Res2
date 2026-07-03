# Reverse Linked List

> **Topic:** [Linked List](../README.md) · **Section:** Patterns
> **Applies to:** Full reverse, k-group reverse, palindrome check, in-place rotation

---

## Core Idea

Iterative three-pointer technique: `prev → curr → next`. Each step unlinks `curr` from its successor and attaches it to `prev`.

```
before: null ← prev  curr → next → ...
after:  null ← prev ← curr  next → ...
```

---

## Template 1 — Full Reverse (LC 206)

```cpp
ListNode* reverseList(ListNode* head) {
    ListNode* prev = nullptr, *curr = head;
    while (curr != nullptr) {
        ListNode* nxt = curr->next; // save before unlinking
        curr->next = prev;
        prev = curr;
        curr = nxt;
    }
    return prev; // new head
}
```

**Recursive variant:**
```cpp
ListNode* reverseList(ListNode* head) {
    if (head == nullptr || head->next == nullptr) return head;
    ListNode* newHead = reverseList(head->next);
    head->next->next = head; // node behind us points back to us
    head->next = nullptr;      // cut our old forward link
    return newHead;
}
```

> **Stack depth:** O(n) for recursion — prefer iterative in interviews unless asked.

---

## Template 2 — Reverse Between Positions (LC 92)

Reverse sublist from index `left` to `right` (1-indexed).

```cpp
ListNode* reverseBetween(ListNode* head, int left, int right) {
    ListNode* dummy = new ListNode(0);
    dummy->next = head;
    ListNode* pre = dummy;

    // Advance pre to node just before position left
    for (int i = 0; i < left - 1; i++) pre = pre->next;

    ListNode* curr = pre->next;
    for (int i = 0; i < right - left; i++) {
        ListNode* nxt = curr->next;
        curr->next = nxt->next;  // unlink nxt
        nxt->next = pre->next;   // nxt points to current sublist head
        pre->next = nxt;        // nxt becomes new sublist head
    }
    return dummy->next;
}
```

**Key:** This is a "front insertion" technique — each iteration inserts `nxt` at the front of the reversed section. No need to track a separate prev pointer.

---

## Template 3 — Reverse Nodes in K-Group (LC 25)

```cpp
ListNode* getKth(ListNode* curr, int k) {
    while (curr != nullptr && k > 0) {
        curr = curr->next;
        k--;
    }
    return curr;
}

ListNode* reverseKGroup(ListNode* head, int k) {
    ListNode* dummy = new ListNode(0);
    dummy->next = head;
    ListNode* groupPrev = dummy;

    while (true) {
        // Check if k nodes remain
        ListNode* kth = getKth(groupPrev, k);
        if (kth == nullptr) break;

        ListNode* groupNext = kth->next;

        // Reverse k nodes
        ListNode* prev = groupNext, *curr = groupPrev->next;
        while (curr != groupNext) {
            ListNode* nxt = curr->next;
            curr->next = prev;
            prev = curr;
            curr = nxt;
        }

        // Connect with rest of list
        ListNode* tmp = groupPrev->next; // was first, now last of group
        groupPrev->next = kth;           // kth is now head of reversed group
        groupPrev = tmp;
    }
    return dummy->next;
}
```

**Note:** If remaining nodes < k, they stay in original order — the `kth == nullptr` break handles this.

---

## Template 4 — Palindrome Linked List (LC 234)

```cpp
bool isPalindrome(ListNode* head) {
    // Step 1: find mid (first middle for odd, last of first half for even)
    ListNode* slow = head, *fast = head;
    while (fast->next != nullptr && fast->next->next != nullptr) {
        slow = slow->next;
        fast = fast->next->next;
    }

    // Step 2: reverse second half
    ListNode* secondHalfHead = reverseList(slow->next);

    // Step 3: compare
    ListNode* p1 = head, *p2 = secondHalfHead;
    bool result = true;
    while (p2 != nullptr) { // p2 is shorter or equal
        if (p1->val != p2->val) { result = false; break; }
        p1 = p1->next;
        p2 = p2->next;
    }

    // Step 4: restore (optional but good practice)
    slow->next = reverseList(secondHalfHead);
    return result;
}
```

---

## Template 5 — Swap Nodes in Pairs (LC 24)

```cpp
ListNode* swapPairs(ListNode* head) {
    ListNode* dummy = new ListNode(0);
    dummy->next = head;
    ListNode* prev = dummy;

    while (prev->next != nullptr && prev->next->next != nullptr) {
        ListNode* a = prev->next, *b = prev->next->next;
        prev->next = b;      // prev connects to b
        a->next = b->next;    // a skips b
        b->next = a;         // b points back to a
        prev = a;           // advance prev to end of swapped pair
    }
    return dummy->next;
}
```

---

## Visualization: k-Group Reversal

```
Before:   dummy → [1 → 2 → 3] → 4 → 5    (k=3)
           groupPrev                groupNext

After reversal:
          dummy → [3 → 2 → 1] → 4 → 5
          groupPrev → (was 1, now tail of group)
```

---

## Common Pitfalls

| Mistake | Fix |
|---------|-----|
| Forgetting `curr->next = prev` — pointer goes nowhere | Always save `nxt` before breaking `->next` |
| In k-group: not setting `groupPrev = tmp` (old first node) | Old first node is tail of reversed group — advance there |
| In reverseBetween: off-by-one on `pre` advancement | Loop `left - 1` times, not `left` times |
| Palindrome: modifying original without restoring | Reverse back after compare, or clone half |

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Full reverse | O(n) | O(1) iterative, O(n) recursive |
| Reverse between | O(n) | O(1) |
| K-group | O(n) | O(1) |
| Palindrome | O(n) | O(1) |
| Swap pairs | O(n) | O(1) |

---

## Related Patterns

- [Fast & Slow Pointer](./Fast%20and%20Slow%20Pointer.md) — used to find midpoint before reversing second half
- [Merge Linked Lists](./Merge%20Linked%20Lists.md) — after reversal, merge interleaved (reorder list)

---

**Back:** [Linked List README](../README.md) | **Prev:** [Fast & Slow Pointer](./Fast%20and%20Slow%20Pointer.md) | **Next:** [Merge Linked Lists](./Merge%20Linked%20Lists.md)

> **Last Updated:** 2026-06-26
