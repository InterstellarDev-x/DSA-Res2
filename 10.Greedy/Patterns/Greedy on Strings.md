> **Topic:** [Greedy Algorithms](../README.md) · **Pattern 3 of 4**

# Greedy on Strings

---

## 1. Core Concept

"Greedy on Strings" problems share a common structure: scan the input left-to-right, maintain a small set of invariant state (counters, running balances, or per-element tracking), and make the greedy locally optimal decision at each character/element without reconsidering earlier choices.

The "string" framing is loose — these problems may work on integer arrays (Candy, ratings), character sequences (Lemonade bills, parentheses), or triplet arrays. The unifying theme is that a single pass (or at most two passes) with a carefully maintained invariant is sufficient.

**Key sub-patterns:**

| Sub-pattern | Example | Invariant Maintained |
|-------------|---------|----------------------|
| Greedy counting / cash register | Lemonade Change | Counts of each denomination held |
| Balance range tracking | Valid Parenthesis String | `[lo, hi]` range of possible open-paren counts |
| Component selection by constraint | Merge Triplets | Running max of valid triplets |
| Two-pass directional greedy | Candy | Left-to-right pass, then right-to-left pass |

---

## 2. Pattern Recognition Signals

| Signal | Approach |
|--------|----------|
| "Make change with limited denominations" | Greedy: prefer larger denominations first to preserve small ones |
| "'*' can be '(', ')' or empty" | Track lo/hi range of open-paren count |
| "Merge/combine elements to reach a target" | Filter elements that would violate target, take max of rest |
| "Each child gets more than neighbors with lower rating" | Two directional passes; take max of both directions |
| "Greedy scan with local rule" | Single-pass with O(1) state; look for invariant that holds throughout |

---

## 3. Problems

---

### 1. Lemonade Change (#860)

**Problem:** Customers pay for a $5 lemonade with bills of $5, $10, or $20. You start with no change. Determine if you can give correct change to every customer.

**Key Insight:**
- $5 bill: no change needed, keep the $5.
- $10 bill: give back one $5. If none, impossible.
- $20 bill: give back $15. **Prefer giving $10 + $5** over $5+$5+$5 because $10 bills are only useful for $20 change, while $5 bills are needed for both $10 and $20 change. Using three $5 bills wastes more flexible change.

**Why give $10 before $5 for $20 change:**
$10 bills can only be used as partial change for $20 bills — they have exactly one use. $5 bills are more versatile: they can satisfy $10 change OR contribute to $20 change. Therefore, whenever we need to make $15 change for a $20 bill, we should burn the less-versatile $10 first, preserving $5 bills for future $10 transactions.

This is the greedy-choice property: using a $10+$5 now cannot be worse than using $5+$5+$5, because the $10 bill saved is useless unless a future $20 arrives (and then we still have the $5 available).

```cpp
#include <bits/stdc++.h>
using namespace std;

bool lemonadeChange(vector<int>& bills) {
    int five = 0, ten = 0;
    for (int bill : bills) {
        if (bill == 5) {
            five++;
        } else if (bill == 10) {
            if (five == 0) return false;
            five--;
            ten++;
        } else {
            // bill == 20: need to give $15 change
            if (ten > 0 && five > 0) {
                ten--;
                five--;
            } else if (five >= 3) {
                five -= 3;
            } else {
                return false;
            }
        }
    }
    return true;
}

int main() {
    vector<int> bills1 = {5, 5, 5, 10, 20};
    cout << lemonadeChange(bills1) << "\n"; // true
    vector<int> bills2 = {5, 5, 10, 10, 20};
    cout << lemonadeChange(bills2) << "\n"; // false
    vector<int> bills3 = {5, 5, 10, 20, 5, 5, 5, 5, 5, 5, 5, 5, 5, 10, 5, 5, 20, 5, 20, 5};
    cout << lemonadeChange(bills3) << "\n"; // true
    return 0;
}
```

**Complexity:**
- Time: O(n) — single pass.
- Space: O(1).

**Dry Run (bills = [5,5,5,10,20]):**
```
five=0, ten=0

bill=5:  five=1, ten=0
bill=5:  five=2, ten=0
bill=5:  five=3, ten=0
bill=10: five=2, ten=1  (give one $5)
bill=20: ten>0 && five>0  ten=0, five=1  (give $10+$5)

End: return true  (all served)
```

---

### 2. Valid Parenthesis String (#678)

**Problem:** Given a string `s` containing `(`, `)`, and `*`, where `*` can represent `(`, `)`, or an empty string, determine if `s` is a valid parenthesis string.

**Key Insight — lo/hi range tracking:**

Instead of tracking one balance (count of open parens), track a **range** `[lo, hi]` of possible open-paren balances:
- `lo` = minimum possible open-paren count (assume each `*` acts as `)` or empty).
- `hi` = maximum possible open-paren count (assume each `*` acts as `(`).

Rules at each character:
- `(`: both `lo` and `hi` increase by 1.
- `)`: both `lo` and `hi` decrease by 1.
- `*`: `lo` decreases by 1 (use `*` as `)`), `hi` increases by 1 (use `*` as `(`).

