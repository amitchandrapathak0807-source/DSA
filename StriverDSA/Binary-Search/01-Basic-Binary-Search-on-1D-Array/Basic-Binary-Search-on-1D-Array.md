# Binary Search — Basic Binary Search on 1D Arrays

## 1. Binary Search — Find X in a Sorted Array

**Problem Statement:** Given a sorted array of `n` integers and a target value `x`, find the index of `x` in the array. If `x` does not exist in the array, return `-1`. The array is sorted in ascending order and (for this basic version) contains distinct elements.

**Example:**
- Input: `arr = [3, 4, 4, 4, 5, 8, 10], target = 4`
- Output: `1` (or any valid index where `4` occurs — for this problem we assume distinct elements, so consider `arr = [1, 3, 5, 7, 9, 11], target = 7`)
- Output: `3`
- Explanation: The element `7` is present at index `3` (0-indexed) in the sorted array.

**Brute Force Approach:** Scan every element from left to right and compare it with the target. Since the array is sorted, this works but does not exploit the ordering.

**Logic (Steps):**
1. Loop `i` from `0` to `arr.Length - 1`.
2. Compare `arr[i]` with `target`.
3. If they match, return `i` immediately.
4. If the loop finishes with no match, return `-1`.

```csharp
public class Solution
{
    public int SearchBruteForce(int[] arr, int target)
    {
        for (int i = 0; i < arr.Length; i++)
        {
            if (arr[i] == target)
            {
                return i;
            }
        }
        return -1;
    }
}
```
Time Complexity: O(n) — every element may need to be checked.
Space Complexity: O(1) — no extra space used.

**Walkthrough:** Using `arr = [1, 3, 5, 7, 9, 11], target = 7`.
- `i=0`: `1≠7`. `i=1`: `3≠7`. `i=2`: `5≠7`. `i=3`: `7==7` → return `3`.
Returned value `3` matches the expected output.

---

**Optimized Approach:** Since the array is sorted, repeatedly compare the target with the middle element and discard half of the search space each time. This is the classic binary search algorithm.

