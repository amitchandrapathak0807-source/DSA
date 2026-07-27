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

**Logic (Steps):**
1. Loop over every candidate `i` from `0` to `n-1`.
2. For each candidate, assume `isCelebrity = true`, then loop over every other person `j`.
3. If `M[i, j] == 1` (candidate knows someone) or `M[j, i] == 0` (someone doesn't know the candidate), mark `isCelebrity = false` and break.
4. If the inner loop completes without disqualifying `i`, return `i` as the celebrity.
5. If no candidate passes, return `-1`.

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

**Walkthrough:** Using `M` from the Example, try candidate `i = 0`: `j = 1`, `M[0,1] = 1` (0 knows 1) → disqualified immediately. Candidate `i = 1`: `M[1,2] = 1` → disqualified. Candidate `i = 2`: `j = 0`, `M[2,0] = 0` and `M[0,2] = 1` (ok); `j = 1`, `M[2,1] = 0` and `M[1,2] = 1` (ok); `j = 3`, `M[2,3] = 0` and `M[3,2] = 1` (ok) — no disqualification, so `isCelebrity` stays `true` and `2` is returned, matching the expected Output of `2`.

---

**Optimized Approach:**
Use a two-pointer elimination technique. Maintain two pointers, `top = 0` and `down = n - 1`. Compare the pair: if `top` knows `down`, then `top` cannot be the celebrity (a celebrity knows no one), so move `top` forward. Otherwise, `down` cannot be the celebrity (a celebrity is known by everyone, but `down` does not know `top`... actually if `top` does NOT know `down`, then `down` cannot be the celebrity since not everyone knows `down`), so move `down` backward. Continue until `top >= down`; the remaining index is the only possible candidate. Finally, verify this candidate against every other person in one more linear pass, since the elimination only produces a candidate, not a guaranteed celebrity.

**Logic (Steps):**
1. Initialize `top = 0` and `down = n - 1`.
2. While `top < down`: compare `M[top, down]`. If it's `1` (top knows down), `top` is disqualified, so `top++`.
3. Otherwise (top does not know down), `down` is disqualified, so `down--`.
4. When `top >= down`, the surviving index is the sole candidate.
5. Run one verification pass over all `j != candidate`, checking `M[candidate, j] == 0` and `M[j, candidate] == 1` for every `j`.
6. Return the candidate if verification passes, otherwise return `-1`.

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

**Walkthrough:** On the example matrix, start `top = 0`, `down = 3`. `M[0,3] = 0` → top does not know down → down is disqualified → `down = 2`. `M[0,2] = 1` → top knows down → top is disqualified → `top = 1`. Now `M[1,2] = 1` → top knows down → `top = 2`. Loop ends since `top == down == 2`; candidate = `2`. Verification: `M[2,0]=0`, `M[2,1]=0`, `M[2,3]=0` (candidate knows no one) and `M[0,2]=1`, `M[1,2]=1`, `M[3,2]=1` (everyone knows candidate) — all checks pass, so `2` is returned, matching the expected Output. Correctness relies on each comparison eliminating exactly one non-celebrity (never the real celebrity), leaving one candidate after at most `n-1` steps, which is then verified.

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

**Logic (Steps):**
1. `Get(key)`: if `key` is not in `map`, return `-1`. Otherwise remove it from `usageOrder` and re-add it at the end (marks most recently used), then return `map[key]`.
2. `Put(key, value)`: if `key` already exists, update `map[key]`, then remove and re-add it at the end of `usageOrder`.
3. If `key` is new and `map.Count >= capacity`, read `usageOrder[0]` (the least recently used key), remove it from `usageOrder` and `map`.
4. Insert the new key into `map` and append it to the end of `usageOrder`.

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

**Walkthrough:** With capacity = 2: `put(1,10)` → `map={1:10}`, `usageOrder=[1]`. `put(2,20)` → `map={1:10,2:20}`, `usageOrder=[1,2]`. `get(1)` → found, move 1 to end → `usageOrder=[2,1]`, returns `10`. `put(3,30)` → new key, `map.Count(2) >= capacity(2)`, evict `usageOrder[0]=2` from both structures → `map={1:10}`, `usageOrder=[1]`; insert 3 → `map={1:10,3:30}`, `usageOrder=[1,3]`. `get(2)` → not in map, returns `-1`, matching the expected trace.

---

**Optimized Approach:**
Combine a `Dictionary<int, LinkedListNode<(int key, int value)>>` with a doubly linked list (`LinkedList<T>` in C#). The linked list maintains usage order: the head is the most recently used, the tail is the least recently used. The dictionary maps each key directly to its node in the linked list, so both lookup and node removal/insertion are `O(1)`.

**Logic (Steps):**
1. `Get(key)`: look up `map`; if absent return `-1`. If present, unlink the node from `order` and re-insert it with `AddFirst` (marks most recently used), then return its value.
2. `Put(key, value)` for an existing key: remove old node from `order`, create/insert a fresh node with the new value at the front, and update `map`.
3. `Put(key, value)` for a new key at full capacity: read `order.Last` (the least recently used node), `RemoveLast()` from `order`, and remove its key from `map`.
4. Create a new node for `(key, value)`, insert it at the front of `order` with `AddFirst`, and register it in `map`.

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

**Walkthrough:** Capacity = 2. `put(1,10)`: insert node `(1,10)` at front → List: `[(1,10)]`, Map: `{1}`. `put(2,20)`: insert at front → List: `[(2,20),(1,10)]`, Map: `{1,2}`. `get(1)`: found, unlink and `AddFirst` → List: `[(1,10),(2,20)]`, returns `10`. `put(3,30)`: new key, `map.Count(2) >= capacity(2)`, evict `order.Last = (2,20)` → List: `[(1,10)]`, Map: `{1}`; insert `(3,30)` at front → List: `[(3,30),(1,10)]`, Map: `{1,3}`. `get(2)`: key 2 absent from map → returns `-1`, matching the expected trace. The dictionary gives O(1) node access while the linked list reorders/evicts in O(1) without shifting elements.

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

**Logic (Steps):**
1. `Get(key)`: if absent return `-1`; otherwise increment `freq[key]`, stamp `lastUsedTime[key] = clock++`, and return the value.
2. `Put(key, value)` for an existing key: update the value, increment its frequency, and refresh its timestamp.
3. `Put(key, value)` for a new key at full capacity: scan every key in `values`, tracking the one with the smallest `freq` (breaking ties with the smallest `lastUsedTime`), and remove that key from `values`, `freq`, and `lastUsedTime`.
4. Insert the new key with `freq = 1` and `lastUsedTime = clock++`.

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

**Walkthrough:** Capacity = 2. `put(1,10)` → `values={1:10}`, `freq={1:1}`. `put(2,20)` → `values={1:10,2:20}`, `freq={1:1,2:1}`. `get(1)` → `freq[1]=2`, `lastUsedTime[1]` updated, returns `10`. `put(3,30)` → capacity full, scan finds key 2 has `freq=1` (smallest), evict it → `values={1:10}`. Insert 3 → `values={1:10,3:30}`, `freq={1:2,3:1}`. `get(2)` → absent, returns `-1`, matching the expected trace of `10, -1, ...`.

---

**Optimized Approach:**
Use the classic two-level HashMap + doubly linked list design:
- `keyToNode`: `Dictionary<int, Node>` mapping each key directly to its node (containing key, value, frequency).
- `freqToList`: `Dictionary<int, LinkedList<Node>>` mapping each frequency count to a doubly linked list of nodes that currently have that frequency, ordered by recency (front = most recently used within that frequency bucket, back = least recently used within that bucket).
- `minFreq`: tracks the smallest frequency currently present, so eviction always pops from the back of `freqToList[minFreq]` in O(1).

On `get`/`put` (update), a key's node is removed from its current frequency bucket and moved to the bucket for `frequency + 1`, added at the front. If the old bucket becomes empty and it was `minFreq`, increment `minFreq`. On `put` of a new key when at capacity, evict the back node of `freqToList[minFreq]` (the least frequently used, and least recently used among ties), then insert the new key with frequency 1 and set `minFreq = 1`.

**Logic (Steps):**
1. `Touch(node)` (shared by `Get` and update-`Put`): remove the node from `freqToList[oldFreq]`; if that bucket becomes empty, delete it and bump `minFreq` if it matched `oldFreq`.
2. Increment `node.Value.Freq`, then insert the node at the front of `freqToList[newFreq]` (creating the bucket if needed).
3. `Get(key)`: if `key` is absent return `-1`; otherwise call `Touch` on its node and return the value.
4. `Put(key, value)` for an existing key: update the value and call `Touch`.
5. `Put(key, value)` for a new key at full capacity: evict `freqToList[minFreq].Last` (least frequently used, least recently used among ties), removing it from both the bucket and `keyToNode`.
6. Insert the new node into `freqToList[1]` at the front, register it in `keyToNode`, and set `minFreq = 1`.

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

**Walkthrough:** Capacity = 2. `put(1,10)`: `freqToList[1]=[1]`, `minFreq=1`. `put(2,20)`: `freqToList[1]=[2,1]`. `get(1)`: `Touch` moves 1 out of bucket 1 (`[2]` remains, `minFreq` stays 1) into bucket 2 (`freqToList[2]=[1]`), returns `10`. `put(3,30)`: capacity full, evict tail of `freqToList[minFreq=1]=[2]` → key 2 removed; insert 3 into bucket 1 → `freqToList[1]=[3]`. `get(2)`: absent → returns `-1`. `get(3)`: `Touch` moves 3 to bucket 2, emptying bucket 1 so `minFreq` becomes 2 → `freqToList[2]=[3,1]`, returns `30`. `put(4,40)`: capacity full, evict tail of `freqToList[2]=[3,1]` → key 1 (least recently used among the frequency-2 tie) removed; insert 4 into bucket 1, `minFreq=1`. `get(1)` → `-1`. `get(3)` → `30`. `get(4)` → `40`. This sequence `10, -1, 30, -1, 30, 40` matches the expected Output, confirming the minFreq pointer and per-frequency buckets correctly resolve ties by recency.
