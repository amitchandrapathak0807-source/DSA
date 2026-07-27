# Array — Dutch National Flag Sort

### 1. Sort an Array of 0s, 1s, and 2s

**Problem Statement:**
Given an array `arr` containing only three distinct values — `0`, `1`, and `2` — sort the array in-place in a single (or minimal) pass, without using any built-in sorting function that runs in more than linear time. This is the classic **Dutch National Flag problem**, named after Edsger Dijkstra's algorithm for sorting a three-color flag (red, white, blue) in one pass. The goal is to group all `0`s first, followed by all `1`s, followed by all `2`s.

**Example:**
- Input: `arr = [0, 2, 1, 2, 0, 1]`
- Output: `[0, 0, 1, 1, 2, 2]`
- Explanation: All `0`s are moved to the front, all `1`s are placed in the middle, and all `2`s are pushed to the end, preserving the count of each element.

**Second Example (already sorted / all same value):**
- Input: `arr = [0, 0, 1, 1, 2, 2]`
- Output: `[0, 0, 1, 1, 2, 2]` (no change, array is already sorted)
- Input: `arr = [1, 1, 1, 1]`
- Output: `[1, 1, 1, 1]` (all elements identical, array remains unchanged)

---

**Brute Force Approach:**
Use a generic comparison-based sorting algorithm (like `Array.Sort` in C#, which is typically an introspective sort — a hybrid of quicksort, heapsort, and insertion sort) to sort the array. Since the array only has 3 distinct values, this works correctly but is wasteful — it does not exploit the fact that there are only 3 possible values, and runs in `O(n log n)` instead of `O(n)`.

**Logic (Steps):**
1. Call the built-in `Array.Sort` on the array.
2. Rely on the general-purpose comparison sort to place elements in ascending order.
3. Since only `0`, `1`, `2` are present, ascending order automatically groups them correctly.

```csharp
public static void SortColorsBrute(int[] arr)
{
    // Generic comparison-based sort - ignores the fact that
    // only values 0, 1, 2 are present.
    Array.Sort(arr);
}
```

**Time Complexity:** `O(n log n)` — this is the time complexity of the underlying comparison sort used by `Array.Sort`.
**Space Complexity:** `O(log n)` — due to the recursion stack used internally by the introspective sort (auxiliary, not counting output array).

**Walkthrough:** For `arr = [0, 2, 1, 2, 0, 1]`: `Array.Sort` reorders it to ascending order `[0, 0, 1, 1, 2, 2]` ✔ matches Output.

---

**Better Approach (Counting Sort — Two Passes):**
Since we know the array contains only `0`, `1`, and `2`, we can count the occurrences of each value in one pass, then overwrite the array in a second pass using those counts.

**Logic (Steps):**
1. First pass: traverse the array and tally `count0`, `count1`, `count2` for how many times each value appears.
2. Second pass: starting from `index = 0`, write `count0` zeros, then `count1` ones, then `count2` twos back into the array.
3. Return — the array is now grouped by value.

```csharp
public static void SortColorsCounting(int[] arr)
{
    int n = arr.Length;
    int count0 = 0, count1 = 0, count2 = 0;

    // Pass 1: count occurrences of each value
    for (int i = 0; i < n; i++)
    {
        if (arr[i] == 0) count0++;
        else if (arr[i] == 1) count1++;
        else count2++;
    }

    // Pass 2: overwrite the array using the counts
    int index = 0;
    for (int i = 0; i < count0; i++) arr[index++] = 0;
    for (int i = 0; i < count1; i++) arr[index++] = 1;
    for (int i = 0; i < count2; i++) arr[index++] = 2;
}
```

**Time Complexity:** `O(n) + O(n) = O(n)` — one pass to count, one pass to overwrite. Although this is linear, it requires **two passes** over the array, whereas the optimized approach below achieves the same result in a **single pass**.
**Space Complexity:** `O(1)` — only three counter variables are used, no extra array.

**Walkthrough:** For `arr = [0, 2, 1, 2, 0, 1]`: pass 1 tallies `count0=2, count1=2, count2=2`. Pass 2 writes `0,0` then `1,1` then `2,2` → `[0,0,1,1,2,2]` ✔ matches Output.

---

**Optimized Approach (Dutch National Flag Algorithm — Single Pass, Three Pointers):**
Use three pointers — `low`, `mid`, and `high` — to partition the array into four regions in a single traversal:
- `[0, low)` → all `0`s
- `[low, mid)` → all `1`s
- `[mid, high]` → unprocessed / unknown elements
- `(high, n-1]` → all `2`s

**Logic (Steps):**
1. Initialize `low = 0`, `mid = 0`, `high = n - 1`.
2. While `mid <= high`, inspect `arr[mid]`.
3. If `arr[mid] == 0`, swap it with `arr[low]`, then advance both `low` and `mid`.
4. If `arr[mid] == 1`, it's already correctly placed — just advance `mid`.
5. If `arr[mid] == 2`, swap it with `arr[high]` and decrement `high` (without advancing `mid`, since the swapped-in value is unverified).
6. Stop when `mid` crosses `high` — the array is now partitioned into `0`s, `1`s, `2`s.

```csharp
public static void SortColors(int[] arr)
{
    int n = arr.Length;
    int low = 0, mid = 0, high = n - 1;

    while (mid <= high)
    {
        if (arr[mid] == 0)
        {
            // Swap arr[low] and arr[mid], then advance both pointers
            (arr[low], arr[mid]) = (arr[mid], arr[low]);
            low++;
            mid++;
        }
        else if (arr[mid] == 1)
        {
            // 1 is already in its correct region, just move on
            mid++;
        }
        else // arr[mid] == 2
        {
            // Swap arr[mid] and arr[high], shrink the high boundary
            // Do NOT increment mid here - the swapped-in value is unverified
            (arr[mid], arr[high]) = (arr[high], arr[mid]);
            high--;
        }
    }
}
```

**Time Complexity:** `O(n)` — each element is visited by `mid` at most once, and the total work done by `low`/`mid`/`high` combined is bounded by `n`, so the array is sorted in a single linear pass.
**Space Complexity:** `O(1)` — sorting is done in-place using only three integer pointers, no auxiliary array.

**Walkthrough:**

The algorithm maintains a loop invariant over four regions of the array at all times, using three pointers `low`, `mid`, and `high`:

- `arr[0 .. low-1]` → contains **only 0s** (finalized)
- `arr[low .. mid-1]` → contains **only 1s** (finalized)
- `arr[mid .. high]` → **unprocessed** region, values not yet examined
- `arr[high+1 .. n-1]` → contains **only 2s** (finalized)

Initially, `low = 0`, `mid = 0`, `high = n - 1`, meaning the entire array is "unprocessed." The loop runs `while (mid <= high)`, examining `arr[mid]` and deciding one of three actions:

1. **`arr[mid] == 0`:** The element belongs in the "0 region." Swap `arr[low]` with `arr[mid]`. Since everything from `low` to `mid-1` was known to be `1`s, the element that lands at `mid` after the swap is guaranteed to be a `1` (or `mid == low`, in which case it's a no-op swap), so it is safe to increment both `low` and `mid`. This extends the 0-region by one and the 1-region shifts along with it.
2. **`arr[mid] == 1`:** The element is already correctly placed in what will become the 1-region. No swap needed — just increment `mid` to expand the boundary of the "1 region" and shrink the unprocessed region.
3. **`arr[mid] == 2`:** The element belongs in the "2 region" at the end. Swap `arr[mid]` with `arr[high]`, then decrement `high` to shrink the unprocessed region from the right. Crucially, `mid` is **not** incremented here, because the value swapped in from `arr[high]` has not yet been examined — it could be a 0, 1, or 2, so it must be re-evaluated in the next iteration.

**Dry run on `arr = [0, 2, 1, 2, 0, 1]`** (indices 0 to 5, `low = 0, mid = 0, high = 5`):

| Step | arr (before) | low | mid | high | arr[mid] | Action |
|------|---------------------------|-----|-----|------|----------|--------|
| 1 | `[0, 2, 1, 2, 0, 1]` | 0 | 0 | 5 | 0 | swap(0,0) → low=1, mid=1 |
| 2 | `[0, 2, 1, 2, 0, 1]` | 1 | 1 | 5 | 2 | swap(mid,high) → `[0, 1, 1, 2, 0, 2]`, high=4 |
| 3 | `[0, 1, 1, 2, 0, 2]` | 1 | 1 | 4 | 1 | mid=2 |
| 4 | `[0, 1, 1, 2, 0, 2]` | 1 | 2 | 4 | 1 | mid=3 |
| 5 | `[0, 1, 1, 2, 0, 2]` | 1 | 3 | 4 | 2 | swap(mid,high) → `[0, 1, 1, 0, 2, 2]`, high=3 |
| 6 | `[0, 1, 1, 0, 2, 2]` | 1 | 3 | 3 | 0 | swap(low,mid) → `[0, 0, 1, 1, 2, 2]`, low=2, mid=4 |

Now `mid (4) > high (3)`, loop terminates. Final array: `[0, 0, 1, 1, 2, 2]` — correctly sorted.

**Why the invariant holds:**
- Elements **before `low`** are always `0` because the only way an element enters that region is by being swapped there in the `arr[mid] == 0` case, and `low` only ever advances past a position after placing a confirmed `0` there.
- Elements **between `low` and `mid`** are always `1` because `mid` only advances without a swap when `arr[mid] == 1` (directly extending the 1-region), or after a `0`-swap where the value moved into the vacated `mid` slot is guaranteed to have come from the 1-region (since everything between `low` and `mid` was already known to be `1`s).
- Elements **after `high`** are always `2` because that region is only extended by swapping a confirmed `2` into position `high` before decrementing it.
- The **unprocessed region** `[mid, high]` shrinks by at least one element on every iteration (either `mid` advances, or `high` retreats, or both), guaranteeing the loop terminates after at most `n` iterations — giving the `O(n)` single-pass time complexity.
