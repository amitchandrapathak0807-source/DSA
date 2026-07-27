# Array — Frequency and Hashing

## 1. Majority Element (> n/2 times)

### 1. Majority Element

**Problem Statement:** Given an array `nums` of size `n`, find the majority element — the element that appears more than `⌊n/2⌋` times. It is guaranteed that the array always has a majority element.

**Example:**
- Input: `nums = [2, 2, 1, 1, 1, 2, 2]`
- Output: `2`
- Explanation: The array has 7 elements, so an element must appear more than 3 times to be the majority element. `2` appears 4 times, so `2` is the majority element.

**Brute Force Approach:** For every distinct element, count its occurrences using a hashmap (or nested loops) and return the element whose frequency exceeds `n/2`.

**Logic (Steps):**
1. Store `n = nums.Length` and create an empty `freq` dictionary.
2. Iterate through each `num` in `nums`.
3. Increment `freq[num]` (initialize to `1` if not seen before).
4. Immediately after updating, check if `freq[num] > n / 2`; if so, return `num` right away.
5. If the loop finishes without any element crossing the threshold, return `-1` as a fallback (won't happen given the problem's guarantee).

```csharp
public int MajorityElementBrute(int[] nums)
{
    int n = nums.Length;
    Dictionary<int, int> freq = new Dictionary<int, int>();

    foreach (int num in nums)
    {
        if (freq.ContainsKey(num))
            freq[num]++;
        else
            freq[num] = 1;

        if (freq[num] > n / 2)
            return num;
    }

    return -1; // no majority element found
}
```
Time Complexity: `O(n)` on average for a single pass with dictionary lookups (each insert/lookup is `O(1)` average), but if implemented with nested loops (counting each element by scanning the whole array again) it becomes `O(n^2)`, since for every element we scan the entire array to count its occurrences.
Space Complexity: `O(n)` in the worst case, since the hashmap can store up to `n` distinct elements.

**Walkthrough:** Using `nums = [2, 2, 1, 1, 1, 2, 2]`, `n = 7`, threshold `n/2 = 3`.
- `num=2`: `freq[2]=1` → not `>3`.
- `num=2`: `freq[2]=2` → not `>3`.
- `num=1`: `freq[1]=1` → not `>3`.
- `num=1`: `freq[1]=2` → not `>3`.
- `num=1`: `freq[1]=3` → not `>3`.
- `num=2`: `freq[2]=3` → not `>3`.
- `num=2`: `freq[2]=4` → `4>3` → return `2`.
Returned value `2` matches the expected output.

---

**Optimized Approach:** Boyer-Moore Voting Algorithm. Maintain a `candidate` and a `count`. Traverse the array: if `count` is `0`, pick the current element as the new candidate. If the current element equals the candidate, increment `count`, otherwise decrement it. Because the majority element occurs more than `n/2` times, it will always survive this cancellation process and end up as the final candidate.

**Logic (Steps):**
1. Initialize `count = 0` and `candidate = 0`.
2. Iterate through each `num` in `nums`.
3. If `count == 0`, set `candidate = num` (start backing a new candidate).
4. Increment `count` if `num == candidate`, otherwise decrement `count` (cast a "for" or "against" vote).
5. After the full pass, `candidate` holds the majority element (guaranteed by the problem), so return it directly.

```csharp
public int MajorityElementOptimal(int[] nums)
{
    int count = 0;
    int candidate = 0;

    foreach (int num in nums)
    {
        if (count == 0)
        {
            candidate = num;
        }

        count += (num == candidate) ? 1 : -1;
    }

    // Since the problem guarantees a majority element always exists,
    // candidate is returned directly. Otherwise, a second pass would
    // be needed to verify candidate's frequency > n / 2.
    return candidate;
}
```

Time Complexity: `O(n)` — a single linear pass over the array.
Space Complexity: `O(1)` — only two extra variables (`candidate` and `count`) are used, no auxiliary data structure.

**Walkthrough:** Dry run of Boyer-Moore Voting on `nums = [2, 2, 1, 1, 1, 2, 2]`:

