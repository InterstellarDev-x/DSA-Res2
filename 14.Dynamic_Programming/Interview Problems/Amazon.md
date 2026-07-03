> **Topic:** [Dynamic Programming](../README.md) · **Section:** Interview Problems

# Amazon — DP Interview Deep Dives

This file walks the four DP problems Amazon interviewers return to most, with the full reasoning an interviewer wants to hear: state definition, recurrence derivation, base cases, complete C++, complexity, and the follow-ups. Each one maps to a leadership-principle behavior at the end.

See also: [OA-Qns → Amazon](../OA-Qns/Amazon.md) · [Interview Problems → Google](./Google.md) · [Interview Problems → Microsoft](./Microsoft.md) · [Topic README](../README.md)

---

## 1. Coin Change — min coins *and* count of ways

**Problem.** Given coins of distinct denominations and an `amount`, return the fewest coins to make `amount`, or `-1` if impossible. Each coin is *unbounded* (infinite supply).

### State and recurrence

- **State:** `dp[a]` = minimum coins to make amount `a`.
- **Recurrence:** `dp[a] = min over coins c of dp[a - c] + 1`.
- **Base case:** `dp[0] = 0` (zero coins make amount 0).
- **Sentinel:** initialize the rest to `amount + 1` (an impossible-to-reach "infinity" that never overflows when you add 1).

### Why `amount + 1` and not `INT_MAX`

If you seed with `INT_MAX` and then compute `dp[a-c] + 1`, the `+1` overflows to `INT_MIN` and corrupts the `min`. `amount + 1` is strictly greater than any real answer (you can never need more than `amount` coins of value ≥ 1), so it acts as a safe infinity.

```cpp
#include <bits/stdc++.h>
using namespace std;

int coinChange(vector<int>& coins, int amount) {
    vector<int> dp(amount + 1, amount + 1);  // safe "infinity"
    dp[0] = 0;
    for (int a = 1; a <= amount; a++) {
        for (auto& c : coins) {
            if (c <= a) {
                dp[a] = min(dp[a], dp[a - c] + 1);
            }
        }
    }
    return dp[amount] > amount ? -1 : dp[amount];
}
```

Time `O(amount · coins)`, space `O(amount)`.

### The count-of-ways twin (Coin Change II) — loop order is everything

To **count** combinations (not permutations), put coins on the **outer** loop and amount on the **inner** loop. This guarantees each coin is "introduced" once, so `{1,2}` and `{2,1}` are not counted twice.

```cpp
#include <bits/stdc++.h>
using namespace std;

int change(int amount, vector<int>& coins) {
    vector<int> dp(amount + 1, 0);
    dp[0] = 1;                      // one way to make 0: take nothing
    for (auto& c : coins) {         // coin OUTER → combinations, not permutations
        for (int a = c; a <= amount; a++) {
            dp[a] += dp[a - c];
        }
    }
    return dp[amount];
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

Put the dictionary in an `std::unordered_set` for O(1) lookups; substring lookups dominate cost.

```cpp
#include <bits/stdc++.h>
using namespace std;

bool wordBreak(string s, vector<string>& wordDict) {
    unordered_set<string> dict(wordDict.begin(), wordDict.end());
    int n = s.length();
    vector<bool> dp(n + 1, false);
    dp[0] = true;
    for (int i = 1; i <= n; i++) {
        for (int j = 0; j < i; j++) {
            if (dp[j] && dict.count(s.substr(j, i - j))) {
                dp[i] = true;
                break;          // one valid split is enough
            }
        }
    }
    return dp[n];
}
```

Time `O(n² · L)` where `L` is average word length (substring + hash), space `O(n)`.

### The memoized-recursion view (what interviewers often ask you to start with)

```cpp
#include <bits/stdc++.h>
using namespace std;

