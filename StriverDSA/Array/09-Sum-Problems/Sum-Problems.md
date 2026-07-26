# Array — Sum Problems

## 1. Two Sum Problem

### 1. Two Sum Problem

**Problem Statement:**
Given an array of integers `nums` and an integer `target`, determine whether there exists a pair of elements whose sum equals `target`. Two common variants are asked:
1. **Existence check** — just return `true`/`false` (or the pair of values) if such a pair exists.
2. **Return indices** — return the indices `(i, j)` of the two numbers such that `nums[i] + nums[j] == target`. Each input is assumed to have exactly one valid answer, and the same element cannot be used twice.

**Example:**
- Input: `nums = [2, 7, 11, 15]`, `target = 9`
- Output: `[0, 1]` (indices variant) or `true` / `(2, 7)` (existence variant)
- Explanation: `nums[0] + nums[1] = 2 + 7 = 9`, which equals the target, so index pair `(0, 1)` (or value pair `(2, 7)`) is the answer.

**Brute Force Approach:**
Use two nested loops to check every possible pair `(i, j)` with `i < j` and test whether `nums[i] + nums[j] == target`. As soon as a matching pair is found, return it. This checks all O(n²) pairs in the worst case.

```csharp
public static int[] TwoSumBrute(int[] nums, int target)
{
    int n = nums.Length;
    for (int i = 0; i < n; i++)
    {
        for (int j = i + 1; j < n; j++)
        {
            if (nums[i] + nums[j] == target)
            {
                return new int[] { i, j };
            }
        }
    }
    return new int[] { -1, -1 }; // no valid pair found
}
```

Time Complexity: **O(n²)** — for every element `i`, we scan all remaining elements `j` to the right of it, giving roughly n*(n-1)/2 comparisons.
Space Complexity: **O(1)** — no extra data structure is used besides the output array.

**Optimized Approach:**
Use a hash map (`Dictionary<int,int>`) to store each visited number and its index. For every element, compute the `complement = target - nums[i]`. If the complement already exists in the map, we have found our pair immediately; otherwise, insert the current number and its index into the map and continue. This turns the lookup for the "other half" of the pair from an O(n) scan into an O(1) average-case hash lookup.

```csharp
public static int[] TwoSumOptimized(int[] nums, int target)
{
    Dictionary<int, int> seen = new Dictionary<int, int>(); // value -> index
    for (int i = 0; i < nums.Length; i++)
    {
        int complement = target - nums[i];
        if (seen.ContainsKey(complement))
        {
            return new int[] { seen[complement], i };
        }
        // Store current value with its index (avoids using the same element twice
        // because we check for the complement BEFORE inserting the current value).
        seen[nums[i]] = i;
    }
    return new int[] { -1, -1 }; // no valid pair found
}

// Existence-check variant (returns bool + the pair of values, if needed)
public static bool TwoSumExists(int[] nums, int target, out (int, int) pair)
{
    HashSet<int> seen = new HashSet<int>();
    foreach (int num in nums)
    {
        int complement = target - num;
        if (seen.Contains(complement))
        {
            pair = (complement, num);
            return true;
        }
        seen.Add(num);
    }
    pair = (0, 0);
    return false;
}
```

Time Complexity: **O(n)** — a single pass through the array; each `Dictionary`/`HashSet` insert and lookup is O(1) on average.
Space Complexity: **O(n)** — in the worst case, we store almost all `n` elements in the hash map/set before finding a match.

**Explanation:**
Dry run of the hashmap approach on `nums = [2, 7, 11, 15]`, `target = 9`:
- `i = 0`, `nums[0] = 2`. `complement = 9 - 2 = 7`. `seen` is empty, so `7` is not found. Insert `seen[2] = 0`. `seen = {2: 0}`.
- `i = 1`, `nums[1] = 7`. `complement = 9 - 7 = 2`. `seen` contains key `2` (mapped to index `0`). Match found! Return `[seen[2], 1] = [0, 1]`.

The algorithm never needs to look at `11` or `15` because it found the answer after just two elements — this is the core benefit over brute force: instead of re-scanning the array for the complement, we ask an O(1) hash map "have I seen this value before?" as we go.

---

## 2. Best Time to Buy and Sell Stock

