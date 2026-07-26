# Sorting Techniques — Basic Sorts

## 1. Selection Sort

### 1. Selection Sort

**Problem Statement:** Sort a given array in ascending order using Selection Sort.

**Example:**
- Input: `arr = [64, 25, 12, 22, 11]`
- Output: `[11, 12, 22, 25, 64]`
- Explanation: The algorithm repeatedly scans the unsorted part of the array to find the minimum element and places it at the front of that unsorted part.

**Approach:**
Selection Sort divides the array into a sorted prefix and an unsorted suffix. On each iteration `i` (from `0` to `n-2`), it scans the unsorted suffix `arr[i..n-1]` to find the index of the minimum element, then swaps that minimum element into position `i`. The invariant is that after `i` iterations, `arr[0..i-1]` contains the `i` smallest elements of the array in sorted order, and every element in `arr[0..i-1]` is `<=` every element in `arr[i..n-1]`.

```csharp
public static void SelectionSort(int[] arr)
{
    int n = arr.Length;
    for (int i = 0; i < n - 1; i++)
    {
        int minIndex = i;
        for (int j = i + 1; j < n; j++)
        {
            if (arr[j] < arr[minIndex])
            {
                minIndex = j;
            }
        }

        if (minIndex != i)
        {
            int temp = arr[i];
            arr[i] = arr[minIndex];
            arr[minIndex] = temp;
        }
    }
}
```

Time Complexity:
- Best Case: O(n^2) — even if the array is already sorted, the algorithm still scans the entire unsorted suffix on every iteration to confirm the minimum, so no early exit is possible.
- Average Case: O(n^2) — for a random arrangement, the same n(n-1)/2 comparisons are always performed.
- Worst Case: O(n^2) — reverse sorted input requires the same number of comparisons as any other case; only the number of swaps varies slightly.

Space Complexity: O(1) — sorting is done in-place using only a few auxiliary variables (`minIndex`, `temp`).

Is it stable? No. Selection Sort is not stable because swapping the minimum element into position `i` can jump it past equal elements that appear earlier in the unsorted suffix, changing their relative order. (It can be made stable with extra work — e.g., shifting instead of swapping — but the standard implementation is not stable.)

**Explanation:**
Dry run on `arr = [64, 25, 12, 22, 11]`:

- Pass 1 (i = 0): Scan `[64, 25, 12, 22, 11]`, minimum is `11` at index 4. Swap `arr[0]` and `arr[4]` → `[11, 25, 12, 22, 64]`.
- Pass 2 (i = 1): Scan `[25, 12, 22, 64]` (indices 1–4), minimum is `12` at index 2. Swap `arr[1]` and `arr[2]` → `[11, 12, 25, 22, 64]`.
- Pass 3 (i = 2): Scan `[25, 22, 64]` (indices 2–4), minimum is `22` at index 3. Swap `arr[2]` and `arr[3]` → `[11, 12, 22, 25, 64]`.
- Pass 4 (i = 3): Scan `[25, 64]` (indices 3–4), minimum is `25` at index 3. No swap needed → `[11, 12, 22, 25, 64]`.
- Array is now sorted: `[11, 12, 22, 25, 64]`.

**How to Improve:**
Selection Sort's comparison count is fixed at n(n-1)/2 regardless of input order — there is no way to add an early-exit check to reduce it below O(n^2), because the algorithm must always inspect the entire unsorted suffix to guarantee it has found the true minimum. What *can* be optimized is the number of swaps: the basic version already performs at most `n - 1` swaps (one per outer iteration, skipped when `minIndex == i`), which is optimal — this makes Selection Sort attractive when write operations are far more expensive than comparisons (e.g., writing to flash memory). A further variant, "double-ended selection sort," finds both the minimum and maximum in each pass and places them at both ends simultaneously, roughly halving the number of passes, but the asymptotic comparison complexity remains O(n^2).

---

## 2. Bubble Sort

### 2. Bubble Sort

**Problem Statement:** Sort a given array in ascending order using Bubble Sort.

**Example:**
- Input: `arr = [64, 25, 12, 22, 11]`
- Output: `[11, 12, 22, 25, 64]`
- Explanation: The algorithm repeatedly steps through the array, swapping adjacent elements that are out of order, so that on each full pass the largest remaining element "bubbles up" to its correct position at the end.

