# Sliding Window and Two Pointer — Advanced Problems

## Concept: "At Most K" Trick

A large class of "exactly K" sliding window problems (count subarrays/substrings with a
property that equals exactly K) are hard to solve directly with a single sliding window,
because a window's validity is not monotonic in a simple way — adding one more element
can flip a window from "valid" to "invalid" and there is no clean single shrink condition
for "equal to K".

However, "at most K" problems (count subarrays/substrings with the property ≤ K) *are*
monotonic: as the right pointer moves forward, if the window becomes invalid (property > K),
we can keep shrinking from the left until it becomes valid again. This is the classic
single-pass two-pointer template, and it always works cleanly for "at most" constraints.

The key trick that bridges the two is:

```
countExactly(K) = countAtMost(K) - countAtMost(K - 1)
```

Why this works: `atMost(K)` counts every subarray whose property value is `0, 1, 2, ..., K`.
`atMost(K-1)` counts every subarray whose property value is `0, 1, ..., K-1`. Subtracting
removes every subarray whose property is strictly less than K, leaving exactly the subarrays
whose property value is precisely K.

This reduces an "exactly K" problem into two calls of an easier, well-understood "at most K"
sliding window helper. We use this trick for Problems 1, 2, and 5 below.

```csharp
// Generic shape of the trick (pseudocode):
// int CountExactlyK(int[] arr, int k)
// {
//     return AtMost(arr, k) - AtMost(arr, k - 1);
// }
```

---

## 1. Number of Substrings/Subarrays with Sum Equal to Goal (binary array)

**Problem Statement:**
Given a binary array `nums` (containing only 0s and 1s) and an integer `goal`, return the
number of non-empty subarrays whose sum equals `goal`.

**Example:**
- Input: `nums = [1,0,1,0,1]`, `goal = 2`
- Output: `4`
- Explanation: The subarrays with sum 2 are `[1,0,1]`, `[0,1,0,1]`, `[1,0,1]` (different
  positions), and `[1,0,1]` again at another offset. Concretely the valid subarrays (by
  index ranges) are `(0..2)`, `(0..3)`, `(1..4)`, `(2..4)` — 4 subarrays sum to 2.

**Brute Force Approach:**
For every pair of start and end indices, compute the subarray sum and check if it equals
`goal`. This is O(n^2) time (or O(n^3) if sums are recomputed from scratch each time
instead of incrementally).

```csharp
public int NumSubarraysWithSumBruteForce(int[] nums, int goal)
{
    int n = nums.Length;
    int count = 0;

    for (int i = 0; i < n; i++)
    {
        int sum = 0;
        for (int j = i; j < n; j++)
        {
            sum += nums[j];
            if (sum == goal)
            {
                count++;
            }
            // Since all elements are 0 or 1, sum is non-decreasing,
            // so once sum > goal we could break early, but we keep it
            // simple/brute here.
        }
    }

    return count;
}
```
Time Complexity: O(n^2). Space Complexity: O(1).

**Optimized Approach:**
Use the "at most K" trick: `exact(goal) = atMost(goal) - atMost(goal - 1)`. `AtMost(nums, k)`
counts the number of subarrays whose sum is ≤ k, computed with a single sliding window that
shrinks from the left whenever the window sum exceeds k. Special-case `k < 0` to return 0
(needed when `goal == 0`).

```csharp
public int NumSubarraysWithSum(int[] nums, int goal)
{
    return AtMost(nums, goal) - AtMost(nums, goal - 1);
}

private int AtMost(int[] nums, int k)
{
    if (k < 0)
    {
        return 0;
    }

    int left = 0;
    int sum = 0;
    int count = 0;

    for (int right = 0; right < nums.Length; right++)
    {
        sum += nums[right];

        while (sum > k)
        {
            sum -= nums[left];
            left++;
        }

        // Every subarray ending at 'right' and starting anywhere in
        // [left, right] has sum <= k.
        count += right - left + 1;
    }

    return count;
}
```
Time Complexity: O(n), Space Complexity: O(1).

