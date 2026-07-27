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

**Logic (Steps):**
1. **Nested-loop version:** for each candidate `i` from `1` to `n`, scan the entire array counting how many times `i` appears.
2. If the count is `2`, set `repeating = i`; if the count is `0`, set `missing = i`.
3. **Frequency-array version:** build `freq` of size `n+1`, incrementing `freq[num]` for every element in one pass over `arr`.
4. Scan `i` from `1` to `n`: `freq[i] == 2` marks `repeating`, `freq[i] == 0` marks `missing`.
5. Return the `(repeating, missing)` tuple.

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

**Walkthrough:** Using `arr = [3, 1, 2, 5, 3]`, `n = 5` (frequency-array version).
- Build `freq[1..5] = [1, 1, 2, 0, 1]` (index = value).
- Scan: `freq[1]=1`, `freq[2]=1`, `freq[3]=2` → `repeating=3`, `freq[4]=0` → `missing=4`, `freq[5]=1`.
Returned `(repeating, missing) = (3, 4)`, matching the expected output.

---

**Optimized Approach:**
Use the mathematical (sum and sum-of-squares) trick to solve in O(n) time and O(1) extra space, without any hashing.

Let `S` = sum of first `n` natural numbers = `n(n+1)/2`, and `S2` = sum of squares of first `n` natural numbers = `n(n+1)(2n+1)/6`.
Let `ActualSum` = sum of the given array, and `ActualSum2` = sum of squares of the given array.

Because the array has one number repeated (`repeating`) and one missing (`missing`) instead of the "perfect" set `1..n`:
- `ActualSum - S = repeating - missing`  → call this `diff = repeating - missing`
- `ActualSum2 - S2 = repeating^2 - missing^2 = (repeating - missing)(repeating + missing)`

Dividing the second equation by `diff` gives `repeating + missing`. Combined with `diff`, we solve a simple 2-variable linear system for both values.

**Logic (Steps):**
1. Compute `expectedSum = n(n+1)/2` and `expectedSumSq = n(n+1)(2n+1)/6` (the perfect `1..n` sums).
2. Traverse `arr` once, accumulating `actualSum` and `actualSumSq` (sum and sum of squares of the given array).
3. Compute `diff1 = actualSum - expectedSum`, which equals `repeating - missing`.
4. Compute `diff2 = actualSumSq - expectedSumSq`, which equals `(repeating - missing)(repeating + missing)`; divide by `diff1` to get `sum1 = repeating + missing`.
5. Solve the 2-variable system: `repeating = (diff1 + sum1) / 2`, then `missing = repeating - diff1`, and return the pair.

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

**Walkthrough:** Dry run on `arr = [3, 1, 2, 5, 3]`, n = 5:
- `expectedSum = 5*6/2 = 15`, `expectedSumSq = 5*6*11/6 = 55`
- `actualSum = 3+1+2+5+3 = 14`, `actualSumSq = 9+1+4+25+9 = 48`
- `diff1 = actualSum - expectedSum = 14 - 15 = -1` → `repeating - missing = -1`
- `diff2 = actualSumSq - expectedSumSq = 48 - 55 = -7`
- `sum1 = diff2 / diff1 = -7 / -1 = 7` → `repeating + missing = 7`
- `repeating = (diff1 + sum1) / 2 = (-1 + 7) / 2 = 3`
- `missing = repeating - diff1 = 3 - (-1) = 4`
Returned `(repeating, missing) = (3, 4)`, matching the expected output.

---

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

**Logic (Steps):**
1. Initialize `count = 0`.
2. Loop the outer index `i` from `0` to `n-1`.
3. Loop the inner index `j` from `i+1` to `n-1`.
4. If `arr[i] > arr[j]`, increment `count` (this pair is an inversion).
5. Return `count` after both loops finish.

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

