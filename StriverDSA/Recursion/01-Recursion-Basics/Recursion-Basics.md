# Recursion — Recursion Basics

## Concept: Recursion

Recursion is a technique where a function solves a problem by calling itself on a smaller (or simpler) version of the same problem, until it reaches a version so small that the answer is already known. That "already known" version is called the **base case**; the part where the function calls itself with a reduced input is called the **recursive case**.

Every recursive function needs both pieces:

- **Base Case:** The condition that stops the recursion. Without it, the function would call itself forever and eventually crash with a `StackOverflowException`.
- **Recursive Case:** The part where the function breaks the problem into a smaller subproblem and calls itself, then (optionally) combines the result with the current step's work.

A generic template looks like this:

```csharp
void Solve(int n)
{
    if (n == 0) // base case
    {
        return;
    }

    // do some work here (before the recursive call)

    Solve(n - 1); // recursive case

    // do some work here (after the recursive call)
}
```

### The Call Stack Model

Every time a function calls itself, the runtime does not "replace" the current call — it **pushes a new stack frame** on top of the call stack. Each frame stores that call's own copy of local variables and parameters, and remembers where to resume once the nested call returns. When a call hits its base case and returns, its frame is **popped**, and control resumes in the frame below it, right after the recursive call.

This means recursion always uses **extra memory proportional to the depth of recursion**, even if the function itself doesn't use any arrays or lists — this is called the **recursion stack space**.

Here is a text diagram of the call stack while computing `factorial(3)`:

```
Call phase (stack grows downward, frames pushed on top):

factorial(3)
  -> factorial(2)
       -> factorial(1)
            -> factorial(0)   <-- base case reached, returns 1

Stack at the deepest point (top of stack is on the right):

[ factorial(3) ] [ factorial(2) ] [ factorial(1) ] [ factorial(0) ]
   waiting          waiting          waiting          returns 1

Return phase (frames popped one by one, values combine on the way back):

factorial(0) returns 1
factorial(1) returns 1 * 1               = 1
factorial(2) returns 2 * 1               = 2
factorial(3) returns 3 * 2               = 6
```

Notice that no work happens "at once" — the stack builds up fully first (call phase), then unwinds from the bottom up (return phase), combining partial results as each frame pops. This call-phase/return-phase mental model is the key to tracing any recursive function.

---

## 1. Print Name N Times Using Recursion

**Problem Statement:** Given a name (string) and a count `N`, print the name exactly `N` times using recursion — no loops allowed.

**Example:**
- Input: `name = "Striver"`, `N = 4`
- Output:
```
Striver
Striver
Striver
Striver
```

**Approach:** Use `N` as a counter that decreases by 1 on every call. The base case is when the counter becomes `0` (nothing left to print). In the recursive case, print the name once, then recurse with `N - 1`.

**Logic (Steps):**
1. Base case: if `n == 0`, there is nothing left to print, so return.
2. Otherwise, print `name` once for the current call.
3. Recurse with `n - 1`, shrinking the counter toward the base case.
4. Each call does exactly one print before recursing, so the total number of prints equals the initial `N`.

```csharp
public static void PrintNameNTimes(string name, int n)
{
    // Base case: nothing more to print
    if (n == 0)
    {
        return;
    }

    Console.WriteLine(name);

    // Recursive case: print one fewer time
    PrintNameNTimes(name, n - 1);
}
```
Time Complexity: O(N) — one print operation per recursive call, N calls total.
Space Complexity: O(N) auxiliary space due to the recursion call stack (N frames are pushed before the base case is hit), even though no extra data structure is used.

**Walkthrough:** Trace `PrintNameNTimes("Striver", 4)`:
- `n=4`: prints "Striver", calls `PrintNameNTimes("Striver", 3)`
- `n=3`: prints "Striver", calls `PrintNameNTimes("Striver", 2)`
- `n=2`: prints "Striver", calls `PrintNameNTimes("Striver", 1)`
- `n=1`: prints "Striver", calls `PrintNameNTimes("Striver", 0)`
- `n=0`: base case hit, returns immediately with nothing more printed.

Total output is "Striver" printed 4 times, matching the expected output.

---

