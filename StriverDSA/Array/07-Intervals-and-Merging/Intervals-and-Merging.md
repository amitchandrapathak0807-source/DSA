# Array — Intervals and Merging

## Merge Overlapping Subintervals

### 1. Merge Overlapping Subintervals

**Problem Statement:** Given an array of intervals where each interval is represented as `[start, end]`, merge all overlapping intervals and return an array of the non-overlapping intervals that cover all the intervals in the input. Two intervals `[a, b]` and `[c, d]` are considered overlapping if `c <= b` (assuming the intervals are sorted by start).

**Example:**
- Input: intervals `[[1,3],[2,6],[8,10],[15,18]]`
- Output: `[[1,6],[8,10],[15,18]]`
- Explanation: Intervals `[1,3]` and `[2,6]` overlap since `2 <= 3`, so they merge into `[1,6]`. `[8,10]` and `[15,18]` don't overlap with anything, so they remain unchanged.

**Brute Force Approach:** Sort the intervals by their start value. Then, for every interval, scan through all other intervals to find every interval that overlaps with it, keep expanding the merged range, and mark the consumed intervals as visited so they are not processed again. This uses an explicit visited array and nested loops to simulate merging.

```csharp
public class Solution
{
    public IList<int[]> MergeBruteForce(int[][] intervals)
    {
        int n = intervals.Length;
        Array.Sort(intervals, (a, b) => a[0].CompareTo(b[0]));
        bool[] visited = new bool[n];
        List<int[]> result = new List<int[]>();

        for (int i = 0; i < n; i++)
        {
            if (visited[i]) continue;

            int start = intervals[i][0];
            int end = intervals[i][1];
            visited[i] = true;

            // Keep scanning all intervals to see if any unvisited interval
            // overlaps with the current merged range; repeat until stable
            bool merged = true;
            while (merged)
            {
                merged = false;
                for (int j = 0; j < n; j++)
                {
                    if (visited[j]) continue;

                    if (intervals[j][0] <= end)
                    {
                        end = Math.Max(end, intervals[j][1]);
                        start = Math.Min(start, intervals[j][0]);
                        visited[j] = true;
                        merged = true;
                    }
                }
            }

            result.Add(new int[] { start, end });
        }

        return result;
    }
}
```

