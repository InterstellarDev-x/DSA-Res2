# Fast & Slow Pointer (Floyd's Tortoise and Hare)

> **Topic:** [Linked List](../README.md) · **Section:** Patterns
> **Applies to:** Cycle detection, middle-finding, k-th from end variants

---

## Core Idea

Two pointers start at `head`. Slow moves 1 step; fast moves 2 steps.

- **No cycle:** Fast hits `null`. Slow is at middle.
- **Cycle present:** They meet inside the cycle.

```
slow: head → 1 → 2 → 3  (3 steps for n=6)
fast: head → 2 → 4 → 6  (6 steps, wraps if cycle)
```

---

## Template 1 — Find Middle

```java
public ListNode middleNode(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow; // for even length, returns SECOND middle
}
```

**Variant — first middle (for palindrome check):**
```java
// Advance fast one step before the loop to bias toward first middle
ListNode slow = head, fast = head.next;
while (fast != null && fast.next != null) { ... }
// slow is now the last node of first half — split here
```

---

## Template 2 — Detect Cycle (LC 141)

```java
public boolean hasCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) return true;
    }
    return false;
}
```

**Why they always meet:** If cycle length = c, when slow enters the cycle, fast is d steps ahead. Each iteration, fast gains 1 step relative to slow. They meet in at most c iterations. Time: O(n). Space: O(1).

---

## Template 3 — Cycle Entry Point (LC 142)

```java
public ListNode detectCycle(ListNode head) {
    ListNode slow = head, fast = head;
    // Phase 1: find meeting point
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) break;
    }
    if (fast == null || fast.next == null) return null; // no cycle

    // Phase 2: find entry — reset one pointer to head
    slow = head;
    while (slow != fast) {
        slow = slow.next;
        fast = fast.next; // both move at 1x now
    }
    return slow; // entry node
}
```

**Mathematical proof:** Let head→entry = F, entry→meeting = a, cycle length = c.
- slow walked: F + a
- fast walked: F + a + n×c (full loops)
- fast = 2×slow → F + a + nc = 2(F + a) → F = nc - a
- From meeting point, moving fast F steps lands exactly at entry (nc − a steps back in cycle = entry).

---

## Template 4 — Happy Number (LC 202)

```java
public boolean isHappy(int n) {
    int slow = n, fast = sumOfSquares(n);
    while (fast != 1 && slow != fast) {
        slow = sumOfSquares(slow);
        fast = sumOfSquares(sumOfSquares(fast));
    }
    return fast == 1;
}

private int sumOfSquares(int n) {
    int sum = 0;
    while (n > 0) {
        int d = n % 10;
        sum += d * d;
        n /= 10;
    }
    return sum;
}
```

---

## Template 5 — Reorder List (LC 143)

Uses fast/slow to split, then reverse second half, then merge:

```java
public void reorderList(ListNode head) {
    if (head == null || head.next == null) return;

    // Step 1: find mid
    ListNode slow = head, fast = head;
    while (fast.next != null && fast.next.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }

    // Step 2: reverse second half
    ListNode prev = null, curr = slow.next;
    slow.next = null; // cut
    while (curr != null) {
        ListNode nxt = curr.next;
        curr.next = prev;
        prev = curr;
        curr = nxt;
    }

    // Step 3: merge alternating
    ListNode l1 = head, l2 = prev;
    while (l2 != null) {
        ListNode tmp1 = l1.next, tmp2 = l2.next;
        l1.next = l2;
        l2.next = tmp1;
        l1 = tmp1;
        l2 = tmp2;
    }
}
```

---

## Common Pitfalls

| Mistake | Fix |
|---------|-----|
| `while (fast != null)` only — misses single-node lists | Always check `fast != null && fast.next != null` |
| Not breaking after meeting in cycle detection | Break when `slow == fast`, then run phase 2 |
| Resetting only `slow` in phase 2 — fast stays at meeting | Reset `slow = head`; fast stays at meeting point, both move 1x |
| Even-length middle bias: first vs second | Use `fast = head.next` init for "first" middle |

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Find middle | O(n) | O(1) |
| Detect cycle | O(n) | O(1) |
| Find cycle entry | O(n) | O(1) |
| Reorder list | O(n) | O(1) |

---

## Related Patterns

- [Reverse Linked List](./Reverse%20Linked%20List.md) — used after finding middle in palindrome/reorder
- [Cycle Detection](./Cycle%20Detection.md) — deep dive on Floyd's with duplicate-number variant
- [Two Pointers on LL](./Two%20Pointers%20on%20LL.md) — nth-from-end gap trick

---

**Back:** [Linked List README](../README.md) | **Next Pattern:** [Reverse Linked List](./Reverse%20Linked%20List.md)

> **Last Updated:** 2026-06-26
