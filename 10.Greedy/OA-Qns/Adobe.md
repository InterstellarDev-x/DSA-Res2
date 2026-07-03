> **Topic:** [Greedy Algorithms](../README.md) · **OA Questions — Adobe**

# Adobe OA — Greedy Algorithms

## Frequency Table

| Problem | LC # | Frequency | Difficulty | Pattern | Last Seen |
|---------|------|-----------|------------|---------|-----------|
| Partition Labels | #763 | ⭐⭐⭐ | Medium | [Interval Scheduling](../Patterns/Interval%20Scheduling.md) | 2025 Q3 |
| Assign Cookies | #455 | ⭐⭐⭐ | Easy | [Interval Scheduling](../Patterns/Interval%20Scheduling.md) | 2025 Q2 |
| Jump Game | #55 | ⭐⭐ | Medium | [Jump Game](../Patterns/Jump%20Game.md) | 2025 Q1 |
| Lemonade Change | #860 | ⭐⭐ | Easy | [Greedy on Strings](../Patterns/Greedy%20on%20Strings.md) | 2024 Q4 |
| Non-overlapping Intervals | #435 | ⭐⭐ | Medium | [Interval Scheduling](../Patterns/Interval%20Scheduling.md) | 2024 Q3 |

## Top Focus: Partition Labels (⭐⭐⭐)

**Why Adobe loves it:** Maps to string/text processing (Adobe's core domain — PDF parsing, document segmentation, font rendering).

**Adobe-specific framing:** "You are parsing a document where each character belongs to a segment. Segment all characters such that each character type appears in exactly one segment. Return segment lengths."

**Adobe follow-up questions:**
1. "What if the string has Unicode characters?" → Use `HashMap<char, i32>` instead of `[usize; 26]`
2. "What if each segment must be at most K characters long?" → Greedy + constraint (force split at K)
3. "What if segments can overlap but you want minimum total overlap?" → Interval DP

**Code they want to see:**
```rust
fn partition_labels(s: String) -> Vec<i32> {
    let s = s.as_bytes();
    let mut last = [0usize; 26];
    for i in 0..s.len() {
        last[(s[i] - b'a') as usize] = i;
    }
    let mut result = Vec::new();
    let mut start = 0;
    let mut end = 0;
    for i in 0..s.len() {
        end = end.max(last[(s[i] - b'a') as usize]);
        if i == end {
            result.push((end - start + 1) as i32);
            start = i + 1;
        }
    }
    result
}
```

---

## Second Focus: Assign Cookies (⭐⭐⭐)

**Why Adobe loves it:** Fundamental greedy — resource assignment. Maps to Adobe's licensing/resource allocation systems.

**Adobe-specific framing:** "You have N software licenses with capacities `s[]` and M users with requirements `g[]`. Assign each user at most one license. Maximize users satisfied."

**Key insight:** Sort both arrays. Try to satisfy smallest unsatisfied user with smallest sufficient license.

**Code they want to see:**
```rust
fn find_content_children(g: &mut Vec<i32>, s: &mut Vec<i32>) -> i32 {
    g.sort();
    s.sort();
    let mut child = 0;
    let mut cookie = 0;
    while child < g.len() && cookie < s.len() {
        if s[cookie] >= g[child] {
            child += 1;
        }
        cookie += 1;
    }
    child as i32
}
```

**Why this greedy works:** If the smallest cookie can satisfy the greediest child, it definitely can. If it can't satisfy the least-greedy child, it's useless. So always pair smallest sufficient cookie with smallest child.

---

## Adobe OA Format Notes
- 2 problems, 60 minutes (Hackerrank)
- Adobe favors medium-difficulty problems over hard ones in OA
- String and array manipulation heavy — Partition Labels and Assign Cookies are staples
- Creative follow-up: "How would this work in a multi-threaded environment?" (not algorithm-focused — system design warmup)

> **Last Updated:** 2026-06-26
