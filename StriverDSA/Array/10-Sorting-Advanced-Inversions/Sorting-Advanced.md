# Array — Sorting Advanced (Inversions & Repeating/Missing)

## Find the Repeating and Missing Number

### 1. Find the Repeating and Missing Number

**Problem Statement:**
You are given an array of size `n` containing numbers from `1` to `n`. Exactly one number from the range is missing, and exactly one number appears twice (the "repeating" number). Find both the repeating number and the missing number.

**Example:**
- Input: `arr = [3, 1, 2, 5, 3]` (n = 5)
- Output: Repeating = 3, Missing = 4
- Explanation: The array should contain `1, 2, 3, 4, 5` exactly once each. Here `3` appears twice and `4` never appears, so `3` is repeating and `4` is missing.

**Brute Force Approach:**
For every number `i` from `1` to `n`, scan the whole array and count how many times `i` occurs. If it occurs `2` times, it is the repeating number. If it occurs `0` times, it is the missing number. This uses two nested loops (outer over `1..n`, inner scanning the array), giving O(n^2) time. A better "brute force" uses a frequency (hash) array of size `n+1` to count occurrences in a single pass, then scans the frequency array once — O(n) time but O(n) extra space.

```csharp
public class RepeatingMissingBruteForce
{
    // O(n^2) time, O(1) extra space (excluding output)
    public static (int repeating, int missing) FindUsingNestedLoop(int[] arr)
    {
        int n = arr.Length;
        int repeating = -1, missing = -1;

        for (int i = 1; i <= n; i++)
        {
            int count = 0;
            for (int j = 0; j < n; j++)
            {
                if (arr[j] == i) count++;
            }

            if (count == 2) repeating = i;
            else if (count == 0) missing = i;
        }

        return (repeating, missing);
    }

    // O(n) time, O(n) extra space using a frequency array (hashing)
    public static (int repeating, int missing) FindUsingFrequencyArray(int[] arr)
    {
        int n = arr.Length;
        int[] freq = new int[n + 1];

        foreach (int num in arr)
        {
            freq[num]++;
        }

        int repeating = -1, missing = -1;
        for (int i = 1; i <= n; i++)
        {
            if (freq[i] == 2) repeating = i;
            else if (freq[i] == 0) missing = i;
        }

        return (repeating, missing);
    }
}
```

Time Complexity: O(n^2) for the nested-loop version because for each of the `n` candidate values we scan the entire array again. The frequency-array version is O(n) since it does one pass to build counts and one pass over `1..n` to read them.
Space Complexity: O(1) extra space for the nested-loop version (only a few variables). O(n) extra space for the frequency-array version because of the `freq` array of size `n+1`.

**Optimized Approach:**
Use the mathematical (sum and sum-of-squares) trick to solve in O(n) time and O(1) extra space, without any hashing.

Let `S` = sum of first `n` natural numbers = `n(n+1)/2`, and `S2` = sum of squares of first `n` natural numbers = `n(n+1)(2n+1)/6`.
Let `ActualSum` = sum of the given array, and `ActualSum2` = sum of squares of the given array.

Because the array has one number repeated (`repeating`) and one missing (`missing`) instead of the "perfect" set `1..n`:
- `ActualSum - S = repeating - missing`  → call this `diff = repeating - missing`
- `ActualSum2 - S2 = repeating^2 - missing^2 = (repeating - missing)(repeating + missing)`

Dividing the second equation by `diff` gives `repeating + missing`. Combined with `diff`, we solve a simple 2-variable linear system for both values.

```csharp
public class RepeatingMissingOptimized
{
    // O(n) time, O(1) extra space
    public static (int repeating, int missing) FindUsingMath(int[] arr)
    {
        long n = arr.Length;

        long expectedSum = n * (n + 1) / 2;
        long expectedSumSq = n * (n + 1) * (2 * n + 1) / 6;

        long actualSum = 0;
        long actualSumSq = 0;

        foreach (int num in arr)
        {
            actualSum += num;
            actualSumSq += (long)num * num;
        }

        // repeating - missing = diff1
        long diff1 = actualSum - expectedSum;

        // repeating^2 - missing^2 = diff2  => (repeating - missing)(repeating + missing) = diff2
        long diff2 = actualSumSq - expectedSumSq;

        // repeating + missing = sum1
        long sum1 = diff2 / diff1;

        // Solve: repeating - missing = diff1, repeating + missing = sum1
        long repeating = (diff1 + sum1) / 2;
        long missing = repeating - diff1;

        return ((int)repeating, (int)missing);
    }
}
```