**Explanation:**
`AtMost` slides a window and, for each right endpoint, counts all windows ending there whose
sum stays ≤ k by shrinking the left edge whenever the sum overshoots. Subtracting
`AtMost(goal - 1)` from `AtMost(goal)` removes all subarrays with sum strictly less than
`goal`, leaving only subarrays whose sum is exactly `goal`. (A full dry run of this exact
"at most" pattern is shown below for Problem 2, which uses the identical mechanics on odd
counts instead of sums.)

---

## 2. Count Number of Nice Subarrays (exactly K odd numbers)

**Problem Statement:**
Given an array of integers `nums` and an integer `k`, a subarray is called "nice" if it
contains exactly `k` odd numbers. Return the number of nice subarrays.

**Example:**
- Input: `nums = [1,1,2,1,1]`, `k = 3`
- Output: `2`
- Explanation: The subarrays with exactly 3 odd numbers are `[1,1,2,1]` and `[1,2,1,1]`.

**Brute Force Approach:**
For every subarray, count how many odd numbers it contains and compare to `k`.

```csharp
public int NumberOfSubarraysBruteForce(int[] nums, int k)
{
    int n = nums.Length;
    int count = 0;

    for (int i = 0; i < n; i++)
    {
        int oddCount = 0;
        for (int j = i; j < n; j++)
        {
            if (nums[j] % 2 != 0)
            {
                oddCount++;
            }
            if (oddCount == k)
            {
                count++;
            }
        }
    }

    return count;
}
```
Time Complexity: O(n^2). Space Complexity: O(1).

**Optimized Approach:**
Apply the "at most K" trick again: `exact(k) = atMost(k) - atMost(k - 1)`, where
`AtMost(nums, k)` counts subarrays containing at most `k` odd numbers, using a sliding
window that shrinks whenever the odd count exceeds `k`.

```csharp
public int NumberOfSubarrays(int[] nums, int k)
{
    return AtMostOdd(nums, k) - AtMostOdd(nums, k - 1);
}

private int AtMostOdd(int[] nums, int k)
{
    if (k < 0)
    {
        return 0;
    }

    int left = 0;
    int oddCount = 0;
    int count = 0;

    for (int right = 0; right < nums.Length; right++)
    {
        if (nums[right] % 2 != 0)
        {
            oddCount++;
        }

        while (oddCount > k)
        {
            if (nums[left] % 2 != 0)
            {
                oddCount--;
            }
            left++;
        }

        count += right - left + 1;
    }

    return count;
}
```
Time Complexity: O(n), Space Complexity: O(1).

**Explanation (dry run on `nums = [1,1,2,1,1]`, `k = 3`):**

We need `AtMostOdd(nums, 3) - AtMostOdd(nums, 2)`.

*Computing `AtMostOdd(nums, 3)`:*
| right | nums[right] | oddCount | window shrinks? | left | count += (right-left+1) | running count |
|---|---|---|---|---|---|---|
| 0 | 1 | 1 | no | 0 | +1 | 1 |
| 1 | 1 | 2 | no | 0 | +2 | 3 |
| 2 | 2 | 2 | no | 0 | +3 | 6 |
| 3 | 1 | 3 | no | 0 | +4 | 10 |
| 4 | 1 | 4 | yes → remove nums[0]=1 (odd), oddCount=3, left=1 | 1 | +4 (right-left+1=4-1+1) | 14 |

`AtMostOdd(nums, 3) = 14`.

*Computing `AtMostOdd(nums, 2)`:*
| right | nums[right] | oddCount | shrinks? | left | count += | running count |
|---|---|---|---|---|---|---|
| 0 | 1 | 1 | no | 0 | +1 | 1 |
| 1 | 1 | 2 | no | 0 | +2 | 3 |
| 2 | 2 | 2 | no | 0 | +3 | 6 |
| 3 | 1 | 3 | yes → remove nums[0]=1, oddCount=2, left=1 | 1 | +3 (3-1+1) | 9 |
| 4 | 1 | 3 | yes → remove nums[1]=1, oddCount=2, left=2 | 2 | +3 (4-2+1) | 12 |