**Iterative Approach:** The iterative version does the same work using a simple `for` loop and uses O(1) auxiliary space, since it never grows a call stack.

**Logic (Steps):**
1. Loop `i` from `0` up to (but not including) `n`.
2. On each iteration, print `name` once.
3. When `i` reaches `n`, the loop ends — no call stack is ever built.

```csharp
public static void PrintNameNTimesIterative(string name, int n)
{
    for (int i = 0; i < n; i++)
    {
        Console.WriteLine(name);
    }
}
```
Time Complexity: O(N) — one print per loop iteration, N iterations total.
Space Complexity: O(1) — no recursion stack, just a loop counter.

**Walkthrough:** For `name = "Striver"`, `n = 4`: `i` takes values `0, 1, 2, 3`, printing "Striver" on each iteration — 4 prints total, then the loop exits when `i == 4`. Matches the expected output of "Striver" printed 4 times.

---

## 2. Print 1 to N Using Recursion (without loops)

**Problem Statement:** Given a positive integer `N`, print all integers from `1` to `N` in increasing order using recursion, without using any loop.

**Example:**
- Input: `N = 5`
- Output:
```
1
2
3
4
5
```

**Approach:** To print in increasing order, print **after** the recursive call returns from the smaller subproblem — or equivalently, recurse down first and print on the way "in" before decrementing. The cleanest way: call the function with `N - 1` first (going toward the base case), and print the current value **before** recursing so that smaller numbers are printed first as the calls go deeper.

**Logic (Steps):**
1. Base case: if `n == 0`, there is nothing to print, so return.
2. Otherwise, first recurse with `n - 1`, going all the way down to the base case before printing anything.
3. After the recursive call returns (on the way back up the stack), print the current `n`.
4. Because printing happens on the return phase, the smallest value (from the deepest call) prints first and the original `N` prints last, giving increasing order.

```csharp
public static void PrintOneToN(int n)
{
    // Base case: nothing to print once we go below 1
    if (n == 0)
    {
        return;
    }

    // Recurse first with the smaller value...
    PrintOneToN(n - 1);

    // ...then print on the way back, so 1 prints first, N prints last
    Console.WriteLine(n);
}
```
Time Complexity: O(N) — N recursive calls, one print each.
Space Complexity: O(N) auxiliary space due to the recursion call stack.

**Walkthrough:** Trace `PrintOneToN(5)`:
- Call phase descends: `PrintOneToN(5) -> PrintOneToN(4) -> PrintOneToN(3) -> PrintOneToN(2) -> PrintOneToN(1) -> PrintOneToN(0)` (base case, returns immediately).
- Return phase: `PrintOneToN(1)` resumes and prints `1`, then `PrintOneToN(2)` prints `2`, then `PrintOneToN(3)` prints `3`, then `PrintOneToN(4)` prints `4`, then `PrintOneToN(5)` prints `5`.
- Output produced in order: `1, 2, 3, 4, 5`, matching the expected output.

---

**Iterative Approach:** The iterative equivalent achieves O(1) auxiliary space.

**Logic (Steps):**
1. Loop `i` from `1` up to and including `n`.
2. On each iteration, print `i`.
3. Since the loop counts upward naturally, the values print in increasing order without needing a call stack.

```csharp
public static void PrintOneToNIterative(int n)
{
    for (int i = 1; i <= n; i++)
    {
        Console.WriteLine(i);
    }
}
```
Time Complexity: O(N) — N loop iterations, one print each.
Space Complexity: O(1) — no recursion stack, just a loop counter.

**Walkthrough:** For `n = 5`: `i` takes values `1, 2, 3, 4, 5` in order, printing each as it goes, producing `1, 2, 3, 4, 5` — matching the expected output.

---

## 3. Print N to 1 Using Recursion (without loops)

**Problem Statement:** Given a positive integer `N`, print all integers from `N` down to `1` using recursion, without using any loop.

**Example:**
- Input: `N = 5`
- Output:
```
5
4
3
2
1
```

**Approach:** This is the mirror image of the previous problem — print the current value **before** making the recursive call, so the largest number prints first as the calls go deeper toward the base case.

