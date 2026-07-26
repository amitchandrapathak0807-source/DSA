# Array — Rearrangement and Permutation

## 1. Rearrange the Array in Alternating Positive and Negative Items

### 1. Rearrange the Array in Alternating Positive and Negative Items

**Problem Statement:**
Given an array of integers containing an equal number of positive and negative numbers, rearrange the array so that positive and negative numbers occupy alternating positions, starting with a positive number at index 0. The relative order of the positive numbers among themselves (and negative numbers among themselves) should be preserved as in the original array.

**Example:**
- Input: `[3, 1, -2, -5, 2, -4]`
- Output: `[3, 2, -2, -5, 1, -4]`
- Explanation: Positive numbers in original relative order are `3, 1, 2`; negative numbers in original relative order are `-2, -5, -4`. Placing them alternately starting with positive gives `3, -2, 1, -5, 2, -4`... but preserving the exact stable interleave, the result places positives at even indices (0, 2, 4) and negatives at odd indices (1, 3, 5) in their original relative order: `3, -2, 1, -5, 2, -4`.

**Brute Force Approach:**
Traverse the array once and collect all positive numbers into one list and all negative numbers into another list, preserving their relative order. Then create the result array and place positive numbers at even indices (0, 2, 4, ...) and negative numbers at odd indices (1, 3, 5, ...), copying from the two lists in order.

```csharp
public List<int> RearrangeBrute(int[] nums)
{
    List<int> positives = new List<int>();
    List<int> negatives = new List<int>();

    // Separate positives and negatives, preserving relative order
    for (int i = 0; i < nums.Length; i++)
    {
        if (nums[i] > 0)
            positives.Add(nums[i]);
        else
            negatives.Add(nums[i]);
    }

    List<int> result = new List<int>(new int[nums.Length]);
    int posIndex = 0, negIndex = 0;

    // Fill even indices with positives, odd indices with negatives
    for (int i = 0; i < nums.Length; i++)
    {
        if (i % 2 == 0)
            result[i] = positives[posIndex++];
        else
            result[i] = negatives[negIndex++];
    }

    return result;
}
```

**Time Complexity:** O(n) — one pass to separate positives/negatives, another pass to merge them back. Two linear passes still amount to O(n).
**Space Complexity:** O(n) — two auxiliary lists (`positives`, `negatives`) plus the result list, each of size up to n.

**Optimized Approach:**
Since the array is guaranteed to have an equal number of positive and negative elements, we can build the answer in a single pass using two pointers: `posIndex` starting at 0 (next even slot for a positive number) and `negIndex` starting at 1 (next odd slot for a negative number). We scan the original array once; whenever we see a positive number we place it at `posIndex` and advance `posIndex` by 2, and whenever we see a negative number we place it at `negIndex` and advance `negIndex` by 2. This avoids the two separate auxiliary lists of the brute force and does it in one pass while still preserving relative order.

```csharp
public int[] RearrangeOptimized(int[] nums)
{
    int n = nums.Length;
    int[] result = new int[n];
    int posIndex = 0; // next available even index for positive numbers
    int negIndex = 1; // next available odd index for negative numbers

    for (int i = 0; i < n; i++)
    {
        if (nums[i] > 0)
        {
            result[posIndex] = nums[i];
            posIndex += 2;
        }
        else
        {
            result[negIndex] = nums[i];
            negIndex += 2;
        }
    }

    return result;
}
```

**Time Complexity:** O(n) — a single pass through the input array, with O(1) work per element.
**Space Complexity:** O(n) — only the output array is used as extra space (required since we must return a new arrangement); no intermediate lists are needed.

**Explanation:**
The optimized approach works because we know in advance exactly where each positive and negative number should land: positives always go to even indices (0, 2, 4, ...) and negatives always go to odd indices (1, 3, 5, ...), and since counts are equal there will be exactly enough slots of each parity. By keeping two independent pointers that jump by 2 each time, we place each element directly into its final position in one left-to-right scan, which naturally preserves the relative order of positives among themselves and negatives among themselves — no need to separately collect and merge them.

---

## 2. Next Permutation

### 2. Next Permutation

**Problem Statement:**
Given an array of integers representing a permutation, rearrange it into the lexicographically next greater permutation of numbers. If no such permutation exists (the array is the highest possible permutation, i.e., sorted in descending order), rearrange it to the lowest possible order (sorted in ascending order). The rearrangement must be done in-place.

