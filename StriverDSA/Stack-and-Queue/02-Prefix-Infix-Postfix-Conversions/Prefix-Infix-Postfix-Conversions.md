# Stack and Queue — Prefix, Infix, and Postfix Conversions

## Concept: Operator Precedence and Associativity

Expression conversions rely on knowing which operator binds tighter, and in which
direction operators of the same precedence group together. The table below is all
that's needed for the conversions in this note (assuming only `^`, `*`, `/`, `+`, `-`
and single-letter operands):

| Operator | Precedence | Associativity |
|----------|------------|----------------|
| `^`      | 3 (highest)| Right to Left  |
| `*`, `/` | 2          | Left to Right  |
| `+`, `-` | 1 (lowest) | Left to Right  |

Rules of thumb used repeatedly below:
- When scanning left-to-right for infix → postfix, an operator on the stack is popped
  before pushing the current operator `op` if the stack-top has **strictly higher**
  precedence, or **equal precedence with left-to-right associativity**. For `^`
  (right-to-left), equal precedence does *not* trigger a pop.
- Parentheses `(` always get pushed and act as a barrier; `)` pops until `(` is found.
- Postfix/prefix conversions that don't involve operator-precedence stacks (postfix↔infix,
  postfix↔prefix, prefix↔infix, prefix↔postfix) instead use a single scan that pushes
  operands and, on encountering an operator, pops exactly two operands and pushes back a
  combined string — precedence is irrelevant there because the postfix/prefix form has
  already encoded the evaluation order unambiguously.

---

## 1. Infix to Postfix Conversion

**Problem Statement:** Given an infix expression (operators between operands, e.g.
`A+B*C`), convert it to postfix form (operators after their operands, e.g. `ABC*+`)
using an operator stack and the precedence/associativity rules above.

**Example:**
- Input: `A+B*C`
- Output: `ABC*+`
- Explanation: `*` has higher precedence than `+`, so `B*C` is evaluated first; in
  postfix that group becomes `BC*`, and appending it after `A` with `+` at the end
  gives `ABC*+`.

**Approach:**
1. Scan the infix expression left to right.
2. If the character is an operand, append it directly to the result.
3. If it's `(`, push it onto the stack.
4. If it's `)`, pop from the stack into the result until `(` is popped (discard the
   `(`).
5. If it's an operator, pop from the stack into the result while the stack top has
   greater precedence, or equal precedence with left-to-right associativity, than the
   current operator; then push the current operator.
6. After the scan, pop all remaining operators from the stack into the result.

```csharp
using System;
using System.Collections.Generic;
using System.Text;

public class InfixToPostfix
{
    private static int Precedence(char op)
    {
        switch (op)
        {
            case '^': return 3;
            case '*':
            case '/': return 2;
            case '+':
            case '-': return 1;
            default: return -1;
        }
    }

    private static bool IsRightAssociative(char op) => op == '^';

    private static bool IsOperator(char c) => c == '+' || c == '-' || c == '*' || c == '/' || c == '^';

    public static string Convert(string infix)
    {
        var result = new StringBuilder();
        var stack = new Stack<char>();

        foreach (char c in infix)
        {
            if (char.IsLetterOrDigit(c))
            {
                result.Append(c);
            }
            else if (c == '(')
            {
                stack.Push(c);
            }
            else if (c == ')')
            {
                while (stack.Count > 0 && stack.Peek() != '(')
                {
                    result.Append(stack.Pop());
                }
                if (stack.Count > 0) stack.Pop(); // discard '('
            }
            else if (IsOperator(c))
            {
                while (stack.Count > 0 && stack.Peek() != '(' &&
                       (Precedence(stack.Peek()) > Precedence(c) ||
                        (Precedence(stack.Peek()) == Precedence(c) && !IsRightAssociative(c))))
                {
                    result.Append(stack.Pop());
                }
                stack.Push(c);
            }
        }

        while (stack.Count > 0)
        {
            result.Append(stack.Pop());
        }

        return result.ToString();
    }
}
```
Time Complexity: O(n), Space Complexity: O(n).

**Explanation:** Dry run on `A+B*C`:

