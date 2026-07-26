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

**Optimized Approach:**
Since each of the `n` elements has exactly two states (present / absent), a subset can be represented by an `n`-bit number (mask) ranging from `0` to `2^n - 1`. For a given mask, bit `j` (from the right) being `1` means "include `arr[j]`" and `0` means "exclude `arr[j]`". Iterating the mask from `0` to `2^n - 1` and checking each bit generates every subset exactly once, with no recursion needed.

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

**Explanation:**
For `arr = [1, 2, 3]` (n = 3), masks range from `000` to `111`:
- `mask = 000 (0)` → no bits set → `[]`
- `mask = 001 (1)` → bit 0 set → `[1]`
- `mask = 010 (2)` → bit 1 set → `[2]`
- `mask = 011 (3)` → bits 0,1 set → `[1, 2]`
- `mask = 100 (4)` → bit 2 set → `[3]`
- `mask = 101 (5)` → bits 0,2 set → `[1, 3]`
- `mask = 110 (6)` → bits 1,2 set → `[2, 3]`
- `mask = 111 (7)` → all bits set → `[1, 2, 3]`

Each mask's binary representation directly encodes one unique subset — this bijection between `{0, ..., 2^n-1}` and the power set is why the technique works.

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

**Optimized Approach:**
Define `xorFrom1ToN(n)` = XOR of all integers from `1` to `n`. This has a well-known closed form based on `n % 4`:
- `n % 4 == 0` → answer is `n`
- `n % 4 == 1` → answer is `1`
- `n % 4 == 2` → answer is `n + 1`
- `n % 4 == 3` → answer is `0`

Then, using the prefix-XOR idea (similar to prefix sums), the XOR of `[L, R]` equals `xorFrom1ToN(R) XOR xorFrom1ToN(L - 1)`, because XOR-ing `1..(L-1)` twice cancels out the prefix that shouldn't be included.

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

**Explanation:**

Dry run of the `n % 4` pattern for `xorFrom1ToN`:
- `n = 1`: `1` → `1 % 4 = 1` → return `1`. Matches directly.
- `n = 2`: `1 ^ 2 = 3` → `2 % 4 = 2` → return `n + 1 = 3`. Matches.
- `n = 3`: `1 ^ 2 ^ 3 = 0` → `3 % 4 = 3` → return `0`. Matches.
- `n = 4`: `1 ^ 2 ^ 3 ^ 4 = 4` → `4 % 4 = 0` → return `n = 4`. Matches.
- `n = 5`: previous XOR(1..4) = 4, then `4 ^ 5 = 1` → `5 % 4 = 1` → return `1`. Matches.

The pattern repeats every 4 numbers because XOR-ing four consecutive integers of the form `4k, 4k+1, 4k+2, 4k+3` always cancels down to `0` (their bits cancel in pairs), so the cumulative XOR only depends on `n mod 4`, not on `n` itself (except when returning `n` or `n+1`).

Dry run of the full range formula for `L = 3, R = 9`:
- `xorFrom1ToN(9)`: `9 % 4 = 1` → returns `1`.
- `xorFrom1ToN(L - 1) = xorFrom1ToN(2)`: `2 % 4 = 2` → returns `2 + 1 = 3`.
- Result: `1 ^ 3 = 2`. This matches the brute-force answer of `2`.

Why this works: `xorFrom1ToN(R)` = XOR of `1..R`. `xorFrom1ToN(L-1)` = XOR of `1..(L-1)`. XOR-ing these two together cancels every number from `1` to `L-1` (each appears twice, and `x ^ x = 0`), leaving only the XOR of `L..R`.

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

**Optimized Approach:**
Instead of subtracting the divisor one unit at a time, subtract the largest possible multiple of the divisor at each step using bit-shift doubling. For the current remaining dividend, start with `pivot = divisor` and `multiple = 1`, and keep doubling both (`pivot <<= 1`, `multiple <<= 1`) as long as `pivot` still fits within the remaining dividend. Once doubling would overshoot, subtract `pivot` from the dividend, add `multiple` to the quotient, and restart the doubling process from `divisor` again. This is effectively binary long division and runs in `O(log^2 n)`.

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

