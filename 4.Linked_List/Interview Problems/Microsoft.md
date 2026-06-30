# Microsoft Interview Problems — Linked List

> **Topic:** [Linked List](../README.md) · **Section:** Interview Problems
> **Last Updated:** 2026-06-26

---

## Question Bank

| # | Problem | Level | Difficulty | Frequency | Pattern | LeetCode |
|---|---------|-------|-----------|----------|---------|---------|
| 1 | LRU Cache | SDE II | Hard | ⭐⭐⭐⭐ | [Design](../Design%20Data%20Structure%20Problems/LRU%20Cache.md) | [146](https://leetcode.com/problems/lru-cache/) |
| 2 | Merge Two Sorted Lists | SDE I | Easy | ⭐⭐⭐⭐ | [Merge](../Patterns/Merge%20Linked%20Lists.md) | [21](https://leetcode.com/problems/merge-two-sorted-lists/) |
| 3 | Remove Nth Node From End | SDE I | Medium | ⭐⭐⭐⭐ | [Two Pointers](../Patterns/Two%20Pointers%20on%20LL.md) | [19](https://leetcode.com/problems/remove-nth-node-from-end-of-list/) |
| 4 | Palindrome Linked List | SDE I | Easy | ⭐⭐⭐ | Fast&Slow + Reverse | [234](https://leetcode.com/problems/palindrome-linked-list/) |
| 5 | Sort List | SDE II | Medium | ⭐⭐⭐ | [Merge Sort](../Patterns/Merge%20Linked%20Lists.md) | [148](https://leetcode.com/problems/sort-list/) |

---

## Deep Dive: Palindrome Linked List (Optimal)

### Approach Comparison

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Convert to array | O(n) | O(n) | Simple but wastes space |
| Stack (push half) | O(n) | O(n/2) | Better but still O(n) |
| Fast/slow + reverse | O(n) | O(1) | **Optimal — expected in MS interview** |

### Full Solution

```java
public boolean isPalindrome(ListNode head) {
    if (head == null || head.next == null) return true;

    // Find end of first half
    ListNode slow = head, fast = head;
    while (fast.next != null && fast.next.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    // slow is now last node of first half

    // Reverse second half
    ListNode secondHalf = reverse(slow.next);
    ListNode p1 = head, p2 = secondHalf;
    boolean isPalin = true;

    while (p2 != null) {
        if (p1.val != p2.val) { isPalin = false; break; }
        p1 = p1.next;
        p2 = p2.next;
    }

    slow.next = reverse(secondHalf); // restore
    return isPalin;
}

private ListNode reverse(ListNode head) {
    ListNode prev = null, curr = head;
    while (curr != null) {
        ListNode nxt = curr.next;
        curr.next = prev;
        prev = curr;
        curr = nxt;
    }
    return prev;
}
```

**Microsoft follow-up:** "What if you couldn't modify the list?" → Use a stack to store the second half, then compare.

---

## Deep Dive: Sort List (Bottom-Up Merge Sort)

Microsoft occasionally asks for the **O(1) space** sort — bottom-up merge sort avoids call stack:

```java
// Key insight: iterate sublist sizes 1, 2, 4, 8, ...
// For each size, merge adjacent pairs of sublists
for (int size = 1; size < length; size <<= 1) {
    // Merge pairs of sublists of this size
}
```

Full implementation in [Merge Linked Lists Pattern](../Patterns/Merge%20Linked%20Lists.md).

---

## Microsoft Interview Tips

- Proactively mention edge cases: `null` head, single node, two nodes
- For LRU Cache: mention `LinkedHashMap` exists in Java but implement from scratch
- Dry run your pointer manipulations out loud — draw boxes and arrows if on whiteboard

---

## Related Files

- [Microsoft OA-Qns](../OA-Qns/Microsoft.md)
- [Linked List README](../README.md)

> **Last Updated:** 2026-06-26