bool solve(const string& s, int start, unordered_set<string>& dict, vector<int>& memo) {
    if (start == (int)s.length()) return true;
    if (memo[start] != -1) return memo[start];
    for (int end = start + 1; end <= (int)s.length(); end++) {
        if (dict.count(s.substr(start, end - start)) && solve(s, end, dict, memo)) {
            return memo[start] = 1;
        }
    }
    return memo[start] = 0;
}

bool wordBreak(string s, vector<string>& wordDict) {
    unordered_set<string> dict(wordDict.begin(), wordDict.end());
    vector<int> memo(s.length() + 1, -1);  // -1 = not computed, 0 = false, 1 = true
    return solve(s, 0, dict, memo);
}
```

Memoizing on the **suffix start index** turns the exponential branching into `O(n²·L)`. The `vector<int>` with `-1` as sentinel lets `−1` mean "not computed", avoiding the "0 is a valid answer" collision discussed in [Common Mistakes](../Interview%20Tips/Common%20Mistakes.md).

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

```cpp
#include <bits/stdc++.h>
using namespace std;

int rob(vector<int>& nums) {
    int prev2 = 0, prev1 = 0;       // dp[i-2], dp[i-1]
    for (auto& x : nums) {
        int cur = max(prev1, prev2 + x);
        prev2 = prev1;
        prev1 = cur;
    }
    return prev1;
}
```

Time `O(n)`, space `O(1)`.

### House Robber II — houses in a circle

Now house `0` and house `n-1` are adjacent. The trick: you can't rob both ends, so the optimal solution either **excludes the first house** or **excludes the last house**. Run the linear robber on each window and take the max.

```cpp
#include <bits/stdc++.h>
using namespace std;

int robLinear(vector<int>& nums, int lo, int hi) {
    int prev2 = 0, prev1 = 0;
    for (int i = lo; i <= hi; i++) {
        int cur = max(prev1, prev2 + nums[i]);
        prev2 = prev1;
        prev1 = cur;
    }
    return prev1;
}

int rob(vector<int>& nums) {
    int n = nums.size();
    if (n == 1) return nums[0];
    return max(
        robLinear(nums, 0, n - 2),   // exclude last
        robLinear(nums, 1, n - 1));  // exclude first
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

```cpp
#include <bits/stdc++.h>
using namespace std;

int expand(const string& s, int l, int r) {
    while (l >= 0 && r < (int)s.length() && s[l] == s[r]) {
        l--;
        r++;
    }
    return r - l - 1;                      // length once the loop overshoots
}

string longestPalindrome(string s) {
    if (s.empty()) return "";
    int start = 0, end = 0;
    for (int i = 0; i < (int)s.length(); i++) {
        int len1 = expand(s, i, i);       // odd-length center
        int len2 = expand(s, i, i + 1);   // even-length center
        int len = max(len1, len2);
        if (len > end - start + 1) {
            start = i - (len - 1) / 2;
            end = i + len / 2;
        }
    }
    return s.substr(start, end - start + 1);
}
```

Time `O(n²)`, space `O(1)`.

### Approach B — Interval DP (when the interviewer wants the table)

- **State:** `dp[i][j]` = is `s[i..j]` a palindrome?
- **Recurrence:** `dp[i][j] = (s[i]==s[j]) && (j - i < 2 || dp[i+1][j-1])`.
- **Order:** because `dp[i][j]` depends on `dp[i+1][j-1]`, iterate by **increasing substring length** (or `i` descending, `j` ascending).

```cpp
#include <bits/stdc++.h>
using namespace std;

string longestPalindrome(string s) {
    int n = s.length();
    vector<vector<bool>> dp(n, vector<bool>(n, false));
    int start = 0, maxLen = 1;
    for (int i = n - 1; i >= 0; i--) {            // i descending
        for (int j = i; j < n; j++) {             // j ascending
            if (s[i] == s[j] && (j - i < 2 || dp[i + 1][j - 1])) {
                dp[i][j] = true;
                if (j - i + 1 > maxLen) {
                    maxLen = j - i + 1;
                    start = i;
                }
            }
        }
    }
    return s.substr(start, maxLen);
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
