# Array — Frequency and Hashing

## 1. Majority Element (> n/2 times)

### 1. Majority Element

**Problem Statement:** Given an array `nums` of size `n`, find the majority element — the element that appears more than `⌊n/2⌋` times. It is guaranteed that the array always has a majority element.

**Example:**
- Input: `nums = [2, 2, 1, 1, 1, 2, 2]`
- Output: `2`
- Explanation: The array has 7 elements, so an element must appear more than 3 times to be the majority element. `2` appears 4 times, so `2` is the majority element.

**Brute Force Approach:** For every distinct element, count its occurrences using a hashmap (or nested loops) and return the element whose frequency exceeds `n/2`.

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

**Optimized Approach:** Boyer-Moore Voting Algorithm. Maintain a `candidate` and a `count`. Traverse the array: if `count` is `0`, pick the current element as the new candidate. If the current element equals the candidate, increment `count`, otherwise decrement it. Because the majority element occurs more than `n/2` times, it will always survive this cancellation process and end up as the final candidate.

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

**Explanation:**

Dry run of Boyer-Moore Voting on `nums = [2, 2, 1, 1, 1, 2, 2]`:

| Index | num | count before | action | candidate | count after |
|-------|-----|--------------|--------|-----------|-------------|
| 0 | 2 | 0 | count==0 → candidate=2 | 2 | 1 |
| 1 | 2 | 1 | num==candidate → count++ | 2 | 2 |
| 2 | 1 | 2 | num!=candidate → count-- | 2 | 1 |
| 3 | 1 | 1 | num!=candidate → count-- | 2 | 0 |
| 4 | 1 | 0 | count==0 → candidate=1 | 1 | 1 |
| 5 | 2 | 1 | num!=candidate → count-- | 1 | 0 |
| 6 | 2 | 0 | count==0 → candidate=2 | 2 | 1 |

Final candidate is `2`, which matches the expected output.

Why this works: think of each occurrence of the candidate as a `+1` vote and every other element as a `-1` vote against it. Since the majority element appears more than `n/2` times, its votes outnumber the combined votes of all other elements put together. Every time a non-candidate element appears, it "cancels out" one occurrence of the candidate, but because the majority element has strictly more than half the total count, it can never be fully cancelled out — it always survives as the final candidate after all cancellations are accounted for.

---

## 2. Majority Element II (> n/3 times)

### 2. Majority Element II

**Problem Statement:** Given an integer array `nums`, find all elements that appear more than `⌊n/3⌋` times. There can be at most two such elements in the array (since three elements each appearing more than `n/3` times would require more than `n` total elements, which is impossible).

**Example:**
- Input: `nums = [11, 33, 33, 11, 33, 11]`
- Output: `[11, 33]`
- Explanation: `n = 6`, so an element must appear more than `2` times to qualify. `11` appears 3 times and `33` appears 3 times — both qualify, so the answer is `[11, 33]`.

**Brute Force Approach:** Count the frequency of every distinct element using a hashmap, then collect all elements whose frequency exceeds `n/3`.

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

**Optimized Approach:** Extended Boyer-Moore Voting with two candidates, since at most two elements can appear more than `n/3` times. Maintain `candidate1`, `count1`, `candidate2`, `count2`. Traverse once to find the two potential candidates, then do a second pass to verify their actual counts exceed `n/3`.

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

**Explanation:**

Dry run of the two-candidate extension on `nums = [11, 33, 33, 11, 33, 11]` (`n = 6`, threshold `n/3 = 2`):

| num | condition matched | candidate1 | count1 | candidate2 | count2 |
|-----|--------------------|------------|--------|------------|--------|
| 11 | count1==0 → new candidate1 | 11 | 1 | - | 0 |
| 33 | count2==0 → new candidate2 | 11 | 1 | 33 | 1 |
| 33 | num==candidate2 → count2++ | 11 | 1 | 33 | 2 |
| 11 | num==candidate1 → count1++ | 11 | 2 | 33 | 2 |
| 33 | num==candidate2 → count2++ | 11 | 2 | 33 | 3 |
| 11 | num==candidate1 → count1++ | 11 | 3 | 33 | 3 |

