# Binary Search — Binary Search on Answer (Basic)

## Concept: Binary Search on Answer

Not every binary search runs on a sorted array. In many problems we are asked to **minimize or maximize some value `X`** (a speed, a capacity, a divisor, a square root, etc.) subject to a condition being satisfied. If we can write a **predicate function `predicate(X)`** that is **monotonic** over the range of possible answers — i.e. it is `false` for all values below some threshold and `true` for all values at or above it (or vice versa) — then the valid answers themselves form a sorted boolean sequence like:

```
false false false ... false true true true ... true
```

This is exactly the shape binary search needs. So instead of searching over the input array, we **binary search over the space of possible answers** `[low, high]`:

1. Identify the search space `[low, high]` — the smallest and largest possible values the answer could take.
2. Write `predicate(mid)` (also called a "feasibility check") that returns `true` if `mid` satisfies the required condition, `false` otherwise.
3. Depending on whether we want the smallest `X` for which `predicate(X)` is `true` (minimize) or the largest `X` for which it is `true` (maximize), move `low`/`high` accordingly, shrinking the search space by half each iteration.
4. This turns an `O(range)` linear scan into an `O(log(range))` search, with each predicate check typically costing `O(n)`, giving `O(n * log(range))` overall.

This pattern is used throughout this file to solve square roots, Nth roots, Koko's banana-eating speed, bouquet-making days, smallest divisor, and shipping capacity.

### 1. Find the Square Root of a Number using Binary Search

**Problem Statement:** Given a non-negative integer `n`, find the floor value of its square root, i.e., the largest integer `x` such that `x * x <= n`, without using built-in power/sqrt functions.

**Example:**
- Input: `n = 28`
- Output: `5`
- Explanation: `5 * 5 = 25 <= 28` but `6 * 6 = 36 > 28`, so the floor of the square root is `5`.

**Brute Force Approach:** Linearly try every integer from `1` upward, checking the predicate `x * x <= n`, and stop as soon as it fails.

```csharp
public class SqrtSolution
{
    public int FloorSqrtBrute(int n)
    {
        if (n < 1) return 0;

        int ans = 1;
        for (int x = 1; (long)x * x <= n; x++)
        {
            ans = x;
        }
        return ans;
    }
}
```

Time Complexity: O(sqrt(n)) — we walk up to sqrt(n) candidates.
Space Complexity: O(1).

**Optimized Approach:** Binary search on the answer `x` in the range `[1, n]`. The predicate `x * x <= n` is monotonic (true for small `x`, false for large `x`), so we search for the largest `x` where it holds.

```csharp
public class SqrtSolution
{
    public int FloorSqrtOptimal(int n)
    {
        if (n < 1) return 0;

        int low = 1, high = n, ans = 0;
        while (low <= high)
        {
            int mid = low + (high - low) / 2;
            if (IsFeasible(mid, n))
            {
                ans = mid;      // mid is a valid candidate, try for a larger one
                low = mid + 1;
            }
            else
            {
                high = mid - 1; // mid too large, shrink upper bound
            }
        }
        return ans;
    }

    // Predicate: does mid*mid stay within n?
    private bool IsFeasible(int mid, int n)
    {
        long square = (long)mid * mid; // avoid overflow
        return square <= n;
    }
}
```

Time Complexity: O(log(n)), Space Complexity: O(1).

### 2. Find the Nth Root of a Number using Binary Search

**Problem Statement:** Given two integers `n` and `m`, find the Nth root of `m`, i.e., an integer `x` such that `x^n == m`. If no such integer exists, return `-1`.

**Example:**
- Input: `n = 3, m = 27`
- Output: `3`
- Explanation: `3^3 = 27`, so the cube root of `27` is `3`.

**Brute Force Approach:** Linearly try every integer from `1` to `m`, checking whether `x^n == m`.

```csharp
public class NthRootSolution
{
    public int NthRootBrute(int n, int m)
    {
        for (int x = 1; x <= m; x++)
        {
            long power = Power(x, n, m);
            if (power == m) return x;
            if (power > m) break; // no point going further
        }
        return -1;
    }

    private long Power(int x, int n, long cap)
    {
        long result = 1;
        for (int i = 0; i < n; i++)
        {
            result *= x;
            if (result > cap) return result; // early exit to avoid overflow/slowness
        }
        return result;
    }
}
```

