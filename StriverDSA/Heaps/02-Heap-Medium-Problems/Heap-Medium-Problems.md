# Heaps — Medium Problems

## 1. Kth Largest Element in an Array

**Problem Statement:**
Given an unsorted array `arr` of `n` integers and an integer `k`, find the `k`th largest element in the array. Note that it is the `k`th largest element in sorted order, not the `k`th distinct element.

**Example:**
- Input: `arr = [3,2,1,5,6,4], k = 2`
- Output: `5`
- Explanation: Sorted descending the array is `[6,5,4,3,2,1]`. The 2nd largest element is `5`.

**Brute Force Approach:** Sort the array in ascending order and pick the element at index `n - k`. O(n log n).

```csharp
public class Solution
{
    public int FindKthLargestBrute(int[] arr, int k)
    {
        int[] copy = (int[])arr.Clone();
        Array.Sort(copy);
        return copy[copy.Length - k];
    }
}
```
Time Complexity: O(n log n) — dominated by sorting.
Space Complexity: O(n) — for the cloned array (O(1) extra if sorting in place is acceptable).

**Optimized Approach:** Maintain a min-heap of size `k`. Push every element; whenever the heap size exceeds `k`, pop the smallest. At the end the heap's top (minimum) is the `k`th largest element, because the heap holds exactly the `k` largest elements seen so far, and the smallest of those `k` is the `k`th largest overall.

```csharp
using System.Collections.Generic;

public class Solution
{
    public int FindKthLargest(int[] arr, int k)
    {
        // Min-heap: PriorityQueue<element, priority> — priority = value itself
        PriorityQueue<int, int> minHeap = new PriorityQueue<int, int>();

        foreach (int num in arr)
        {
            minHeap.Enqueue(num, num);
            if (minHeap.Count > k)
            {
                minHeap.Dequeue(); // remove smallest, keeping heap size == k
            }
        }

        return minHeap.Peek(); // smallest among the k largest = kth largest
    }
}
```
Time Complexity: O(n log k) — each of the n elements does at most one push and one pop on a heap of size at most k.
Space Complexity: O(k) — the heap never holds more than k elements.

**Explanation:**
Dry run of the size-k min-heap technique on `arr = [3,2,1,5,6,4], k = 2`:

| Step | Element | Action | Heap contents after action |
|------|---------|--------|-----------------------------|
| 1 | 3 | push 3 (size 1 ≤ k) | `{3}` |
| 2 | 2 | push 2 (size 2 ≤ k) | `{2,3}` |
| 3 | 1 | push 1 → size 3 > k → pop min (1) | `{2,3}` |
| 4 | 5 | push 5 → size 3 > k → pop min (2) | `{3,5}` |
| 5 | 6 | push 6 → size 3 > k → pop min (3) | `{5,6}` |
| 6 | 4 | push 4 → size 3 > k → pop min (4) | `{5,6}` |

Final heap = `{5,6}`, whose top (minimum) is `5`, which matches the expected answer — the 2nd largest element in the array.

---

## 2. Kth Smallest Element in an Array

**Problem Statement:**
Given an unsorted array `arr` of `n` integers and an integer `k`, find the `k`th smallest element in the array.

**Example:**
- Input: `arr = [7,10,4,3,20,15], k = 3`
- Output: `7`
- Explanation: Sorted ascending the array is `[3,4,7,10,15,20]`. The 3rd smallest element is `7`.

**Brute Force Approach:** Sort the array in ascending order and pick the element at index `k - 1`. O(n log n).

```csharp
public class Solution
{
    public int FindKthSmallestBrute(int[] arr, int k)
    {
        int[] copy = (int[])arr.Clone();
        Array.Sort(copy);
        return copy[k - 1];
    }
}
```
Time Complexity: O(n log n) — dominated by sorting.
Space Complexity: O(n) — for the cloned array.

**Optimized Approach:** Maintain a max-heap of size `k`. Push every element; whenever the heap size exceeds `k`, pop the largest. At the end the heap's top (maximum) is the `k`th smallest element, because the heap holds exactly the `k` smallest elements seen so far, and the largest of those `k` is the `k`th smallest overall.