Time Complexity: O(n) — a single pass through the array to compute `actualSum` and `actualSumSq`; everything else is O(1) arithmetic.
Space Complexity: O(1) — only a fixed number of long variables are used, no auxiliary arrays.

**Explanation:**
Derivation of the two equations:

1. If the array were perfect (each number `1..n` exactly once), its sum would be `S = n(n+1)/2`. Because `repeating` is counted twice and `missing` is counted zero times instead of once each, the actual sum differs from `S` by exactly `repeating - missing`:
   `ActualSum = S - missing + repeating`
   `=> ActualSum - S = repeating - missing`   ... (Equation 1)

2. Similarly for sum of squares, `S2 = n(n+1)(2n+1)/6`. The actual sum of squares differs from `S2` by `repeating^2 - missing^2`:
   `ActualSum2 = S2 - missing^2 + repeating^2`
   `=> ActualSum2 - S2 = repeating^2 - missing^2`   ... (Equation 2)

Factor Equation 2 using difference of squares: `repeating^2 - missing^2 = (repeating - missing)(repeating + missing)`.
Since `(repeating - missing)` is already known from Equation 1 (call it `diff1`), we get:
`repeating + missing = (ActualSum2 - S2) / diff1 = sum1`

Now we have a simple system:
`repeating - missing = diff1`
`repeating + missing = sum1`

Adding both: `2 * repeating = diff1 + sum1` → `repeating = (diff1 + sum1) / 2`, then `missing = repeating - diff1`.

Dry run on `arr = [3, 1, 2, 5, 3]`, n = 5:
- `S = 5*6/2 = 15`, `S2 = 5*6*11/6 = 55`
- `ActualSum = 3+1+2+5+3 = 14`, `ActualSumSq = 9+1+4+25+9 = 48`
- `diff1 = ActualSum - S = 14 - 15 = -1`  → `repeating - missing = -1`
- `diff2 = ActualSumSq - S2 = 48 - 55 = -7`
- `sum1 = diff2 / diff1 = -7 / -1 = 7`  → `repeating + missing = 7`
- `repeating = (diff1 + sum1) / 2 = (-1 + 7) / 2 = 3`
- `missing = repeating - diff1 = 3 - (-1) = 4`

Result: repeating = 3, missing = 4, matching the expected output.

## Count Inversions in an Array

### 2. Count Inversions in an Array

**Problem Statement:**
Given an array `arr` of `n` integers, count the number of inversions. An inversion is a pair of indices `(i, j)` such that `i < j` and `arr[i] > arr[j]`. Intuitively this measures "how far" the array is from being sorted.

**Example:**
- Input: `arr = [5, 3, 2, 4, 1]`
- Output: `8`
- Explanation: The inversion pairs (by value, with i<j) are: (5,3), (5,2), (5,4), (5,1), (3,2), (3,1), (2,1), (4,1) — 8 pairs total where an earlier element is greater than a later one.

**Brute Force Approach:**
Use two nested loops: for every pair `(i, j)` with `i < j`, check if `arr[i] > arr[j]`, and if so increment the inversion count.

```csharp
public class CountInversionsBruteForce
{
    // O(n^2) time, O(1) extra space
    public static long CountInversions(int[] arr)
    {
        int n = arr.Length;
        long count = 0;

        for (int i = 0; i < n; i++)
        {
            for (int j = i + 1; j < n; j++)
            {
                if (arr[i] > arr[j])
                {
                    count++;
                }
            }
        }

        return count;
    }
}
```

Time Complexity: O(n^2) because every pair `(i, j)` with `i < j` is examined exactly once via the nested loops.
Space Complexity: O(1) extra space — only a counter variable is used beyond the input array.

**Optimized Approach:**
Use merge sort. While merging two already-sorted halves, whenever an element from the right half is placed before some remaining elements of the left half, all those remaining left-half elements form an inversion with it (since the left half is sorted, if `left[i] > right[j]`, then every element after `left[i]` in the left half is also `> right[j]`). Count these during the merge step instead of doing a separate O(n^2) scan.

