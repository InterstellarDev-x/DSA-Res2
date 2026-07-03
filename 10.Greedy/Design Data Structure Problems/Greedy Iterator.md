> **Topic:** [Greedy Algorithms](../README.md) · **Design Problem 1 of 1**

# Greedy Iterator — Sorted Interval Stream

## Problem Statement
Design a data structure that supports a stream of intervals. It must:
- `void addInterval(int start, int end)` — Insert interval in O(log n), merging with any overlapping existing intervals
- `vector<pair<int,int>> getIntervals()` — Return all intervals in sorted order by start time in O(n)

**Example:**
```
addInterval(1, 3)  → intervals: [[1,3]]
addInterval(6, 9)  → intervals: [[1,3],[6,9]]
addInterval(2, 5)  → intervals: [[1,5],[6,9]]  (merged with [1,3])
addInterval(8,10)  → intervals: [[1,5],[6,10]] (merged with [6,9])
```

## Core Idea: std::map with Merge-on-Insert
Use `std::map<int, int>` mapping `start → end`. On insert:
1. Find left neighbor (largest key ≤ new start) — merge if it overlaps
2. Find right neighbors (all keys between new start and new end) — merge all of them
3. Put merged interval

## Complete C++ Implementation

```cpp
#include <bits/stdc++.h>
using namespace std;

class GreedyIntervalIterator {
    // Key: interval start, Value: interval end
    map<int, int> mp;

public:
    /**
     * Inserts [start, end] into the sorted structure, merging overlapping intervals.
     * Time: O(k log n) where k = number of intervals merged (amortized O(log n) per insert
     *       since each interval is inserted once and removed at most once)
     */
    void addInterval(int start, int end) {
        // Step 1: Check left neighbor — does it extend into our interval?
        bool leftUsed = false;
        auto lit = mp.upper_bound(start);
        if (lit != mp.begin()) {
            --lit;
            if (lit->second >= start) {
                // Left interval overlaps: extend our start to left's start, end to max of both ends
                start = lit->first;
                end = max(end, lit->second);
                leftUsed = true;
            }
        }

        // Step 2: Absorb all right neighbors that overlap with [start, end]
        // Collect all entries that start within (start, end]
        // We need to check up to end because a right interval starting at 'end' overlaps
        auto lo = mp.upper_bound(start);
        auto hi = mp.upper_bound(end);
        for (auto jt = lo; jt != hi; ++jt) {
            end = max(end, jt->second);
        }
        mp.erase(lo, hi);

        // Step 3: Remove the left neighbor if it was merged
        if (leftUsed) {
            // Only remove left if we actually used it
            auto check = mp.upper_bound(start);
            if (check != mp.begin()) {
                --check;
                if (check->second >= start) {
                    mp.erase(check);
                }
            }
        }

        // Step 4: Insert merged interval
        mp[start] = end;
    }

    /**
     * Returns all intervals sorted by start time.
     * Time: O(n)
     */
    vector<pair<int,int>> getIntervals() {
        vector<pair<int,int>> result;
        for (auto& [k, v] : mp) {
            result.push_back({k, v});
        }
        return result;
    }

    // ── Clean implementation (production-ready) ────────────────────────────────
    /**
     * Cleaner addInterval using a rebuilt approach for clarity.
     * Same asymptotic complexity.
     */
    void addIntervalClean(int start, int end) {
        // Extend left if left neighbor overlaps
        auto lo = mp.upper_bound(start);
        if (lo != mp.begin()) {
            --lo;
            if (lo->second >= start) {
                start = lo->first;
                end = max(end, lo->second);
                mp.erase(lo);
            }
        }

        // Absorb all intervals that start within [start, end]
        while (true) {
            auto next = mp.lower_bound(start);
            if (next == mp.end() || next->first > end) break;
            end = max(end, next->second);
            mp.erase(next);
        }

        mp[start] = end;
    }
};

static string formatIntervals(const vector<pair<int,int>>& intervals) {
    string sb = "[";
    for (int i = 0; i < (int)intervals.size(); i++) {
        sb += "[";
        sb += to_string(intervals[i].first);
        sb += ",";
        sb += to_string(intervals[i].second);
        sb += "]";
        if (i < (int)intervals.size() - 1) sb += ",";
    }
    return sb + "]";
}

int main() {
    GreedyIntervalIterator gii;
    gii.addIntervalClean(1, 3);
    gii.addIntervalClean(6, 9);
    cout << "After [1,3],[6,9]: " << formatIntervals(gii.getIntervals()) << "\n";
    // Expected: [[1,3],[6,9]]

    gii.addIntervalClean(2, 5);
    cout << "After adding [2,5]: " << formatIntervals(gii.getIntervals()) << "\n";
    // Expected: [[1,5],[6,9]]

    gii.addIntervalClean(8, 10);
    cout << "After adding [8,10]: " << formatIntervals(gii.getIntervals()) << "\n";
    // Expected: [[1,5],[6,10]]

    gii.addIntervalClean(0, 20);
    cout << "After adding [0,20]: " << formatIntervals(gii.getIntervals()) << "\n";
    // Expected: [[0,20]] (all merged)

    return 0;
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

## Why std::map?

`std::map` provides:
- `prev(map.upper_bound(key))` — find largest key ≤ query in O(log n)
- `map.lower_bound(key)` — find smallest key ≥ query in O(log n)
- iterators from `lower_bound`/`upper_bound` — range view in O(log n) + O(k) to iterate
- Sorted iteration via range-based for loop in O(n)

A `std::unordered_map` would require O(n) scan to find overlapping intervals.

## Related Problems
- [Insert Interval (LC #57)](../Patterns/Interval%20Scheduling.md) — one-shot insertion into sorted array
- [Merge Intervals (LC #56)](../Patterns/Interval%20Scheduling.md) — batch merge of unsorted intervals

> **Last Updated:** 2026-06-26
