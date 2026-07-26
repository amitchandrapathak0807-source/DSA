# Stack and Queue — Implementation

## 1. Implement Stack Using Arrays

**Problem Statement:** Design a Stack data structure using a fixed/resizable array as the underlying storage. Support the following operations:
- `Push(x)` — insert element `x` on top of the stack.
- `Pop()` — remove and return the element on top of the stack.
- `Top()` / `Peek()` — return the element on top without removing it.
- `IsEmpty()` — return whether the stack has no elements.

**Example:**
- Input: `Push(1), Push(2), Push(3), Top() -> 3, Pop() -> 3, Top() -> 2, IsEmpty() -> false`
- Output: `3, 3, 2, false`

**Approach:** Maintain an internal array `arr` and an integer `top` (initialized to `-1`) that always points to the index of the last inserted element. `Push` increments `top` and stores the value at `arr[top]` (resizing the array — typically doubling its capacity — when `top` reaches the array's length). `Pop` reads `arr[top]` and decrements `top`. `Top` simply reads `arr[top]`. `IsEmpty` checks whether `top == -1`. Because insertion/removal only happens at one end (the "top"), all operations only touch a single index and never require shifting elements.

```csharp
using System;

public class ArrayStack
{
    private int[] arr;
    private int top;      // index of the top element, -1 when empty
    private int capacity;

    public ArrayStack(int initialCapacity = 4)
    {
        capacity = initialCapacity;
        arr = new int[capacity];
        top = -1;
    }

    public bool IsEmpty() => top == -1;

    public bool IsFull() => top == capacity - 1;

    public void Push(int x)
    {
        if (IsFull())
        {
            Resize(capacity * 2);
        }
        arr[++top] = x;
    }

    public int Pop()
    {
        if (IsEmpty())
            throw new InvalidOperationException("Stack is empty");
        return arr[top--];
    }

    public int Top()
    {
        if (IsEmpty())
            throw new InvalidOperationException("Stack is empty");
        return arr[top];
    }

    public int Size() => top + 1;

    private void Resize(int newCapacity)
    {
        int[] newArr = new int[newCapacity];
        Array.Copy(arr, newArr, arr.Length);
        arr = newArr;
        capacity = newCapacity;
    }
}
```

Time Complexity: `Push` — O(1) amortized (O(n) on the rare resize), `Pop` — O(1), `Top` — O(1), `IsEmpty` — O(1).
Space Complexity: O(n), where `n` is the number of elements currently stored (plus unused pre-allocated capacity).

**Explanation:** Since `top` always tracks the last filled index, `Push` and `Pop` never need to touch any element other than the one at `top`. The array only grows (doubling) when full, which amortizes the O(n) copy cost across many O(1) pushes, giving amortized O(1) per push. There is no shifting of elements ever, unlike inserting/removing at the front of an array — that is what makes an array a natural fit for stack behavior.

---

## 2. Implement Stack Using a Linked List

**Problem Statement:** Design a Stack data structure using a singly linked list, so that no resizing is ever required. Support:
- `Push(x)` — insert element `x` on top of the stack.
- `Pop()` — remove and return the element on top of the stack.
- `Top()` / `Peek()` — return the top element without removing it.
- `IsEmpty()` — return whether the stack has no elements.

**Example:**
- Input: `Push(10), Push(20), Push(30), Top() -> 30, Pop() -> 30, IsEmpty() -> false, Pop(), Pop(), IsEmpty() -> true`
- Output: `30, 30, false, true`

**Approach:** Keep a `head` pointer that always represents the "top" of the stack. `Push` creates a new node whose `Next` points to the current `head`, then reassigns `head` to this new node — an O(1) insertion at the front. `Pop` reads `head.Value`, then reassigns `head` to `head.Next`, effectively unlinking the old head — also O(1). Because insertion/removal always happens at the head (never traversing the list), there is no need to track size separately unless a `Size()` operation is wanted, and no array resizing is ever necessary.

```csharp
using System;

public class LinkedListStack
{
    private class Node
    {
        public int Value;
        public Node Next;
        public Node(int value, Node next = null)
        {
            Value = value;
            Next = next;
        }
    }

    private Node head;
    private int count;

    public bool IsEmpty() => head == null;

    public void Push(int x)
    {
        head = new Node(x, head);
        count++;
    }

    public int Pop()
    {
        if (IsEmpty())
            throw new InvalidOperationException("Stack is empty");
        int value = head.Value;
        head = head.Next;
        count--;
        return value;
    }

    public int Top()
    {
        if (IsEmpty())
            throw new InvalidOperationException("Stack is empty");
        return head.Value;
    }

    public int Size() => count;
}
```

Time Complexity: `Push` — O(1), `Pop` — O(1), `Top` — O(1), `IsEmpty` — O(1).
Space Complexity: O(n) for `n` elements, with an extra pointer per node (higher constant factor than the array version, but no wasted pre-allocated capacity).

**Explanation:** Because `head` is always the most recently pushed node, pushing simply "prepends" a node in constant time by rewiring one pointer, and popping "removes the head" by moving `head` one step forward and letting the old node become unreachable (garbage collected). Neither operation ever walks the list, so both stay strictly O(1) regardless of stack size — no amortization needed, unlike the array version's occasional resize.

---

## 3. Implement Queue Using Arrays (circular array)

**Problem Statement:** Design a Queue data structure using a fixed-size circular array to support:
- `Enqueue(x)` — insert element `x` at the rear of the queue.
- `Dequeue()` — remove and return the element at the front of the queue.
- `Front()` — return the front element without removing it.
- `IsEmpty()` / `IsFull()` — return whether the queue has no elements / is at capacity.

**Example:**
- Input: `Enqueue(1), Enqueue(2), Enqueue(3), Dequeue() -> 1, Front() -> 2, Enqueue(4), Dequeue() -> 2`
- Output: `1, 2, 2`

**Approach:** Use a fixed-size array `arr` of capacity `cap`, along with a `front` index, a `rear` index, and a `count` (number of currently stored elements). `Enqueue` places the new element at `arr[rear]`, then advances `rear = (rear + 1) % cap` (wrapping around to index 0 once it passes the end) and increments `count`. `Dequeue` reads `arr[front]`, advances `front = (front + 1) % cap`, and decrements `count`. The modulo wrap-around is what makes the array "circular" — it reuses slots freed by dequeues instead of shifting all elements forward, which a naive linear array queue would require. `count` (rather than comparing `front == rear`, which is ambiguous) is used to distinguish a full queue from an empty one, since both states can otherwise show `front == rear`.

```csharp
using System;

public class CircularArrayQueue
{
    private int[] arr;
    private int front;
    private int rear;
    private int count;
    private readonly int capacity;

    public CircularArrayQueue(int capacity)
    {
        this.capacity = capacity;
        arr = new int[capacity];
        front = 0;
        rear = 0;
        count = 0;
    }

    public bool IsEmpty() => count == 0;

    public bool IsFull() => count == capacity;

    public void Enqueue(int x)
    {
        if (IsFull())
            throw new InvalidOperationException("Queue is full");
        arr[rear] = x;
        rear = (rear + 1) % capacity;
        count++;
    }

    public int Dequeue()
    {
        if (IsEmpty())
            throw new InvalidOperationException("Queue is empty");
        int value = arr[front];
        front = (front + 1) % capacity;
        count--;
        return value;
    }

    public int Front()
    {
        if (IsEmpty())
            throw new InvalidOperationException("Queue is empty");
        return arr[front];
    }

    public int Size() => count;
}
```

Time Complexity: `Enqueue` — O(1), `Dequeue` — O(1), `Front` — O(1), `IsEmpty`/`IsFull` — O(1).
Space Complexity: O(capacity) — fixed regardless of how many enqueue/dequeue cycles occur.

**Explanation:** In a plain (non-circular) array queue, dequeuing from the front either wastes the freed slot forever or forces an O(n) shift of all remaining elements. The circular array avoids both by wrapping `rear` and `front` around with modulo arithmetic, so slots vacated near the beginning are reused once `rear` wraps past the end. `count` cleanly resolves the classic ambiguity where `front == rear` could mean either "empty" or "full" if only two pointers were used.

---

## 4. Implement Queue Using a Linked List

**Problem Statement:** Design a Queue data structure using a singly linked list, avoiding any fixed capacity limit. Support:
- `Enqueue(x)` — insert element `x` at the rear of the queue.
- `Dequeue()` — remove and return the element at the front of the queue.
- `Front()` — return the front element without removing it.
- `IsEmpty()` — return whether the queue has no elements.

**Example:**
- Input: `Enqueue(5), Enqueue(6), Enqueue(7), Front() -> 5, Dequeue() -> 5, Dequeue() -> 6, Front() -> 7`
- Output: `5, 5, 6, 7`

**Approach:** Maintain two pointers into a singly linked list: `head` (the front of the queue) and `tail` (the rear of the queue). `Enqueue` creates a new node, links the current `tail.Next` to it, and moves `tail` to point to this new node (if the list was empty, both `head` and `tail` are set to the new node). `Dequeue` reads `head.Value`, then advances `head = head.Next` (if this empties the list, `tail` is reset to `null` too). Keeping an explicit `tail` pointer is what allows `Enqueue` to be O(1) — without it, appending to the end of a singly linked list would require an O(n) traversal from `head`.

```csharp
using System;

public class LinkedListQueue
{
    private class Node
    {
        public int Value;
        public Node Next;
        public Node(int value)
        {
            Value = value;
            Next = null;
        }
    }

    private Node head;
    private Node tail;
    private int count;

    public bool IsEmpty() => head == null;

    public void Enqueue(int x)
    {
        Node node = new Node(x);
        if (IsEmpty())
        {
            head = node;
            tail = node;
        }
        else
        {
            tail.Next = node;
            tail = node;
        }
        count++;
    }

    public int Dequeue()
    {
        if (IsEmpty())
            throw new InvalidOperationException("Queue is empty");
        int value = head.Value;
        head = head.Next;
        if (head == null)
            tail = null;
        count--;
        return value;
    }

    public int Front()
    {
        if (IsEmpty())
            throw new InvalidOperationException("Queue is empty");
        return head.Value;
    }

    public int Size() => count;
}
```

Time Complexity: `Enqueue` — O(1), `Dequeue` — O(1), `Front` — O(1), `IsEmpty` — O(1).
Space Complexity: O(n) for `n` elements, plus one pointer per node.

**Explanation:** `head` always identifies the node to dequeue next, and `tail` always identifies the node after which a new node should be appended, so both operations touch a constant number of pointers regardless of queue length. There is no capacity limit and no resizing, unlike the array-based version, at the cost of extra per-node pointer memory.

---

## 5. Implement Stack Using a Queue

**Problem Statement:** Design a Stack (LIFO) using only queue operations (`Enqueue`, `Dequeue`, `Front`, `IsEmpty`) as primitives — i.e., using one or two underlying `Queue<int>` instances. Support `Push(x)`, `Pop()`, `Top()`, `IsEmpty()` with standard stack (LIFO) semantics.

**Example:**
- Input: `Push(1), Push(2), Push(3), Top() -> 3, Pop() -> 3, Top() -> 2`
- Output: `3, 3, 2`

**Approach:** The classic single-queue technique makes `Push` do the expensive work so `Pop`/`Top` stay trivial. To `Push(x)`: enqueue `x` onto the queue, then rotate the queue by dequeuing and re-enqueuing all the elements that were in front of `x` (i.e., dequeue `size - 1` elements and immediately enqueue each one back) — this moves `x` from the back of the queue to the front. After this rotation, the queue's front always holds the most-recently-pushed element, so `Pop` and `Top` are simply `Dequeue()`/`Front()` on the underlying queue.

```csharp
using System;
using System.Collections.Generic;

public class QueueStack
{
    private readonly Queue<int> queue = new Queue<int>();

    public bool IsEmpty() => queue.Count == 0;

    public void Push(int x)
    {
        queue.Enqueue(x);
        // Rotate: move every element that was already in the queue
        // behind the newly enqueued x, so x ends up at the front.
        int rotations = queue.Count - 1;
        for (int i = 0; i < rotations; i++)
        {
            queue.Enqueue(queue.Dequeue());
        }
    }

    public int Pop()
    {
        if (IsEmpty())
            throw new InvalidOperationException("Stack is empty");
        return queue.Dequeue();
    }

    public int Top()
    {
        if (IsEmpty())
            throw new InvalidOperationException("Stack is empty");
        return queue.Peek();
    }

    public int Size() => queue.Count;
}
```

Time Complexity: `Push` — O(n) (rotation of all previously stored elements), `Pop` — O(1), `Top` — O(1), `IsEmpty` — O(1).
Space Complexity: O(n) for `n` elements stored in the single queue.

**Explanation:** Dry run with `Push(1), Push(2), Push(3)`:
- `Push(1)`: enqueue 1 → queue = `[1]`. Rotations = 0. Queue: `[1]`.
- `Push(2)`: enqueue 2 → queue = `[1, 2]`. Rotations = 1: dequeue 1, enqueue 1 → queue = `[2, 1]`.
- `Push(3)`: enqueue 3 → queue = `[2, 1, 3]`. Rotations = 2: dequeue 2, enqueue 2 → `[1, 3, 2]`; dequeue 1, enqueue 1 → `[3, 2, 1]`.

Now the queue front-to-back is `[3, 2, 1]`, exactly matching LIFO order (most recently pushed is at the front). `Top() -> 3` and `Pop() -> 3` simply read/remove the front, leaving `[2, 1]`, and `Top() -> 2` correctly reflects the new top of the stack.

---

## 6. Implement Queue Using a Stack

**Problem Statement:** Design a Queue (FIFO) using only stack operations (`Push`, `Pop`, `Top`, `IsEmpty`) as primitives — i.e., using two underlying `Stack<int>` instances (an "in-stack" and an "out-stack"). Support `Enqueue(x)`, `Dequeue()`, `Front()`, `IsEmpty()` with standard queue (FIFO) semantics, achieving amortized O(1) per operation.

**Example:**
- Input: `Enqueue(1), Enqueue(2), Enqueue(3), Dequeue() -> 1, Front() -> 2, Enqueue(4), Dequeue() -> 2`
- Output: `1, 2, 2`

**Approach:** Keep two stacks: `inStack` for incoming elements and `outStack` for outgoing elements. `Enqueue(x)` always just does `inStack.Push(x)` — O(1). `Dequeue`/`Front` first check whether `outStack` is empty; if so, they transfer every element from `inStack` to `outStack` by repeatedly popping from `inStack` and pushing onto `outStack` — this reversal makes the oldest-enqueued element end up on top of `outStack`. Then `Dequeue`/`Front` simply operate on `outStack`'s top. If `outStack` is already non-empty, the transfer is skipped and the existing top of `outStack` is used directly (it is still the oldest un-dequeued element).

```csharp
using System;
using System.Collections.Generic;

public class StackQueue
{
    private readonly Stack<int> inStack = new Stack<int>();
    private readonly Stack<int> outStack = new Stack<int>();

    public bool IsEmpty() => inStack.Count == 0 && outStack.Count == 0;

    public void Enqueue(int x)
    {
        inStack.Push(x);
    }

    private void TransferIfNeeded()
    {
        if (outStack.Count == 0)
        {
            while (inStack.Count > 0)
            {
                outStack.Push(inStack.Pop());
            }
        }
    }

    public int Dequeue()
    {
        if (IsEmpty())
            throw new InvalidOperationException("Queue is empty");
        TransferIfNeeded();
        return outStack.Pop();
    }

    public int Front()
    {
        if (IsEmpty())
            throw new InvalidOperationException("Queue is empty");
        TransferIfNeeded();
        return outStack.Peek();
    }

    public int Size() => inStack.Count + outStack.Count;
}
```

Time Complexity: `Enqueue` — O(1) always. `Dequeue`/`Front` — O(1) amortized, O(n) worst case (only when a transfer is triggered).
Space Complexity: O(n) across both stacks combined.

**Explanation (amortized analysis dry run):** Consider `Enqueue(1), Enqueue(2), Enqueue(3), Dequeue(), Front(), Enqueue(4), Dequeue()`.
- `Enqueue(1)`: `inStack = [1]`, `outStack = []`.
- `Enqueue(2)`: `inStack = [1, 2]`, `outStack = []`.
- `Enqueue(3)`: `inStack = [1, 2, 3]`, `outStack = []`.
- `Dequeue()`: `outStack` is empty, so transfer all of `inStack`: pop 3 → push to outStack, pop 2 → push, pop 1 → push. Now `inStack = []`, `outStack = [1, 2, 3]` (top is 1). Pop 1 from `outStack` → returns `1`. `outStack = [2, 3]`.
- `Front()`: `outStack` non-empty (`[2, 3]`, top 2), no transfer needed → returns `2`.
- `Enqueue(4)`: `inStack = [4]`, `outStack = [2, 3]` unchanged.
- `Dequeue()`: `outStack` non-empty, no transfer → pop top → returns `2`. `outStack = [3]`.

Every element is pushed onto `inStack` exactly once (during `Enqueue`), and over its entire lifetime is popped off `inStack` and pushed onto `outStack` at most once (during a transfer), and popped off `outStack` exactly once (during its own `Dequeue`). That is at most 4 stack operations total per element across the element's whole lifetime — a constant independent of how many other operations happen in between. So while a single `Dequeue` that triggers a transfer can cost O(n) in that instant, summing the total cost of all operations over `n` elements is O(n), giving amortized O(1) per operation.

---

## 7. Check for Balanced Parentheses

**Problem Statement:** Given a string containing only the bracket characters `(`, `)`, `{`, `}`, `[`, `]`, determine whether the brackets are "balanced" — every opening bracket has a matching closing bracket of the same type, and pairs are properly nested (not overlapping out of order).

**Example:**
- Input: `"{[()]}"`
- Output: `true` (balanced)
- Input: `"{[(])}"`
- Output: `false` (not balanced — brackets close out of order)

**Approach:** Use a stack of characters. Scan the string left to right. Whenever an opening bracket (`(`, `{`, `[`) is seen, push it onto the stack. Whenever a closing bracket (`)`, `}`, `]`) is seen, check that the stack is non-empty and that its top is the matching opening bracket; if so, pop it, otherwise the string is unbalanced. After scanning the entire string, it is balanced only if the stack ends up empty (every opening bracket found its match).

```csharp
using System;
using System.Collections.Generic;

public class BalancedParentheses
{
    public static bool IsBalanced(string s)
    {
        Stack<char> stack = new Stack<char>();
        Dictionary<char, char> matchingOpen = new Dictionary<char, char>
        {
            [')'] = '(',
            ['}'] = '{',
            [']'] = '['
        };

        foreach (char c in s)
        {
            if (c == '(' || c == '{' || c == '[')
            {
                stack.Push(c);
            }
            else if (matchingOpen.ContainsKey(c))
            {
                if (stack.Count == 0 || stack.Pop() != matchingOpen[c])
                    return false;
            }
            // any non-bracket character is ignored
        }

        return stack.Count == 0;
    }
}
```

Time Complexity: O(n), where `n` is the length of the string — each character is pushed/popped at most once.
Space Complexity: O(n) worst case, for a string made entirely of opening brackets.

**Explanation (dry runs):**

Success case `"{[()]}"`:
| Char | Action | Stack after |
|---|---|---|
| `{` | push | `{` |
| `[` | push | `{ [` |
| `(` | push | `{ [ (` |
| `)` | pop, matches `(` | `{ [` |
| `]` | pop, matches `[` | `{` |
| `}` | pop, matches `{` | (empty) |

Stack ends empty → `true`, balanced.

Failure case `"{[(])}"`:
| Char | Action | Stack after |
|---|---|---|
| `{` | push | `{` |
| `[` | push | `{ [` |
| `(` | push | `{ [ (` |
| `]` | pop top `(`, but expected match for `]` is `[` — mismatch! | fails here |

At the fourth character `]`, the stack's top is `(` (the matching-open lookup expects `[`), so `stack.Pop() != matchingOpen[']']` is true and the function returns `false` immediately — correctly detecting the out-of-order closing.

