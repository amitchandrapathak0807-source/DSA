# Binary Search — Binary Search on Answer (Allocation & Partition Problems)

## 1. Kth Missing Positive Number

**Problem Statement:** Given a sorted array of distinct positive integers `arr` and an integer `k`, find the `k`th positive integer that is missing from this array.

**Example:**
- Input: `arr = [2, 3, 4, 7, 11], k = 5`
- Output: `9`
- Explanation: The missing positive numbers in order are `1, 5, 6, 8, 9, 10, ...`. The 5th missing number is `9`.

**Brute Force Approach:** Walk through positive integers `1, 2, 3, ...` one at a time, checking (via a pointer or linear search) whether each is present in `arr`. Count the missing ones until we reach the `k`th missing number.

```csharp
public int FindKthPositiveBruteForce(int[] arr, int k)
{
    int missingCount = 0;
    int num = 1;
    int index = 0;

    while (true)
    {
        // Advance the array pointer past numbers smaller than 'num'
        if (index < arr.Length && arr[index] == num)
        {
            index++;
        }
        else
        {
            missingCount++;
            if (missingCount == k)
            {
                return num;
            }
        }
        num++;
    }
}
```
Time Complexity: O(n + ans) in the worst case (scans forward until the kth missing number is found).
Space Complexity: O(1).

**Optimized Approach:** For every index `i` (0-based) in `arr`, the count of missing numbers before `arr[i]` is `arr[i] - (i + 1)`. This quantity is non-decreasing as `i` increases, so we binary search for the smallest index where `arr[i] - (i + 1) >= k`. Once found, the answer is `low + k` (where `low` is the number of "fully present" elements before the gap).

```csharp
public int FindKthPositiveOptimized(int[] arr, int k)
{
    int low = 0, high = arr.Length - 1;

    while (low <= high)
    {
        int mid = low + (high - low) / 2;
        int missingBeforeMid = arr[mid] - (mid + 1);

        if (missingBeforeMid < k)
        {
            low = mid + 1;
        }
        else
        {
            high = mid - 1;
        }
    }

    // 'high' is the last index where missing count < k
    // 'low' is the first index where missing count >= k
    return low + k;
}
```
Time Complexity: O(log n). Space Complexity: O(1).

**Explanation:** Dry run with `arr = [2, 3, 4, 7, 11], k = 5`.
Missing-before-index values: index0(2) -> 2-1=1, index1(3) -> 3-2=1, index2(4) -> 4-3=1, index3(7) -> 7-4=3, index4(11) -> 11-5=6.
- `low=0, high=4, mid=2` -> missing=1 < 5 -> `low=3`.
- `low=3, high=4, mid=3` -> missing=3 < 5 -> `low=4`.
- `low=4, high=4, mid=4` -> missing=6 >= 5 -> `high=3`.
- Loop ends: `low=4`. Answer = `low + k = 4 + 5 = 9`. Matches expected output `9`.
The binary search finds the boundary between "not enough missing numbers yet" and "enough missing numbers", and `low + k` accounts for the `low` present numbers we skipped over.

## 2. Aggressive Cows Problem (maximize minimum distance)

**Problem Statement:** Given an array of stall positions and an integer `k` (number of cows), place all `k` cows into stalls such that the minimum distance between any two cows is as large as possible. Return that maximum possible minimum distance.

**Example:**
- Input: `stalls = [1, 2, 4, 8, 9], k = 3`
- Output: `3`
- Explanation: Placing cows at positions `1, 4, 8` (sorted stalls) gives minimum pairwise distance `3`, which is the largest achievable minimum distance for 3 cows.

**Brute Force Approach:** Sort the stalls. Try every possible minimum distance from `1` up to `maxStall - minStall`, and for each candidate distance check greedily whether all `k` cows can be placed while maintaining at least that distance. Keep the largest distance that works.

