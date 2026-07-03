# Amazon — Stacks & Queues Interview Problems

> **Topic:** [Stacks & Queues](../README.md) · **Company:** Amazon
> [← Amazon OA](../OA-Qns/Amazon.md)

---

## Problem 1: Min Stack — Deep Dive

**LC 155** · Medium · O(1) all operations

### The Interview Flow

Amazon interviewers typically start here then escalate:
1. Min Stack → Max Stack → Min-Max Stack
2. Followed by: "What if concurrent threads are pushing/popping?"

### Solution with Key Insight Explanation

```cpp
#include <bits/stdc++.h>
using namespace std;

class MinStack {
    stack<pair<int,int>> st; // {val, minAtThisLevel}

public:
    void push(int val) {
        int mn = st.empty() ? val : min(val, st.top().second);
        st.push({val, mn});
    }
    void pop()    { st.pop(); }
    int top()     { return st.top().first; }
    int getMin()  { return st.top().second; }
};
```

**Q: Why not use a separate variable to track global minimum?**
A: A single variable breaks on `pop()`. If we pop the current minimum, we have no way to recover the previous minimum. The pair approach stores the running minimum at every stack level so any `pop()` naturally restores the previous minimum.

**Q: Can you achieve O(1) min with O(1) space?**
A: No — with a stack of N elements each of which could be the minimum, we fundamentally need O(N) information to track all possible minimums. The encoding trick (`val - min` difference) doesn't reduce asymptotic space, just the constant factor.

---

## Problem 2: Largest Rectangle in Histogram — Deep Dive

**LC 84** · Hard · O(n) time, O(n) space

```cpp
#include <bits/stdc++.h>
using namespace std;

int largestRectangleArea(vector<int>& heights) {
    int n = heights.size();
    stack<int> st;
    int maxArea = 0;
    for (int i = 0; i <= n; i++) {
        int h = (i == n) ? 0 : heights[i];
        while (!st.empty() && heights[st.top()] > h) {
            int height = heights[st.top()]; st.pop();
            int width = st.empty() ? i : i - st.top() - 1;
            maxArea = max(maxArea, height * width);
        }
        st.push(i);
    }
    return maxArea;
}
```

**Q: Walk me through the width calculation.**
A: When we pop index `j` with height `h`, the rectangle of height `h` extends:
- **Right boundary:** `i - 1` (the current index minus 1, since `heights[i] < h`)
- **Left boundary:** `stack.top() + 1` (the new top is the last bar shorter than `h`, so our rectangle starts after it)
- **Width:** `i - stack.top() - 1` = `(i-1) - (stack.top()+1) + 1`
- **If stack is empty:** The rectangle extends all the way to index 0, so width = `i`.

**Q: Why the sentinel 0 at index n?**
A: After processing all bars, some may remain in the stack (never found a shorter bar to their right). The sentinel `height[n]=0` forces all remaining bars to be popped and compute their areas. Without it, we'd need a post-loop flush — sentinel is cleaner.

**Q: Extend this to Maximal Rectangle in binary matrix.**
A: Build a histogram per row: for each cell `(r, c)`, `heights[c]` = consecutive `1`s in column `c` ending at row `r` (reset to 0 if `'0'`). Apply LRH to each row's histogram. Total: O(rows × cols).

---

## Problem 3: Trapping Rain Water — Deep Dive

**LC 42** · Hard

### Two Approaches Compared

```cpp
#include <bits/stdc++.h>
using namespace std;

// Approach 1: Two Pointers — O(n) time, O(1) space
int trap(vector<int>& height) {
    int left = 0, right = (int)height.size() - 1;
    int maxL = 0, maxR = 0, water = 0;
    while (left < right) {
        if (height[left] <= height[right]) {
            maxL = max(maxL, height[left]);
            water += maxL - height[left];
            left++;
        } else {
            maxR = max(maxR, height[right]);
            water += maxR - height[right];
            right--;
        }
    }
    return water;
}

// Approach 2: Monotonic Stack — O(n) time, O(n) space
int trap(vector<int>& height) {
    stack<int> st;
    int water = 0;
    for (int i = 0; i < (int)height.size(); i++) {
        while (!st.empty() && height[st.top()] < height[i]) {
            int bottom = st.top(); st.pop();
            if (st.empty()) break;
            int width = i - st.top() - 1;
            int h = min(height[st.top()], height[i]) - height[bottom];
            water += width * h;
        }
        st.push(i);
    }
    return water;
}
```

**Q: How does the two-pointer approach know which side to process?**
A: Water trapped at position `x` = `min(maxLeft[x], maxRight[x]) - height[x]`. If `height[left] ≤ height[right]`, then `maxRight` from the left's perspective is at least `height[right]`, so `min(maxLeft, maxRight) = maxLeft`. We can safely compute water for `left` without knowing the exact `maxRight`.

**Q: When would you prefer the monotonic stack approach?**
A: When computing in a streaming/online fashion where the full array isn't available up front. The stack naturally processes left-to-right. Also useful when the interviewer asks for variations like "which indices trap water" — easier to extract from stack approach.

---

## Problem 4: 132 Pattern — Amazon Phone Screen

**LC 456** · Medium · O(n) time

```cpp
#include <bits/stdc++.h>
using namespace std;

bool find132pattern(vector<int>& nums) {
    int third = INT_MIN;  // candidate for nums[k] (132's "2")
    stack<int> st;  // candidates for nums[j] (132's "3")
    for (int i = (int)nums.size() - 1; i >= 0; i--) {
        if (nums[i] < third) return true;
        while (!st.empty() && st.top() < nums[i]) {
            third = st.top(); st.pop();
        }
        st.push(nums[i]);
    }
    return false;
}
```

**Q: Explain why we scan right-to-left.**
A: We're building a "j, k" pair as we go. When we pop from the stack (because current element > stack top), the popped value becomes the best `k` candidate (it was pushed as a potential `j` but is now below a new, taller `j`). Then we check if any future `i` (to the left) is smaller than this `k`. Scanning left-to-right would require us to look ahead for `j` and `k` simultaneously.

---

## Amazon LP Alignment

| LP Principle | Connection |
|-------------|-----------|
| Dive Deep | Width formula derivation for histogram; sentinel element explanation |
| Invent and Simplify | Min Stack pair encoding; Score of Parentheses O(1) space |
| Deliver Results | "My O(n) solution is guaranteed single pass — no edge case can make it slower" |
| Are Right, A Lot | Justify two-pointer approach for Trapping Rain Water; when to prefer stack |

---

## Related Files

- [Amazon OA](../OA-Qns/Amazon.md)
- [Google Interview Problems](./Google.md)
- [Histogram & Rectangle Pattern](../Patterns/Histogram%20and%20Rectangle.md)
- [Monotonic Stack Pattern](../Patterns/Monotonic%20Stack.md)

> **Last Updated:** 2026-06-26
