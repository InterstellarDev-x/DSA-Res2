# Microsoft — Stacks & Queues OA Questions

> **Topic:** [Stacks & Queues](../README.md) · **Company:** Microsoft
> [← Back to OA Overview](../README.md#oa-questions)

---

## OA Question Bank

| # | Question | Frequency | Difficulty | Pattern | Link |
|---|---------|-----------|-----------|---------|------|
| 1 | [Valid Parentheses](https://leetcode.com/problems/valid-parentheses/) | ⭐⭐⭐⭐ | Easy | [Stack Basics](../Patterns/Stack%20Basics.md) | LC 20 |
| 2 | [Implement Queue using Stacks](https://leetcode.com/problems/implement-queue-using-stacks/) | ⭐⭐⭐⭐ | Easy | [Queue & Deque](../Patterns/Queue%20and%20Deque.md) | LC 232 |
| 3 | [Backspace String Compare](https://leetcode.com/problems/backspace-string-compare/) | ⭐⭐⭐ | Easy | [Stack Basics](../Patterns/Stack%20Basics.md) | LC 844 |
| 4 | [Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/) | ⭐⭐⭐ | Hard | [Monotonic Stack](../Patterns/Monotonic%20Stack.md) | LC 42 |
| 5 | [Basic Calculator II](https://leetcode.com/problems/basic-calculator-ii/) | ⭐⭐⭐ | Medium | [Stack Expressions](../Patterns/Stack%20for%20Expressions.md) | LC 227 |
| 6 | [Implement Stack using Queues](https://leetcode.com/problems/implement-stack-using-queues/) | ⭐⭐⭐ | Easy | [Queue & Deque](../Patterns/Queue%20and%20Deque.md) | LC 225 |
| 7 | [Design Circular Queue](https://leetcode.com/problems/design-circular-queue/) | ⭐⭐⭐ | Medium | [Design](../Design%20Data%20Structure%20Problems/Min%20Stack.md) | LC 622 |
| 8 | [Min Stack](https://leetcode.com/problems/min-stack/) | ⭐⭐ | Medium | [Design](../Design%20Data%20Structure%20Problems/Min%20Stack.md) | LC 155 |
| 9 | [Next Greater Element I](https://leetcode.com/problems/next-greater-element-i/) | ⭐⭐ | Easy | [Monotonic Stack](../Patterns/Monotonic%20Stack.md) | LC 496 |

---

## Top 4 Must-Know for Microsoft

### 1. Valid Parentheses ⭐⭐⭐⭐
Microsoft's most-asked stack question in phone screens. Occasionally extended to XML/HTML tag validation.
- **Common extension:** Return which bracket type is missing or mismatched

### 2. Implement Queue using Stacks ⭐⭐⭐⭐
Microsoft frequently tests queue/stack implementation because it validates understanding of ADT internals — foundational for backend roles.
- **Key:** Amortized O(1); explain the transfer logic clearly
- **Extension:** Implement both queue and stack using a single deque

### 3. Design Circular Queue ⭐⭐⭐
Microsoft cloud/Azure team OA includes buffer/ring-buffer design.
- **Key:** `size` variable to disambiguate full vs empty; modulo wrap

### 4. Trapping Rain Water ⭐⭐⭐
Microsoft asks this in SDE rounds. Prefer two-pointer O(1) space for efficiency discussion.

---

## Microsoft Interview Style Notes

- Microsoft emphasizes clean code and edge case handling over clever algorithms.
- For stack/queue implementations, be prepared to whiteboard the class with proper null checks and error messages.
- Circular Queue design is common in the Microsoft Azure distributed systems context — frame your solution in terms of producer/consumer queues.

---

## Related Files

- [Microsoft Interview Problems](../Interview%20Problems/Microsoft.md)
- [Amazon OA](./Amazon.md)
- [Goldman Sachs OA](./Goldman%20Sachs.md)

> **Last Updated:** 2026-06-26
