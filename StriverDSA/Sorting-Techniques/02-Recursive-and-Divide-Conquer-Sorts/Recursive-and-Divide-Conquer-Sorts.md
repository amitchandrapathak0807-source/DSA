# Sorting Techniques — Recursive and Divide & Conquer Sorts

## 1. Recursive Bubble Sort

### 1. Recursive Bubble Sort
**Problem Statement:** Sort a given array in ascending order using Bubble Sort, implemented recursively.

**Example:**
- Input: `arr = [5, 2, 9, 1, 5, 6]`
- Output: `[1, 2, 5, 5, 6, 9]`

**Approach:**
The recursive version of Bubble Sort works on the same principle as the iterative one — repeatedly swap adjacent elements that are out of order so the largest element "bubbles" to the end of the array. Instead of using a nested loop, recursion is used to reduce the problem size:
1. Base case: if `n == 1`, the array (of size 1) is trivially sorted, so return.
2. Perform a single pass over the first `n` elements, swapping adjacent out-of-order elements. After this pass, the largest element among the first `n` elements is at index `n - 1`.
3. Recursively call the function on the first `n - 1` elements.

Each recursive call shrinks the "unsorted" boundary by one, exactly like one outer-loop iteration of the iterative version.

```csharp
public static class RecursiveBubbleSort
{
    public static void Sort(int[] arr, int n)
    {
        // Base case: array of size 0 or 1 is already sorted
        if (n <= 1)
        {
            return;
        }

        // One pass: bubble the largest element of arr[0..n-1] to the end
        for (int i = 0; i < n - 1; i++)
        {
            if (arr[i] > arr[i + 1])
            {
                int temp = arr[i];
                arr[i] = arr[i + 1];
                arr[i + 1] = temp;
            }
        }

        // Recurse on the remaining n - 1 elements
        Sort(arr, n - 1);
    }
}
```

Time Complexity:
- Best Case: O(n^2) — even if the array is already sorted, this naive recursive version still performs a full pass at every recursion level (no early-exit flag), so it always does about n + (n-1) + ... + 1 comparisons. (An optimized variant can add an early-stop flag to achieve O(n) best case, just like iterative bubble sort.)
- Average Case: O(n^2) — roughly n^2/2 comparisons and swaps across all recursive passes.
- Worst Case: O(n^2) — occurs when the array is sorted in reverse order, requiring the maximum number of swaps.

Space Complexity: O(n) auxiliary — due to the recursion call stack, since there are `n` nested recursive calls before the base case is hit (in-place sorting otherwise, no extra array is used).

Is it stable? Yes — equal elements are never swapped (swap only happens on strict `>`), so relative order of equal elements is preserved.

**Explanation:**
No special dry run format is requested for this algorithm beyond the general trace: for `arr = [5, 2, 9, 1, 5, 6]`, `Sort(arr, 6)` bubbles the max (9) to index 5, then `Sort(arr, 5)` bubbles the next max (6) to index 4, and so on until `Sort(arr, 1)` hits the base case, leaving the array fully sorted.

**How to Improve:**
- Add an early-termination flag: if a full pass makes zero swaps, the array is already sorted and recursion can stop immediately, improving the best case to O(n).
- For small subarrays (small `n`), switch to a simple insertion sort pass — this is a common hybrid-sort optimization (similar in spirit to Timsort) since insertion sort has lower constant-factor overhead on nearly-sorted or tiny inputs.
- In general, prefer Merge Sort or Quick Sort over recursive Bubble Sort for real workloads — Bubble Sort's O(n^2) behavior makes it impractical for large inputs regardless of recursion.

## 2. Recursive Insertion Sort

### 2. Recursive Insertion Sort
**Problem Statement:** Sort a given array in ascending order using Insertion Sort, implemented recursively.

**Example:**
- Input: `arr = [5, 2, 9, 1, 5, 6]`
- Output: `[1, 2, 5, 5, 6, 9]`

**Approach:**
Recursive Insertion Sort mirrors the iterative version's logic but expresses it top-down:
1. Base case: if `n <= 1`, an array of size 0 or 1 is already sorted, so return.
2. Recursively sort the first `n - 1` elements: `Sort(arr, n - 1)`.
3. Once the first `n - 1` elements are sorted, take the `n`-th element (`arr[n - 1]`) and insert it into its correct position within the already-sorted first `n - 1` elements, by shifting larger elements one position to the right.

This "sort smaller problem first, then insert the extra element" pattern is the hallmark of recursive Insertion Sort.

