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

**Logic (Steps):**
1. Traverse the list from `head`, appending each node's `val` into a `List<int> values`.
2. Reset `curr` back to `head`, and set an index `i` to `values.Count - 1` (pointing at the last stored value).
3. Traverse the list again; at each node, overwrite `curr.val` with `values[i]`, then decrement `i` and advance `curr`.
4. Return the original `head` — the node objects and pointers are unchanged, only the values are now in reverse order.

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

**Walkthrough:** For `1 -> 2 -> 3 -> 4 -> 5 -> NULL`, the first pass collects `values = [1, 2, 3, 4, 5]`. On the second pass, `i` starts at `4`: node 1 (originally `1`) gets `values[4] = 5`; `i` becomes `3`, node 2 gets `values[3] = 4`; `i` becomes `2`, node 3 gets `values[2] = 3`; `i` becomes `1`, node 4 gets `values[1] = 2`; `i` becomes `0`, node 5 gets `values[0] = 1`. The list (same node objects, same `head` reference) now reads `5 -> 4 -> 3 -> 2 -> 1 -> NULL`, matching the expected output.

---

**Optimized Approach:** Use three pointers (`prev`, `curr`, `next`) and walk through the list once, flipping each node's `next` pointer to point backward instead of forward. No extra data structure is needed — only pointer manipulation, giving O(1) space. A recursive version achieves the same result by recursing to the end of the list first, then re-linking `next.next = curr` and `curr.next = null` on the way back up the call stack (the recursion stack itself uses O(n) space, so the iterative version is preferred when O(1) space is required).

**Logic (Steps):**
1. **Iterative:** Initialize `prev = null` and `curr = head`.
2. Inside the loop, save `curr.next` into a temporary `next` before it gets overwritten.
3. Point `curr.next` backward to `prev` (this is the actual reversal of the link).
4. Advance both pointers: `prev = curr`, then `curr = next`. Repeat until `curr` is `null`.
5. Return `prev`, which now sits on the last node visited — the new head.
6. **Recursive:** Recurse down to the base case (`head == null || head.next == null`), then on the way back up set `head.next.next = head` and `head.next = null` to flip each link, propagating the deepest node up as `newHead`.

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

**Walkthrough:** Trace the iterative version on `1 -> 2 -> 3 -> NULL` (a shorter slice of the example list). Start: `prev = null`, `curr = 1`. Iteration 1: `next = 2`, `curr.next = null` (node 1 now points to null), `prev = 1`, `curr = 2`. Iteration 2: `next = 3`, `curr.next = 1` (node 2 points back to node 1), `prev = 2`, `curr = 3`. Iteration 3: `next = null`, `curr.next = 2` (node 3 points back to node 2), `prev = 3`, `curr = null` — loop ends. Return `prev = 3`, and following `next` pointers from there gives `3 -> 2 -> 1 -> NULL`, the reversed list. Applied to the full example `1 -> 2 -> 3 -> 4 -> 5`, the same process yields `5 -> 4 -> 3 -> 2 -> 1 -> NULL`, matching the expected output.

---

### 2. Find the Middle of a Linked List

**Problem Statement:** Given the head of a singly linked list, return the middle node of the list. If there are two middle nodes (i.e., the list has an even number of nodes), return the second of the two middle nodes.

**Example:**
- Input: `1 -> 2 -> 3 -> 4 -> 5 -> NULL`
- Output: `3`
- Explanation: The list has 5 nodes, so the exact middle node, `3`, is returned. For an even-length list such as `1 -> 2 -> 3 -> 4`, the output would be `3` (the second of the two middle nodes `2` and `3`).

**Brute Force Approach:** First traverse the entire list once just to count the total number of nodes, `n`. Then traverse the list again from the head, stopping after `n / 2` steps to land on the middle node. This requires two passes over the list.

**Logic (Steps):**
1. Traverse the whole list once with `curr`, incrementing `count` for every node, to find the total node count `n`.
2. Compute `steps = count / 2` — the number of hops from `head` needed to reach the middle (integer division naturally lands on the second middle node for even-length lists).
3. Reset `curr` to `head` and advance it `steps` times using a `for` loop.
4. Return `curr`, which is now sitting on the middle node.

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

**Walkthrough:** For `1 -> 2 -> 3 -> 4 -> 5 -> NULL`, the first pass counts `count = 5`. Then `steps = 5 / 2 = 2`. Resetting `curr` to `head` (node `1`) and advancing 2 times: after 1 step `curr = 2`, after 2 steps `curr = 3`. The function returns node `3`, matching the expected output.

---

