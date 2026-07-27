# Recursion — Permutations and Partitions (Backtracking)

## 1. Print All Permutations of an Array/String

**Problem Statement:**
Given an array (or string) of `n` distinct elements, print/return all possible permutations (arrangements) of the elements. There are `n!` such permutations.

**Example:**
- Input: `nums = [1, 2, 3]`
- Output: `[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]`
- Explanation: Every possible ordering of the three elements is generated exactly once — `3! = 6` permutations in total.

**Brute Force / Backtracking Approach:**
The classic pattern is *choose → explore → un-choose*:

**Logic (Steps):**
1. **Approach A (`used[]` array):** maintain a `current` list and a `used[]` boolean array. At each call, loop over every index `i`; skip indices already `used`.
2. Choose: mark `used[i] = true` and append `nums[i]` to `current`. Explore: recurse to fill the next position.
3. Un-choose (backtrack): after the recursive call returns, remove the last element from `current` and reset `used[i] = false`, then try the next `i`.
4. Base case: when `current.Count == nums.Length`, a full permutation is formed — add a copy to `result`.
5. **Approach B (swapping):** at depth `idx`, loop `i` from `idx` to `n-1`; swap `nums[idx]` and `nums[i]` (choose), recurse on `idx+1` (explore), then swap back (un-choose) to restore the array before the next `i`.
6. Base case: when `idx == nums.Length`, copy the current arrangement of `nums` into `result`.

```csharp
using System.Collections.Generic;

public class PermutationsSolver
{
    // Approach A: used[] array + separate list
    public IList<IList<int>> PermuteUsingUsedArray(int[] nums)
    {
        var result = new List<IList<int>>();
        var current = new List<int>();
        var used = new bool[nums.Length];
        Backtrack(nums, used, current, result);
        return result;
    }

    private void Backtrack(int[] nums, bool[] used, List<int> current, IList<IList<int>> result)
    {
        if (current.Count == nums.Length)
        {
            result.Add(new List<int>(current)); // record a full permutation
            return;
        }

        for (int i = 0; i < nums.Length; i++)
        {
            if (used[i]) continue;

            // choose
            used[i] = true;
            current.Add(nums[i]);

            // explore
            Backtrack(nums, used, current, result);

            // un-choose (backtrack)
            current.RemoveAt(current.Count - 1);
            used[i] = false;
        }
    }

    // Approach B: in-place swapping (no extra used[] / current list)
    public IList<IList<int>> PermuteUsingSwapping(int[] nums)
    {
        var result = new List<IList<int>>();
        SwapBacktrack(nums, 0, result);
        return result;
    }

    private void SwapBacktrack(int[] nums, int idx, IList<IList<int>> result)
    {
        if (idx == nums.Length)
        {
            result.Add(new List<int>(nums));
            return;
        }

        for (int i = idx; i < nums.Length; i++)
        {
            Swap(nums, idx, i);          // choose: place nums[i] at position idx
            SwapBacktrack(nums, idx + 1, result); // explore
            Swap(nums, idx, i);          // un-choose: restore original order
        }
    }

    private void Swap(int[] nums, int i, int j)
    {
        (nums[i], nums[j]) = (nums[j], nums[i]);
    }
}
```

**Time Complexity:** `O(n! * n)` — there are `n!` permutations, and each one takes `O(n)` time to copy into the result (or to build up via the `n` recursive levels). The recursion tree itself has `n!` leaves with total work proportional to `n! * n` across all levels (sum of `n * (n-1) * ... ` placements).
**Space Complexity:** `O(n)` for the recursion stack / `current` list / `used[]` array (auxiliary, excluding the output), plus `O(n! * n)` if we count the space needed to store all permutations in the result.

**Walkthrough:** For `nums = [1,2,3]` with `PermuteUsingUsedArray`: `Backtrack` starts with `current=[]`, `used=[F,F,F]`. Try `i=0`: choose `1` → `used=[T,F,F]`, `current=[1]`, recurse. There try `i=1`: choose `2` → `current=[1,2]`, recurse; try `i=2`: choose `3` → `current=[1,2,3]`, length matches `nums.Length` → record `[1,2,3]`, un-choose back to `current=[1,2]`. No more `i` to try at that level, un-choose back to `current=[1]`; try `i=2`: choose `3` → `current=[1,3]`, recurse, choose `2` → record `[1,3,2]`. Backtracking continues similarly for `i=1` (`2`) and `i=2` (`3`) as the first element, producing `[2,1,3],[2,3,1],[3,1,2],[3,2,1]` — all 6 permutations, matching the stated Output.

**Optimized Approach:**
Generating *all* permutations is inherently `O(n!)` work since that many outputs must be produced — there is no way to asymptotically beat this when all permutations are required. The swapping technique above is already the practical optimization over the `used[]`-array approach: it avoids extra boolean-array bookkeeping and a separate mutable list, operating directly in-place on the array, which reduces constant-factor overhead (no `current.Add`/`RemoveAt` calls, no `used` lookups). No further asymptotic optimization applies when all `n!` permutations are genuinely needed — the true optimization opportunity is not generating all of them at all, which is exactly what the Kth Permutation problem below achieves.

