# Bit Manipulation — Problems

## 1. Power Set — Print All Possible Subsets Using Bit Masking

**Problem Statement:**
Given an array (or string) of `n` distinct elements, print/return all possible subsets (the power set). A set of `n` elements has exactly `2^n` subsets, including the empty set and the full set itself.

**Example:**
- Input: `arr = [1, 2, 3]`
- Output: `[[], [1], [2], [1,2], [3], [1,3], [2,3], [1,2,3]]`
- Explanation: Every element can either be included or excluded from a subset, giving `2^3 = 8` total subsets.

**Brute Force Approach:**
Use recursion. At every index, branch into two recursive calls — one where the current element is included in the running subset, and one where it is excluded. When the index reaches the end of the array, the accumulated subset is a valid answer.

**Logic (Steps):**
1. Define `Recurse(index, current)`: the base case is `index == arr.Length`, at which point `current` is a complete subset and gets copied into `result`.
2. First branch: recurse to `index + 1` without adding `arr[index]` (exclude case).
3. Second branch: add `arr[index]` to `current`, recurse to `index + 1` (include case), then remove it (backtrack) so `current` is clean for the caller's next branch.
4. Because every index makes an independent include/exclude choice, all `2^n` combinations get explored.

```csharp
public class PowerSetBruteForce
{
    public static IList<IList<int>> SubsetsRecursive(int[] arr)
    {
        var result = new List<IList<int>>();
        var current = new List<int>();
        Recurse(0, arr, current, result);
        return result;
    }

    private static void Recurse(int index, int[] arr, List<int> current, List<IList<int>> result)
    {
        if (index == arr.Length)
        {
            result.Add(new List<int>(current));
            return;
        }

        // Exclude arr[index]
        Recurse(index + 1, arr, current, result);

        // Include arr[index]
        current.Add(arr[index]);
        Recurse(index + 1, arr, current, result);
        current.RemoveAt(current.Count - 1);
    }
}
```

Time Complexity: `O(2^n * n)` — there are `2^n` subsets and each subset takes up to `O(n)` time to copy.
Space Complexity: `O(2^n * n)` to store all subsets (plus `O(n)` recursion stack).

**Walkthrough:** For `arr = [1, 2, 3]`, `Recurse(0, [])` branches: exclude 1 → exclude 2 → exclude 3 → `[]` added; exclude 1 → exclude 2 → include 3 → `[3]` added; and so on through all combinations, backtracking after each include. After all branches finish, `result` contains all 8 subsets `[[], [3], [2], [2,3], [1], [1,3], [1,2], [1,2,3]]` (order depends on recursion), matching the expected 8-subset output.

**Optimized Approach:**
Since each of the `n` elements has exactly two states (present / absent), a subset can be represented by an `n`-bit number (mask) ranging from `0` to `2^n - 1`. For a given mask, bit `j` (from the right) being `1` means "include `arr[j]`" and `0` means "exclude `arr[j]`". Iterating the mask from `0` to `2^n - 1` and checking each bit generates every subset exactly once, with no recursion needed.

**Logic (Steps):**
1. Compute `totalMasks = 1 << n` (i.e. `2^n`), the total number of subsets.
2. For each `mask` from `0` to `totalMasks - 1`, build a subset by checking every bit `j` from `0` to `n-1`.
3. If bit `j` of `mask` is set (`mask & (1 << j) != 0`), include `arr[j]` in the current subset.
4. Add the completed subset to `result`; repeat for the next mask until all masks are processed.

```csharp
public class PowerSetOptimized
{
    public static IList<IList<int>> Subsets(int[] arr)
    {
        int n = arr.Length;
        int totalMasks = 1 << n; // 2^n
        var result = new List<IList<int>>();

        for (int mask = 0; mask < totalMasks; mask++)
        {
            var subset = new List<int>();
            for (int j = 0; j < n; j++)
            {
                // Check if the j-th bit is set in mask
                if ((mask & (1 << j)) != 0)
                {
                    subset.Add(arr[j]);
                }
            }
            result.Add(subset);
        }

        return result;
    }
}
```

Time Complexity: `O(2^n * n)` — `2^n` masks, and for each mask we inspect `n` bits.
Space Complexity: `O(2^n * n)` to hold the output; `O(1)` extra working space besides the output.

