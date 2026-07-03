> **Topic:** [Greedy Algorithms](../README.md) · **Pattern 2 of 4**

# Jump Game

---

## 1. Core Concept — "Reachability Window" Greedy

Jump Game problems give you an array where each element represents the maximum jump length from that position. The key insight is to think in terms of a **reachability window** rather than explicit paths.

At any point in the scan, maintain the farthest index currently reachable. If the current index is outside this window, you cannot proceed. If you need to count jumps, treat each jump as exhausting the current window and opening the next one — exactly like a BFS layer expansion.

**Two variations of the same idea:**

| Problem | What to track | When to act |
|---------|--------------|-------------|
| Jump Game I (can reach end?) | `maxReach` — farthest reachable index | If `i > maxReach`, return false |
| Jump Game II (min jumps to reach end) | `curEnd`, `farthest` | When `i == curEnd`, increment jumps and extend to `farthest` |

---

## 2. Pattern Recognition Signals

| Signal | Approach |
|--------|----------|
| "Can you reach the last index?" | Track `maxReach`; return false when `i > maxReach` |
| "Minimum jumps to reach end" | BFS-style window: jump when window exhausted |
| "Variable jump length at each position" | Greedy reachability (not DP for optimal) |
| "Jump from position `i` at most `nums[i]` steps" | Reachability window expands by `i + nums[i]` |
| "Reach end with minimum cost/moves" | Often greedy window works; verify optimal substructure |

---

## 3. Problems

---

### 1. Jump Game (#55)

**Problem:** Given `nums[]` where `nums[i]` is the maximum jump length from index `i`, determine if you can reach the last index starting from index 0.

**Key Insight:** Maintain `maxReach` — the farthest index reachable so far. At each index `i`:
- If `i > maxReach`, this index is unreachable; return false.
- Otherwise, extend `maxReach` to `max(maxReach, i + nums[i])`.

If we process all indices without encountering an unreachable one, return true.

```cpp
#include <bits/stdc++.h>
using namespace std;

class JumpGame {
public:
    bool canJump(vector<int>& nums) {
        int maxReach = 0;
        for (int i = 0; i < (int)nums.size(); i++) {
            if (i > maxReach) return false;
            maxReach = max(maxReach, i + nums[i]);
        }
        return true;
    }
};

int main() {
    JumpGame sol;
    vector<int> v1 = {2, 3, 1, 1, 4};
    vector<int> v2 = {3, 2, 1, 0, 4};
    cout << sol.canJump(v1) << "\n"; // true
    cout << sol.canJump(v2) << "\n"; // false
    return 0;
}
```

**Complexity:**
- Time: O(n) — single left-to-right pass.
- Space: O(1).

**Dry Run (nums = [3,2,1,0,4]):**
```
i=0: i=0 <= maxReach=0  maxReach=max(0,0+3)=3
i=1: i=1 <= maxReach=3  maxReach=max(3,1+2)=3
i=2: i=2 <= maxReach=3  maxReach=max(3,2+1)=3
i=3: i=3 <= maxReach=3  maxReach=max(3,3+0)=3
i=4: i=4 > maxReach=3   return false
```

**Dry Run (nums = [2,3,1,1,4]):**
```
i=0: maxReach=max(0,0+2)=2
i=1: maxReach=max(2,1+3)=4
i=2: maxReach=max(4,2+1)=4
i=3: maxReach=max(4,3+1)=4
i=4: maxReach=max(4,4+4)=8

Loop completes without returning false  return true
```

---

### 2. Jump Game II (#45)

**Problem:** Given `nums[]`, return the minimum number of jumps to reach the last index. It is guaranteed you can always reach the end.

**Key Insight:** BFS analogy — each "jump level" covers a range of indices reachable in exactly `k` jumps. Within the current level `[prevEnd+1, curEnd]`, find the farthest index reachable (`farthest`). When `i` reaches `curEnd`, we must take a new jump to get further; set `curEnd = farthest` and increment `jumps`.

We stop the loop at `nums.length - 1` (not `nums.length`) because once we can reach the last index (i.e., `farthest >= last`), we don't need another jump.

