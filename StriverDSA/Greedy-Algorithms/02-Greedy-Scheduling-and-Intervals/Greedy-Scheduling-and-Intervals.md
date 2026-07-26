# Greedy Algorithms — Scheduling and Intervals

## 1. N Meetings in One Room

**Problem Statement:**
Given the start time and end time of `N` meetings, find the maximum number of meetings that can be accommodated in a single meeting room, such that no two selected meetings overlap. If a meeting ends at time `x`, another meeting can start at time `x` (they don't overlap).

**Example:**
- Input: `start = [1,3,0,5,8,5], end = [2,4,6,7,9,9]`
- Output: `4`
- Explanation: Selecting meetings `(1,2), (3,4), (5,7), (8,9)` gives 4 non-overlapping meetings, which is the maximum possible.

**Brute Force Approach:**
Generate all possible subsets of meetings, check which subsets have no overlapping meetings, and return the size of the largest valid subset. This is exponential because there are `2^n` subsets.

```csharp
public class NMeetingsBruteForce
{
    private int maxCount = 0;

    public int MaxMeetings(int[] start, int[] end)
    {
        int n = start.Length;
        Solve(start, end, 0, n, -1, 0);
        return maxCount;
    }

    private void Solve(int[] start, int[] end, int index, int n, int lastEndTime, int count)
    {
        if (index == n)
        {
            maxCount = Math.Max(maxCount, count);
            return;
        }

        // Option 1: skip this meeting
        Solve(start, end, index + 1, n, lastEndTime, count);

        // Option 2: take this meeting if it doesn't overlap with the last chosen one
        if (start[index] > lastEndTime)
        {
            Solve(start, end, index + 1, n, end[index], count + 1);
        }
    }
}
```
Time Complexity: O(2^n) — every meeting is either taken or skipped.
Space Complexity: O(n) — recursion stack depth.

**Optimized Approach:**
Sort meetings by their end time. Greedily pick a meeting if its start time is strictly greater than the end time of the last picked meeting. Picking the meeting that finishes earliest always leaves the most room for future meetings.

```csharp
public class NMeetingsOptimal
{
    public int MaxMeetings(int[] start, int[] end)
    {
        int n = start.Length;
        var meetings = new (int Start, int End)[n];
        for (int i = 0; i < n; i++)
        {
            meetings[i] = (start[i], end[i]);
        }

        Array.Sort(meetings, (a, b) => a.End.CompareTo(b.End));

        int count = 1; // first meeting (earliest end) is always picked
        int lastEndTime = meetings[0].End;

        for (int i = 1; i < n; i++)
        {
            if (meetings[i].Start > lastEndTime)
            {
                count++;
                lastEndTime = meetings[i].End;
            }
        }

        return count;
    }
}
```
Time Complexity: O(n log n) for sorting, followed by O(n) for the single greedy pass.
Space Complexity: O(n) for the array of (start, end) pairs used for sorting.

**Explanation:**
Sorting by end time is the key insight — a meeting that ends earlier always leaves at least as much free room as one that ends later, so greedily choosing the earliest-finishing available meeting can never be worse than any other choice. This is a classic activity-selection exchange argument: any optimal solution can be transformed to include the earliest-ending meeting without losing count.

---

## 2. Jump Game

**Problem Statement:**
Given an array `nums` where `nums[i]` represents the maximum jump length from index `i`, determine if it's possible to reach the last index starting from index `0`.

**Example:**
- Input: `nums = [2,3,1,1,4]`
- Output: `true`
- Explanation: Jump 1 step from index 0 to 1, then 3 steps to the last index (index 4).

**Brute Force Approach:**
Try every possible jump length from the current index recursively (backtracking) and check if any path reaches the last index.

```csharp
public class JumpGameBruteForce
{
    public bool CanJump(int[] nums)
    {
        return CanReach(nums, 0);
    }

    private bool CanReach(int[] nums, int position)
    {
        if (position >= nums.Length - 1)
        {
            return true;
        }

        int maxJump = nums[position];
        for (int step = maxJump; step >= 1; step--)
        {
            if (CanReach(nums, position + step))
            {
                return true;
            }
        }

        return false;
    }
}
```
Time Complexity: O(2^n) in the worst case — each position branches into multiple recursive calls.
Space Complexity: O(n) for the recursion stack.

**Optimized Approach:**
Traverse the array once, maintaining the farthest index reachable so far. If at any index `i` the farthest reachable index is less than `i`, we're stuck and cannot proceed, so return false. Otherwise, keep extending the farthest reachable index.

```csharp
public class JumpGameOptimal
{
    public bool CanJump(int[] nums)
    {
        int farthest = 0;
        int n = nums.Length;

        for (int i = 0; i < n; i++)
        {
            if (i > farthest)
            {
                return false; // current index is unreachable
            }
            farthest = Math.Max(farthest, i + nums[i]);

            if (farthest >= n - 1)
            {
                return true; // last index already reachable
            }
        }

        return true;
    }
}
```
Time Complexity: O(n) — single pass through the array.
Space Complexity: O(1) — only a couple of scalar trackers used.

**Explanation:**
`farthest` greedily tracks the maximum index reachable using jumps decided so far. Since we only care about whether the last index is reachable (not the specific path), it's always optimal to assume we can use the best jump length seen up to the current index — this greedy relaxation never loses correctness because reachability is monotonic (if index `i` is reachable, everything jumped-to from any reachable index before `i` is also valid).

---

## 3. Jump Game II

**Problem Statement:**
Given an array `nums` where `nums[i]` represents the maximum jump length from index `i`, return the minimum number of jumps required to reach the last index, starting from index `0`. It is guaranteed that the last index is always reachable.

**Example:**
- Input: `nums = [2,3,1,1,4]`
- Output: `2`
- Explanation: Jump 1 step from index 0 to index 1, then jump 3 steps to the last index (index 4).

**Brute Force Approach:**
Recursively try every possible jump length from each index and take the minimum number of jumps among all valid paths that reach the end.

```csharp
public class JumpGameIIBruteForce
{
    public int MinJumps(int[] nums)
    {
        int result = MinJumpsFrom(nums, 0);
        return result;
    }

    private int MinJumpsFrom(int[] nums, int position)
    {
        if (position >= nums.Length - 1)
        {
            return 0;
        }

        int minJumps = int.MaxValue;
        int maxJump = nums[position];

        for (int step = 1; step <= maxJump; step++)
        {
            int subResult = MinJumpsFrom(nums, position + step);
            if (subResult != int.MaxValue)
            {
                minJumps = Math.Min(minJumps, 1 + subResult);
            }
        }

        return minJumps;
    }
}
```
Time Complexity: O(2^n) in the worst case due to exponential branching of jump choices.
Space Complexity: O(n) for the recursion stack.

**Optimized Approach:**
Use a greedy BFS-like sweep with two pointers: `currentJumpEnd` marks the boundary of the current jump's range, and `farthest` tracks the farthest index reachable using one more jump. Whenever we reach `currentJumpEnd`, we are forced to take another jump, so we increment the jump count and move the boundary to `farthest`.

```csharp
public class JumpGameIIOptimal
{
    public int MinJumps(int[] nums)
    {
        int n = nums.Length;
        int jumps = 0;
        int currentJumpEnd = 0;
        int farthest = 0;

        for (int i = 0; i < n - 1; i++)
        {
            farthest = Math.Max(farthest, i + nums[i]);

            if (i == currentJumpEnd)
            {
                jumps++;
                currentJumpEnd = farthest;
            }
        }

        return jumps;
    }
}
```
Time Complexity: O(n) — single pass through the array with O(1) work per index.
Space Complexity: O(1) — only scalar trackers used.

**Explanation:**
Dry run on `nums = [2,3,1,1,4]` (indices 0..4):

| i | nums[i] | farthest = max(farthest, i+nums[i]) | i == currentJumpEnd? | jumps | currentJumpEnd (after update) |
|---|---------|--------------------------------------|-----------------------|-------|--------------------------------|
| 0 | 2       | max(0, 0+2)=2                        | yes (0==0)            | 1     | 2                               |
| 1 | 3       | max(2, 1+3)=4                        | no (1!=2)              | 1     | 2                               |
| 2 | 1       | max(4, 2+1)=4                        | yes (2==2)             | 2     | 4                               |
| 3 | 1       | max(4, 3+1)=4                        | no (3!=4)              | 2     | 4                               |

Loop ends at `i = n-1 = 4` (loop condition is `i < n-1`). Final answer: `jumps = 2`.

Interpretation: `currentJumpEnd` is the farthest index reachable with the jumps taken so far; `farthest` is continuously updated to the best possible reach one jump beyond that. When `i` catches up to `currentJumpEnd`, it means we've exhausted the current jump's range and must commit to a new jump, so we "use" one jump and extend the boundary to `farthest`. This greedily simulates a level-by-level BFS expansion in O(n) instead of O(n) per level.

---

## 4. Minimum Number of Platforms Required for a Railway Station

**Problem Statement:**
Given the arrival and departure times of all trains that arrive at a railway station on a given day, find the minimum number of platforms required so that no train has to wait for a platform to become free.

**Example:**
- Input: `arrival = [900, 940, 950, 1100, 1500, 1800], departure = [910, 1200, 1120, 1130, 1900, 2000]`
- Output: `3`
- Explanation: At time 950, trains arriving at 900 (dep 910 has already left), 940 (dep 1200), and 950 (dep 1120) — combined with checking overlaps carefully, three trains are simultaneously in the station's schedule window at peak, requiring 3 platforms.

**Brute Force Approach:**
For each train, count how many other trains' intervals overlap with it (i.e., how many trains are at the station at the same time as its arrival). The answer is the maximum such overlap count across all trains.

```csharp
public class MinPlatformsBruteForce
{
    public int FindMinPlatforms(int[] arrival, int[] departure)
    {
        int n = arrival.Length;
        int maxPlatforms = 0;

        for (int i = 0; i < n; i++)
        {
            int platformsNeeded = 1;
            for (int j = 0; j < n; j++)
            {
                if (i == j) continue;

                // train j overlaps with train i's arrival time
                if (arrival[j] <= arrival[i] && arrival[i] <= departure[j])
                {
                    platformsNeeded++;
                }
            }
            maxPlatforms = Math.Max(maxPlatforms, platformsNeeded);
        }

        return maxPlatforms;
    }
}
```
Time Complexity: O(n^2) — for every train, scan all other trains.
Space Complexity: O(1) — no extra data structures beyond counters.

**Optimized Approach:**
Sort arrival and departure times independently. Use two pointers to sweep through events in time order: whenever a train arrives (before or at the time the earliest still-active train would depart), increment the platform count; whenever a train departs, decrement it. Track the maximum concurrent platform count.

```csharp
public class MinPlatformsOptimal
{
    public int FindMinPlatforms(int[] arrival, int[] departure)
    {
        int n = arrival.Length;
        int[] sortedArrival = (int[])arrival.Clone();
        int[] sortedDeparture = (int[])departure.Clone();

        Array.Sort(sortedArrival);
        Array.Sort(sortedDeparture);

        int platformsNeeded = 0;
        int maxPlatforms = 0;
        int arrivalPointer = 0;
        int departurePointer = 0;

        while (arrivalPointer < n && departurePointer < n)
        {
            if (sortedArrival[arrivalPointer] <= sortedDeparture[departurePointer])
            {
                platformsNeeded++;
                maxPlatforms = Math.Max(maxPlatforms, platformsNeeded);
                arrivalPointer++;
            }
            else
            {
                platformsNeeded--;
                departurePointer++;
            }
        }

        return maxPlatforms;
    }
}
```
Time Complexity: O(n log n) — dominated by sorting both arrays; the two-pointer sweep itself is O(n).
Space Complexity: O(n) for the cloned, sorted arrival and departure arrays.

**Explanation:**
Sorting arrivals and departures separately lets us process station events strictly in chronological order without needing to track which arrival matches which departure (only counts matter, not identities). Every time an arrival event occurs before or simultaneously with the earliest pending departure, one more platform is occupied; every departure frees one. The running maximum of occupied platforms is the answer, since it represents the worst-case moment of simultaneous train presence.

---

## 5. Job Sequencing Problem

**Problem Statement:**
Given a set of `N` jobs, where each job has a deadline and a profit associated with it, and each job takes exactly 1 unit of time to complete (only one job can be scheduled at a time), find the maximum profit achievable and the number of jobs done, if only one job can be scheduled at any given time slot and a job can only be scheduled before or on its deadline.

**Example:**
- Input: `jobs = [(id=1, deadline=4, profit=20), (id=2, deadline=1, profit=10), (id=3, deadline=1, profit=40), (id=4, deadline=1, profit=30)]`
- Output: `Jobs done = 2, Max profit = 60`
- Explanation: Schedule job 3 (profit 40) at slot 1, and job 1 (profit 20) at slot 4. Total profit = 60, jobs done = 2.

**Brute Force Approach:**
Try every permutation of jobs (or every subset with valid deadline assignment) and compute the profit for each valid schedule, keeping track of the maximum.

```csharp
public class JobSequencingBruteForce
{
    public class Job
    {
        public int Id;
        public int Deadline;
        public int Profit;
    }

    private int maxProfit = 0;
    private int maxJobsAtBestProfit = 0;

    public (int JobsDone, int Profit) FindMaxProfit(List<Job> jobs)
    {
        int maxDeadline = jobs.Max(j => j.Deadline);
        bool[] slots = new bool[maxDeadline + 1];
        Solve(jobs, 0, slots, 0, 0);
        return (maxJobsAtBestProfit, maxProfit);
    }

    private void Solve(List<Job> jobs, int index, bool[] slots, int jobsDone, int profit)
    {
        if (index == jobs.Count)
        {
            if (profit > maxProfit)
            {
                maxProfit = profit;
                maxJobsAtBestProfit = jobsDone;
            }
            return;
        }

        // Try placing current job at every free slot up to its deadline
        for (int slot = jobs[index].Deadline; slot >= 1; slot--)
        {
            if (!slots[slot])
            {
                slots[slot] = true;
                Solve(jobs, index + 1, slots, jobsDone + 1, profit + jobs[index].Profit);
                slots[slot] = false; // backtrack
            }
        }

        // Try skipping the current job
        Solve(jobs, index + 1, slots, jobsDone, profit);
    }
}
```
Time Complexity: O(2^n * maxDeadline) roughly, since each job can be placed in any free slot or skipped — exponential in the worst case.
Space Complexity: O(n + maxDeadline) for recursion stack and slot array.

**Optimized Approach:**
Sort jobs by profit in descending order. For each job (highest profit first), greedily place it in the latest available free slot at or before its deadline (scanning from `deadline` down to `1`). This ensures earlier slots stay open for jobs with tighter deadlines.

```csharp
public class JobSequencingOptimal
{
    public class Job
    {
        public int Id;
        public int Deadline;
        public int Profit;
    }

    public (int JobsDone, int Profit) FindMaxProfit(List<Job> jobs)
    {
        var sortedJobs = jobs.OrderByDescending(j => j.Profit).ToList();
        int maxDeadline = jobs.Max(j => j.Deadline);

        int[] slot = new int[maxDeadline + 1]; // slot[d] = job id occupying slot d, 0 = free
        int jobsDone = 0;
        int totalProfit = 0;

        foreach (var job in sortedJobs)
        {
            // find the latest free slot <= job's deadline
            for (int d = Math.Min(maxDeadline, job.Deadline); d >= 1; d--)
            {
                if (slot[d] == 0)
                {
                    slot[d] = job.Id;
                    jobsDone++;
                    totalProfit += job.Profit;
                    break;
                }
            }
        }

        return (jobsDone, totalProfit);
    }
}
```
Time Complexity: O(n log n + n * maxDeadline) — sorting takes O(n log n); for each job, scanning backward for a free slot takes up to O(maxDeadline) in the worst case. (This can be improved to near O(n log n) using a Union-Find/Disjoint Set structure that jumps directly to the nearest free slot.)
Space Complexity: O(maxDeadline) for the slot array.

**Explanation:**
Choosing the highest-profit jobs first is greedy on the objective directly (profit), and placing each chosen job as late as possible (closest to its deadline) preserves earlier slots for jobs that might have tighter deadlines later in the iteration — this maximizes the chance that all remaining high-profit jobs can still be scheduled. An optional Union-Find optimization stores, for each slot, a pointer to the nearest available slot at or before it, allowing O(alpha(n)) slot lookups instead of a linear backward scan, improving the total complexity to O(n log n).

---

## 6. Candy / Distribute Candies

**Problem Statement:**
There are `N` children standing in a line, each with a rating value given in an array `ratings`. You must distribute candies such that: (1) each child gets at least one candy, and (2) any child with a higher rating than an immediate neighbor gets more candies than that neighbor. Return the minimum total number of candies required.

**Example:**
- Input: `ratings = [1,0,2]`
- Output: `5`
- Explanation: Give candies `[2,1,2]`. Child 0 (rating 1) > child 1 (rating 0), so child 0 gets more than child 1. Child 2 (rating 2) > child 1 (rating 0), so child 2 gets more than child 1. Total = 2+1+2 = 5, which is minimal.

**Brute Force Approach:**
Start every child with 1 candy. Repeatedly scan the array left to right and right to left, incrementing a child's candies whenever a neighbor constraint is violated, until no more updates occur (this converges but may take multiple passes, O(n) per pass, potentially O(n) passes in the worst case).

```csharp
public class CandyBruteForce
{
    public int MinCandies(int[] ratings)
    {
        int n = ratings.Length;
        int[] candies = new int[n];
        Array.Fill(candies, 1);

        bool changed = true;
        while (changed)
        {
            changed = false;
            for (int i = 0; i < n; i++)
            {
                if (i > 0 && ratings[i] > ratings[i - 1] && candies[i] <= candies[i - 1])
                {
                    candies[i] = candies[i - 1] + 1;
                    changed = true;
                }
                if (i < n - 1 && ratings[i] > ratings[i + 1] && candies[i] <= candies[i + 1])
                {
                    candies[i] = candies[i + 1] + 1;
                    changed = true;
                }
            }
        }

        int total = 0;
        for (int i = 0; i < n; i++)
        {
            total += candies[i];
        }
        return total;
    }
}
```
Time Complexity: O(n^2) in the worst case — repeated full passes until convergence.
Space Complexity: O(n) for the candies array.

**Optimized Approach:**
Do exactly two linear passes. Left-to-right pass: if `ratings[i] > ratings[i-1]`, then `candies[i] = candies[i-1] + 1` (ensures the right-neighbor-higher constraint). Right-to-left pass: if `ratings[i] > ratings[i+1]`, then `candies[i] = max(candies[i], candies[i+1] + 1)` (ensures the left-neighbor-higher constraint without breaking the first pass's guarantee). Sum the final candies array.

```csharp
public class CandyOptimal
{
    public int MinCandies(int[] ratings)
    {
        int n = ratings.Length;
        int[] candies = new int[n];
        Array.Fill(candies, 1);

        // Left-to-right pass
        for (int i = 1; i < n; i++)
        {
            if (ratings[i] > ratings[i - 1])
            {
                candies[i] = candies[i - 1] + 1;
            }
        }

        // Right-to-left pass
        for (int i = n - 2; i >= 0; i--)
        {
            if (ratings[i] > ratings[i + 1])
            {
                candies[i] = Math.Max(candies[i], candies[i + 1] + 1);
            }
        }

        int total = 0;
        for (int i = 0; i < n; i++)
        {
            total += candies[i];
        }
        return total;
    }
}
```
Time Complexity: O(n) — two linear passes plus one summation pass, all O(n).
Space Complexity: O(n) for the candies array (can be optimized to O(1) with a slope-counting technique, but the two-pass array version is the standard clean solution).

**Explanation:**
Dry run on `ratings = [1, 0, 2]` (n = 3):

Initialize: `candies = [1, 1, 1]`

Left-to-right pass (i = 1 to 2):
- i=1: `ratings[1]=0`, `ratings[0]=1` → `0 > 1`? No → `candies[1]` stays `1`. `candies = [1, 1, 1]`
- i=2: `ratings[2]=2`, `ratings[1]=0` → `2 > 0`? Yes → `candies[2] = candies[1] + 1 = 2`. `candies = [1, 1, 2]`

Right-to-left pass (i = 1 down to 0):
- i=1: `ratings[1]=0`, `ratings[2]=2` → `0 > 2`? No → `candies[1]` stays `1`. `candies = [1, 1, 2]`
- i=0: `ratings[0]=1`, `ratings[1]=0` → `1 > 0`? Yes → `candies[0] = max(candies[0], candies[1]+1) = max(1, 2) = 2`. `candies = [2, 1, 2]`

Final candies array: `[2, 1, 2]`. Sum = `2 + 1 + 2 = 5`.

Interpretation: the left-to-right pass alone correctly handles all "ascending" neighbor relationships (child gets more than the left neighbor when rating is higher), but it cannot fix "descending" relationships (child 0 needing more than child 1 due to comparing against the right). The right-to-left pass fixes those descending cases using `max` so it never undoes a guarantee already established by the first pass — each child's final candy count is the maximum of what both passes independently require, satisfying both neighbor constraints simultaneously.

---

## 7. Insert Interval

**Problem Statement:**
Given a set of non-overlapping intervals sorted by their start times, and a new interval, insert the new interval into the correct position and merge if necessary so that the resulting intervals are still sorted and non-overlapping. Return the final list of intervals.

**Example:**
- Input: `intervals = [[1,3],[6,9]], newInterval = [2,5]`
- Output: `[[1,5],[6,9]]`
- Explanation: The new interval `[2,5]` overlaps with `[1,3]`, so they merge into `[1,5]`. `[6,9]` does not overlap with `[1,5]`, so it remains unchanged.

**Brute Force Approach:**
Add the new interval to the list, sort all intervals by start time, then merge overlapping intervals in a single pass (the standard "merge intervals" technique applied after insertion and sorting).

```csharp
public class InsertIntervalBruteForce
{
    public int[][] Insert(int[][] intervals, int[] newInterval)
    {
        var all = new List<int[]>(intervals) { newInterval };
        all.Sort((a, b) => a[0].CompareTo(b[0]));

        var merged = new List<int[]>();
        foreach (var current in all)
        {
            if (merged.Count == 0 || merged[merged.Count - 1][1] < current[0])
            {
                merged.Add(current);
            }
            else
            {
                merged[merged.Count - 1][1] = Math.Max(merged[merged.Count - 1][1], current[1]);
            }
        }

        return merged.ToArray();
    }
}
```
Time Complexity: O(n log n) — dominated by sorting all n+1 intervals (even though the original list was already sorted, this approach re-sorts everything).
Space Complexity: O(n) for the merged list and the combined list.

**Optimized Approach:**
Since the original intervals are already sorted and non-overlapping, process them in a single linear pass with three phases: (1) add all intervals that end strictly before the new interval starts (no overlap, come before), (2) merge all intervals that overlap with the new interval by expanding its bounds, then add the merged interval, (3) add all remaining intervals that start strictly after the (possibly expanded) new interval ends.

```csharp
public class InsertIntervalOptimal
{
    public int[][] Insert(int[][] intervals, int[] newInterval)
    {
        var result = new List<int[]>();
        int n = intervals.Length;
        int i = 0;

        // Phase 1: intervals ending before newInterval starts
        while (i < n && intervals[i][1] < newInterval[0])
        {
            result.Add(intervals[i]);
            i++;
        }

        // Phase 2: merge all overlapping intervals with newInterval
        int mergedStart = newInterval[0];
        int mergedEnd = newInterval[1];
        while (i < n && intervals[i][0] <= mergedEnd)
        {
            mergedStart = Math.Min(mergedStart, intervals[i][0]);
            mergedEnd = Math.Max(mergedEnd, intervals[i][1]);
            i++;
        }
        result.Add(new int[] { mergedStart, mergedEnd });

        // Phase 3: remaining intervals starting after mergedEnd
        while (i < n)
        {
            result.Add(intervals[i]);
            i++;
        }

        return result.ToArray();
    }
}
```
Time Complexity: O(n) — a single linear pass through the intervals array with no sorting needed since the input is already sorted.
Space Complexity: O(n) for the result list (input is not counted as extra space).

**Explanation:**
Because the input is guaranteed sorted and non-overlapping, we never need to re-sort — we only need to find the contiguous "window" of existing intervals that overlap the new one and collapse that window into a single merged interval. Phase 1 greedily emits everything strictly before the overlap window untouched, phase 2 greedily expands the merged interval's bounds by absorbing every interval that touches or overlaps it, and phase 3 emits everything after untouched — this three-phase greedy sweep achieves linear time versus the O(n log n) brute-force resort.

---

## 8. Non-overlapping Intervals

**Problem Statement:**
Given an array of intervals, find the minimum number of intervals that must be removed so that the remaining intervals do not overlap with each other. Intervals that merely touch at endpoints (e.g., `[1,2]` and `[2,3]`) are not considered overlapping.

**Example:**
- Input: `intervals = [[1,2],[2,3],[3,4],[1,3]]`
- Output: `1`
- Explanation: Remove `[1,3]`. The remaining intervals `[1,2],[2,3],[3,4]` do not overlap with each other.

**Brute Force Approach:**
Try every subset of intervals (or equivalently, decide for each interval whether to keep or remove it) and check if the kept subset is entirely non-overlapping, tracking the maximum size of a valid non-overlapping subset. The answer is `n - maxValidSubsetSize`.

```csharp
public class NonOverlappingIntervalsBruteForce
{
    private int maxKept = 0;

    public int EraseOverlapIntervals(int[][] intervals)
    {
        int n = intervals.Length;
        Solve(intervals, 0, n, -1, 0);
        return n - maxKept;
    }

    private void Solve(int[][] intervals, int index, int n, double lastEnd, int kept)
    {
        if (index == n)
        {
            maxKept = Math.Max(maxKept, kept);
            return;
        }

        // Option 1: remove current interval (skip it)
        Solve(intervals, index + 1, n, lastEnd, kept);

        // Option 2: keep current interval if it doesn't overlap with the last kept one
        if (intervals[index][0] >= lastEnd)
        {
            Solve(intervals, index + 1, n, intervals[index][1], kept + 1);
        }
    }
}
```
Time Complexity: O(2^n) — every interval is either kept or removed, giving exponential branching.
Space Complexity: O(n) for the recursion stack.

**Optimized Approach:**
Sort intervals by their end time. Greedily keep an interval if its start time is greater than or equal to the end time of the last kept interval; otherwise, it must be removed (since we always prefer to keep the interval that frees up the earliest room for subsequent intervals). Count removals directly.

```csharp
public class NonOverlappingIntervalsOptimal
{
    public int EraseOverlapIntervals(int[][] intervals)
    {
        if (intervals.Length == 0)
        {
            return 0;
        }

        Array.Sort(intervals, (a, b) => a[1].CompareTo(b[1]));

        int removals = 0;
        int lastEnd = intervals[0][1];

        for (int i = 1; i < intervals.Length; i++)
        {
            if (intervals[i][0] < lastEnd)
            {
                // overlap detected, remove current interval (it ends later or equal since sorted by end)
                removals++;
            }
            else
            {
                lastEnd = intervals[i][1];
            }
        }

        return removals;
    }
}
```
Time Complexity: O(n log n) for sorting by end time, followed by O(n) for the single greedy pass.
Space Complexity: O(1) extra space if the sort is done in place (excluding sort's internal overhead), aside from O(n)/O(log n) used internally by the sorting algorithm.

**Explanation:**
This is the complement of the "N Meetings in One Room" problem: maximizing the count of non-overlapping intervals kept is equivalent to minimizing the count removed (`removals = n - maxKept`). Sorting by end time and greedily keeping the earliest-ending non-conflicting interval at each step maximizes the number retained, by the same exchange-argument logic used in interval scheduling — keeping the interval that ends soonest never constrains future choices more than any alternative would.
