# Binary Search — Rotated Sorted Arrays

## 1. Search in Rotated Sorted Array I (all elements distinct)

**Problem Statement:** Given a sorted array of distinct integers that has been rotated at some unknown pivot, and a target value, find the index of the target in the array. If the target is not present, return -1. The array must be searched in `O(log n)` time.

**Example:**
- Input: `arr = [4,5,6,7,0,1,2], target = 0`
- Output: `4`
- Explanation: `0` is located at index `4` in the rotated array.

**Brute Force Approach:** linear scan O(n).
```csharp
public class Solution
{
    public int SearchBruteForce(int[] arr, int target)
    {
        for (int i = 0; i < arr.Length; i++)
        {
            if (arr[i] == target)
                return i;
        }
        return -1;
    }
}
```
Time Complexity: O(n) — every element may need to be checked.
Space Complexity: O(1) — no extra space used.

**Optimized Approach:** modified binary search identifying which half is sorted at each step.
```csharp
public class Solution
{
    public int Search(int[] arr, int target)
    {
        int low = 0, high = arr.Length - 1;

        while (low <= high)
        {
            int mid = low + (high - low) / 2;

            if (arr[mid] == target)
                return mid;

            // Check which half is sorted
            if (arr[low] <= arr[mid])
            {
                // Left half [low..mid] is sorted
                if (arr[low] <= target && target < arr[mid])
                    high = mid - 1;
                else
                    low = mid + 1;
            }
            else
            {
                // Right half [mid..high] is sorted
                if (arr[mid] < target && target <= arr[high])
                    low = mid + 1;
                else
                    high = mid - 1;
            }
        }

        return -1;
    }
}
```
Time Complexity: O(log n) — search space halves every iteration.
Space Complexity: O(1) — only pointers are used.

**Explanation:** Dry run on `arr = [4,5,6,7,0,1,2], target = 0`:
- `low = 0, high = 6`, `mid = 3` → `arr[mid] = 7`. Not the target.
  Check `arr[low] (4) <= arr[mid] (7)` → left half `[4,5,6,7]` is sorted.
  Is `target (0)` within `[arr[low]=4, arr[mid]=7)`? No, `0 < 4`. So target must be in the right half → `low = mid + 1 = 4`.
- `low = 4, high = 6`, `mid = 5` → `arr[mid] = 1`. Not the target.
  Check `arr[low] (0) <= arr[mid] (1)` → left half `[0,1]` is sorted.
  Is `target (0)` within `[arr[low]=0, arr[mid]=1)`? Yes, `0 <= 0 < 1`. So search left half → `high = mid - 1 = 4`.
- `low = 4, high = 4`, `mid = 4` → `arr[mid] = 0` — matches target. Return `4`.

The core idea: at every step, one half of the array (relative to `mid`) is guaranteed to be strictly sorted, because a rotated sorted array can have at most one "break point". Comparing `arr[low]` with `arr[mid]` tells us which half is sorted; then we simply check whether the target's value falls within that sorted half's range. If it does, we recurse/iterate into that half; otherwise the target must be in the other (unsorted-looking) half, so we move there.

---

## 2. Search in Rotated Sorted Array II (may contain duplicates)

**Problem Statement:** Given a rotated sorted array that may contain duplicate integers, and a target value, determine whether the target exists in the array. Return `true` or `false`.

**Example:**
- Input: `arr = [3,1,2,3,3,3,3], target = 1`
- Output: `true`
- Explanation: `1` is present in the array at index `1`.

**Brute Force Approach:** linear scan O(n).
```csharp
public class Solution
{
    public bool SearchBruteForce(int[] arr, int target)
    {
        for (int i = 0; i < arr.Length; i++)
        {
            if (arr[i] == target)
                return true;
        }
        return false;
    }
}
```
Time Complexity: O(n) — every element may need to be checked.
Space Complexity: O(1) — no extra space used.

**Optimized Approach:** modified binary search with an extra check to handle duplicates at the boundaries.
```csharp
public class Solution
{
    public bool Search(int[] arr, int target)
    {
        int low = 0, high = arr.Length - 1;

        while (low <= high)
        {
            int mid = low + (high - low) / 2;

            if (arr[mid] == target)
                return true;

            // Edge case: cannot decide which side is sorted
            if (arr[low] == arr[mid] && arr[mid] == arr[high])
            {
                low++;
                high--;
                continue;
            }

            if (arr[low] <= arr[mid])
            {
                // Left half is sorted
                if (arr[low] <= target && target < arr[mid])
                    high = mid - 1;
                else
                    low = mid + 1;
            }
            else
            {
                // Right half is sorted
                if (arr[mid] < target && target <= arr[high])
                    low = mid + 1;
                else
                    high = mid - 1;
            }
        }

        return false;
    }
}
```
Time Complexity: O(log n) on average; O(n) in the worst case, because duplicates can make `arr[low] == arr[mid] == arr[high]` true repeatedly, forcing the algorithm to shrink the search space one element at a time from both ends (e.g., `arr = [3,1,3,3,3]` searching for `1`: with `low`/`mid`/`high` all equal to `3`, we cannot tell which half is sorted, so we only shrink by one on each side instead of halving).
Space Complexity: O(1) — only pointers are used.