**Optimized Approach:** Use the slow/fast (tortoise-hare) pointer technique. Both pointers start at `head`. On each step, `slow` moves one node forward while `fast` moves two nodes forward. Since `fast` moves twice as fast as `slow`, by the time `fast` reaches the end of the list, `slow` will be exactly at the middle — all done in a single pass.

**Logic (Steps):**
1. Initialize both `slow` and `fast` to `head`.
2. Loop while `fast != null && fast.next != null` (guards against null-reference when `fast` has no next node to jump two ahead).
3. On each iteration, move `slow` one node forward (`slow = slow.next`) and `fast` two nodes forward (`fast = fast.next.next`).
4. When the loop exits, `fast` has reached (or passed) the end, so `slow` is exactly at the middle; return `slow`.

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

**Walkthrough:** For `1 -> 2 -> 3 -> 4 -> 5 -> NULL`: start `slow = 1`, `fast = 1`. Iteration 1: `slow = 2`, `fast = 3`. Iteration 2: `slow = 3`, `fast = 5`. Now `fast.next` is `null`, so the loop stops. Return `slow = 3`, matching the expected output — reached in a single pass instead of two.

---

### 3. Detect a Cycle in a Linked List

**Problem Statement:** Given the head of a singly linked list, determine whether the list contains a cycle, i.e., whether some node's `next` pointer eventually loops back to a previously visited node instead of terminating in `null`.

**Example:**
- Input: `1 -> 2 -> 3 -> 4 -> 5`, where node `5`'s `next` points back to node `3`
- Output: `true`
- Explanation: Starting from `1`, following `next` pointers repeatedly cycles through `3 -> 4 -> 5 -> 3 -> 4 -> 5 -> ...` forever, so the list contains a cycle.

**Brute Force Approach:** Traverse the list while storing each visited node's reference in a `HashSet`. Before visiting a node, check whether it is already present in the set — if it is, a cycle exists. If traversal reaches `null` without hitting a repeat, there is no cycle.

**Logic (Steps):**
1. Create an empty `HashSet<ListNode> visited` and set `curr = head`.
2. While `curr != null`, check whether `visited` already contains `curr`.
3. If it does, `curr` has been visited before, meaning a `next` pointer looped back — return `true`.
4. Otherwise, add `curr` to `visited` and advance `curr = curr.next`.
5. If the loop exits normally (`curr` becomes `null`), the end of the list was reached without a repeat — return `false`.

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

**Walkthrough:** For `1 -> 2 -> 3 -> 4 -> 5` with node `5.next` pointing back to node `3`: `curr` visits `1, 2, 3, 4, 5` in turn, adding each to `visited` (none were already present). From node `5`, `curr` moves to node `3` again — this time `visited.Contains(3)` is `true`, so the function returns `true`, matching the expected output.

---

**Optimized Approach:** Floyd's Cycle Detection (slow/fast pointers). Both pointers start at `head`; `slow` advances one node per step, `fast` advances two nodes per step. If there is no cycle, `fast` reaches `null` and the loop ends normally. If there is a cycle, `fast` (moving faster) eventually laps `slow` inside the loop and the two pointers become equal — confirming a cycle, all in O(1) extra space.

**Logic (Steps):**
1. Initialize `slow = head` and `fast = head`.
2. Loop while `fast != null && fast.next != null`.
3. Advance `slow` by one node and `fast` by two nodes each iteration.
4. After each move, check if `slow == fast`; if so, `fast` has lapped `slow` inside a cycle — return `true`.
5. If the loop exits because `fast` (or `fast.next`) hit `null`, the list has an end — return `false`.

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

**Walkthrough:** Using the same list `1 -> 2 -> 3 -> 4 -> 5` with node `5.next -> 3`: start `slow = 1`, `fast = 1`. Iteration 1: `slow = 2`, `fast = 3`. Iteration 2: `slow = 3`, `fast = 5`. Iteration 3: `slow = 4`, `fast` moves `5 -> 3 -> 4`, so `fast = 4`. Now `slow == fast` (both at node `4`), so the function returns `true`, matching the expected output — without any auxiliary storage.

---

### 4. Find the Starting Point of the Cycle in a Linked List

**Problem Statement:** Given the head of a singly linked list that may contain a cycle, return the node where the cycle begins. If there is no cycle, return `null`.

**Example:**
- Input: `1 -> 2 -> 3 -> 4 -> 5`, where node `5`'s `next` points back to node `3`
- Output: node `3`
- Explanation: The cycle loops through `3 -> 4 -> 5 -> 3 -> ...`, so node `3` is where the cycle starts.

**Brute Force Approach:** Traverse the list while storing each visited node reference in a `HashSet`. The first node found to already be in the set is the start of the cycle (since it is the first node visited twice). If the traversal reaches `null`, there is no cycle.

