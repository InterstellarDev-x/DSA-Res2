# Sliding Window Rate Limiter

> **Topic:** [Sliding Window](../README.md) · **Design 1 of 1**
> **Problems:** Design Hit Counter (LC 362) · Number of Recent Calls (LC 933) · Rate Limiter design

---

## Problem 1: Number of Recent Calls — LC 933

Return the number of calls in the last 3000 milliseconds.

```rust
use std::collections::VecDeque;

struct RecentCounter {
    window: VecDeque<i32>,
}

impl RecentCounter {
    fn new() -> Self {
        RecentCounter { window: VecDeque::new() }
    }

    fn ping(&mut self, t: i32) -> i32 {
        self.window.push_back(t);
        while *self.window.front().unwrap() < t - 3000 {
            self.window.pop_front();
        }
        self.window.len() as i32
    }
}
```

**Why this works:** `t` is strictly increasing. The `VecDeque` is a sliding window `[t-3000, t]`. Each timestamp is added once and removed once — O(1) amortized.

---

## Problem 2: Design Hit Counter — LC 362

Count hits in the last 5 minutes (300 seconds). `hit(timestamp)` records a hit. `getHits(timestamp)` returns hits in `[timestamp-299, timestamp]`.

### Approach 1: Queue-based sliding window

```rust
use std::collections::VecDeque;

struct HitCounter {
    hits: VecDeque<i32>,
}

impl HitCounter {
    fn new() -> Self {
        HitCounter { hits: VecDeque::new() }
    }

    fn hit(&mut self, timestamp: i32) {
        self.hits.push_back(timestamp);
    }

    fn get_hits(&mut self, timestamp: i32) -> i32 {
        while !self.hits.is_empty() && *self.hits.front().unwrap() <= timestamp - 300 {
            self.hits.pop_front();
        }
        self.hits.len() as i32
    }
}
```

**Complexity:** O(1) amortized for both operations. `VecDeque` can hold at most all hits in the last 300 seconds.

### Approach 2: Fixed-size circular array (O(1) worst case, O(1) space)

```rust
struct HitCounter {
    times: Vec<i32>,
    counts: Vec<i32>,
}

impl HitCounter {
    fn new() -> Self {
        HitCounter {
            times: vec![0; 300],
            counts: vec![0; 300],
        }
    }

    fn hit(&mut self, timestamp: i32) {
        let idx = (timestamp % 300) as usize;
        if self.times[idx] == timestamp {
            self.counts[idx] += 1;        // same second → increment
        } else {
            self.times[idx] = timestamp;  // new second → overwrite
            self.counts[idx] = 1;
        }
    }

    fn get_hits(&self, timestamp: i32) -> i32 {
        let mut total = 0;
        for i in 0..300 {
            if timestamp - self.times[i] < 300 {
                total += self.counts[i];
            }
        }
        total
    }
}
```

**Circular array insight:** Timestamps 1 and 301 map to the same slot (`1 % 300 == 1`). We store the timestamp to detect stale entries. If `timestamp - times[idx] >= 300`, the slot is stale and not counted.

**Complexity:** O(1) hit, O(300) = O(1) get_hits. Handles high-frequency same-second hits efficiently.

---

## Problem 3: Token Bucket Rate Limiter (System Design Component)

Design a rate limiter that allows at most `capacity` requests per `window_ms` milliseconds.

```rust
use std::collections::VecDeque;
use std::sync::Mutex;

struct RateLimiter {
    capacity: usize,
    window_ms: i64,
    requests: Mutex<VecDeque<i64>>,
}

impl RateLimiter {
    fn new(capacity: usize, window_ms: i64) -> Self {
        RateLimiter {
            capacity,
            window_ms,
            requests: Mutex::new(VecDeque::new()),
        }
    }

    fn allow(&self, current_time_ms: i64) -> bool {
        let mut requests = self.requests.lock().unwrap();
        // Remove requests outside the sliding window
        while !requests.is_empty() && *requests.front().unwrap() <= current_time_ms - self.window_ms {
            requests.pop_front();
        }
        if requests.len() < self.capacity {
            requests.push_back(current_time_ms);
            return true;
        }
        false
    }
}
```

**Key properties:**
- O(1) amortized per `allow()` call
- `Mutex::lock().unwrap()` for thread safety in single-instance deployments
- Sliding window (not fixed window) — avoids the "double burst at boundary" problem of fixed windows

---

## Sliding Window vs Token Bucket vs Fixed Window

| Algorithm | Burst Handling | Complexity | Consistency |
|-----------|---------------|-----------|-------------|
| Fixed Window | Can double-burst at boundary | O(1) | No |
| Sliding Window | Smooth — no boundary burst | O(1) amortized | Yes |
| Token Bucket | Allows burst up to bucket size | O(1) | Yes (configurable) |
| Leaky Bucket | Constant output rate | O(1) | Yes |

**The boundary problem with fixed windows:** If window is 60 seconds and limit is 100, a user can send 100 at second 59 and 100 more at second 61 (new window) — 200 requests in 2 seconds, 2× the limit.

---

## Interview Discussion Points

**Q: How would you scale this to a distributed system?**
A: Use Redis with sorted sets: `ZADD key timestamp requestId` + `ZREMRANGEBYSCORE` to remove old entries + `ZCARD` to count. This is O(log n) per operation but works across nodes.

**Q: What if the same timestamp is hit by many concurrent requests?**
A: The circular array approach handles this — same timestamp maps to same slot, we increment `counts[idx]` instead of inserting a new entry.

**Q: How do you handle clock skew between servers?**
A: Use a monotonic clock or logical timestamps. If clocks drift, the sliding window may count the same request twice or miss requests — a distributed coordination service (e.g., Redis Lua scripts for atomic check-and-set) is needed.

---

## Related Files

- [Queue & Deque](../../7.Stacks_and_Queues/Patterns/Queue%20and%20Deque.md)
- [Fixed Size Window](../Patterns/Fixed%20Size%20Window.md)
- [Variable Size Window](../Patterns/Variable%20Size%20Window.md)

> **Last Updated:** 2026-06-26
