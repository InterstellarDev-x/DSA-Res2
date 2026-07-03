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

```rust
fn solve(params: Type) -> ReturnType {
    // 1. Base case
    if trivial_condition { return base_value; }

    // 2. Recurse on smaller subproblem
    let sub_result = solve(smaller_params);

    // 3. Combine and return
    combine(sub_result, current_contribution)
}
```

---

## Template 1 — Factorial / Fibonacci

```rust
// Factorial: n! = n × (n-1)!
fn factorial(n: i32) -> i32 {
    if n <= 1 { return 1; }
    n * factorial(n - 1)
}

// Fibonacci: fib(n) = fib(n-1) + fib(n-2)
// Naive: O(2^n) time — exponential due to recomputation
fn fib(n: i32) -> i32 {
    if n <= 1 { return n; }
    fib(n - 1) + fib(n - 2)
}

// Memoized: O(n) time, O(n) space
fn fib_memo(n: usize, memo: &mut Vec<i32>) -> i32 {
    if n <= 1 { return n as i32; }
    if memo[n] != 0 { return memo[n]; }
    memo[n] = fib_memo(n - 1, memo) + fib_memo(n - 2, memo);
    memo[n]
}
```

---

## Template 2 — Fast Power: Pow(x, n) (LC 50)

Key insight: `x^n = x^(n/2) * x^(n/2)` — reduces to O(log n) multiplications.

```rust
fn my_pow(x: f64, n: i32) -> f64 {
    if n == 0 { return 1.0; }
    if n < 0 {
        return my_pow(1.0 / x, -(n as i64) as i32); // careful: i32::MIN negation overflows i32
    }
    if n % 2 == 0 {
        my_pow(x * x, n / 2)
    } else {
        x * my_pow(x * x, n / 2)
    }
}
```

**i32::MIN trap:** `-n` when `n = i32::MIN` overflows. Use `i64`:

```rust
fn my_pow(x: f64, n: i32) -> f64 {
    power(x, n as i64)
}

fn power(x: f64, n: i64) -> f64 {
    if n == 0 { return 1.0; }
    if n < 0 { return power(1.0 / x, -n); }
    if n % 2 == 0 { return power(x * x, n / 2); }
    x * power(x * x, n / 2)
}
```

---

## Template 3 — Reverse String / Array (LC 344)

```rust
fn reverse_helper(s: &mut Vec<char>, lo: usize, hi: usize) {
    if lo >= hi { return; }
    s.swap(lo, hi);
    reverse_helper(s, lo + 1, hi - 1);
}

fn reverse_string(s: &mut Vec<char>) {
    let n = s.len();
    if n > 0 {
        reverse_helper(s, 0, n - 1);
    }
}
```

**Note:** Tail-recursive — compiles to the same as the iterative two-pointer approach conceptually.

---

## Template 4 — Flood Fill (LC 733)

```rust
fn dfs(img: &mut Vec<Vec<i32>>, r: i32, c: i32, orig: i32, new_color: i32) {
    if r < 0 || r >= img.len() as i32 || c < 0 || c >= img[0].len() as i32 { return; }
    let (ru, cu) = (r as usize, c as usize);
    if img[ru][cu] != orig { return; }
    img[ru][cu] = new_color; // mark before recursing (replaces visited set)
    dfs(img, r + 1, c, orig, new_color);
    dfs(img, r - 1, c, orig, new_color);
    dfs(img, r, c + 1, orig, new_color);
    dfs(img, r, c - 1, orig, new_color);
}

fn flood_fill(image: Vec<Vec<i32>>, sr: i32, sc: i32, color: i32) -> Vec<Vec<i32>> {
    let mut image = image;
    let original_color = image[sr as usize][sc as usize];
    if original_color == color { return image; } // avoid infinite loop
    dfs(&mut image, sr, sc, original_color, color);
    image
}
```

---

## Template 5 — Decode Ways (LC 91)

At each position, decide: decode one digit, or two digits (if valid 10–26).

```rust
fn decode(s: &[u8], i: usize, memo: &mut Vec<i32>) -> i32 {
    if i == s.len() { return 1; }
    if s[i] == b'0' { return 0; } // leading zero, invalid
    if memo[i] != -1 { return memo[i]; }

    let mut ways = decode(s, i + 1, memo); // take one digit
    if i + 1 < s.len() {
        let two_digit = (s[i] - b'0') as i32 * 10 + (s[i + 1] - b'0') as i32;
        if two_digit >= 10 && two_digit <= 26 {
            ways += decode(s, i + 2, memo); // take two digits
        }
    }
    memo[i] = ways;
    ways
}

fn num_decodings(s: &str) -> i32 {
    let bytes = s.as_bytes();
    let mut memo = vec![-1; s.len()];
    decode(bytes, 0, &mut memo)
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
| `n = i32::MIN` negation overflow in Pow | Cast to `i64` before negating |
| Flood fill infinite loop (same color) | Guard `if (originalColor == color) return` |
| Fib without memoization (exponential) | Use `Vec<i32>` memo initialized to -1, or convert to DP |

---

## Related Patterns

- [Backtracking](./Backtracking.md) — recursion with undo step
- [Divide & Conquer](./Divide%20and%20Conquer.md) — split-and-merge recursion
- [DP](../../14.Dynamic_Programming/README.md) — memoized recursion = top-down DP

---

**Back:** [Recursion README](../README.md) | **Next:** [Backtracking](./Backtracking.md)

> **Last Updated:** 2026-06-26
