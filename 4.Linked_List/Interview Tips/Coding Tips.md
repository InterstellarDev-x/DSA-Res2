# Coding Tips — Linked List

> **Topic:** [Linked List](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## Java Node Definitions

```java
// Singly Linked List
class ListNode {
    int val;
    ListNode next;
    ListNode(int val) { this.val = val; }
    ListNode(int val, ListNode next) { this.val = val; this.next = next; }
}

// Doubly Linked List (for LRU/LFU)
class Node {
    int key, val;
    Node prev, next;
    Node(int k, int v) { key = k; val = v; }
}
```

---

## The Dummy Node Idiom

Always use a dummy head when the result list's head might change:

```java
// BAD — head can change, requires if (head == null) checks everywhere
ListNode head = null, tail = null;
if (head == null) head = tail = new ListNode(val);
else { tail.next = new ListNode(val); tail = tail.next; }

// GOOD — dummy head never changes; result is always dummy.next
ListNode dummy = new ListNode(0), tail = dummy;
tail.next = new ListNode(val);
tail = tail.next;
return dummy.next;
```

Use dummy head for: merge, partition, remove nodes, reverse sublist.

---

## Pointer Manipulation Order

When relinking pointers, always save references **before** overwriting:

```java
// Reverse step — save nxt BEFORE breaking curr.next
ListNode nxt = curr.next; // 1. save
curr.next = prev;          // 2. relink (curr.next is now gone)
prev = curr;               // 3. advance prev
curr = nxt;                // 4. advance curr using saved reference
```

The order `save → relink → advance` prevents losing nodes.

---

## Linked List Length

Count once and store — don't recount:

```java
int length = 0;
ListNode curr = head;
while (curr != null) { length++; curr = curr.next; }
// Reuse length, don't traverse again
```

---

## Avoid Null Pointer in Fast Pointer

Fast pointer skips 2 nodes per step — always check both `fast` and `fast.next`:

```java
while (fast != null && fast.next != null) {
    slow = slow.next;
    fast = fast.next.next; // safe because fast.next != null checked above
}
```

If you check only `fast != null`, `fast.next.next` can throw NPE.

---

## Even vs Odd Length Bias in Mid-Finding

The init of `fast` controls which mid you get for even-length lists:

```java
// fast = head → second middle (default for most problems)
ListNode slow = head, fast = head;
// fast = head.next → first middle (for palindrome: split after first half)
ListNode slow = head, fast = head.next;
```

For `[1, 2, 3, 4]`:
- `fast = head`: slow ends at 3 (second middle)
- `fast = head.next`: slow ends at 2 (first middle)

---

## In-Place Tricks

**Delete Node without head reference (LC 237):**
```java
// Copy next node's value, then skip next node
node.val = node.next.val;
node.next = node.next.next;
```

**Cycle detection phase 2 — after meeting, both pointers move 1x:**
```java
slow = head; // reset to head
while (slow != fast) { slow = slow.next; fast = fast.next; }
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

## Common Java APIs

```java
// Deque as stack (preferred over Stack class)
Deque<Integer> stack = new ArrayDeque<>();
stack.push(x);    // = addFirst
stack.pop();      // = removeFirst
stack.peek();     // = peekFirst

// PriorityQueue for min-heap (Merge K Sorted)
PriorityQueue<ListNode> pq = new PriorityQueue<>((a, b) -> a.val - b.val);
// Safe comparator (no overflow):
PriorityQueue<ListNode> pq = new PriorityQueue<>(Comparator.comparingInt(a -> a.val));
```

---

## Related Files

- [Common Mistakes](./Common%20Mistakes.md)
- [Dummy Node Technique](./Dummy%20Node%20Technique.md)
- [Complexity Analysis](./Complexity%20Analysis.md)

> **Last Updated:** 2026-06-26