| Step | Char | Action                                   | Operator Stack | Result   |
|------|------|-------------------------------------------|-----------------|----------|
| 1    | `A`  | operand → append                          | (empty)         | `A`      |
| 2    | `+`  | stack empty → push                        | `+`             | `A`      |
| 3    | `B`  | operand → append                          | `+`             | `AB`     |
| 4    | `*`  | top is `+`, prec(`+`)=1 < prec(`*`)=2 → don't pop, push `*` | `+ *`           | `AB`     |
| 5    | `C`  | operand → append                          | `+ *`           | `ABC`    |
| end  | —    | pop all remaining: `*` then `+`           | (empty)         | `ABC*+`  |

`*` stays on top of `+` (isn't popped when pushed) because it binds tighter — that's
why it ends up appearing *before* `+` in the final result once everything unwinds at
the end. Final answer: `ABC*+`.

---

## 2. Infix to Prefix Conversion

**Problem Statement:** Given an infix expression, convert it to prefix form (operator
before its operands, e.g. `+A*BC`).

**Example:**
- Input: `A+B*C`
- Output: `+A*BC`
- Explanation: `B*C` becomes `*BC` in prefix; combined with `A` and `+`, we get
  `+A*BC`.

**Approach:** Reuse the infix-to-postfix idea with two twists:
1. Reverse the infix string, and while reversing swap every `(` with `)` and vice
   versa.
2. Run the (adjusted) infix-to-postfix algorithm on this reversed string. The only
   change needed is that for `^` — and more generally to handle equal-precedence
   operators correctly after reversal — ties should NOT cause a pop (i.e., treat the
   scan as strictly "pop only if strictly greater precedence"), since we are
   effectively building right-to-left.
3. Reverse the resulting postfix-like string to get the prefix expression.

```csharp
using System;
using System.Collections.Generic;
using System.Text;

public class InfixToPrefix
{
    private static int Precedence(char op)
    {
        switch (op)
        {
            case '^': return 3;
            case '*':
            case '/': return 2;
            case '+':
            case '-': return 1;
            default: return -1;
        }
    }

    private static bool IsOperator(char c) => c == '+' || c == '-' || c == '*' || c == '/' || c == '^';

    public static string Convert(string infix)
    {
        // Step 1: reverse infix, swapping parentheses.
        char[] arr = infix.ToCharArray();
        Array.Reverse(arr);
        var reversed = new StringBuilder();
        foreach (char c in arr)
        {
            if (c == '(') reversed.Append(')');
            else if (c == ')') reversed.Append('(');
            else reversed.Append(c);
        }

        // Step 2: modified infix-to-postfix on the reversed string.
        var result = new StringBuilder();
        var stack = new Stack<char>();

        foreach (char c in reversed.ToString())
        {
            if (char.IsLetterOrDigit(c))
            {
                result.Append(c);
            }
            else if (c == '(')
            {
                stack.Push(c);
            }
            else if (c == ')')
            {
                while (stack.Count > 0 && stack.Peek() != '(')
                {
                    result.Append(stack.Pop());
                }
                if (stack.Count > 0) stack.Pop();
            }
            else if (IsOperator(c))
            {
                // Strictly-greater comparison (no pop on equal precedence) since we
                // are scanning right-to-left conceptually.
                while (stack.Count > 0 && stack.Peek() != '(' &&
                       Precedence(stack.Peek()) > Precedence(c))
                {
                    result.Append(stack.Pop());
                }
                stack.Push(c);
            }
        }

        while (stack.Count > 0)
        {
            result.Append(stack.Pop());
        }

        // Step 3: reverse the postfix-of-reversed to get prefix.
        char[] finalArr = result.ToString().ToCharArray();
        Array.Reverse(finalArr);
        return new string(finalArr);
    }
}
```
Time Complexity: O(n), Space Complexity: O(n).

**Explanation:** For `A+B*C`: reversing gives `C*B+A` (no parens to swap here). Running
the modified infix-to-postfix scan on `C*B+A` yields `CB*A+` (mirroring the postfix
logic from Problem 1 but scanning the mirrored string). Reversing `CB*A+` gives
`+A*BC`, which is the prefix form: `+` applies to `A` and the sub-expression `*BC`
(i.e., `B*C`), matching `A + (B*C)`.

---

## 3. Postfix to Infix Conversion

**Problem Statement:** Given a postfix expression (e.g. `AB+C*`), convert it back to a
fully parenthesized infix expression (e.g. `((A+B)*C)`).

**Example:**
- Input: `AB+C*`
- Output: `((A+B)*C)`
- Explanation: `AB+` combines to `(A+B)`; then combining that with `C` using `*` gives
  `((A+B)*C)`.

**Approach:**
1. Scan postfix left to right using a stack of strings.
2. If the character is an operand, push it as a single-character string.
3. If it's an operator, pop the top two strings `op2` then `op1` (in that pop order,
   since `op1` was pushed first), and push the combined string
   `"(" + op1 + operator + op2 + ")"`.