---

## 8. Implement a Min Stack (getMin in O(1))

**Problem Statement:** Design a stack that, in addition to the standard `Push(x)`, `Pop()`, and `Top()` operations, supports retrieving the minimum element currently in the stack via `GetMin()` — all operations must run in O(1) time.

**Example:**
- Input: `Push(3), Push(5), GetMin() -> 3, Push(2), Push(1), GetMin() -> 1, Pop(), GetMin() -> 2`
- Output: `3, 1, 2`

**Approach:** Maintain a single stack of pairs `(value, currentMin)`, where `currentMin` records the minimum of all elements at or below that position in the stack at the time it was pushed. When pushing `x`, compute `newMin = Math.Min(x, currentTopMin)` (or just `x` if the stack was empty) and push the pair `(x, newMin)`. `GetMin()` simply reads the `currentMin` field of the top pair — no scanning required. `Pop()` removes the top pair entirely, which automatically "restores" the previous minimum because the new top's stored `currentMin` reflects the minimum among the remaining elements.

```csharp
using System;
using System.Collections.Generic;

public class MinStack
{
    private class Pair
    {
        public int Value;
        public int CurrentMin;
        public Pair(int value, int currentMin)
        {
            Value = value;
            CurrentMin = currentMin;
        }
    }

    private readonly Stack<Pair> stack = new Stack<Pair>();

    public bool IsEmpty() => stack.Count == 0;

    public void Push(int x)
    {
        int newMin = IsEmpty() ? x : Math.Min(x, stack.Peek().CurrentMin);
        stack.Push(new Pair(x, newMin));
    }

    public int Pop()
    {
        if (IsEmpty())
            throw new InvalidOperationException("Stack is empty");
        return stack.Pop().Value;
    }

    public int Top()
    {
        if (IsEmpty())
            throw new InvalidOperationException("Stack is empty");
        return stack.Peek().Value;
    }

    public int GetMin()
    {
        if (IsEmpty())
            throw new InvalidOperationException("Stack is empty");
        return stack.Peek().CurrentMin;
    }
}
```

