# Topic 13: 1-D Dynamic Programming

## Core concepts / data structures

### Dynamic Programming (DP)
**What it is:** a technique for solving problems by breaking them into smaller **overlapping** sub-problems, solving each distinct sub-problem **only once**, and reusing (rather than recomputing) that answer every time it's needed again. "1-D" DP specifically means the sub-problems can be indexed by a **single** number (like "the answer for the first `i` elements" or "the answer for amount `i`"), as opposed to 2-D DP's grid of sub-problems.

**Simple explanation:** imagine calculating the 40th Fibonacci number using plain recursion (`fib(n) = fib(n-1) + fib(n-2)`) - without any memory, this recomputes `fib(20)` many, many times over, buried inside different branches of the recursion tree, because both `fib(39)` and `fib(38)` eventually need `fib(20)` again, and neither remembers the other already solved it. DP is simply: **write down the answer to each sub-problem the first time you solve it, and look it up instead of recomputing it every subsequent time it's needed.**

**The key signal that DP applies:** a problem has **overlapping sub-problems** (the same smaller question gets asked multiple times during a brute-force/recursive solution) **and** has **optimal substructure** (the best answer to the whole problem can be built directly from the best answers to its sub-problems - not from some fundamentally different, unrelated computation).

### Top-down (memoization) vs. bottom-up (tabulation)

**Top-down (memoization):** write the natural recursive solution first, then add a cache (a dict or array) that stores each sub-problem's answer the first time it's computed, and checks the cache before recomputing.
```python
def fib(n, memo={}):
    if n <= 1:
        return n
    if n in memo:
        return memo[n]
    memo[n] = fib(n - 1, memo) + fib(n - 2, memo)
    return memo[n]
```

**Bottom-up (tabulation):** instead of recursing from the top (n) down to the base cases, build a table starting from the base cases and work **up** to n, iteratively - no recursion, no call stack risk.
```python
def fib(n):
    if n <= 1:
        return n
    dp = [0] * (n + 1)
    dp[1] = 1
    for i in range(2, n + 1):
        dp[i] = dp[i - 1] + dp[i - 2]
    return dp[n]
```

**Space-optimized bottom-up:** many 1-D DP problems only ever need the **last one or two** previous values, not the entire table - in that case, you can replace the whole array with just a couple of variables, dropping O(n) space down to O(1).
```python
def fib(n):
    if n <= 1:
        return n
    prev2, prev1 = 0, 1
    for _ in range(2, n + 1):
        prev2, prev1 = prev1, prev2 + prev1
    return prev1
```

## Common patterns / techniques in this topic

| Pattern | When it applies |
|---|---|
| **"Ways to reach n" (Fibonacci-shaped)** | Climbing Stairs, Coin Change-style counting problems - `dp[i]` depends on a small, fixed set of earlier values (`dp[i-1]`, `dp[i-2]`, etc.). |
| **"Best answer using or not using element i"** | House Robber-style problems - at each step, decide whether including the current element helps or hurts, comparing against the best answer that excludes it. |
| **"Longest/best sequence ending at i"** | Longest Increasing Subsequence-style problems - `dp[i]` represents the best answer for a sequence that specifically ends at index i, and the final answer is the max over all `dp[i]`. |
| **String-building DP** | Word Break, Decode Ways - `dp[i]` represents "can/how many ways can the first i characters be validly formed," built from checking substrings ending at position i. |
| **Prefix/suffix running values** | Maximum Product Subarray - tracking a running max *and* min simultaneously (since multiplying by a negative number can flip which one becomes the new extreme). |

## Key terminology

- **Sub-problem** - a smaller instance of the same problem (e.g. "the answer for the first i items" is a sub-problem of "the answer for all n items").
- **Overlapping sub-problems** - the same sub-problem gets solved multiple times in a naive recursive approach - the core justification for using DP at all.
- **Optimal substructure** - the optimal solution to a problem can be constructed from optimal solutions to its sub-problems.
- **Memoization** - caching results of a top-down (recursive) approach.
- **Tabulation** - building a table bottom-up (iteratively), without recursion.
- **State** - what a sub-problem is indexed by (in 1-D DP, typically a single index or amount); **transition** - the formula/rule for computing `dp[i]` from earlier states.
- **Base case(s)** - the smallest sub-problem(s), answered directly without needing the recurrence (e.g. `dp[0]` or `dp[1]`).

## Common beginner mistakes

1. **Writing the recursive solution but forgetting to memoize** - this alone is often the difference between an exponential-time and a linear/polynomial-time solution; always ask "am I solving the same sub-problem more than once?"
2. **Off-by-one errors in the DP array's indexing/size**, especially around whether `dp[i]` means "using the first i elements" or "using element i itself," and whether the array needs size `n` or `n+1` to comfortably hold a `dp[0]` base case.
3. **Getting the base case(s) wrong** - a single wrong base case can silently corrupt every value built on top of it.
4. **Trying to space-optimize (reduce to O(1) variables) before getting the O(n)-space tabulated version correct first** - it's much easier to verify correctness with the full table, then optimize space afterward, than to try to do both at once.
5. **Not recognizing that a "brute force recursion" solution and a "DP" solution are the SAME solution, just with or without caching** - beginners sometimes treat these as unrelated techniques to memorize separately, when DP is really just "recursion, but smart about not repeating work."
6. **Applying a 1-D DP shape to a problem that actually needs 2 dimensions of state** (or vice versa) - carefully identify exactly what varies between sub-problems; if the answer genuinely depends on two independent changing quantities, you likely need 2-D DP (the next topic) instead.

## How this compares to Backtracking

Backtracking explores *every* possibility, undoing choices as it goes, without remembering answers to repeated sub-problems - this can be exponentially slow when sub-problems overlap. DP is best understood as **"backtracking's cousin, but with a memory"**: many DP problems can be derived by first writing the brute-force recursive/backtracking solution, noticing which sub-problems repeat, and adding a cache - exactly the relationship called out at the end of the Backtracking topic.

## Starter problems (easy, to warm up)

1. **Climbing Stairs** (LeetCode #70) - in your Blind 75 list; the cleanest possible introduction to the "Fibonacci-shaped" DP pattern.
2. **House Robber** (LeetCode #198) - also in your Blind 75 list; the classic "include or exclude the current element" pattern.
3. **Fibonacci Number** (LeetCode #509) - not in Blind 75, but if the recursion-to-memoization-to-tabulation progression above still feels unfamiliar, implementing all three versions of Fibonacci yourself is the single best warm-up exercise for this entire topic.

## What carries over from here

Every core idea here (memoization vs. tabulation, identifying overlapping sub-problems, base cases, space optimization) applies directly and immediately to **2-D Dynamic Programming**, the next topic - the only real difference is that a sub-problem's "state" is described by **two** changing quantities (like two string indices, or a row and column) instead of one, so the DP table becomes a grid instead of a line.
