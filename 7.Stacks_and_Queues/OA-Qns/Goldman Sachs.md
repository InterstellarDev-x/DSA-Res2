# Goldman Sachs — Stacks & Queues OA Questions

> **Topic:** [Stacks & Queues](../README.md) · **Company:** Goldman Sachs
> [← Back to OA Overview](../README.md#oa-questions)

---

## OA Question Bank

| # | Question | Frequency | Difficulty | Pattern | Link |
|---|---------|-----------|-----------|---------|------|
| 1 | [Evaluate Reverse Polish Notation](https://leetcode.com/problems/evaluate-reverse-polish-notation/) | ⭐⭐⭐⭐ | Medium | [Stack Expressions](../Patterns/Stack%20for%20Expressions.md) | LC 150 |
| 2 | [Min Stack](https://leetcode.com/problems/min-stack/) | ⭐⭐⭐⭐ | Medium | [Design](../Design%20Data%20Structure%20Problems/Min%20Stack.md) | LC 155 |
| 3 | [Asteroid Collision](https://leetcode.com/problems/asteroid-collision/) | ⭐⭐⭐ | Medium | [Monotonic Stack](../Patterns/Monotonic%20Stack.md) | LC 735 |
| 4 | [Valid Parentheses](https://leetcode.com/problems/valid-parentheses/) | ⭐⭐⭐ | Easy | [Stack Basics](../Patterns/Stack%20Basics.md) | LC 20 |
| 5 | [Basic Calculator II](https://leetcode.com/problems/basic-calculator-ii/) | ⭐⭐ | Medium | [Stack Expressions](../Patterns/Stack%20for%20Expressions.md) | LC 227 |
| 6 | [Daily Temperatures](https://leetcode.com/problems/daily-temperatures/) | ⭐⭐ | Medium | [Monotonic Stack](../Patterns/Monotonic%20Stack.md) | LC 739 |

---

## Goldman Sachs Focus Areas

Goldman Sachs (Technology Division) tends to focus on:
1. **Expression evaluation** — RPN and infix with precedence rules (ties to financial formula evaluation)
2. **Stack design** — Min Stack and similar O(1) auxiliary structure designs (ties to trading system requirements)
3. **Collision/conflict resolution** — Asteroid Collision (models order matching systems)

### Evaluate Reverse Polish Notation ⭐⭐⭐⭐
GS uses this as an OA question for their "Strats" (quantitative strategist) pipeline. Framed as "evaluate a formula in stack-based notation."
- **Key:** Pop b first, then a = `stack.pop() op b`. Order matters for `-` and `/`.

### Min Stack ⭐⭐⭐⭐
GS infrastructure teams ask this as a design question.
- **Extension:** Max Stack — same idea but reversed
- **Extension:** MinMaxStack — track both in O(1)

---

## Related Files

- [Adobe OA](./Adobe.md)
- [Amazon OA](./Amazon.md)
- [Stack for Expressions](../Patterns/Stack%20for%20Expressions.md)

> **Last Updated:** 2026-06-26
