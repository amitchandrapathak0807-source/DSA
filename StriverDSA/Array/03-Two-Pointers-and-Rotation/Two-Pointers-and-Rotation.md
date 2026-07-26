# Array — Two Pointers and Rotation

## 1. Left Rotate an Array by One Place

### 1. Left Rotate an Array by One Place

**Problem Statement:** Given an array `arr` of size `n`, left rotate it by one position. This means every element shifts one index to the left, and the first element wraps around to become the last element.

**Example:**
- Input: `arr = [1, 2, 3, 4, 5]`
- Output: `[2, 3, 4, 5, 1]`
- Explanation: `1` (the first element) moves to the end, and every other element shifts one position left.

**Brute Force Approach:** Store the first element in a temporary variable, shift every remaining element one position to the left, then place the temporary value at the last index. This only needs a single extra variable (not a full temp array), so it is effectively already optimal, but conceptually it is the straightforward "shift and wrap" approach.

```csharp
public static int[] LeftRotateByOne(int[] arr)
{
    int n = arr.Length;
    if (n <= 1) return arr;

    int first = arr[0]; // store the element that will wrap around

    for (int i = 0; i < n - 1; i++)
    {
        arr[i] = arr[i + 1]; // shift each element one step left
    }

    arr[n - 1] = first; // place the stored first element at the end

    return arr;
}
```

**Time Complexity:** O(n) — we make a single pass through the array to shift every element once.
**Space Complexity:** O(1) — only one extra integer variable (`first`) is used, no extra array.

**Optimized Approach:** This brute force solution is already optimal for rotating by one place — there is no better way than touching each element once. The same code is repeated below as the "optimized" version since no further improvement is possible for this specific sub-problem.

```csharp
public static void LeftRotateByOneInPlace(int[] arr)
{
    int n = arr.Length;
    if (n <= 1) return;

    int temp = arr[0];
    for (int i = 0; i < n - 1; i++)
    {
        arr[i] = arr[i + 1];
    }
    arr[n - 1] = temp;
}
```

Time Complexity: O(n), Space Complexity: O(1)

**Explanation:** Dry run on `arr = [1, 2, 3, 4, 5]`:
1. `temp = arr[0] = 1`
2. `arr[0] = arr[1] = 2` → `[2, 2, 3, 4, 5]`
3. `arr[1] = arr[2] = 3` → `[2, 3, 3, 4, 5]`
4. `arr[2] = arr[3] = 4` → `[2, 3, 4, 4, 5]`
5. `arr[3] = arr[4] = 5` → `[2, 3, 4, 5, 5]`
6. `arr[4] = temp = 1` → `[2, 3, 4, 5, 1]`

Final array: `[2, 3, 4, 5, 1]`, which matches the expected output.

---

## 2. Left Rotate an Array by D Places

### 2. Left Rotate an Array by D Places

**Problem Statement:** Given an array `arr` of size `n` and an integer `d`, left rotate the array by `d` positions. Each element moves `d` steps to the left, and the elements that "fall off" the front wrap around to the end, preserving relative order. Handle the case where `d > n` by taking `d % n`, since rotating by `n` places brings the array back to its original state.

**Example:**
- Input: `arr = [1, 2, 3, 4, 5], d = 2`
- Output: `[3, 4, 5, 1, 2]`
- Explanation: The first two elements `[1, 2]` are moved to the end, while `[3, 4, 5]` shift to the front, preserving order.

**Brute Force Approach:** Use a temporary array to hold the first `d` elements, shift the remaining `n - d` elements to the front of the original array, and then copy the temporary elements to the end. This uses O(d) extra space.

```csharp
public static int[] LeftRotateByD_Brute(int[] arr, int d)
{
    int n = arr.Length;
    if (n == 0) return arr;

    d = d % n; // handle d > n
    if (d == 0) return arr;

    int[] temp = new int[d];

    // store first d elements in temp
    for (int i = 0; i < d; i++)
    {
        temp[i] = arr[i];
    }

    // shift remaining n - d elements to the front
    for (int i = d; i < n; i++)
    {
        arr[i - d] = arr[i];
    }

    // place the stored elements at the end
    for (int i = 0; i < d; i++)
    {
        arr[n - d + i] = temp[i];
    }

    return arr;
}
```

Time Complexity: O(n) — one pass to copy `d` elements into `temp`, one pass to shift `n - d` elements, and one pass to copy `d` elements back; all linear in `n`.
Space Complexity: O(d) — extra temporary array of size `d` is used.

**Optimized Approach:** Use the **reversal algorithm** to rotate in-place with O(1) extra space:
1. Reverse the first `d` elements.
2. Reverse the remaining `n - d` elements.
3. Reverse the whole array.

This works because reversing sub-segments and then the whole array effectively re-orders the blocks without needing extra storage.

```csharp
public static void LeftRotateByD_Optimal(int[] arr, int d)
{
    int n = arr.Length;
    if (n == 0) return;

    d = d % n; // handle d > n (and d == 0 case naturally becomes a no-op)
    if (d == 0) return;

    Reverse(arr, 0, d - 1);       // reverse first d elements
    Reverse(arr, d, n - 1);       // reverse remaining n - d elements
    Reverse(arr, 0, n - 1);       // reverse the whole array
}

private static void Reverse(int[] arr, int start, int end)
{
    while (start < end)
    {
        int temp = arr[start];
        arr[start] = arr[end];
        arr[end] = temp;
        start++;
        end--;
    }
}
```