```csharp
public class CountInversionsOptimized
{
    // O(n log n) time, O(n) extra space
    public static long CountInversions(int[] arr)
    {
        int[] temp = new int[arr.Length];
        return MergeSortAndCount(arr, temp, 0, arr.Length - 1);
    }

    private static long MergeSortAndCount(int[] arr, int[] temp, int left, int right)
    {
        long count = 0;
        if (left >= right) return count;

        int mid = left + (right - left) / 2;

        count += MergeSortAndCount(arr, temp, left, mid);
        count += MergeSortAndCount(arr, temp, mid + 1, right);
        count += MergeAndCount(arr, temp, left, mid, right);

        return count;
    }

    private static long MergeAndCount(int[] arr, int[] temp, int left, int mid, int right)
    {
        int i = left;      // pointer into left half [left..mid]
        int j = mid + 1;   // pointer into right half [mid+1..right]
        int k = left;      // pointer into temp
        long count = 0;

        while (i <= mid && j <= right)
        {
            if (arr[i] <= arr[j])
            {
                temp[k++] = arr[i++];
            }
            else
            {
                // arr[i] > arr[j]: arr[i..mid] are all > arr[j] because left half is sorted
                count += (mid - i + 1);
                temp[k++] = arr[j++];
            }
        }

        while (i <= mid) temp[k++] = arr[i++];
        while (j <= right) temp[k++] = arr[j++];

        for (int idx = left; idx <= right; idx++)
        {
            arr[idx] = temp[idx];
        }

        return count;
    }
}
```

Time Complexity: O(n log n) — merge sort splits the array in O(log n) levels, and the merge step at each level does O(n) work across all sub-arrays combined, giving O(n log n) overall for both sorting and counting.
Space Complexity: O(n) — the `temp` auxiliary array used during merging (plus O(log n) recursion stack space).

**Explanation:**
Merge sort works by recursively splitting the array into halves until single elements remain, then merging sorted halves back together. During a merge of two sorted halves `left = arr[left..mid]` and `right = arr[mid+1..right]`:

- We walk pointers `i` over the left half and `j` over the right half.
- If `arr[i] <= arr[j]`, no inversion is formed by placing `arr[i]` now; we simply copy it to `temp` and advance `i`.
- If `arr[i] > arr[j]`, then because the left half is sorted in ascending order, **every** element from `arr[i]` to `arr[mid]` is also greater than `arr[j]` (since they are all `>= arr[i]`). That means all of `(mid - i + 1)` elements form an inversion with `arr[j]`. We add `mid - i + 1` to the count in one shot instead of comparing each individually, then copy `arr[j]` to `temp` and advance `j`.
- This way, all cross-inversions between the two halves are counted in O(n) time during the merge, instead of the O(n^2) brute-force nested comparison. Inversions entirely within the left half or entirely within the right half are already counted by the recursive calls on those halves before the merge happens. So total inversions = left-half inversions + right-half inversions + cross inversions counted during merge.

Dry run on `arr = [5, 3, 2, 4, 1]`:
1. Split into `[5, 3, 2]` and `[4, 1]`, recursively split further to single elements, then merge back up.
2. Merging `[5]` and `[3]` → `3 < 5`, so count += 1 (element 5 is greater), result `[3, 5]`.
3. Merging `[3, 5]` and `[2]` → `2` is less than both `3` and `5`; when comparing, `arr[i]=3 > arr[j]=2` triggers `count += (mid - i + 1) = 2` (both 3 and 5 are greater than 2), result `[2, 3, 5]`. Running count = 1 + 2 = 3.
4. Merging `[4]` and `[1]` → `1 < 4`, count += 1, result `[1, 4]`. Running count = 3 + 1 = 4.
5. Merging `[2, 3, 5]` and `[1, 4]`:
   - Compare `2` vs `1`: `2 > 1` → count += (mid - i + 1) = 3 (elements 2, 3, 5 all greater than 1). Place `1`.
   - Compare `2` vs `4`: `2 <= 4` → place `2`.
   - Compare `3` vs `4`: `3 <= 4` → place `3`.
   - Compare `5` vs `4`: `5 > 4` → count += 1 (only element 5 remains, mid-i+1 = 1). Place `4`.
   - Remaining `5` copied directly.
   - Cross-merge count here = 3 + 1 = 4. Running total = 4 + 4 = 8.
6. Final result: `[1, 2, 3, 4, 5]`, total inversion count = 8, matching the expected output.

## Reverse Pairs

### 3. Reverse Pairs

**Problem Statement:**
Given an array `arr` of `n` integers, count the number of "reverse pairs". A reverse pair is a pair of indices `(i, j)` such that `i < j` and `arr[i] > 2 * arr[j]`.