### 2. Best Time to Buy and Sell Stock

**Problem Statement:**
Given an array `prices` where `prices[i]` is the price of a stock on day `i`, find the maximum profit achievable by choosing a single day to buy and a later day to sell. Only one transaction is allowed (buy once, sell once), and you must buy before you sell. If no profit is possible, return `0`.

**Example:**
- Input: `prices = [7, 1, 5, 3, 6, 4]`
- Output: `5`
- Explanation: Buy on day index `1` (price = `1`) and sell on day index `4` (price = `6`), profit = `6 - 1 = 5`. This is the maximum possible profit; buying on day `0` and selling later never beats this.

**Brute Force Approach:**
For every pair of days `(i, j)` with `i < j` (buy day before sell day), compute the profit `prices[j] - prices[i]` and track the maximum profit seen across all such pairs.

```csharp
public static int MaxProfitBrute(int[] prices)
{
    int n = prices.Length;
    int maxProfit = 0;
    for (int i = 0; i < n; i++)
    {
        for (int j = i + 1; j < n; j++)
        {
            int profit = prices[j] - prices[i];
            if (profit > maxProfit)
            {
                maxProfit = profit;
            }
        }
    }
    return maxProfit;
}
```

Time Complexity: **O(n²)** — every pair `(i, j)` with `i < j` is examined, roughly n*(n-1)/2 combinations.
Space Complexity: **O(1)** — only a couple of scalar variables are used.

**Optimized Approach:**
Make a single pass through the array while tracking the minimum price seen so far (`minPriceSoFar`). At each day `i`, the best possible profit if we sold today is `prices[i] - minPriceSoFar`. Update the running maximum profit, then update `minPriceSoFar` if the current price is lower than what we've seen before. This way we never need to look backward — we always know the cheapest "buy" opportunity up to the current day.

```csharp
public static int MaxProfitOptimized(int[] prices)
{
    if (prices.Length == 0) return 0;

    int minPriceSoFar = prices[0];
    int maxProfit = 0;

    for (int i = 1; i < prices.Length; i++)
    {
        int profitIfSoldToday = prices[i] - minPriceSoFar;
        maxProfit = Math.Max(maxProfit, profitIfSoldToday);
        minPriceSoFar = Math.Min(minPriceSoFar, prices[i]);
    }

    return maxProfit;
}
```

Time Complexity: **O(n)** — a single linear pass through the prices array.
Space Complexity: **O(1)** — only `minPriceSoFar` and `maxProfit` are tracked.

**Explanation:**
Dry run on `prices = [7, 1, 5, 3, 6, 4]`:
- Start: `minPriceSoFar = 7`, `maxProfit = 0`.
- `i = 1`, price = `1`: `profitIfSoldToday = 1 - 7 = -6` → `maxProfit` stays `0`. Update `minPriceSoFar = min(7, 1) = 1`.
- `i = 2`, price = `5`: `profitIfSoldToday = 5 - 1 = 4` → `maxProfit = max(0, 4) = 4`. `minPriceSoFar` stays `1`.
- `i = 3`, price = `3`: `profitIfSoldToday = 3 - 1 = 2` → `maxProfit` stays `4`. `minPriceSoFar` stays `1`.
- `i = 4`, price = `6`: `profitIfSoldToday = 6 - 1 = 5` → `maxProfit = max(4, 5) = 5`. `minPriceSoFar` stays `1`.
- `i = 5`, price = `4`: `profitIfSoldToday = 4 - 1 = 3` → `maxProfit` stays `5`.
- Final answer: `maxProfit = 5`, matching the expected output.

---

## 3. 3Sum Problem

### 3. 3Sum Problem

**Problem Statement:**
Given an array `nums`, find all unique triplets `[nums[i], nums[j], nums[k]]` (with distinct indices `i`, `j`, `k`) such that `nums[i] + nums[j] + nums[k] == 0`. The result must not contain duplicate triplets (the same combination of values, regardless of order, should appear only once).

**Example:**
- Input: `nums = [-1, 0, 1, 2, -1, -4]`
- Output: `[[-1, -1, 2], [-1, 0, 1]]`
- Explanation: The triplets `(-1, -1, 2)` and `(-1, 0, 1)` each sum to `0`. Although `-1` appears twice in the array and could form other index combinations, only these two *distinct value combinations* are valid — duplicate triplets like a second `(-1, 0, 1)` are excluded.

