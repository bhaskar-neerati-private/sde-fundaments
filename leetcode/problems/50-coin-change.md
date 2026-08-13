# 50. Coin Change

**LeetCode:** [#322 - Coin Change](https://leetcode.com/problems/coin-change/) · **Topic:** [1-D Dynamic Programming](../topics/13-1d-dynamic-programming.md) · **Difficulty:** Medium

## Problem statement

Given coin denominations `coins` and a `target` amount, return the **fewest number of coins** needed to make up that amount (unlimited supply of each coin). If it's impossible, return `-1`.

**Example:**
```
Input: coins = [1,2,5], amount = 11
Output: 3   (5 + 5 + 1)
```

## Applicable approaches

- **Brute Force Recursion** - try every coin at every step.
- **Top-down Memoization**.
- **Bottom-up Tabulation** - the standard, expected optimal solution.

## Approach 1: Brute Force Recursion

### Intuition
For a given remaining amount, try using each coin denomination as the "next" coin, and recursively solve for the smaller remaining amount - take the best (fewest total coins) result across all choices.

### Python code
```python
def coinChange(coins, amount):
    def dp(remaining):
        if remaining == 0:
            return 0
        if remaining < 0:
            return float("inf")  # impossible from here

        best = float("inf")
        for coin in coins:
            best = min(best, dp(remaining - coin) + 1)
        return best

    result = dp(amount)
    return result if result != float("inf") else -1
```

### Time & space complexity
- **Time: O(coins^amount)** in the worst case - massive branching, recomputes the same `remaining` values repeatedly from different paths.
- **Space: O(amount)** recursion depth.

---

## Approach 2: Top-down Memoization

### Intuition
The same recursive structure, but caching each `remaining` amount's answer the first time it's computed.

### Python code
```python
def coinChange(coins, amount):
    memo = {}

    def dp(remaining):
        if remaining == 0:
            return 0
        if remaining < 0:
            return float("inf")
        if remaining in memo:
            return memo[remaining]

        best = float("inf")
        for coin in coins:
            best = min(best, dp(remaining - coin) + 1)

        memo[remaining] = best
        return best

    result = dp(amount)
    return result if result != float("inf") else -1
```

### Time & space complexity
- **Time: O(amount · len(coins))** - each distinct `remaining` value (0 to amount) computed once, trying each coin.
- **Space: O(amount)** for the memo and recursion depth.

---

## Approach 3: Optimal - Bottom-up Tabulation

### Intuition
Build the answer for every amount from `0` up to the target, iteratively. `dp[amount]` = the fewest coins needed to make exactly `amount`, computed from smaller already-solved amounts.

### Algorithm
1. Create `dp` array of size `amount + 1`, initialized to a large "impossible" sentinel (e.g. `amount + 1`, guaranteed larger than any real valid answer since you'd never need more than `amount` coins of value 1).
2. `dp[0] = 0` (zero coins needed to make amount 0).
3. For each amount `a` from `1` to `target`, for each coin: if `coin <= a`, then `dp[a] = min(dp[a], dp[a - coin] + 1)`.
4. Return `dp[amount]` if it's not still the sentinel value, else `-1`.

### Python code
```python
def coinChange(coins, amount):
    dp = [amount + 1] * (amount + 1)
    dp[0] = 0

    for a in range(1, amount + 1):
        for coin in coins:
            if coin <= a:
                dp[a] = min(dp[a], dp[a - coin] + 1)

    return dp[amount] if dp[amount] != amount + 1 else -1
```

### Line-by-line explanation
- `dp = [amount + 1] * (amount + 1)` - initialize every amount's answer to `amount + 1`, a sentinel value guaranteed to be larger than any real valid answer (since the worst-case real answer, using only 1-value coins, is `amount` coins) - this makes it easy to detect "still unreachable" without a separate infinity/None check.
- `dp[0] = 0` - base case: zero coins needed to make zero amount.
- `for a in range(1, amount + 1):` - build up every amount's answer in increasing order, since `dp[a]` depends only on smaller amounts (`a - coin`, always < a since coins are positive).
- `for coin in coins: if coin <= a:` - try using this coin as the "last" coin added; only valid if the coin's value doesn't exceed the amount we're building toward.
- `dp[a] = min(dp[a], dp[a - coin] + 1)` - if using this coin, the total is "however many coins were needed for the remaining amount after using it" plus 1 (for this coin itself); keep the best (minimum) over all coin choices.
- `return dp[amount] if dp[amount] != amount + 1 else -1` - if `dp[amount]` never got updated below the sentinel, that amount is genuinely unreachable with the given coins.

### Dry run
`coins = [1,2,5]`, `amount = 11`

Building up `dp[0..11]` (showing key values): `dp[0]=0`. `dp[1]=min(dp[0]+1)=1` (using coin 1). `dp[2]=min(dp[1]+1, dp[0]+1)=min(2,1)=1` (using coin 2 directly). `dp[3]=min(dp[2]+1,dp[1]+1)=min(2,2)=2`. `dp[4]=min(dp[3]+1,dp[2]+1)=min(3,2)=2`. `dp[5]=min(dp[4]+1,dp[3]+1,dp[0]+1)=min(3,3,1)=1` (using coin 5 directly). `dp[6]=min(dp[5]+1,dp[4]+1,dp[1]+1)=min(2,3,2)=2`. Continuing this pattern up through `dp[11]`: `dp[11] = min(dp[10]+1, dp[9]+1, dp[6]+1)`. Following the chain, `dp[10]=2` (5+5), so `dp[11] = dp[10]+1 = 3` (5+5+1) - matching the expected output.

Final: `dp[11] = 3` ✅

### Time & space complexity
- **Time: O(amount · len(coins))** - same as memoization, but without recursion overhead.
- **Space: O(amount)** for the `dp` array.

---

## Common mistakes & misconceptions

1. **Confusing this with Combination Sum (#39, backtracking) and trying to enumerate every combination of coins.** This problem only asks for the *minimum count*, not the combinations themselves - collecting every possible combination and taking the shortest is enormously wasteful compared to the DP formulation, which never needs to construct an actual combination at all.
2. **Using `float("inf")` correctly in the recursive approaches but forgetting to convert it back to `-1` at the end.** Returning `float("inf")` directly instead of `-1` when no valid combination exists is a common oversight - always match the function's actual required return contract.
3. **Assuming a greedy "always use the largest coin that fits" strategy works.** It doesn't, in general - e.g. `coins = [1, 3, 4]`, `amount = 6`: greedy picks `4 + 1 + 1` (3 coins), but the optimal is `3 + 3` (2 coins). This is exactly why DP (checking all coin choices) is required, not just a greedy shortcut - a classic and important misconception for this specific problem.
4. **Confusing this problem's DP shape with Combination Sum IV's.** Both iterate "amount as outer loop, coins as inner loop," but Coin Change takes a **min** over choices (fewest coins) while Combination Sum IV takes a **sum** over choices (count of ordered ways) - superficially similar code, genuinely different recurrence semantics.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute force recursion | O(coins^amount) | O(amount) | Exponential, recomputes shared sub-problems repeatedly. |
| Top-down memoization | O(amount · coins) | O(amount) | Same shape, cached. |
| Bottom-up tabulation | O(amount · coins) | O(amount) | The standard, most commonly expected solution - no recursion. |

**Key takeaway:** "fewest number of X to reach a target amount" problems have the classic 1-D DP shape where `dp[amount]` is built from `dp[smaller amount]` by trying every available choice (here, every coin) and taking the best result - the sentinel-value trick (`amount + 1` representing "impossible so far") is a clean, reusable way to handle "no valid way found" without needing separate infinity or None-checking logic scattered through the code.
