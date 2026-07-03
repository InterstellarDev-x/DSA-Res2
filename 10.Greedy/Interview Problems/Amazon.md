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

### Full Rust Solution
```rust
fn can_complete_circuit(gas: &[i32], cost: &[i32]) -> i32 {
    let mut total_gain = 0i32;      // Sum of all gains — determines feasibility
    let mut current_gain = 0i32;    // Running gain from current candidate start
    let mut start_station = 0usize; // Current candidate for starting station

    for i in 0..gas.len() {
        let gain = gas[i] - cost[i]; // Net gain at station i
        total_gain += gain;
        current_gain += gain;

        // If we can't reach station i+1 from our current start, reset
        if current_gain < 0 {
            start_station = i + 1; // Try starting from next station
            current_gain = 0;      // Reset running gain
        }
    }

    // If total gain is negative, no solution exists
    if total_gain >= 0 { start_station as i32 } else { -1 }
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

`cur_end` = right boundary of current BFS level
`farthest` = farthest index any node in current level can reach (= right boundary of next level)
`jumps` = current BFS depth

When `i == cur_end`, we've exhausted the current level → increment jumps, expand to next level.

### Full Rust Solution
```rust
fn jump(nums: &[i32]) -> i32 {
    let mut jumps = 0i32;       // BFS depth (number of jumps)
    let mut cur_end = 0usize;   // Right boundary of current BFS level
    let mut farthest = 0usize;  // Farthest reach of any node in current level

    // We stop at nums.len() - 2 because we don't need to jump FROM the last index
    for i in 0..nums.len() - 1 {
        farthest = farthest.max(i + nums[i] as usize);
        if i == cur_end {    // End of current BFS level
            jumps += 1;      // Move to next level
            cur_end = farthest; // Expand to next level boundary
        }
    }
    jumps
}
```

### BFS Visualization: nums=[2,3,1,1,4]

```
Index:   0   1   2   3   4
Value:   2   3   1   1   4

Level 0: [0]
  - From 0: can reach 1,2 (0+2=2)
  - farthest = 2, cur_end = 0 → at i=0, jump! jumps=1, cur_end=2

Level 1: [1, 2]
  - From 1: can reach 1..4 (1+3=4)
  - From 2: can reach 3 (2+1=3)
  - farthest = 4, cur_end = 2 → at i=2, jump! jumps=2, cur_end=4

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
