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

**Logic (Steps):**
1. Fix a starting index `i` and reset a running `sum` to 0.
2. Extend `j` from `i` to the end, adding `nums[j]` to `sum` incrementally.
3. Whenever `sum == goal`, increment `count`.
4. Repeat for every starting index `i`, giving the total count of subarrays summing to `goal`.

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

**Logic (Steps):**
1. Return `AtMost(nums, goal) - AtMost(nums, goal - 1)` — the count of subarrays with sum exactly `goal`.
2. Inside `AtMost(nums, k)`: if `k < 0`, return 0 immediately (no subarray can have a negative sum).
3. Slide `right` forward, adding `nums[right]` to a running `sum`.
4. While `sum > k`, shrink from the left: subtract `nums[left]` from `sum` and advance `left`, restoring the invariant `sum <= k`.
5. For each `right`, every subarray starting anywhere in `[left, right]` and ending at `right` has sum `<= k`, so add `right - left + 1` to `count`.

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

**Walkthrough:** On `nums = [1,0,1,0,1]`, `goal = 2`, compute `AtMost(nums, 2) - AtMost(nums, 1)`. For `AtMost(nums, 2)`: `right=0..3` keep `sum<=2` with no shrink, contributing `1+2+3+4=10` to `count`; at `right=4`, `sum=3>2` forces one shrink (`left=1`, `sum=2`), contributing `4` more, giving `AtMost(2)=14`. For `AtMost(nums, 1)`: `right=0,1` contribute `1+2=3`; at `right=2`, `sum=2>1` shrinks once (`left=1`), contributing `2`; at `right=3`, `sum=1<=1`, contributing `3`; at `right=4`, `sum=2>1` shrinks twice (`left=3`), contributing `2`; total `AtMost(1)=10`. Final result: `14 - 10 = 4`, matching the expected output. (A full dry run of this exact "at most" pattern is shown below for Problem 2, which uses the identical mechanics on odd counts instead of sums.)

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

**Logic (Steps):**
1. Fix a starting index `i` and reset `oddCount` to 0.
2. Extend `j` from `i` to the end, incrementing `oddCount` whenever `nums[j]` is odd.
3. Whenever `oddCount == k`, increment `count` (the subarray `nums[i..j]` has exactly `k` odd numbers).
4. Repeat for every starting index `i`.

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

**Logic (Steps):**
1. Return `AtMostOdd(nums, k) - AtMostOdd(nums, k - 1)` — the count of subarrays with exactly `k` odd numbers.
2. Inside `AtMostOdd(nums, k)`: if `k < 0`, return 0 immediately.
3. Slide `right` forward, incrementing `oddCount` whenever `nums[right]` is odd.
4. While `oddCount > k`, shrink from the left: if `nums[left]` is odd, decrement `oddCount`, then advance `left`, until `oddCount <= k` again.
5. For each `right`, add `right - left + 1` to `count` — every subarray ending at `right` and starting in `[left, right]` has at most `k` odd numbers.

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

**Walkthrough:** On `nums = [1,1,2,1,1]`, `k = 3`, compute `AtMostOdd(nums, 3) - AtMostOdd(nums, 2)`.
For `AtMostOdd(nums, 3)`: `right=0..3` never exceed `oddCount=3`, contributing `1+2+3+4=10`; at `right=4`, `oddCount=4>3` forces one shrink (`left=1`, `oddCount=3`), contributing `4` more, giving `AtMostOdd(3)=14`.
For `AtMostOdd(nums, 2)`: `right=0,1` contribute `1+2=3`; at `right=2` (even number) `oddCount` stays 2, contributing `3` (total 6); at `right=3`, `oddCount=3>2` shrinks once (`left=1`), contributing `3` (total 9); at `right=4`, `oddCount=3>2` shrinks again (`left=2`), contributing `3`, giving `AtMostOdd(2)=12`.
Result: `14 - 12 = 2`, matching the expected output — the 2 nice subarrays are `[1,1,2,1]` (indices 0..3) and `[1,2,1,1]` (indices 1..4), each with exactly 3 odd numbers.

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

**Logic (Steps):**
1. Fix a starting index `i` and reset a `HashSet<char> seen`.
2. Extend `j` from `i` to the end, adding `s[j]` to `seen`.
3. Whenever `seen` contains `'a'`, `'b'`, and `'c'`, increment `count` (the substring `s[i..j]` is valid).
4. Repeat for every starting index `i`.

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

