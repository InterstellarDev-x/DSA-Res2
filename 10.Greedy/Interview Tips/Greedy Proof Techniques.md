> **Topic:** [Greedy Algorithms](../README.md) · **Interview Tips — Proofs**

# Greedy Proof Techniques

## Overview

Three main proof techniques for greedy algorithms:

| Technique | When to Use | Difficulty | Example Problem |
|-----------|-------------|------------|-----------------|
| Exchange Argument | Greedy makes a locally optimal ordering choice | Medium | Non-overlapping Intervals, Assign Cookies |
| Greedy Stays Ahead | Greedy maintains an invariant ≥ any other algorithm | Medium | Jump Game II |
| Matroid / Exchange Property | Independent set structure with hereditary property | Hard | Activity Selection (theoretical) |

---

## Technique 1: Exchange Argument

### Template
1. Let `G = [g₁, g₂, ..., gₙ]` be the greedy solution.
2. Let `O = [o₁, o₂, ..., oₙ]` be any optimal solution.
3. Find the first position `k` where `G` and `O` differ: `gₖ ≠ oₖ`.
4. Show we can "swap" `oₖ` with `gₖ` in `O` to get `O'` that is **at least as good as `O`**.
5. Repeat until `O` becomes `G` — greedy is optimal.

### Example: Activity Selection (Non-overlapping Intervals)

**Claim:** Greedy (always pick earliest-ending compatible activity) is optimal.

**Proof:**
- Let `G` select activities sorted by finish time.
- Let `O` be any optimal selection.
- If `O = G`, done.
- Otherwise, let `aᵢ` be the first activity in `G` not in `O`, and `bᵢ` the activity `O` chose instead.
- Since `G` chose `aᵢ` (earlier ending), `finish(aᵢ) ≤ finish(bᵢ)`.
- Replace `bᵢ` with `aᵢ` in `O` to form `O'`:
  - `aᵢ` is compatible with all activities before it (same start constraint as `bᵢ`).
  - `aᵢ` ends no later than `bᵢ`, so it is compatible with all activities after it.
  - `|O'| = |O|` (same count).
- Repeat: `O'` is optimal and agrees with `G` one more step. By induction, `G` is optimal. ∎

### Example: Assign Cookies

**Claim:** Always assign the smallest sufficient cookie to the least-greedy child.

**Proof:**
- Suppose `O` assigns cookie `cⱼ` (size s) to child `cᵢ` (greed g), but greedy would assign cookie `cₖ` (size s' ≤ s, s' ≥ g).
- In `O`, swap `cⱼ` with `cₖ`:
  - `cᵢ` still gets satisfied (s' ≥ g).
  - The child that was getting `cₖ` might lose their cookie — but `cₖ` was the smallest sufficient cookie for `cᵢ`, meaning `cⱼ` (size s ≥ s') can still satisfy them if they are a child with greed ≤ s'.
- The swap does not reduce total satisfied children. ∎

---

## Technique 2: Greedy Stays Ahead

### Template
Define a measure `M(G, k)` = value of greedy after k steps and `M(O, k)` = value of any algorithm after k steps.

**Prove by induction:**
- Base case: `M(G, 1) ≥ M(O, 1)` (greedy is at least as good after step 1).
- Inductive step: If `M(G, k) ≥ M(O, k)`, then `M(G, k+1) ≥ M(O, k+1)`.
- Conclusion: After all steps, `M(G, n) ≥ M(O, n)`, so greedy ≥ optimal.

### Example: Jump Game II

**Measure:** `reach(k)` = farthest index reachable in exactly `k` jumps.

**Claim:** Greedy's `reach(k)` ≥ any other strategy's `reach(k)` for all k.

**Proof by induction:**
- Base: `reach(0) = 0` for all strategies (start at index 0). Equal.
- Inductive step: Assume `reachᴳ(k) ≥ reachᴼ(k)`.
  - In step k+1, greedy considers all indices `i ≤ reachᴳ(k)` and takes the best jump: `reachᴳ(k+1) = max(i + nums[i])` for `i ≤ reachᴳ(k)`.
  - Any other strategy can only use indices `i ≤ reachᴼ(k) ≤ reachᴳ(k)`.
  - Since greedy has at least as large a pool of positions to jump from, `reachᴳ(k+1) ≥ reachᴼ(k+1)`.
- Therefore, greedy reaches the destination in the minimum number of jumps. ∎

### Example: Gas Station

**Measure:** `currentGain(k)` = fuel remaining after visiting station k.

**Claim:** If a valid starting station exists, the greedy candidate `startStation` is correct.

**Key lemma:** If starting from station `s` leads to deficit at station `t` (i.e., `sum(gas[s..t] - cost[s..t]) < 0`), then no station `s'` with `s < s' ≤ t` can be the valid start.

**Proof of lemma:**
- `sum(gas[s..t] - cost[s..t]) < 0`
- For any `s'` with `s < s' ≤ t`: `sum(gas[s'..t] - cost[s'..t]) = sum(gas[s..t]) - sum(gas[s..s'-1])`
- Since starting from `s` already produced a deficit by `t`, skipping the gain from `s` to `s'-1` makes the situation strictly worse.
- More directly: Starting at `s'` means we skip the gains from `s` to `s'-1`. If we could not make it with those gains, we certainly cannot make it without them. ∎

---

## Technique 3: Matroid / Exchange Property (Advanced)

### Definition
A **matroid** is a pair `(S, I)` where:
- `S` is a ground set
- `I` is a collection of "independent sets" (subsets of S)

Satisfying:
1. **Hereditary:** If `A ∈ I` and `B ⊆ A`, then `B ∈ I`.
2. **Exchange property:** If `A, B ∈ I` and `|A| < |B|`, then there exists `x ∈ B \ A` such that `A ∪ {x} ∈ I`.

### Graphic Matroid Example (for Activity Selection)
- `S` = set of all activities/intervals
- `I` = collection of all non-overlapping subsets of activities
- This forms a matroid (hereditary and exchange property both hold).

**Theorem:** Greedy on a matroid (sort by weight, add if independent) yields optimal solution.

**Relevance:** Activity selection is essentially: maximize cardinality of an independent set in this matroid → greedy by end-time sort is provably optimal.

---

## Quick Proof Selection Guide

| "Why does greedy work for...?" | Proof to use |
|-------------------------------|--------------|
| Non-overlapping Intervals | Exchange argument: swap to earlier-ending interval |
| Assign Cookies | Exchange argument: swap to smaller sufficient cookie |
| Jump Game II | Greedy stays ahead: maxReach after k jumps |
| Partition Labels | Direct: window cannot be split (character spans boundary) |
| Gas Station | Greedy stays ahead + lemma: failed start eliminates range |
| Candy | Direct: two constraints, two passes, take max |
| Hand of Straights | Forced choice: smallest card must start a group |
| Meeting Rooms II | Exchange: assigning to earliest-freed room never wastes rooms |

> **Last Updated:** 2026-06-26
