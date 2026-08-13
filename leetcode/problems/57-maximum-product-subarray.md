# 57. Maximum Product Subarray

**LeetCode:** [#152 - Maximum Product Subarray](https://leetcode.com/problems/maximum-product-subarray/) · **Topic:** [1-D Dynamic Programming](../topics/13-1d-dynamic-programming.md) · **Difficulty:** Medium

## Problem statement

Given an integer array `nums`, find a **contiguous** subarray with the largest product, and return that product.

**Example:**
```
Input: nums = [2,3,-2,4]
Output: 6   ([2,3])

Input: nums = [-2,3,-4]
Output: 24  ([-2,3,-4], since the two negatives cancel out)
```

## Applicable approaches

- **Brute Force - Check every subarray.**
- **Sliding Window / Running Max (naive, WRONG for this problem)** - shown to explain why the simple "running max" trick from Maximum Subarray (sum version) doesn't directly work here.
- **Optimal - Track Running Max AND Min Simultaneously.** The standard, expected solution.

## Approach 1: Brute Force

### Intuition
Check every possible contiguous subarray's product directly, tracking the best found.

### Python code
```python
def maxProduct(nums):
    n = len(nums)
    best = nums[0]

    for i in range(n):
        product = 1
        for j in range(i, n):
            product *= nums[j]
            best = max(best, product)

    return best
```

### Time & space complexity
- **Time: O(n²)**, **Space: O(1)**.

---

## Why a Simple "Running Max" (like the sum version) Doesn't Work

For a *sum*-based version of this problem (Maximum Subarray, the next topic's problem), you can track a simple running best ending at each position, because adding a negative number to a running sum only ever makes it *smaller* - there's no surprising flip. But with **products**, multiplying by a **negative** number **flips the sign** - so a very *negative* running product could suddenly become the *largest* positive product, the moment it's multiplied by another negative number. A naive "just track the running max product" approach would incorrectly discard a very negative running product as "clearly bad," missing the fact that it could flip into the best answer on the next step.

## Approach 2: Optimal - Track Running Max AND Min Simultaneously

### Intuition
Because multiplying by a negative number can turn the **smallest** (most negative) running product into the **largest** one, we need to track **both** a running maximum *and* a running minimum product ending at each position - since either one could become the new maximum once multiplied by the next number (positive or negative).

### Algorithm
1. Track `curr_max` and `curr_min` (the best and worst product of a subarray *ending at the current position*), both initialized to `nums[0]`.
2. Track `best` (the overall best found), initialized to `nums[0]`.
3. For each subsequent number `num`:
   - **Before updating**, if `num` is negative, swap `curr_max` and `curr_min` (multiplying by a negative number flips which one would become the larger result, so pre-swapping means the "max" formula below naturally does the right thing).
   - `curr_max = max(num, curr_max * num)` - either start a fresh subarray at `num` alone, or extend the previous best product by multiplying in `num`.
   - `curr_min = min(num, curr_min * num)` - the same idea, tracking the worst-case running product (which might become useful later if a negative number flips it into the best).
   - Update `best = max(best, curr_max)`.

### Python code
```python
def maxProduct(nums):
    curr_max = curr_min = best = nums[0]

    for num in nums[1:]:
        if num < 0:
            curr_max, curr_min = curr_min, curr_max  # swap before updating

        curr_max = max(num, curr_max * num)
        curr_min = min(num, curr_min * num)

        best = max(best, curr_max)

    return best
```

### Line-by-line explanation
- `curr_max = curr_min = best = nums[0]` - all three start at the first element (a subarray of just one element is a valid starting point).
- `for num in nums[1:]:` - process the rest of the array.
- `if num < 0: curr_max, curr_min = curr_min, curr_max` - **the key insight**: if the current number is negative, multiplying it by the previous *maximum* product would produce a *small* (very negative) result, while multiplying it by the previous *minimum* (most negative) product would produce a *large* (very positive) result - so before computing the new max/min, we swap the roles, letting the same `max(...)`/`min(...)` formula below work correctly regardless of sign.
- `curr_max = max(num, curr_max * num)` - either this number alone is better than extending the previous subarray (e.g. if the previous running product was small/zero), or extending it (`curr_max * num`) is better - take the larger.
- `curr_min = min(num, curr_min * num)` - the mirrored calculation for the running minimum.
- `best = max(best, curr_max)` - update the overall best answer using the current position's best-ending-here product.

### Dry run
`nums = [2,3,-2,4]`

`curr_max=curr_min=best=2`.

- num=3 (positive, no swap): `curr_max=max(3, 2*3)=max(3,6)=6`. `curr_min=min(3,2*3)=min(3,6)=3`. `best=max(2,6)=6`.
- num=-2 (negative → swap first): swap `curr_max,curr_min = 3,6` (now curr_max=3, curr_min=6). `curr_max=max(-2, 3*-2)=max(-2,-6)=-2`. `curr_min=min(-2, 6*-2)=min(-2,-12)=-12`. `best=max(6,-2)=6`.
- num=4 (positive, no swap): `curr_max=max(4, -2*4)=max(4,-8)=4`. `curr_min=min(4,-12*4)=min(4,-48)=-48`. `best=max(6,4)=6`.

Final: `best = 6` ✅ matches expected output (subarray `[2,3]`, product 6).

**A dry run showing the swap's importance:** `nums = [-2,3,-4]`

`curr_max=curr_min=best=-2`.
- num=3 (positive): `curr_max=max(3,-2*3)=max(3,-6)=3`. `curr_min=min(3,-2*3)=min(3,-6)=-6`. `best=max(-2,3)=3`.
- num=-4 (negative → swap): swap `curr_max,curr_min = -6, 3` (curr_max=-6, curr_min=3). `curr_max=max(-4, -6*-4)=max(-4,24)=24`. `curr_min=min(-4, 3*-4)=min(-4,-12)=-12`. `best=max(3,24)=24`.

Final: `best = 24` ✅ (matches expected: `[-2,3,-4]` → `(-2)*3*(-4) = 24` - correctly found via the swap allowing the very negative running product from `curr_min` to flip into the new maximum).

### Time & space complexity
- **Time: O(n)** - a single pass.
- **Space: O(1)** - only a few running variables.

---

## Common mistakes & misconceptions

1. **Tracking only a running maximum, applying the "sum version" trick by habit.** As explained above, this is the single most important misconception for this exact problem - it's a very natural mistake to make coming straight from Maximum Subarray, and it produces *silently wrong* answers on inputs with negative numbers (not a crash, just a wrong number), making it especially easy to miss without deliberately testing negative-containing cases.
2. **Forgetting to swap `curr_max` and `curr_min` before recomputing them**, or swapping *after* instead of *before* - the swap must happen first, so that the subsequent `max(...)`/`min(...)` calls read the already-swapped values; swapping afterward (or not at all) uses stale values and produces incorrect results.
3. **Forgetting that a zero in the array resets everything.** `curr_max = max(num, curr_max * num)` naturally handles this correctly (multiplying by 0 always loses to starting fresh at `num` itself, or a subsequent nonzero value would restart from there) - but it's worth explicitly verifying this behavior with a dry run including a `0`, since it's a common edge case interviewers probe for.
4. **Not handling a single-element array correctly.** With `nums = [x]`, the loop over `nums[1:]` never executes, and the function correctly returns the initial `best = nums[0]` - worth confirming this trivial case explicitly rather than assuming the general logic "just works" without checking.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute force | O(n²) | O(1) | Checks every subarray directly. |
| Track running max & min | O(n) | O(1) | The standard, expected optimal solution. |

**Key takeaway:** whenever a running "best so far" DP involves **multiplication** (rather than addition), always consider whether tracking only a running *maximum* is enough - if negative numbers can appear, a running *minimum* often needs to be tracked alongside it, since a sign flip (via multiplying by a negative) can turn the worst running value into the best one on the very next step.
