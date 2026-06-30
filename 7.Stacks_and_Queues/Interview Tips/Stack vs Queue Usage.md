# Stack vs Queue Usage Guide

> **Topic:** [Stacks & Queues](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## The Core Distinction

| Property | Stack | Queue |
|----------|-------|-------|
| Ordering | LIFO (Last-In-First-Out) | FIFO (First-In-First-Out) |
| Java class | `ArrayDeque` with `push/pop/peek` | `ArrayDeque` with `offer/poll/peek` |
| Conceptual model | Undo/history, DFS, nesting | Scheduling, BFS, buffering |
| Real-world analogy | Browser back button, call stack | Printer queue, task scheduler |

---

## When to Use a Stack

**Nesting and matching:**
- Bracket matching (Valid Parentheses)
- HTML/XML tag matching
- Nested function calls

**Undo / cancel last action:**
- Backspace string compare
- Text editor undo
- Browser back navigation

**DFS (Depth-First Search):**
- Explicit stack for iterative DFS on graphs/trees
- Backtracking problems

**Monotonic property queries:**
- Next greater/smaller element
- Daily temperatures, stock span

**Expression evaluation:**
- RPN evaluation (operand stack)
- Infix with parentheses (operator + result stacks)

**Path/context save-restore:**
- Simplify path (`..` = pop, name = push)
- Basic Calculator with parentheses (push context, pop on `)`)

---

## When to Use a Queue

**BFS (Breadth-First Search):**
- Shortest path in unweighted graph
- Level-order tree traversal

**Sliding window:**
- Monotonic deque for sliding window max/min

**Scheduling / rate limiting:**
- Number of recent calls (sliding time window)
- Task scheduler (producer-consumer)

**Implement other structures:**
- Queue-based stack (LC 225)
- Two-queue implementations

---

## When to Use a Deque

A Deque (double-ended queue) is the **superset** — it can act as both stack and queue. Use explicitly as deque when you need both-end access:

| Use Case | Deque Operation |
|----------|----------------|
| Sliding window max | `offerLast` + `pollFirst` (expire) + `pollLast` (maintain order) |
| Palindrome check (on characters) | Check `first == last`, remove both |
| Min/Max deque | Similar to sliding window, both ends |

---

## Monotonic Stack vs Monotonic Deque

| Aspect | Monotonic Stack | Monotonic Deque |
|--------|----------------|----------------|
| Access pattern | Only top | Both front and back |
| Use case | NGE, histogram, expressions | Sliding window max/min |
| Window management | No expiration needed | Expire from front by index |
| Order | Increasing or decreasing | Increasing or decreasing |

**Monotonic Stack:** Elements are processed once — either they find their "answer" when popped, or they remain on the stack.

**Monotonic Deque:** Elements can also expire from the front when they slide out of the window. This requires storing indices (to check expiry) even more critically.

---

## Choosing Between Two Approaches for the Same Problem

### Trapping Rain Water (LC 42)

| Approach | Time | Space | When to Choose |
|----------|------|-------|---------------|
| Two Pointers | O(n) | O(1) | Default; O(1) space is optimal |
| Monotonic Stack | O(n) | O(n) | If asked for online/streaming version, or follow-up with "which bars trap water" |
| Prefix Arrays | O(n) | O(n) | If asked for max left/right separately or multi-query |

### Min Stack (LC 155)

| Approach | Time | Space | When to Choose |
|----------|------|-------|---------------|
| Auxiliary Min Stack | O(1) | O(n) | Default; clearest code |
| Pair in single stack | O(1) | O(n) | Same complexity; slightly fewer objects |
| Difference encoding | O(1) | O(n) | Only if interviewer explicitly asks for minimal space constant |

---

## Pattern Recognition Table

| Input Signal | Likely Data Structure |
|-------------|----------------------|
| "Next greater/smaller element" | Monotonic Stack |
| "Sliding window max/min" | Monotonic Deque |
| "Balanced/valid parentheses" | Stack |
| "Shortest path", "minimum steps" | Queue (BFS) |
| "All paths", "combinations" | Stack (DFS / Backtracking) |
| "Most frequent, then most recent" | HashMap + HashMap of Stacks |
| "Min/max accessible in O(1)" | Stack + Auxiliary Stack |
| "Encode/decode nested structure" | Two Stacks (count + content) |
| "Expression with precedence" | Stack of terms/operands |

---

## Java API Quick Reference

```java
// Stack
Deque<T> stack = new ArrayDeque<>();
stack.push(x);          // O(1) amortized
stack.pop();            // O(1)
stack.peek();           // O(1)
stack.isEmpty();        // O(1)

// Queue
Queue<T> queue = new ArrayDeque<>();
queue.offer(x);         // O(1) amortized — use over add() (add throws on failure)
queue.poll();           // O(1) — use over remove() (remove throws on empty)
queue.peek();           // O(1) — use over element() (element throws on empty)

// Deque (explicit both-end access)
Deque<T> dq = new ArrayDeque<>();
dq.offerFirst(x);  dq.offerLast(x);
dq.pollFirst();    dq.pollLast();
dq.peekFirst();    dq.peekLast();

// Priority Queue (heap — not FIFO but common in "queue" context)
PriorityQueue<T> pq = new PriorityQueue<>(Comparator.comparingInt(...));
pq.offer(x);
pq.poll();   // removes smallest (min-heap)
pq.peek();   // views smallest
```

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Common Mistakes](./Common%20Mistakes.md)
- [Complexity Analysis](./Complexity%20Analysis.md)
- [Queue & Deque Pattern](../Patterns/Queue%20and%20Deque.md)
- [Monotonic Stack Pattern](../Patterns/Monotonic%20Stack.md)
