> **Topic:** [Greedy Algorithms](../README.md) · **Interview Tips — Coding**

# Greedy Algorithms — Coding Tips

## Tip 1: Sort-First Discipline

The most important habit in greedy: **sort before you greedily scan**.

| Problem Type | Sort Key | Why |
|-------------|----------|-----|
| Maximize non-overlapping intervals | End time ascending | Earliest-ending interval leaves maximum future room |
| Merge / insert intervals | Start time ascending | Process intervals left to right |
| Assign resources (cookies, tasks) | Both arrays ascending | Match smallest sufficient resource to smallest need |
| Meeting Rooms II | Start time ascending | Process meetings in chronological order |
| Minimum Arrows | End time ascending | Arrow at end of first balloon pops maximum overlapping |

**C++ idiom:**
```cpp
#include <bits/stdc++.h>
using namespace std;
// Sort by end time (interval scheduling)
sort(intervals.begin(), intervals.end(), [](const auto& a, const auto& b) { return a[1] < b[1]; });

// Sort by start time (merge intervals)
sort(intervals.begin(), intervals.end(), [](const auto& a, const auto& b) { return a[0] < b[0]; });

// Sort 2D array by first column ascending, break ties by second column descending
sort(intervals.begin(), intervals.end(), [](const auto& a, const auto& b) {
    return a[0] != b[0] ? a[0] < b[0] : b[1] < a[1];
});

// Sort two arrays together (Assign Cookies)
sort(g.begin(), g.end());
sort(s.begin(), s.end());
```

**Never use `a - b` as a comparator** — it overflows for large negative values. Always use explicit comparison lambdas.

---

## Tip 2: Exchange Argument Sketch Template

When asked "why does greedy work?", use this template:

```
1. Assume optimal solution O does NOT use our greedy choice G.
2. Identify where O first diverges from greedy.
3. Show we can swap O's choice with G (or swap two adjacent elements in O's ordering).
4. Show the swapped solution is at least as good as O (or strictly better).
5. By induction / repeated swapping, conclude greedy = optimal.
```

**Example for Non-overlapping Intervals:**
> "Assume optimal keeps interval X (ends at time t₂) over Y (ends at time t₁ < t₂). Both start after the previous interval. Swap X with Y: Y ends earlier, creating more room for future intervals. The swap cannot make things worse. By induction, always picking the earliest-ending interval is optimal."

---

## Tip 3: Two-Pass Pattern (Candy / Ratings Problems)

When a constraint depends on **both neighbors**, a single pass is insufficient. Use two passes:

```cpp
#include <bits/stdc++.h>
using namespace std;
// Generic two-pass template
vector<int> result(n, 1); // Initialize minimum

// Pass 1: Left → Right (satisfy left-neighbor constraint)
for (int i = 1; i < n; i++) {
    if (condition(i, i - 1)) {
        result[i] = result[i - 1] + 1;
    }
}

// Pass 2: Right → Left (satisfy right-neighbor constraint)
for (int i = n - 2; i >= 0; i--) {
    if (condition(i, i + 1)) {
        result[i] = max(result[i], result[i + 1] + 1);
    }
}
```

**Key:** Use `max` in Pass 2 — do NOT overwrite Pass 1 results. Each position must satisfy BOTH passes.

**Why single-pass fails:** Consider [3, 2, 1]. Left-to-right pass gives [1, 1, 1] (each rating is lower → stays at 1). But we need [3, 2, 1] because each person must have fewer candies than their left neighbor (who has a higher rating). The single pass in either direction cannot enforce both constraints simultaneously.

---

## Tip 4: lo/hi Bounds for Range-Tracking Problems (Valid Parenthesis String)

When a character can represent multiple states (like `*` = `(`, `)`, or empty), track the **range of possible valid states**:

```cpp
#include <bits/stdc++.h>
using namespace std;
int lo = 0; // Minimum open-paren count (treating '*' as ')' or '')
int hi = 0; // Maximum open-paren count (treating '*' as '(')

for (char c : s) {
    if (c == '(')      { lo++; hi++; }
    else if (c == ')') { lo--; hi--; }
    else               { lo--; hi++; } // '*' decrements min, increments max

    if (hi < 0) return false;  // Even optimistically, too many ')'
    lo = max(lo, 0);           // lo can't be negative (can't have negative open parens)
}
return lo == 0; // Minimum open count must reach 0 (all closed)
```

**Invariant:** `lo ≤ actual_open_count ≤ hi` at every step.
**Valid condition:** `hi >= 0` (not impossible) AND `lo == 0` at end (can be balanced).

---

## Tip 5: std::map for Greedy with Ordering

When greedy requires processing in sorted order AND dynamic insertion/deletion, use `std::map`:

```cpp
#include <bits/stdc++.h>
using namespace std;
map<int, int> count;

// Insert / increment
for (int card : hand) count[card]++;

// Process smallest key first (greedy order)
while (!count.empty()) {
    int first = count.begin()->first; // O(log n)
    for (int i = first; i < first + groupSize; i++) {
        if (count.count(i) == 0) return false;
        count[i]--;
        if (count[i] == 0) count.erase(i); // Clean up zeros
    }
}
```

**When to use std::map vs std::priority_queue:**
| Feature | std::map | std::priority_queue |
|---------|----------|---------------------|
| Access by value | ✓ `[key]` | ✗ |
| Access min/max | ✓ `begin()->first`/`rbegin()->first` | ✓ `top()` |
| Range queries | ✓ `lower_bound()`/`upper_bound()` | ✗ |
| Duplicates | Via value field | ✓ naturally |
| Use when | Need both lookup + ordering | Need min/max only |

---

## Tip 6: Greedy Template Checklist

Before writing greedy code, answer these questions:

- [ ] What is the greedy choice at each step?
- [ ] What order should elements be processed? (What to sort by?)
- [ ] What state do I need to track between steps?
- [ ] What is the termination / success condition?
- [ ] Can I prove the greedy choice never invalidates future optimal choices?

---

## Tip 7: Common Greedy Code Patterns

```cpp
#include <bits/stdc++.h>
using namespace std;
// Pattern A: Two-pointer greedy (Assign Cookies)
int i = 0, j = 0;
while (i < (int)a.size() && j < (int)b.size()) {
    if (b[j] >= a[i]) i++; // match found
    j++;                    // always advance the "resource" pointer
}

// Pattern B: Interval keep/remove (Non-overlapping, Min Arrows)
int prevEnd = INT_MIN; // or first interval's end
int kept = 0;
for (auto& interval : sortedIntervals) {
    if (interval[0] >= prevEnd) { // no overlap
        kept++;
        prevEnd = interval[1];
    }
}

// Pattern C: Window stretching (Partition Labels)
int start = 0, end = 0;
for (int i = 0; i < n; i++) {
    end = max(end, last[element(i)]);
    if (i == end) { recordPartition(); start = i + 1; }
}

// Pattern D: BFS-level greedy (Jump Game II)
int jumps = 0, curEnd = 0, farthest = 0;
for (int i = 0; i < n - 1; i++) {
    farthest = max(farthest, reach(i));
    if (i == curEnd) { jumps++; curEnd = farthest; }
}
```

> **Last Updated:** 2026-06-26
