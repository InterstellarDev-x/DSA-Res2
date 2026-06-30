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

```java
class MinStack {
    private Deque<int[]> stack = new ArrayDeque<>();  // [val, minAtThisLevel]

    public void push(int val) {
        int min = stack.isEmpty() ? val : Math.min(val, stack.peek()[1]);
        stack.push(new int[]{val, min});
    }
    public void pop()    { stack.pop(); }
    public int top()     { return stack.peek()[0]; }
    public int getMin()  { return stack.peek()[1]; }
}
```

**Q: Why not use a separate variable to track global minimum?**
A: A single variable breaks on `pop()`. If we pop the current minimum, we have no way to recover the previous minimum. The pair approach stores the running minimum at every stack level so any `pop()` naturally restores the previous minimum.

**Q: Can you achieve O(1) min with O(1) space?**
A: No — with a stack of N elements each of which could be the minimum, we fundamentally need O(N) information to track all possible minimums. The encoding trick (`val - min` difference) doesn't reduce asymptotic space, just the constant factor.

---

## Problem 2: Largest Rectangle in Histogram — Deep Dive

**LC 84** · Hard · O(n) time, O(n) space

```java
public int largestRectangleArea(int[] heights) {
    int n = heights.length;
    Deque<Integer> stack = new ArrayDeque<>();
    int maxArea = 0;
    for (int i = 0; i <= n; i++) {
        int h = (i == n) ? 0 : heights[i];
        while (!stack.isEmpty() && heights[stack.peek()] > h) {
            int height = heights[stack.pop()];
            int width = stack.isEmpty() ? i : i - stack.peek() - 1;
            maxArea = Math.max(maxArea, height * width);
        }
        stack.push(i);
    }
    return maxArea;
}
```

**Q: Walk me through the width calculation.**
A: When we pop index `j` with height `h`, the rectangle of height `h` extends:
- **Right boundary:** `i - 1` (the current index minus 1, since `heights[i] < h`)
- **Left boundary:** `stack.peek() + 1` (the new top is the last bar shorter than `h`, so our rectangle starts after it)
- **Width:** `i - stack.peek() - 1` = `(i-1) - (stack.peek()+1) + 1`
- **If stack is empty:** The rectangle extends all the way to index 0, so width = `i`.

**Q: Why the sentinel 0 at index n?**
A: After processing all bars, some may remain in the stack (never found a shorter bar to their right). The sentinel `height[n]=0` forces all remaining bars to be popped and compute their areas. Without it, we'd need a post-loop flush — sentinel is cleaner.

**Q: Extend this to Maximal Rectangle in binary matrix.**
A: Build a histogram per row: for each cell `(r, c)`, `heights[c]` = consecutive `1`s in column `c` ending at row `r` (reset to 0 if `'0'`). Apply LRH to each row's histogram. Total: O(rows × cols).

---

## Problem 3: Trapping Rain Water — Deep Dive

**LC 42** · Hard

### Two Approaches Compared

```java
// Approach 1: Two Pointers — O(n) time, O(1) space
public int trap(int[] height) {
    int left = 0, right = height.length - 1;
    int maxL = 0, maxR = 0, water = 0;
    while (left < right) {
        if (height[left] <= height[right]) {
            maxL = Math.max(maxL, height[left]);
            water += maxL - height[left];
            left++;
        } else {
            maxR = Math.max(maxR, height[right]);
            water += maxR - height[right];
            right--;
        }
    }
    return water;
}

// Approach 2: Monotonic Stack — O(n) time, O(n) space
public int trap(int[] height) {
    Deque<Integer> stack = new ArrayDeque<>();
    int water = 0;
    for (int i = 0; i < height.length; i++) {
        while (!stack.isEmpty() && height[stack.peek()] < height[i]) {
            int bottom = stack.pop();
            if (stack.isEmpty()) break;
            int width = i - stack.peek() - 1;
            int h = Math.min(height[stack.peek()], height[i]) - height[bottom];
            water += width * h;
        }
        stack.push(i);
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

```java
public boolean find132pattern(int[] nums) {
    int third = Integer.MIN_VALUE;  // candidate for nums[k] (132's "2")
    Deque<Integer> stack = new ArrayDeque<>();  // candidates for nums[j] (132's "3")
    for (int i = nums.length - 1; i >= 0; i--) {
        if (nums[i] < third) return true;
        while (!stack.isEmpty() && stack.peek() < nums[i]) {
            third = stack.pop();
        }
        stack.push(nums[i]);
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
