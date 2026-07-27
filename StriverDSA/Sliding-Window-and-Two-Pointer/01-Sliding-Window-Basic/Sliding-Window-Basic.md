# Sliding Window and Two Pointer — Basic Problems

## Concept: Sliding Window

The sliding window technique is used to solve problems that involve finding a contiguous subarray or substring that satisfies some condition, without having to re-examine every possible subarray/substring from scratch (which would normally cost O(n^2) or O(n^3) time).

The core idea is to maintain a "window" defined by two pointers, `left` and `right`, that both start at the beginning of the array/string:

1. **Expand** — Move the `right` pointer forward one step at a time, adding the new element into the window's running state (a count, a sum, a `HashSet`/`Dictionary` of characters seen, etc.).
2. **Check the constraint** — After adding the new element, check whether the window still satisfies the problem's constraint (e.g., "at most K distinct characters", "no duplicate characters", "at most K zeros").
3. **Shrink** — If the constraint is violated, move the `left` pointer forward (removing elements from the window's state) until the constraint is satisfied again.
4. **Track the answer** — After every expansion (and any necessary shrinking), update a running answer — usually `Math.Max(answer, right - left + 1)` for "longest" problems, or accumulate counts for "count of subarrays" problems.

Because `left` and `right` each only move forward and never backward, the total work done across the whole array is O(n) + O(n) = O(n), rather than O(n^2).

**When it applies:**
- The problem asks for something about a **contiguous** subarray or substring (not arbitrary subsequences).
- There is a **monotonic constraint** — once a window of size `w` violates the condition, every larger window starting at the same `left` also violates it, and shrinking from the left is guaranteed to eventually restore validity. This monotonicity is what guarantees `left` never needs to move backward.
- Typical signals in the problem statement: "longest/shortest substring/subarray such that...", "at most K distinct...", "no more than K...", "all characters equal after at most K replacements", etc.

There are two common flavors:
- **Variable-size window** (used in all problems below): the window grows and shrinks as needed; used for "longest subarray/substring satisfying X".
- **Fixed-size window**: the window size is given directly (e.g., "subarray of size K"); `left` and `right` move together once the window reaches size K.

---

## 1. Longest Substring Without Repeating Characters

**Problem Statement:** Given a string `s`, find the length of the longest substring without repeating characters.

**Example:**
- Input: `s = "abcabcbb"`
- Output: `3`
- Explanation: The answer is `"abc"`, with a length of 3. Note that `"bca"` and `"cab"` are also valid substrings of length 3, but not longer.

**Brute Force Approach:** Check every substring and verify whether all its characters are unique.

**Logic (Steps):**
1. Fix a starting index `i` and try every ending index `j >= i`.
2. For each `(i, j)` pair, call `AllUnique` to scan the substring `s[i..j]` using a `HashSet<char>`.
3. `AllUnique` returns `false` as soon as it finds a character it has already added to the set (a repeat).
4. If the substring is all unique, update `maxLen` with `j - i + 1` if it's larger.
5. Repeat for every `(i, j)` pair, giving the longest unique-character substring length.

```csharp
public class Solution {
    public int LengthOfLongestSubstringBruteForce(string s) {
        int n = s.Length;
        int maxLen = 0;

        for (int i = 0; i < n; i++) {
            for (int j = i; j < n; j++) {
                if (AllUnique(s, i, j)) {
                    maxLen = Math.Max(maxLen, j - i + 1);
                }
            }
        }

        return maxLen;
    }

    private bool AllUnique(string s, int start, int end) {
        HashSet<char> seen = new HashSet<char>();
        for (int k = start; k <= end; k++) {
            if (!seen.Add(s[k])) {
                return false;
            }
        }
        return true;
    }
}
```

Time Complexity: O(n^3) — O(n^2) substrings, each verified in O(n).
Space Complexity: O(min(n, charset size)) for the `HashSet` used during verification.

**Walkthrough:** On `s = "abcabcbb"`, when `i = 0`: `j = 0` gives `"a"` (unique, maxLen=1), `j = 1` gives `"ab"` (unique, maxLen=2), `j = 2` gives `"abc"` (unique, maxLen=3), `j = 3` gives `"abca"` — `AllUnique` finds `'a'` already in the set and returns `false`, so no update, and larger `j` values also fail. The same process repeats for `i = 1, 2, ...`, but no substring beats length 3. Final `maxLen = 3`, matching the expected output.

---

**Optimized Approach:** Use a variable-size sliding window with a `HashSet<char>` to track the characters currently in the window. Expand `right`; whenever the incoming character is already in the window, shrink `left` (removing characters from the set) until the duplicate is gone.

**Logic (Steps):**
1. Maintain a `HashSet<char> window` holding the characters currently between `left` and `right`.
2. Move `right` forward one step at a time.
3. Before adding `s[right]`, while it's already in the set, remove `s[left]` from the set and advance `left` — this shrinks the window until the duplicate is gone (the invariant "no repeated characters in the window" is restored).
4. Add `s[right]` to the set.
5. Update `maxLen = Math.Max(maxLen, right - left + 1)` after every expansion.

```csharp
public class Solution {
    public int LengthOfLongestSubstring(string s) {
        HashSet<char> window = new HashSet<char>();
        int left = 0;
        int maxLen = 0;

        for (int right = 0; right < s.Length; right++) {
            while (window.Contains(s[right])) {
                window.Remove(s[left]);
                left++;
            }
            window.Add(s[right]);
            maxLen = Math.Max(maxLen, right - left + 1);
        }

        return maxLen;
    }
}
```

Time Complexity: O(n), Space Complexity: O(min(n, charset size)).

**Walkthrough:** Dry run on `s = "abcabcbb"`. `right=0,1,2` add `'a','b','c'` with no duplicates → window `{a,b,c}`, `maxLen` grows to 3. At `right=3`, `'a'` is already in the window, so `left` shrinks past `s[0]='a'` (left=1), then `'a'` is added → window `{b,c,a}`, length 3. At `right=4`, `'b'` duplicates → shrink past `s[1]='b'` (left=2), add `'b'` → window `{c,a,b}`, length 3. At `right=5`, `'c'` duplicates → shrink past `s[2]='c'` (left=3), add `'c'` → length 3. At `right=6`, `'b'` duplicates → shrink past `s[3]='a'` and `s[4]='b'` (left=5), add `'b'` → window `{c,b}`, length 2. At `right=7`, `'b'` duplicates again → shrink past `s[5]='c'` and `s[6]='b'` (left=7), add `'b'` → window `{b}`, length 1. `maxLen` never exceeds 3, so the function returns `3`, matching the expected output.

---

## 2. Max Consecutive Ones III

**Problem Statement:** Given a binary array `nums` and an integer `k`, return the maximum number of consecutive `1`s in the array if you can flip at most `k` `0`s to `1`s.

**Example:**
- Input: `nums = [1,1,1,0,0,0,1,1,1,1,0]`, `k = 2`
- Output: `6`
- Explanation: Flip the two `0`s at indices 3 and 4 (or the trailing ones), giving `[1,1,1,1,1,1,1,1,1,1,0]`. The longest run of consecutive `1`s is 6, from index 5 through index 10 (`0,0,1,1,1,1,1,1,1,1,0` → underlined `1,1,1,1,1,1` at indices 5-10).

**Brute Force Approach:** For every subarray, count zeros; keep it valid if zero count `<= k`.

**Logic (Steps):**
1. Fix a starting index `i` and try every ending index `j >= i`.
2. Maintain a running `zeroCount` for the subarray `nums[i..j]`, incrementing it whenever `nums[j] == 0`.
3. If `zeroCount <= k`, the subarray is valid — update `maxLen` with `j - i + 1`.
4. As soon as `zeroCount > k`, break out of the inner loop (extending further from this `i` can only add more zeros).
5. Repeat for every starting index `i`.

```csharp
public class Solution {
    public int LongestOnesBruteForce(int[] nums, int k) {
        int n = nums.Length;
        int maxLen = 0;

        for (int i = 0; i < n; i++) {
            int zeroCount = 0;
            for (int j = i; j < n; j++) {
                if (nums[j] == 0) {
                    zeroCount++;
                }
                if (zeroCount <= k) {
                    maxLen = Math.Max(maxLen, j - i + 1);
                } else {
                    break;
                }
            }
        }

        return maxLen;
    }
}
```

Time Complexity: O(n^2) — for each starting index `i`, we scan forward until the zero-count exceeds `k`.
Space Complexity: O(1).

**Walkthrough:** On `nums = [1,1,1,0,0,0,1,1,1,1,0]`, `k = 2`: starting at `i=0`, zeros accumulate as `j` advances — `zeroCount` stays `<= 2` through `j=4` (window `[1,1,1,0,0]`, length 5), but at `j=5` (third zero) `zeroCount=3 > k`, so the inner loop breaks. Trying every other starting index `i` similarly caps out — the best is `i=4`, which reaches `j=9` (`[0,0,1,1,1,1,1,1]`... actually `nums[4..9]`, containing 2 zeros and 4 ones, length 6) before the zero at index 10 would push the count to 3. `maxLen` ends at `6`, matching the expected output.

---

**Optimized Approach:** Use a variable-size sliding window that tracks the number of zeros currently inside it. Expand `right`; if `zeroCount > k`, shrink `left` until `zeroCount <= k` again.

**Logic (Steps):**
1. Maintain `zeroCount`, the number of zeros currently inside the window `[left, right]`.
2. Move `right` forward; if `nums[right] == 0`, increment `zeroCount`.
3. While `zeroCount > k` (too many zeros to flip), shrink from the left: if `nums[left] == 0`, decrement `zeroCount`, then advance `left`. This restores the invariant `zeroCount <= k`.
4. After the shrink step, the window `[left, right]` is guaranteed valid — update `maxLen = Math.Max(maxLen, right - left + 1)`.
5. Repeat until `right` reaches the end of the array.

```csharp
public class Solution {
    public int LongestOnes(int[] nums, int k) {
        int left = 0;
        int zeroCount = 0;
        int maxLen = 0;

        for (int right = 0; right < nums.Length; right++) {
            if (nums[right] == 0) {
                zeroCount++;
            }

            while (zeroCount > k) {
                if (nums[left] == 0) {
                    zeroCount--;
                }
                left++;
            }

            maxLen = Math.Max(maxLen, right - left + 1);
        }

        return maxLen;
    }
}
```

Time Complexity: O(n), Space Complexity: O(1).

**Walkthrough:** Dry run on `nums = [1,1,1,0,0,0,1,1,1,1,0]`, `k = 2`. `right=0..2` are all 1s, `zeroCount` stays 0, window grows to length 3. `right=3,4` are zeros, `zeroCount` becomes 1 then 2 — still `<= k`, window grows to length 5 (`maxLen=5`). At `right=5`, a third zero makes `zeroCount=3 > 2`, so `left` shrinks past three 1s and the zero at index 3, dropping `zeroCount` back to 2 and landing `left=4`. From `right=6` to `right=9` all are 1s, so the window grows from length 4 up to length 6 (`left=4..right=9`, `maxLen=6`). At `right=10`, another zero pushes `zeroCount` to 3, so `left` shrinks past `nums[4]=0`, dropping `zeroCount` to 2 and `left=5`, giving window length 6 again. The maximum window length recorded is `6`, matching the expected output.

---

## 3. Fruits Into Baskets (Longest Subarray with At Most 2 Distinct Types)

**Problem Statement:** You are visiting a farm with a row of fruit trees, given as an array `fruits` where `fruits[i]` is the type of fruit the `i`-th tree produces. You have exactly 2 baskets, and each basket can hold only a single type of fruit (unlimited quantity). Starting from any tree, you must pick exactly one fruit from every tree while moving to the right, stopping once you encounter a fruit type that doesn't fit in either basket. Return the maximum number of fruits you can pick — equivalently, the length of the longest contiguous subarray containing at most 2 distinct values.

**Example:**
- Input: `fruits = [1,2,1,2,3]`
- Output: `4`
- Explanation: The longest subarray with at most 2 distinct fruit types is `[1,2,1,2]` (indices 0-3), giving 4 fruits. Adding index 4 (fruit type 3) would introduce a third type.

**Brute Force Approach:** For every subarray, count the number of distinct fruit types and check if it is at most 2.

**Logic (Steps):**
1. Fix a starting index `i` and try every ending index `j >= i`.
2. Add `fruits[j]` into a `HashSet<int> types` for the subarray `fruits[i..j]`.
3. If `types.Count <= 2`, the subarray is valid — update `maxLen` with `j - i + 1`.
4. As soon as `types.Count > 2`, break the inner loop (a third type can't be undone by extending further).
5. Repeat for every starting index `i`.

```csharp
public class Solution {
    public int TotalFruitBruteForce(int[] fruits) {
        int n = fruits.Length;
        int maxLen = 0;

        for (int i = 0; i < n; i++) {
            HashSet<int> types = new HashSet<int>();
            for (int j = i; j < n; j++) {
                types.Add(fruits[j]);
                if (types.Count <= 2) {
                    maxLen = Math.Max(maxLen, j - i + 1);
                } else {
                    break;
                }
            }
        }

        return maxLen;
    }
}
```

Time Complexity: O(n^2) — for each start index, extend until distinct count exceeds 2.
Space Complexity: O(1) (at most 3 distinct types are ever stored in the set before breaking).

**Walkthrough:** On `fruits = [1,2,1,2,3]`: starting at `i=0`, `types` grows as `{1}`, `{1,2}`, stays `{1,2}` at `j=2,3` (length 4, `maxLen=4`), then at `j=4` type `3` makes `types.Count=3`, so the inner loop breaks. Starting at later indices (`i=1,2,3,4`) can reach at most length 3 or less before hitting a third type. `maxLen` ends at `4`, matching the expected output.

---

**Optimized Approach:** Use a variable-size sliding window with a `Dictionary<int,int>` mapping fruit type to its count within the window. Expand `right`; whenever the dictionary has more than 2 keys, shrink `left`, decrementing counts and removing keys whose count drops to 0.

**Logic (Steps):**
1. Maintain a `Dictionary<int,int> basket` mapping each fruit type currently in the window to its count.
2. Move `right` forward; increment `basket[fruits[right]]` (adding a new key if the type is new).
3. While `basket.Count > 2` (more than 2 distinct types), shrink from the left: decrement `basket[fruits[left]]`, remove the key entirely if its count hits 0, then advance `left`. This restores the invariant of at most 2 distinct types.
4. After shrinking, update `maxLen = Math.Max(maxLen, right - left + 1)`.
5. Repeat until `right` reaches the end of the array.

```csharp
public class Solution {
    public int TotalFruit(int[] fruits) {
        Dictionary<int, int> basket = new Dictionary<int, int>();
        int left = 0;
        int maxLen = 0;

        for (int right = 0; right < fruits.Length; right++) {
            int type = fruits[right];
            if (!basket.ContainsKey(type)) {
                basket[type] = 0;
            }
            basket[type]++;

            while (basket.Count > 2) {
                int leftType = fruits[left];
                basket[leftType]--;
                if (basket[leftType] == 0) {
                    basket.Remove(leftType);
                }
                left++;
            }

            maxLen = Math.Max(maxLen, right - left + 1);
        }

        return maxLen;
    }
}
```

Time Complexity: O(n), Space Complexity: O(1) (dictionary holds at most 3 keys at any time).

**Walkthrough:** Dry run on `fruits = [1,2,1,2,3]`. `right=0`: `basket={1:1}`, length 1. `right=1`: `basket={1:1,2:1}`, length 2. `right=2`: `basket={1:2,2:1}`, still 2 keys, length 3. `right=3`: `basket={1:2,2:2}`, still 2 keys, length 4 (`maxLen=4`). `right=4`: type `3` is added, `basket={1:2,2:2,3:1}` has 3 keys, so `left` shrinks: decrement `fruits[0]=1` to count 1 (key stays, still 3 keys, `left=1`); decrement `fruits[1]=2` to count 1 (key stays, still 3 keys, `left=2`); decrement `fruits[2]=1` to count 0 (key `1` removed, `basket.Count=2`, `left=3`) — stop. New window is `left=3..right=4`, length 2, not larger than `maxLen`. Final `maxLen = 4`, matching the expected output.

---

## 4. Longest Repeating Character Replacement

**Problem Statement:** Given a string `s` consisting of uppercase English letters and an integer `k`, you can choose up to `k` characters in the string and replace each with any other uppercase English letter. Return the length of the longest substring containing the same letter after performing at most `k` replacements.

**Example:**
- Input: `s = "AABABBA"`, `k = 1`
- Output: `4`
- Explanation: Replace the one `'B'` at index 3 with `'A'` to get `"AAAABBA"`. The substring `"AAAA"` (indices 0-3) has length 4, all the same character, using only 1 replacement.

**Brute Force Approach:** For every substring, find the most frequent character's count; the substring is valid if `(length - maxFreqCount) <= k` (i.e., the number of characters that need replacing does not exceed `k`).

**Logic (Steps):**
1. Fix a starting index `i` and try every ending index `j >= i`.
2. Maintain a frequency array `freq[26]` for the substring `s[i..j]`, and `maxFreq`, the highest single-character count seen so far in this substring.
3. The number of characters needing replacement is `windowLen - maxFreq`; if that's `<= k`, update `maxLen`.
4. As soon as `windowLen - maxFreq > k`, break the inner loop.
5. Repeat for every starting index `i`.

```csharp
public class Solution {
    public int CharacterReplacementBruteForce(string s, int k) {
        int n = s.Length;
        int maxLen = 0;

        for (int i = 0; i < n; i++) {
            int[] freq = new int[26];
            int maxFreq = 0;
            for (int j = i; j < n; j++) {
                freq[s[j] - 'A']++;
                maxFreq = Math.Max(maxFreq, freq[s[j] - 'A']);
                int windowLen = j - i + 1;
                if (windowLen - maxFreq <= k) {
                    maxLen = Math.Max(maxLen, windowLen);
                } else {
                    break;
                }
            }
        }

        return maxLen;
    }
}
```

Time Complexity: O(n^2) — for each start index, extend right while recomputing the max frequency, until the replacement count exceeds `k`.
Space Complexity: O(26) = O(1) for the frequency array.

**Walkthrough:** On `s = "AABABBA"`, `k = 1`: starting at `i=0`, the substring grows as `"A"`, `"AA"`, `"AAB"` (`maxFreq=2`, `windowLen-maxFreq=1<=1`, length 3), `"AABA"` (`maxFreq=3`, replacements=1, length 4, `maxLen=4`), `"AABAB"` (`maxFreq=3`, replacements=2>1, break). No other starting index reaches a longer valid substring. Final `maxLen = 4`, matching the expected output.

---

**Optimized Approach:** Use a variable-size sliding window with a `Dictionary<char,int>` tracking character frequencies in the window, plus a running `maxFreq` (count of the most frequent character seen in any window so far — it never needs to decrease, only the window shrinks when invalid). The window is valid when `windowLength - maxFreq <= k`; otherwise shrink `left`.

**Logic (Steps):**
1. Maintain a `Dictionary<char,int> freq` of character counts in the window `[left, right]`, and `maxFreq`, the highest count of any single character seen in any window so far.
2. Move `right` forward, increment `freq[s[right]]`, and update `maxFreq = Math.Max(maxFreq, freq[s[right]])`.
3. The window needs `windowLen - maxFreq` replacements to become uniform; if that exceeds `k`, shrink by exactly one from the left: decrement `freq[s[left]]` and advance `left` (note `maxFreq` is deliberately not recomputed here, since the window only ever shrinks by 1, so it can't overstate the true answer).
4. Update `maxLen = Math.Max(maxLen, right - left + 1)` after every step.
5. Repeat until `right` reaches the end of the string.

```csharp
public class Solution {
    public int CharacterReplacement(string s, int k) {
        Dictionary<char, int> freq = new Dictionary<char, int>();
        int left = 0;
        int maxFreq = 0;
        int maxLen = 0;

        for (int right = 0; right < s.Length; right++) {
            char c = s[right];
            if (!freq.ContainsKey(c)) {
                freq[c] = 0;
            }
            freq[c]++;
            maxFreq = Math.Max(maxFreq, freq[c]);

            int windowLen = right - left + 1;
            if (windowLen - maxFreq > k) {
                char leftChar = s[left];
                freq[leftChar]--;
                left++;
            }

            maxLen = Math.Max(maxLen, right - left + 1);
        }

        return maxLen;
    }
}
```

Time Complexity: O(n), Space Complexity: O(26) = O(1).

**Walkthrough:** Dry run on `s = "AABABBA"`, `k = 1`. `right=0,1`: `freq={A:2}`, `maxFreq=2`, `windowLen=2`, replacements=0, `maxLen=2`. `right=2` (`'B'`): `freq={A:2,B:1}`, `maxFreq=2`, `windowLen=3`, replacements=1<=1, `maxLen=3`. `right=3` (`'A'`): `freq={A:3,B:1}`, `maxFreq=3`, `windowLen=4`, replacements=1<=1, `maxLen=4`. `right=4` (`'B'`): `freq={A:3,B:2}`, `maxFreq` stays 3, `windowLen=5`, replacements=2>1 → shrink: decrement `s[0]='A'` to 2, `left=1`; window is now length 4. `right=5` (`'B'`): `freq={A:2,B:3}`, `maxFreq=3`, `windowLen=5`, replacements=2>1 → shrink: decrement `s[1]='A'` to 1, `left=2`; window length 4. `right=6` (`'A'`): `freq={A:2,B:3}`, `maxFreq=3`, `windowLen=5`, replacements=2>1 → shrink: decrement `s[2]='B'` to 2, `left=3`; window length 4. `maxLen` never exceeds 4, so the function returns `4`, matching the expected output.

---

## 5. Longest Substring with At Most K Distinct Characters

**Problem Statement:** Given a string `s` and an integer `k`, find the length of the longest substring that contains at most `k` distinct characters.

**Example:**
- Input: `s = "eceba"`, `k = 2`
- Output: `3`
- Explanation: The longest substring with at most 2 distinct characters is `"ece"`, with length 3.

**Brute Force Approach:** For every substring, count distinct characters and check if it is at most `k`.

**Logic (Steps):**
1. Fix a starting index `i` and try every ending index `j >= i`.
2. Add `s[j]` into a `HashSet<char> distinct` for the substring `s[i..j]`.
3. If `distinct.Count <= k`, the substring is valid — update `maxLen` with `j - i + 1`.
4. As soon as `distinct.Count > k`, break the inner loop.
5. Repeat for every starting index `i`.

```csharp
public class Solution {
    public int LongestKDistinctBruteForce(string s, int k) {
        int n = s.Length;
        int maxLen = 0;

        for (int i = 0; i < n; i++) {
            HashSet<char> distinct = new HashSet<char>();
            for (int j = i; j < n; j++) {
                distinct.Add(s[j]);
                if (distinct.Count <= k) {
                    maxLen = Math.Max(maxLen, j - i + 1);
                } else {
                    break;
                }
            }
        }

        return maxLen;
    }
}
```

Time Complexity: O(n^2) — for each start index, extend right until distinct count exceeds `k`.
Space Complexity: O(min(n, charset size)) for the `HashSet`.

**Walkthrough:** On `s = "eceba"`, `k = 2`: starting at `i=0`, `distinct` grows as `{e}`, `{e,c}`, stays `{e,c}` at `j=2` (`"ece"`, length 3, `maxLen=3`), then at `j=3` (`'b'`) `distinct.Count=3`, so the inner loop breaks. Starting indices `i=1,2,3,4` reach at most length 2 or 3, none exceeding 3. Final `maxLen = 3`, matching the expected output.

---

**Optimized Approach:** Use a variable-size sliding window with a `Dictionary<char,int>` tracking character frequencies. Expand `right`; whenever the dictionary has more than `k` keys, shrink `left`, decrementing counts and removing keys whose count reaches 0, until at most `k` distinct characters remain.

**Logic (Steps):**
1. Handle the edge case `k == 0` directly by returning `0` (no distinct characters allowed).
2. Maintain a `Dictionary<char,int> freq` mapping each character in the window `[left, right]` to its count.
3. Move `right` forward, incrementing `freq[s[right]]`.
4. While `freq.Count > k`, shrink from the left: decrement `freq[s[left]]`, remove the key entirely if its count hits 0, then advance `left`, until at most `k` distinct characters remain.
5. Update `maxLen = Math.Max(maxLen, right - left + 1)` after every step.

```csharp
public class Solution {
    public int LongestKDistinct(string s, int k) {
        if (k == 0) {
            return 0;
        }

        Dictionary<char, int> freq = new Dictionary<char, int>();
        int left = 0;
        int maxLen = 0;

        for (int right = 0; right < s.Length; right++) {
            char c = s[right];
            if (!freq.ContainsKey(c)) {
                freq[c] = 0;
            }
            freq[c]++;

            while (freq.Count > k) {
                char leftChar = s[left];
                freq[leftChar]--;
                if (freq[leftChar] == 0) {
                    freq.Remove(leftChar);
                }
                left++;
            }

            maxLen = Math.Max(maxLen, right - left + 1);
        }

        return maxLen;
    }
}
```

Time Complexity: O(n), Space Complexity: O(min(n, charset size)).

**Walkthrough:** Dry run on `s = "eceba"`, `k = 2`. `right=0` (`'e'`): `freq={e:1}`, length 1. `right=1` (`'c'`): `freq={e:1,c:1}`, 2 keys, length 2. `right=2` (`'e'`): `freq={e:2,c:1}`, still 2 keys, length 3 (`maxLen=3`, window `"ece"`). `right=3` (`'b'`): `freq={e:2,c:1,b:1}` has 3 keys → shrink: decrement `s[0]='e'` to 1 (key stays, still 3 keys, `left=1`); decrement `s[1]='c'` to 0 (key removed, `freq.Count=2`, `left=2`) — stop; window is now length 2. `right=4` (`'a'`): `freq={e:1,b:1,a:1}` has 3 keys → shrink: decrement `s[2]='e'` to 0 (key removed, `freq.Count=2`, `left=3`) — stop; window length 2. `maxLen` never exceeds 3, so the function returns `3`, matching the expected output.

---