After each step:
- If `hi < 0`: even with all `*` as `(`, we have more `)` than `(` — impossible, return false.
- Clamp `lo = max(lo, 0)`: balance can never be negative (we can always discard `*` as empty).

At the end, if `lo == 0`, there is a valid assignment of `*` that gives zero unclosed parens.

**Why lo/hi works:** `lo` represents the most pessimistic assignment of wildcards (every `*` tries to close or disappear), and `hi` represents the most optimistic (every `*` opens). Any integer in `[lo, hi]` is achievable. We need 0 to be achievable at the end.

```cpp
#include <bits/stdc++.h>
using namespace std;

bool checkValidString(string s) {
    int lo = 0, hi = 0;
    for (char c : s) {
        if (c == '(') {
            lo++;
            hi++;
        } else if (c == ')') {
            lo--;
            hi--;
        } else {
            // '*' acts as ')' for lo (most pessimistic), '(' for hi (most optimistic)
            lo--;
            hi++;
        }
        if (hi < 0) return false;  // too many ')' even with all '*' as '('
        lo = max(lo, 0);           // balance can't go below 0
    }
    return lo == 0;
}

int main() {
    cout << checkValidString("()") << "\n";    // true
    cout << checkValidString("(*)") << "\n";   // true
    cout << checkValidString("(*))") << "\n";  // true
    cout << checkValidString("((*)") << "\n";  // true
    cout << checkValidString("((*)") << "\n";  // true
    return 0;
}
```

**Complexity:**
- Time: O(n) — single pass.
- Space: O(1).

**Dry Run (s = "(*)"):**
```
lo=0, hi=0

c='(': lo=1, hi=1
c='*': lo=0, hi=2  (lo-- to 0, hi++ to 2)
       hi=2 >= 0, lo=max(0,0)=0
c=')': lo=-1, hi=1
       hi=1 >= 0, lo=max(-1,0)=0

End: lo=0  return true
```

**Dry Run (s = ")*"):**
```
lo=0, hi=0

c=')': lo=-1, hi=-1
       hi=-1 < 0  return false immediately
```

---

### 3. Merge Triplets to Form Target (#1899)

**Problem:** Given `triplets[]` where each triplet is `[a,b,c]`, and a `target = [x,y,z]`, you can choose any subset of triplets and merge them (component-wise max). Determine if you can form exactly `target`.

**Key Insight:** A triplet is **disqualifying** if any of its components exceeds the corresponding target component — merging such a triplet would make the result exceed the target somewhere, making it impossible to match exactly. Filter those out.

Among the remaining (valid) triplets, take the component-wise maximum. If that maximum equals `target`, return true; otherwise false.

```cpp
#include <bits/stdc++.h>
using namespace std;

bool mergeTriplets(vector<vector<int>>& triplets, vector<int>& target) {
    vector<int> result(3, 0);
    for (auto& t : triplets) {
        if (t[0] <= target[0] && t[1] <= target[1] && t[2] <= target[2]) {
            result[0] = max(result[0], t[0]);
            result[1] = max(result[1], t[1]);
            result[2] = max(result[2], t[2]);
        }
    }
    return result == target;
}

int main() {
    // target = [5,5,5], triplets include [5,5,5] directly
    vector<vector<int>> t1 = {{2,5,3},{1,8,4},{1,7,5}};
    vector<int> target1 = {5, 5, 5};
    cout << mergeTriplets(t1, target1) << "\n"; // false (no valid triplet has first=5 without exceeding second)

    vector<vector<int>> t2 = {{3,4,5},{4,5,6}};
    vector<int> target2 = {3, 2, 5};
    cout << mergeTriplets(t2, target2) << "\n"; // false

    vector<vector<int>> t3 = {{2,5,3},{2,3,4},{1,2,5}};
    vector<int> target3 = {2, 5, 5};
    cout << mergeTriplets(t3, target3) << "\n"; // true ([2,5,3] and [1,2,5] merge to [2,5,5])
    return 0;
}
```

**Complexity:**
- Time: O(n) — single pass over triplets.
- Space: O(1).

**Dry Run (triplets=[[2,5,3],[2,3,4],[1,2,5]], target=[2,5,5]):**
```
result = [0,0,0]

t=[2,5,3]: 2<=2, 5<=5, 3<=5  valid  result=max([0,0,0],[2,5,3])=[2,5,3]
t=[2,3,4]: 2<=2, 3<=5, 4<=5  valid  result=max([2,5,3],[2,3,4])=[2,5,4]
t=[1,2,5]: 1<=2, 2<=5, 5<=5  valid  result=max([2,5,4],[1,2,5])=[2,5,5]

result == target  →  [2,5,5] == [2,5,5]  return true
```

---

### 4. Candy (#135)

**Problem:** `n` children stand in a line, each assigned a rating. Give each child at least 1 candy. A child with a higher rating than a neighbor must receive more candies than that neighbor. Minimize total candies.