**Walkthrough:** On `s = "abcabc"`: starting at `i=0`, the substring becomes valid (contains a, b, c) once `j=2` (`"abc"`), and stays valid for every `j` from 2 to 5, contributing 4 valid substrings. Starting at `i=1,2,3` similarly become valid once all three letters reappear within the remaining suffix, contributing more counts, while `i=4,5` can never contain all three (too few characters left). Summing across all starting indices gives `count = 10`, matching the expected output.

---

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

**Logic (Steps):**
1. Maintain `freq`, a count of `'a'`, `'b'`, `'c'` currently in the window `[left, right]`.
2. Move `right` forward, incrementing `freq[s[right]]`.
3. While the window contains at least one of each of `'a'`, `'b'`, `'c'`, every substring ending anywhere from `right` to the end of `s` and starting at the current `left` is valid, so add `s.Length - right` to `count`.
4. Then shrink: decrement `freq[s[left]]` and advance `left`, and re-check the while condition — this greedily pushes `left` as far right as possible while the window stays valid, counting each valid `left` position once.
5. Repeat until `right` reaches the end of the string.

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

**Walkthrough:** Dry run on `s = "abcabc"`. `right=0,1` (`'a'`,`'b'`) don't yet complete all 3 letters. `right=2` (`'c'`): `freq={a:1,b:1,c:1}`, all present → `count += 6-2 = 4` (count=4); shrink removes `s[0]='a'` (`freq[a]=0`, `left=1`), no longer all present, stop. `right=3` (`'a'`): `freq={a:1,b:1,c:1}` again all present → `count += 6-3 = 3` (count=7); shrink removes `s[1]='b'` (`left=2`), stop. `right=4` (`'b'`): all present → `count += 6-4 = 2` (count=9); shrink removes `s[2]='c'` (`left=3`), stop. `right=5` (`'c'`): all present → `count += 6-5 = 1` (count=10); shrink removes `s[3]='a'` (`left=4`), stop. Loop ends with `count = 10`, matching the expected output.

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

**Logic (Steps):**
1. Try every split `i` from `0` to `k`: take `i` cards from the front and `k - i` cards from the back.
2. Sum the first `i` elements (`cardPoints[0..i-1]`).
3. Sum the last `k - i` elements (`cardPoints[n-1] down to n-(k-i)`).
4. Track `best`, the maximum total across all splits.
5. Repeat for every split, giving the best achievable score.

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

**Walkthrough:** On `cardPoints = [1,2,3,4,5,6,1]`, `k = 3`: `i=0` (0 front, 3 back) → back = `5+6+1=12`, total 12. `i=1` (1 front, 2 back) → front=1, back=`6+1=7`, total 8. `i=2` (2 front, 1 back) → front=`1+2=3`, back=`1`, total 4. `i=3` (3 front, 0 back) → front=`1+2+3=6`, total 6. The best across all splits is `12`, matching the expected output.

---

**Optimized Approach:**
Picking `k` cards total from the two ends is equivalent to REMOVING a contiguous subarray
of length `n - k` from the middle and keeping the rest. So:

```
maxScore = totalSum - minimumSumOfWindowOfSize(n - k)
```

We compute the total sum once, then slide a fixed-size window of length `n - k` across the
array to find the minimum window sum; subtracting that minimum from the total gives the
maximum sum of the remaining (front + back) cards.

**Logic (Steps):**
1. Handle the edge case `windowSize = n - k == 0` (taking all cards) by returning the total sum directly.
2. Compute `totalSum`, the sum of the entire array.
3. Compute the sum of the first `windowSize` elements as the initial `windowSum`, and set `minWindowSum = windowSum`.
4. Slide the fixed-size window rightward one element at a time: add the incoming element `cardPoints[right]` and subtract the outgoing element `cardPoints[right - windowSize]`, updating `minWindowSum` whenever a smaller sum is found.
5. Return `totalSum - minWindowSum` — minimizing the untaken middle block maximizes the picked front+back cards.

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

**Walkthrough:** On `cardPoints = [1,2,3,4,5,6,1]`, `k = 3`, `n = 7`, so `windowSize = 4`. `totalSum = 22`. The initial window `cardPoints[0..3] = [1,2,3,4]` has sum `10`, so `minWindowSum = 10`. Sliding: `right=4` → `windowSum += cardPoints[4]-cardPoints[0] = 5-1 = 4`, new sum `14` (not smaller). `right=5` → `windowSum += cardPoints[5]-cardPoints[1] = 6-2 = 4`, new sum `18` (not smaller). `right=6` → `windowSum += cardPoints[6]-cardPoints[2] = 1-3 = -2`, new sum `16` (not smaller). `minWindowSum` stays `10`. Result: `totalSum - minWindowSum = 22 - 10 = 12`, matching the expected output.

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

