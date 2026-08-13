# 4. Group Anagrams

**LeetCode:** [#49 - Group Anagrams](https://leetcode.com/problems/group-anagrams/) · **Topic:** [Arrays & Hashing](../topics/01-arrays-hashing.md) · **Difficulty:** Medium

## Problem statement

Given an array of strings `strs`, group the anagrams together. You can return the groups in any order.

**Example:**
```
Input: strs = ["eat","tea","tan","ate","nat","bat"]
Output: [["bat"],["nat","tan"],["ate","eat","tea"]]
```

## Applicable approaches

- **Brute Force** — compare every string against every group formed so far.
- **Optimal — Group by Sorted-Key (Hash Map)** — the standard, expected solution.
- **Optimal (alternative) — Group by Character-Count Key** — avoids sorting entirely.

## Approach 1: Brute Force

### Intuition

This is the natural first instinct once you already know how to check if two strings are anagrams (previous problem): for each new string, compare it against a representative of every group formed *so far*; if it matches one, join that group, otherwise start a new one. It's correct because it directly implements the definition of "grouping" — but it repeats the anagram-check work for every existing group, every time, which is where the inefficiency lives.

### Algorithm

1. Keep a list of groups, where each group is a list of strings.
2. For each string `s` in the input: compare it (via the anagram check) against the *first* string of every existing group.
3. If a match is found, append `s` to that group.
4. If no group matches, start a new group containing just `s`.

### Python code
```python
def groupAnagrams(strs):
    groups = []
    for s in strs:
        placed = False
        for group in groups:
            if sorted(s) == sorted(group[0]):
                group.append(s)
                placed = True
                break
        if not placed:
            groups.append([s])
    return groups
```

### Line-by-line explanation

- `groups = []` — our list of groups so far, each a list of strings.
- `for s in strs` — process each string once.
- `for group in groups` — check this string against every existing group's representative (`group[0]`). **This inner loop is the source of the inefficiency**: in the worst case (e.g. all strings are anagrams of each other, so there's only ever one group), this doesn't cost much — but if there are many *distinct* groups (worst case: every string is unique, no two are anagrams), this inner loop grows linearly with the number of groups formed so far, and there can be up to n of them.
- `if sorted(s) == sorted(group[0])` — reuses the Valid Anagram sorting check to test if `s` belongs in this group.
- `group.append(s); placed = True; break` — found its group, add it, stop checking other groups.
- `if not placed: groups.append([s])` — didn't match any existing group, so `s` starts a brand new one.

### Dry run

`strs = ["eat", "tea", "tan"]`

| s | check against groups | result |
|---|---|---|
| "eat" | `groups` is empty | new group → `[["eat"]]` |
| "tea" | `sorted("tea")==sorted("eat")` → `['a','e','t']==['a','e','t']` yes | append → `[["eat","tea"]]` |
| "tan" | `sorted("tan")` vs `sorted("eat")` → no match (only one group to check, and it fails) | new group → `[["eat","tea"], ["tan"]]` |

### Time & space complexity

- **Time: O(n² · k log k)** where n = number of strings, k = max string length. For each of n strings, we may compare against up to n groups (worst case: no two strings are anagrams, so every string forms its own group and later strings check against a growing list of singleton groups), and each comparison sorts a string of length k (O(k log k)). The n² factor comes specifically from "checking each string against a number of groups that itself grows with n."
- **Space: O(n · k)** to store all the output groups.

---

## Approach 2: Optimal — Group by Sorted-Key (Hash Map)

### Intuition

The brute force's waste is re-comparing each new string against *every existing group*, one at a time, when a hash map could tell it exactly which group a string belongs to in O(1). The insight that makes this possible: **all anagrams of each other produce the exact same sorted string** — sorting is a canonical form, meaning every member of an anagram-group maps to one identical representative. If we use that sorted string as a hash map **key**, we're not asking "which of these existing groups matches me?" (a search over groups) — we're asking "what is my own canonical identity, and where does that identity live?" (a computation + lookup), exactly the addressing-not-searching shift from the topic overview. The hash map does the grouping automatically, as a side effect of every anagram computing the same key.