**Explanation:**
For `dividend = 22, divisor = 3`:
- Outer iteration 1: `absDividend = 22`. Start `pivot = 3, multiple = 1`.
  - `pivot << 1 = 6 <= 22` → `pivot = 6, multiple = 2`
  - `pivot << 1 = 12 <= 22` → `pivot = 12, multiple = 4`
  - `pivot << 1 = 24 > 22` → stop doubling
  - Subtract: `absDividend = 22 - 12 = 10`, `quotient = 0 + 4 = 4`
- Outer iteration 2: `absDividend = 10`. Start `pivot = 3, multiple = 1`.
  - `pivot << 1 = 6 <= 10` → `pivot = 6, multiple = 2`
  - `pivot << 1 = 12 > 10` → stop doubling
  - Subtract: `absDividend = 10 - 6 = 4`, `quotient = 4 + 2 = 6`
- Outer iteration 3: `absDividend = 4`. Start `pivot = 3, multiple = 1`.
  - `pivot << 1 = 6 > 4` → stop doubling
  - Subtract: `absDividend = 4 - 3 = 1`, `quotient = 6 + 1 = 7`
- Now `absDividend = 1 < 3 = absDivisor` → loop ends.
- Final quotient = `7`, matching the expected output.

Each outer iteration finds the largest power-of-two multiple of the divisor that fits, subtracting a big chunk at once instead of one unit at a time — this is what brings the complexity down from linear to logarithmic.

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

**Optimized Approach:**
For each of the 32 bit positions, count how many numbers in the array have that bit set. Since every number except one appears three times, the total count at each bit position is a multiple of 3, plus possibly 1 contributed by the unique element. So `(count at bit i) % 3` recovers whether the unique element has that bit set. Reconstruct the answer bit by bit.

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

**Explanation:**
Dry run on `nums = [2, 2, 3, 2]` (binary: `2 = 010`, `3 = 011`):
- Bit 0 (value 1): `2` has bit0 = 0 (×3 times) → 0 ones; `3` has bit0 = 1 → 1 one. Total count at bit 0 = `1`. `1 % 3 = 1 ≠ 0` → set bit 0 in answer.
- Bit 1 (value 2): `2` has bit1 = 1 (×3 times) → 3 ones; `3` has bit1 = 1 → 1 one. Total count at bit 1 = `4`. `4 % 3 = 1 ≠ 0` → set bit 1 in answer.
- Bit 2 and above: all zero contributions, `0 % 3 = 0` → bits stay 0.
- Answer so far: bit0 set, bit1 set → binary `011 = 3`. This matches the expected output of `3`.

The key insight: if a bit is set in the triplicated numbers, it contributes a multiple of 3 to that bit's total count across the whole array (since each such number appears exactly 3 times). Only the unique element can break the "multiple of 3" pattern, so `count % 3` isolates exactly its bits.

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

**Optimized Approach:**
XOR all numbers together. Since duplicated numbers cancel out (`x ^ x = 0`), the result is `a ^ b`, where `a` and `b` are the two unique numbers. Because `a != b`, `a ^ b` is non-zero, so it has at least one set bit. Pick any set bit (commonly the lowest set bit, via `xorAll & (-xorAll)`) — this bit must differ between `a` and `b` (since XOR-ing them produced a 1 there). Use this bit to partition all numbers into two groups: those with the bit set, and those without. XOR each group independently; since `a` and `b` fall into different groups, and every duplicate pair falls into the same group (duplicates are identical numbers, so they share the same bit value and always land together), each group's XOR isolates exactly one of `a` or `b`.

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

**Explanation:**
Dry run on `nums = [1, 2, 1, 3, 2, 5]`:

1. XOR all elements: `1 ^ 2 ^ 1 ^ 3 ^ 2 ^ 5`. Rearranging (XOR is commutative/associative): `(1^1) ^ (2^2) ^ 3 ^ 5 = 0 ^ 0 ^ 3 ^ 5 = 3 ^ 5`.
   - `3 = 011`, `5 = 101` → `3 ^ 5 = 110 = 6`. So `xorAll = 6`.
