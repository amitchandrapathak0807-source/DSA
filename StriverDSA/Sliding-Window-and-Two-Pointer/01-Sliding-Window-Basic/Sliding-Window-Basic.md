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

**Optimized Approach:** Use a variable-size sliding window with a `HashSet<char>` to track the characters currently in the window. Expand `right`; whenever the incoming character is already in the window, shrink `left` (removing characters from the set) until the duplicate is gone.

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

**Explanation (dry run on `s = "abcabcbb"`):**

| right | char | action | window (HashSet) | left | window length | maxLen |
|---|---|---|---|---|---|---|
| 0 | a | add 'a' | {a} | 0 | 1 | 1 |
| 1 | b | add 'b' | {a,b} | 0 | 2 | 2 |
| 2 | c | add 'c' | {a,b,c} | 0 | 3 | 3 |
| 3 | a | 'a' already in window → remove s[0]='a', left=1; now 'a' not in window → add | {b,c,a} | 1 | 3 | 3 |
| 4 | b | 'b' already in window → remove s[1]='b', left=2; add 'b' | {c,a,b} | 2 | 3 | 3 |
| 5 | c | 'c' already in window → remove s[2]='c', left=3; add 'c' | {a,b,c} | 3 | 3 | 3 |
| 6 | b | 'b' already in window → remove s[3]='a', left=4, 'b' still in window → remove s[4]='b', left=5; add 'b' | {c,b} | 5 | 2 | 3 |
| 7 | b | 'b' already in window → remove s[5]='c', left=6, 'b' still in window → remove s[6]='b', left=7; add 'b' | {b} | 7 | 1 | 3 |

Final answer: `maxLen = 3` (from the window `"abc"` seen at `right = 2`).

---

## 2. Max Consecutive Ones III

**Problem Statement:** Given a binary array `nums` and an integer `k`, return the maximum number of consecutive `1`s in the array if you can flip at most `k` `0`s to `1`s.

**Example:**
- Input: `nums = [1,1,1,0,0,0,1,1,1,1,0]`, `k = 2`
- Output: `6`
- Explanation: Flip the two `0`s at indices 3 and 4 (or the trailing ones), giving `[1,1,1,1,1,1,1,1,1,1,0]`. The longest run of consecutive `1`s is 6, from index 5 through index 10 (`0,0,1,1,1,1,1,1,1,1,0` → underlined `1,1,1,1,1,1` at indices 5-10).

**Brute Force Approach:** For every subarray, count zeros; keep it valid if zero count `<= k`.

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

**Optimized Approach:** Use a variable-size sliding window that tracks the number of zeros currently inside it. Expand `right`; if `zeroCount > k`, shrink `left` until `zeroCount <= k` again.

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

**Explanation (dry run on `nums = [1,1,1,0,0,0,1,1,1,1,0]`, `k = 2`):**

| right | nums[right] | zeroCount after add | shrink? | left after shrink | window length | maxLen |
|---|---|---|---|---|---|---|
| 0 | 1 | 0 | no | 0 | 1 | 1 |
| 1 | 1 | 0 | no | 0 | 2 | 2 |
| 2 | 1 | 0 | no | 0 | 3 | 3 |
| 3 | 0 | 1 | no (1<=2) | 0 | 4 | 4 |
| 4 | 0 | 2 | no (2<=2) | 0 | 5 | 5 |
| 5 | 0 | 3 | yes: 3>2 → remove nums[0]=1 (zeroCount stays 3), left=1; still 3>2 → remove nums[1]=1, left=2; still 3>2 → remove nums[2]=1, left=3; still 3>2 → remove nums[3]=0, zeroCount=2, left=4; 2<=2 stop | 4 | 2 | 5 |
| 6 | 1 | 2 | no | 4 | 3 | 5 |
| 7 | 1 | 2 | no | 4 | 4 | 5 |
| 8 | 1 | 2 | no | 4 | 5 | 5 |
| 9 | 1 | 2 | no | 4 | 6 | 6 |
| 10 | 0 | 3 | yes: 3>2 → remove nums[4]=0, zeroCount=2, left=5; 2<=2 stop | 5 | 6 | 6 |

Final answer: `maxLen = 6`, matching the window `nums[4..9] = [0,0,1,1,1,1,1]`... (window from `left=4` to `right=9`, containing two flippable zeros and six ones), the longest valid window found.

---

## 3. Fruits Into Baskets (Longest Subarray with At Most 2 Distinct Types)