### Algorithm

1. Create a hash map from `key → list of original strings`.
2. For each string `s`, compute its sorted-letters key (sort the characters and join back into a string).
3. Append `s` to the list stored under that key (creating the list if it's the first string with this key).
4. Return all the values (the lists) from the map.

### Python code
```python
def groupAnagrams(strs):
    groups = {}  # sorted-key -> list of original strings
    for s in strs:
        key = "".join(sorted(s))
        if key not in groups:
            groups[key] = []
        groups[key].append(s)
    return list(groups.values())
```

Using `defaultdict` to skip the "if key not in groups" check:
```python
from collections import defaultdict

def groupAnagrams(strs):
    groups = defaultdict(list)
    for s in strs:
        key = "".join(sorted(s))
        groups[key].append(s)
    return list(groups.values())
```

### Line-by-line explanation

- `groups = {}` (or `defaultdict(list)`) — maps a sorted-letters key to the list of original strings that share it.
- `key = "".join(sorted(s))` — `sorted(s)` gives a list of characters in alphabetical order; `"".join(...)` turns that list back into a string, e.g. `"eat"` → `sorted` → `['a','e','t']` → joined → `"aet"`. Every anagram of "eat" produces this exact same key — this is the load-bearing fact the whole approach relies on.
- `if key not in groups: groups[key] = []` — first time we see this key, create an empty list for it.
- `groups[key].append(s)` — add the *original* (unsorted) string to its group — note we store `s`, not `key`, since the output needs the original strings, not their sorted forms.
- `list(groups.values())` — the dict's values are exactly the groups we want; order of groups doesn't matter per the problem.
- With `defaultdict(list)`, the "create empty list if missing" step happens automatically, so the loop body is just one line — functionally identical, just less boilerplate.

### Dry run

`strs = ["eat", "tea", "tan", "ate", "nat", "bat"]`

| s | key = sorted(s) joined | groups after this step |
|---|---|---|
| eat | "aet" | `{"aet": ["eat"]}` |
| tea | "aet" | `{"aet": ["eat","tea"]}` |
| tan | "ant" | `{"aet": [...], "ant": ["tan"]}` |
| ate | "aet" | `{"aet": ["eat","tea","ate"], "ant": ["tan"]}` |
| nat | "ant" | `{"aet": [...], "ant": ["tan","nat"]}` |
| bat | "abt" | `{"aet": [...], "ant": [...], "abt": ["bat"]}` |

Notice: no string was ever compared against another string directly — every single grouping decision was made by computing one key and doing one O(1) lookup, regardless of how many groups already existed. This is the direct fix for the brute force's O(n) group-scan.

Final `groups.values()`: `[["eat","tea","ate"], ["tan","nat"], ["bat"]]` ✅ matches the expected grouping (order of groups/order within groups can vary).

### Time & space complexity

- **Time: O(n · k log k)** — n strings, each needing O(k log k) to sort (k = string length). No factor of n from group-searching, since each string's group is found via one hash lookup, not a scan — this is what removes the brute force's n² term entirely.
- **Space: O(n · k)** — storing all strings in the map.

---

## Approach 3: Optimal (alternative) — Group by Character-Count Key

### Intuition

Sorting each string costs O(k log k), and if the alphabet is known and small (e.g. lowercase English letters, as LeetCode guarantees for this problem), there's a cheaper way to build a canonical key: **count how many times each of the 26 letters appears**, and use that fixed-size count-array (converted to a hashable `tuple`) as the key instead. Two strings are anagrams exactly when their letter counts match — this is a direct restatement of the anagram definition, and it avoids sorting's O(k log k) cost by building the key in O(k) via straightforward counting.

### Algorithm

1. For each string, build a length-26 array where index 0 = count of 'a', index 1 = count of 'b', etc.
2. Convert that array to a `tuple` (so it can be used as a dict key — lists aren't hashable, per the topic overview's explanation of why keys must be immutable).
3. Use this tuple as the key, same grouping logic as before.

### Python code
```python
from collections import defaultdict

def groupAnagrams(strs):
    groups = defaultdict(list)
    for s in strs:
        count = [0] * 26
        for ch in s:
            count[ord(ch) - ord('a')] += 1
        key = tuple(count)
        groups[key].append(s)
    return list(groups.values())
```

### Line-by-line explanation

- `count = [0] * 26` — a fresh 26-slot counter for this string, one slot per lowercase letter.
- `for ch in s: count[ord(ch) - ord('a')] += 1` — `ord(ch)` gives a character's numeric code (e.g. `ord('a') = 97`, `ord('b') = 98`); subtracting `ord('a')` maps `'a'`→0, `'b'`→1, ..., `'z'`→25, so we increment the right slot for each letter. This is direct addressing again, this time to compute an array index from a character.
- `key = tuple(count)` — lists can't be dict keys (not hashable, because their contents — and therefore their hash — could change after insertion, breaking the map, as explained in the topic overview); tuples are immutable and therefore hashable, converting the count list into a usable key.
- The rest is identical to Approach 2, just with a different (faster to build, no sorting) key.

### Dry run

`s = "eat"` → count array: `e`→ increment index 4, `a`→ index 0, `t`→ index 19 → `[1,0,0,0,1,0,...,1,...]` (1 at positions 0, 4, 19, zeros elsewhere) → converted to a tuple → used as the key. `s = "tea"` produces the exact same count array (same letters, same amounts) → same key → grouped together, without ever sorting either string — you can verify this by noting the *order* letters were encountered in never affects which count-array slot gets incremented.

### Time & space complexity

- **Time: O(n · (k + 26)) = O(n · k)** — building each string's count array is O(k) (one pass over its characters), no log factor from sorting; the fixed `+26` (iterating or converting a 26-slot array to a tuple) is a constant that doesn't scale with input.
- **Space: O(n · k)** for the output, plus O(26) = O(1) extra per string for the count array — genuinely constant per-string overhead, unlike the O(k log k) sort's larger constant.

---

## Common mistakes & misconceptions

1. **Using the sorted string as the key but forgetting to `join()` it back into a string.** `sorted(s)` returns a `list`, and lists aren't hashable — using the raw list as a dict key crashes with `TypeError`. The `"".join(...)` step isn't cosmetic; it's what makes the key usable at all.
2. **Assuming the character-count approach "just works" for any alphabet.** The `ord(ch) - ord('a')` trick specifically assumes lowercase English letters (values 0–25); it silently produces wrong (or out-of-bounds/negative) indices for uppercase letters, digits, or Unicode characters unless you widen the count array or handle the character set explicitly.
3. **Believing Approach 3 is always strictly better than Approach 2.** It has better asymptotic time complexity (O(k) vs O(k log k) per string), but for short strings the constant-factor overhead of building and hashing a 26-tuple can be comparable to or worse than sorting a short string — the asymptotic win only clearly matters as k grows large. Both are considered fully correct, expected-quality answers.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute Force | O(n² · k log k) | O(n·k) | Too slow for large inputs — group-searching grows with n. |
| Sorted-key hash map | O(n · k log k) | O(n·k) | Clean, easy to explain, the most common answer. |
| Character-count key | O(n · k) | O(n·k) | Fastest asymptotically; slightly more code, relies on a known fixed alphabet. |

**Key takeaway:** "group things that share a property" is a hash-map pattern — compute a **key that's identical for everything in the same group**, and let the map do the grouping in O(1) per item instead of scanning existing groups. The interesting engineering decision is just *how cheaply you can compute that key* (sorting vs. counting), not whether hashing applies — it always does, once you've found the right canonical key.
