# 18. Search in Rotated Sorted Array

**LeetCode:** [#33 - Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array/) · **Topic:** [Binary Search](../topics/05-binary-search.md) · **Difficulty:** Medium

## Problem statement

Given a rotated sorted array `nums` (no duplicates) and a `target`, return the index of `target` if it exists, or `-1` if it doesn't — in O(log n) time.

**Example:**
```
Input: nums = [4,5,6,7,0,1,2], target = 0
Output: 4
```

## Applicable approaches

- **Brute Force — Linear Scan** — check every element.
- **Optimal — Modified Binary Search** — the standard, expected O(log n) solution.

## Approach 1: Brute Force — Linear Scan

### Intuition

Just check every element until you find the target (or don't) — correct, and doesn't depend on the array's rotation structure at all, but for exactly that reason it also can't achieve better than O(n).

### Python code
```python
def search(nums, target):
    for i, num in enumerate(nums):
        if num == target:
            return i
    return -1
```

### Time & space complexity

- **Time: O(n)**, **Space: O(1)**. Correct, but doesn't meet the required O(log n) — it never exploits the fact that most of the array is still sorted, in two pieces.

---

## Approach 2: Optimal — Modified Binary Search

### Intuition

This builds on the exact same structural fact used in Find Minimum in Rotated Sorted Array: at every step, **one half of the current range is always properly sorted**, even if the whole array isn't. But this problem asks a different question — not "where's the minimum," but "does a specific target exist, and where" — so the algorithm needs an extra step beyond identifying the sorted half: once we know which half is sorted, we can ask a simple, decisive question about it — **does the target's value fall within that sorted half's value range?** Because that half is genuinely sorted, "is the target between its endpoints" is something we can check directly with a comparison, the same way we would in an ordinary binary search on a fully sorted array. If yes, search there; if no, the target (if it exists at all) must be in the *other* half — even though that other half isn't sorted itself, we don't need it to be, because we're not searching it directly this iteration, we're just recursing the same logic onto it next.

### Algorithm

1. Set `left = 0`, `right = len(nums) - 1`.
2. While `left <= right`:
   - Compute `mid = (left + right) // 2`. If `nums[mid] == target`, return `mid`.
   - Determine which half is sorted:
     - **If `nums[left] <= nums[mid]`:** the left half (`left` to `mid`) is sorted.
       - Check if `target` falls within `[nums[left], nums[mid])`: if so, search the left half (`right = mid - 1`); otherwise, search the right half (`left = mid + 1`).
     - **Else (`nums[mid] < nums[left]`):** the right half (`mid` to `right`) is sorted instead.
       - Check if `target` falls within `(nums[mid], nums[right]]`: if so, search the right half (`left = mid + 1`); otherwise, search the left half (`right = mid - 1`).
3. If the loop ends without finding the target, return `-1`.

### Python code
```python
def search(nums, target):
    left, right = 0, len(nums) - 1

    while left <= right:
        mid = (left + right) // 2
        if nums[mid] == target:
            return mid

        if nums[left] <= nums[mid]:
            # left half [left..mid] is sorted
            if nums[left] <= target < nums[mid]:
                right = mid - 1
            else:
                left = mid + 1
        else:
            # right half [mid..right] is sorted
            if nums[mid] < target <= nums[right]:
                left = mid + 1
            else:
                right = mid - 1

    return -1
```

### Line-by-line explanation

- `mid = (left + right) // 2` then `if nums[mid] == target: return mid` — standard "check the middle first" step, unchanged from the classic template.
- `if nums[left] <= nums[mid]:` — this comparison determines *which half is the properly-sorted one*, and it's worth being precise about why: if the value at `left` is ≤ the value at `mid`, there's no "drop" (rotation point) between them, since a drop is the only way a later value could be smaller than an earlier one in an otherwise-sorted sequence — so that whole segment is sorted normally.
  - `if nums[left] <= target < nums[mid]:` — since we know this half is genuinely sorted, we can directly check whether `target` would fall inside its value range using ordinary comparison — if so, narrow the search there (`right = mid - 1`).
  - `else: left = mid + 1` — target isn't in this sorted left half's range, so (if it exists at all in the array) it must be in the other half — we don't need to know anything about that other half's internal order to conclude this, only that it's "everything not in the range we just ruled out."
- `else:` (the left half must contain the rotation point, so the *right* half, `mid` to `right`, is the sorted one instead) — mirror logic:
  - `if nums[mid] < target <= nums[right]:` — target falls in the sorted right half's range → search there.
  - `else: right = mid - 1` — otherwise, search the left half instead.
- `return -1` — loop ended without a match, target isn't present.

### Dry run

`nums = [4,5,6,7,0,1,2]`, `target = 0`

| left | right | mid | nums[mid] | match? | nums[left]<=nums[mid]? | target in range? | action |
|---|---|---|---|---|---|---|---|
| 0 | 6 | 3 | 7 | no (7≠0) | `nums[0]=4 <= nums[3]=7` yes, left half sorted | is `4<=0<7`? no | target not in left half → `left = 4` |
| 4 | 6 | 5 | 1 | no (1≠0) | `nums[4]=0 <= nums[5]=1` yes, left half sorted | is `0<=0<1`? **yes** | search left half → `right = 4` |
| 4 | 4 | 4 | 0 | **yes (0==0)** | - | - | return `4` |

Final answer: `4` ✅. Notice at each step, exactly one half was ever *examined for the value range check* — the other half was discarded based purely on "target isn't in the sorted half's range," without ever needing to know its internal structure, which is the entire point of the reduction.

### Time & space complexity

- **Time: O(log n)** — the search range halves every iteration, same mechanism as ordinary binary search, applied to a range that's always provably split into "one sorted half we can directly check" and "one half we recurse into next."
- **Space: O(1)**.

---

## Common mistakes & misconceptions

1. **Getting the range-check boundaries wrong (`<` vs `<=`).** Because `nums` has no duplicates (per the problem's guarantee), the boundaries are precise: `nums[left] <= target < nums[mid]` (left-inclusive, mid-exclusive, since `nums[mid] == target` was already handled and ruled out above) — getting this backward (e.g. `<=` on both ends) can cause the target to be "searched for" in a half that provably doesn't contain it, wasting the log n guarantee without necessarily causing a wrong final answer, but is worth getting precisely right rather than by trial and error.
2. **Assuming the "other" (non-sorted-checked) half is itself unsorted garbage that needs special handling.** It isn't garbage — it's simply "the half we haven't examined *this iteration*." The algorithm doesn't need it to be sorted right now; the same logic (find which half is sorted, check the range) will apply to it correctly on the *next* iteration, since the same rotated-sorted-array structure holds recursively on any sub-range of the original array.
3. **Confusing this problem with Find Minimum in Rotated Sorted Array and reusing that problem's comparison (`nums[mid]` vs `nums[right]`) without adapting it.** That problem only needed to find *which half could contain the minimum*; this problem needs to find *which half is safe to directly range-check a target against* — the comparisons are related but serve different purposes, and blindly reusing one problem's exact branching logic for the other produces incorrect results.
4. **Trying to first find the rotation point, then binary-search each sorted half separately.** This works and is a valid two-phase approach, but it's strictly more code and doesn't run any faster (still O(log n) overall, since finding the pivot is itself O(log n)) — the single-pass version shown above is more commonly expected and avoids the bookkeeping of a two-phase search.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Linear Scan | O(n) | O(1) | Correct, but doesn't meet the required O(log n). |
| Modified Binary Search | O(log n) | O(1) | The required, expected optimal solution. |

**Key takeaway:** for rotated-array binary search problems, the recipe is always: (1) determine which half is sorted (using a comparison you can prove is airtight, like `nums[left] <= nums[mid]`), (2) check if the target's value falls in that sorted half's range using ordinary comparison, (3) search that half if so, otherwise search the other half without needing to know its internal order yet. This "identify the trustworthy half, then decide based on the target's value" reasoning is the general skill, reused with small variations across many rotated-array problems.
