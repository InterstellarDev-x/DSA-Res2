# Merge Linked Lists

> **Topic:** [Linked List](../README.md) · **Section:** Patterns
> **Applies to:** Merge two sorted, merge K sorted, sort list, add two numbers

---

## Core Idea

Use a **dummy head** node so you never special-case the head pointer. Maintain a `tail` pointer. Always attach the smaller current node to `tail.next`, then advance `tail`.

```java
ListNode dummy = new ListNode(0);
ListNode tail = dummy;
// ... merge logic ...
return dummy.next;
```

---

## Template 1 — Merge Two Sorted Lists (LC 21)

```java
public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(0), tail = dummy;
    while (l1 != null && l2 != null) {
        if (l1.val <= l2.val) { tail.next = l1; l1 = l1.next; }
        else                  { tail.next = l2; l2 = l2.next; }
        tail = tail.next;
    }
    tail.next = (l1 != null) ? l1 : l2; // attach remaining
    return dummy.next;
}
```

**Time:** O(n + m). **Space:** O(1) — reuses existing nodes.

---

## Template 2 — Merge K Sorted Lists (LC 23)

**Approach 1: Min-Heap (best for large k)**

```java
public ListNode mergeKLists(ListNode[] lists) {
    PriorityQueue<ListNode> pq = new PriorityQueue<>((a, b) -> a.val - b.val);
    for (ListNode node : lists) {
        if (node != null) pq.offer(node);
    }

    ListNode dummy = new ListNode(0), tail = dummy;
    while (!pq.isEmpty()) {
        ListNode curr = pq.poll();
        tail.next = curr;
        tail = tail.next;
        if (curr.next != null) pq.offer(curr.next);
    }
    return dummy.next;
}
```

**Time:** O(N log k) where N = total nodes, k = number of lists. **Space:** O(k) for the heap.

**Approach 2: Divide & Conquer (better constant factor)**

```java
public ListNode mergeKLists(ListNode[] lists) {
    if (lists.length == 0) return null;
    return mergeRange(lists, 0, lists.length - 1);
}

private ListNode mergeRange(ListNode[] lists, int lo, int hi) {
    if (lo == hi) return lists[lo];
    int mid = lo + (hi - lo) / 2;
    ListNode left = mergeRange(lists, lo, mid);
    ListNode right = mergeRange(lists, mid + 1, hi);
    return mergeTwoLists(left, right);
}
```

**Time:** O(N log k). **Space:** O(log k) call stack.

---

## Template 3 — Sort List (LC 148)

Merge sort on a linked list: O(n log n) time, O(log n) space (call stack only, no extra array).

```java
public ListNode sortList(ListNode head) {
    if (head == null || head.next == null) return head;

    // Split at middle
    ListNode mid = getMid(head);
    ListNode rightHead = mid.next;
    mid.next = null; // cut the list

    ListNode left = sortList(head);
    ListNode right = sortList(rightHead);
    return mergeTwoLists(left, right);
}

private ListNode getMid(ListNode head) {
    ListNode slow = head, fast = head.next; // fast=head.next → slow ends at first mid
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow;
}
```

**Bottom-up iterative version (O(1) space):**

```java
public ListNode sortList(ListNode head) {
    int length = 0;
    ListNode node = head;
    while (node != null) { length++; node = node.next; }

    ListNode dummy = new ListNode(0);
    dummy.next = head;

    for (int size = 1; size < length; size <<= 1) {
        ListNode curr = dummy.next, tail = dummy;
        while (curr != null) {
            ListNode left = curr;
            ListNode right = split(left, size);
            curr = split(right, size);
            ListNode[] merged = mergeAndGetTail(left, right);
            tail.next = merged[0];
            tail = merged[1];
        }
    }
    return dummy.next;
}

private ListNode split(ListNode head, int size) {
    for (int i = 1; head != null && i < size; i++) head = head.next;
    if (head == null) return null;
    ListNode rest = head.next;
    head.next = null;
    return rest;
}

private ListNode[] mergeAndGetTail(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(0), tail = dummy;
    while (l1 != null && l2 != null) {
        if (l1.val <= l2.val) { tail.next = l1; l1 = l1.next; }
        else                  { tail.next = l2; l2 = l2.next; }
        tail = tail.next;
    }
    tail.next = (l1 != null) ? l1 : l2;
    while (tail.next != null) tail = tail.next;
    return new ListNode[]{dummy.next, tail};
}
```

