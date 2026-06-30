# Stacks & Queues

> **Topic 9 of 18** — 30 problems (8E / 14M / 8H)
> Java-centric, pattern-driven reference for interview preparation.
> [← Bit Manipulation](../6.Bit_Manipulation/README.md) · [→ Sliding Window](../8.Sliding_Window/README.md)

---

## Quick Navigation

| Section | Files |
|---------|-------|
| [Patterns](#patterns) | Stack Basics · Queue & Deque · Monotonic Stack · Histogram & Rectangle · Stack for Expressions |
| [Design Problems](#design-data-structure-problems) | Min Stack · Maximum Frequency Stack |
| [OA Questions](#oa-questions) | Amazon · Google · Microsoft · Goldman Sachs · Adobe |
| [Interview Problems](#interview-problems) | Amazon · Google · Microsoft |
| [Interview Tips](#interview-tips) | Coding Tips · Common Mistakes · Stack vs Queue Usage · Complexity Analysis |
| [Recent Questions](#most-recent-questions) | 2024 · 2025 · 2026 |

---

## Problem List

| # | Problem | Difficulty | Pattern | Companies |
|---|---------|-----------|---------|-----------|
| 1 | [Valid Parentheses](https://leetcode.com/problems/valid-parentheses/) | Easy | [Stack Basics](./Patterns/Stack%20Basics.md) | Amazon, Google, Microsoft |
| 2 | [Backspace String Compare](https://leetcode.com/problems/backspace-string-compare/) | Easy | [Stack Basics](./Patterns/Stack%20Basics.md) | Microsoft, Amazon |
| 3 | [Remove Outermost Parentheses](https://leetcode.com/problems/remove-outermost-parentheses/) | Easy | [Stack Basics](./Patterns/Stack%20Basics.md) | Adobe |
| 4 | [Implement Queue using Stacks](https://leetcode.com/problems/implement-queue-using-stacks/) | Easy | [Queue & Deque](./Patterns/Queue%20and%20Deque.md) | Amazon, Microsoft |
| 5 | [Implement Stack using Queues](https://leetcode.com/problems/implement-stack-using-queues/) | Easy | [Queue & Deque](./Patterns/Queue%20and%20Deque.md) | Microsoft |
| 6 | [Next Greater Element I](https://leetcode.com/problems/next-greater-element-i/) | Easy | [Monotonic Stack](./Patterns/Monotonic%20Stack.md) | Amazon |
| 7 | [Number of Recent Calls](https://leetcode.com/problems/number-of-recent-calls/) | Easy | [Queue & Deque](./Patterns/Queue%20and%20Deque.md) | Google |
| 8 | [Simplify Path](https://leetcode.com/problems/simplify-path/) | Medium | [Stack Basics](./Patterns/Stack%20Basics.md) | Amazon |
| 9 | [Min Stack](https://leetcode.com/problems/min-stack/) | Medium | [Design](./Design%20Data%20Structure%20Problems/Min%20Stack.md) | Amazon, Google, Adobe |
| 10 | [Evaluate Reverse Polish Notation](https://leetcode.com/problems/evaluate-reverse-polish-notation/) | Medium | [Stack Expressions](./Patterns/Stack%20for%20Expressions.md) | Amazon, Google |
| 11 | [Design Circular Queue](https://leetcode.com/problems/design-circular-queue/) | Medium | [Design](./Design%20Data%20Structure%20Problems/Min%20Stack.md) | Amazon, Microsoft |
| 12 | [Next Greater Element II](https://leetcode.com/problems/next-greater-element-ii/) | Medium | [Monotonic Stack](./Patterns/Monotonic%20Stack.md) | Amazon |
| 13 | [Daily Temperatures](https://leetcode.com/problems/daily-temperatures/) | Medium | [Monotonic Stack](./Patterns/Monotonic%20Stack.md) | Amazon, Google |
| 14 | [Online Stock Span](https://leetcode.com/problems/online-stock-span/) | Medium | [Monotonic Stack](./Patterns/Monotonic%20Stack.md) | Amazon |
| 15 | [Decode String](https://leetcode.com/problems/decode-string/) | Medium | [Stack Expressions](./Patterns/Stack%20for%20Expressions.md) | Google, Amazon |
| 16 | [Asteroid Collision](https://leetcode.com/problems/asteroid-collision/) | Medium | [Monotonic Stack](./Patterns/Monotonic%20Stack.md) | Amazon, Google |
| 17 | [Remove K Digits](https://leetcode.com/problems/remove-k-digits/) | Medium | [Monotonic Stack](./Patterns/Monotonic%20Stack.md) | Google |
| 18 | [132 Pattern](https://leetcode.com/problems/132-pattern/) | Medium | [Monotonic Stack](./Patterns/Monotonic%20Stack.md) | Amazon |
| 19 | [Basic Calculator II](https://leetcode.com/problems/basic-calculator-ii/) | Medium | [Stack Expressions](./Patterns/Stack%20for%20Expressions.md) | Amazon, Google |
| 20 | [Sum of Subarray Minimums](https://leetcode.com/problems/sum-of-subarray-minimums/) | Medium | [Monotonic Stack](./Patterns/Monotonic%20Stack.md) | Google |
| 21 | [Score of Parentheses](https://leetcode.com/problems/score-of-parentheses/) | Medium | [Stack Expressions](./Patterns/Stack%20for%20Expressions.md) | Google |
| 22 | [Remove Duplicate Letters](https://leetcode.com/problems/remove-duplicate-letters/) | Medium | [Monotonic Stack](./Patterns/Monotonic%20Stack.md) | Google |
| 23 | [Maximum Width Ramp](https://leetcode.com/problems/maximum-width-ramp/) | Medium | [Monotonic Stack](./Patterns/Monotonic%20Stack.md) | Amazon |
| 24 | [Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/) | Hard | [Queue & Deque](./Patterns/Queue%20and%20Deque.md) | Amazon, Google |
| 25 | [Largest Rectangle in Histogram](https://leetcode.com/problems/largest-rectangle-in-histogram/) | Hard | [Histogram](./Patterns/Histogram%20and%20Rectangle.md) | Amazon, Google |
| 26 | [Maximal Rectangle](https://leetcode.com/problems/maximal-rectangle/) | Hard | [Histogram](./Patterns/Histogram%20and%20Rectangle.md) | Amazon, Google |
| 27 | [Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/) | Hard | [Monotonic Stack](./Patterns/Monotonic%20Stack.md) | Amazon, Google, Microsoft |
| 28 | [Number of Visible People in Queue](https://leetcode.com/problems/number-of-visible-people-in-a-queue/) | Hard | [Monotonic Stack](./Patterns/Monotonic%20Stack.md) | Google |
| 29 | [Maximum Frequency Stack](https://leetcode.com/problems/maximum-frequency-stack/) | Hard | [Design](./Design%20Data%20Structure%20Problems/Maximum%20Frequency%20Stack.md) | Amazon, Google |
| 30 | [Basic Calculator](https://leetcode.com/problems/basic-calculator/) | Hard | [Stack Expressions](./Patterns/Stack%20for%20Expressions.md) | Google |

---

## Patterns

| Pattern | Key Problems | Core Idea |
|---------|-------------|-----------|
| [Stack Basics](./Patterns/Stack%20Basics.md) | Valid Parens, Simplify Path, Backspace Compare | LIFO for matching, undo, path resolution |
| [Queue & Deque](./Patterns/Queue%20and%20Deque.md) | Sliding Window Max, Queue/Stack impls | FIFO + monotonic deque for range queries |
| [Monotonic Stack](./Patterns/Monotonic%20Stack.md) | NGE, Daily Temps, Trapping Rain Water, 132 Pattern | Maintain invariant to answer "next greater/smaller" |
| [Histogram & Rectangle](./Patterns/Histogram%20and%20Rectangle.md) | LRH, Maximal Rectangle | Extend monotonic stack to 2D area problems |
| [Stack for Expressions](./Patterns/Stack%20for%20Expressions.md) | RPN, Basic Calc I/II, Decode String | Stack manages operator precedence and nesting |

---

## Design Data Structure Problems

| Design | Key Operations | Complexity |
|--------|---------------|------------|
| [Min Stack](./Design%20Data%20Structure%20Problems/Min%20Stack.md) | push, pop, top, getMin — all O(1) | O(1) time, O(n) space |
| [Maximum Frequency Stack](./Design%20Data%20Structure%20Problems/Maximum%20Frequency%20Stack.md) | push, pop (most frequent, latest) — O(1) | O(1) time, O(n) space |

---

## Company Coverage

| Company | Top OA Problems | Top Interview Problems |
|---------|----------------|------------------------|
| Amazon | Valid Parens ⭐⭐⭐⭐⭐, Trapping Rain Water ⭐⭐⭐⭐, Daily Temps | Min Stack, Largest Histogram, 132 Pattern |
| Google | Largest Histogram ⭐⭐⭐⭐⭐, Basic Calc ⭐⭐⭐⭐, NGE II | Max Freq Stack, Remove K Digits, Sliding Window Max |
| Microsoft | Valid Parens ⭐⭐⭐⭐, Implement Queue/Stack | Trapping Rain Water, Basic Calc II |
| Goldman Sachs | Evaluate RPN ⭐⭐⭐⭐, Min Stack | Asteroid Collision |
| Adobe | Decode String ⭐⭐⭐⭐, Remove Outermost Parens | Valid Parens |

---

## OA Questions

- [Amazon OA](./OA-Qns/Amazon.md)
- [Google OA](./OA-Qns/Google.md)
- [Microsoft OA](./OA-Qns/Microsoft.md)
- [Goldman Sachs OA](./OA-Qns/Goldman%20Sachs.md)
- [Adobe OA](./OA-Qns/Adobe.md)

---

## Interview Problems

- [Amazon](./Interview%20Problems/Amazon.md)
- [Google](./Interview%20Problems/Google.md)
- [Microsoft](./Interview%20Problems/Microsoft.md)

---

## Interview Tips

- [Coding Tips](./Interview%20Tips/Coding%20Tips.md)
- [Common Mistakes](./Interview%20Tips/Common%20Mistakes.md)
- [Stack vs Queue Usage](./Interview%20Tips/Stack%20vs%20Queue%20Usage.md)
- [Complexity Analysis](./Interview%20Tips/Complexity%20Analysis.md)

---

## Most Recent Questions

- [2024](./Most%20Recent%20Questions/2024.md)
- [2025](./Most%20Recent%20Questions/2025.md)
- [2026](./Most%20Recent%20Questions/2026.md)

---

> **Last Updated:** 2026-06-26
> **Problems:** 30 (8E / 14M / 8H)
> [← Back to Root](../README.md)