Time Complexity: O(m * n).
Space Complexity: O(1).

**Optimized Approach:** Binary search on the answer `x` in the range `[1, m]`. The predicate "x^n >= m" is monotonic, so we search for the `x` whose `n`th power exactly equals `m`.

```csharp
public class NthRootSolution
{
    public int NthRootOptimal(int n, int m)
    {
        int low = 1, high = m;
        while (low <= high)
        {
            int mid = low + (high - low) / 2;
            int cmp = ComparePower(mid, n, m);

            if (cmp == 0) return mid;       // mid^n == m, found exact Nth root
            else if (cmp < 0) low = mid + 1; // mid^n < m, need a bigger base
            else high = mid - 1;             // mid^n > m, need a smaller base
        }
        return -1;
    }

    // Predicate/helper: compares mid^n against m without overflowing.
    // Returns -1 if mid^n < m, 0 if equal, 1 if mid^n > m.
    private int ComparePower(int mid, int n, int m)
    {
        long result = 1;
        for (int i = 0; i < n; i++)
        {
            result *= mid;
            if (result > m) return 1;
        }
        if (result == m) return 0;
        return -1;
    }
}
```

Time Complexity: O(n * log(m)), Space Complexity: O(1).

### 3. Koko Eating Bananas

**Problem Statement:** Koko loves bananas. There are `n` piles of bananas, the `i`th pile has `piles[i]` bananas. She has `h` hours to eat all the bananas before the guards return. Each hour she chooses a pile and eats up to `k` bananas from it (if the pile has fewer than `k`, she eats all of it and moves on — she does not eat from another pile that same hour). Find the minimum integer eating speed `k` such that she can finish all piles within `h` hours.

**Example:**
- Input: `piles = [3,6,7,11], h = 8`
- Output: `4`
- Explanation: With speed `k = 4`, hours needed = ceil(3/4) + ceil(6/4) + ceil(7/4) + ceil(11/4) = 1 + 2 + 2 + 3 = 8, which fits exactly within `h = 8`. Any smaller speed needs more than 8 hours.

**Brute Force Approach:** Try every possible speed `k` from `1` to `max(piles)`, compute the hours needed for each, and return the first `k` for which hours needed `<= h`.

```csharp
public class KokoSolution
{
    public int MinEatingSpeedBrute(int[] piles, int h)
    {
        int maxPile = 0;
        foreach (int p in piles) maxPile = Math.Max(maxPile, p);

        for (int k = 1; k <= maxPile; k++)
        {
            if (HoursNeeded(piles, k) <= h)
            {
                return k;
            }
        }
        return maxPile;
    }

    private long HoursNeeded(int[] piles, int k)
    {
        long hours = 0;
        foreach (int p in piles)
        {
            hours += (p + k - 1) / k; // ceil(p / k)
        }
        return hours;
    }
}
```

Time Complexity: O(maxPile * n).
Space Complexity: O(1).

**Optimized Approach:** Binary search on the eating speed `k` in the range `[1, max(piles)]`. The predicate "hours needed at speed k <= h" is monotonic (higher speed never increases hours needed), so we search for the smallest feasible `k`.

```csharp
public class KokoSolution
{
    public int MinEatingSpeedOptimal(int[] piles, int h)
    {
        int low = 1, high = 0;
        foreach (int p in piles) high = Math.Max(high, p);

        int ans = high;
        while (low <= high)
        {
            int mid = low + (high - low) / 2;
            if (IsFeasible(piles, mid, h))
            {
                ans = mid;       // mid works, try a smaller (slower) speed
                high = mid - 1;
            }
            else
            {
                low = mid + 1;   // too slow, need a bigger speed
            }
        }
        return ans;
    }

    // Predicate: can Koko finish all piles within h hours at speed k?
    private bool IsFeasible(int[] piles, int k, int h)
    {
        long hours = 0;
        foreach (int p in piles)
        {
            hours += (p + k - 1) / k; // ceil(p / k)
        }
        return hours <= h;
    }
}
```

