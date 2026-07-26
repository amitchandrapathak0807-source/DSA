# Recursion — Subsequences and Subset Sum Pattern

## Concept: The Pick/Not-Pick Pattern

The **pick/not-pick** pattern is the single most important recursion template for generating subsequences, subsets, and combinations. It is used to solve every problem in this file.

A **subsequence** is any selection of elements from an array/string that preserves relative order (elements need not be contiguous). For an array of size `n`, there are exactly `2^n` possible subsequences (including the empty one), because each of the `n` elements independently has exactly two states: **included** or **excluded**.

### The Template

At every index `i` of the array, the recursive function makes a binary decision:

1. **Pick** `arr[i]` — add it to the current running subsequence, then recurse on `i + 1`.
2. **Don't pick** `arr[i]` — leave the current running subsequence untouched, then recurse on `i + 1`.

```csharp
void Solve(int i, int[] arr, List<int> current, List<List<int>> result)
{
    // Base case: we've made a decision for every index
    if (i == arr.Length)
    {
        result.Add(new List<int>(current)); // record a copy of the current subsequence
        return;
    }

    // Branch 1: PICK arr[i]
    current.Add(arr[i]);
    Solve(i + 1, arr, current, result);
    current.RemoveAt(current.Count - 1); // backtrack

    // Branch 2: DON'T PICK arr[i]
    Solve(i + 1, arr, current, result);
}
```

### Why This Produces a Recursion Tree of Depth `n` with `2^n` Leaves

- The recursion goes exactly `n` levels deep — one level per index — because every call advances `i` by 1 and the base case triggers only when `i == n`.
- At **every** level, the tree branches into 2 (pick / not-pick), so the total number of leaf nodes (complete decision paths) is `2 × 2 × 2 × ... (n times) = 2^n`.
- Each leaf corresponds to exactly one unique combination of "picked" / "not picked" decisions across all `n` indices — i.e., exactly one subsequence (some leaves may produce the same *values* if the array has duplicates, which is why "II" variants need extra duplicate-handling logic).

```
                              solve(0)
                    /pick 1/            \not pick/
              solve(1)                      solve(1)
            /        \                    /        \
      solve(2)     solve(2)          solve(2)     solve(2)
      /    \        /    \            /    \        /    \
   ...    ...     ...    ...        ...    ...     ...    ...
   (depth = n, total leaves = 2^n)
```

Every problem below is a variation of this same skeleton:
- **Print all subsequences** — record at every leaf.
- **Subsequence(s) with sum K** — track a running sum and check it at the leaf (or prune early).
- **Combination Sum I** — reuse allowed, so the "pick" branch stays at the same index instead of moving to `i + 1`.
- **Combination Sum II / Subset Sum II** — array has duplicates, so we sort first and explicitly skip adjacent duplicate values at the same recursion depth to avoid generating the same combination/subset twice.

---

## 1. Print All Subsequences of an Array/String

**Problem Statement:** Given an array (or string) of `n` elements, print/return all possible subsequences (contiguous or non-contiguous selections that preserve order), including the empty subsequence.

**Example:**
- Input: `arr = [3, 1, 2]`
- Output: `[3,1,2], [3,1], [3,2], [3], [1,2], [1], [2], []`
- Explanation: Every element can either be included or excluded, giving `2^3 = 8` subsequences in total.

**Approach:**
At index `i`, we have two choices for `arr[i]`: pick it (add to the current list and recurse on `i+1`) or don't pick it (recurse on `i+1` without adding it). When `i` reaches `arr.Length`, the current list is a complete subsequence — add a copy of it to the result. This is the pure, unmodified pick/not-pick template with no pruning and no target condition, so it explores the full recursion tree.

```csharp
public class PrintAllSubsequences
{
    public List<List<int>> GetAllSubsequences(int[] arr)
    {
        var result = new List<List<int>>();
        Solve(0, arr, new List<int>(), result);
        return result;
    }

    private void Solve(int i, int[] arr, List<int> current, List<List<int>> result)
    {
        if (i == arr.Length)
        {
            result.Add(new List<int>(current));
            return;
        }

        // Pick arr[i]
        current.Add(arr[i]);
        Solve(i + 1, arr, current, result);
        current.RemoveAt(current.Count - 1); // backtrack

        // Don't pick arr[i]
        Solve(i + 1, arr, current, result);
    }
}
```

