# Google — Stacks & Queues OA Questions

> **Topic:** [Stacks & Queues](../README.md) · **Company:** Google
> [← Back to OA Overview](../README.md#oa-questions)

---

## OA Question Bank

| # | Question | Frequency | Difficulty | Pattern | Link |
|---|---------|-----------|-----------|---------|------|
| 1 | [Largest Rectangle in Histogram](https://leetcode.com/problems/largest-rectangle-in-histogram/) | ⭐⭐⭐⭐⭐ | Hard | [Histogram](../Patterns/Histogram%20and%20Rectangle.md) | LC 84 |
| 2 | [Basic Calculator](https://leetcode.com/problems/basic-calculator/) | ⭐⭐⭐⭐ | Hard | [Stack Expressions](../Patterns/Stack%20for%20Expressions.md) | LC 224 |
| 3 | [Maximum Frequency Stack](https://leetcode.com/problems/maximum-frequency-stack/) | ⭐⭐⭐⭐ | Hard | [Design](../Design%20Data%20Structure%20Problems/Maximum%20Frequency%20Stack.md) | LC 895 |
| 4 | [Remove K Digits](https://leetcode.com/problems/remove-k-digits/) | ⭐⭐⭐⭐ | Medium | [Monotonic Stack](../Patterns/Monotonic%20Stack.md) | LC 402 |
| 5 | [Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/) | ⭐⭐⭐⭐ | Hard | [Queue & Deque](../Patterns/Queue%20and%20Deque.md) | LC 239 |
| 6 | [Daily Temperatures](https://leetcode.com/problems/daily-temperatures/) | ⭐⭐⭐ | Medium | [Monotonic Stack](../Patterns/Monotonic%20Stack.md) | LC 739 |
| 7 | [Score of Parentheses](https://leetcode.com/problems/score-of-parentheses/) | ⭐⭐⭐ | Medium | [Stack Expressions](../Patterns/Stack%20for%20Expressions.md) | LC 856 |
| 8 | [Remove Duplicate Letters](https://leetcode.com/problems/remove-duplicate-letters/) | ⭐⭐⭐ | Medium | [Monotonic Stack](../Patterns/Monotonic%20Stack.md) | LC 316 |
| 9 | [Decode String](https://leetcode.com/problems/decode-string/) | ⭐⭐⭐ | Medium | [Stack Expressions](../Patterns/Stack%20for%20Expressions.md) | LC 394 |
| 10 | [Sum of Subarray Minimums](https://leetcode.com/problems/sum-of-subarray-minimums/) | ⭐⭐⭐ | Medium | [Monotonic Stack](../Patterns/Monotonic%20Stack.md) | LC 907 |
| 11 | [Maximal Rectangle](https://leetcode.com/problems/maximal-rectangle/) | ⭐⭐⭐ | Hard | [Histogram](../Patterns/Histogram%20and%20Rectangle.md) | LC 85 |
| 12 | [Number of Visible People in Queue](https://leetcode.com/problems/number-of-visible-people-in-a-queue/) | ⭐⭐ | Hard | [Monotonic Stack](../Patterns/Monotonic%20Stack.md) | LC 1944 |
| 13 | [Number of Recent Calls](https://leetcode.com/problems/number-of-recent-calls/) | ⭐⭐ | Easy | [Queue & Deque](../Patterns/Queue%20and%20Deque.md) | LC 933 |
| 14 | [Next Greater Element II](https://leetcode.com/problems/next-greater-element-ii/) | ⭐⭐ | Medium | [Monotonic Stack](../Patterns/Monotonic%20Stack.md) | LC 503 |

---

## Top 5 Must-Know for Google

### 1. Largest Rectangle in Histogram ⭐⭐⭐⭐⭐
Google's canonical hard stack problem. Appears in virtually every on-site loop.
- **Must know:** Sentinel element trick; exact width formula; O(n) proof
- **Extension asked at Google:** Maximal Rectangle (row-by-row histogram reduction)
- **Pattern:** [Histogram & Rectangle](../Patterns/Histogram%20and%20Rectangle.md)

### 2. Basic Calculator ⭐⭐⭐⭐
Tests understanding of expression evaluation, which is a classic CS foundations question.
- **Variants asked:** LC 224 (with parens), LC 227 (with `*/`), LC 772 (all combined)
- **Google L4+ style:** Asked to write recursive descent parser instead of stack
- **Pattern:** [Stack for Expressions](../Patterns/Stack%20for%20Expressions.md)

### 3. Maximum Frequency Stack ⭐⭐⭐⭐
Tests ability to design custom data structures — a Google L4 style problem.
- **Key:** `group` map + `maxFreq` decrements exactly by 1
- **Pattern:** [Design](../Design%20Data%20Structure%20Problems/Maximum%20Frequency%20Stack.md)

### 4. Sliding Window Maximum ⭐⭐⭐⭐
Tests knowledge of monotonic deque for O(n) range query. Often embedded in larger problems.
- **Extension:** Sliding window minimum (same, increasing deque)
- **Pattern:** [Queue & Deque](../Patterns/Queue%20and%20Deque.md)

### 5. Remove K Digits ⭐⭐⭐⭐
Greedy + monotonic stack combination. Tests problem decomposition skills.
- **Key:** Increasing stack; leading zero removal; leftover digits
- **Pattern:** [Monotonic Stack](../Patterns/Monotonic%20Stack.md)

---

## Google Interview Style Notes

- Google asks histogram problems more than any other company.
- Expression evaluation is tested as both a stack problem and a recursive descent parsing problem — prepare both.
- `Number of Visible People in Queue` is a newer Google L4 problem testing "can you model observation as monotonic stack?"
- Expect follow-ups requiring the O(1) space optimization (Score of Parentheses depth trick).

---

## Related Files

- [Google Interview Problems](../Interview%20Problems/Google.md)
- [Amazon OA](./Amazon.md)
- [Microsoft OA](./Microsoft.md)
- [Most Recent 2025](../Most%20Recent%20Questions/2025.md)

> **Last Updated:** 2026-06-26