**Logic (Steps):**
1. Initialize `low = 0`, `high = arr.Length - 1`.
2. While `low <= high`, compute `mid = low + (high - low) / 2`.
3. If `arr[mid] == target`, return `mid` immediately.
4. If `arr[mid] < target`, the target must be in the right half, so set `low = mid + 1`; otherwise set `high = mid - 1` (search left half).
5. If the loop exits without a match, return `-1`.

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
            {
                return mid;
            }
            else if (arr[mid] < target)
            {
                low = mid + 1;
            }
            else
            {
                high = mid - 1;
            }
        }

        return -1;
    }
}
```

Time Complexity: O(log n), Space Complexity: O(1).

**Walkthrough:** Dry run on `arr = [1, 3, 5, 7, 9, 11], target = 7` (indices 0..5):
- Iteration 1: `low=0, high=5`, `mid=2`. `arr[2]=5 < 7` → `low=3`.
- Iteration 2: `low=3, high=5`, `mid=4`. `arr[4]=9 > 7` → `high=3`.
- Iteration 3: `low=3, high=3`, `mid=3`. `arr[3]=7` → match, return `3`.
Returned value `3` matches the expected output — 3 comparisons instead of up to 6 in the linear scan.

---

## 2. Implement Lower Bound

**Problem Statement:** Given a sorted array `arr` and an integer `x`, find the index of the first element in the array that is **greater than or equal to** `x` (the lower bound). If no such element exists, return `n` (the length of the array). Formally, lower bound is the smallest index `ind` such that `arr[ind] >= x`.

**Example:**
- Input: `arr = [3, 4, 4, 4, 5, 8, 10], target = 4`
- Output: `1`
- Explanation: `arr[1] = 4` is the first element that is `>= 4`.

**Brute Force Approach:** Traverse the array from left to right and return the index of the first element that is `>= x`.

**Logic (Steps):**
1. Loop `i` from `0` to `n-1`.
2. Check if `arr[i] >= x`.
3. If so, return `i` immediately (first qualifying index).
4. If the loop finishes with no match, return `n`.

```csharp
public class Solution
{
    public int LowerBoundBruteForce(int[] arr, int x)
    {
        int n = arr.Length;
        for (int i = 0; i < n; i++)
        {
            if (arr[i] >= x)
            {
                return i;
            }
        }
        return n;
    }
}
```
Time Complexity: O(n), Space Complexity: O(1).

**Walkthrough:** Using `arr = [3, 4, 4, 4, 5, 8, 10], x = 4`.
- `i=0`: `3>=4`? No. `i=1`: `4>=4`? Yes → return `1`.
Returned value `1` matches the expected output.

---

**Optimized Approach:** Use binary search. Whenever `arr[mid] >= x`, this index is a *candidate* answer, so record it and try to find an even smaller valid index by searching the left half. Otherwise, move to the right half.

**Logic (Steps):**
1. Initialize `low = 0`, `high = n - 1`, and `ans = n` (default when no element qualifies).
2. While `low <= high`, compute `mid = low + (high - low) / 2`.
3. If `arr[mid] >= x`, record `ans = mid` as a candidate and search the left half (`high = mid - 1`) for a smaller valid index.
4. Otherwise, `arr[mid] < x`, so discard the left half (`low = mid + 1`).
5. Return `ans` once the loop ends.

```csharp
public class Solution
{
    public int LowerBound(int[] arr, int x)
    {
        int n = arr.Length;
        int low = 0, high = n - 1;
        int ans = n; // default: no element is >= x

        while (low <= high)
        {
            int mid = low + (high - low) / 2;

            if (arr[mid] >= x)
            {
                ans = mid;      // candidate answer
                high = mid - 1; // look for a smaller index on the left
            }
            else
            {
                low = mid + 1;  // arr[mid] < x, discard left half
            }
        }

        return ans;
    }
}
```

Time Complexity: O(log n), Space Complexity: O(1).

**Walkthrough:** Dry run on `arr = [3, 4, 4, 4, 5, 8, 10], x = 4` (indices 0..6):
- Iteration 1: `low=0, high=6`, `mid=3`. `arr[3]=4 >= 4` → `ans=3`, `high=2`.
- Iteration 2: `low=0, high=2`, `mid=1`. `arr[1]=4 >= 4` → `ans=1`, `high=0`.
- Iteration 3: `low=0, high=0`, `mid=0`. `arr[0]=3 < 4` → `low=1`.
- `low=1 > high=0`, loop ends.
Final `ans = 1`, matching the expected output.

---

## 3. Implement Upper Bound

**Problem Statement:** Given a sorted array `arr` and an integer `x`, find the index of the first element in the array that is **strictly greater than** `x` (the upper bound). If no such element exists, return `n` (the length of the array). Formally, upper bound is the smallest index `ind` such that `arr[ind] > x`.

**Example:**
- Input: `arr = [3, 4, 4, 4, 5, 8, 10], target = 4`
- Output: `4`
- Explanation: `arr[4] = 5` is the first element that is strictly `> 4`.

**Brute Force Approach:** Traverse the array from left to right and return the index of the first element strictly greater than `x`.

**Logic (Steps):**
1. Loop `i` from `0` to `n-1`.
2. Check if `arr[i] > x`.
3. If so, return `i` immediately.
4. If the loop finishes with no match, return `n`.

```csharp
public class Solution
{
    public int UpperBoundBruteForce(int[] arr, int x)
    {
        int n = arr.Length;
        for (int i = 0; i < n; i++)
        {
            if (arr[i] > x)
            {
                return i;
            }
        }
        return n;
    }
}
```
Time Complexity: O(n), Space Complexity: O(1).

**Walkthrough:** Using `arr = [3, 4, 4, 4, 5, 8, 10], x = 4`.
- `i=0..3`: `3,4,4,4` none `>4`. `i=4`: `5>4` → return `4`.
Returned value `4` matches the expected output.

---

**Optimized Approach:** This is almost identical to lower bound, except the comparison condition uses strict inequality (`arr[mid] > x` instead of `arr[mid] >= x`).

**Logic (Steps):**
1. Initialize `low = 0`, `high = n - 1`, `ans = n`.
2. While `low <= high`, compute `mid = low + (high - low) / 2`.
3. If `arr[mid] > x`, record `ans = mid` and search left (`high = mid - 1`) for a smaller valid index.
4. Otherwise, `arr[mid] <= x`, so discard the left half (`low = mid + 1`).
5. Return `ans` once the loop ends.

```csharp
public class Solution
{
    public int UpperBound(int[] arr, int x)
    {
        int n = arr.Length;
        int low = 0, high = n - 1;
        int ans = n; // default: no element is > x

        while (low <= high)
        {
            int mid = low + (high - low) / 2;

            if (arr[mid] > x)
            {
                ans = mid;      // candidate answer
                high = mid - 1; // look for a smaller index on the left
            }
            else
            {
                low = mid + 1;  // arr[mid] <= x, discard left half
            }
        }

        return ans;
    }
}
```

Time Complexity: O(log n), Space Complexity: O(1).

**Walkthrough:** On `arr = [3, 4, 4, 4, 5, 8, 10], x = 4`:
- `mid=3`: `arr[3]=4`, not `>4` → `low=4`.
- `mid=5`: `arr[5]=8 > 4` → `ans=5`, `high=4`.
- `mid=4`: `arr[4]=5 > 4` → `ans=4`, `high=3`.
- `low=4 > high=3`, loop ends.
Final `ans = 4`, matching the expected output.

---

## 4. Search Insert Position

**Problem Statement:** Given a sorted array of distinct integers `arr` and a target value `x`, return the index if the target is found. If not, return the index where it would be inserted to keep the array sorted (this is exactly the lower bound of `x`).

**Example:**
- Input: `arr = [1, 3, 5, 6], target = 5`
- Output: `2`
- Explanation: `5` is found at index `2`.

- Input: `arr = [1, 3, 5, 6], target = 2`
- Output: `1`
- Explanation: `2` is not present; inserting it at index `1` keeps `[1, 2, 3, 5, 6]` sorted.

**Brute Force Approach:** Scan the array and return the index of the first element that is `>= x`; if none found, the insert position is at the end (`n`).

**Logic (Steps):**
1. Loop `i` from `0` to `n-1`.
2. Check if `arr[i] >= x`.
3. If so, return `i` immediately.
4. If the loop finishes with no match, return `n` (insert at the end).

```csharp
public class Solution
{
    public int SearchInsertBruteForce(int[] arr, int x)
    {
        int n = arr.Length;
        for (int i = 0; i < n; i++)
        {
            if (arr[i] >= x)
            {
                return i;
            }
        }
        return n;
    }
}
```
Time Complexity: O(n), Space Complexity: O(1).

**Walkthrough:** Using `arr = [1, 3, 5, 6], x = 2`.
- `i=0`: `1>=2`? No. `i=1`: `3>=2`? Yes → return `1`.
Returned value `1` matches the expected output.

---

**Optimized Approach:** The search insert position is exactly the **lower bound** of `x` in the array — the first index where `arr[index] >= x`. We reuse the lower bound binary search logic from Problem 2 directly.

**Logic (Steps):**
1. Initialize `low = 0`, `high = n - 1`, `ans = n`.
2. While `low <= high`, compute `mid = low + (high - low) / 2`.
3. If `arr[mid] >= x`, record `ans = mid` and search left (`high = mid - 1`).
4. Otherwise, discard the left half (`low = mid + 1`).
5. Return `ans`, which is the target's index if found, or its correct insert position otherwise.

```csharp
public class Solution
{
    public int SearchInsert(int[] arr, int x)
    {
        int n = arr.Length;
        int low = 0, high = n - 1;
        int ans = n;

        while (low <= high)
        {
            int mid = low + (high - low) / 2;

            if (arr[mid] >= x)
            {
                ans = mid;
                high = mid - 1;
            }
            else
            {
                low = mid + 1;
            }
        }

        return ans;
    }
}
```

Time Complexity: O(log n), Space Complexity: O(1).

**Walkthrough:** For `arr = [1, 3, 5, 6], x = 2`:
- `low=0, high=3, mid=1`: `arr[1]=3 >= 2` → `ans=1`, `high=0`.
- `low=0, high=0, mid=0`: `arr[0]=1 < 2` → `low=1`.
- Loop ends (`low > high`).
Final `ans = 1`, matching the expected output.

---

## 5. Floor and Ceil in a Sorted Array

**Problem Statement:** Given a sorted array `arr` and an integer `x`, find the **floor** (the largest element in the array `<= x`) and the **ceil** (the smallest element in the array `>= x`). If floor or ceil does not exist, return `-1` for that value.

**Example:**
- Input: `arr = [3, 4, 4, 4, 5, 8, 10], x = 7`
- Output: `Floor = 5, Ceil = 8`
- Explanation: The largest value `<= 7` is `5`; the smallest value `>= 7` is `8`.

**Brute Force Approach:** Traverse the array once, tracking the largest value seen so far that is `<= x` (floor) and the first value encountered that is `>= x` (ceil).

**Logic (Steps):**
1. Initialize `floor = -1`, `ceil = -1`.
2. Loop through each `arr[i]`.
3. If `arr[i] <= x`, keep overwriting `floor = arr[i]` (the last such value seen is the largest one `<= x`).
4. If `arr[i] >= x` and `ceil` hasn't been set yet, set `ceil = arr[i]` (the first such value is the smallest one `>= x`).
5. Return `(floor, ceil)` after the loop.

```csharp
public class Solution
{
    public (int Floor, int Ceil) FindFloorCeilBruteForce(int[] arr, int x)
    {
        int floor = -1, ceil = -1;

        for (int i = 0; i < arr.Length; i++)
        {
            if (arr[i] <= x)
            {
                floor = arr[i]; // keep updating; last one <= x is the floor
            }
            if (arr[i] >= x && ceil == -1)
            {
                ceil = arr[i]; // first one >= x is the ceil
            }
        }

        return (floor, ceil);
    }
}
```

Time Complexity: O(n), Space Complexity: O(1).

**Walkthrough:** Using `arr = [3, 4, 4, 4, 5, 8, 10], x = 7`.
- Scanning left to right: `floor` gets updated to `3, 4, 4, 4, 5` (last one `<= 7` is `5`).
- `ceil` is set the first time `arr[i] >= 7`, which is `8`.
Returned `(Floor, Ceil) = (5, 8)`, matching the expected output.

---

**Optimized Approach:** Ceil is the value at the **lower bound** index of `x` (first element `>= x`). Floor is the value just before the **upper bound** index of `x` (last element `<= x`, i.e., one position before the first element `> x`). We can compute both with a single binary search each, reusing lower/upper bound ideas.

**Logic (Steps):**
1. `FindFloor`: initialize `low=0, high=n-1, ans=-1`; while `low<=high`, if `arr[mid] <= x`, record `ans = arr[mid]` as a candidate and move right (`low = mid+1`) to look for an even larger value still `<= x`; otherwise move left (`high = mid-1`).
2. `FindCeil`: same pattern but with `arr[mid] >= x`, recording `ans = arr[mid]` and moving left (`high = mid-1`) to look for a smaller value still `>= x`; otherwise move right (`low = mid+1`).
3. Call `FindFloor(arr, x)` and `FindCeil(arr, x)` independently, each in `O(log n)`.
4. Return the pair `(floor, ceil)`.

```csharp
public class Solution
{
    public (int Floor, int Ceil) FindFloorAndCeil(int[] arr, int x)
    {
        int floor = FindFloor(arr, x);
        int ceil = FindCeil(arr, x);
        return (floor, ceil);
    }

