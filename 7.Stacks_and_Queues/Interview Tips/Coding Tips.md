# Coding Tips — Stacks & Queues

> **Topic:** [Stacks & Queues](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## 1. Always Use `ArrayDeque`, Never `Stack<>`

```java
// BAD — legacy class, synchronized, inherits Vector methods
Stack<Integer> stack = new Stack<>();

// GOOD — faster, implements Deque interface
Deque<Integer> stack = new ArrayDeque<>();
stack.push(x);    // push to front (top)
stack.pop();      // remove from front (top)
stack.peek();     // view front (top)
```

`Stack<T>` extends `Vector` — it has synchronized methods, random access, and legacy baggage. `ArrayDeque` is the canonical replacement.

---

## 2. Monotonic Stack — Store Indices, Not Values

```java
// BAD — can't compute distances or write back to result array
Deque<Integer> stack = new ArrayDeque<>();
stack.push(nums[i]);   // pushing value

// GOOD
Deque<Integer> stack = new ArrayDeque<>();
stack.push(i);         // pushing index
// Then: nums[stack.peek()] gives value, stack.peek() gives position
```

Reason: Distance-based answers (Daily Temperatures: `i - prevIdx`) and writing answers back to result arrays both require the index.

---

## 3. Monotonic Deque — Use `offerFirst/Last` and `pollFirst/Last`

```java
// For sliding window maximum (decreasing deque):
Deque<Integer> dq = new ArrayDeque<>();
// Add to back (maintain decreasing order from front to back)
while (!dq.isEmpty() && nums[dq.peekLast()] <= nums[i]) dq.pollLast();
dq.offerLast(i);
// Get max from front
int max = nums[dq.peekFirst()];
// Expire old window from front
if (dq.peekFirst() <= i - k) dq.pollFirst();
```

Never mix `push/pop` (stack style, operates on front) with `offerLast/pollFirst` (queue style). Be explicit about which end you're accessing.

---

## 4. Histogram Width Formula

```java
while (!stack.isEmpty() && heights[stack.peek()] > h) {
    int height = heights[stack.pop()];
    // LEFT boundary is exclusive: stack.peek() + 1
    // RIGHT boundary is exclusive: i (current index)
    int width = stack.isEmpty() ? i : i - stack.peek() - 1;
    maxArea = Math.max(maxArea, height * width);
}
```

Mnemonic: "Width is the gap between the two shorter walls." `i` is the right wall, `stack.peek()` is the left wall. Width = `right - left - 1` (excluding both walls).

---

## 5. Basic Calculator — The `sign` Variable Pattern

```java
int num = 0;
char sign = '+';   // sign of the UPCOMING number, not current

// When we see an operator or reach end of string:
// 1. Apply 'sign' to 'num' (process the number we just built)
// 2. Update 'sign' = current operator
// 3. Reset num = 0
```

This lookahead trick works because in `"3+5*2"`, when we see `+` we're done reading `3`. The sign `+` applies to the next operand `5` (we don't know yet).

---

## 6. Decode String — Multi-Digit Counts

```java
if (Character.isDigit(c)) {
    k = k * 10 + (c - '0');  // accumulate: "12[" → k = 1*10+2 = 12
}
```

Easy to forget when writing the solution quickly. `"12[a]"` → `"aaaaaaaaaaaa"` (12 a's). Without the `k * 10`, you'd get `k = 2`.

---

## 7. Queue via Two Stacks — Transfer Only When Needed

```java
private void transfer() {
    if (outStack.isEmpty()) {    // ← only transfer when outStack is EMPTY
        while (!inStack.isEmpty()) outStack.push(inStack.pop());
    }
}
```

Partial transfer (when outStack has some items) would break FIFO ordering. Transfer only when `outStack` is completely drained.

---

## 8. Max Frequency Stack — `computeIfAbsent` Pattern

```java
group.computeIfAbsent(f, k -> new ArrayDeque<>()).push(val);
```

Equivalent to:
```java
if (!group.containsKey(f)) group.put(f, new ArrayDeque<>());
group.get(f).push(val);
```

The `computeIfAbsent` version is idiomatic Java and avoids the double-lookup.

---

## 9. Circular Queue — `size` Variable vs One Slot Wasted

```java
// Option A (recommended): Track size separately
if (isFull()) return false;  // size == capacity
tail = (tail + 1) % capacity;
data[tail] = value;
size++;

// Option B: One slot wasted (capacity+1 slots for N-element queue)
// isFull = (tail + 1) % capacity == head
// This wastes one slot but avoids a separate size counter
```

Option A with `size` is cleaner and avoids the off-by-one confusion inherent in Option B.

---

## 10. Sentinel in Histogram

```java
for (int i = 0; i <= n; i++) {          // ← i goes to n (inclusive)
    int h = (i == n) ? 0 : heights[i];   // ← sentinel height = 0
    // ...
}
```

The sentinel forces all remaining stack entries to flush after the loop. Forgetting this is the most common bug in histogram problems — elements that are never smaller than their right neighbor stay in the stack forever.

---

## Quick Reference: Method Names

| Operation | Stack (ArrayDeque) | Queue (ArrayDeque) | Deque (explicit) |
|-----------|-------------------|-------------------|-----------------|
| Add top/front | `push(x)` | `offer(x)` | `offerFirst(x)` |
| Add bottom/back | — | — | `offerLast(x)` |
| Remove top/front | `pop()` | `poll()` | `pollFirst()` |
| Remove back | — | — | `pollLast()` |
| View top/front | `peek()` | `peek()` | `peekFirst()` |
| View back | — | — | `peekLast()` |

---

## Related Files

- [Common Mistakes](./Common%20Mistakes.md)
- [Stack vs Queue Usage](./Stack%20vs%20Queue%20Usage.md)
- [Complexity Analysis](./Complexity%20Analysis.md)
- [Monotonic Stack Pattern](../Patterns/Monotonic%20Stack.md)