**Logic (Steps):**
1. Create an empty `HashSet<ListNode> visited` and set `curr = head`.
2. While `curr != null`, check whether `curr` is already in `visited`.
3. If it is, `curr` is the first node revisited, i.e. the cycle's start — return `curr`.
4. Otherwise add `curr` to `visited` and move `curr = curr.next`.
5. If the loop ends by reaching `null`, there is no cycle — return `null`.

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

**Walkthrough:** For `1 -> 2 -> 3 -> 4 -> 5` with node `5.next -> 3`: `curr` visits `1, 2, 3, 4, 5`, adding each to `visited`. From `5`, `curr` moves to `3` again — `visited.Contains(3)` is `true`, so the function returns node `3`, matching the expected output.

---

**Optimized Approach:** Extend Floyd's Cycle Detection. First, run slow/fast pointers until they meet inside the cycle (as in problem 3). Then reset one pointer (say `slow`) back to `head`, keep the other (`fast`) at the meeting point, and move both one step at a time. The node where they meet again is the start of the cycle. This works purely with pointers, in O(1) space.

**Logic (Steps):**
1. Phase 1: initialize `slow = head`, `fast = head`, `hasCycle = false`, and run the standard Floyd loop (`slow` +1, `fast` +2) until either `slow == fast` (cycle found, break) or `fast`/`fast.next` hits `null` (no cycle).
2. If no cycle was found, return `null` immediately.
3. Phase 2: reset `slow` back to `head`, keep `fast` at the meeting point found in phase 1.
4. Move both `slow` and `fast` one step at a time (`slow = slow.next`, `fast = fast.next`) until they become equal.
5. Return `slow` (== `fast`), which is the start of the cycle.

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

**Walkthrough:** For `1 -> 2 -> 3 -> 4 -> 5` with node `5.next -> 3` (cycle `3 -> 4 -> 5 -> 3 -> ...`). Phase 1: `slow = 1`, `fast = 1` → step 1: `slow = 2`, `fast = 3` → step 2: `slow = 3`, `fast = 5` → step 3: `slow = 4`, `fast` goes `5 -> 3 -> 4`, so `fast = 4`; `slow == fast` at node `4`, cycle confirmed, `hasCycle = true`. Phase 2: reset `slow = head = 1`, keep `fast = 4`. Step 1: `slow = 2`, `fast = 5`. Step 2: `slow = 3`, `fast = 3` — equal at node `3`. Return node `3`, matching the expected output.

---

### 5. Length of the Loop in a Linked List

**Problem Statement:** Given the head of a singly linked list that may contain a cycle, find the number of nodes present in the loop (cycle). If there is no cycle, return `0`.

**Example:**
- Input: `1 -> 2 -> 3 -> 4 -> 5`, where node `5`'s `next` points back to node `3`
- Output: `3`
- Explanation: The loop consists of nodes `3 -> 4 -> 5 -> back to 3`, which is 3 nodes long.

**Brute Force Approach:** Traverse the list while storing each visited node along with the step number at which it was visited in a `Dictionary<ListNode, int>`. The moment a node is encountered that is already in the dictionary, the loop length is the current step number minus the step number recorded when that node was first visited.

**Logic (Steps):**
1. Create an empty `Dictionary<ListNode, int> visitedAtStep`, set `curr = head` and `step = 0`.
2. While `curr != null`, check whether `curr` is already a key in `visitedAtStep`.
3. If it is, the loop length is `step - visitedAtStep[curr]` (the number of steps taken between the first and second visit to this node) — return it.
4. Otherwise, record `visitedAtStep[curr] = step`, then advance `curr = curr.next` and `step++`.
5. If the loop exits by reaching `null`, there is no loop — return `0`.

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

**Walkthrough:** For `1 -> 2 -> 3 -> 4 -> 5` with node `5.next -> 3`: nodes `1, 2, 3, 4, 5` get recorded at steps `0, 1, 2, 3, 4` respectively. At `step = 5`, `curr` is back at node `3`, which is already a key with value `2`. The function returns `5 - 2 = 3`, matching the expected output.

---

**Optimized Approach:** Use Floyd's slow/fast pointers to first detect a meeting point inside the cycle (same as problem 3). Once `slow` and `fast` meet, keep one pointer fixed and move the other one step at a time, counting steps, until it comes back around to the fixed pointer — that count is the length of the loop.

**Logic (Steps):**
1. Initialize `slow = head`, `fast = head`, and run the Floyd loop, advancing `slow` by one and `fast` by two each iteration.
2. When `slow == fast`, a cycle has been found; start measuring its length from this meeting point.
3. Set `length = 1` and a temp pointer `temp = slow.next`, then advance `temp` one node at a time, incrementing `length` each time, until `temp` comes back around to `slow`.
4. Return `length`. If the main loop ends without `slow` and `fast` ever meeting (i.e. `fast` hits `null`), there is no loop — return `0`.

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