```cpp
#include <bits/stdc++.h>
using namespace std;

class JumpGameII {
public:
    int jump(vector<int>& nums) {
        int jumps = 0, curEnd = 0, farthest = 0;
        for (int i = 0; i < (int)nums.size() - 1; i++) {
            farthest = max(farthest, i + nums[i]);
            if (i == curEnd) {
                jumps++;
                curEnd = farthest;
            }
        }
        return jumps;
    }
};

int main() {
    JumpGameII sol;
    vector<int> v1 = {2, 3, 1, 1, 4};
    vector<int> v2 = {2, 3, 0, 1, 4};
    cout << sol.jump(v1) << "\n"; // 2
    cout << sol.jump(v2) << "\n"; // 2
    return 0;
}
```

**Complexity:**
- Time: O(n) — single left-to-right pass.
- Space: O(1).

---

### Detailed Dry Run — Jump Game II: [2,3,1,1,4]

```
nums = [2, 3, 1, 1, 4]
         0  1  2  3  4

jumps=0, curEnd=0, farthest=0

i=0: farthest=max(0, 0+2)=2
     i==curEnd(0==0)  jumps=1, curEnd=2
     State: jumps=1, curEnd=2, farthest=2
     (Jump 1 covers indices 0..2)

i=1: farthest=max(2, 1+3)=4
     i!=curEnd(1!=2)  no jump yet
     State: jumps=1, curEnd=2, farthest=4

i=2: farthest=max(4, 2+1)=4
     i==curEnd(2==2)  jumps=2, curEnd=4
     State: jumps=2, curEnd=4, farthest=4
     (Jump 2 extends cover to index 4)

i=3: loop ends at nums.length-1=4 so i goes 0..3
     farthest=max(4, 3+1)=4
     i!=curEnd(3!=4)  no jump

Loop ends (i < nums.length-1 = 4, so i goes 0,1,2,3)

Return jumps = 2
```

**Jump levels visualization:**
```
Index:  0   1   2   3   4
nums:  [2]  [3] [1] [1] [4]

Jump 0 (start): reachable = {0}
Jump 1:         from 0 reach up to 2  -> reachable = {1, 2}
Jump 2:         from 1 reach up to 4, from 2 reach up to 3 -> reachable = {3, 4}

Index 4 = last index, reached in 2 jumps.
```

---

## 4. Why Greedy Works — Greedy Stays Ahead

**Claim:** The greedy algorithm for Jump Game II always produces the minimum number of jumps.

**Proof (Greedy Stays Ahead):**

Define `reach(k)` = farthest index reachable in exactly `k` jumps.

- `reach_greedy(k)` = farthest index reachable after `k` greedy jumps.
- `reach_opt(k)` = farthest index reachable after `k` optimal jumps.

**Claim:** `reach_greedy(k) >= reach_opt(k)` for all `k`.

**Proof by induction:**
- Base case `k=0`: Both start at 0. `reach_greedy(0) = reach_opt(0) = 0`. ✓
- Inductive step: Assume `reach_greedy(k) >= reach_opt(k)`. The greedy algorithm's (k+1)-th jump chooses the maximum reachable index from any index in `[0, reach_greedy(k)]`. The optimal algorithm's (k+1)-th jump chooses from `[0, reach_opt(k)]`. Since `reach_greedy(k) >= reach_opt(k)`, greedy has a superset of positions to jump from, so `reach_greedy(k+1) >= reach_opt(k+1)`. ✓

Since greedy reaches the end in `m` jumps and greedy stays at least as far as optimal at every step, `m_greedy <= m_opt`. QED.

**BFS Analogy:**

Think of the problem as BFS on an implicit graph where each node `i` has edges to `[i+1, i+nums[i]]`. BFS finds the shortest path (minimum jumps). The greedy algorithm simulates BFS layer by layer:
- Each "BFS layer" corresponds to all indices reachable in `k` jumps.
- `curEnd` marks the boundary of the current layer.
- `farthest` marks the boundary of the next layer.
- Incrementing `jumps` at `i == curEnd` is the BFS layer transition.

This avoids the O(n) overhead of maintaining an explicit queue, making it O(1) space.

---

## 5. Comparison Table

| Aspect | Jump Game I (#55) | Jump Game II (#45) |
|--------|------------------|--------------------|
| Goal | Can reach last index? | Minimum jumps to last index |
| Key variable | `maxReach` | `curEnd`, `farthest`, `jumps` |
| Action at each step | Update `maxReach` | Update `farthest`; when `i==curEnd`, jump |
| Loop bound | `i < nums.length` | `i < nums.length - 1` |
| Return value | `true`/`false` | Integer count |
| Time | O(n) | O(n) |
| Space | O(1) | O(1) |

---

> **Last Updated:** 2026-06-26
