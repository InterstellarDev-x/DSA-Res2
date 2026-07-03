# Min Stack & Design Circular Queue

> **Topic:** [Stacks & Queues](../README.md) · **Design 1 of 2**
> **Problems:** Min Stack (LC 155) · Design Circular Queue (LC 622)

---

## Min Stack — LC 155

**Requirements:** Design a stack that supports `push`, `pop`, `top`, and `getMin` in O(1) time.

The challenge: standard stacks don't track minimums. After a `pop`, the minimum might change — so we need to track minimums through the history.

---

### Approach 1: Auxiliary Min Stack — O(1) all operations, O(n) space

Maintain a second stack (`min_stk`) that tracks the running minimum at each level.

```rust
struct MinStack {
    stk: Vec<i32>,
    min_stk: Vec<i32>,
}

impl MinStack {
    fn new() -> Self {
        MinStack {
            stk: Vec::new(),
            min_stk: Vec::new(),
        }
    }

    fn push(&mut self, val: i32) {
        self.stk.push(val);
        // Push to min_stk if it's the new minimum (or min_stk is empty)
        let mn = if self.min_stk.is_empty() {
            val
        } else {
            val.min(*self.min_stk.last().unwrap())
        };
        self.min_stk.push(mn);
    }

    fn pop(&mut self) {
        self.stk.pop();
        self.min_stk.pop();    // always pop both together
    }

    fn top(&self) -> i32     { *self.stk.last().unwrap() }
    fn get_min(&self) -> i32 { *self.min_stk.last().unwrap() }
}
```

**Why push min every time (not only when it changes)?** Because `pop()` must always pop both stacks together. If we only pushed to `min_stk` when value decreases, the two stacks would have different sizes and fall out of sync.

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

Store `(value, minAtThisPoint)` as a tuple in each stack entry. Uses one structure instead of two.

```rust
struct MinStack {
    stk: Vec<(i32, i32)>,  // (val, min_so_far)
}

impl MinStack {
    fn new() -> Self {
        MinStack { stk: Vec::new() }
    }

    fn push(&mut self, val: i32) {
        let mn = if self.stk.is_empty() {
            val
        } else {
            val.min(self.stk.last().unwrap().1)
        };
        self.stk.push((val, mn));
    }

    fn pop(&mut self)         { self.stk.pop(); }
    fn top(&self) -> i32      { self.stk.last().unwrap().0 }
    fn get_min(&self) -> i32  { self.stk.last().unwrap().1 }
}
```

**Trade-off:** Slightly more memory per entry (two i32s), but one data structure. The tuple approach is cleaner in code.

---

### Approach 3: Encoding trick — O(1) time, O(n) space (avoids extra structure)

Store the difference `(val - currentMin)` in the stack. When difference is negative, we know `val` was a new minimum.

```rust
struct MinStack {
    stk: Vec<i64>,
    mn: i64,
}

impl MinStack {
    fn new() -> Self {
        MinStack {
            stk: Vec::new(),
            mn: i64::MAX,
        }
    }

    fn push(&mut self, val: i32) {
        let val = val as i64;
        if self.stk.is_empty() {
            self.stk.push(0);
            self.mn = val;
        } else {
            self.stk.push(val - self.mn);
            if val < self.mn { self.mn = val; }
        }
    }

    fn pop(&mut self) {
        let diff = self.stk.pop().unwrap();
        if diff < 0 { self.mn = self.mn - diff; }   // restore previous min
    }

    fn top(&self) -> i32 {
        let diff = *self.stk.last().unwrap();
        if diff < 0 { self.mn as i32 } else { (self.mn + diff) as i32 }
    }

    fn get_min(&self) -> i32 { self.mn as i32 }
}
```

**Why `i64`?** `val - min` can overflow `i32` if `val` is large positive and `min` is large negative (or vice versa).

**Restore logic on pop:** If stored `diff < 0`, it means `min` was updated to `val = min + diff` when this was pushed. The previous min = `min - diff`.

---

## Design Circular Queue — LC 622

**Requirements:** `enQueue`, `deQueue`, `Front`, `Rear`, `isEmpty`, `isFull` — all O(1).

### Array-based Circular Buffer

```rust
struct MyCircularQueue {
    data: Vec<i32>,
    head: usize,
    tail: usize,   // initialized to capacity-1; first enqueue wraps to 0
    sz: usize,
    capacity: usize,
}

impl MyCircularQueue {
    fn new(k: usize) -> Self {
        MyCircularQueue {
            data: vec![0; k],
            head: 0,
            tail: k - 1,  // equivalent to -1: (k-1+1)%k = 0 on first enqueue
            sz: 0,
            capacity: k,
        }
    }

    fn en_queue(&mut self, value: i32) -> bool {
        if self.is_full() { return false; }
        self.tail = (self.tail + 1) % self.capacity;
        self.data[self.tail] = value;
        self.sz += 1;
        true
    }

    fn de_queue(&mut self) -> bool {
        if self.is_empty() { return false; }
        self.head = (self.head + 1) % self.capacity;
        self.sz -= 1;
        true
    }

    fn front(&self) -> i32 { if self.is_empty() { -1 } else { self.data[self.head] } }
    fn rear(&self) -> i32  { if self.is_empty() { -1 } else { self.data[self.tail] } }
    fn is_empty(&self) -> bool { self.sz == 0 }
    fn is_full(&self) -> bool  { self.sz == self.capacity }
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
A: The tuple/two-stack approaches are O(n) by necessity — we must remember the minimum at each stack level in case pops reveal a new minimum. The encoding trick reduces constant factor but is still O(n). Fundamental lower bound: O(n) space for a stack supporting getMin in O(1).

**Q: Thread safety?**
A: Wrap the struct in `std::sync::Mutex` and use `MutexGuard` for access. The two-stack approach has a window where the main stack has been popped but `min_stk` hasn't — this is a critical section.

### Circular Queue
**Q: Why circular instead of `VecDeque`?**
A: Array has O(1) index access and better cache locality. A linked list has pointer overhead per node. For a fixed-size bounded queue, circular array is standard.

**Q: How to implement a circular DEQUE?**
A: Same structure but add `enqueue_front` and `dequeue_rear`:
- `enqueue_front`: `head = (head + capacity - 1) % capacity`
- `dequeue_rear`: `tail = (tail + capacity - 1) % capacity`

---

## Related Files

- [Maximum Frequency Stack](./Maximum%20Frequency%20Stack.md)
- [Stack Basics](../Patterns/Stack%20Basics.md)
- [Queue & Deque](../Patterns/Queue%20and%20Deque.md)
- [Complexity Analysis](../Interview%20Tips/Complexity%20Analysis.md)

> **Last Updated:** 2026-06-26
