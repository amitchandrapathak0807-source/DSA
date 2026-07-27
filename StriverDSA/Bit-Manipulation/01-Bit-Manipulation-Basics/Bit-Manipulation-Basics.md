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

**Logic (Steps):**
1. Build `mask = 1 << i`, a value with a single `1` bit at position `i` and `0`s elsewhere.
2. Compute `n & mask` — every bit of the result is `0` except possibly position `i`.
3. If `n`'s bit at position `i` was `1`, the AND yields the non-zero mask value; if it was `0`, the AND yields `0`.
4. Return whether the AND result is non-zero.

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

**Walkthrough:** For `n = 13 (1101)`, `i = 2`: `mask = 1 << 2 = 0100`. Computing `n & mask` gives `1101 & 0100 = 0100`, which is non-zero, so the function returns `true`. This matches the expected output `true` — the 2nd bit of `13` is indeed set.

---

## 2. Check if a Number is Odd or Even Using Bit Manipulation

**Problem Statement:** Given an integer `n`, determine whether it is odd or even without using the modulo (`%`) operator.

**Example:**
- Input: `n = 7 (0111)`
- Output: `Odd`
- Explanation: `7` in binary is `0111`. The last bit (LSB, index `0`) is `1`, which means the number is odd. If the LSB were `0`, the number would be even.

**Approach:** In binary, the least significant bit (LSB) determines parity — it is `1` for odd numbers and `0` for even numbers, because every bit position except index `0` represents an even contribution (`2, 4, 8, ...`). AND `n` with `1` (mask `...0001`) to isolate the LSB. If the result is `1`, `n` is odd; if `0`, `n` is even.

**Logic (Steps):**
1. Use the constant mask `1` (binary `...0001`) which isolates only the LSB.
2. Compute `n & 1`.
3. If the LSB of `n` was `1`, the result is `1` (odd); if the LSB was `0`, the result is `0` (even).
4. Return `(n & 1) == 1`.

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

**Walkthrough:** For `n = 7 (0111)`: `0111 & 0001 = 0001`, which equals `1`, so `IsOdd` returns `true`, matching the expected output `Odd`. As a contrast, for `n = 8 (1000)`: `1000 & 0001 = 0000`, so the function would return `false` (Even).

---

## 3. Check if a Number is a Power of Two Using Bit Manipulation

**Problem Statement:** Given a positive integer `n`, determine whether it is a power of two (i.e., `n = 2^k` for some non-negative integer `k`).

**Example:**
- Input: `n = 16 (10000)`
- Output: `true`
- Explanation: `16` in binary is `10000`, which has exactly one set bit. Numbers that are powers of two always have exactly one set bit in their binary representation.

**Approach:** A power of two always has exactly one set bit (e.g., `1 = 0001`, `2 = 0010`, `4 = 0100`, `8 = 1000`). The expression `n & (n - 1)` clears the lowest set bit of `n`. If `n` has only one set bit, `n & (n - 1)` becomes `0`. So `n` is a power of two if `n > 0` and `(n & (n - 1)) == 0`.

**Logic (Steps):**
1. First check `n > 0`, since powers of two are positive by definition.
2. Compute `n - 1`, which flips the lowest set bit of `n` to `0` and sets every bit below it to `1`.
3. Compute `n & (n - 1)` — this clears exactly the lowest set bit of `n`.
4. If `n` had only one set bit, this AND produces `0`; otherwise it leaves other set bits behind, producing a non-zero value.
5. Return `n > 0 && (n & (n - 1)) == 0`.

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

**Walkthrough:** For `n = 16 (10000)`: `n - 1 = 01111`, and `10000 & 01111 = 00000`. Since `n > 0` and the AND result is `0`, the function returns `true`, matching the expected output. As a contrast, for `n = 12 (01100)`: `n - 1 = 01011`, and `01100 & 01011 = 01000` (non-zero), so `IsPowerOfTwo` would return `false`.

---

## 4. Count the Number of Set Bits in an Integer

**Problem Statement:** Given an integer `n`, count the number of `1`s (set bits) in its binary representation. This is also known as computing the Hamming Weight, and the technique used is Brian Kernighan's Algorithm.

**Example:**
- Input: `n = 11 (1011)`
- Output: `3`
- Explanation: `11` in binary is `1011`, which has three `1` bits (at positions `0`, `1`, and `3`).

