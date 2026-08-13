# 2. Contains Duplicate

**LeetCode:** [#217 - Contains Duplicate](https://leetcode.com/problems/contains-duplicate/) · **Topic:** [Arrays & Hashing](../topics/01-arrays-hashing.md) · **Difficulty:** Easy

## Problem statement

Given an integer array `nums`, return `true` if any value appears **at least twice**, and `false` if every element is distinct.

**Example:**
```
Input: nums = [1, 2, 3, 1]
Output: true   (1 appears twice)

Input: nums = [1, 2, 3, 4]
Output: false  (all distinct)
```

## Applicable approaches

- **Brute Force** — compare every pair.
- **Sorting** — sort first, then check neighbors.
- **Optimal — Hash Set** — the standard, expected solution.

---

## Approach 1: Brute Force

### Intuition

The most literal reading of "does any value repeat" is: check every value against every *other* value, and if any two match, you're done. This is the same "try everything" starting point as Two Sum, and it's worth stating explicitly why it's correct even though it's slow — every possible pair genuinely gets examined, so no duplicate can hide from it.

### Algorithm

1. For each index `i`, compare `nums[i]` against every `nums[j]` where `j > i`.
2. If any pair matches, return `True`.
3. If no pair ever matches, return `False`.

### Python code
```python
def containsDuplicate(nums):
    n = len(nums)
    for i in range(n):
        for j in range(i + 1, n):
            if nums[i] == nums[j]:
                return True
    return False
```

### Line-by-line explanation

- Same double-loop shape as Two Sum's brute force, and for the same reason: `j` starts at `i + 1` so we never compare an element to itself and never check the same unordered pair twice.
- `if nums[i] == nums[j]` — found two equal values anywhere in the array → a duplicate exists, by definition.
- If the loops finish with no match, every element is provably unique — the double loop examined every pair, leaving no possibility unchecked.

### Dry run

`nums = [1, 2, 3, 1]`

| i | j | nums[i] | nums[j] | equal? |
|---|---|---|---|---|
| 0 | 1 | 1 | 2 | no |
| 0 | 2 | 1 | 3 | no |
| 0 | 3 | 1 | 1 | **yes** → return `True` |

Notice this only needed to reach `j=3` for `i=0` — it didn't need to check pairs like `(1,2)` or `(1,3)` at all, because the function returns the instant a match is found. But if the input were `[1,2,3,4]` (no duplicate), the loop would have to exhaust *every* one of the `C(4,2) = 6` pairs before concluding `False` — this is the worst case, and it's the case you should reason about for complexity, not the lucky early-exit shown here.

### Time & space complexity

- **Time: O(n²)** worst case (no duplicate exists, or the duplicate is the very last pair checked) — every pair examined, same reasoning as Two Sum's brute force.
- **Space: O(1)**.

---

## Approach 2: Sorting

### Intuition

The brute force's waste is comparing *every* pair, when most pairs are obviously irrelevant — if two equal values exist anywhere in the array, you don't need to check them against everything else, you just need to notice they're equal to *each other*. Sorting exploits this: **once sorted, any duplicate values are forced to sit directly next to each other**, because sorting is exactly "put equal or increasing values in sequence," and two equal values can't have anything sort *between* them. This collapses the question from "do any two of the n² pairs match" to "does any *adjacent* pair match" — a single linear scan.

### Algorithm

1. Sort the array.
2. Walk through it once, comparing `nums[i]` to `nums[i - 1]`.
3. If any adjacent pair is equal, return `True`.
4. Otherwise return `False`.

### Python code
```python
def containsDuplicate(nums):
    nums.sort()
    for i in range(1, len(nums)):
        if nums[i] == nums[i - 1]:
            return True
    return False
```

### Line-by-line explanation

- `nums.sort()` — sorts in place, O(n log n). This is the step that does the real work: it's what *guarantees* any duplicate becomes adjacent, which is the entire reason the rest of the function can be a simple linear scan.
- `for i in range(1, len(nums))` — start at index 1 so `nums[i - 1]` is always a valid, already-visited index.
- `if nums[i] == nums[i - 1]` — **this single comparison is sufficient to catch every duplicate in the array, precisely because of the sort.** It would *not* be sufficient on an unsorted array — this is a case where a check that looks "too simple" is actually correct only because of a precondition established earlier (the sort), which is worth noticing as a general pattern: an O(n) scan can replace an O(n²) check *only when* some earlier step has restructured the data to make adjacency meaningful.

### Dry run

`nums = [1, 2, 3, 1]` → sorted: `[1, 1, 2, 3]`

| i | nums[i] | nums[i-1] | equal? |
|---|---|---|---|
| 1 | 1 | 1 | **yes** → return `True` |

The two `1`s, originally at opposite ends of the array (indices 0 and 3), became adjacent (indices 0 and 1) after sorting — that's the mechanism doing the work here.

### Time & space complexity

- **Time: O(n log n)** — dominated by the sort; the scan afterward is O(n), which doesn't change the overall order.
- **Space: O(1) extra** if using Python's in-place `list.sort()` (its own internal working space is typically considered O(n) worst case for Timsort's merge step, but conventionally this is described as O(1) *extra* relative to the input when sorting in place, or O(n) if you must preserve the original order and use `sorted()` instead, which allocates a new list).

