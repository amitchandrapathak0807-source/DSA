# Dynamic Programming — DP on LIS (Longest Increasing Subsequence)

## 1. Longest Increasing Subsequence (length only)

**Problem Statement:** Given an array of integers, find the length of the longest strictly increasing subsequence (LIS). A subsequence need not be contiguous but must preserve the relative order of elements.

**Example:**
- Input: `arr = [10, 9, 2, 5, 3, 7, 101, 18]`
- Output: `4`
- Explanation: One valid LIS is `[2, 3, 7, 101]` (or `[2, 3, 7, 18]`), which has length 4.

**Brute Force / DP Approach:** Define `dp[i]` as the length of the LIS that ends exactly at index `i`. For every index `i`, look back at all indices `j < i`; if `arr[j] < arr[i]`, then index `i` can extend the subsequence ending at `j`, so `dp[i] = max(dp[i], dp[j] + 1)`. The base case is `dp[i] = 1` for every index (each element is an LIS of length 1 by itself). The answer is the maximum value in `dp`.

```csharp
public int LengthOfLIS(int[] arr)
{
    int n = arr.Length;
    if (n == 0) return 0;

    int[] dp = new int[n];
    Array.Fill(dp, 1); // every element is an LIS of length 1 by itself

    int maxLen = 1;
    for (int i = 1; i < n; i++)
    {
        for (int j = 0; j < i; j++)
        {
            if (arr[j] < arr[i])
            {
                dp[i] = Math.Max(dp[i], dp[j] + 1);
            }
        }
        maxLen = Math.Max(maxLen, dp[i]);
    }

    return maxLen;
}
```

Time Complexity: O(n^2), Space Complexity: O(n).

**Optimized Approach:** See Problem 3 for the O(n log n) `tails`-array technique, which applies directly to this length-only version.

**Explanation:** For `arr = [10, 9, 2, 5, 3, 7, 101, 18]`:
- `dp = [1,1,1,1,1,1,1,1]` initially.
- `i=1 (9)`: no `j` with `arr[j]<9` before it except none valid (10 is not < 9) → `dp[1]=1`.
- `i=2 (2)`: nothing smaller before it → `dp[2]=1`.
- `i=3 (5)`: `arr[2]=2<5` → `dp[3]=dp[2]+1=2`.
- `i=4 (3)`: `arr[2]=2<3` → `dp[4]=dp[2]+1=2`.
- `i=5 (7)`: `arr[2]=2,arr[3]=5,arr[4]=3` all `<7` → best is `dp[3]+1=3` → `dp[5]=3`.
- `i=6 (101)`: best predecessor is `dp[5]=3` → `dp[6]=4`.
- `i=7 (18)`: best predecessor is `dp[5]=3` (7<18) → `dp[7]=4`.
- Final `dp = [1,1,1,2,2,3,4,4]`, max is `4`.

## 2. Print the Longest Increasing Subsequence

**Problem Statement:** Given an array of integers, return the actual longest strictly increasing subsequence itself (not just its length). If multiple LIS of the same maximum length exist, returning any one of them is acceptable.

**Example:**
- Input: `arr = [10, 9, 2, 5, 3, 7, 101, 18]`
- Output: `[2, 3, 7, 101]`
- Explanation: This is one valid subsequence of length 4 that is strictly increasing and of maximum possible length.

**Brute Force / DP Approach:** Use the same `dp[i]` definition as Problem 1, but additionally maintain a `parent[i]` (or `prevIndex[i]`) array that records which earlier index was used to extend the LIS ending at `i`. After filling `dp`, find the index with the maximum `dp` value, then walk backward through `parent` pointers to reconstruct the sequence, and finally reverse it.

