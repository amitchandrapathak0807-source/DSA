# Heaps — Hard Problems

## 1. Design a Twitter-like Feed

### 1. Design a Twitter-like Feed

**Problem Statement:** Design a simplified version of Twitter where users can post tweets, follow other users, unfollow users they follow, and retrieve the 10 most recent tweet IDs in the user's news feed. The news feed is composed of tweets posted by the user themselves and by the people they follow, ordered from most recent to least recent. The design must support:
- `PostTweet(userId, tweetId)` — compose a new tweet.
- `GetNewsFeed(userId)` — retrieve the 10 most recent tweet IDs in the user's news feed.
- `Follow(followerId, followeeId)` — follower starts following followee.
- `Unfollow(followerId, followeeId)` — follower stops following followee.

**Example:**
- Input:
  - `PostTweet(1, 5)`
  - `GetNewsFeed(1)`
  - `Follow(1, 2)`
  - `PostTweet(2, 6)`
  - `GetNewsFeed(1)`
  - `Unfollow(1, 2)`
  - `GetNewsFeed(1)`
- Output:
  - `[5]`
  - `[6, 5]`
  - `[5]`

**Brute Force Approach:** Store every tweet ever posted (by every user) in one big list along with a global increasing timestamp and the author's id. When `GetNewsFeed(userId)` is called, scan the entire tweet list, keep only tweets whose author is `userId` or someone `userId` follows, sort the filtered tweets by timestamp descending, and return the top 10. This re-scans and re-sorts on every single call, which is wasteful but simple.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

public class TwitterBrute
{
    private class Tweet
    {
        public int UserId;
        public int TweetId;
        public int Timestamp;
    }

    private readonly List<Tweet> allTweets = new List<Tweet>();
    private readonly Dictionary<int, HashSet<int>> following = new Dictionary<int, HashSet<int>>();
    private int clock = 0;

    public void PostTweet(int userId, int tweetId)
    {
        allTweets.Add(new Tweet { UserId = userId, TweetId = tweetId, Timestamp = clock++ });
        if (!following.ContainsKey(userId))
            following[userId] = new HashSet<int>();
    }

    public List<int> GetNewsFeed(int userId)
    {
        var followees = following.ContainsKey(userId) ? following[userId] : new HashSet<int>();

        return allTweets
            .Where(t => t.UserId == userId || followees.Contains(t.UserId))
            .OrderByDescending(t => t.Timestamp)
            .Take(10)
            .Select(t => t.TweetId)
            .ToList();
    }

    public void Follow(int followerId, int followeeId)
    {
        if (!following.ContainsKey(followerId))
            following[followerId] = new HashSet<int>();
        if (followerId != followeeId)
            following[followerId].Add(followeeId);
    }

    public void Unfollow(int followerId, int followeeId)
    {
        if (following.ContainsKey(followerId))
            following[followerId].Remove(followeeId);
    }
}
```

Time Complexity: `PostTweet` is O(1); `Follow`/`Unfollow` are O(1) amortized (HashSet operations); `GetNewsFeed` is O(T log T) where T is the total number of tweets ever posted, because every call filters and sorts the entire tweet history.
Space Complexity: O(T + U) for storing all tweets and the follow graph across U users.

**Optimized Approach:** Keep a per-user linked list (most recent tweet at the head) of that user's own tweets, each node storing `(timestamp, tweetId)`. To build a news feed we only need the most recent tweets from the user themselves and each followee — this is a classic "merge K sorted lists" problem, solved with a min-heap (or max-heap by timestamp) that initially holds only the head (most recent) tweet-node from each of the up-to-(1 + followees) relevant lists. Repeatedly pop the maximum timestamp, push it to the result, and if that node has a "next" (older) tweet, push that next node into the heap. Stop after collecting 10 tweets or the heap empties. This avoids ever looking at a user's older tweets unless needed.

```csharp
using System;
using System.Collections.Generic;

public class Twitter
{
    private class TweetNode
    {
        public int TweetId;
        public int Timestamp;
        public TweetNode Next; // points to the user's next-older tweet
    }

    private readonly Dictionary<int, TweetNode> userTweetHead = new Dictionary<int, TweetNode>();
    private readonly Dictionary<int, HashSet<int>> following = new Dictionary<int, HashSet<int>>();
    private int clock = 0;