---

## Approach 3: Optimal — Hash Set

### Intuition

Sorting solves the problem but pays an O(n log n) cost to do it — and that cost buys you *ordering*, which you don't actually need here. The question "have I seen this value before" doesn't care about order at all; it only cares about set membership. A hash set answers exactly that question in O(1) per check (see the topic overview's addressing-not-searching mental model), so you can catch a duplicate the instant it reappears, in a single O(n) pass, without ever needing the values to be sorted.

### Algorithm

1. Create an empty set `seen`.
2. For each number in the array: if it's already in `seen`, return `True` immediately.
3. Otherwise, add it to `seen` and continue.
4. If the loop finishes without a match, return `False`.

### Python code
```python
def containsDuplicate(nums):
    seen = set()
    for num in nums:
        if num in seen:
            return True
        seen.add(num)
    return False
```

A one-line alternative using set size (less instructive, but common in practice):
```python
def containsDuplicate(nums):
    return len(nums) != len(set(nums))
```
This works because converting to a `set` automatically drops duplicates — if the set is smaller than the original list, something was repeated. It's O(n) time and O(n) space, same as the loop version, just less explicit about *how* it decides, and it can't short-circuit early the way the loop version can.

### Line-by-line explanation (loop version)

- `seen = set()` — empty hash set to track numbers encountered so far.
- `for num in nums` — walk through the array once.
- `if num in seen` — O(1) check: have we already recorded this exact value?
- `return True` — yes, so this value is a duplicate — no need to scan further, unlike the one-liner version above which always processes the entire array regardless of where the duplicate is.
- `seen.add(num)` — not seen yet, so remember it for future comparisons.
- `return False` — if we get through the entire array without a match, every value was unique, because we checked *every* value against the *complete* memory of everything before it.

### Dry run

`nums = [1, 2, 3, 1]`

| num | in seen? | action | seen after |
|---|---|---|---|
| 1 | No | add | `{1}` |
| 2 | No | add | `{1, 2}` |
| 3 | No | add | `{1, 2, 3}` |
| 1 | **Yes** | return `True` | — |

### Time & space complexity

- **Time: O(n)** average case — one pass, O(1) average per set operation (see the topic overview's note on hash collisions for why this is "average," not an absolute worst-case guarantee).
- **Space: O(n)** worst case (no duplicates exist) — the set ends up holding every element.
- **Contrast across all three approaches:** brute force is O(n²)/O(1), sorting is O(n log n)/O(1) extra, hash set is O(n)/O(n). The hash set is fastest in time but spends the most memory — this tradeoff pattern (trading space for time via a hash structure) is the dominant theme of this entire topic.

---

## Common mistakes & misconceptions

1. **Checking `if num in seen` after adding `num` to `seen`, instead of before.** Reversing the order means every number trivially "finds itself" in the set (since it was just added), incorrectly reporting a duplicate for *every* input, including ones with no real repeats. Always check membership *before* inserting.
2. **Using the sorting approach but sorting a copy incorrectly, or forgetting the original array's order is destroyed.** `nums.sort()` mutates the input in place — if the caller needed the original order preserved for something else downstream, this is a real bug, not just a style choice. Use `sorted(nums)` (which returns a new list) if the original order must survive.
3. **Assuming "hash set lookups are O(1)" means "always exactly as fast as an array index lookup."** They're both O(1) *asymptotically*, but a hash lookup involves computing a hash and probing, which has a larger constant factor than direct array indexing — this rarely matters for correctness or even for passing time limits, but it's a common misconception that all O(1) operations are equally fast in practice.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute Force | O(n²) | O(1) | Simple, slow — every pair genuinely checked. |
| Sorting | O(n log n) | O(1) extra (in-place) | Good if you're told not to use extra memory. |
| Hash Set | O(n) | O(n) | Fastest; the expected interview answer. |

**Key takeaway:** "have I seen this before?" is one of the most common questions in coding interviews, and a hash set answers it in O(1) by turning membership-checking into addressing rather than scanning. This exact pattern (loop + set membership check, insert-after-check ordering) will reappear constantly throughout the rest of this list.
