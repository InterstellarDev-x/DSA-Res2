# Follow-up Questions — Arrays

> **Topic:** [Arrays](../README.md) · **Section:** Interview Tips
> **Tags:** `follow-up` `interview` `arrays` `deep-dive`

---

## Table of Contents

1. [Why Follow-ups Matter](#why-follow-ups-matter)
2. [Universal Follow-up Questions](#universal-follow-up-questions)
3. [Problem-Specific Follow-ups](#problem-specific-follow-ups)
4. [How to Handle Follow-ups](#how-to-handle-follow-ups)
5. [Related Files](#related-files)

---

## Why Follow-ups Matter

Interviewers use follow-ups to:
1. Probe **depth of understanding** beyond the textbook solution
2. Test **scalability thinking** (what if n = 10^9?)
3. Verify **edge case awareness**
4. Check **adaptability** — can you extend your solution?
5. Assess **senior-level thinking** — system design, concurrency, streaming

---

## Universal Follow-up Questions

These are asked for almost every array problem:

| Follow-up | Expected Answer Direction |
|-----------|--------------------------|
| "Can you do it in O(1) space?" | In-place manipulation, cyclic sort, bit tricks |
| "What if the array can be updated?" | Move from static to dynamic DS (Segment Tree, BIT) |
| "What if n = 10^9 — won't fit in memory?" | Streaming / external sort / MapReduce thinking |
| "What if the array arrives as a stream?" | Online algorithm, sliding window in O(1) memory |
| "Can you do it in one pass?" | Redesign algorithm to avoid multi-pass |
| "Print the actual subarray, not just the value" | Track start/end indices |
| "What if there are multiple valid answers?" | Return all of them, change return type |
| "What if input can have duplicates?" | Add de-duplication logic |

---

## Problem-Specific Follow-ups

### Two Sum

| Follow-up | Approach |
|-----------|----------|
| Array is sorted | Two Pointers O(n) O(1) |
| Return all unique pairs | HashSet for pairs, skip duplicates |
| Count pairs, not return them | Increment counter instead of returning |
| Values can be very large | Use `long` keys in HashMap |

### Maximum Subarray (Kadane's)

| Follow-up | Approach |
|-----------|----------|
| Print the actual subarray | Track `tempStart`, `start`, `end` indices |
| All numbers are negative | Initialize from `nums[0]`, not 0 |
| Circular array version | `max(kadane, totalSum - minKadane)` |
| At most k deletions allowed | DP with state `(i, deletions)` |
| Maximum sum with no two adjacent | House Robber (LC 198) |

### Merge Intervals

| Follow-up | Approach |
|-----------|----------|
| Intervals arrive in stream | TreeMap keyed by start |
| Delete an interval | Reconstruct affected merges |
| Count intervals overlapping at point x | Binary search + sweep |
| Weighted intervals (maximize weight) | DP: Weighted Interval Scheduling |

### Sliding Window (General)

| Follow-up | Approach |
|-----------|----------|
| Values can be negative | Switch to Prefix Sum + HashMap |
| Need exact sum = K (not ≤ K) | `atMost(K) - atMost(K-1)` trick |
| Window minimum (not sum) | Monotonic Deque |
| Multiple arrays / parallel windows | Track each independently |

### Trapping Rain Water

| Follow-up | Approach |
|-----------|----------|
| 2D version (3D water) | LC 407 — BFS from boundary |
| Dynamic heights (updates) | Segment Tree for range max |
| K arrays side by side | Generalize to multiple walls |

---

## How to Handle Follow-ups

**Framework:**

1. **Acknowledge** — "Good question. Let me think about that."
2. **Identify the change** — "The difference here is that values can now be negative, which breaks sliding window because..."
3. **State the new approach** — "Instead, I'd use prefix sum + hashmap because..."
4. **Trade-off** — "This costs O(n) extra space instead of O(1), but maintains O(n) time."
5. **Code if asked** — Only write code if explicitly asked; otherwise describe at high level.

**If you don't know:**

> "I'm not immediately sure of the optimal approach. Let me think through it... [pause, reason aloud]... I think a Segment Tree would work for dynamic updates because it supports O(log n) point updates and range queries. I haven't implemented this specific variant before but that's the direction I'd explore."

---

## Related Files

- [Communication Tips](./Communication.md)
- [Optimization Tips](./Optimization.md)
- [Complexity Analysis](./Complexity%20Analysis.md)
- [Arrays README](../README.md)

> **Last Updated:** 2026-06-26