    private void EnsureUser(int userId)
    {
        if (!following.ContainsKey(userId))
            following[userId] = new HashSet<int>();
    }

    public void PostTweet(int userId, int tweetId)
    {
        EnsureUser(userId);
        var node = new TweetNode
        {
            TweetId = tweetId,
            Timestamp = clock++,
            Next = userTweetHead.ContainsKey(userId) ? userTweetHead[userId] : null
        };
        userTweetHead[userId] = node; // new head = most recent tweet
    }

    public List<int> GetNewsFeed(int userId)
    {
        EnsureUser(userId);

        // Max-heap on timestamp: PriorityQueue in C# is a min-heap, so negate the priority.
        var heap = new PriorityQueue<TweetNode, int>();

        if (userTweetHead.ContainsKey(userId))
            heap.Enqueue(userTweetHead[userId], -userTweetHead[userId].Timestamp);

        foreach (var followeeId in following[userId])
        {
            if (userTweetHead.ContainsKey(followeeId))
                heap.Enqueue(userTweetHead[followeeId], -userTweetHead[followeeId].Timestamp);
        }

        var result = new List<int>();
        while (heap.Count > 0 && result.Count < 10)
        {
            var node = heap.Dequeue();
            result.Add(node.TweetId);
            if (node.Next != null)
                heap.Enqueue(node.Next, -node.Next.Timestamp);
        }

        return result;
    }

    public void Follow(int followerId, int followeeId)
    {
        EnsureUser(followerId);
        EnsureUser(followeeId);
        if (followerId != followeeId)
            following[followerId].Add(followeeId);
    }

    public void Unfollow(int followerId, int followeeId)
    {
        EnsureUser(followerId);
        following[followerId].Remove(followeeId);
    }
}
```

Time Complexity: `PostTweet`, `Follow`, `Unfollow` are O(1). `GetNewsFeed` is O((1 + F) log(1 + F) + 10 log(1 + F)) where F is the number of followees — the heap never holds more than `1 + F` elements at a time, and we do at most `10 + F` push/pop operations, each O(log(1 + F)). This is far better than sorting the entire tweet history.
Space Complexity: O(U + T) to store the follow graph and each user's tweet linked list; the heap itself only uses O(F) auxiliary space per call.

**Explanation:** The key insight is that we never need to compare *all* tweets — we only ever need the "current frontier" (the most recent still-unconsidered tweet) of each relevant user. The heap always holds exactly one candidate tweet per followee/self; whenever we consume the newest candidate from some user, we immediately push that same user's next-most-recent tweet as the new candidate. This mirrors merging K sorted linked lists with a heap, giving us the top 10 overall without sorting everything.

---

## 2. Connect N Ropes with Minimum Cost

### 2. Connect N Ropes with Minimum Cost

**Problem Statement:** You are given `N` ropes of different lengths. You need to connect these ropes into one rope. The cost to connect two ropes is equal to the sum of their lengths. Find the minimum total cost to connect all the ropes into a single rope.

**Example:**
- Input: `ropes = [4, 3, 2, 6]`
- Output: `29`
  - Explanation: Connect `2 + 3 = 5` (cost 5, ropes now `[4, 5, 6]`), connect `4 + 5 = 9` (cost 9, ropes now `[9, 6]`), connect `9 + 6 = 15` (cost 15). Total = `5 + 9 + 15 = 29`.

**Brute Force Approach:** At each step, scan the entire current list of rope lengths to find the two smallest, remove them, connect them (add their sum back into the list), and add the connection cost to a running total. Repeat until only one rope remains. Finding the two minimums by scanning is O(N) per step, and there are N-1 steps.

```csharp
using System;
using System.Collections.Generic;

