# Common Mistakes — Linked List

> **Topic:** [Linked List](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## 1. Fast/Slow Pointer — Wrong Loop Termination

```java
// BUG: only checks fast
while (fast != null) {
    slow = slow.next;
    fast = fast.next.next; // NPE if fast.next is null
}

// FIX
while (fast != null && fast.next != null) { ... }
```

---

## 2. Reverse — Forgetting to Save Next

```java
// BUG: curr.next is lost
curr.next = prev;
prev = curr;
curr = curr.next; // curr.next is now prev, not the original next!

// FIX: save before relinking
ListNode nxt = curr.next;
curr.next = prev;
prev = curr;
curr = nxt;
```

---

## 3. K-Group Reverse — Wrong groupPrev Advancement

```java
// BUG: advancing groupPrev to kth (new head of group) instead of old head (new tail)
groupPrev = kth; // wrong — skips to new head

// FIX: tmp = groupPrev.next (old first = new tail of reversed group)
ListNode tmp = groupPrev.next;
groupPrev.next = kth; // kth is new head
groupPrev = tmp;       // advance to new tail
```

---

## 4. Partition List — Not Null-Terminating Greater List

```java
// BUG: greater.next may still point into the original list → cycle
less.next = greaterHead.next;
return lessHead.next;

// FIX: terminate before connecting
greater.next = null; // MUST do this first
less.next = greaterHead.next;
return lessHead.next;
```

---

## 5. Remove Nth from End — Off-by-One on Gap

```java
// BUG: advance fast n times → slow lands ON the target, not before it
for (int i = 0; i < n; i++) fast = fast.next;

// FIX: advance n+1 times → slow lands one before target
for (int i = 0; i <= n; i++) fast = fast.next;
// Also: use dummy head so removing the first node works uniformly
```

---

## 6. LRU Cache — Not Storing Key in Node

```java
// BUG: can't evict LRU from HashMap without knowing its key
Node lru = tail.prev;
map.remove(???); // don't know the key!

// FIX: store key in Node
class Node { int key, val; ... }
map.remove(lru.key); // works
```

---

## 7. Floyd's Phase 2 — Moving Fast at 2x

```java
// BUG: in phase 2, fast still moves 2x
slow = head;
while (slow != fast) {
    slow = slow.next;
    fast = fast.next.next; // wrong — phase 2 needs 1x speed
}

// FIX: both move 1x in phase 2
while (slow != fast) {
    slow = slow.next;
    fast = fast.next;
}
```

---

## 8. Intersecting Lists — Wrong Head Assignment on Restart

```java
// BUG: a restarts to headA, b restarts to headB (no progress)
a = (a != null) ? a.next : headA;
b = (b != null) ? b.next : headB;

// FIX: each restarts to the OTHER list's head
a = (a != null) ? a.next : headB;
b = (b != null) ? b.next : headA;
```

---

## 9. Cycle Detection — Checking Before Advancing

```java
// BUG: slow == fast is true at start (both at head) → false positive no-cycle
if (slow == fast) return true; // before any movement!
while (fast != null && fast.next != null) { ... }

// FIX: check AFTER advancing
while (fast != null && fast.next != null) {
    slow = slow.next;
    fast = fast.next.next;
    if (slow == fast) return true; // check after movement
}
```

---

## 10. Recursion Stack Overflow

Recursive reverse / recursive DFS on large inputs (n = 10^5) will StackOverflow.

```java
// RISKY for large input
ListNode reverseList(ListNode head) {
    if (head == null || head.next == null) return head;
    ListNode rest = reverseList(head.next); // O(n) stack depth
    ...
}

// SAFE
ListNode reverseList(ListNode head) {
    ListNode prev = null, curr = head; // iterative O(1) space
    while (curr != null) { ... }
    return prev;
}
```

Default JVM stack depth: ~8000–10000 frames. A list of length 10^5 will overflow.

---

## 11. Heap Comparator Integer Overflow

```java
// BUG: a.val - b.val overflows for large negative/positive pairs
new PriorityQueue<>((a, b) -> a.val - b.val);

// FIX: use Integer.compare or Comparator.comparingInt
new PriorityQueue<>(Comparator.comparingInt(node -> node.val));
```

---

## Quick Checklist Before Submitting

- [ ] Dummy head used where head might change?
- [ ] `fast != null && fast.next != null` (both conditions)?
- [ ] `nxt` saved before `curr.next = prev`?
- [ ] Greater list null-terminated in partition?
- [ ] Phase 2 of Floyd's uses 1x speed?
- [ ] Key stored in LRU/LFU node?

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Dummy Node Technique](./Dummy%20Node%20Technique.md)

> **Last Updated:** 2026-06-26