**Walkthrough:** For `arr = [1, 2, 3]` (n = 3), masks range from `000` to `111`: `mask=000→[]`, `mask=001→[1]`, `mask=010→[2]`, `mask=011→[1,2]`, `mask=100→[3]`, `mask=101→[1,3]`, `mask=110→[2,3]`, `mask=111→[1,2,3]`. All 8 masks together produce every subset exactly once, matching the expected output `[[], [1], [2], [1,2], [3], [1,3], [2,3], [1,2,3]]`.

---

## 2. Find XOR of Numbers from L to R

**Problem Statement:**
Given two integers `L` and `R`, find the XOR of all numbers in the inclusive range `[L, R]`, i.e. compute `L XOR (L+1) XOR (L+2) XOR ... XOR R`.

**Example:**
- Input: `L = 3, R = 9`
- Output: `2`
- Explanation: `3 ^ 4 ^ 5 ^ 6 ^ 7 ^ 8 ^ 9 = 2` (verified by direct XOR of all numbers in range).

**Brute Force Approach:**
Iterate through every number from `L` to `R` and XOR them all together in a running accumulator.

**Logic (Steps):**
1. Initialize `result = 0`.
2. Loop `i` from `L` to `R` inclusive.
3. XOR each `i` into `result` (`result ^= i`).
4. After the loop, `result` holds the XOR of the whole range.

```csharp
public class XorRangeBruteForce
{
    public static int XorFromLToR(int L, int R)
    {
        int result = 0;
        for (int i = L; i <= R; i++)
        {
            result ^= i;
        }
        return result;
    }
}
```

Time Complexity: `O(R - L)` — one iteration per number in the range.
Space Complexity: `O(1)`.

**Walkthrough:** For `L = 3, R = 9`: `result` accumulates `3^4=7, 7^5=2, 2^6=4, 4^7=3, 3^8=11, 11^9=2`. Final `result = 2`, matching the expected output.

**Optimized Approach:**
Define `xorFrom1ToN(n)` = XOR of all integers from `1` to `n`. This has a well-known closed form based on `n % 4`:
- `n % 4 == 0` → answer is `n`
- `n % 4 == 1` → answer is `1`
- `n % 4 == 2` → answer is `n + 1`
- `n % 4 == 3` → answer is `0`

Then, using the prefix-XOR idea (similar to prefix sums), the XOR of `[L, R]` equals `xorFrom1ToN(R) XOR xorFrom1ToN(L - 1)`, because XOR-ing `1..(L-1)` twice cancels out the prefix that shouldn't be included.

**Logic (Steps):**
1. Implement `XorFrom1ToN(n)` using the `n % 4` pattern to get the XOR of `1..n` in O(1).
2. Compute `XorFrom1ToN(R)`, the XOR of `1..R`.
3. Compute `XorFrom1ToN(L - 1)`, the XOR of `1..(L-1)`.
4. XOR these two results together; the prefix `1..(L-1)` cancels itself out, leaving the XOR of exactly `L..R`.

```csharp
public class XorRangeOptimized
{
    // XOR of all integers from 1 to n, using the n % 4 pattern
    public static int XorFrom1ToN(int n)
    {
        if (n < 0) return 0;

        switch (n % 4)
        {
            case 0: return n;
            case 1: return 1;
            case 2: return n + 1;
            default: return 0; // n % 4 == 3
        }
    }

    public static int XorFromLToR(int L, int R)
    {
        return XorFrom1ToN(R) ^ XorFrom1ToN(L - 1);
    }
}
```

Time Complexity: `O(1)` — only constant-time arithmetic and modulo operations.
Space Complexity: `O(1)`.

**Walkthrough:** For `L = 3, R = 9`: `XorFrom1ToN(9)` → `9 % 4 = 1` → returns `1`. `XorFrom1ToN(L-1) = XorFrom1ToN(2)` → `2 % 4 = 2` → returns `2 + 1 = 3`. Final result: `1 ^ 3 = 2`, matching the expected output `2`. This works because XOR-ing `XorFrom1ToN(R)` with `XorFrom1ToN(L-1)` cancels every number from `1` to `L-1` (each appears twice, and `x ^ x = 0`), leaving only the XOR of `L..R`.

---

## 3. Divide Two Integers Without Using Multiplication, Division, or Mod Operators

**Problem Statement:**
Given two integers `dividend` and `divisor`, divide them without using the `*`, `/`, or `%` operators. Return the integer quotient, truncating toward zero. Handle sign correctly (result is negative if exactly one of `dividend`/`divisor` is negative). Assume standard 32-bit integer overflow constraints (clamp to `int.MaxValue` if the result overflows).