| Index | num | count before | action | candidate | count after |
|-------|-----|--------------|--------|-----------|-------------|
| 0 | 2 | 0 | count==0 → candidate=2 | 2 | 1 |
| 1 | 2 | 1 | num==candidate → count++ | 2 | 2 |
| 2 | 1 | 2 | num!=candidate → count-- | 2 | 1 |
| 3 | 1 | 1 | num!=candidate → count-- | 2 | 0 |
| 4 | 1 | 0 | count==0 → candidate=1 | 1 | 1 |
| 5 | 2 | 1 | num!=candidate → count-- | 1 | 0 |
| 6 | 2 | 0 | count==0 → candidate=2 | 2 | 1 |

Final candidate is `2`, which matches the expected output. Each occurrence of the candidate acts as a `+1` vote and every other element as `-1`; since the majority element appears more than `n/2` times, it can never be fully cancelled out and always survives as the final candidate.

---

## 2. Majority Element II (> n/3 times)

### 2. Majority Element II

**Problem Statement:** Given an integer array `nums`, find all elements that appear more than `⌊n/3⌋` times. There can be at most two such elements in the array (since three elements each appearing more than `n/3` times would require more than `n` total elements, which is impossible).

**Example:**
- Input: `nums = [11, 33, 33, 11, 33, 11]`
- Output: `[11, 33]`
- Explanation: `n = 6`, so an element must appear more than `2` times to qualify. `11` appears 3 times and `33` appears 3 times — both qualify, so the answer is `[11, 33]`.

**Brute Force Approach:** Count the frequency of every distinct element using a hashmap, then collect all elements whose frequency exceeds `n/3`.

**Logic (Steps):**
1. Store `n = nums.Length` and create an empty `freq` dictionary.
2. Iterate through `nums`, incrementing `freq[num]` for each element (initializing to `1` on first sight).
3. Create an empty `result` list.
4. Iterate over every entry in `freq`; if `entry.Value > n / 3`, add `entry.Key` to `result`.
5. Return `result`.

```csharp
public List<int> MajorityElementIIBrute(int[] nums)
{
    int n = nums.Length;
    Dictionary<int, int> freq = new Dictionary<int, int>();

    foreach (int num in nums)
    {
        if (freq.ContainsKey(num))
            freq[num]++;
        else
            freq[num] = 1;
    }

    List<int> result = new List<int>();
    foreach (KeyValuePair<int, int> entry in freq)
    {
        if (entry.Value > n / 3)
            result.Add(entry.Key);
    }

    return result;
}
```

Time Complexity: `O(n)` for building the frequency map in a single pass, plus `O(d)` for scanning distinct entries (`d <= n`), giving overall `O(n)`. (If done via nested loops re-scanning the array for each distinct element, it becomes `O(n^2)`.)
Space Complexity: `O(n)` in the worst case, for storing up to `n` distinct elements in the hashmap.

**Walkthrough:** Using `nums = [11, 33, 33, 11, 33, 11]`, `n = 6`, threshold `n/3 = 2`.
- Build `freq = {11: 3, 33: 3}`.
- Scan entries: `freq[11] = 3 > 2` → add `11` to `result`. `freq[33] = 3 > 2` → add `33` to `result`.
- Return `result = [11, 33]`, matching the expected output.

---

**Optimized Approach:** Extended Boyer-Moore Voting with two candidates, since at most two elements can appear more than `n/3` times. Maintain `candidate1`, `count1`, `candidate2`, `count2`. Traverse once to find the two potential candidates, then do a second pass to verify their actual counts exceed `n/3`.

**Logic (Steps):**
1. Initialize `count1 = count2 = 0` and `candidate1 = candidate2 = int.MinValue`.
2. Phase 1 — for each `num`: if `count1 == 0` and `num` isn't `candidate2`, make `num` the new `candidate1`; else if `count2 == 0` and `num` isn't `candidate1`, make `num` the new `candidate2`; else if `num` matches `candidate1` or `candidate2`, increment that count; otherwise decrement both counts.
3. After phase 1, `candidate1` and `candidate2` are the only possible elements that could appear more than `n/3` times.
4. Phase 2 — reset `actualCount1`/`actualCount2` to `0` and re-scan `nums`, tallying how many times each candidate truly occurs.
5. Add `candidate1` to `result` if `actualCount1 > n/3`, and likewise for `candidate2`, then return `result`.