```csharp
using System.Collections.Generic;

public class Solution
{
    public int FindKthSmallest(int[] arr, int k)
    {
        // Max-heap simulated via PriorityQueue with negated priority
        PriorityQueue<int, int> maxHeap = new PriorityQueue<int, int>();

        foreach (int num in arr)
        {
            maxHeap.Enqueue(num, -num); // negate so largest value has smallest priority
            if (maxHeap.Count > k)
            {
                maxHeap.Dequeue(); // removes element with largest value
            }
        }

        return maxHeap.Peek(); // largest among the k smallest = kth smallest
    }
}
```
Time Complexity: O(n log k) — each of the n elements does at most one push and one pop on a heap of size at most k.
Space Complexity: O(k) — the heap never holds more than k elements.

**Explanation:**
Dry run on `arr = [7,10,4,3,20,15], k = 3` (heap stores values; shown sorted for readability, "top" = largest = next to pop):

| Step | Element | Action | Heap contents after action |
|------|---------|--------|-----------------------------|
| 1 | 7 | push 7 | `{7}` |
| 2 | 10 | push 10 | `{7,10}` |
| 3 | 4 | push 4 (size 3 ≤ k) | `{4,7,10}` |
| 4 | 3 | push 3 → size 4 > k → pop max (10) | `{3,4,7}` |
| 5 | 20 | push 20 → size 4 > k → pop max (20) | `{3,4,7}` |
| 6 | 15 | push 15 → size 4 > k → pop max (15) | `{3,4,7}` |

Final heap = `{3,4,7}`, whose top (maximum) is `7`, matching the expected 3rd smallest element.

---

## 3. Sort a Nearly Sorted (K Sorted) Array

**Problem Statement:**
Given an array of `n` elements where each element is at most `k` positions away from its sorted position, sort the array efficiently.

**Example:**
- Input: `arr = [6,5,3,2,8,10,9], k = 3`
- Output: `[2,3,5,6,8,9,10]`
- Explanation: Every element in the input is at most 3 positions away from where it ends up in the sorted output.

**Brute Force Approach:** Simply sort the whole array, ignoring the k-sorted property. O(n log n).

```csharp
public class Solution
{
    public int[] SortKSortedBrute(int[] arr, int k)
    {
        int[] result = (int[])arr.Clone();
        Array.Sort(result);
        return result;
    }
}
```
Time Complexity: O(n log n) — standard comparison sort, does not exploit the k-sorted structure.
Space Complexity: O(n) — for the cloned/output array.

**Optimized Approach:** Use a min-heap of size `k + 1`. Push the first `k + 1` elements. Then for each remaining element, the heap's minimum is guaranteed to be the next smallest element overall (since no unseen element can be smaller than something more than k positions behind it), so pop it into the result, then push the new element. Finally drain the heap.

```csharp
using System.Collections.Generic;

public class Solution
{
    public int[] SortKSortedArray(int[] arr, int k)
    {
        int n = arr.Length;
        int[] result = new int[n];
        int idx = 0;

        PriorityQueue<int, int> minHeap = new PriorityQueue<int, int>();

        // Push first k+1 elements (or fewer if array is smaller)
        int initialCount = Math.Min(k + 1, n);
        for (int i = 0; i < initialCount; i++)
        {
            minHeap.Enqueue(arr[i], arr[i]);
        }

        // For each remaining element: pop min into result, push new element
        for (int i = initialCount; i < n; i++)
        {
            result[idx++] = minHeap.Dequeue();
            minHeap.Enqueue(arr[i], arr[i]);
        }

        // Drain remaining elements in the heap
        while (minHeap.Count > 0)
        {
            result[idx++] = minHeap.Dequeue();
        }

        return result;
    }
}
```
Time Complexity: O(n log k) — heap never exceeds size k+1, and every element is pushed and popped once.
Space Complexity: O(k) — for the heap (plus O(n) for the output array, which is required regardless).

**Explanation:**
The general technique matches the size-k min-heap idea from Problem 1: instead of dry-running this problem again, note it is directly analogous — the heap always holds a window of `k+1` "candidate smallest" elements, and popping the minimum at each step guarantees correctness because any element not yet seen is too far away to be smaller. See Problem 4 below for a full step-by-step heap dry run in the same spirit (K-way merge).

---

## 4. Merge K Sorted Arrays

**Problem Statement:**
Given `k` sorted arrays, merge them into a single sorted array.

**Example:**
- Input: `arrays = [[1,4,5], [1,3,4], [2,6]]`
- Output: `[1,1,2,3,4,4,5,6]`
- Explanation: All elements from the 3 sorted arrays combined and sorted.

