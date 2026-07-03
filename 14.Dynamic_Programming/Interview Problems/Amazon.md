> **Topic:** [Dynamic Programming](../README.md) · **Section:** Interview Problems

# Amazon — DP Interview Deep Dives

This file walks the four DP problems Amazon interviewers return to most, with the full reasoning an interviewer wants to hear: state definition, recurrence derivation, base cases, complete Rust, complexity, and the follow-ups. Each one maps to a leadership-principle behavior at the end.

See also: [OA-Qns → Amazon](../OA-Qns/Amazon.md) · [Interview Problems → Google](./Google.md) · [Interview Problems → Microsoft](./Microsoft.md) · [Topic README](../README.md)

---

## 1. Coin Change — min coins *and* count of ways

**Problem.** Given coins of distinct denominations and an `amount`, return the fewest coins to make `amount`, or `-1` if impossible. Each coin is *unbounded* (infinite supply).

### State and recurrence

- **State:** `dp[a]` = minimum coins to make amount `a`.
- **Recurrence:** `dp[a] = min over coins c of dp[a - c] + 1`.
- **Base case:** `dp[0] = 0` (zero coins make amount 0).
- **Sentinel:** initialize the rest to `amount + 1` (an impossible-to-reach "infinity" that never overflows when you add 1).

### Why `amount + 1` and not `i32::MAX`

If you seed with `i32::MAX` and then compute `dp[a-c] + 1`, the `+1` overflows and corrupts the `min`. `amount + 1` is strictly greater than any real answer (you can never need more than `amount` coins of value ≥ 1), so it acts as a safe infinity.

```rust
fn coin_change(coins: &[i32], amount: i32) -> i32 {
    let amount = amount as usize;
    let mut dp = vec![amount + 1; amount + 1];  // safe "infinity"
    dp[0] = 0;
    for a in 1..=amount {
        for &c in coins {
            let c = c as usize;
            if c <= a {
                dp[a] = dp[a].min(dp[a - c] + 1);
            }
        }
    }
    if dp[amount] > amount { -1 } else { dp[amount] as i32 }
}
```

Time `O(amount · coins)`, space `O(amount)`.

### The count-of-ways twin (Coin Change II) — loop order is everything

To **count** combinations (not permutations), put coins on the **outer** loop and amount on the **inner** loop. This guarantees each coin is "introduced" once, so `{1,2}` and `{2,1}` are not counted twice.

```rust
fn change(amount: i32, coins: &[i32]) -> i32 {
    let amount = amount as usize;
    let mut dp = vec![0i32; amount + 1];
    dp[0] = 1;                      // one way to make 0: take nothing
    for &c in coins {               // coin OUTER → combinations, not permutations
        let c = c as usize;
        for a in c..=amount {
            dp[a] += dp[a - c];
        }
    }
    dp[amount]
}
```

If you swap the loops (amount outer, coin inner) you count *permutations* — a classic Amazon "spot the bug" follow-up. See the loop-order discussion in [Knapsack (Subset Sum)](../Patterns/Knapsack%20(Subset%20Sum).md).

### Edge cases
- `amount == 0` → `0` (min) or `1` (count).
- No coin can reach the amount → `-1` (min).
- A coin larger than the amount is simply skipped by `c <= a`.

### Common follow-ups
- "Count ways" → above.
- "Coins are limited (k of each)" → bounded knapsack, add a count dimension.
- "Largest coin used / lexicographically smallest set" → reconstruct by walking back through choices.

---

## 2. Word Break — tabulation and memoized recursion

**Problem.** Given a string `s` and a dictionary `wordDict`, return `true` if `s` can be segmented into a space-separated sequence of dictionary words.

### State and recurrence

- **State:** `dp[i]` = can the prefix `s[0..i)` be fully segmented?
- **Recurrence:** `dp[i] = OR over j in [0, i) of (dp[j] && dict.contains(s[j..i)))`.
- **Base case:** `dp[0] = true` (the empty prefix is trivially segmentable).
- **Answer:** `dp[n]`.

