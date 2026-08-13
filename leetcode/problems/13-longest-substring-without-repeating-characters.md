# 13. Longest Substring Without Repeating Characters

**LeetCode:** [#3 - Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/) · **Topic:** [Sliding Window](../topics/03-sliding-window.md) · **Difficulty:** Medium

## Problem statement

Given a string `s`, find the length of the longest substring **without repeating characters**.

**Example:**
```
Input: s = "abcabcbb"
Output: 3
Explanation: "abc" has length 3, and is the longest substring without repeats.
```

## Applicable approaches

- **Brute Force** — check every substring for repeated characters.
- **Better — Sliding Window with a Set** — variable window, shrink one step at a time.
- **Optimal — Sliding Window with a Hash Map (index jumps)** — shrink the window in one jump instead of one step at a time.

## Approach 1: Brute Force

### Intuition

Check every possible substring, and for each one, verify whether all its characters are distinct. This directly implements the definition — a substring qualifies if and only if it has no repeats — but it re-derives "does this substring have repeats" from scratch for every single substring, even ones that overlap heavily with substrings already checked, which is exactly the redundancy the sliding window approaches eliminate.

### Algorithm

1. For every pair `i <= j`, take the substring `s[i:j+1]`.
2. Check whether all characters in that substring are unique (e.g. by comparing `len(substring) == len(set(substring))`).
3. Track the longest substring that passes this check.

### Python code
```python
def lengthOfLongestSubstring(s):
    n = len(s)
    best = 0
    for i in range(n):
        for j in range(i, n):
            substring = s[i:j + 1]
            if len(substring) == len(set(substring)):
                best = max(best, len(substring))
    return best
```

### Line-by-line explanation

- Nested loops generate every substring `s[i:j+1]`, covering the full O(n²) space of contiguous substrings.
- `len(substring) == len(set(substring))` — converting to a set removes duplicates; if the lengths match, every character was already unique. This check itself costs O(j - i) — proportional to the substring's length — which is what pushes the total complexity past O(n²).
- `best = max(best, len(substring))` — track the longest valid substring found.

### Time & space complexity

- **Time: O(n³)** — O(n²) substrings, and checking each one (building a set from it) costs up to O(n) more work, since the check itself scans the substring.
- **Space: O(n)** for the set built per substring (this space is transient — discarded and rebuilt for the next substring — not accumulated).

---

## Approach 2: Better — Sliding Window with a Set

### Intuition

The brute force re-derives uniqueness from scratch for every substring, even though most substrings differ from their neighbors by just one character. Instead, maintain a **window** `[left, right]` that always contains only unique characters, tracked incrementally in a hash set — exactly the sliding window principle from the topic overview: update the tracked state (the set of characters currently in view) by adding/removing single elements, rather than recomputing it. Expand `right` one step at a time; if the new character is already in the window's set, shrink from the left (removing characters one at a time) until the duplicate is gone — then the window is valid again.

### Algorithm

1. Maintain a set `window` of characters currently in the window, and `left = 0`.
2. For each `right` from 0 to n-1:
   - While `s[right]` is already in `window`: remove `s[left]` from `window` and advance `left`.
   - Add `s[right]` to `window`.
   - Update `best = max(best, right - left + 1)` (current window size).

### Python code
```python
def lengthOfLongestSubstring(s):
    window = set()
    left = 0
    best = 0

    for right in range(len(s)):
        while s[right] in window:
            window.remove(s[left])
            left += 1
        window.add(s[right])
        best = max(best, right - left + 1)

    return best
```

### Line-by-line explanation

- `window = set()` — the set of characters currently "inside" the window.
- `left = 0` — the window's left edge.
- `for right in range(len(s))` — expand the window one character at a time.
- `while s[right] in window:` — the character we're about to add is already present somewhere in the window, meaning the window would no longer have all-unique characters if we added it as-is.
- `window.remove(s[left]); left += 1` — shrink the window from the left, one character at a time, until the duplicate character has been removed from the window entirely. **This is a `while`, not an `if`**, because the character equal to `s[right]` might not be the very first thing at `left` — several characters might need to be removed before the specific duplicate is gone.
- `window.add(s[right])` — now safe to add the new character; the window is valid again.
- `best = max(best, right - left + 1)` — `right - left + 1` is the current window's size (inclusive on both ends, hence the `+1` — see the topic overview's off-by-one warning); track the best seen.