```csharp
public static class RecursiveInsertionSort
{
    public static void Sort(int[] arr, int n)
    {
        // Base case: array of size 0 or 1 is already sorted
        if (n <= 1)
        {
            return;
        }

        // Recursively sort the first n - 1 elements
        Sort(arr, n - 1);

        // Insert the n-th element (arr[n - 1]) into the sorted arr[0..n-2]
        int last = arr[n - 1];
        int j = n - 2;

        while (j >= 0 && arr[j] > last)
        {
            arr[j + 1] = arr[j];
            j--;
        }

        arr[j + 1] = last;
    }
}
```

Time Complexity:
- Best Case: O(n) — occurs when the array is already sorted; the inner `while` loop never shifts elements (single comparison per insertion).
- Average Case: O(n^2) — each insertion, on average, shifts roughly half of the already-sorted prefix.
- Worst Case: O(n^2) — occurs when the array is sorted in reverse order, so every insertion shifts the entire sorted prefix.

Space Complexity: O(n) auxiliary — due to the recursion call stack depth of `n` (in-place sorting otherwise, no extra array is used).

Is it stable? Yes — the shifting-insertion process only moves an element when a strictly greater element is found, so equal elements retain their relative order.

**Explanation:**
General trace for `arr = [5, 2, 9, 1, 5, 6]`: recursion unwinds down to `Sort(arr, 1)` (base case with just `[5]`), then as calls return, each level inserts one more element into the growing sorted prefix — first `[5]`, then insert `2` to get `[2, 5]`, then insert `9` to get `[2, 5, 9]`, then insert `1` to get `[1, 2, 5, 9]`, then insert `5` to get `[1, 2, 5, 5, 9]`, then insert `6` to get the final sorted array `[1, 2, 5, 5, 6, 9]`.

**How to Improve:**
- For larger arrays, insertion sort's O(n^2) average case becomes a bottleneck — use it as the base-case optimization for a hybrid sort (e.g., switch from Merge Sort or Quick Sort to insertion sort once the subarray size drops below a threshold like 10–16 elements), since insertion sort has excellent constant factors on small/nearly-sorted inputs. This is exactly the strategy Timsort uses.
- Use binary search to find the insertion point (binary insertion sort) to reduce comparisons from O(n) to O(log n) per insertion, though shifting still costs O(n), so overall complexity remains O(n^2) — this mainly helps when comparisons are expensive.

## 3. Merge Sort

### 3. Merge Sort
**Problem Statement:** Sort a given array in ascending order using Merge Sort, implemented recursively.

**Example:**
- Input: `arr = [5, 2, 9, 1, 5, 6]`
- Output: `[1, 2, 5, 5, 6, 9]`

**Approach:**
Merge Sort is a classic Divide and Conquer algorithm:
1. **Divide:** split the array into two halves using the midpoint index.
2. **Conquer:** recursively sort each half.
3. **Combine:** merge the two sorted halves back into a single sorted array using an auxiliary array, by repeatedly picking the smaller of the two current front elements.

Recursion bottoms out when a subarray has 0 or 1 elements (trivially sorted), and the merge step does all the real work of restoring order across the two halves.

```csharp
public static class MergeSortAlgo
{
    public static void MergeSort(int[] arr, int low, int high)
    {
        // Base case: a single element (or empty range) is already sorted
        if (low >= high)
        {
            return;
        }

        int mid = low + (high - low) / 2;

        MergeSort(arr, low, mid);       // sort left half
        MergeSort(arr, mid + 1, high);  // sort right half
        Merge(arr, low, mid, high);     // merge the two sorted halves
    }

    private static void Merge(int[] arr, int low, int mid, int high)
    {
        int[] temp = new int[high - low + 1];
        int left = low;
        int right = mid + 1;
        int k = 0;

        // Merge the two sorted halves into temp
        while (left <= mid && right <= high)
        {
            if (arr[left] <= arr[right])
            {
                temp[k++] = arr[left++];
            }
            else
            {
                temp[k++] = arr[right++];
            }
        }

        // Copy any remaining elements from the left half
        while (left <= mid)
        {
            temp[k++] = arr[left++];
        }

        // Copy any remaining elements from the right half
        while (right <= high)
        {
            temp[k++] = arr[right++];
        }

        // Copy merged result back into the original array
        for (int i = 0; i < temp.Length; i++)
        {
            arr[low + i] = temp[i];
        }
    }
}
```

Time Complexity: O(n log n) in the best, average, and worst case. The array is always split into two halves (log n levels of recursion, since the array size halves each time), and merging at each level touches all n elements across the subarrays, giving n work per level and log n levels — hence O(n log n) regardless of input order.

Space Complexity: O(n) auxiliary — the `Merge` step allocates a temporary array proportional to the size of the range being merged; across the recursion this totals O(n) extra space (plus O(log n) recursion stack, dominated by the O(n) temp array usage).

Is it stable? Yes — in the `Merge` step, when `arr[left] == arr[right]`, the element from the left half (`arr[left] <= arr[right]`) is picked first, preserving the original relative order of equal elements.

