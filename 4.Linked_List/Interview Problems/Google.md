# Google Interview Problems — Linked List

> **Topic:** [Linked List](../README.md) · **Section:** Interview Problems
> **Last Updated:** 2026-06-26

---

## Question Bank

| # | Problem | Level | Difficulty | Frequency | Pattern | LeetCode |
|---|---------|-------|-----------|----------|---------|---------|
| 1 | Reverse Nodes in K-Group | L4–L6 | Hard | ⭐⭐⭐⭐⭐ | [Reverse](../Patterns/Reverse%20Linked%20List.md) | [25](https://leetcode.com/problems/reverse-nodes-in-k-group/) |
| 2 | Merge K Sorted Lists | L4–L5 | Hard | ⭐⭐⭐⭐⭐ | [Merge + D&C](../Patterns/Merge%20Linked%20Lists.md) | [23](https://leetcode.com/problems/merge-k-sorted-lists/) |
| 3 | Find the Duplicate Number (O(1) space) | L4 | Medium | ⭐⭐⭐⭐ | [Floyd's](../Patterns/Cycle%20Detection.md) | [287](https://leetcode.com/problems/find-the-duplicate-number/) |
| 4 | LFU Cache | L5–L6 | Hard | ⭐⭐⭐ | [Design](../Design%20Data%20Structure%20Problems/LFU%20Cache.md) | [460](https://leetcode.com/problems/lfu-cache/) |
| 5 | Flatten Multilevel Doubly LL | L4 | Medium | ⭐⭐ | DFS | [430](https://leetcode.com/problems/flatten-a-multilevel-doubly-linked-list/) |

---

## Deep Dive: Reverse K-Group (Google Favorite)

### Common Follow-ups at Google

**Q1: What if k > length of remaining list?**
Leave them in original order. The check `getKth(groupPrev, k) == null` handles this.

**Q2: Can you do it recursively?**

```java
public ListNode reverseKGroup(ListNode head, int k) {
    // Check if k nodes exist
    ListNode node = head;
    for (int i = 0; i < k; i++) {
        if (node == null) return head; // < k nodes, no reversal
        node = node.next;
    }

    // Reverse k nodes
    ListNode prev = null, curr = head;
    for (int i = 0; i < k; i++) {
        ListNode nxt = curr.next;
        curr.next = prev;
        prev = curr;
        curr = nxt;
    }

    // head is now tail of reversed group; connect to recursed rest
    head.next = reverseKGroup(curr, k);
    return prev; // prev is new head
}
```

Space: O(n/k) call stack. Iterative is O(1) — mention the tradeoff.

**Q3: What's the time complexity?**
O(n) — every node is visited exactly twice (once to check if k remain, once to reverse).

---

## Deep Dive: Find Duplicate Number (Floyd's)

### The Array-as-Graph Insight

- Array contains n+1 integers in `[1, n]`
- Treat index as node, `nums[index]` as the "next" pointer
- `nums[0]` = "head" (index 0 always points somewhere in `[1, n]`)
- Duplicate value = two indices pointing to the same next = cycle exists

```
nums = [1, 3, 4, 2, 2]
index:  0  1  2  3  4
next:   1  3  4  2  2
Graph: 0→1→3→2→4→2 (cycle at 2)
```

```java
public int findDuplicate(int[] nums) {
    int slow = nums[0], fast = nums[nums[0]];
    while (slow != fast) {
        slow = nums[slow];
        fast = nums[nums[fast]];
    }
    slow = 0;
    while (slow != fast) {
        slow = nums[slow];
        fast = nums[fast];
    }
    return slow;
}
```

**Google follow-up:** What if there are multiple duplicates? This Floyd's approach only finds one. For all duplicates, sort or use frequency map.

---

## Google Interview Framework

Google values **abstraction and generalization**:
1. State brute force first, then optimize
2. Identify the invariant (what's always true at each step of the loop)
3. Explain why your solution terminates (especially for while loops)
4. Trace through an example with 3–4 nodes minimum

---

## Related Files

- [Google OA-Qns](../OA-Qns/Google.md)
- [Linked List README](../README.md)

> **Last Updated:** 2026-06-26