2. Isolate the lowest set bit: `-6` in two's complement is `...11111010`, and `6 & (-6)`:
   - `6 = 00000110`
   - `-6 = 11111010`
   - AND = `00000010 = 2`. So `diffBit = 2` (binary `010`, i.e. bit position 1).
   - This works because `x & (-x)` always isolates the lowest set bit of `x`.
3. Partition the array by whether bit 1 (`010`) is set:
   - `1 = 001` → bit1 = 0 → group B
   - `2 = 010` → bit1 = 1 → group A
   - `1 = 001` → bit1 = 0 → group B
   - `3 = 011` → bit1 = 1 → group A
   - `2 = 010` → bit1 = 1 → group A
   - `5 = 101` → bit1 = 0 → group B
4. Group A = `[2, 3, 2]` → XOR = `2 ^ 3 ^ 2 = 3`. Group B = `[1, 1, 5]` → XOR = `1 ^ 1 ^ 5 = 5`.
5. Result: `a = 3, b = 5`, matching the expected output `[3, 5]`.

Why it works: `3` and `5` differ at bit 1 (`3` has it unset, `5` has it set — wait, checking again: `3 = 011` has bit1 = 1, `5 = 101` has bit1 = 0), so they land in different groups by construction (since `diffBit` was derived from `3 ^ 5`, guaranteeing they differ there). All duplicate pairs (`1,1` and `2,2`) always land in the same group as each other because identical numbers have identical bits, so they cancel out via XOR within their group, leaving each group's XOR equal to exactly one unique number.

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

**Optimized Approach:**
Compute `orAll`, the bitwise OR of every element in the array (equivalently, this is the same as the "OR of all elements" which captures every bit that is set in at least one element). Key insight: for any bit position that is set in at least one array element, exactly half of all `2^n` subsets will have an odd number of elements with that bit set (i.e., the XOR of the subset has that bit set), and the other half will have it unset. This is because you can pair up subsets by toggling the inclusion of just one element that has that bit set — this pairing flips the parity of that bit in exactly one subset of each pair, so set/unset occurrences split evenly, giving exactly `2^(n-1)` subsets where that bit is set in the subset-XOR. Therefore the total contribution of that bit across all subsets is `bitValue * 2^(n-1)`. Summing this over every bit that appears in `orAll` gives: `answer = orAll * 2^(n-1)`.

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

**Explanation:**
Dry run on `arr = [1, 3]` (`n = 2`):
- `orAll = 1 | 3 = 01 | 11 = 11 = 3`
- `subsetsHalf = 2^(n-1) = 2^1 = 2`
- `answer = orAll * subsetsHalf = 3 * 2 = 6`

This matches the brute-force sum of `6` computed above.

Why the formula works, bit by bit for `arr = [1, 3]`:
- Bit 0 (value 1): set in `1` (`01`) and in `3` (`11`) — so bit 0 is set in `orAll`. Checking all 4 subsets' XOR at bit 0: `{}=0→bit0=0`, `{1}=1→bit0=1`, `{3}=3→bit0=1`, `{1,3}=2→bit0=0`. Bit 0 is set in exactly `2` out of `4` subsets, i.e. `2^(n-1) = 2`. Contribution to total sum = `1 * 2 = 2`.
- Bit 1 (value 2): set in `3` (`11`) but not in `1` (`01`) — still set in `orAll` since at least one element has it. Checking subsets at bit 1: `{}=0→bit1=0`, `{1}=1→bit1=0`, `{3}=3→bit1=1`, `{1,3}=2→bit1=1`. Bit 1 is set in exactly `2` out of `4` subsets = `2^(n-1) = 2`. Contribution = `2 * 2 = 4`.
- Total = `2 + 4 = 6`, matching both the brute-force result and the formula's output.

If a bit were never set in any array element, it can never be set in any subset's XOR either, so it contributes `0` — this is exactly why only bits present in `orAll` matter, and each such bit always contributes across exactly half of all subsets regardless of how many elements carry it.
