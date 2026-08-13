# 55. House Robber II

**LeetCode:** [#213 - House Robber II](https://leetcode.com/problems/house-robber-ii/) · **Topic:** [1-D Dynamic Programming](../topics/13-1d-dynamic-programming.md) · **Difficulty:** Medium

## Problem statement

Same as House Robber, but the houses are arranged in a **circle** - meaning the **first and last houses are also adjacent** to each other (robbing both triggers the alarm too).

**Example:**
```
Input: nums = [2,3,2]
Output: 3
Explanation: Can't rob houses 0 and 2 together (they're adjacent in the circle), so the best is just house 1 (value 3), which beats robbing only house 0 (value 2).
```

## Applicable approaches

- **Reduce to Two House Robber I Calls** - the standard, elegant, expected solution.

## Approach: Reduce to Two House Robber I Calls

### Intuition
The circular constraint means we can never rob **both** house 0 and house n-1 (the first and last) at the same time. But that's the *only* thing that's different from the regular House Robber problem. So: the answer must fall into one of two cases - either house 0 is **excluded** from consideration entirely (rob only among houses `1` to `n-1`, a plain non-circular line), or house `n-1` is **excluded** entirely (rob only among houses `0` to `n-2`, also a plain non-circular line). The true answer is the **better of these two cases** - and each case is exactly the regular (non-circular) House Robber problem, applied to a sub-array!

**Why this correctly covers every possibility:** any valid robbery plan either includes house 0 or it doesn't. If it doesn't include house 0, it's entirely captured by "rob among houses 1 to n-1." If it *does* include house 0, then it definitely does **not** include house n-1 (they're adjacent in the circle) - so that plan is entirely captured by "rob among houses 0 to n-2." Every valid plan falls into (at least) one of these two cases, so taking the best of both cases is guaranteed to find the true optimal answer.

### Algorithm
1. Handle the trivial edge case: if there's only 1 house, just return its value (no circular adjacency issue possible with a single house).
2. Compute the regular House Robber answer for the sub-array `nums[0 : n-1]` (excluding the last house).
3. Compute the regular House Robber answer for the sub-array `nums[1 : n]` (excluding the first house).
4. Return the maximum of the two.

### Python code
```python
def rob(nums):
    if len(nums) == 1:
        return nums[0]

    def robLine(houses):
        prev2, prev1 = 0, 0
        for num in houses:
            prev2, prev1 = prev1, max(prev1, num + prev2)
        return prev1

    return max(robLine(nums[:-1]), robLine(nums[1:]))
```

### Line-by-line explanation
- `if len(nums) == 1: return nums[0]` - special-cased separately: with only one house, there's no "first and last are adjacent" issue at all (a single house can't be adjacent to itself), so the general reduction below (which relies on excluding either the first or last of at least 2 houses) doesn't cleanly apply and would actually give an incorrect answer of 0 in this edge case if not handled separately (since `nums[:-1]` and `nums[1:]` would both become empty).
- `robLine(houses)` - this is exactly the space-optimized House Robber I solution from the previous problem, reused as a helper (unchanged logic - the "circular" complexity is entirely handled by *which sub-array* we call it on, not by changing the core algorithm itself).
- `nums[:-1]` - all houses except the last (Python slice notation: everything up to, but not including, the final element).
- `nums[1:]` - all houses except the first.
- `return max(robLine(nums[:-1]), robLine(nums[1:]))` - the better of the two cases, as justified in the intuition above.

### Dry run
`nums = [2,3,2]`

- `robLine(nums[:-1])` = `robLine([2,3])`: `prev2,prev1=0,0`. num=2: `(0, max(0,2+0)=2)` → `(0,2)`. num=3: `(2, max(2,3+0)=3)` → `(2,3)`. Return `3`.
- `robLine(nums[1:])` = `robLine([3,2])`: `prev2,prev1=0,0`. num=3: `(0,3)`. num=2: `(3, max(3,2+0)=3)` → `(3,3)`. Return `3`.

`max(3, 3) = 3` ✅ matches expected output.

**A second dry run to show the two cases genuinely differing:** `nums = [1,2,3,1]`

- `robLine([1,2,3])` (excluding last): best is `1+3=4` (rob houses at values 1 and 3, non-adjacent within this sub-array) → returns `4`.
- `robLine([2,3,1])` (excluding first): best is `max(2+1, 3) = max(3,3) = 3`... let's trace precisely: `prev2,prev1=0,0`. num=2:`(0,2)`. num=3:`(2,max(2,3+0)=3)`→`(2,3)`. num=1:`(3,max(3,1+2)=3)`→`(3,3)`. Returns `3`.
- `max(4, 3) = 4` ✅ (matches the known correct answer for this classic example: rob houses 0 and 2, values 1+3=4, valid since houses 0 and 2 aren't adjacent even in the circular sense - only 0-and-3 and consecutive pairs are restricted).

### Time & space complexity
- **Time: O(n)** - two linear passes (each `robLine` call is O(n)), so O(n) overall (constant factor of 2, still linear).
- **Space: O(n)** for the slice copies (`nums[:-1]`, `nums[1:]` each create new lists), or **O(1)** extra if implemented with index bounds instead of actual slicing (a minor further optimization).

---

## Common mistakes & misconceptions

1. **Forgetting the `n == 1` special case**, which would otherwise cause both `nums[:-1]` and `nums[1:]` to become empty lists, incorrectly returning `0` instead of the single house's actual value.
2. **Assuming you need a fundamentally different DP recurrence for the circular case.** You don't - the *only* new idea is the reduction (run the existing linear solution twice, on two overlapping sub-arrays); the core "rob or skip" recurrence inside `robLine` is completely unchanged from House Robber I.
3. **Incorrectly excluding both the first AND last house simultaneously in a single pass**, rather than considering the two cases *separately* and taking the best of each. Excluding both at once would be overly conservative and could miss the true optimal plan (e.g. a plan that robs house 0 and doesn't need house n-1 at all still deserves full consideration in the "exclude only the last house" case).
4. **Not noticing that the two sub-array cases can overlap in which houses they consider** (e.g. houses `1` through `n-2` appear in *both* `nums[:-1]` and `nums[1:]`) - this overlap is fine and expected; each case still independently enforces its own adjacency constraint correctly within its own sub-array.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Reduce to two House Robber I calls | O(n) | O(n) (or O(1) with index-based bounds instead of slicing) | The standard, elegant, expected solution. |

**Key takeaway:** when a new constraint (here: circular adjacency) only affects a small, specific part of a problem you already know how to solve, look for a way to **reduce** the new problem to one or more calls of the solution you already have, rather than redesigning the whole algorithm from scratch. Recognizing "this is really just two instances of a problem I already solved" is a high-value, broadly transferable problem-solving skill.