**Walkthrough:** Using `arr = [5, 3, 2, 4, 1]`.
- `i=0`: `5>3`,`5>2`,`5>4`,`5>1` → 4 inversions.
- `i=1`: `3>2`,`3>1` → 2 inversions (`3<4` doesn't count).
- `i=2`: `2>1` → 1 inversion.
- `i=3`: `4>1` → 1 inversion.
Total `count = 4+2+1+1 = 8`, matching the expected output.

---

**Optimized Approach:**
Use merge sort. While merging two already-sorted halves, whenever an element from the right half is placed before some remaining elements of the left half, all those remaining left-half elements form an inversion with it (since the left half is sorted, if `left[i] > right[j]`, then every element after `left[i]` in the left half is also `> right[j]`). Count these during the merge step instead of doing a separate O(n^2) scan.

**Logic (Steps):**
1. Recursively split `arr[left..right]` into two halves at `mid`, and recurse on each half, accumulating their inversion counts.
2. Merge the two now-sorted halves using pointers `i` (left half) and `j` (right half), writing into `temp`.
3. While merging, if `arr[i] <= arr[j]`, copy `arr[i]` and advance `i` (no inversion).
4. If `arr[i] > arr[j]`, then every remaining element from `i` to `mid` is also greater than `arr[j]` (left half is sorted), so add `mid - i + 1` to the count in one shot, copy `arr[j]`, and advance `j`.
5. Copy any leftover elements, write `temp` back into `arr[left..right]`, and return the sum of left-half, right-half, and cross-merge inversion counts.

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

**Walkthrough:** Dry run on `arr = [5, 3, 2, 4, 1]`:
1. Split into `[5, 3, 2]` and `[4, 1]`, recursively split further to single elements, then merge back up.
2. Merging `[5]` and `[3]` → `3 < 5`, count += 1, result `[3, 5]`.
3. Merging `[3, 5]` and `[2]` → `arr[i]=3 > arr[j]=2` triggers `count += (mid-i+1) = 2` (both 3 and 5 greater than 2), result `[2, 3, 5]`. Running count = 1 + 2 = 3.
4. Merging `[4]` and `[1]` → `1 < 4`, count += 1, result `[1, 4]`. Running count = 3 + 1 = 4.
5. Merging `[2, 3, 5]` and `[1, 4]`: `2>1` → count += 3 (2,3,5 all > 1), place `1`; `2<=4` place `2`; `3<=4` place `3`; `5>4` → count += 1, place `4`; remaining `5` copied. Cross-merge count = 3 + 1 = 4. Running total = 4 + 4 = 8.
Final sorted `[1, 2, 3, 4, 5]`, total inversion count = 8, matching the expected output.

---

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

**Logic (Steps):**
1. Initialize `count = 0`.
2. Loop the outer index `i` from `0` to `n-1`.
3. Loop the inner index `j` from `i+1` to `n-1`.
4. If `(long)arr[i] > 2L * arr[j]`, increment `count` (using `long` to avoid overflow).
5. Return `count` after both loops finish.

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

**Walkthrough:** Using `arr = [1, 3, 2, 3, 1]`.
- `i=0` (`1`): `j=1..4`: `1>2*3=6`? No. `1>2*2=4`? No. `1>2*3=6`? No. `1>2*1=2`? No. 0 pairs.
- `i=1` (`3`): `j=2..4`: `3>2*2=4`? No. `3>2*3=6`? No. `3>2*1=2`? Yes → 1 pair.
- `i=2` (`2`): `j=3..4`: `2>2*3=6`? No. `2>2*1=2`? No. 0 pairs.
- `i=3` (`3`): `j=4`: `3>2*1=2`? Yes → 1 pair.
Total `count = 1 + 1 = 2`, matching the expected output.

---

**Optimized Approach:**
Use merge sort, similar to Count Inversions, but with a crucial difference: the condition `arr[i] > 2 * arr[j]` is not "merge-compatible" the same way `arr[i] > arr[j]` is, because two pointers moving strictly in lockstep during the merge-and-place step can skip valid pairs (the `2*arr[j]` condition is not monotonic in the same simple sense across the placement loop). So we run a **separate counting pass** over the two already-sorted halves (using two independent pointers) **before** doing the actual merge — while `left[left..mid]` and `right[mid+1..right]` are still in their original (untouched by merge) sorted state. Only after this counting pass do we perform the standard merge (which then also physically sorts the subarray for higher recursion levels).

**Logic (Steps):**
1. Recursively split `arr[left..right]` at `mid`, recurse on each half, and accumulate their reverse-pair counts.
2. Before merging, run `CountCrossReversePairs`: for each `i` in the sorted left half, advance a pointer `j` (starting at `mid+1`, never resetting) through the sorted right half while `arr[i] > 2*arr[j]` holds.
3. After the inner `while` stops, add `j - (mid+1)` to the count — the number of right-half elements satisfying the condition for this `i`.
4. Since both halves are sorted and `j` never moves backward across the outer loop, this cross-counting pass is O(n) total per level.
5. Only after counting, run the ordinary `Merge` to physically produce the sorted `arr[left..right]` for higher recursion levels, and return the total count.

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

**Walkthrough:** Dry run on `arr = [1, 3, 2, 3, 1]` (0-indexed), recursion tree splits into `[1,3,2]` (idx 0-2) and `[3,1]` (idx 3-4):
- `[3,1]` (idx 3-4): split into `[3]`,`[1]`. Cross count: `3 > 2*1=2` → count=1. Merge → `[1,3]`.
- `[1,3,2]` (idx 0-2): split into `[1,3]` (idx 0-1) and `[2]` (idx 2). `[1,3]` cross count: `1>2*3=6`? No → count=0, merge → `[1,3]`. Merge `[1,3]` with `[2]`: `1>2*2=4`? No; `3>2*2=4`? No → count=0, merge → `[1,2,3]`.
- Top-level merge of `[1,2,3]` (left) with `[1,3]` (right), `j` starting at right index 0: `i=1`(`1>2*1=2`? No, `j` stays); `i=2`(`2>2*1=2`? No, `j` stays); `i=3`(`3>2*1=2`? Yes → advance `j` to index1; `3>2*3=6`? No → stop; `j` moved 1 step → count += 1). This merge contributes 1.
Total count = 1 (from `[3,1]`) + 0 + 0 + 1 (top-level) = 2, matching the expected output.
