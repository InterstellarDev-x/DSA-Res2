> **Topic:** [Greedy Algorithms](../README.md) · **Interview Problems — Amazon**

# Amazon Interview — Greedy Algorithms Deep Dive

## Problem 1: Gas Station (#134) — Deep Dive

### Problem Statement
There are `n` gas stations in a circle. Station `i` has `gas[i]` liters available and costs `cost[i]` to travel to station `(i+1) % n`. Starting with an empty tank, find the starting station index that allows completing the full circuit, or return -1 if impossible.

### Key Insight 1: Feasibility Check
If `sum(gas) < sum(cost)`, no solution exists. If `sum(gas) >= sum(cost)`, exactly one solution exists.

**Proof:** If total gain ≥ 0, the cumulative gain array has a minimum. Starting just after the minimum guarantees we never go negative. This is the correct start.

### Key Insight 2: Local Candidate Reset
If starting from station `s` leads to a deficit at station `t`, then no station between `s` and `t` can be the correct start.

**Why:** If we fail at `t` starting from `s`, then the partial sums from `s+1`, `s+2`, ..., `t-1` are all less than the partial sum from `s` (because `gain[s] >= 0` is required to start). So they would also fail at `t`.

### Full C++ Solution
```cpp
#include <bits/stdc++.h>
using namespace std;

int canCompleteCircuit(vector<int>& gas, vector<int>& cost) {
    int totalGain = 0;    // Sum of all gains — determines feasibility
    int currentGain = 0;  // Running gain from current candidate start
    int startStation = 0; // Current candidate for starting station

    for (int i = 0; i < (int)gas.size(); i++) {
        int gain = gas[i] - cost[i]; // Net gain at station i
        totalGain += gain;
        currentGain += gain;

        // If we can't reach station i+1 from our current start, reset
        if (currentGain < 0) {
            startStation = i + 1; // Try starting from next station
            currentGain = 0;      // Reset running gain
        }
    }

    // If total gain is negative, no solution exists
    return totalGain >= 0 ? startStation : -1;
}
```

### Trace: gas=[1,2,3,4,5], cost=[3,4,5,1,2]

| i | gas | cost | gain | totalGain | currentGain | startStation |
|---|-----|------|------|-----------|-------------|--------------|
| 0 | 1 | 3 | -2 | -2 | -2 | 1 (reset) |
| 1 | 2 | 4 | -2 | -4 | -2 | 2 (reset) |
| 2 | 3 | 5 | -2 | -6 | -2 | 3 (reset) |
| 3 | 4 | 1 | +3 | -3 | +3 | 3 (no reset) |
| 4 | 5 | 2 | +3 | 0 | +6 | 3 (no reset) |

`totalGain = 0 >= 0` → Answer: **3**

Verify: Start at 3 (gas=4, cost=1) → tank=3 → station 4 (gas=5, cost=2) → tank=6 → station 0 (gas=1, cost=3) → tank=4 → station 1 (gas=2, cost=4) → tank=2 → station 2 (gas=3, cost=5) → tank=0 ✓

### Amazon LP Alignment

| Leadership Principle | Connection |
|---------------------|-----------|
| **Dive Deep** | Understanding WHY reset works (no station between s and t can be valid) |
| **Invent and Simplify** | O(n) one-pass vs O(n²) brute force |
| **Are Right, A Lot** | Proof that solution is unique when totalGain ≥ 0 |
| **Deliver Results** | Handles all edge cases: single station, no solution, exact 0 gain |

---

## Problem 2: Jump Game II (#45) — BFS Analogy

### Problem Statement
Given integer array `nums` where `nums[i]` is the maximum jump length from index `i`, return the minimum number of jumps to reach `nums[n-1]`.

### BFS Layer Analogy
Think of the array as a BFS graph:
- **Level 0:** Index 0
- **Level 1:** All indices reachable from level 0 in one jump
- **Level 2:** All indices reachable from level 1 in one jump
- ...

`curEnd` = right boundary of current BFS level
`farthest` = farthest index any node in current level can reach (= right boundary of next level)
`jumps` = current BFS depth

When `i == curEnd`, we've exhausted the current level → increment jumps, expand to next level.

### Full C++ Solution
```cpp
#include <bits/stdc++.h>
using namespace std;

int jump(vector<int>& nums) {
    int jumps = 0;    // BFS depth (number of jumps)
    int curEnd = 0;   // Right boundary of current BFS level
    int farthest = 0; // Farthest reach of any node in current level

    // We stop at nums.size() - 2 because we don't need to jump FROM the last index
    for (int i = 0; i < (int)nums.size() - 1; i++) {
        farthest = max(farthest, i + nums[i]);
        if (i == curEnd) {    // End of current BFS level
            jumps++;           // Move to next level
            curEnd = farthest; // Expand to next level boundary
        }
    }
    return jumps;
}
```

### BFS Visualization: nums=[2,3,1,1,4]

```
Index:   0   1   2   3   4
Value:   2   3   1   1   4

Level 0: [0]
  - From 0: can reach 1,2 (0+2=2)
  - farthest = 2, curEnd = 0 → at i=0, jump! jumps=1, curEnd=2

Level 1: [1, 2]
  - From 1: can reach 1..4 (1+3=4)
  - From 2: can reach 3 (2+1=3)
  - farthest = 4, curEnd = 2 → at i=2, jump! jumps=2, curEnd=4

Level 2: [3, 4] — index 4 is the destination ✓
```

**Answer: 2 jumps**

### Complexity Analysis

| Aspect | Value | Reason |
|--------|-------|--------|
| Time | O(n) | Single pass |
| Space | O(1) | Three integer variables |
| Greedy correctness | ✓ | Each level minimizes jumps (BFS = shortest path) |

---

## Amazon Interview Progression

| Round | Expected | Greedy Focus |
|-------|----------|--------------|
| Phone Screen | Working solution, basic complexity | Jump Game I or Gas Station |
| Virtual Onsite 1 | Optimal O(n) solution + proof | Jump Game II + follow-ups |
| Virtual Onsite 2 | System design tie-in | "Design a trip planner using greedy scheduling" |
| Bar Raiser | Edge cases + alternative approaches | All three problems + why greedy vs DP |

> **Last Updated:** 2026-06-26