```csharp
public List<int> PrintLIS(int[] arr)
{
    int n = arr.Length;
    if (n == 0) return new List<int>();

    int[] dp = new int[n];
    int[] parent = new int[n];
    Array.Fill(dp, 1);
    for (int i = 0; i < n; i++) parent[i] = i; // self means "no predecessor"

    int maxLen = 1, lastIndex = 0;
    for (int i = 1; i < n; i++)
    {
        for (int j = 0; j < i; j++)
        {
            if (arr[j] < arr[i] && dp[j] + 1 > dp[i])
            {
                dp[i] = dp[j] + 1;
                parent[i] = j;
            }
        }
        if (dp[i] > maxLen)
        {
            maxLen = dp[i];
            lastIndex = i;
        }
    }

    // Reconstruct by walking parent pointers backward from lastIndex
    List<int> lis = new List<int>();
    int cur = lastIndex;
    while (true)
    {
        lis.Add(arr[cur]);
        if (parent[cur] == cur) break; // reached the start of the chain
        cur = parent[cur];
    }
    lis.Reverse();
    return lis;
}
```

Time Complexity: O(n^2), Space Complexity: O(n) for `dp` and `parent` arrays.

**Optimized Approach:** Reconstruction is naturally suited to the O(n^2) `parent`-pointer method shown above. The O(n log n) technique (Problem 3) can also be adapted to print the sequence, but it requires extra bookkeeping (a separate parent array tracked alongside the `tails` array) since `tails` itself does not represent an actual subsequence — see the note in Problem 3.

**Explanation:** Using the dry run from Problem 1, `dp = [1,1,1,2,2,3,4,4]`. Tracking `parent` alongside:
- `parent[3]=2` (5 extends from 2), `parent[4]=2` (3 extends from 2), `parent[5]=4` (7 extends from 3, index 4), `parent[6]=5` (101 extends from 7), `parent[7]=5` (18 extends from 7).
- `maxLen=4` first achieved at `lastIndex=6` (value 101).
- Walk back: `6→arr=101`, `parent[6]=5→arr=7`, `parent[5]=4→arr=3`, `parent[4]=2→arr=2`, `parent[2]=2` (self, stop).
- Collected in reverse-add order: `[101,7,3,2]`, reverse → `[2,3,7,101]`.

## 3. Longest Increasing Subsequence Using Binary Search (O(n log n) approach)

**Problem Statement:** Given an array of integers, find the length of the longest strictly increasing subsequence, but do so more efficiently than the O(n^2) DP, using an O(n log n) technique based on binary search (patience sorting style).

**Example:**
- Input: `arr = [10, 9, 2, 5, 3, 7, 101, 18]`
- Output: `4`
- Explanation: Same answer as Problem 1, but computed in O(n log n) time instead of O(n^2).

**Brute Force / DP Approach:** The baseline is identical to Problem 1's O(n^2) `dp[i]` formulation (each `dp[i]` = LIS length ending at `i`, computed by scanning all earlier smaller elements). It is repeated here for comparison before showing the optimized version.

```csharp
public int LengthOfLISBruteForce(int[] arr)
{
    int n = arr.Length;
    if (n == 0) return 0;

    int[] dp = new int[n];
    Array.Fill(dp, 1);

    int maxLen = 1;
    for (int i = 1; i < n; i++)
    {
        for (int j = 0; j < i; j++)
        {
            if (arr[j] < arr[i])
                dp[i] = Math.Max(dp[i], dp[j] + 1);
        }
        maxLen = Math.Max(maxLen, dp[i]);
    }
    return maxLen;
}
```

Time Complexity: O(n^2), Space Complexity: O(n).

**Optimized Approach:** Maintain a `tails` array (a `List<int>`) where `tails[k]` holds the *smallest possible tail value* of any strictly increasing subsequence of length `k + 1` found so far. For each element `x` in the array:
- If `x` is greater than every element currently in `tails`, append it (it extends the longest subsequence found so far by one).
- Otherwise, use binary search (lower bound: first position where `tails[pos] >= x`) to find the leftmost tail that is `>= x`, and overwrite it with `x`. This keeps future extensions as "easy" as possible by minimizing tail values, without changing the *length* represented by `tails`.

The final length of the LIS is simply the length of the `tails` array. Note that `tails` is NOT an actual increasing subsequence found in the array — it is only used to track lengths efficiently.

