# Histogram & Rectangle

> **Topic:** [Stacks & Queues](../README.md) · **Pattern 4 of 5**
> **Problems:** Largest Rectangle in Histogram · Maximal Rectangle

---

## Core Concept

The histogram pattern extends the monotonic stack to compute **maximum area** problems. For each bar, we need:
- The **left boundary**: index of the nearest bar shorter than current (use previous smaller element)
- The **right boundary**: index of the nearest bar shorter than current (use next smaller element)

Area for bar `i` = `height[i] × (right[i] - left[i] - 1)`

The monotonic stack computes both boundaries in one O(n) pass.

---

## Problem 1: Largest Rectangle in Histogram — LC 84

**Input:** `heights = [2,1,5,6,2,3]` → `10` (bars 5,6 with height 5, width 2)

### Approach 1: Monotonic Stack — O(n) time, O(n) space

**Invariant:** Stack maintains indices of bars in **increasing height order**. When we find a shorter bar, all taller bars on the stack are "blocked" — their right boundary has been found.

```java
public int largestRectangleArea(int[] heights) {
    int n = heights.length;
    Deque<Integer> stack = new ArrayDeque<>();
    int maxArea = 0;

    for (int i = 0; i <= n; i++) {
        // Sentinel: treat index n as height 0 to flush stack at end
        int currHeight = (i == n) ? 0 : heights[i];

        while (!stack.isEmpty() && heights[stack.peek()] > currHeight) {
            int h = heights[stack.pop()];
            // Left boundary = new stack top + 1 (or 0 if stack empty)
            int leftBound = stack.isEmpty() ? 0 : stack.peek() + 1;
            int rightBound = i - 1;
            int width = rightBound - leftBound + 1;
            maxArea = Math.max(maxArea, h * width);
        }
        stack.push(i);
    }
    return maxArea;
}
```

**Trace for [2,1,5,6,2,3]:**
```
i=0: stack=[0(h=2)]
i=1: h[0]=2 > h[1]=1 → pop 0: h=2, leftBound=0 (empty), rightBound=0, area=2×1=2. stack=[1(h=1)]
i=2: stack=[1,2(h=5)]
i=3: stack=[1,2,3(h=6)]
i=4: h[3]=6>h[4]=2 → pop 3: h=6, left=3, right=3, area=6×1=6. h[2]=5>2 → pop 2: h=5, left=2, right=3, area=5×2=10. stack=[1,4(h=2)]
i=5: stack=[1,4,5(h=3)]
i=6 (sentinel=0): pop 5: h=3, left=5, right=5, area=3. pop 4: h=2, left=2, right=5, area=2×4=8. pop 1: h=1, left=0, right=5, area=1×6=6
maxArea = 10
```

**Why `i <= n` and sentinel height 0?** After the last bar, any remaining bars in the stack haven't had their right boundary resolved. The sentinel forces all remaining bars to compute their area.

### Approach 2: Precompute Left/Right Smaller — O(n) time, O(n) space

```java
public int largestRectangleArea(int[] heights) {
    int n = heights.length;
    int[] left = new int[n];   // index of first bar shorter to the left
    int[] right = new int[n];  // index of first bar shorter to the right
    Deque<Integer> stack = new ArrayDeque<>();

    // Compute left smaller
    for (int i = 0; i < n; i++) {
        while (!stack.isEmpty() && heights[stack.peek()] >= heights[i]) stack.pop();
        left[i] = stack.isEmpty() ? -1 : stack.peek();
        stack.push(i);
    }
    stack.clear();

    // Compute right smaller
    for (int i = n - 1; i >= 0; i--) {
        while (!stack.isEmpty() && heights[stack.peek()] >= heights[i]) stack.pop();
        right[i] = stack.isEmpty() ? n : stack.peek();
        stack.push(i);
    }

    int maxArea = 0;
    for (int i = 0; i < n; i++) {
        maxArea = Math.max(maxArea, heights[i] * (right[i] - left[i] - 1));
    }
    return maxArea;
}
```

