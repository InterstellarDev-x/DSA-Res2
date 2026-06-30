# XOR Linked List

> **Topic:** [Bit Manipulation](../README.md) · **Section:** Design Data Structure Problems
> **Concept:** Memory-efficient doubly linked list using XOR of adjacent addresses

---

## Concept

A regular doubly linked list stores both `prev` and `next` pointers per node — 2 pointers per node.

An **XOR Linked List** stores only `both = prev XOR next` per node — 1 pointer per node. To traverse forward, XOR `both` with the known `prev` address to get `next`. To traverse backward, XOR with known `next` to get `prev`.

```
node.both = address(prev) XOR address(next)
next = node.both XOR address(prev)
prev = node.both XOR address(next)
```

**Memory saved:** 50% pointer storage vs standard DLL.

---

## Java Note

Java has no explicit pointer/address access (`&addr`). XOR linked lists are a **C/C++ data structure**. In Java interviews, the concept is tested theoretically:
- Explain the XOR trick
- Analyze space complexity
- Compare with standard DLL

This is a **knowledge / system design question**, not an implementation question in Java.

---

## Theoretical Implementation (C-style pseudocode)

```
insert(head, val):
    newNode.both = 0 XOR address(head)  // prev=null, next=head
    head.both = address(newNode) XOR (head.both XOR 0)  // update head's prev
    head = newNode

traverse(head):
    prev = null
    curr = head
    while curr != null:
        print curr.val
        next = prev XOR curr.both
        prev = curr
        curr = next
```

---

## Complexity Comparison

| Aspect | Standard DLL | XOR Linked List |
|--------|-------------|-----------------|
| Pointer space | 2 per node | 1 per node |
| Traversal | O(n) | O(n) |
| Insert at head | O(1) | O(1) |
| Delete node | O(1) with prev ptr | O(1) if traversed to node |
| Language support | All | C/C++ only |
| Debuggability | Easy | Difficult — addresses aren't human-readable |
| Practical use | Common | Rare (embedded/constrained memory) |

---

## When Asked in Interviews

**Pattern:** "How would you design a doubly linked list using half the memory?"

**Expected answer:**
1. Describe XOR trick: `both = prev_addr XOR next_addr`
2. Show how to recover next: `next = curr.both XOR prev_addr`
3. Mention Java limitation: requires unsafe memory access
4. Acknowledge practical tradeoffs: debugging difficulty, no garbage collection safety

---

## Related Files

- [Bitset](./Bitset.md)
- [XOR Tricks](../Patterns/XOR%20Tricks.md)
- [Linked List README](../../4.Linked_List/README.md)

---

**Back:** [Bit Manipulation README](../README.md)

> **Last Updated:** 2026-06-26
