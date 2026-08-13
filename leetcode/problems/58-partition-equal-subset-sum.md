# 58. Partition Equal Subset Sum

**LeetCode:** [#416 - Partition Equal Subset Sum](https://leetcode.com/problems/partition-equal-subset-sum/) · **Topic:** [1-D Dynamic Programming](../topics/13-1d-dynamic-programming.md) · **Difficulty:** Medium

## Problem statement

Given an integer array `nums` (all positive), determine if it can be partitioned into **two subsets** such that the sum of elements in both subsets is **equal**.

**Example:**
```
Input: nums = [1,5,11,5]
Output: true  ([1,5,5] and [11], both sum to 11)
```

## Applicable approaches

- **Brute Force Recursion / Backtracking** - try including or excluding each number.
- **Recursion through Memoization.**
- **Bottom-up DP (Subset Sum) - the standard, expected optimal solution.**
- **Space-optimized (1-D rolling array)** - a further refinement.

## Key reframing: this is really "Subset Sum"

If the two subsets must have equal sums, and the total sum of all numbers is `S`, then each subset must sum to exactly `S / 2`. **First check: if `S` is odd, it's immediately impossible** (can't split an odd total into two equal integer halves) - return `False` right away. Otherwise, the problem becomes: **"does some subset of `nums` sum to exactly `S / 2`?"** - a classic **Subset Sum** problem (closely related to a 0/1 Knapsack problem, where each item can be used at most once).

## Approach 1: Brute Force Recursion / Backtracking

### Intuition
For each number, decide whether to include it in "subset A" or not (implicitly, everything not in A is in "subset B") - recursively check if some choice sequence hits the target sum exactly.

### Python code
```python
def canPartition(nums):
    total = sum(nums)
    if total % 2 != 0:
        return False
    target = total // 2

    def canReach(i, remaining):
        if remaining == 0:
            return True
        if i == len(nums) or remaining < 0:
            return False
        return canReach(i + 1, remaining - nums[i]) or canReach(i + 1, remaining)

    return canReach(0, target)
```

### Time & space complexity
- **Time: O(2^n)** - two choices (include/exclude) at every element.
- **Space: O(n)** recursion depth.

---

## Approach 2: Memoization

### Python code
```python
def canPartition(nums):
    total = sum(nums)
    if total % 2 != 0:
        return False
    target = total // 2

    memo = {}

    def canReach(i, remaining):
        if remaining == 0:
            return True
        if i == len(nums) or remaining < 0:
            return False
        if (i, remaining) in memo:
            return memo[(i, remaining)]

        result = canReach(i + 1, remaining - nums[i]) or canReach(i + 1, remaining)
        memo[(i, remaining)] = result
        return result

    return canReach(0, target)
```

### Time & space complexity
- **Time: O(n · target)** - each distinct `(i, remaining)` pair computed once.
- **Space: O(n · target)** for the memo, plus O(n) recursion depth.

---

## Approach 3: Optimal - Bottom-up Subset Sum DP

### Intuition
Define `dp[s]` = "can some subset of the numbers processed so far sum to exactly `s`?" This is a **2-D-looking** problem (which numbers have been considered, and what sum is being targeted) that can be collapsed into a clever **1-D rolling array**, processed carefully: for each number, update the `dp` array **from high sums down to low sums** (right to left) - this ordering ensures each number is only ever "used" **once** per subset (avoiding accidentally reusing the same number multiple times within a single subset, which isn't allowed here).

### Algorithm
1. Compute `target = sum(nums) // 2` (after checking the total is even).
2. `dp = [False] * (target + 1)`, with `dp[0] = True` (a sum of 0 is always achievable, using no numbers).
3. For each number `num` in `nums`: for each possible sum `s` from `target` down to `num` (**descending**, crucial - see explanation below): `dp[s] = dp[s] or dp[s - num]`.
4. Return `dp[target]`.

### Python code
```python
def canPartition(nums):
    total = sum(nums)
    if total % 2 != 0:
        return False
    target = total // 2

    dp = [False] * (target + 1)
    dp[0] = True

    for num in nums:
        for s in range(target, num - 1, -1):
            dp[s] = dp[s] or dp[s - num]

    return dp[target]
```

### Line-by-line explanation
- `dp[0] = True` - the empty subset always sums to 0.
- `for num in nums:` - process each number one at a time.
- `for s in range(target, num - 1, -1):` - **iterate `s` from `target` down to `num`, descending.** This direction is essential: if we went *ascending* instead, updating `dp[s]` using `dp[s - num]` could accidentally use an update that *already includes* the current `num` (since `s - num` might have just been updated earlier in the *same* pass over this same number), effectively allowing `num` to be used more than once in forming a single subset - which isn't allowed (each number can only be used once, since it physically only exists once in the array). Going descending guarantees `dp[s - num]` still reflects the state *before* this number was considered, giving the correct "0/1" (use it or don't, exactly once) behavior.
- `dp[s] = dp[s] or dp[s - num]` - a sum of `s` is achievable either if it was already achievable without this number (`dp[s]`, unchanged from before), or if `s - num` was achievable without this number and we now add `num` to reach `s` (`dp[s - num]`).
- `return dp[target]` - is the target sum achievable using some subset of all the numbers?