**Brute Force Approach:** Concatenate all arrays into one list, then sort it. O(n log n) where n is the total number of elements.

```csharp
using System.Collections.Generic;

public class Solution
{
    public int[] MergeKSortedBrute(int[][] arrays)
    {
        List<int> all = new List<int>();
        foreach (int[] arr in arrays)
        {
            all.AddRange(arr);
        }
        all.Sort();
        return all.ToArray();
    }
}
```
Time Complexity: O(n log n) — n = total elements across all arrays; ignores that each array is already sorted.
Space Complexity: O(n) — for the combined list/result.

**Optimized Approach:** Use a min-heap that holds at most `k` elements — one "current head" element per array, tagged with (arrayIndex, elementIndex). Repeatedly pop the smallest, append it to the result, and push the next element from that same array (if any).

```csharp
using System.Collections.Generic;

public class Solution
{
    public int[] MergeKSortedArrays(int[][] arrays)
    {
        int k = arrays.Length;
        int totalLength = 0;
        foreach (int[] arr in arrays) totalLength += arr.Length;

        int[] result = new int[totalLength];
        int idx = 0;

        // Element: (value, arrayIndex, elementIndex); Priority: value
        PriorityQueue<(int value, int arrIdx, int elemIdx), int> minHeap =
            new PriorityQueue<(int, int, int), int>();

        // Push the first element of each non-empty array
        for (int i = 0; i < k; i++)
        {
            if (arrays[i].Length > 0)
            {
                minHeap.Enqueue((arrays[i][0], i, 0), arrays[i][0]);
            }
        }

        while (minHeap.Count > 0)
        {
            var (value, arrIdx, elemIdx) = minHeap.Dequeue();
            result[idx++] = value;

            int nextElemIdx = elemIdx + 1;
            if (nextElemIdx < arrays[arrIdx].Length)
            {
                int nextValue = arrays[arrIdx][nextElemIdx];
                minHeap.Enqueue((nextValue, arrIdx, nextElemIdx), nextValue);
            }
        }

        return result;
    }
}
```
Time Complexity: O(n log k) — n total elements, each pushed and popped once from a heap of size at most k.
Space Complexity: O(k) for the heap, plus O(n) for the output array (output space is unavoidable).

**Explanation:**
Dry run of the K-way merge on `arrays = [[1,4,5], [1,3,4], [2,6]]` (label arrays A=`[1,4,5]`, B=`[1,3,4]`, C=`[2,6]`):

Initial push: heads of A, B, C → heap = `{(1,A,0), (1,B,0), (2,C,0)}`

| Step | Popped (value, array) | Result so far | Push next from that array | Heap after |
|------|------------------------|----------------|-----------------------------|-------------|
| 1 | (1, A) — index0 | `[1]` | push A[1]=4 | `{(1,B),(2,C),(4,A)}` |
| 2 | (1, B) — index0 | `[1,1]` | push B[1]=3 | `{(2,C),(3,B),(4,A)}` |
| 3 | (2, C) — index0 | `[1,1,2]` | push C[1]=6 | `{(3,B),(4,A),(6,C)}` |
| 4 | (3, B) — index1 | `[1,1,2,3]` | push B[2]=4 | `{(4,A),(4,B),(6,C)}` |
| 5 | (4, A) — index1 | `[1,1,2,3,4]` | push A[2]=5 | `{(4,B),(5,A),(6,C)}` |
| 6 | (4, B) — index2 | `[1,1,2,3,4,4]` | B exhausted, nothing pushed | `{(5,A),(6,C)}` |
| 7 | (5, A) — index2 | `[1,1,2,3,4,4,5]` | A exhausted, nothing pushed | `{(6,C)}` |
| 8 | (6, C) — index1 | `[1,1,2,3,4,4,5,6]` | C exhausted, nothing pushed | `{}` |

Heap empties after 8 pops, giving the final merged result `[1,1,2,3,4,4,5,6]`, which matches the expected output.

---

## 5. Replace Each Array Element by Its Corresponding Rank

**Problem Statement:**
Given an array of `n` integers, replace every element with its rank in the array. Rank 1 goes to the smallest element, rank 2 to the next smallest, and so on. Equal elements get the same rank.

**Example:**
- Input: `arr = [20,15,26,2,98,6]`
- Output: `[4,3,5,1,6,2]`
- Explanation: Sorted order is `[2,6,15,20,26,98]`, so ranks are `2→1, 6→2, 15→3, 20→4, 26→5, 98→6`. Replacing each original element with its rank gives `[4,3,5,1,6,2]`.