public class ConnectRopesBrute
{
    public int MinCost(int[] ropes)
    {
        List<int> lengths = new List<int>(ropes);
        int totalCost = 0;

        while (lengths.Count > 1)
        {
            // find index of smallest
            int firstMinIdx = 0;
            for (int i = 1; i < lengths.Count; i++)
                if (lengths[i] < lengths[firstMinIdx]) firstMinIdx = i;
            int first = lengths[firstMinIdx];
            lengths.RemoveAt(firstMinIdx);

            // find index of second smallest
            int secondMinIdx = 0;
            for (int i = 1; i < lengths.Count; i++)
                if (lengths[i] < lengths[secondMinIdx]) secondMinIdx = i;
            int second = lengths[secondMinIdx];
            lengths.RemoveAt(secondMinIdx);

            int cost = first + second;
            totalCost += cost;
            lengths.Add(cost);
        }

        return totalCost;
    }
}
```

Time Complexity: O(N^2) — each of the N-1 merge steps scans the list (up to O(N)) twice to find the two minimums, plus an O(N) removal.
Space Complexity: O(N) for the working list.

**Optimized Approach:** This is exactly the greedy strategy used in Huffman coding. Push all rope lengths into a min-heap. Repeatedly pop the two smallest elements, add their sum to the total cost, and push the sum back into the heap. Continue until only one element remains in the heap. Because the heap always gives the two smallest values in O(log N) time, this greedy approach is provably optimal (always merging the cheapest options first minimizes the total accumulated cost).

```csharp
using System;
using System.Collections.Generic;

public class ConnectRopes
{
    public int MinCost(int[] ropes)
    {
        // C#'s PriorityQueue is a min-heap by default; use rope length as its own priority.
        var minHeap = new PriorityQueue<int, int>();
        foreach (var rope in ropes)
            minHeap.Enqueue(rope, rope);

        int totalCost = 0;

        while (minHeap.Count > 1)
        {
            int first = minHeap.Dequeue();
            int second = minHeap.Dequeue();
            int cost = first + second;
            totalCost += cost;
            minHeap.Enqueue(cost, cost);
        }

        return totalCost;
    }
}
```

Time Complexity: O(N log N) — building the heap is O(N) (or O(N log N) with repeated enqueues), and each of the N-1 merge steps does two O(log N) dequeues and one O(log N) enqueue.
Space Complexity: O(N) for the heap.

**Explanation (Dry run on `[4, 3, 2, 6]`):**

Initial min-heap contents: `[2, 3, 4, 6]`

1. Pop two smallest: `2` and `3` → cost = `5`. Running total = `5`. Push `5` back. Heap: `[4, 5, 6]`.
2. Pop two smallest: `4` and `5` → cost = `9`. Running total = `5 + 9 = 14`. Push `9` back. Heap: `[6, 9]`.
3. Pop two smallest: `6` and `9` → cost = `15`. Running total = `14 + 15 = 29`. Push `15` back. Heap: `[15]` — only one element left, stop.

Final minimum total cost = **29**, matching the connections `2+3=5`, `4+5=9`, `9+6=15`, summing to `5+9+15=29`.

---

## 3. Kth Largest Element in a Stream of Running Integers

### 3. Kth Largest Element in a Stream of Running Integers

**Problem Statement:** Design a class that continuously receives integers one at a time from a stream and, after each new integer is added, can report the Kth largest element seen so far (considering all elements added up to that point, including duplicates). The class should support:
- A constructor that takes `k` and an initial array of numbers.
- `Add(val)` — adds `val` to the stream and returns the current Kth largest element.

**Example:**
- Input:
  - `KthLargest(3, [4, 5, 8, 2])`
  - `Add(3)` → returns `4`
  - `Add(5)` → returns `5`
  - `Add(10)` → returns `5`
  - `Add(9)` → returns `8`
  - `Add(4)` → returns `8`
- Output: `4, 5, 5, 8, 8`

**Brute Force Approach:** Maintain a plain list of all numbers seen so far. On every `Add(val)`, append `val` to the list, sort the entire list in descending order, and return the element at index `k-1`. Sorting on every insertion is expensive.

```csharp
using System;
using System.Collections.Generic;

public class KthLargestBrute
{
    private readonly List<int> numbers;
    private readonly int k;

    public KthLargestBrute(int k, int[] nums)
    {
        this.k = k;
        numbers = new List<int>(nums);
    }