**Key Insight — Why single-pass fails:**

Consider ratings `[1, 2, 3]`. Left-to-right: `[1,2,3]` — correct. Now consider `[3,2,1]`. Left-to-right: `[1,1,1]` — wrong (3>2>1 means left children need more than right). A single left-to-right pass only handles "going up" relationships. A single right-to-left pass only handles "going down." We need both.

The two-pass strategy:
1. **Left-to-right pass:** If `ratings[i] > ratings[i-1]`, give `candies[i] = candies[i-1] + 1`. This satisfies the "higher rating than left neighbor" constraint.
2. **Right-to-left pass:** If `ratings[i] > ratings[i+1]`, ensure `candies[i] = max(candies[i], candies[i+1] + 1)`. This satisfies the "higher rating than right neighbor" constraint without breaking the left-to-right constraints (we take the max).

The max operation is crucial: taking the max preserves both directional constraints simultaneously.

```cpp
#include <bits/stdc++.h>
using namespace std;

int candy(vector<int>& ratings) {
    int n = ratings.size();
    vector<int> candies(n, 1);

    // Left to right: higher than left neighbor gets one more
    for (int i = 1; i < n; i++) {
        if (ratings[i] > ratings[i - 1]) candies[i] = candies[i - 1] + 1;
    }

    // Right to left: higher than right neighbor gets at least one more than right
    for (int i = n - 2; i >= 0; i--) {
        if (ratings[i] > ratings[i + 1]) {
            candies[i] = max(candies[i], candies[i + 1] + 1);
        }
    }

    int total = 0;
    for (int c : candies) total += c;
    return total;
}

int main() {
    vector<int> r1 = {1, 0, 2};
    cout << candy(r1) << "\n"; // 5
    vector<int> r2 = {1, 2, 2};
    cout << candy(r2) << "\n"; // 4
    vector<int> r3 = {3, 2, 1};
    cout << candy(r3) << "\n"; // 6
    return 0;
}
```

**Complexity:**
- Time: O(n) — two passes.
- Space: O(n) for the `candies[]` array.

---

#### Detailed Dry Run — Candy: [1, 0, 2]

```
ratings = [1, 0, 2]
n = 3

After initialization:
candies = [1, 1, 1]

Left-to-right pass:
  i=1: ratings[1]=0 < ratings[0]=1  no change
  i=2: ratings[2]=2 > ratings[1]=0  candies[2] = candies[1]+1 = 2
candies = [1, 1, 2]

Right-to-left pass:
  i=1: ratings[1]=0 < ratings[2]=2  no change (0 is not greater)
  i=0: ratings[0]=1 > ratings[1]=0  candies[0] = max(candies[0], candies[1]+1) = max(1, 2) = 2
candies = [2, 1, 2]

Total = 2 + 1 + 2 = 5
```

**Verification:**
- Child 0: rating=1 > rating[1]=0, has 2 > 1 candies ✓
- Child 1: rating=0 < both neighbors, has minimum 1 candy ✓
- Child 2: rating=2 > rating[1]=0, has 2 > 1 candies ✓

**Another dry run — Candy: [3, 2, 1] (descending):**
```
ratings = [3, 2, 1]
After init: candies = [1, 1, 1]

Left-to-right:
  i=1: ratings[1]=2 < ratings[0]=3  no change
  i=2: ratings[2]=1 < ratings[1]=2  no change
candies = [1, 1, 1]  (left-pass can't help descending sequence)

Right-to-left:
  i=1: ratings[1]=2 > ratings[2]=1  candies[1]=max(1,1+1)=2
  i=0: ratings[0]=3 > ratings[1]=2  candies[0]=max(1,2+1)=3
candies = [3, 2, 1]

Total = 3 + 2 + 1 = 6
```

---

## 4. Why Single-Pass Fails for Candy

Consider ratings `[1, 3, 2, 2, 1]`:
- After left-to-right: `[1, 2, 1, 1, 1]`
  - Child 1 (rating 3) got 2 because 3 > 1.
  - Child 2 (rating 2) got 1 because 2 < 3.
- But the right-to-left pass fixes child 2 and 3:
  - After right-to-left: Child 3 (rating 2) > Child 4 (rating 1), so `candies[3] = max(1, 1+1) = 2`.
  - Child 2 (rating 2) equals child 3, no change.
  - Final: `[1, 2, 1, 2, 1]`, total = 7.

If we tried to handle both directions in one pass, we would need to look-ahead (what is the length of the upcoming descent slope?) which requires preprocessing. The two-pass approach is simpler, correct, and O(n).

---

## 5. Complexity Summary

| Problem | Time | Space | Key State |
|---------|------|-------|-----------|
| Lemonade Change (#860) | O(n) | O(1) | `five`, `ten` counters |
| Valid Parenthesis String (#678) | O(n) | O(1) | `lo`, `hi` balance range |
| Merge Triplets (#1899) | O(n) | O(1) | `result[3]` running max |
| Candy (#135) | O(n) | O(n) | `candies[]` array |

---

> **Last Updated:** 2026-06-26