### Dry run

`s = "abcabcbb"`

| right | s[right] | in window? | shrink | window after | left | window size | best |
|---|---|---|---|---|---|---|---|
| 0 | a | no | - | {a} | 0 | 1 | 1 |
| 1 | b | no | - | {a,b} | 0 | 2 | 2 |
| 2 | c | no | - | {a,b,c} | 0 | 3 | 3 |
| 3 | a | **yes** | remove s[0]='a', left→1 | {b,c} then add 'a' → {a,b,c} | 1 | 3 | 3 |
| 4 | b | **yes** | remove s[1]='b', left→2 | {a,c} then add 'b' → {a,b,c} | 2 | 3 | 3 |
| 5 | c | **yes** | remove s[2]='c', left→3 | {a,b} then add 'c' → {a,b,c} | 3 | 3 | 3 |
| 6 | b | **yes** | remove s[3]='a' (b still in window!) - so loop continues: remove s[4]='b', left→5 | {c} then add 'b' → {b,c} | 5 | 2 | 3 |
| 7 | b | **yes** | remove s[5]='c', left→6; s[6]='b' still in window → remove s[6]='b', left→7 | {} then add 'b' → {b} | 7 | 1 | 3 |

Final `best = 3` ✅ (the substring `"abc"`). Notice row `right=6`: the `while` loop had to run *twice* to fully clear the duplicate — this is the concrete case that proves why an `if` wouldn't suffice here.

### Time & space complexity

- **Time: O(n)** — `right` moves forward n times, and `left` also only ever moves forward, never resetting backward, so the total combined movement across the whole algorithm is O(n), not O(n²) — the exact amortized argument from the topic overview.
- **Space: O(min(n, alphabet size))** — the set holds at most one of each distinct character.

---

## Approach 3: Optimal — Sliding Window with a Hash Map (Index Jumps)

### Intuition

The set-based approach sometimes shrinks the window **one character at a time** even when we already *know* exactly where the duplicate is — the `while` loop in row `right=6` above removed two characters one at a time, even though we could have computed in advance that `left` needed to jump straight to index 5. A hash map storing **the most recent index of each character** lets us make that jump directly, in O(1), instead of stepping through the removal one character at a time.

### Algorithm

