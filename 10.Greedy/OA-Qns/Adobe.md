> **Topic:** [Greedy Algorithms](../README.md) · **OA Questions — Adobe**

# Adobe OA — Greedy Algorithms

## Frequency Table

| Problem | LC # | Frequency | Difficulty | Pattern | Last Seen |
|---------|------|-----------|------------|---------|-----------|
| Partition Labels | #763 | ⭐⭐⭐ | Medium | [Interval Scheduling](../Patterns/Interval%20Scheduling.md) | 2025 Q3 |
| Assign Cookies | #455 | ⭐⭐⭐ | Easy | [Interval Scheduling](../Patterns/Interval%20Scheduling.md) | 2025 Q2 |
| Jump Game | #55 | ⭐⭐ | Medium | [Jump Game](../Patterns/Jump%20Game.md) | 2025 Q1 |
| Lemonade Change | #860 | ⭐⭐ | Easy | [Greedy on Strings](../Patterns/Greedy%20on%20Strings.md) | 2024 Q4 |
| Non-overlapping Intervals | #435 | ⭐⭐ | Medium | [Interval Scheduling](../Patterns/Interval%20Scheduling.md) | 2024 Q3 |

## Top Focus: Partition Labels (⭐⭐⭐)

**Why Adobe loves it:** Maps to string/text processing (Adobe's core domain — PDF parsing, document segmentation, font rendering).

**Adobe-specific framing:** "You are parsing a document where each character belongs to a segment. Segment all characters such that each character type appears in exactly one segment. Return segment lengths."

**Adobe follow-up questions:**
1. "What if the string has Unicode characters?" → Use `std::unordered_map<char, int>` instead of `int[26]`
2. "What if each segment must be at most K characters long?" → Greedy + constraint (force split at K)
3. "What if segments can overlap but you want minimum total overlap?" → Interval DP

**Code they want to see:**
```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> partitionLabels(string s) {
    int last[26] = {};
    for (int i = 0; i < (int)s.size(); i++) last[s[i] - 'a'] = i;
    vector<int> result;
    int start = 0, end = 0;
    for (int i = 0; i < (int)s.size(); i++) {
        end = max(end, last[s[i] - 'a']);
        if (i == end) {
            result.push_back(end - start + 1);
            start = i + 1;
        }
    }
    return result;
}
```

---

## Second Focus: Assign Cookies (⭐⭐⭐)

**Why Adobe loves it:** Fundamental greedy — resource assignment. Maps to Adobe's licensing/resource allocation systems.

**Adobe-specific framing:** "You have N software licenses with capacities `s[]` and M users with requirements `g[]`. Assign each user at most one license. Maximize users satisfied."

**Key insight:** Sort both arrays. Try to satisfy smallest unsatisfied user with smallest sufficient license.

**Code they want to see:**
```cpp
#include <bits/stdc++.h>
using namespace std;

int findContentChildren(vector<int>& g, vector<int>& s) {
    sort(g.begin(), g.end());
    sort(s.begin(), s.end());
    int child = 0, cookie = 0;
    while (child < (int)g.size() && cookie < (int)s.size()) {
        if (s[cookie] >= g[child]) child++;
        cookie++;
    }
    return child;
}
```

**Why this greedy works:** If the smallest cookie can satisfy the greediest child, it definitely can. If it can't satisfy the least-greedy child, it's useless. So always pair smallest sufficient cookie with smallest child.

---

## Adobe OA Format Notes
- 2 problems, 60 minutes (Hackerrank)
- Adobe favors medium-difficulty problems over hard ones in OA
- String and array manipulation heavy — Partition Labels and Assign Cookies are staples
- Creative follow-up: "How would this work in a multi-threaded environment?" (not algorithm-focused — system design warmup)

> **Last Updated:** 2026-06-26