**Brute Force Approach:** Create a sorted copy of the array, then for each original element do a linear (or binary) search in the sorted copy to find its rank (1-based first occurrence index).

```csharp
using System;

public class Solution
{
    public int[] ReplaceWithRankBrute(int[] arr)
    {
        int n = arr.Length;
        int[] sorted = (int[])arr.Clone();
        Array.Sort(sorted);

        int[] result = new int[n];
        for (int i = 0; i < n; i++)
        {
            // Linear search for first occurrence (rank)
            for (int j = 0; j < n; j++)
            {
                if (sorted[j] == arr[i])
                {
                    result[i] = j + 1;
                    break;
                }
            }
        }
        return result;
    }
}
```
Time Complexity: O(n^2) with linear search (O(n log n) if binary search is used instead for each lookup).
Space Complexity: O(n) — for the sorted copy and result array.

**Optimized Approach:** Sort a copy of the array, then build a dictionary mapping each distinct value to its rank (assigned only once, the first time that value is encountered in sorted order). Finally map every original element through the dictionary. (A min-heap of (value, originalIndex) pairs can equivalently be used to pop elements in sorted order and assign increasing ranks — shown below.)

```csharp
using System;
using System.Collections.Generic;

public class Solution
{
    public int[] ReplaceWithRank(int[] arr)
    {
        int n = arr.Length;
        int[] result = new int[n];

        // Min-heap of (value, originalIndex), ordered by value
        PriorityQueue<(int value, int originalIndex), int> minHeap =
            new PriorityQueue<(int, int), int>();

        for (int i = 0; i < n; i++)
        {
            minHeap.Enqueue((arr[i], i), arr[i]);
        }

        int rank = 0;
        int? lastValue = null;

        while (minHeap.Count > 0)
        {
            var (value, originalIndex) = minHeap.Dequeue();

            // Increase rank only when we see a new distinct value
            if (lastValue == null || value != lastValue.Value)
            {
                rank++;
                lastValue = value;
            }

            result[originalIndex] = rank;
        }

        return result;
    }
}
```
Time Complexity: O(n log n) — n pushes and n pops on a heap that can hold up to n elements.
Space Complexity: O(n) — for the heap and the result array.

**Explanation:**
Applying the size-n min-heap ranking technique to `arr = [20,15,26,2,98,6]`:
Push all pairs `(20,0), (15,1), (26,2), (2,3), (98,4), (6,5)`.
Pop in increasing order of value: `2`(idx3)→rank1, `6`(idx5)→rank2, `15`(idx1)→rank3, `20`(idx0)→rank4, `26`(idx2)→rank5, `98`(idx4)→rank6.
Placing ranks back at their original indices gives `result = [4,3,5,1,6,2]`, matching the expected output. Duplicate values would simply reuse the previous rank instead of incrementing, per the `lastValue` check.

---

## 6. Task Scheduler (with cooldown period n between same tasks)

**Problem Statement:**
Given a character array `tasks` representing CPU tasks (each letter is a task type) and a non-negative integer `n` representing the cooldown period, find the minimum number of CPU intervals required to finish all tasks. The same task type must be separated by at least `n` intervals; the CPU can either process a task or stay idle in a given interval.

**Example:**
- Input: `tasks = ['A','A','A','B','B','B'], n = 2`
- Output: `8`
- Explanation: One valid schedule is `A -> B -> idle -> A -> B -> idle -> A -> B`, which takes 8 intervals total.

**Brute Force Approach:** Sort tasks by frequency (descending) each round conceptually via full sort at start, then simulate interval by interval, at each interval picking the most frequent task not currently on cooldown by scanning/sorting the frequency array. Using `Array.Sort` on the frequency counts repeatedly emulates the "sort-based" approach.