**Explanation:**
Dry run of `MergeSort(arr, 0, 5)` on `arr = [5, 2, 9, 1, 5, 6]`:

Divide phase (recursion tree splitting down to single elements):
```
[5, 2, 9, 1, 5, 6]
        |
   split at mid
   /            \
[5, 2, 9]      [1, 5, 6]
   |                |
 split            split
 /    \            /    \
[5]  [2, 9]      [1]   [5, 6]
       |                 |
     split              split
     /   \               /  \
   [2]   [9]           [5]  [6]
```
Every subarray keeps splitting until each has exactly one element (base case `low >= high`), which is trivially sorted.

Merge phase (combining back up the tree):
- Merge `[2]` and `[9]` → `[2, 9]`
- Merge `[5]` and `[2, 9]` → `[2, 5, 9]`
- Merge `[5]` and `[6]` → `[5, 6]`
- Merge `[1]` and `[5, 6]` → `[1, 5, 6]`
- Finally, merge `[2, 5, 9]` and `[1, 5, 6]`:
  - Compare 2 vs 1 → take 1 → `[1]`
  - Compare 2 vs 5 → take 2 → `[1, 2]`
  - Compare 5 vs 5 → take left's 5 (stability) → `[1, 2, 5]`
  - Compare 9 vs 5 → take 5 → `[1, 2, 5, 5]`
  - Compare 9 vs 6 → take 6 → `[1, 2, 5, 5, 6]`
  - Right half exhausted, copy remaining 9 → `[1, 2, 5, 5, 6, 9]`

Final sorted array: `[1, 2, 5, 5, 6, 9]`.

**How to Improve:**
- Switch to Insertion Sort for small subarrays (e.g., size <= 10–16) instead of recursing all the way down to size 1 — insertion sort has lower constant-factor overhead on small inputs, and this hybrid approach (used by Timsort and many production sort implementations) reduces overall runtime.
- Avoid allocating a new temporary array on every merge call; instead, allocate one auxiliary array of size n up front and reuse it across all merge calls, reducing memory allocation overhead.
- Check if `arr[mid] <= arr[mid + 1]` before merging — if true, the two halves are already in order and the merge step can be skipped entirely (useful for nearly-sorted data).

## 4. Quick Sort

### 4. Quick Sort
**Problem Statement:** Sort a given array in ascending order using Quick Sort, implemented recursively.

**Example:**
- Input: `arr = [5, 2, 9, 1, 5, 6]`
- Output: `[1, 2, 5, 5, 6, 9]`

**Approach:**
Quick Sort is also a Divide and Conquer algorithm, but unlike Merge Sort it does the "hard work" during the divide step instead of the combine step:
1. **Choose a pivot** element from the array (e.g., the last element, using the Lomuto partition scheme).
2. **Partition:** rearrange the array so all elements smaller than the pivot come before it, and all elements greater come after it. After partitioning, the pivot is in its final sorted position.
3. **Recurse:** recursively apply Quick Sort to the subarray of elements before the pivot and the subarray of elements after the pivot.

No explicit "merge" step is needed because partitioning already places the pivot correctly, and once both sides are recursively sorted the whole array is sorted.

```csharp
public static class QuickSortAlgo
{
    public static void QuickSort(int[] arr, int low, int high)
    {
        if (low < high)
        {
            int pivotIndex = Partition(arr, low, high);

            QuickSort(arr, low, pivotIndex - 1);   // sort left partition
            QuickSort(arr, pivotIndex + 1, high);  // sort right partition
        }
    }

    // Lomuto partition scheme: uses the last element as pivot
    private static int Partition(int[] arr, int low, int high)
    {
        int pivot = arr[high];
        int i = low - 1; // boundary of elements <= pivot

        for (int j = low; j < high; j++)
        {
            if (arr[j] <= pivot)
            {
                i++;
                Swap(arr, i, j);
            }
        }

        Swap(arr, i + 1, high); // place pivot in its correct position
        return i + 1;
    }

    private static void Swap(int[] arr, int a, int b)
    {
        int temp = arr[a];
        arr[a] = arr[b];
        arr[b] = temp;
    }
}
```

Time Complexity:
- Best Case: O(n log n) — occurs when the pivot consistently splits the array into two roughly equal halves, giving log n levels of recursion with O(n) partitioning work per level.
- Average Case: O(n log n) — for random input data, pivots tend to split the array reasonably well on average.
- Worst Case: O(n^2) — occurs when the pivot is always the smallest or largest element (e.g., picking the last element as pivot on an already-sorted or reverse-sorted array), causing maximally unbalanced partitions (one side has n-1 elements, the other has 0), leading to n levels of recursion with O(n) work each. This is why naive fixed-position pivot selection is risky; using a **randomized pivot** or **median-of-three pivot selection** makes the worst case extremely unlikely in practice, restoring expected O(n log n) behavior even on sorted or adversarial input.

