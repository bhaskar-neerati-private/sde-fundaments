# 49. Climbing Stairs

**LeetCode:** [#70 - Climbing Stairs](https://leetcode.com/problems/climbing-stairs/) · **Topic:** [1-D Dynamic Programming](../topics/13-1d-dynamic-programming.md) · **Difficulty:** Easy

## Problem statement

You're climbing a staircase with `n` steps. Each move, you can climb **1 or 2 steps**. Return the number of **distinct ways** to reach the top.

**Example:**
```
Input: n = 3
Output: 3   (1+1+1, 1+2, 2+1)
```

## Applicable approaches

- **Brute Force Recursion (no memoization)** - exponential, but shows the natural recursive structure.
- **Top-down Memoization**.
- **Bottom-up Tabulation**.
- **Space-optimized (O(1) space)** - the optimal version.

## Approach 1: Brute Force Recursion

### Intuition
To reach step `n`, your very last move was either a **1-step** from step `n-1`, or a **2-step** from step `n-2`. So the number of ways to reach step `n` is simply the number of ways to reach step `n-1`, plus the number of ways to reach step `n-2` - this is exactly the Fibonacci recurrence.

### Python code
```python
def climbStairs(n):
    if n <= 2:
        return n
    return climbStairs(n - 1) + climbStairs(n - 2)
```

### Why this is slow
This recomputes the same sub-problems repeatedly - e.g. computing `climbStairs(5)` calls `climbStairs(4)` and `climbStairs(3)`, but `climbStairs(4)` *also* calls `climbStairs(3)` again independently, recomputing it from scratch. This branching redundancy compounds exponentially.

### Time & space complexity
- **Time: O(2^n)** - each call branches into two more calls, down to the base cases.
- **Space: O(n)** for the recursion call stack depth.

---

## Approach 2: Top-down Memoization

### Intuition
Add a cache: the first time we compute `climbStairs(k)` for any specific `k`, store the answer; every subsequent request for that same `k` is an instant lookup instead of a recomputation.

### Python code
```python
def climbStairs(n):
    memo = {}

    def dp(k):
        if k <= 2:
            return k
        if k in memo:
            return memo[k]
        memo[k] = dp(k - 1) + dp(k - 2)
        return memo[k]

    return dp(n)
```

### Line-by-line explanation
- `memo = {}` - cache mapping a step count to its already-computed answer.
- `if k in memo: return memo[k]` - the key addition over Approach 1: instant lookup instead of recomputation.
- `memo[k] = dp(k - 1) + dp(k - 2)` - compute and **store** the answer before returning it.

### Time & space complexity
- **Time: O(n)** - each distinct sub-problem (`k` from 1 to n) is computed exactly once.
- **Space: O(n)** for the memo dict plus the recursion call stack.

---

## Approach 3: Bottom-up Tabulation

### Intuition
Instead of recursing top-down, build the answers **iteratively** from the base cases up to `n` - no recursion, no call stack at all.

### Python code
```python
def climbStairs(n):
    if n <= 2:
        return n

    dp = [0] * (n + 1)
    dp[1], dp[2] = 1, 2

    for i in range(3, n + 1):
        dp[i] = dp[i - 1] + dp[i - 2]

    return dp[n]
```

### Line-by-line explanation
- `dp = [0] * (n + 1)` - a table where `dp[i]` will hold the number of ways to reach step `i`.
- `dp[1], dp[2] = 1, 2` - base cases: 1 way to reach step 1 (a single 1-step), 2 ways to reach step 2 (two 1-steps, or one 2-step).
- `for i in range(3, n + 1): dp[i] = dp[i-1] + dp[i-2]` - build up from the base cases, applying the same recurrence as before, iteratively.

### Dry run
`n = 5`: `dp[1]=1, dp[2]=2`. `dp[3]=dp[2]+dp[1]=2+1=3`. `dp[4]=dp[3]+dp[2]=3+2=5`. `dp[5]=dp[4]+dp[3]=5+3=8`. Return `8` ✅ (matches the known Fibonacci-shaped answer sequence).

### Time & space complexity
- **Time: O(n)**, **Space: O(n)** for the table.

---

## Approach 4: Optimal - Space-Optimized (O(1) Space)

### Intuition
Look closely at the tabulation loop: `dp[i]` only ever depends on the **two most recent** values, `dp[i-1]` and `dp[i-2]` - never anything further back. So there's no need to keep the entire table around; just track the last two values in simple variables.

### Python code
```python
def climbStairs(n):
    if n <= 2:
        return n

    prev2, prev1 = 1, 2  # ways to reach step 1, step 2

    for _ in range(3, n + 1):
        prev2, prev1 = prev1, prev1 + prev2

    return prev1
```

### Line-by-line explanation
- `prev2, prev1 = 1, 2` - initialize to `dp[1]` and `dp[2]`.
- `for _ in range(3, n + 1):` - iterate the same number of times as the tabulated version, but without indexing into an array.
- `prev2, prev1 = prev1, prev1 + prev2` - Python evaluates the right-hand side **entirely** before assigning, so this correctly computes the new `prev1` (the current step's answer, `dp[i-1] + dp[i-2]`, using the *old* values of both variables) while simultaneously shifting `prev2` forward to what used to be `prev1` - a clean one-line update.

### Time & space complexity
- **Time: O(n)**, **Space: O(1)** - only two variables, regardless of how large `n` is. This is the optimal solution.

---

## Common mistakes & misconceptions

1. **Not recognizing this is Fibonacci in disguise.** Because the problem is phrased in terms of stairs, not numbers, beginners sometimes try to enumerate every path explicitly (e.g. generating all sequences of 1s and 2s that sum to `n`) instead of noticing that only the **count** of ways matters, and that count satisfies the exact same recurrence as Fibonacci - once you see `ways(n) = ways(n-1) + ways(n-2)`, the entire DP toolkit from the topic overview applies directly.
2. **Off-by-one errors in the base cases.** `dp[1] = 1` and `dp[2] = 2` are easy to swap or misremember - a wrong base case doesn't crash the program, it just silently produces a wrong (but plausible-looking) number for every step built on top of it, which is exactly the kind of bug the topic overview warns is hard to notice.
3. **Believing memoization and tabulation are fundamentally different algorithms.** As the topic overview stresses, they're the *same* recurrence, just computed top-down-with-cache versus bottom-up-with-a-loop - if you understand one, deriving the other is mechanical, not a separate thing to memorize.
4. **Attempting the O(1) space optimization before confirming the O(n) tabulated version is correct.** It's easy to introduce a subtle bug in the "shift the two tracked variables forward" line under time pressure - verify correctness with the full table first, exactly as the topic overview recommends.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute force recursion | O(2^n) | O(n) | Exponential - recomputes overlapping sub-problems repeatedly. |
| Top-down memoization | O(n) | O(n) | Same recursive shape, but cached. |
| Bottom-up tabulation | O(n) | O(n) | Iterative, no recursion/call stack risk. |
| Space-optimized | O(n) | O(1) | The optimal solution - only needs the last two values. |

**Key takeaway:** this problem is the cleanest possible illustration of the full DP progression - from exponential brute force recursion, through memoization, through tabulation, down to a space-optimized O(1) solution - and recognizing "this problem's recurrence only looks at the last k values" is exactly what unlocks that final space optimization, a pattern that reappears throughout this entire topic.
