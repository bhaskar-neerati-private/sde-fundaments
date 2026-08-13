# 3. Valid Anagram

**LeetCode:** [#242 - Valid Anagram](https://leetcode.com/problems/valid-anagram/) · **Topic:** [Arrays & Hashing](../topics/01-arrays-hashing.md) · **Difficulty:** Easy

## Problem statement

Given two strings `s` and `t`, return `true` if `t` is an **anagram** of `s` (uses exactly the same letters, the same number of times each, just possibly rearranged), and `false` otherwise.

**Example:**
```
Input: s = "anagram", t = "nagaram"
Output: true

Input: s = "rat", t = "car"
Output: false
```

## Applicable approaches

- **Sorting (baseline).**
- **Optimal — Frequency Counting (Hash Map).**

**What doesn't apply, and why:** there isn't a meaningful O(n²) "brute force" here the way there is for pair-finding problems — the naive idea (try every permutation of `t` and see if any equals `s`) is O(n!), not O(n²), and it's such an extreme overkill that it's not a useful stepping stone; nobody writes it even as a first pass, because the "compare sorted versions" idea is just as immediate to think of and is already reasonable. So this problem's progression is baseline (sorting) → optimal (counting), not brute force → optimal.

## Approach 1: Sorting (baseline)

### Intuition

Two strings are anagrams exactly when they contain the same letters the same number of times — order doesn't matter. Sorting is a direct way to *cancel out* the "order doesn't matter" part of the definition: if you rearrange both strings into the one canonical order (alphabetical), two strings that are anagrams of each other must become **character-for-character identical**, and two strings that aren't can never become identical this way, because sorting doesn't change *what* letters exist or how many of each — only their order. This reduces "are these anagrams" (a fuzzy, order-independent comparison) to "are these two strings exactly equal" (a precise, trivial comparison).

### Algorithm

1. If the strings have different lengths, they can't be anagrams — return `False` immediately (a cheap early exit).
2. Sort the characters of `s` and the characters of `t`.
3. Compare the sorted results — if equal, it's an anagram.

### Python code
```python
def isAnagram(s, t):
    if len(s) != len(t):
        return False
    return sorted(s) == sorted(t)
```

### Line-by-line explanation

- `if len(s) != len(t): return False` — different lengths can never be anagrams (an anagram by definition uses *exactly* the same multiset of letters, which requires the same count). This check is O(1) and avoids doing O(n log n) sorting work on inputs that can be rejected instantly.
- `sorted(s)` — returns a list of `s`'s characters in alphabetical order, e.g. `sorted("anagram")` → `['a','a','a','g','m','n','r']`.
- `sorted(s) == sorted(t)` — if both sorted letter-lists are identical, the original strings contained exactly the same letters the same number of times; this is a direct consequence of sorting being a deterministic, order-canceling transformation.

### Dry run

`s = "anagram"`, `t = "nagaram"`

- `sorted("anagram")` → `['a','a','a','g','m','n','r']`
- `sorted("nagaram")` → `['a','a','a','g','m','n','r']`
- Equal → return `True`.

`s = "rat"`, `t = "car"` (expected `False`):
- `len("rat") == len("car")` → 3 == 3, passes the length check.
- `sorted("rat")` → `['a','r','t']`. `sorted("car")` → `['a','c','r']`.
- Not equal (`'t'` vs `'c'` differ at index 2, and in fact `'t'` doesn't appear in `sorted("car")` at all) → return `False`.

### Time & space complexity

- **Time: O(n log n)** — dominated by sorting both strings; each sort is O(n log n) where n is the string length.
- **Space: O(n)** — `sorted()` creates new lists (Python strings are immutable, so sorting always produces a new list, never modifies in place).

---

## Approach 2: Optimal — Frequency Counting (Hash Map)

### Intuition

Sorting proves two strings have the same letters by putting them in the *same order* — but order was never actually part of the question; it was only a tool to make comparison easy. We can prove "same letters, same counts" more directly, and more cheaply, by literally counting how many times each letter appears in each string and comparing those counts — no ordering needed at all. This is the general lesson: **when a problem's real question is "same multiset of things," counting is a more direct tool than sorting, and it avoids paying the O(n log n) sorting cost for information you don't actually need (the order).**

### Algorithm

1. If lengths differ, return `False` (same early exit as before).
2. Build a frequency count of every character in `s` (dict: char → count).
3. Walk through `t`, and for each character, decrease its count in the same dict.
4. If any character's count goes negative (or doesn't exist), `t` uses a letter more often than `s` does → not an anagram, return `False`.
5. If we get through all of `t` without going negative, and lengths matched, all counts balanced to zero → it's an anagram.

### Python code
```python
def isAnagram(s, t):
    if len(s) != len(t):
        return False

    counts = {}
    for ch in s:
        counts[ch] = counts.get(ch, 0) + 1

    for ch in t:
        if ch not in counts or counts[ch] == 0:
            return False
        counts[ch] -= 1

    return True
```

A shorter version using `collections.Counter` (does the same thing internally):
```python
from collections import Counter

def isAnagram(s, t):
    return Counter(s) == Counter(t)
```

### Line-by-line explanation (explicit version)

- `if len(s) != len(t): return False` — same cheap early exit as before.
- `counts = {}` — our frequency map, char → how many times it appears in `s`.
- `for ch in s: counts[ch] = counts.get(ch, 0) + 1` — for every character in `s`, increment its count. `.get(ch, 0)` returns 0 if `ch` isn't in the dict yet, which is what makes the first occurrence of a letter correctly become `1` rather than crashing on a missing key.
- `for ch in t:` — now walk through `t` and try to "cancel out" what we counted from `s`. **This is the key idea**: building up a count and then tearing it back down is a common technique for checking two things balance out without ever needing a second dictionary.
- `if ch not in counts or counts[ch] == 0:` — if `t` has a letter that either never appeared in `s`, or has already been fully "used up" (count reached 0, meaning `t` has *more* of this letter than `s` had), then `t` has more of that letter than `s` does → not an anagram. **This single line handles two distinct failure modes** — a letter genuinely absent from `s`, and a letter present but over-used — and it's worth noticing both are needed, because checking only `ch not in counts` would miss the case where `t` uses a valid letter too many times.
- `counts[ch] -= 1` — "use up" one occurrence of this letter.
- `return True` — if we survive the whole loop over `t` without hitting the failure condition, and lengths matched (checked at the start), every letter balanced out exactly — this final claim relies on the length check: without it, a shorter `t` that's a strict subset of `s`'s letters would incorrectly pass, since it would never trigger the failure condition but also wouldn't have "used up" everything.

### Dry run

`s = "rat"`, `t = "car"` (expected `False`)

Build counts from `s`: `{'r': 1, 'a': 1, 't': 1}`

Walk through `t = "car"`:
| ch | in counts & count > 0? | action |
|---|---|---|
| c | **No** (`'c'` never appeared in `s`) | return `False` |

Correctly detects `t` has a letter (`c`) that `s` doesn't have — and notice it fails on the *first* character, without needing to check `a` or `r` at all, since one failure is enough.

**A passing dry run:** `s = "anagram"`, `t = "nagaram"`

Counts from `s`: `{'a': 3, 'n': 1, 'g': 1, 'r': 1, 'm': 1}`

Walk through `t = "nagaram"`, decrementing each letter as we go — every letter is found with count > 0 each time, and after processing all 7 characters of `t`, every count that was touched has returned to 0. No failure triggered → return `True`.

### Time & space complexity

- **Time: O(n)** — two linear passes (one to build counts, one to consume them); dict operations are O(1) average.
- **Space: O(1)** *if the alphabet is fixed and small* (e.g. lowercase English letters → at most 26 keys in the dict, bounded regardless of string length, so treated as constant space). In general, **O(k)** where k is the number of distinct characters, which becomes O(n) in the worst case for arbitrary Unicode input where every character could be distinct.

---

## Common mistakes & misconceptions

1. **Only checking `ch not in counts`, forgetting the `counts[ch] == 0` case.** This misses "t uses a valid letter too many times" — e.g. `s = "aab"`, `t = "abb"` would incorrectly pass if you only checked presence, not remaining count, because `'b'` genuinely exists in `counts` even after it's been fully used up.
2. **Comparing sorted strings but forgetting the length check first.** `sorted("ab") == sorted("aab")` is trivially `False` since Python list comparison checks length implicitly — so this specific omission happens to be harmless for the *sorting* approach, but it's worth understanding *why* it's harmless there (list equality checks length as part of the comparison) versus why it's *not* harmless for the counting approach (a missing length check there can let a strict-subset case slip through, as noted above).
3. **Assuming Unicode/case-sensitivity is handled automatically.** Both approaches here are inherently case-sensitive and treat every distinct character as distinct — if a problem variant wanted case-insensitive matching, you'd need to `.lower()` both strings first; don't assume normalization is "free" just because the core algorithm is correct.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Sorting | O(n log n) | O(n) | Simple, easy to explain, slightly slower. |
| Frequency Counting | O(n) | O(1)–O(n) | Faster; the standard optimized answer. |

**Key takeaway:** whenever a problem is really asking "do these two collections contain the same elements, the same number of times, regardless of order?" — frequency counting with a hash map is almost always faster than sorting, because sorting pays for information (order) the question never actually needed.
