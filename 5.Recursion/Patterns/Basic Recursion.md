# Basic Recursion

> **Topic:** [Recursion](../README.md) · **Section:** Patterns
> **Applies to:** Factorial, Fibonacci, Power, Palindrome, Flood Fill, Decode Ways

---

## The Three Laws of Recursion

1. **Base case:** A condition that returns immediately (no further recursive call).
2. **Recurrence:** Express the problem in terms of a strictly smaller subproblem.
3. **Trust:** Assume the recursive call returns the correct answer for the smaller problem.

---

## Template — Recursive Framework

```java
returnType solve(params) {
    // 1. Base case
    if (trivial condition) return base value;

    // 2. Recurse on smaller subproblem
    returnType subResult = solve(smaller params);

    // 3. Combine and return
    return combine(subResult, current contribution);
}
```

---

## Template 1 — Factorial / Fibonacci

```java
// Factorial: n! = n × (n-1)!
public int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}

// Fibonacci: fib(n) = fib(n-1) + fib(n-2)
// Naive: O(2^n) time — exponential due to recomputation
public int fib(int n) {
    if (n <= 1) return n;
    return fib(n - 1) + fib(n - 2);
}

// Memoized: O(n) time, O(n) space
public int fib(int n, int[] memo) {
    if (n <= 1) return n;
    if (memo[n] != 0) return memo[n];
    return memo[n] = fib(n - 1, memo) + fib(n - 2, memo);
}
```

---

## Template 2 — Fast Power: Pow(x, n) (LC 50)

Key insight: `x^n = x^(n/2) * x^(n/2)` — reduces to O(log n) multiplications.

```java
public double myPow(double x, int n) {
    if (n == 0) return 1;
    if (n < 0) {
        x = 1.0 / x;
        n = -n; // careful: Integer.MIN_VALUE negation overflows int
    }
    return (n % 2 == 0)
        ? myPow(x * x, n / 2)
        : x * myPow(x * x, n / 2);
}
```

**Integer.MIN_VALUE trap:** `-n` when `n = Integer.MIN_VALUE` overflows. Use `long`:

```java
public double myPow(double x, int n) {
    return power(x, (long) n);
}
private double power(double x, long n) {
    if (n == 0) return 1;
    if (n < 0) return power(1.0 / x, -n);
    if (n % 2 == 0) return power(x * x, n / 2);
    return x * power(x * x, n / 2);
}
```

---

## Template 3 — Reverse String / Array (LC 344)

```java
public void reverseString(char[] s) {
    reverse(s, 0, s.length - 1);
}
private void reverse(char[] s, int lo, int hi) {
    if (lo >= hi) return;
    char tmp = s[lo]; s[lo] = s[hi]; s[hi] = tmp;
    reverse(s, lo + 1, hi - 1);
}
```

**Note:** Tail-recursive — compiles to the same as the iterative two-pointer approach conceptually.

---

## Template 4 — Flood Fill (LC 733)

```java
public int[][] floodFill(int[][] image, int sr, int sc, int color) {
    int originalColor = image[sr][sc];
    if (originalColor == color) return image; // avoid infinite loop
    dfs(image, sr, sc, originalColor, color);
    return image;
}

private void dfs(int[][] img, int r, int c, int orig, int newColor) {
    if (r < 0 || r >= img.length || c < 0 || c >= img[0].length) return;
    if (img[r][c] != orig) return;
    img[r][c] = newColor; // mark before recursing (replaces visited set)
    dfs(img, r + 1, c, orig, newColor);
    dfs(img, r - 1, c, orig, newColor);
    dfs(img, r, c + 1, orig, newColor);
    dfs(img, r, c - 1, orig, newColor);
}
```

---

## Template 5 — Decode Ways (LC 91)

At each position, decide: decode one digit, or two digits (if valid 10–26).

```java
public int numDecodings(String s) {
    return decode(s, 0, new Integer[s.length()]);
}

private int decode(String s, int i, Integer[] memo) {
    if (i == s.length()) return 1;
    if (s.charAt(i) == '0') return 0; // leading zero, invalid
    if (memo[i] != null) return memo[i];

    int ways = decode(s, i + 1, memo); // take one digit
    if (i + 1 < s.length()) {
        int twoDigit = Integer.parseInt(s.substring(i, i + 2));
        if (twoDigit >= 10 && twoDigit <= 26) {
            ways += decode(s, i + 2, memo); // take two digits
        }
    }
    return memo[i] = ways;
}
```

---

## Recursion Tree for fib(4)

```
                fib(4)
               /      \
          fib(3)      fib(2)
          /    \       /   \
       fib(2) fib(1) fib(1) fib(0)
       /    \
    fib(1) fib(0)
```

Nodes: 9. With memoization, each fib(k) computed once → 5 calls total.

---

## Call Stack Depth

| Problem | Stack Depth | Notes |
|---------|------------|-------|
| `factorial(n)` | O(n) | Linear recursion |
| `fib(n)` naive | O(n) | Height of tree |
| `power(x, n)` | O(log n) | Halves each level |
| `floodFill` | O(rows × cols) | Worst case entire grid |
| `reverseString(n)` | O(n) | One frame per swap |

---

## Common Pitfalls

| Mistake | Fix |
|---------|-----|
| Missing base case → StackOverflow | Always identify termination condition first |
| `n = Integer.MIN_VALUE` negation overflow in Pow | Cast to `long` before negating |
| Flood fill infinite loop (same color) | Guard `if (originalColor == color) return` |
| Fib without memoization (exponential) | Use `Integer[]` memo or convert to DP |

---

## Related Patterns

- [Backtracking](./Backtracking.md) — recursion with undo step
- [Divide & Conquer](./Divide%20and%20Conquer.md) — split-and-merge recursion
- [DP](../../14.Dynamic_Programming/README.md) — memoized recursion = top-down DP

---

**Back:** [Recursion README](../README.md) | **Next:** [Backtracking](./Backtracking.md)

> **Last Updated:** 2026-06-26