4. After the scan, the stack contains exactly one string — the fully parenthesized
   infix expression.

```csharp
using System.Collections.Generic;

public class PostfixToInfix
{
    private static bool IsOperator(char c) => c == '+' || c == '-' || c == '*' || c == '/' || c == '^';

    public static string Convert(string postfix)
    {
        var stack = new Stack<string>();

        foreach (char c in postfix)
        {
            if (char.IsLetterOrDigit(c))
            {
                stack.Push(c.ToString());
            }
            else if (IsOperator(c))
            {
                string op2 = stack.Pop();
                string op1 = stack.Pop();
                string combined = "(" + op1 + c + op2 + ")";
                stack.Push(combined);
            }
        }

        return stack.Pop();
    }
}
```
Time Complexity: O(n), Space Complexity: O(n).

**Explanation:** Dry run on `AB+C*`:

| Step | Char | Action                                             | Operand Stack (top → right) |
|------|------|------------------------------------------------------|------------------------------|
| 1    | `A`  | push operand                                          | `A`                          |
| 2    | `B`  | push operand                                          | `A`, `B`                     |
| 3    | `+`  | pop `B`(op2), `A`(op1) → push `(A+B)`                 | `(A+B)`                      |
| 4    | `C`  | push operand                                          | `(A+B)`, `C`                 |
| 5    | `*`  | pop `C`(op2), `(A+B)`(op1) → push `((A+B)*C)`         | `((A+B)*C)`                  |

Final answer: `((A+B)*C)`.

---

## 4. Postfix to Prefix Conversion

**Problem Statement:** Given a postfix expression (e.g. `AB+C*`), convert it to
prefix form (e.g. `*+ABC`).

**Example:**
- Input: `AB+C*`
- Output: `*+ABC`
- Explanation: `AB+` becomes prefix `+AB`; combining with `C` using `*` (prefix form
  puts the operator first) gives `*+ABC`.

**Approach:** Same left-to-right scan and stack-of-strings idea as postfix-to-infix,
except when combining two operands for an operator, the operator goes in *front*:
`operator + op1 + op2` (no parentheses needed, since prefix/postfix are unambiguous
without them).

```csharp
using System.Collections.Generic;

public class PostfixToPrefix
{
    private static bool IsOperator(char c) => c == '+' || c == '-' || c == '*' || c == '/' || c == '^';

    public static string Convert(string postfix)
    {
        var stack = new Stack<string>();

        foreach (char c in postfix)
        {
            if (char.IsLetterOrDigit(c))
            {
                stack.Push(c.ToString());
            }
            else if (IsOperator(c))
            {
                string op2 = stack.Pop();
                string op1 = stack.Pop();
                string combined = c + op1 + op2;
                stack.Push(combined);
            }
        }

        return stack.Pop();
    }
}
```
Time Complexity: O(n), Space Complexity: O(n).

**Explanation:** Dry run on `AB+C*`:

| Step | Char | Action                                          | Operand Stack     |
|------|------|---------------------------------------------------|--------------------|
| 1    | `A`  | push                                               | `A`                |
| 2    | `B`  | push                                               | `A`, `B`           |
| 3    | `+`  | pop `B`(op2), `A`(op1) → push `+AB`               | `+AB`              |
| 4    | `C`  | push                                               | `+AB`, `C`         |
| 5    | `*`  | pop `C`(op2), `+AB`(op1) → push `*+ABC`           | `*+ABC`            |

Final answer: `*+ABC`.

---

## 5. Prefix to Infix Conversion

**Problem Statement:** Given a prefix expression (e.g. `*+ABC`), convert it to a fully
parenthesized infix expression (e.g. `((A+B)*C)`).