    private int FindFloor(int[] arr, int x)
    {
        int low = 0, high = arr.Length - 1;
        int ans = -1;

        while (low <= high)
        {
            int mid = low + (high - low) / 2;

            if (arr[mid] <= x)
            {
                ans = arr[mid]; // candidate floor
                low = mid + 1;  // try to find a larger value still <= x
            }
            else
            {
                high = mid - 1;
            }
        }

        return ans;
    }

    private int FindCeil(int[] arr, int x)
    {
        int low = 0, high = arr.Length - 1;
        int ans = -1;

        while (low <= high)
        {
            int mid = low + (high - low) / 2;

            if (arr[mid] >= x)
            {
                ans = arr[mid]; // candidate ceil
                high = mid - 1; // try to find a smaller value still >= x
            }
            else
            {
                low = mid + 1;
            }
        }

        return ans;
    }
}
```

Time Complexity: O(log n) (two binary searches, each O(log n)), Space Complexity: O(1).

**Walkthrough:** On `arr = [3, 4, 4, 4, 5, 8, 10], x = 7`:
- Ceil search: `mid=3` (`arr=4`, not `>=7`) → `low=4`; `mid=5` (`arr=8>=7`) → `ans=8`, `high=4`; `mid=4` (`arr=5`, not `>=7`) → `low=5`; loop ends, ceil `= 8`.
- Floor search: `mid=3` (`arr=4<=7`) → `ans=4`, `low=4`; `mid=5` (`arr=8`, not `<=7`) → `high=4`; `mid=4` (`arr=5<=7`) → `ans=5`, `low=5`; loop ends (`low>high`), floor `= 5`.
Returned `(Floor, Ceil) = (5, 8)`, matching the expected output.

---

## 6. Find First and Last Occurrence of an Element in a Sorted Array

**Problem Statement:** Given a sorted array `arr` (which may contain duplicates) and a target value `x`, find the index of the first and last occurrence of `x` in the array. If `x` is not present, return `[-1, -1]`.

**Example:**
- Input: `arr = [3, 4, 4, 4, 5, 8, 10], target = 4`
- Output: `First = 1, Last = 3`
- Explanation: `4` first appears at index `1` and last appears at index `3`.

**Brute Force Approach:** Scan the whole array once, recording the first index where `arr[i] == x` and continuously updating the last index where `arr[i] == x`.

**Logic (Steps):**
1. Initialize `first = -1`, `last = -1`.
2. Loop through each index `i`.
3. If `arr[i] == x`, set `first = i` only if it hasn't been set yet (first match wins).
4. Regardless, keep overwriting `last = i` on every match (the final match wins).
5. Return `(first, last)` after the loop.

```csharp
public class Solution
{
    public (int First, int Last) FindFirstLastBruteForce(int[] arr, int x)
    {
        int first = -1, last = -1;

        for (int i = 0; i < arr.Length; i++)
        {
            if (arr[i] == x)
            {
                if (first == -1)
                {
                    first = i;
                }
                last = i;
            }
        }

        return (first, last);
    }
}
```

Time Complexity: O(n), Space Complexity: O(1).

**Walkthrough:** Using `arr = [3, 4, 4, 4, 5, 8, 10], x = 4`.
- `i=1`: `arr[1]=4==4` → `first=1`, `last=1`. `i=2`: match → `last=2`. `i=3`: match → `last=3`.
Returned `(First, Last) = (1, 3)`, matching the expected output.

---

**Optimized Approach:** The first occurrence of `x` is exactly the **lower bound** index of `x` (provided that index actually holds `x`). The last occurrence is exactly one position before the **upper bound** index of `x` (i.e., `upperBound(x) - 1`). We reuse the lower bound / upper bound logic from Problems 2 and 3.

**Logic (Steps):**
1. Compute `lb = LowerBound(arr, x)` — the first index with `arr[index] >= x`.
2. If `lb == n` or `arr[lb] != x`, `x` isn't present, so return `(-1, -1)`.
3. Otherwise compute `ub = UpperBound(arr, x)` — the first index with `arr[index] > x`.
4. The last occurrence is `last = ub - 1` (the position right before the first element greater than `x`).
5. Return `(lb, last)`.

```csharp
public class Solution
{
    public (int First, int Last) FindFirstAndLast(int[] arr, int x)
    {
        int n = arr.Length;
        int lb = LowerBound(arr, x); // first index with arr[index] >= x

        if (lb == n || arr[lb] != x)
        {
            return (-1, -1); // x doesn't exist in the array
        }

        int ub = UpperBound(arr, x); // first index with arr[index] > x
        int last = ub - 1;           // last occurrence of x

        return (lb, last);
    }

