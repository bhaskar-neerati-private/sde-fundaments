# 53. Combination Sum IV

**LeetCode:** [#377 - Combination Sum IV](https://leetcode.com/problems/combination-sum-iv/) · **Topic:** [1-D Dynamic Programming](../topics/13-1d-dynamic-programming.md) · **Difficulty:** Medium

## Problem statement

Given an array of **distinct** positive integers `nums` and a `target`, return the number of possible **ordered combinations** (i.e. different orderings count as different sequences - this is really counting arrangements, not combinations in the strict sense, despite the problem's name) that add up to `target`. Numbers may be reused.

**Example:**
```
Input: nums = [1,2,3], target = 4
Output: 7
Explanation: (1,1,1,1),(1,1,2),(1,2,1),(1,3),(2,1,1),(2,2),(3,1)
```

**Important distinction from Combination Sum (#39):** that problem counted `[2,2,3]` as the same as `[3,2,2]` (order doesn't matter, standard combinations). This problem counts them as **different** (order matters - these are really permutation-style counts) - this single difference completely changes the DP structure needed, as shown below.

## Applicable approaches

- **Brute Force Recursion (try every next number, at every step).**
- **Top-down Memoization.**
- **Bottom-up Tabulation** - the standard, expected optimal solution.

## Approach 1: Brute Force Recursion

### Intuition
For a given `remaining` target, try using **each** number in `nums` as the *next* number in the sequence (not restricted to a "starting index" this time - since order matters, every number can be tried at every position, unlike Combination Sum's index-restricted approach), and sum up the counts from each choice.

### Python code
```python
def combinationSum4(nums, target):
    def count(remaining):
        if remaining == 0:
            return 1
        if remaining < 0:
            return 0

        total = 0
        for num in nums:
            total += count(remaining - num)
        return total

    return count(target)
```

### Why no "start index" is needed here (unlike Combination Sum #39)
Since `(1,1,2)` and `(1,2,1)` and `(2,1,1)` should all be counted as **different**, we must allow trying **any** number at **every** step, regardless of what was chosen before - there's no "only look forward from here" restriction like in the standard combinations problem.

### Time & space complexity
- **Time: O(len(nums)^target)** in the worst case - massive branching without memoization.
- **Space: O(target)** recursion depth.

---

## Approach 2: Top-down Memoization

### Python code
```python
def combinationSum4(nums, target):
    memo = {}

    def count(remaining):
        if remaining == 0:
            return 1
        if remaining < 0:
            return 0
        if remaining in memo:
            return memo[remaining]

        total = 0
        for num in nums:
            total += count(remaining - num)

        memo[remaining] = total
        return total

    return count(target)
```

### Time & space complexity
- **Time: O(target · len(nums))** - each distinct `remaining` value computed once.
- **Space: O(target)**.

---

## Approach 3: Optimal - Bottom-up Tabulation

### Intuition
Define `dp[i]` = the number of ordered sequences of numbers from `nums` that sum to exactly `i`. Build up from `dp[0] = 1` (there's exactly one way to make sum 0 - use zero numbers) up to `dp[target]`, where `dp[i]` is the sum, over every number `num` in `nums`, of `dp[i - num]` (any valid sequence summing to `i - num`, with `num` appended as the *new last element*, gives a valid sequence summing to `i`).

### Python code
```python
def combinationSum4(nums, target):
    dp = [0] * (target + 1)
    dp[0] = 1

    for i in range(1, target + 1):
        for num in nums:
            if num <= i:
                dp[i] += dp[i - num]

    return dp[target]
```

### Line-by-line explanation
- `dp[0] = 1` - base case: exactly one way to form sum 0 (the empty sequence).
- `for i in range(1, target + 1):` - build every amount's count, in increasing order (since `dp[i]` only depends on smaller amounts).
- `for num in nums: if num <= i: dp[i] += dp[i - num]` - **crucially, this is the OUTER loop over amounts, with numbers as the INNER loop** (the reverse nesting from what a "combinations, order doesn't matter" DP would use) - this ordering is exactly what allows every distinct ordering to be counted separately: for each amount `i`, we consider "what could the *last* number added have been," trying every possibility, and each choice of last-number, combined with however many ways the *remainder* could be formed (in any order), contributes to the count.

### Why the loop order matters (contrast with a "combinations" DP)
If numbers were the **outer** loop and amounts the **inner** loop (the structure typically used for "count unordered combinations," like a coin-change-counting variant), each number would only ever be considered as extending sequences built using *only numbers processed so far* - which forces a canonical order onto each combination and only counts each *set* of numbers once, regardless of arrangement. Making amount the outer loop instead removes that ordering restriction entirely, since at every amount, *all* numbers are reconsidered as valid "next" choices - which is exactly the "order matters" behavior this problem needs.

### Dry run
`nums = [1,2,3]`, `target = 4`

`dp[0]=1`.
- `i=1`: num=1(≤1): `dp[1]+=dp[0]=1` → `dp[1]=1`. num=2,3 >1, skip. `dp[1]=1`.
- `i=2`: num=1: `dp[2]+=dp[1]=1`→`dp[2]=1`. num=2: `dp[2]+=dp[0]=1`→`dp[2]=2`. num=3>2 skip. `dp[2]=2`.
- `i=3`: num=1: `dp[3]+=dp[2]=2`→`dp[3]=2`. num=2: `dp[3]+=dp[1]=1`→`dp[3]=3`. num=3: `dp[3]+=dp[0]=1`→`dp[3]=4`. `dp[3]=4`.
- `i=4`: num=1: `dp[4]+=dp[3]=4`→`dp[4]=4`. num=2: `dp[4]+=dp[2]=2`→`dp[4]=6`. num=3: `dp[4]+=dp[1]=1`→`dp[4]=7`. `dp[4]=7`.

Final: `dp[4] = 7` ✅ matches expected output exactly.

### Time & space complexity
- **Time: O(target · len(nums))**, **Space: O(target)**.

---

## Common mistakes & misconceptions

1. **Copying Coin Change's or a "combinations" DP's loop nesting (numbers outer, amount inner) out of habit.** As explained above, that nesting order forces a canonical ordering and undercounts this problem, which specifically wants every ordering counted separately - always re-derive the correct nesting from what the problem is actually asking, rather than pattern-matching to a superficially similar problem.
2. **Misreading the problem name.** "Combination Sum IV" is a misleading name - despite "combination" in the title, this problem is really counting **permutations** (ordered arrangements), not combinations in the mathematical sense - the explicit example `(1,2)` and `(2,1)` both counting separately is the tell.
3. **Forgetting `dp[0] = 1`, not `0`.** It's tempting to think "zero ways to make sum zero," but the correct base case is **one** way (the empty sequence) - getting this wrong makes every subsequent `dp[i]` come out to 0, since the entire table is built multiplicatively from this base case via addition chains rooted at it.
4. **Not considering that `target` values with no valid `nums` combination correctly stay at `dp[i] = 0`** without needing any special-case handling - unlike Coin Change's `-1` sentinel for "impossible," Combination Sum IV's "impossible" case naturally falls out as `0`, requiring no extra post-processing.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute force recursion | O(len(nums)^target) | O(target) | Exponential. |
| Top-down memoization | O(target·len(nums)) | O(target) | Same shape, cached. |
| Bottom-up tabulation | O(target·len(nums)) | O(target) | The standard, expected solution. |

**Key takeaway:** this problem is deceptively similar to Coin Change / Combination Sum, but the "order matters" requirement flips the natural DP loop nesting (amount as the outer loop, choices as the inner loop) compared to a same-looking "count unordered combinations" DP - always check carefully whether a counting problem wants ordered sequences (permutations) or unordered groupings (combinations), since it changes which loop goes on the outside.
