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

**How to Improve:** The iterative version does the same work using a simple `for` loop and uses O(1) auxiliary space, since it never grows a call stack:

```csharp
public static void PrintNameNTimesIterative(string name, int n)
{
    for (int i = 0; i < n; i++)
    {
        Console.WriteLine(name);
    }
}
```

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

**How to Improve:** The iterative equivalent achieves O(1) auxiliary space:

```csharp
public static void PrintOneToNIterative(int n)
{
    for (int i = 1; i <= n; i++)
    {
        Console.WriteLine(i);
    }
}
```

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

**How to Improve:** The iterative equivalent uses a simple descending loop and O(1) auxiliary space:

```csharp
public static void PrintNToOneIterative(int n)
{
    for (int i = n; i >= 1; i--)
    {
        Console.WriteLine(i);
    }
}
```

---

## 4. Sum of First N Numbers Using Recursion

**Problem Statement:** Given a positive integer `N`, find the sum of the first `N` natural numbers (`1 + 2 + ... + N`) using recursion.

**Example:**
- Input: `N = 5`
- Output: `15` (since `1 + 2 + 3 + 4 + 5 = 15`)

**Approach:** Define the recursive relation `Sum(n) = n + Sum(n - 1)`, with base case `Sum(0) = 0`. Each call adds its own value to the result of the sum of everything smaller.

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

**How to Improve:** The iterative version uses O(1) auxiliary space, and there is even an O(1) time closed-form formula `N * (N + 1) / 2`:

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

---

## 5. Factorial of N Using Recursion

**Problem Statement:** Given a non-negative integer `N`, compute `N!` (the product of all positive integers from `1` to `N`), using recursion. By definition, `0! = 1`.

**Example:**
- Input: `N = 5`
- Output: `120` (since `5! = 5 * 4 * 3 * 2 * 1 = 120`)

**Approach:** Define the recursive relation `Factorial(n) = n * Factorial(n - 1)`, with base case `Factorial(0) = 1`. This mirrors the call stack diagram shown in the Concept section above.

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

**How to Improve:** The iterative version computes the same product in a loop using O(1) auxiliary space:

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

---

## 6. Reverse an Array Using Recursion (two-pointer recursive)

**Problem Statement:** Given an array of integers, reverse it in place using recursion, using a two-pointer technique (a `left` index and a `right` index) instead of a loop.

**Example:**
- Input: `arr = [1, 2, 3, 4, 5]`
- Output: `arr = [5, 4, 3, 2, 1]`

**Approach:** Maintain two pointers, `left` starting at index `0` and `right` starting at index `arr.Length - 1`. The base case is when `left >= right` — the pointers have met or crossed, meaning the whole array is reversed. Otherwise, swap the elements at `left` and `right`, then recurse with `left + 1` and `right - 1`.

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

**Explanation:** Trace `ReverseArray([1, 2, 3, 4, 5], 0, 4)`:

```
Call phase:
ReverseArray(arr, 0, 4): left=0, right=4 -> not base case, swap arr[0] and arr[4]
   arr becomes [5, 2, 3, 4, 1]
   -> ReverseArray(arr, 1, 3): left=1, right=3 -> not base case, swap arr[1] and arr[3]
        arr becomes [5, 4, 3, 2, 1]
        -> ReverseArray(arr, 2, 2): left=2, right=2 -> base case (left >= right), return

Stack at deepest point (top of stack is on the right):
[ ReverseArray(0,4) ] [ ReverseArray(1,3) ] [ ReverseArray(2,2) ]
     waiting               waiting            returns immediately

Return phase:
ReverseArray(2,2) returns (did nothing)
ReverseArray(1,3) returns (already swapped before recursing)
ReverseArray(0,4) returns (already swapped before recursing)

Final array: [5, 4, 3, 2, 1]
```

Since the swap happens **before** the recursive call, all the actual work is done during the call phase; the return phase here is just frames popping with nothing left to do — but it still costs O(N) stack space while the calls are in flight.

**How to Improve:** The iterative two-pointer version does identical swaps in a `while` loop with O(1) auxiliary space:

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

---

## 7. Check if a String is a Palindrome Using Recursion

**Problem Statement:** Given a string, determine whether it reads the same forwards and backwards (a palindrome), using recursion.

**Example:**
- Input: `s = "madam"`
- Output: `true`

- Input: `s = "hello"`
- Output: `false`

**Approach:** Use two pointers, `left` starting at `0` and `right` starting at `s.Length - 1`. The base case is when `left >= right` — the middle has been reached (or crossed) without finding a mismatch, so the string is a palindrome, return `true`. If `s[left] != s[right]` at any point, it is not a palindrome, return `false` immediately. Otherwise, recurse inward with `left + 1` and `right - 1`.

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

**Explanation:** Trace `IsPalindrome("madam", 0, 4)`:

```
Call phase:
IsPalindrome("madam", 0, 4): left=0('m'), right=4('m') -> match, recurse
   -> IsPalindrome("madam", 1, 3): left=1('a'), right=3('a') -> match, recurse
        -> IsPalindrome("madam", 2, 2): left=2, right=2 -> base case (left >= right)
             returns true

Stack at deepest point (top of stack is on the right):
[ IsPalindrome(0,4) ] [ IsPalindrome(1,3) ] [ IsPalindrome(2,2) ]
    waiting on            waiting on            returns true
   recursive call        recursive call

Return phase (values bubble back up unchanged since no mismatch occurred):
IsPalindrome(2,2) returns true
IsPalindrome(1,3) returns true  (because the recursive call returned true)
IsPalindrome(0,4) returns true  (because the recursive call returned true)

Final result: true
```

Now trace the mismatching case `IsPalindrome("hello", 0, 4)`:

```
Call phase:
IsPalindrome("hello", 0, 4): left=0('h'), right=4('o') -> mismatch!
   returns false immediately, no further recursive calls are made

Final result: false
```

This shows the short-circuit behavior — as soon as a mismatch is detected, the function returns `false` directly without descending any further into the recursion, so the call stack for a non-palindrome can be much shallower than N/2 frames.

**How to Improve:** The iterative two-pointer version performs the same comparisons in a `while` loop with O(1) auxiliary space:

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