`AtMostOdd(nums, 2) = 12`.

Result: `14 - 12 = 2`, matching the expected output. The 2 nice subarrays
(`[1,1,2,1]` at indices 0..3, and `[1,2,1,1]` at indices 1..4) are exactly the ones with
odd-count 3 that get included in the "at most 3" count but excluded from the "at most 2" count.

---

## 3. Number of Substrings Containing All Three Characters 'a', 'b', 'c'

**Problem Statement:**
Given a string `s` consisting only of characters 'a', 'b', and 'c', return the number of
substrings that contain at least one occurrence of all three characters.

**Example:**
- Input: `s = "abcabc"`
- Output: `10`
- Explanation: Every substring of length ≥ 3 that contains all of a, b, c counts. Substrings
  like `"abc"`, `"bca"`, `"cab"`, `"abca"`, `"bcab"`, `"cabc"`, `"abcab"`, `"bcabc"`,
  `"abcabc"`, and one more (`"abc"` occurring again) total 10.

**Brute Force Approach:**
Check every substring and verify it contains all three characters using a frequency count.

```csharp
public int NumberOfSubstringsBruteForce(string s)
{
    int n = s.Length;
    int count = 0;

    for (int i = 0; i < n; i++)
    {
        var seen = new HashSet<char>();
        for (int j = i; j < n; j++)
        {
            seen.Add(s[j]);
            if (seen.Contains('a') && seen.Contains('b') && seen.Contains('c'))
            {
                count++;
            }
        }
    }

    return count;
}
```
Time Complexity: O(n^2). Space Complexity: O(1) (at most 3 distinct chars tracked).

**Optimized Approach:**
Maintain a frequency map of the 3 characters within a sliding window `[left, right]`. Expand
`right` one character at a time; whenever the window contains at least one of each of 'a',
'b', 'c', every substring starting at any index from `left` to `right` (and ending at
`right` or beyond, but more precisely: every substring `s[left..right], s[left+1..right],
..., ` up to the point we can no longer remove `s[left]` without breaking validity) is valid.
The clean way to count: once the window `[left, right]` contains all three characters, ALL
substrings ending at `right` and starting at any index ≤ `left` are also valid (since
shrinking from the left as far as possible while remaining valid, then everything to the
left of that boundary still contains all 3 chars). So for each `right`, add `left + 1` to
the count (number of valid starting points from index 0 to `left`), where `left` is
advanced as far right as possible while the window remains valid.

```csharp
public int NumberOfSubstrings(string s)
{
    var freq = new Dictionary<char, int> { { 'a', 0 }, { 'b', 0 }, { 'c', 0 } };
    int left = 0;
    int count = 0;

    for (int right = 0; right < s.Length; right++)
    {
        freq[s[right]]++;

        // Shrink from the left while the window still contains all 3 chars.
        while (freq['a'] > 0 && freq['b'] > 0 && freq['c'] > 0)
        {
            // All substrings starting at [0..left] up through this point are
            // valid once we know s[left..right] has all 3; add them, then
            // try shrinking further.
            count += s.Length - right;
            freq[s[left]]--;
            left++;
        }
    }

    return count;
}
```
Time Complexity: O(n), Space Complexity: O(1) (fixed 3-character map).

**Explanation:**
For a fixed `left`, once `[left, right]` contains all 3 characters, then `[left, right]`,
`[left, right+1]`, ..., `[left, n-1]` are ALL valid too (extending right only keeps all
characters present). That is exactly `s.Length - right` valid substrings starting at
`left`. We then increment `left` (shrinking the window) and repeat the check — if the
window is still valid after removing `s[left]`, we add another batch of `s.Length - right`
for the new `left`, and keep shrinking until the window no longer contains all three
characters, at which point we resume expanding `right`. This way every valid substring is
counted exactly once, in O(n) total work since `left` only moves forward.

