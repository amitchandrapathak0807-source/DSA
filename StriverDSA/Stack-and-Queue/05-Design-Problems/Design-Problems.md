# Stack and Queue — Design Problems

## 1. The Celebrity Problem

### 1. The Celebrity Problem

**Problem Statement:**
In a party of `N` people, a celebrity is defined as someone who is known by everyone else but knows no one else. You are given an `N x N` boolean matrix `M` (the "knows" matrix), where `M[i][j] == 1` means person `i` knows person `j`, and `M[i][j] == 0` means person `i` does not know person `j`. Note `M[i][i]` is typically `0` (a person may be assumed not to "know" themselves for this problem, or it is ignored). Find the index of the celebrity, or return `-1` if no celebrity exists. A valid solution should run better than the naive `O(n^2)` full scan when possible — the optimized approach runs in `O(n)` time using `O(1)` extra space.

**Example:**
- Input:
  ```
  M = [
    [0, 1, 1, 0],
    [0, 0, 1, 0],
    [0, 0, 0, 0],
    [0, 1, 1, 0]
  ]
  ```
  (rows/cols indexed 0..3; M[i][j] = 1 means i knows j)
- Output: `2`
  (Person 2 knows no one — row 2 is all zeros — and everyone else knows person 2 — column 2 is all ones except M[2][2].)

**Brute Force Approach:**
For every candidate person `i`, check whether every other person `j` knows `i` (i.e., `M[j][i] == 1` for all `j != i`) and that `i` knows no one else (i.e., `M[i][j] == 0` for all `j != i`). This requires checking two conditions for every pair, giving `O(n^2)` time.

```csharp
using System;

public class CelebrityBruteForce
{
    public int FindCelebrity(int[,] M, int n)
    {
        for (int i = 0; i < n; i++)
        {
            bool isCelebrity = true;

            for (int j = 0; j < n; j++)
            {
                if (i == j) continue;

                // i must not know anyone
                // everyone else must know i
                if (M[i, j] == 1 || M[j, i] == 0)
                {
                    isCelebrity = false;
                    break;
                }
            }

            if (isCelebrity)
                return i;
        }

        return -1;
    }
}
```

Time Complexity: `O(n^2)` — for each of the `n` candidates, we scan a row of size `n`.
Space Complexity: `O(1)` — no extra data structures used.

**Optimized Approach:**
Use a two-pointer elimination technique. Maintain two pointers, `top = 0` and `down = n - 1`. Compare the pair: if `top` knows `down`, then `top` cannot be the celebrity (a celebrity knows no one), so move `top` forward. Otherwise, `down` cannot be the celebrity (a celebrity is known by everyone, but `down` does not know `top`... actually if `top` does NOT know `down`, then `down` cannot be the celebrity since not everyone knows `down`), so move `down` backward. Continue until `top >= down`; the remaining index is the only possible candidate. Finally, verify this candidate against every other person in one more linear pass, since the elimination only produces a candidate, not a guaranteed celebrity.

```csharp
using System;

public class CelebrityOptimized
{
    public int FindCelebrity(int[,] M, int n)
    {
        int top = 0, down = n - 1;

        // Step 1: Eliminate candidates using two pointers
        while (top < down)
        {
            if (M[top, down] == 1)
            {
                // top knows down => top can't be celebrity
                top++;
            }
            else
            {
                // top doesn't know down => down can't be celebrity
                down--;
            }
        }

        int candidate = top; // top == down here

        // Step 2: Verify the candidate
        for (int j = 0; j < n; j++)
        {
            if (j == candidate) continue;

            if (M[candidate, j] == 1 || M[j, candidate] == 0)
                return -1; // not a celebrity
        }

        return candidate;
    }
}
```

Time Complexity: `O(n)` — the elimination phase takes `O(n)` (each step advances one pointer), and the verification phase takes `O(n)`.
Space Complexity: `O(1)` — only two pointers used.

**Explanation:**

Dry run of two-pointer elimination on the example matrix:
```
      0  1  2  3
  0 [ 0, 1, 1, 0 ]
  1 [ 0, 0, 1, 0 ]
  2 [ 0, 0, 0, 0 ]
  3 [ 0, 1, 1, 0 ]
```