Phase 1 ends with `candidate1 = 11`, `candidate2 = 33`. Phase 2 verification: actual count of `11` is 3 (> 2), actual count of `33` is 3 (> 2). Both pass, so the result is `[11, 33]`, matching the expected output.

Why this works: at most two elements can appear more than `n/3` times each, because three such elements would together need more than `n` total positions. The two-candidate cancellation works exactly like single-candidate Boyer-Moore, but now each step either grows one of the two candidate counts or "cancels" one occurrence from each candidate simultaneously when the current element matches neither. Any true majority element (more than `n/3` occurrences) has enough occurrences to survive this cancellation and remain one of the two final candidates. The verification pass is still required because the algorithm only guarantees the true answers (if they exist) are among the two candidates — it does not guarantee that both candidates found are actually valid majority elements.

---

## 3. Longest Consecutive Sequence in an Array

### 3. Longest Consecutive Sequence

**Problem Statement:** Given an unsorted array of integers `nums`, find the length of the longest sequence of consecutive integers (elements do not need to be contiguous in the array itself, but the values must form a consecutive run, e.g. `4, 5, 6, 7`).

**Example:**
- Input: `nums = [100, 4, 200, 1, 3, 2]`
- Output: `4`
- Explanation: The longest consecutive sequence of values is `1, 2, 3, 4`, which has length `4`.

**Brute Force Approach:** For each element in the array, repeatedly search for the next consecutive element (`element + 1`, `element + 2`, ...) using a hashmap (or by scanning the array) and track the length of the longest chain found.

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

**Optimized Approach:** Use a `HashSet<int>` for `O(1)` lookups, but only start counting a sequence from an element that is the *start* of a sequence — i.e., `arr[i] - 1` is not present in the set. This ensures every element is visited as part of at most one chain-building walk across the entire algorithm.

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

**Explanation:**

Dry run on `nums = [100, 4, 200, 1, 3, 2]`:

1. Build `numSet = {100, 4, 200, 1, 3, 2}`.
2. Iterate over `numSet`:
   - `num = 100`: check `numSet.Contains(99)` → false, so `100` is a sequence start. Walk: `101` not in set → chain length `1`. `longest = 1`.
   - `num = 4`: check `numSet.Contains(3)` → true, so `4` is **not** a sequence start; skip (it will be reached later as part of the `1,2,3,4` chain).
   - `num = 200`: check `numSet.Contains(199)` → false, sequence start. Walk: `201` not in set → chain length `1`. `longest` stays `1`.
   - `num = 1`: check `numSet.Contains(0)` → false, sequence start. Walk: `2` in set → `currentNum=2, length=2`; `3` in set → `currentNum=3, length=3`; `4` in set → `currentNum=4, length=4`; `5` not in set → stop. `longest = 4`.
   - `num = 3`: check `numSet.Contains(2)` → true, not a sequence start; skip.
   - `num = 2`: check `numSet.Contains(1)` → true, not a sequence start; skip.
3. Final answer: `longest = 4`, matching the expected output.

Why checking `!set.Contains(x - 1)` before starting a count keeps this `O(n)` overall: without this check, the brute-force version would attempt to build a chain starting from *every* element, including elements in the middle of a long run (like `2` and `3` above), causing the same chain to be walked repeatedly — once from `1`, once from `2`, once from `3`, and once from `4` — which degrades to `O(n^2)` for arrays with long consecutive runs. By only starting the walk when the current element has no predecessor in the set (i.e., it is genuinely the smallest element of its consecutive run), each chain is walked exactly once, from its true starting point to its true end. Since every element belongs to exactly one consecutive run, and each run is walked exactly once from start to finish, the total number of steps taken across all `while` loop iterations, summed over the whole algorithm, is bounded by `n` — giving an overall `O(n)` time complexity using just a `HashSet<int>` for constant-time membership checks.