**Explanation:** The extra edge case `arr[low] == arr[mid] == arr[high]` is needed because with duplicates present, this condition makes it genuinely impossible to determine which half (left of `mid` or right of `mid`) is properly sorted — both halves could look "flat" while the actual break point (and the target) hides inside one of them. For example, in `arr = [3,3,3,1,3]`, `arr[low]=3, arr[mid]=3, arr[high]=3`, yet the rotation point is between index 2 and 3. In such cases we cannot safely jump `low` or `high` by more than one position, so we conservatively do `low++; high--;` and re-evaluate. This is the one extra check that distinguishes this problem from Rotated Sorted Array I, and it's also what degrades the worst-case complexity from O(log n) to O(n) when the array is full of duplicates (e.g., all `3`s except one `1`).

---

## 3. Find Minimum in Rotated Sorted Array

**Problem Statement:** Given a sorted array of distinct integers that has been rotated at some unknown pivot, find the minimum element in the array. The array must be searched in `O(log n)` time.

**Example:**
- Input: `arr = [4,5,6,7,0,1,2]`
- Output: `0`
- Explanation: The minimum element is `0`, located at index `4` (the rotation point).

**Brute Force Approach:** linear scan O(n).
```csharp
public class Solution
{
    public int FindMinBruteForce(int[] arr)
    {
        int minVal = arr[0];
        for (int i = 1; i < arr.Length; i++)
        {
            if (arr[i] < minVal)
                minVal = arr[i];
        }
        return minVal;
    }
}
```
Time Complexity: O(n) — every element must be inspected.
Space Complexity: O(1) — no extra space used.

**Optimized Approach:** modified binary search identifying which half is sorted, and tracking the minimum of the sorted half.
```csharp
public class Solution
{
    public int FindMin(int[] arr)
    {
        int low = 0, high = arr.Length - 1;
        int result = int.MaxValue;

        while (low <= high)
        {
            int mid = low + (high - low) / 2;

            if (arr[low] <= arr[high])
            {
                // Entire remaining range is already sorted
                result = Math.Min(result, arr[low]);
                break;
            }

            if (arr[low] <= arr[mid])
            {
                // Left half is sorted; its minimum is arr[low]
                result = Math.Min(result, arr[low]);
                low = mid + 1;
            }
            else
            {
                // Right half is sorted; its minimum is arr[mid]
                result = Math.Min(result, arr[mid]);
                high = mid - 1;
            }
        }

        return result;
    }
}
```
Time Complexity: O(log n) — search space halves every iteration.
Space Complexity: O(1) — only pointers and a running result are used.

**Explanation:** This reuses the same "identify the sorted half" logic from problem 1. At every step, whichever half is sorted has its minimum sitting at its leftmost index (`arr[low]` for a sorted left half, `arr[mid]` for a sorted right half), so we record that candidate minimum and then move into the *other* half, which must still contain the actual rotation point (and thus a potentially smaller value). Once `arr[low] <= arr[high]`, the remaining segment is fully sorted, so `arr[low]` is immediately the minimum of that segment and we can stop.

---

## 4. Find How Many Times an Array Has Been Rotated

**Problem Statement:** Given a sorted array of distinct integers that has been rotated at some unknown pivot, find the number of times the array has been rotated (i.e., how many left rotations were performed on the original sorted array).

**Example:**
- Input: `arr = [4,5,6,7,0,1,2]`
- Output: `4`
- Explanation: The minimum element `0` is at index `4`, meaning the original sorted array `[0,1,2,4,5,6,7]` was rotated 4 times to the left to produce this array.

**Brute Force Approach:** linear scan O(n).
```csharp
public class Solution
{
    public int FindRotationCountBruteForce(int[] arr)
    {
        int minIndex = 0;
        for (int i = 1; i < arr.Length; i++)
        {
            if (arr[i] < arr[minIndex])
                minIndex = i;
        }
        return minIndex;
    }
}
```
Time Complexity: O(n) — every element must be inspected.
Space Complexity: O(1) — no extra space used.