    public int Add(int val)
    {
        numbers.Add(val);
        numbers.Sort((a, b) => b.CompareTo(a)); // descending
        return numbers[k - 1];
    }
}
```

Time Complexity: O(N log N) per `Add` call, where N is the number of elements seen so far, because the whole list is re-sorted each time.
Space Complexity: O(N) to store all elements.

**Optimized Approach:** Maintain a min-heap that is kept at size exactly `K`, containing the K largest elements seen so far. The smallest element in this heap (the root) is always the current Kth largest overall. On `Add(val)`: push `val` into the heap; if the heap size exceeds `K`, pop the smallest (discarding an element that is no longer among the top K). The root of the heap after this is the answer.

```csharp
using System;
using System.Collections.Generic;

public class KthLargest
{
    private readonly int k;
    private readonly PriorityQueue<int, int> minHeap; // holds the K largest elements seen so far

    public KthLargest(int k, int[] nums)
    {
        this.k = k;
        minHeap = new PriorityQueue<int, int>();

        foreach (var num in nums)
            AddToHeap(num);
    }

    public int Add(int val)
    {
        AddToHeap(val);
        return minHeap.Peek();
    }

    private void AddToHeap(int val)
    {
        minHeap.Enqueue(val, val);
        if (minHeap.Count > k)
            minHeap.Dequeue(); // remove the smallest, keeping only the K largest
    }
}
```

Time Complexity: O(log K) per `Add` call (one enqueue and at most one dequeue on a heap bounded to size K). Building from the initial array costs O(N log K).
Space Complexity: O(K), since the heap never grows beyond K elements.

**Explanation:** Because the heap is capped at size K and always discards its current minimum whenever it overflows, it is guaranteed to contain exactly the K largest values seen so far (ties broken arbitrarily but correctly by value). Its root (minimum of these K values) is therefore precisely the Kth largest value overall — we never need to look at, or sort, the full history of the stream, only maintain this small bounded structure.

---

## 4. Maximum Sum Combination

### 4. Maximum Sum Combination

**Problem Statement:** Given two integer arrays `A` and `B`, each of size `N`, and an integer `K`, find the `K` pairs `(A[i], B[j])` — one element from each array — whose sums `A[i] + B[j]` are the largest, and return these K maximum sums in descending order.

**Example:**
- Input: `A = [3, 2], B = [1, 4], K = 2`
- Output: `[7, 6]`
  - Explanation: All possible sums are `3+1=4`, `3+4=7`, `2+1=3`, `2+4=6`. The top 2 sums are `7` and `6`.

**Brute Force Approach:** Compute the sum of every possible pair `(A[i], B[j])` for all `i, j` — there are `N*N` such pairs. Store all sums in a list, sort the list in descending order, and take the first `K` values.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

public class MaxSumCombinationBrute
{
    public List<int> MaxSumCombinations(int[] a, int[] b, int k)
    {
        var allSums = new List<int>();
        for (int i = 0; i < a.Length; i++)
            for (int j = 0; j < b.Length; j++)
                allSums.Add(a[i] + b[j]);

        allSums.Sort((x, y) => y.CompareTo(x)); // descending
        return allSums.Take(k).ToList();
    }
}
```

Time Complexity: O(N^2 log(N^2)) to generate all N^2 pairwise sums and sort them.
Space Complexity: O(N^2) to store all pairwise sums.

**Optimized Approach:** First sort both arrays `A` and `B` in descending order — this guarantees the overall maximum possible sum is `A[0] + B[0]`. Use a max-heap of tuples `(sum, i, j)` seeded with `(A[0]+B[0], 0, 0)`, plus a `HashSet<(int,int)>` to avoid revisiting the same `(i, j)` pair. Repeatedly pop the largest sum, add it to the result, and push its two "neighbors" `(i+1, j)` and `(i, j+1)` (if within bounds and not already visited/queued). Stop once K sums have been extracted. This never materializes all N^2 combinations — it only explores promising neighbors of already-popped maximal pairs.

