# 54. House Robber

**LeetCode:** [#198 - House Robber](https://leetcode.com/problems/house-robber/) · **Topic:** [1-D Dynamic Programming](../topics/13-1d-dynamic-programming.md) · **Difficulty:** Medium

## Problem statement

Given an array `nums` representing money in houses along a street, return the maximum amount you can rob **without robbing two adjacent houses** (robbing adjacent houses triggers an alarm).

**Example:**
```
Input: nums = [2,7,9,3,1]
Output: 12   (rob house 0, house 2, house 4: 2+9+1=12)
```

## Applicable approaches

- **Pure Recursion** - exponential, shown for intuition.
- **Top-down Memoization.**
- **Bottom-up Tabulation.**
- **Space-optimized (O(1) space)** - the optimal version.

## Approach 1: Pure Recursion

### Intuition
At each house, you have exactly two choices: **rob it** (and skip the next house, since robbing adjacent houses isn't allowed) or **skip it** (and move to the next house freely). Take whichever choice yields more money.

### Python code
```python
def rob(nums):
    def dp(i):
        if i >= len(nums):
            return 0
        rob_this = nums[i] + dp(i + 2)   # rob house i, skip house i+1
        skip_this = dp(i + 1)             # don't rob house i
        return max(rob_this, skip_this)

    return dp(0)
```

### Time & space complexity
- **Time: O(2^n)** - branches into two choices at every house.
- **Space: O(n)** recursion depth.

---

## Approach 2: Top-down Memoization

### Python code
```python
def rob(nums):
    memo = {}

    def dp(i):
        if i >= len(nums):
            return 0
        if i in memo:
            return memo[i]

        memo[i] = max(nums[i] + dp(i + 2), dp(i + 1))
        return memo[i]

    return dp(0)
```

### Time & space complexity
- **Time: O(n)**, **Space: O(n)**.

---

## Approach 3: Bottom-up Tabulation

### Intuition
Define `dp[i]` = the maximum amount robbable using only houses `0` through `i`. At each house, the best answer is either: skip it (`dp[i-1]`), or rob it plus whatever was best two houses back (`nums[i] + dp[i-2]`).

### Python code
```python
def rob(nums):
    n = len(nums)
    if n == 1:
        return nums[0]

    dp = [0] * n
    dp[0] = nums[0]
    dp[1] = max(nums[0], nums[1])

    for i in range(2, n):
        dp[i] = max(dp[i - 1], nums[i] + dp[i - 2])

    return dp[n - 1]
```

### Line-by-line explanation
- `dp[0] = nums[0]` - only one house, rob it.
- `dp[1] = max(nums[0], nums[1])` - with two houses, rob whichever is worth more (can't rob both, they're adjacent).
- `dp[i] = max(dp[i-1], nums[i] + dp[i-2])` - for each subsequent house: either skip it (carry forward the best answer using houses up to `i-1`), or rob it (add its value to the best answer using houses up to `i-2`, since house `i-1` must be skipped if we rob house `i`).

### Dry run
`nums = [2,7,9,3,1]`

`dp[0]=2`. `dp[1]=max(2,7)=7`. `dp[2]=max(dp[1], nums[2]+dp[0])=max(7, 9+2)=max(7,11)=11`. `dp[3]=max(dp[2], nums[3]+dp[1])=max(11, 3+7)=max(11,10)=11`. `dp[4]=max(dp[3], nums[4]+dp[2])=max(11, 1+11)=max(11,12)=12`.

Final: `dp[4]=12` ✅ (rob houses 0, 2, 4: values 2+9+1=12).

### Time & space complexity
- **Time: O(n)**, **Space: O(n)**.

---

## Approach 4: Optimal - Space-Optimized (O(1) Space)

### Intuition
`dp[i]` only ever depends on `dp[i-1]` and `dp[i-2]` - exactly like Climbing Stairs - so we only need to track the last two values.

### Python code
```python
def rob(nums):
    prev2, prev1 = 0, 0  # dp[i-2], dp[i-1], conceptually before any houses

    for num in nums:
        prev2, prev1 = prev1, max(prev1, num + prev2)

    return prev1
```

### Line-by-line explanation
- `prev2, prev1 = 0, 0` - represents `dp[-2]` and `dp[-1]` (before any real houses), both trivially 0 (no houses, no money).
- `for num in nums:` - process each house's value in order.
- `prev2, prev1 = prev1, max(prev1, num + prev2)` - the new "best up to and including this house" is either the previous best (skip this house) or this house's value plus the best from two houses back (rob this house); simultaneously shift `prev2` forward.

### Dry run
`nums = [2,7,9,3,1]`: start `prev2=0, prev1=0`.
- num=2: `prev2,prev1 = 0, max(0, 2+0)=2` → `(0,2)`.
- num=7: `prev2,prev1 = 2, max(2, 7+0)=7` → `(2,7)`.
- num=9: `prev2,prev1 = 7, max(7, 9+2)=11` → `(7,11)`.
- num=3: `prev2,prev1 = 11, max(11, 3+7)=11` → `(11,11)`.
- num=1: `prev2,prev1 = 11, max(11, 1+11)=12` → `(11,12)`.

Return `prev1 = 12` ✅

### Time & space complexity
- **Time: O(n)**, **Space: O(1)** - the optimal solution.

---

## Common mistakes & misconceptions

1. **Believing a greedy "always rob the more valuable of every adjacent pair" strategy works.** It doesn't - e.g. `[2,1,1,20]`: greedily comparing house 0 (2) vs house 1 (1) picks house 0, but the true optimal robs houses 1 and 3 (1+20=21), which greedy would never find by only comparing immediate neighbors. Only checking *all* valid combinations (via DP) is correct.
2. **Confusing `dp[i]` ("best using houses 0..i") with "must rob house i."** `dp[i]` represents the best achievable using only the *first* `i+1` houses as *available* options, not a requirement to rob house `i` itself - this is exactly why the recurrence takes a `max` between robbing and skipping.
3. **Forgetting the `n == 1` edge case** in the tabulated version, where `dp[1]` would be undefined/out-of-bounds if not guarded separately before the main loop begins.
4. **Mixing up which index feeds into "rob" vs. "skip."** `nums[i] + dp[i-2]` (rob, skip the immediately preceding house) is easy to accidentally write as `dp[i-1]` (which would incorrectly allow robbing two adjacent houses) - always double check the index arithmetic against the adjacency constraint.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Pure recursion | O(2^n) | O(n) | Exponential. |
| Top-down memoization | O(n) | O(n) | Cached recursion. |
| Bottom-up tabulation | O(n) | O(n) | Iterative. |
| Space-optimized | O(n) | O(1) | The optimal solution. |

**Key takeaway:** "rob it or skip it, can't use two adjacent" is the archetypal "include or exclude the current element" DP pattern - at every position, compare the best result *with* the current element against the best result *without* it, and this exact "rob vs. skip" shape reappears (with a twist) in House Robber II next.
