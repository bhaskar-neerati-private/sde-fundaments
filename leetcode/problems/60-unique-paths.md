# 60. Unique Paths

**LeetCode:** [#62 - Unique Paths](https://leetcode.com/problems/unique-paths/) · **Topic:** [2-D Dynamic Programming](../topics/14-2d-dynamic-programming.md) · **Difficulty:** Medium

## Problem statement

A robot starts at the top-left corner of an `m x n` grid and wants to reach the bottom-right corner. It can only move **right** or **down** at each step. Return the number of unique possible paths.

**Example:**
```
Input: m = 3, n = 7
Output: 28
```

## Applicable approaches

- **Brute Force Recursion.**
- **Top-down Memoization.**
- **Bottom-up 2-D Tabulation.**
- **Space-optimized (1-D rolling row).**
- **Pure Math - Combinatorics.** A clever closed-form alternative worth knowing exists.

## Approach 1: Brute Force Recursion

### Intuition
The number of ways to reach cell `(i, j)` is the number of ways to reach the cell **above** it (`(i-1, j)`, then move down) plus the number of ways to reach the cell **to its left** (`(i, j-1)`, then move right) - since those are the only two ways to arrive at `(i, j)`.

### Python code
```python
def uniquePaths(m, n):
    def paths(i, j):
        if i == 0 or j == 0:
            return 1  # only one way to reach any cell in the first row/column: go straight
        return paths(i - 1, j) + paths(i, j - 1)

    return paths(m - 1, n - 1)
```

### Time & space complexity
- **Time: O(2^(m+n))** - massive overlapping recomputation.
- **Space: O(m + n)** recursion depth.

---

## Approach 2: Top-down Memoization

### Python code
```python
def uniquePaths(m, n):
    memo = {}

    def paths(i, j):
        if i == 0 or j == 0:
            return 1
        if (i, j) in memo:
            return memo[(i, j)]

        memo[(i, j)] = paths(i - 1, j) + paths(i, j - 1)
        return memo[(i, j)]

    return paths(m - 1, n - 1)
```

### Time & space complexity
- **Time: O(m · n)**, **Space: O(m · n)**.

---

## Approach 3: Bottom-up 2-D Tabulation

### Intuition
Build the grid of path counts directly, row by row, using the same "from above plus from the left" recurrence.

### Python code
```python
def uniquePaths(m, n):
    dp = [[1] * n for _ in range(m)]

    for i in range(1, m):
        for j in range(1, n):
            dp[i][j] = dp[i - 1][j] + dp[i][j - 1]

    return dp[m - 1][n - 1]
```

### Line-by-line explanation
- `dp = [[1] * n for _ in range(m)]` - initialize the **entire** grid to 1s. This cleverly handles the base cases for free: the entire first row and entire first column correctly stay at 1 (there's only one way to reach any cell along the top edge or left edge - keep going straight in one direction), since the main loop below only overwrites cells starting from `i=1, j=1` onward.
- `dp[i][j] = dp[i - 1][j] + dp[i][j - 1]` - the number of ways to reach `(i,j)` is the sum of the ways to reach the cell above it and the cell to its left (the only two possible "last moves").
- `return dp[m - 1][n - 1]` - the bottom-right corner.

### Dry run
`m=3, n=3` (small example, expected answer 6)

Initial: all 1s.
```
1 1 1
1 1 1
1 1 1
```
- `dp[1][1] = dp[0][1] + dp[1][0] = 1+1=2`.
- `dp[1][2] = dp[0][2] + dp[1][1] = 1+2=3`.
- `dp[2][1] = dp[1][1] + dp[2][0] = 2+1=3`.
- `dp[2][2] = dp[1][2] + dp[2][1] = 3+3=6`.

Final grid:
```
1 1 1
1 2 3
1 3 6
```
`dp[2][2] = 6` ✅ (matches the known correct answer for a 3x3 grid).

### Time & space complexity
- **Time: O(m · n)**, **Space: O(m · n)**.

---

## Approach 4: Space-Optimized (1-D Rolling Row)

### Intuition
`dp[i][j]` only depends on the row directly above and values already computed in the current row - collapse to a single 1-D row, updated in place as we move left to right, reusing the "old" (above-row) value at `row[j]` before it gets overwritten with the "new" (current-row) value.

### Python code
```python
def uniquePaths(m, n):
    row = [1] * n

    for i in range(1, m):
        for j in range(1, n):
            row[j] += row[j - 1]

    return row[n - 1]
```

### Line-by-line explanation
- `row = [1] * n` - represents the first row initially (all 1s, matching the base case).
- `for i in range(1, m):` - process each subsequent row.
- `row[j] += row[j - 1]` - **before this line runs, `row[j]` still holds the value from the *previous* row** (it hasn't been updated yet in this pass) - so this correctly computes "value from above" (`row[j]`, not yet overwritten) plus "value from the left" (`row[j-1]`, *already* updated earlier in this same row's left-to-right pass) - collapsing the 2-D update into a single in-place 1-D array.
- `return row[n - 1]` - after processing all `m` rows, `row` holds the final row's values.

### Time & space complexity
- **Time: O(m · n)**, **Space: O(n)** - a significant improvement over the full 2-D table.

---

## Approach 5 (Bonus): Pure Math - Combinatorics

### Intuition
Every path from top-left to bottom-right consists of exactly `(m-1)` "down" moves and `(n-1)` "right" moves, in **some order** - a path is fully described by *which* of the total `(m-1)+(n-1)` moves are "down" moves (the rest are automatically "right" moves). The number of ways to choose which positions are "down" moves, out of the total moves, is a direct combinatorics formula: `C(m+n-2, m-1)` (choose `m-1` positions to be "down" out of `m+n-2` total moves).

### Python code
```python
from math import comb

def uniquePaths(m, n):
    return comb(m + n - 2, m - 1)
```

### Time & space complexity
- **Time: O(min(m, n))** (Python's `math.comb` computes this efficiently) - or O(1) if you consider the computation itself a "primitive" operation, though the underlying arithmetic on large numbers technically isn't strictly constant time. Either way, dramatically faster than any DP-based approach for very large grids.
- **Space: O(1)**.

---

## Common mistakes & misconceptions

1. **Using `[[1] * n] * m` instead of `[[1] * n for _ in range(m)]`.** This is exactly the shared-reference bug flagged in the topic overview - `[[1]*n]*m` creates `m` references to the *same* inner list, so updating `dp[1][1]` would incorrectly also change `dp[0][1]`, `dp[2][1]`, etc. This bug is especially sneaky here because the initial "all 1s" state looks correct at a glance; the corruption only becomes visible once the main loop starts writing.
2. **Forgetting there are no obstacles in this base version and mentally over-complicating the recurrence.** (A very common LeetCode follow-up, "Unique Paths II," adds obstacles - it's worth being aware that's a distinct, harder variant with an extra check, not this problem.)
3. **In the space-optimized version, updating `row[j]` before reading the "from above" value.** The one-liner `row[j] += row[j-1]` relies on `row[j]` still holding last row's value at the moment it's read - if you accidentally computed `row[j-1] + row[j]` using an already-updated `row[j]` from earlier logic, you'd corrupt the "from above" contribution; the specific line as written is correct precisely because `row[j]`'s old value is read *before* being reassigned in that same statement.
4. **Assuming the combinatorics formula is "the correct" solution and DP is just a slower workaround.** In an interview setting, the DP approach is usually what's actually being evaluated (understanding of the pattern, base cases, transitions) - the combinatorics bonus is worth mentioning as a demonstration of deeper insight, but jumping straight to it without first showing the DP reasoning can come across as skipping the actual assessment.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute force recursion | O(2^(m+n)) | O(m+n) | Exponential. |
| Top-down memoization | O(m·n) | O(m·n) | Cached. |
| Bottom-up 2-D tabulation | O(m·n) | O(m·n) | The standard, most commonly taught solution. |
| Space-optimized (1-D row) | O(m·n) | O(n) | Same time, less memory. |
| Combinatorics formula | ~O(min(m,n)) | O(1) | Elegant bonus - worth knowing exists, though DP is the expected primary answer in most interview settings. |

**Key takeaway:** this is the archetypal "grid traversal" 2-D DP problem - initializing the entire grid to the base-case value (1s here) is a clean trick for handling first-row/first-column base cases without special-casing them separately in the main loop, and the combinatorics bonus solution is a good reminder that DP isn't always the mathematically optimal approach, even when it's the expected one to demonstrate in an interview.