```csharp
using System;
using System.Collections.Generic;

public class MaxSumCombination
{
    public List<int> MaxSumCombinations(int[] a, int[] b, int k)
    {
        int n = a.Length;
        var sortedA = (int[])a.Clone();
        var sortedB = (int[])b.Clone();
        Array.Sort(sortedA); Array.Reverse(sortedA); // descending
        Array.Sort(sortedB); Array.Reverse(sortedB); // descending

        // Max-heap on sum: PriorityQueue is a min-heap, so negate the priority.
        var maxHeap = new PriorityQueue<(int sum, int i, int j), int>();
        var visited = new HashSet<(int, int)>();

        int initialSum = sortedA[0] + sortedB[0];
        maxHeap.Enqueue((initialSum, 0, 0), -initialSum);
        visited.Add((0, 0));

        var result = new List<int>();

        while (result.Count < k && maxHeap.Count > 0)
        {
            var (sum, i, j) = maxHeap.Dequeue();
            result.Add(sum);

            // neighbor (i+1, j): advance in A
            if (i + 1 < n && !visited.Contains((i + 1, j)))
            {
                int nextSum = sortedA[i + 1] + sortedB[j];
                maxHeap.Enqueue((nextSum, i + 1, j), -nextSum);
                visited.Add((i + 1, j));
            }

            // neighbor (i, j+1): advance in B
            if (j + 1 < n && !visited.Contains((i, j + 1)))
            {
                int nextSum = sortedA[i] + sortedB[j + 1];
                maxHeap.Enqueue((nextSum, i, j + 1), -nextSum);
                visited.Add((i, j + 1));
            }
        }

        return result;
    }
}
```

Time Complexity: O(N log N) to sort both arrays, plus O(K log K) for the heap operations (at most `2K` pushes and `K` pops, each O(log K) since the heap holds at most O(K) elements at a time). Overall O(N log N + K log K).
Space Complexity: O(N) for the sorted arrays plus O(K) for the heap and visited set.

**Explanation:** Sorting both arrays descending ensures the largest achievable sum is always at index `(0,0)`. Every time we pop a pair `(i,j)`, the next-largest candidates involving that "row" or "column" can only be `(i+1,j)` or `(i,j+1)` — moving to a smaller value in one array while holding the other fixed — because arrays are sorted, any pair with a larger index in both dimensions simultaneously cannot exceed a sum already reachable via one of these two neighbors. The visited set prevents the same `(i,j)` from being enqueued twice (since `(i+1,j)` might be reached both from `(i,j)` and `(i+1,j-1)`).

---

## 5. Find Median from a Data Stream

### 5. Find Median from a Data Stream

**Problem Statement:** Design a data structure that supports adding integers from a continuous data stream and finding the median of all elements added so far, at any point in time, efficiently. The class should support:
- `AddNum(num)` — adds an integer to the data structure.
- `FindMedian()` — returns the median of all elements added so far. If the count is even, return the average of the two middle elements; if odd, return the single middle element.
- Target complexity: O(log n) for `AddNum` and O(1) for `FindMedian`.

**Example:**
- Input:
  - `AddNum(5)`
  - `AddNum(15)`
  - `FindMedian()` → `10.0`
  - `AddNum(1)`
  - `FindMedian()` → `5.0`
  - `AddNum(3)`
  - `FindMedian()` → `4.0`
- Output: `10.0, 5.0, 4.0`

**Brute Force Approach:** Maintain a plain, unsorted list of all numbers added so far. On every `FindMedian()` call, sort a copy of the list and compute the median from the middle element(s). This wastes time re-sorting on every query (or alternatively, keep the list sorted at insertion time via binary-search + insert, which still costs O(n) per insertion due to shifting elements).

```csharp
using System;
using System.Collections.Generic;

public class MedianFinderBrute
{
    private readonly List<int> numbers = new List<int>();

    public void AddNum(int num)
    {
        numbers.Add(num);
    }

    public double FindMedian()
    {
        var sorted = new List<int>(numbers);
        sorted.Sort();
        int n = sorted.Count;

        if (n == 0) throw new InvalidOperationException("No elements added yet.");

        if (n % 2 == 1)
            return sorted[n / 2];
        else
            return (sorted[n / 2 - 1] + sorted[n / 2]) / 2.0;
    }
}
```

Time Complexity: `AddNum` is O(1), but `FindMedian` is O(n log n) because it sorts all elements on every call.
Space Complexity: O(n) to store all elements.

**Optimized Approach:** Maintain two heaps: a max-heap (`lower`) holding the smaller half of the numbers, and a min-heap (`upper`) holding the larger half. Keep them balanced so that `lower` has either the same number of elements as `upper`, or exactly one more. On `AddNum(num)`: push into `lower` first, then move `lower`'s max into `upper` to maintain the heap invariant (every element in `lower` ≤ every element in `upper`); if `upper` grows larger than `lower`, move `upper`'s min back into `lower`. `FindMedian()` then just looks at the top(s) of the two heaps: if `lower` has one more element, its top is the median; if equal sizes, the median is the average of both tops — both O(1) peek operations.

