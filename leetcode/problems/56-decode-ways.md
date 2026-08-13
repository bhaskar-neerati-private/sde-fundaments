# 56. Decode Ways

**LeetCode:** [#91 - Decode Ways](https://leetcode.com/problems/decode-ways/) · **Topic:** [1-D Dynamic Programming](../topics/13-1d-dynamic-programming.md) · **Difficulty:** Medium

## Problem statement

A message of digits can be decoded using the mapping `'A'->1, 'B'->2, ..., 'Z'->26`. Given a string `s` of digits, return the number of ways it can be decoded.

**Example:**
```
Input: s = "226"
Output: 3
Explanation: "BZ" (2,26), "VF" (22,6), "BBF" (2,2,6)
```

## Applicable approaches

- **Brute Force Recursion** - try decoding 1 or 2 digits at each position.
- **Top-down Memoization.**
- **Bottom-up Tabulation** - the standard, expected optimal solution.

## Approach 1: Brute Force Recursion

### Intuition
At each position, we can either decode **one digit** (as long as it's not `'0'`, since there's no letter for 0) or **two digits together** (as long as that two-digit number is between 10 and 26). Try both options (where valid) and sum the number of ways each leads to.

### Python code
```python
def numDecodings(s):
    def count(i):
        if i == len(s):
            return 1
        if s[i] == '0':
            return 0  # a leading zero can never start a valid decode

        ways = count(i + 1)  # decode s[i] alone
        if i + 1 < len(s) and 10 <= int(s[i:i+2]) <= 26:
            ways += count(i + 2)  # decode s[i:i+2] together

        return ways

    return count(0)
```

### Time & space complexity
- **Time: O(2^n)** - branches into up to 2 choices at every position.
- **Space: O(n)** recursion depth.

---

## Approach 2: Top-down Memoization

### Python code
```python
def numDecodings(s):
    memo = {}

    def count(i):
        if i == len(s):
            return 1
        if s[i] == '0':
            return 0
        if i in memo:
            return memo[i]

        ways = count(i + 1)
        if i + 1 < len(s) and 10 <= int(s[i:i+2]) <= 26:
            ways += count(i + 2)

        memo[i] = ways
        return ways

    return count(0)
```

### Time & space complexity
- **Time: O(n)**, **Space: O(n)**.

---

## Approach 3: Optimal - Bottom-up Tabulation

### Intuition
Define `dp[i]` = the number of ways to decode the prefix `s[0:i]`. Build up from `dp[0] = 1` (empty prefix, one trivial way - decode nothing). For each position `i`, add in the ways from decoding the single digit `s[i-1]` alone (if it's not `'0'`) and the ways from decoding the two digits `s[i-2:i]` together (if that two-digit number is between 10 and 26).

### Python code
```python
def numDecodings(s):
    n = len(s)
    dp = [0] * (n + 1)
    dp[0] = 1
    dp[1] = 1 if s[0] != '0' else 0

    for i in range(2, n + 1):
        one_digit = s[i - 1]
        two_digit = s[i - 2:i]

        if one_digit != '0':
            dp[i] += dp[i - 1]
        if 10 <= int(two_digit) <= 26:
            dp[i] += dp[i - 2]

    return dp[n]
```

### Line-by-line explanation
- `dp[0] = 1` - base case: the empty prefix has exactly one (trivial) way to be decoded.
- `dp[1] = 1 if s[0] != '0' else 0` - the first single character alone: valid (1 way) unless it's `'0'` (invalid, 0 ways) - a string can never start with `'0'`.
- `for i in range(2, n + 1):` - build up the count for each prefix length.
- `one_digit = s[i - 1]` - the single last character of the current prefix `s[0:i]`.
- `two_digit = s[i - 2:i]` - the last two characters together.
- `if one_digit != '0': dp[i] += dp[i - 1]` - if the last character alone is a valid decode (non-zero), every way of decoding the prefix *before* it (`dp[i-1]`) extends validly by adding this one character as its own letter.
- `if 10 <= int(two_digit) <= 26: dp[i] += dp[i - 2]` - if the last two characters together form a valid letter (10-26), every way of decoding the prefix before *those two* (`dp[i-2]`) extends validly by treating them as one combined letter.
- `return dp[n]` - the total ways to decode the entire string.

### Dry run
`s = "226"`, `n=3`

`dp[0]=1`. `dp[1] = 1` (since `s[0]='2' != '0'`).

- `i=2`: `one_digit = s[1] = '2'` (≠'0') → `dp[2] += dp[1] = 1` → `dp[2]=1`. `two_digit = s[0:2] = "22"`, `10<=22<=26` → `dp[2] += dp[0] = 1` → `dp[2]=2`.
- `i=3`: `one_digit = s[2] = '6'` (≠'0') → `dp[3] += dp[2] = 2` → `dp[3]=2`. `two_digit = s[1:3] = "26"`, `10<=26<=26` → `dp[3] += dp[1] = 1` → `dp[3]=3`.

Final: `dp[3] = 3` ✅ matches expected output ("BZ", "VF", "BBF").

### Time & space complexity
- **Time: O(n)**, **Space: O(n)** for the `dp` array (can be further space-optimized to O(1), since `dp[i]` only depends on the last two values - same trick as Climbing Stairs/House Robber).

---

## Common mistakes & misconceptions

1. **Forgetting that `'0'` can never be decoded alone.** Since there's no letter mapped to 0, a lone `'0'` digit immediately makes that decoding path invalid - this is why every approach explicitly checks `s[i] != '0'` (or equivalent) before allowing the single-digit branch.
2. **Allowing two-digit combinations starting with `'0'`** (e.g. treating `"06"` as valid, mapping to a nonexistent "06" letter). Only two-digit values from **10 to 26** are valid - `"01"` through `"09"` must be rejected, which the `10 <= int(two_digit) <= 26` check correctly enforces (since any string starting with '0' parses to a number under 10).
3. **Mixing up `dp[i]`'s meaning as "ways to decode `s[0:i]`" vs. "ways to decode using the first `i` digits ending exactly at digit `i`."** Both phrasings describe the same thing here, but it's easy to get the off-by-one index wrong when translating between "prefix length" and "0-indexed character position" - always double check `s[i-1]` (single) and `s[i-2:i]` (double) index correctly relative to `dp[i]`.
4. **Assuming a leading `'0'` anywhere blocks the entire string from being decodable**, rather than recognizing it only blocks *that specific position* from starting a single-digit decode (it might still be validly consumed as the second digit of a two-digit combination with the digit before it, if that combination is 10-20... though specifically only 10 or 20, since the tens digit must make the pair 10-26 - e.g. `"10"` and `"20"` are valid two-digit decodes even though they end in the otherwise-invalid-alone `'0'`).

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute force recursion | O(2^n) | O(n) | Exponential. |
| Top-down memoization | O(n) | O(n) | Cached. |
| Bottom-up tabulation | O(n) | O(n) | The standard, expected solution (further space-optimizable to O(1)). |

**Key takeaway:** string-decoding/segmentation DP problems (this, and Word Break) share the same shape: `dp[i]` built from checking a small, fixed number of ways to form the "last piece" ending at position `i`, added to the already-known count for whatever comes before that piece. Recognizing "how many valid last-pieces could end here, and what do I add for each" is the core skill for this whole family of problems.
