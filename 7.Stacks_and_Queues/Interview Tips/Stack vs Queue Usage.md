# Stack vs Queue Usage Guide

> **Topic:** [Stacks & Queues](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## The Core Distinction

| Property | Stack | Queue |
|----------|-------|-------|
| Ordering | LIFO (Last-In-First-Out) | FIFO (First-In-First-Out) |
| Rust type | `Vec<T>` with `push/pop/last` | `VecDeque<T>` with `push_back/front/pop_front` |
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
| Sliding window max | `push_back` + `pop_front` (expire) + `pop_back` (maintain order) |
| Palindrome check (on characters) | Check `front == back`, remove both |
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
| "Most frequent, then most recent" | HashMap + HashMap of Vecs (stacks) |
| "Min/max accessible in O(1)" | Stack + Auxiliary Stack |
| "Encode/decode nested structure" | Two Stacks (count + content) |
| "Expression with precedence" | Stack of terms/operands |

---

## Rust API Quick Reference

```rust
use std::collections::{VecDeque, BinaryHeap};
use std::cmp::Reverse;

// Stack (use Vec<T>)
let mut stk: Vec<T> = Vec::new();
stk.push(x);            // O(1) amortized
stk.pop();              // O(1) — returns Option<T>
stk.last();             // O(1) — view top without removing, returns Option<&T>
stk.is_empty();         // O(1)

// Queue (use VecDeque<T>)
let mut q: VecDeque<T> = VecDeque::new();
q.push_back(x);         // O(1) amortized
q.front(); q.pop_front(); // O(1) — front() to view, pop_front() to remove
q.front();              // O(1) — view front without removing, returns Option<&T>

// Deque (explicit both-end access, also VecDeque<T>)
let mut dq: VecDeque<T> = VecDeque::new();
dq.push_front(x);  dq.push_back(x);
dq.pop_front();    dq.pop_back();
dq.front();        dq.back();

// Priority Queue / BinaryHeap (heap — not FIFO but common in "queue" context)
let mut pq: BinaryHeap<Reverse<T>> = BinaryHeap::new(); // min-heap
pq.push(Reverse(x));
pq.pop();    // removes smallest (min-heap), returns Option<Reverse<T>>
pq.peek();   // views smallest, returns Option<&Reverse<T>>
```

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Common Mistakes](./Common%20Mistakes.md)
- [Complexity Analysis](./Complexity%20Analysis.md)
- [Queue & Deque Pattern](../Patterns/Queue%20and%20Deque.md)
- [Monotonic Stack Pattern](../Patterns/Monotonic%20Stack.md)