```csharp
using System;
using System.Collections.Generic;

public class MedianFinder
{
    // Max-heap for the lower half: negate priority since PriorityQueue is a min-heap.
    private readonly PriorityQueue<int, int> lower = new PriorityQueue<int, int>();
    // Min-heap for the upper half.
    private readonly PriorityQueue<int, int> upper = new PriorityQueue<int, int>();

    public void AddNum(int num)
    {
        // Step 1: always add to lower (max-heap) first.
        lower.Enqueue(num, -num);

        // Step 2: move lower's max into upper to maintain lower <= upper invariant.
        int lowerMax = lower.Dequeue();
        upper.Enqueue(lowerMax, lowerMax);

        // Step 3: rebalance so lower has equal count or exactly one more than upper.
        if (upper.Count > lower.Count)
        {
            int upperMin = upper.Dequeue();
            lower.Enqueue(upperMin, -upperMin);
        }
    }

    public double FindMedian()
    {
        if (lower.Count == 0 && upper.Count == 0)
            throw new InvalidOperationException("No elements added yet.");

        if (lower.Count > upper.Count)
            return lower.Peek();

        return (lower.Peek() + upper.Peek()) / 2.0;
    }
}
```

Time Complexity: `AddNum` is O(log n) due to at most three heap push/pop operations, each O(log n). `FindMedian` is O(1) since it only peeks at heap roots.
Space Complexity: O(n) to store all elements across both heaps.

**Explanation (Dry run of two-heap technique on stream `5, 15, 1, 3`):**

Start: `lower` (max-heap, smaller half) = `{}`, `upper` (min-heap, larger half) = `{}`.

1. **AddNum(5):**
   - Push 5 into `lower` → `lower = {5}`.
   - Move `lower`'s max (5) into `upper` → `lower = {}`, `upper = {5}`.
   - Rebalance: `upper.Count (1) > lower.Count (0)`, so move `upper`'s min (5) back → `lower = {5}`, `upper = {}`.
   - State: `lower = {5}`, `upper = {}`.

2. **AddNum(15):**
   - Push 15 into `lower` → `lower = {5, 15}` (max-heap top = 15).
   - Move `lower`'s max (15) into `upper` → `lower = {5}`, `upper = {15}`.
   - Rebalance: `upper.Count (1) == lower.Count (1)`, no move needed.
   - State: `lower = {5}`, `upper = {15}`.
   - **FindMedian()** → sizes equal → `(5 + 15) / 2.0 = 10.0`. ✓ matches expected output.

3. **AddNum(1):**
   - Push 1 into `lower` → `lower = {5, 1}` (max-heap top = 5).
   - Move `lower`'s max (5) into `upper` → `lower = {1}`, `upper = {5, 15}` (min-heap top = 5).
   - Rebalance: `upper.Count (2) > lower.Count (1)`, move `upper`'s min (5) back → `lower = {1, 5}` (top = 5), `upper = {15}`.
   - State: `lower = {1, 5}`, `upper = {15}`.
   - **FindMedian()** → `lower.Count (2) > upper.Count (1)` → median = `lower.Peek() = 5`. ✓ matches expected output (5.0).

4. **AddNum(3):**
   - Push 3 into `lower` → `lower = {1, 5, 3}` (max-heap top = 5).
   - Move `lower`'s max (5) into `upper` → `lower = {1, 3}` (top = 3), `upper = {15, 5}` (min-heap top = 5).
   - Rebalance: `upper.Count (2) == lower.Count (2)`, no move needed.
   - State: `lower = {1, 3}`, `upper = {5, 15}`.
   - **FindMedian()** → sizes equal → `(lower.Peek() + upper.Peek()) / 2.0 = (3 + 5) / 2.0 = 4.0`. ✓ matches expected output.

At every step, `lower` contains the smaller half of all numbers seen so far (root = largest of the small half) and `upper` contains the larger half (root = smallest of the large half), so the median is always obtainable from the two heap roots in O(1) without ever sorting the full stream.