---

## 4. Maximum Points You Can Obtain from Cards

**Problem Statement:**
There are several cards arranged in a row, each with a point value, given in array
`cardPoints`. In one step you can take one card from the beginning or from the end of the
row. You must take exactly `k` cards in total. Return the maximum total points you can
obtain.

**Example:**
- Input: `cardPoints = [1,2,3,4,5,6,1]`, `k = 3`
- Output: `12`
- Explanation: Take the last 3 cards: `1 + 6 + 5 = 12`, which is better than any other
  combination of 3 cards from the ends.

**Brute Force Approach:**
Try every way to split `k` cards between `i` taken from the front and `k - i` taken from
the back (`i` from 0 to k), summing each combination directly (recomputing sums from
scratch each time).

```csharp
public int MaxScoreBruteForce(int[] cardPoints, int k)
{
    int n = cardPoints.Length;
    int best = 0;

    for (int i = 0; i <= k; i++)
    {
        int fromFront = i;
        int fromBack = k - i;
        int sum = 0;

        for (int f = 0; f < fromFront; f++)
        {
            sum += cardPoints[f];
        }

        for (int b = 0; b < fromBack; b++)
        {
            sum += cardPoints[n - 1 - b];
        }

        best = Math.Max(best, sum);
    }

    return best;
}
```
Time Complexity: O(k^2) (or O(n*k) if k close to n). Space Complexity: O(1).

**Optimized Approach:**
Picking `k` cards total from the two ends is equivalent to REMOVING a contiguous subarray
of length `n - k` from the middle and keeping the rest. So:

```
maxScore = totalSum - minimumSumOfWindowOfSize(n - k)
```

We compute the total sum once, then slide a fixed-size window of length `n - k` across the
array to find the minimum window sum; subtracting that minimum from the total gives the
maximum sum of the remaining (front + back) cards.

```csharp
public int MaxScore(int[] cardPoints, int k)
{
    int n = cardPoints.Length;
    int windowSize = n - k;

    if (windowSize == 0)
    {
        // Taking all cards.
        int allSum = 0;
        foreach (int p in cardPoints)
        {
            allSum += p;
        }
        return allSum;
    }

    int totalSum = 0;
    foreach (int p in cardPoints)
    {
        totalSum += p;
    }

    int windowSum = 0;
    for (int i = 0; i < windowSize; i++)
    {
        windowSum += cardPoints[i];
    }

    int minWindowSum = windowSum;

    for (int right = windowSize; right < n; right++)
    {
        windowSum += cardPoints[right] - cardPoints[right - windowSize];
        minWindowSum = Math.Min(minWindowSum, windowSum);
    }

    return totalSum - minWindowSum;
}
```
Time Complexity: O(n), Space Complexity: O(1).

**Explanation:**
Any selection of `k` cards from the two ends leaves behind a single contiguous block of
`n - k` untaken cards in the middle. To maximize the picked sum, we must minimize the sum
of that leftover middle block. We slide a fixed window of size `n - k` across the array
(classic fixed-size sliding window: add the incoming element, subtract the outgoing
element) to find the minimum possible middle-block sum, then `totalSum - minWindowSum`
gives the best achievable score.

---

## 5. Subarrays with K Different Integers (exactly K distinct integers)

**Problem Statement:**
Given an integer array `nums` and an integer `k`, return the number of "good" subarrays,
where a good subarray has exactly `k` different integers in it.

**Example:**
- Input: `nums = [1,2,1,2,3]`, `k = 2`
- Output: `7`
- Explanation: Subarrays with exactly 2 distinct values: `[1,2]`, `[2,1]`, `[1,2]`,
  `[2,1,2]`, `[1,2,1]`, `[2,1,2,3]` is NOT valid (3 distinct) ... the correct 7 subarrays
  are `[1,2]`, `[2,1]`, `[1,2]`, `[2,1,2]`, `[1,2,1]`, `[1,2,1,2]`... Concretely there are
  7 such subarrays as verified by the formula below.