**Example:**
- Input: `dividend = 22, divisor = 3`
- Output: `7`
- Explanation: `3 * 7 = 21 <= 22`, and `3 * 8 = 24 > 22`, so quotient is `7` (remainder 1 discarded).

**Brute Force Approach:**
Repeatedly subtract the divisor from the (absolute value of) dividend, counting how many subtractions occur until the remaining value is smaller than the divisor. That count is the quotient.

**Logic (Steps):**
1. Determine the sign of the result by comparing the signs of `dividend` and `divisor`; work with absolute values from here on.
2. Repeatedly subtract `absDivisor` from `absDividend`, incrementing `quotient` each time, while `absDividend >= absDivisor`.
3. Once `absDividend < absDivisor`, `quotient` holds the number of subtractions performed (the integer quotient).
4. Apply the sign, then clamp the result to `[int.MinValue, int.MaxValue]` before returning.

```csharp
public class DivideBruteForce
{
    public static int Divide(int dividend, int divisor)
    {
        if (dividend == 0) return 0;

        bool negativeResult = (dividend < 0) != (divisor < 0);

        long absDividend = Math.Abs((long)dividend);
        long absDivisor = Math.Abs((long)divisor);

        long quotient = 0;
        while (absDividend >= absDivisor)
        {
            absDividend -= absDivisor;
            quotient++;
        }

        long result = negativeResult ? -quotient : quotient;

        if (result > int.MaxValue) return int.MaxValue;
        if (result < int.MinValue) return int.MinValue;
        return (int)result;
    }
}
```

Time Complexity: `O(dividend / divisor)` — in the worst case (e.g. dividing by 1), this degrades to `O(n)` subtractions where `n` is the dividend, which is very slow for large values.
Space Complexity: `O(1)`.

**Walkthrough:** For `dividend = 22, divisor = 3`: subtract 3 repeatedly — `22→19→16→13→10→7→4→1`, seven subtractions performed before `1 < 3`. `quotient = 7`, matching the expected output `7`.

**Optimized Approach:**
Instead of subtracting the divisor one unit at a time, subtract the largest possible multiple of the divisor at each step using bit-shift doubling. For the current remaining dividend, start with `pivot = divisor` and `multiple = 1`, and keep doubling both (`pivot <<= 1`, `multiple <<= 1`) as long as `pivot` still fits within the remaining dividend. Once doubling would overshoot, subtract `pivot` from the dividend, add `multiple` to the quotient, and restart the doubling process from `divisor` again. This is effectively binary long division and runs in `O(log^2 n)`.

**Logic (Steps):**
1. Handle the `dividend == 0` and `int.MinValue / -1` overflow edge cases up front, and determine the result's sign from the operand signs.
2. Work with absolute values `absDividend`, `absDivisor`. While `absDividend >= absDivisor`, run an inner doubling loop: start `pivot = absDivisor`, `multiple = 1`, and repeatedly double both (`pivot <<= 1`, `multiple <<= 1`) as long as `pivot << 1` still fits within `absDividend`.
3. After the inner loop stops, subtract the found `pivot` from `absDividend` and add `multiple` to `quotient` — this removes the largest possible power-of-two multiple of the divisor in one step.
4. Repeat the outer loop until `absDividend < absDivisor`; apply the sign and clamp to `int` bounds before returning `quotient`.

```csharp
public class DivideOptimized
{
    public static int Divide(int dividend, int divisor)
    {
        if (dividend == 0) return 0;
        if (dividend == int.MinValue && divisor == -1) return int.MaxValue; // overflow guard

        bool negativeResult = (dividend < 0) != (divisor < 0);

        long absDividend = Math.Abs((long)dividend);
        long absDivisor = Math.Abs((long)divisor);

        long quotient = 0;

        while (absDividend >= absDivisor)
        {
            long pivot = absDivisor;
            long multiple = 1;

            // Double the pivot (and the multiple) while it still fits
            while (absDividend >= (pivot << 1))
            {
                pivot <<= 1;
                multiple <<= 1;
            }

            absDividend -= pivot;
            quotient += multiple;
        }

        long result = negativeResult ? -quotient : quotient;

        if (result > int.MaxValue) return int.MaxValue;
        if (result < int.MinValue) return int.MinValue;
        return (int)result;
    }
}
```

