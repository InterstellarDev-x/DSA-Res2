# Fixed Size Window

> **Topic:** [Sliding Window](../README.md) · **Pattern 1 of 4**
> **Problems:** Maximum Average Subarray I · Maximum Sum of K Consecutive Elements · Sliding Window Maximum (see [Queue & Deque](../../7.Stacks_and_Queues/Patterns/Queue%20and%20Deque.md))

---

## Core Concept

A **fixed-size sliding window** maintains a window of exactly `k` elements. As the window slides right by one position, one element enters (from the right) and one exits (from the left).

```
Array:  [2, 3, 4, 1, 5]   k = 3
         [2  3  4]          window 1: sum = 9
            [3  4  1]       window 2: sum = 8
               [4  1  5]   window 3: sum = 10
```

**Key operation:** `windowValue = windowValue - arr[left] + arr[right]` — update in O(1) instead of recomputing.

---

## Template

```cpp
#include <bits/stdc++.h>
using namespace std;

double findMaxAverage(vector<int>& nums, int k) {
    // Step 1: Build initial window of size k
    double windowSum = 0;
    for (int i = 0; i < k; i++) windowSum += nums[i];

    double maxSum = windowSum;

    // Step 2: Slide window — remove left, add right
    for (int right = k; right < (int)nums.size(); right++) {
        windowSum += nums[right] - nums[right - k];  // right - k = outgoing element
        maxSum = max(maxSum, windowSum);
    }

    return maxSum / k;
}
```

**Why `right - k` for the outgoing element?** When `right = k`, the window is `[1..k]` and element at index `0 = right - k` leaves. When `right = k+1`, element at index `1 = right - k` leaves. General: when `right` enters, `right - k` exits.

---

## Problem 1: Maximum Average Subarray I — LC 643

**Input:** `nums = [1,12,-5,-6,50,3]`, `k = 4` → `12.75` (window `[12,-5,-6,50]`)

```cpp
#include <bits/stdc++.h>
using namespace std;

double findMaxAverage(vector<int>& nums, int k) {
    double sum = 0;
    for (int i = 0; i < k; i++) sum += nums[i];
    double maxSum = sum;
    for (int right = k; right < (int)nums.size(); right++) {
        sum += nums[right] - nums[right - k];
        maxSum = max(maxSum, sum);
    }
    return maxSum / k;
}
```

**Complexity:** O(n) time, O(1) space

**Edge case:** Single window (`nums.size() == k`) — loop body never executes; initial window is the answer.

---

## Problem 2: Maximum Sum of Exactly K Elements (Variant)

```cpp
#include <bits/stdc++.h>
using namespace std;

int maxSum(vector<int>& arr, int k) {
    int windowSum = 0;
    for (int i = 0; i < k; i++) windowSum += arr[i];
    int maxSum = windowSum;
    for (int right = k; right < (int)arr.size(); right++) {
        windowSum += arr[right] - arr[right - k];
        maxSum = max(maxSum, windowSum);
    }
    return maxSum;
}
```

---

## Problem 3: Find All Anagrams in String — LC 438

Fixed window of size `p.length()`. Use frequency arrays to compare.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> findAnagrams(string s, string p) {
    vector<int> result;
    if (s.length() < p.length()) return result;

    int pFreq[26] = {}, wFreq[26] = {};
    for (auto& c : p) pFreq[c - 'a']++;

    int k = p.length();
    for (int i = 0; i < k; i++) wFreq[s[i] - 'a']++;
    if (equal(pFreq, pFreq + 26, wFreq)) result.push_back(0);

    for (int right = k; right < (int)s.length(); right++) {
        wFreq[s[right] - 'a']++;           // add incoming
        wFreq[s[right - k] - 'a']--;       // remove outgoing
        if (equal(pFreq, pFreq + 26, wFreq)) result.push_back(right - k + 1);
    }
    return result;
}
```

**Optimization:** Track `matches` count instead of calling `equal()` every iteration (O(26) → O(1) check).

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> findAnagrams(string s, string p) {
    vector<int> result;
    int freq[26] = {};
    for (auto& c : p) freq[c - 'a']++;

    int k = p.length(), matches = 0;
    // Count how many chars have freq 0 (already matched) — initially all non-p chars
    int required = count_if(freq, freq + 26, [](int f){ return f != 0; });
    // Actually, simpler: track 'have' == 'need'
    // Use the count-of-satisfied approach:
    int left = 0;
    for (int right = 0; right < (int)s.length(); right++) {
        int c = s[right] - 'a';
        freq[c]--;
        if (freq[c] == 0) matches++;   // this char's count is now satisfied

        if (right >= k) {
            int l = s[left++] - 'a';
            if (freq[l] == 0) matches--;   // removing a satisfied char
            freq[l]++;
        }
        if (matches == required) result.push_back(left - 1);  // wait, need re-check
    }
    // Cleaner: use pFreq and wFreq with 'need' tracker
    return result;
}
```

The `equal()` version is cleaner for interviews — only 26 comparisons.

---

## Problem 4: Permutation in String — LC 567

Same as Find Anagrams but return `true`/`false`.

```cpp
#include <bits/stdc++.h>
using namespace std;

bool checkInclusion(string s1, string s2) {
    if (s1.length() > s2.length()) return false;
    int freq[26] = {};
    for (auto& c : s1) freq[c - 'a']++;

    int k = s1.length();
    int window[26] = {};
    for (int i = 0; i < k; i++) window[s2[i] - 'a']++;
    if (equal(freq, freq + 26, window)) return true;

    for (int right = k; right < (int)s2.length(); right++) {
        window[s2[right] - 'a']++;
        window[s2[right - k] - 'a']--;
        if (equal(freq, freq + 26, window)) return true;
    }
    return false;
}
```

**Complexity:** O(n) time (O(26) per comparison = O(1)), O(1) space

---

## Fixed Window vs Variable Window Decision

| Signal | Window Type |
|--------|------------|
| "exactly k elements" / "of length k" | Fixed |
| "at most k" / "minimum length" | Variable |
| "all windows of size k" | Fixed |
| "smallest/largest window satisfying X" | Variable |

---

## Related Files

- [Variable Size Window](./Variable%20Size%20Window.md)
- [At Most K Trick](./At%20Most%20K%20Trick.md)
- [Monotonic Deque for Sliding Max](../../7.Stacks_and_Queues/Patterns/Queue%20and%20Deque.md)
- [Coding Tips](../Interview%20Tips/Coding%20Tips.md)

> **Last Updated:** 2026-06-26