**Complexity:** O(n) time, O(n) space. Uses 2 separate passes vs 1 pass with sentinel.

---

## Problem 2: Maximal Rectangle — LC 85

**Input:** Binary matrix of `'0'` and `'1'`. Find the largest rectangle of all `'1'`s.

**Key insight:** Build a histogram for each row where `heights[j]` = number of consecutive `'1'`s in column `j` ending at the current row. Then apply Largest Rectangle in Histogram to each row's heights.

```java
public int maximalRectangle(char[][] matrix) {
    if (matrix.length == 0) return 0;
    int cols = matrix[0].length;
    int[] heights = new int[cols];
    int maxArea = 0;

    for (char[] row : matrix) {
        // Update histogram heights
        for (int j = 0; j < cols; j++) {
            heights[j] = (row[j] == '1') ? heights[j] + 1 : 0;
        }
        // Apply LRH on current row's histogram
        maxArea = Math.max(maxArea, largestRectangleArea(heights));
    }
    return maxArea;
}

private int largestRectangleArea(int[] heights) {
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

**Complexity:** O(rows × cols) time, O(cols) space

**Trace for:**
```
matrix = [
  ['1','0','1','0','0'],
  ['1','0','1','1','1'],
  ['1','1','1','1','1'],
  ['1','0','0','1','0']
]
Row 0: heights = [1,0,1,0,0] → max area = 1
Row 1: heights = [2,0,2,1,1] → max area = 3 (cols 2,3,4 with height 1)
Row 2: heights = [3,1,3,2,2] → max area = 6 (cols 0..4 with height 2? Actually [3,1,3,2,2]: try 3×1=3, 1×5=5, 3×1=3, 2×2=4, 2×2=4 ... for [3,1,3,2,2]: bar at idx2 h=3: left=0(h=1 blocks?), careful trace gives 6)
Row 3: heights = [4,0,0,3,0] → max area = 4
Overall: 6
```

---

## Why This Pattern Is Hard

1. **Width calculation is subtle:** `width = i - stack.peek() - 1` where `stack.peek()` is the left boundary (exclusive), not inclusive.
2. **Sentinel element needed:** Without it, bars remaining in stack after the loop never compute their area.
3. **Equal heights:** Using `>=` vs `>` in the stack condition affects correctness. Use `>` (pop only when strictly taller) — otherwise equal bars could get wrong widths.
4. **2D extension (Maximal Rectangle):** The jump from 1D histogram to 2D matrix requires recognizing the "row-by-row histogram" decomposition.

---

## Width Formula Reference

| Scenario | Formula |
|----------|---------|
| Stack is empty after pop | `width = i` (extends from 0 to i-1) |
| Stack has element after pop | `width = i - stack.peek() - 1` (extends from `stack.peek()+1` to `i-1`) |

For the precompute approach: `width = right[i] - left[i] - 1` where `left[i]` and `right[i]` are the exclusive boundaries.

---

## Classic Interview Follow-ups

**Q: What if all bars have equal height?**
A: The maximum rectangle spans the entire array. Width = n, height = h, area = n×h. The algorithm handles this correctly.

**Q: Can Maximal Rectangle be solved without histogram reduction?**
A: The dynamic programming approach exists (`dp[i][j]` = height of the column of consecutive 1s ending at row i, col j) — it's essentially the same algorithm applied row by row, just differently framed. The histogram reduction is cleaner.

**Q: Time complexity of Maximal Rectangle?**
A: O(rows × cols) — each cell is processed once for height update, and the LRH subroutine is O(cols) per row.

---

## Related Files

- [Monotonic Stack](./Monotonic%20Stack.md)
- [Stack Basics](./Stack%20Basics.md)
- [Interview Problems: Amazon](../Interview%20Problems/Amazon.md)
- [Interview Problems: Google](../Interview%20Problems/Google.md)
- [Complexity Analysis](../Interview%20Tips/Complexity%20Analysis.md)

> **Last Updated:** 2026-06-26
