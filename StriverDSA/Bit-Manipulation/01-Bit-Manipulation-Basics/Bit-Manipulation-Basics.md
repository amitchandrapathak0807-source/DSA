# Bit Manipulation — Basics

## Concept: Bitwise Operators in C#

Bitwise operators work directly on the binary representation of integers. Understanding them is the foundation for every bit manipulation trick.

- **`&` (AND)** — Sets a result bit to `1` only if both operand bits are `1`.
  Example: `5 & 3` → `101 & 011 = 001` → `1`
- **`|` (OR)** — Sets a result bit to `1` if at least one operand bit is `1`.
  Example: `5 | 3` → `101 | 011 = 111` → `7`
- **`^` (XOR)** — Sets a result bit to `1` if the operand bits differ.
  Example: `5 ^ 3` → `101 ^ 011 = 110` → `6`
- **`~` (NOT / Complement)** — Flips every bit (0 becomes 1, 1 becomes 0).
  Example: `~5` → `~00000101 = 11111010` (in two's complement, this equals `-6`)
- **`<<` (Left Shift)** — Shifts bits left, filling with zeros; equivalent to multiplying by `2^k`.
  Example: `5 << 1` → `101 << 1 = 1010` → `10`
- **`>>` (Right Shift)** — Shifts bits right, discarding low bits; equivalent to integer-dividing by `2^k`.
  Example: `5 >> 1` → `101 >> 1 = 010` → `2`

---

## 1. Check if the i-th Bit is Set

**Problem Statement:** Given an integer `n` and an index `i` (0-based from the right, i.e., LSB), determine whether the bit at position `i` in the binary representation of `n` is set (`1`) or not (`0`).

**Example:**
- Input: `n = 13 (1101), i = 2`
- Output: `true` (bit is set)
- Explanation: `13` in binary is `1101`. Counting from the right starting at index `0`: index `0 = 1`, index `1 = 0`, index `2 = 1`, index `3 = 1`. The bit at index `2` is `1`, so it is set.

**Approach:** Left-shift `1` by `i` positions to create a mask that has only the `i`-th bit set (e.g., `1 << 2 = 0100`). Then AND (`&`) this mask with `n`. If the result is non-zero, the `i`-th bit of `n` was `1`; otherwise it was `0`.

```csharp
public class BitBasics
{
    public static bool IsBitSet(int n, int i)
    {
        // Mask has only the i-th bit set: 1 << i
        int mask = 1 << i;

        // AND with n: non-zero result means the bit was set
        return (n & mask) != 0;
    }
}
```

Time Complexity: O(1) — a fixed number of bitwise operations.
Space Complexity: O(1) — no extra space used.

**Explanation:**
```
n = 13 = 1101
i = 2

mask = 1 << 2 = 0100

  1101   (n)
& 0100   (mask)
------
  0100   (non-zero => bit is set)
```
Since `n & mask != 0`, the 2nd bit of `13` is confirmed set.

---

## 2. Check if a Number is Odd or Even Using Bit Manipulation

**Problem Statement:** Given an integer `n`, determine whether it is odd or even without using the modulo (`%`) operator.

**Example:**
- Input: `n = 7 (0111)`
- Output: `Odd`
- Explanation: `7` in binary is `0111`. The last bit (LSB, index `0`) is `1`, which means the number is odd. If the LSB were `0`, the number would be even.

**Approach:** In binary, the least significant bit (LSB) determines parity — it is `1` for odd numbers and `0` for even numbers, because every bit position except index `0` represents an even contribution (`2, 4, 8, ...`). AND `n` with `1` (mask `...0001`) to isolate the LSB. If the result is `1`, `n` is odd; if `0`, `n` is even.

```csharp
public class BitBasics
{
    public static bool IsOdd(int n)
    {
        // LSB set => odd, LSB clear => even
        return (n & 1) == 1;
    }
}
```

Time Complexity: O(1).
Space Complexity: O(1).

**Explanation:**
```
n = 7 = 0111
mask   = 0001

  0111
& 0001
------
  0001   => result is 1, so n is Odd

n = 8 = 1000
mask   = 0001

  1000
& 0001
------
  0000   => result is 0, so n is Even
```

---

## 3. Check if a Number is a Power of Two Using Bit Manipulation

**Problem Statement:** Given a positive integer `n`, determine whether it is a power of two (i.e., `n = 2^k` for some non-negative integer `k`).

**Example:**
- Input: `n = 16 (10000)`
- Output: `true`
- Explanation: `16` in binary is `10000`, which has exactly one set bit. Numbers that are powers of two always have exactly one set bit in their binary representation.

**Approach:** A power of two always has exactly one set bit (e.g., `1 = 0001`, `2 = 0010`, `4 = 0100`, `8 = 1000`). The expression `n & (n - 1)` clears the lowest set bit of `n`. If `n` has only one set bit, `n & (n - 1)` becomes `0`. So `n` is a power of two if `n > 0` and `(n & (n - 1)) == 0`.

```csharp
public class BitBasics
{
    public static bool IsPowerOfTwo(int n)
    {
        // Must be positive, and clearing the lowest set bit must yield 0
        return n > 0 && (n & (n - 1)) == 0;
    }
}
```

Time Complexity: O(1).
Space Complexity: O(1).

**Explanation:**
```
n = 16 = 10000
n - 1  = 01111

  10000
& 01111
-------
  00000   => result is 0, so 16 IS a power of two

n = 12 = 01100
n - 1  = 01011

  01100
& 01011
-------
  01000   => result is non-zero (8), so 12 is NOT a power of two
```
The trick `n & (n-1)` works because subtracting `1` flips the lowest set bit to `0` and turns every bit to its right (all zeros) into `1`s. ANDing with the original number then clears exactly that lowest set bit — if that was the only set bit, the result is `0`.

---

## 4. Count the Number of Set Bits in an Integer

**Problem Statement:** Given an integer `n`, count the number of `1`s (set bits) in its binary representation. This is also known as computing the Hamming Weight, and the technique used is Brian Kernighan's Algorithm.

**Example:**
- Input: `n = 11 (1011)`
- Output: `3`
- Explanation: `11` in binary is `1011`, which has three `1` bits (at positions `0`, `1`, and `3`).

**Approach:** Repeatedly apply `n = n & (n - 1)`, which clears the lowest set bit on each iteration, and count how many iterations it takes for `n` to become `0`. This is Brian Kernighan's Algorithm — it runs once per set bit rather than once per total bit, making it efficient when the number of set bits is small.

```csharp
public class BitBasics
{
    public static int CountSetBits(int n)
    {
        int count = 0;

        while (n != 0)
        {
            n = n & (n - 1); // clears the lowest set bit
            count++;
        }

        return count;
    }
}
```

Time Complexity: O(k), where k is the number of set bits in `n` (worst case O(log n)).
Space Complexity: O(1).

**Explanation:**
```
n = 11 = 1011

Step 1: n & (n-1)
  1011
& 1010
------
  1010   (cleared the lowest set bit; count = 1)

Step 2: n & (n-1)
  1010
& 1001
------
  1000   (cleared next lowest set bit; count = 2)

Step 3: n & (n-1)
  1000
& 0111
------
  0000   (cleared last set bit; count = 3)

n is now 0 => loop stops, total set bits = 3
```

---

## 5. Swap Two Numbers Without Using a Temporary Variable (XOR trick)

**Problem Statement:** Given two integers `a` and `b`, swap their values without using any extra temporary/auxiliary variable.

**Example:**
- Input: `a = 5, b = 9`
- Output: `a = 9, b = 5`
- Explanation: Using successive XOR operations, the values of `a` and `b` are exchanged in place using only bitwise operations.

**Approach:** XOR has the property that `x ^ x = 0` and `x ^ 0 = x`, and XOR is its own inverse. By performing `a = a ^ b`, `b = a ^ b`, `a = a ^ b` in sequence, the original values effectively swap places without needing a third variable.

```csharp
public class BitBasics
{
    public static void SwapUsingXor(ref int a, ref int b)
    {
        a = a ^ b; // a now holds (original a) XOR (original b)
        b = a ^ b; // b becomes original a
        a = a ^ b; // a becomes original b
    }
}
```

Time Complexity: O(1).
Space Complexity: O(1) — no temporary variable used.

**Explanation:**
```
a = 5 = 0101
b = 9 = 1001

Step 1: a = a ^ b
  0101
^ 1001
------
  1100   => a = 12 (0101 ^ 1001), b unchanged = 1001 (9)

Step 2: b = a ^ b
  1100
^ 1001
------
  0101   => b = 5 (this is the original a!)

Step 3: a = a ^ b
  1100
^ 0101
------
  1001   => a = 9 (this is the original b!)

Final: a = 9, b = 5  -> values successfully swapped
```

---

## 6. Minimum Number of Bit Flips to Convert Number A to Number B

**Problem Statement:** Given two integers `a` and `b`, find the minimum number of bit flips required to convert `a` into `b`. A bit flip changes a `0` to `1` or a `1` to `0` at a single position.

**Example:**
- Input: `a = 10 (1010), b = 7 (0111)`
- Output: `3`
- Explanation: XORing `a` and `b` gives `1010 ^ 0111 = 1101`, which has three set bits. Each set bit in the XOR result marks a position where `a` and `b` differ, and each such differing position requires exactly one flip.

**Approach:** XOR `a` and `b`. Every bit position where `a` and `b` differ produces a `1` in `a ^ b`, and every position where they match produces a `0`. Therefore, the number of set bits in `a ^ b` equals the minimum number of flips needed. Count the set bits using Brian Kernighan's Algorithm (`n & (n - 1)`), the same technique from problem 4.

```csharp
public class BitBasics
{
    public static int MinBitFlips(int a, int b)
    {
        int xorResult = a ^ b; // 1s mark differing bit positions
        int flips = 0;

        while (xorResult != 0)
        {
            xorResult = xorResult & (xorResult - 1); // clear lowest set bit
            flips++;
        }

        return flips;
    }
}
```

Time Complexity: O(k), where k is the number of set bits in `a ^ b` (worst case O(log(max(a, b)))).
Space Complexity: O(1).

**Explanation:**
```
a = 10 = 1010
b = 7  = 0111

a ^ b:
  1010
^ 0111
------
  1101   (differing positions marked with 1)

Now count set bits in 1101 using n & (n-1):

Step 1: 1101 & 1100
  1101
& 1100
------
  1100   (flips = 1)

Step 2: 1100 & 1011
  1100
& 1011
------
  1000   (flips = 2)

Step 3: 1000 & 0111
  1000
& 0111
------
  0000   (flips = 3)

Result: 3 set bits => 3 flips required to convert a into b
```
