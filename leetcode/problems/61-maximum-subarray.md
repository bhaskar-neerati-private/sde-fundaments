# 61. Maximum Subarray

**LeetCode:** [#53 - Maximum Subarray](https://leetcode.com/problems/maximum-subarray/) · **Topic:** [Greedy](../topics/15-greedy.md) · **Difficulty:** Medium

## Problem statement

Given an integer array `nums`, find the **contiguous subarray** (containing at least one number) with the largest sum, and return that sum.

**Example:**
```
Input: nums = [-2,1,-3,4,-1,2,1,-5,4]
Output: 6   ([4,-1,2,1])
```

## Applicable approaches

- **Brute Force - Check every subarray.**
- **Recursion / DP variants (top-down and bottom-up).**
- **Optimal - Kadane's Algorithm (Greedy).** The standard, expected, most elegant solution.
- **Bonus - Divide and Conquer.** A further alternative, worth knowing exists.

## Approach 1: Brute Force

### Intuition
Check the sum of every possible contiguous subarray directly.

### Python code
```python
def maxSubArray(nums):
    n = len(nums)
    best = nums[0]

    for i in range(n):
        current_sum = 0
        for j in range(i, n):
            current_sum += nums[j]
            best = max(best, current_sum)

    return best
```

### Time & space complexity
- **Time: O(n²)**, **Space: O(1)**.

---

## Approach 2: DP Variants

### Intuition
Define `dp[i]` = the maximum sum of a subarray **ending exactly at index i**. At each position, either extend the previous best-ending-here subarray (`dp[i-1] + nums[i]`), or start fresh at `nums[i]` alone (if the previous running sum was actually dragging the total down, i.e. negative). The final answer is the max over the whole `dp` array.

### Python code (bottom-up tabulation)
```python
def maxSubArray(nums):
    n = len(nums)
    dp = [0] * n
    dp[0] = nums[0]
    best = dp[0]

    for i in range(1, n):
        dp[i] = max(nums[i], dp[i - 1] + nums[i])
        best = max(best, dp[i])

    return best
```

### Time & space complexity
- **Time: O(n)**, **Space: O(n)** for the `dp` array (though this can trivially be reduced to O(1), since `dp[i]` only ever needs `dp[i-1]` - which is exactly what Kadane's algorithm below is).

---

## Approach 3: Optimal - Kadane's Algorithm (Greedy)

### Intuition
This is exactly the space-optimized version of the DP idea above, framed as a greedy rule: keep a running sum as you scan left to right. At each element, **greedily decide**: is it better to extend the current run, or is the current run actually *hurting* (negative), in which case it's better to abandon it and start fresh from this element? Track the best sum seen at any point during the scan.

### Algorithm
1. Initialize `current_sum = nums[0]` and `best = nums[0]`.
2. For each subsequent number `num`:
   - `current_sum = max(num, current_sum + num)` - either start fresh here, or extend the running sum, whichever is larger.
   - `best = max(best, current_sum)`.
3. Return `best`.

### Python code
```python
def maxSubArray(nums):
    current_sum = best = nums[0]

    for num in nums[1:]:
        current_sum = max(num, current_sum + num)
        best = max(best, current_sum)

    return best
```

### Line-by-line explanation
- `current_sum = best = nums[0]` - both start at the first element (a single-element subarray is always a valid starting candidate).
- `for num in nums[1:]:` - scan the rest of the array once.
- `current_sum = max(num, current_sum + num)` - **the greedy decision**: if the running sum so far (`current_sum`) is negative, adding it to `num` would only make things worse than starting fresh - so compare "start fresh at `num` alone" against "extend the run by adding `num`," and take whichever is larger. This is greedy because we commit to this decision permanently and never revisit it.
- `best = max(best, current_sum)` - track the best sum seen at any point, since the best subarray might not end at the very last position.

### Why greedy is provably safe here (brief justification)
If `current_sum` ever becomes negative, it can **never** help any future subarray to include it - adding a negative number to any future sum only makes that future sum smaller than it would've been without it. So the moment the running sum goes negative, discarding it entirely (starting fresh) is guaranteed to be at least as good as keeping it - this is exactly the kind of "locally optimal choice never hurts globally" property that makes a greedy strategy valid.

### Dry run
`nums = [-2,1,-3,4,-1,2,1,-5,4]`

`current_sum=best=-2`.

| num | current_sum = max(num, current_sum+num) | best |
|---|---|---|
| 1 | max(1, -2+1=-1) = 1 | max(-2,1)=1 |
| -3 | max(-3, 1-3=-2) = -2 | max(1,-2)=1 |
| 4 | max(4, -2+4=2) = 4 | max(1,4)=4 |
| -1 | max(-1, 4-1=3) = 3 | max(4,3)=4 |
| 2 | max(2, 3+2=5) = 5 | max(4,5)=5 |
| 1 | max(1, 5+1=6) = 6 | max(5,6)=6 |
| -5 | max(-5, 6-5=1) = 1 | max(6,1)=6 |
| 4 | max(4, 1+4=5) = 5 | max(6,5)=6 |

Final: `best = 6` ✅ matches expected output (subarray `[4,-1,2,1]`, sum 6).

### Time & space complexity
- **Time: O(n)** - a single pass.
- **Space: O(1)** - two running variables. This is the optimal solution.

---

## Bonus: Divide and Conquer

### Intuition
Split the array in half. The maximum subarray either lies entirely in the left half, entirely in the right half, or **crosses the midpoint**. Recursively find the best in each half, and separately compute the best "crossing" subarray (by extending outward from the midpoint in both directions), then take the best of all three. Worth knowing this exists (and that it achieves the same O(n) time as Kadane's when analyzed carefully... actually classic divide and conquer here is O(n log n), slightly worse than Kadane's O(n) - included as a well-known alternative technique, not because it's more efficient).

### Time & space complexity
- **Time: O(n log n)**, **Space: O(log n)** for the recursion.

---

## Common mistakes & misconceptions

1. **Resetting `current_sum` to 0 instead of to `num` when starting fresh.** The correct reset is "start a brand-new subarray consisting of just this element" (`current_sum = num`), not "reset to zero and separately add `num`" - though these end up numerically equivalent in this specific formula (`max(num, current_sum + num)` already handles it correctly), it's a common conceptual confusion, and matters more directly in variant problems.
2. **Forgetting the array can contain all-negative numbers**, and that the problem requires the subarray to be non-empty (at least one number) - the answer in an all-negative array is simply the single largest (least negative) element, not 0. Initializing `best` to `nums[0]` (as shown) rather than `0` correctly handles this; initializing to `0` would incorrectly allow "the empty subarray" to compete and win.
3. **Believing Kadane's algorithm and the "reset when negative" DP formulation are two different techniques to learn separately.** As the key takeaway notes, they're the *same* algorithm - failing to see this connection means re-deriving the logic from scratch for every superficially different framing.
4. **Assuming this problem needs the maximum-tracking approach from Maximum Product Subarray (tracking both max AND min).** It doesn't - that dual-tracking is specifically required for *products*, where a sign flip can turn a very negative running value into the best positive one; for *sums*, adding a negative number only ever makes a running total smaller, never bigger, so tracking just the running maximum is sufficient here.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute force | O(n²) | O(1) | Checks every subarray. |
| DP (tabulated) | O(n) | O(n) | Correct, but uses more space than necessary. |
| Kadane's algorithm (greedy) | O(n) | O(1) | The standard, optimal, most elegant solution. |
| Divide and conquer | O(n log n) | O(log n) | A well-known alternative, not more efficient here, but a good exercise. |

**Key takeaway:** Kadane's algorithm is a great example of how a greedy strategy and a space-optimized DP solution can be **the exact same algorithm**, just described through two different lenses - the "reset if the running sum goes negative" rule is both a valid greedy choice (provably never hurts) and the natural space-optimization of the DP formulation.
