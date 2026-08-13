# 62. Jump Game

**LeetCode:** [#55 - Jump Game](https://leetcode.com/problems/jump-game/) · **Topic:** [Greedy](../topics/15-greedy.md) · **Difficulty:** Medium

## Problem statement

Given an array `nums` where `nums[i]` is the maximum jump length from position `i`, starting at index 0, return `true` if you can reach the **last index**.

**Example:**
```
Input: nums = [2,3,1,1,4]
Output: true  (0 -> 1 -> 4, or 0 -> 2 -> 3 -> 4)

Input: nums = [3,2,1,0,4]
Output: false  (index 3 is a "dead end" with value 0, and it's the only way past it)
```

## Applicable approaches

- **Brute Force Recursion** - try every possible jump from every position.
- **Top-down DP (Memoization)**.
- **Bottom-up DP** - O(n²) in the worst case.
- **Optimal - Greedy (Track Farthest Reachable Position).** The standard, expected O(n) solution.

## Approach 1: Brute Force Recursion

### Intuition
From the current position, try every possible jump length (1 up to `nums[i]`), and recursively check if reaching *any* of those resulting positions can eventually reach the end.

### Python code
```python
def canJump(nums):
    n = len(nums)

    def canReachEnd(i):
        if i >= n - 1:
            return True
        for jump in range(1, nums[i] + 1):
            if canReachEnd(i + jump):
                return True
        return False

    return canReachEnd(0)
```

### Time & space complexity
- **Time: O(2^n)** worst case - branches heavily at every position.
- **Space: O(n)** recursion depth.

---

## Approach 2: Top-down Memoization / Bottom-up DP

### Intuition
Define `dp[i]` = "can we reach the end starting from position `i`?" Cache each position's answer to avoid recomputing it.

### Python code (bottom-up)
```python
def canJump(nums):
    n = len(nums)
    dp = [False] * n
    dp[n - 1] = True

    for i in range(n - 2, -1, -1):
        farthest = min(i + nums[i], n - 1)
        for j in range(i + 1, farthest + 1):
            if dp[j]:
                dp[i] = True
                break

    return dp[0]
```

### Time & space complexity
- **Time: O(n²)** worst case - for each position, checking up to `nums[i]` further positions.
- **Space: O(n)** for the `dp` array.

*(Correct, but the greedy approach below achieves the same result in O(n), by realizing we don't actually need to track reachability from every individual position separately.)*

---

## Approach 3: Optimal - Greedy (Track Farthest Reachable Position)

### Intuition
We don't actually need to know, for every single position, whether the end is reachable *from* it - we only need to track **one running number**: the farthest position reachable **at all**, considering everything seen so far. Scan left to right; at each position `i`, if `i` is beyond the farthest reachable position so far, we're stuck (can never have gotten here) - fail immediately. Otherwise, update the farthest reachable position using `i + nums[i]`. If the farthest reachable position ever reaches or passes the last index, success.

### Algorithm
1. Track `farthest = 0` (the farthest index reachable so far).
2. For each index `i` from 0 to n-1:
   - If `i > farthest`, this position is unreachable - return `False` immediately.
   - Update `farthest = max(farthest, i + nums[i])`.
3. If the loop completes without ever returning `False`, return `True` (we always stayed reachable, all the way to the end).

### Python code
```python
def canJump(nums):
    farthest = 0

    for i in range(len(nums)):
        if i > farthest:
            return False
        farthest = max(farthest, i + nums[i])

    return True
```

### Line-by-line explanation
- `farthest = 0` - initially, only index 0 itself is "reachable" (the starting position).
- `for i in range(len(nums)):` - scan every position in order.
- `if i > farthest: return False` - **the key greedy check**: if we've reached a position in our scan that's beyond anything we could actually jump to, given everything we've seen so far, then this position (and therefore the end, if it's beyond here) is genuinely unreachable - stop immediately.
- `farthest = max(farthest, i + nums[i])` - update our running "how far can we possibly get" tracker, greedily always keeping the best (largest) reach discovered so far, and never needing to "undo" this - a larger reachable distance is never something we'd want to give back.
- `return True` - if we scan all the way through without ever finding an unreachable position, the last index (being ≤ `len(nums)-1`, which we'd have checked reachability for along the way) must have been reachable.

### Why this greedy approach is correct
The key insight is that we only ever care about the **single best (farthest) reach** achievable using everything up to position `i` - we don't need to remember *which specific position* achieves that farthest reach, because from a pure "can I get further" standpoint, only the maximum value matters; any position that jumps less far than the current best is strictly dominated by it and can be safely ignored, since it never provides more options than the current farthest reach already implies. This is exactly why a single running `farthest` variable, updated greedily and never revisited, is enough.

### Dry run
`nums = [2,3,1,1,4]`

`farthest = 0`.

| i | i > farthest? | farthest = max(farthest, i+nums[i]) |
|---|---|---|
| 0 | 0>0? No | max(0, 0+2)=2 |
| 1 | 1>2? No | max(2, 1+3)=4 |
| 2 | 2>4? No | max(4, 2+1)=4 |
| 3 | 3>4? No | max(4, 3+1)=4 |
| 4 | 4>4? No | max(4, 4+4)=8 |

Loop completes without ever returning `False` → return `True` ✅

**A failing dry run:** `nums = [3,2,1,0,4]`

`farthest=0`.
- i=0: `0>0`?No. `farthest=max(0,0+3)=3`.
- i=1: `1>3`?No. `farthest=max(3,1+2)=3`.
- i=2: `2>3`?No. `farthest=max(3,2+1)=3`.
- i=3: `3>3`?No. `farthest=max(3,3+0)=3` (unchanged, since `nums[3]=0`).
- i=4: `4>3`? **Yes** → return `False` ✅ (correctly detects that index 4 is unreachable - index 3's zero jump length created a dead end, and no earlier position could jump far enough to skip over it).

### Time & space complexity
- **Time: O(n)** - a single pass.
- **Space: O(1)** - one running variable. This is the optimal solution.

---

## Common mistakes & misconceptions

1. **Checking `i >= farthest` instead of `i > farthest`.** Landing exactly on the current farthest-reachable index is perfectly fine (it means we just barely reached it) - only going *beyond* it (`i > farthest`) is actually a failure; using `>=` would incorrectly reject valid paths that land exactly on the frontier.
2. **Believing you need to track the actual sequence of jumps to answer this problem.** The problem only asks whether the end is reachable at all, not what the path is - tracking a single "farthest reachable" number is sufficient; reconstructing an actual jump sequence would be a related but different (and unnecessary) task.
3. **Assuming a `nums[i] == 0` always means "stuck forever."** A zero only causes failure if it's impossible to jump *over* that position from some earlier point - as the second dry run shows, a zero can be safely "jumped over" if an earlier position's reach already extends past it; the greedy `farthest` tracker naturally accounts for this without needing a special case.
4. **Confusing this problem with Jump Game II** (the "minimum number of jumps" variant), which requires tracking jump *counts*, not just reachability - a superficially similar-looking greedy idea (tracking a "current end of reachable range" and a separate "farthest seen") is needed there, and it's a genuinely different (though related) problem from this yes/no reachability question.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute force recursion | O(2^n) | O(n) | Exponential. |
| Top-down/bottom-up DP | O(n²) | O(n) | Correct, but tracks more information than actually necessary. |
| Greedy (farthest reachable) | O(n) | O(1) | The standard, optimal solution. |

**Key takeaway:** many "can I reach X" reachability problems don't actually need full DP - if the only thing that matters is the **best** (farthest/largest/smallest) value achievable so far, and earlier, "worse" options never provide anything a later, "better" option doesn't already cover, a single running greedy variable is enough, collapsing what looks like an O(n²) DP problem down to O(n).
