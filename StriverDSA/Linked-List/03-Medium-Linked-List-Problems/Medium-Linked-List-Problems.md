# Linked List — Medium Linked List Problems

All problems below assume the following standard singly linked list node definition:

```csharp
public class ListNode {
    public int val;
    public ListNode next;
    public ListNode(int val) { this.val = val; this.next = null; }
}
```

---

### 1. Reverse a Singly Linked List (iterative and recursive)

**Problem Statement:** Given the head of a singly linked list, reverse the list and return the new head, i.e., the node that was previously the tail becomes the head, and all `next` pointers are flipped so the list is traversed in the opposite order.

**Example:**
- Input: `1 -> 2 -> 3 -> 4 -> 5 -> NULL`
- Output: `5 -> 4 -> 3 -> 2 -> 1 -> NULL`
- Explanation: Every node's `next` pointer now points to the node that originally preceded it, and the original head (`1`) becomes the new tail.

**Brute Force Approach:** Traverse the list once, pushing every node's value into an array (or a stack). Then traverse the original list again from the head, overwriting each node's value with values popped from the stack (or read from the array in reverse order). This reverses the *data* without touching any pointers, using O(n) extra space.

```csharp
public ListNode ReverseListBruteForce(ListNode head) {
    if (head == null) return null;

    List<int> values = new List<int>();
    ListNode curr = head;
    while (curr != null) {
        values.Add(curr.val);
        curr = curr.next;
    }

    curr = head;
    int i = values.Count - 1;
    while (curr != null) {
        curr.val = values[i];
        i--;
        curr = curr.next;
    }

    return head;
}
```

Time Complexity: O(n) — two linear passes over the list.
Space Complexity: O(n) — an auxiliary array/stack stores all node values.

**Optimized Approach:** Use three pointers (`prev`, `curr`, `next`) and walk through the list once, flipping each node's `next` pointer to point backward instead of forward. No extra data structure is needed — only pointer manipulation, giving O(1) space. A recursive version achieves the same result by recursing to the end of the list first, then re-linking `next.next = curr` and `curr.next = null` on the way back up the call stack (the recursion stack itself uses O(n) space, so the iterative version is preferred when O(1) space is required).

```csharp
// Iterative — O(1) space
public ListNode ReverseListIterative(ListNode head) {
    ListNode prev = null;
    ListNode curr = head;

    while (curr != null) {
        ListNode next = curr.next; // store the next node before overwriting
        curr.next = prev;          // reverse the link
        prev = curr;                // advance prev
        curr = next;                 // advance curr
    }

    return prev; // prev is the new head
}

// Recursive — O(n) recursion stack space
public ListNode ReverseListRecursive(ListNode head) {
    // base case: empty list or single node is already "reversed"
    if (head == null || head.next == null) return head;

    ListNode newHead = ReverseListRecursive(head.next);

    // head.next is the last node of the reversed sub-list;
    // make it point back to head, then detach head's old forward link
    head.next.next = head;
    head.next = null;

    return newHead;
}
```

Time Complexity: O(n), Space Complexity: O(1) for the iterative version (O(n) call-stack space for the recursive version).

---

### 2. Find the Middle of a Linked List

**Problem Statement:** Given the head of a singly linked list, return the middle node of the list. If there are two middle nodes (i.e., the list has an even number of nodes), return the second of the two middle nodes.

**Example:**
- Input: `1 -> 2 -> 3 -> 4 -> 5 -> NULL`
- Output: `3`
- Explanation: The list has 5 nodes, so the exact middle node, `3`, is returned. For an even-length list such as `1 -> 2 -> 3 -> 4`, the output would be `3` (the second of the two middle nodes `2` and `3`).

**Brute Force Approach:** First traverse the entire list once just to count the total number of nodes, `n`. Then traverse the list again from the head, stopping after `n / 2` steps to land on the middle node. This requires two passes over the list.

```csharp
public ListNode MiddleNodeBruteForce(ListNode head) {
    int count = 0;
    ListNode curr = head;
    while (curr != null) {
        count++;
        curr = curr.next;
    }

    int steps = count / 2;
    curr = head;
    for (int i = 0; i < steps; i++) {
        curr = curr.next;
    }

    return curr;
}
```

Time Complexity: O(n) — two passes (one to count, one to walk to the middle).
Space Complexity: O(1) — only a counter and a pointer are used.

