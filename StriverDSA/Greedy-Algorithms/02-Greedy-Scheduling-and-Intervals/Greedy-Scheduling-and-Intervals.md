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

**Logic (Steps):**
1. Recurse through meetings by index, tracking `lastEndTime` (end of the last chosen meeting) and `count`.
2. At each index, branch into two options: skip the meeting, or take it.
3. Taking is only valid if `start[index] > lastEndTime` (no overlap); if valid, recurse with `end[index]` as the new `lastEndTime` and `count + 1`.
4. When `index == n`, update the running `maxCount` with the current `count`.
5. Return `maxCount` after all branches are explored.

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

**Walkthrough:** With `start = [1,3,0,5,8,5], end = [2,4,6,7,9,9]`, the recursion explores every skip/take combination. Taking `(1,2)` (index 0) sets `lastEndTime = 2`; from there `(3,4)` is valid (`3 > 2`) giving `lastEndTime = 4`; then `(5,7)` is valid (`5 > 4`) giving `lastEndTime = 7`; then `(8,9)` is valid (`8 > 7`) giving `count = 4`. No other branch produces more than 4 non-overlapping picks, so `maxCount = 4` is returned, matching the expected output.

**Optimized Approach:**
Sort meetings by their end time. Greedily pick a meeting if its start time is strictly greater than the end time of the last picked meeting. Picking the meeting that finishes earliest always leaves the most room for future meetings.

**Logic (Steps):**
1. Pair up `start[i]` and `end[i]` into tuples and sort them ascending by `End`.
2. Always pick the first meeting (earliest end); set `lastEndTime` to its end and `count = 1`.
3. Scan the remaining sorted meetings; if `meetings[i].Start > lastEndTime`, pick it: increment `count` and update `lastEndTime = meetings[i].End`.
4. Otherwise skip the meeting since it overlaps the last picked one.
5. Return `count` after the scan.

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

**Walkthrough:** With `start = [1,3,0,5,8,5], end = [2,4,6,7,9,9]`, pairing and sorting by end gives `(1,2), (3,4), (5,7), (0,6), (8,9), (5,9)`. `count = 1`, `lastEndTime = 2` (from `(1,2)`).
- `(3,4)`: `3 > 2` → pick, `count = 2`, `lastEndTime = 4`.
- `(5,7)`: `5 > 4` → pick, `count = 3`, `lastEndTime = 7`.
- `(0,6)`: `0 > 7`? No → skip.
- `(8,9)`: `8 > 7` → pick, `count = 4`, `lastEndTime = 9`.
- `(5,9)`: `5 > 9`? No → skip.

Return `count = 4`, matching the expected output. Sorting by end time is the key insight — a meeting that ends earlier always leaves at least as much free room as one that ends later, so greedily choosing the earliest-finishing available meeting can never be worse than any other choice.

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

**Logic (Steps):**
1. `CanReach(nums, position)`: if `position >= nums.Length - 1`, the end is reached, return `true`.
2. Otherwise read `maxJump = nums[position]`.
3. Try every step length from `maxJump` down to `1`, recursively calling `CanReach(nums, position + step)`.
4. If any recursive call returns `true`, propagate `true` immediately.
5. If no step length works, return `false`.

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

**Walkthrough:** With `nums = [2,3,1,1,4]`, `CanReach(nums, 0)`: `maxJump = 2`, try `step = 2` → `CanReach(nums, 2)`: `maxJump = 1`, try `step = 1` → `CanReach(nums, 3)`: `maxJump = 1`, try `step = 1` → `CanReach(nums, 4)`: `position >= 4` → `true`. This `true` propagates back up through each call, so `CanReach(nums, 0)` returns `true`, matching the expected output.

**Optimized Approach:**
Traverse the array once, maintaining the farthest index reachable so far. If at any index `i` the farthest reachable index is less than `i`, we're stuck and cannot proceed, so return false. Otherwise, keep extending the farthest reachable index.

**Logic (Steps):**
1. Initialize `farthest = 0`.
2. For each index `i`, if `i > farthest`, index `i` is unreachable — return `false`.
3. Otherwise update `farthest = max(farthest, i + nums[i])`.
4. If `farthest >= n - 1`, the last index is already reachable — return `true` early.
5. If the loop completes without early return, return `true`.

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

**Walkthrough:** With `nums = [2,3,1,1,4]`, `n = 5`, `farthest = 0`.
- `i=0`: `0 > 0`? No. `farthest = max(0, 0+2) = 2`. `2 >= 4`? No.
- `i=1`: `1 > 2`? No. `farthest = max(2, 1+3) = 4`. `4 >= 4`? Yes → return `true` immediately.