```csharp
public List<int> MajorityElementIIOptimal(int[] nums)
{
    int n = nums.Length;
    int count1 = 0, count2 = 0;
    int candidate1 = int.MinValue, candidate2 = int.MinValue;

    // Phase 1: find two potential candidates
    foreach (int num in nums)
    {
        if (count1 == 0 && candidate2 != num)
        {
            count1 = 1;
            candidate1 = num;
        }
        else if (count2 == 0 && candidate1 != num)
        {
            count2 = 1;
            candidate2 = num;
        }
        else if (num == candidate1)
        {
            count1++;
        }
        else if (num == candidate2)
        {
            count2++;
        }
        else
        {
            count1--;
            count2--;
        }
    }

    // Phase 2: verify candidates actually exceed n / 3 occurrences
    int actualCount1 = 0, actualCount2 = 0;
    foreach (int num in nums)
    {
        if (num == candidate1) actualCount1++;
        else if (num == candidate2) actualCount2++;
    }

    List<int> result = new List<int>();
    if (actualCount1 > n / 3) result.Add(candidate1);
    if (actualCount2 > n / 3) result.Add(candidate2);

    return result;
}
```

Time Complexity: `O(n)` — two linear passes over the array (one to find candidates, one to verify them), which is still `O(n)` overall.
Space Complexity: `O(1)` — only a constant number of extra variables are used (candidates and counts), no auxiliary data structure that scales with input size.

**Walkthrough:** Dry run of the two-candidate extension on `nums = [11, 33, 33, 11, 33, 11]` (`n = 6`, threshold `n/3 = 2`):

| num | condition matched | candidate1 | count1 | candidate2 | count2 |
|-----|--------------------|------------|--------|------------|--------|
| 11 | count1==0 → new candidate1 | 11 | 1 | - | 0 |
| 33 | count2==0 → new candidate2 | 11 | 1 | 33 | 1 |
| 33 | num==candidate2 → count2++ | 11 | 1 | 33 | 2 |
| 11 | num==candidate1 → count1++ | 11 | 2 | 33 | 2 |
| 33 | num==candidate2 → count2++ | 11 | 2 | 33 | 3 |
| 11 | num==candidate1 → count1++ | 11 | 3 | 33 | 3 |

Phase 1 ends with `candidate1 = 11`, `candidate2 = 33`. Phase 2 verification: actual count of `11` is 3 (> 2), actual count of `33` is 3 (> 2). Both pass, so `result = [11, 33]`, matching the expected output.

---

## 3. Longest Consecutive Sequence in an Array

### 3. Longest Consecutive Sequence

**Problem Statement:** Given an unsorted array of integers `nums`, find the length of the longest sequence of consecutive integers (elements do not need to be contiguous in the array itself, but the values must form a consecutive run, e.g. `4, 5, 6, 7`).

**Example:**
- Input: `nums = [100, 4, 200, 1, 3, 2]`
- Output: `4`
- Explanation: The longest consecutive sequence of values is `1, 2, 3, 4`, which has length `4`.

**Brute Force Approach:** For each element in the array, repeatedly search for the next consecutive element (`element + 1`, `element + 2`, ...) using a hashmap (or by scanning the array) and track the length of the longest chain found.

**Logic (Steps):**
1. Build a `HashSet<int> numSet` from `nums` for `O(1)` membership checks, and initialize `longest = 0`.
2. For every distinct `num` in `numSet`, start a chain with `currentNum = num` and `currentLength = 1`.
3. While `numSet` contains `currentNum + 1`, advance `currentNum` and increment `currentLength` (extend the chain forward).
4. Update `longest = Math.Max(longest, currentLength)` after each element's chain walk finishes.
5. Since every element (not just sequence starts) triggers a walk, overlapping chains get re-walked repeatedly; return `longest` after processing all elements.

```csharp
public int LongestConsecutiveBrute(int[] nums)
{
    if (nums.Length == 0) return 0;

    HashSet<int> numSet = new HashSet<int>(nums);
    int longest = 0;

    // For every element, try to build a consecutive chain starting from it
    foreach (int num in numSet)
    {
        int currentNum = num;
        int currentLength = 1;

        // Keep looking for the next consecutive number
        while (numSet.Contains(currentNum + 1))
        {
            currentNum++;
            currentLength++;
        }

        longest = Math.Max(longest, currentLength);
    }

    return longest;
}
```