**Logic (Steps):**
1. Recurse with an `idx` parameter marking how many leading positions are already fixed.
2. Base case: when `idx == nums.Length`, the array is a complete permutation — copy it into `result`.
3. Otherwise, loop `i` from `idx` to `n-1`; swap `nums[idx]` with `nums[i]` to place a candidate at position `idx` (choose).
4. Recurse on `idx + 1` to fill the remaining positions (explore).
5. Swap `nums[idx]` and `nums[i]` back (undo/backtrack) so the array is restored before trying the next `i`.

```csharp
// Same as Approach B above — the swapping method is the accepted optimized
// version for "print all permutations" since it avoids auxiliary structures.
public void PermuteOptimized(int[] nums, int idx, IList<IList<int>> result)
{
    if (idx == nums.Length)
    {
        result.Add(new List<int>(nums));
        return;
    }

    for (int i = idx; i < nums.Length; i++)
    {
        (nums[idx], nums[i]) = (nums[i], nums[idx]);
        PermuteOptimized(nums, idx + 1, result);
        (nums[idx], nums[i]) = (nums[i], nums[idx]); // undo
    }
}
```

**Time Complexity:** `O(n! * n)` — same asymptotic bound as the brute force, since all `n!` permutations must still be produced and copied out; the improvement is only in constant factors (fewer allocations/removals).
**Space Complexity:** `O(n)` auxiliary (recursion depth), `O(n! * n)` for the output.

**Walkthrough:** For `nums = [1,2,3]` using swapping (`Output: [[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]`): at `idx=0` we try `i=0,1,2`, swapping each of `1,2,3` into position 0. For `i=0` (no-op swap), recurse to `idx=1` with `[1,2,3]`; there we try `i=1,2`, giving `[1,2,3]` and `[1,3,2]`. Backtrack, swap `idx=0` with `i=1` → `[2,1,3]`, recurse to get `[2,1,3]` and `[2,3,1]`. Backtrack, swap `idx=0` with `i=2` → `[3,2,1]`, recurse to get `[3,2,1]` and `[3,1,2]`. All six permutations are produced exactly once, each swap being undone immediately after its recursive branch returns — matching the stated output.

---

## 2. Find the Kth Permutation Sequence (of numbers 1..N) Without Generating All Permutations

**Problem Statement:**
Given `n` and `k`, return the `k`-th permutation sequence (1-indexed) among all permutations of the numbers `1, 2, ..., n` arranged in lexicographically increasing order.

**Example:**
- Input: `n = 4, k = 9`
- Output: `"2314"`
- Explanation: The lexicographically sorted permutations of `{1,2,3,4}` are: 1234, 1243, 1324, 1342, 1423, 1432, 2134, 2143, **2314**, 2341, ... The 9th one is `"2314"`.

**Brute Force / Backtracking Approach:**
Generate all `n!` permutations using the same choose/explore/un-choose backtracking pattern as Problem 1, collect them all, sort lexicographically (they can be generated in sorted order directly by iterating candidates in ascending order and using a `used[]` array), and simply return the `k`-th one (index `k-1`).

**Logic (Steps):**
1. Maintain `current` (partial permutation as digits) and `used[]` marking placed digits 1..n.
2. At each call, loop `digit` from `1` to `n` in ascending order (guarantees permutations are produced in lexicographic order); skip if `used[digit]`.
3. Choose: mark `used[digit] = true`, append `digit` to `current`. Explore: recurse to fill the next position.
4. Base case: when `current.Count == n`, join the digits into a string and append to `result`.
5. Un-choose: remove the last digit from `current` and reset `used[digit] = false` before trying the next digit.
6. After all `n!` strings are collected in sorted order, return `result[k - 1]` (convert 1-indexed `k` to 0-indexed).

```csharp
using System;
using System.Collections.Generic;

public class KthPermutationBruteForce
{
    public string GetPermutation(int n, int k)
    {
        var result = new List<string>();
        var used = new bool[n + 1];
        var current = new List<int>();
        Backtrack(n, used, current, result);
        return result[k - 1]; // k is 1-indexed
    }

    private void Backtrack(int n, bool[] used, List<int> current, List<string> result)
    {
        if (current.Count == n)
        {
            result.Add(string.Concat(current)); // permutations are added in lexicographic order
            return;
        }

        // iterating digits 1..n in ascending order guarantees lexicographic generation order
        for (int digit = 1; digit <= n; digit++)
        {
            if (used[digit]) continue;

            used[digit] = true;
            current.Add(digit);

            Backtrack(n, used, current, result);

            current.RemoveAt(current.Count - 1);
            used[digit] = false;
        }
    }
}
```