**Walkthrough:** For `1 -> 2 -> 3 -> 4 -> 5` with node `5.next -> 3`: `slow`/`fast` run as before and meet at node `4` after 3 iterations (as traced in problem 4). Measuring from there: `length = 1`, `temp = slow.next = 5`. `temp != slow` (`5 != 4`), so `length = 2`, `temp = temp.next = 3`. `temp != slow` (`3 != 4`), so `length = 3`, `temp = temp.next = 4`. Now `temp == slow`, loop stops. Return `length = 3`, matching the expected output.

---

### 6. Check if a Linked List is a Palindrome

**Problem Statement:** Given the head of a singly linked list, determine whether the sequence of values it represents reads the same forwards and backwards.

**Example:**
- Input: `1 -> 2 -> 3 -> 2 -> 1 -> NULL`
- Output: `true`
- Explanation: Reading the values front-to-back gives `1, 2, 3, 2, 1`, which is identical to reading them back-to-front, so the list is a palindrome.

**Brute Force Approach:** Traverse the list once, copying every value into an array (or `List<int>`). Then use two index pointers, one starting at the front of the array and one at the back, moving toward each other and comparing values at each step. If any pair of values differs, the list is not a palindrome.

**Logic (Steps):**
1. Traverse the list from `head`, appending each node's `val` into a `List<int> values`.
2. Initialize `left = 0` and `right = values.Count - 1`.
3. While `left < right`, compare `values[left]` and `values[right]`; if they differ, return `false` immediately.
4. Otherwise increment `left` and decrement `right`, continuing to close in toward the center.
5. If the loop finishes without a mismatch, return `true`.

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

**Walkthrough:** For `1 -> 2 -> 3 -> 2 -> 1 -> NULL`, the first pass builds `values = [1, 2, 3, 2, 1]`. Comparing: `left = 0, right = 4` → `values[0] = 1`, `values[4] = 1`, match, so `left = 1, right = 3`. Next: `values[1] = 2`, `values[3] = 2`, match, so `left = 2, right = 2`, loop ends (`left < right` is false). No mismatch was found, so the function returns `true`, matching the expected output.

---

**Optimized Approach:** Find the middle of the list using slow/fast pointers, reverse the second half of the list in place, then compare the first half and the reversed second half node by node. Optionally, reverse the second half back afterward to restore the original list structure. This uses only pointer manipulation, giving O(1) extra space.

**Logic (Steps):**
1. Handle the trivial case: if the list is empty or has one node, it is trivially a palindrome.
2. Find the middle using slow/fast pointers (`slow` advances one node, `fast` advances two, stopping when `fast` can no longer move two nodes ahead) — `slow` ends up on the last node of the first half.
3. Reverse the second half of the list starting at `slow.next`, producing `secondHalfHead`.
4. Walk `firstPtr` from `head` and `secondPtr` from `secondHalfHead` in lockstep, comparing values; on any mismatch set `isPalindrome = false` and stop.
5. Optionally reverse the second half back and reattach it to `slow.next`, restoring the list's original shape.
6. Return `isPalindrome`.

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

**Walkthrough:** For `1 -> 2 -> 3 -> 2 -> 1 -> NULL`:
- Step 1 — find the middle: `slow`/`fast` start at `1`. After one iteration, `slow = 2`, `fast = 3`. The loop condition `fast.next != null && fast.next.next != null` fails next (fast is at the last node `1`, `fast.next` is `null`), so `slow` stops at node `3` (the middle element, value `3`).
- Step 2 — reverse the second half starting at `slow.next` (the sublist `2 -> 1`): after reversal it becomes `1 -> 2`, and `secondHalfHead` points to this new sublist's head (value `1`).
- Step 3 — compare: `firstPtr` starts at head (`1`), `secondPtr` starts at `secondHalfHead` (`1`).
  - Compare `1 == 1` → match, advance both: `firstPtr = 2`, `secondPtr = 2`.
  - Compare `2 == 2` → match, advance both: `secondPtr` becomes `null`, loop ends.
  - No mismatches found, so `isPalindrome = true`.
- Step 4 — optionally reverse the second half back to `2 -> 1` and reattach it after node `3`, restoring the original list `1 -> 2 -> 3 -> 2 -> 1`.

The result is `true`, correctly identifying the list as a palindrome, matching the expected output, using only O(1) extra space since all work is done via pointer reversal rather than auxiliary arrays.