    private int LowerBound(int[] arr, int x)
    {
        int low = 0, high = arr.Length - 1, ans = arr.Length;

        while (low <= high)
        {
            int mid = low + (high - low) / 2;

            if (arr[mid] >= x)
            {
                ans = mid;
                high = mid - 1;
            }
            else
            {
                low = mid + 1;
            }
        }

        return ans;
    }

    private int UpperBound(int[] arr, int x)
    {
        int low = 0, high = arr.Length - 1, ans = arr.Length;

        while (low <= high)
        {
            int mid = low + (high - low) / 2;

            if (arr[mid] > x)
            {
                ans = mid;
                high = mid - 1;
            }
            else
            {
                low = mid + 1;
            }
        }

        return ans;
    }
}
```

Time Complexity: O(log n) (two binary searches back to back), Space Complexity: O(1).

**Walkthrough:** Dry run on `arr = [3, 4, 4, 4, 5, 8, 10], x = 4` (indices 0..6):
- Lower bound: `mid=3` (`arr=4>=4`) → `ans=3, high=2`; `mid=1` (`arr=4>=4`) → `ans=1, high=0`; `mid=0` (`arr=3<4`) → `low=1`; loop ends. Lower bound `= 1`, and `arr[1]==4`, so `first=1`.
- Upper bound: `mid=3` (`arr=4`, not `>4`) → `low=4`; `mid=5` (`arr=8>4`) → `ans=5, high=4`; `mid=4` (`arr=5>4`) → `ans=4, high=3`; loop ends. Upper bound `= 4`, so `last = 4-1 = 3`.
Returned `(First, Last) = (1, 3)`, matching the expected output.

---

## 7. Count Occurrences of a Number in a Sorted Array

**Problem Statement:** Given a sorted array `arr` (which may contain duplicates) and an integer `x`, count how many times `x` occurs in the array.

**Example:**
- Input: `arr = [3, 4, 4, 4, 5, 8, 10], target = 4`
- Output: `3`
- Explanation: `4` occurs at indices `1`, `2`, and `3`, i.e., 3 times.

**Brute Force Approach:** Traverse the array once and increment a counter every time `arr[i] == x`.

**Logic (Steps):**
1. Initialize `count = 0`.
2. Loop through each `arr[i]`.
3. If `arr[i] == x`, increment `count`.
4. Return `count` after the loop.

```csharp
public class Solution
{
    public int CountOccurrencesBruteForce(int[] arr, int x)
    {
        int count = 0;

        for (int i = 0; i < arr.Length; i++)
        {
            if (arr[i] == x)
            {
                count++;
            }
        }

        return count;
    }
}
```

Time Complexity: O(n), Space Complexity: O(1).

**Walkthrough:** Using `arr = [3, 4, 4, 4, 5, 8, 10], x = 4`.
- Matches occur at `i=1,2,3` → `count` increments three times to `3`.
Returned value `3` matches the expected output.

---

**Optimized Approach:** The count of occurrences of `x` is simply `lastOccurrence - firstOccurrence + 1`. We directly reuse the first-and-last-occurrence logic from Problem 6, which itself is built on lower bound / upper bound.

**Logic (Steps):**
1. Compute `lb = LowerBound(arr, x)`.
2. If `lb == n` or `arr[lb] != x`, `x` isn't present, so return `0`.
3. Otherwise compute `ub = UpperBound(arr, x)` and `last = ub - 1`.
4. Return `last - lb + 1` (the count of occurrences between first and last, inclusive).

```csharp
public class Solution
{
    public int CountOccurrences(int[] arr, int x)
    {
        int n = arr.Length;
        int lb = LowerBound(arr, x);

        if (lb == n || arr[lb] != x)
        {
            return 0; // x doesn't exist in the array
        }

        int ub = UpperBound(arr, x);
        int last = ub - 1;

        return last - lb + 1; // lastOccurrence - firstOccurrence + 1
    }

    private int LowerBound(int[] arr, int x)
    {
        int low = 0, high = arr.Length - 1, ans = arr.Length;

        while (low <= high)
        {
            int mid = low + (high - low) / 2;

            if (arr[mid] >= x)
            {
                ans = mid;
                high = mid - 1;
            }
            else
            {
                low = mid + 1;
            }
        }

        return ans;
    }

    private int UpperBound(int[] arr, int x)
    {
        int low = 0, high = arr.Length - 1, ans = arr.Length;

        while (low <= high)
        {
            int mid = low + (high - low) / 2;

            if (arr[mid] > x)
            {
                ans = mid;
                high = mid - 1;
            }
            else
            {
                low = mid + 1;
            }
        }

        return ans;
    }
}
```

Time Complexity: O(log n) (two binary searches), Space Complexity: O(1).

**Walkthrough:** Using `arr = [3, 4, 4, 4, 5, 8, 10], x = 4`:
- `lb = LowerBound(arr, 4) = 1`.
- `ub = UpperBound(arr, 4) = 4`, so `last = 4 - 1 = 3`.
- Return `last - lb + 1 = 3 - 1 + 1 = 3`.
Returned value `3` matches the expected output, computed in O(log n) instead of a full O(n) scan.