**Brute Force Approach:**
For every subarray, use a hash set (or frequency map) to count distinct integers and
compare to `k`.

```csharp
public int SubarraysWithKDistinctBruteForce(int[] nums, int k)
{
    int n = nums.Length;
    int count = 0;

    for (int i = 0; i < n; i++)
    {
        var seen = new HashSet<int>();
        for (int j = i; j < n; j++)
        {
            seen.Add(nums[j]);
            if (seen.Count == k)
            {
                count++;
            }
            else if (seen.Count > k)
            {
                break;
            }
        }
    }

    return count;
}
```
Time Complexity: O(n^2). Space Complexity: O(n) worst case for the set.

**Optimized Approach:**
Same "at most K" trick: `exact(k) = atMost(k) - atMost(k - 1)`, where `AtMost(nums, k)`
counts subarrays with at most `k` distinct integers using a frequency map and a sliding
window that shrinks whenever the number of distinct integers exceeds `k`.

```csharp
public int SubarraysWithKDistinct(int[] nums, int k)
{
    return AtMostKDistinct(nums, k) - AtMostKDistinct(nums, k - 1);
}

private int AtMostKDistinct(int[] nums, int k)
{
    if (k < 0)
    {
        return 0;
    }

    var freq = new Dictionary<int, int>();
    int left = 0;
    int count = 0;

    for (int right = 0; right < nums.Length; right++)
    {
        if (!freq.ContainsKey(nums[right]))
        {
            freq[nums[right]] = 0;
        }
        freq[nums[right]]++;

        while (freq.Count > k)
        {
            freq[nums[left]]--;
            if (freq[nums[left]] == 0)
            {
                freq.Remove(nums[left]);
            }
            left++;
        }

        count += right - left + 1;
    }

    return count;
}
```
Time Complexity: O(n), Space Complexity: O(K) for the frequency map.

**Explanation:**
`AtMostKDistinct` uses the standard variable-size window: expand `right`, and whenever the
number of distinct keys in `freq` exceeds `k`, shrink from `left` (decrementing/removing
counts) until it's back to at most `k` distinct values. For each `right`, `right - left + 1`
counts all valid windows ending at `right`. Subtracting `AtMost(k-1)` from `AtMost(k)`
isolates subarrays with exactly `k` distinct integers, using the same reasoning as
Problems 1 and 2.

---

## 6. Minimum Window Substring

**Problem Statement:**
Given two strings `s` and `t`, return the minimum window substring of `s` such that every
character in `t` (including duplicates) is included in the window. If there is no such
substring, return the empty string.

**Example:**
- Input: `s = "ADOBECODEBANC"`, `t = "ABC"`
- Output: `"BANC"`
- Explanation: `"BANC"` is the smallest substring of `s` that contains 'A', 'B', and 'C'.

**Brute Force Approach:**
Check every substring of `s`, verify (via frequency counting) that it contains all
characters of `t` with sufficient counts, and track the smallest one that qualifies.

```csharp
public string MinWindowBruteForce(string s, string t)
{
    int n = s.Length;
    string best = "";
    int bestLen = int.MaxValue;

    for (int i = 0; i < n; i++)
    {
        for (int j = i; j < n; j++)
        {
            string candidate = s.Substring(i, j - i + 1);
            if (Contains(candidate, t))
            {
                if (candidate.Length < bestLen)
                {
                    bestLen = candidate.Length;
                    best = candidate;
                }
            }
        }
    }

    return best;
}

private bool Contains(string window, string t)
{
    var need = new Dictionary<char, int>();
    foreach (char c in t)
    {
        if (!need.ContainsKey(c)) need[c] = 0;
        need[c]++;
    }

    var have = new Dictionary<char, int>();
    foreach (char c in window)
    {
        if (!have.ContainsKey(c)) have[c] = 0;
        have[c]++;
    }

    foreach (var kvp in need)
    {
        if (!have.ContainsKey(kvp.Key) || have[kvp.Key] < kvp.Value)
        {
            return false;
        }
    }

    return true;
}
```
Time Complexity: O(n^3) (O(n^2) substrings, O(n) check each). Space Complexity: O(charset).

