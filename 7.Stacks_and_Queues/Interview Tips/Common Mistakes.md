# Common Mistakes — Stacks & Queues

> **Topic:** [Stacks & Queues](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## Mistake 1: Using `Stack<>` Instead of `ArrayDeque`

```java
// BAD
Stack<Integer> stack = new Stack<>();
stack.push(1);

// GOOD
Deque<Integer> stack = new ArrayDeque<>();
stack.push(1);
```

`Stack<T>` is a legacy class that extends `Vector` and is synchronized by default. It also exposes `get(i)` random access, which doesn't belong on a stack. `ArrayDeque` is the correct choice.

---

## Mistake 2: Pushing Values Instead of Indices in Monotonic Stack

```java
// BAD — for Daily Temperatures
while (!stack.isEmpty() && nums[stack.peek()] < nums[i]) {
    int prevIdx = stack.pop();
    result[prevIdx] = i - prevIdx;  // ERROR: stack.pop() returns the VALUE, can't do i - value
}

// GOOD
while (!stack.isEmpty() && temperatures[stack.peek()] < temperatures[i]) {
    int prevIdx = stack.pop();
    result[prevIdx] = i - prevIdx;  // prevIdx is an INDEX, subtraction gives days
}
stack.push(i);  // push index, not temperatures[i]
```

If you push values, you can't compute distances or write back to the result array by position.

---

## Mistake 3: Histogram Width Formula Off-By-One

```java
// BAD — wrong width
int width = i - stack.peek();           // off by 1 (includes left wall)

// GOOD
int width = stack.isEmpty() ? i : i - stack.peek() - 1;

// Explanation:
// - Right boundary (exclusive): i (heights[i] is shorter, so rectangle ends before i)
// - Left boundary (exclusive): stack.peek() (this is the next-shorter bar to the left)
// - Width: (i-1) - (stack.peek()+1) + 1 = i - stack.peek() - 1
```

The `-1` is the most common off-by-one in this problem. Verify with a 1-element array: `heights=[5]`, at sentinel `i=1`, stack.pop()=0, stack is empty, width=1 ✓.

---

## Mistake 4: Forgetting the Histogram Sentinel

```java
// BAD — bars that are never smaller than their right neighbor stay in stack
for (int i = 0; i < n; i++) {
    while (!stack.isEmpty() && heights[stack.peek()] > heights[i]) {
        // compute area
    }
    stack.push(i);
}
// Stack still has entries! Never flushed for bars like [3, 5, 7]

// GOOD
for (int i = 0; i <= n; i++) {         // <= n, not < n
    int h = (i == n) ? 0 : heights[i]; // height 0 flushes all remaining
    // ...
}
```

For input `[3, 5, 7]` (monotonically increasing), no pops occur in the main loop. The sentinel forces all three to be popped and their areas computed.

---

## Mistake 5: Wrong Transfer Order in Queue via Two Stacks

```java
// BAD — partial transfer breaks FIFO
private void transfer() {
    while (!inStack.isEmpty()) outStack.push(inStack.pop());  // always transfers
}

// GOOD — only transfer when outStack is completely empty
private void transfer() {
    if (outStack.isEmpty())
        while (!inStack.isEmpty()) outStack.push(inStack.pop());
}
```

If `outStack` has `[3,2,1]` (1 on top = dequeue next) and you transfer `inStack=[6,5,4]` (4 on top), you'd get `outStack=[4,5,6,3,2,1]` with 4 as the next to dequeue — wrong order.

---

## Mistake 6: Circular Queue — Forgetting `size` or Using Wrong Condition

```java
// BAD — ambiguous: head==tail could mean empty OR full
boolean isEmpty() { return head == tail; }
boolean isFull()  { return head == tail; }  // same condition!

// GOOD — track size separately
private int size = 0;
boolean isEmpty() { return size == 0; }
boolean isFull()  { return size == capacity; }
```

Without a size counter (or a `isFull` boolean flag), you can't distinguish an empty circular buffer from a full one when `head == tail`.

---

## Mistake 7: Basic Calculator — Forgetting the Last Number

```java
// BAD — misses the last token (no operator after it)
for (int i = 0; i < s.length(); i++) {
    char c = s.charAt(i);
    if (Character.isDigit(c)) num = num * 10 + (c - '0');
    if (!Character.isDigit(c) && c != ' ') {  // WRONG: only processes on operator
        // apply sign to num
    }
}
// Last number never gets processed!

// GOOD — process on operator OR at end of string
if ((!Character.isDigit(c) && c != ' ') || i == s.length() - 1) {
    // apply sign to num
}
```

The input `"3+5"` ends with a digit. Without the `i == s.length() - 1` condition, `5` is never pushed to the stack.

---

## Mistake 8: Decode String — Not Resetting `k` After `[`

```java
// BAD
} else if (c == '[') {
    countStack.push(k);
    strStack.push(current);
    current = new StringBuilder();
    // MISSING: k = 0
}

// GOOD
} else if (c == '[') {
    countStack.push(k);
    strStack.push(current);
    k = 0;                    // ← reset for the next number
    current = new StringBuilder();
}
```

Without resetting `k`, nested brackets like `"2[3[a]]"` would use `k=3` again for the outer bracket instead of finding `k=2`.

---

## Mistake 9: Max Frequency Stack — `maxFreq` Decrement Condition

```java
// BAD — always decrement maxFreq on pop
public int pop() {
    Deque<Integer> stack = group.get(maxFreq);
    int val = stack.pop();
    freq.put(val, freq.get(val) - 1);
    maxFreq--;   // WRONG: only decrement if group[maxFreq] is now empty
    return val;
}

// GOOD
public int pop() {
    Deque<Integer> stack = group.get(maxFreq);
    int val = stack.pop();
    freq.put(val, freq.get(val) - 1);
    if (stack.isEmpty()) {
        group.remove(maxFreq);
        maxFreq--;   // only decrement when bucket is empty
    }
    return val;
}
```

After popping from `group[3]`, if `group[3]` still has elements, `maxFreq` should stay 3. Decrementing unconditionally would skip valid elements.

---

## Mistake 10: 132 Pattern — Confusing Which Pointer Is Which

The pattern requires three pointers: `nums[i] < nums[k] < nums[j]` where `i < j < k`.
- `stack` = candidates for `nums[j]` (the "3" in 132 — the largest)
- `third` = best candidate for `nums[k]` (the "2" in 132 — middle value)
- `nums[i]` = current element being checked as the smallest ("1")

```java
// BAD — checking wrong condition
if (nums[i] > third) return true;   // WRONG: we want nums[i] < third

// GOOD
if (nums[i] < third) return true;   // nums[i] is "1", must be LESS than "2" (third)
```

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Stack vs Queue Usage](./Stack%20vs%20Queue%20Usage.md)
- [Complexity Analysis](./Complexity%20Analysis.md)
