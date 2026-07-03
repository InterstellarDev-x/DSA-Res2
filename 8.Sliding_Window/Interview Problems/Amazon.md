# Amazon — Sliding Window Interview Problems

> **Topic:** [Sliding Window](../README.md) · **Company:** Amazon

---

## Problem 1: Minimum Window Substring — Deep Dive

**LC 76** · Hard · O(|s| + |t|) time, O(1) space

### Full Solution with `have/need` Counter

```cpp
#include <bits/stdc++.h>
using namespace std;

string minWindow(string s, string t) {
    int need[128] = {};
    for (char c : t) need[c]++;

    int left = 0, have = 0, required = t.length();
    int minLen = INT_MAX, minLeft = 0;

    for (int right = 0; right < (int)s.length(); right++) {
        char c = s[right];
        if (need[c] > 0) have++;   // satisfying a genuine need
        need[c]--;

        while (have == required) {
            if (right - left + 1 < minLen) {
                minLen = right - left + 1;
                minLeft = left;
            }
            char l = s[left];
            need[l]++;
            if (need[l] > 0) have--;   // we lost a needed char
            left++;
        }
    }
    return minLen == INT_MAX ? "" : s.substr(minLeft, minLen);
}
```

**Q: Why does `need[c] > 0` determine if we increment `have`?**
A: `need` array starts with counts from `t`. As we add characters from `s`, `need[c]` decreases. If `need[c] > 0` before decrement, this character was genuinely needed — we're satisfying a requirement. If `need[c] ≤ 0`, this is an excess character (already have enough of this type) — `have` shouldn't increase.

**Q: On shrinking, why does `need[l]++ > 0` trigger `have--`?**
A: After `need[l]++`, if the count is now positive (> 0), it means we needed this char and removing it from the window means we're no longer satisfying that need. If the count is ≤ 0 after increment, we still have excess copies in the window — no need to decrement `have`.

**Q: What if `t` has duplicate characters like "AAB"?**
A: The `need` array handles this — `need['A'] = 2`, so you need two A's in the window. Each A added from `s` decrements `need['A']`; `have` increments when `need['A']` goes from positive to zero/negative, meaning we've satisfied all A requirements.

**Amazon LP Alignment:**

| LP | Connection |
|----|-----------|
| Dive Deep | `need[c] > 0` vs `need[c] <= 0` distinction |
| Invent & Simplify | Using `need` both as frequency count and as "satisfied" tracker |
| Deliver Results | O(|s|) single pass with O(1) decision per character |

---

## Problem 2: Longest Repeating Character Replacement — Deep Dive

**LC 424** · Medium · O(n) time

```cpp
#include <bits/stdc++.h>
using namespace std;

int characterReplacement(string s, int k) {
    int freq[26] = {};
    int left = 0, maxFreq = 0, maxLen = 0;

    for (int right = 0; right < (int)s.length(); right++) {
        freq[s[right] - 'A']++;
        maxFreq = max(maxFreq, freq[s[right] - 'A']);

        if ((right - left + 1) - maxFreq > k) {
            freq[s[left] - 'A']--;
            left++;
        }

        maxLen = max(maxLen, right - left + 1);
    }
    return maxLen;
}
```

**Q: Why `if` not `while` for shrinking?**
A: We're maximizing window size. Once we shrink by 1, the window is at the same size as before this iteration's expansion — it can never be smaller than `maxLen`. We don't need to keep shrinking; we just maintain size until the window can grow again.

**Q: Prove `maxFreq` doesn't need to be recomputed on shrink.**
A: We're looking for the maximum window. Any valid window must have `windowSize - maxFreq ≤ k`. When we shrink, the window decreases to the previous maximum size. For the answer to improve, we need a larger `maxFreq` in a future window. So not updating `maxFreq` downward is safe — it represents "the best we've seen so far," and only a better value can make the window grow.

---

## Problem 3: Fruits Into Baskets — Amazon Phone Screen

**LC 904** · Medium

```cpp
#include <bits/stdc++.h>
using namespace std;

int totalFruit(vector<int>& fruits) {
    unordered_map<int, int> basket;
    int left = 0, maxLen = 0;

    for (int right = 0; right < (int)fruits.size(); right++) {
        basket[fruits[right]]++;

        while (basket.size() > 2) {
            int f = fruits[left];
            basket[f]--;
            if (basket[f] == 0) basket.erase(f);
            left++;
        }

        maxLen = max(maxLen, right - left + 1);
    }
    return maxLen;
}
```

**Q: Generalize to at most k distinct types?**
A: Replace `> 2` with `> k`. Same O(n) solution.

**Q: What if fruits[] can be very large values (not 0..n)?**
A: `std::unordered_map` handles arbitrary values. If values were bounded (e.g., 0..1000), an int array of size 1001 would be more efficient.

---

## Related Files

- [Amazon OA](../OA-Qns/Amazon.md)
- [Google Interview Problems](./Google.md)
- [Variable Size Window](../Patterns/Variable%20Size%20Window.md)

> **Last Updated:** 2026-06-26