**Problem Statement:** You are visiting a farm with a row of fruit trees, given as an array `fruits` where `fruits[i]` is the type of fruit the `i`-th tree produces. You have exactly 2 baskets, and each basket can hold only a single type of fruit (unlimited quantity). Starting from any tree, you must pick exactly one fruit from every tree while moving to the right, stopping once you encounter a fruit type that doesn't fit in either basket. Return the maximum number of fruits you can pick — equivalently, the length of the longest contiguous subarray containing at most 2 distinct values.

**Example:**
- Input: `fruits = [1,2,1,2,3]`
- Output: `4`
- Explanation: The longest subarray with at most 2 distinct fruit types is `[1,2,1,2]` (indices 0-3), giving 4 fruits. Adding index 4 (fruit type 3) would introduce a third type.

**Brute Force Approach:** For every subarray, count the number of distinct fruit types and check if it is at most 2.

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

**Optimized Approach:** Use a variable-size sliding window with a `Dictionary<int,int>` mapping fruit type to its count within the window. Expand `right`; whenever the dictionary has more than 2 keys, shrink `left`, decrementing counts and removing keys whose count drops to 0.

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

**Explanation:** The window always tracks at most 2 distinct fruit types via the dictionary's key count. When a third type is introduced (`basket.Count > 2`), the left pointer advances, decrementing the count of the leftmost fruit type and removing it from the dictionary entirely once its count reaches zero, thereby shrinking the window back down to exactly 2 distinct types before continuing to expand `right`.

---

## 4. Longest Repeating Character Replacement

**Problem Statement:** Given a string `s` consisting of uppercase English letters and an integer `k`, you can choose up to `k` characters in the string and replace each with any other uppercase English letter. Return the length of the longest substring containing the same letter after performing at most `k` replacements.

**Example:**
- Input: `s = "AABABBA"`, `k = 1`
- Output: `4`
- Explanation: Replace the one `'B'` at index 3 with `'A'` to get `"AAAABBA"`. The substring `"AAAA"` (indices 0-3) has length 4, all the same character, using only 1 replacement.

**Brute Force Approach:** For every substring, find the most frequent character's count; the substring is valid if `(length - maxFreqCount) <= k` (i.e., the number of characters that need replacing does not exceed `k`).

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

**Optimized Approach:** Use a variable-size sliding window with a `Dictionary<char,int>` tracking character frequencies in the window, plus a running `maxFreq` (count of the most frequent character seen in any window so far — it never needs to decrease, only the window shrinks when invalid). The window is valid when `windowLength - maxFreq <= k`; otherwise shrink `left`.

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

**Explanation:** `maxFreq` tracks the highest frequency of any single character observed within any window during the scan. A window of length `windowLen` needs at most `windowLen - maxFreq` replacements to become uniform. If that exceeds `k`, the window shrinks by one from the left (note `maxFreq` is not recomputed on shrink — it's fine because the window size only ever shrinks by 1 per invalid step, so the answer, which tracks the maximum valid window length, is never overstated). `maxLen` records the best valid window length seen.

---

## 5. Longest Substring with At Most K Distinct Characters

**Problem Statement:** Given a string `s` and an integer `k`, find the length of the longest substring that contains at most `k` distinct characters.

**Example:**
- Input: `s = "eceba"`, `k = 2`
- Output: `3`
- Explanation: The longest substring with at most 2 distinct characters is `"ece"`, with length 3.

**Brute Force Approach:** For every substring, count distinct characters and check if it is at most `k`.

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

**Optimized Approach:** Use a variable-size sliding window with a `Dictionary<char,int>` tracking character frequencies. Expand `right`; whenever the dictionary has more than `k` keys, shrink `left`, decrementing counts and removing keys whose count reaches 0, until at most `k` distinct characters remain.

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

**Explanation:** This mirrors the Fruits Into Baskets pattern, generalized to `k` distinct characters instead of a fixed 2. The `Dictionary<char,int>` key count represents the number of distinct characters currently in the window; whenever it exceeds `k`, the left pointer advances, shrinking the window until the distinct-character constraint is satisfied again, and `maxLen` tracks the longest valid window observed. On `s = "eceba"`, `k = 2`: the window grows to `"ec"` (2 distinct, len 2), then `"ece"` (still 2 distinct since 'e' repeats, len 3, new max), then adding `'b'` makes 3 distinct so `left` shrinks past `'e'` and `'c'` until the window is `"eb"` wait — actually shrinks to `"cb"`... in general terms, whenever a third distinct character appears, characters are dropped from the left until only 2 distinct characters remain, and the maximum window length of 3 (`"ece"`) is retained as the final answer.