Space Complexity: O(log n) auxiliary on average — Quick Sort partitions in-place (no extra array like Merge Sort), so space is dominated by the recursion call stack, which is O(log n) deep for balanced partitions. In the worst case (highly unbalanced partitions), the stack depth can degrade to O(n).

Is it stable? No — the partitioning step swaps elements based on position relative to the pivot without regard to preserving the relative order of equal elements, so equal elements can be reordered.

**Explanation:**
Dry run using the Lomuto partition scheme (pivot = last element) on `arr = [5, 2, 9, 1, 5, 6]`, `QuickSort(arr, 0, 5)`:

**First partition call:** `Partition(arr, 0, 5)`, pivot = `arr[5] = 6`, `i = -1`
- j=0: arr[0]=5 <= 6 → i=0, swap(0,0) → `[5, 2, 9, 1, 5, 6]`
- j=1: arr[1]=2 <= 6 → i=1, swap(1,1) → `[5, 2, 9, 1, 5, 6]`
- j=2: arr[2]=9 > 6 → skip
- j=3: arr[3]=1 <= 6 → i=2, swap(2,3) → `[5, 2, 1, 9, 5, 6]`
- j=4: arr[4]=5 <= 6 → i=3, swap(3,4) → `[5, 2, 1, 5, 9, 6]`
- End of loop, swap(i+1=4, high=5) → `[5, 2, 1, 5, 6, 9]`
- Pivot 6 is now at index 4 (its correct sorted position). Returns pivotIndex = 4.

Array after this partition: `[5, 2, 1, 5, 6, 9]`. Recurse on left partition `[0, 3]` = `[5, 2, 1, 5]` and right partition `[5, 5]` = `[9]` (single element, base case, already sorted).

**Partition on `[0, 3]`:** `arr = [5, 2, 1, 5, ...]`, pivot = `arr[3] = 5`, `i = -1`
- j=0: arr[0]=5 <= 5 → i=0, swap(0,0) → `[5, 2, 1, 5, ...]`
- j=1: arr[1]=2 <= 5 → i=1, swap(1,1) → `[5, 2, 1, 5, ...]`
- j=2: arr[2]=1 <= 5 → i=2, swap(2,2) → `[5, 2, 1, 5, ...]`
- End of loop, swap(i+1=3, high=3) → `[5, 2, 1, 5, ...]` (no change, pivot already at index 3)
- Returns pivotIndex = 3. Recurse on left `[0, 2]` = `[5, 2, 1]` and right `[4, 3]` (empty, base case).

**Partition on `[0, 2]`:** `arr = [5, 2, 1, ...]`, pivot = `arr[2] = 1`, `i = -1`
- j=0: arr[0]=5 > 1 → skip
- j=1: arr[1]=2 > 1 → skip
- End of loop, swap(i+1=0, high=2) → `[1, 2, 5, ...]`
- Returns pivotIndex = 0. Recurse on left `[0, -1]` (empty, base case) and right `[1, 2]` = `[2, 5]`.

**Partition on `[1, 2]`:** `arr = [..., 2, 5, ...]`, pivot = `arr[2] = 5`, `i = 0`
- j=1: arr[1]=2 <= 5 → i=1, swap(1,1) → no change
- End of loop, swap(i+1=2, high=2) → no change
- Returns pivotIndex = 2. Both sides now single elements or empty (base case).

At this point the array is `[1, 2, 5, 5, 6, 9]`, fully sorted. All remaining recursive calls hit the base case `low >= high` and return immediately.

**How to Improve:**
- **Randomized pivot selection:** before partitioning, swap a randomly chosen element into the pivot position (e.g., the last index). This makes the worst-case O(n^2) scenario extremely unlikely regardless of input order, since the adversary can no longer predict which element will be chosen as pivot.
- **Median-of-three pivot selection:** pick the median of the first, middle, and last elements as the pivot. This avoids consistently bad splits on already-sorted or reverse-sorted arrays without the overhead of full randomization.
- **Hybrid with Insertion Sort:** switch to insertion sort once a partition's size drops below a small threshold (e.g., 10–16 elements), since insertion sort has lower constant-factor overhead on small arrays — this is the same hybrid-sort idea used to optimize Merge Sort and is common in real-world sort implementations (e.g., introsort, which also falls back to heap sort to guarantee O(n log n) worst case).
- **Tail-call/smaller-first recursion:** recurse first on the smaller partition and loop (rather than recurse) on the larger one, bounding the recursion stack depth to O(log n) even in unbalanced cases.
