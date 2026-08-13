# 19. Reverse Linked List

**LeetCode:** [#206 - Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/) · **Topic:** [Linked List](../topics/06-linked-list.md) · **Difficulty:** Easy

## Problem statement

Given the `head` of a singly linked list, reverse the list, and return the new head.

**Example:**
```
Input: 1 -> 2 -> 3 -> 4 -> 5 -> None
Output: 5 -> 4 -> 3 -> 2 -> 1 -> None
```

## Applicable approaches

- **Brute Force — Convert to array, rebuild reversed.** Works, but defeats the point of practicing linked list manipulation.
- **Optimal — Iterative pointer reversal.** The standard, expected O(n) time, O(1) space solution.
- **Optimal (alternative) — Recursive.** Elegant, but O(n) space due to the call stack.

## Approach 1: Brute Force — Convert to Array

### Intuition

Walk the list, collect all values into a Python list (where random access and easy reversal are trivial), then rebuild a brand new linked list with the values in reverse order. This works because a linked list's *values*, once extracted, no longer have the "must walk node by node" restriction — but reconstructing brand-new nodes means we're not actually reversing the given list's structure, just producing a list with the same values in reverse order, which happens to satisfy the problem's output but sidesteps the pointer-manipulation skill the problem is testing.

### Python code
```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

def reverseList(head):
    values = []
    node = head
    while node:
        values.append(node.val)
        node = node.next

    dummy = ListNode()
    current = dummy
    for val in reversed(values):
        current.next = ListNode(val)
        current = current.next

    return dummy.next
```

### Time & space complexity

- **Time: O(n)** — one pass to collect values, one pass to rebuild.
- **Space: O(n)** for the `values` list and the newly allocated nodes — this is the real cost: the optimal approach below achieves the same result using the *existing* nodes, with zero new allocation.

*(This works, but it doesn't demonstrate the actual pointer-manipulation skill the problem is testing, and uses more memory than necessary — shown mainly as a bridge before the "real" solution.)*

---

## Approach 2: Optimal — Iterative Pointer Reversal

### Intuition

Reversing a linked list means: for every node, its `.next` pointer should point to what used to be **before** it, instead of what used to come after it. We walk through the list once, and at each node, flip its `.next` pointer to point backward. The one thing that makes this genuinely tricky, and the entire reason the topic overview's "save before overwrite" warning exists: **the moment we flip `current.next` to point backward, we lose the only reference we had to what comes next in the original list** — so we must capture that reference *before* performing the flip, every single iteration, with no exceptions.

### Algorithm

1. Keep two pointers: `prev` (starts as `None` — nothing comes before the original head) and `current` (starts at `head`).
2. While `current` is not `None`:
   - Save `next_node = current.next` (before we overwrite it — this is the non-negotiable first step).
   - Reverse the link: `current.next = prev`.
   - Move both pointers forward: `prev = current`, `current = next_node`.
3. When the loop ends (`current` is `None`), `prev` is now the new head of the reversed list.

### Python code
```python
def reverseList(head):
    prev = None
    current = head

    while current:
        next_node = current.next  # save before overwriting
        current.next = prev       # reverse the pointer
        prev = current             # advance prev
        current = next_node        # advance current

    return prev  # prev is the new head
```

### Line-by-line explanation

- `prev = None` — represents "what should come before the current node in the reversed list" — initially nothing, since the original head will become the new *tail*, and a tail's `.next` should correctly be `None`.
- `current = head` — our walking pointer through the original list.
- `while current:` — keep going until we've processed every node (reached the original list's natural end, `None`).
- `next_node = current.next` — **this is the critical line, and it must run first, before anything else in the loop body touches `current.next`.** It saves where the list continues *before* that pointer gets destroyed on the next line.
- `current.next = prev` — the actual reversal: this node now points backward, to whatever came before it in our walk (which is the reversed direction, since we're walking the original list forward but building the new list's links backward).
- `prev = current` — the node we just processed becomes the new "previous" for the next iteration.
- `current = next_node` — move to the node we saved earlier, continuing the original forward walk — this is *only* possible because we captured `next_node` before overwriting `current.next`; without that saved reference, this line would have nothing to advance to.
- `return prev` — once `current` becomes `None` (we've walked off the end of the original list), `prev` is sitting on the *last* node we processed — which was the original list's last node, now correctly the new head.

### Dry run

`1 -> 2 -> 3 -> None`

| step | prev | current | next_node = current.next | current.next = prev (rewire) | prev = current | current = next_node |
|---|---|---|---|---|---|---|
| start | None | 1 | - | - | - | - |
| 1 | None | 1 | 2 | node(1).next = None | prev = 1 | current = 2 |
| 2 | 1 | 2 | 3 | node(2).next = 1 | prev = 2 | current = 3 |
| 3 | 2 | 3 | None | node(3).next = 2 | prev = 3 | current = None |

Loop ends (`current is None`). Return `prev = 3`.

Resulting list, following `.next` from the returned head: `3 -> 2 -> 1 -> None` ✅ (node 3 points to node 2 — set in step 3; node 2 points to node 1 — set in step 2; node 1 points to `None` — set in step 1). Notice this required *zero* new node allocation — every node from the original list is reused, just rewired.

### Time & space complexity

- **Time: O(n)** — visits each node exactly once, doing O(1) work per node.
- **Space: O(1)** — only a few pointer variables, no new nodes created, entirely in-place. This is the real advantage over Approach 1.

---

## Approach 3: Optimal (alternative) — Recursive

### Intuition

"Reverse a list starting at `head`" can be thought of recursively: "reverse everything *after* `head` first (a smaller version of the exact same problem), and once that's done, attach the original `head` to the *end* of that now-reversed sub-list." This is a genuine recursive decomposition — the sub-problem (reversing `head.next` onward) is strictly smaller and identical in shape to the original problem, which is what justifies the recursive structure.

### Algorithm

1. **Base case:** if `head` is `None` or `head.next` is `None` (0 or 1 nodes), it's already "reversed" (trivially) — return it as-is.
2. Recursively reverse everything after `head`, and capture the new head of that reversed sub-list (`new_head`).
3. After the recursive call, `head.next` (the node right after `head`) is now the **tail** of the reversed sub-list. Make it point back to `head`: `head.next.next = head`.
4. Set `head.next = None` (since `head` is now the new tail of the *whole* reversed list, it must point to nothing).
5. Return `new_head` (unchanged all the way up the recursion — it's always the *original last node* of the list, found once at the deepest base case).

### Python code
```python
def reverseList(head):
    if not head or not head.next:
        return head

    new_head = reverseList(head.next)
    head.next.next = head
    head.next = None

    return new_head
```

### Line-by-line explanation

- `if not head or not head.next: return head` — base case: an empty list or a single node needs no reversal — it's already correct as-is.
- `new_head = reverseList(head.next)` — recursively reverse everything after `head`. By the time this line finishes, everything after `head` is already fully reversed *among itself*, and `new_head` holds a reference to what will become the final head of the *entire* reversed list — the same value bubbles up unchanged through every subsequent stack frame.
- `head.next.next = head` — `head.next` is the node that used to come right after `head` in the original list — after the recursive reversal, that node is now the **last** node of the reversed sub-list (its own `.next` is currently `None`). Point its `.next` back to `head`, extending the reversed chain to include `head` at the end.
- `head.next = None` — `head` will become the new tail once everything is reversed, so it must point to nothing — **this line runs *after* `head.next.next = head`, and the order matters**: if you set `head.next = None` first, you'd lose the reference to `head.next` needed for the line above.
- `return new_head` — pass the same "true new head" (found once, at the bottom of the recursion) back up through every level, unchanged.

### Dry run

`1 -> 2 -> 3 -> None`

Recursive calls go **down** first: `reverseList(1)` calls `reverseList(2)` calls `reverseList(3)`.
- `reverseList(3)`: `head.next` is `None` → base case → return `3` (node 3 itself). This becomes `new_head` at every level above.
- Back in `reverseList(2)`: `new_head = 3`. `head.next.next = head` → `node(3).next = node(2)`. `head.next = None` → `node(2).next = None`. Return `new_head = 3`.
- Back in `reverseList(1)`: `new_head = 3`. `head.next.next = head` → `node(2).next = node(1)`. `head.next = None` → `node(1).next = None`. Return `new_head = 3`.

Final structure, starting from the returned `new_head` (node 3): `3 -> 2 -> 1 -> None` ✅ (matches: node 3 → node 2 set in the `reverseList(2)` frame, node 2 → node 1 set in the `reverseList(1)` frame, node 1 → None set in the same frame).

### Time & space complexity

- **Time: O(n)** — each node is processed once across all recursive calls.
- **Space: O(n)** — the recursion uses the **call stack**, going n frames deep before any of the rewiring work happens (all the base-case returns happen first, then the rewiring unwinds back up) — this is a real, measurable cost, unlike the O(1) space of the iterative version, and for a very long list, this could risk hitting Python's recursion depth limit, which the iterative version never can.

---

## Common mistakes & misconceptions

1. **Overwriting `current.next` before saving it.** This is *the* signature bug for this entire topic, not just this problem — once `current.next = prev` executes, the original next node is unreachable unless it was saved first. There is no recovery once this happens.
2. **Setting `head.next = None` before `head.next.next = head` in the recursive version.** This destroys the reference needed for the line that should come first — order matters here in a way it's easy to get backward under time pressure.
3. **Returning `current` instead of `prev` at the end of the iterative version.** By the time the loop exits, `current` is `None` — the natural instinct to "return the last thing I was looking at" is wrong here; `prev` is the actual last node processed, and therefore the new head.
4. **Assuming the recursive version is "just as good" as the iterative one because it's shorter.** It has the same O(n) time, but genuinely worse space (O(n) call stack vs. O(1)) — for interview purposes, the iterative version is usually the expected primary answer, with the recursive version offered as a secondary, more elegant-looking alternative if asked.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Convert to array | O(n) | O(n) | Works, but doesn't practice pointer manipulation, and allocates new nodes. |
| Iterative reversal | O(n) | O(1) | The standard, best-in-class solution. |
| Recursive | O(n) | O(n) (call stack) | Elegant to write, but costs stack space; good to know both. |

**Key takeaway:** the "save the next reference before you overwrite the current pointer" discipline in the iterative approach is the single most important habit for linked list problems — almost every bug in linked list code comes from breaking this rule and permanently losing access to part of the list.