Put the dictionary in a `HashSet` for O(1) lookups; substring lookups dominate cost.

```rust
use std::collections::HashSet;

fn word_break(s: String, word_dict: Vec<String>) -> bool {
    let dict: HashSet<String> = word_dict.into_iter().collect();
    let n = s.len();
    let mut dp = vec![false; n + 1];
    dp[0] = true;
    for i in 1..=n {
        for j in 0..i {
            if dp[j] && dict.contains(&s[j..i]) {
                dp[i] = true;
                break;          // one valid split is enough
            }
        }
    }
    dp[n]
}
```

Time `O(n² · L)` where `L` is average word length (substring + hash), space `O(n)`.

### The memoized-recursion view (what interviewers often ask you to start with)

```rust
use std::collections::HashSet;

fn solve(s: &str, start: usize, dict: &HashSet<String>, memo: &mut Vec<i32>) -> bool {
    if start == s.len() { return true; }
    if memo[start] != -1 { return memo[start] == 1; }
    for end in (start + 1)..=s.len() {
        if dict.contains(&s[start..end]) && solve(s, end, dict, memo) {
            memo[start] = 1;
            return true;
        }
    }
    memo[start] = 0;
    false
}

fn word_break(s: String, word_dict: Vec<String>) -> bool {
    let dict: HashSet<String> = word_dict.into_iter().collect();
    let mut memo = vec![-1i32; s.len() + 1];  // -1 = not computed, 0 = false, 1 = true
    solve(&s, 0, &dict, &mut memo)
}
```

Memoizing on the **suffix start index** turns the exponential branching into `O(n²·L)`. The `Vec<i32>` with `-1` as sentinel lets `-1` mean "not computed", avoiding the "0 is a valid answer" collision discussed in [Common Mistakes](../Interview%20Tips/Common%20Mistakes.md).

### Edge cases
- Empty `s` → `true`.
- Word longer than remaining suffix → the substring just won't be in the dict.
- Repeated dictionary words are deduped by the set.

### Common follow-ups
- **Word Break II** — return all sentences; backtrack with memo keyed on suffix to avoid recomputation.
- "Return one valid segmentation" — store the chosen split point.

---

## 3. House Robber I → II — the circular trick

**Problem (I).** Maximize the sum of chosen `nums` such that no two adjacent elements are chosen.

### State and recurrence

- **State:** `dp[i]` = max loot considering houses `0..i`.
- **Recurrence:** `dp[i] = max(dp[i-1] /* skip i */, dp[i-2] + nums[i] /* rob i */)`.
- **Base:** `dp[0] = nums[0]`, `dp[1] = max(nums[0], nums[1])`.

Reduce to two rolling variables:

```rust
fn rob(nums: &[i32]) -> i32 {
    let (mut prev2, mut prev1) = (0, 0);       // dp[i-2], dp[i-1]
    for &x in nums {
        let cur = prev1.max(prev2 + x);
        prev2 = prev1;
        prev1 = cur;
    }
    prev1
}
```

Time `O(n)`, space `O(1)`.

### House Robber II — houses in a circle

Now house `0` and house `n-1` are adjacent. The trick: you can't rob both ends, so the optimal solution either **excludes the first house** or **excludes the last house**. Run the linear robber on each window and take the max.

```rust
fn rob_linear(nums: &[i32], lo: usize, hi: usize) -> i32 {
    let (mut prev2, mut prev1) = (0, 0);
    for i in lo..=hi {
        let cur = prev1.max(prev2 + nums[i]);
        prev2 = prev1;
        prev1 = cur;
    }
    prev1
}

fn rob(nums: &[i32]) -> i32 {
    let n = nums.len();
    if n == 1 { return nums[0]; }
    rob_linear(nums, 0, n - 2).max(   // exclude last
        rob_linear(nums, 1, n - 1))   // exclude first
}
```

