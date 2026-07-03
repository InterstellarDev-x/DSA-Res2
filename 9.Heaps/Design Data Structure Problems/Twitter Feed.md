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

```cpp
#include <bits/stdc++.h>
using namespace std;

class Twitter {
    int timestamp = 0;
    unordered_map<int, vector<vector<int>>> tweets;  // userId → [(time, tweetId)]
    unordered_map<int, unordered_set<int>> follows;  // userId → set<followeeId>

public:
    void postTweet(int userId, int tweetId) {
        tweets[userId].push_back({timestamp++, tweetId});
    }

    vector<int> getNewsFeed(int userId) {
        // Max-heap: (timestamp, tweetId, userId, tweetIndex)
        // We want 10 most recent → keep max at top
        auto cmp = [](const vector<int>& a, const vector<int>& b) {
            return a[0] < b[0]; // max by timestamp
        };
        priority_queue<vector<int>, vector<vector<int>>, decltype(cmp)> heap(cmp);

        // Add user's own latest tweet
        unordered_set<int> users;
        if (follows.count(userId)) users = follows[userId];
        users.insert(userId);  // include self

        for (int uid : users) {
            if (tweets.count(uid) && !tweets[uid].empty()) {
                int lastIdx = (int)tweets[uid].size() - 1;
                auto& latest = tweets[uid][lastIdx];
                heap.push({latest[0], latest[1], uid, lastIdx});
            }
        }

        vector<int> feed;
        while (!heap.empty() && feed.size() < 10) {
            auto curr = heap.top(); heap.pop();
            feed.push_back(curr[1]);  // tweetId
            int idx = curr[3] - 1;
            if (idx >= 0) {
                auto& prev = tweets[curr[2]][idx];
                heap.push({prev[0], prev[1], curr[2], idx});
            }
        }
        return feed;
    }

    void follow(int followerId, int followeeId) {
        follows[followerId].insert(followeeId);
    }

    void unfollow(int followerId, int followeeId) {
        if (follows.count(followerId)) follows[followerId].erase(followeeId);
    }
};
```

**K-Way Merge application:** Each user's tweet list is sorted by time (insertion order). We merge across all followees + self to get the top 10, using a max-heap on timestamp.

**Complexity:**
- `postTweet`: O(1)
- `getNewsFeed`: O(F log F + 10 log F) where F = number of followees. Initial heap = O(F log F). Each of 10 iterations = O(log F).
- `follow`/`unfollow`: O(1) average with `std::unordered_set`

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
A: Mark tweets as deleted with a `bool` flag. Skip deleted tweets during `getNewsFeed` merge.

---

## Related Files

- [K-Way Merge Pattern](../Patterns/K%20Way%20Merge.md)
- [Median Finder Design](./Median%20Finder.md)
- [Interview Problems: Google](../Interview%20Problems/Google.md)

> **Last Updated:** 2026-06-26