```csharp
public int AggressiveCowsBruteForce(int[] stalls, int k)
{
    Array.Sort(stalls);
    int maxDistance = stalls[stalls.Length - 1] - stalls[0];
    int best = 0;

    for (int distance = 1; distance <= maxDistance; distance++)
    {
        if (CanPlaceCows(stalls, k, distance))
        {
            best = distance;
        }
    }

    return best;
}

private bool CanPlaceCows(int[] stalls, int cows, int minDistance)
{
    int count = 1;
    int lastPosition = stalls[0];

    for (int i = 1; i < stalls.Length; i++)
    {
        if (stalls[i] - lastPosition >= minDistance)
        {
            count++;
            lastPosition = stalls[i];
        }
    }

    return count >= cows;
}
```
Time Complexity: O(n log n + n * maxDistance). Space Complexity: O(1).

**Optimized Approach:** Sort the stalls and binary search on the candidate minimum distance in the range `[1, maxStall - minStall]`. For each `mid` distance, use the same greedy feasibility check `CanPlaceCows`. If feasible, try a larger distance; otherwise try smaller.

```csharp
public int AggressiveCowsOptimized(int[] stalls, int k)
{
    Array.Sort(stalls);
    int low = 1;
    int high = stalls[stalls.Length - 1] - stalls[0];
    int answer = 0;

    while (low <= high)
    {
        int mid = low + (high - low) / 2;

        if (CanPlaceCows(stalls, k, mid))
        {
            answer = mid;
            low = mid + 1;
        }
        else
        {
            high = mid - 1;
        }
    }

    return answer;
}

private bool CanPlaceCows(int[] stalls, int cows, int minDistance)
{
    int count = 1;
    int lastPosition = stalls[0];

    for (int i = 1; i < stalls.Length; i++)
    {
        if (stalls[i] - lastPosition >= minDistance)
        {
            count++;
            lastPosition = stalls[i];
        }
    }

    return count >= cows;
}
```
Time Complexity: O(n log n) for sorting + O(n log(maxDistance)) for the binary search = O(n log(maxDistance)). Space Complexity: O(1).

**Explanation:** Dry run of `CanPlaceCows` for `stalls = [1, 2, 4, 8, 9]` (sorted), `k = 3`, candidate `minDistance = 3`:
Place first cow at `1` (`lastPosition = 1`, `count = 1`).
- `stalls[1]=2`: `2 - 1 = 1 < 3` -> skip.
- `stalls[2]=4`: `4 - 1 = 3 >= 3` -> place cow, `count = 2`, `lastPosition = 4`.
- `stalls[3]=8`: `8 - 4 = 4 >= 3` -> place cow, `count = 3`, `lastPosition = 8`.
- `stalls[4]=9`: `9 - 8 = 1 < 3` -> skip.
Final `count = 3 >= k(3)` -> feasible, so distance 3 works. The binary search over `[1, 8]` (since max-min = 9-1=8) narrows in: it will find that distance `4` fails (only 2 cows fit: `1, 8` and maybe `4` fails too — count would be `1(at 1), 4(4-1=3<4 skip), 8(8-1=7>=4 place, count=2)` -> count=2 < 3, infeasible), so the largest feasible distance is `3`, matching the expected output.

## 3. Book Allocation Problem (minimize the maximum pages assigned)

**Problem Statement:** Given an array `books` where `books[i]` is the number of pages in the `i`th book, and an integer `students`, allocate all books to students such that each student gets a contiguous set of books, every book is allocated to exactly one student, and the maximum number of pages assigned to any student is minimized. Each student must get at least one book; if allocation is not possible (students > books), return -1.

**Example:**
- Input: `books = [12, 34, 67, 90], students = 2`
- Output: `113`
- Explanation: One valid split is `[12, 34, 67] | [90]` with sums `113` and `90` (max = 113). Another split `[12, 34] | [67, 90]` gives sums `46` and `157` (max = 157, worse). The split minimizing the maximum is `[12, 34, 67]` and `[90]`, giving max pages `113`.

**Brute Force Approach:** Generate every way to partition the array into `students` contiguous, non-empty groups (recursively decide where each partition boundary goes), compute the maximum sum per partition, and track the overall minimum of these maximums.