**Optimized Approach:**
Classic two-pointer template with a `need` frequency map (built from `t`) and a `have`
counter tracking how many distinct required characters currently satisfy their needed
count. Expand `right` to include characters; once all required characters are satisfied
("have == need.Count distinct chars satisfied"), shrink `left` as far as possible while
still valid, recording the smallest valid window along the way.

```csharp
public string MinWindow(string s, string t)
{
    if (string.IsNullOrEmpty(s) || string.IsNullOrEmpty(t) || t.Length > s.Length)
    {
        return "";
    }

    var need = new Dictionary<char, int>();
    foreach (char c in t)
    {
        if (!need.ContainsKey(c)) need[c] = 0;
        need[c]++;
    }

    var windowCounts = new Dictionary<char, int>();
    int required = need.Count;   // distinct characters we must fully satisfy
    int formed = 0;               // distinct characters currently fully satisfied

    int left = 0;
    int bestLen = int.MaxValue;
    int bestStart = 0;

    for (int right = 0; right < s.Length; right++)
    {
        char c = s[right];
        if (!windowCounts.ContainsKey(c)) windowCounts[c] = 0;
        windowCounts[c]++;

        if (need.ContainsKey(c) && windowCounts[c] == need[c])
        {
            formed++;
        }

        // Try to shrink the window from the left while it's still valid.
        while (left <= right && formed == required)
        {
            if (right - left + 1 < bestLen)
            {
                bestLen = right - left + 1;
                bestStart = left;
            }

            char leftChar = s[left];
            windowCounts[leftChar]--;
            if (need.ContainsKey(leftChar) && windowCounts[leftChar] < need[leftChar])
            {
                formed--;
            }

            left++;
        }
    }

    return bestLen == int.MaxValue ? "" : s.Substring(bestStart, bestLen);
}
```
Time Complexity: O(|s| + |t|), Space Complexity: O(charset size of t).

**Explanation (dry run of Minimum Window Substring on `s = "ADOBECODEBANC"`, `t = "ABC"`):**

`need = {A:1, B:1, C:1}`, `required = 3`.

- `right=0 ('A')`: windowCounts{A:1}. A matches need (1==1) → formed=1. formed(1) != required(3), no shrink.
- `right=1 ('D')`: windowCounts{A:1,D:1}. D not in need. formed=1.
- `right=2 ('O')`: windowCounts{...,O:1}. formed=1.
- `right=3 ('B')`: windowCounts{...,B:1}. B matches (1==1) → formed=2.
- `right=4 ('E')`: windowCounts{...,E:1}. formed=2.
- `right=5 ('C')`: windowCounts{...,C:1}. C matches (1==1) → formed=3 = required!
  - Window is now `s[0..5] = "ADOBEC"`, length 6. bestLen=6, bestStart=0.
  - Shrink: remove s[0]='A'. windowCounts[A]=0 < need[A]=1 → formed=2. left=1.
  - formed != required, stop shrinking.