**Optimized Approach:** Use the slow/fast (tortoise-hare) pointer technique. Both pointers start at `head`. On each step, `slow` moves one node forward while `fast` moves two nodes forward. Since `fast` moves twice as fast as `slow`, by the time `fast` reaches the end of the list, `slow` will be exactly at the middle — all done in a single pass.

```csharp
public ListNode MiddleNodeOptimized(ListNode head) {
    ListNode slow = head;
    ListNode fast = head;

    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }

    return slow; // slow is now at the middle node
}
```

Time Complexity: O(n), Space Complexity: O(1).

---

### 3. Detect a Cycle in a Linked List

**Problem Statement:** Given the head of a singly linked list, determine whether the list contains a cycle, i.e., whether some node's `next` pointer eventually loops back to a previously visited node instead of terminating in `null`.

**Example:**
- Input: `1 -> 2 -> 3 -> 4 -> 5`, where node `5`'s `next` points back to node `3`
- Output: `true`
- Explanation: Starting from `1`, following `next` pointers repeatedly cycles through `3 -> 4 -> 5 -> 3 -> 4 -> 5 -> ...` forever, so the list contains a cycle.

**Brute Force Approach:** Traverse the list while storing each visited node's reference in a `HashSet`. Before visiting a node, check whether it is already present in the set — if it is, a cycle exists. If traversal reaches `null` without hitting a repeat, there is no cycle.

```csharp
public bool HasCycleBruteForce(ListNode head) {
    HashSet<ListNode> visited = new HashSet<ListNode>();
    ListNode curr = head;

    while (curr != null) {
        if (visited.Contains(curr)) {
            return true;
        }
        visited.Add(curr);
        curr = curr.next;
    }

    return false;
}
```

Time Complexity: O(n) — each node is visited at most once.
Space Complexity: O(n) — the hash set can hold up to all n node references.

**Optimized Approach:** Floyd's Cycle Detection (slow/fast pointers). Both pointers start at `head`; `slow` advances one node per step, `fast` advances two nodes per step. If there is no cycle, `fast` reaches `null` and the loop ends normally. If there is a cycle, `fast` (moving faster) eventually laps `slow` inside the loop and the two pointers become equal — confirming a cycle, all in O(1) extra space.

```csharp
public bool HasCycleOptimized(ListNode head) {
    ListNode slow = head;
    ListNode fast = head;

    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;

        if (slow == fast) {
            return true; // pointers met inside the cycle
        }
    }

    return false; // fast reached the end, no cycle
}
```

Time Complexity: O(n), Space Complexity: O(1).

---

### 4. Find the Starting Point of the Cycle in a Linked List

**Problem Statement:** Given the head of a singly linked list that may contain a cycle, return the node where the cycle begins. If there is no cycle, return `null`.

**Example:**
- Input: `1 -> 2 -> 3 -> 4 -> 5`, where node `5`'s `next` points back to node `3`
- Output: node `3`
- Explanation: The cycle loops through `3 -> 4 -> 5 -> 3 -> ...`, so node `3` is where the cycle starts.

**Brute Force Approach:** Traverse the list while storing each visited node reference in a `HashSet`. The first node found to already be in the set is the start of the cycle (since it is the first node visited twice). If the traversal reaches `null`, there is no cycle.

```csharp
public ListNode DetectCycleStartBruteForce(ListNode head) {
    HashSet<ListNode> visited = new HashSet<ListNode>();
    ListNode curr = head;

    while (curr != null) {
        if (visited.Contains(curr)) {
            return curr; // first repeated node = start of cycle
        }
        visited.Add(curr);
        curr = curr.next;
    }

    return null; // no cycle
}
```

Time Complexity: O(n).
Space Complexity: O(n) — hash set of visited nodes.

**Optimized Approach:** Extend Floyd's Cycle Detection. First, run slow/fast pointers until they meet inside the cycle (as in problem 3). Then reset one pointer (say `slow`) back to `head`, keep the other (`fast`) at the meeting point, and move both one step at a time. The node where they meet again is the start of the cycle. This works purely with pointers, in O(1) space.