```csharp
public int BookAllocationBruteForce(int[] books, int students)
{
    if (students > books.Length) return -1;

    int best = int.MaxValue;
    Partition(books, students, 0, new List<int>(), ref best);
    return best;
}

private void Partition(int[] books, int studentsLeft, int startIndex, List<int> currentSums, ref int best)
{
    if (studentsLeft == 0)
    {
        if (startIndex == books.Length)
        {
            best = Math.Min(best, currentSums.Max());
        }
        return;
    }

    int sum = 0;
    for (int end = startIndex; end < books.Length; end++)
    {
        sum += books[end];

        // Ensure remaining books are enough for remaining students
        int remainingBooks = books.Length - (end + 1);
        if (remainingBooks < studentsLeft - 1) break;

        currentSums.Add(sum);
        Partition(books, studentsLeft - 1, end + 1, currentSums, ref best);
        currentSums.RemoveAt(currentSums.Count - 1);
    }
}
```
Time Complexity: Exponential, roughly O(n^students) due to trying every partition split. Space Complexity: O(students) recursion depth plus O(students) for sums list.

**Optimized Approach:** Binary search on the answer (the maximum pages per student), ranging from `max(books)` (a single book must fit with one student) to `sum(books)` (one student gets everything). For each candidate `mid`, greedily check the minimum number of students required so that no student's total exceeds `mid`. If required students <= given students, `mid` is feasible; try smaller. Otherwise try larger.

```csharp
public int BookAllocationOptimized(int[] books, int students)
{
    if (students > books.Length) return -1;

    int low = books.Max();
    int high = books.Sum();
    int answer = high;

    while (low <= high)
    {
        int mid = low + (high - low) / 2;

        if (CountStudentsRequired(books, mid) <= students)
        {
            answer = mid;
            high = mid - 1;
        }
        else
        {
            low = mid + 1;
        }
    }

    return answer;
}

private int CountStudentsRequired(int[] books, int maxPages)
{
    int studentsNeeded = 1;
    int currentSum = 0;

    foreach (int pages in books)
    {
        if (currentSum + pages > maxPages)
        {
            studentsNeeded++;
            currentSum = pages;
        }
        else
        {
            currentSum += pages;
        }
    }

    return studentsNeeded;
}
```
Time Complexity: O(n log(sum(books) - max(books))). Space Complexity: O(1).

**Explanation:** The Book Allocation Problem and the Painter's Partition Problem (Section 5) are the *same underlying pattern*: both split an array into contiguous groups among a fixed number of "workers" (students / painters) to minimize the maximum "load" (pages / boards) any single worker handles. The feasibility predicate — "can we partition so no group exceeds `mid`?" — is identical in structure; only the domain vocabulary differs.

Dry run `CountStudentsRequired` for `books = [12, 34, 67, 90]`, candidate `mid = 113`:
- Start: `studentsNeeded = 1`, `currentSum = 0`.
- `12`: `0 + 12 = 12 <= 113` -> `currentSum = 12`.
- `34`: `12 + 34 = 46 <= 113` -> `currentSum = 46`.
- `67`: `46 + 67 = 113 <= 113` -> `currentSum = 113`.
- `90`: `113 + 90 = 203 > 113` -> new student, `studentsNeeded = 2`, `currentSum = 90`.
Final `studentsNeeded = 2 <= students(2)` -> feasible. Binary search then tries smaller `mid` values (e.g. `112`) and finds them infeasible (would require 3 students), so `113` is confirmed as the minimum feasible maximum, matching the expected output.

## 4. Split Array — Largest Subarray Sum Minimized

**Problem Statement:** Given an array of non-negative integers `nums` and an integer `k`, split `nums` into `k` non-empty contiguous subarrays such that the largest sum among these subarrays is minimized. Return that minimized largest sum.

**Example:**
- Input: `nums = [7, 2, 5, 10, 8], k = 2`
- Output: `18`
- Explanation: Split as `[7, 2, 5] | [10, 8]` giving sums `14` and `18` (max = 18). This is the best possible split; any other split yields a larger maximum (e.g. `[7,2,5,10] | [8]` gives max 24).

**Brute Force Approach:** Recursively try every way to split the array into `k` contiguous non-empty groups, compute the maximum subarray sum for each split, and keep the overall minimum.