**Example:**
- Input: `arr = [1, 3, 2, 3, 1]`
- Output: `2`
- Explanation: Valid pairs (i<j) with `arr[i] > 2*arr[j]`: index(1,4) → `3 > 2*1=2` true; index(3,4) → `3 > 2*1=2` true. Total = 2.

**Brute Force Approach:**
Use two nested loops: for every pair `(i, j)` with `i < j`, check if `arr[i] > 2 * arr[j]`, and if so increment the count.

```csharp
public class ReversePairsBruteForce
{
    // O(n^2) time, O(1) extra space
    public static long CountReversePairs(int[] arr)
    {
        int n = arr.Length;
        long count = 0;

        for (int i = 0; i < n; i++)
        {
            for (int j = i + 1; j < n; j++)
            {
                // Use long to avoid overflow when computing 2 * arr[j]
                if ((long)arr[i] > 2L * arr[j])
                {
                    count++;
                }
            }
        }

        return count;
    }
}
```

Time Complexity: O(n^2) because every pair `(i, j)` with `i < j` is checked exactly once through the nested loops.
Space Complexity: O(1) extra space — only a counter variable beyond the input array.

**Optimized Approach:**
Use merge sort, similar to Count Inversions, but with a crucial difference: the condition `arr[i] > 2 * arr[j]` is not "merge-compatible" the same way `arr[i] > arr[j]` is, because two pointers moving strictly in lockstep during the merge-and-place step can skip valid pairs (the `2*arr[j]` condition is not monotonic in the same simple sense across the placement loop). So we run a **separate counting pass** over the two already-sorted halves (using two independent pointers) **before** doing the actual merge — while `left[left..mid]` and `right[mid+1..right]` are still in their original (untouched by merge) sorted state. Only after this counting pass do we perform the standard merge (which then also physically sorts the subarray for higher recursion levels).

```csharp
public class ReversePairsOptimized
{
    // O(n log n) time, O(n) extra space
    public static long CountReversePairs(int[] arr)
    {
        int[] temp = new int[arr.Length];
        return MergeSortAndCount(arr, temp, 0, arr.Length - 1);
    }

    private static long MergeSortAndCount(int[] arr, int[] temp, int left, int right)
    {
        long count = 0;
        if (left >= right) return count;

        int mid = left + (right - left) / 2;

        count += MergeSortAndCount(arr, temp, left, mid);
        count += MergeSortAndCount(arr, temp, mid + 1, right);

        // Count cross reverse-pairs BEFORE merging (halves are sorted, but not yet merged/modified)
        count += CountCrossReversePairs(arr, left, mid, right);

        // Now perform the normal merge to produce a sorted [left..right]
        Merge(arr, temp, left, mid, right);

        return count;
    }

    private static long CountCrossReversePairs(int[] arr, int left, int mid, int right)
    {
        long count = 0;
        int j = mid + 1;

        for (int i = left; i <= mid; i++)
        {
            // Advance j while the reverse-pair condition holds; since both halves
            // are sorted, j only moves forward overall (never resets) across the outer loop.
            while (j <= right && (long)arr[i] > 2L * arr[j])
            {
                j++;
            }
            // All elements right[mid+1..j-1] satisfy arr[i] > 2*right[element]
            count += (j - (mid + 1));
        }

        return count;
    }

    private static void Merge(int[] arr, int[] temp, int left, int mid, int right)
    {
        int i = left;
        int j = mid + 1;
        int k = left;

        while (i <= mid && j <= right)
        {
            if (arr[i] <= arr[j])
            {
                temp[k++] = arr[i++];
            }
            else
            {
                temp[k++] = arr[j++];
            }
        }

        while (i <= mid) temp[k++] = arr[i++];
        while (j <= right) temp[k++] = arr[j++];

        for (int idx = left; idx <= right; idx++)
        {
            arr[idx] = temp[idx];
        }
    }
}
```

Time Complexity: O(n log n) — there are O(log n) recursion levels; at each level the counting pass (`CountCrossReversePairs`) is O(n) total because pointer `j` only moves forward and never resets within a call, and the merge step is also O(n) total, giving O(n) work per level and O(n log n) overall.
Space Complexity: O(n) — the `temp` auxiliary array used for merging (plus O(log n) recursion stack space).

