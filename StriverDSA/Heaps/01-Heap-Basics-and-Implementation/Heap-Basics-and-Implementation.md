# Heaps — Basics and Implementation

## Concept: Binary Heap

A **Binary Heap** is a **complete binary tree** that satisfies the **heap-order property**:

- **Min-Heap**: every parent node's value is **less than or equal to** the values of its children. The smallest element always sits at the root (index `0`).
- **Max-Heap**: every parent node's value is **greater than or equal to** the values of its children. The largest element always sits at the root (index `0`).

Note that a heap only enforces the relationship between a **parent and its immediate children** — it says nothing about the ordering between siblings or across different subtrees. This is what makes a heap weaker than a fully sorted structure, but also what makes insert/extract operations run in `O(log n)` instead of `O(n)`.

### Complete Binary Tree as an Array

Because a heap is always a **complete binary tree** (every level is fully filled except possibly the last, which is filled left to right), it can be stored compactly in a plain array — no pointers to children/parent are needed. For a node stored at index `i` (0-indexed array):

- **Parent index**: `(i - 1) / 2` (integer division)
- **Left child index**: `2 * i + 1`
- **Right child index**: `2 * i + 2`

```
Array:  [1, 3, 6, 5, 9, 8]
Index:   0  1  2  3  4  5

Tree:
                1(0)
              /      \
           3(1)       6(2)
          /    \      /
       5(3)   9(4)  8(5)
```

This array representation is what every implementation below uses — no explicit `Node` class with `left`/`right`/`parent` pointers is required.

### Heapify-Up (Bubble Up) — used during Insert

When a new element is appended at the end of the array (the next free leaf slot, keeping the tree complete), it may violate the heap property with its parent. **Heapify-up** repeatedly compares the new node with its parent and swaps if the heap property is violated, moving the node up the tree until either the root is reached or the parent is no longer violated.

```
insert(x):
    array.append(x)
    i = array.length - 1
    while i > 0 and array[parent(i)] violates order with array[i]:
        swap(array[i], array[parent(i)])
        i = parent(i)
```

### Heapify-Down (Sift Down / Bubble Down) — used during Extract and Build-Heap

When the root is removed (extract-min/max), the **last element** of the array is moved to the root to keep the tree complete, and then **heapify-down** repeatedly compares this node with its children, swapping with the child that would violate the heap property the most (smallest child for a min-heap, largest child for a max-heap), moving the node down until it settles into a valid position.

```
heapifyDown(i, n):
    smallest (or largest) = i
    left = 2*i + 1
    right = 2*i + 2
    if left < n and array[left] beats array[smallest]: smallest = left
    if right < n and array[right] beats array[smallest]: smallest = right
    if smallest != i:
        swap(array[i], array[smallest])
        heapifyDown(smallest, n)
```

