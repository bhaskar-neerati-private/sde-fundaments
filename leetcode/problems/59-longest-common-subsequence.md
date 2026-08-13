# 59. Longest Common Subsequence

**LeetCode:** [#1143 - Longest Common Subsequence](https://leetcode.com/problems/longest-common-subsequence/) · **Topic:** [2-D Dynamic Programming](../topics/14-2d-dynamic-programming.md) · **Difficulty:** Medium

## Problem statement

Given two strings `text1` and `text2`, return the length of their **longest common subsequence** (characters appearing in the same relative order in both strings, but not necessarily contiguous) - or 0 if there's no common subsequence.

**Example:**
```
Input: text1 = "abcde", text2 = "ace"
Output: 3   ("ace" is a subsequence of both)
```

## Applicable approaches

- **Brute Force Recursion** - try all combinations of including/skipping characters from both strings.
- **Top-down Memoization.**
- **Bottom-up 2-D Tabulation** - the standard, expected optimal solution.
- **Space-optimized (1-D rolling rows).**

## Approach 1: Brute Force Recursion

### Intuition
Compare the strings from the end (or start) backward. At each pair of positions `(i, j)`: if the characters match, they must both be part of the LCS - move both pointers inward and add 1. If they don't match, the LCS can't use *both* of these characters together - try skipping one from `text1` OR skipping one from `text2`, and take whichever gives the longer result.

### Python code
```python
def longestCommonSubsequence(text1, text2):
    def lcs(i, j):
        if i == len(text1) or j == len(text2):
            return 0
        if text1[i] == text2[j]:
            return 1 + lcs(i + 1, j + 1)
        return max(lcs(i + 1, j), lcs(i, j + 1))

    return lcs(0, 0)
```

### Time & space complexity
- **Time: O(2^(n+m))** - branches into two recursive calls whenever characters don't match.
- **Space: O(n + m)** recursion depth.

---

## Approach 2: Top-down Memoization

### Python code
```python
def longestCommonSubsequence(text1, text2):
    memo = {}

    def lcs(i, j):
        if i == len(text1) or j == len(text2):
            return 0
        if (i, j) in memo:
            return memo[(i, j)]

        if text1[i] == text2[j]:
            result = 1 + lcs(i + 1, j + 1)
        else:
            result = max(lcs(i + 1, j), lcs(i, j + 1))

        memo[(i, j)] = result
        return result

    return lcs(0, 0)
```

### Time & space complexity
- **Time: O(n · m)** - each distinct `(i, j)` pair computed once.
- **Space: O(n · m)** for the memo, plus O(n+m) recursion depth.

---

## Approach 3: Optimal - Bottom-up 2-D Tabulation

### Intuition
Build a 2-D table `dp[i][j]` = "the LCS length using the first `i` characters of `text1` and the first `j` characters of `text2`." Fill it in order of increasing `i` and `j`, using the same match/no-match logic as the recursive version, but iteratively.

### Algorithm
1. Create `dp` of size `(n+1) x (m+1)`, all zeros (`dp[0][j]` and `dp[i][0]` are correctly 0 - an empty string has no common subsequence with anything).
2. For each `i` from 1 to n, each `j` from 1 to m:
   - If `text1[i-1] == text2[j-1]` (comparing the actual characters, offset by 1 from the DP indices): `dp[i][j] = dp[i-1][j-1] + 1`.
   - Else: `dp[i][j] = max(dp[i-1][j], dp[i][j-1])`.
3. Return `dp[n][m]`.

### Python code
```python
def longestCommonSubsequence(text1, text2):
    n, m = len(text1), len(text2)
    dp = [[0] * (m + 1) for _ in range(n + 1)]

    for i in range(1, n + 1):
        for j in range(1, m + 1):
            if text1[i - 1] == text2[j - 1]:
                dp[i][j] = dp[i - 1][j - 1] + 1
            else:
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])

    return dp[n][m]
```

### Line-by-line explanation
- `dp = [[0] * (m + 1) for _ in range(n + 1)]` - a genuinely independent 2-D array (using the list-comprehension form, avoiding the shared-reference bug described in the topic overview), sized one larger than each string to comfortably represent "using zero characters" at index 0.
- `if text1[i - 1] == text2[j - 1]:` - **note the `-1` offset**: `dp[i][j]` represents using the first `i` characters, so the *i-th* character (1-indexed) is `text1[i-1]` (0-indexed) - this offset is the single most common source of bugs in this kind of DP.
- `dp[i][j] = dp[i - 1][j - 1] + 1` - if the current characters match, extend the LCS found using *both* strings' characters up to (but not including) this position - i.e. build on the diagonal predecessor.
- `dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])` - if they don't match, the best LCS here is whichever is better: ignoring the current character of `text1` (`dp[i-1][j]`), or ignoring the current character of `text2` (`dp[i][j-1]`).
- `return dp[n][m]` - the answer using the *entirety* of both strings.

### Dry run
`text1 = "abcde"`, `text2 = "ace"`

Building the table (`dp[i][j]`, rows = text1 prefix length 0-5, cols = text2 prefix length 0-3):

|      | "" | a | c | e |
|---|---|---|---|---|
| **""** | 0 | 0 | 0 | 0 |
| **a** | 0 | 1 | 1 | 1 |
| **ab** | 0 | 1 | 1 | 1 |
| **abc** | 0 | 1 | 2 | 2 |
| **abcd** | 0 | 1 | 2 | 2 |
| **abcde** | 0 | 1 | 2 | 3 |

Key steps: `dp[1][1]` ('a' vs 'a', match) = `dp[0][0]+1=1`. `dp[3][2]` ('c' vs 'c', match) = `dp[2][1]+1 = 1+1=2`. `dp[5][3]` ('e' vs 'e', match) = `dp[4][2]+1 = 2+1=3`.

Final: `dp[5][3] = 3` ✅ matches expected output ("ace").

### Time & space complexity
- **Time: O(n · m)**, **Space: O(n · m)** for the full 2-D table.

---

## Approach 4: Space-Optimized (1-D Rolling Rows)

### Intuition
`dp[i][j]` only ever depends on the **previous row** (`dp[i-1][...]`) and values already computed in the **current row** (`dp[i][...]`) - never any row further back. So we can collapse the full 2-D table down to just **two** 1-D rows (previous and current), swapping which is which after each row is completed.

### Python code
```python
def longestCommonSubsequence(text1, text2):
    n, m = len(text1), len(text2)
    prev_row = [0] * (m + 1)

    for i in range(1, n + 1):
        curr_row = [0] * (m + 1)
        for j in range(1, m + 1):
            if text1[i - 1] == text2[j - 1]:
                curr_row[j] = prev_row[j - 1] + 1
            else:
                curr_row[j] = max(prev_row[j], curr_row[j - 1])
        prev_row = curr_row

    return prev_row[m]
```

### Time & space complexity
- **Time: O(n · m)** - unchanged.
- **Space: O(m)** - only two rows of length `m+1` at any time, a significant improvement over the full O(n·m) table.

---

## Common mistakes & misconceptions

1. **Confusing "subsequence" with "substring."** A subsequence doesn't need to be contiguous - `"ace"` is a valid subsequence of `"abcde"` even though the characters aren't adjacent in the original string. Solving this as if it required contiguous matching (a completely different, substring-based problem) gives wrong answers.
2. **Using `dp[i][j-1]` or `dp[i-1][j]` (single-string skip) when characters actually match**, instead of the diagonal `dp[i-1][j-1] + 1`. When characters match, both must be "consumed together" to get credit for extending the LCS by exactly one - reaching for the general no-match formula even when a match was found under-counts the true LCS length.
3. **Off-by-one indexing between the DP table and the actual strings** (the `-1` offset flagged above) - one of the most common bugs in any 2-D string-comparison DP; always be deliberate about whether an index refers to "characters considered so far" (DP table index) or "the actual character at this position" (string index, offset by one).
4. **Assuming the "which characters were actually chosen" can be trivially read off `dp[n][m]` alone.** The table only stores the *length*; reconstructing the actual subsequence requires walking back through the table from `dp[n][m]`, following which transition (diagonal vs. up vs. left) produced each cell's value - a related but genuinely separate task from what's shown here.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute force recursion | O(2^(n+m)) | O(n+m) | Exponential. |
| Top-down memoization | O(n·m) | O(n·m) | Cached recursion. |
| Bottom-up 2-D tabulation | O(n·m) | O(n·m) | The standard, most commonly taught solution. |
| Space-optimized (rolling rows) | O(n·m) | O(m) | Further optimization once the 2-D version is understood. |

**Key takeaway:** this is the archetypal "compare two strings" 2-D DP problem - the match/no-match transition (`dp[i-1][j-1]+1` vs. `max(dp[i-1][j], dp[i][j-1])`) is worth memorizing thoroughly, since it's the direct foundation for other two-string DP problems (like Edit Distance) beyond this specific list.