Time Complexity: `Push`, `Pop`, `Top`, `GetMin` — all O(1).
Space Complexity: O(n) — each element stores one extra integer (`CurrentMin`) alongside its value, so O(2n) = O(n).

**Explanation (dry run):** Sequence `Push(3), Push(5), GetMin(), Push(2), Push(1), GetMin(), Pop(), GetMin()`:
- `Push(3)`: stack empty, `newMin = 3` → push `(3, 3)`. Stack (bottom→top): `(3,3)`.
- `Push(5)`: top's min is `3`, `newMin = min(5,3) = 3` → push `(5, 3)`. Stack: `(3,3) (5,3)`.
- `GetMin()`: read top's `CurrentMin` → `3`. Correct, since {3,5} min is 3.
- `Push(2)`: top's min is `3`, `newMin = min(2,3) = 2` → push `(2, 2)`. Stack: `(3,3) (5,3) (2,2)`.
- `Push(1)`: top's min is `2`, `newMin = min(1,2) = 1` → push `(1, 1)`. Stack: `(3,3) (5,3) (2,2) (1,1)`.
- `GetMin()`: read top's `CurrentMin` → `1`. Correct, since {3,5,2,1} min is 1.
- `Pop()`: removes `(1,1)`, returns value `1`. Stack: `(3,3) (5,3) (2,2)`.
- `GetMin()`: read new top's `CurrentMin` → `2`. Correct, since remaining {3,5,2} min is 2.

Each pair carries the minimum "as of that point in the stack's history," so popping an element never requires recomputation — the newly exposed top already has the correct minimum for the remaining elements baked in, which is what keeps `GetMin` at O(1) instead of needing an O(n) scan.