**Example:**
- Input: `[1, 2, 3]`
- Output: `[1, 3, 2]`
- Explanation: Among all permutations of `{1, 2, 3}` in lexicographic order — `123, 132, 213, 231, 312, 321` — the permutation right after `123` is `132`.

**Brute Force Approach:**
Generate all possible permutations of the array, sort them lexicographically, find the current permutation in that sorted list, and return the one immediately after it (wrapping around to the first/smallest if the current one is the last). This is conceptually simple but factorially expensive.

```csharp
public void NextPermutationBrute(int[] nums)
{
    List<int[]> allPermutations = new List<int[]>();
    Permute(nums, 0, allPermutations);

    // Sort all permutations lexicographically
    allPermutations.Sort((a, b) =>
    {
        for (int i = 0; i < a.Length; i++)
        {
            if (a[i] != b[i]) return a[i] - b[i];
        }
        return 0;
    });

    // Find current permutation's index
    int currentIndex = -1;
    for (int i = 0; i < allPermutations.Count; i++)
    {
        if (ArraysEqual(allPermutations[i], nums))
        {
            currentIndex = i;
            break;
        }
    }

    // Next permutation (wrap around if it was the last one)
    int[] next = (currentIndex == allPermutations.Count - 1)
        ? allPermutations[0]
        : allPermutations[currentIndex + 1];

    Array.Copy(next, nums, nums.Length);
}

private void Permute(int[] nums, int start, List<int[]> result)
{
    if (start == nums.Length)
    {
        result.Add((int[])nums.Clone());
        return;
    }
    for (int i = start; i < nums.Length; i++)
    {
        Swap(nums, start, i);
        Permute(nums, start + 1, result);
        Swap(nums, start, i); // backtrack
    }
}

private void Swap(int[] nums, int i, int j)
{
    int temp = nums[i];
    nums[i] = nums[j];
    nums[j] = temp;
}

private bool ArraysEqual(int[] a, int[] b)
{
    for (int i = 0; i < a.Length; i++)
        if (a[i] != b[i]) return false;
    return true;
}
```

**Time Complexity:** O(n! · n) — generating all n! permutations takes O(n! · n), plus O(n! log n!) to sort them; overall dominated by the factorial blow-up.
**Space Complexity:** O(n! · n) — storing every permutation of length n.

**Optimized Approach:**
Use the classic in-place algorithm:
1. Scan from the right to find the first index `i` such that `nums[i] < nums[i + 1]` (the "break point" / pivot). This identifies the longest non-increasing suffix.
2. If such an index exists, scan from the right again to find the smallest element in the suffix that is still greater than `nums[i]` (the "next greater" element), and swap it with `nums[i]`.
3. Reverse the suffix starting at index `i + 1` to put it into ascending order (the smallest possible arrangement for that suffix), which yields the immediate next permutation. If no break point was found in step 1, the whole array is in descending order (it is the last permutation), so simply reverse the entire array to get the first (smallest) permutation.

```csharp
public void NextPermutationOptimized(int[] nums)
{
    int n = nums.Length;
    int breakPoint = -1;

    // Step 1: find first index from the right where nums[i] < nums[i+1]
    for (int i = n - 2; i >= 0; i--)
    {
        if (nums[i] < nums[i + 1])
        {
            breakPoint = i;
            break;
        }
    }

    // If no break point, array is the last permutation (fully descending)
    if (breakPoint == -1)
    {
        Array.Reverse(nums);
        return;
    }

    // Step 2: find the smallest element in suffix greater than nums[breakPoint]
    for (int i = n - 1; i > breakPoint; i--)
    {
        if (nums[i] > nums[breakPoint])
        {
            Swap(nums, i, breakPoint);
            break;
        }
    }

    // Step 3: reverse the suffix after breakPoint to make it ascending
    Array.Reverse(nums, breakPoint + 1, n - breakPoint - 1);
}

private void Swap(int[] nums, int i, int j)
{
    int temp = nums[i];
    nums[i] = nums[j];
    nums[j] = temp;
}
```

