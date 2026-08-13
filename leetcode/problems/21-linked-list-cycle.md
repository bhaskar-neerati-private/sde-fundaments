# 21. Linked List Cycle

**LeetCode:** [#141 - Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/) · **Topic:** [Linked List](../topics/06-linked-list.md) · **Difficulty:** Easy

## Problem statement

Given the `head` of a linked list, determine if the list has a **cycle** in it (some node's `.next` eventually points back to a node already visited, instead of reaching `None`).

**Example:**
```
Input: 3 -> 2 -> 0 -> -4 -> (back to node with value 2)
Output: true
```

## Applicable approaches

- **Brute Force — Hash Set of visited nodes.**
- **Optimal — Floyd's Tortoise and Hare (fast & slow pointers).** The standard, expected O(1)-space solution.

## Approach 1: Brute Force — Hash Set of Visited Nodes

### Intuition

Walk through the list, remembering every node **object** we've already visited (not values — two different nodes could coincidentally have the same value, so we must track node *identity*, not node *value*, to correctly detect a true structural cycle). If we ever land on a node we've seen before, there's a cycle, because reaching an already-visited node means we followed `.next` pointers back to somewhere we've been — the only way that's possible in a singly linked structure. If we reach `None`, the chain has a genuine end, so there's no cycle.

### Algorithm

1. Create an empty set `seen`.
2. Walk the list with a pointer `node`, starting at `head`.
3. If `node` is already in `seen`, return `True` (cycle found).
4. Otherwise, add `node` to `seen`, and move to `node.next`.
5. If we ever reach `None`, return `False` (no cycle).

### Python code
```python
def hasCycle(head):
    seen = set()
    node = head
    while node:
        if node in seen:
            return True
        seen.add(node)
        node = node.next
    return False
```

### Line-by-line explanation

- `seen = set()` — stores node *references* (Python's default equality/hashing for a plain object like `ListNode` is based on identity — memory address, essentially — so this correctly distinguishes different nodes even if they happen to hold equal `.val`s).
- `if node in seen: return True` — we've been here before → this node's `.next` chain loops back onto itself somewhere → cycle exists.
- `seen.add(node)` — remember this node.
- `node = node.next` — advance.
- `return False` — reached the natural end of the list (`None`) without ever revisiting a node → no cycle; every node was distinct and the chain genuinely terminated.

### Time & space complexity

- **Time: O(n)** — each node visited once, each set operation O(1) average.
- **Space: O(n)** — the set stores up to every node in the list, which is exactly the cost the optimal approach below eliminates.

---

## Approach 2: Optimal — Floyd's Tortoise and Hare

### Intuition

Imagine two runners on a track: a slow one (`slow`, moving 1 step at a time) and a fast one (`fast`, moving 2 steps at a time). If the track is a straight line with an end (`None`), the fast runner simply reaches the end first (or reaches a point where taking 2 more steps isn't possible), and they never meet. But if the track has a **loop** in it, both runners are eventually forced to keep circling that loop forever — and because the fast runner gains on the slow one by exactly 1 extra step of distance every iteration (moving 2 vs. 1), that gap between them, once both are trapped in a finite loop, must eventually shrink to exactly 0 — they're forced to land on the same node. This isn't a heuristic or a "usually works" claim: within a cycle of length `L`, the gap between the two runners decreases by 1 (modulo `L`) every iteration, so within at most `L` iterations after both have entered the cycle, the gap must hit exactly 0. No extra memory is needed at all — just two pointers, a genuine O(1)-space improvement over the hash-set approach.

### Algorithm

1. Set `slow = head`, `fast = head`.
2. Loop while `fast` and `fast.next` are both not `None` (if `fast` ever reaches the end, there's no cycle — a real end means no loop, since a fast runner on a genuine dead-end track always reaches the end first):
   - Move `slow` forward by 1: `slow = slow.next`.
   - Move `fast` forward by 2: `fast = fast.next.next`.
   - If `slow is fast` (the same node object), a cycle has been detected — return `True`.
3. If the loop exits because `fast` (or `fast.next`) reached `None`, there's no cycle — return `False`.

### Python code
```python
def hasCycle(head):
    slow, fast = head, head

    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow is fast:
            return True

    return False
```

### Line-by-line explanation

- `slow, fast = head, head` — both pointers start at the same place.
- `while fast and fast.next:` — this condition checks that the fast pointer can safely take its next 2-step move without crashing on `None.next` — if `fast` (or `fast.next`) is `None`, the list has a genuine end reachable by the *faster* pointer, meaning no cycle exists (if there were a cycle, `fast` could never escape it to reach `None`, since every node in a cycle has a valid `.next`).
- `slow = slow.next` — the slow pointer advances one node.
- `fast = fast.next.next` — the fast pointer advances two nodes. This is safe specifically because the `while` condition already confirmed `fast.next` exists before we try to access `fast.next.next`.
- `if slow is fast:` — **use `is`, not `==`** — we need to check they're the *same node object* (identity), the same distinction made in the brute force's hash-set approach. `is` makes this intent explicit and unambiguous.
- `return True` — the two pointers landed on the same node → a cycle exists, by the gap-shrinking argument above: in a cycle-free list, a faster pointer moving in the same direction as a slower one can never "catch up" to it on a straight, non-looping path — meeting is only possible if they're both trapped looping around the same cycle.
- `return False` — the loop ended naturally (fast pointer hit the real end) → no cycle.

### Dry run

List: `3 -> 2 -> 0 -> -4 -> (back to node "2")` (a cycle exists, the tail `-4` node points back to the second node)

Let's label nodes by position for clarity: `A(3) -> B(2) -> C(0) -> D(-4) -> back to B`

| iteration | slow (before move) | fast (before move) | slow after (slow.next) | fast after (fast.next.next) | slow is fast? |
|---|---|---|---|---|---|
| 1 | A | A | B | C (A→B→C) | no |
| 2 | B | C | C | B (C→D→B) | no |
| 3 | C | B | D | D (B→C→D) | **yes!** |

At iteration 3, both pointers land on node D → cycle detected → return `True` ✅

(For a cycle-free list, `fast` would simply reach `None` — or `fast.next` would be `None` — within roughly `n/2` iterations, and the loop would exit normally via the `while` condition, returning `False`. Try this yourself on `1 -> 2 -> None` to confirm: `fast` starts at 1, moves to `1.next.next` which requires `1.next` = 2 to have a `.next` — it's `None`, so `fast.next.next` would fail; but the `while fast and fast.next` check catches this *before* attempting the move, since after the first iteration `fast` lands on `None` and the loop condition correctly stops.)

### Time & space complexity

- **Time: O(n)** — even though `fast` moves twice as fast, both pointers still only take a number of steps proportional to n. In the worst case, once `slow` enters the cycle, `fast` might need to lap it up to once more before catching up, but this is still bounded by O(cycle length) ≤ O(n) additional steps — the total is still O(n), not more.
- **Space: O(1)** — only two pointers, no matter how long the list is. This is the key advantage over the hash-set approach — genuinely constant memory regardless of input size.

---

## Common mistakes & misconceptions

1. **Using `==` instead of `is` to compare `slow` and `fast`.** For a plain class like `ListNode` without a custom `__eq__` defined, Python's default `==` actually falls back to identity comparison too — so this specific mistake happens to not cause a bug *in this exact problem* — but relying on that default is fragile and easy to break if `ListNode` is ever given a custom `__eq__` (e.g. comparing by `.val`) elsewhere in a larger codebase. Using `is` explicitly is always correct and always communicates the actual intent.
2. **Checking `while fast.next and fast.next.next` instead of `while fast and fast.next`.** These are subtly different: the version in the code above checks `fast` isn't `None` *first* (to safely access `fast.next`), then checks `fast.next` isn't `None` (to safely access `fast.next.next`) — reordering or restructuring this check can lead to an `AttributeError` on `None.next` if not done carefully.
3. **Believing the fast/slow pointers meet at the start of the cycle.** They don't — they meet at *some* point inside the cycle, not necessarily its beginning. Finding *where* the cycle begins (a common follow-up question) requires an additional step after detection (resetting one pointer to `head` and advancing both at the same speed until they meet again), not something this basic detection algorithm gives you for free.
4. **Assuming this approach works for detecting cycles in structures other than singly linked lists** (e.g. general graphs) without modification — Floyd's algorithm specifically relies on each node having exactly one outgoing `.next` pointer; a general graph node can have multiple outgoing edges, and "fast/slow" doesn't have an unambiguous meaning there. Graph cycle detection (covered in the Graphs topic) uses different techniques entirely.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Hash Set of visited nodes | O(n) | O(n) | Simple and intuitive, but uses extra memory proportional to list length. |
| Floyd's Tortoise and Hare | O(n) | O(1) | The standard, expected, memory-optimal solution. |

**Key takeaway:** the fast/slow pointer trick isn't just for cycle detection — the same "two pointers moving at different speeds" idea also finds the *middle* of a list in one pass (slow ends up at the midpoint when fast reaches the end, since fast covers exactly twice the distance), and (with additional steps) finds exactly *where* a cycle begins, which is a common follow-up question to this exact problem.