Time Complexity: O(n), Space Complexity: O(1)

**Explanation:** The reversal trick relies on a neat property: if you reverse two adjacent blocks individually and then reverse the entire combined sequence, the blocks end up swapped while each block's internal order is restored to the original.

Dry run on `arr = [1, 2, 3, 4, 5], d = 2`:

1. **Reverse first `d = 2` elements** (indices 0 to 1): `[1, 2]` → `[2, 1]`
   Array becomes: `[2, 1, 3, 4, 5]`

2. **Reverse remaining `n - d = 3` elements** (indices 2 to 4): `[3, 4, 5]` → `[5, 4, 3]`
   Array becomes: `[2, 1, 5, 4, 3]`

3. **Reverse the whole array** (indices 0 to 4): `[2, 1, 5, 4, 3]` → `[3, 4, 5, 1, 2]`
   Array becomes: `[3, 4, 5, 1, 2]`

Final array: `[3, 4, 5, 1, 2]`, which matches the expected output. The `d = d % n` step ensures correctness even if `d` is larger than `n` — e.g., rotating an array of size 5 by `d = 7` is equivalent to rotating by `7 % 5 = 2`.

---

## 3. Move Zeros to the End

### 3. Move Zeros to the End

**Problem Statement:** Given an array `arr`, move all the `0`s to the end of the array while maintaining the relative order of the non-zero elements. The operation must be done in-place.

**Example:**
- Input: `arr = [0, 1, 0, 3, 12]`
- Output: `[1, 3, 12, 0, 0]`
- Explanation: The non-zero elements `1, 3, 12` retain their relative order and are moved to the front, while both `0`s are pushed to the end.

**Brute Force Approach:** Create a temporary array/list, copy all non-zero elements into it in order, then fill the rest of the temporary array with zeros, and finally copy it back into the original array. This uses O(n) extra space.

```csharp
public static void MoveZeros_Brute(int[] arr)
{
    int n = arr.Length;
    List<int> temp = new List<int>();

    // collect all non-zero elements in order
    for (int i = 0; i < n; i++)
    {
        if (arr[i] != 0)
        {
            temp.Add(arr[i]);
        }
    }

    // fill the front of arr with non-zero elements
    int index = 0;
    for (; index < temp.Count; index++)
    {
        arr[index] = temp[index];
    }

    // fill the rest with zeros
    for (; index < n; index++)
    {
        arr[index] = 0;
    }
}
```

Time Complexity: O(n) — one pass to collect non-zero elements, one pass to copy them back, one pass to fill zeros; all linear.
Space Complexity: O(n) — extra list to store non-zero elements.

**Optimized Approach:** Use the **two-pointer (read/write) technique** to solve this in-place with O(1) extra space. Maintain a `writeIndex` (or "insert position") that tracks where the next non-zero element should go, and a `readIndex` that scans through the array. Whenever a non-zero element is found at `readIndex`, swap it with `arr[writeIndex]` and advance `writeIndex`.

```csharp
public static void MoveZeros_Optimal(int[] arr)
{
    int n = arr.Length;
    int writeIndex = 0; // position where the next non-zero element should be placed

    // move all non-zero elements to the front, preserving order
    for (int readIndex = 0; readIndex < n; readIndex++)
    {
        if (arr[readIndex] != 0)
        {
            int temp = arr[writeIndex];
            arr[writeIndex] = arr[readIndex];
            arr[readIndex] = temp;
            writeIndex++;
        }
    }
}
```

Time Complexity: O(n), Space Complexity: O(1)

**Explanation:** The invariant maintained is: **at all times, every element in `arr[0..writeIndex-1]` is non-zero**, and `writeIndex` marks the next free slot for a non-zero element. `readIndex` scans ahead looking for non-zero values; when found, it is swapped into the `writeIndex` slot. Since `writeIndex` never advances past `readIndex`, the swap either places the non-zero element directly (when `writeIndex == readIndex`, a no-op swap) or swaps it with a zero that was left behind earlier (when `writeIndex < readIndex`), pushing that zero further to the right.

Dry run on `arr = [0, 1, 0, 3, 12]`, `writeIndex = 0`:

1. `readIndex = 0`, `arr[0] = 0` → skip (zero, no swap). Array: `[0, 1, 0, 3, 12]`, `writeIndex = 0`
2. `readIndex = 1`, `arr[1] = 1` → non-zero, swap `arr[0]` and `arr[1]`: `[1, 0, 0, 3, 12]`, `writeIndex = 1`
3. `readIndex = 2`, `arr[2] = 0` → skip. Array: `[1, 0, 0, 3, 12]`, `writeIndex = 1`
4. `readIndex = 3`, `arr[3] = 3` → non-zero, swap `arr[1]` and `arr[3]`: `[1, 3, 0, 0, 12]`, `writeIndex = 2`
5. `readIndex = 4`, `arr[4] = 12` → non-zero, swap `arr[2]` and `arr[4]`: `[1, 3, 12, 0, 0]`, `writeIndex = 3`

Final array: `[1, 3, 12, 0, 0]`, which matches the expected output. All non-zero elements retain their original relative order (`1, 3, 12`), and both zeros are pushed to the end.