**Brute Force Approach:**
Use three nested loops to try every combination of three distinct indices `(i, j, k)`, check if they sum to zero, and if so add the sorted triplet to a set (to filter out duplicate value-combinations) before finally converting to a list.

```csharp
public static List<List<int>> ThreeSumBrute(int[] nums)
{
    int n = nums.Length;
    // Use a HashSet of a comma-joined sorted string (or tuple) to dedupe triplets.
    HashSet<string> seenTriplets = new HashSet<string>();
    List<List<int>> result = new List<List<int>>();

    for (int i = 0; i < n; i++)
    {
        for (int j = i + 1; j < n; j++)
        {
            for (int k = j + 1; k < n; k++)
            {
                if (nums[i] + nums[j] + nums[k] == 0)
                {
                    List<int> triplet = new List<int> { nums[i], nums[j], nums[k] };
                    triplet.Sort();
                    string key = string.Join(",", triplet);
                    if (!seenTriplets.Contains(key))
                    {
                        seenTriplets.Add(key);
                        result.Add(triplet);
                    }
                }
            }
        }
    }
    return result;
}
```

Time Complexity: **O(n³)** — three nested loops over all index combinations `i < j < k`.
Space Complexity: **O(n)** to **O(n²)** for the set used to store/deduplicate triplet keys (in the worst case, plus the output list itself), excluding the output.

**Optimized Approach:**
Sort the array first (`Array.Sort`), which is what enables the two-pointer technique. Fix the first element with an outer loop index `i`. Then use two pointers, `left = i + 1` and `right = n - 1`, moving inward: if `nums[i] + nums[left] + nums[right] < 0`, increase `left` (need a bigger sum); if `> 0`, decrease `right` (need a smaller sum); if `== 0`, record the triplet and move both pointers inward, skipping over duplicate values at each of the three positions to avoid duplicate triplets in the output.

```csharp
public static List<List<int>> ThreeSumOptimized(int[] nums)
{
    Array.Sort(nums);
    int n = nums.Length;
    List<List<int>> result = new List<List<int>>();

    for (int i = 0; i < n; i++)
    {
        // Skip duplicate values for the first pointer.
        if (i > 0 && nums[i] == nums[i - 1]) continue;

        int left = i + 1;
        int right = n - 1;

        while (left < right)
        {
            long sum = (long)nums[i] + nums[left] + nums[right];

            if (sum < 0)
            {
                left++;
            }
            else if (sum > 0)
            {
                right--;
            }
            else
            {
                result.Add(new List<int> { nums[i], nums[left], nums[right] });
                left++;
                right--;

                // Skip duplicates for the second and third pointers.
                while (left < right && nums[left] == nums[left - 1]) left++;
                while (left < right && nums[right] == nums[right + 1]) right--;
            }
        }
    }

    return result;
}
```

Time Complexity: **O(n²)** — the outer loop runs O(n) times, and for each fixed `i`, the two-pointer scan across the remaining subarray takes O(n), giving O(n) * O(n) = O(n²). Sorting itself is O(n log n), dominated by the O(n²) main loop.
Space Complexity: **O(1)** extra (excluding the output list) if sorting is done in place; O(n log n)–O(n) may be used internally by the sort, and the output itself takes O(k) for k triplets found.

**Explanation:**
Dry run on the sorted array. `nums = [-1, 0, 1, 2, -1, -4]` sorted becomes `[-4, -1, -1, 0, 1, 2]`.

