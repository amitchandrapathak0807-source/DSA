# Stack and Queue — Monotonic Stack Problems (Basic)

## Concept: Monotonic Stack

A **monotonic stack** is a stack whose elements are kept in a strictly increasing or strictly decreasing order from bottom to top at all times. We enforce this ordering ourselves: before pushing a new element, we pop off every element from the top of the stack that violates the desired order. Each element gets pushed exactly once and popped at most once, so any algorithm built around this idea runs in **O(n) total time**, even though it "looks like" a nested loop.

There are two flavors:

- **Monotonic decreasing stack** (top-to-bottom values decrease, i.e., the stack holds values from largest at bottom to smallest at top): useful for finding the **next greater element**. While scanning left to right, whenever the current element is greater than the element at the top of the stack, that top element has just found its "next greater element" — pop it and record the answer.
- **Monotonic increasing stack**: useful for finding the **next smaller element**, using the symmetric logic (pop while current element is smaller than the stack top).

### Why it beats the brute force

The brute force way to answer "next greater element for every index" is: for each index `i`, scan forward through `j = i+1 ... n-1` until you find a greater element. In the worst case (e.g., a strictly decreasing array) this degenerates to **O(n²)** comparisons.

The key insight of the monotonic stack is that **once we know an element's next greater/smaller element, we never need to look at it again** — it is resolved and can be discarded (popped). Every element is pushed onto the stack exactly once (O(n) pushes) and popped at most once (O(n) pops), so the total work across the entire pass is bounded by O(n) + O(n) = **O(n)**, no matter how the popping is distributed across iterations. This is a classic example of **amortized analysis**: individual steps can pop many elements, but summed over the whole run, the total number of pop operations can never exceed the number of pushes.

This same trick generalizes to:

- Next/previous greater element (linear or circular array)
- Next/previous smaller element
- Stock span (previous greater element, counting distance)
- Sum of subarray minimums/maximums (contribution technique using previous/next smaller or greater boundaries)
- Asteroid collision (simulate collisions using a stack of "surviving" asteroids)
- Removing digits to form the smallest number (greedy monotonic stack over digits)

---

## 1. Next Greater Element

**Problem Statement:** Given an array `arr` of `n` integers, for every element find the **first element to its right that is strictly greater than it**. If no such element exists, the answer for that index is `-1`.

**Example:**
- Input: `arr = [2, 1, 2, 4, 3]`
- Output: `[4, 2, 4, -1, -1]`
- Explanation: For `arr[0] = 2`, the first greater element to the right is `4`. For `arr[1] = 1`, it's `2`. For `arr[2] = 2`, it's `4`. For `arr[3] = 4`, there is nothing greater to its right, so `-1`. For `arr[4] = 3`, nothing to its right at all, so `-1`.

**Brute Force Approach:** For every index `i`, scan forward from `i+1` to `n-1` and stop at the first strictly greater value.

```csharp
public static int[] NextGreaterElementBrute(int[] arr)
{
    int n = arr.Length;
    int[] result = new int[n];

    for (int i = 0; i < n; i++)
    {
        result[i] = -1;
        for (int j = i + 1; j < n; j++)
        {
            if (arr[j] > arr[i])
            {
                result[i] = arr[j];
                break;
            }
        }
    }

    return result;
}
```

Time Complexity: O(n²) — for each element we may scan the rest of the array.
Space Complexity: O(1) extra (excluding the output array).