Result: `true`, matching the expected output.

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

**Logic (Steps):**
1. `MinJumpsFrom(nums, position)`: if `position >= nums.Length - 1`, return `0` (already at the end).
2. Otherwise read `maxJump = nums[position]`.
3. For every `step` from `1` to `maxJump`, recursively compute `subResult = MinJumpsFrom(nums, position + step)`.
4. If `subResult` is a valid (non-infinite) result, update `minJumps = min(minJumps, 1 + subResult)`.
5. Return the smallest `minJumps` found across all step choices.

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

**Walkthrough:** With `nums = [2,3,1,1,4]`, `MinJumpsFrom(nums, 0)` tries `step=2` → `MinJumpsFrom(nums, 2)`, which tries `step=1` → `MinJumpsFrom(nums, 3)`, which tries `step=1` → `MinJumpsFrom(nums, 4)` returns `0` (at end). So `MinJumpsFrom(3) = 1`, `MinJumpsFrom(2) = 2`. Back at position 0, `step=2` gives `1 + 2 = 3`, but trying `step=1` → `MinJumpsFrom(nums, 1)` can jump `step=3` straight to index 4, giving `MinJumpsFrom(1) = 1`, so `step=1` path gives `1 + 1 = 2`. The minimum across all branches is `2`, matching the expected output.

**Optimized Approach:**
Use a greedy BFS-like sweep with two pointers: `currentJumpEnd` marks the boundary of the current jump's range, and `farthest` tracks the farthest index reachable using one more jump. Whenever we reach `currentJumpEnd`, we are forced to take another jump, so we increment the jump count and move the boundary to `farthest`.

**Logic (Steps):**
1. Initialize `jumps = 0`, `currentJumpEnd = 0`, `farthest = 0`.
2. For each index `i` from `0` to `n-2`, update `farthest = max(farthest, i + nums[i])`.
3. If `i == currentJumpEnd`, the current jump's range is exhausted: increment `jumps` and set `currentJumpEnd = farthest`.
4. Continue scanning until the loop ends (index reaches `n-1`).
5. Return `jumps` as the minimum number of jumps.

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

**Walkthrough:** Dry run on `nums = [2,3,1,1,4]` (indices 0..3, since loop runs `i < n-1 = 4`):

| i | nums[i] | farthest = max(farthest, i+nums[i]) | i == currentJumpEnd? | jumps | currentJumpEnd (after update) |
|---|---------|--------------------------------------|-----------------------|-------|--------------------------------|
| 0 | 2       | max(0, 0+2)=2                        | yes (0==0)            | 1     | 2                               |
| 1 | 3       | max(2, 1+3)=4                        | no (1!=2)              | 1     | 2                               |
| 2 | 1       | max(4, 2+1)=4                        | yes (2==2)             | 2     | 4                               |
| 3 | 1       | max(4, 3+1)=4                        | no (3!=4)              | 2     | 4                               |

Loop ends after `i = 3`. Return `jumps = 2`, matching the expected output. `currentJumpEnd` marks the farthest reachable with jumps taken so far; when `i` catches up to it, the current jump's range is exhausted, forcing a new jump and extending the boundary to `farthest` — simulating a level-by-level BFS expansion in O(n).

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

**Logic (Steps):**
1. For each train `i`, start `platformsNeeded = 1` (counting itself).
2. Compare against every other train `j`: if `arrival[j] <= arrival[i] <= departure[j]`, train `j` is still at the station when `i` arrives, so increment `platformsNeeded`.
3. Track `maxPlatforms = max(maxPlatforms, platformsNeeded)` across all trains `i`.
4. Return `maxPlatforms` after checking every train.

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