- `i = 0`, `nums[i] = -4`: `left = 1` (`-1`), `right = 5` (`2`). Sum `= -4 + -1 + 2 = -3 < 0` → `left++` → `left = 2` (`-1`). Sum `= -4 + -1 + 2 = -3 < 0` → `left++` → `left = 3` (`0`). Sum `= -4 + 0 + 2 = -2 < 0` → `left++` → `left = 4` (`1`). Sum `= -4 + 1 + 2 = -1 < 0` → `left++` → `left = 5 = right`, loop ends. No triplet for `i = 0`.
- `i = 1`, `nums[i] = -1`: `left = 2` (`-1`), `right = 5` (`2`). Sum `= -1 + -1 + 2 = 0` → **triplet found: `[-1, -1, 2]`**. Move `left++` → `3`, `right--` → `4`. Check duplicate skip: `nums[left] = nums[3] = 0`, `nums[left-1] = nums[2] = -1` → not equal, no skip. `nums[right] = nums[4] = 1`, `nums[right+1] = nums[5] = 2` → not equal, no skip.
  Continue: `left = 3` (`0`), `right = 4` (`1`). Sum `= -1 + 0 + 1 = 0` → **triplet found: `[-1, 0, 1]`**. Move `left++` → `4`, `right--` → `3`. Now `left >= right`, loop ends.
- `i = 2`, `nums[i] = -1`: since `nums[2] == nums[1] == -1`, this is a **duplicate of the first pointer position**, so it is **skipped** via `if (i > 0 && nums[i] == nums[i-1]) continue;` — this prevents re-finding the same triplets again.
- `i = 3`, `nums[i] = 0`: `left = 4` (`1`), `right = 5` (`2`). Sum `= 0 + 1 + 2 = 3 > 0` → `right--` → `right = 4 = left`, loop ends. No triplet.
- `i = 4, 5`: `left >= right` immediately since only one or zero elements remain to the right, loop bodies don't execute.

Final result: `[[-1, -1, 2], [-1, 0, 1]]`, matching the expected output. The key duplicate-avoidance mechanism is: **(a)** skip the outer index `i` if it equals the previous `i`'s value, and **(b)** after recording a valid triplet, advance `left`/`right` past any run of equal values before continuing the search.

---

## 4. 4Sum Problem

### 4. 4Sum Problem

**Problem Statement:**
Given an array `nums` and a target value `target`, find all unique quadruplets `[nums[a], nums[b], nums[c], nums[d]]` (four distinct indices) such that `nums[a] + nums[b] + nums[c] + nums[d] == target`. The result must not contain duplicate quadruplets.

**Example:**
- Input: `nums = [1, 0, -1, 0, -2, 2]`, `target = 0`
- Output: `[[-2, -1, 1, 2], [-2, 0, 0, 2], [-1, 0, 0, 1]]`
- Explanation: Each of these four-number combinations sums to `0`, and each is a distinct combination of values — no duplicates are included even though values like `0` repeat in the input array.

**Brute Force Approach:**
Use four nested loops over all combinations of four distinct indices `(a, b, c, d)`, check if their sum equals the target, and store the sorted quadruplet in a set to filter duplicates.

```csharp
public static List<List<int>> FourSumBrute(int[] nums, int target)
{
    int n = nums.Length;
    HashSet<string> seenQuadruplets = new HashSet<string>();
    List<List<int>> result = new List<List<int>>();

    for (int a = 0; a < n; a++)
    {
        for (int b = a + 1; b < n; b++)
        {
            for (int c = b + 1; c < n; c++)
            {
                for (int d = c + 1; d < n; d++)
                {
                    long sum = (long)nums[a] + nums[b] + nums[c] + nums[d];
                    if (sum == target)
                    {
                        List<int> quad = new List<int> { nums[a], nums[b], nums[c], nums[d] };
                        quad.Sort();
                        string key = string.Join(",", quad);
                        if (!seenQuadruplets.Contains(key))
                        {
                            seenQuadruplets.Add(key);
                            result.Add(quad);
                        }
                    }
                }
            }
        }
    }
    return result;
}
```

Time Complexity: **O(n⁴)** — four nested loops over all index combinations `a < b < c < d`.
Space Complexity: **O(n) to O(n³)** in the worst case for the deduplication set (excluding output).

**Optimized Approach:**
Sort the array (`Array.Sort`). Fix the first two elements with nested outer loops `i` and `j` (skipping duplicates at each level), then apply the same two-pointer technique used in 3Sum on the remaining subarray using `left = j + 1` and `right = n - 1`. This reduces one loop level via two pointers, bringing total complexity down from O(n⁴) to O(n³). Use `long` for sum accumulation to avoid integer overflow.

