# Monotonic Stack

> **Topic:** [Stacks & Queues](../README.md) · **Pattern 3 of 5**
> **Problems:** NGE I & II · Daily Temperatures · Online Stock Span · 132 Pattern · Asteroid Collision · Remove K Digits · Trapping Rain Water · Sum of Subarray Minimums · Remove Duplicate Letters · Maximum Width Ramp · Number of Visible People

---

## Core Concept

A **monotonic stack** maintains elements in a strictly increasing or decreasing order at all times. When a new element violates the invariant, pop until invariant holds. The moment of popping is when answers are computed.

**Invariant types:**
- **Decreasing stack:** each new element pops all smaller elements → popped element's "Next Greater Element" is the new element
- **Increasing stack:** each new element pops all larger elements → popped element's "Next Smaller Element" is the new element

---

## Template 1: Next Greater Element (NGE)

```cpp
#include <bits/stdc++.h>
using namespace std;

// For each element, find the first element to its RIGHT that is greater
vector<int> nextGreaterElements(vector<int>& nums) {
    int n = nums.size();
    vector<int> result(n, -1);          // default: no NGE found
    stack<int> stk;  // stores INDICES

    for (int i = 0; i < n; i++) {
        // Pop all elements smaller than nums[i] — nums[i] is their NGE
        while (!stk.empty() && nums[stk.top()] < nums[i]) {
            result[stk.top()] = nums[i];
            stk.pop();
        }
        stk.push(i);
    }
    return result;
    // Remaining indices in stack have no NGE → stay -1
}
```

**Why store indices?** We need to write the answer back to `result[index]`. Values alone are insufficient.

---

## Problem 1: Next Greater Element I — LC 496

Two arrays: find NGE of each element of `nums1` within `nums2`.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> nextGreaterElement(vector<int>& nums1, vector<int>& nums2) {
    unordered_map<int, int> nge;
    stack<int> stk;

    for (int val : nums2) {
        while (!stk.empty() && stk.top() < val) {
            nge[stk.top()] = val;   // store value→NGE mapping
            stk.pop();
        }
        stk.push(val);
    }

    vector<int> result(nums1.size());
    for (int i = 0; i < (int)nums1.size(); i++) {
        result[i] = (nge.count(nums1[i]) ? nge[nums1[i]] : -1);
    }
    return result;
}
```

**Complexity:** O(n + m) time, O(n) space

---

## Problem 2: Daily Temperatures — LC 739

Find, for each day, how many days until a warmer temperature.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> dailyTemperatures(vector<int>& temperatures) {
    int n = temperatures.size();
    vector<int> result(n);
    stack<int> stk;  // indices

    for (int i = 0; i < n; i++) {
        while (!stk.empty() && temperatures[stk.top()] < temperatures[i]) {
            int prevIdx = stk.top();
            stk.pop();
            result[prevIdx] = i - prevIdx;   // days to wait = index difference
        }
        stk.push(i);
    }
    return result;
}
```

**Key insight:** `i - prevIdx` gives the distance (days to wait). This only works because we store indices, not temperatures.

---

## Problem 3: Next Greater Element II (Circular) — LC 503

Array is circular — wrap around once.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> nextGreaterElements(vector<int>& nums) {
    int n = nums.size();
    vector<int> result(n, -1);
    stack<int> stk;

    // Loop twice to simulate circular array
    for (int i = 0; i < 2 * n; i++) {
        int idx = i % n;
        while (!stk.empty() && nums[stk.top()] < nums[idx]) {
            result[stk.top()] = nums[idx];
            stk.pop();
        }
        if (i < n) stk.push(idx);   // only push indices from first pass
    }
    return result;
}
```

**Why `if (i < n)` for push?** During the second pass, we only need to resolve remaining stack elements — we don't push new indices again (they're already in result or will be resolved).

---

## Problem 4: Online Stock Span — LC 901

Design: `next(price)` returns the number of consecutive days (including today) for which today's price ≥ the price on those days.

```cpp
#include <bits/stdc++.h>
using namespace std;

