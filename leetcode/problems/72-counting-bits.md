# 72. Counting Bits

**LeetCode:** [#338 - Counting Bits](https://leetcode.com/problems/counting-bits/) · **Topic:** [Bit Manipulation](../topics/18-bit-manipulation.md) · **Difficulty:** Easy

## Problem statement

Given an integer `n`, return an array `ans` of length `n+1` where `ans[i]` is the number of `1` bits in the binary representation of `i`, for every `i` from `0` to `n`.

**Example:**
```
Input: n = 5
Output: [0,1,1,2,1,2]
Explanation: 0=0b000(0), 1=0b001(1), 2=0b010(1), 3=0b011(2), 4=0b100(1), 5=0b101(2)
```

## Applicable approaches

- **Per-Number Bit Checking (using Number of 1 Bits' logic, repeated for every i).**
- **Optimal - DP, Building Each Answer from a Smaller Already-Computed One.** The standard, expected O(n) solution.

## Approach 1: Per-Number Bit Checking

### Intuition
Just apply the "count set bits" logic (either the 32-position check, or the `n & (n-1)` trick) independently to every number from 0 to n.

### Python code
```python
def countBits(n):
    def popcount(x):
        count = 0
        while x:
            x &= (x - 1)
            count += 1
        return count

    return [popcount(i) for i in range(n + 1)]
```

### Time & space complexity
- **Time: O(n · k)** where k = average number of set bits (up to ~32/2 in the worst case, or more precisely bounded by O(log i) bits for number `i`) - each of the n+1 numbers independently recomputes its bit count from scratch.
- **Space: O(n)** for the output array.

*(Correct, but doesn't reuse any work between numbers - the optimal DP approach below builds each answer directly from an already-computed smaller one, achieving true O(n) overall.)*

---

## Approach 2: Optimal - DP Using `i & (i-1)`

### Intuition
Recall: `i & (i-1)` clears the lowest set bit of `i`, producing a **smaller** number that's **already been computed** earlier in our scan (since we're building the answer array in increasing order). So: `ans[i] = ans[i & (i-1)] + 1` - "the bit count of `i`" equals "the bit count of `i` with its lowest set bit removed" (already known) "plus 1" (for that bit we just removed).

### Algorithm
1. Create `ans = [0] * (n + 1)` (base case: `ans[0] = 0`, zero has no set bits).
2. For each `i` from 1 to n: `ans[i] = ans[i & (i - 1)] + 1`.
3. Return `ans`.

### Python code
```python
def countBits(n):
    ans = [0] * (n + 1)

    for i in range(1, n + 1):
        ans[i] = ans[i & (i - 1)] + 1

    return ans
```

### Line-by-line explanation
- `ans[0] = 0` - base case, zero has zero set bits (already correctly initialized by `[0] * (n+1)`).
- `for i in range(1, n + 1):` - build up every subsequent answer using already-computed smaller values.
- `ans[i] = ans[i & (i - 1)] + 1` - `i & (i-1)` is always strictly smaller than `i` (it has one fewer set bit), so `ans[i & (i-1)]` is guaranteed to have already been computed earlier in this same loop; add 1 to account for the bit that was cleared to get there.

### Dry run
`n = 5`

`ans = [0,0,0,0,0,0]` initially.

- `i=1`: `1 & 0 = 0`. `ans[1] = ans[0] + 1 = 0+1 = 1`.
- `i=2`: `2 & 1 = 10 & 01 = 00 = 0`. `ans[2] = ans[0] + 1 = 1`.
- `i=3`: `3 & 2 = 11 & 10 = 10 = 2`. `ans[3] = ans[2] + 1 = 1+1 = 2`.
- `i=4`: `4 & 3 = 100 & 011 = 000 = 0`. `ans[4] = ans[0] + 1 = 1`.
- `i=5`: `5 & 4 = 101 & 100 = 100 = 4`. `ans[5] = ans[4] + 1 = 1+1 = 2`.

Final: `ans = [0,1,1,2,1,2]` ✅ matches expected output exactly.

### Time & space complexity
- **Time: O(n)** - each of the n+1 answers computed in O(1) using an already-known smaller value.
- **Space: O(n)** for the output array (unavoidable, since it's the required output), O(1) truly extra.

---

## Alternative DP Relation (Also Worth Knowing): Using `i >> 1`

### Intuition
Another valid recurrence: `i >> 1` (right-shifting `i` by 1) is equivalent to dropping the lowest bit entirely - `ans[i]` then equals `ans[i >> 1]` (the bit count of everything except the lowest bit) **plus** whether that dropped lowest bit was itself a 1 (`i & 1`).

### Python code
```python
def countBits(n):
    ans = [0] * (n + 1)

    for i in range(1, n + 1):
        ans[i] = ans[i >> 1] + (i & 1)

    return ans
```

### Why this also works
`i >> 1` is always `i // 2`, strictly smaller than `i` for `i > 0`, so `ans[i >> 1]` is always already computed. Whether we add 0 or 1 depends on whether `i`'s lowest bit (the one that got shifted away) was set - captured by `i & 1`.

### Time & space complexity
- **Time: O(n)**, **Space: O(n)** - identical complexity to the `i & (i-1)` version; this is simply an alternative, equally valid recurrence relation.

---

## Common mistakes & misconceptions

1. **Believing `ans[i & (i-1)]` might not be computed yet when processing `i`.** Since `i & (i-1)` always clears a bit from `i`, the result is always strictly smaller than `i`, and because the loop processes indices in increasing order, that smaller value's answer is guaranteed to already be filled in - this is precisely why the loop direction (ascending) is essential, not optional.
2. **Trying to use this same "build from a smaller related index" idea with a recurrence that *doesn't* actually guarantee a strictly smaller index.** Both valid recurrences shown here (`i & (i-1)` and `i >> 1`) specifically produce strictly smaller values for every `i > 0`; it's worth deliberately verifying this property before trusting a new recurrence idea in similar DP-on-a-range problems.
3. **Off-by-one on the output array size.** The problem asks for bit counts for every value from `0` to `n` *inclusive*, meaning the output array needs length `n + 1`, not `n` - easy to get wrong when translating "up to n" into an array size.
4. **Not recognizing this as fundamentally the same "build each answer from an already-solved smaller sub-problem" idea as the 1-D DP topic**, and instead treating it as an unrelated "bit manipulation-only" trick - the DP framing (recognizing overlapping structure across the range of inputs) is exactly what elevates this from O(n·k) to O(n).

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Per-number independent bit checking | O(n · k) | O(n) | Correct, but recomputes each answer from scratch. |
| DP using `i & (i-1)` | O(n) | O(n) | The standard, expected optimal solution. |
| DP using `i >> 1` | O(n) | O(n) | An equally valid alternative recurrence. |

**Key takeaway:** whenever a problem asks for the same kind of computation across a whole **range** of numbers (here: bit counts for every `i` from 0 to n), check whether each answer can be built from a smaller, already-computed answer using a cheap relationship between consecutive/related numbers - this is the same "avoid recomputation via DP" principle from the Dynamic Programming topics, applied here to bit-counting specifically.