- Initialize `top = 0`, `down = 3`.
- Compare `M[top, down] = M[0, 3] = 0` → top(0) does NOT know down(3) → down cannot be the celebrity (since 0 does not know 3, not everyone knows 3) → `down--` → `down = 2`.
- Compare `M[top, down] = M[0, 2] = 1` → top(0) knows down(2) → top cannot be the celebrity (a celebrity knows no one) → `top++` → `top = 1`.
- Now `top = 1`, `down = 2`. Compare `M[1, 2] = 1` → 1 knows 2 → top cannot be celebrity → `top++` → `top = 2`.
- Now `top == down == 2`, loop ends. Candidate = `2`.
- Verification pass for candidate 2: check `M[2, 0] = 0`, `M[2, 1] = 0`, `M[2, 3] = 0` (candidate knows no one — good); check `M[0, 2] = 1`, `M[1, 2] = 1`, `M[3, 2] = 1` (everyone knows candidate — good).
- Candidate 2 passes verification, so `2` is returned as the celebrity.

Why elimination is correct: at each comparison between `top` and `down`, at least one of them is provably NOT the celebrity (either `top` knows someone, disqualifying `top`, or `top` does not know `down`, meaning `down` is not known by everyone, disqualifying `down`). Since in every step exactly one candidate is eliminated and a real celebrity (if one exists) is never eliminated by this rule, after `n - 1` eliminations only one possible candidate remains — which must still be verified because the matrix might not contain a celebrity at all.

---

## 2. Design and Implement an LRU (Least Recently Used) Cache

### 2. Design and Implement an LRU Cache

**Problem Statement:**
Design a data structure for a Least Recently Used (LRU) cache with a fixed positive capacity. It should support two operations:
- `get(key)`: Return the value of the key if it exists in the cache, otherwise return `-1`. Accessing a key marks it as "recently used."
- `put(key, value)`: Insert or update the value of the key. If inserting a new key causes the cache to exceed its capacity, evict the least recently used key first.

Both `get` and `put` must run in `O(1)` average time complexity.

**Example:**
- Input (capacity = 2):
  ```
  put(1, 10)
  put(2, 20)
  get(1)        -> returns 10, and marks key 1 as most recently used
  put(3, 30)    -> capacity exceeded, evicts key 2 (least recently used)
  get(2)        -> returns -1 (evicted)
  put(4, 40)    -> evicts key 1 (now least recently used)
  get(1)        -> returns -1 (evicted)
  get(3)        -> returns 30
  get(4)        -> returns 40
  ```
- Output: `10, -1, -1, 30, 40`

**Brute Force Approach:**
Use a `Dictionary<int, int>` for key-value storage plus a `List<int>` to track usage order. On every `get` or `put`, move the accessed key to the end of the list (marking it most recently used) by removing and re-adding it — an `O(n)` operation. When capacity is exceeded, remove the key at the front of the list (least recently used), which is also `O(n)` for the dictionary-independent search but the list removal from front is `O(n)` due to shifting.

```csharp
using System;
using System.Collections.Generic;

public class LRUCacheBruteForce
{
    private readonly int capacity;
    private readonly Dictionary<int, int> map;
    private readonly List<int> usageOrder; // front = least recently used, back = most recently used

    public LRUCacheBruteForce(int capacity)
    {
        this.capacity = capacity;
        map = new Dictionary<int, int>();
        usageOrder = new List<int>();
    }

    public int Get(int key)
    {
        if (!map.ContainsKey(key))
            return -1;

        // Move key to most recently used position
        usageOrder.Remove(key);       // O(n) search + shift
        usageOrder.Add(key);

        return map[key];
    }

    public void Put(int key, int value)
    {
        if (map.ContainsKey(key))
        {
            map[key] = value;
            usageOrder.Remove(key);   // O(n)
            usageOrder.Add(key);
            return;
        }

        if (map.Count >= capacity)
        {
            int lruKey = usageOrder[0];
            usageOrder.RemoveAt(0);   // O(n) shift
            map.Remove(lruKey);
        }

        map[key] = value;
        usageOrder.Add(key);
    }
}
```

Time Complexity: `O(n)` per `get`/`put` due to linear search and shifting in the `List<int>`.
Space Complexity: `O(capacity)` for the dictionary and the list.