`heapifyDown` is also the workhorse behind **building a heap from an arbitrary array in `O(n)`** (see Problem 2's complexity analysis) — it is called on every internal node from the last non-leaf node up to the root.

---

## 1. Introduction to Priority Queues Using Binary Heaps

**Problem Statement:**
A **Priority Queue** is an abstract data type where each element has a priority, and elements are served according to their priority rather than insertion order (unlike a plain FIFO queue). A binary heap is the classic underlying data structure for an efficient priority queue because it supports the three core operations — `Insert`, `Peek` (look at the highest-priority element), and `Extract` (remove and return the highest-priority element) — all in `O(log n)` or better. Demonstrate these basic operations using a min-heap-backed priority queue (smallest value = highest priority).

**Example:**
- Input: a sequence of operations — `Insert(5), Insert(3), Insert(8), Insert(1), Peek(), ExtractMin(), Peek()`
- Output: `Peek() -> 1`, `ExtractMin() -> 1`, `Peek() -> 3`
- Explanation: After inserting `5, 3, 8, 1`, the smallest element `1` sits at the root, so `Peek` returns `1`. `ExtractMin` removes and returns `1`, after which the new smallest element `3` becomes the root, so the next `Peek` returns `3`.

**Approach:**
`Insert` appends the new element at the end of the internal array (preserving completeness) and then runs **heapify-up** to restore the heap property against its ancestors. `Peek` simply returns `array[0]` in `O(1)` since the root is always the min (or max). `ExtractMin`/`ExtractMax` saves `array[0]` to return, moves the last element to index `0`, shrinks the array by one, and runs **heapify-down** from the root to restore the heap property.

```csharp
using System;
using System.Collections.Generic;

public class PriorityQueueMinHeap
{
    private List<int> heap = new List<int>();

    public int Count => heap.Count;

    private int Parent(int i) => (i - 1) / 2;
    private int Left(int i) => 2 * i + 1;
    private int Right(int i) => 2 * i + 2;

    public void Insert(int value)
    {
        heap.Add(value);
        HeapifyUp(heap.Count - 1);
    }

    public int Peek()
    {
        if (heap.Count == 0) throw new InvalidOperationException("Priority queue is empty");
        return heap[0];
    }

    public int ExtractMin()
    {
        if (heap.Count == 0) throw new InvalidOperationException("Priority queue is empty");

        int min = heap[0];
        int last = heap.Count - 1;
        heap[0] = heap[last];
        heap.RemoveAt(last);

        if (heap.Count > 0)
            HeapifyDown(0);

        return min;
    }

    private void HeapifyUp(int i)
    {
        while (i > 0 && heap[Parent(i)] > heap[i])
        {
            (heap[Parent(i)], heap[i]) = (heap[i], heap[Parent(i)]);
            i = Parent(i);
        }
    }

    private void HeapifyDown(int i)
    {
        int n = heap.Count;
        while (true)
        {
            int smallest = i;
            int left = Left(i);
            int right = Right(i);

            if (left < n && heap[left] < heap[smallest]) smallest = left;
            if (right < n && heap[right] < heap[smallest]) smallest = right;

            if (smallest == i) break;

            (heap[i], heap[smallest]) = (heap[smallest], heap[i]);
            i = smallest;
        }
    }
}
```

Time Complexity: `Insert` is `O(log n)` (heapify-up traverses at most tree height), `ExtractMin` is `O(log n)` (heapify-down traverses at most tree height), `Peek` is `O(1)`. Space Complexity: `O(n)` to store the `n` elements.

**Explanation:**
Dry run of `Insert(5), Insert(3), Insert(8), Insert(1)`:
- `Insert(5)`: heap = `[5]`. No parent, nothing to bubble.
- `Insert(3)`: heap = `[5, 3]`. `3` is at index 1, parent index `0` = `5`. Since `5 > 3`, swap → heap = `[3, 5]`.
- `Insert(8)`: heap = `[3, 5, 8]`. `8` is at index 2, parent index `0` = `3`. Since `3 < 8`, no swap.
- `Insert(1)`: heap = `[3, 5, 8, 1]`. `1` is at index 3, parent index `1` = `5`. Since `5 > 1`, swap → `[3, 1, 8, 5]`. Now `1` is at index 1, parent index `0` = `3`. Since `3 > 1`, swap → `[1, 3, 8, 5]`. Index 0 reached, stop.

`Peek()` returns `heap[0] = 1`.

`ExtractMin()`: save `min = 1`. Move last element (`5`, at index 3) to root → `[5, 3, 8]` (after removing the trailing slot). Heapify-down from index 0: children are index 1 (`3`) and index 2 (`8`); smallest child is `3` at index 1, and `3 < 5`, so swap → `[3, 5, 8]`. Index 1 is now a leaf (no children within bounds), stop. Returns `1`.

`Peek()` now returns `heap[0] = 3`, matching the expected output.

---

## 2. Implement a Min Heap and a Max Heap from Scratch (array-based)

**Problem Statement:**
Implement both a **Min Heap** and a **Max Heap** from scratch using only a plain array (`List<int>` in C#) as the backing store — no built-in `PriorityQueue<T>` allowed. Both must support `Insert`, `ExtractMin`/`ExtractMax`, `Peek`, and building a heap directly from an existing unsorted array (`Heapify`/`BuildHeap`) in linear time.

**Example:**
- Input: `arr = [3, 5, 9, 6, 8, 20, 10, 12, 18, 9]` used to build a heap in-place.
- Output: MinHeap built from `arr` → root = `3` (smallest); MaxHeap built from the same `arr` → root = `20` (largest).
- Explanation: `BuildHeap` rearranges the array in-place so the heap-order property holds at every parent-child pair; the overall smallest (or largest) value bubbles up to index `0`.

**Approach:**
Both heaps share the same array-based skeleton (`Parent = (i-1)/2`, `Left = 2i+1`, `Right = 2i+2`); only the comparison direction differs (`<` for min-heap, `>` for max-heap). `BuildHeap(arr)` copies the input array and calls `HeapifyDown` starting from the **last non-leaf node** (`index = n/2 - 1`) down to index `0`, in reverse order — this bottom-up construction is what achieves `O(n)` instead of `O(n log n)`.

```csharp
using System;
using System.Collections.Generic;

public class MinHeap
{
    private List<int> heap = new List<int>();
    public int Count => heap.Count;

    public MinHeap() { }

    public MinHeap(int[] arr)
    {
        heap = new List<int>(arr);
        BuildHeap();
    }

    private int Parent(int i) => (i - 1) / 2;
    private int Left(int i) => 2 * i + 1;
    private int Right(int i) => 2 * i + 2;

    private void BuildHeap()
    {
        // Start from the last non-leaf node and heapify-down every internal node.
        for (int i = heap.Count / 2 - 1; i >= 0; i--)
            HeapifyDown(i);
    }

    public void Insert(int value)
    {
        heap.Add(value);
        int i = heap.Count - 1;
        while (i > 0 && heap[Parent(i)] > heap[i])
        {
            (heap[Parent(i)], heap[i]) = (heap[i], heap[Parent(i)]);
            i = Parent(i);
        }
    }

    public int Peek() => heap[0];

    public int ExtractMin()
    {
        int min = heap[0];
        int last = heap.Count - 1;
        heap[0] = heap[last];
        heap.RemoveAt(last);
        if (heap.Count > 0) HeapifyDown(0);
        return min;
    }

    private void HeapifyDown(int i)
    {
        int n = heap.Count;
        while (true)
        {
            int smallest = i, l = Left(i), r = Right(i);
            if (l < n && heap[l] < heap[smallest]) smallest = l;
            if (r < n && heap[r] < heap[smallest]) smallest = r;
            if (smallest == i) break;
            (heap[i], heap[smallest]) = (heap[smallest], heap[i]);
            i = smallest;
        }
    }

    public List<int> ToList() => new List<int>(heap);
}

public class MaxHeap
{
    private List<int> heap = new List<int>();
    public int Count => heap.Count;

    public MaxHeap() { }

    public MaxHeap(int[] arr)
    {
        heap = new List<int>(arr);
        BuildHeap();
    }

    private int Parent(int i) => (i - 1) / 2;
    private int Left(int i) => 2 * i + 1;
    private int Right(int i) => 2 * i + 2;

    private void BuildHeap()
    {
        for (int i = heap.Count / 2 - 1; i >= 0; i--)
            HeapifyDown(i);
    }

    public void Insert(int value)
    {
        heap.Add(value);
        int i = heap.Count - 1;
        while (i > 0 && heap[Parent(i)] < heap[i])
        {
            (heap[Parent(i)], heap[i]) = (heap[i], heap[Parent(i)]);
            i = Parent(i);
        }
    }

    public int Peek() => heap[0];

    public int ExtractMax()
    {
        int max = heap[0];
        int last = heap.Count - 1;
        heap[0] = heap[last];
        heap.RemoveAt(last);
        if (heap.Count > 0) HeapifyDown(0);
        return max;
    }

    private void HeapifyDown(int i)
    {
        int n = heap.Count;
        while (true)
        {
            int largest = i, l = Left(i), r = Right(i);
            if (l < n && heap[l] > heap[largest]) largest = l;
            if (r < n && heap[r] > heap[largest]) largest = r;
            if (largest == i) break;
            (heap[i], heap[largest]) = (heap[largest], heap[i]);
            i = largest;
        }
    }

    public List<int> ToList() => new List<int>(heap);
}
```

Time Complexity: `Insert` and `Extract` are `O(log n)` each. `BuildHeap` from an arbitrary array is **`O(n)`**, not `O(n log n)`. Why: `BuildHeap` calls `HeapifyDown` once per internal node, and the cost of `HeapifyDown` at a node is proportional to the **height of the subtree rooted there**, not to the total number of nodes `n`. In a complete binary tree of `n` nodes, there are roughly `n/2` nodes of height `0` (leaves, skipped entirely), `n/4` nodes of height `1`, `n/8` nodes of height `2`, and so on — in general `n / 2^(h+1)` nodes at height `h`. The total work is the sum `Σ (n / 2^(h+1)) * h` for `h = 0` to `log n`, which is `n * Σ h / 2^(h+1)`. The series `Σ h / 2^(h+1)` converges to a constant (`≈ 1`) as `h → ∞`, so the whole sum is bounded by `O(n) * O(1) = O(n)`. Intuitively: most nodes are near the bottom of the tree and only need to sift down a little, so the naive `n * O(log n)` upper bound is very loose. Space Complexity: `O(n)` to store the heap array (`O(1)` extra if built in-place over the given array as done here).

**Explanation:**
Dry run inserting `5, 2, 8` into an empty `MinHeap`:
- `Insert(5)`: heap = `[5]`.
- `Insert(2)`: heap = `[5, 2]`. Index 1's parent (index 0) = `5 > 2` → swap → `[2, 5]`.
- `Insert(8)`: heap = `[2, 5, 8]`. Index 2's parent (index 0) = `2 < 8` → no swap.

Dry run `ExtractMin()` on `[2, 5, 8]`: save `min = 2`. Move last element `8` to root → `[8, 5]`. Heapify-down at index 0: only child is index 1 = `5`; `5 < 8` → swap → `[5, 8]`. No further children, stop. Returns `2`.

Dry run `BuildHeap` (min-heap) on `arr = [3, 5, 9, 6, 8, 20, 10, 12, 18, 9]` (n = 10, so last non-leaf index = `10/2 - 1 = 4`):
- `i = 4` (value `8`): children at 9 (`9`, out of range at index 10 — only left child index 9 exists = `9`). `8 < 9`, no swap.
- `i = 3` (value `6`): children at 7 (`12`), 8 (`18`). `6` is already smallest, no swap.
- `i = 2` (value `9`): children at 5 (`20`), 6 (`10`). `9` is smallest, no swap.
- `i = 1` (value `5`): children at 3 (`6`), 4 (`8`). `5` is smallest, no swap.
- `i = 0` (value `3`): children at 1 (`5`), 2 (`9`). `3` is smallest, no swap.

Since the input already happens to satisfy min-heap order at every internal node, no swaps occur and the root stays `3` — matching the expected output that the min-heap's root is `3`.

---

## 3. Check if a Given Array Represents a Min-Heap

**Problem Statement:**
Given an array `arr` of `n` integers, determine whether it represents a valid **Min-Heap** when interpreted as a complete binary tree using the standard array indexing (`Left(i) = 2i+1`, `Right(i) = 2i+2`). Return `true` if every parent is less than or equal to both of its children, `false` otherwise.

**Example:**
- Input: `arr = [3, 5, 9, 6, 8, 20, 10, 12, 18, 9]`
- Output: `true`
- Explanation: Checking every internal node (indices `0` to `4`): index `0` (`3`) ≤ children `5, 9`; index `1` (`5`) ≤ children `6, 8`; index `2` (`9`) ≤ children `20, 10`; index `3` (`6`) ≤ children `12, 18`; index `4` (`8`) ≤ child `9`. All parent-child pairs satisfy the min-heap property, so the array is a valid min-heap.

**Approach:**
No actual heapify is required to *check* — simply iterate over every index `i` from `0` up to the last non-leaf node (`n/2 - 1`) and verify `arr[i] <= arr[left]` and `arr[i] <= arr[right]` whenever those children exist. If any parent-child pair violates this, the array is not a valid min-heap. This directly mirrors the condition that `HeapifyDown` would otherwise need to fix.

```csharp
using System;

public class MinHeapValidator
{
    public static bool IsMinHeap(int[] arr)
    {
        int n = arr.Length;
        // Only internal (non-leaf) nodes need checking: indices 0 .. n/2 - 1
        for (int i = 0; i <= n / 2 - 1; i++)
        {
            int left = 2 * i + 1;
            int right = 2 * i + 2;

            if (left < n && arr[i] > arr[left])
                return false;

            if (right < n && arr[i] > arr[right])
                return false;
        }
        return true;
    }

    public static void Main()
    {
        int[] arr = { 3, 5, 9, 6, 8, 20, 10, 12, 18, 9 };
        Console.WriteLine(IsMinHeap(arr)); // true
    }
}
```

Time Complexity: `O(n)` — each internal node (`n/2` of them) is checked in `O(1)` against its up to two children, a single linear pass, no recursion or heapify needed. Space Complexity: `O(1)` extra space (only loop variables), the check is done in-place on the given array.

**Explanation:**
Dry run on `arr = [3, 5, 9, 6, 8, 20, 10, 12, 18, 9]` (`n = 10`, loop `i` from `0` to `4`):
- `i = 0`, value `3`: `left = 1` → `arr[1] = 5`, `3 <= 5` OK. `right = 2` → `arr[2] = 9`, `3 <= 9` OK.
- `i = 1`, value `5`: `left = 3` → `arr[3] = 6`, `5 <= 6` OK. `right = 4` → `arr[4] = 8`, `5 <= 8` OK.
- `i = 2`, value `9`: `left = 5` → `arr[5] = 20`, `9 <= 20` OK. `right = 6` → `arr[6] = 10`, `9 <= 10` OK.
- `i = 3`, value `6`: `left = 7` → `arr[7] = 12`, `6 <= 12` OK. `right = 8` → `arr[8] = 18`, `6 <= 18` OK.
- `i = 4`, value `8`: `left = 9` → `arr[9] = 9`, `8 <= 9` OK. `right = 10` is out of bounds (`n = 10`), skipped.

No violation found across any internal node, so the function returns `true`. If, for example, `arr[4]` had been `15` instead of `8` (parent `15` > child `9` at index 9), the check at `i = 4` would immediately return `false`.

---

## 4. Convert a Min Heap to a Max Heap In-Place

**Problem Statement:**
Given an array that represents a valid **Min-Heap**, convert it **in-place** into a valid **Max-Heap** without using any extra array (only `O(1)` auxiliary space besides the input array itself).

**Example:**
- Input: `arr = [3, 5, 9, 6, 8, 20, 10, 12, 18, 9]` (a valid min-heap, as verified in Problem 3)
- Output: `arr = [20, 18, 10, 12, 9, 9, 3, 6, 5, 8]` (one valid max-heap arrangement; exact array depends on sibling ordering but the max-heap property holds everywhere and root = `20`)
- Explanation: The key insight is that a min-heap array is just an array — once you stop caring about its previous ordering, you can treat it as an arbitrary unsorted array and run the standard **max-heap `BuildHeap`** procedure on it in-place. There is no need to "convert" node by node; simply re-heapify with the opposite comparison.

**Approach:**
Ignore the fact that the array currently satisfies the min-heap property — that ordering is irrelevant once we switch comparison direction. Apply the same bottom-up **build-heap** technique from Problem 2, but using **max-heapify-down** (largest child wins) instead of min-heapify-down, starting from the last non-leaf node (`n/2 - 1`) down to the root (`0`). Because every subtree below the current node has already been fixed into a valid max-heap by the time we process the current node (bottom-up order), a single pass of `n/2` heapify-down calls is sufficient — this is exactly the `O(n)` build-heap proof from Problem 2, now reused for min-to-max conversion.

```csharp
using System;

public class MinHeapToMaxHeap
{
    public static void ConvertMinToMaxHeap(int[] arr)
    {
        int n = arr.Length;
        // Standard bottom-up build-heap, but with max-heapify comparisons.
        for (int i = n / 2 - 1; i >= 0; i--)
            MaxHeapifyDown(arr, n, i);
    }

    private static void MaxHeapifyDown(int[] arr, int n, int i)
    {
        while (true)
        {
            int largest = i;
            int left = 2 * i + 1;
            int right = 2 * i + 2;

            if (left < n && arr[left] > arr[largest]) largest = left;
            if (right < n && arr[right] > arr[largest]) largest = right;

            if (largest == i) break;

            (arr[i], arr[largest]) = (arr[largest], arr[i]);
            i = largest;
        }
    }

    public static void Main()
    {
        int[] arr = { 3, 5, 9, 6, 8, 20, 10, 12, 18, 9 };
        ConvertMinToMaxHeap(arr);
        Console.WriteLine(string.Join(", ", arr));
    }
}
```

Time Complexity: `O(n)` — identical build-heap argument as Problem 2: the sum of subtree heights across all internal nodes of a complete binary tree with `n` nodes is bounded by `O(n)`, since the number of nodes at height `h` shrinks geometrically (`n / 2^(h+1)`) while the per-node work only grows linearly with `h`; the resulting series `Σ h * n / 2^(h+1)` converges to `O(n)`. Space Complexity: `O(1)` extra space — the conversion mutates the input array in-place, only a constant number of index/temp variables are used.

**Explanation:**
Dry run converting `arr = [3, 5, 9, 6, 8, 20, 10, 12, 18, 9]` (`n = 10`) to a max-heap, processing internal nodes from `i = 4` down to `i = 0`, using max-heapify-down:

- `i = 4`, value `8`: left child index `9` = `9` (no right child, index `10` out of bounds). `9 > 8` → swap → `[3, 5, 9, 6, 9, 20, 10, 12, 18, 8]`. Index `9` is a leaf, stop.
- `i = 3`, value `6`: left index `7` = `12`, right index `8` = `18`. Largest child is `18` (index 8). `18 > 6` → swap → `[3, 5, 9, 18, 9, 20, 10, 12, 6, 8]`. Now at index `8`: children would be indices `17, 18` — out of bounds, stop.
- `i = 2`, value `9`: left index `5` = `20`, right index `6` = `10`. Largest child is `20` (index 5). `20 > 9` → swap → `[3, 5, 20, 18, 9, 9, 10, 12, 6, 8]`. Now at index `5`: children indices `11, 12` — out of bounds, stop.
- `i = 1`, value `5`: left index `3` = `18`, right index `4` = `9`. Largest child is `18` (index 3). `18 > 5` → swap → `[3, 18, 20, 5, 9, 9, 10, 12, 6, 8]`. Now at index `3`: left index `7` = `12`, right index `8` = `6`. Largest is `12` (index 7). `12 > 5` → swap → `[3, 18, 20, 12, 9, 9, 10, 5, 6, 8]`. Index `7` is a leaf, stop.
- `i = 0`, value `3`: left index `1` = `18`, right index `2` = `20`. Largest child is `20` (index 2). `20 > 3` → swap → `[20, 18, 3, 12, 9, 9, 10, 5, 6, 8]`. Now at index `2`: left index `5` = `9`, right index `6` = `10`. Largest is `10` (index 6). `10 > 3` → swap → `[20, 18, 10, 12, 9, 9, 3, 5, 6, 8]`. Index `6` is a leaf, stop.

Final array: `[20, 18, 10, 12, 9, 9, 3, 5, 6, 8]`. Verifying the max-heap property holds at every internal node (root `20` ≥ `18, 10`; `18` ≥ `12, 9`; `10` ≥ `9, 3`; `12` ≥ `5, 6`) confirms this is now a valid max-heap, with the overall largest value `20` correctly at the root — the min-heap has been converted to a max-heap entirely in-place using the standard `O(n)` build-heap procedure.
