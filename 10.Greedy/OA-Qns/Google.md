> **Topic:** [Greedy Algorithms](../README.md) · **OA Questions — Google**

# Google OA — Greedy Algorithms

## Frequency Table

| Problem | LC # | Frequency | Difficulty | Pattern | Last Seen |
|---------|------|-----------|------------|---------|-----------|
| Partition Labels | #763 | ⭐⭐⭐⭐⭐ | Medium | [Interval Scheduling](../Patterns/Interval%20Scheduling.md) | 2025 Q4 |
| Meeting Rooms II | #253 | ⭐⭐⭐⭐ | Medium | [Greedy with Heap](../Patterns/Greedy%20with%20Heap.md) | 2025 Q4 |
| Minimum Arrows | #452 | ⭐⭐⭐⭐ | Medium | [Interval Scheduling](../Patterns/Interval%20Scheduling.md) | 2025 Q3 |
| Jump Game II | #45 | ⭐⭐⭐ | Medium | [Jump Game](../Patterns/Jump%20Game.md) | 2025 Q2 |
| Hand of Straights | #846 | ⭐⭐⭐ | Medium | [Greedy with Heap](../Patterns/Greedy%20with%20Heap.md) | 2025 Q1 |
| Merge Triplets | #1899 | ⭐⭐ | Medium | [Greedy on Strings](../Patterns/Greedy%20on%20Strings.md) | 2024 Q4 |

## Top Focus: Partition Labels (⭐⭐⭐⭐⭐)

**Why Google loves it:** Tests two-pass greedy reasoning — build a map, then greedily extend windows. Clean, elegant code expected.

**Google-specific follow-up questions:**
1. "What if characters can appear in at most 2 partitions?" (→ modified last-occurrence logic)
2. "What if we want the minimum number of partitions where no two same characters are in different parts?" (→ same algorithm)
3. "Can you do it in one pass?" (→ No — you need last-occurrence info before scanning)

**Code they want to see:**
```java
public List<Integer> partitionLabels(String s) {
    int[] last = new int[26];
    for (int i = 0; i < s.length(); i++) last[s.charAt(i) - 'a'] = i;
    List<Integer> result = new ArrayList<>();
    int start = 0, end = 0;
    for (int i = 0; i < s.length(); i++) {
        end = Math.max(end, last[s.charAt(i) - 'a']);
        if (i == end) {
            result.add(end - start + 1);
            start = i + 1;
        }
    }
    return result;
}
```

**Google style notes:**
- Use meaningful variable names (`start`, `end`, not `i`, `j`)
- Comment the "stretch window" logic
- Handle edge case: single character string

---

## Second Focus: Meeting Rooms II (⭐⭐⭐⭐)

**Why Google loves it:** Tests data structure selection (why min-heap over max-heap or sorted array), amortized analysis, and clean abstraction.

**Google-specific follow-up questions:**
1. "What is the amortized complexity of each insert?" → O(log n) for heap offer/poll
2. "How would you solve this with a sweep line instead?" → event-based +1/-1 count
3. "What if meetings have priorities and high-priority meetings can't share rooms?" → modified assignment

**Sweep line alternative** (Google sometimes asks for this):
```java
public int minMeetingRoomsSweep(int[][] intervals) {
    int[] starts = new int[intervals.length];
    int[] ends = new int[intervals.length];
    for (int i = 0; i < intervals.length; i++) {
        starts[i] = intervals[i][0];
        ends[i] = intervals[i][1];
    }
    Arrays.sort(starts);
    Arrays.sort(ends);
    int rooms = 0, endPtr = 0;
    for (int start : starts) {
        if (start < ends[endPtr]) rooms++;
        else endPtr++;
    }
    return rooms;
}
```

---

## Third Focus: Minimum Arrows (⭐⭐⭐⭐)

**Why Google loves it:** Non-obvious connection between "minimum arrows" and "maximum non-overlapping intervals + 1".

**Key insight:** Minimum arrows = 1 + number of gaps between balloon groups = same as Activity Selection count.

---

## Google OA Format Notes
- 2–3 problems, 60–90 minutes (Kickstart / online assessment varies)
- Google expects clean code with O(n log n) or better
- Verbal/written justification for greedy choice expected in system design rounds
- Greedy appears heavily in Google L4/L5 loops

> **Last Updated:** 2026-06-26
