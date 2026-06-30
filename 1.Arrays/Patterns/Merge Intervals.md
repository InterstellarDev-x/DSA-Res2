# Merge Intervals Pattern

> **Topic:** [Arrays](../README.md) · **Difficulty:** Medium–Hard
> **Tags:** `intervals` `sorting` `greedy` `sweep-line`

---

## Table of Contents

1. [Pattern Overview](#pattern-overview)
2. [When to Use](#when-to-use)
3. [Recognition Cues](#recognition-cues)
4. [Complexity](#complexity)
5. [Java Templates](#java-templates)
6. [Common Mistakes](#common-mistakes)
7. [Variations](#variations)
8. [Practice Problems](#practice-problems)
9. [Related Patterns](#related-patterns)

---

## Pattern Overview

The Merge Intervals pattern processes a list of `[start, end]` intervals, most commonly by **sorting on start time** and then doing a greedy linear scan to merge, insert, or count overlaps.

**Core merge rule:**
```
If current.start <= last.end  → overlapping → extend last.end
Else                          → non-overlapping → add new interval
```

The sort step is the key enabler: once sorted, all overlapping intervals are consecutive.

---

## When to Use

- Merging overlapping intervals
- Inserting a new interval into a sorted list
- Finding gaps between intervals
- Counting intervals that overlap at a given point
- Meeting rooms / scheduling problems
- Activity selection

---

## Recognition Cues

| Cue in Problem | Pattern |
|----------------|---------|
| "merge all overlapping intervals" | Classic Merge Intervals |
| "insert interval and merge" | Insert + merge |
| "minimum meeting rooms" | Sweep line / min-heap |
| "maximum overlap at any point" | Sweep line events |
| "non-overlapping intervals to remove" | Greedy activity selection |
| "employee free time" | Multi-list merge |

---

## Complexity

| Variant | Time | Space |
|---------|------|-------|
| Sort + merge | O(n log n) | O(n) |
| Insert interval (pre-sorted) | O(n) | O(n) |
| Meeting rooms I (can hold?) | O(n log n) | O(1) |
| Meeting rooms II (min rooms) | O(n log n) | O(n) |

---

## Java Templates

### 1. Merge Overlapping Intervals

```java
public int[][] merge(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> a[0] - b[0]); // sort by start
    List<int[]> merged = new ArrayList<>();

    for (int[] interval : intervals) {
        if (merged.isEmpty() || merged.get(merged.size() - 1)[1] < interval[0]) {
            merged.add(interval); // no overlap
        } else {
            // overlap: extend the end
            merged.get(merged.size() - 1)[1] =
                Math.max(merged.get(merged.size() - 1)[1], interval[1]);
        }
    }
    return merged.toArray(new int[0][]);
}
// Time: O(n log n) | Space: O(n)
```

### 2. Insert Interval

```java
public int[][] insert(int[][] intervals, int[] newInterval) {
    List<int[]> result = new ArrayList<>();
    int i = 0, n = intervals.length;

    // Add all intervals ending before newInterval starts
    while (i < n && intervals[i][1] < newInterval[0]) {
        result.add(intervals[i++]);
    }
    // Merge all overlapping intervals with newInterval
    while (i < n && intervals[i][0] <= newInterval[1]) {
        newInterval[0] = Math.min(newInterval[0], intervals[i][0]);
        newInterval[1] = Math.max(newInterval[1], intervals[i][1]);
        i++;
    }
    result.add(newInterval);
    // Add remaining intervals
    while (i < n) result.add(intervals[i++]);

    return result.toArray(new int[0][]);
}
// Time: O(n) | Space: O(n)
```

### 3. Meeting Rooms I — Can One Person Attend All?

```java
public boolean canAttendMeetings(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> a[0] - b[0]);
    for (int i = 1; i < intervals.length; i++) {
        if (intervals[i][0] < intervals[i - 1][1]) return false; // overlap
    }
    return true;
}
// Time: O(n log n) | Space: O(1)
```

### 4. Meeting Rooms II — Minimum Rooms Required

```java
public int minMeetingRooms(int[][] intervals) {
    int[] starts = new int[intervals.length];
    int[] ends   = new int[intervals.length];
    for (int i = 0; i < intervals.length; i++) {
        starts[i] = intervals[i][0];
        ends[i]   = intervals[i][1];
    }
    Arrays.sort(starts);
    Arrays.sort(ends);

    int rooms = 0, endPtr = 0;
    for (int i = 0; i < starts.length; i++) {
        if (starts[i] < ends[endPtr]) rooms++; // new room needed
        else endPtr++;                          // room freed
    }
    return rooms;
}
// Time: O(n log n) | Space: O(n)
```

### 5. Non-overlapping Intervals — Minimum Removals (Activity Selection)

```java
public int eraseOverlapIntervals(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> a[1] - b[1]); // sort by END
    int count = 0, lastEnd = Integer.MIN_VALUE;
    for (int[] iv : intervals) {
        if (iv[0] >= lastEnd) {
            lastEnd = iv[1]; // keep this interval
        } else {
            count++; // remove this interval
        }
    }
    return count;
}
// Greedy: sort by end, greedily keep non-overlapping intervals
// Time: O(n log n) | Space: O(1)
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Sorting by end instead of start (for merge) | Sort by `start` for merge; sort by `end` for activity selection |
| Overlap condition: `<` vs `<=` | `intervals[i][0] <= lastEnd` means touching intervals merge; adjust per problem |
| Mutating the original input array | Clone interval arrays when needed |
| Not handling empty input | Guard `if (intervals.length == 0) return ...` |
| Meeting rooms: using a min-heap when two sorted arrays work | Two-pointer approach on sorted starts/ends is cleaner |

---

## Variations

| Variation | Key Idea |
|-----------|----------|
| Employee Free Time | Merge all intervals from all employees, find gaps |
| Interval List Intersections | Two pointer on two sorted interval lists |
| My Calendar I/II/III | TreeMap / Sweep line for dynamic insertions |
| Maximum CPU Load | Sweep line, track sum of loads |

### Interval Intersection

```java
public int[][] intervalIntersection(int[][] A, int[][] B) {
    List<int[]> result = new ArrayList<>();
    int i = 0, j = 0;
    while (i < A.length && j < B.length) {
        int lo = Math.max(A[i][0], B[j][0]);
        int hi = Math.min(A[i][1], B[j][1]);
        if (lo <= hi) result.add(new int[]{lo, hi});
        if (A[i][1] < B[j][1]) i++;
        else j++;
    }
    return result.toArray(new int[0][]);
}
```

---

## Practice Problems

| Problem | Difficulty | LeetCode |
|---------|-----------|----------|
| [Merge Intervals](https://leetcode.com/problems/merge-intervals/) | Medium | LC 56 |
| [Insert Interval](https://leetcode.com/problems/insert-interval/) | Medium | LC 57 |
| [Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/) | Medium | LC 435 |
| [Meeting Rooms](https://leetcode.com/problems/meeting-rooms/) | Easy | LC 252 |
| [Meeting Rooms II](https://leetcode.com/problems/meeting-rooms-ii/) | Medium | LC 253 |
| [Interval List Intersections](https://leetcode.com/problems/interval-list-intersections/) | Medium | LC 986 |
| [Employee Free Time](https://leetcode.com/problems/employee-free-time/) | Hard | LC 759 |
| [My Calendar III](https://leetcode.com/problems/my-calendar-iii/) | Hard | LC 732 |

---

## Related Patterns

- [Two Pointers](./Two%20Pointers.md) — used in interval intersection
- [Greedy](../../10.Greedy/README.md) — activity selection is a greedy problem
- [Sweep Line](../../13.Graphs/Patterns/Sweep%20Line.md) — for counting overlaps at any point

---

> **Interview Tip:** Always clarify: do touching intervals `[1,2],[2,3]` merge or not? The problem usually specifies. Default assumption: `[1,2],[2,3]` → merge to `[1,3]` (non-strict overlap).

> **Last Updated:** 2026-06-26