```csharp
public int LengthOfLISOptimized(int[] arr)
{
    List<int> tails = new List<int>();

    foreach (int x in arr)
    {
        int pos = LowerBound(tails, x);
        if (pos == tails.Count)
            tails.Add(x);           // x extends the longest subsequence so far
        else
            tails[pos] = x;         // x replaces the first tail >= x
    }

    return tails.Count;
}

// Returns the index of the first element in tails that is >= target,
// or tails.Count if no such element exists (classic lower_bound).
private int LowerBound(List<int> tails, int target)
{
    int lo = 0, hi = tails.Count;
    while (lo < hi)
    {
        int mid = lo + (hi - lo) / 2;
        if (tails[mid] < target)
            lo = mid + 1;
        else
            hi = mid;
    }
    return lo;
}
```

Time Complexity: O(n log n) — for each of the `n` elements, a binary search over `tails` (at most size `n`) takes O(log n). Space Complexity: O(n) for the `tails` array in the worst case (fully increasing input).

**Explanation:** Dry run of the `tails` array technique on `arr = [10, 9, 2, 5, 3, 7, 101, 18]`:

| Step | x   | LowerBound result | Action                  | tails after step   |
|------|-----|--------------------|--------------------------|---------------------|
| 1    | 10  | pos = 0 (empty)     | append                   | `[10]`              |
| 2    | 9   | pos = 0 (10 >= 9)   | replace tails[0]         | `[9]`               |
| 3    | 2   | pos = 0 (9 >= 2)    | replace tails[0]         | `[2]`               |
| 4    | 5   | pos = 1 (end)       | append                   | `[2, 5]`            |
| 5    | 3   | pos = 1 (5 >= 3)    | replace tails[1]         | `[2, 3]`            |
| 6    | 7   | pos = 2 (end)       | append                   | `[2, 3, 7]`         |
| 7    | 101 | pos = 3 (end)       | append                   | `[2, 3, 7, 101]`    |
| 8    | 18  | pos = 3 (101 >= 18) | replace tails[3]         | `[2, 3, 7, 18]`     |