```csharp
public int SplitArrayBruteForce(int[] nums, int k)
{
    int best = int.MaxValue;
    Split(nums, k, 0, new List<int>(), ref best);
    return best;
}

private void Split(int[] nums, int groupsLeft, int startIndex, List<int> sums, ref int best)
{
    if (groupsLeft == 0)
    {
        if (startIndex == nums.Length)
        {
            best = Math.Min(best, sums.Max());
        }
        return;
    }

    int sum = 0;
    for (int end = startIndex; end < nums.Length; end++)
    {
        sum += nums[end];

        int remaining = nums.Length - (end + 1);
        if (remaining < groupsLeft - 1) break;

        sums.Add(sum);
        Split(nums, groupsLeft - 1, end + 1, sums, ref best);
        sums.RemoveAt(sums.Count - 1);
    }
}
```
Time Complexity: Exponential, roughly O(n^k). Space Complexity: O(k) recursion depth.

**Optimized Approach:** Binary search on the answer (largest subarray sum) between `max(nums)` and `sum(nums)`. For a candidate `mid`, greedily count the minimum number of subarrays needed so that none exceeds `mid`. If that count is <= `k`, `mid` is feasible.

```csharp
public int SplitArrayOptimized(int[] nums, int k)
{
    int low = nums.Max();
    int high = nums.Sum();
    int answer = high;

    while (low <= high)
    {
        int mid = low + (high - low) / 2;

        if (CountSubarraysRequired(nums, mid) <= k)
        {
            answer = mid;
            high = mid - 1;
        }
        else
        {
            low = mid + 1;
        }
    }

    return answer;
}

private int CountSubarraysRequired(int[] nums, int maxSum)
{
    int subarraysNeeded = 1;
    int currentSum = 0;

    foreach (int num in nums)
    {
        if (currentSum + num > maxSum)
        {
            subarraysNeeded++;
            currentSum = num;
        }
        else
        {
            currentSum += num;
        }
    }

    return subarraysNeeded;
}
```
Time Complexity: O(n log(sum(nums) - max(nums))). Space Complexity: O(1).

**Explanation:** This problem is structurally identical to Book Allocation and Painter's Partition — split an array into `k` contiguous groups minimizing the maximum group sum. Dry run `CountSubarraysRequired` for `nums = [7, 2, 5, 10, 8]`, candidate `mid = 18`:
- `7`: `currentSum = 7`.
- `2`: `7+2=9 <= 18` -> `currentSum = 9`.
- `5`: `9+5=14 <= 18` -> `currentSum = 14`.
- `10`: `14+10=24 > 18` -> new subarray, `subarraysNeeded = 2`, `currentSum = 10`.
- `8`: `10+8=18 <= 18` -> `currentSum = 18`.
Final `subarraysNeeded = 2 <= k(2)` -> feasible. Testing `mid = 17` would fail (requires 3 groups: `[7,2,5]`, `[10]`, `[8]` since `10+8=18>17`), confirming `18` is the minimized largest sum.

## 5. Painter's Partition Problem

**Problem Statement:** Given an array `boards` where `boards[i]` is the length of the `i`th board, and an integer `painters`, allocate all boards to painters such that each painter paints a contiguous set of boards, every board is painted by exactly one painter, and the maximum length painted by any single painter is minimized (each painter takes 1 unit time per unit length). Return that minimized maximum length.

**Example:**
- Input: `boards = [10, 20, 30, 40], painters = 2`
- Output: `60`
- Explanation: Split as `[10, 20, 30] | [40]` with lengths `60` and `40` (max = 60). This is the best achievable split for 2 painters.

**Brute Force Approach:** Generate every possible way to split the boards into `painters` contiguous non-empty groups, compute the max group length sum for each split, and track the minimum such maximum. (Identical structure to Book Allocation brute force, applied to "boards" instead of "books".)

