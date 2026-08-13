# Topic 2: Two Pointers

## Core concepts / data structures

### The Two Pointers technique

**What it is:** instead of using one index to walk through an array/string, you use **two** index variables ("pointers") that move through the data — usually toward each other (from both ends inward) or in the same direction at different speeds — to solve a problem in a single pass, instead of checking every possible pair with nested loops.

**The conceptual shift, not just the mechanics:** the reason Two Pointers works isn't "using two variables instead of one" — plenty of O(n²) algorithms use two variables (nested loops) and are still O(n²). The real idea is that **each pointer only ever moves in one direction, and every single-step move is justified by a proof that it can't skip the answer.** That's what turns "checking pairs" from O(n²) (every pair, because you don't know which ones are safe to skip) into O(n) (each pointer crosses the array once, because you've *proven* which pairs are safe to never check at all). If you can't articulate why moving a specific pointer is always safe, you don't actually have a Two Pointers algorithm — you have two loop variables that happen to be at risk of missing the correct answer.

**Simple analogy:** imagine two people searching a single-file line of lockers for a matching pair of items — one starting from the left end, one from the right end, walking toward each other. As long as the lockers are sorted by some useful property, each person can make a provably correct decision about which direction to move based on what they currently see — they never have to backtrack, because moving forward is only ever safe when the option they're leaving behind is safe to abandon forever.

**Why it only works efficiently on structured data:** the whole approach depends on being able to *predict* the effect of moving a pointer. That prediction is only reliable when there's structure to exploit — almost always, **the array is sorted** (or can be treated as sorted), so moving a pointer left or right has a known, monotonic effect (e.g. moving the left pointer right always increases the value at that position, in a sorted ascending array). Without that structure, you can't reason about which pointer move is safe, and the technique doesn't apply.

### The two common shapes

**1. Opposite-direction pointers** (start at both ends, move inward)
```python
left, right = 0, len(arr) - 1
while left < right:
    # look at arr[left] and arr[right], decide which to move (or both)
    if some_condition:
        left += 1
    else:
        right -= 1
```
Used for: checking pairs/triplets that sum to a target in a *sorted* array, palindrome checks, "container with most water"-style area maximization.

**2. Same-direction pointers (slow/fast, or read/write)** (both start near the beginning, move forward at different rates)
```python
slow = 0
for fast in range(len(arr)):
    # decide whether to advance slow based on arr[fast]
    if some_condition:
        arr[slow] = arr[fast]
        slow += 1
```
Used for: removing duplicates in-place, partitioning arrays, and (in linked lists) cycle detection or finding the middle node.

## Common patterns / techniques in this topic

| Pattern | When it applies | Why the pointer move is safe |
|---|---|---|
| **Opposite-end pointers on a sorted array** | Find a pair (or triplet, via one pointer fixed + two moving) that sums to a target. | If the current sum is too small, the *only* way to increase it is to move `left` right (since the array is sorted ascending, that's the only direction that increases a value); moving `right` left could only make the sum smaller, which can't fix "too small." So `right` staying put while `left` moves is provably the only useful move. |
| **Palindrome check** | Compare characters from both ends moving inward. | A palindrome must match at every mirrored position; if the two ends don't match, no amount of adjusting the *other* end can fix that specific mismatch — it's an immediate, final answer. |
| **Area/container maximization** | Two pointers at the ends of a range; the side with the smaller "height" is always the one limiting the result. | Keeping the shorter wall's position and moving on can never improve the answer for *that* wall (moving the taller wall inward can only shrink width without ever increasing the limiting height) — see the Container With Most Water write-up for the full proof. |
| **Fast/slow pointers** | Cycle detection (Floyd's algorithm), finding the middle of a sequence, removing duplicates in-place. | These use the pointers' *relative speed*, not sortedness, to guarantee a property (e.g. two pointers moving at different speeds around a cycle must eventually coincide). |

## Key terminology

- **In-place** — modifying the array directly using O(1) extra space, rather than building a new array/list.
- **Monotonic movement** — each pointer only ever moves in one direction (never backtracks), which is what keeps two-pointer algorithms O(n) instead of O(n²) — each pointer takes at most n steps total across the whole algorithm, so the total work across both pointers combined is O(n), not O(n) per starting position.
- **Two Sum II-style problems** — the family of problems (this exact pattern, not literally the LeetCode problem of that name) where a sorted array + target sum calls for opposite-direction pointers instead of a hash map.

## Common beginner mistakes (and *why* each one is wrong)

1. **Trying to use Two Pointers on an unsorted array without sorting first.** The technique relies on being able to *predict* what happens when you move a pointer — that prediction is only valid if the data has exploitable order. If the problem needs indices from the *original* unsorted array, sorting the raw values destroys that information; sort `(value, original_index)` pairs instead, so the original position travels alongside the value.
2. **Moving both pointers when only one should move.** E.g. in the "pair sums to target" pattern, if the current sum is too small, only `left` should move — moving `right` too can jump *past* the correct pairing entirely, because you've made two changes when you'd only proven one of them safe.
3. **Off-by-one in the loop condition.** `while left < right` vs `while left <= right` matters a lot: using `<=` when the two pointers should never be allowed to reference the same element (or cross) is a classic source of bugs, especially in problems where a self-pairing (an element paired with itself) is invalid.
4. **Forgetting to skip duplicate values** in problems like 3Sum, which causes duplicate *answers* in the output (not incorrect individual answers, but a correctness bug against the "no duplicate triplets" requirement).
5. **Confusing fast/slow pointers with opposite-end pointers.** They solve different kinds of problems (cycle/middle-finding vs. pair-sum/palindrome-style problems), rely on different underlying guarantees (relative speed vs. sortedness), and move in different directions — picking the wrong shape for a problem produces an algorithm that doesn't terminate correctly or doesn't converge on the right answer.

## How this compares to Arrays & Hashing

Both topics often solve "does a pair exist that sums to X?"-style problems, but they use **opposite strategies for the same underlying question**: hashing works on **unsorted** data by trading space for speed — remember what you've seen, in O(n) extra space, to answer each query in O(1). Two Pointers works on **sorted** data by trading a sort (or given order) for space efficiency — O(1) extra space instead of O(n) for a hash map, because sortedness lets you *prove* which pointer move is safe instead of needing to remember everything you've seen. If a problem's array is already sorted (or you're free to sort it and don't need to preserve original indices), consider Two Pointers before reaching for a hash map — it often gets you the same O(n) time complexity with meaningfully less memory.

## Starter problems (easy, to warm up)

1. **Valid Palindrome** (LeetCode #125) — classic opposite-end pointer comparison. (Also in your Blind 75 list.)
2. **Two Sum II - Input Array Is Sorted** (LeetCode #167) — not in Blind 75, but the purest example of the opposite-pointer sum pattern, worth doing to build intuition before 3Sum.
3. **Merge Sorted Array** (LeetCode #88) — not in Blind 75, but a great same-direction / write-pointer warm-up.
4. **Remove Duplicates from Sorted Array** (LeetCode #26) — not in Blind 75, but the standard slow/fast in-place pattern.

## What carries over from here

The fast/slow pointer idea reappears directly in **Linked List** problems (cycle detection, finding the middle node) and the "shrinking window based on a condition" idea is the direct ancestor of the next topic, **Sliding Window** — which is essentially Two Pointers where the region *between* the pointers (the "window") is the thing being tracked and grown/shrunk, rather than just the two endpoint values.