**Logic (Steps):**
1. Base case: if `n == 0`, stop and return.
2. Otherwise, print the current `n` first (on the way down, before recursing).
3. Recurse with `n - 1`.
4. Because printing happens on the call phase, the largest value (`N`) prints first and the recursion unwinds silently afterward, giving decreasing order.

```csharp
public static void PrintNToOne(int n)
{
    // Base case: stop once we go below 1
    if (n == 0)
    {
        return;
    }

    // Print current value first...
    Console.WriteLine(n);

    // ...then recurse with a smaller value
    PrintNToOne(n - 1);
}
```
Time Complexity: O(N) — N recursive calls, one print each.
Space Complexity: O(N) auxiliary space due to the recursion call stack.

**Walkthrough:** Trace `PrintNToOne(5)`:
- `PrintNToOne(5)` prints `5`, calls `PrintNToOne(4)`
- `PrintNToOne(4)` prints `4`, calls `PrintNToOne(3)`
- `PrintNToOne(3)` prints `3`, calls `PrintNToOne(2)`
- `PrintNToOne(2)` prints `2`, calls `PrintNToOne(1)`
- `PrintNToOne(1)` prints `1`, calls `PrintNToOne(0)`
- `PrintNToOne(0)` is the base case, returns immediately; all frames then unwind with no more work.

Output produced in order: `5, 4, 3, 2, 1`, matching the expected output.

---

**Iterative Approach:** The iterative equivalent uses a simple descending loop and O(1) auxiliary space.

**Logic (Steps):**
1. Loop `i` from `n` down to `1`.
2. On each iteration, print `i`.
3. The loop naturally counts down, so no call stack is needed for the descending order.

```csharp
public static void PrintNToOneIterative(int n)
{
    for (int i = n; i >= 1; i--)
    {
        Console.WriteLine(i);
    }
}
```
Time Complexity: O(N) — N loop iterations, one print each.
Space Complexity: O(1) — no recursion stack, just a loop counter.

**Walkthrough:** For `n = 5`: `i` takes values `5, 4, 3, 2, 1` in order, printing each, producing `5, 4, 3, 2, 1` — matching the expected output.

---

## 4. Sum of First N Numbers Using Recursion

**Problem Statement:** Given a positive integer `N`, find the sum of the first `N` natural numbers (`1 + 2 + ... + N`) using recursion.

**Example:**
- Input: `N = 5`
- Output: `15` (since `1 + 2 + 3 + 4 + 5 = 15`)

**Approach:** Define the recursive relation `Sum(n) = n + Sum(n - 1)`, with base case `Sum(0) = 0`. Each call adds its own value to the result of the sum of everything smaller.

**Logic (Steps):**
1. Base case: if `n == 0`, the sum of the first 0 numbers is `0`, so return `0`.
2. Otherwise, recursively compute `SumOfFirstN(n - 1)` — the sum of everything smaller.
3. When that call returns, add the current `n` to it and return the total.
4. Each frame's returned value flows back up combined with its own `n`, building the final sum on the way back up the stack.

```csharp
public static int SumOfFirstN(int n)
{
    // Base case: sum of first 0 numbers is 0
    if (n == 0)
    {
        return 0;
    }

    // Recursive case: n plus the sum of the first (n - 1) numbers
    return n + SumOfFirstN(n - 1);
}
```
Time Complexity: O(N) — N recursive calls, each doing O(1) work.
Space Complexity: O(N) auxiliary space due to the recursion call stack (N frames must exist simultaneously to compute the additions on the way back up).

**Walkthrough:** Trace `SumOfFirstN(5)`:
- `SumOfFirstN(5) = 5 + SumOfFirstN(4)`
- `SumOfFirstN(4) = 4 + SumOfFirstN(3)`
- `SumOfFirstN(3) = 3 + SumOfFirstN(2)`
- `SumOfFirstN(2) = 2 + SumOfFirstN(1)`
- `SumOfFirstN(1) = 1 + SumOfFirstN(0)`
- `SumOfFirstN(0) = 0` (base case)

Unwinding: `SumOfFirstN(1) = 1 + 0 = 1`, `SumOfFirstN(2) = 2 + 1 = 3`, `SumOfFirstN(3) = 3 + 3 = 6`, `SumOfFirstN(4) = 4 + 6 = 10`, `SumOfFirstN(5) = 5 + 10 = 15`. Final result: `15`, matching the expected output.