Time Complexity: O(n * log(maxPile)), Space Complexity: O(1).

**Explanation:** Dry run with `piles = [3,6,7,11], h = 8`.

- Search space for speed `k` is `[1, 11]` (11 is the largest pile).
- `low = 1, high = 11`. `mid = 6`. Hours needed = ceil(3/6)+ceil(6/6)+ceil(7/6)+ceil(11/6) = 1+1+2+2 = 6. `6 <= 8` → feasible, so `ans = 6`, `high = 5`.
- `low = 1, high = 5`. `mid = 3`. Hours needed = ceil(3/3)+ceil(6/3)+ceil(7/3)+ceil(11/3) = 1+2+3+4 = 10. `10 > 8` → not feasible, so `low = 4`.
- `low = 4, high = 5`. `mid = 4`. Hours needed = ceil(3/4)+ceil(6/4)+ceil(7/4)+ceil(11/4) = 1+2+2+3 = 8. `8 <= 8` → feasible, so `ans = 4`, `high = 3`.
- Now `low = 4 > high = 3`, loop ends. Final answer `ans = 4`.

At every step the predicate "hours needed at speed k" is recomputed by summing `ceil(pile / k)` for each pile, and compared against `h`; because increasing `k` never increases the hours needed, the feasible speeds form a monotonic `false...false true...true` sequence, letting binary search converge on the minimum feasible speed.

### 4. Minimum Number of Days to Make M Bouquets

**Problem Statement:** Given an array `bloomDay` where `bloomDay[i]` is the day on which the `i`th flower blooms, and integers `m` and `k`, find the minimum number of days needed to wait so that we can make `m` bouquets, each using `k` adjacent (contiguous) bloomed flowers. If it is impossible to make `m` bouquets, return `-1`.

**Example:**
- Input: `bloomDay = [1,10,3,10,2], m = 3, k = 1`
- Output: `3`
- Explanation: On day 3, flowers at indices 0, 2, 4 have bloomed (values 1, 3, 2 <= 3), giving us at least 3 single-flower bouquets (`k = 1`), satisfying `m = 3`.

**Brute Force Approach:** Try every possible day from `min(bloomDay)` to `max(bloomDay)`, and for each day check if `m` bouquets can be made.

```csharp
public class BouquetSolution
{
    public int MinDaysBrute(int[] bloomDay, int m, int k)
    {
        long need = (long)m * k;
        if (need > bloomDay.Length) return -1;

        int minDay = int.MaxValue, maxDay = int.MinValue;
        foreach (int d in bloomDay)
        {
            minDay = Math.Min(minDay, d);
            maxDay = Math.Max(maxDay, d);
        }

        for (int day = minDay; day <= maxDay; day++)
        {
            if (CanMakeBouquets(bloomDay, day, m, k))
            {
                return day;
            }
        }
        return -1;
    }

    private bool CanMakeBouquets(int[] bloomDay, int day, int m, int k)
    {
        int bouquets = 0, adjacent = 0;
        foreach (int d in bloomDay)
        {
            if (d <= day)
            {
                adjacent++;
                if (adjacent == k)
                {
                    bouquets++;
                    adjacent = 0;
                }
            }
            else
            {
                adjacent = 0;
            }
        }
        return bouquets >= m;
    }
}
```

Time Complexity: O((maxDay - minDay) * n).
Space Complexity: O(1).

**Optimized Approach:** Binary search on the day in the range `[min(bloomDay), max(bloomDay)]`. The predicate "can make m bouquets by this day" is monotonic (later days never hurt), so we search for the smallest feasible day.

