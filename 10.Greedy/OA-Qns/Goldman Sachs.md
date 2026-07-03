> **Topic:** [Greedy Algorithms](../README.md) · **OA Questions — Goldman Sachs**

# Goldman Sachs OA — Greedy Algorithms

## Frequency Table

| Problem | LC # | Frequency | Difficulty | Pattern | Last Seen |
|---------|------|-----------|------------|---------|-----------|
| Gas Station | #134 | ⭐⭐⭐⭐ | Medium | [Interval Scheduling](../Patterns/Interval%20Scheduling.md) | 2025 Q4 |
| Candy | #135 | ⭐⭐⭐⭐ | Hard | [Greedy on Strings](../Patterns/Greedy%20on%20Strings.md) | 2025 Q3 |
| Task Scheduler | #621 | ⭐⭐⭐ | Medium | [Greedy with Heap](../Patterns/Greedy%20with%20Heap.md) | 2025 Q2 |
| Jump Game II | #45 | ⭐⭐⭐ | Medium | [Jump Game](../Patterns/Jump%20Game.md) | 2025 Q1 |
| Non-overlapping Intervals | #435 | ⭐⭐ | Medium | [Interval Scheduling](../Patterns/Interval%20Scheduling.md) | 2024 Q4 |

## Top Focus: Gas Station (⭐⭐⭐⭐)

**Why Goldman Sachs loves it:** Maps directly to financial portfolio optimization — "can you complete a circuit with these cash flows?" Tests both algorithmic thinking and business intuition.

**Goldman-specific framing:** Often presented as: "You are a trader visiting N markets in a circle. Each market i gives you `profit[i]` and costs `fee[i]` to travel to the next. Starting with 0 capital, find the starting market that lets you complete the full circuit."

**Goldman follow-up questions:**
1. "What is the minimum starting capital needed if you must start at market 0?" → Track minimum prefix sum deficit
2. "What if the markets are not circular?" → Simpler: just find the first position where prefix sum is always ≥ 0
3. "What if there are multiple valid starting points?" → Return the lexicographically smallest (first valid)

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

**Business context Goldman appreciates:**
- `totalGain >= 0` ↔ "Net cash flow is non-negative — the route is viable in aggregate"
- Local candidate reset ↔ "If you're in deficit after visiting station i, no earlier start could have helped — reset"

---

## Second Focus: Candy (⭐⭐⭐⭐)

**Why Goldman Sachs loves it:** Hard greedy problem requiring two-pass technique. Tests thoroughness and ability to reason about constraints from both directions.

**Goldman-specific framing:** "You manage N analysts ranked by performance. Each must receive a bonus (≥ 1 unit). Analysts with higher performance than their neighbor must receive strictly more bonus. Minimize total bonus payout."

**Goldman follow-up questions:**
1. "What if the constraint is ≥ (not strict >)?" → Only need 1 candy per person (all equal ratings okay)
2. "What if the bonus must be from a set {1, 2, 5, 10}?" → DP required, greedy may not work
3. "What is the minimum bonus if we allow circular arrangement?" → Circular candy problem — NP-hard variant

**Code they want to see:**
```cpp
#include <bits/stdc++.h>
using namespace std;

int candy(vector<int>& ratings) {
    int n = ratings.size();
    vector<int> candies(n, 1);
    for (int i = 1; i < n; i++) {
        if (ratings[i] > ratings[i - 1]) candies[i] = candies[i - 1] + 1;
    }
    for (int i = n - 2; i >= 0; i--) {
        if (ratings[i] > ratings[i + 1]) {
            candies[i] = max(candies[i], candies[i + 1] + 1);
        }
    }
    int total = 0;
    for (auto& c : candies) total += c;
    return total;
}
```

**Why two passes:** Left-to-right ensures left-neighbor constraint; right-to-left ensures right-neighbor constraint. `max` at each position satisfies both simultaneously.

---

## Goldman Sachs OA Format Notes
- 3 problems, 90 minutes (Hackerrank-based)
- Strong emphasis on financial modeling — expect problems framed in trading/portfolio context
- Complexity analysis required in code comments
- Hard problems (like Candy) distinguish top candidates
- Strong preference for candidates who can explain WHY the greedy choice is optimal

> **Last Updated:** 2026-06-26