**Time Complexity:** `O(n! * n)` — all `n!` permutations are generated and each costs `O(n)` to build/store; heavily wasteful when `k` is small relative to `n!`.
**Space Complexity:** `O(n! * n)` to store every permutation, plus `O(n)` recursion stack.

**Walkthrough:** For `n=4, k=9` (0-indexed target `k-1=8`): the backtracking generates digits in ascending order, so `current` walks through `1234, 1243, 1324, 1342, 1423, 1432, 2134, 2143, 2314, ...` in exactly that lexicographic order because `digit` is always tried from `1` upward. After all `4! = 24` strings are collected into `result`, `result[8]` (0-indexed, since `k=9`) is `"2314"`, matching the stated Output — but note this required generating and storing all 24 permutations to find the 9th.

**Optimized Approach:**
Use the **factorial number system** trick — no backtracking or permutation generation needed at all. Key insight: fixing the first digit, there are `(n-1)!` permutations for each choice of that first digit (since the remaining `n-1` digits can be arranged in any order). So:
1. Maintain a list of remaining available digits `[1, 2, ..., n]` and convert `k` to 0-indexed (`k = k - 1`).
2. At each step, compute `block = (n - 1)!` (factorial of the remaining count minus one). The index of the digit to pick from the remaining list is `k / block`.
3. Pick that digit, append it to the answer, remove it from the remaining list, update `k = k % block`, decrement the remaining count, and repeat until all digits are placed.

This directly computes the answer in `O(n^2)` time (no sorting, no generating `n!` sequences).

**Logic (Steps):**
1. Precompute `factorial[i] = i!` for `i = 0..n-1`, and build `remaining = [1, 2, ..., n]`.
2. Convert `k` to 0-indexed: `k = k - 1`.
3. Loop `i` from `n` down to `1`: compute `block = factorial[i - 1]` (permutations per leading-digit choice), and `index = k / block` (which remaining digit to pick).
4. Append `remaining[index]` to the result, remove it from `remaining`, and narrow `k = k % block`.
5. Repeat until `remaining` is empty; return the built string.

```csharp
using System.Collections.Generic;
using System.Text;

public class KthPermutationOptimized
{
    public string GetPermutation(int n, int k)
    {
        var factorial = new int[n];
        var remaining = new List<int>();

        factorial[0] = 1;
        remaining.Add(1);
        for (int i = 1; i < n; i++)
        {
            factorial[i] = factorial[i - 1] * i; // factorial[i] = i!
            remaining.Add(i + 1);
        }

        k = k - 1; // convert to 0-indexed
        var result = new StringBuilder();

        for (int i = n; i >= 1; i--)
        {
            int block = factorial[i - 1]; // (i-1)! permutations per leading-digit choice
            int index = k / block;        // which remaining digit to pick
            result.Append(remaining[index]);
            remaining.RemoveAt(index);
            k = k % block;                 // narrow down k within the chosen block
        }

        return result.ToString();
    }
}
```

**Time Complexity:** `O(n^2)` — the outer loop runs `n` times, and `List<int>.RemoveAt(index)` on the remaining-digits list costs `O(n)` in the worst case (shifting elements), giving `O(n) * O(n) = O(n^2)` total. This is a massive improvement over `O(n!)`.
**Space Complexity:** `O(n)` for the `factorial` array, the `remaining` list, and the result string.

**Walkthrough:** Dry run for `n=4, k=9`:
- `remaining = [1,2,3,4]`, `factorial = [1,1,2,6]` (i.e. `0!=1, 1!=1, 2!=2, 3!=6`). Convert `k = 9 - 1 = 8` (0-indexed).
- **i = 4:** `block = factorial[3] = 6`. `index = 8 / 6 = 1` → pick `remaining[1] = 2`. Append `"2"`. `remaining = [1,3,4]`. `k = 8 % 6 = 2`.
- **i = 3:** `block = factorial[2] = 2`. `index = 2 / 2 = 1` → pick `remaining[1] = 3`. Append `"3"`. `remaining = [1,4]`. `k = 2 % 2 = 0`.
- **i = 2:** `block = factorial[1] = 1`. `index = 0 / 1 = 0` → pick `remaining[0] = 1`. Append `"1"`. `remaining = [4]`. `k = 0 % 1 = 0`.
- **i = 1:** `block = factorial[0] = 1`. `index = 0 / 1 = 0` → pick `remaining[0] = 4`. Append `"4"`. `remaining = []`.
- Result: `"2314"` — matches the brute-force list's 9th entry and the stated Output, computed directly without ever generating the other 8 permutations.

---

## 3. Palindrome Partitioning

**Problem Statement:**
Given a string `s`, partition it such that every substring of the partition is a palindrome. Return all possible palindrome partitioning of `s`.