class StockSpanner {
    // Stack stores (price, span) pairs
    stack<pair<int,int>> stk;

public:
    int next(int price) {
        int span = 1;
        // Absorb all previous days where price was ≤ current price
        while (!stk.empty() && stk.top().first <= price) {
            span += stk.top().second;
            stk.pop();
        }
        stk.push({price, span});
        return span;
    }
};
```

**Key trick:** Store cumulative span. When absorbing a previous day, we don't need to re-examine what it absorbed — its stored span already accounts for those days. This achieves O(1) amortized.

**Trace:** prices = [100, 80, 60, 70, 60, 75, 85]
```
100: stack=[(100,1)], span=1
80:  stack=[(100,1),(80,1)], span=1
60:  stack=[(100,1),(80,1),(60,1)], span=1
70:  pop (60,1)→span=1+1=2. stack=[(100,1),(80,1),(70,2)], span=2
60:  stack=[(100,1),(80,1),(70,2),(60,1)], span=1
75:  pop (60,1)→3, pop (70,2)→5. stack=[(100,1),(80,1),(75,5)], span=5
85:  pop (75,5)→6, pop (80,1)→7. stack=[(100,1),(85,7)], span=7
```

---

## Problem 5: 132 Pattern — LC 456

Find i < j < k such that `nums[i] < nums[k] < nums[j]`.

**Approach:** Traverse from right to left with a decreasing monotonic stack.
- `third` = the best candidate for `nums[k]` (largest value seen below a "mountain top")
- Stack maintains potential `nums[j]` values (decreasing)

```cpp
#include <bits/stdc++.h>
using namespace std;

bool find132pattern(vector<int>& nums) {
    int n = nums.size();
    int third = INT_MIN;       // best candidate for nums[k]
    stack<int> stk;  // candidates for nums[j]

    for (int i = n - 1; i >= 0; i--) {
        if (nums[i] < third) return true;    // nums[i] is nums[1] (smallest)
        while (!stk.empty() && stk.top() < nums[i]) {
            third = stk.top();   // this is nums[k] — it was below nums[j]=nums[i]
            stk.pop();
        }
        stk.push(nums[i]);
    }
    return false;
}
```

**Why right-to-left?** We want to find a smaller element to the left of a "valley above the minimum." Going right-to-left, whenever we see a larger element than the stack top, the popped element becomes the best `k` candidate.

---

## Problem 6: Asteroid Collision — LC 735

Asteroids: positive = moving right, negative = moving left. Collision: right-moving hits left-moving. Larger absolute value survives; equal sizes both explode.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> asteroidCollision(vector<int>& asteroids) {
    stack<int> stk;
    for (int a : asteroids) {
        bool exploded = false;
        while (!stk.empty() && a < 0 && stk.top() > 0) {
            if (stk.top() < -a) {
                stk.pop();   // right-moving asteroid explodes
                continue;
            } else if (stk.top() == -a) {
                stk.pop();   // both explode
            }
            // stk.top() > -a: new asteroid explodes
            exploded = true;
            break;
        }
        if (!exploded) stk.push(a);
    }

    // Convert stack to vector (stack is in reverse order)
    vector<int> result(stk.size());
    for (int i = result.size() - 1; i >= 0; i--) {
        result[i] = stk.top();
        stk.pop();
    }
    return result;
}
```

**Collision only when:** current asteroid is negative (`a < 0`) AND stack top is positive (`stk.top() > 0`). Same direction = no collision.

---

## Problem 7: Remove K Digits — LC 402

Remove `k` digits from `num` string to get the smallest possible number.

**Greedy with monotonic stack:** maintain an increasing stack. Whenever we see a digit smaller than the top, pop (that digit is a larger digit we should remove).

```cpp
#include <bits/stdc++.h>
using namespace std;

string removeKdigits(string num, int k) {
    string stk;
    for (char c : num) {
        while (k > 0 && !stk.empty() && stk.back() > c) {
            stk.pop_back();
            k--;
        }
        stk.push_back(c);
    }
    // If k > 0, remove from the end (digits are already in increasing order)
    while (k-- > 0) stk.pop_back();

    // Build result, skipping leading zeros
    string result;
    bool leadingZero = true;
    // stk is already in insertion order
    for (char c : stk) {
        if (leadingZero && c == '0') continue;
        leadingZero = false;
        result += c;
    }
    return result.empty() ? "0" : result;
}
```

**Complexity:** O(n) time, O(n) space

---

## Problem 8: Trapping Rain Water — LC 42

**Input:** `[0,1,0,2,1,0,1,3,2,1,2,1]` → `6`

### Monotonic Stack Approach — O(n) time, O(n) space

```cpp
#include <bits/stdc++.h>
using namespace std;

int trap(vector<int>& height) {
    stack<int> stk;  // decreasing stack of indices
    int water = 0;

    for (int i = 0; i < (int)height.size(); i++) {
        while (!stk.empty() && height[stk.top()] < height[i]) {
            int bottom = stk.top();
            stk.pop();
            if (stk.empty()) break;
            int left = stk.top();
            int width = i - left - 1;
            int boundedHeight = min(height[left], height[i]) - height[bottom];
            water += width * boundedHeight;
        }
        stk.push(i);
    }
    return water;
}
```

**Layer-by-layer concept:** The stack maintains a valley shape. When a right wall (taller element) is found, we compute the water trapped between the left wall (new stack top after pop) and right wall at the height of the just-popped element.

### Two-Pointer Approach — O(n) time, O(1) space