**Approach:**
Bubble Sort repeatedly traverses the array and compares each pair of adjacent elements, swapping them if they are in the wrong order. After the first full pass, the largest element is guaranteed to be at the last index. After the second pass, the second-largest is in place, and so on. The invariant is that after `i` passes, the last `i` elements `arr[n-i..n-1]` are the `i` largest elements, sorted in place, so each subsequent pass only needs to consider `arr[0..n-i-1]`.

```csharp
public static void BubbleSort(int[] arr)
{
    int n = arr.Length;
    for (int i = 0; i < n - 1; i++)
    {
        for (int j = 0; j < n - 1 - i; j++)
        {
            if (arr[j] > arr[j + 1])
            {
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
            }
        }
    }
}
```

Time Complexity:
- Best Case: O(n^2) for this basic version — it always runs the full nested loop regardless of whether the array is already sorted, because there is no mechanism to detect "no swaps occurred." (With the early-exit optimization shown below, best case becomes O(n) for an already-sorted array, since the single pass with no swaps triggers an immediate exit.)
- Average Case: O(n^2) — roughly n^2/4 swaps and n^2/2 comparisons are expected for random data.
- Worst Case: O(n^2) — occurs for a reverse-sorted array, where every adjacent pair in every pass needs to be swapped.

Space Complexity: O(1) — sorting is done in-place using only a temporary variable for swapping.

Is it stable? Yes. Bubble Sort is stable because it only swaps adjacent elements when the left one is strictly greater than the right one — equal elements are never swapped, so their relative order is preserved.

**Explanation:**
Dry run on `arr = [64, 25, 12, 22, 11]`:

- Pass 1 (i = 0), comparing adjacent pairs from j = 0 to 3:
  - `64 > 25` → swap → `[25, 64, 12, 22, 11]`
  - `64 > 12` → swap → `[25, 12, 64, 22, 11]`
  - `64 > 22` → swap → `[25, 12, 22, 64, 11]`
  - `64 > 11` → swap → `[25, 12, 22, 11, 64]`
  - End of pass 1: largest element `64` has bubbled to the last index. Array: `[25, 12, 22, 11, 64]`.
- Pass 2 (i = 1), comparing j = 0 to 2 (last index now excluded):
  - `25 > 12` → swap → `[12, 25, 22, 11, 64]`
  - `25 > 22` → swap → `[12, 22, 25, 11, 64]`
  - `25 > 11` → swap → `[12, 22, 11, 25, 64]`
  - End of pass 2: second-largest `25` is now in place. Array: `[12, 22, 11, 25, 64]`.
- Remaining passes continue similarly until the array becomes `[11, 12, 22, 25, 64]`.

**How to Improve:**
The basic version always performs all `n-1` passes even if the array becomes sorted early. The optimization is to track whether any swap occurred during a pass — if a full pass completes with zero swaps, the array is already sorted and the algorithm can exit immediately, giving a best-case O(n) for already-sorted (or nearly-sorted) input while keeping the same O(n^2) worst/average case.

```csharp
public static void BubbleSortOptimized(int[] arr)
{
    int n = arr.Length;
    for (int i = 0; i < n - 1; i++)
    {
        bool swapped = false;
        for (int j = 0; j < n - 1 - i; j++)
        {
            if (arr[j] > arr[j + 1])
            {
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
                swapped = true;
            }
        }

        if (!swapped)
        {
            // No swaps in this pass means the array is already sorted.
            break;
        }
    }
}
```

---

## 3. Insertion Sort

### 3. Insertion Sort

**Problem Statement:** Sort a given array in ascending order using Insertion Sort.

**Example:**
- Input: `arr = [64, 25, 12, 22, 11]`
- Output: `[11, 12, 22, 25, 64]`
- Explanation: The algorithm builds a sorted prefix one element at a time, taking each next element from the unsorted part and inserting it into its correct position within the already-sorted prefix by shifting larger elements to the right.

**Approach:**
Insertion Sort maintains a sorted prefix `arr[0..i-1]` and, on each iteration, takes the element at index `i` (the "key"), then shifts all elements in the sorted prefix that are greater than the key one position to the right, finally placing the key into the resulting gap. The invariant is that after processing index `i`, `arr[0..i]` is fully sorted, though not necessarily in its final overall position since later insertions can still shift these elements.