### Dry run
`nums = [1,5,11,5]`, `total=22`, `target=11`

`dp = [T,F,F,F,F,F,F,F,F,F,F,F]` (index 0 True, rest False; indices 0-11)

- `num=1`: `s` from 11 down to 1: `dp[1] = dp[1] or dp[0] = False or True = True`. (all other `s` from 2-11: `dp[s-1]` is False except when s-1=0 i.e s=1, already handled). After: `dp[0]=T, dp[1]=T`, rest False.
- `num=5`: `s` from 11 down to 5: `dp[5]=dp[5] or dp[0]=T`. `dp[6]=dp[6] or dp[1]=T`. Others (s=7..11) check `dp[s-5]` for s-5 in {2,3,4,5,6}: only relevant ones already False at this point except we just set dp[5],dp[6] in *this same descending pass* - but since we're going descending and 5,6 < 7..11's dependencies check lower indices already processed earlier in *this* descending sweep (s=11 checks dp[6] which hasn't been updated yet at that point since we go from 11 down to 5, so s=11 is processed BEFORE s=6 sets dp[6] - wait descending means s=11 first, then 10,9,8,7,6,5. So when processing s=11 (checks dp[6], which is still False, the old value before this num's pass) - correctly uses the pre-this-num value). Let's just state result: after num=5: `dp[0]=T,dp[1]=T,dp[5]=T,dp[6]=T`, rest False.
- `num=11`: `s` from 11 down to 11: `dp[11] = dp[11] or dp[0] = False or True = True`.
- `num=5` (second 5): `s` from 11 down to 5: `dp[11]=dp[11] or dp[6] = True or True = True` (already true). `dp[10]=dp[10] or dp[5]=False or True=True`. `dp[6]=dp[6] or dp[1]=True or True=True` (already true). Others stay as computed.

Final `dp[11] = True` ✅ (achieved via 1+5+5=11, using the two 5's and the 1 - correctly allowed since both 5's are separate array elements, each usable once, and this DP's descending-order update correctly permits using each exactly once).

### Time & space complexity
- **Time: O(n · target)** where target = sum(nums)/2 - for each of n numbers, iterate through up to `target` possible sums.
- **Space: O(target)** for the 1-D `dp` array - already space-optimized compared to a naive 2-D `dp[i][s]` table (which would be O(n · target)).

---

## Common mistakes & misconceptions

1. **Iterating the sum dimension ascending instead of descending in the 1-D rolling DP.** This is the single most important and most commonly tested subtlety in this problem - ascending order silently allows a single array element to be "used" multiple times within one subset (turning 0/1 knapsack into unbounded knapsack), producing wrong (too permissive) results without any crash or obvious symptom.
2. **Forgetting the odd-total early exit.** Skipping the `total % 2 != 0` check and proceeding to search for a subset summing to `total // 2` (using integer division, silently rounding down) would search for the *wrong* target sum on odd totals, potentially returning a wrong `True`.
3. **Confusing this problem with Combination Sum IV / Coin Change's "unlimited reuse" DP shape.** Those problems allow reusing the same value indefinitely (loop nesting: amount outer, item inner, ascending); this problem restricts each *array element* to being used at most once (loop nesting: item outer, amount inner, ascending in the naive 2D version but **descending** in the space-optimized 1-D version) - conflating the two shapes is a very common source of bugs when problems are solved back-to-back.
4. **Believing the two `5`s in `nums = [1,5,11,5]` are indistinguishable and only "half-counting" duplicate values.** Even though both 5s have the same numeric value, they're separate array elements, each independently eligible to be included or excluded - the subset-sum DP correctly treats them as two distinct usable items (both contributing toward the achievable-sums count) precisely because it processes `nums` element by element, not value by distinct-value.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute force recursion | O(2^n) | O(n) | Exponential. |
| Memoization | O(n·target) | O(n·target) | Cached, but more memory than needed. |
| Bottom-up 1-D subset sum DP | O(n·target) | O(target) | The standard, expected optimal solution - note the essential descending iteration order. |

**Key takeaway:** "can some subset achieve exactly sum X" is the classic **Subset Sum** / 0-1 Knapsack shape - and when collapsing a 2-D DP (item index × target sum) into a 1-D rolling array, the **direction you iterate the sum dimension matters enormously**: descending order is required whenever each item can only be used once (0/1 knapsack-style), while ascending order would be used instead if items could be reused unlimited times (like Coin Change) - this distinction is one of the most commonly tested subtleties in this entire DP topic.