```cpp
#include <bits/stdc++.h>
using namespace std;

int trap(vector<int>& height) {
    int left = 0, right = height.size() - 1;
    int maxLeft = 0, maxRight = 0, water = 0;
    while (left < right) {
        if (height[left] <= height[right]) {
            if (height[left] >= maxLeft) maxLeft = height[left];
            else water += maxLeft - height[left];
            left++;
        } else {
            if (height[right] >= maxRight) maxRight = height[right];
            else water += maxRight - height[right];
            right--;
        }
    }
    return water;
}
```

**Which to use in interview?** Two-pointer is O(1) space and simpler to code. Monotonic stack approach generalizes more easily to follow-ups.

---

## Problem 9: Sum of Subarray Minimums — LC 907

For each subarray, find its minimum and sum all minimums. Answer modulo 10^9 + 7.

**Key insight:** For each element `A[i]`, count how many subarrays have `A[i]` as their minimum. Use monotonic increasing stack to find:
- `left[i]` = number of subarrays ending at `i` where `A[i]` is minimum (distance to previous smaller element)
- `right[i]` = number of subarrays starting at `i` where `A[i]` is minimum (distance to next smaller or equal element)

```cpp
#include <bits/stdc++.h>
using namespace std;

int sumSubarrayMins(vector<int>& arr) {
    int MOD = 1'000'000'007;
    int n = arr.size();
    vector<int> left(n);    // left[i] = # subarrays where arr[i] is min, ending at i
    vector<int> right(n);   // right[i] = # subarrays where arr[i] is min, starting at i
    stack<int> stk;

    // Compute left[i]: distance to previous STRICTLY smaller element
    for (int i = 0; i < n; i++) {
        while (!stk.empty() && arr[stk.top()] >= arr[i]) stk.pop();
        left[i] = stk.empty() ? i + 1 : i - stk.top();
        stk.push(i);
    }
    while (!stk.empty()) stk.pop();
    // Compute right[i]: distance to next SMALLER OR EQUAL element
    // Use <= for right to avoid double-counting equal elements
    for (int i = n - 1; i >= 0; i--) {
        while (!stk.empty() && arr[stk.top()] > arr[i]) stk.pop();
        right[i] = stk.empty() ? n - i : stk.top() - i;
        stk.push(i);
    }

    long ans = 0;
    for (int i = 0; i < n; i++) {
        ans = (ans + (long) arr[i] * left[i] * right[i]) % MOD;
    }
    return (int) ans;
}
```

**Why strict `>=` for left but `>` for right?** Prevents double-counting subarrays with equal minimums. The convention: equal elements are counted on the left side.

---

## Problem 10: Remove Duplicate Letters — LC 316

Return the smallest lexicographic subsequence containing all unique characters exactly once.

```cpp
#include <bits/stdc++.h>
using namespace std;

string removeDuplicateLetters(string s) {
    int count[26] = {};
    bool inStack[26] = {};
    for (char c : s) count[c - 'a']++;

    string stk;
    for (char c : s) {
        count[c - 'a']--;
        if (inStack[c - 'a']) continue;    // already in result
        // Pop if current char is smaller AND the top char appears later
        while (!stk.empty() && stk.back() > c && count[stk.back() - 'a'] > 0) {
            inStack[stk.back() - 'a'] = false;
            stk.pop_back();
        }
        stk.push_back(c);
        inStack[c - 'a'] = true;
    }
    return stk;
}
```

**Three conditions to pop:** (1) stack not empty, (2) top > current (can improve lexicographic order), (3) top's count > 0 (top will appear again later, so safe to remove now).

---

## Monotonic Stack — Pattern Summary

| Problem Type | Stack Order | Pop When | Answer At |
|-------------|-------------|----------|-----------|
| Next Greater Element | Decreasing | new > top | pop time |
| Next Smaller Element | Increasing | new < top | pop time |
| Stock Span (prev consecutive ≤) | Decreasing | new > top | pop time (accumulate span) |
| 132 Pattern | Decreasing | new > top | maintains `third` |
| Trapping Rain Water | Decreasing | new > top | compute layer on pop |
| Remove K Digits | Increasing | new < top AND k>0 | build from stack |
| Sum of Subarray Mins | Increasing | new < top | left/right distances |
| Remove Duplicate Letters | Stack (lexicographic) | top > new AND will recur | build from stack |

---

## Related Files

- [Stack Basics](./Stack%20Basics.md)
- [Histogram & Rectangle](./Histogram%20and%20Rectangle.md)
- [Queue & Deque](./Queue%20and%20Deque.md)
- [Coding Tips](../Interview%20Tips/Coding%20Tips.md)
- [Common Mistakes](../Interview%20Tips/Common%20Mistakes.md)

> **Last Updated:** 2026-06-26