**Example:**
- Input: `s = "aab"`
- Output: `[["a","a","b"], ["aa","b"]]`
- Explanation: `"aab"` can be split into `"a"|"a"|"b"` (all three substrings are palindromes) or `"aa"|"b"` (both substrings are palindromes). No other split keeps every piece a palindrome (e.g. `"a"|"ab"` is invalid since `"ab"` isn't a palindrome).

**Brute Force / Backtracking Approach:**
Use choose/explore/un-choose over the possible "cut points":

**Logic (Steps):**
1. Starting at index `start`, try every possible end index `end` for the next piece `s[start..end]`.
2. Check if `s[start..end]` is a palindrome via `IsPalindrome`; if not, skip this `end`.
3. If it is, "choose" it — add it to the current partition (`path`) — and "explore" by recursing on the remainder starting at `end + 1`.
4. Base case: when `start` reaches the end of the string, the current `path` is a valid full partition — record a copy of it into `result`.
5. "Un-choose" by removing the last added piece from `path` before trying the next `end`.

```csharp
using System.Collections.Generic;

public class PalindromePartitioning
{
    public IList<IList<string>> Partition(string s)
    {
        var result = new List<IList<string>>();
        var path = new List<string>();
        Backtrack(s, 0, path, result);
        return result;
    }

    private void Backtrack(string s, int start, List<string> path, IList<IList<string>> result)
    {
        if (start == s.Length)
        {
            result.Add(new List<string>(path)); // full partition found
            return;
        }

        for (int end = start; end < s.Length; end++)
        {
            if (!IsPalindrome(s, start, end)) continue;

            // choose
            path.Add(s.Substring(start, end - start + 1));

            // explore
            Backtrack(s, end + 1, path, result);

            // un-choose
            path.RemoveAt(path.Count - 1);
        }
    }

    private bool IsPalindrome(string s, int left, int right)
    {
        while (left < right)
        {
            if (s[left] != s[right]) return false;
            left++;
            right--;
        }
        return true;
    }
}
```

**Time Complexity:** `O(n * 2^n)` — there are up to `2^n` ways to partition a string of length `n` (each of the `n-1` gaps between characters can be a cut or not), and for each partition, checking/copying substrings costs up to `O(n)`. The `IsPalindrome` check itself is `O(n)` in the worst case per call, making the true bound closer to `O(n * 2^n)` when counting palindrome checks across the recursion tree.
**Space Complexity:** `O(n)` for recursion depth and the `path` list (auxiliary), plus `O(n * 2^n)` in the worst case to store all output partitions.

**Walkthrough:** For `s = "aab"` (Output: `[["a","a","b"], ["aa","b"]]`): `Backtrack(start=0)` tries `end=0` → `"a"` is a palindrome, choose it, `path=["a"]`, recurse `start=1`; there `end=1` → `"a"` is a palindrome, choose, `path=["a","a"]`, recurse `start=2`; `end=2` → `"b"` palindrome, choose, `path=["a","a","b"]`, recurse `start=3=len` → record `["a","a","b"]`; un-choose back to `path=["a","a"]`, no more `end` at `start=2`, un-choose to `path=["a"]`. Back at `start=1`, no more `end` options. Un-choose to `path=[]`. Back at `start=0`, try `end=1` → `"aa"` is a palindrome, choose, `path=["aa"]`, recurse `start=2`; `end=2` → `"b"` palindrome, choose, `path=["aa","b"]`, recurse `start=3=len` → record `["aa","b"]`. Try `end=2` from `start=0` → `"aab"` not a palindrome, skip. Final `result = [["a","a","b"],["aa","b"]]`, matching the stated Output.

**Optimized Approach:**
Precompute palindrome-validity with a 2D DP table `isPalin[i][j]` = true if `s[i..j]` is a palindrome, filled in `O(n^2)` using the recurrence `isPalin[i][j] = (s[i] == s[j]) && (j - i <= 2 || isPalin[i+1][j-1])`. This turns each palindrome check inside the backtracking loop from `O(n)` into `O(1)`, removing the repeated re-scanning of substrings.

**Logic (Steps):**
1. Build the `isPalin[i, j]` DP table bottom-up: iterate `i` from `n-1` down to `0`, `j` from `i` to `n-1`; set `isPalin[i,j] = true` if `s[i] == s[j]` and either `j - i <= 2` or `isPalin[i+1, j-1]` is already true.
2. Run the same choose/explore/un-choose backtracking as the brute force, but replace the `IsPalindrome` scan with an `O(1)` lookup: `isPalin[start, end]`.
3. If `isPalin[start, end]` is true, choose the substring `s[start..end]`, append to `path`, and recurse on `end + 1`.
4. Base case: when `start == s.Length`, record a copy of `path` into `result`.
5. Un-choose by removing the last piece from `path` before trying the next `end`.

```csharp
using System.Collections.Generic;

public class PalindromePartitioningOptimized
{
    public IList<IList<string>> Partition(string s)
    {
        int n = s.Length;
        var isPalin = new bool[n, n];

        // build palindrome DP table: isPalin[i, j] true iff s[i..j] is a palindrome
        for (int i = n - 1; i >= 0; i--)
        {
            for (int j = i; j < n; j++)
            {
                if (s[i] == s[j] && (j - i <= 2 || isPalin[i + 1, j - 1]))
                {
                    isPalin[i, j] = true;
                }
            }
        }

        var result = new List<IList<string>>();
        var path = new List<string>();
        Backtrack(s, 0, isPalin, path, result);
        return result;
    }

    private void Backtrack(string s, int start, bool[,] isPalin, List<string> path, IList<IList<string>> result)
    {
        if (start == s.Length)
        {
            result.Add(new List<string>(path));
            return;
        }

        for (int end = start; end < s.Length; end++)
        {
            if (!isPalin[start, end]) continue; // O(1) lookup instead of O(n) scan

            path.Add(s.Substring(start, end - start + 1));
            Backtrack(s, end + 1, isPalin, path, result);
            path.RemoveAt(path.Count - 1);
        }
    }
}
```

**Time Complexity:** `O(n^2)` for building the DP table, plus `O(n * 2^n)` for the backtracking enumeration itself (still bound by the number of valid partitions and substring copies) — the DP removes the redundant palindrome-checking cost, leaving substring construction as the dominant remaining factor.
**Space Complexity:** `O(n^2)` for the `isPalin` table, plus `O(n)` recursion depth (auxiliary), plus output space.

**Walkthrough:** For `s = "aab"` (Output: `[["a","a","b"], ["aa","b"]]`): DP table gives `isPalin[0,0]=true ("a")`, `isPalin[1,1]=true ("a")`, `isPalin[0,1]=true ("aa")`, `isPalin[2,2]=true ("b")`, `isPalin[0,2]=false ("aab")`, `isPalin[1,2]=false ("ab")`. Backtracking from `start=0`: try `end=0` → `isPalin[0,0]` true (choose "a"), recurse `start=1`; try `end=1` → `isPalin[1,1]` true (choose "a"), recurse `start=2`; try `end=2` → `isPalin[2,2]` true (choose "b"), recurse `start=3=len` → record `["a","a","b"]`; backtrack all the way. Back at `start=0`, try `end=1` → `isPalin[0,1]` true (choose "aa"), recurse `start=2`; try `end=2` → `isPalin[2,2]` true (choose "b"), recurse `start=3=len` → record `["aa","b"]`. Try `end=2` from `start=0` → `isPalin[0,2]` false, skip. Final result: `[["a","a","b"], ["aa","b"]]`, matching the stated Output — same result as brute force, produced with O(1) palindrome checks.

---

## 4. Word Break

**Problem Statement:**
Given a string `s` and a dictionary of strings `wordDict`, determine if `s` can be segmented into a space-separated sequence of one or more dictionary words (words may be reused any number of times). Optionally, return all such segmentations.

**Example:**
- Input: `s = "leetcode"`, `wordDict = ["leet", "code"]`
- Output: `true` (segmentation returns `["leet code"]`)
- Explanation: `"leetcode"` can be segmented as `"leet"` + `"code"`, both of which are present in the dictionary.

**Brute Force / Backtracking Approach:**
Same choose/explore/un-choose pattern as palindrome partitioning, but the "valid piece" check is dictionary membership instead of a palindrome check:

**Logic (Steps):**
1. From index `start`, try every `end` such that `s[start..end)` is a word present in `wordSet`.
2. Base case: if `start == s.Length`, the whole string has been segmented — return `true` (or, for `WordBreakAll`, record a copy of `path`).
3. For each valid dictionary word found, "choose" it (append to `path`, or just recurse in the boolean variant), "explore" the remainder from `end`.
4. In the boolean version, memoize the result for `start` in a `Dictionary<int,bool>` so each starting index is resolved only once.
5. "Un-choose" (backtrack): in `WordBreakAll`, remove the last piece from `path` before trying the next `end`; in the boolean version the piece is simply discarded and the loop moves to the next `end`.

```csharp
using System.Collections.Generic;

public class WordBreakBacktracking
{
    public bool WordBreak(string s, IList<string> wordDict)
    {
        var wordSet = new HashSet<string>(wordDict);
        return Backtrack(s, 0, wordSet, new Dictionary<int, bool>());
    }

    private bool Backtrack(string s, int start, HashSet<string> wordSet, Dictionary<int, bool> memo)
    {
        if (start == s.Length) return true;
        if (memo.TryGetValue(start, out bool cached)) return cached;

        for (int end = start + 1; end <= s.Length; end++)
        {
            string piece = s.Substring(start, end - start);
            if (!wordSet.Contains(piece)) continue; // not a valid dictionary word, skip

            // choose piece, explore remainder
            if (Backtrack(s, end, wordSet, memo))
            {
                memo[start] = true;
                return true; // found a valid full segmentation
            }
            // implicit un-choose: piece is simply discarded, loop tries the next 'end'
        }

        memo[start] = false;
        return false;
    }

    // Variant: return ALL segmentations (pure backtracking, choose/explore/un-choose)
    public IList<string> WordBreakAll(string s, IList<string> wordDict)
    {
        var wordSet = new HashSet<string>(wordDict);
        var result = new List<string>();
        var path = new List<string>();
        BacktrackAll(s, 0, wordSet, path, result);
        return result;
    }

    private void BacktrackAll(string s, int start, HashSet<string> wordSet, List<string> path, List<string> result)
    {
        if (start == s.Length)
        {
            result.Add(string.Join(" ", path)); // full segmentation found
            return;
        }

        for (int end = start + 1; end <= s.Length; end++)
        {
            string piece = s.Substring(start, end - start);
            if (!wordSet.Contains(piece)) continue;

            path.Add(piece);              // choose
            BacktrackAll(s, end, wordSet, path, result); // explore
            path.RemoveAt(path.Count - 1); // un-choose
        }
    }
}
```

**Time Complexity:** Without memoization, `O(2^n)` in the worst case (every position could be a cut point, forming an exponential recursion tree), each level doing up to `O(n)` substring work → `O(n * 2^n)`. With memoization (as in `WordBreak` above), it drops to `O(n^2)` since each `start` index is resolved once, and each resolution scans up to `n` end positions doing `O(n)` substring extraction.
**Space Complexity:** `O(n)` recursion depth, `O(n)` memo table (when used), `O(n)` for the dictionary `HashSet`, plus output space for the "return all" variant which can be exponential in the worst case.

**Walkthrough:** For `s = "leetcode"`, `wordDict = ["leet","code"]` (Output: `true`, segmentation `"leet code"`): `Backtrack(start=0)` tries `end=1..8`; `end=4` gives piece `"leet"`, which is in `wordSet` → recurse `Backtrack(start=4)`. There, `end=5..8`; `end=8` gives piece `"code"`, in `wordSet` → recurse `Backtrack(start=8)`, and since `start == s.Length`, return `true` immediately. This bubbles back up as `true` through both calls, so `WordBreak` returns `true`. For `WordBreakAll`, the same path builds `path = ["leet", "code"]`, joined into `"leet code"`, matching the stated Output.

**Optimized Approach:**
Convert to bottom-up DP: let `dp[i]` = true if `s[0..i)` (the prefix of length `i`) can be segmented using dictionary words. `dp[0] = true` (empty prefix). For each `i` from `1` to `n`, check all `j < i`: if `dp[j]` is true and `s[j..i)` is in the dictionary, set `dp[i] = true`. This avoids recursion entirely and is a clean iterative `O(n^2)` solution (with `O(1)` average dictionary lookups via a `HashSet`).

**Logic (Steps):**
1. Build `wordSet` from `wordDict` and allocate `dp` of size `n+1`, all `false`; set `dp[0] = true` (empty prefix is trivially breakable).
2. Loop `i` from `1` to `n` (end of the prefix being tested).
3. For each `i`, loop `j` from `0` to `i-1`: if `dp[j]` is true and `s[j..i)` is in `wordSet`, set `dp[i] = true` and break out of the inner loop (no need to check other splits for this `i`).
4. After both loops finish, return `dp[n]` — whether the entire string can be segmented.

```csharp
using System.Collections.Generic;

public class WordBreakOptimized
{
    public bool WordBreak(string s, IList<string> wordDict)
    {
        var wordSet = new HashSet<string>(wordDict);
        int n = s.Length;
        var dp = new bool[n + 1];
        dp[0] = true; // empty prefix is trivially "breakable"

        for (int i = 1; i <= n; i++)
        {
            for (int j = 0; j < i; j++)
            {
                if (dp[j] && wordSet.Contains(s.Substring(j, i - j)))
                {
                    dp[i] = true;
                    break; // no need to check further splits for this i
                }
            }
        }

        return dp[n];
    }
}
```

**Time Complexity:** `O(n^2)` — the nested loop over `i` and `j` is `O(n^2)`, and each `HashSet` lookup / substring extraction is `O(n)` in the worst case for the substring, giving `O(n^3)` if substring cost is counted strictly, though in practice with average word lengths bounded, it behaves close to `O(n^2)`. This is far better than the unmemoized exponential backtracking.
**Space Complexity:** `O(n)` for the `dp` array, plus `O(sum of word lengths)` for the `wordSet`.

**Walkthrough:** For `s = "leetcode"`, `wordDict = ["leet","code"]` (Output: `true`): `dp[0] = true`. Checking `i=4`: `j=0`, `dp[0]=true` and `s[0..4)="leet"` is in `wordSet` → `dp[4] = true`. Checking `i=8`: `j` ranges `0..7`; at `j=4`, `dp[4]=true` and `s[4..8)="code"` is in `wordSet` → `dp[8] = true`. Since `dp[n=8] = true`, the function returns `true`, matching the stated Output — computed iteratively without any recursion, corresponding to the segmentation `"leet" + "code"` found by the backtracking approach.

---

## 5. N-Queens Problem

**Problem Statement:**
Place `N` queens on an `N x N` chessboard such that no two queens attack each other (no shared row, column, or diagonal). Return all distinct board configurations that satisfy this constraint.

**Example:**
- Input: `n = 4`
- Output:
  ```
  [
    [".Q..", "...Q", "Q...", "..Q."],
    ["..Q.", "Q...", "...Q", ".Q.."]
  ]
  ```
- Explanation: For a `4x4` board there are exactly 2 valid arrangements of 4 non-attacking queens. In the first, queens sit at columns `(1, 3, 0, 2)` for rows `(0, 1, 2, 3)` respectively, and no two share a row, column, or diagonal.

**Brute Force / Backtracking Approach:**
Place one queen per row, choosing a column for each row and checking that it doesn't conflict with any previously placed queen:

**Logic (Steps):**
1. Recurse row by row (`row = 0` to `n-1`). For the current `row`, try every `col` from `0` to `n-1`.
2. "Choose": before placing, verify the cell is safe by scanning all previously placed queens (rows `0..row-1`) via `IsSafe` — check same column and both diagonals (`O(row)` scan, i.e. `O(n)` worst case).
3. If safe, place the queen (mark `board[row][col] = 'Q'`), "explore" by recursing to `row + 1`.
4. Base case: when `row == n`, all `n` queens are placed validly — record a snapshot of the board configuration.
5. "Un-choose": reset `board[row][col] = '.'` before trying the next `col`.

```csharp
using System.Collections.Generic;

public class NQueensBacktracking
{
    public IList<IList<string>> SolveNQueens(int n)
    {
        var result = new List<IList<string>>();
        var board = new char[n][];
        for (int i = 0; i < n; i++)
        {
            board[i] = new char[n];
            for (int j = 0; j < n; j++) board[i][j] = '.';
        }

        Backtrack(board, 0, n, result);
        return result;
    }

    private void Backtrack(char[][] board, int row, int n, IList<IList<string>> result)
    {
        if (row == n)
        {
            result.Add(BuildSnapshot(board, n)); // valid full placement found
            return;
        }

        for (int col = 0; col < n; col++)
        {
            if (!IsSafe(board, row, col, n)) continue;

            // choose
            board[row][col] = 'Q';

            // explore
            Backtrack(board, row + 1, n, result);

            // un-choose
            board[row][col] = '.';
        }
    }

    private bool IsSafe(char[][] board, int row, int col, int n)
    {
        // check column above
        for (int r = 0; r < row; r++)
            if (board[r][col] == 'Q') return false;

        // check upper-left diagonal
        for (int r = row - 1, c = col - 1; r >= 0 && c >= 0; r--, c--)
            if (board[r][c] == 'Q') return false;

        // check upper-right diagonal
        for (int r = row - 1, c = col + 1; r >= 0 && c < n; r--, c++)
            if (board[r][c] == 'Q') return false;

        return true;
    }

    private List<string> BuildSnapshot(char[][] board, int n)
    {
        var snapshot = new List<string>();
        for (int i = 0; i < n; i++) snapshot.Add(new string(board[i]));
        return snapshot;
    }
}
```

**Time Complexity:** Roughly `O(n!)` — the first row has `n` column choices, the second row has at most `n-1` remaining "safe-ish" choices (pruned further by diagonal conflicts), and so on, giving a search tree bounded above by `n!` branches; each `IsSafe` check costs `O(n)`, giving an overall bound around `O(n! * n)`.
**Space Complexity:** `O(n^2)` for the board, `O(n)` recursion depth (auxiliary), plus `O(solutions * n)` to store all output configurations.

**Walkthrough:** For `n=4` (Output has 2 solutions): `Backtrack(row=0)` tries `col=0`, `IsSafe` passes trivially (empty board), place `Q` at `(0,0)`, recurse `row=1`. At `row=1`, `col=0` fails (`IsSafe` sees column conflict), `col=1` fails (diagonal conflict with `(0,0)`), `col=2` passes, place `Q` at `(1,2)`, recurse `row=2`. This branch eventually finds no safe column at some deeper row, so it backtracks (removing queens and trying the next `col`) until it reaches a fully valid placement — e.g. queens at columns `(1,3,0,2)` for rows `(0,1,2,3)`, recorded as `[".Q..", "...Q", "Q...", "..Q."]`. Continued backtracking across all branches eventually also finds columns `(2,0,3,1)`, giving `["..Q.", "Q...", "...Q", ".Q.."]`. Final `result` has exactly these 2 configurations, matching the stated Output.

**Optimized Approach:**
Avoid the `O(n)` rescanning in `IsSafe` by maintaining three boolean arrays (or bitmasks) that track, in `O(1)`, whether a column or diagonal is currently under attack:
- `colUsed[col]` — true if some queen already occupies this column.
- `diag1Used[row + col]` — true if some queen occupies this "`\`"-diagonal (constant `row + col`).
- `diag2Used[row - col + n - 1]` — true if some queen occupies this "`/`"-diagonal (constant `row - col`, offset by `n-1` to keep the index non-negative).

Checking safety becomes three array lookups instead of scanning up to `3n` cells, and placing/removing a queen becomes three flag toggles.

**Logic (Steps):**
1. Allocate `colUsed[n]`, `diag1Used[2n-1]` (indexed by `row+col`), `diag2Used[2n-1]` (indexed by `row-col+n-1`), all initially `false`.
2. Recurse row by row; for the current `row`, loop `col` from `0` to `n-1`, computing `d1 = row+col` and `d2 = row-col+n-1`.
3. Base case: if `colUsed[col] || diag1Used[d1] || diag2Used[d2]` is true, skip (`O(1)` conflict check); otherwise the cell is safe.
4. Choose: place `board[row][col]='Q'` and set `colUsed[col]=diag1Used[d1]=diag2Used[d2]=true`; explore by recursing to `row+1`.
5. When `row == n`, record a snapshot of the board.
6. Un-choose: reset `board[row][col]='.'` and clear the three flags before trying the next `col`.

```csharp
using System.Collections.Generic;

public class NQueensOptimized
{
    public IList<IList<string>> SolveNQueens(int n)
    {
        var result = new List<IList<string>>();
        var board = new char[n][];
        for (int i = 0; i < n; i++)
        {
            board[i] = new char[n];
            for (int j = 0; j < n; j++) board[i][j] = '.';
        }

        var colUsed = new bool[n];
        var diag1Used = new bool[2 * n - 1]; // indexed by row + col
        var diag2Used = new bool[2 * n - 1]; // indexed by row - col + (n - 1)

        Backtrack(board, 0, n, colUsed, diag1Used, diag2Used, result);
        return result;
    }

    private void Backtrack(char[][] board, int row, int n,
        bool[] colUsed, bool[] diag1Used, bool[] diag2Used, IList<IList<string>> result)
    {
        if (row == n)
        {
            var snapshot = new List<string>();
            for (int i = 0; i < n; i++) snapshot.Add(new string(board[i]));
            result.Add(snapshot);
            return;
        }

        for (int col = 0; col < n; col++)
        {
            int d1 = row + col;
            int d2 = row - col + n - 1;

            if (colUsed[col] || diag1Used[d1] || diag2Used[d2]) continue; // O(1) safety check

            // choose
            board[row][col] = 'Q';
            colUsed[col] = diag1Used[d1] = diag2Used[d2] = true;

            // explore
            Backtrack(board, row + 1, n, colUsed, diag1Used, diag2Used, result);

            // un-choose
            board[row][col] = '.';
            colUsed[col] = diag1Used[d1] = diag2Used[d2] = false;
        }
    }
}
```

**Time Complexity:** Still roughly `O(n!)` for the search tree itself (that many placements must be explored/pruned in the worst case), but each safety check and placement/removal is now `O(1)` instead of `O(n)`, so the overall runtime drops from `~O(n! * n)` to `~O(n!)`, a meaningful constant-factor (and asymptotic-in-the-per-node-cost) improvement.
**Space Complexity:** `O(n^2)` for the board, `O(n)` for the three tracking arrays (auxiliary), `O(n)` recursion depth, plus output storage.

**Walkthrough:** Dry run for `n=4` (Output has 2 solutions), showing a pruned branch:
- `row=0`: try `col=0` → safe (nothing placed yet). Place queen at `(0,0)`. Mark `colUsed[0]=true`, `diag1Used[0+0=0]=true`, `diag2Used[0-0+3=3]=true`.
- `row=1`: try `col=0` → `colUsed[0]` is true → **pruned** (same column as row 0's queen). Try `col=1` → `diag2Used[1-1+3=3]` is true (matches row 0's diag2 value 3, since `(0,0)` and `(1,1)` share `row-col=0`) → **pruned**. Try `col=2` → `colUsed[2]=false`, `diag1Used[1+2=3]=false`, `diag2Used[1-2+3=2]=false` → safe. Place queen at `(1,2)`.
- `row=2`: `col=0` → `colUsed[0]` true → pruned. `col=1` → all three flags clear → safe, place `(2,1)`. But then `row=3` finds no safe column for any of `col=0,1,2,3` (all conflict via column or diagonal with the three placed queens), so this whole branch is pruned and the recursion backtracks up to `row=2` to try `col=3` instead, then further up to `row=1`'s remaining columns, and so on.
- This systematic prune-and-backtrack process is exactly how the search explores the tree until it finds the two valid solutions: queens at columns `(1,3,0,2)` and `(2,0,3,1)` for rows `(0,1,2,3)`, matching the stated Output of 2 board configurations.