Time Complexity: `O(n^2)` in the worst case — although a `HashSet` is used for `O(1)` lookups, without the "start of sequence" check every element can trigger a chain walk of up to `O(n)`, and this happens for up to `n` elements (e.g., a fully consecutive array causes repeated overlapping scans), giving quadratic behavior overall.
Space Complexity: `O(n)` for storing all elements in the `HashSet<int>`.

**Walkthrough:** Using `nums = [100, 4, 200, 1, 3, 2]`, `numSet = {100, 4, 200, 1, 3, 2}`.
- `num=100`: walk finds no `101` → length `1`. `longest=1`.
- `num=4`: walk finds no `5` → length `1`. `longest` stays `1`.
- `num=200`: walk finds no `201` → length `1`. `longest` stays `1`.
- `num=1`: walk finds `2`,`3`,`4`, then no `5` → length `4`. `longest=4`.
- `num=3`: walk finds `4`, then no `5` → length `2`. `longest` stays `4`.
- `num=2`: walk finds `3`,`4`, then no `5` → length `3`. `longest` stays `4`.
Returned value `4` matches the expected output (note the redundant re-walking of the `1,2,3,4` chain from `4`, `3`, and `2`, which is exactly the inefficiency the optimized approach removes).

---

**Optimized Approach:** Use a `HashSet<int>` for `O(1)` lookups, but only start counting a sequence from an element that is the *start* of a sequence — i.e., `arr[i] - 1` is not present in the set. This ensures every element is visited as part of at most one chain-building walk across the entire algorithm.

**Logic (Steps):**
1. Build a `HashSet<int> numSet` from `nums` for `O(1)` membership checks, and initialize `longest = 0`.
2. For every distinct `num` in `numSet`, check whether `numSet.Contains(num - 1)`.
3. If `num - 1` is present, `num` is not the start of a sequence, so skip it (it will be counted when its true start is processed).
4. If `num - 1` is absent, `num` is a sequence start: walk forward with `currentNum`/`currentLength` while `numSet.Contains(currentNum + 1)`.
5. Update `longest = Math.Max(longest, currentLength)` after each qualifying walk, and return `longest` at the end.

```csharp
public int LongestConsecutiveOptimal(int[] nums)
{
    if (nums.Length == 0) return 0;

    HashSet<int> numSet = new HashSet<int>(nums);
    int longest = 0;

    foreach (int num in numSet)
    {
        // Only start counting if 'num' is the start of a sequence
        if (!numSet.Contains(num - 1))
        {
            int currentNum = num;
            int currentLength = 1;

            while (numSet.Contains(currentNum + 1))
            {
                currentNum++;
                currentLength++;
            }

            longest = Math.Max(longest, currentLength);
        }
    }

    return longest;
}
```

Time Complexity: `O(n)` — each element is inserted into the `HashSet` once (`O(n)`), and the `while` loop that extends a chain only runs for elements that are the start of a sequence; across the whole algorithm every element is visited by the inner `while` loop at most once in total, so the combined work is `O(n)`.
Space Complexity: `O(n)` — the `HashSet<int>` stores up to `n` distinct elements.

**Walkthrough:** Dry run on `nums = [100, 4, 200, 1, 3, 2]`, `numSet = {100, 4, 200, 1, 3, 2}`.
- `num=100`: `numSet.Contains(99)` → false, sequence start. Walk: `101` not in set → length `1`. `longest=1`.
- `num=4`: `numSet.Contains(3)` → true, not a start; skip (covered later as part of `1,2,3,4`).
- `num=200`: `numSet.Contains(199)` → false, sequence start. Walk: `201` not in set → length `1`. `longest` stays `1`.
- `num=1`: `numSet.Contains(0)` → false, sequence start. Walk: `2`→`3`→`4` all in set, `5` not → length `4`. `longest=4`.
- `num=3`: `numSet.Contains(2)` → true, not a start; skip.
- `num=2`: `numSet.Contains(1)` → true, not a start; skip.
Final `longest = 4`, matching the expected output. Because only true sequence starts trigger a walk, each consecutive run is walked exactly once end-to-end, keeping total work across all walks bounded by `n`.