Time Complexity: `O(log^2 n)` where `n = dividend`. The outer loop runs `O(log n)` times (each iteration removes at least half of what remains), and the inner doubling loop is also `O(log n)`.
Space Complexity: `O(1)` — only a few long variables used.

**Walkthrough:** For `dividend = 22, divisor = 3`: outer iteration 1 doubles `pivot` from 3 → 6 → 12 (stops since `24 > 22`), subtracts `12` leaving `absDividend = 10`, `quotient = 4`. Iteration 2 doubles `pivot` 3 → 6 (stops since `12 > 10`), subtracts `6` leaving `absDividend = 4`, `quotient = 6`. Iteration 3 can't double past `3` (since `6 > 4`), subtracts `3` leaving `absDividend = 1`, `quotient = 7`. Now `1 < 3` so the loop ends, returning `7`, matching the expected output.

---

## 4. Single Number II — Every Element Appears Thrice Except One

**Problem Statement:**
Given a non-empty array of integers where every element appears exactly three times, except for one element which appears exactly once, find that single element. Must run in linear time and, ideally, constant extra space.

**Example:**
- Input: `nums = [2, 2, 3, 2]`
- Output: `3`
- Explanation: `2` appears three times and `3` appears once, so `3` is the answer.

**Brute Force Approach:**
Use a hash map to count the frequency of each element, then scan the map for the element whose count is `1`.

**Logic (Steps):**
1. Build a frequency dictionary by scanning `nums` once, incrementing each element's count.
2. Scan the dictionary's key-value pairs.
3. Return the key whose value equals `1`.

```csharp
public class SingleNumberIIBruteForce
{
    public static int SingleNumber(int[] nums)
    {
        var freq = new Dictionary<int, int>();

        foreach (int num in nums)
        {
            if (freq.ContainsKey(num))
                freq[num]++;
            else
                freq[num] = 1;
        }

        foreach (var kvp in freq)
        {
            if (kvp.Value == 1)
                return kvp.Key;
        }

        throw new ArgumentException("No single element found");
    }
}
```

Time Complexity: `O(n)` for building and scanning the hash map.
Space Complexity: `O(n)` in the worst case for the hash map (up to `n/3` distinct keys).

**Walkthrough:** For `nums = [2, 2, 3, 2]`: the frequency map becomes `{2: 3, 3: 1}`. Scanning it, the entry `3 → 1` has count `1`, so the function returns `3`, matching the expected output.

**Optimized Approach:**
For each of the 32 bit positions, count how many numbers in the array have that bit set. Since every number except one appears three times, the total count at each bit position is a multiple of 3, plus possibly 1 contributed by the unique element. So `(count at bit i) % 3` recovers whether the unique element has that bit set. Reconstruct the answer bit by bit.

**Logic (Steps):**
1. For each bit position `bit` from `0` to `31`, count how many numbers in `nums` have that bit set (`countAtBit`).
2. Since every triplicated number contributes a multiple of 3 to `countAtBit`, `countAtBit % 3` is non-zero exactly when the unique element has that bit set.
3. If `countAtBit % 3 != 0`, set bit `bit` in `answer` via `answer |= (1 << bit)`.
4. After all 32 bits are processed, `answer` is the fully reconstructed unique element.

```csharp
public class SingleNumberIIOptimized
{
    public static int SingleNumber(int[] nums)
    {
        int answer = 0;

        for (int bit = 0; bit < 32; bit++)
        {
            int countAtBit = 0;
            foreach (int num in nums)
            {
                if (((num >> bit) & 1) == 1)
                {
                    countAtBit++;
                }
            }

            if (countAtBit % 3 != 0)
            {
                answer |= (1 << bit);
            }
        }

        return answer;
    }
}
```

Time Complexity: `O(32 * n) = O(n)` — 32 fixed passes over the array, one per bit position.
Space Complexity: `O(1)` — only a few integer counters used.

**Walkthrough:** On `nums = [2, 2, 3, 2]` (binary: `2 = 010`, `3 = 011`): bit 0 count = 1 (`1 % 3 = 1 ≠ 0` → set); bit 1 count = 4 (`4 % 3 = 1 ≠ 0` → set); bits 2+ contribute nothing. Answer built as bit0 + bit1 = `011 = 3`, matching the expected output `3`.

---

## 5. Single Number III — Every Element Appears Twice Except Two

**Problem Statement:**
Given an array of integers where exactly two elements appear only once and every other element appears exactly twice, find the two elements that appear only once. Return them in any order.

