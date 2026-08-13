# 17. Find Minimum in Rotated Sorted Array

**LeetCode:** [#153 - Find Minimum in Rotated Sorted Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/) · **Topic:** [Binary Search](../topics/05-binary-search.md) · **Difficulty:** Medium

## Problem statement

A sorted array (no duplicates) has been **rotated** at some unknown pivot (e.g. `[0,1,2,4,5,6,7]` rotated becomes `[4,5,6,7,0,1,2]`). Given the rotated array `nums`, find the **minimum** element, in O(log n) time.

**Example:**
```
Input: nums = [3,4,5,1,2]
Output: 1
```

## Applicable approaches

- **Brute Force — Linear Scan** — check every element.
- **Optimal — Binary Search** — the standard, expected O(log n) solution.

## Approach 1: Brute Force — Linear Scan

### Intuition

Just walk through the array and track the smallest value seen — this is correct regardless of any rotation, since it doesn't rely on order at all, only on visiting every element. It's the natural default when you don't yet see how to exploit the array's partial sortedness.

### Python code
```python
def findMin(nums):
    return min(nums)
```

(Python's built-in `min()` does exactly this internally — one linear pass, no shortcuts.)

### Time & space complexity

- **Time: O(n)** — checks every element; there's no way to skip any element without knowing something about the array's structure.
- **Space: O(1)**.

*(This is correct and simple, but the problem specifically wants O(log n), which requires actually using the fact that the array was originally sorted before rotation — a linear scan throws that structural information away entirely.)*

---

## Approach 2: Optimal — Binary Search

### Intuition

Even though the whole array isn't sorted anymore (because of the rotation), there's still exploitable structure: **at every step, at least one half of the current search range is properly sorted.** This is the key fact that makes binary search applicable at all here, despite the array not being globally sorted — see the topic overview's point that binary search only needs *some* monotonic structure, not necessarily a fully sorted array.

The specific test for finding the minimum: compare `nums[mid]` to `nums[right]`.
- If `nums[mid] > nums[right]`, the minimum must be somewhere in the **right half**. Why: since the array is a rotated *sorted* array, values only ever "drop" once (at the rotation point) as you scan left to right; if `nums[mid]` is *bigger* than `nums[right]`, that drop must occur somewhere between `mid` and `right` (otherwise `nums[right]` couldn't be smaller than `nums[mid]`) — so the minimum is strictly after `mid`.
- If `nums[mid] <= nums[right]`, the segment from `mid` to `right` is **already properly sorted** (no drop occurs in it, since nothing in it exceeds `nums[right]`), meaning the minimum of the whole array must be at `mid` or somewhere in the **left half** instead (or `mid` itself could already be the minimum).

By always keeping the search range narrowed toward wherever the "drop point" (rotation pivot) must be, we converge on the minimum in O(log n) steps — each comparison eliminates a half we've *proven* can't contain the answer, exactly the certainty the topic overview says binary search requires.

### Algorithm

1. Set `left = 0`, `right = len(nums) - 1`.
2. While `left < right`:
   - Compute `mid = (left + right) // 2`.
   - If `nums[mid] > nums[right]`: the minimum is in the right half, *excluding* `mid` itself (since `nums[mid]` is confirmed larger than `nums[right]`, it can't be the minimum) → `left = mid + 1`.
   - Else (`nums[mid] <= nums[right]`): the minimum is at `mid` or to its left → `right = mid` (note: **not** `mid - 1`, because `nums[mid]` itself could still be the minimum, so we can't rule it out).
3. When the loop ends, `left == right`, and that index holds the minimum.

### Python code
```python
def findMin(nums):
    left, right = 0, len(nums) - 1

    while left < right:
        mid = (left + right) // 2
        if nums[mid] > nums[right]:
            left = mid + 1
        else:
            right = mid

    return nums[left]
```

### Line-by-line explanation

- `left, right = 0, len(nums) - 1` — the full array is the initial search range.
- `while left < right:` — **note this uses `<`, not `<=`**, which is a deliberate deviation from the classic search template — this loop is narrowing down to a single *index* (the minimum's position), not searching for a value and returning early on a match, so it naturally terminates once `left` and `right` converge to the same index, and we don't want to keep looping (or risk an off-by-one) past that point.
- `mid = (left + right) // 2` — the middle of the current range.
- `if nums[mid] > nums[right]:` — the value at `mid` is bigger than the value at the range's right edge. As explained above, this can only happen if the rotation's "drop" is between `mid` and `right`, so the true minimum is in that right portion, strictly *after* `mid`.
- `left = mid + 1` — narrow the search to the right half, excluding `mid` (which is provably not the answer).
- `else: right = mid` — `nums[mid] <= nums[right]` means no drop occurs between `mid` and `right`, so the minimum (if it's in this segment at all) must be at the very start of it — i.e. at `mid` — or, the minimum could be further left still. **This is why we set `right = mid`, not `right = mid - 1`**: `mid` is still a *candidate* for the answer (unlike in the `if` branch, where `nums[mid]` was provably too large), so it must remain in the search range.
- `return nums[left]` — once `left == right`, we've narrowed all the way down to the single index holding the minimum.

### Dry run

`nums = [4,5,6,7,0,1,2]`

| left | right | mid | nums[mid] | nums[right] | nums[mid] > nums[right]? | action |
|---|---|---|---|---|---|---|
| 0 | 6 | 3 | 7 | 2 | yes (7>2) | left = 4 |
| 4 | 6 | 5 | 1 | 2 | no (1<=2) | right = 5 |
| 4 | 5 | 4 | 0 | 1 | no (0<=1) | right = 4 |

Now `left == right == 4`, loop ends. `nums[4] = 0` ✅ correct minimum.

**A second dry run with the example input:** `nums = [3,4,5,1,2]`

| left | right | mid | nums[mid] | nums[right] | compare | action |
|---|---|---|---|---|---|---|
| 0 | 4 | 2 | 5 | 2 | 5>2 | left=3 |
| 3 | 4 | 3 | 1 | 2 | 1<=2 | right=3 |

`left == right == 3`, `nums[3] = 1` ✅

### Time & space complexity

- **Time: O(log n)** — the search range halves every iteration, exactly the "eliminate half with each comparison" mechanism from the topic overview, and the elimination here is provably safe (not a heuristic) because of the "at most one drop point" property of a rotated sorted array.
- **Space: O(1)** — only a few index variables.

---

## Common mistakes & misconceptions

1. **Comparing `nums[mid]` to `nums[left]` instead of `nums[right]`.** This is a genuinely different (and more error-prone) comparison for *this specific problem* — comparing against `right` gives a clean, unconditional rule (`nums[mid] > nums[right]` always means "drop is between mid and right"), whereas comparing against `left` requires extra case-handling to distinguish "no rotation in this half" from "rotation point is exactly at the boundary." Using `nums[right]` consistently avoids this extra complexity.
2. **Using `right = mid - 1` in the else branch, mirroring the classic binary search template on reflex.** As explained above, `mid` is still a valid candidate for the minimum in this branch (unlike the classic template, where an exact match at `mid` ends the search immediately) — excluding it with `mid - 1` can skip over the correct answer.
3. **Assuming both halves need to be checked "just in case."** The entire point of the proof above is that you never need to check both halves — one is *guaranteed* to be safe to discard. Falling back to checking both (e.g. via extra `if`/`elif` branches "to be safe") abandons the O(log n) guarantee and isn't actually necessary for correctness.
4. **Forgetting the problem guarantees no duplicate values.** With duplicates allowed (a related but harder variant of this problem, not this exact one), `nums[mid] == nums[right]` becomes ambiguous — you can't tell which half is sorted, and the worst-case complexity degrades to O(n). This exact solution relies on the "no duplicates" guarantee to always make a clean comparison.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Linear Scan | O(n) | O(1) | Correct, simple, but doesn't meet the O(log n) requirement. |
| Binary Search | O(log n) | O(1) | The required, expected optimal solution. |

**Key takeaway:** a rotated sorted array isn't fully sorted, but **one half of any split is always properly sorted** — the core skill in rotated-array binary search problems is figuring out, at each step, which half you can *prove* is trustworthy, using a comparison against the boundary (`nums[right]` here, or sometimes `nums[left]`, depending on exactly what's being searched for).