**Time Complexity:** `O(2^n * n)` — there are `2^n` subsequences (leaves of the recursion tree), and copying each one into the result list costs up to `O(n)`.
**Space Complexity:** `O(n)` auxiliary recursion stack depth (plus `O(2^n * n)` to store all output subsequences). No pruning is applied here since every subsequence, regardless of content, must be printed.

**Explanation:**
For `arr = [1, 2]`:
- `Solve(0, [], ...)` → pick `1` → `current = [1]` → `Solve(1, ...)`
  - pick `2` → `current = [1,2]` → `Solve(2, ...)` → base case, record `[1,2]`, backtrack → `current = [1]`
  - don't pick `2` → `Solve(2, ...)` → base case, record `[1]`, backtrack → `current = []`
- back in `Solve(0,...)`, don't pick `1` → `Solve(1, ...)`
  - pick `2` → `current = [2]` → `Solve(2, ...)` → base case, record `[2]`, backtrack → `current = []`
  - don't pick `2` → `Solve(2, ...)` → base case, record `[]`
- Final result: `[1,2], [1], [2], []` — all `2^2 = 4` subsequences, matching the depth-2 tree with 4 leaves.

---

## 2. Print All Subsequences with Sum Equal to K (print just one valid subsequence)

**Problem Statement:** Given an array of `n` positive integers and a target `K`, find **any one** subsequence whose elements sum exactly to `K`. Once found, stop searching (no need to explore the rest of the tree).

**Example:**
- Input: `arr = [1, 2, 1], target = 2`
- Output: `[1, 1]` (one valid answer; `[2]` would also be valid, but only one needs to be returned)
- Explanation: We need just the *first* subsequence found (via a fixed pick/not-pick traversal order) whose sum equals `2`.

**Approach:**
Same pick/not-pick tree, but we track `currentSum` alongside the current list. At the base case (`i == n`), check if `currentSum == K`; if so, this is a valid answer. To return only **one** answer instead of exploring the whole tree, make the recursive function return a `bool` — as soon as the "pick" branch finds a valid answer, short-circuit and skip exploring the "don't pick" branch (and vice versa up the call chain). This is a classic use of a boolean-returning recursion to prune the search once a solution is found.

```csharp
public class OneSubsequenceWithSumK
{
    public List<int> FindOne(int[] arr, int k)
    {
        var current = new List<int>();
        Solve(0, arr, k, 0, current);
        return current; // empty list means no valid subsequence exists
    }

    // Returns true as soon as one valid subsequence is found; stops further exploration.
    private bool Solve(int i, int[] arr, int k, int currentSum, List<int> current)
    {
        if (i == arr.Length)
        {
            return currentSum == k;
        }

        // Pick arr[i]
        current.Add(arr[i]);
        if (Solve(i + 1, arr, k, currentSum + arr[i], current))
        {
            return true; // found it — stop, keep `current` as-is
        }
        current.RemoveAt(current.Count - 1); // backtrack, this pick didn't lead to a solution

        // Don't pick arr[i]
        if (Solve(i + 1, arr, k, currentSum, current))
        {
            return true;
        }

        return false;
    }
}
```

**Time Complexity:** Worst case `O(2^n)` if no valid subsequence exists (full tree explored), but typically much faster in practice because the search stops immediately once one valid path is found — the boolean short-circuit prunes all remaining branches at every level once `true` propagates up.
**Space Complexity:** `O(n)` for the recursion stack and the `current` list.

---

## 3. Count All Subsequences with Sum Equal to K

**Problem Statement:** Given an array of `n` positive integers and a target `K`, count how many subsequences (not the subsequences themselves, just the count) sum exactly to `K`.

**Example:**
- Input: `arr = [1, 2, 1], target = 2`
- Output: `2`
- Explanation: The valid subsequences summing to `2` are `[2]` and `[1, 1]` (the two `1`s at index 0 and 2). Even though there's only one `2` and one pair of `1`s, we count each valid *selection*, not each distinct set of values.

**Approach:**
Same tree again, but instead of collecting the actual subsequence, the recursive function returns an `int` count. At the base case, return `1` if `currentSum == k`, else `0`. The total count for a given index is `(count from picking) + (count from not picking)`. No need to maintain an explicit `current` list here since we only care about the running sum, not the actual elements — this shrinks the state we carry through the recursion.