---

**Iterative Approach:** The iterative version uses O(1) auxiliary space, and there is even an O(1) time closed-form formula `N * (N + 1) / 2`.

**Logic (Steps):**
1. Initialize `sum = 0`.
2. Loop `i` from `1` to `n`, adding `i` to `sum` on each iteration.
3. Return `sum` after the loop finishes.
4. Alternatively, `SumOfFirstNFormula` skips the loop entirely and computes the closed-form `n * (n + 1) / 2` directly in O(1) time.

```csharp
public static int SumOfFirstNIterative(int n)
{
    int sum = 0;
    for (int i = 1; i <= n; i++)
    {
        sum += i;
    }
    return sum;
}

public static int SumOfFirstNFormula(int n)
{
    return n * (n + 1) / 2;
}
```
Time Complexity: O(N) for the loop version (O(1) for the formula version).
Space Complexity: O(1) — no recursion stack, just a running total (and no extra variables at all for the formula version).

**Walkthrough:** For `n = 5`: the loop accumulates `sum = 1, 3, 6, 10, 15` as `i` goes `1, 2, 3, 4, 5`, returning `15`. The formula version computes `5 * 6 / 2 = 15` directly. Both match the expected output of `15`.

---

## 5. Factorial of N Using Recursion

**Problem Statement:** Given a non-negative integer `N`, compute `N!` (the product of all positive integers from `1` to `N`), using recursion. By definition, `0! = 1`.

**Example:**
- Input: `N = 5`
- Output: `120` (since `5! = 5 * 4 * 3 * 2 * 1 = 120`)

**Approach:** Define the recursive relation `Factorial(n) = n * Factorial(n - 1)`, with base case `Factorial(0) = 1`. This mirrors the call stack diagram shown in the Concept section above.

**Logic (Steps):**
1. Base case: if `n == 0`, return `1` (since `0! = 1`).
2. Otherwise, recursively compute `Factorial(n - 1)`.
3. When that call returns, multiply it by the current `n` and return the product.
4. The multiplications accumulate on the way back up the stack, exactly as shown in the `factorial(3)` call-stack diagram in the Concept section.

```csharp
public static long Factorial(int n)
{
    // Base case: 0! = 1
    if (n == 0)
    {
        return 1;
    }

    // Recursive case: n! = n * (n - 1)!
    return n * Factorial(n - 1);
}
```
Time Complexity: O(N) — N recursive calls, each doing one multiplication.
Space Complexity: O(N) auxiliary space due to the recursion call stack (see the call-stack diagram for `factorial(3)` in the Concept section — N frames exist at the deepest point before unwinding).

**Walkthrough:** Trace `Factorial(5)`:
- `Factorial(5) = 5 * Factorial(4)`
- `Factorial(4) = 4 * Factorial(3)`
- `Factorial(3) = 3 * Factorial(2)`
- `Factorial(2) = 2 * Factorial(1)`
- `Factorial(1) = 1 * Factorial(0)`
- `Factorial(0) = 1` (base case)

Unwinding: `Factorial(1) = 1*1 = 1`, `Factorial(2) = 2*1 = 2`, `Factorial(3) = 3*2 = 6`, `Factorial(4) = 4*6 = 24`, `Factorial(5) = 5*24 = 120`. Final result: `120`, matching the expected output.

---

**Iterative Approach:** The iterative version computes the same product in a loop using O(1) auxiliary space.

**Logic (Steps):**
1. Initialize `result = 1`.
2. Loop `i` from `2` to `n`, multiplying `result` by `i` on each iteration.
3. Return `result` after the loop finishes.

```csharp
public static long FactorialIterative(int n)
{
    long result = 1;
    for (int i = 2; i <= n; i++)
    {
        result *= i;
    }
    return result;
}
```
Time Complexity: O(N) — N loop iterations, one multiplication each.
Space Complexity: O(1) — no recursion stack, just a running product.

**Walkthrough:** For `n = 5`: `result` accumulates `1 -> 2 -> 6 -> 24 -> 120` as `i` goes `2, 3, 4, 5`, returning `120` — matching the expected output.