**Walkthrough:** With `arrival = [900, 940, 950, 1100, 1500, 1800], departure = [910, 1200, 1120, 1130, 1900, 2000]`, checking train at index 2 (`arrival=950`): train 0 (900-910) does not cover 950; train 1 (940-1200) covers 950; train 2 is itself; train 3 (1100-1130) does not cover 950. So `platformsNeeded` for train 2 counts itself plus train 1 = 2 so far, but similarly checking around 940-950 window across all trains yields a peak of 3 overlapping trains (as stated in the problem's example). The maximum `platformsNeeded` over all trains is `3`, matching the expected output.

**Optimized Approach:**
Sort arrival and departure times independently. Use two pointers to sweep through events in time order: whenever a train arrives (before or at the time the earliest still-active train would depart), increment the platform count; whenever a train departs, decrement it. Track the maximum concurrent platform count.

**Logic (Steps):**
1. Clone and sort `arrival` and `departure` arrays independently, ascending.
2. Use two pointers `arrivalPointer` and `departurePointer`, both starting at 0, plus counters `platformsNeeded = 0` and `maxPlatforms = 0`.
3. If `sortedArrival[arrivalPointer] <= sortedDeparture[departurePointer]`, a new train arrives before the earliest pending departure: increment `platformsNeeded`, update `maxPlatforms`, and advance `arrivalPointer`.
4. Otherwise a train departs first: decrement `platformsNeeded` and advance `departurePointer`.
5. Continue until one pointer exhausts the array; return `maxPlatforms`.

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

**Walkthrough:** With `arrival = [900, 940, 950, 1100, 1500, 1800]`, `departure = [910, 1200, 1120, 1130, 1900, 2000]`, sorted independently: `sortedArrival = [900, 940, 950, 1100, 1500, 1800]`, `sortedDeparture = [910, 1120, 1130, 1200, 1900, 2000]`.
- `900 <= 910` → arrive: `platformsNeeded=1`, `maxPlatforms=1`.
- `940 <= 910`? No → depart: `platformsNeeded=0`, `departurePointer=1`.
- `940 <= 1120` → arrive: `platformsNeeded=1`, `maxPlatforms=1`.
- `950 <= 1120` → arrive: `platformsNeeded=2`, `maxPlatforms=2`.
- `1100 <= 1120` → arrive: `platformsNeeded=3`, `maxPlatforms=3`.
- `1500 <= 1130`? No → depart repeatedly until `1500`'s arrival fits, decrementing `platformsNeeded` back down.

The peak `platformsNeeded` reached during the sweep is `3`, so `maxPlatforms = 3`, matching the expected output.

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

**Logic (Steps):**
1. Recurse through jobs by index, tracking a `slots` boolean array (which time slots are occupied), `jobsDone`, and `profit`.
2. For the current job, try placing it in every free slot from its `deadline` down to `1`; mark the slot occupied, recurse, then backtrack (unmark) after returning.
3. Also try skipping the current job entirely (no slot placement).
4. When `index == jobs.Count`, compare `profit` against the best seen `maxProfit` and update `maxProfit`/`maxJobsAtBestProfit` if it's higher.
5. Return `(maxJobsAtBestProfit, maxProfit)` after all branches are explored.

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

**Walkthrough:** With `jobs = [(1,d=4,p=20), (2,d=1,p=10), (3,d=1,p=40), (4,d=1,p=30)]`, one explored branch places job 3 (profit 40) in slot 1, then job 1 (profit 20) in slot 4 (job 2 and job 4 can't find free slots at deadline 1 since it's taken), giving `jobsDone=2, profit=60`. The recursion explores all other combinations too, but none exceeds this profit, so `maxProfit=60`, `maxJobsAtBestProfit=2` is returned, matching the expected output.

**Optimized Approach:**
Sort jobs by profit in descending order. For each job (highest profit first), greedily place it in the latest available free slot at or before its deadline (scanning from `deadline` down to `1`). This ensures earlier slots stay open for jobs with tighter deadlines.

**Logic (Steps):**
1. Sort jobs descending by `Profit`.
2. Create a `slot` array of size `maxDeadline + 1`, initialized to `0` (free), where `slot[d]` stores the job id occupying slot `d`.
3. For each job (highest profit first), scan slots from `min(maxDeadline, job.Deadline)` down to `1`, looking for the first free slot.
4. If a free slot `d` is found, assign `slot[d] = job.Id`, increment `jobsDone`, add `job.Profit` to `totalProfit`, and stop scanning for this job.
5. If no free slot is found at or before its deadline, the job is skipped.
6. Return `(jobsDone, totalProfit)` after processing all jobs.

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

**Walkthrough:** With `jobs = [(1,d=4,p=20), (2,d=1,p=10), (3,d=1,p=40), (4,d=1,p=30)]`, sorted descending by profit: `[job3(p=40,d=1), job4(p=30,d=1), job1(p=20,d=4), job2(p=10,d=1)]`. `slot = [0,0,0,0,0]` (indices 0..4), `jobsDone=0`, `totalProfit=0`.
- job3 (d=1): scan from slot 1 down to 1 → slot 1 free → `slot[1]=3`, `jobsDone=1`, `totalProfit=40`.
- job4 (d=1): scan from slot 1 down to 1 → slot 1 taken → no free slot → skip.
- job1 (d=4): scan from slot 4 down to 1 → slot 4 free → `slot[4]=1`, `jobsDone=2`, `totalProfit=60`.
- job2 (d=1): scan from slot 1 down to 1 → slot 1 taken → skip.

Return `(jobsDone=2, totalProfit=60)`, matching the expected output "Jobs done = 2, Max profit = 60". Choosing the highest-profit jobs first and placing each as late as possible preserves earlier slots for jobs with tighter deadlines, maximizing how many high-profit jobs can still be scheduled.

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

**Logic (Steps):**
1. Initialize every child's `candies` to `1`.
2. Repeat a full left-to-right scan of all children while any change occurred in the previous pass (`changed = true`).
3. For each child `i`, if `ratings[i] > ratings[i-1]` and `candies[i] <= candies[i-1]`, fix it: `candies[i] = candies[i-1] + 1`, mark `changed = true`.
4. Similarly, if `ratings[i] > ratings[i+1]` and `candies[i] <= candies[i+1]`, fix it: `candies[i] = candies[i+1] + 1`, mark `changed = true`.
5. Once a full pass produces no changes, sum and return the `candies` array.

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

**Walkthrough:** With `ratings = [1, 0, 2]`, `candies = [1, 1, 1]` initially.
- Pass 1: `i=1`: `ratings[1]=0 > ratings[0]=1`? No. `ratings[1]=0 > ratings[2]=2`? No. `i=2`: `ratings[2]=2 > ratings[1]=0` and `candies[2]=1 <= candies[1]=1` → `candies[2] = 2`, `changed=true`. `i=0`: `ratings[0]=1 > ratings[1]=0` and `candies[0]=1 <= candies[1]=1` → `candies[0] = 2`, `changed=true`. After pass 1: `candies = [2, 1, 2]`.
- Pass 2: no violations found, `changed=false`, loop ends.

Sum = `2+1+2 = 5`, matching the expected output.

**Optimized Approach:**
Do exactly two linear passes. Left-to-right pass: if `ratings[i] > ratings[i-1]`, then `candies[i] = candies[i-1] + 1` (ensures the right-neighbor-higher constraint). Right-to-left pass: if `ratings[i] > ratings[i+1]`, then `candies[i] = max(candies[i], candies[i+1] + 1)` (ensures the left-neighbor-higher constraint without breaking the first pass's guarantee). Sum the final candies array.

**Logic (Steps):**
1. Initialize every child's `candies` to `1`.
2. Left-to-right pass (`i` from `1` to `n-1`): if `ratings[i] > ratings[i-1]`, set `candies[i] = candies[i-1] + 1`.
3. Right-to-left pass (`i` from `n-2` down to `0`): if `ratings[i] > ratings[i+1]`, set `candies[i] = max(candies[i], candies[i+1] + 1)` (uses `max` so it never undoes the left-to-right pass's guarantee).
4. Sum all values in `candies`.
5. Return the total.

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

**Walkthrough:** Dry run on `ratings = [1, 0, 2]` (n = 3). Initialize: `candies = [1, 1, 1]`.

Left-to-right pass (i = 1 to 2):
- i=1: `ratings[1]=0 > ratings[0]=1`? No → `candies[1]` stays `1`.
- i=2: `ratings[2]=2 > ratings[1]=0`? Yes → `candies[2] = candies[1] + 1 = 2`. `candies = [1, 1, 2]`.

Right-to-left pass (i = 1 down to 0):
- i=1: `ratings[1]=0 > ratings[2]=2`? No → `candies[1]` stays `1`.
- i=0: `ratings[0]=1 > ratings[1]=0`? Yes → `candies[0] = max(1, candies[1]+1) = max(1, 2) = 2`. `candies = [2, 1, 2]`.

Sum = `2 + 1 + 2 = 5`, matching the expected output. The left-to-right pass handles ascending neighbor relationships, and the right-to-left pass fixes descending ones using `max` so it never undoes the first pass's guarantee.

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

**Logic (Steps):**
1. Append `newInterval` to the list of `intervals`, then sort the combined list ascending by start value.
2. Walk the sorted list, maintaining a `merged` result list.
3. If `merged` is empty, or the last merged interval's end is strictly less than the current interval's start, append the current interval as a new entry.
4. Otherwise the current interval overlaps the last merged one: expand the last merged interval's end to `max(lastEnd, current end)`.
5. Return `merged` as an array after processing all intervals.

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

**Walkthrough:** With `intervals = [[1,3],[6,9]]`, `newInterval = [2,5]`, the combined list sorted by start is `[[1,3],[2,5],[6,9]]`.
- `[1,3]`: `merged` empty → add. `merged = [[1,3]]`.
- `[2,5]`: last merged end `3` is not `< 2` → overlap → expand: `merged[-1][1] = max(3,5) = 5`. `merged = [[1,5]]`.
- `[6,9]`: last merged end `5 < 6` → no overlap → add. `merged = [[1,5],[6,9]]`.

Return `[[1,5],[6,9]]`, matching the expected output.

**Optimized Approach:**
Since the original intervals are already sorted and non-overlapping, process them in a single linear pass with three phases: (1) add all intervals that end strictly before the new interval starts (no overlap, come before), (2) merge all intervals that overlap with the new interval by expanding its bounds, then add the merged interval, (3) add all remaining intervals that start strictly after the (possibly expanded) new interval ends.

**Logic (Steps):**
1. Phase 1: while `intervals[i][1] < newInterval[0]`, add `intervals[i]` unchanged to `result` and advance `i`.
2. Phase 2: initialize `mergedStart = newInterval[0]`, `mergedEnd = newInterval[1]`; while `intervals[i][0] <= mergedEnd`, absorb the interval by updating `mergedStart = min(mergedStart, intervals[i][0])` and `mergedEnd = max(mergedEnd, intervals[i][1])`, advancing `i` each time.
3. Add the single merged `[mergedStart, mergedEnd]` interval to `result`.
4. Phase 3: add all remaining intervals (starting after `mergedEnd`) unchanged to `result`.
5. Return `result` as an array.

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

**Walkthrough:** With `intervals = [[1,3],[6,9]]`, `newInterval = [2,5]`, `i = 0`.
- Phase 1: `intervals[0][1]=3 < newInterval[0]=2`? No → skip phase 1, `i` stays `0`.
- Phase 2: `mergedStart=2, mergedEnd=5`. `intervals[0][0]=1 <= 5` → absorb: `mergedStart=min(2,1)=1`, `mergedEnd=max(5,3)=5`, `i=1`. `intervals[1][0]=6 <= 5`? No → stop phase 2. Add `[1,5]` to `result`.
- Phase 3: `intervals[1]=[6,9]` remains → add unchanged. `result = [[1,5],[6,9]]`.

Return `[[1,5],[6,9]]`, matching the expected output.

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

**Logic (Steps):**
1. Recurse through intervals by index, tracking `lastEnd` (end of the last kept interval) and `kept` count.
2. At each index, branch into two options: remove (skip) the current interval, or keep it.
3. Keeping is only valid if `intervals[index][0] >= lastEnd`; if valid, recurse with `intervals[index][1]` as the new `lastEnd` and `kept + 1`.
4. When `index == n`, update `maxKept = max(maxKept, kept)`.
5. Return `n - maxKept` as the minimum number of removals.

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

**Walkthrough:** With `intervals = [[1,2],[2,3],[3,4],[1,3]]`, one branch keeps `[1,2]` (`lastEnd=2`), then `[2,3]` (`2>=2` valid, `lastEnd=3`), then `[3,4]` (`3>=3` valid, `lastEnd=4`), skipping `[1,3]` (`1>=4` invalid) — giving `kept=3`. No branch keeps more than 3 non-overlapping intervals out of 4, so `maxKept=3`. Return `n - maxKept = 4 - 3 = 1`, matching the expected output.

**Optimized Approach:**
Sort intervals by their end time. Greedily keep an interval if its start time is greater than or equal to the end time of the last kept interval; otherwise, it must be removed (since we always prefer to keep the interval that frees up the earliest room for subsequent intervals). Count removals directly.

**Logic (Steps):**
1. Sort `intervals` ascending by end value.
2. Initialize `removals = 0` and `lastEnd = intervals[0][1]`.
3. For each subsequent interval, if `intervals[i][0] < lastEnd`, it overlaps the last kept interval: increment `removals` (discard it, since sorted-by-end means it can't end earlier than the one already kept).
4. Otherwise it doesn't overlap: update `lastEnd = intervals[i][1]` to keep it.
5. Return `removals` after scanning all intervals.

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

**Walkthrough:** With `intervals = [[1,2],[2,3],[3,4],[1,3]]`, sorted by end: `[[1,2],[2,3],[1,3],[3,4]]`. `removals=0`, `lastEnd=2` (from `[1,2]`).
- `[2,3]`: `2 < 2`? No → keep, `lastEnd=3`.
- `[1,3]`: `1 < 3`? Yes → overlap → `removals=1`.
- `[3,4]`: `3 < 3`? No → keep, `lastEnd=4`.

Return `removals=1`, matching the expected output. This is the complement of the "N Meetings in One Room" problem: sorting by end time and greedily keeping the earliest-ending non-conflicting interval maximizes the count retained, minimizing removals.

---
