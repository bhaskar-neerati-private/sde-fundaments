# 23. Remove Nth Node From End of List

**LeetCode:** [#19 - Remove Nth Node From End of List](https://leetcode.com/problems/remove-nth-node-from-end-of-list/) · **Topic:** [Linked List](../topics/06-linked-list.md) · **Difficulty:** Medium

## Problem statement

Given the head of a linked list, remove the `n`-th node **from the end** of the list, and return the head. Do it in **one pass** if possible.

**Example:**
```
Input: head = [1,2,3,4,5], n = 2
Output: [1,2,3,5]  (the 2nd node from the end, value 4, is removed)
```

## Applicable approaches

- **Brute Force — Two Pass** — count the list's length first, then walk to the right spot.
- **Optimal — One Pass, Two Pointers (n apart) + Dummy Head.** The standard, expected O(n) single-pass solution.

## Approach 1: Brute Force — Two Pass

### Intuition

"The n-th node from the end" is an awkward thing to locate directly in a structure you can only walk forward through — you don't know how many nodes total there are until you've reached the end. But if you *do* know the total length `L`, then "the n-th node from the end" translates directly into "the `(L - n)`-th node from the start" (0-indexed) — a position you *can* walk to directly, forward, once you know it. So: count the length first (one pass), then walk to that computed position and remove it (a second pass).

### Python code
```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

def removeNthFromEnd(head, n):
    length = 0
    node = head
    while node:
        length += 1
        node = node.next

    dummy = ListNode(0, head)
    current = dummy
    for _ in range(length - n):
        current = current.next

    current.next = current.next.next
    return dummy.next
```

### Line-by-line explanation

- First loop counts the list's length — necessary because there's no way to know "how far from the end" a position is without first knowing the total.
- `dummy = ListNode(0, head)` — handles the edge case where the node to remove is the head itself, per the topic overview's dummy-node pattern.
- `for _ in range(length - n): current = current.next` — walk `current` to sit *just before* the node that needs removing (position `length - n - 1` from the dummy, i.e. `length - n` steps from the dummy) — we need the *predecessor*, not the target node itself, since removing a node from a singly linked list requires modifying its predecessor's `.next`.
- `current.next = current.next.next` — skip over (remove) the target node by re-linking around it.

### Time & space complexity

- **Time: O(L)** — two passes, but both linear, so still O(n) overall in the usual sense (two passes of O(n) is still O(n), just with a bigger constant) — however, this doesn't satisfy the problem's explicit "one pass if possible" challenge, which is a meaningful distinction worth taking seriously even though the asymptotic class is the same.
- **Space: O(1)**.

---

## Approach 2: Optimal — One Pass, Two Pointers (n apart)

### Intuition

Instead of counting the length first and then computing a position, we can maintain a **fixed gap of `n`** between two pointers from the very start. If the second pointer (`fast`) starts `n` steps ahead of the first (`slow`), then the *distance between them stays exactly `n` nodes forever*, no matter how far both advance together — and specifically, when `fast` reaches the end of the list, `slow` must be exactly `n` nodes from the end, by the definition of a fixed gap. This sidesteps ever needing to know the total length explicitly — the gap itself encodes the "distance from the end" information, updated implicitly as both pointers move.

### Algorithm

1. Create a `dummy` node pointing to `head` (handles removing the head itself cleanly, same as the topic overview's pattern).
2. Set `slow = dummy`, `fast = dummy`.
3. Advance `fast` **n+1** steps ahead of `slow` first (n+1, not n — this is deliberate: we need `slow` to end up *just before* the node to remove, not exactly *on* it, since removal requires access to the predecessor).
4. Then advance `slow` and `fast` together, one step at a time, until `fast` reaches `None` (the end).
5. At this point, `slow` is sitting on the node **just before** the one to remove. Unlink it: `slow.next = slow.next.next`.
6. Return `dummy.next`.

### Python code
```python
def removeNthFromEnd(head, n):
    dummy = ListNode(0, head)
    slow, fast = dummy, dummy

    for _ in range(n + 1):
        fast = fast.next

    while fast:
        slow = slow.next
        fast = fast.next

    slow.next = slow.next.next

    return dummy.next
```

### Line-by-line explanation

- `dummy = ListNode(0, head)` — lets us uniformly handle removing the first real node without special-casing it separately.
- `slow, fast = dummy, dummy` — both start at the dummy, establishing the baseline before the gap is introduced.
- `for _ in range(n + 1): fast = fast.next` — move `fast` ahead by `n + 1` nodes. **The `+1` is the detail most likely to be gotten wrong**: it establishes a gap of `n + 1` (not `n`) between `slow` and `fast`, so that when `fast` eventually falls off the end, `slow` lands one position *before* the target — exactly where we need to be to perform the removal.
- `while fast: slow = slow.next; fast = fast.next` — advance both pointers together, one step at a time, preserving that fixed gap, until `fast` falls off the end (`None`). At that moment, `slow` is exactly `n + 1` positions behind where `fast` ended up (one past the last node) — which works out to `slow` sitting on the node **immediately before** the n-th-from-end node.
- `slow.next = slow.next.next` — unlink the target node by pointing `slow` directly at the node *after* it, skipping the target entirely.
- `return dummy.next` — the real head (possibly changed, if the removed node was the original head).

### Dry run

`head = [1,2,3,4,5]`, `n = 2` (remove the 2nd node from the end, value `4`)

`dummy -> 1 -> 2 -> 3 -> 4 -> 5 -> None`

**Advance fast by n+1=3 steps** (starting at dummy): `dummy → 1 → 2 → 3`. So `fast` is now at node `3`.

**Advance both together until fast is None:**
| slow (before) | fast (before) | slow after | fast after |
|---|---|---|---|
| dummy | 3 | 1 | 4 |
| 1 | 4 | 2 | 5 |
| 2 | 5 | 3 | None |

Loop stops (`fast` is `None`). `slow = 3` (the node with value 3).

`slow.next = slow.next.next` → node `3`'s next was node `4`; `slow.next.next` is node `4`'s next, which is node `5` → so `node(3).next = node(5)`, skipping over node `4` entirely.

Final list, from `dummy.next`: `1 -> 2 -> 3 -> 5 -> None` ✅ matches expected `[1,2,3,5]`.

### Time & space complexity

- **Time: O(L)** — a single pass through the list (the initial `n+1`-step advance plus the combined walk together still totals at most `L + 1` node visits) — this genuinely satisfies the problem's "one pass" challenge, unlike Approach 1.
- **Space: O(1)**.

---

## Common mistakes & misconceptions

1. **Advancing `fast` by `n` steps instead of `n + 1`.** This is the single most common bug: with only `n` steps of gap, `slow` ends up landing *on* the target node rather than just before it, leaving no reference to the predecessor needed to actually remove it.
2. **Forgetting the dummy node, and special-casing "removing the head" separately.** Without a dummy, removing the head (when `n` equals the list's length) requires different code from removing any other node (since there's no predecessor to relink) — the dummy node uniformly sidesteps this by ensuring every real node always has *some* predecessor to work with.
3. **Advancing `fast` before creating the loop, then reusing the same loop structure for both the setup and the main walk without being careful about the exact starting gap.** It's worth explicitly separating "establish the gap" from "walk together" as two distinct steps (as the code above does), rather than trying to cleverly merge them into one loop, since the off-by-one risk is high.
4. **Assuming this problem needs the list's length known in advance.** The two-pointer version specifically demonstrates it doesn't — the fixed-gap trick extracts "distance from the end" information without ever counting the list, which is the core insight worth taking away, applicable to size-relative-to-the-end problems beyond just this one.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Two Pass (count then walk) | O(L) | O(1) | Simple, correct, but does two separate passes, not meeting the "one pass" challenge. |
| One Pass, Two Pointers (n apart) | O(L) | O(1) | The standard, single-pass, expected solution. |

**Key takeaway:** "the k-th node from the end" is a classic signal for **two pointers with a fixed gap** — advance one pointer `k` (or `k+1`, depending on whether you need the target itself or its predecessor) steps ahead first, then move both together; when the lead pointer reaches the end, the trailing pointer is exactly at the corresponding position from the end. This avoids ever needing to know the list's length ahead of time.
