# 22. Reorder List

**LeetCode:** [#143 - Reorder List](https://leetcode.com/problems/reorder-list/) · **Topic:** [Linked List](../topics/06-linked-list.md) · **Difficulty:** Medium

## Problem statement

Given the head of a singly linked list `L0 -> L1 -> ... -> Ln-1 -> Ln`, reorder it in-place to: `L0 -> Ln -> L1 -> Ln-1 -> L2 -> Ln-2 -> ...` (alternate first-from-start, first-from-end).

**Example:**
```
Input: 1 -> 2 -> 3 -> 4
Output: 1 -> 4 -> 2 -> 3
```

## Applicable approaches

- **Brute Force — Convert to array, rebuild by indexing from both ends.**
- **Optimal — Find Middle (fast/slow) + Reverse Second Half + Merge.** The standard, expected O(n) time, O(1) space solution.

## Approach 1: Brute Force — Array Indexing

### Intuition

The problem's real difficulty is that "alternate from the start and from the end" requires reading from *both directions* of the list — but a singly linked list can only be walked forward, never backward, so accessing "the last node" or "the second-to-last node" directly isn't possible without either extra memory or extra passes. Converting to an array sidesteps this entirely: arrays support O(1) access by index from either end, so we can dump every node **reference** (not just value) into an array, then rewire `.next` pointers by alternately picking from the front and back of that array.

### Python code
```python
def reorderList(head):
    nodes = []
    node = head
    while node:
        nodes.append(node)
        node = node.next

    left, right = 0, len(nodes) - 1
    while left < right:
        nodes[left].next = nodes[right]
        left += 1
        if left == right:
            break
        nodes[right].next = nodes[left]
        right -= 1

    nodes[left].next = None
```

### Time & space complexity

- **Time: O(n)** — one pass to build the array, one pass to rewire.
- **Space: O(n)** for the array of node references — this is the cost the optimal approach eliminates, by finding a way to access "from the end" without ever materializing an array.

*(Works, but uses O(n) extra space; the optimal approach below achieves the same result with O(1) extra space by combining three linked-list techniques you already know.)*

---

## Approach 2: Optimal — Find Middle + Reverse Second Half + Merge

### Intuition

This problem combines three linked-list skills you've already built into one, and recognizing that combination is the real skill being tested here, not any single new technique:
1. **Find the middle** of the list (fast/slow pointers, the exact technique from Linked List Cycle).
2. **Reverse the second half** (the exact iterative reversal from Reverse Linked List).
3. **Merge the two halves** by alternating nodes (similar spirit to Merge Two Sorted Lists, but interleaving unconditionally instead of comparing values).

Why this combination solves the "can't walk backward" problem: reversing the second half turns "the last node," "the second-to-last node," etc. into "the *first* node of the reversed second half," "the *second* node of the reversed second half," and so on — i.e., it converts backward-access into forward-access, by physically flipping the direction of that portion of the list. Once both halves can be walked forward, "alternate from start and end" becomes "alternate between two lists, both walked from their own front," which is a straightforward interleaving merge.

### Algorithm

1. **Find the middle:** use slow/fast pointers; when `fast` reaches the end, `slow` is at (or just before) the middle.
2. **Split** the list into two halves at `slow`, and **reverse** the second half.
3. **Merge** the two halves by alternating nodes: take one from the first half, then one from the reversed second half, repeating until the second half runs out.

### Python code
```python
def reorderList(head):
    # 1. find the middle
    slow, fast = head, head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

    # 2. split and reverse the second half
    second = slow.next
    slow.next = None  # cut the first half off from the second

    prev = None
    while second:
        next_node = second.next
        second.next = prev
        prev = second
        second = next_node
    second = prev  # head of the reversed second half

    # 3. merge the two halves, alternating
    first = head
    while second:
        first_next = first.next
        second_next = second.next

        first.next = second
        second.next = first_next

        first = first_next
        second = second_next
```

### Line-by-line explanation

- **Step 1 (find middle):** identical fast/slow pointer walk to Linked List Cycle's technique; when `fast` (moving 2 steps) reaches the end, `slow` (moving 1 step) has covered half the distance, landing at the middle.
- **Step 2 (split + reverse):**
  - `second = slow.next; slow.next = None` — cut the list into two independent halves: `head..slow` (first half) and `second..end` (second half). This split is essential — without physically severing the link, the "reverse" step below would end up looping the *whole* list, not just the intended second portion.
  - The `while second:` loop is the exact same iterative reversal from Reverse Linked List, applied to this second half — including the same "save `next_node` before overwriting" discipline.
  - `second = prev` — after reversing, `prev` holds the new head of the reversed second half (same as `reverseList`'s return value).
- **Step 3 (merge, alternating):**
  - `first_next = first.next; second_next = second.next` — save both "what comes next" references before we start rewiring, the same "save before overwrite" discipline from Reverse Linked List, now applied to *two* lists simultaneously.
  - `first.next = second` — attach the current second-half node right after the current first-half node.
  - `second.next = first_next` — attach the *next* first-half node right after that, continuing the alternation.
  - `first = first_next; second = second_next` — advance both pointers to continue the interleaving.
  - The loop naturally stops once `second` runs out — the second half is guaranteed (by construction from the middle-finding step) to be the same length as the first half or exactly one shorter, so `first` never runs out before `second` does.

### Dry run

`1 -> 2 -> 3 -> 4`

**Step 1:** slow/fast: start both at `1`. iter1: slow→2, fast→3(via 1→2→3). Check `fast(3) and fast.next(4)` true, continue: iter2: slow→3, fast→ (3.next.next = 4.next = None). Now `fast` is `None`, loop stops. `slow = 3`.

**Step 2:** `second = slow.next = 4`. `slow.next = None` → first half is now `1->2->3->None`. Reverse `second` (`4->None`, only one node) → reversed is just `4->None`. `second = 4`.

**Step 3:** `first = 1` (head of first half), `second = 4`.
- iter1: `first_next = 2`, `second_next = None`. `first.next = second` → `1.next = 4`. `second.next = first_next` → `4.next = 2`. `first = 2`, `second = None`.
- loop condition `while second:` → `second` is `None` → stop.

Final list, following from `head = 1`: `1 -> 4 -> 2 -> ...` — node 2's `.next` was never touched by step 3 and still points to `3` (from the original list, untouched since the cut only happened *after* `slow`), and node 3's `.next` was set to `None` back in step 2. So: `1 -> 4 -> 2 -> 3 -> None` ✅ matches expected output exactly.

### Time & space complexity

- **Time: O(n)** — each of the three steps (find middle, reverse, merge) is a single O(n) pass; O(n) + O(n) + O(n) = O(n) overall (three passes is still linear, just with a larger constant factor than a single pass).
- **Space: O(1)** — all done via pointer rewiring on the existing nodes, no new nodes or arrays allocated at any point.

---

## Common mistakes & misconceptions

1. **Forgetting to cut the list at the middle before reversing.** Without `slow.next = None`, the "reverse" step would walk into the first half too (since the list is still fully connected), producing an incorrect result — the split step isn't optional bookkeeping, it's what makes "reverse just the second half" actually mean what it says.
2. **Getting the fast/slow pointer starting position wrong for even-length lists**, causing the split point to be off by one node — as the topic overview warns generally, always verify against a concrete small example (like this 4-node dry run) rather than assuming the standard template's exact behavior without checking.
3. **Forgetting the "save before overwrite" discipline during the merge step.** Since this step rewires *two* lists simultaneously, it's easy to save only `first_next` and forget `second_next` (or vice versa) — both are needed, since both `first.next` and `second.next` get overwritten in the same iteration.
4. **Assuming this problem needs a fundamentally new technique.** It doesn't — it's a strong example of how complex-looking linked-list problems are frequently just a **composition of a few simpler techniques you already know**, applied in sequence. Recognizing "this is really three problems I've already solved, stitched together" — rather than searching for one unfamiliar trick — is the core skill this problem tests.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Array indexing | O(n) | O(n) | Simple, sidesteps the "can't walk backward" issue with extra memory. |
| Find Middle + Reverse + Merge | O(n) | O(1) | The standard, best-in-class solution — solves "can't walk backward" by physically reversing instead. |

**Key takeaway:** complex linked-list problems are very often just a **combination of a few simpler techniques you already know** (here: find-the-middle, reversal, and merging), applied in sequence — and the specific insight that makes the combination work is that reversing a portion of the list converts "access from the end" (impossible in a forward-only structure) into "access from the (new) front" (trivial).