```csharp
public class BouquetSolution
{
    public int MinDaysOptimal(int[] bloomDay, int m, int k)
    {
        long need = (long)m * k;
        if (need > bloomDay.Length) return -1;

        int low = int.MaxValue, high = int.MinValue;
        foreach (int d in bloomDay)
        {
            low = Math.Min(low, d);
            high = Math.Max(high, d);
        }

        int ans = -1;
        while (low <= high)
        {
            int mid = low + (high - low) / 2;
            if (IsFeasible(bloomDay, mid, m, k))
            {
                ans = mid;        // mid works, try an earlier day
                high = mid - 1;
            }
            else
            {
                low = mid + 1;    // not enough bloomed, wait longer
            }
        }
        return ans;
    }

    // Predicate: can we make m bouquets of k adjacent flowers by this day?
    private bool IsFeasible(int[] bloomDay, int day, int m, int k)
    {
        int bouquets = 0, adjacent = 0;
        foreach (int d in bloomDay)
        {
            if (d <= day)
            {
                adjacent++;
                if (adjacent == k)
                {
                    bouquets++;
                    adjacent = 0;
                }
            }
            else
            {
                adjacent = 0;
            }
        }
        return bouquets >= m;
    }
}
```

Time Complexity: O(n * log(maxDay - minDay)), Space Complexity: O(1).

### 5. Find the Smallest Divisor Given a Threshold

**Problem Statement:** Given an array `nums` and an integer `threshold`, find the smallest positive integer divisor such that the sum of `ceil(nums[i] / divisor)` for all elements is `<= threshold`.

**Example:**
- Input: `nums = [1,2,5,9], threshold = 6`
- Output: `5`
- Explanation: With divisor `5`, sum = ceil(1/5)+ceil(2/5)+ceil(5/5)+ceil(9/5) = 1+1+1+2 = 5, which is `<= 6`. Divisor `4` gives sum = 1+1+2+3 = 7 > 6, so `5` is the smallest valid divisor.

**Brute Force Approach:** Try every divisor from `1` to `max(nums)`, compute the sum of ceiling divisions, and return the first divisor for which the sum is within the threshold.

```csharp
public class SmallestDivisorSolution
{
    public int SmallestDivisorBrute(int[] nums, int threshold)
    {
        int maxNum = 0;
        foreach (int x in nums) maxNum = Math.Max(maxNum, x);

        for (int divisor = 1; divisor <= maxNum; divisor++)
        {
            if (SumOfDivisions(nums, divisor) <= threshold)
            {
                return divisor;
            }
        }
        return maxNum;
    }

    private long SumOfDivisions(int[] nums, int divisor)
    {
        long sum = 0;
        foreach (int x in nums)
        {
            sum += (x + divisor - 1) / divisor; // ceil(x / divisor)
        }
        return sum;
    }
}
```

Time Complexity: O(maxNum * n).
Space Complexity: O(1).

**Optimized Approach:** Binary search on the divisor in the range `[1, max(nums)]`. The predicate "sum of ceiling divisions <= threshold" is monotonic (larger divisor never increases the sum), so we search for the smallest feasible divisor.

```csharp
public class SmallestDivisorSolution
{
    public int SmallestDivisorOptimal(int[] nums, int threshold)
    {
        int low = 1, high = 0;
        foreach (int x in nums) high = Math.Max(high, x);

        int ans = high;
        while (low <= high)
        {
            int mid = low + (high - low) / 2;
            if (IsFeasible(nums, mid, threshold))
            {
                ans = mid;        // mid works, try a smaller divisor
                high = mid - 1;
            }
            else
            {
                low = mid + 1;    // sum too large, need a bigger divisor
            }
        }
        return ans;
    }

    // Predicate: is the sum of ceil(x / divisor) within the threshold?
    private bool IsFeasible(int[] nums, int divisor, int threshold)
    {
        long sum = 0;
        foreach (int x in nums)
        {
            sum += (x + divisor - 1) / divisor; // ceil(x / divisor)
        }
        return sum <= threshold;
    }
}
```

Time Complexity: O(n * log(maxNum)), Space Complexity: O(1).

### 6. Capacity to Ship Packages within D Days

**Problem Statement:** Given an array `weights` representing the weight of packages that must be shipped in order within `days` days, find the least weight capacity of a ship such that all packages can be shipped within `days` days (each day the ship loads packages in order, up to its capacity, without exceeding it).

