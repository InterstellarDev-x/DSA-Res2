# Sliding Window Rate Limiter

> **Topic:** [Sliding Window](../README.md) · **Design 1 of 1**
> **Problems:** Design Hit Counter (LC 362) · Number of Recent Calls (LC 933) · Rate Limiter design

---

## Problem 1: Number of Recent Calls — LC 933

Return the number of calls in the last 3000 milliseconds.

```java
class RecentCounter {
    private Queue<Integer> window = new ArrayDeque<>();

    public int ping(int t) {
        window.offer(t);
        while (window.peek() < t - 3000) window.poll();
        return window.size();
    }
}
```

**Why this works:** `t` is strictly increasing. The queue is a sliding window `[t-3000, t]`. Each timestamp is added once and removed once — O(1) amortized.

---

## Problem 2: Design Hit Counter — LC 362

Count hits in the last 5 minutes (300 seconds). `hit(timestamp)` records a hit. `getHits(timestamp)` returns hits in `[timestamp-299, timestamp]`.

### Approach 1: Queue-based sliding window

```java
class HitCounter {
    private Queue<Integer> hits = new ArrayDeque<>();

    public void hit(int timestamp) {
        hits.offer(timestamp);
    }

    public int getHits(int timestamp) {
        while (!hits.isEmpty() && hits.peek() <= timestamp - 300) {
            hits.poll();
        }
        return hits.size();
    }
}
```

**Complexity:** O(1) amortized for both operations. Queue can hold at most all hits in the last 300 seconds.

### Approach 2: Fixed-size circular array (O(1) worst case, O(1) space)

```java
class HitCounter {
    private int[] times = new int[300];
    private int[] counts = new int[300];

    public void hit(int timestamp) {
        int idx = timestamp % 300;
        if (times[idx] == timestamp) {
            counts[idx]++;         // same second → increment
        } else {
            times[idx] = timestamp;  // new second → overwrite
            counts[idx] = 1;
        }
    }

    public int getHits(int timestamp) {
        int total = 0;
        for (int i = 0; i < 300; i++) {
            if (timestamp - times[i] < 300) {
                total += counts[i];
            }
        }
        return total;
    }
}
```

**Circular array insight:** Timestamps 1 and 301 map to the same slot (`1 % 300 == 1`). We store the timestamp to detect stale entries. If `timestamp - times[idx] >= 300`, the slot is stale and not counted.

**Complexity:** O(1) hit, O(300) = O(1) getHits. Handles high-frequency same-second hits efficiently.

---

## Problem 3: Token Bucket Rate Limiter (System Design Component)

Design a rate limiter that allows at most `capacity` requests per `windowMs` milliseconds.

```java
class RateLimiter {
    private final int capacity;
    private final long windowMs;
    private final Queue<Long> requests = new ArrayDeque<>();

    public RateLimiter(int capacity, long windowMs) {
        this.capacity = capacity;
        this.windowMs = windowMs;
    }

    public synchronized boolean allow(long currentTimeMs) {
        // Remove requests outside the sliding window
        while (!requests.isEmpty() && requests.peek() <= currentTimeMs - windowMs) {
            requests.poll();
        }
        if (requests.size() < capacity) {
            requests.offer(currentTimeMs);
            return true;
        }
        return false;
    }
}
```

**Key properties:**
- O(1) amortized per `allow()` call
- `synchronized` for thread safety in single-instance deployments
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
