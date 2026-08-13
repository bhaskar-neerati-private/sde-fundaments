# 52. Word Break

**LeetCode:** [#139 - Word Break](https://leetcode.com/problems/word-break/) · **Topic:** [1-D Dynamic Programming](../topics/13-1d-dynamic-programming.md) · **Difficulty:** Medium

## Problem statement

Given a string `s` and a list of strings `wordDict`, return `true` if `s` can be segmented into a space-separated sequence of one or more words from `wordDict` (words can be reused any number of times).

**Example:**
```
Input: s = "leetcode", wordDict = ["leet","code"]
Output: true  ("leet" + "code")
```

## Applicable approaches

- **Brute Force Recursion** - try every possible split point.
- **Top-down Memoization**.
- **Bottom-up Tabulation** - the standard, expected optimal solution.

## Approach 1: Brute Force Recursion

### Intuition
To check if `s[i:]` (the remaining suffix starting at index `i`) can be broken into dictionary words: try every possible "first word" length - if some prefix of `s[i:]` is a valid dictionary word, recursively check if the *rest* of the suffix can also be broken.

### Python code
```python
def wordBreak(s, wordDict):
    words = set(wordDict)

    def canBreak(start):
        if start == len(s):
            return True
        for end in range(start + 1, len(s) + 1):
            if s[start:end] in words and canBreak(end):
                return True
        return False

    return canBreak(0)
```

### Time & space complexity
- **Time: O(2^n)** worst case - many overlapping sub-problems (`canBreak(i)` for the same `i` gets called repeatedly via different split paths).
- **Space: O(n)** recursion depth.

---

## Approach 2: Top-down Memoization

### Intuition
Cache the result of `canBreak(i)` for each starting index, since the same index gets asked about repeatedly across different recursive paths.

### Python code
```python
def wordBreak(s, wordDict):
    words = set(wordDict)
    memo = {}

    def canBreak(start):
        if start == len(s):
            return True
        if start in memo:
            return memo[start]

        for end in range(start + 1, len(s) + 1):
            if s[start:end] in words and canBreak(end):
                memo[start] = True
                return True

        memo[start] = False
        return False

    return canBreak(0)
```

### Time & space complexity
- **Time: O(n²)** - n distinct starting indices, each trying up to n possible end points, with O(n) substring slicing cost per check (some implementations reduce this further, but O(n²) to O(n³) is typical depending on substring handling).
- **Space: O(n)** for the memo and recursion depth.

---

## Approach 3: Optimal - Bottom-up Tabulation

### Intuition
Define `dp[i]` = "can the prefix `s[0:i]` be fully segmented into dictionary words?" Build this up from `dp[0] = True` (an empty prefix trivially "breaks" into zero words) up to `dp[n]` (the answer for the whole string), by checking, for each position `i`, whether there's some earlier valid breakpoint `j` such that `dp[j]` is `True` **and** `s[j:i]` is a dictionary word.

### Algorithm
1. `dp = [False] * (n + 1)`, with `dp[0] = True` (base case: empty string is trivially breakable).
2. For each `i` from 1 to n: for each `j` from 0 to i-1: if `dp[j]` is `True` and `s[j:i]` is in the word set, set `dp[i] = True` and stop checking further `j` for this `i` (no need to find more than one valid way).
3. Return `dp[n]`.

### Python code
```python
def wordBreak(s, wordDict):
    words = set(wordDict)
    n = len(s)
    dp = [False] * (n + 1)
    dp[0] = True

    for i in range(1, n + 1):
        for j in range(i):
            if dp[j] and s[j:i] in words:
                dp[i] = True
                break

    return dp[n]
```

### Line-by-line explanation
- `dp[0] = True` - the empty prefix (before any characters) is trivially "successfully segmented" (using zero words) - the necessary base case that lets the rest of the recurrence bootstrap correctly.
- `for i in range(1, n + 1):` - build up whether each prefix `s[0:i]` is breakable, in increasing order of `i`.
- `for j in range(i):` - try every possible position for the **start** of the *last* word in a valid segmentation of `s[0:i]`.
- `if dp[j] and s[j:i] in words:` - this combination is valid exactly when the prefix *before* `j` (i.e. `s[0:j]`) is already known to be breakable (`dp[j]` is `True`) **and** the remaining piece `s[j:i]` is itself a valid dictionary word.
- `dp[i] = True; break` - found one valid way to break `s[0:i]` - that's enough (we don't need to find every way, just whether it's *possible*), so stop checking further `j` values for this `i`.
- `return dp[n]` - is the *entire* string (`s[0:n]`, i.e. all of `s`) breakable?

### Dry run
`s = "leetcode"`, `wordDict = ["leet","code"]` → `words = {"leet","code"}`, `n=8`

`dp[0] = True`.

- `i=1..3`: no `j` makes `s[j:i]` a dictionary word (checking single/short prefixes like "l","le","lee" - none match). `dp[1..3] = False`.
- `i=4`: `j=0`: `dp[0]=True`, `s[0:4]="leet"` in words? **Yes** → `dp[4]=True`.
- `i=5..7`: checking various `j` where `dp[j]=True` - only `dp[4]=True` is available as a "True" predecessor so far. `s[4:5]="c"`, `s[4:6]="co"`, `s[4:7]="cod"` - none are in `words`. `dp[5..7]=False`.
- `i=8`: `j=4`: `dp[4]=True`, `s[4:8]="code"` in words? **Yes** → `dp[8]=True`.

`dp[8] = True` ✅ (the whole string "leetcode" is breakable as "leet"+"code").

### Time & space complexity
- **Time: O(n² · k)** where n = len(s), k = average cost of a substring slice/lookup (roughly O(n) for slicing, though set lookup itself is O(1) average for the substring once created) - overall commonly cited as O(n²) to O(n³) depending on how substring costs are counted; regardless, a dramatic improvement over the brute force's exponential blowup.
- **Space: O(n)** for the `dp` array (plus O(k) for the word set).

---

## Common mistakes & misconceptions

1. **Trying to greedily match the longest possible dictionary word at each position.** This is a classic trap: greedily taking the longest match can lead down a path that turns out to be a dead end later, when a *shorter* first match would have allowed the rest of the string to be segmented successfully - only exhaustive checking (backtracking or DP) is correct, not a greedy shortcut.
2. **Forgetting the `dp[0] = True` base case, or getting confused about what index `dp[i]` represents.** `dp[i]` means "is the prefix of length `i`" (i.e. `s[0:i]`) breakable - it's easy to off-by-one this into meaning `s[0:i-1]` or `s[i]` itself, which silently shifts every subsequent check by one position.
3. **Not breaking out of the inner loop once a valid split is found**, and instead continuing to check all `j` unnecessarily. Not a correctness bug, but a performance one worth being aware of - the moment `dp[i]` is confirmed `True`, no further `j` values can add any new information for that `i`.
4. **Re-deriving the string-slicing cost as if it were free.** `s[j:i]` in Python creates a new string, an O(i-j) operation, not O(1) - the true worst-case time complexity is often understated as plain O(n²) when it's more precisely closer to O(n³) once slicing costs are counted, as the complexity note above acknowledges rather than glossing over.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute force recursion | O(2^n) | O(n) | Exponential, massive redundant recomputation. |
| Top-down memoization | O(n²) | O(n) | Same recursive shape, cached. |
| Bottom-up tabulation | O(n²) | O(n) | The standard, most commonly expected solution. |

**Key takeaway:** "can this string/sequence be built from smaller valid pieces" is a classic 1-D DP shape - `dp[i]` represents "is the prefix up to i valid," built by checking every possible last-piece boundary `j` against `dp[j]` and whether the piece from `j` to `i` is itself valid. This exact shape (boolean DP over string prefixes) reappears in several other string-segmentation-style problems beyond this specific one.