**Optimized Approach:**
Combine a `Dictionary<int, LinkedListNode<(int key, int value)>>` with a doubly linked list (`LinkedList<T>` in C#). The linked list maintains usage order: the head is the most recently used, the tail is the least recently used. The dictionary maps each key directly to its node in the linked list, so both lookup and node removal/insertion are `O(1)`.

```csharp
using System;
using System.Collections.Generic;

public class LRUCache
{
    private readonly int capacity;
    private readonly Dictionary<int, LinkedListNode<(int key, int value)>> map;
    private readonly LinkedList<(int key, int value)> order; // front = most recently used, back = least recently used

    public LRUCache(int capacity)
    {
        this.capacity = capacity;
        map = new Dictionary<int, LinkedListNode<(int key, int value)>>();
        order = new LinkedList<(int key, int value)>();
    }

    public int Get(int key)
    {
        if (!map.TryGetValue(key, out var node))
            return -1;

        // Move accessed node to the front (most recently used)
        order.Remove(node);
        order.AddFirst(node);

        return node.Value.value;
    }

    public void Put(int key, int value)
    {
        if (map.TryGetValue(key, out var existingNode))
        {
            order.Remove(existingNode);
            var updatedNode = new LinkedListNode<(int key, int value)>((key, value));
            order.AddFirst(updatedNode);
            map[key] = updatedNode;
            return;
        }

        if (map.Count >= capacity)
        {
            // Evict least recently used (tail of the list)
            var lruNode = order.Last;
            order.RemoveLast();
            map.Remove(lruNode.Value.key);
        }

        var newNode = new LinkedListNode<(int key, int value)>((key, value));
        order.AddFirst(newNode);
        map[key] = newNode;
    }
}
```

Time Complexity: `O(1)` per `get` and `put` operation.
Space Complexity: `O(capacity)` for the dictionary and doubly linked list.

**Explanation:**

Dry run with capacity = 2:

1. `put(1, 10)`: cache empty, insert node `(1,10)` at front. List: `[(1,10)]`. Map: `{1 -> node(1,10)}`.
2. `put(2, 20)`: insert node `(2,20)` at front. List: `[(2,20), (1,10)]`. Map: `{1 -> node(1,10), 2 -> node(2,20)}`.
3. `get(1)`: found in map, node `(1,10)` is removed from its current position and re-added at front. List becomes: `[(1,10), (2,20)]`. Returns `10`.
4. `put(3, 30)`: key 3 not present, and `map.Count (2) >= capacity (2)`, so evict tail — which is `(2,20)`, the least recently used — remove it from the list and the map. List: `[(1,10)]`, Map: `{1 -> node(1,10)}`. Insert new node `(3,30)` at front. List: `[(3,30), (1,10)]`. Map: `{1 -> node(1,10), 3 -> node(3,30)}`.
5. `get(2)`: key 2 no longer in map → returns `-1` (confirms eviction).

This shows why the doubly linked list + dictionary combination achieves O(1): the dictionary gives instant access to any node, and the linked list allows removing/re-inserting that node at the front in constant time without shifting any other elements, unlike an array-based list.

---

## 3. Design and Implement an LFU (Least Frequently Used) Cache

### 3. Design and Implement an LFU Cache

**Problem Statement:**
Design a data structure for a Least Frequently Used (LFU) cache with a fixed positive capacity. It should support:
- `get(key)`: Return the value of the key if it exists, otherwise return `-1`. A successful access increments the key's usage frequency.
- `put(key, value)`: Insert or update the value of the key. If inserting a new key exceeds capacity, evict the least frequently used key. If there is a tie in frequency among multiple keys, evict the least recently used one among them (the standard tie-breaking rule).

Both `get` and `put` should run in `O(1)` average time complexity.

**Example:**
- Input (capacity = 2):
  ```
  put(1, 10)
  put(2, 20)
  get(1)        -> returns 10, freq(1) = 2, freq(2) = 1
  put(3, 30)    -> capacity exceeded; freq(2)=1 is lowest, evict key 2
  get(2)        -> returns -1 (evicted)
  get(3)        -> returns 30, freq(3) = 2
  put(4, 40)    -> capacity exceeded; freq(1)=2 and freq(3)=2 tie, key 1 is least recently used among them, evict key 1
  get(1)        -> returns -1 (evicted)
  get(3)        -> returns 30
  get(4)        -> returns 40
  ```
- Output: `10, -1, 30, -1, 30, 40`

**Brute Force Approach:**
Use a `Dictionary<int, int>` for values and a separate `Dictionary<int, int>` for frequency counts. On eviction, do an `O(n)` linear scan over all keys to find the one with the minimum frequency (breaking ties by insertion/access order tracked separately, e.g., with a simple counter or a list).

```csharp
using System;
using System.Collections.Generic;

public class LFUCacheBruteForce
{
    private readonly int capacity;
    private readonly Dictionary<int, int> values;
    private readonly Dictionary<int, int> freq;
    private readonly Dictionary<int, long> lastUsedTime; // for tie-breaking
    private long clock;

    public LFUCacheBruteForce(int capacity)
    {
        this.capacity = capacity;
        values = new Dictionary<int, int>();
        freq = new Dictionary<int, int>();
        lastUsedTime = new Dictionary<int, long>();
        clock = 0;
    }

    public int Get(int key)
    {
        if (!values.ContainsKey(key))
            return -1;

        freq[key]++;
        lastUsedTime[key] = clock++;
        return values[key];
    }

    public void Put(int key, int value)
    {
        if (capacity <= 0) return;

        if (values.ContainsKey(key))
        {
            values[key] = value;
            freq[key]++;
            lastUsedTime[key] = clock++;
            return;
        }

        if (values.Count >= capacity)
        {
            // Linear scan for least frequently used, tie-break by least recently used
            int evictKey = -1;
            int minFreq = int.MaxValue;
            long oldestTime = long.MaxValue;

            foreach (var k in values.Keys)
            {
                if (freq[k] < minFreq || (freq[k] == minFreq && lastUsedTime[k] < oldestTime))
                {
                    minFreq = freq[k];
                    oldestTime = lastUsedTime[k];
                    evictKey = k;
                }
            }

            values.Remove(evictKey);
            freq.Remove(evictKey);
            lastUsedTime.Remove(evictKey);
        }

        values[key] = value;
        freq[key] = 1;
        lastUsedTime[key] = clock++;
    }
}
```

Time Complexity: `O(n)` per `put` that triggers eviction, due to the linear scan for the minimum-frequency key; `O(1)` for `get`.
Space Complexity: `O(capacity)`.

**Optimized Approach:**
Use the classic two-level HashMap + doubly linked list design:
- `keyToNode`: `Dictionary<int, Node>` mapping each key directly to its node (containing key, value, frequency).
- `freqToList`: `Dictionary<int, LinkedList<Node>>` mapping each frequency count to a doubly linked list of nodes that currently have that frequency, ordered by recency (front = most recently used within that frequency bucket, back = least recently used within that bucket).
- `minFreq`: tracks the smallest frequency currently present, so eviction always pops from the back of `freqToList[minFreq]` in O(1).

On `get`/`put` (update), a key's node is removed from its current frequency bucket and moved to the bucket for `frequency + 1`, added at the front. If the old bucket becomes empty and it was `minFreq`, increment `minFreq`. On `put` of a new key when at capacity, evict the back node of `freqToList[minFreq]` (the least frequently used, and least recently used among ties), then insert the new key with frequency 1 and set `minFreq = 1`.

```csharp
using System;
using System.Collections.Generic;

public class LFUCache
{
    private class Node
    {
        public int Key;
        public int Value;
        public int Freq;

        public Node(int key, int value)
        {
            Key = key;
            Value = value;
            Freq = 1;
        }
    }

    private readonly int capacity;
    private int minFreq;
    private readonly Dictionary<int, LinkedListNode<Node>> keyToNode;
    private readonly Dictionary<int, LinkedList<Node>> freqToList;

    public LFUCache(int capacity)
    {
        this.capacity = capacity;
        minFreq = 0;
        keyToNode = new Dictionary<int, LinkedListNode<Node>>();
        freqToList = new Dictionary<int, LinkedList<Node>>();
    }

    private void Touch(LinkedListNode<Node> node)
    {
        int oldFreq = node.Value.Freq;
        var oldList = freqToList[oldFreq];
        oldList.Remove(node);

        if (oldList.Count == 0)
        {
            freqToList.Remove(oldFreq);
            if (minFreq == oldFreq)
                minFreq++;
        }

        node.Value.Freq++;
        int newFreq = node.Value.Freq;

        if (!freqToList.ContainsKey(newFreq))
            freqToList[newFreq] = new LinkedList<Node>();

        freqToList[newFreq].AddFirst(node);
    }

    public int Get(int key)
    {
        if (!keyToNode.TryGetValue(key, out var node))
            return -1;

        Touch(node);
        return node.Value.Value;
    }

    public void Put(int key, int value)
    {
        if (capacity <= 0) return;

        if (keyToNode.TryGetValue(key, out var existingNode))
        {
            existingNode.Value.Value = value;
            Touch(existingNode);
            return;
        }

        if (keyToNode.Count >= capacity)
        {
            // Evict least frequently used; tie-break by least recently used (tail of minFreq bucket)
            var evictList = freqToList[minFreq];
            var evictNode = evictList.Last;
            evictList.RemoveLast();

            if (evictList.Count == 0)
                freqToList.Remove(minFreq);

            keyToNode.Remove(evictNode.Value.Key);
        }

        var newNode = new Node(key, value);
        var listNode = new LinkedListNode<Node>(newNode);
        keyToNode[key] = listNode;

        if (!freqToList.ContainsKey(1))
            freqToList[1] = new LinkedList<Node>();

        freqToList[1].AddFirst(listNode);
        minFreq = 1;
    }
}
```

Time Complexity: `O(1)` per `get` and `put` operation.
Space Complexity: `O(capacity)` for the two dictionaries and the linked lists they hold.

**Explanation:**

Frequency-bucket design summary: instead of one global recency list (as in LRU), the LFU cache keeps one doubly linked list per frequency value. `freqToList[f]` holds all keys currently accessed exactly `f` times, ordered by recency (front = most recently touched at that frequency, back = least recently touched at that frequency). `keyToNode` gives O(1) access to any key's node so it can be unlinked from its current bucket instantly. `minFreq` always points at the lowest non-empty frequency bucket, so eviction is simply "remove the tail node of `freqToList[minFreq]`" — O(1), and it naturally satisfies the tie-breaking rule (least recently used among least frequently used) because ties live in the same bucket, ordered by recency.

Dry run continuing the example (capacity = 2):

1. `put(1, 10)`: `keyToNode = {1}`, `freqToList[1] = [1]`, `minFreq = 1`.
2. `put(2, 20)`: `keyToNode = {1, 2}`, `freqToList[1] = [2, 1]` (2 added at front), `minFreq = 1`.
3. `get(1)`: `Touch(1)` — remove 1 from `freqToList[1]` → `freqToList[1] = [2]` (not empty, `minFreq` stays 1); add 1 to `freqToList[2] = [1]`. Now `freq[1] = 2`, `freq[2] = 1`. Returns `10`.
4. `put(3, 30)`: capacity (2) reached. Evict from `freqToList[minFreq] = freqToList[1] = [2]` → evict tail = key 2. Remove key 2 from `keyToNode`; `freqToList[1]` becomes empty and is removed. Insert key 3: `freqToList[1] = [3]`, `minFreq = 1`. State: `keyToNode = {1, 3}`, `freqToList[1] = [3]`, `freqToList[2] = [1]`.
5. `get(2)`: key 2 not in `keyToNode` → returns `-1`.
6. `get(3)`: `Touch(3)` — remove from `freqToList[1]` → empties and removed, `minFreq` becomes 2; add 3 to `freqToList[2] = [3, 1]`. Returns `30`.
7. `put(4, 40)`: capacity reached. `minFreq = 2`, `freqToList[2] = [3, 1]`, evict tail = key 1 (least recently used among the tied frequency-2 keys). Remove key 1; `freqToList[2] = [3]`. Insert key 4: `freqToList[1] = [4]`, `minFreq = 1`.
8. `get(1)`: key 1 not present → returns `-1`.
9. `get(3)`: found, returns `30` (and its frequency/bucket updates accordingly).
10. `get(4)`: found, returns `40`.

This matches the expected output and demonstrates how the minFreq pointer and per-frequency doubly linked lists together give O(1) eviction while correctly resolving ties by recency.