**Time Complexity:** O(n) — each of the three steps (find break point, find next greater, reverse suffix) is a single linear scan over the array.
**Space Complexity:** O(1) — the rearrangement is done in-place using only a constant number of extra variables.

**Explanation:**
Let's dry run with `nums = [2, 1, 5, 4, 3, 0, 0]`.

Step 1 — Find the break point: We scan from the right comparing `nums[i]` with `nums[i+1]`, looking for the first place where the sequence stops being non-increasing (i.e., where `nums[i] < nums[i+1]`). Reading right to left: `0` vs `0` (not less), `0` vs `3` (not less, 0<3 is actually true — let's index carefully).

Indices: `0:2, 1:1, 2:5, 3:4, 4:3, 5:0, 6:0`.
- i=5: nums[5]=0, nums[6]=0 → 0<0 false.
- i=4: nums[4]=3, nums[5]=0 → 3<0 false.
- i=3: nums[3]=4, nums[4]=3 → 4<3 false.
- i=2: nums[2]=5, nums[3]=4 → 5<4 false.
- i=1: nums[1]=1, nums[2]=5 → 1<5 true. breakPoint = 1.

Why we look for this from the right: the suffix after the break point (`5, 4, 3, 0, 0`) is already the largest possible arrangement of those digits (non-increasing), so no rearrangement of just the suffix can make the whole number bigger — we must involve `nums[breakPoint]=1` to get a bigger permutation, and `1` is the rightmost digit that can be increased by swapping with something in the suffix.

Step 2 — Find the next greater element in the suffix: We scan the suffix (`index 2..6`) from the right and find the smallest value that is still greater than `nums[breakPoint] = 1`. Scanning right to left: 0 (no, not > 1), 0 (no), 3 (yes, 3 > 1) → pick index 4 (value 3). We pick the smallest such value (found by scanning from the right, since the suffix is non-increasing, the first value from the right that exceeds `nums[breakPoint]` is the smallest one that does) because we want the smallest possible increase — this guarantees we get the *immediate next* permutation, not a much larger jump.

Swap `nums[1]=1` and `nums[4]=3`: array becomes `[2, 3, 5, 4, 1, 0, 0]`.

Step 3 — Reverse the suffix: The suffix after the break point (`index 2..6`: `5, 4, 1, 0, 0`) is still in non-increasing order (largest-to-smallest) because swapping only replaced one element while preserving the relative descending order of the rest. To get the smallest possible arrangement of this suffix (and thus the smallest possible increase overall, i.e., the *next* permutation rather than some larger one), we reverse it to ascending order: `0, 0, 1, 4, 5`.

Final array: `[2, 3, 0, 0, 1, 4, 5]`, which is indeed the lexicographically next permutation after `[2, 1, 5, 4, 3, 0, 0]`.

If no break point exists (e.g., `[3, 2, 1]`), the entire array is in descending order, meaning it is already the largest permutation possible; there is no "next" one, so by convention we wrap around to the smallest permutation, obtained by simply reversing the whole array to ascending order (`[1, 2, 3]`).

---

## 3. Leaders in an Array

### 3. Leaders in an Array

**Problem Statement:**
Given an array of integers, find all the "leader" elements. An element is a leader if it is strictly greater than all the elements to its right. The rightmost element is always a leader (there is nothing to its right). Return the leaders in the order they appear in the array (or, depending on convention, in the order found — here we return them left-to-right).

**Example:**
- Input: `[10, 22, 12, 3, 0, 6]`
- Output: `[22, 12, 6]`
- Explanation: `22` is greater than everything to its right (`12, 3, 0, 6`). `12` is greater than everything to its right (`3, 0, 6`). `6` is the last element, so it's trivially a leader. `10` is not a leader because `22` is to its right and is larger. `3` and `0` are not leaders because `6` is to their right.

**Brute Force Approach:**
For every element, check all elements to its right; if the current element is strictly greater than all of them, it is a leader. This uses two nested loops.

```csharp
public List<int> LeadersBrute(int[] nums)
{
    List<int> leaders = new List<int>();
    int n = nums.Length;

    for (int i = 0; i < n; i++)
    {
        bool isLeader = true;
        for (int j = i + 1; j < n; j++)
        {
            if (nums[j] >= nums[i])
            {
                isLeader = false;
                break;
            }
        }
        if (isLeader)
            leaders.Add(nums[i]);
    }

    return leaders;
}
```

**Time Complexity:** O(n^2) — for each of the n elements, we may scan up to n elements to its right in the worst case.
**Space Complexity:** O(1) extra (excluding the output list) — no auxiliary data structures beyond the result.

**Optimized Approach:**
Traverse the array from right to left while keeping track of the maximum value seen so far (`maxSoFar`). The rightmost element is always a leader (initialize `maxSoFar` with it). For each element moving leftward, if the current element is strictly greater than `maxSoFar`, it is a leader; update `maxSoFar` to this element. Since we build the leaders while scanning right-to-left, we either insert at the front or collect them and reverse at the end to preserve left-to-right order in the output.

```csharp
public List<int> LeadersOptimized(int[] nums)
{
    List<int> leaders = new List<int>();
    int n = nums.Length;
    if (n == 0) return leaders;

    int maxSoFar = nums[n - 1];
    leaders.Add(maxSoFar); // rightmost element is always a leader

    for (int i = n - 2; i >= 0; i--)
    {
        if (nums[i] > maxSoFar)
        {
            leaders.Add(nums[i]);
            maxSoFar = nums[i];
        }
    }

    leaders.Reverse(); // restore left-to-right order
    return leaders;
}
```

**Time Complexity:** O(n) — a single right-to-left pass through the array, with O(1) work per element (the final reverse is also O(n)).
**Space Complexity:** O(1) extra (excluding the output list) — only a single `maxSoFar` variable is needed during the scan.

**Explanation:**
Scanning from the right lets us maintain a running maximum (`maxSoFar`) of everything we've already visited, which is exactly "everything to the right" of the current position. An element is a leader precisely when it beats this running maximum, so a single comparison per element replaces the inner loop of the brute force. Since leaders are discovered in right-to-left order, we either prepend them or (as done here) append and reverse at the end to present them in their natural left-to-right array order.

---

## 4. Pascal's Triangle

### 4. Pascal's Triangle

**Problem Statement:**
Pascal's Triangle is a triangular array where each row starts and ends with 1, and every other entry is the sum of the two entries directly above it in the previous row. Row `r` (0-indexed) and column `c` (0-indexed, `0 <= c <= r`) of Pascal's Triangle equals the binomial coefficient `C(r, c) = r! / (c! * (r-c)!)`. This problem has three common variants:
1. Find the value at a specific `(row, col)`.
2. Generate an entire given row.
3. Print/generate the entire triangle up to `n` rows.

**Example:**
- Input (Variant 1): `row = 4, col = 2`
- Output: `6`
- Explanation: Row 4 of Pascal's Triangle is `1, 4, 6, 4, 1`; the element at column index 2 is `6`, which equals `C(4, 2) = 4! / (2! * 2!) = 6`.

- Input (Variant 2): `row = 4`
- Output: `[1, 4, 6, 4, 1]`
- Explanation: Each row `r` has `r + 1` elements, computed as `C(r, 0), C(r, 1), ..., C(r, r)`.

- Input (Variant 3): `n = 5`
- Output:
  ```
  [1]
  [1, 1]
  [1, 2, 1]
  [1, 3, 3, 1]
  [1, 4, 6, 4, 1]
  ```
- Explanation: The full triangle with 5 rows (rows 0 through 4), each generated the same way as variant 2.

**Brute Force Approach:**
For each requested value, directly compute the factorial-based binomial coefficient formula `C(r, c) = r! / (c! * (r - c)!)` using a helper factorial function. For a full row, loop `c` from `0` to `row` and compute each value this way; for the full triangle, loop over every row and every column. Factorials grow extremely fast and can overflow standard integer types even for modest inputs, which is the main drawback.

```csharp
private long Factorial(int num)
{
    long result = 1;
    for (int i = 2; i <= num; i++)
        result *= i;
    return result;
}

// Variant 1: element at (row, col)
public long PascalElementBrute(int row, int col)
{
    return Factorial(row) / (Factorial(col) * Factorial(row - col));
}

// Variant 2: entire given row
public List<long> PascalRowBrute(int row)
{
    List<long> result = new List<long>();
    for (int col = 0; col <= row; col++)
    {
        result.Add(PascalElementBrute(row, col));
    }
    return result;
}

// Variant 3: full triangle with n rows
public List<List<long>> PascalTriangleBrute(int n)
{
    List<List<long>> triangle = new List<List<long>>();
    for (int row = 0; row < n; row++)
    {
        triangle.Add(PascalRowBrute(row));
    }
    return triangle;
}
```

**Time Complexity:**
- Variant 1: O(row) — computing each factorial takes O(row) time.
- Variant 2: O(row^2) — computing `row + 1` elements, each an O(row) factorial computation.
- Variant 3: O(n^3) — for n rows, each row costs O(row^2), summing to O(n^3).

**Space Complexity:** O(1) extra for Variant 1 (excluding output); O(row) for Variant 2's output list; O(n^2) for Variant 3's output (total elements across all rows). Factorial computation itself uses O(1) extra space.

**Optimized Approach:**
Instead of computing full factorials (which overflow quickly and redo a lot of multiplication), use the multiplicative formula for `nCr` that builds the value incrementally:

`C(n, r) = C(n, r-1) * (n - r + 1) / r`

Starting from `C(n, 0) = 1`, each next term is obtained by multiplying by `(n - r + 1)` and dividing by `r`. This keeps intermediate numbers much smaller and avoids ever computing a full factorial. For a whole row we compute this iteratively left to right, reusing the previous term. For the whole triangle we generate each row this way (this is also equivalent to the well-known additive approach where each row is built from the previous one, but the multiplicative nCr trick is used here per the problem's instructions).

```csharp
// nCr using the multiplicative trick — avoids overflow and recomputation
private long NCR(int n, int r)
{
    long result = 1;
    // C(n, r) = C(n, r-1) * (n - r + 1) / r, built incrementally from C(n,0) = 1
    for (int i = 0; i < r; i++)
    {
        result *= (n - i);
        result /= (i + 1);
    }
    return result;
}

// Variant 1: element at (row, col) — O(col) via the multiplicative trick
public long PascalElementOptimized(int row, int col)
{
    return NCR(row, col);
}

// Variant 2: entire given row — build incrementally, reusing the previous value
public List<long> PascalRowOptimized(int row)
{
    List<long> result = new List<long>();
    long value = 1; // C(row, 0)
    result.Add(value);

    for (int col = 1; col <= row; col++)
    {
        // C(row, col) = C(row, col-1) * (row - col + 1) / col
        value = value * (row - col + 1) / col;
        result.Add(value);
    }

    return result;
}

// Variant 3: full triangle with n rows, each row built with the incremental trick
public List<List<long>> PascalTriangleOptimized(int n)
{
    List<List<long>> triangle = new List<List<long>>();
    for (int row = 0; row < n; row++)
    {
        triangle.Add(PascalRowOptimized(row));
    }
    return triangle;
}
```

**Time Complexity:**
- Variant 1: O(col) — the multiplicative trick computes `C(row, col)` in `col` incremental steps instead of full factorials.
- Variant 2: O(row) — each of the `row + 1` elements is derived from the previous one in O(1), so the whole row takes linear time.
- Variant 3: O(n^2) — generating n rows, where row `r` takes O(r) time, summing to O(n^2) overall — a significant improvement over the brute force's O(n^3).

**Space Complexity:** O(1) extra for Variant 1 (excluding output); O(row) for Variant 2's output; O(n^2) total for Variant 3's output (sum of all row lengths). No factorial storage is needed at any point.

**Explanation:**
The nCr multiplicative trick avoids computing full factorials (which grow astronomically large and quickly overflow even 64-bit integers for moderately sized rows) by expressing `C(n, r)` as a running product: starting at `C(n, 0) = 1`, each subsequent term `C(n, k)` is obtained from `C(n, k-1)` by multiplying by `(n - k + 1)` and dividing by `k`. This works because algebraically `C(n,k)/C(n,k-1) = (n-k+1)/k`, so we never need the full `n!`, `k!`, or `(n-k)!` — only small, incremental multiply/divide steps. This is both faster (no redundant recomputation of shared factorial terms across columns) and safer against overflow than the brute-force factorial-division formula, since intermediate values stay proportional to the final result rather than to the much larger factorial terms. For the full triangle, the same incremental logic is reused per row, so the total work is proportional to the total number of elements in the triangle rather than repeating expensive per-element factorial computations.
