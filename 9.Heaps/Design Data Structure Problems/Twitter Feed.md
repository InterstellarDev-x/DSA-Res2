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

```rust
use std::collections::{HashMap, HashSet, BinaryHeap};

struct Twitter {
    timestamp: i32,
    tweets: HashMap<i32, Vec<(i32, i32)>>,  // userId → Vec<(time, tweetId)>
    follows: HashMap<i32, HashSet<i32>>,     // userId → set of followeeId
}

impl Twitter {
    fn new() -> Self {
        Twitter {
            timestamp: 0,
            tweets: HashMap::new(),
            follows: HashMap::new(),
        }
    }

    fn post_tweet(&mut self, user_id: i32, tweet_id: i32) {
        self.tweets.entry(user_id).or_default().push((self.timestamp, tweet_id));
        self.timestamp += 1;
    }

    fn get_news_feed(&self, user_id: i32) -> Vec<i32> {
        // Max-heap: (timestamp, tweetId, userId, tweetIndex)
        // We want 10 most recent → keep max at top
        let mut heap: BinaryHeap<(i32, i32, i32, usize)> = BinaryHeap::new();

        let mut users: HashSet<i32> = HashSet::new();
        if let Some(following) = self.follows.get(&user_id) {
            users.extend(following);
        }
        users.insert(user_id);  // include self

        for &uid in &users {
            if let Some(user_tweets) = self.tweets.get(&uid) {
                if !user_tweets.is_empty() {
                    let last_idx = user_tweets.len() - 1;
                    let (time, tid) = user_tweets[last_idx];
                    heap.push((time, tid, uid, last_idx));
                }
            }
        }

        let mut feed = Vec::new();
        while !heap.is_empty() && feed.len() < 10 {
            let (_, tweet_id, uid, idx) = heap.pop().unwrap();
            feed.push(tweet_id);
            if idx > 0 {
                let prev_idx = idx - 1;
                let (time, tid) = self.tweets[&uid][prev_idx];
                heap.push((time, tid, uid, prev_idx));
            }
        }
        feed
    }

    fn follow(&mut self, follower_id: i32, followee_id: i32) {
        self.follows.entry(follower_id).or_default().insert(followee_id);
    }

    fn unfollow(&mut self, follower_id: i32, followee_id: i32) {
        if let Some(following) = self.follows.get_mut(&follower_id) {
            following.remove(&followee_id);
        }
    }
}
```

**K-Way Merge application:** Each user's tweet list is sorted by time (insertion order). We merge across all followees + self to get the top 10, using a max-heap on timestamp.

**Complexity:**
- `postTweet`: O(1)
- `getNewsFeed`: O(F log F + 10 log F) where F = number of followees. Initial heap = O(F log F). Each of 10 iterations = O(log F).
- `follow`/`unfollow`: O(1) average with `HashSet`

---

## Design Decisions

**Why timestamp (not tweet ID) for ordering?**
Tweet IDs are not guaranteed to be monotonically increasing. A global `timestamp += 1` provides a reliable ordering.

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
A: Mark tweets as deleted with a `bool` flag. Skip deleted tweets during `getNewsFeed` merge.

---

## Related Files

- [K-Way Merge Pattern](../Patterns/K%20Way%20Merge.md)
- [Median Finder Design](./Median%20Finder.md)
- [Interview Problems: Google](../Interview%20Problems/Google.md)

> **Last Updated:** 2026-06-26
