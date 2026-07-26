# Binary Search — Basic Binary Search on 1D Arrays

## 1. Binary Search — Find X in a Sorted Array

**Problem Statement:** Given a sorted array of `n` integers and a target value `x`, find the index of `x` in the array. If `x` does not exist in the array, return `-1`. The array is sorted in ascending order and (for this basic version) contains distinct elements.

**Example:**
- Input: `arr = [3, 4, 4, 4, 5, 8, 10], target = 4`
- Output: `1` (or any valid index where `4` occurs — for this problem we assume distinct elements, so consider `arr = [1, 3, 5, 7, 9, 11], target = 7`)
- Output: `3`
- Explanation: The element `7` is present at index `3` (0-indexed) in the sorted array.

**Brute Force Approach:** Scan every element from left to right and compare it with the target. Since the array is sorted, this works but does not exploit the ordering.

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

**Optimized Approach:** Since the array is sorted, repeatedly compare the target with the middle element and discard half of the search space each time. This is the classic binary search algorithm.

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

**Explanation:** Dry run on `arr = [1, 3, 5, 7, 9, 11], target = 7` (indices 0..5):

- Iteration 1: `low = 0, high = 5`, `mid = 0 + (5 - 0) / 2 = 2`. `arr[2] = 5`. Since `5 < 7`, discard the left half and set `low = mid + 1 = 3`.
- Iteration 2: `low = 3, high = 5`, `mid = 3 + (5 - 3) / 2 = 4`. `arr[4] = 9`. Since `9 > 7`, discard the right half and set `high = mid - 1 = 3`.
- Iteration 3: `low = 3, high = 3`, `mid = 3 + (3 - 3) / 2 = 3`. `arr[3] = 7`. Match found — return `3`.

The search space shrank from 6 elements → 3 elements → 1 element, taking only 3 comparisons instead of up to 6 in the linear scan.

---

## 2. Implement Lower Bound

**Problem Statement:** Given a sorted array `arr` and an integer `x`, find the index of the first element in the array that is **greater than or equal to** `x` (the lower bound). If no such element exists, return `n` (the length of the array). Formally, lower bound is the smallest index `ind` such that `arr[ind] >= x`.

**Example:**
- Input: `arr = [3, 4, 4, 4, 5, 8, 10], target = 4`
- Output: `1`
- Explanation: `arr[1] = 4` is the first element that is `>= 4`.

**Brute Force Approach:** Traverse the array from left to right and return the index of the first element that is `>= x`.

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

**Optimized Approach:** Use binary search. Whenever `arr[mid] >= x`, this index is a *candidate* answer, so record it and try to find an even smaller valid index by searching the left half. Otherwise, move to the right half.

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

**Explanation:** Dry run on `arr = [3, 4, 4, 4, 5, 8, 10], x = 4` (indices 0..6):

- Iteration 1: `low = 0, high = 6`, `mid = 3`. `arr[3] = 4`, which is `>= 4` → candidate `ans = 3`, shrink right side: `high = mid - 1 = 2`.
- Iteration 2: `low = 0, high = 2`, `mid = 1`. `arr[1] = 4`, which is `>= 4` → better candidate `ans = 1`, shrink right side: `high = mid - 1 = 0`.
- Iteration 3: `low = 0, high = 0`, `mid = 0`. `arr[0] = 3`, which is `< 4` → discard left side: `low = mid + 1 = 1`.
- Now `low = 1 > high = 0`, loop ends.

Final answer: `ans = 1`. Each iteration halved the search space (7 → 3 → 1 elements) while continuously tightening the best candidate index.

---

## 3. Implement Upper Bound

**Problem Statement:** Given a sorted array `arr` and an integer `x`, find the index of the first element in the array that is **strictly greater than** `x` (the upper bound). If no such element exists, return `n` (the length of the array). Formally, upper bound is the smallest index `ind` such that `arr[ind] > x`.

**Example:**
- Input: `arr = [3, 4, 4, 4, 5, 8, 10], target = 4`
- Output: `4`
- Explanation: `arr[4] = 5` is the first element that is strictly `> 4`.

**Brute Force Approach:** Traverse the array from left to right and return the index of the first element strictly greater than `x`.

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

**Optimized Approach:** This is almost identical to lower bound, except the comparison condition uses strict inequality (`arr[mid] > x` instead of `arr[mid] >= x`).

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

**Explanation:** On `arr = [3, 4, 4, 4, 5, 8, 10], x = 4`, binary search proceeds similarly to lower bound but only accepts strictly greater values as candidates: `mid = 3` (`arr[3] = 4`, not `> 4`, move right), `mid = 5` (`arr[5] = 8 > 4`, candidate `ans = 5`, move left), `mid = 4` (`arr[4] = 5 > 4`, candidate `ans = 4`, move left), search ends with `low > high`. Final answer: `4`.

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

**Optimized Approach:** The search insert position is exactly the **lower bound** of `x` in the array — the first index where `arr[index] >= x`. We reuse the lower bound binary search logic from Problem 2 directly.

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

**Explanation:** Because "search insert position" and "lower bound" are the same concept — the first index whose value is `>= x` — the binary search dry run behaves exactly like Problem 2. For `arr = [1, 3, 5, 6], x = 2`: `low=0, high=3, mid=1` → `arr[1]=3 >= 2` → `ans=1, high=0`; `low=0, high=0, mid=0` → `arr[0]=1 < 2` → `low=1`; loop ends since `low > high`. Final answer: `1`.

