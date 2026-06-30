> **Topic:** [Greedy Algorithms](../README.md) · **Design Problem 1 of 1**

# Greedy Iterator — Sorted Interval Stream

## Problem Statement
Design a data structure that supports a stream of intervals. It must:
- `void addInterval(int start, int end)` — Insert interval in O(log n), merging with any overlapping existing intervals
- `List<int[]> getIntervals()` — Return all intervals in sorted order by start time in O(n)

**Example:**
```
addInterval(1, 3)  → intervals: [[1,3]]
addInterval(6, 9)  → intervals: [[1,3],[6,9]]
addInterval(2, 5)  → intervals: [[1,5],[6,9]]  (merged with [1,3])
addInterval(8,10)  → intervals: [[1,5],[6,10]] (merged with [6,9])
```

## Core Idea: TreeMap with Merge-on-Insert
Use `TreeMap<Integer, Integer>` mapping `start → end`. On insert:
1. Find left neighbor (largest key ≤ new start) — merge if it overlaps
2. Find right neighbors (all keys between new start and new end) — merge all of them
3. Put merged interval

## Complete Java Implementation

```java
import java.util.*;

class GreedyIntervalIterator {
    // Key: interval start, Value: interval end
    private final TreeMap<Integer, Integer> map = new TreeMap<>();

    /**
     * Inserts [start, end] into the sorted structure, merging overlapping intervals.
     * Time: O(k log n) where k = number of intervals merged (amortized O(log n) per insert
     *       since each interval is inserted once and removed at most once)
     */
    public void addInterval(int start, int end) {
        // Step 1: Check left neighbor — does it extend into our interval?
        Map.Entry<Integer, Integer> left = map.floorEntry(start);
        if (left != null && left.getValue() >= start) {
            // Left interval overlaps: extend our start to left's start, end to max of both ends
            start = left.getKey();
            end = Math.max(end, left.getValue());
        }

        // Step 2: Absorb all right neighbors that overlap with [start, end]
        NavigableMap<Integer, Integer> rightOverlap = map.subMap(start, false, end, true);
        // Collect all entries that start within (start, end]
        // We need to check up to end because a right interval starting at 'end' overlaps
        Iterator<Map.Entry<Integer, Integer>> it = rightOverlap.entrySet().iterator();
        while (it.hasNext()) {
            Map.Entry<Integer, Integer> entry = it.next();
            end = Math.max(end, entry.getValue());
            it.remove();
        }

        // Step 3: Remove the left neighbor if it was merged
        if (left != null && left.getValue() >= start - (start - left.getKey())) {
            // Only remove left if we actually used it
            Map.Entry<Integer, Integer> check = map.floorEntry(start);
            if (check != null && check.getValue() >= start) {
                map.remove(check.getKey());
            }
        }

        // Step 4: Insert merged interval
        map.put(start, end);
    }

    /**
     * Returns all intervals sorted by start time.
     * Time: O(n)
     */
    public List<int[]> getIntervals() {
        List<int[]> result = new ArrayList<>();
        for (Map.Entry<Integer, Integer> e : map.entrySet()) {
            result.add(new int[]{e.getKey(), e.getValue()});
        }
        return result;
    }

    // ── Clean implementation (production-ready) ────────────────────────────────
    /**
     * Cleaner addInterval using a rebuilt approach for clarity.
     * Same asymptotic complexity.
     */
    public void addIntervalClean(int start, int end) {
        // Extend left if left neighbor overlaps
        Map.Entry<Integer, Integer> lo = map.floorEntry(start);
        if (lo != null && lo.getValue() >= start) {
            map.remove(lo.getKey());
            start = lo.getKey();
            end = Math.max(end, lo.getValue());
        }

        // Absorb all intervals that start within [start, end]
        while (true) {
            Map.Entry<Integer, Integer> next = map.ceilingEntry(start);
            if (next == null || next.getKey() > end) break;
            end = Math.max(end, next.getValue());
            map.remove(next.getKey());
        }

        map.put(start, end);
    }

    public static void main(String[] args) {
        GreedyIntervalIterator gii = new GreedyIntervalIterator();
        gii.addIntervalClean(1, 3);
        gii.addIntervalClean(6, 9);
        System.out.println("After [1,3],[6,9]: " + formatIntervals(gii.getIntervals()));
        // Expected: [[1,3],[6,9]]

        gii.addIntervalClean(2, 5);
        System.out.println("After adding [2,5]: " + formatIntervals(gii.getIntervals()));
        // Expected: [[1,5],[6,9]]

        gii.addIntervalClean(8, 10);
        System.out.println("After adding [8,10]: " + formatIntervals(gii.getIntervals()));
        // Expected: [[1,5],[6,10]]

        gii.addIntervalClean(0, 20);
        System.out.println("After adding [0,20]: " + formatIntervals(gii.getIntervals()));
        // Expected: [[0,20]] (all merged)
    }

    private static String formatIntervals(List<int[]> intervals) {
        StringBuilder sb = new StringBuilder("[");
        for (int i = 0; i < intervals.size(); i++) {
            sb.append("[").append(intervals.get(i)[0]).append(",").append(intervals.get(i)[1]).append("]");
            if (i < intervals.size() - 1) sb.append(",");
        }
        return sb.append("]").toString();
    }
}
```

## Detailed Trace

| Step | Operation | Map State (start→end) | Note |
|------|-----------|----------------------|------|
| 1 | addInterval(1,3) | {1→3} | No neighbors |
| 2 | addInterval(6,9) | {1→3, 6→9} | No overlap with [1,3] |
| 3 | addInterval(2,5) | {1→5, 6→9} | Floor of 2 is 1→3; 3≥2 so merge: start=1, end=max(5,3)=5 |
| 4 | addInterval(8,10)| {1→5, 6→10} | Floor of 8 is 6→9; 9≥8 so merge: start=6, end=max(10,9)=10 |
| 5 | addInterval(0,20)| {0→20} | Floor of 0 is null; absorbs 1→5, 6→10 |

## Complexity Analysis

| Operation | Time | Space |
|-----------|------|-------|
| `addInterval` | O(k log n) amortized O(log n) | O(1) extra |
| `getIntervals` | O(n) | O(n) |
| Space (structure) | — | O(n) |

Where k = intervals merged (each interval inserted once, removed at most once → amortized O(log n)).

## Why TreeMap?

TreeMap provides:
- `floorEntry(key)` — find largest key ≤ query in O(log n)
- `ceilingEntry(key)` — find smallest key ≥ query in O(log n)
- `subMap(from, to)` — range view in O(log n) + O(k) to iterate
- Sorted iteration via `entrySet()` in O(n)

A `HashMap` would require O(n) scan to find overlapping intervals.

## Related Problems
- [Insert Interval (LC #57)](../Patterns/Interval%20Scheduling.md) — one-shot insertion into sorted array
- [Merge Intervals (LC #56)](../Patterns/Interval%20Scheduling.md) — batch merge of unsorted intervals

> **Last Updated:** 2026-06-26