```csharp
public int PaintersPartitionBruteForce(int[] boards, int painters)
{
    int best = int.MaxValue;
    Partition(boards, painters, 0, new List<int>(), ref best);
    return best;
}

private void Partition(int[] boards, int paintersLeft, int startIndex, List<int> sums, ref int best)
{
    if (paintersLeft == 0)
    {
        if (startIndex == boards.Length)
        {
            best = Math.Min(best, sums.Max());
        }
        return;
    }

    int sum = 0;
    for (int end = startIndex; end < boards.Length; end++)
    {
        sum += boards[end];

        int remaining = boards.Length - (end + 1);
        if (remaining < paintersLeft - 1) break;

        sums.Add(sum);
        Partition(boards, paintersLeft - 1, end + 1, sums, ref best);
        sums.RemoveAt(sums.Count - 1);
    }
}
```
Time Complexity: Exponential, roughly O(n^painters). Space Complexity: O(painters) recursion depth.

**Optimized Approach:** Binary search on the answer (maximum length assigned per painter) between `max(boards)` and `sum(boards)`. For a candidate `mid`, greedily count the minimum painters required so no painter's total exceeds `mid`.

```csharp
public int PaintersPartitionOptimized(int[] boards, int painters)
{
    int low = boards.Max();
    int high = boards.Sum();
    int answer = high;

    while (low <= high)
    {
        int mid = low + (high - low) / 2;

        if (CountPaintersRequired(boards, mid) <= painters)
        {
            answer = mid;
            high = mid - 1;
        }
        else
        {
            low = mid + 1;
        }
    }

    return answer;
}

private int CountPaintersRequired(int[] boards, int maxLength)
{
    int paintersNeeded = 1;
    int currentSum = 0;

    foreach (int length in boards)
    {
        if (currentSum + length > maxLength)
        {
            paintersNeeded++;
            currentSum = length;
        }
        else
        {
            currentSum += length;
        }
    }

    return paintersNeeded;
}
```
Time Complexity: O(n log(sum(boards) - max(boards))). Space Complexity: O(1).