**Logic (Steps):**
1. Fix a starting index `i` and reset a `HashSet<int> seen`.
2. Extend `j` from `i` to the end, adding `nums[j]` to `seen`.
3. If `seen.Count == k`, increment `count`; if `seen.Count > k`, break (extending further can only add more distinct values).
4. Repeat for every starting index `i`.

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

**Walkthrough:** On `nums = [1,2,1,2,3]`, `k = 2`: `i=0` gives valid subarrays at `j=1,2,3` (3 hits, `count=3`) before `j=4` introduces a third distinct value and breaks. `i=1` gives hits at `j=2,3` (`count=5`). `i=2` gives a hit at `j=3` (`count=6`). `i=3` gives a hit at `j=4` (`count=7`). `i=4` alone never reaches 2 distinct values. Final `count = 7`, matching the expected output.

---

**Optimized Approach:**
Same "at most K" trick: `exact(k) = atMost(k) - atMost(k - 1)`, where `AtMost(nums, k)`
counts subarrays with at most `k` distinct integers using a frequency map and a sliding
window that shrinks whenever the number of distinct integers exceeds `k`.

**Logic (Steps):**
1. Return `AtMostKDistinct(nums, k) - AtMostKDistinct(nums, k - 1)` — the count of subarrays with exactly `k` distinct integers.
2. Inside `AtMostKDistinct(nums, k)`: if `k < 0`, return 0 immediately.
3. Slide `right` forward, incrementing `freq[nums[right]]` (adding a new key if needed).
4. While `freq.Count > k`, shrink from the left: decrement `freq[nums[left]]`, remove the key if it hits 0, then advance `left`, until `freq.Count <= k` again.
5. For each `right`, add `right - left + 1` to `count`.

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

**Walkthrough:** On `nums = [1,2,1,2,3]`, `k = 2`, compute `AtMostKDistinct(nums, 2) - AtMostKDistinct(nums, 1)`. For `AtMost(2)`: `right=0..3` never exceed 2 distinct keys, contributing `1+2+3+4=10`; at `right=4`, a third key (`3`) appears, forcing 3 shrinks down to `left=3`, contributing `2` more, giving `AtMost(2)=12`. For `AtMost(1)`: every `right` from 0 to 4 immediately exceeds 1 distinct key (except `right=0`), so `left` shrinks to keep exactly 1 key each time, contributing `1` at every step, giving `AtMost(1)=5`. Result: `12 - 5 = 7`, matching the expected output.

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

**Logic (Steps):**
1. Try every substring `s[i..j]` by iterating over all `(i, j)` pairs.
2. For each candidate, call `Contains` to build frequency maps `need` (from `t`) and `have` (from the candidate substring).
3. The candidate is valid only if `have` meets or exceeds `need` for every character in `t`.
4. Among all valid candidates, track the shortest one (`bestLen`, `best`).
5. Return the shortest valid substring found, or the empty string if none qualifies.

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

**Walkthrough:** On `s = "ADOBECODEBANC"`, `t = "ABC"`: the brute force checks every substring `s[i..j]` for containing at least one `'A'`, `'B'`, and `'C'`. The first valid substring found while scanning is `"ADOBEC"` (indices 0-5, length 6). As `i` and `j` sweep further, shorter valid substrings are discovered, including `"CODEBA"` (length 6, tied) and eventually `"BANC"` (indices 9-12, length 4), which is never beaten by any other substring. `bestLen` ends at 4 with `best = "BANC"`, matching the expected output.

---

**Optimized Approach:**
Classic two-pointer template with a `need` frequency map (built from `t`) and a `have`
counter tracking how many distinct required characters currently satisfy their needed
count. Expand `right` to include characters; once all required characters are satisfied
("have == need.Count distinct chars satisfied"), shrink `left` as far as possible while
still valid, recording the smallest valid window along the way.