**Approach:** Repeatedly apply `n = n & (n - 1)`, which clears the lowest set bit on each iteration, and count how many iterations it takes for `n` to become `0`. This is Brian Kernighan's Algorithm — it runs once per set bit rather than once per total bit, making it efficient when the number of set bits is small.

**Logic (Steps):**
1. Initialize `count = 0`.
2. While `n != 0`, apply `n = n & (n - 1)`, which clears the lowest set bit of `n`.
3. Increment `count` on each clearing operation.
4. The loop terminates once all set bits have been cleared (`n == 0`); `count` now holds the total number of set bits.

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

**Walkthrough:** For `n = 11 (1011)`: iteration 1 gives `1011 & 1010 = 1010` (count = 1); iteration 2 gives `1010 & 1001 = 1000` (count = 2); iteration 3 gives `1000 & 0111 = 0000` (count = 3). Now `n == 0`, the loop ends, and `CountSetBits` returns `3`, matching the expected output.

---

## 5. Swap Two Numbers Without Using a Temporary Variable (XOR trick)

**Problem Statement:** Given two integers `a` and `b`, swap their values without using any extra temporary/auxiliary variable.

**Example:**
- Input: `a = 5, b = 9`
- Output: `a = 9, b = 5`
- Explanation: Using successive XOR operations, the values of `a` and `b` are exchanged in place using only bitwise operations.

**Approach:** XOR has the property that `x ^ x = 0` and `x ^ 0 = x`, and XOR is its own inverse. By performing `a = a ^ b`, `b = a ^ b`, `a = a ^ b` in sequence, the original values effectively swap places without needing a third variable.

**Logic (Steps):**
1. Set `a = a ^ b`. Now `a` holds the XOR of both original values, while `b` is unchanged.
2. Set `b = a ^ b`. Since `a` is `origA ^ origB`, XORing with `origB` cancels `origB` out, leaving `b = origA`.
3. Set `a = a ^ b`. Since `a` is still `origA ^ origB` and `b` is now `origA`, XORing cancels `origA` out, leaving `a = origB`.
4. `a` and `b` now hold each other's original values, achieved purely with XOR and no temporary storage.

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

**Walkthrough:** For `a = 5 (0101)`, `b = 9 (1001)`: step 1 gives `a = 0101 ^ 1001 = 1100` (12), `b` still `1001` (9). Step 2 gives `b = 1100 ^ 1001 = 0101` (5, the original `a`). Step 3 gives `a = 1100 ^ 0101 = 1001` (9, the original `b`). Final result: `a = 9, b = 5`, matching the expected output.

---

## 6. Minimum Number of Bit Flips to Convert Number A to Number B

**Problem Statement:** Given two integers `a` and `b`, find the minimum number of bit flips required to convert `a` into `b`. A bit flip changes a `0` to `1` or a `1` to `0` at a single position.

**Example:**
- Input: `a = 10 (1010), b = 7 (0111)`
- Output: `3`
- Explanation: XORing `a` and `b` gives `1010 ^ 0111 = 1101`, which has three set bits. Each set bit in the XOR result marks a position where `a` and `b` differ, and each such differing position requires exactly one flip.

**Approach:** XOR `a` and `b`. Every bit position where `a` and `b` differ produces a `1` in `a ^ b`, and every position where they match produces a `0`. Therefore, the number of set bits in `a ^ b` equals the minimum number of flips needed. Count the set bits using Brian Kernighan's Algorithm (`n & (n - 1)`), the same technique from problem 4.

**Logic (Steps):**
1. Compute `xorResult = a ^ b`; a `1` at any position marks a bit where `a` and `b` differ.
2. Count the set bits in `xorResult` using Brian Kernighan's Algorithm: repeatedly apply `xorResult = xorResult & (xorResult - 1)`, incrementing `flips` each time, until `xorResult` becomes `0`.
3. Each cleared bit corresponds to exactly one required flip, so the final `flips` count is the minimum number of bit flips needed to turn `a` into `b`.

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

**Walkthrough:** For `a = 10 (1010)`, `b = 7 (0111)`: `xorResult = 1010 ^ 0111 = 1101`. Clearing set bits: `1101 & 1100 = 1100` (flips = 1), `1100 & 1011 = 1000` (flips = 2), `1000 & 0111 = 0000` (flips = 3). `xorResult` is now `0`, the loop ends, and `MinBitFlips` returns `3`, matching the expected output.