---

## 5. Floor and Ceil in a Sorted Array

**Problem Statement:** Given a sorted array `arr` and an integer `x`, find the **floor** (the largest element in the array `<= x`) and the **ceil** (the smallest element in the array `>= x`). If floor or ceil does not exist, return `-1` for that value.

**Example:**
- Input: `arr = [3, 4, 4, 4, 5, 8, 10], x = 7`
- Output: `Floor = 5, Ceil = 8`
- Explanation: The largest value `<= 7` is `5`; the smallest value `>= 7` is `8`.

**Brute Force Approach:** Traverse the array once, tracking the largest value seen so far that is `<= x` (floor) and the first value encountered that is `>= x` (ceil).

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

**Optimized Approach:** Ceil is the value at the **lower bound** index of `x` (first element `>= x`). Floor is the value just before the **upper bound** index of `x` (last element `<= x`, i.e., one position before the first element `> x`). We can compute both with a single binary search each, reusing lower/upper bound ideas.

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

**Explanation:** `FindCeil` is structurally identical to the lower bound binary search from Problem 2 (just returning the value instead of the index), and `FindFloor` is its mirror image. On `arr = [3, 4, 4, 4, 5, 8, 10], x = 7`: for ceil, `mid=3 (arr=4) < 7 → low=4`; `mid=5 (arr=8) >= 7 → ans=8, high=4`; `mid=4 (arr=5) < 7 → low=5`; loop ends, ceil `= 8`. For floor, `mid=3 (arr=4) <= 7 → ans=4, low=4`; `mid=5 (arr=8) > 7 → high=4`; `mid=4 (arr=5) <= 7 → ans=5, low=5`; loop ends since `low > high`, floor `= 5`.

---

## 6. Find First and Last Occurrence of an Element in a Sorted Array

**Problem Statement:** Given a sorted array `arr` (which may contain duplicates) and a target value `x`, find the index of the first and last occurrence of `x` in the array. If `x` is not present, return `[-1, -1]`.

**Example:**
- Input: `arr = [3, 4, 4, 4, 5, 8, 10], target = 4`
- Output: `First = 1, Last = 3`
- Explanation: `4` first appears at index `1` and last appears at index `3`.

**Brute Force Approach:** Scan the whole array once, recording the first index where `arr[i] == x` and continuously updating the last index where `arr[i] == x`.

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

**Optimized Approach:** The first occurrence of `x` is exactly the **lower bound** index of `x` (provided that index actually holds `x`). The last occurrence is exactly one position before the **upper bound** index of `x` (i.e., `upperBound(x) - 1`). We reuse the lower bound / upper bound logic from Problems 2 and 3.

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

**Explanation:** Dry run on `arr = [3, 4, 4, 4, 5, 8, 10], x = 4` (indices 0..6):

*Lower bound (first occurrence):*
- `low = 0, high = 6, mid = 3`. `arr[3] = 4 >= 4` → `ans = 3, high = 2`.
- `low = 0, high = 2, mid = 1`. `arr[1] = 4 >= 4` → `ans = 1, high = 0`.
- `low = 0, high = 0, mid = 0`. `arr[0] = 3 < 4` → `low = 1`.
- Loop ends (`low = 1 > high = 0`). Lower bound `= 1`, and `arr[1] == 4`, so `first = 1`.

*Upper bound (to derive last occurrence):*
- `low = 0, high = 6, mid = 3`. `arr[3] = 4`, not `> 4` → `low = 4`.
- `low = 4, high = 6, mid = 5`. `arr[5] = 8 > 4` → `ans = 5, high = 4`.
- `low = 4, high = 4, mid = 4`. `arr[4] = 5 > 4` → `ans = 4, high = 3`.
- Loop ends (`low = 4 > high = 3`). Upper bound `= 4`, so `last = 4 - 1 = 3`.

Final result: `First = 1, Last = 3`, matching the expected output — and computed using only two O(log n) binary searches instead of a full O(n) scan.

---

## 7. Count Occurrences of a Number in a Sorted Array

**Problem Statement:** Given a sorted array `arr` (which may contain duplicates) and an integer `x`, count how many times `x` occurs in the array.

**Example:**
- Input: `arr = [3, 4, 4, 4, 5, 8, 10], target = 4`
- Output: `3`
- Explanation: `4` occurs at indices `1`, `2`, and `3`, i.e., 3 times.

**Brute Force Approach:** Traverse the array once and increment a counter every time `arr[i] == x`.

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

**Optimized Approach:** The count of occurrences of `x` is simply `lastOccurrence - firstOccurrence + 1`. We directly reuse the first-and-last-occurrence logic from Problem 6, which itself is built on lower bound / upper bound.

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

**Explanation:** Using the same example, `arr = [3, 4, 4, 4, 5, 8, 10], x = 4`, the lower bound binary search yields `first = 1` and the upper bound binary search yields `last = ub - 1 = 4 - 1 = 3` (exactly as dry-run in Problem 6). The count is therefore `last - first + 1 = 3 - 1 + 1 = 3`, matching the expected output. This approach avoids scanning all `n` elements and instead does the work in O(log n) time by building directly on the lower bound and upper bound routines.
