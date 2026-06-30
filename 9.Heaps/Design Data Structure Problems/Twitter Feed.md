# Twitter Feed (Design Twitter)

> **Topic:** [Heaps](../README.md) · **Design 2 of 2**
> **Problems:** Design Twitter (LC 355)

---

## Problem Statement — LC 355

Design a simplified version of Twitter with:
- `postTweet(userId, tweetId)` — compose a new tweet
- `getNewsFeed(userId)` — return 10 most recent tweets from the user and their followees
- `follow(followerId, followeeId)`
- `unfollow(followerId, followeeId)`

---

## Solution: K-Way Merge via Min-Heap

```java
class Twitter {
    private int timestamp = 0;
    private Map<Integer, List<int[]>> tweets = new HashMap<>();  // userId → [(time, tweetId)]
    private Map<Integer, Set<Integer>> follows = new HashMap<>();  // userId → Set<followeeId>

    public void postTweet(int userId, int tweetId) {
        tweets.computeIfAbsent(userId, k -> new ArrayList<>())
              .add(new int[]{timestamp++, tweetId});
    }

    public List<Integer> getNewsFeed(int userId) {
        // Min-heap: (timestamp, tweetId, userId, tweetIndex)
        // We want 10 most recent → keep min at top to evict old ones
        // Actually: max-heap for most recent first
        PriorityQueue<int[]> heap = new PriorityQueue<>((a, b) -> b[0] - a[0]); // max by timestamp

        // Add user's own latest tweet
        Set<Integer> users = new HashSet<>(follows.getOrDefault(userId, Collections.emptySet()));
        users.add(userId);  // include self

        for (int uid : users) {
            List<int[]> userTweets = tweets.get(uid);
            if (userTweets != null && !userTweets.isEmpty()) {
                int lastIdx = userTweets.size() - 1;
                int[] latest = userTweets.get(lastIdx);
                heap.offer(new int[]{latest[0], latest[1], uid, lastIdx});
            }
        }

        List<Integer> feed = new ArrayList<>();
        while (!heap.isEmpty() && feed.size() < 10) {
            int[] curr = heap.poll();
            feed.add(curr[1]);  // tweetId
            int idx = curr[3] - 1;
            if (idx >= 0) {
                int[] prev = tweets.get(curr[2]).get(idx);
                heap.offer(new int[]{prev[0], prev[1], curr[2], idx});
            }
        }
        return feed;
    }

    public void follow(int followerId, int followeeId) {
        follows.computeIfAbsent(followerId, k -> new HashSet<>()).add(followeeId);
    }

    public void unfollow(int followerId, int followeeId) {
        if (follows.containsKey(followerId)) follows.get(followerId).remove(followeeId);
    }
}
```

**K-Way Merge application:** Each user's tweet list is sorted by time (insertion order). We merge across all followees + self to get the top 10, using a max-heap on timestamp.

**Complexity:**
- `postTweet`: O(1)
- `getNewsFeed`: O(F log F + 10 log F) where F = number of followees. Initial heap = O(F log F). Each of 10 iterations = O(log F).
- `follow`/`unfollow`: O(1) average with HashSet

---

## Design Decisions

**Why timestamp (not tweet ID) for ordering?**
Tweet IDs are not guaranteed to be monotonically increasing. A global `timestamp++` provides a reliable ordering.

**Why store `(time, tweetId, userId, index)` in heap?**
- `time`: for comparison (max-heap)
- `tweetId`: for the result
- `userId`: to look up the user's tweet list when advancing
- `index`: pointer to current position in that user's tweet list

**Why not just merge and sort all tweets?**
With many followees each having many tweets, sorting all is O(N log N) where N = total tweets. K-way merge with heap is O(F log F + 10 log F) — much better when we only need 10.

---

## Follow-up Discussion Points

**Q: Scale to millions of users?**
A: Shard tweet storage by `userId` across servers. `getNewsFeed` becomes a distributed fan-out: query each followee's shard, merge results at an aggregation layer using the same heap approach.

**Q: What if a user follows 10,000 people?**
A: The heap initialization becomes O(10,000 log 10,000) ≈ O(130,000) ops per `getNewsFeed` — expensive. Real Twitter uses pre-computed "fan-out on write" for popular users.

**Q: How would you handle tweet deletion?**
A: Mark tweets as deleted with a `boolean` flag. Skip deleted tweets during `getNewsFeed` merge.

---

## Related Files

- [K-Way Merge Pattern](../Patterns/K%20Way%20Merge.md)
- [Median Finder Design](./Median%20Finder.md)
- [Interview Problems: Google](../Interview%20Problems/Google.md)

> **Last Updated:** 2026-06-26