---

## Template 4 — Add Two Numbers (LC 2)

Numbers stored in **forward order** (digits head to tail = least to most significant).

```java
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(0), tail = dummy;
    int carry = 0;
    while (l1 != null || l2 != null || carry != 0) {
        int sum = carry;
        if (l1 != null) { sum += l1.val; l1 = l1.next; }
        if (l2 != null) { sum += l2.val; l2 = l2.next; }
        carry = sum / 10;
        tail.next = new ListNode(sum % 10);
        tail = tail.next;
    }
    return dummy.next;
}
```

**Add Two Numbers II (LC 445 — digits in reverse order):** Push both lists onto stacks, then pop + add with carry.

```java
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
    Deque<Integer> s1 = new ArrayDeque<>(), s2 = new ArrayDeque<>();
    while (l1 != null) { s1.push(l1.val); l1 = l1.next; }
    while (l2 != null) { s2.push(l2.val); l2 = l2.next; }
    int carry = 0;
    ListNode head = null;
    while (!s1.isEmpty() || !s2.isEmpty() || carry != 0) {
        int sum = carry;
        if (!s1.isEmpty()) sum += s1.pop();
        if (!s2.isEmpty()) sum += s2.pop();
        carry = sum / 10;
        ListNode node = new ListNode(sum % 10);
        node.next = head; // prepend (building in reverse)
        head = node;
    }
    return head;
}
```

---

## Template 5 — Reorder List (LC 143)

L0 → L1 → L2 → ... → Ln  becomes  L0 → Ln → L1 → Ln-1 → L2 → ...

```java
public void reorderList(ListNode head) {
    // 1. Find mid (use fast=head.next to get first mid)
    ListNode slow = head, fast = head.next;
    while (fast != null && fast.next != null) {
        slow = slow.next; fast = fast.next.next;
    }

    // 2. Reverse second half
    ListNode second = slow.next;
    slow.next = null; // cut
    ListNode prev = null, curr = second;
    while (curr != null) {
        ListNode nxt = curr.next; curr.next = prev; prev = curr; curr = nxt;
    }

    // 3. Interleave merge
    ListNode first = head;
    second = prev;
    while (second != null) {
        ListNode tmp1 = first.next, tmp2 = second.next;
        first.next = second;
        second.next = tmp1;
        first = tmp1;
        second = tmp2;
    }
}
```

---

## Common Pitfalls

| Mistake | Fix |
|---------|-----|
| Not initializing `tail = dummy` — dereferencing null | Set `ListNode tail = dummy` immediately |
| In Add Two Numbers: forgetting final carry | Loop condition is `l1 != null \|\| l2 != null \|\| carry != 0` |
| Heap comparator `a.val - b.val` can overflow for large ints | Use `Integer.compare(a.val, b.val)` or `a.val < b.val ? -1 : ...` |
| Sort list: not cutting the list at mid | `mid.next = null` before recursive calls |

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Merge two sorted | O(n + m) | O(1) |
| Merge K sorted (heap) | O(N log k) | O(k) |
| Merge K sorted (D&C) | O(N log k) | O(log k) |
| Sort list (top-down) | O(n log n) | O(log n) |
| Sort list (bottom-up) | O(n log n) | O(1) |
| Add two numbers | O(max(n, m)) | O(max(n, m)) |

---

## Related Patterns

- [Reverse Linked List](./Reverse%20Linked%20List.md) — used in reorder list
- [Fast & Slow Pointer](./Fast%20and%20Slow%20Pointer.md) — split list for merge sort
- [Heaps](../../9.Heaps/README.md) — min-heap for k-way merge

---

**Back:** [Linked List README](../README.md) | **Prev:** [Reverse Linked List](./Reverse%20Linked%20List.md) | **Next:** [Cycle Detection](./Cycle%20Detection.md)

> **Last Updated:** 2026-06-26