---

## 6. Reverse an Array Using Recursion (two-pointer recursive)

**Problem Statement:** Given an array of integers, reverse it in place using recursion, using a two-pointer technique (a `left` index and a `right` index) instead of a loop.

**Example:**
- Input: `arr = [1, 2, 3, 4, 5]`
- Output: `arr = [5, 4, 3, 2, 1]`

**Approach:** Maintain two pointers, `left` starting at index `0` and `right` starting at index `arr.Length - 1`. The base case is when `left >= right` — the pointers have met or crossed, meaning the whole array is reversed. Otherwise, swap the elements at `left` and `right`, then recurse with `left + 1` and `right - 1`.

**Logic (Steps):**
1. Base case: if `left >= right`, the pointers have met or crossed, so the array is fully reversed — return.
2. Otherwise, swap `arr[left]` and `arr[right]`.
3. Recurse with `left + 1` and `right - 1`, moving both pointers one step inward.
4. Because the swap happens before the recursive call, all the actual work finishes during the call phase; the return phase just pops frames with nothing left to do.

```csharp
public static void ReverseArray(int[] arr, int left, int right)
{
    // Base case: pointers met or crossed, nothing more to swap
    if (left >= right)
    {
        return;
    }

    // Swap the two ends
    int temp = arr[left];
    arr[left] = arr[right];
    arr[right] = temp;

    // Recurse inward
    ReverseArray(arr, left + 1, right - 1);
}

// Convenience overload to start the recursion
public static void ReverseArray(int[] arr)
{
    ReverseArray(arr, 0, arr.Length - 1);
}
```

Time Complexity: O(N) — the pointers close the gap by 2 each call, giving roughly N/2 calls, each doing O(1) work.
Space Complexity: O(N) auxiliary space due to the recursion call stack (up to N/2 frames on the stack at the deepest point), even though the swapping itself is done in place with no extra array.

**Walkthrough:** Trace `ReverseArray([1, 2, 3, 4, 5], 0, 4)`:
- `ReverseArray(arr, 0, 4)`: `left=0, right=4` -> not base case, swap `arr[0]` and `arr[4]` -> `arr` becomes `[5, 2, 3, 4, 1]`, recurse into `(1, 3)`.
- `ReverseArray(arr, 1, 3)`: `left=1, right=3` -> not base case, swap `arr[1]` and `arr[3]` -> `arr` becomes `[5, 4, 3, 2, 1]`, recurse into `(2, 2)`.
- `ReverseArray(arr, 2, 2)`: `left=2, right=2` -> base case (`left >= right`), returns immediately.
- Return phase: all three frames pop with nothing more to do, since every swap already happened before recursing.

Final array: `[5, 4, 3, 2, 1]`, matching the expected output. It still costs O(N) stack space while the calls are in flight, even though the return phase does no extra work.

---

**Iterative Approach:** The iterative two-pointer version does identical swaps in a `while` loop with O(1) auxiliary space.

**Logic (Steps):**
1. Initialize `left = 0` and `right = arr.Length - 1`.
2. While `left < right`, swap `arr[left]` and `arr[right]`.
3. Increment `left` and decrement `right` after each swap.
4. Stop when the pointers meet or cross.

```csharp
public static void ReverseArrayIterative(int[] arr)
{
    int left = 0;
    int right = arr.Length - 1;

    while (left < right)
    {
        int temp = arr[left];
        arr[left] = arr[right];
        arr[right] = temp;
        left++;
        right--;
    }
}
```
Time Complexity: O(N) — the loop runs roughly N/2 iterations, each doing O(1) work.
Space Complexity: O(1) — no recursion stack, just the `left`/`right` pointers and a temp variable.

**Walkthrough:** For `arr = [1, 2, 3, 4, 5]`: swap `arr[0]` and `arr[4]` -> `[5, 2, 3, 4, 1]` (`left=1, right=3`); swap `arr[1]` and `arr[3]` -> `[5, 4, 3, 2, 1]` (`left=2, right=2`); loop stops since `left < right` is false. Final array: `[5, 4, 3, 2, 1]`, matching the expected output.