**Optimized Approach:** same modified binary search as "Find Minimum", but track the index of the minimum instead of its value.
```csharp
public class Solution
{
    public int FindRotationCount(int[] arr)
    {
        int low = 0, high = arr.Length - 1;
        int minValue = int.MaxValue;
        int minIndex = 0;

        while (low <= high)
        {
            int mid = low + (high - low) / 2;

            if (arr[low] <= arr[high])
            {
                // Entire remaining range is already sorted
                if (arr[low] < minValue)
                {
                    minValue = arr[low];
                    minIndex = low;
                }
                break;
            }

            if (arr[low] <= arr[mid])
            {
                // Left half is sorted; its minimum is arr[low]
                if (arr[low] < minValue)
                {
                    minValue = arr[low];
                    minIndex = low;
                }
                low = mid + 1;
            }
            else
            {
                // Right half is sorted; its minimum is arr[mid]
                if (arr[mid] < minValue)
                {
                    minValue = arr[mid];
                    minIndex = mid;
                }
                high = mid - 1;
            }
        }

        return minIndex;
    }
}
```
Time Complexity: O(log n) — search space halves every iteration.
Space Complexity: O(1) — only pointers and running trackers are used.

**Explanation:** The number of rotations performed on a sorted array equals exactly the index at which the minimum element sits — because a left rotation by `k` positions moves the original first `k` elements to the end, pushing the smallest element (which was at index 0) to index `k`. This problem is therefore just problem 3 ("Find Minimum") with the bookkeeping changed to remember `minIndex` alongside `minValue`. The binary search logic — determine the sorted half, compare its boundary value against the current best minimum, then move into the other half — is completely reused.

---

## 5. Single Element in a Sorted Array (every element appears twice except one)

**Problem Statement:** Given a sorted array of integers where every element appears exactly twice except for one element which appears exactly once, find that single element. The solution must run in `O(log n)` time and `O(1)` space.

**Example:**
- Input: `arr = [1,1,2,3,3,4,4,8,8]`
- Output: `2`
- Explanation: All elements appear twice except `2`, which appears once.

**Brute Force Approach:** linear scan O(n).
```csharp
public class Solution
{
    public int SingleNonDuplicateBruteForce(int[] arr)
    {
        int n = arr.Length;

        if (n == 1)
            return arr[0];

        for (int i = 0; i < n; i++)
        {
            bool matchesPrev = (i > 0) && arr[i] == arr[i - 1];
            bool matchesNext = (i < n - 1) && arr[i] == arr[i + 1];

            if (!matchesPrev && !matchesNext)
                return arr[i];
        }

        return -1; // should not happen given problem constraints
    }
}
```
Time Complexity: O(n) — every element may need to be checked.
Space Complexity: O(1) — no extra space used.

**Optimized Approach:** even/odd index parity trick combined with binary search.
```csharp
public class Solution
{
    public int SingleNonDuplicate(int[] arr)
    {
        int n = arr.Length;

        if (n == 1)
            return arr[0];

        int low = 0, high = n - 2; // search only within valid pairs

        while (low <= high)
        {
            int mid = low + (high - low) / 2;

            // Normalize mid to the even index of its pair
            int evenIndex = (mid % 2 == 0) ? mid : mid - 1;

            if (arr[evenIndex] == arr[evenIndex + 1])
            {
                // Pair is intact; single element lies after this pair
                low = mid + 1;
            }
            else
            {
                // Pair is broken; single element lies at or before this pair
                high = mid - 1;
            }
        }

        // low now points to the index of the single element
        return arr[low];
    }
}
```
Time Complexity: O(log n) — search space halves every iteration.
Space Complexity: O(1) — only pointers are used.

**Explanation:** Before the single element, every pair is aligned so that the first occurrence sits at an even index and the second occurrence sits at the very next (odd) index — e.g., pairs look like `(even, even+1)`. After the single element disrupts this pattern, that alignment flips: the first occurrence of each subsequent pair now sits at an odd index instead. So the parity trick is: for any index `i` before the single element, if `i` is even then `arr[i] == arr[i+1]`; if `i` is odd then `arr[i] == arr[i-1]`. Using this, at each `mid` we normalize to the even index of its pair (`evenIndex`) and check whether `arr[evenIndex] == arr[evenIndex + 1]`:
- If the pair is intact, the disruption (and thus the single element) must lie to the right, so we move `low = mid + 1`.
- If the pair is broken, the disruption is at or before this pair, so we move `high = mid - 1`.

Dry run on `arr = [1,1,2,3,3,4,4,8,8]` (n = 9): `low = 0, high = 7`.
- `mid = 3` → odd, so `evenIndex = 2`. `arr[2] = 2, arr[3] = 3` → pair broken → `high = 2`.
- `mid = 1` → odd, so `evenIndex = 0`. `arr[0] = 1, arr[1] = 1` → pair intact → `low = 2`.
- `low = 2, high = 2`, `mid = 2` → even, `evenIndex = 2`. `arr[2] = 2, arr[3] = 3` → pair broken → `high = 1`.
- Loop ends (`low = 2 > high = 1`). Return `arr[low] = arr[2] = 2`. Correct.

This binary search over pair parity converges on the single element in O(log n) time by discarding half of the intact pairs at each step, just like the rotation-based searches above discard half of the array based on which portion is "well-behaved" (sorted, or in this case, correctly paired).