**Example:**
- Input: `weights = [1,2,3,4,5,6,7,8,9,10], days = 5`
- Output: `15`
- Explanation: With capacity `15`, shipments are `[1,2,3,4,5]`, `[6,7]`, `[8]`, `[9]`, `[10]` — 5 days, matching `days = 5`. Any smaller capacity requires more than 5 days.

**Brute Force Approach:** Try every capacity from `max(weights)` (minimum possible, since a single package must fit) to `sum(weights)` (maximum possible, ship everything in one day), and return the first capacity that ships within `days`.

```csharp
public class ShipCapacitySolution
{
    public int ShipWithinDaysBrute(int[] weights, int days)
    {
        int maxWeight = 0;
        long totalWeight = 0;
        foreach (int w in weights)
        {
            maxWeight = Math.Max(maxWeight, w);
            totalWeight += w;
        }

        for (int capacity = maxWeight; capacity <= totalWeight; capacity++)
        {
            if (DaysNeeded(weights, capacity) <= days)
            {
                return capacity;
            }
        }
        return (int)totalWeight;
    }

    private int DaysNeeded(int[] weights, int capacity)
    {
        int days = 1;
        long currentLoad = 0;
        foreach (int w in weights)
        {
            if (currentLoad + w > capacity)
            {
                days++;
                currentLoad = 0;
            }
            currentLoad += w;
        }
        return days;
    }
}
```

Time Complexity: O((sum(weights) - maxWeight) * n).
Space Complexity: O(1).

**Optimized Approach:** Binary search on the capacity in the range `[max(weights), sum(weights)]`. The predicate "days needed at this capacity <= days" is monotonic (larger capacity never increases days needed), so we search for the smallest feasible capacity.

```csharp
public class ShipCapacitySolution
{
    public int ShipWithinDaysOptimal(int[] weights, int days)
    {
        int low = 0;
        long high = 0;
        foreach (int w in weights)
        {
            low = Math.Max(low, w);
            high += w;
        }

        int ans = (int)high;
        while (low <= high)
        {
            int mid = (int)(low + (high - low) / 2);
            if (IsFeasible(weights, mid, days))
            {
                ans = mid;         // mid works, try a smaller capacity
                high = mid - 1;
            }
            else
            {
                low = mid + 1;     // too many days needed, increase capacity
            }
        }
        return ans;
    }

    // Predicate: can all packages be shipped within 'days' using this capacity?
    private bool IsFeasible(int[] weights, int capacity, int days)
    {
        int daysNeeded = 1;
        long currentLoad = 0;
        foreach (int w in weights)
        {
            if (currentLoad + w > capacity)
            {
                daysNeeded++;
                currentLoad = 0;
            }
            currentLoad += w;
        }
        return daysNeeded <= days;
    }
}
```

Time Complexity: O(n * log(sum(weights) - maxWeight)), Space Complexity: O(1).

**Explanation:** The predicate here is analogous to Koko's hours-needed check, but instead computes "days needed" for a given capacity: walk through `weights` in order, greedily loading packages onto the current day's shipment; whenever adding the next package would exceed `capacity`, start a new day. For example, with `weights = [1,2,3,4,5,6,7,8,9,10], days = 5`, the search space for capacity is `[max=10, sum=55]`.

- `low = 10, high = 55`. `mid = 32`. Simulating: loads `1..10` all fit within 32 easily in very few days (day 1 carries `1+2+...+8=36`? — recomputing greedily gives around 2 days), so `daysNeeded <= 5` → feasible, `ans = 32`, `high = 31`.
- The search continues shrinking `high` whenever the simulated `daysNeeded` is `<= 5`, and shrinking from `low` whenever `daysNeeded > 5`, until `low` and `high` converge.
- Eventually the search narrows to `mid = 15`: shipments become `[1,2,3,4,5]` (sum 15), `[6,7]` (sum 13), `[8]`, `[9]`, `[10]` — exactly 5 days, feasible. Trying `mid = 14` gives more than 5 days (infeasible), confirming `15` is the minimum capacity.
- Because increasing capacity never increases the days needed, the feasibility sequence over capacities is monotonic, and binary search correctly converges on the smallest capacity for which `daysNeeded <= days`.