**Explanation:**
Just like Count Inversions, recursive calls first count reverse pairs entirely within the left half and entirely within the right half. What remains is counting **cross** reverse pairs where `i` is in the left half and `j` is in the right half, with `arr[i] > 2 * arr[j]`.

Why a separate pass, and why before merging:
- The standard merge step interleaves elements of the two halves into sorted order using a "take the smaller front element" rule; its pointers `i, j` advance based on `arr[i] <= arr[j]`, which correctly captures the inversion condition `arr[i] > arr[j]` as a side effect (as in Count Inversions) because both conditions compare `arr[i]` directly to `arr[j]`.
- But the reverse-pair condition compares `arr[i]` to `2 * arr[j]`, a different threshold. The pointer movements needed to correctly count `arr[i] > 2*arr[j]` pairs do not coincide with the pointer movements needed to produce the merged sorted output. Trying to count during the merge itself would require the merge pointer `j` to sometimes be "ahead of" or "behind" where the counting logic needs it, corrupting either the merge or the count.
- Therefore we do a **dedicated pass first**, while `arr[left..mid]` and `arr[mid+1..right]` are still in their pristine sorted state (sorted by the recursive calls, but not yet interleaved/merged into `arr[left..right]`). In this pass, for each `i` in the left half (moving left to right, values increasing), we advance `j` in the right half as far as `arr[i] > 2*arr[j]` holds. Because the left half is increasing, once `arr[i]` increases, `j` can only need to move further right (never backward) to keep satisfying the shrinking-relative condition — so `j` also moves monotonically forward across the whole loop, giving O(n) total for this pass, not O(n^2).
- Only after this counting pass do we run the ordinary `Merge` to physically produce the sorted `arr[left..right]`, which is needed so that higher levels of recursion see correctly sorted halves for their own counting passes.

Dry run on `arr = [1, 3, 2, 3, 1]`:
1. Recursively split down to single elements and merge back up level by level.
2. Merging `[1]` and `[3]` (left=1 half `[1]`, right half `[3]`): counting pass: i=1(left val), j starts at right half `[3]`; is `1 > 2*3=6`? No. count=0. Merge → `[1,3]`.
3. Merging `[2]` and `[3,1]`... (note: right side `[3,1]` first needs its own merge sort: merging `[3]` and `[1]`: counting pass: is `3 > 2*1=2`? Yes → count=1. Merge → `[1,3]`.) So right half of the top becomes sorted `[1,3]` with 1 reverse pair counted so far.
4. Now merge left half `[1,3]` (from step 2, covering original indices 0-1) with the just-produced `[2]` ... Let's follow the actual recursion tree for `[1,3,2,3,1]` (0-indexed):
   - Split into `[1,3,2]` (idx 0-2) and `[3,1]` (idx 3-4).
   - `[3,1]` (idx 3-4): split into `[3]`,`[1]`. Cross count: `3 > 2*1=2` → count=1. Merge → `[1,3]`.
   - `[1,3,2]` (idx 0-2): split into `[1,3]` (idx 0-1) and `[2]` (idx 2).
     - `[1,3]` (idx 0-1): split into `[1]`,`[3]`. Cross count: `1 > 2*3=6`? No → count=0. Merge → `[1,3]`.
     - Merge `[1,3]` (left, idx0-1) with `[2]` (right, idx2): counting pass: i at value1 → `1>2*2=4`? No. i at value3 → `3>2*2=4`? No. count=0. Merge → `[1,2,3]`.
   - Now merge `[1,2,3]` (idx 0-2, left) with `[1,3]` (idx 3-4, right): counting pass with j starting at first element of right (`1`):
     - i=1 (left val 1): is `1 > 2*1=2`? No → j stays at 0 (pointing to right[0]=1). count += 0.
     - i=2 (left val 2): is `2 > 2*1=2`? No (not strictly greater) → j stays. count += 0.
     - i=3 (left val 3): is `3 > 2*1=2`? Yes → advance j to next right element `3`. Is `3 > 2*3=6`? No → stop. j moved from index0 to index1, so 1 element satisfied → count += 1.
     - This merge step contributes 1 to the count.
   - Merge to produce fully sorted `[1,1,2,3,3]`.
5. Total count = (from `[3,1]` merge) 1 + (from `[1,3]` small merge) 0 + (from `[1,3]`+`[2]` merge) 0 + (from top-level merge) 1 = 2.

Result: 2 reverse pairs, matching the expected output.