- `right=6 ('O')`: windowCounts O:2. formed=2.
- `right=7 ('D')`: windowCounts D:2. formed=2.
- `right=8 ('E')`: windowCounts E:2. formed=2.
- `right=9 ('B')`: windowCounts B:2. B already satisfied, formed stays 2 (still only counts distinct-satisfied once)... but wait B was already counted; formed remains 2 since B was already "formed" — actually formed only increments the first time count reaches need; it's already been satisfied so formed stays at 2 here, but note A is currently NOT satisfied (windowCounts[A]=0), B and C were the two formed... let's recheck: after right=5 shrink, formed=2 means B and C are satisfied, A is not. At right=9, B count goes to 2 (still satisfied, formed unchanged=2).
- `right=10 ('A')`: windowCounts A:1. A matches need(1==1) → formed=3 = required!
  - Window is `s[1..10] = "DOBECODEBA"`, length 10. Not better than bestLen=6.
  - Shrink: remove s[1]='D' (not in need). left=2. formed still 3 (D removal doesn't affect need).
    - Window `s[2..10]="OBECODEBA"` length 9, still formed=3, check length 9 not better; keep shrinking.
  - remove s[2]='O'. left=3. formed=3. Window `s[3..10]="BECODEBA"` length 8, not better.
  - remove s[3]='B'. windowCounts[B]=1, still >= need[B]=1 → formed stays 3. left=4. Window `s[4..10]="ECODEBA"` length 7, not better.
  - remove s[4]='E'. left=5. formed=3. Window `s[5..10]="CODEBA"` length 6, ties bestLen but not strictly less, so bestLen/bestStart unchanged (still 0,6) per `<` comparison.
  - remove s[5]='C'. windowCounts[C]=0 < need[C]=1 → formed=2. left=6. Stop shrinking (formed != required).
- `right=11 ('N')`: windowCounts N:1. Not in need. formed=2.
- `right=12 ('C')`: windowCounts C:1. C matches (1==1) → formed=3 = required!
  - Window `s[6..12] = "ODEBANC"`, length 7. Not better than 6.
  - Shrink: remove s[6]='O'. left=7. formed=3 (O not in need). Window `s[7..12]="DEBANC"` length 6, ties, not strictly better.
  - remove s[7]='D'. left=8. formed=3. Window `s[8..12]="EBANC"` length 5 < bestLen(6)! bestLen=5, bestStart=8.
  - remove s[8]='E'. left=9. formed=3. Window `s[9..12]="BANC"` length 4 < 5! bestLen=4, bestStart=9.
  - remove s[9]='B'. windowCounts[B]=0 < need[B]=1 → formed=2. left=10. Stop shrinking.
- Loop ends (right reaches end of string).

Final answer: `bestStart=9, bestLen=4` → `s.Substring(9,4) = "BANC"`, matching the expected
output exactly.

---

## 7. Minimum Window Subsequence

**Problem Statement:**
Given strings `S` and `T`, find the minimum (contiguous) substring `W` of `S`, so that `T`
is a subsequence of `W` (characters of `T` appear in `W` in the same relative order, but not
necessarily contiguously). If there is no such window in `S` that covers all characters of
`T`, return the empty string. Unlike Minimum Window Substring, here character ORDER matters
and characters need not be a simple frequency match — this requires a different, two-pass
two-pointer technique.

**Example:**
- Input: `S = "abcdebdde"`, `T = "bde"`
- Output: `"bcde"`
- Explanation: `"bcde"` is the smallest substring of `S` containing `T = "bde"` as a
  subsequence (`b` then `d` then `e`, in order). Note `"deb"` also contains letters but not
  in the required order, and `"bdde"` is longer than `"bcde"`.

**Brute Force Approach:**
For every possible starting index in `S`, and every possible ending index, check whether
`T` is a subsequence of that substring, and track the minimum length window that works.

```csharp
public string MinWindowSubsequenceBruteForce(string s, string t)
{
    int n = s.Length;
    string best = "";
    int bestLen = int.MaxValue;

    for (int i = 0; i < n; i++)
    {
        for (int j = i; j < n; j++)
        {
            string candidate = s.Substring(i, j - i + 1);
            if (IsSubsequence(t, candidate) && candidate.Length < bestLen)
            {
                bestLen = candidate.Length;
                best = candidate;
            }
        }
    }

    return best;
}

private bool IsSubsequence(string t, string window)
{
    int ti = 0;
    for (int wi = 0; wi < window.Length && ti < t.Length; wi++)
    {
        if (window[wi] == t[ti])
        {
            ti++;
        }
    }
    return ti == t.Length;
}
```
Time Complexity: O(n^2 * m) where n = |S|, m = |T|. Space Complexity: O(n) for substrings.

**Optimized Approach:**
Use a specialized two-pointer "expand then contract" technique, run in a single left-to-right
scan of `S`:

1. **Expand phase:** Move a pointer `right` through `S`, and a pointer `ti` through `T`,
   matching characters of `T` in order. Once `ti` reaches the end of `T` (all of `T` matched
   as a subsequence ending at `right`), we've found a candidate window ending at `right`.
2. **Contract phase:** From that ending position, walk BACKWARD matching `T` in reverse to
   find the tightest possible start of the window (the last position where the window
   `[start, right]` still contains `T` as a subsequence). This gives the true minimal window
   ending at `right`.
3. Record the window if it's smaller than the best found so far, then resume the expand
   phase from just after this window's start (`right = start + 1`) to look for the next
   candidate.

This differs fundamentally from Minimum Window Substring because we cannot use a simple
frequency map — order matters, so we must literally walk through characters matching them
positionally, both forwards (to find *a* valid end) and backwards (to tighten the start).

```csharp
public string MinWindowSubsequence(string s, string t)
{
    int n = s.Length;
    int m = t.Length;
    int bestStart = -1;
    int bestLen = int.MaxValue;

    int right = 0;
    while (right < n)
    {
        // Expand phase: advance 'right' and match characters of t in order.
        int ti = 0;
        while (right < n)
        {
            if (s[right] == t[ti])
            {
                ti++;
                if (ti == m)
                {
                    break; // fully matched t ending at index 'right'
                }
            }
            right++;
        }

        if (ti < m)
        {
            // Ran out of s before fully matching t; no more candidates possible.
            break;
        }

        // Contract phase: walk backward from 'right' to find the tightest start.
        int end = right; // index in s where match of t completed
        int start = end;
        ti = m - 1;
        while (ti >= 0)
        {
            if (s[start] == t[ti])
            {
                ti--;
            }
            start--;
        }
        start++; // step back to the actual start index of the valid window

        if (end - start + 1 < bestLen)
        {
            bestLen = end - start + 1;
            bestStart = start;
        }

        // Resume expand phase just after this window's start to find the next candidate.
        right = start + 1;
    }

    return bestStart == -1 ? "" : s.Substring(bestStart, bestLen);
}
```
Time Complexity: O(n * m) in the worst case (each of the up-to-n expand/contract cycles can
scan up to n characters, but in practice this is efficient; the tight bound is O(n * m) since
each contraction is bounded by needing to match m characters). Space Complexity: O(1)
(excluding the output substring).

**Explanation:**
Because Minimum Window Subsequence requires `T` to appear as an ORDERED subsequence (not
just "all characters present with sufficient counts" like Minimum Window Substring), a
frequency-map `need`/`have` approach does not work — two windows could have identical
character frequencies but only one might have them in the right order. Instead:
- The **expand phase** greedily finds the earliest position where a subsequence match of
  `T` completes, by scanning forward and advancing a pointer into `T` whenever the current
  character of `S` matches the next needed character of `T`.
- The **contract phase** then scans backward from that completion point, matching `T` in
  reverse, to find the latest possible start such that `T` is still a subsequence of
  `[start, end]`. This tightens the window as much as possible (removing any unnecessary
  prefix characters that expand phase passed over but weren't strictly needed once we know
  the exact end position).
- After recording this candidate, the scan resumes expanding from `start + 1`, since any
  smaller/better window must start after the current tightest start.

This guarantees we examine every minimal candidate window efficiently without needing to
re-scan all of `S` from scratch for each candidate, avoiding the O(n^2 * m) blowup of the
brute-force approach.