**Explanation:** As noted in Section 3, Painter's Partition and Book Allocation are the exact same pattern — only the labels differ ("painters"/"boards" vs "students"/"books"). Dry run `CountPaintersRequired` for `boards = [10, 20, 30, 40]`, candidate `mid = 60`:
- `10`: `currentSum = 10`.
- `20`: `10+20=30 <= 60` -> `currentSum = 30`.
- `30`: `30+30=60 <= 60` -> `currentSum = 60`.
- `40`: `60+40=100 > 60` -> new painter, `paintersNeeded = 2`, `currentSum = 40`.
Final `paintersNeeded = 2 <= painters(2)` -> feasible. Testing `mid = 59` fails (`10+20+30=60>59` forces a split earlier, ultimately requiring 3 painters: `[10,20]`(30), `[30]`(30... actually `10+20=30<=59`, `+30=60>59` so new group; `[30]`=30<=59, `+40=70>59` new group -> `[40]`; total 3 groups), confirming `60` is the minimized maximum length.

## 6. Minimize Maximum Distance Between Gas Stations

**Problem Statement:** Given a sorted array `stations` representing positions of existing gas stations along a road, and an integer `k` representing the number of new gas stations to add, place the `k` new stations (at any real-valued position, not necessarily integers) to minimize the maximum distance between any two adjacent gas stations. Return that minimized maximum distance.

**Example:**
- Input: `stations = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10], k = 9`
- Output: `0.5`
- Explanation: There are 9 gaps of length 1 each between the 10 stations. Adding 9 new stations (one per gap) splits each gap of length 1 into two gaps of length 0.5, making the maximum gap `0.5`.

**Brute Force Approach:** Repeat `k` times: find the current largest gap between adjacent stations, and place one new station exactly in its middle (splitting it in half), updating the gap array. This greedy simulation directly reduces the largest gap each iteration.

```csharp
public double MinimizeMaxDistanceBruteForce(int[] stations, int k)
{
    int n = stations.Length;
    double[] gapAdditions = new double[n - 1]; // number of stations added into each gap

    for (int i = 0; i < k; i++)
    {
        int maxGapIndex = 0;
        double maxGapLength = -1;

        for (int j = 0; j < n - 1; j++)
        {
            double gapLength = (stations[j + 1] - stations[j]) / (gapAdditions[j] + 1.0);
            if (gapLength > maxGapLength)
            {
                maxGapLength = gapLength;
                maxGapIndex = j;
            }
        }

        gapAdditions[maxGapIndex]++;
    }

    double result = 0;
    for (int j = 0; j < n - 1; j++)
    {
        double gapLength = (stations[j + 1] - stations[j]) / (gapAdditions[j] + 1.0);
        result = Math.Max(result, gapLength);
    }

    return result;
}
```
Time Complexity: O(k * n) since each of the `k` insertions scans all gaps to find the maximum. Space Complexity: O(n).

**Optimized Approach:** Binary search on the answer (the maximum allowed gap distance), a real number between `0` and the largest original gap, with enough precision iterations (or an epsilon threshold). For a candidate `mid` distance, compute how many new stations would be required so that every gap is at most `mid` (for a gap of length `g`, that requires `ceil(g / mid) - 1` stations). If total required stations <= `k`, `mid` is feasible (try smaller); otherwise try larger.

```csharp
public double MinimizeMaxDistanceOptimized(int[] stations, int k)
{
    int n = stations.Length;
    double low = 0;
    double high = 0;

    for (int j = 0; j < n - 1; j++)
    {
        high = Math.Max(high, stations[j + 1] - stations[j]);
    }

    double precision = 1e-6;

    while (high - low > precision)
    {
        double mid = low + (high - low) / 2.0;

        if (CountStationsRequired(stations, mid) <= k)
        {
            high = mid;
        }
        else
        {
            low = mid;
        }
    }

    return high;
}

private int CountStationsRequired(int[] stations, double maxDistance)
{
    int count = 0;

    for (int j = 0; j < stations.Length - 1; j++)
    {
        double gap = stations[j + 1] - stations[j];
        count += (int)Math.Ceiling(gap / maxDistance) - 1;
    }

    return count;
}
```
Time Complexity: O(n log(range / precision)) — a fixed number of iterations (roughly 50 for double precision) each doing an O(n) scan. Space Complexity: O(1).

**Explanation:** Dry run `CountStationsRequired` for `stations = [1..10]` (9 gaps of length 1 each), candidate `mid = 0.5`:
For each gap `g = 1`: `Ceiling(1 / 0.5) - 1 = Ceiling(2) - 1 = 1` station needed per gap. Total across 9 gaps: `9 * 1 = 9`.
`9 <= k(9)` -> feasible, so `high` shrinks toward `0.5`. Testing a smaller candidate like `mid = 0.4`: `Ceiling(1/0.4) - 1 = Ceiling(2.5) - 1 = 3 - 1 = 2` per gap, total `9 * 2 = 18 > 9` -> infeasible, so `low` rises back toward `0.5`. The binary search converges on `0.5`, matching the expected output.

## 7. Median of Two Sorted Arrays (using Binary Search, O(log(min(n,m))))

**Problem Statement:** Given two sorted arrays `nums1` and `nums2` of sizes `n` and `m`, find the median of the two arrays combined, without fully merging them, in `O(log(min(n, m)))` time.

**Example:**
- Input: `nums1 = [1, 3], nums2 = [2]`
- Output: `2.0`
- Explanation: The merged sorted array is `[1, 2, 3]`, which has an odd length of 3, so the median is the middle element `2`.

**Brute Force Approach:** Merge both sorted arrays into a single sorted array (using the standard merge-two-sorted-arrays technique), then directly compute the median from the merged array based on whether its length is odd or even.

```csharp
public double FindMedianBruteForce(int[] nums1, int[] nums2)
{
    int n = nums1.Length, m = nums2.Length;
    int[] merged = new int[n + m];
    int i = 0, j = 0, k = 0;

    while (i < n && j < m)
    {
        if (nums1[i] <= nums2[j])
        {
            merged[k++] = nums1[i++];
        }
        else
        {
            merged[k++] = nums2[j++];
        }
    }

    while (i < n) merged[k++] = nums1[i++];
    while (j < m) merged[k++] = nums2[j++];

    int total = n + m;
    if (total % 2 == 1)
    {
        return merged[total / 2];
    }
    else
    {
        return (merged[total / 2 - 1] + merged[total / 2]) / 2.0;
    }
}
```
Time Complexity: O(n + m). Space Complexity: O(n + m) for the merged array.

**Optimized Approach:** Binary search on the smaller array to find a "partition" point such that all elements on the left side (across both arrays combined) are less than or equal to all elements on the right side, and the left side has exactly `(n + m + 1) / 2` elements. The median is then derived from the boundary elements around this partition, without merging.

```csharp
public double FindMedianOptimized(int[] nums1, int[] nums2)
{
    // Always binary search on the smaller array for O(log(min(n, m)))
    if (nums1.Length > nums2.Length)
    {
        return FindMedianOptimized(nums2, nums1);
    }

    int n = nums1.Length;
    int m = nums2.Length;
    int totalLeft = (n + m + 1) / 2;

    int low = 0, high = n;

    while (low <= high)
    {
        int cut1 = low + (high - low) / 2;
        int cut2 = totalLeft - cut1;

        int left1 = (cut1 == 0) ? int.MinValue : nums1[cut1 - 1];
        int left2 = (cut2 == 0) ? int.MinValue : nums2[cut2 - 1];
        int right1 = (cut1 == n) ? int.MaxValue : nums1[cut1];
        int right2 = (cut2 == m) ? int.MaxValue : nums2[cut2];

        if (left1 <= right2 && left2 <= right1)
        {
            int totalLength = n + m;
            if (totalLength % 2 == 1)
            {
                return Math.Max(left1, left2);
            }
            else
            {
                return (Math.Max(left1, left2) + Math.Min(right1, right2)) / 2.0;
            }
        }
        else if (left1 > right2)
        {
            high = cut1 - 1;
        }
        else
        {
            low = cut1 + 1;
        }
    }

    throw new ArgumentException("Input arrays are not sorted or invalid.");
}
```
Time Complexity: O(log(min(n, m))). Space Complexity: O(1).

**Explanation:** The core idea is the *partition invariant*: we choose a cut point `cut1` in the smaller array (`nums1`) and a corresponding cut point `cut2 = totalLeft - cut1` in the larger array (`nums2`), such that together the left portions contain exactly the first half of all combined elements. The partition is valid when every element in the combined "left" side is `<=` every element in the combined "right" side, i.e. `left1 <= right2` AND `left2 <= right1`.

Dry run with `nums1 = [1, 3]`, `nums2 = [2]` (`n = 2`, `m = 1`, already `nums1` is not longer than `nums2`... actually `n=2 > m=1`, so we swap: call with `nums1 = [2]`, `nums2 = [1, 3]`). Now `n = 1`, `m = 2`, `totalLeft = (1 + 2 + 1) / 2 = 2`.
- `low = 0, high = 1`. `cut1 = 0 + (1-0)/2 = 0`. `cut2 = 2 - 0 = 2`.
  - `left1 = (cut1==0) -> MinValue`.
  - `left2 = nums2[cut2-1] = nums2[1] = 3`.
  - `right1 = nums1[0] = 2`.
  - `right2 = (cut2==m=2) -> MaxValue`.
  - Check: `left1(MinValue) <= right2(MaxValue)` true; `left2(3) <= right1(2)` false. Since `left2 > right1`, we need to move `cut1` right: `low = cut1 + 1 = 1`.
- `low = 1, high = 1`. `cut1 = 1`. `cut2 = 2 - 1 = 1`.
  - `left1 = nums1[0] = 2`.
  - `left2 = nums2[0] = 1`.
  - `right1 = (cut1==n=1) -> MaxValue`.
  - `right2 = nums2[1] = 3`.
  - Check: `left1(2) <= right2(3)` true; `left2(1) <= right1(MaxValue)` true. Valid partition found.
  - Total length `n+m = 3` is odd, so median = `Math.Max(left1, left2) = Math.Max(2, 1) = 2`.
Result: `2.0`, matching the expected output. This confirms the partition correctly separates `[1, 2]` (left) from `[3]` (right) across both arrays, with the median being the largest element on the left side since the total count is odd.