```csharp
public class CountSubsequencesWithSumK
{
    public int CountWays(int[] arr, int k)
    {
        return Solve(0, arr, k, 0);
    }

    private int Solve(int i, int[] arr, int k, int currentSum)
    {
        if (i == arr.Length)
        {
            return currentSum == k ? 1 : 0;
        }

        // Pick arr[i]
        int pickCount = Solve(i + 1, arr, k, currentSum + arr[i]);

        // Don't pick arr[i]
        int notPickCount = Solve(i + 1, arr, k, currentSum);

        return pickCount + notPickCount;
    }
}
```

**Time Complexity:** `O(2^n)` in the worst case — the full recursion tree of depth `n` must be traversed since we need an exact count, not just existence, so no branch can be skipped once we've found one answer. (This can be optimized further with DP/memoization on `(i, currentSum)` if `arr` contains only non-negative integers, since overlapping subproblems arise — but the pure recursive version explores all `2^n` leaves.)
**Space Complexity:** `O(n)` recursion stack depth; no auxiliary list needed since only a running sum (an `int`) is passed down instead of a growing `List<int>`.

---

## 4. Combination Sum I (same element can be reused unlimited times)

**Problem Statement:** Given an array of **distinct** positive integers `candidates` and a target `target`, return all unique combinations of `candidates` where the chosen numbers sum to `target`. The **same number may be chosen from `candidates` an unlimited number of times**. Combinations are considered unique based on the *frequency* of chosen numbers, not the order.

**Example:**
- Input: `candidates = [2, 3, 6, 7], target = 7`
- Output: `[[2, 2, 3], [7]]`
- Explanation: `2 + 2 + 3 = 7` (using `2` twice, which is allowed since reuse is permitted) and `7 = 7`. No other combination of these candidates sums to `7`.

**Approach:**
This still uses the pick/not-pick tree, but with **one key twist**: because a number can be reused unlimited times, when we **pick** `candidates[i]`, we recurse again on the **same index `i`** (not `i + 1`) — this allows `candidates[i]` to be picked again in the next recursive call. Only the **don't-pick** branch advances to `i + 1`, permanently moving past this candidate. We also prune: if the running sum ever exceeds `target`, or if `i` reaches the end of the array, we stop that branch — this pruning is essential because without it, the "keep picking the same small number" branch would recurse infinitely (e.g., picking `2` forever).

```csharp
public class CombinationSumI
{
    public List<List<int>> CombinationSum(int[] candidates, int target)
    {
        var result = new List<List<int>>();
        Solve(0, candidates, target, new List<int>(), result);
        return result;
    }

    private void Solve(int i, int[] candidates, int target, List<int> current, List<List<int>> result)
    {
        // Base case: exact match found
        if (target == 0)
        {
            result.Add(new List<int>(current));
            return;
        }

        // Base case: no more candidates to try, or running sum already overshot
        if (i == candidates.Length || target < 0)
        {
            return;
        }

        // Pick candidates[i] — stay at index i to allow reuse
        current.Add(candidates[i]);
        Solve(i, candidates, target - candidates[i], current, result); // pruning happens via target < 0 check above
        current.RemoveAt(current.Count - 1); // backtrack

        // Don't pick candidates[i] — move on, this candidate is never used again
        Solve(i + 1, candidates, target, current, result);
    }
}
```

**Time Complexity:** Exponential, bounded roughly by `O(2^target)` in the worst case (e.g., all candidates equal to `1`), though in practice it's closer to `O(k^(target/min(candidates)))` for `k` candidates, since the "pick" branch can recurse many times at the same index. Pruning via `target < 0` cuts off entire subtrees early — as soon as the running sum overshoots `target`, we stop descending further on that branch instead of continuing to add more (always-positive) candidates.
**Space Complexity:** `O(target / min(candidates))` for the recursion stack depth (the longest possible chain of "pick" calls before `target` hits 0 or goes negative), plus space for the output.

**Explanation (dry run of the "stay at same index" trick):**
Using `candidates = [2, 3], target = 5`:

- `Solve(0, target=5, current=[])`
  - **Pick** `2` → `current=[2]` → `Solve(0, target=3, current=[2])` (still index `0`!)
    - **Pick** `2` again → `current=[2,2]` → `Solve(0, target=1, current=[2,2])`
      - Pick `2` again → `current=[2,2,2]` → `Solve(0, target=-1,...)` → `target < 0`, return (pruned)
      - backtrack → `current=[2,2]`
      - Don't pick `2` → `Solve(1, target=1, current=[2,2])`
        - Pick `3` → `target=1-3=-2 < 0` → pruned
        - Don't pick `3` → `Solve(2, target=1, ...)` → `i == candidates.Length` → return (no match)
    - backtrack → `current=[2]`
    - Don't pick `2` → `Solve(1, target=3, current=[2])`
      - Pick `3` → `current=[2,3]` → `Solve(1, target=0, current=[2,3])` → `target == 0` → **record `[2,3]`**
      - backtrack → `current=[2]`; Don't pick `3` → `Solve(2, target=3,...)` → end of array, return
  - backtrack → `current=[]`
  - **Don't pick** `2` → `Solve(1, target=5, current=[])`
    - Pick `3` → `current=[3]` → `Solve(1, target=2, current=[3])`
      - Pick `3` again → `target=2-3=-1 < 0` → pruned
      - Don't pick `3` → `Solve(2, target=2,...)` → end of array, return
    - Don't pick `3` → `Solve(2, target=5,...)` → end of array, return

Final result: `[[2, 3]]`. Notice how staying at index `0` after picking `2` is exactly what allowed the algorithm to consider `2` being used twice (in the `[2,2,...]` branch) before eventually rejecting it via pruning — that's the "reuse" mechanism, contrasted with the "don't pick" branch which permanently advances past index `0` to `1`.

---

## 5. Combination Sum II (each element used once, array may have duplicates, no duplicate combinations in output)

**Problem Statement:** Given a collection of candidate numbers `candidates` (which **may contain duplicates**) and a target `target`, find all unique combinations where the numbers sum to `target`. Each number in `candidates` may only be used **once** in each combination. The output must **not** contain duplicate combinations.

**Example:**
- Input: `candidates = [10, 1, 2, 7, 6, 1, 5], target = 8`
- Output: `[[1,1,6], [1,2,5], [1,7], [2,6]]`
- Explanation: Even though `1` appears twice in the input, only genuinely distinct multisets are included exactly once in the output — e.g. `[1,7]` appears once, not twice, even though there are two different `1`s that could each pair with `7`.

**Approach:**
Sort the array first. Now use pick/not-pick, but since each element can only be used once, both branches move to `i + 1` (unlike Combination Sum I where the pick branch stayed at `i`). The duplicate-avoidance trick: at each recursion level, when deciding which value to place at the *current* position, we loop through candidates starting at index `start` and skip any candidate equal to the previous one **at the same recursion depth** (`if (i > start && candidates[i] == candidates[i - 1]) continue;`). This ensures that among several equal values, only the *first* one at a given tree depth is used to "start" a new branch — preventing two identical subtrees from being generated. Sorting is essential so duplicates become adjacent and comparable.

```csharp
public class CombinationSumII
{
    public List<List<int>> CombinationSum2(int[] candidates, int target)
    {
        Array.Sort(candidates); // duplicates become adjacent — required for the skip trick
        var result = new List<List<int>>();
        Solve(0, candidates, target, new List<int>(), result);
        return result;
    }

    private void Solve(int start, int[] candidates, int target, List<int> current, List<List<int>> result)
    {
        if (target == 0)
        {
            result.Add(new List<int>(current));
            return;
        }

        for (int i = start; i < candidates.Length; i++)
        {
            // Pruning: sorted array means all subsequent candidates are >= candidates[i]
            if (candidates[i] > target)
            {
                break;
            }

            // Skip duplicates at the same recursion depth (same 'start' level)
            if (i > start && candidates[i] == candidates[i - 1])
            {
                continue;
            }

            // Pick candidates[i]
            current.Add(candidates[i]);
            Solve(i + 1, candidates, target - candidates[i], current, result); // move to i+1: no reuse
            current.RemoveAt(current.Count - 1); // backtrack
        }
    }
}
```

**Time Complexity:** `O(2^n)` worst case for exploring combinations (same exponential branching as the base pick/not-pick tree), with an additional `O(n log n)` for the initial sort. The `if (candidates[i] > target) break;` pruning cuts off entire remaining iterations early once sorted candidates exceed the remaining target, and the duplicate-skip check avoids doing redundant work down identical subtrees.
**Space Complexity:** `O(n)` for recursion depth (at most `n` elements can be picked before target hits 0 or the array ends), plus output storage.

**Explanation (dry run of the duplicate-skipping logic):**
Using `candidates = [1, 1, 2]` (already sorted), `target = 3`:

- `Solve(start=0, target=3, current=[])`
  - `i=0`: `candidates[0]=1`. `i > start`? No (`i == start`), so no skip. Pick `1` → `current=[1]` → `Solve(start=1, target=2, current=[1])`
    - `i=1`: `candidates[1]=1`. `i > start`? No (`i == start == 1`), no skip. Pick `1` → `current=[1,1]` → `Solve(start=2, target=1, current=[1,1])`
      - `i=2`: `candidates[2]=2 > target(1)` → **break** (pruned, no valid extension)
      - backtrack → `current=[1]`
    - `i=2`: `candidates[2]=2`. Pick `2` → `current=[1,2]` → `Solve(start=3, target=0, ...)` → `target==0` → **record `[1,2]`**... 

    wait — recompute: after picking index1(value1), target was 2; then loop continues to i=2 within the SAME for-loop (this is `Solve(start=1,...)`'s loop, second iteration): `candidates[2]=2`, `2 <= target(2)`, not a duplicate skip since `candidates[2]!=candidates[1]`. Pick `2` → `current=[1,2]` → `Solve(start=3, target=0,...)` → `target==0` → **record `[1,2]`**
    - backtrack → `current=[1]`, loop ends (i=3 out of bounds) → backtrack → `current=[]`
  - `i=1`: `candidates[1]=1`. `i > start` (`1 > 0`) **and** `candidates[1] == candidates[0]` (`1 == 1`) → **skip** (this is exactly the trick — without it we'd redundantly explore picking the *second* `1` first, generating `[1,2]` again and other duplicate combinations)
  - `i=2`: `candidates[2]=2`. Pick `2` → `current=[2]` → `Solve(start=3, target=1,...)` → loop doesn't execute (`start==3==length`), `target!=0`, returns without recording.

Final result: `[[1,1], ...]` wait — target was 3, recheck: `[1,1]` sums to 2, not 3, so it wouldn't be recorded (matches trace above — `[1,1]` branch got pruned at `target=1` with `candidates[2]=2>1`). Actual recorded combination: `[1,2]` (sum 3). This shows how skipping `i=1` at the top level (the second `1`) prevented a duplicate `[1,2]` from being generated a second time via a different index path, while still allowing `1` to appear twice in a combination when reached via the *inner* recursive call (`Solve(start=1,...)` picking its own `i=1`) — the key distinction is that the skip only applies to the **first candidate chosen at a given `start` level**, not to reuse of the value deeper in the same branch.

---

## 6. Subset Sum I (return sums of all possible subsets)

**Problem Statement:** Given an array of `n` non-negative integers, return the sum of every possible subset (there will be `2^n` subsets, hence `2^n` sums, in any order).

**Example:**
- Input: `arr = [3, 1, 2]`
- Output: `[6, 4, 5, 3, 3, 1, 2, 0]` (order may vary based on traversal)
- Explanation: `{3,1,2}=6, {3,1}=4, {3,2}=5, {3}=3, {1,2}=3, {1}=1, {2}=2, {}=0` — one sum per subset, all `2^3=8` subsets accounted for.

**Approach:**
Identical pick/not-pick tree as Problem 1, but instead of storing the actual subsequence at each leaf, we just carry a running `sum` and record that integer at the base case. This is a simplified/lighter version of the pattern since we never need to store or copy a growing list — just a single accumulating value.

```csharp
public class SubsetSumI
{
    public List<int> SubsetSums(int[] arr)
    {
        var result = new List<int>();
        Solve(0, arr, 0, result);
        return result;
    }

    private void Solve(int i, int[] arr, int currentSum, List<int> result)
    {
        if (i == arr.Length)
        {
            result.Add(currentSum);
            return;
        }

        // Pick arr[i]
        Solve(i + 1, arr, currentSum + arr[i], result);

        // Don't pick arr[i]
        Solve(i + 1, arr, currentSum, result);
    }
}
```

**Time Complexity:** `O(2^n)` — every one of the `2^n` leaves of the recursion tree contributes exactly one sum to the result, and no pruning is applicable since all subsets (even those with sum `0`) must be reported.
**Space Complexity:** `O(n)` recursion stack depth, plus `O(2^n)` to store all resulting sums.

---

## 7. Subset Sum II (return all unique subsets, array may have duplicates)

**Problem Statement:** Given an array `nums` that may contain duplicates, return all possible **unique** subsets (the power set), with no duplicate subset appearing in the output.

**Example:**
- Input: `nums = [1, 2, 2]`
- Output: `[[], [1], [1,2], [1,2,2], [2], [2,2]]`
- Explanation: Although `2` appears twice, the subset `[2]` should appear only once in the output (not twice, even though there are two different `2`s that individually could form `[2]`), and `[2,2]` (using both) is also included once.

**Approach:**
Sort the array so duplicates are adjacent. Then, instead of a strict binary pick/not-pick recursion, use the same "explore starting from index `start`" loop style as Combination Sum II: at each recursive call, record the current subset immediately (every node in the tree — not just leaves — is a valid subset), then loop `i` from `start` to the end, skipping any `i > start` where `nums[i] == nums[i-1]` (duplicate at the same depth). This guarantees that among repeated values, only the first occurrence at a given level starts a new distinct subset branch, so we never generate the same subset twice.

```csharp
public class SubsetSumII
{
    public List<List<int>> SubsetsWithDup(int[] nums)
    {
        Array.Sort(nums); // duplicates become adjacent — required for the skip trick
        var result = new List<List<int>>();
        Solve(0, nums, new List<int>(), result);
        return result;
    }

    private void Solve(int start, int[] nums, List<int> current, List<List<int>> result)
    {
        // Every node (not just leaves) represents a valid subset — record immediately
        result.Add(new List<int>(current));

        for (int i = start; i < nums.Length; i++)
        {
            // Skip duplicates at the same recursion depth (same 'start' level)
            if (i > start && nums[i] == nums[i - 1])
            {
                continue;
            }

            current.Add(nums[i]);
            Solve(i + 1, nums, current, result); // move to i+1: each element used once
            current.RemoveAt(current.Count - 1); // backtrack
        }
    }
}
```

**Time Complexity:** `O(2^n * n)` — there are up to `2^n` unique subsets, and each is copied into the result at cost `O(n)`; the duplicate-skip check avoids doing redundant recursive work for repeated values but does not change the asymptotic bound in the worst case (all-distinct array).
**Space Complexity:** `O(n)` recursion depth, plus `O(2^n * n)` for the output storage.

**Explanation (dry run of the duplicate-skipping logic):**
Using `nums = [1, 2, 2]` (already sorted):

- `Solve(start=0, current=[])` → record `[]`
  - `i=0`: `nums[0]=1`, `i==start`, no skip. Pick `1` → `current=[1]` → `Solve(start=1, current=[1])` → record `[1]`
    - `i=1`: `nums[1]=2`, `i==start`, no skip. Pick `2` → `current=[1,2]` → `Solve(start=2, current=[1,2])` → record `[1,2]`
      - `i=2`: `nums[2]=2`, `i==start`, no skip. Pick `2` → `current=[1,2,2]` → `Solve(start=3, current=[1,2,2])` → record `[1,2,2]` (loop doesn't execute, `start==length`)
      - backtrack → `current=[1,2]`
    - backtrack → `current=[1]`
    - `i=2`: `nums[2]=2`. `i > start` (`2>1`) **and** `nums[2]==nums[1]` (`2==2`) → **skip** (prevents generating `[1,2]` a second time via a different index path)
    - backtrack → `current=[]`
  - `i=1`: `nums[1]=2`, `i==start`? No — `start=0`, `i=1`, so `i>start` **and** compare `nums[1]` to `nums[0]`: `2 != 1` → not a duplicate at this level, no skip. Pick `2` → `current=[2]` → `Solve(start=2, current=[2])` → record `[2]`
    - `i=2`: `nums[2]=2`, `i==start`, no skip (this is a *fresh* level, `start=2`, so `i==start` bypasses the duplicate check). Pick `2` → `current=[2,2]` → `Solve(start=3, current=[2,2])` → record `[2,2]`
    - backtrack → `current=[]`
  - `i=2`: `nums[2]=2`. `i>start` (`2>0`) **and** `nums[2]==nums[1]` (`2==2`) → **skip** (prevents generating `[2]` a second time using the other `2`)

Final result: `[[], [1], [1,2], [1,2,2], [2], [2,2]]` — exactly 6 unique subsets, with no duplicates, even though the raw pick/not-pick tree on `[1,2,2]` would otherwise produce `2^3=8` nodes including duplicate `[2]` and `[1,2]` entries. The `i > start` condition is what makes this safe: it only forbids picking a duplicate value as the **starting** choice of a given recursive call's loop, while still allowing that same value to be picked immediately afterward as part of a deeper, already-committed branch (e.g., `[2,2]` still gets generated because the second `2` is picked at `start=2`, where `i==start` so the check doesn't apply).