```csharp
using System;
using System.Collections.Generic;

public class Solution
{
    public int LeastIntervalBrute(char[] tasks, int n)
    {
        int[] freq = new int[26];
        foreach (char t in tasks) freq[t - 'A']++;

        int time = 0;
        while (true)
        {
            // Sort frequencies descending each interval (brute force re-sort)
            Array.Sort(freq);
            Array.Reverse(freq);

            if (freq[0] == 0) break; // all tasks done

            int cycleTime = Math.Min(n + 1, CountRemaining(freq));
            List<int> executedIndices = new List<int>();

            for (int i = 0; i < cycleTime; i++)
            {
                if (i < 26 && freq[i] > 0)
                {
                    freq[i]--;
                    time++;
                    if (freq[i] > 0) executedIndices.Add(i);
                }
                else
                {
                    time++; // idle slot
                }

                // Check if remaining work is finished mid-cycle
                if (AllZero(freq)) break;
            }
        }

        return time;
    }

    private int CountRemaining(int[] freq)
    {
        int count = 0;
        foreach (int f in freq) if (f > 0) count++;
        return Math.Max(count, 1);
    }

    private bool AllZero(int[] freq)
    {
        foreach (int f in freq) if (f > 0) return false;
        return true;
    }
}
```
Time Complexity: O(T * (n + log 26)) ≈ O(T * n) where T is total number of tasks — repeated sorting each cycle is wasteful (this is the "brute force" cost compared to the heap approach).
Space Complexity: O(1) — fixed 26-size frequency array (26 is a constant).

**Optimized Approach:** Count frequency of each task, push all non-zero frequencies into a max-heap. Repeatedly pop up to `n + 1` most frequent tasks for one cooldown cycle, decrement their counts, collect leftovers, and re-push leftovers after the cycle. Count idle slots when the heap runs out of distinct tasks mid-cycle.

```csharp
using System.Collections.Generic;

public class Solution
{
    public int LeastInterval(char[] tasks, int n)
    {
        int[] freq = new int[26];
        foreach (char t in tasks) freq[t - 'A']++;

        // Max-heap via negated priority
        PriorityQueue<int, int> maxHeap = new PriorityQueue<int, int>();
        foreach (int f in freq)
        {
            if (f > 0) maxHeap.Enqueue(f, -f);
        }

        int time = 0;

        while (maxHeap.Count > 0)
        {
            List<int> leftover = new List<int>();
            int slotsUsed = 0;

            // One cooldown cycle of length n+1
            for (int i = 0; i < n + 1; i++)
            {
                if (maxHeap.Count > 0)
                {
                    int count = maxHeap.Dequeue();
                    slotsUsed++;
                    if (count - 1 > 0) leftover.Add(count - 1);
                }

                // If heap and leftover both empty, remaining tasks are done
                if (maxHeap.Count == 0 && leftover.Count == 0)
                {
                    // still need to count this slot if a task was executed
                    break;
                }
            }

            foreach (int c in leftover)
            {
                maxHeap.Enqueue(c, -c);
            }

            // If there are still tasks left after this cycle, the cycle was fully used (n+1);
            // otherwise only 'slotsUsed' intervals were needed.
            time += (maxHeap.Count == 0) ? slotsUsed : n + 1;
        }

        return time;
    }
}
```
Time Complexity: O(T log 26) which simplifies to O(T) — T is total tasks; heap operations are on at most 26 distinct task types (a constant), so each cycle's heap work is O(log 26), repeated O(T / (n+1)) cycles worst case, overall linear in T.
Space Complexity: O(1) — heap holds at most 26 distinct task types.

**Explanation:**
This problem uses the max-heap + frequency greedy technique (not the K-way merge dry run format, since it is a simulation rather than a merge). On `tasks = [A,A,A,B,B,B], n = 2`: frequencies are `A:3, B:3`. Max-heap starts as `{3(A), 3(B)}`.
- Cycle 1 (3 slots, n+1=3): pop A(3)→execute, leftover 2; pop B(3)→execute, leftover 2; heap empty, leftover has 2 items so idle for 1 slot → 3 slots used, time=3. Re-push leftovers: heap = `{2(A), 2(B)}`.
- Cycle 2: pop A(2)→execute, leftover 1; pop B(2)→execute, leftover 1; idle 1 slot → time=6. Re-push: heap = `{1(A), 1(B)}`.
- Cycle 3: pop A(1)→execute, leftover 0 (not re-pushed); pop B(1)→execute, leftover 0; heap now empty after this cycle so only slotsUsed=2 intervals counted → time=8.

Final time = 8, matching the expected output, corresponding to schedule `A B idle A B idle A B`.

---

## 7. Hand of Straights (check if cards can be grouped into consecutive groups of size W)

**Problem Statement:**
Given an array `hand` of integers representing card values and an integer `groupSize` (W), determine whether the cards can be rearranged into groups of exactly `groupSize` cards each, where each group consists of `groupSize` consecutive integers.