Final `tails.Count = 4`, matching the expected LIS length. Note that the final `tails` array `[2, 3, 7, 18]` happens to look like a valid LIS here, but this is coincidental — in general `tails` only tracks the smallest tail per length, not an actual subsequence that occurred in order. **Reconstructing the real LIS from this technique requires extra bookkeeping**: alongside `tails`, you would maintain a `parent[i]` array (predecessor index for each array element, recorded at the time it's placed) and an `indexOfTail[k]` array (which original array index currently holds the tail for length `k`). After processing, you'd walk backward through `parent` starting from `indexOfTail[tails.Count - 1]`, just like in Problem 2, but layered on top of the binary-search bookkeeping — considerably more involved than the straightforward O(n^2) version.

## 4. Largest Divisible Subset

**Problem Statement:** Given a set of distinct positive integers, find the largest subset such that every pair `(a, b)` in the subset satisfies either `a % b == 0` or `b % a == 0`.

**Example:**
- Input: `arr = [1, 2, 4, 8]`
- Output: `[1, 2, 4, 8]`
- Explanation: Every pair divides evenly (2%1=0, 4%2=0, 8%4=0, 4%1=0, 8%1=0, 8%2=0), so the whole array is a valid divisible subset.

**Brute Force / DP Approach:** This is LIS in disguise. First sort the array ascending — this guarantees that if `arr[j] < arr[i]` and `arr[i] % arr[j] == 0`, then `arr[j]` can safely precede `arr[i]` in the chain (divisibility respects the sorted order transitively). Then apply the same `dp[i]` = "length of the best chain ending at `i`" recurrence as LIS, except the condition to extend is `arr[i] % arr[j] == 0` instead of `arr[j] < arr[i]`.

```csharp
public List<int> LargestDivisibleSubsetBruteForce(int[] arr)
{
    int n = arr.Length;
    if (n == 0) return new List<int>();

    int[] sorted = (int[])arr.Clone();
    Array.Sort(sorted);

    int[] dp = new int[n];
    int[] parent = new int[n];
    Array.Fill(dp, 1);
    for (int i = 0; i < n; i++) parent[i] = i;

    int maxLen = 1, lastIndex = 0;
    for (int i = 1; i < n; i++)
    {
        for (int j = 0; j < i; j++)
        {
            if (sorted[i] % sorted[j] == 0 && dp[j] + 1 > dp[i])
            {
                dp[i] = dp[j] + 1;
                parent[i] = j;
            }
        }
        if (dp[i] > maxLen)
        {
            maxLen = dp[i];
            lastIndex = i;
        }
    }

    List<int> result = new List<int>();
    int cur = lastIndex;
    while (true)
    {
        result.Add(sorted[cur]);
        if (parent[cur] == cur) break;
        cur = parent[cur];
    }
    result.Reverse();
    return result;
}
```

Time Complexity: O(n^2) for the DP double loop (plus O(n log n) for sorting, dominated by n^2). Space Complexity: O(n) for `dp`, `parent`, and the sorted copy.

**Optimized Approach:** The core logic cannot be reduced below O(n^2) in general because divisibility (unlike simple `<` comparison) does not admit the same binary-search trick used for plain LIS — the "tails" idea relies on a total order where a single scalar comparison determines extendability, but divisibility chains still need pairwise checks. The main optimization applied here is the same sort-then-DP-with-parent-pointers structure as the brute force; the code above already represents the practical optimum for this variant.

```csharp
// Same implementation as above; sorting + O(n^2) DP with parent pointers
// is the standard optimized solution for Largest Divisible Subset.
public List<int> LargestDivisibleSubset(int[] arr) => LargestDivisibleSubsetBruteForce(arr);
```

Time Complexity: O(n^2), Space Complexity: O(n).

**Explanation:** For `arr = [1, 2, 4, 8]` (already sorted):
- `dp = [1,1,1,1]` initially, `parent[i]=i`.
- `i=1 (2)`: `2%1==0` → `dp[1]=2`, `parent[1]=0`.
- `i=2 (4)`: `4%1==0` gives `dp=2`; `4%2==0` gives `dp=dp[1]+1=3` (better) → `dp[2]=3`, `parent[2]=1`.
- `i=3 (8)`: `8%1==0`→2, `8%2==0`→`dp[1]+1=3`, `8%4==0`→`dp[2]+1=4` (best) → `dp[3]=4`, `parent[3]=2`.
- `maxLen=4` at `lastIndex=3`. Walk back: `8→4→2→1`. Reverse → `[1,2,4,8]`.

## 5. Longest String Chain

**Problem Statement:** Given a list of words, find the length of the longest possible "word chain" — a sequence of words `w1, w2, ..., wk` such that `w1` is a predecessor of `w2`, `w2` is a predecessor of `w3`, and so on. A word `A` is a predecessor of word `B` if you can insert exactly one letter anywhere in `A` (without reordering the other letters) to make it equal to `B`.

**Example:**
- Input: `words = ["a", "b", "ba", "bca", "bda", "bdca"]`
- Output: `4`
- Explanation: One longest chain is `"a" -> "ba" -> "bda" -> "bdca"`, which has length 4.

**Brute Force / DP Approach:** This is LIS where the "increasing" relation is "is a predecessor of" and the ordering key is word length. First sort the words by length ascending — this guarantees any valid predecessor of a word appears earlier in the sorted order. Then apply the LIS-style DP: `dp[i]` = length of the longest chain ending at word `i`; for each `j < i`, if `words[j]` is a predecessor of `words[i]`, then `dp[i] = max(dp[i], dp[j] + 1)`.

```csharp
public int LongestStrChainBruteForce(string[] words)
{
    int n = words.Length;
    if (n == 0) return 0;

    string[] sorted = (string[])words.Clone();
    Array.Sort(sorted, (a, b) => a.Length - b.Length);

    int[] dp = new int[n];
    Array.Fill(dp, 1);

    int maxLen = 1;
    for (int i = 1; i < n; i++)
    {
        for (int j = 0; j < i; j++)
        {
            if (IsPredecessor(sorted[j], sorted[i]))
                dp[i] = Math.Max(dp[i], dp[j] + 1);
        }
        maxLen = Math.Max(maxLen, dp[i]);
    }
    return maxLen;
}

// Returns true if 'shorter' becomes 'longer' by inserting exactly one character.
private bool IsPredecessor(string shorter, string longer)
{
    if (longer.Length - shorter.Length != 1) return false;

    int i = 0, j = 0;
    bool usedSkip = false;
    while (i < shorter.Length && j < longer.Length)
    {
        if (shorter[i] == longer[j])
        {
            i++; j++;
        }
        else
        {
            if (usedSkip) return false;
            usedSkip = true;
            j++;
        }
    }
    return true; // remaining characters (if any) are covered by the final skip
}
```

Time Complexity: O(n^2 * L) where `L` is the average word length (each predecessor check is O(L)), plus O(n log n) for sorting. Space Complexity: O(n).

**Optimized Approach:** The standard optimization avoids the O(n^2) pairwise scan by using a hash map keyed on each word, storing `dp[word]` = longest chain ending at that word. For each word (processed in increasing length order), try removing each single character to generate all possible predecessors (`O(L)` candidates, each an `O(L)`-length string), and look each one up in the map in O(1) (amortized) instead of scanning all previous words. This reduces the pairwise O(n) scan to an O(L) predecessor-generation step per word.

```csharp
public int LongestStrChain(string[] words)
{
    string[] sorted = (string[])words.Clone();
    Array.Sort(sorted, (a, b) => a.Length - b.Length);

    Dictionary<string, int> dp = new Dictionary<string, int>();
    int maxLen = 1;

    foreach (string word in sorted)
    {
        int best = 1; // chain of length 1 by itself
        for (int i = 0; i < word.Length; i++)
        {
            string predecessor = word.Remove(i, 1); // delete char at position i
            if (dp.TryGetValue(predecessor, out int predLen))
                best = Math.Max(best, predLen + 1);
        }
        dp[word] = best;
        maxLen = Math.Max(maxLen, best);
    }

    return maxLen;
}
```

Time Complexity: O(n * L^2) — for each of `n` words, generating `L` candidate predecessors and building each one costs O(L), so O(L^2) per word (dictionary operations are O(L) for hashing the string); overall better in practice than O(n^2 * L) when `n >> L`. Space Complexity: O(n * L) for the dictionary of words to chain lengths.

**Explanation:** This extends the base LIS DP by (1) sorting by length instead of by value, and (2) replacing the `<` comparison with the "is predecessor" check, exactly mirroring how Problem 4 replaced `<` with divisibility.

## 6. Longest Bitonic Subsequence

**Problem Statement:** Given an array of integers, find the length of the longest subsequence that first strictly increases, then strictly decreases. Either the increasing or decreasing part may be empty-length-1 at the peak, but a valid bitonic sequence must have at least one increasing step or one decreasing step (i.e., it is not just a flat single element unless the array has only one element).

**Example:**
- Input: `arr = [1, 11, 2, 10, 4, 5, 2, 1]`
- Output: `6`
- Explanation: One longest bitonic subsequence is `[1, 2, 10, 4, 2, 1]` — it increases `1 -> 2 -> 10` then decreases `10 -> 4 -> 2 -> 1`, total length 6.

**Brute Force / DP Approach:** Compute two auxiliary arrays using the standard LIS DP: `lis[i]` = length of the longest strictly increasing subsequence ending at index `i` (computed left-to-right, exactly as in Problem 1), and `lds[i]` = length of the longest strictly decreasing subsequence starting at index `i` (computed right-to-left, which is the same LIS DP mirrored). For every index `i` treated as the "peak," the bitonic length through that peak is `lis[i] + lds[i] - 1` (subtracting 1 because the peak element is counted in both arrays). The answer is the maximum over all peaks.

```csharp
public int LongestBitonicSubsequence(int[] arr)
{
    int n = arr.Length;
    if (n == 0) return 0;

    // lis[i] = length of longest strictly increasing subsequence ending at i
    int[] lis = new int[n];
    Array.Fill(lis, 1);
    for (int i = 1; i < n; i++)
    {
        for (int j = 0; j < i; j++)
        {
            if (arr[j] < arr[i])
                lis[i] = Math.Max(lis[i], lis[j] + 1);
        }
    }

    // lds[i] = length of longest strictly decreasing subsequence starting at i
    int[] lds = new int[n];
    Array.Fill(lds, 1);
    for (int i = n - 2; i >= 0; i--)
    {
        for (int j = i + 1; j < n; j++)
        {
            if (arr[j] < arr[i])
                lds[i] = Math.Max(lds[i], lds[j] + 1);
        }
    }

    int maxLen = 0;
    for (int i = 0; i < n; i++)
    {
        // require both a rising and falling part to avoid trivial single-direction cases
        if (lis[i] > 1 && lds[i] > 1)
            maxLen = Math.Max(maxLen, lis[i] + lds[i] - 1);
    }

    // fallback: if no index had both slopes (e.g., strictly monotonic array),
    // the best bitonic sequence degenerates to the plain LIS or LDS
    if (maxLen == 0)
    {
        for (int i = 0; i < n; i++)
            maxLen = Math.Max(maxLen, Math.Max(lis[i], lds[i]));
    }

    return maxLen;
}
```

Time Complexity: O(n^2) — two O(n^2) DP passes (forward LIS and backward LDS) plus an O(n) combination step. Space Complexity: O(n) for the `lis` and `lds` arrays.

**Optimized Approach:** This problem directly extends the base LIS DP by running it twice — once forward for `lis[]` and once backward (mirrored) for `lds[]` — then combining the two arrays at each index as the "peak" of the bitonic sequence. There is no known way to reduce this below O(n^2) in general using the O(n log n) `tails` trick, because that trick only tracks lengths, not per-index values needed to combine both directions at every peak; the code above already represents the standard optimized (combined two-pass) approach.

```csharp
// Same two-pass forward-LIS / backward-LDS combination as above is the
// standard optimized technique for Longest Bitonic Subsequence.
public int LongestBitonicSubsequenceOptimized(int[] arr) => LongestBitonicSubsequence(arr);
```

Time Complexity: O(n^2), Space Complexity: O(n).

**Explanation:** For `arr = [1, 11, 2, 10, 4, 5, 2, 1]`:
- Forward `lis = [1, 2, 2, 3, 3, 4, 2, 1]` (e.g., `lis[3]` for value 10: best predecessor is 2 giving `lis=2+1=3`; `lis[5]` for value 5: predecessors 1,2,4 all smaller, best chain `1,2,4,5` gives `lis=4`).
- Backward `lds = [1, 5, 4, 4, 3, 3, 2, 1]` (e.g., `lds[1]` for value 11: decreasing chain starting at 11 is `11,10,5,2,1` or `11,10,4,2,1`, length 5).
- Combine at each peak: index 5 (value 5) gives `lis[5]+lds[5]-1 = 4+3-1=6`; index 3 (value 10) gives `3+4-1=6`. Maximum is `6`, matching one bitonic subsequence `[1,2,10,4,2,1]` (peak at value 10) or `[1,2,4,5,2,1]` (peak at value 5).

## 7. Number of Longest Increasing Subsequences

**Problem Statement:** Given an array of integers, find the total number of distinct longest increasing subsequences (i.e., how many different subsequences achieve the maximum LIS length).

**Example:**
- Input: `arr = [1, 3, 5, 4, 7]`
- Output: `2`
- Explanation: The LIS length is 4. There are two subsequences of length 4: `[1, 3, 5, 7]` and `[1, 3, 4, 7]`.

**Brute Force / DP Approach:** Maintain two parallel arrays alongside the standard LIS DP: `dp[i]` = length of the LIS ending at index `i` (as before), and `count[i]` = number of distinct LIS of that maximum length `dp[i]` ending at index `i`. Both start at 1 for every index. When scanning `j < i` with `arr[j] < arr[i]`: if `dp[j] + 1 > dp[i]`, a strictly better chain was found through `j`, so reset `dp[i] = dp[j] + 1` and `count[i] = count[j]` (inherit j's count, discarding any previously accumulated count). If `dp[j] + 1 == dp[i]`, `j` offers an equally good chain, so accumulate: `count[i] += count[j]`. After filling both arrays, find `maxLen = max(dp)`, then sum `count[i]` for every index where `dp[i] == maxLen`.

```csharp
public int FindNumberOfLIS(int[] arr)
{
    int n = arr.Length;
    if (n == 0) return 0;

    int[] dp = new int[n];
    int[] count = new int[n];
    Array.Fill(dp, 1);
    Array.Fill(count, 1);

    int maxLen = 1;
    for (int i = 1; i < n; i++)
    {
        for (int j = 0; j < i; j++)
        {
            if (arr[j] < arr[i])
            {
                if (dp[j] + 1 > dp[i])
                {
                    dp[i] = dp[j] + 1;
                    count[i] = count[j];       // start fresh, inherit j's count
                }
                else if (dp[j] + 1 == dp[i])
                {
                    count[i] += count[j];      // another way to reach the same best length
                }
            }
        }
        maxLen = Math.Max(maxLen, dp[i]);
    }

    int total = 0;
    for (int i = 0; i < n; i++)
    {
        if (dp[i] == maxLen)
            total += count[i];
    }

    return total;
}
```

Time Complexity: O(n^2), Space Complexity: O(n) for the `dp` and `count` arrays.

**Optimized Approach:** This problem directly extends the base LIS DP by tracking a parallel `count[]` array alongside `dp[]` during the same O(n^2) scan, rather than requiring a separate pass. No sub-O(n^2) technique is commonly used here because counting requires examining every valid predecessor to merge counts correctly (unlike length-only LIS, where the O(n log n) trick only needs the single best length, not all contributing paths); the code above already is the optimized, single-pass combined solution.

```csharp
// Same combined dp[] + count[] single-pass approach as above is the
// standard technique for counting Longest Increasing Subsequences.
public int FindNumberOfLISOptimized(int[] arr) => FindNumberOfLIS(arr);
```

Time Complexity: O(n^2), Space Complexity: O(n).

**Explanation:** Dry run of the combined `dp[]` + `count[]` tracking on `arr = [1, 3, 5, 4, 7]`:
- Initialize `dp = [1,1,1,1,1]`, `count = [1,1,1,1,1]`.
- `i=1 (3)`: `j=0 (1<3)`: `dp[0]+1=2 > dp[1]=1` → `dp[1]=2`, `count[1]=count[0]=1`. Result: `dp=[1,2,1,1,1]`, `count=[1,1,1,1,1]`.
- `i=2 (5)`: `j=0 (1<5)`: `dp[0]+1=2 > dp[2]=1` → `dp[2]=2`, `count[2]=count[0]=1`. `j=1 (3<5)`: `dp[1]+1=3 > dp[2]=2` → `dp[2]=3`, `count[2]=count[1]=1`. Result: `dp=[1,2,3,1,1]`, `count=[1,1,1,1,1]`.
- `i=3 (4)`: `j=0 (1<4)`: `dp[0]+1=2 > dp[3]=1` → `dp[3]=2`, `count[3]=count[0]=1`. `j=1 (3<4)`: `dp[1]+1=3 > dp[3]=2` → `dp[3]=3`, `count[3]=count[1]=1`. `j=2 (5<4? no, skip)`. Result: `dp=[1,2,3,3,1]`, `count=[1,1,1,1,1]`.
- `i=4 (7)`: `j=0 (1<7)`: `dp[0]+1=2 > dp[4]=1` → `dp[4]=2`, `count[4]=1`. `j=1 (3<7)`: `dp[1]+1=3 > dp[4]=2` → `dp[4]=3`, `count[4]=1`. `j=2 (5<7)`: `dp[2]+1=4 > dp[4]=3` → `dp[4]=4`, `count[4]=count[2]=1`. `j=3 (4<7)`: `dp[3]+1=4 == dp[4]=4` → **tie**, accumulate: `count[4] += count[3]` → `count[4]=1+1=2`. Result: `dp=[1,2,3,3,4]`, `count=[1,1,1,1,2]`.
- `maxLen = 4`, only index 4 has `dp[i]==4`, so `total = count[4] = 2`. This matches the expected output of 2, corresponding to the two LIS `[1,3,5,7]` (via index 2, the "reset" path) and `[1,3,4,7]` (via index 3, the "tie" path that got merged into `count[4]`).