**Example:**
- Input: `*+ABC`
- Output: `((A+B)*C)`
- Explanation: Reading `+AB` first gives `(A+B)`; then `*` combined with `(A+B)` and
  `C` gives `((A+B)*C)`.

**Approach:** Prefix expressions are naturally scanned **right to left** (mirroring
why postfix is scanned left to right):
1. Scan the prefix string from right to left using a stack of strings.
2. If the character is an operand, push it.
3. If it's an operator, pop the top two strings `op1` then `op2` (in that pop order,
   since we're going right to left, the first popped is the *left* operand), and push
   `"(" + op1 + operator + op2 + ")"`.
4. After the scan, the stack holds one string — the infix expression.

```csharp
using System.Collections.Generic;

public class PrefixToInfix
{
    private static bool IsOperator(char c) => c == '+' || c == '-' || c == '*' || c == '/' || c == '^';

    public static string Convert(string prefix)
    {
        var stack = new Stack<string>();

        for (int i = prefix.Length - 1; i >= 0; i--)
        {
            char c = prefix[i];
            if (char.IsLetterOrDigit(c))
            {
                stack.Push(c.ToString());
            }
            else if (IsOperator(c))
            {
                string op1 = stack.Pop();
                string op2 = stack.Pop();
                string combined = "(" + op1 + c + op2 + ")";
                stack.Push(combined);
            }
        }

        return stack.Pop();
    }
}
```
Time Complexity: O(n), Space Complexity: O(n).

**Explanation:** Dry run on `*+ABC` (scanned right to left: `C`, `B`, `A`, `+`, `*`):

| Step | Char | Action                                             | Operand Stack     |
|------|------|-------------------------------------------------------|--------------------|
| 1    | `C`  | push                                                   | `C`                |
| 2    | `B`  | push                                                   | `C`, `B`           |
| 3    | `A`  | push                                                   | `C`, `B`, `A`      |
| 4    | `+`  | pop `A`(op1), `B`(op2) → push `(A+B)`                 | `C`, `(A+B)`       |
| 5    | `*`  | pop `(A+B)`(op1), `C`(op2) → push `((A+B)*C)`         | `((A+B)*C)`        |

Final answer: `((A+B)*C)`.

---

## 6. Prefix to Postfix Conversion

**Problem Statement:** Given a prefix expression (e.g. `*+ABC`), convert it to postfix
form (e.g. `AB+C*`).

**Example:**
- Input: `*+ABC`
- Output: `AB+C*`
- Explanation: `+AB` becomes postfix `AB+`; combining with `C` and `*` (operator now
  goes last) gives `AB+C*`.

**Approach:** Same right-to-left scan as prefix-to-infix, but combine without
parentheses and put the operator at the end: `op1 + op2 + operator`.

```csharp
using System.Collections.Generic;

public class PrefixToPostfix
{
    private static bool IsOperator(char c) => c == '+' || c == '-' || c == '*' || c == '/' || c == '^';

    public static string Convert(string prefix)
    {
        var stack = new Stack<string>();

        for (int i = prefix.Length - 1; i >= 0; i--)
        {
            char c = prefix[i];
            if (char.IsLetterOrDigit(c))
            {
                stack.Push(c.ToString());
            }
            else if (IsOperator(c))
            {
                string op1 = stack.Pop();
                string op2 = stack.Pop();
                string combined = op1 + op2 + c;
                stack.Push(combined);
            }
        }

        return stack.Pop();
    }
}
```
Time Complexity: O(n), Space Complexity: O(n).

**Explanation:** Dry run on `*+ABC` (scanned right to left: `C`, `B`, `A`, `+`, `*`):

| Step | Char | Action                                          | Operand Stack   |
|------|------|-----------------------------------------------------|------------------|
| 1    | `C`  | push                                                 | `C`              |
| 2    | `B`  | push                                                 | `C`, `B`         |
| 3    | `A`  | push                                                 | `C`, `B`, `A`    |
| 4    | `+`  | pop `A`(op1), `B`(op2) → push `AB+`                 | `C`, `AB+`       |
| 5    | `*`  | pop `AB+`(op1), `C`(op2) → push `AB+C*`             | `AB+C*`          |

Final answer: `AB+C*`.
