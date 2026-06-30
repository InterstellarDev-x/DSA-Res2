# Adobe — Stacks & Queues OA Questions

> **Topic:** [Stacks & Queues](../README.md) · **Company:** Adobe
> [← Back to OA Overview](../README.md#oa-questions)

---

## OA Question Bank

| # | Question | Frequency | Difficulty | Pattern | Link |
|---|---------|-----------|-----------|---------|------|
| 1 | [Decode String](https://leetcode.com/problems/decode-string/) | ⭐⭐⭐⭐ | Medium | [Stack Expressions](../Patterns/Stack%20for%20Expressions.md) | LC 394 |
| 2 | [Remove Outermost Parentheses](https://leetcode.com/problems/remove-outermost-parentheses/) | ⭐⭐⭐ | Easy | [Stack Basics](../Patterns/Stack%20Basics.md) | LC 1021 |
| 3 | [Min Stack](https://leetcode.com/problems/min-stack/) | ⭐⭐⭐ | Medium | [Design](../Design%20Data%20Structure%20Problems/Min%20Stack.md) | LC 155 |
| 4 | [Valid Parentheses](https://leetcode.com/problems/valid-parentheses/) | ⭐⭐⭐ | Easy | [Stack Basics](../Patterns/Stack%20Basics.md) | LC 20 |
| 5 | [Simplify Path](https://leetcode.com/problems/simplify-path/) | ⭐⭐ | Medium | [Stack Basics](../Patterns/Stack%20Basics.md) | LC 71 |

---

## Adobe Focus Areas

Adobe (Creative Cloud, Document Cloud) tends to ask:
1. **String manipulation stacks** — Decode String (encoding/decoding is core to document formats)
2. **Parentheses problems** — Valid Parens and variants (used in expression parsing for formulas)
3. **Path normalization** — Simplify Path (file system operations in Creative Cloud)

### Decode String ⭐⭐⭐⭐
Adobe's top stack interview question. Aligns with document compression/encoding (like RLE — run-length encoding) used in PDF and image formats.
- **Key:** Two stacks — count stack and string stack. Multi-digit counts (`k = k*10 + digit`).
- **Follow-up:** Decode with nested brackets of depth > 1 (handled by same solution)

---

## Related Files

- [Goldman Sachs OA](./Goldman%20Sachs.md)
- [Amazon OA](./Amazon.md)
- [Stack for Expressions](../Patterns/Stack%20for%20Expressions.md)

> **Last Updated:** 2026-06-26