**Optimized Approach:** Traverse the array **right to left**, maintaining a **monotonic decreasing stack** of "candidate next greater elements". For each index `i`, pop everything from the stack that is `<= arr[i]` (those values can never be the answer for anyone to the left of `i`, since `arr[i]` is a better, closer, and larger blocker... actually they are simply not greater than `arr[i]`, so they're useless as an answer and also useless as a blocker for elements further left because `arr[i]` itself is closer and will be checked first). Whatever remains on top of the stack (if any) is the next greater element for `arr[i]`. Then push `arr[i]`.

```csharp
public static int[] NextGreaterElementOptimized(int[] arr)
{
    int n = arr.Length;
    int[] result = new int[n];
    Stack<int> stack = new Stack<int>(); // monotonic decreasing stack of values

    for (int i = n - 1; i >= 0; i--)
    {
        // Remove everything that is not greater than arr[i]; it can't be
        // the answer for arr[i], and it's now "hidden" behind arr[i]
        // for everyone further to the left too.
        while (stack.Count > 0 && stack.Peek() <= arr[i])
        {
            stack.Pop();
        }

        result[i] = stack.Count == 0 ? -1 : stack.Peek();

        stack.Push(arr[i]);
    }

    return result;
}
```

Time Complexity: O(n) — every element is pushed once and popped at most once.
Space Complexity: O(n) — for the stack and the output array.

**Explanation:** Dry run on `arr = [2, 1, 2, 4, 3]`, scanning from right to left, stack shown top-first:

| i | arr[i] | Pop while top <= arr[i] | Stack after pops | result[i] | Stack after push |
|---|--------|--------------------------|-------------------|-----------|--------------------|
| 4 | 3 | stack empty, nothing to pop | `[]` | -1 | `[3]` |
| 3 | 4 | pop `3` (3 <= 4) | `[]` | -1 | `[4]` |
| 2 | 2 | top is `4`, `4 <= 2`? no | `[4]` | 4 | `[2, 4]` |
| 1 | 1 | top is `2`, `2 <= 1`? no | `[2, 4]` | 2 | `[1, 2, 4]` |
| 0 | 2 | pop `1` (1 <= 2); top now `2`, `2 <= 2`? yes, pop `2`; top now `4`, `4 <= 2`? no | `[4]` | 4 | `[2, 4]` |

Final `result = [4, 2, 4, -1, -1]`, matching the expected output. Notice how the stack only ever holds values in decreasing order from bottom to top, which is exactly what lets us answer each query with a quick peek instead of a rescan.

---

## 2. Next Greater Element II

**Problem Statement:** Given a **circular array** `arr` of `n` integers (the next element after the last one wraps around to the first), find the next greater element for every element, searching circularly. If none exists even after wrapping all the way around, output `-1`.

**Example:**
- Input: `arr = [1, 2, 1]`
- Output: `[2, -1, 2]`
- Explanation: For `arr[0] = 1`, the next greater element is `arr[1] = 2`. For `arr[1] = 2`, there's no greater element anywhere (even wrapping around), so `-1`. For `arr[2] = 1`, wrapping around, the next greater is `arr[0]`'s neighbor... actually we continue past index 2 to index 0 (`1`, not greater) then index 1 (`2`, greater), so the answer is `2`.

**Brute Force Approach:** For every index `i`, scan up to `n-1` further elements using modulo indexing to simulate wraparound.

```csharp
public static int[] NextGreaterElementIIBrute(int[] arr)
{
    int n = arr.Length;
    int[] result = new int[n];

    for (int i = 0; i < n; i++)
    {
        result[i] = -1;
        for (int k = 1; k < n; k++)
        {
            int j = (i + k) % n;
            if (arr[j] > arr[i])
            {
                result[i] = arr[j];
                break;
            }
        }
    }

    return result;
}
```

Time Complexity: O(n²) — each element may scan almost the whole array.
Space Complexity: O(1) extra (excluding output array).

**Optimized Approach:** Reuse the same monotonic decreasing stack idea, but simulate the circular behavior by iterating the index from `2n - 1` down to `0` and mapping it back with `i % n`. The first pass over the "virtual" second half of the array (indices `n` to `2n-1`) pre-loads the stack with the wraparound information; the second pass (indices `0` to `n-1`) then records the real answers.

```csharp
public static int[] NextGreaterElementIIOptimized(int[] arr)
{
    int n = arr.Length;
    int[] result = new int[n];
    Stack<int> stack = new Stack<int>(); // monotonic decreasing stack of values

    for (int i = 2 * n - 1; i >= 0; i--)
    {
        int idx = i % n;

        while (stack.Count > 0 && stack.Peek() <= arr[idx])
        {
            stack.Pop();
        }

        if (i < n) // only record answers during the "real" pass
        {
            result[idx] = stack.Count == 0 ? -1 : stack.Peek();
        }

        stack.Push(arr[idx]);
    }

    return result;
}
```

Time Complexity: O(n) — we do 2n iterations, each still amortized O(1) due to the push/pop bound, so total O(n).
Space Complexity: O(n) — for the stack and the output array.

**Explanation:** For `arr = [1, 2, 1]`, `n = 3`, we iterate `i` from `5` down to `0`, using `idx = i % n`:

- `i = 5, idx = 2, arr[idx] = 1`: stack empty → nothing popped, (no record, `i >= n`), push `1` → stack `[1]`
- `i = 4, idx = 1, arr[idx] = 2`: pop `1` (1 <= 2), stack empty (no record), push `2` → stack `[2]`
- `i = 3, idx = 0, arr[idx] = 1`: top `2`, `2 <= 1`? no (no record, `i >= n`), push `1` → stack `[1, 2]`
- `i = 2, idx = 2, arr[idx] = 1`: top `1`, `1 <= 1`? yes pop; top now `2`, `2 <= 1`? no → `result[2] = 2`, push `1` → stack `[1, 2]`
- `i = 1, idx = 1, arr[idx] = 2`: pop `1` (1<=2); top now `2`, `2<=2`? yes pop; stack empty → `result[1] = -1`, push `2` → stack `[2]`
- `i = 0, idx = 0, arr[idx] = 1`: top `2`, `2 <= 1`? no → `result[0] = 2`, push `1` → stack `[1, 2]`

Final `result = [2, -1, 2]`, matching the expected output. The first pass (`i` from `2n-1` down to `n`) effectively "primes" the stack with the wraparound candidates before we start recording real answers.

---

## 3. Next Smaller Element

**Problem Statement:** Given an array `arr` of `n` integers, for every element find the **first element to its right that is strictly smaller than it**. If no such element exists, the answer is `-1`.

**Example:**
- Input: `arr = [4, 8, 5, 2, 25]`
- Output: `[2, 5, 2, -1, -1]`
- Explanation: For `arr[0] = 4`, the first smaller value to the right is `2`. For `arr[1] = 8`, it's `5`. For `arr[2] = 5`, it's `2`. For `arr[3] = 2`, nothing smaller to the right, so `-1`. For `arr[4] = 25`, nothing to its right at all, so `-1`.

**Brute Force Approach:** For every index `i`, scan forward until the first strictly smaller value is found.

```csharp
public static int[] NextSmallerElementBrute(int[] arr)
{
    int n = arr.Length;
    int[] result = new int[n];

    for (int i = 0; i < n; i++)
    {
        result[i] = -1;
        for (int j = i + 1; j < n; j++)
        {
            if (arr[j] < arr[i])
            {
                result[i] = arr[j];
                break;
            }
        }
    }

    return result;
}
```

Time Complexity: O(n²) — nested scan for each element.
Space Complexity: O(1) extra (excluding output array).

**Optimized Approach:** Same idea as Next Greater Element, but with a **monotonic increasing stack** (bottom-to-top values increase), scanning right to left. Pop everything `>= arr[i]` since it can never be the answer; whatever remains on top is the next smaller element.

```csharp
public static int[] NextSmallerElementOptimized(int[] arr)
{
    int n = arr.Length;
    int[] result = new int[n];
    Stack<int> stack = new Stack<int>(); // monotonic increasing stack of values

    for (int i = n - 1; i >= 0; i--)
    {
        while (stack.Count > 0 && stack.Peek() >= arr[i])
        {
            stack.Pop();
        }

        result[i] = stack.Count == 0 ? -1 : stack.Peek();

        stack.Push(arr[i]);
    }

    return result;
}
```

Time Complexity: O(n) — amortized, each element pushed and popped at most once.
Space Complexity: O(n) — for the stack and the output array.

**Explanation:** Dry run on `arr = [4, 8, 5, 2, 25]`, right to left:

- `i=4, arr[i]=25`: stack empty → `result[4] = -1`, push `25` → `[25]`
- `i=3, arr[i]=2`: top `25 >= 2`, pop; stack empty → `result[3] = -1`, push `2` → `[2]`
- `i=2, arr[i]=5`: top `2 >= 5`? no → `result[2] = 2`, push `5` → `[2, 5]`
- `i=1, arr[i]=8`: top `5 >= 8`? no → `result[1] = 5`, push `8` → `[5, 8]` (with `2` still under `5`)
- `i=0, arr[i]=4`: top `8 >= 4`, pop; top `5 >= 4`, pop; top `2 >= 4`? no → `result[0] = 2`, push `4`

Final `result = [2, 5, 2, -1, -1]`, matching the expected output.

---

## 4. Stock Span Problem

**Problem Statement:** Given a series of daily stock prices, compute the **span** of the stock's price on each day. The span on day `i` is defined as the maximum number of consecutive days (including today) ending at `i` for which the price was **less than or equal to** today's price.

**Example:**
- Input: `prices = [100, 80, 60, 70, 60, 75, 85]`
- Output: `[1, 1, 1, 2, 1, 4, 6]`
- Explanation: Day 0 (`100`): span `1` (only itself). Day 3 (`70`): the previous day `60 <= 70`, day before that `80 > 70` stops it, so span `2` (days at index 2,3). Day 6 (`85`): looking back, `75, 60, 70, 60, 80` are all `<= 85`, then `100 > 85` stops it, so span `6`.

**Brute Force Approach:** For each day `i`, walk backward counting consecutive days whose price is `<= prices[i]`, stopping at the first larger price or the start of the array.

```csharp
public static int[] StockSpanBrute(int[] prices)
{
    int n = prices.Length;
    int[] span = new int[n];

    for (int i = 0; i < n; i++)
    {
        int count = 1;
        int j = i - 1;
        while (j >= 0 && prices[j] <= prices[i])
        {
            count++;
            j--;
        }
        span[i] = count;
    }

    return span;
}
```

Time Complexity: O(n²) — worst case (strictly increasing prices) scans backward through the whole array each day.
Space Complexity: O(1) extra (excluding output array).

**Optimized Approach:** This is exactly a **"previous greater element"** query. Maintain a monotonic decreasing stack of `(price, index)` pairs while scanning left to right. Pop everything `<= prices[i]` (they can never block anyone to the right of `i` either, since `prices[i]` is closer and at least as large). The span is `i - indexOfPreviousGreater`. If the stack becomes empty, the previous greater doesn't exist, so span is `i + 1` (all days so far).

```csharp
public static int[] StockSpanOptimized(int[] prices)
{
    int n = prices.Length;
    int[] span = new int[n];
    Stack<(int price, int index)> stack = new Stack<(int, int)>(); // monotonic decreasing by price

    for (int i = 0; i < n; i++)
    {
        while (stack.Count > 0 && stack.Peek().price <= prices[i])
        {
            stack.Pop();
        }

        int previousGreaterIndex = stack.Count == 0 ? -1 : stack.Peek().index;
        span[i] = i - previousGreaterIndex;

        stack.Push((prices[i], i));
    }

    return span;
}
```

Time Complexity: O(n) — amortized, each day pushed and popped at most once.
Space Complexity: O(n) — for the stack and the output array.

**Explanation:** Dry run on `prices = [100, 80, 60, 70, 60, 75, 85]`, left to right, stack holds `(price, index)`:

- `i=0, price=100`: stack empty → prevGreaterIdx=-1, span=`0-(-1)=1`, push `(100,0)`
- `i=1, price=80`: top `(100,0)`, `100<=80`? no → prevGreaterIdx=0, span=`1-0=1`, push `(80,1)`
- `i=2, price=60`: top `(80,1)`, `80<=60`? no → prevGreaterIdx=1, span=`2-1=1`, push `(60,2)`
- `i=3, price=70`: pop `(60,2)` (60<=70); top `(80,1)`, `80<=70`? no → prevGreaterIdx=1, span=`3-1=2`, push `(70,3)`
- `i=4, price=60`: top `(70,3)`, `70<=60`? no → prevGreaterIdx=3, span=`4-3=1`, push `(60,4)`
- `i=5, price=75`: pop `(60,4)`, pop `(70,3)`; top `(80,1)`, `80<=75`? no → prevGreaterIdx=1, span=`5-1=4`, push `(75,5)`
- `i=6, price=85`: pop `(75,5)`; top `(80,1)`, `80<=85`? yes pop; top `(100,0)`, `100<=85`? no → prevGreaterIdx=0, span=`6-0=6`, push `(85,6)`

Final `span = [1, 1, 1, 2, 1, 4, 6]`, matching the expected output.

---

## 5. Sum of Subarray Minimums

**Problem Statement:** Given an array `arr` of `n` integers, consider every contiguous subarray. For each subarray, find its minimum element, and sum up all these minimums (typically taken modulo `1_000_000_007` since the sum can be huge).

**Example:**
- Input: `arr = [3, 1, 2, 4]`
- Output: `17`
- Explanation: All subarrays and their minimums: `[3]`→3, `[1]`→1, `[2]`→2, `[4]`→4, `[3,1]`→1, `[1,2]`→1, `[2,4]`→2, `[3,1,2]`→1, `[1,2,4]`→1, `[3,1,2,4]`→1. Sum = 3+1+2+4+1+1+2+1+1+1 = 17.

**Brute Force Approach:** Enumerate every subarray with two nested loops, tracking the running minimum as the right end extends, and add it to the total.

```csharp
public static long SumOfSubarrayMinimumsBrute(int[] arr)
{
    const int MOD = 1_000_000_007;
    int n = arr.Length;
    long total = 0;

    for (int i = 0; i < n; i++)
    {
        int currentMin = arr[i];
        for (int j = i; j < n; j++)
        {
            currentMin = Math.Min(currentMin, arr[j]);
            total = (total + currentMin) % MOD;
        }
    }

    return total;
}
```

Time Complexity: O(n²) — one pass per starting index `i`, extending `j`.
Space Complexity: O(1) extra.

**Optimized Approach — Contribution Technique:** Instead of summing "minimum per subarray", flip the perspective and sum "contribution per element": for each element `arr[i]`, count how many subarrays have `arr[i]` as their minimum, and add `arr[i] * count` to the total.

For element at index `i`, let:
- `left[i]` = distance to the **previous element that is strictly less than** `arr[i]` (use `<=` on one side and `<` on the other, consistently, to avoid double counting when duplicates exist) — i.e., how many consecutive positions to the left (including `i`) `arr[i]` remains the minimum.
- `right[i]` = distance to the **next element that is less than or equal to** `arr[i]`.

Both are found using a **monotonic increasing stack** (for "previous/next smaller element" indices). The number of subarrays where `arr[i]` is the minimum is `left[i] * right[i]`, because we can independently choose any of `left[i]` starting points and any of `right[i]` ending points and `arr[i]` will still be the minimum of that subarray.

```csharp
public static long SumOfSubarrayMinimumsOptimized(int[] arr)
{
    const int MOD = 1_000_000_007;
    int n = arr.Length;

    int[] left = new int[n];  // count of subarrays where arr[i] is min, extending left
    int[] right = new int[n]; // count of subarrays where arr[i] is min, extending right

    Stack<int> stack = new Stack<int>(); // holds indices, monotonic increasing by value

    // Previous Less Element (strictly less, to break ties only once)
    for (int i = 0; i < n; i++)
    {
        while (stack.Count > 0 && arr[stack.Peek()] >= arr[i])
        {
            stack.Pop();
        }
        int prevLessIndex = stack.Count == 0 ? -1 : stack.Peek();
        left[i] = i - prevLessIndex;
        stack.Push(i);
    }

    stack.Clear();

    // Next Less-or-Equal Element (use <= here so each duplicate value's subarrays
    // are counted exactly once across the array, avoiding double counting)
    for (int i = n - 1; i >= 0; i--)
    {
        while (stack.Count > 0 && arr[stack.Peek()] > arr[i])
        {
            stack.Pop();
        }
        int nextLessOrEqualIndex = stack.Count == 0 ? n : stack.Peek();
        right[i] = nextLessOrEqualIndex - i;
        stack.Push(i);
    }

    long total = 0;
    for (int i = 0; i < n; i++)
    {
        total = (total + (long)arr[i] * left[i] % MOD * right[i]) % MOD;
    }

    return total;
}
```

Time Complexity: O(n) — two linear passes, each with an amortized O(1) monotonic stack.
Space Complexity: O(n) — for the stack and the `left`/`right` arrays.

**Explanation:**

**Part 1 — dry run of the monotonic decreasing stack for Next Greater Element** (re-using `arr = [2, 1, 2, 4, 3]` from Problem 1, scanned right to left with a decreasing stack): already dry-run in detail in Problem 1's Explanation section above — see that table for the push/pop trace (result `[4, 2, 4, -1, -1]`).

**Part 2 — dry run of the contribution technique for Sum of Subarray Minimums** on `arr = [3, 1, 2, 4]` (indices 0..3):

*Previous Less Element pass (left to right, stack of indices, values shown in brackets):*
- `i=0 (val=3)`: stack empty → prevLessIndex=-1, `left[0] = 0-(-1) = 1`, push `0` → stack `[0]`
- `i=1 (val=1)`: top is index `0` (val 3), `3 >= 1`? yes pop; stack empty → prevLessIndex=-1, `left[1] = 1-(-1) = 2`, push `1` → stack `[1]`
- `i=2 (val=2)`: top is index `1` (val 1), `1 >= 2`? no → prevLessIndex=1, `left[2] = 2-1 = 1`, push `2` → stack `[1,2]`
- `i=3 (val=4)`: top is index `2` (val 2), `2 >= 4`? no → prevLessIndex=2, `left[3] = 3-2 = 1`, push `3` → stack `[1,2,3]`

So `left = [1, 2, 1, 1]`.

*Next Less-or-Equal pass (right to left, stack of indices):*
- `i=3 (val=4)`: stack empty → nextIndex=n=4, `right[3] = 4-3 = 1`, push `3` → stack `[3]`
- `i=2 (val=2)`: top is index `3` (val 4), `4 > 2`? yes pop; stack empty → nextIndex=4, `right[2] = 4-2 = 2`, push `2` → stack `[2]`
- `i=1 (val=1)`: top is index `2` (val 2), `2 > 1`? yes pop; stack empty → nextIndex=4, `right[1] = 4-1 = 3`, push `1` → stack `[1]`
- `i=0 (val=3)`: top is index `1` (val 1), `1 > 3`? no → nextIndex=1, `right[0] = 1-0 = 1`, push `0` → stack `[0,1]`

So `right = [1, 3, 2, 1]`.

*Focusing on element `arr[1] = 1`* (index 1, value 1): `left[1] = 2` means there are 2 valid left boundaries (starting index can be 0 or 1), and `right[1] = 3` means there are 3 valid right boundaries (ending index can be 1, 2, or 3). So `1` is the minimum of `left[1] * right[1] = 2 * 3 = 6` subarrays: `[3,1]`, `[1]`, `[3,1,2]`, `[1,2]`, `[3,1,2,4]`, `[1,2,4]` — indeed exactly 6 subarrays, each containing value `1` as their minimum. Its contribution to the total is `1 * 6 = 6`.

*Total contributions:* `arr[0]=3`: `3*1*1=3`. `arr[1]=1`: `1*2*3=6`. `arr[2]=2`: `2*1*2=4`. `arr[3]=4`: `4*1*1=4`. Sum = `3+6+4+4 = 17`, matching the expected output.

---

## 6. Asteroid Collision

**Problem Statement:** Given an array `asteroids` where each integer represents an asteroid: its absolute value is its size, and its sign is its direction (positive = moving right, negative = moving left). All asteroids move at the same speed. When two asteroids meet, the smaller one explodes; if both are the same size, both explode. Two asteroids moving in the same direction never meet. Return the state of the asteroids after all collisions.

**Example:**
- Input: `asteroids = [5, 10, -5]`
- Output: `[5, 10]`
- Explanation: The `10` and `-5` collide; `10` survives because it's bigger, so `-5` explodes. `5` is moving right and `-5` no longer exists, and `5` never collided with `10` (both moving right at the time). Result: `[5, 10]`.

**Brute Force Approach:** Repeatedly scan the array left to right looking for an adjacent "right-moving then left-moving" (`>0` followed by `<0`) collision pair, resolve it (remove the exploded one(s)), and restart the scan, until no more collisions are found. Using a `List<int>` to allow removal.

```csharp
public static List<int> AsteroidCollisionBrute(int[] asteroids)
{
    List<int> list = new List<int>(asteroids);
    bool collisionHappened = true;

    while (collisionHappened)
    {
        collisionHappened = false;

        for (int i = 0; i < list.Count - 1; i++)
        {
            if (list[i] > 0 && list[i + 1] < 0)
            {
                int right = list[i];
                int left = -list[i + 1]; // size of the left-moving asteroid

                if (right > left)
                {
                    list.RemoveAt(i + 1);
                }
                else if (right < left)
                {
                    list.RemoveAt(i);
                }
                else
                {
                    list.RemoveAt(i + 1);
                    list.RemoveAt(i);
                }

                collisionHappened = true;
                break; // restart the scan after any change
            }
        }
    }

    return list;
}
```

Time Complexity: O(n²) — worst case, each collision requires rescanning from the start (e.g., a long chain of collisions).
Space Complexity: O(n) — for the working list.

**Optimized Approach:** Process asteroids left to right using a **stack** representing the currently surviving asteroids so far (this is a monotonic-stack-style simulation: the stack only grows/shrinks based on comparisons with the top, discarding "destroyed" elements, giving amortized O(n)). For each new asteroid:
- If it's moving right (`> 0`), or the stack is empty, or the top of the stack is moving left (`< 0`, meaning no collision is possible), just push it.
- If it's moving left (`< 0`) and the stack top is moving right (`> 0`), a collision occurs: compare sizes, pop the smaller one(s), and keep resolving collisions with the new stack top until either the current asteroid is destroyed, the stack top wins, or they annihilate each other, or the stack no longer has a right-moving top.

```csharp
public static int[] AsteroidCollisionOptimized(int[] asteroids)
{
    Stack<int> stack = new Stack<int>();

    foreach (int asteroid in asteroids)
    {
        bool destroyed = false;
        int current = asteroid;

        // Only right-moving stack top can collide with a left-moving current asteroid
        while (!destroyed && current < 0 && stack.Count > 0 && stack.Peek() > 0)
        {
            int top = stack.Peek();

            if (top < -current) // stack top is smaller, it explodes
            {
                stack.Pop();
            }
            else if (top == -current) // same size, both explode
            {
                stack.Pop();
                destroyed = true;
            }
            else // stack top is bigger, current explodes
            {
                destroyed = true;
            }
        }

        if (!destroyed)
        {
            stack.Push(current);
        }
    }

    int[] result = stack.ToArray();
    Array.Reverse(result); // stack enumerates top-first, we want bottom-to-top order
    return result;
}
```

Time Complexity: O(n) — amortized, each asteroid is pushed once and popped at most once across the whole run.
Space Complexity: O(n) — for the stack.

**Explanation:** Dry run on `asteroids = [5, 10, -5]`:

- Process `5`: `current=5 > 0`, no collision check needed (only negative current can collide) → push `5` → stack (bottom→top) `[5]`
- Process `10`: `current=10 > 0` → push `10` → stack `[5, 10]`
- Process `-5`: `current=-5 < 0`, stack top is `10 > 0`, so check collision: `top=10`, `-current=5`, `top < -current`? `10 < 5`? no. `top == -current`? `10==5`? no. So "stack top is bigger, current explodes" → `destroyed=true`, loop ends. Since destroyed, do not push `-5`.

Final stack (bottom to top) `[5, 10]`, reversed-back-to-order gives `[5, 10]`, matching the expected output.

---

## 7. Sum of Subarray Ranges

**Problem Statement:** Given an array `arr` of `n` integers, the **range** of a subarray is `max(subarray) - min(subarray)`. Return the sum of ranges of all contiguous subarrays.

**Example:**
- Input: `arr = [1, 2, 3]`
- Output: `4`
- Explanation: Subarrays and ranges: `[1]`→0, `[2]`→0, `[3]`→0, `[1,2]`→1, `[2,3]`→1, `[1,2,3]`→2. Sum = 0+0+0+1+1+2 = 4.

**Brute Force Approach:** For every subarray, track running max and min as the right end extends, and accumulate `max - min`.

```csharp
public static long SumOfSubarrayRangesBrute(int[] arr)
{
    int n = arr.Length;
    long total = 0;

    for (int i = 0; i < n; i++)
    {
        int currentMax = arr[i];
        int currentMin = arr[i];
        for (int j = i; j < n; j++)
        {
            currentMax = Math.Max(currentMax, arr[j]);
            currentMin = Math.Min(currentMin, arr[j]);
            total += (currentMax - currentMin);
        }
    }

    return total;
}
```

Time Complexity: O(n²) — nested loop over all subarrays.
Space Complexity: O(1) extra.

**Optimized Approach:** Since `Sum of Ranges = Sum of Subarray Maximums - Sum of Subarray Minimums`, we can compute each half separately using the exact same **contribution technique** as Problem 5. For the sum of minimums, use previous-smaller / next-smaller-or-equal boundaries (monotonic increasing stack). For the sum of maximums, use previous-greater / next-greater-or-equal boundaries (monotonic decreasing stack) — symmetric logic, just flipping the comparison direction.

```csharp
public static long SumOfSubarrayRangesOptimized(int[] arr)
{
    return SumOfSubarrayMaximums(arr) - SumOfSubarrayMinimumsNoMod(arr);
}

private static long SumOfSubarrayMinimumsNoMod(int[] arr)
{
    int n = arr.Length;
    int[] left = new int[n];
    int[] right = new int[n];
    Stack<int> stack = new Stack<int>();

    for (int i = 0; i < n; i++)
    {
        while (stack.Count > 0 && arr[stack.Peek()] >= arr[i]) stack.Pop();
        left[i] = i - (stack.Count == 0 ? -1 : stack.Peek());
        stack.Push(i);
    }
    stack.Clear();

    for (int i = n - 1; i >= 0; i--)
    {
        while (stack.Count > 0 && arr[stack.Peek()] > arr[i]) stack.Pop();
        right[i] = (stack.Count == 0 ? n : stack.Peek()) - i;
        stack.Push(i);
    }

    long total = 0;
    for (int i = 0; i < n; i++)
        total += (long)arr[i] * left[i] * right[i];
    return total;
}

private static long SumOfSubarrayMaximums(int[] arr)
{
    int n = arr.Length;
    int[] left = new int[n];  // count extending left where arr[i] is the max
    int[] right = new int[n]; // count extending right where arr[i] is the max
    Stack<int> stack = new Stack<int>();

    // Previous Greater Element (strictly greater)
    for (int i = 0; i < n; i++)
    {
        while (stack.Count > 0 && arr[stack.Peek()] <= arr[i]) stack.Pop();
        left[i] = i - (stack.Count == 0 ? -1 : stack.Peek());
        stack.Push(i);
    }
    stack.Clear();

    // Next Greater-or-Equal Element
    for (int i = n - 1; i >= 0; i--)
    {
        while (stack.Count > 0 && arr[stack.Peek()] < arr[i]) stack.Pop();
        right[i] = (stack.Count == 0 ? n : stack.Peek()) - i;
        stack.Push(i);
    }

    long total = 0;
    for (int i = 0; i < n; i++)
        total += (long)arr[i] * left[i] * right[i];
    return total;
}
```

Time Complexity: O(n) — four linear passes total (two for minimums, two for maximums), each amortized O(1) per element.
Space Complexity: O(n) — for the stacks and auxiliary arrays.

**Explanation:**

**Part 1 — Next Greater Element dry run:** identical mechanics to the table already traced in Problem 1 (monotonic decreasing stack, right-to-left scan on `arr = [2, 1, 2, 4, 3]`, producing `[4, 2, 4, -1, -1]`); refer to that table for the full push/pop-by-push/pop trace.

**Part 2 — Contribution technique dry run** on `arr = [3, 1, 2, 4]` for the minimum side (identical computation as Problem 5's Part 2): recall `left = [1, 2, 1, 1]` and `right = [1, 3, 2, 1]`, giving `SumOfMinimums = 3*1*1 + 1*2*3 + 2*1*2 + 4*1*1 = 3+6+4+4 = 17`.

Now, focusing on element `arr[i] = 1` at index 1 specifically for the **maximum** side would give trivial contribution since `1` is never a subarray maximum unless it's a singleton — so instead let's verify the **minimum** boundary logic once more explicitly for this element: `left[1] = 2` counts starting points `{0, 1}` (subarrays `[3,1]` and `[1]`), and `right[1] = 3` counts ending points `{1, 2, 3}` (extending the subarray up to index 1, 2, or 3 while `1` remains the minimum, since `2` and `4` are both `> 1`). Multiplying gives `2 * 3 = 6` subarrays where `1` is the minimum, contributing `1 * 6 = 6` to the minimum-sum — consistent with Problem 5.

Applying the symmetric logic on `arr = [1, 2, 3]` for the full "Sum of Subarray Ranges" example: `SumOfMaximums = 0*1*1(for val1,left1,right1)...` — computing directly: for `arr=[1,2,3]`, `left` for maximums `= [1,1,1]`... walking through quickly: `arr[2]=3` is the max of all 3 subarrays ending at index 2 that start anywhere (`[3]`,`[2,3]`,`[1,2,3]`), contributing `3*3=9`; `arr[1]=2` is max only of `[2]`, contributing `2*1=2`; `arr[0]=1` is max only of `[1]`, contributing `1*1=1`. `SumOfMaximums = 9+2+1=12`. For minimums: `arr[0]=1` is min of `[1]`,`[1,2]`,`[1,2,3]` → `1*3=3`; `arr[1]=2` is min of `[2]` only → `2*1=2`; `arr[2]=3` is min of `[3]` only → `3*1=3`. `SumOfMinimums = 3+2+3=8`. `SumOfRanges = 12 - 8 = 4`, matching the expected output.

---

## 8. Remove K Digits

**Problem Statement:** Given a non-negative integer represented as a string `num` and an integer `k`, remove exactly `k` digits from `num` so that the resulting number is the **smallest possible**. The result should have no leading zeros (unless the result is `"0"` itself), and if all digits are removed the result is `"0"`.

**Example:**
- Input: `num = "1432219"`, `k = 3`
- Output: `"1219"`
- Explanation: Removing the digits `4`, `3`, and `2` (the first `2`, at index 4) from `1432219` leaves `1219`, which is the smallest possible number obtainable by removing exactly 3 digits.

**Brute Force Approach:** Try every combination of removing `k` digits out of `n` (i.e., every way to choose which `n-k` digit indices to keep, preserving relative order), build the resulting number for each, and keep the smallest. This is exponential — shown here restricted to a clearer but still very slow "greedy digit-by-digit brute force" that, at each of the `k` removals, scans the whole current string to find and remove the single digit whose removal yields the smallest result.

```csharp
public static string RemoveKDigitsBrute(string num, int k)
{
    string current = num;

    for (int step = 0; step < k; step++)
    {
        string best = null;
        for (int i = 0; i < current.Length; i++)
        {
            string candidate = current.Remove(i, 1);
            if (best == null || CompareNumericStrings(candidate, best) < 0)
            {
                best = candidate;
            }
        }
        current = best;
    }

    current = current.TrimStart('0');
    return current.Length == 0 ? "0" : current;
}

private static int CompareNumericStrings(string a, string b)
{
    // Equal-length numeric strings (both derived from num minus one char)
    // can be compared lexicographically after stripping leading zeros
    // for a fair "value" comparison, but for this brute force both have
    // equal length at each step, so simple lexicographic comparison works.
    return string.CompareOrdinal(a, b);
}
```

Time Complexity: O(k * n²) — `k` rounds, each scanning O(n) positions and building an O(n) candidate string.
Space Complexity: O(n) — for the current string / candidates.

**Optimized Approach:** Use a **monotonic increasing stack of digits** (as characters). Scan the digits left to right; while the top of the stack is **greater than** the current digit and we still have removals left (`k > 0`), pop the stack (this simulates "removing" that larger digit, which always helps make the number smaller when a smaller digit follows it). Push the current digit. After the scan, if `k` removals remain unused, remove them from the end of the stack (the largest remaining suffix). Finally strip leading zeros and handle the empty-result case.

```csharp
public static string RemoveKDigitsOptimized(string num, int k)
{
    Stack<char> stack = new Stack<char>(); // monotonic increasing stack (bottom-to-top)
    // We need bottom-to-top left-to-right order, so use a helper structure
    // that supports peeking at the "top" efficiently and reading in order at the end.
    // Stack<char> in C# works fine: Peek()/Pop() give the most recently pushed digit.

    foreach (char digit in num)
    {
        while (k > 0 && stack.Count > 0 && stack.Peek() > digit)
        {
            stack.Pop();
            k--;
        }
        stack.Push(digit);
    }

    // If removals remain, remove from the end (top of stack = end of number)
    while (k > 0 && stack.Count > 0)
    {
        stack.Pop();
        k--;
    }

    // Stack currently holds digits with the last-pushed (rightmost original) on top;
    // convert to array and reverse to get left-to-right order.
    char[] resultChars = stack.ToArray();
    Array.Reverse(resultChars);

    string result = new string(resultChars).TrimStart('0');
    return result.Length == 0 ? "0" : result;
}
```

Time Complexity: O(n) — amortized, each digit is pushed once and popped at most once.
Space Complexity: O(n) — for the stack.

**Explanation:** Dry run on `num = "1432219"`, `k = 3`, stack shown bottom→top left-to-right, processing digits left to right:

- digit `'1'`: stack empty → push → `[1]`, k=3
- digit `'4'`: top `'1' > '4'`? no → push → `[1,4]`, k=3
- digit `'3'`: top `'4' > '3'`? yes, pop (`k` 3→2) → `[1]`; top `'1' > '3'`? no → push → `[1,3]`, k=2
- digit `'2'`: top `'3' > '2'`? yes, pop (`k` 2→1) → `[1]`; top `'1' > '2'`? no → push → `[1,2]`, k=1
- digit `'2'`: top `'2' > '2'`? no (not strictly greater) → push → `[1,2,2]`, k=1
- digit `'1'`: top `'2' > '1'`? yes, pop (`k` 1→0) → `[1,2]`; k is now 0, stop popping → push → `[1,2,1]`, k=0
- digit `'9'`: k=0, no more popping allowed → push → `[1,2,1,9]`

End of scan, `k=0` so no trailing removal needed. Stack bottom-to-top is `1,2,1,9`, i.e., the string `"1219"`. No leading zeros to strip. Final result `"1219"`, matching the expected output.