```csharp
public ListNode DetectCycleStartOptimized(ListNode head) {
    ListNode slow = head;
    ListNode fast = head;
    bool hasCycle = false;

    // Phase 1: detect whether a cycle exists and find the meeting point
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) {
            hasCycle = true;
            break;
        }
    }

    if (!hasCycle) return null;

    // Phase 2: find the start of the cycle
    slow = head;
    while (slow != fast) {
        slow = slow.next;
        fast = fast.next;
    }

    return slow; // start of the cycle
}
```

Time Complexity: O(n), Space Complexity: O(1).

---

### 5. Length of the Loop in a Linked List

**Problem Statement:** Given the head of a singly linked list that may contain a cycle, find the number of nodes present in the loop (cycle). If there is no cycle, return `0`.

**Example:**
- Input: `1 -> 2 -> 3 -> 4 -> 5`, where node `5`'s `next` points back to node `3`
- Output: `3`
- Explanation: The loop consists of nodes `3 -> 4 -> 5 -> back to 3`, which is 3 nodes long.

**Brute Force Approach:** Traverse the list while storing each visited node along with the step number at which it was visited in a `Dictionary<ListNode, int>`. The moment a node is encountered that is already in the dictionary, the loop length is the current step number minus the step number recorded when that node was first visited.

```csharp
public int LengthOfLoopBruteForce(ListNode head) {
    Dictionary<ListNode, int> visitedAtStep = new Dictionary<ListNode, int>();
    ListNode curr = head;
    int step = 0;

    while (curr != null) {
        if (visitedAtStep.ContainsKey(curr)) {
            return step - visitedAtStep[curr];
        }
        visitedAtStep[curr] = step;
        curr = curr.next;
        step++;
    }

    return 0; // no loop
}
```

Time Complexity: O(n).
Space Complexity: O(n) — dictionary of visited nodes.

**Optimized Approach:** Use Floyd's slow/fast pointers to first detect a meeting point inside the cycle (same as problem 3). Once `slow` and `fast` meet, keep one pointer fixed and move the other one step at a time, counting steps, until it comes back around to the fixed pointer — that count is the length of the loop.

```csharp
public int LengthOfLoopOptimized(ListNode head) {
    ListNode slow = head;
    ListNode fast = head;

    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;

        if (slow == fast) {
            // Pointers met inside the cycle; now measure its length
            int length = 1;
            ListNode temp = slow.next;
            while (temp != slow) {
                length++;
                temp = temp.next;
            }
            return length;
        }
    }

    return 0; // no loop
}
```

Time Complexity: O(n), Space Complexity: O(1).

---

### 6. Check if a Linked List is a Palindrome

**Problem Statement:** Given the head of a singly linked list, determine whether the sequence of values it represents reads the same forwards and backwards.

**Example:**
- Input: `1 -> 2 -> 3 -> 2 -> 1 -> NULL`
- Output: `true`
- Explanation: Reading the values front-to-back gives `1, 2, 3, 2, 1`, which is identical to reading them back-to-front, so the list is a palindrome.

**Brute Force Approach:** Traverse the list once, copying every value into an array (or `List<int>`). Then use two index pointers, one starting at the front of the array and one at the back, moving toward each other and comparing values at each step. If any pair of values differs, the list is not a palindrome.

```csharp
public bool IsPalindromeBruteForce(ListNode head) {
    List<int> values = new List<int>();
    ListNode curr = head;
    while (curr != null) {
        values.Add(curr.val);
        curr = curr.next;
    }

    int left = 0;
    int right = values.Count - 1;
    while (left < right) {
        if (values[left] != values[right]) {
            return false;
        }
        left++;
        right--;
    }

    return true;
}
```

Time Complexity: O(n) — one pass to copy values, one pass to compare.
Space Complexity: O(n) — the array holding all node values.

**Optimized Approach:** Find the middle of the list using slow/fast pointers, reverse the second half of the list in place, then compare the first half and the reversed second half node by node. Optionally, reverse the second half back afterward to restore the original list structure. This uses only pointer manipulation, giving O(1) extra space.

