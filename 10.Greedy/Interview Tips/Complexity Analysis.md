> **Topic:** [Greedy Algorithms](../README.md) · **Interview Tips — Complexity**

# Greedy Algorithms — Complexity Analysis

## Master Complexity Table

| Problem | LC # | Time | Space | Bottleneck | O(n) Possible? |
|---------|------|------|-------|------------|----------------|
| Assign Cookies | #455 | O(n log n + m log m) | O(1) | Sorting both arrays | No (must sort) |
| Lemonade Change | #860 | O(n) | O(1) | Single scan | Yes ✓ |
| Jump Game | #55 | O(n) | O(1) | Single scan | Yes ✓ |
| Jump Game II | #45 | O(n) | O(1) | Single scan | Yes ✓ |
| Gas Station | #134 | O(n) | O(1) | Single scan | Yes ✓ |
| Hand of Straights | #846 | O(n log n) | O(n) | TreeMap operations | No (need sorted order) |
| Merge Triplets | #1899 | O(n) | O(1) | Single scan | Yes ✓ |
| Partition Labels | #763 | O(n) | O(1) | Two scans | Yes ✓ |
| Valid Parenthesis String | #678 | O(n) | O(1) | Single scan | Yes ✓ |
| Non-overlapping Intervals | #435 | O(n log n) | O(1) | Sorting | No (must sort) |
| Meeting Rooms II | #253 | O(n log n) | O(n) | Sort + heap | No (sort + heap) |
| Candy | #135 | O(n) | O(n) | Two passes | Yes ✓ (but O(n) space) |
| Insert Interval | #57 | O(n) | O(n) | Single pass (already sorted) | Yes ✓ |
| Minimum Arrows | #452 | O(n log n) | O(1) | Sorting | No (must sort) |
| Task Scheduler | #621 | O(n log n) or O(n) | O(1) | Sort freq / formula | Yes ✓ (formula) |

---

## When Does O(n log n) Sorting Dominate?

**Rule:** If the problem requires comparing elements to each other (ordering-based greedy), sorting is O(n log n) and dominates. If the problem processes elements sequentially with a fixed invariant, O(n) is achievable.

### O(n log n) Bottleneck Problems
These require sorting → O(n log n) is optimal:
- **Interval problems** (Non-overlapping, Meeting Rooms II, Assign Cookies, Minimum Arrows): Must sort before greedy scan.
- **Hand of Straights**: TreeMap maintains sorted order; all operations O(log n); total O(n log n).
- **Task Scheduler (heap version)**: Sorting task frequencies O(26 log 26) = O(1) effectively, but general case O(k log k) where k = distinct tasks.

### O(n) Greedy Problems
These process elements in given order (no sorting needed):
- **Jump Game I & II**: Array already defines the order.
- **Gas Station**: Process stations in given circular order.
- **Partition Labels**: Two scans of the string.
- **Valid Parenthesis String**: Single scan.
- **Merge Triplets**: Single scan.
- **Lemonade Change**: Process bills in arrival order.

---

## Space Complexity Deep Dive

### O(1) Space Solutions (Best)
| Problem | Why O(1) |
|---------|----------|
| Jump Game I | Only `maxReach` variable |
| Jump Game II | Three integers: `jumps`, `curEnd`, `farthest` |
| Gas Station | Three integers: `totalGain`, `currentGain`, `startStation` |
| Lemonade Change | Two integers: `five`, `ten` |
| Valid Parenthesis String | Two integers: `lo`, `hi` |
| Merge Triplets | Three-element `result` array (constant size) |
| Non-overlapping Intervals | Sort in-place; two integers: `kept`, `prevEnd` |
| Minimum Arrows | Sort in-place; two integers: `arrows`, `arrowPos` |

### O(n) Space Solutions
| Problem | Space Used For | Can It Be Reduced? |
|---------|---------------|-------------------|
| Candy | `int[] candies` array | No — need both neighbors' counts |
| Meeting Rooms II | Min-heap (up to n end times) | No — must track all active rooms |
| Hand of Straights | TreeMap with all card counts | No — need frequency lookup |
| Insert Interval | Result `List<int[]>` | Inherent to output size |
| Partition Labels | `int[] last` (size 26, so technically O(1)) + result list | Result is inherent |

---

## Amortized Analysis: Hand of Straights

TreeMap operations in `isNStraightHand`:
- Each card is inserted once: n × O(log n) = O(n log n)
- Each card is removed at most once: n × O(log n) = O(n log n)
- Total operations: 2n → **O(n log n) amortized**

Even though the inner loop runs `groupSize` times per group, each element participates in exactly one group → total inner loop iterations = n.

---

## Amortized Analysis: Meeting Rooms II

For n meetings:
- `offer()` called exactly n times: n × O(log n) = O(n log n)
- `poll()` called at most n times: n × O(log n) = O(n log n)
- `peek()` called n times: n × O(1) = O(n)
- Sort: O(n log n)
- **Total: O(n log n)**

Heap size at any point = number of concurrent meetings = number of rooms currently in use.

---

## Task Scheduler: Two Approaches Compared

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Formula | O(n) to count + O(26 log 26) = O(n) | O(26) = O(1) | Best approach |
| Max-Heap simulation | O(n log 26) = O(n) | O(26) = O(1) | Also O(n) in practice |
| General k-task simulation | O(n log k) | O(k) | k = distinct task types |

**Formula derivation:**
Let `maxFreq` = frequency of most common task.
Let `maxCount` = number of tasks with frequency `maxFreq`.

Minimum intervals = `max(n, (maxFreq - 1) × (n + 1) + maxCount)`

- `(maxFreq - 1) × (n + 1)`: `maxFreq - 1` full cycles of `n + 1` slots each.
- `+ maxCount`: The last partial cycle fills slots with `maxCount` tasks.
- `max(..., n)`: If tasks fill the cycles completely, no idle time needed; total = n.

---

## Complexity Summary by Pattern

| Pattern | Typical Time | Typical Space | Key Factor |
|---------|-------------|---------------|-----------|
| Interval Scheduling | O(n log n) | O(1)–O(n) | Sort dominates |
| Jump Game | O(n) | O(1) | Single pass |
| Greedy on Strings | O(n) | O(1)–O(n) | Linear scan |
| Greedy with Heap | O(n log n) | O(n) | Heap operations |

> **Last Updated:** 2026-06-26