```csharp
public static List<List<int>> FourSumOptimized(int[] nums, int target)
{
    Array.Sort(nums);
    int n = nums.Length;
    List<List<int>> result = new List<List<int>>();

    for (int i = 0; i < n; i++)
    {
        // Skip duplicate values for the 1st pointer.
        if (i > 0 && nums[i] == nums[i - 1]) continue;

        for (int j = i + 1; j < n; j++)
        {
            // Skip duplicate values for the 2nd pointer.
            if (j > i + 1 && nums[j] == nums[j - 1]) continue;

            int left = j + 1;
            int right = n - 1;

            while (left < right)
            {
                long sum = (long)nums[i] + nums[j] + nums[left] + nums[right];

                if (sum < target)
                {
                    left++;
                }
                else if (sum > target)
                {
                    right--;
                }
                else
                {
                    result.Add(new List<int> { nums[i], nums[j], nums[left], nums[right] });
                    left++;
                    right--;

                    // Skip duplicates for the 3rd and 4th pointers.
                    while (left < right && nums[left] == nums[left - 1]) left++;
                    while (left < right && nums[right] == nums[right + 1]) right--;
                }
            }
        }
    }

    return result;
}
```

Time Complexity: **O(n³)** — two nested outer loops (`i`, `j`) each O(n), combined with an inner O(n) two-pointer scan: O(n) * O(n) * O(n) = O(n³). This is a major improvement over the brute force O(n⁴). Sorting is O(n log n), dominated by the O(n³) main loops.
Space Complexity: **O(1)** extra (excluding output and the sort's internal usage), since only pointer variables are used; output takes O(k) for k quadruplets found.

**Explanation:**
Dry run on sorted array. `nums = [1, 0, -1, 0, -2, 2]` sorted becomes `[-2, -1, 0, 0, 1, 2]`, `target = 0`.

- `i = 0`, `nums[i] = -2`:
  - `j = 1`, `nums[j] = -1`: `left = 2` (`0`), `right = 5` (`2`). Sum `= -2 + -1 + 0 + 2 = -1 < 0` → `left++` → `left = 3` (`0`). Sum `= -2 + -1 + 0 + 2 = -1 < 0` → `left++` → `left = 4 = right - 1`... continue: `left = 4` (`1`), `right = 5` (`2`). Sum `= -2 + -1 + 1 + 2 = 0` → **quadruplet found: `[-2, -1, 1, 2]`**. `left++` → `5`, `right--` → `4`, loop ends (`left >= right`).
  - `j = 2`, `nums[j] = 0`: `left = 3` (`0`), `right = 5` (`2`). Sum `= -2 + 0 + 0 + 2 = 0` → **quadruplet found: `[-2, 0, 0, 2]`**. `left++` → `4`, `right--` → `4`, loop ends.
  - `j = 3`, `nums[j] = 0`: since `nums[3] == nums[2] == 0` and `j > i+1`, this is **skipped** (duplicate 2nd-pointer value).
- `i = 1`, `nums[i] = -1`:
  - `j = 2`, `nums[j] = 0`: `left = 3` (`0`), `right = 5` (`2`). Sum `= -1 + 0 + 0 + 2 = 1 > 0` → `right--` → `right = 4` (`1`). Sum `= -1 + 0 + 0 + 1 = 0` → **quadruplet found: `[-1, 0, 0, 1]`**. `left++` → `4`, `right--` → `3`, loop ends.
  - `j = 3`, `nums[j] = 0`: skipped, duplicate of `nums[2]`.
- `i = 2`, `nums[i] = 0`: proceeds similarly but yields no new quadruplets (any valid combination would already have been captured at a smaller `i`, and remaining pointer ranges shrink too much to produce new sums).
- `i = 3, 4, 5`: not enough remaining elements to the right for two pointers plus `j`, so no further quadruplets.

Final result: `[[-2, -1, 1, 2], [-2, 0, 0, 2], [-1, 0, 0, 1]]`, matching the expected output. Duplicates are avoided at **four levels**: skip repeated values for `i` (1st pointer), skip repeated values for `j` (2nd pointer, only when `j > i + 1`), and after finding a valid quadruplet, skip repeated values while advancing `left` and while retreating `right` (3rd and 4th pointers) — ensuring each distinct value-combination is added to the result exactly once.
