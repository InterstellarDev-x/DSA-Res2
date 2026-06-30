# Heap Patterns Guide

> **Topic:** [Heaps](../README.md) · **Section:** Interview Tips

---

## Pattern Recognition Flowchart

```
Does the problem involve repeatedly finding min/max?
    YES →
        Fixed k? (k largest, k smallest, k frequent)
            YES → Top K Pattern (opposite heap, size k)
        Merging sorted sequences?
            YES → K-Way Merge (min-heap with (val, listIdx, elemIdx))
        Tracking median or two halves?
            YES → Two Heaps (lo max-heap + hi min-heap)
        Scheduling with frequency/cooldown?
            YES → Heap + Frequency (max-heap by freq + cooldown queue)
    NO → Consider sorting, prefix sum, or other approach
```

---

## Pattern 1: Top K Elements

**Signal:** "Find k largest/smallest/frequent/closest"
**Structure:** Opposite heap of size k; root = the kth element

| Variant | Heap Type | Size | Eviction |
|---------|----------|------|---------|
| k largest | Min-heap | k | Remove smallest when size > k |
| k smallest | Max-heap | k | Remove largest when size > k |
| k most frequent | Min-heap (by freq) | k | Remove least frequent |
| k closest | Max-heap (by dist²) | k | Remove farthest |

**When to use quickselect instead:** k = 1 or k ≈ n/2, array is mutable, O(n) average is needed.

---

## Pattern 2: K-Way Merge

**Signal:** "Given k sorted lists/arrays, find the kth smallest" or "merge k sorted sequences"
**Structure:** Min-heap of size k, one entry per list, advance within list on each poll

**Heap entry:** `[value, listIndex, elementIndex]`
**Init:** Push first element of each list
**Advance:** After polling `[v, i, j]`, push `[list[i][j+1], i, j+1]` if j+1 valid

**Problems:**
- Merge K Sorted Lists — full merge until heap empty
- Kth Smallest in Matrix — k polls
- K Pairs Smallest Sums — k polls with 2D index
- Smallest Range from K Lists — advance minimum until a list exhausted

---

## Pattern 3: Two Heaps

**Signal:** "Median", "balance two groups", "greedy pick from two pools"
**Structure:** lo = max-heap (lower half), hi = min-heap (upper half)

**Invariants:**
- `|lo.size() - hi.size()| ≤ 1` (lo can have 1 extra)
- `lo.peek() ≤ hi.peek()` (every lo element ≤ every hi element)

**Median extraction:**
- Odd total: `lo.peek()`
- Even total: `(lo.peek() + hi.peek()) / 2.0`

**Variant — IPO (two pools, different semantics):**
- `locked` = min-heap by capital (projects not yet affordable)
- `available` = max-heap by profit (affordable projects)
- Pattern: unlock from `locked` to `available` as capital grows; always pick max profit from `available`

---

## Pattern 4: Heap + Frequency

**Signal:** "Arrange/schedule tasks", "no two adjacent same", "rearrange by frequency"
**Structure:** Frequency map → max-heap by frequency → greedy pick → cooldown

**Key check:** If max frequency > `(n+1)/2`, it's impossible to arrange without adjacent duplicates.

**Template:**
```java
Map<T, Integer> freq = buildFreqMap(input);
PriorityQueue<int[]> maxHeap = new PriorityQueue<>((a, b) -> b[0] - a[0]); // [freq, elem]
for (var e : freq.entrySet()) maxHeap.offer(new int[]{e.getValue(), encode(e.getKey())});

// Process in rounds: pick most frequent, place, decrement, put back if still needed
// For cooldown: queue of (freq, available_at_time)
```

---

## Heap vs Other Approaches

| Problem Type | Heap | Sort | Prefix Sum | Hash Map |
|-------------|------|------|-----------|---------|
| k largest (offline) | O(n log k) | O(n log n) | — | — |
| k largest (streaming) | O(log k) per element | Not applicable | — | — |
| k most frequent | O(n log k) | O(n log n) | — | O(n) bucket sort |
| Median stream | O(log n) add | Not applicable | — | O(1) bucket (bounded range) |
| Merge k sorted | O(n log k) | O(n log n) | — | — |
| Kth smallest matrix | O(k log k) | O(n² log n) | Binary search | — |

---

## Java PriorityQueue Patterns for Custom Objects

```java
// Records/ints: simple comparator
PriorityQueue<int[]> pq = new PriorityQueue<>(Comparator.comparingInt(a -> a[0]));

// Multi-key sort: primary by val, secondary by index
PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> 
    a[0] != b[0] ? Integer.compare(a[0], b[0]) : Integer.compare(a[1], b[1]));

// String with frequency: lower freq → evict first; for equal freq, lex later → evict first
PriorityQueue<String> pq = new PriorityQueue<>((a, b) -> {
    int fa = freq.get(a), fb = freq.get(b);
    return fa != fb ? fa - fb : b.compareTo(a);
});
```

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Common Mistakes](./Common%20Mistakes.md)
- [Complexity Analysis](./Complexity%20Analysis.md)

> **Last Updated:** 2026-06-26