**Example:**
- Input: `nums = [1, 2, 1, 3, 2, 5]`
- Output: `[3, 5]`
- Explanation: `1` and `2` each appear twice; `3` and `5` each appear once.

**Brute Force Approach:**
Use a hash map to count frequencies, then collect the two keys whose count equals `1`.

**Logic (Steps):**
1. Build a frequency dictionary over `nums`.
2. Scan the dictionary and collect every key whose value equals `1` into a result list.
3. Return the collected keys as an array (there will be exactly two).

```csharp
public class SingleNumberIIIBruteForce
{
    public static int[] SingleNumber(int[] nums)
    {
        var freq = new Dictionary<int, int>();

        foreach (int num in nums)
        {
            if (freq.ContainsKey(num))
                freq[num]++;
            else
                freq[num] = 1;
        }

        var result = new List<int>();
        foreach (var kvp in freq)
        {
            if (kvp.Value == 1)
                result.Add(kvp.Key);
        }

        return result.ToArray();
    }
}
```

Time Complexity: `O(n)` for building and scanning the hash map.
Space Complexity: `O(n)` for the hash map in the worst case.

**Walkthrough:** For `nums = [1, 2, 1, 3, 2, 5]`: the frequency map is `{1:2, 2:2, 3:1, 5:1}`. Scanning it collects keys `3` and `5` (both count `1`), returning `[3, 5]`, matching the expected output.

**Optimized Approach:**
XOR all numbers together. Since duplicated numbers cancel out (`x ^ x = 0`), the result is `a ^ b`, where `a` and `b` are the two unique numbers. Because `a != b`, `a ^ b` is non-zero, so it has at least one set bit. Pick any set bit (commonly the lowest set bit, via `xorAll & (-xorAll)`) — this bit must differ between `a` and `b` (since XOR-ing them produced a 1 there). Use this bit to partition all numbers into two groups: those with the bit set, and those without. XOR each group independently; since `a` and `b` fall into different groups, and every duplicate pair falls into the same group (duplicates are identical numbers, so they share the same bit value and always land together), each group's XOR isolates exactly one of `a` or `b`.

**Logic (Steps):**
1. XOR every number in `nums` together into `xorAll`; duplicates cancel (`x ^ x = 0`), leaving `xorAll = a ^ b`.
2. Isolate the lowest set bit of `xorAll` via `diffBit = xorAll & (-xorAll)` — this bit is guaranteed to differ between `a` and `b`.
3. Partition `nums` into two groups based on whether each number has `diffBit` set, XOR-ing each group into accumulators `a` and `b` respectively.
4. Because duplicate pairs always share the same bits, they always land in the same group and cancel out there, so each group's final XOR isolates exactly one unique number.

```csharp
public class SingleNumberIIIOptimized
{
    public static int[] SingleNumber(int[] nums)
    {
        int xorAll = 0;
        foreach (int num in nums)
        {
            xorAll ^= num;
        }

        // Isolate the lowest set bit of xorAll; this bit differs between a and b
        int diffBit = xorAll & (-xorAll);

        int a = 0, b = 0;
        foreach (int num in nums)
        {
            if ((num & diffBit) != 0)
            {
                a ^= num;
            }
            else
            {
                b ^= num;
            }
        }

        return new int[] { a, b };
    }
}
```

Time Complexity: `O(n)` — two linear passes over the array.
Space Complexity: `O(1)` — only a few integer variables used.

**Walkthrough:** For `nums = [1, 2, 1, 3, 2, 5]`: `xorAll = 1^2^1^3^2^5 = 3^5 = 6 (110)`. `diffBit = 6 & (-6) = 2 (010)`, i.e. bit 1. Partitioning by bit 1: group A (bit1 set) = `[2, 3, 2]` → XOR = `3`; group B (bit1 unset) = `[1, 1, 5]` → XOR = `5`. Result `a = 3, b = 5`, matching the expected output `[3, 5]`.

---

## 6. Sum of XOR of All Subsets of an Array

**Problem Statement:**
Given an array of `n` non-negative integers, consider all `2^n` subsets (including the empty subset, whose XOR is `0`). For each subset compute the XOR of its elements, then sum all those XOR values together. Return the total sum.

