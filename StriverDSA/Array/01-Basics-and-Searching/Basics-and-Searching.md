# Array — Basics and Searching

### 1. Largest Element in an Array
**Problem Statement:** Given an array of integers, find the largest element present in the array.

**Example:**
- Input: `arr = [2, 5, 1, 3, 0]`
- Output: `5`
- Explanation: Among all elements, `5` is the greatest.
- Input: `arr = [-3, -1, -7, -2]`
- Output: `-1`
- Explanation: Even with all negative numbers, `-1` is the largest since it is closest to zero.

**Brute Force Approach:** Sort the array and return the last element. Sorting rearranges elements in ascending order, so the maximum naturally lands at the end.
```csharp
public static int LargestElementBrute(int[] arr)
{
    int[] copy = (int[])arr.Clone();
    Array.Sort(copy);
    return copy[copy.Length - 1];
}
```
Time Complexity: O(n log n) — dominated by the sorting step
Space Complexity: O(n) — extra array created to avoid mutating the input (O(log n) to O(n) for sort's internal recursion/temp storage depending on algorithm)

**Optimized Approach:** Traverse the array once, keeping track of the maximum value seen so far. No sorting is needed since we only care about the single largest value.
```csharp
public static int LargestElement(int[] arr)
{
    if (arr == null || arr.Length == 0)
        throw new ArgumentException("Array must not be empty.");

    int maxVal = arr[0];
    for (int i = 1; i < arr.Length; i++)
    {
        if (arr[i] > maxVal)
            maxVal = arr[i];
    }
    return maxVal;
}
```
Time Complexity: O(n) — single pass through the array
Space Complexity: O(1) — only a single variable used for tracking the maximum

---

### 2. Second Largest Element in an Array without sorting
**Problem Statement:** Given an array of integers, find the second largest distinct element without sorting the array.

**Example:**
- Input: `arr = [8, 8, 3, 5, 1]`
- Output: `5`
- Explanation: Largest is `8`, the next distinct value is `5`.
- Input: `arr = [1, 1, 1]`
- Output: `-1` (no second largest exists since all elements are equal)
- Explanation: Since there is no distinct second largest value, return a sentinel like `-1`.

**Brute Force Approach:** Sort the array, then scan from the end to find the first value strictly less than the largest element.
```csharp
public static int SecondLargestBrute(int[] arr)
{
    int[] copy = (int[])arr.Clone();
    Array.Sort(copy);
    int largest = copy[copy.Length - 1];

    for (int i = copy.Length - 2; i >= 0; i--)
    {
        if (copy[i] != largest)
            return copy[i];
    }
    return -1;
}
```
Time Complexity: O(n log n) — sorting dominates
Space Complexity: O(n) — extra array for the sorted copy

**Optimized Approach:** Do a single pass maintaining two variables: `largest` and `secondLargest`. Whenever a new largest is found, the old largest becomes the second largest. Whenever a value falls strictly between the two, it updates `secondLargest`.
```csharp
public static int SecondLargest(int[] arr)
{
    int largest = int.MinValue;
    int secondLargest = int.MinValue;

    foreach (int num in arr)
    {
        if (num > largest)
        {
            secondLargest = largest;
            largest = num;
        }
        else if (num > secondLargest && num != largest)
        {
            secondLargest = num;
        }
    }

    return secondLargest == int.MinValue ? -1 : secondLargest;
}
```
Time Complexity: O(n) — single traversal of the array
Space Complexity: O(1) — only two tracking variables used

**Explanation:** Dry run on `arr = [8, 8, 3, 5, 1]`:
- Start: `largest = MinValue`, `secondLargest = MinValue`
- `8`: `8 > largest` → `secondLargest = MinValue`, `largest = 8`
- `8`: not `> largest` (equal), and `8 != largest` is false → skip
- `3`: `3 > secondLargest (MinValue)` and `3 != 8` → `secondLargest = 3`
- `5`: `5 > secondLargest (3)` and `5 != 8` → `secondLargest = 5`
- `1`: no update
- Final: `largest = 8`, `secondLargest = 5` ✔
The key insight is that the second largest can only change when we either dethrone the current largest (promoting it down) or find a value that beats the current second largest but is not equal to the largest, all in one pass.

---

### 3. Check if an Array is Sorted
**Problem Statement:** Given an array, determine whether it is sorted in non-decreasing (ascending) order.

**Example:**
- Input: `arr = [1, 2, 2, 3, 4]`
- Output: `true`
- Explanation: Each element is less than or equal to the next.
- Input: `arr = [1, 3, 2, 4]`
- Output: `false`
- Explanation: `3 > 2`, so the order is broken at index 1-2.

**Brute Force Approach:** Compare every pair of elements `(i, j)` where `i < j` and check that `arr[i] <= arr[j]` for all such pairs.
```csharp
public static bool IsSortedBrute(int[] arr)
{
    for (int i = 0; i < arr.Length; i++)
    {
        for (int j = i + 1; j < arr.Length; j++)
        {
            if (arr[i] > arr[j])
                return false;
        }
    }
    return true;
}
```
Time Complexity: O(n^2) — checks all pairs
Space Complexity: O(1) — no extra space used

**Optimized Approach:** It suffices to check only adjacent elements. If every adjacent pair satisfies `arr[i] <= arr[i+1]`, the whole array is sorted (this property is transitive).
```csharp
public static bool IsSorted(int[] arr)
{
    for (int i = 0; i < arr.Length - 1; i++)
    {
        if (arr[i] > arr[i + 1])
            return false;
    }
    return true;
}
```
Time Complexity: O(n) — single pass comparing adjacent elements
Space Complexity: O(1) — no extra space used

---

### 4. Remove Duplicates from a Sorted Array (in-place)
**Problem Statement:** Given a sorted array, remove duplicates in-place such that each unique element appears only once, and return the count of unique elements. The relative order must be preserved and the first `k` positions of the array should hold the unique elements.

**Example:**
- Input: `arr = [1, 1, 2, 2, 3, 4, 4]`
- Output: `k = 4`, array becomes `[1, 2, 3, 4, ...]` (first 4 elements matter)
- Explanation: The distinct elements are `1, 2, 3, 4`.
- Input: `arr = []`
- Output: `k = 0`
- Explanation: An empty array has zero unique elements.

**Brute Force Approach:** Use a `HashSet` (or similar) to collect unique elements in order, then copy them back into the original array.
```csharp
public static int RemoveDuplicatesBrute(int[] arr)
{
    if (arr.Length == 0) return 0;

    List<int> unique = new List<int>();
    HashSet<int> seen = new HashSet<int>();

    foreach (int num in arr)
    {
        if (seen.Add(num))
            unique.Add(num);
    }

    for (int i = 0; i < unique.Count; i++)
        arr[i] = unique[i];

    return unique.Count;
}
```
Time Complexity: O(n) average — hash set insertion is O(1) average per element
Space Complexity: O(n) — extra HashSet and List used to track uniqueness

**Optimized Approach:** Since the array is already sorted, duplicates are always adjacent. Use a two-pointer technique: a slow pointer `i` marks the last unique position, and a fast pointer `j` scans forward looking for the next distinct value.
```csharp
public static int RemoveDuplicates(int[] arr)
{
    if (arr.Length == 0) return 0;

    int i = 0;
    for (int j = 1; j < arr.Length; j++)
    {
        if (arr[j] != arr[i])
        {
            i++;
            arr[i] = arr[j];
        }
    }

    return i + 1;
}
```
Time Complexity: O(n) — single pass with two pointers, no hashing needed
Space Complexity: O(1) — modifies the array in-place, no extra data structures

**Explanation:** Dry run on `arr = [1, 1, 2, 2, 3, 4, 4]`, `i = 0`:
- `j=1`: `arr[1]=1 == arr[0]=1` → skip
- `j=2`: `arr[2]=2 != arr[0]=1` → `i=1`, `arr[1]=2` → array: `[1,2,2,2,3,4,4]`
- `j=3`: `arr[3]=2 == arr[1]=2` → skip
- `j=4`: `arr[4]=3 != arr[1]=2` → `i=2`, `arr[2]=3` → array: `[1,2,3,2,3,4,4]`
- `j=5`: `arr[5]=4 != arr[2]=3` → `i=3`, `arr[3]=4` → array: `[1,2,3,4,3,4,4]`
- `j=6`: `arr[6]=4 == arr[3]=4` → skip
- Final: `i+1 = 4`, first 4 elements `[1,2,3,4]` are the unique values, exactly as expected. Because the array is sorted, the pointer `i` always points at the last confirmed-unique element, so we never need to search backward or use extra memory.

---

### 5. Linear Search
**Problem Statement:** Given an array and a target value, find the index of the target in the array. Return `-1` if the target is not present.

**Example:**
- Input: `arr = [4, 2, 7, 1, 9]`, `target = 7`
- Output: `2`
- Explanation: `arr[2] = 7` matches the target.
- Input: `arr = [4, 2, 7, 1, 9]`, `target = 5`
- Output: `-1`
- Explanation: `5` does not exist anywhere in the array.

**Brute Force Approach:** Since the array is unsorted (or order is unknown), the only reliable way is to check every element one by one until a match is found. This brute force approach IS the optimal approach for an unsorted array — there is no faster general strategy without preprocessing (like sorting or hashing) since the target could be anywhere.
```csharp
public static int LinearSearch(int[] arr, int target)
{
    for (int i = 0; i < arr.Length; i++)
    {
        if (arr[i] == target)
            return i;
    }
    return -1;
}
```
Time Complexity: O(n) — worst case checks every element (target absent or at the last index)
Space Complexity: O(1) — no extra space used

---

### 6. Find the Union of Two Sorted Arrays
**Problem Statement:** Given two sorted arrays, find their union — a sorted collection of all distinct elements present in either array.

**Example:**
- Input: `arr1 = [1, 2, 2, 3, 4]`, `arr2 = [2, 3, 5, 6]`
- Output: `[1, 2, 3, 4, 5, 6]`
- Explanation: All distinct values from both arrays, merged and sorted.
- Input: `arr1 = []`, `arr2 = [1, 1, 2]`
- Output: `[1, 2]`
- Explanation: Union still removes duplicates even if one array is empty.

**Brute Force Approach:** Insert all elements from both arrays into a `HashSet` to remove duplicates, then sort the result.
```csharp
public static List<int> UnionBrute(int[] arr1, int[] arr2)
{
    HashSet<int> set = new HashSet<int>();

    foreach (int num in arr1) set.Add(num);
    foreach (int num in arr2) set.Add(num);

    List<int> result = new List<int>(set);
    result.Sort();
    return result;
}
```
Time Complexity: O((n + m) log(n + m)) — dominated by sorting the combined set of elements
Space Complexity: O(n + m) — HashSet and result list storage

**Optimized Approach:** Since both arrays are already sorted, use the two-pointer technique to merge them while skipping duplicates, similar to the merge step of merge sort. This avoids sorting entirely.
```csharp
public static List<int> Union(int[] arr1, int[] arr2)
{
    List<int> result = new List<int>();
    int i = 0, j = 0;

    while (i < arr1.Length && j < arr2.Length)
    {
        if (arr1[i] <= arr2[j])
        {
            if (result.Count == 0 || result[result.Count - 1] != arr1[i])
                result.Add(arr1[i]);
            i++;
        }
        else
        {
            if (result.Count == 0 || result[result.Count - 1] != arr2[j])
                result.Add(arr2[j]);
            j++;
        }
    }

    while (i < arr1.Length)
    {
        if (result.Count == 0 || result[result.Count - 1] != arr1[i])
            result.Add(arr1[i]);
        i++;
    }

    while (j < arr2.Length)
    {
        if (result.Count == 0 || result[result.Count - 1] != arr2[j])
            result.Add(arr2[j]);
        j++;
    }

    return result;
}
```
Time Complexity: O(n + m) — each pointer advances at most once per element across both arrays
Space Complexity: O(n + m) — result list holds the union (excluding the output, auxiliary space is O(1))

**Explanation:** Dry run on `arr1 = [1, 2, 2, 3, 4]`, `arr2 = [2, 3, 5, 6]`:
- `i=0,j=0`: `1 <= 2` → add `1` → result `[1]`, `i=1`
- `i=1,j=0`: `2 <= 2` → last added is `1 != 2` → add `2` → result `[1,2]`, `i=2`
- `i=2,j=0`: `2 <= 2` → last added is `2 == 2` → skip add, `i=3`
- `i=3,j=0`: `3 <= 2`? No → compare `arr2[0]=2` vs last `2` → skip add, `j=1`
- `i=3,j=1`: `3 <= 3` → last is `2 != 3` → add `3` → result `[1,2,3]`, `i=4`
- `i=4,j=1`: `4 <= 3`? No → `arr2[1]=3` vs last `3` → skip, `j=2`
- `i=4,j=2`: `4 <= 5` → last `3 != 4` → add `4` → result `[1,2,3,4]`, `i=5` (arr1 exhausted)
- Remaining `arr2` from `j=2`: `5` (last `4 != 5`, add), `6` (last `5 != 6`, add)
- Final: `[1,2,3,4,5,6]` ✔
Because both arrays are pre-sorted, comparing the last inserted element to the current candidate is enough to detect duplicates in O(1), so no hashing or post-sort is required.

---

### 7. Find the Missing Number in an Array (1 to N)
**Problem Statement:** Given an array containing `n` distinct numbers taken from the range `1` to `n+1` with exactly one number missing, find the missing number.

**Example:**
- Input: `arr = [1, 2, 4, 5]` (range is 1 to 5)
- Output: `3`
- Explanation: All numbers from 1 to 5 should be present except `3`.
- Input: `arr = [2, 3, 1, 5]` (range is 1 to 5)
- Output: `4`
- Explanation: `4` is missing regardless of the order of the other elements.

**Brute Force Approach:** For every number from `1` to `n+1`, linearly search the array to check if it is present. Return the first number not found.
```csharp
public static int FindMissingBrute(int[] arr)
{
    int n = arr.Length + 1;

    for (int num = 1; num <= n; num++)
    {
        bool found = false;
        for (int i = 0; i < arr.Length; i++)
        {
            if (arr[i] == num)
            {
                found = true;
                break;
            }
        }
        if (!found)
            return num;
    }

    return -1;
}
```
Time Complexity: O(n^2) — for each of the n+1 candidate numbers, scan the whole array
Space Complexity: O(1) — no extra space used

**Optimized Approach:** Use the mathematical sum formula. The sum of numbers from `1` to `n` is `n*(n+1)/2`. Subtract the actual sum of the array elements from this expected sum; the difference is the missing number.
```csharp
public static int FindMissing(int[] arr)
{
    int n = arr.Length + 1;
    long expectedSum = (long)n * (n + 1) / 2;

    long actualSum = 0;
    foreach (int num in arr)
        actualSum += num;

    return (int)(expectedSum - actualSum);
}
```
Time Complexity: O(n) — one pass to compute the actual sum
Space Complexity: O(1) — only a couple of accumulator variables used

**Explanation:** For `arr = [1, 2, 4, 5]`, `n = 5`, expected sum `= 5*6/2 = 15`. Actual sum `= 1+2+4+5 = 12`. Missing `= 15 - 12 = 3` ✔. This works because exactly one number from the full range `1..n` is absent, so the arithmetic difference isolates it directly without needing to search or sort. (An XOR-based variant works identically: XOR all numbers `1..n` with all array elements — every pair cancels out except the missing number.)

---

### 8. Maximum Consecutive Ones
**Problem Statement:** Given a binary array (containing only `0`s and `1`s), find the maximum number of consecutive `1`s in the array.

**Example:**
- Input: `arr = [1, 1, 0, 1, 1, 1, 0, 1]`
- Output: `3`
- Explanation: The longest run of consecutive `1`s is `[1, 1, 1]` at indices 3-5.
- Input: `arr = [0, 0, 0]`
- Output: `0`
- Explanation: There are no `1`s at all, so the max streak is `0`.

**Brute Force Approach:** For every starting index, count how many consecutive `1`s follow, and track the maximum. This brute force IS effectively the optimal approach's core idea — but doing it via nested loops is wasteful; the single-pass version below is the natural optimal form since each element is examined only once regardless.
```csharp
public static int MaxConsecutiveOnesBrute(int[] arr)
{
    int maxCount = 0;

    for (int i = 0; i < arr.Length; i++)
    {
        if (arr[i] == 1)
        {
            int count = 0;
            int j = i;
            while (j < arr.Length && arr[j] == 1)
            {
                count++;
                j++;
            }
            maxCount = Math.Max(maxCount, count);
        }
    }

    return maxCount;
}
```
Time Complexity: O(n^2) worst case — e.g., an all-ones array re-counts overlapping runs from every starting index
Space Complexity: O(1) — no extra space used

**Optimized Approach:** Traverse the array once with a running counter. Increment the counter on `1`, reset it to `0` on `0`, and update the maximum after every step.
```csharp
public static int MaxConsecutiveOnes(int[] arr)
{
    int maxCount = 0;
    int currentCount = 0;

    foreach (int num in arr)
    {
        if (num == 1)
        {
            currentCount++;
            maxCount = Math.Max(maxCount, currentCount);
        }
        else
        {
            currentCount = 0;
        }
    }

    return maxCount;
}
```
Time Complexity: O(n) — single pass through the array
Space Complexity: O(1) — only two counter variables used

---

### 9. Find the Number that Appears Once While Others Appear Twice
**Problem Statement:** Given a non-empty array where every element appears exactly twice except for one element which appears exactly once, find that single element.

**Example:**
- Input: `arr = [4, 1, 2, 1, 2]`
- Output: `4`
- Explanation: `1` and `2` each appear twice; `4` appears only once.
- Input: `arr = [7]`
- Output: `7`
- Explanation: A single-element array trivially returns that element.

**Brute Force Approach:** For each element, count its occurrences in the rest of the array using a nested loop (or a hash map). Return the element whose count is exactly one.
```csharp
public static int SingleNumberBrute(int[] arr)
{
    for (int i = 0; i < arr.Length; i++)
    {
        int count = 0;
        for (int j = 0; j < arr.Length; j++)
        {
            if (arr[j] == arr[i])
                count++;
        }
        if (count == 1)
            return arr[i];
    }

    return -1;
}
```
Time Complexity: O(n^2) — nested loop compares every element with every other element
Space Complexity: O(1) — no extra space used (O(n) if using a HashMap instead)

**Optimized Approach:** Use the XOR bitwise operator. XOR-ing a number with itself yields `0`, and XOR-ing any number with `0` yields the number unchanged. XOR-ing all elements together cancels out every pair, leaving only the unique element.
```csharp
public static int SingleNumber(int[] arr)
{
    int result = 0;
    foreach (int num in arr)
    {
        result ^= num;
    }
    return result;
}
```
Time Complexity: O(n) — single pass XOR-ing every element
Space Complexity: O(1) — only one accumulator variable used

**Explanation:** Dry run on `arr = [4, 1, 2, 1, 2]`, `result = 0`:
- `result = 0 ^ 4 = 4`
- `result = 4 ^ 1 = 5`
- `result = 5 ^ 2 = 7`
- `result = 7 ^ 1 = 6` (since `1` cancels with the earlier `1`: `4 ^ 1 ^ 2 ^ 1 = 4 ^ 2`)
- `result = 6 ^ 2 = 4` (the second `2` cancels the first `2`, leaving just `4`)
- Final: `result = 4` ✔
This works because XOR is commutative and associative, so the order of operations doesn't matter — all pairs of identical numbers cancel to `0`, and `0 ^ x = x` leaves only the number that had no pair.