The `n == 1` guard matters: with one house, both windows would be empty and return 0, losing the answer.

### Common follow-ups
- **House Robber III** — houses form a *binary tree*; switch to tree DP returning `{rob, skip}` pairs per node.
- "Print which houses were robbed" — reconstruct from the decisions.

---

## 4. Longest Palindromic Substring — expand-around-center vs DP

**Problem.** Return the longest contiguous palindromic substring of `s`.

### Approach A — Expand around center (preferred in interviews: O(1) space)

Every palindrome has a center: either a single character (odd length) or a gap between two characters (even length). There are `2n - 1` centers; expand each outward.

```rust
fn expand(s: &[u8], l: i32, r: i32) -> i32 {
    let (mut l, mut r) = (l, r);
    while l >= 0 && r < s.len() as i32 && s[l as usize] == s[r as usize] {
        l -= 1;
        r += 1;
    }
    r - l - 1                           // length once the loop overshoots
}

fn longest_palindrome(s: String) -> String {
    if s.is_empty() { return String::new(); }
    let bytes = s.as_bytes();
    let (mut start, mut end) = (0i32, 0i32);
    for i in 0..s.len() as i32 {
        let len1 = expand(bytes, i, i);       // odd-length center
        let len2 = expand(bytes, i, i + 1);   // even-length center
        let len = len1.max(len2);
        if len > end - start + 1 {
            start = i - (len - 1) / 2;
            end = i + len / 2;
        }
    }
    s[start as usize..=end as usize].to_string()
}
```

Time `O(n²)`, space `O(1)`.

### Approach B — Interval DP (when the interviewer wants the table)

- **State:** `dp[i][j]` = is `s[i..j]` a palindrome?
- **Recurrence:** `dp[i][j] = (s[i]==s[j]) && (j - i < 2 || dp[i+1][j-1])`.
- **Order:** because `dp[i][j]` depends on `dp[i+1][j-1]`, iterate by **increasing substring length** (or `i` descending, `j` ascending).

```rust
fn longest_palindrome(s: String) -> String {
    let n = s.len();
    let bytes = s.as_bytes();
    let mut dp = vec![vec![false; n]; n];
    let (mut start, mut max_len) = (0usize, 1usize);
    for i in (0..n).rev() {                    // i descending
        for j in i..n {                        // j ascending
            if bytes[i] == bytes[j] && (j - i < 2 || dp[i + 1][j - 1]) {
                dp[i][j] = true;
                if j - i + 1 > max_len {
                    max_len = j - i + 1;
                    start = i;
                }
            }
        }
    }
    s[start..start + max_len].to_string()
}
```

Time `O(n²)`, space `O(n²)`. The iteration order is the classic trap — see [Common Mistakes](../Interview%20Tips/Common%20Mistakes.md).

### Edge cases
- Empty / single char → returns itself.
- All identical chars → the whole string.

### Common follow-ups
- "Count palindromic substrings" → accumulate count instead of tracking the longest.
- "Longest palindromic *subsequence*" → different problem; LCS of `s` and its reverse — see [Longest Common Subsequence](../Patterns/Longest%20Common%20Subsequence.md).

---

## Amazon Leadership Principle alignment

| Problem | LP demonstrated | How to narrate it |
|---------|-----------------|-------------------|
| Coin Change | **Insist on the Highest Standards** | Proactively handle `-1` and overflow; explain the `amount+1` sentinel before being asked. |
| Word Break | **Dive Deep** | Start from brute-force recursion, expose overlapping suffixes, then memoize on the start index. |
| House Robber I→II | **Invent and Simplify** | Reduce the table to two variables; reframe the circular case as two linear passes. |
| Longest Palindromic Substring | **Bias for Action** | Ship the O(1)-space expand-around-center quickly, then offer the DP table if pushed. |

> **Last Updated:** 2026-06-26