**Logic (Steps):**
1. Build `need`, a frequency map of every character in `t`; `required = need.Count` is the number of distinct characters that must be fully satisfied.
2. Move `right` forward, incrementing `windowCounts[s[right]]`; whenever a character's count in the window first reaches its required count in `need`, increment `formed`.
3. While `formed == required` (the window currently satisfies all of `t`), record the window if it's the smallest so far (`bestLen`, `bestStart`).
4. Then shrink from the left: decrement `windowCounts[s[left]]`; if that drops a required character below its needed count, decrement `formed` (window becomes invalid) and stop shrinking; either way advance `left`.
5. Repeat until `right` reaches the end of `s`, then return the smallest recorded window (or empty string if none was found).

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

**Walkthrough:** On `s = "ADOBECODEBANC"`, `t = "ABC"`: `need = {A:1,B:1,C:1}`, `required = 3`. Scanning forward, `formed` reaches `required` first at `right=5` (`'C'`), giving window `"ADOBEC"` (length 6) → `bestLen=6, bestStart=0`; shrinking removes `'A'` (`left=1`), which drops `formed` back to 2. Expanding again, `formed` reaches 3 next at `right=10` (`'A'`), giving window `s[1..10]="DOBECODEBA"` (length 10, not better); shrinking removes non-required and required characters down to `s[5..10]="CODEBA"` (length 6, ties but not strictly smaller) before removing `'C'` drops `formed` to 2 (`left=6`). Expanding once more, `formed` reaches 3 at `right=12` (`'C'`), giving window `s[6..12]="ODEBANC"` (length 7); shrinking tightens it through `"DEBANC"` (6), `"EBANC"` (5, new best `bestLen=5, bestStart=8`), and `"BANC"` (4, new best `bestLen=4, bestStart=9`), before removing `'B'` drops `formed` to 2. The loop ends with `bestStart=9, bestLen=4` → `s.Substring(9,4) = "BANC"`, matching the expected output.

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

**Logic (Steps):**
1. Try every substring `s[i..j]` by iterating over all `(i, j)` pairs.
2. For each candidate, call `IsSubsequence` to check whether `t`'s characters appear in order (not necessarily contiguously) within the candidate.
3. If valid and shorter than the current best, update `bestLen`/`best`.
4. Repeat for every `(i, j)` pair, returning the shortest valid window found.

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

**Walkthrough:** On `S = "abcdebdde"`, `T = "bde"`: every length-3 candidate (`"bcd"`, `"cde"`, `"deb"`, `"ebd"`, `"bdd"`, `"dde"`, ...) fails `IsSubsequence` because none contains `'b'` then `'d'` then `'e'` in that exact order. The candidate `"bcde"` (indices 1-4) does contain `b`, `d`, `e` in order (skipping `c`), so it's recorded as valid with length 4, and no shorter valid window exists. `best = "bcde"`, matching the expected output.

---

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

**Logic (Steps):**
1. From the current `right`, run the expand phase: advance `right` through `s` and `ti` through `t`, matching characters in order until `ti` reaches `m` (all of `t` matched) or `s` runs out.
2. If `s` ran out before matching all of `t`, no further candidates are possible — stop.
3. Otherwise, run the contract phase: walk backward from `end = right`, matching `t` in reverse, to find the tightest `start` such that `t` is still a subsequence of `s[start..end]`.
4. If `end - start + 1` beats the current best, record `bestStart`/`bestLen`.
5. Resume the expand phase from `right = start + 1` (any better window must start after this tightest start), and repeat until `s` is exhausted.

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

**Walkthrough:** On `S = "abcdebdde"`, `T = "bde"` (`m=3`). First expand phase: scanning from `right=0`, characters match `t` in order at `s[1]='b'`, `s[3]='d'`, `s[4]='e'`, completing the subsequence at `end=4`. Contract phase walks backward from `end=4` matching `t` in reverse (`e` at 4, `d` at 3, `b` at 1 — `'c'` at index 2 is skipped since it's not needed), giving `start=1`. Window `s[1..4]="bcde"`, length 4 → `bestLen=4, bestStart=1`. Resuming from `right=start+1=2`, the next expand phase completes another match at `end=8` (`s[5]='b'`, `s[6]='d'`, `s[8]='e'`), and its contract phase gives `start=5`, window `s[5..8]="bdde"`, length 4 — a tie, not strictly smaller, so the best stays unchanged. Resuming from `right=6`, no further match of `t` completes before `S` runs out, so the scan stops. Final answer: `bestStart=1, bestLen=4` → `s.Substring(1,4) = "bcde"`, matching the expected output.

---
