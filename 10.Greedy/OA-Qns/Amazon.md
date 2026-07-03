> **Topic:** [Greedy Algorithms](../README.md) · **OA Questions — Amazon**

# Amazon OA — Greedy Algorithms

## Frequency Table

| Problem | LC # | Frequency | Difficulty | Pattern | Last Seen |
|---------|------|-----------|------------|---------|-----------|
| Jump Game II | #45 | ⭐⭐⭐⭐⭐ | Medium | [Jump Game](../Patterns/Jump%20Game.md) | 2025 Q4 |
| Gas Station | #134 | ⭐⭐⭐⭐ | Medium | [Interval Scheduling](../Patterns/Interval%20Scheduling.md) | 2025 Q3 |
| Non-overlapping Intervals | #435 | ⭐⭐⭐⭐ | Medium | [Interval Scheduling](../Patterns/Interval%20Scheduling.md) | 2025 Q2 |
| Valid Parenthesis String | #678 | ⭐⭐⭐ | Medium | [Greedy on Strings](../Patterns/Greedy%20on%20Strings.md) | 2025 Q1 |
| Task Scheduler | #621 | ⭐⭐⭐ | Medium | [Greedy with Heap](../Patterns/Greedy%20with%20Heap.md) | 2024 Q4 |
| Lemonade Change | #860 | ⭐⭐ | Easy | [Greedy on Strings](../Patterns/Greedy%20on%20Strings.md) | 2024 Q3 |
| Minimum Arrows | #452 | ⭐⭐ | Medium | [Interval Scheduling](../Patterns/Interval%20Scheduling.md) | 2024 Q2 |

## Top Focus: Jump Game II (⭐⭐⭐⭐⭐)

**Why Amazon loves it:** Tests BFS-level thinking, greedy optimization, and clean code under time pressure.

**Expected approach:** O(n) greedy with `curEnd` / `farthest` variables — NOT recursion.

**Amazon-specific follow-up questions:**
1. "What if each jump has a cost proportional to its distance? Minimize total cost." (→ DP)
2. "What if you can move left or right?" (→ BFS)
3. "What is the minimum number of jumps if you can start from any index?" (→ modified BFS)

**Code they want to see:**
```cpp
#include <bits/stdc++.h>
using namespace std;

int jump(vector<int>& nums) {
    int jumps = 0, curEnd = 0, farthest = 0;
    for (int i = 0; i < (int)nums.size() - 1; i++) {
        farthest = max(farthest, i + nums[i]);
        if (i == curEnd) {
            jumps++;
            curEnd = farthest;
        }
    }
    return jumps;
}
```

**Common mistakes Amazon penalizes:**
- Using `i <= nums.length - 1` instead of `i < nums.length - 1` (off-by-one)
- Returning `jumps + 1` instead of `jumps`
- Using recursive DFS (TLE on large inputs)

---

## Second Focus: Gas Station (⭐⭐⭐⭐)

**Why Amazon loves it:** Tests "circular array + greedy candidate reset" — a non-obvious technique.

**Key insight they want:** If total gas ≥ total cost, a solution always exists. The candidate start is the index after the last deficit.

**Amazon-specific follow-up questions:**
1. "What if multiple valid starting stations exist?" (→ return any; proof shows only one)
2. "What is the minimum fuel tank capacity needed?" (→ track maximum deficit)
3. "What if roads have varying distances?" (→ same algorithm, different cost array)

**Code they want to see:**
```cpp
#include <bits/stdc++.h>
using namespace std;

int canCompleteCircuit(vector<int>& gas, vector<int>& cost) {
    int totalGain = 0, currentGain = 0, startStation = 0;
    for (int i = 0; i < (int)gas.size(); i++) {
        int gain = gas[i] - cost[i];
        totalGain += gain;
        currentGain += gain;
        if (currentGain < 0) {
            startStation = i + 1;
            currentGain = 0;
        }
    }
    return totalGain >= 0 ? startStation : -1;
}
```

---

## Third Focus: Non-overlapping Intervals (⭐⭐⭐⭐)

**Why Amazon loves it:** Classic "why sort by end?" question — tests greedy proof knowledge.

**Amazon behavioral tie-in:** Often paired with "tell me about a time you had to prioritize tasks with conflicting deadlines" — connect the greedy choice to real decision-making.

---

## Amazon OA Format Notes
- 2 problems, 90 minutes
- Greedy problems appear in ~60% of Amazon SDE-1/SDE-2 OAs
- Auto-graded: edge cases (empty array, single element, all same value) are always tested
- No partial credit — all test cases must pass

> **Last Updated:** 2026-06-26
