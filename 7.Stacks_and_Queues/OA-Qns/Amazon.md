# Amazon — Stacks & Queues OA Questions

> **Topic:** [Stacks & Queues](../README.md) · **Company:** Amazon
> [← Back to OA Overview](../README.md#oa-questions)

---

## OA Question Bank

| # | Question | Frequency | Difficulty | Pattern | Link |
|---|---------|-----------|-----------|---------|------|
| 1 | [Valid Parentheses](https://leetcode.com/problems/valid-parentheses/) | ⭐⭐⭐⭐⭐ | Easy | [Stack Basics](../Patterns/Stack%20Basics.md) | LC 20 |
| 2 | [Daily Temperatures](https://leetcode.com/problems/daily-temperatures/) | ⭐⭐⭐⭐⭐ | Medium | [Monotonic Stack](../Patterns/Monotonic%20Stack.md) | LC 739 |
| 3 | [Asteroid Collision](https://leetcode.com/problems/asteroid-collision/) | ⭐⭐⭐⭐ | Medium | [Monotonic Stack](../Patterns/Monotonic%20Stack.md) | LC 735 |
| 4 | [Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/) | ⭐⭐⭐⭐ | Hard | [Monotonic Stack](../Patterns/Monotonic%20Stack.md) | LC 42 |
| 5 | [Implement Queue using Stacks](https://leetcode.com/problems/implement-queue-using-stacks/) | ⭐⭐⭐ | Easy | [Queue & Deque](../Patterns/Queue%20and%20Deque.md) | LC 232 |
| 6 | [Simplify Path](https://leetcode.com/problems/simplify-path/) | ⭐⭐⭐ | Medium | [Stack Basics](../Patterns/Stack%20Basics.md) | LC 71 |
| 7 | [Basic Calculator II](https://leetcode.com/problems/basic-calculator-ii/) | ⭐⭐⭐ | Medium | [Stack Expressions](../Patterns/Stack%20for%20Expressions.md) | LC 227 |
| 8 | [Evaluate Reverse Polish Notation](https://leetcode.com/problems/evaluate-reverse-polish-notation/) | ⭐⭐⭐ | Medium | [Stack Expressions](../Patterns/Stack%20for%20Expressions.md) | LC 150 |
| 9 | [Next Greater Element I](https://leetcode.com/problems/next-greater-element-i/) | ⭐⭐⭐ | Easy | [Monotonic Stack](../Patterns/Monotonic%20Stack.md) | LC 496 |
| 10 | [132 Pattern](https://leetcode.com/problems/132-pattern/) | ⭐⭐⭐ | Medium | [Monotonic Stack](../Patterns/Monotonic%20Stack.md) | LC 456 |
| 11 | [Largest Rectangle in Histogram](https://leetcode.com/problems/largest-rectangle-in-histogram/) | ⭐⭐⭐ | Hard | [Histogram](../Patterns/Histogram%20and%20Rectangle.md) | LC 84 |
| 12 | [Online Stock Span](https://leetcode.com/problems/online-stock-span/) | ⭐⭐ | Medium | [Monotonic Stack](../Patterns/Monotonic%20Stack.md) | LC 901 |
| 13 | [Maximum Width Ramp](https://leetcode.com/problems/maximum-width-ramp/) | ⭐⭐ | Medium | [Monotonic Stack](../Patterns/Monotonic%20Stack.md) | LC 962 |
| 14 | [Maximum Frequency Stack](https://leetcode.com/problems/maximum-frequency-stack/) | ⭐⭐ | Hard | [Design](../Design%20Data%20Structure%20Problems/Maximum%20Frequency%20Stack.md) | LC 895 |

---

## Top 5 Must-Know for Amazon

### 1. Valid Parentheses ⭐⭐⭐⭐⭐
Amazon uses this as a phone screen warm-up and follow-up to string parsing questions.
- **Common follow-up:** Handle wildcards (`*` can be `(`, `)`, or empty) — LC 678
- **Pattern:** [Stack Basics](../Patterns/Stack%20Basics.md)

### 2. Daily Temperatures ⭐⭐⭐⭐⭐
Appears in AWS infrastructure team OA — "next time a server load exceeds threshold."
- **Key:** Store indices, not values. Answer = `i - prevIdx`
- **Follow-up:** Return temperatures instead of days
- **Pattern:** [Monotonic Stack](../Patterns/Monotonic%20Stack.md)

### 3. Trapping Rain Water ⭐⭐⭐⭐
Amazon top 10 all-time. Can appear in disguised form ("water storage in containers").
- **Two approaches:** Two-pointer O(1) space vs monotonic stack O(n) space — know both
- **Pattern:** [Monotonic Stack](../Patterns/Monotonic%20Stack.md)

### 4. Largest Rectangle in Histogram ⭐⭐⭐
Amazon system design teams ask this for "optimal buffer allocation" framing.
- **Critical:** Sentinel 0 at end to flush stack; width calculation formula
- **Extension:** Maximal Rectangle (LC 85)
- **Pattern:** [Histogram & Rectangle](../Patterns/Histogram%20and%20Rectangle.md)

### 5. Maximum Frequency Stack ⭐⭐
Amazon SDE-II+ system design component question.
- **Key:** `group[freq]` + `maxFreq` decrement only by 1
- **Pattern:** [Design](../Design%20Data%20Structure%20Problems/Maximum%20Frequency%20Stack.md)

---

## Amazon-Style Problem Variants

| Base Problem | Amazon Variant |
|-------------|----------------|
| Valid Parens | Tag validation in HTML/XML parser |
| Daily Temperatures | Next time a metric exceeds threshold |
| Asteroid Collision | Server request conflict resolution |
| Simplify Path | S3 path normalization |
| Basic Calculator | Inventory pricing rule evaluation |

---

## Related Files

- [Amazon Interview Problems](../Interview%20Problems/Amazon.md)
- [Google OA](./Google.md)
- [Microsoft OA](./Microsoft.md)
- [Most Recent 2025](../Most%20Recent%20Questions/2025.md)

> **Last Updated:** 2026-06-26
