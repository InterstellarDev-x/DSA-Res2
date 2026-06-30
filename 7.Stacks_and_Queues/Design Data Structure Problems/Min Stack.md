# Min Stack & Design Circular Queue

> **Topic:** [Stacks & Queues](../README.md) · **Design 1 of 2**
> **Problems:** Min Stack (LC 155) · Design Circular Queue (LC 622)

---

## Min Stack — LC 155

**Requirements:** Design a stack that supports `push`, `pop`, `top`, and `getMin` in O(1) time.

The challenge: standard stacks don't track minimums. After a `pop`, the minimum might change — so we need to track minimums through the history.

---

### Approach 1: Auxiliary Min Stack — O(1) all operations, O(n) space

Maintain a second stack (`minStack`) that tracks the running minimum at each level.

```java
class MinStack {
    private Deque<Integer> stack    = new ArrayDeque<>();
    private Deque<Integer> minStack = new ArrayDeque<>();

    public void push(int val) {
        stack.push(val);
        // Push to minStack if it's the new minimum (or minStack is empty)
        int min = minStack.isEmpty() ? val : Math.min(val, minStack.peek());
        minStack.push(min);
    }

    public void pop() {
        stack.pop();
        minStack.pop();    // always pop both together
    }

    public int top()    { return stack.peek(); }
    public int getMin() { return minStack.peek(); }
}
```

**Why push min every time (not only when it changes)?** Because `pop()` must always pop both stacks together. If we only pushed to `minStack` when value decreases, the two stacks would have different sizes and fall out of sync.

**Trace:**
```
push(5): stack=[5], minStack=[5]
push(3): stack=[3,5], minStack=[3,3]
push(7): stack=[7,3,5], minStack=[3,3,3] — min doesn't change but we sync
push(2): stack=[2,7,3,5], minStack=[2,2,3,3]
getMin()=2 ✓
pop(): stack=[7,3,5], minStack=[2,3,3]
getMin()=2 ✓
pop(): stack=[3,5], minStack=[3,3]
getMin()=3 ✓
```

---

### Approach 2: Single Stack with Pairs — O(1) all operations, O(n) space

Store `(value, minAtThisPoint)` as a pair in each stack entry. Uses one structure instead of two.

```java
class MinStack {
    private Deque<int[]> stack = new ArrayDeque<>();  // [val, minSoFar]

    public void push(int val) {
        int min = stack.isEmpty() ? val : Math.min(val, stack.peek()[1]);
        stack.push(new int[]{val, min});
    }

    public void pop()    { stack.pop(); }
    public int top()     { return stack.peek()[0]; }
    public int getMin()  { return stack.peek()[1]; }
}
```

**Trade-off:** Slightly more memory per entry (two ints), but one data structure. The pair approach is cleaner in code.

---

### Approach 3: Encoding trick — O(1) time, O(n) space (avoids extra structure)

Store the difference `(val - currentMin)` in the stack. When difference is negative, we know `val` was a new minimum.

```java
class MinStack {
    private Deque<Long> stack = new ArrayDeque<>();
    private long min = Long.MAX_VALUE;

    public void push(int val) {
        if (stack.isEmpty()) {
            stack.push(0L);
            min = val;
        } else {
            stack.push((long) val - min);
            if (val < min) min = val;
        }
    }

    public void pop() {
        long diff = stack.pop();
        if (diff < 0) min = min - diff;   // restore previous min
    }

    public int top() {
        long diff = stack.peek();
        return diff < 0 ? (int) min : (int)(min + diff);
    }

    public int getMin() { return (int) min; }
}
```

**Why `long`?** `val - min` can overflow `int` if `val` is large positive and `min` is large negative (or vice versa).

**Restore logic on pop:** If stored `diff < 0`, it means `min` was updated to `val = min + diff` when this was pushed. The previous min = `min - diff`.

---

## Design Circular Queue — LC 622

**Requirements:** `enQueue`, `deQueue`, `Front`, `Rear`, `isEmpty`, `isFull` — all O(1).

### Array-based Circular Buffer

```java
class MyCircularQueue {
    private int[] data;
    private int head, tail, size, capacity;

    public MyCircularQueue(int k) {
        data = new int[k];
        head = 0; tail = -1; size = 0; capacity = k;
    }

    public boolean enQueue(int value) {
        if (isFull()) return false;
        tail = (tail + 1) % capacity;
        data[tail] = value;
        size++;
        return true;
    }

    public boolean deQueue() {
        if (isEmpty()) return false;
        head = (head + 1) % capacity;
        size--;
        return true;
    }

    public int Front() { return isEmpty() ? -1 : data[head]; }
    public int Rear()  { return isEmpty() ? -1 : data[tail]; }
    public boolean isEmpty() { return size == 0; }
    public boolean isFull()  { return size == capacity; }
}
```

**Why track `size` separately?** Avoids the "one slot wasted" trick (reserving one empty slot to distinguish full from empty based on head/tail positions).

**Modulo wrap-around:** `(index + 1) % capacity` is the key operation that makes the array circular. When `tail` reaches the end, it wraps to 0.

**Trace for capacity=3:**
```
enQueue(1): tail=0, data=[1,_,_], size=1
enQueue(2): tail=1, data=[1,2,_], size=2
enQueue(3): tail=2, data=[1,2,3], size=3  ← full
deQueue(): head=1, data=[_,2,3], size=2
enQueue(4): tail=(2+1)%3=0, data=[4,2,3], size=3  ← wraps!
Front()=data[1]=2, Rear()=data[0]=4
```

---

### Alternative: Size-based tracking vs head==tail

**Ambiguity problem:** If `head == tail`, it could mean empty OR full. Solutions:
1. Track `size` (used above — cleanest)
2. Use a `boolean full` flag
3. Reserve one slot (capacity+1 allocated, full when `(tail+1)%cap == head`)

Option 1 is cleanest for interviews.

---

## Interview Discussion Points

### Min Stack
**Q: Can you do O(1) getMin with O(1) push/pop using less space?**
A: The pair/two-stack approaches are O(n) by necessity — we must remember the minimum at each stack level in case pops reveal a new minimum. The encoding trick reduces constant factor but is still O(n). Fundamental lower bound: O(n) space for a stack supporting getMin in O(1).

**Q: Thread safety?**
A: Wrap `synchronized` on methods or use `java.util.concurrent.locks.ReentrantLock`. The two-stack approach has a window where stack has been popped but minStack hasn't — this is a critical section.

### Circular Queue
**Q: Why circular instead of LinkedList?**
A: Array has O(1) index access and better cache locality. LinkedList has pointer overhead per node. For a fixed-size bounded queue, circular array is standard.

**Q: How to implement a circular DEQUE?**
A: Same structure but add `enqueueFront` and `dequeueRear`:
- `enqueueFront`: `head = (head - 1 + capacity) % capacity`
- `dequeueRear`: `tail = (tail - 1 + capacity) % capacity`

---

## Related Files

- [Maximum Frequency Stack](./Maximum%20Frequency%20Stack.md)
- [Stack Basics](../Patterns/Stack%20Basics.md)
- [Queue & Deque](../Patterns/Queue%20and%20Deque.md)
- [Complexity Analysis](../Interview%20Tips/Complexity%20Analysis.md)

> **Last Updated:** 2026-06-26