---

## 7. Check if a String is a Palindrome Using Recursion

**Problem Statement:** Given a string, determine whether it reads the same forwards and backwards (a palindrome), using recursion.

**Example:**
- Input: `s = "madam"`
- Output: `true`

- Input: `s = "hello"`
- Output: `false`

**Approach:** Use two pointers, `left` starting at `0` and `right` starting at `s.Length - 1`. The base case is when `left >= right` — the middle has been reached (or crossed) without finding a mismatch, so the string is a palindrome, return `true`. If `s[left] != s[right]` at any point, it is not a palindrome, return `false` immediately. Otherwise, recurse inward with `left + 1` and `right - 1`.

**Logic (Steps):**
1. Base case: if `left >= right`, the pointers met or crossed with no mismatch found, so return `true`.
2. If `s[left] != s[right]`, a mismatch is found, so return `false` immediately (short-circuit, no further recursion).
3. Otherwise the characters match, so recurse inward with `left + 1` and `right - 1`.
4. The boolean result of the recursive call is returned as-is, propagating `true`/`false` back up the stack unchanged.

```csharp
public static bool IsPalindrome(string s, int left, int right)
{
    // Base case: pointers met or crossed, no mismatch found so far
    if (left >= right)
    {
        return true;
    }

    // Mismatch found: not a palindrome
    if (s[left] != s[right])
    {
        return false;
    }

    // Characters match, recurse inward
    return IsPalindrome(s, left + 1, right - 1);
}

// Convenience overload to start the recursion
public static bool IsPalindrome(string s)
{
    return IsPalindrome(s, 0, s.Length - 1);
}
```

Time Complexity: O(N) — pointers close the gap by 2 each call, giving roughly N/2 calls in the worst case, each doing O(1) work.
Space Complexity: O(N) auxiliary space due to the recursion call stack (up to N/2 frames at the deepest point), even though no extra data structure is used to store characters.

**Walkthrough:** Trace `IsPalindrome("madam", 0, 4)`:
- `IsPalindrome("madam", 0, 4)`: `left=0('m')`, `right=4('m')` -> match, recurse into `(1, 3)`.
- `IsPalindrome("madam", 1, 3)`: `left=1('a')`, `right=3('a')` -> match, recurse into `(2, 2)`.
- `IsPalindrome("madam", 2, 2)`: `left=2, right=2` -> base case (`left >= right`), returns `true`.
- Return phase (values bubble back unchanged since no mismatch occurred): `(1,3)` returns `true`, then `(0,4)` returns `true`.

Final result: `true`, matching the expected output.

Now trace the mismatching case `IsPalindrome("hello", 0, 4)`: `left=0('h')`, `right=4('o')` -> mismatch found immediately, so the function returns `false` right away with no further recursive calls made. This shows the short-circuit behavior — the call stack for a non-palindrome can be much shallower than N/2 frames. Final result: `false`, matching the expected output.

---

**Iterative Approach:** The iterative two-pointer version performs the same comparisons in a `while` loop with O(1) auxiliary space.

**Logic (Steps):**
1. Initialize `left = 0` and `right = s.Length - 1`.
2. While `left < right`, compare `s[left]` and `s[right]`; if they differ, return `false` immediately.
3. Otherwise increment `left` and decrement `right`.
4. If the loop finishes without a mismatch, return `true`.

```csharp
public static bool IsPalindromeIterative(string s)
{
    int left = 0;
    int right = s.Length - 1;

    while (left < right)
    {
        if (s[left] != s[right])
        {
            return false;
        }
        left++;
        right--;
    }

    return true;
}
```
Time Complexity: O(N) — the loop runs up to roughly N/2 iterations, each doing O(1) work.
Space Complexity: O(1) — no recursion stack, just the `left`/`right` pointers.

**Walkthrough:** For `s = "madam"`: compare `s[0]='m'` vs `s[4]='m'` (match), then `s[1]='a'` vs `s[3]='a'` (match); loop stops when `left=2, right=2`; returns `true`. For `s = "hello"`: compare `s[0]='h'` vs `s[4]='o'` -> mismatch, returns `false` immediately. Both match the expected outputs.