**Example:**
- Input: `hand = [1,2,3,6,2,3,4,7,8], groupSize = 3`
- Output: `true`
- Explanation: The cards can be split into three consecutive groups: `[1,2,3]`, `[2,3,4]`, `[6,7,8]`.

**Brute Force Approach:** Sort the array. Repeatedly take the smallest remaining card, then greedily search (linear scan) for the next `groupSize - 1` consecutive values among the remaining cards, removing them from a list as they're used. If at any point a needed value cannot be found, return false.

```csharp
using System;
using System.Collections.Generic;

public class Solution
{
    public bool IsNStraightHandBrute(int[] hand, int groupSize)
    {
        int n = hand.Length;
        if (n % groupSize != 0) return false;

        List<int> cards = new List<int>(hand);
        cards.Sort();

        while (cards.Count > 0)
        {
            int start = cards[0];
            cards.RemoveAt(0);

            for (int i = 1; i < groupSize; i++)
            {
                int need = start + i;
                int foundIndex = cards.IndexOf(need); // linear search
                if (foundIndex == -1) return false;
                cards.RemoveAt(foundIndex);
            }
        }

        return true;
    }
}
```
Time Complexity: O(n log n) for the sort plus O(n * groupSize) for repeated linear searches/removals ≈ O(n^2) worst case.
Space Complexity: O(n) — for the mutable list copy of cards.

**Optimized Approach:** Build a frequency dictionary of card values and push all distinct values into a min-heap (or simply sort the distinct keys, since a min-heap of distinct values achieves the same increasing-order extraction). Repeatedly take the smallest available card (peek the heap), and if its count is still positive, consume `groupSize` consecutive values starting there directly from the dictionary; pop exhausted values off the heap.

```csharp
using System.Collections.Generic;

public class Solution
{
    public bool IsNStraightHand(int[] hand, int groupSize)
    {
        int n = hand.Length;
        if (n % groupSize != 0) return false;

        Dictionary<int, int> count = new Dictionary<int, int>();
        foreach (int card in hand)
        {
            count[card] = count.GetValueOrDefault(card, 0) + 1;
        }

        // Min-heap of distinct card values
        PriorityQueue<int, int> minHeap = new PriorityQueue<int, int>();
        foreach (int value in count.Keys)
        {
            minHeap.Enqueue(value, value);
        }

        while (minHeap.Count > 0)
        {
            int start = minHeap.Peek();

            // Skip values that have already been fully consumed
            if (count[start] == 0)
            {
                minHeap.Dequeue();
                continue;
            }

            int openings = count[start]; // how many groups start with this card right now

            for (int i = 0; i < groupSize; i++)
            {
                int need = start + i;
                if (!count.ContainsKey(need) || count[need] < openings)
                {
                    return false; // not enough consecutive cards available
                }
                count[need] -= openings;
            }
        }

        return true;
    }
}
```
Time Complexity: O(n log n) — n cards, up to n distinct values pushed into the heap (log n each); each distinct value is processed once, and each groupSize-length consumption is bounded so total work across all groups is O(n).
Space Complexity: O(n) — for the frequency dictionary and the heap of distinct values.

**Explanation:**
Using the size-n min-heap of distinct values on `hand = [1,2,3,6,2,3,4,7,8], groupSize = 3`:
Frequency map: `{1:1, 2:2, 3:2, 4:1, 6:1, 7:1, 8:1}`. Min-heap of distinct keys = `{1,2,3,4,6,7,8}` (smallest on top).
- Peek `1`, count[1]=1>0, openings=1. Consume `1,2,3` (need 1 each): counts become `{1:0,2:1,3:1,4:1,6:1,7:1,8:1}`.
- Peek `1` again (still on heap), count[1]=0 → pop and skip.
- Peek `2`, count[2]=1>0, openings=1. Consume `2,3,4`: counts become `{2:0,3:0,4:0,6:1,7:1,8:1}`.
- Peek `2` again, count=0 → pop and skip. Peek `3`, count=0 → pop and skip. Peek `4`, count=0 → pop and skip.
- Peek `6`, count[6]=1>0, openings=1. Consume `6,7,8`: counts become `{6:0,7:0,8:0}`.
- Remaining heap entries `6,7,8` are popped and skipped since their counts are 0.

Heap empties with no failure, so the function returns `true`, matching the expected output — groups formed were `[1,2,3]`, `[2,3,4]`, `[6,7,8]`.