1. Maintain a hash map `last_seen` mapping character → the most recent index where it appeared.
2. Maintain `left = 0`.
3. For each `right` from 0 to n-1:
   - If `s[right]` is in `last_seen` **and** its last occurrence is at or after `left` (i.e. it's inside the *current* window, not an old occurrence that's already outside the window): jump `left` directly to `last_seen[s[right]] + 1` (one past the duplicate's last position).
   - Update `last_seen[s[right]] = right`.
   - Update `best = max(best, right - left + 1)`.

### Python code
```python
def lengthOfLongestSubstring(s):
    last_seen = {}
    left = 0
    best = 0

    for right in range(len(s)):
        ch = s[right]
        if ch in last_seen and last_seen[ch] >= left:
            left = last_seen[ch] + 1
        last_seen[ch] = right
        best = max(best, right - left + 1)

    return best
```

### Line-by-line explanation

- `last_seen = {}` — maps each character to the most recent index it was seen at, *regardless* of whether that occurrence is still inside the current window.
- `if ch in last_seen and last_seen[ch] >= left:` — the character was seen before, *and* that earlier occurrence is still inside the current window. **This second condition is not optional or redundant** — it's what prevents incorrectly shrinking the window for an "old" duplicate that's no longer actually in view (see the dry run below for a concrete case where skipping this check would produce a wrong answer).
- `left = last_seen[ch] + 1` — jump the window's left edge to just past where the duplicate last occurred — a single O(1) jump instead of a `while` loop removing characters one at a time.
- `last_seen[ch] = right` — always update the character's most recent position, whether or not the branch above ran.
- `best = max(best, right - left + 1)` — same window-size tracking as before.

### Dry run

`s = "abba"`

| right | ch | in last_seen & >= left? | left update | last_seen after | window size | best |
|---|---|---|---|---|---|---|
| 0 | a | no | left stays 0 | {a:0} | 0-0+1=1 | 1 |
| 1 | b | no | left stays 0 | {a:0,b:1} | 1-0+1=2 | 2 |
| 2 | b | **yes**, last_seen['b']=1 >= left(0) | left = 1+1 = 2 | {a:0,b:2} | 2-2+1=1 | 2 |
| 3 | a | last_seen['a']=0, but 0 >= left(2)? **No** (0 < 2, that 'a' is already outside the window) | left stays 2 | {a:3,b:2} | 3-2+1=2 | 2 |

Final `best = 2` ✅ (matches: `"abba"` has no length-3 substring without a repeat — the best is `"ab"` or `"ba"`, length 2).

**This dry run is specifically chosen to prove why `last_seen[ch] >= left` matters**: at `right=3`, `'a'` was last seen at index 0, but the window's `left` had already moved past that (to index 2) due to the earlier `'b'` collision. That old `'a'` is no longer actually inside the window — it's stale information. If the code jumped `left` based on it anyway (ignoring whether it's still in-window), it would compute `left = 0 + 1 = 1`, incorrectly *shrinking* the window backward when `left` was already at `2` — a genuine correctness bug, not just an inefficiency, since it could make the window invalidly re-include the already-excluded first `'b'`.

### Time & space complexity

- **Time: O(n)** — a single pass; `left` only ever moves forward (in jumps, not one-by-one, but the total distance it travels across the whole algorithm is still bounded by n, since it never exceeds `right`).
- **Space: O(min(n, alphabet size))** — the hash map holds at most one entry per distinct character.

---

## Common mistakes & misconceptions

1. **Omitting the `last_seen[ch] >= left` check when jumping.** As shown in the dry run above, this isn't a performance-only detail — it's a genuine correctness bug that causes `left` to move *backward*, silently corrupting the window's invariant (that it's currently duplicate-free) for later iterations.
2. **Believing the hash-map version is asymptotically faster than the set version.** Both are O(n) — the hash-map version is a **practical** optimization (fewer real operations, since it jumps instead of stepping), not an asymptotic improvement. In an interview, either is a fully correct optimal answer; don't claim a complexity difference that isn't there.
3. **Using `if` instead of `while` in the set-based version's shrink step.** As the dry run at `right=6` shows, a single duplicate can require removing more than one character from the left before the window is valid again — an `if` would leave the duplicate still present after one removal.
4. **Forgetting the `+1` in `right - left + 1` for window size.** A very common off-by-one, flagged generally in the topic overview — always sanity-check against a 1-element window (`left == right` should give size 1, not 0).
5. **Assuming this problem requires an actual substring to be returned.** The problem only asks for the *length* — code that unnecessarily slices and stores the actual substring at every step (rather than just tracking `right - left + 1`) adds needless O(n) work per update, which can silently degrade the overall complexity if done inside the loop.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute Force | O(n³) | O(n) | Checks every substring individually, re-deriving uniqueness from scratch each time. |
| Sliding Window + Set | O(n) | O(n) | Shrinks one character at a time; already optimal time complexity. |
| Sliding Window + Hash Map (jumps) | O(n) | O(n) | Same time complexity, but fewer actual operations in practice (jumps instead of one-by-one removal) — and requires the "still in window" check for correctness. |

**Key takeaway:** both sliding-window versions here are O(n) — the hash-map version is a **practical** optimization, not an asymptotically faster one. The "remember the last index something happened, but verify it's still relevant" trick — checking `last_seen[ch] >= left`, not just `ch in last_seen` — reappears in many other sliding-window problems and is the single most common correctness pitfall in this pattern.
