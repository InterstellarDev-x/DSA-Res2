> **Topic:** [Greedy Algorithms](../README.md) · **OA Questions — Microsoft**

# Microsoft OA — Greedy Algorithms

## Frequency Table

| Problem | LC # | Frequency | Difficulty | Pattern | Last Seen |
|---------|------|-----------|------------|---------|-----------|
| Jump Game | #55 | ⭐⭐⭐⭐ | Medium | [Jump Game](../Patterns/Jump%20Game.md) | 2025 Q4 |
| Non-overlapping Intervals | #435 | ⭐⭐⭐⭐ | Medium | [Interval Scheduling](../Patterns/Interval%20Scheduling.md) | 2025 Q3 |
| Insert Interval | #57 | ⭐⭐⭐⭐ | Medium | [Interval Scheduling](../Patterns/Interval%20Scheduling.md) | 2025 Q3 |
| Meeting Rooms II | #253 | ⭐⭐⭐ | Medium | [Greedy with Heap](../Patterns/Greedy%20with%20Heap.md) | 2025 Q2 |
| Task Scheduler | #621 | ⭐⭐⭐ | Medium | [Greedy with Heap](../Patterns/Greedy%20with%20Heap.md) | 2025 Q1 |
| Lemonade Change | #860 | ⭐⭐ | Easy | [Greedy on Strings](../Patterns/Greedy%20on%20Strings.md) | 2024 Q3 |

## Top Focus: Jump Game (⭐⭐⭐⭐)

**Why Microsoft loves it:** Simple code, deep insight required. Tests if candidates understand greedy vs DP trade-offs.

**Microsoft-specific follow-up questions:**
1. "How would you solve it with DP?" → `dp[i] = any j < i where dp[j] && j + nums[j] >= i` → O(n²)
2. "Why is greedy better?" → O(n) vs O(n²); greedy invariant: `maxReach` strictly non-decreasing
3. "What if nums[i] is the exact number of steps (not maximum)?" → BFS required, greedy fails

**Code they want to see:**
```cpp
#include <bits/stdc++.h>
using namespace std;

bool canJump(vector<int>& nums) {
    int maxReach = 0;
    for (int i = 0; i < (int)nums.size(); i++) {
        if (i > maxReach) return false;
        maxReach = max(maxReach, i + nums[i]);
    }
    return true;
}
```

**Microsoft style preference:**
- No early return inside loop unless clearly necessary
- Single variable (`maxReach`) over two-variable approaches
- Comments explaining the invariant

---

## Second Focus: Insert Interval (⭐⭐⭐⭐)

**Why Microsoft loves it:** Tests systematic interval manipulation — a common scenario in scheduling and calendar systems (core Microsoft products).

**Key insight:** Three phases: (1) copy non-overlapping before, (2) merge overlapping, (3) copy non-overlapping after.

**Microsoft-specific follow-up questions:**
1. "How would you modify this to delete an interval instead?" → Similar 3-phase approach
2. "What if the input isn't sorted?" → Sort first, then insert, O(n log n)
3. "What if intervals can have the same start?" → Algorithm handles it; explain why

**Code they want to see:**
```cpp
#include <bits/stdc++.h>
using namespace std;

vector<vector<int>> insert(vector<vector<int>>& intervals, vector<int>& newInterval) {
    vector<vector<int>> result;
    int i = 0, n = intervals.size();
    // Phase 1: Add all intervals ending before newInterval starts
    while (i < n && intervals[i][1] < newInterval[0]) result.push_back(intervals[i++]);
    // Phase 2: Merge all overlapping intervals
    while (i < n && intervals[i][0] <= newInterval[1]) {
        newInterval[0] = min(newInterval[0], intervals[i][0]);
        newInterval[1] = max(newInterval[1], intervals[i][1]);
        i++;
    }
    result.push_back(newInterval);
    // Phase 3: Add remaining intervals
    while (i < n) result.push_back(intervals[i++]);
    return result;
}
```

---

## Third Focus: Non-overlapping Intervals (⭐⭐⭐⭐)

**Key insight they test:** "Why sort by end time, not start time?" Answer: sorting by end maximizes the number of intervals we can keep (greedy exchange argument).

---

## Microsoft OA Format Notes
- 2 problems, 60 minutes (AZ-900/SDE online assessments)
- Microsoft favors practical problems (calendars, scheduling) that map to real product scenarios
- Clean, well-commented code preferred over terse one-liners
- Edge case handling (empty array, single interval, fully contained intervals) expected

> **Last Updated:** 2026-06-26