Time Complexity: `O(n log n + n^2)` — sorting takes `O(n log n)`, and for each interval we may rescan all `n` intervals repeatedly until no more merges happen, giving a worst case of `O(n^2)`.
Space Complexity: `O(n)` for the `visited` array and the result list (ignoring the sort's internal space).

**Optimized Approach:** Sort the intervals by their start value. Then do a single linear pass: maintain a "last merged interval" (initially the first interval). For each subsequent interval, if its start is less than or equal to the end of the last merged interval, they overlap — update the end of the last merged interval to the max of the two ends. Otherwise, the current interval does not overlap, so push the last merged interval to the result and make the current interval the new "last merged interval". Finally push the last one after the loop ends.

```csharp
public class Solution
{
    public IList<int[]> Merge(int[][] intervals)
    {
        if (intervals == null || intervals.Length == 0)
            return new List<int[]>();

        Array.Sort(intervals, (a, b) => a[0].CompareTo(b[0]));

        List<int[]> merged = new List<int[]>();
        int[] last = intervals[0];

        for (int i = 1; i < intervals.Length; i++)
        {
            int[] current = intervals[i];

            if (current[0] <= last[1])
            {
                // Overlaps with the last merged interval — extend its end
                last[1] = Math.Max(last[1], current[1]);
            }
            else
            {
                // No overlap — finalize the last interval, start a new one
                merged.Add(last);
                last = current;
            }
        }

        merged.Add(last); // push the final interval

        return merged;
    }
}
```

Time Complexity: `O(n log n)` — dominated by sorting the intervals; the linear merge pass afterward is `O(n)`.
Space Complexity: `O(n)` for the output list (or `O(log n)` to `O(n)` extra for the sort itself, plus `O(n)` for the result).

**Explanation:** The key idea is that once intervals are sorted by start value, any interval that overlaps with the current merged range must appear immediately after it in sorted order — we never need to look backward or rescan. Dry run on `[[1,3],[2,6],[8,10],[15,18]]`:
- After sorting: `[[1,3],[2,6],[8,10],[15,18]]` (already sorted).
- Initialize `last = [1,3]`.
- `i=1`, `current = [2,6]`: `2 <= 3` (overlap) → `last = [1, max(3,6)] = [1,6]`.
- `i=2`, `current = [8,10]`: `8 <= 6`? No → push `last = [1,6]` to result; `last = [8,10]`.
- `i=3`, `current = [15,18]`: `15 <= 10`? No → push `last = [8,10]` to result; `last = [15,18]`.
- Loop ends → push `last = [15,18]`.
- Final result: `[[1,6],[8,10],[15,18]]`, matching the expected output.

## Merge Two Sorted Arrays Without Using Extra Space

### 2. Merge Two Sorted Arrays Without Using Extra Space

**Problem Statement:** Given two sorted arrays `arr1` of size `n` and `arr2` of size `m`, merge them in-place such that after the operation, `arr1` contains the first `n` smallest elements in sorted order, and `arr2` contains the remaining `m` largest elements in sorted order. This must be done without using any extra space (no auxiliary array of size `O(n+m)`).

**Example:**
- Input: two sorted arrays `arr1 = [1,3,5,7]`, `arr2 = [0,2,6,8,9]`
- Output: `arr1 = [0,1,2,3]`, `arr2 = [5,6,7,8,9]`
- Explanation: Combining both arrays gives the sorted sequence `[0,1,2,3,5,6,7,8,9]`. The first `n = 4` elements `[0,1,2,3]` go into `arr1`, and the remaining `m = 5` elements `[5,6,7,8,9]` go into `arr2`.

**Brute Force Approach:** Use a merge-like comparison, swapping elements whenever an element of `arr1` is greater than an element of `arr2`. Iterate `i` over `arr1` and `j` over `arr2` (starting at 0); whenever `arr1[i] > arr2[j]`, swap them, then re-sort `arr2` (or bubble the swapped element into place) so `arr2` stays sorted, and continue. A simpler brute force is to swap-if-greater in a nested fashion and then sort both arrays at the end — this avoids extra space but costs extra time.

```csharp
public class Solution
{
    public void MergeBruteForce(int[] arr1, int n, int[] arr2, int m)
    {
        // Compare every element of arr1 with every element of arr2,
        // swapping whenever arr1[i] > arr2[j] to push larger values into arr2
        for (int i = 0; i < n; i++)
        {
            if (arr1[i] > arr2[0])
            {
                int temp = arr1[i];
                arr1[i] = arr2[0];
                arr2[0] = temp;

                // Re-insert arr2[0] into its correct sorted position in arr2
                int first = arr2[0];
                int k = 1;
                while (k < m && arr2[k] < first)
                {
                    arr2[k - 1] = arr2[k];
                    k++;
                }
                arr2[k - 1] = first;
            }
        }
    }
}
```

Time Complexity: `O(n * m)` — for each of the `n` elements in `arr1`, we may need up to `O(m)` work to re-insert the swapped element into its correct sorted position in `arr2`.
Space Complexity: `O(1)` — no extra array is used, only a temporary variable for swapping.

**Optimized Approach:** Use the Gap Method (based on Shell Sort's gap sequence). Treat `arr1` and `arr2` as one logical combined array of length `n + m` (indices `0..n-1` map to `arr1`, indices `n..n+m-1` map to `arr2`). Start with `gap = ceil((n+m)/2)`. In each pass, compare elements that are `gap` apart in this logical array; if the left element is greater than the right element, swap them. After completing a pass, shrink the gap using `gap = ceil(gap/2)`, and repeat until `gap == 0`. This progressively moves smaller elements toward the front and larger elements toward the back across both arrays, achieving a full sort with `O(1)` extra space.

```csharp
public class Solution
{
    public void Merge(int[] arr1, int n, int[] arr2, int m)
    {
        int totalLength = n + m;
        int gap = (totalLength / 2) + (totalLength % 2); // ceil(totalLength / 2)

        while (gap > 0)
        {
            int left = 0;
            int right = left + gap;

            while (right < totalLength)
            {
                int leftVal = GetValue(arr1, arr2, n, left);
                int rightVal = GetValue(arr1, arr2, n, right);

                if (leftVal > rightVal)
                {
                    SetValue(arr1, arr2, n, left, rightVal);
                    SetValue(arr1, arr2, n, right, leftVal);
                }

                left++;
                right++;
            }

            if (gap == 1) break;
            gap = (gap / 2) + (gap % 2); // ceil(gap / 2)
        }
    }

    // Maps a logical index (0..n+m-1) to the correct value in arr1 or arr2
    private int GetValue(int[] arr1, int[] arr2, int n, int index)
    {
        return index < n ? arr1[index] : arr2[index - n];
    }

    // Maps a logical index (0..n+m-1) and sets the value in arr1 or arr2
    private void SetValue(int[] arr1, int[] arr2, int n, int index, int value)
    {
        if (index < n)
            arr1[index] = value;
        else
            arr2[index - n] = value;
    }
}
```

Time Complexity: `O((n+m) * log(n+m))` — the gap shrinks by roughly half each pass (`log(n+m)` passes total), and each pass does `O(n+m)` comparisons.
Space Complexity: `O(1)` — only the `gap`, `left`, `right`, and temporary swap values are used; no auxiliary array.

**Explanation:**

*Merge Overlapping Subintervals* relies on sorting first so overlapping intervals become adjacent, then a single linear scan comparing each interval's start against the end of the last merged interval — covered in the dry run above.

*The Gap Method* works by treating `arr1` (length `n`) and `arr2` (length `m`) as a single virtual array of length `n+m`, where logical index `k` maps to `arr1[k]` if `k < n`, else `arr2[k-n]`. Instead of comparing adjacent elements (gap = 1) like insertion sort — which would be slow — it starts with a large gap and compares far-apart elements first, swapping if out of order, then shrinks the gap (`gap = ceil(gap/2)`) each pass until `gap` reaches `0`. This is the same idea as Shell Sort and guarantees the combined logical array ends up fully sorted in `O((n+m) log(n+m))` time using no extra space.

Dry run on `arr1 = [1,3,5,7]` (n=4), `arr2 = [0,2,6,8,9]` (m=5), total length = 9:
- **gap = ceil(9/2) = 5**
  - Compare logical indices (0,5): `arr1[0]=1` vs `arr2[1]=2` → `1 <= 2`, no swap.
  - Compare (1,6): `arr1[1]=3` vs `arr2[2]=6` → no swap.
  - Compare (2,7): `arr1[2]=5` vs `arr2[3]=8` → no swap.
  - Compare (3,8): `arr1[3]=7` vs `arr2[4]=9` → no swap.
  - State unchanged: `arr1=[1,3,5,7]`, `arr2=[0,2,6,8,9]`.
- **gap = ceil(5/2) = 3**
  - Compare (0,3): `arr1[0]=1` vs `arr1[3]=7` → no swap.
  - Compare (1,4): `arr1[1]=3` vs `arr2[0]=0` → `3 > 0` → swap → `arr1=[1,0,5,7]`, `arr2=[3,2,6,8,9]`.
  - Compare (2,5): `arr1[2]=5` vs `arr2[1]=2` → `5 > 2` → swap → `arr1=[1,0,2,7]`, `arr2=[3,5,6,8,9]`.
  - Compare (3,6): `arr1[3]=7` vs `arr2[2]=6` → `7 > 6` → swap → `arr1=[1,0,2,6]`, `arr2=[3,5,7,8,9]`.
  - Compare (4,7): `arr2[0]=3` vs `arr2[3]=8` → no swap.
  - Compare (5,8): `arr2[1]=5` vs `arr2[4]=9` → no swap.
  - State: `arr1=[1,0,2,6]`, `arr2=[3,5,7,8,9]`.
- **gap = ceil(3/2) = 2**
  - Compare (0,2): `arr1[0]=1` vs `arr1[2]=2` → no swap.
  - Compare (1,3): `arr1[1]=0` vs `arr1[3]=6` → no swap.
  - Compare (2,4): `arr1[2]=2` vs `arr2[0]=3` → no swap.
  - Compare (3,5): `arr1[3]=6` vs `arr2[1]=5` → `6 > 5` → swap → `arr1=[1,0,2,5]`, `arr2=[6,3,7,8,9]`.
  - Compare (4,6): `arr2[0]=6` vs `arr2[2]=7` → no swap.
  - Compare (5,7): `arr2[1]=3` vs `arr2[3]=8` → no swap.
  - Compare (6,8): `arr2[2]=7` vs `arr2[4]=9` → no swap.
  - State: `arr1=[1,0,2,5]`, `arr2=[6,3,7,8,9]`.
- **gap = ceil(2/2) = 1** (logical array going in: `[1,0,2,5,3,6,7,8,9]`)
  - Compare (0,1): `arr1[0]=1` vs `arr1[1]=0` → `1 > 0` → swap → logical array `[0,1,2,5,3,6,7,8,9]`.
  - Compare (1,2): `1` vs `2` → no swap.
  - Compare (2,3): `2` vs `5` → no swap.
  - Compare (3,4): `arr1[3]=5` vs `arr2[0]=3` → `5 > 3` → swap → logical array `[0,1,2,3,5,6,7,8,9]`.
  - Compare (4,5): `arr2[0]=5` vs `arr2[1]=6` → no swap.
  - Compare (5,6): `6` vs `7` → no swap.
  - Compare (6,7): `7` vs `8` → no swap.
  - Compare (7,8): `8` vs `9` → no swap.
  - Since `gap == 1`, the loop terminates after this pass.

Final state: logical array `[0,1,2,3,5,6,7,8,9]` splits back into `arr1 = [0,1,2,3]` and `arr2 = [5,6,7,8,9]`, exactly matching the expected output.
