# 14. Longest Repeating Character Replacement

**LeetCode:** [#424 - Longest Repeating Character Replacement](https://leetcode.com/problems/longest-repeating-character-replacement/) · **Topic:** [Sliding Window](../topics/03-sliding-window.md) · **Difficulty:** Medium

## Problem statement

Given a string `s` (uppercase English letters) and an integer `k`, you can change **at most `k`** characters in the string to any other uppercase letter. Return the length of the longest substring you could make consist of a **single repeated character**, after performing at most `k` such changes.

**Example:**
```
Input: s = "AABABBA", k = 1
Output: 4
Explanation: Change the one 'B' in "AABA" (at index 3) to 'A' → "AAAA", length 4.
```

## Applicable approaches

- **Brute Force** — check every substring, count how many changes it needs.
- **Optimal — Sliding Window with Character Counts** — the standard, expected O(n) solution.

## Approach 1: Brute Force

### Intuition

For every substring, find its **most frequent** character — that's the character we'd keep, changing everything else to match it. The number of changes needed equals `(substring length) - (count of the most frequent character)`, since every non-majority character needs exactly one change. If that number is ≤ k, the substring is achievable within budget. This directly implements the problem's definition, but — like the previous problem's brute force — it re-derives the frequency breakdown for every substring almost from scratch.

### Algorithm

1. For every substring `s[i:j+1]`, count the frequency of each character within it.
2. Find the max frequency in that substring.
3. If `(length of substring) - (max frequency) <= k`, this substring is achievable; track its length as a candidate answer.

### Python code
```python
def characterReplacement(s, k):
    n = len(s)
    best = 0
    for i in range(n):
        counts = {}
        for j in range(i, n):
            counts[s[j]] = counts.get(s[j], 0) + 1
            length = j - i + 1
            max_freq = max(counts.values())
            if length - max_freq <= k:
                best = max(best, length)
    return best
```

### Line-by-line explanation

- Outer loop `i` fixes the substring's start.
- Inner loop `j` extends the substring one character at a time, updating a running frequency count `counts` for characters within `s[i:j+1]` — this is built incrementally *within* a fixed start `i`, which is why this is "only" O(n²·26) rather than the full O(n³) of the previous problem's brute force (each individual substring's count is built incrementally as `j` advances, rather than fully recomputed for every `(i,j)` pair) — but it still restarts entirely every time `i` advances, which is the specific redundancy the sliding window fixes.
- `max_freq = max(counts.values())` — the count of whichever character is currently most common in this substring; this costs up to O(26) since the alphabet is bounded.
- `length - max_freq` — how many characters would need to change (everything that *isn't* the most frequent character).
- `if ... <= k:` — achievable within the allowed number of changes; update `best`.

### Time & space complexity

- **Time: O(n² · 26) ≈ O(n²)** — O(n²) substrings considered (via the nested loop, with the inner loop building counts incrementally per fixed `i`), and for each, finding `max(counts.values())` costs up to O(26) (bounded alphabet size, treated as a constant).
- **Space: O(26) = O(1)** for the counts dict at any given time.

---

## Approach 2: Optimal — Sliding Window with Character Counts

### Intuition

This is the exact same "expand right, shrink left while invalid" shape as the Sliding Window pattern: maintain a window and a running frequency count of characters inside it. A window `[left, right]` is **valid** (achievable with ≤ k changes) exactly when `(window length) - (count of the most frequent character in the window) <= k` — i.e. the number of "non-majority" characters that would need changing doesn't exceed k. Expand the window by moving `right`; whenever it becomes invalid, shrink from the left until it's valid again. Track the largest valid window size seen.

**A subtle but important optimization worth understanding, not just copying:** we don't need to *recompute* the max frequency in the window from scratch every time the window shrinks — we can track `max_freq` as "the highest frequency value we've *ever* seen so far in any window state," and simply never bother decreasing it when the window shrinks. This looks incorrect at first — how can a stale, possibly-too-high `max_freq` still give a correct answer? — but it isn't: once we've established that a window of a certain size was achievable using some `max_freq`, we're only ever interested afterward in windows *at least that large* (since `best` only grows). An outdated (too-high) `max_freq` can only make the shrink condition `length - max_freq <= k` *more lenient* than it should be for the *current* window's true best character count — meaning the window might briefly stay "valid" (by this relaxed check) even when its *true* max frequency is lower. But critically, this can never cause the window to grow *larger* than a size that was actually achievable at some point with that `max_freq` value or higher — so `best` is never incorrectly inflated beyond a size that really was valid at some point in the scan. This is a genuinely subtle argument, and it's fine to treat it as a known, provably-safe trick rather than re-deriving it from scratch every time — but it's worth knowing *why* it's not just "an approximation that happens to work."

### Algorithm

1. Maintain a hash map `counts` of character → count within the current window, `left = 0`, and `max_freq = 0` (the highest single-character frequency ever seen in any window state so far).
2. For each `right` from 0 to n-1:
   - Increment `counts[s[right]]`.
   - Update `max_freq = max(max_freq, counts[s[right]])`.
   - While the window is invalid (`(right - left + 1) - max_freq > k`): decrement `counts[s[left]]` and advance `left`.
   - Update `best = max(best, right - left + 1)`.

### Python code
```python
def characterReplacement(s, k):
    counts = {}
    left = 0
    max_freq = 0
    best = 0

    for right in range(len(s)):
        counts[s[right]] = counts.get(s[right], 0) + 1
        max_freq = max(max_freq, counts[s[right]])

        while (right - left + 1) - max_freq > k:
            counts[s[left]] -= 1
            left += 1

        best = max(best, right - left + 1)

    return best
```

### Line-by-line explanation

- `counts = {}` — character → frequency, within the current window.
- `left = 0`, `max_freq = 0` — window's left edge, and the best (highest) single-character frequency observed in any window state so far.
- `counts[s[right]] = counts.get(s[right], 0) + 1` — expand the window by including `s[right]`, incrementing its count.
- `max_freq = max(max_freq, counts[s[right]])` — if this character's new count is a new record, update `max_freq` (as discussed above, we never explicitly *decrease* `max_freq` even when the window shrinks — this is a deliberate, provably-safe choice, not an oversight).
- `while (right - left + 1) - max_freq > k:` — window size minus the best single-character count so far exceeds the allowed number of changes → too many characters would need replacing → shrink.
- `counts[s[left]] -= 1; left += 1` — remove the leftmost character from the window's count, advance `left`.
- `best = max(best, right - left + 1)` — track the largest *valid* window size found; this line only runs after the `while` loop above has already run to completion, guaranteeing the window is valid at this point.

### Dry run

`s = "AABABBA"`, `k = 1`

| right | s[right] | counts after inc | max_freq | window len - max_freq | valid? (<=k) | shrink? | left | best |
|---|---|---|---|---|---|---|---|---|
| 0 | A | {A:1} | 1 | 1-1=0 | yes | no | 0 | 1 |
| 1 | A | {A:2} | 2 | 2-2=0 | yes | no | 0 | 2 |
| 2 | B | {A:2,B:1} | 2 | 3-2=1 | yes (1<=1) | no | 0 | 3 |
| 3 | A | {A:3,B:1} | 3 | 4-3=1 | yes | no | 0 | 4 |
| 4 | B | {A:3,B:2} | 3 | 5-3=2 | **no** (2>1) | shrink | — | — |

At `right=4`, before shrinking, window is `[0,4]` = "AABAB", length 5, `max_freq=3`, `5-3=2 > 1` → invalid, shrink once: remove `s[0]='A'`, `counts={A:2,B:2}`, `left=1`. Re-check: window `[1,4]`="ABAB", length 4, `max_freq` is still `3` (never lowered) → `4-3=1 <= 1` → valid now, stop shrinking. `best = max(4, 4) = 4`.

| right | continues... |
| 5 | B | counts[B]+=1 → {A:2,B:3} (window [1,5]="ABABB") | max_freq=max(3,3)=3 | len=5, 5-3=2>1 invalid | shrink: remove s[1]='A' →{A:1,B:3}, left=2 | recheck: window[2,5]="BABB" len4, 4-3=1<=1 valid | best=max(4,4)=4 |
| 6 | A | counts[A]+=1 → {A:2,B:3} (window [2,6]="BABBA") | max_freq=max(3,2)=3 | len=5, 5-3=2>1 invalid | shrink: remove s[2]='B'→{A:2,B:2}, left=3 | recheck: window[3,6]="ABBA" len4, 4-3=1<=1 valid | best=max(4,4)=4 |

Final `best = 4` ✅ matches expected output. Note how `max_freq` stayed at 3 — a value technically established for an *earlier* window state (`"AABA"`) — through the rest of the algorithm, and the answer was still computed correctly. Even though the true max frequency in the final window `"ABBA"` is only 2 (not 3), the check `4 - 3 = 1 <= 1` still correctly judged the window valid, and no window larger than 4 was ever incorrectly accepted — exactly the safety property argued in the intuition section.

### Time & space complexity

- **Time: O(n)** — `right` moves forward n times; `left` only ever moves forward across the whole algorithm (never resets), so total combined movement of both pointers is O(n) — the same amortized argument as every other sliding window problem.
- **Space: O(26) = O(1)** — the counts map holds at most one entry per letter of a fixed-size alphabet (uppercase English letters, per the problem's constraints).

---

## Common mistakes & misconceptions

1. **"Fixing" the perceived bug by recomputing `max_freq` from scratch after every shrink.** This is unnecessary work (it would cost O(26) per shrink, adding a constant factor) — and worse, some naive attempts to "fix" it introduce actual bugs by using `max(counts.values())` on a dict that's had keys with 0 counts inconsistently left in or removed. The stale `max_freq` approach is correct as-is; recomputing isn't required and isn't free either.
2. **Believing the window ever shrinks back below its best-ever size.** It doesn't need to, and the algorithm doesn't try to make it — the window's *length* is monotonically non-decreasing in aggregate (it can shrink locally but the `best` tracked separately never un-records a valid answer); don't confuse "the window's current size might drop" with "the answer can decrease."
3. **Forgetting this problem is restricted to uppercase English letters, and hardcoding a 26-slot array without checking the constraint.** A dict-based `counts` (as shown above) is naturally robust to this, but if you optimize toward a fixed-size array for a small constant speedup, make sure the problem's actual character set matches your assumption.
4. **Confusing this with Longest Substring Without Repeating Characters.** That problem forbids *any* repeats; this one specifically wants to *maximize* repeats of a single character, tolerating up to k "wrong" characters. The validity conditions are near-opposites in spirit, and mixing up which one you're solving leads to reversing the shrink condition.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute Force | O(n²) | O(1) | Recomputes frequency info per substring start, incrementally within each start but not across starts. |
| Sliding Window + Counts | O(n) | O(1) | The standard, expected optimal solution. |

**Key takeaway:** when a window's "validity" depends on an aggregate property (here: count of the most frequent character), track that aggregate incrementally as the window changes, rather than recomputing it from scratch. And don't assume every tracked value needs to stay perfectly up-to-date on every shrink — sometimes (as with `max_freq` here) a value that's allowed to go "stale" in one specific, provably-safe direction is still correct for the exact comparison it's used in.
