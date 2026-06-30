> **Topic:** [Greedy Algorithms](../README.md) · **Interview Problems — Microsoft**

# Microsoft Interview — Greedy Algorithms Deep Dive

## Problem 1: Non-overlapping Intervals (#435) — Deep Dive

### Problem Statement
Given an array of intervals, find the minimum number of intervals to remove to make the rest non-overlapping.

### Why Sort by End Time (Not Start Time)?

**Common mistake:** Sort by start time. This fails because:
- Consider: [[1,10], [2,3], [4,8], [5,9]]
- Sorted by start: [[1,10], [2,3], [4,8], [5,9]]
- Greedy by start: Keep [1,10], then everything overlaps with it → remove 3. **Answer: 3 (wrong)**
- Sorted by end: [[2,3], [4,8], [5,9], [1,10]]
- Greedy by end: Keep [2,3], [4,8], skip [5,9] (overlaps [4,8]), skip [1,10] → **Answer: 2 (correct)**

**Why end-time sort maximizes kept intervals:**
Keeping the interval that ends earliest leaves maximum room for future intervals. This is the Activity Selection Theorem.

**Exchange argument proof:**
Suppose optimal solution keeps interval `X` but not `Y`, where `Y` ends before `X` and both start after the previous kept interval.
- Swap `X` with `Y` in the solution.
- `Y` still doesn't overlap with previous (same start requirement).
- `Y` ends no later than `X`, so it creates at most as much conflict with future intervals as `X`.
- Therefore swapping to `Y` is at least as good.
- By induction, always choosing the earliest-ending non-overlapping interval is optimal. ∎

### Full Java Solution
```java
public int eraseOverlapIntervals(int[][] intervals) {
    // Sort by end time — greedy choice: keep interval that ends earliest
    Arrays.sort(intervals, Comparator.comparingInt(a -> a[1]));

    int kept = 0;                    // Count of kept (non-overlapping) intervals
    int prevEnd = Integer.MIN_VALUE; // End time of last kept interval

    for (int[] interval : intervals) {
        if (interval[0] >= prevEnd) {
            // No overlap with previous kept interval — keep this one
            kept++;
            prevEnd = interval[1];
        }
        // Else: overlaps — skip (increment removal count implicitly)
    }

    return intervals.length - kept; // Minimum removals = total - max kept
}
```

### Trace: [[1,2],[2,3],[3,4],[1,3]]

Sorted by end: [[1,2],[2,3],[1,3],[3,4]]

| i | interval | prevEnd | overlap? | kept | prevEnd updated |
|---|----------|---------|----------|------|-----------------|
| 0 | [1,2] | MIN | No (1≥MIN) | 1 | 2 |
| 1 | [2,3] | 2 | No (2≥2) | 2 | 3 |
| 2 | [1,3] | 3 | Yes (1<3) | 2 | 3 (no change) |
| 3 | [3,4] | 3 | No (3≥3) | 3 | 4 |

`kept = 3`, result = `4 - 3 = 1` ✓

---

## Problem 2: Insert Interval (#57) — Merge Logic

### Problem Statement
Given a list of non-overlapping intervals sorted by start time, insert a new interval and merge if necessary. Return the resulting interval list.

### Three-Phase Logic

```
Phase 1: Collect all intervals that END before newInterval STARTS
         condition: intervals[i][1] < newInterval[0]

Phase 2: Merge all intervals that OVERLAP with newInterval
         condition: intervals[i][0] <= newInterval[1]
         action: expand newInterval to cover all

Phase 3: Append remaining intervals (they start after newInterval ends)
```

### Visual Explanation
```
Original: [1,3] [6,9]     New: [2,5]

Phase 1: [1,3]? → end(3) < start(2)? NO → skip to Phase 2
Phase 2: [1,3]? → start(1) ≤ end(5)? YES → merge: newInterval = [min(2,1), max(5,3)] = [1,5]
         [6,9]? → start(6) ≤ end(5)? NO → stop Phase 2
Phase 3: [6,9] added as-is

Result: [[1,5],[6,9]] ✓
```

### Full Java Solution
```java
public int[][] insert(int[][] intervals, int[] newInterval) {
    List<int[]> result = new ArrayList<>();
    int i = 0, n = intervals.length;

    // Phase 1: Add all intervals ending before newInterval starts (no overlap)
    while (i < n && intervals[i][1] < newInterval[0]) {
        result.add(intervals[i++]);
    }

    // Phase 2: Merge all overlapping intervals into newInterval
    while (i < n && intervals[i][0] <= newInterval[1]) {
        newInterval[0] = Math.min(newInterval[0], intervals[i][0]);
        newInterval[1] = Math.max(newInterval[1], intervals[i][1]);
        i++;
    }
    result.add(newInterval); // Add the merged interval

    // Phase 3: Add remaining non-overlapping intervals
    while (i < n) {
        result.add(intervals[i++]);
    }

    return result.toArray(new int[0][]);
}
```

### Edge Cases Microsoft Tests

| Case | Example Input | Expected Output |
|------|--------------|-----------------|
| Insert at beginning | intervals=[[3,5]], new=[1,2] | [[1,2],[3,5]] |
| Insert at end | intervals=[[1,2]], new=[3,5] | [[1,2],[3,5]] |
| Fully contained | intervals=[[1,5]], new=[2,3] | [[1,5]] |
| Merges all | intervals=[[1,3],[4,6]], new=[2,5] | [[1,6]] |
| Empty input | intervals=[], new=[1,2] | [[1,2]] |

### Complexity Analysis

| Algorithm | Time | Space |
|-----------|------|-------|
| Insert Interval | O(n) | O(n) |
| Non-overlapping Intervals | O(n log n) sort + O(n) scan = O(n log n) | O(1) |

---

## Microsoft Interview Progression

| Round | Expected | Greedy Focus |
|-------|----------|--------------|
| Online Assessment | Working solution | Insert Interval or Non-overlapping Intervals |
| Phone Screen | Clean code + edge cases | Both problems with explained approach |
| Onsite 1 | Optimal solution + "why sort by end?" | Exchange argument for Non-overlapping |
| Onsite 2 | System design | "Design a calendar app using interval merging" |

> **Last Updated:** 2026-06-26