```csharp
public bool IsPalindromeOptimized(ListNode head) {
    if (head == null || head.next == null) return true;

    // Step 1: find the middle of the list (slow ends at the middle)
    ListNode slow = head;
    ListNode fast = head;
    while (fast.next != null && fast.next.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }

    // Step 2: reverse the second half starting right after slow
    ListNode secondHalfHead = ReverseListIterative(slow.next);

    // Step 3: compare first half with reversed second half
    ListNode firstPtr = head;
    ListNode secondPtr = secondHalfHead;
    bool isPalindrome = true;
    while (secondPtr != null) {
        if (firstPtr.val != secondPtr.val) {
            isPalindrome = false;
            break;
        }
        firstPtr = firstPtr.next;
        secondPtr = secondPtr.next;
    }

    // Step 4 (optional): restore the list by reversing the second half back
    slow.next = ReverseListIterative(secondHalfHead);

    return isPalindrome;
}

// Helper (same as the optimized reversal from problem 1)
private ListNode ReverseListIterative(ListNode head) {
    ListNode prev = null;
    ListNode curr = head;
    while (curr != null) {
        ListNode next = curr.next;
        curr.next = prev;
        prev = curr;
        curr = next;
    }
    return prev;
}
```

Time Complexity: O(n), Space Complexity: O(1).

**Explanation:**

*Dry run of Floyd's Tortoise and Hare cycle detection, and finding the cycle start:*

Consider the list `1 -> 2 -> 3 -> 4 -> 5`, where node `5`'s `next` points back to node `3` (so the cycle is `3 -> 4 -> 5 -> 3 -> ...`).

Phase 1 — detecting the cycle:
- Start: `slow = 1`, `fast = 1`
- Step 1: `slow = 2`, `fast = 3`
- Step 2: `slow = 3`, `fast = 5`
- Step 3: `slow = 4`, `fast = 4` (fast went `5 -> 3 -> 4`) — `slow == fast`, cycle confirmed. They meet at node `4`.

Phase 2 — finding the start of the cycle:
- Reset `slow = head = 1`, keep `fast = 4` (the meeting point).
- Move both one step at a time:
  - Step 1: `slow = 2`, `fast = 5`
  - Step 2: `slow = 3`, `fast = 3` — `slow == fast` at node `3`, which is indeed the start of the cycle.

*Why this works (the distance argument):* Let `L` be the distance from the head to the start of the cycle, and `C` be the length of the cycle. Let `x` be the distance from the cycle's start to the meeting point (measured going forward around the cycle). When `slow` and `fast` first meet, `slow` has traveled `L + x` steps, and `fast`, moving twice as fast, has traveled `2(L + x)` steps. Since `fast` covers the same ground as `slow` plus some whole number of extra laps around the cycle, the extra distance `2(L + x) - (L + x) = L + x` must be a multiple of `C`, i.e., `L + x = k * C` for some integer `k`. Rearranging, `L = k * C - x`, which is the same as `L = (k - 1) * C + (C - x)`. Since `C - x` is exactly the remaining distance from the meeting point forward to the start of the cycle, this equation says: walking `L` steps from the head is equivalent to walking `(C - x)` steps from the meeting point (plus some whole number of full extra laps, which don't change the landing node). That is precisely why moving one pointer from `head` and another from the meeting point, both one step at a time, causes them to land on the cycle's start node at the same time.

*Dry run of the reverse-second-half technique for palindrome check on `1 -> 2 -> 3 -> 2 -> 1`:*

- Step 1 — find the middle: `slow`/`fast` start at `1`. After one iteration, `slow = 2`, `fast = 3`. The loop condition `fast.next != null && fast.next.next != null` fails next (fast is at the last node `1`, `fast.next` is `null`), so `slow` stops at node `3` (the middle element, value `3`).
- Step 2 — reverse the second half starting at `slow.next` (the sublist `2 -> 1`): after reversal it becomes `1 -> 2`, and `secondHalfHead` points to this new sublist's head (value `1`).
- Step 3 — compare: `firstPtr` starts at head (`1`), `secondPtr` starts at `secondHalfHead` (`1`).
  - Compare `1 == 1` → match, advance both: `firstPtr = 2`, `secondPtr = 2`.
  - Compare `2 == 2` → match, advance both: `secondPtr` becomes `null`, loop ends.
  - No mismatches found, so `isPalindrome = true`.
- Step 4 — optionally reverse the second half back to `2 -> 1` and reattach it after node `3`, restoring the original list `1 -> 2 -> 3 -> 2 -> 1`.

The result is `true`, correctly identifying the list as a palindrome, using only O(1) extra space since all work is done via pointer reversal rather than auxiliary arrays.
