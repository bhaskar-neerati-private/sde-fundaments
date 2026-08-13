# Topic 14: 2-D Dynamic Programming

## Core concepts / data structures

### 2-D DP
**What it is:** the same core idea as 1-D DP - break a problem into smaller overlapping sub-problems, cache/build up answers to avoid recomputation - except now each sub-problem's "state" is described by **two** independently changing quantities, so the table of answers is a **grid** (`dp[i][j]`) instead of a line (`dp[i]`).

**When you need 2-D instead of 1-D:** whenever a problem's answer genuinely depends on **two separate positions/indices at once** - very commonly, this shows up when comparing **two different strings** (one index into each), or when moving through an actual **2-D grid** (row and column), or when a 1-D problem gains an extra dimension of constraint (like "the last k items" or "how much capacity is left," on top of "which item are we considering").

### Two common shapes

**1. Two strings compared** (`dp[i][j]` = "answer using the first i characters of string A and the first j characters of string B")
```python
dp = [[0] * (len(s2) + 1) for _ in range(len(s1) + 1)]
for i in range(1, len(s1) + 1):
    for j in range(1, len(s2) + 1):
        if s1[i-1] == s2[j-1]:
            dp[i][j] = dp[i-1][j-1] + 1   # characters match, extend a shared result
        else:
            dp[i][j] = max(dp[i-1][j], dp[i][j-1])  # they don't match, take the better sub-result
```
Used for: Longest Common Subsequence, Edit Distance, and other two-string comparison problems.

**2. Grid traversal** (`dp[i][j]` = "answer for reaching/using cell (i, j) of an actual grid")
```python
dp = [[0] * cols for _ in range(rows)]
dp[0][0] = 1  # base case
for i in range(rows):
    for j in range(cols):
        if i > 0:
            dp[i][j] += dp[i-1][j]
        if j > 0:
            dp[i][j] += dp[i][j-1]
```
Used for: Unique Paths, minimum path sum, and other problems where movement happens on an actual 2-D board.

**Creating a 2-D array in Python - a critical gotcha:** `[[0] * cols] * rows` looks reasonable but is a **serious, classic bug** - it creates `rows` references to the **same** inner list, so modifying one row accidentally modifies every row. Always use a list comprehension instead: `[[0] * cols for _ in range(rows)]`, which creates `rows` genuinely independent lists.

## Common patterns / techniques in this topic

| Pattern | When it applies |
|---|---|
| **Two-string comparison grid** | Longest Common Subsequence, Edit Distance - `dp[i][j]` compares prefixes of two strings. |
| **Grid path counting/optimization** | Unique Paths, minimum/maximum path sum - `dp[i][j]` represents an answer for reaching cell (i,j), built from the cell(s) that could lead into it (typically the cell above and the cell to the left). |
| **Base case along the first row/column** | Grid DP problems almost always need special handling for row 0 and column 0, since cells there are missing one of their usual two "predecessor" directions. |
| **Space optimization to 1-D** | Since `dp[i][j]` in many of these problems only depends on the **previous row** (`dp[i-1][...]`) and the **current row so far** (`dp[i][...]`), the full 2-D table can often be collapsed to just one or two 1-D rolling rows - the 2-D analog of the space optimizations seen throughout the 1-D DP topic. |

## Key terminology

- **`dp[i][j]`** - the answer to the sub-problem defined by "the first i elements of one dimension, and the first j elements of the other."
- **Base row / base column** - the `i=0` row and `j=0` column of the table, which usually need to be filled in with a separate, simpler rule before the main double-loop can safely reference them.
- **Transition** - the formula for computing `dp[i][j]` from previously-computed values (usually `dp[i-1][j]`, `dp[i][j-1]`, and/or `dp[i-1][j-1]`).

## Common beginner mistakes

1. **The `[[0]*cols]*rows` shared-reference bug** described above - always use a list comprehension to build a genuinely independent 2-D array.
2. **Off-by-one confusion between "index into the string/grid" and "index into the DP table."** A very common convention is to make the DP table one size larger than the input in each dimension (e.g. `dp[i][j]` represents "using the first `i` characters," so `dp[0][...]` represents "using zero characters," and the *actual* character being considered at DP-index `i` is `s[i-1]`, not `s[i]`) - mixing this up causes subtle off-by-one bugs.
3. **Forgetting to properly initialize the base row/column** for grid problems, leaving cells that should represent "only one path exists" (e.g. always coming from the left along row 0) as their default zero value instead.
4. **Not recognizing when a problem's 2-D DP can be space-optimized to 1-D** - not incorrect to skip this, but worth knowing it's usually possible when `dp[i][j]` only depends on the row directly above and the current row so far.
5. **Choosing the wrong recurrence when characters match vs. don't match** in two-string comparison problems (e.g. LCS's "extend by 1, from the diagonal" when characters match, vs. "take the best of skipping a character from either string" when they don't) - getting this backward or conflating the two cases is a common source of wrong answers.

## How this compares to 1-D Dynamic Programming

Every core idea (memoization vs. tabulation, base cases, avoiding overlapping recomputation, even space optimization) carries over completely unchanged from 1-D DP - the only genuinely new skill is correctly identifying **what the two independent dimensions of state actually are** for a given problem, and correctly working out the transition formula relating `dp[i][j]` to its neighboring smaller sub-problems. If 1-D DP felt comfortable, 2-D DP is largely "the same discipline, applied twice."

## Starter problems

Given this topic (in your Blind 75 list) only contains two problems, the best "warm-up" is making sure the two common shapes above (two-string comparison, and grid traversal) both feel completely natural on paper before diving in - Longest Common Subsequence is the purer "two strings" example, and Unique Paths is the purer "grid traversal" example, so they naturally warm each other up.

## What carries over from here

2-D DP is as far as this specific curriculum goes structurally, but the same "identify the independent dimensions of state" skill generalizes to 3-D (and higher) DP problems you may encounter later (e.g. problems with two string indices *plus* a remaining-budget dimension) - the mechanics don't fundamentally change, just the number of dimensions the table needs.