**Example:**
- Input: `arr = [1, 3]`
- Output: `8`
- Explanation: Subsets and their XORs: `{} → 0`, `{1} → 1`, `{3} → 3`, `{1,3} → 1^3 = 2`. Sum = `0 + 1 + 3 + 2 = 6`... 

  Let's verify precisely: `1 = 01`, `3 = 11`. `{}=0, {1}=1, {3}=3, {1,3}=1^3=2`. Sum = `0+1+3+2 = 6`. (Using `arr = [1, 3]`, the correct output is `6`.)

**Brute Force Approach:**
Generate every subset (using bitmasking over `0` to `2^n - 1`), compute the XOR of each subset's elements, and accumulate the sum of all these XOR values.

**Logic (Steps):**
1. Compute `totalMasks = 1 << n`, the number of subsets.
2. For each `mask` from `0` to `totalMasks - 1`, compute `subsetXor` by XOR-ing `arr[j]` for every bit `j` that is set in `mask`.
3. Add `subsetXor` to a running `totalSum`.
4. After all masks are processed, `totalSum` is the sum of XORs of every subset.

```csharp
public class SumXorSubsetsBruteForce
{
    public static int SumOfXorOfAllSubsets(int[] arr)
    {
        int n = arr.Length;
        int totalMasks = 1 << n;
        int totalSum = 0;

        for (int mask = 0; mask < totalMasks; mask++)
        {
            int subsetXor = 0;
            for (int j = 0; j < n; j++)
            {
                if ((mask & (1 << j)) != 0)
                {
                    subsetXor ^= arr[j];
                }
            }
            totalSum += subsetXor;
        }

        return totalSum;
    }
}
```

Time Complexity: `O(2^n * n)` — `2^n` subsets, each taking `O(n)` to compute its XOR.
Space Complexity: `O(1)` extra space (excluding input).

**Walkthrough:** For `arr = [1, 3]`: mask `00 → subsetXor=0`, `01 → 1`, `10 → 3`, `11 → 1^3=2`. Sum = `0+1+3+2 = 6`, matching the expected output `6`.

**Optimized Approach:**
Compute `orAll`, the bitwise OR of every element in the array (equivalently, this is the same as the "OR of all elements" which captures every bit that is set in at least one element). Key insight: for any bit position that is set in at least one array element, exactly half of all `2^n` subsets will have an odd number of elements with that bit set (i.e., the XOR of the subset has that bit set), and the other half will have it unset. This is because you can pair up subsets by toggling the inclusion of just one element that has that bit set — this pairing flips the parity of that bit in exactly one subset of each pair, so set/unset occurrences split evenly, giving exactly `2^(n-1)` subsets where that bit is set in the subset-XOR. Therefore the total contribution of that bit across all subsets is `bitValue * 2^(n-1)`. Summing this over every bit that appears in `orAll` gives: `answer = orAll * 2^(n-1)`.

**Logic (Steps):**
1. Compute `orAll`, the bitwise OR of every element in `arr` — this captures every bit that appears in at least one element.
2. Compute `subsetsHalf = 2^(n-1)`, the number of subsets in which any given "present" bit ends up set in the subset's XOR.
3. Multiply `orAll * subsetsHalf` — each set bit in `orAll` contributes its value across exactly half of all `2^n` subsets, so this single multiplication gives the total sum directly, with no per-subset iteration.

```csharp
public class SumXorSubsetsOptimized
{
    public static long SumOfXorOfAllSubsets(int[] arr)
    {
        int n = arr.Length;

        int orAll = 0;
        foreach (int num in arr)
        {
            orAll |= num;
        }

        // Each bit present in orAll contributes to exactly half (2^(n-1)) of all subsets
        long subsetsHalf = 1L << (n - 1); // 2^(n-1)
        return (long)orAll * subsetsHalf;
    }
}
```

Time Complexity: `O(n)` — a single pass to compute the OR of all elements.
Space Complexity: `O(1)` — only a couple of integer/long variables used.

**Walkthrough:** For `arr = [1, 3]` (`n = 2`): `orAll = 1 | 3 = 011 = 3`, `subsetsHalf = 2^(n-1) = 2`, so `answer = 3 * 2 = 6`, matching the brute-force sum computed earlier. Bit by bit: bit 0 is set in `orAll` and is set in 2 of the 4 subset-XORs (`{1}` and `{3}`), contributing `1*2=2`; bit 1 is set in `orAll` and is set in 2 of the 4 subset-XORs (`{3}` and `{1,3}`), contributing `2*2=4`; total `2+4=6`, confirming the formula.