```csharp
public static void InsertionSort(int[] arr)
{
    int n = arr.Length;
    for (int i = 1; i < n; i++)
    {
        int key = arr[i];
        int j = i - 1;

        while (j >= 0 && arr[j] > key)
        {
            arr[j + 1] = arr[j];
            j--;
        }

        arr[j + 1] = key;
    }
}
```

Time Complexity:
- Best Case: O(n) — occurs when the array is already sorted; the inner `while` loop condition `arr[j] > key` fails immediately for every `i`, so each element requires only one comparison and no shifting.
- Average Case: O(n^2) — for random data, each key is expected to shift past roughly half of the sorted prefix.
- Worst Case: O(n^2) — occurs for a reverse-sorted array, where each key must shift past all previously sorted elements.

Space Complexity: O(1) — sorting is done in-place using only the `key`, `i`, and `j` variables.

Is it stable? Yes. Insertion Sort is stable because the shifting only moves an element right when it is strictly greater than the key, so equal elements are never reordered relative to each other — the key is always inserted immediately after any equal elements already in the sorted prefix.

**Explanation:**
Dry run on `arr = [64, 25, 12, 22, 11]`:

- Pass 1 (i = 1): key = `25`. Compare with `arr[0] = 64`: `64 > 25`, shift `64` right → `[64, 64, 12, 22, 11]`. j becomes -1, loop ends. Insert key at index 0 → `[25, 64, 12, 22, 11]`.
- Pass 2 (i = 2): key = `12`. Compare with `arr[1] = 64`: shift → `[25, 64, 64, 22, 11]`. Compare with `arr[0] = 25`: shift → `[25, 25, 64, 22, 11]`. j becomes -1, insert key at index 0 → `[12, 25, 64, 22, 11]`.
- Pass 3 (i = 3): key = `22`. Compare with `arr[2] = 64`: shift → `[12, 25, 64, 64, 11]`. Compare with `arr[1] = 25`: shift → `[12, 25, 25, 64, 11]`. Compare with `arr[0] = 12`: `12 > 22` is false, stop. Insert key at index 1 → `[12, 22, 25, 64, 11]`.
- Pass 4 (i = 4): key = `11`. Shifts past `64`, `25`, `22`, `12` (all greater than 11) → `[12, 22, 25, 64, 64]` progressively becomes `[22, 25, 64, ...]` shifted, and finally `11` is inserted at index 0 → `[11, 12, 22, 25, 64]`.
- Array is now sorted: `[11, 12, 22, 25, 64]`.

**How to Improve:**
Insertion Sort is already the best choice among the three basic sorts for nearly-sorted or small datasets, since its best case is O(n) and it does minimal work when elements are close to their final position — this is why it's often used as the base case for hybrid algorithms like Timsort and as the fallback for small partitions in Quicksort/Introsort. One refinement is **Binary Insertion Sort**: instead of linearly scanning the sorted prefix to find the insertion point, use binary search (O(log n) per element) to locate where the key belongs. This reduces the total number of *comparisons* to O(n log n), but the overall time complexity remains O(n^2) in the worst case, because inserting the key still requires shifting up to `i` elements to open a gap — an O(n) operation per insertion that binary search cannot avoid. The tradeoff is worthwhile only when comparisons are expensive relative to shifts (e.g., comparing large objects or strings), since it lowers comparison cost but not the dominant shifting cost.

```csharp
public static void BinaryInsertionSort(int[] arr)
{
    int n = arr.Length;
    for (int i = 1; i < n; i++)
    {
        int key = arr[i];
        int low = 0, high = i - 1;

        // Binary search for the correct insertion position within arr[0..i-1].
        while (low <= high)
        {
            int mid = low + (high - low) / 2;
            if (arr[mid] > key)
            {
                high = mid - 1;
            }
            else
            {
                low = mid + 1;
            }
        }

        // Shift elements to make room for the key (still O(n) per insertion).
        for (int j = i - 1; j >= low; j--)
        {
            arr[j + 1] = arr[j];
        }

        arr[low] = key;
    }
}
```
