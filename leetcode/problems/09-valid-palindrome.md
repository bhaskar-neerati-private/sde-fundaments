# 9. Valid Palindrome

**LeetCode:** [#125 - Valid Palindrome](https://leetcode.com/problems/valid-palindrome/) · **Topic:** [Two Pointers](../topics/02-two-pointers.md) · **Difficulty:** Easy

## Problem statement

Given a string `s`, return `true` if it's a palindrome after: converting all uppercase letters to lowercase, and removing all non-alphanumeric characters (keep only letters and digits).

**Example:**
```
Input: s = "A man, a plan, a canal: Panama"
Output: true
Explanation: "amanaplanacanalpanama" is a palindrome.

Input: s = "race a car"
Output: false
```

## Applicable approaches

- **Brute Force — Build a cleaned string, then compare to its reverse.**
- **Optimal — Two Pointers (in-place, no extra cleaned string needed).**

## Approach 1: Brute Force — Clean, then Compare to Reverse

### Intuition

Follow the problem's instructions as literally as possible: physically produce the cleaned version of the string the problem describes (lowercase letters and digits only), then check whether that cleaned string reads the same forwards and backwards by comparing it to its own reverse. This is a completely direct translation of the problem statement into code — no cleverness needed to get it *correct*, which makes it a good baseline; the only cost is that it allocates new strings to do so.

### Algorithm

1. Walk through `s`, keeping only alphanumeric characters, converted to lowercase, building a new string.
2. Compare the cleaned string to its reverse.
3. Return whether they're equal.

### Python code
```python
def isPalindrome(s):
    cleaned = "".join(ch.lower() for ch in s if ch.isalnum())
    return cleaned == cleaned[::-1]
```

### Line-by-line explanation

- `ch.isalnum()` — `True` if `ch` is a letter or digit (filters out spaces, commas, colons, etc.).
- `ch.lower()` — normalizes case so `'A'` and `'a'` are treated as the same character.
- `"".join(... for ch in s if ...)` — builds the cleaned string from only the characters that passed the filter.
- `cleaned[::-1]` — Python slice notation meaning "the whole string, reversed" (step -1 walks backwards); this allocates a brand new string.
- `cleaned == cleaned[::-1]` — a string is a palindrome exactly when it equals its own reverse, by definition.

### Dry run

`s = "A man, a plan, a canal: Panama"`

Cleaned (lowercased, alnum only): `"amanaplanacanalpanama"`

Reversed: `"amanaplanacanalpanama"` (same!) → equal → return `True`.

### Time & space complexity

- **Time: O(n)** — one pass to clean (visiting every character once), and reversing/comparing a string of length ≤ n is also O(n).
- **Space: O(n)** — we build an entirely new cleaned string, and Python's `[::-1]` allocates *another* new string for the reverse. This is exactly the cost the optimal approach below eliminates.

---

## Approach 2: Optimal — Two Pointers (No Extra String)

### Intuition

The brute force's O(n) extra space comes from a design choice, not a necessity: it transforms the *entire* string into a clean copy before checking anything, even though checking a palindrome only ever needs to compare pairs of characters from the two ends inward. We can perform that same comparison **directly on the original string**, using two pointers that start at both ends and move inward — skipping over non-alphanumeric characters as they're encountered, and comparing letters (case-insensitively) only when both pointers land on genuinely valid characters. This is the opposite-end pointer pattern from the topic overview: the "sortedness" this pattern usually depends on is replaced here by an even simpler structural guarantee — a palindrome must match at *mirrored* positions, so comparing from both ends inward can never skip past a mismatch, because a mismatch at any mirrored pair is immediately, unconditionally disqualifying.

### Algorithm

1. Set `left = 0`, `right = len(s) - 1`.
2. Loop while `left < right`:
   - If `s[left]` isn't alphanumeric, move `left` forward (skip it) and continue the loop without comparing yet.
   - If `s[right]` isn't alphanumeric, move `right` backward (skip it) and continue.
   - Otherwise, both pointers are on valid characters: compare them (case-insensitively). If they don't match, return `False`.
   - If they match, move both pointers inward (`left += 1`, `right -= 1`).
3. If the loop finishes without a mismatch, return `True`.

### Python code
```python
def isPalindrome(s):
    left, right = 0, len(s) - 1

    while left < right:
        if not s[left].isalnum():
            left += 1
            continue
        if not s[right].isalnum():
            right -= 1
            continue

        if s[left].lower() != s[right].lower():
            return False

        left += 1
        right -= 1

    return True
```

### Line-by-line explanation

- `left, right = 0, len(s) - 1` — pointers start at the very first and very last characters of the original (uncleaned) string.
- `while left < right` — stop once the pointers meet or cross. **Using `<`, not `<=`, is deliberate**: a single middle character (odd-length string) or two pointers meeting exactly (even-length) never needs comparison against itself — that case is trivially fine and the loop correctly does nothing further for it.
- `if not s[left].isalnum(): left += 1; continue` — skip past punctuation/spaces from the left side *without* comparing anything yet, then re-check the loop condition and re-run this same logic for the new `left`. This is what replaces the brute force's pre-pass cleaning: instead of building a clean string first, we skip junk characters lazily, on demand.
- `if not s[right].isalnum(): right -= 1; continue` — same idea from the right side.
- `if s[left].lower() != s[right].lower(): return False` — once both pointers are sitting on real letters/digits, compare them case-insensitively. Any mismatch anywhere is an immediate, final answer — there is no way a mismatch discovered here could be "fixed" by continuing (this is the correctness argument that justifies returning immediately, mirroring the topic overview's palindrome-check row).
- `left += 1; right -= 1` — characters matched, so move both pointers inward to check the next pair — advancing *both* pointers together is safe specifically because a match provides no reason to re-examine either character again.
- `return True` — if we make it all the way through without a mismatch, every mirrored pair matched, which is the definition of a palindrome.

### Dry run

`s = "race a car"` (expected `False`)

| left | right | s[left] | s[right] | action |
|---|---|---|---|---|
| 0 | 9 | 'r' | 'r' | alnum both, match → move in |
| 1 | 8 | 'a' | 'a' | match → move in |
| 2 | 7 | 'c' | 'c' | match → move in |
| 3 | 6 | 'e' | ' ' | right not alnum → right -= 1 |
| 3 | 5 | 'e' | 'a' | both alnum, `'e' != 'a'` → return `False` ✅ |

**A dry run that specifically exercises the skip logic**, using a shortened version of the full example: `s = "a, a"` (should be `True` — cleaned, it's just `"aa"`).

| left | right | s[left] | s[right] | action |
|---|---|---|---|---|
| 0 | 3 | 'a' | 'a' | both alnum, match → move in |
| 1 | 2 | ',' | ' ' | left not alnum → left += 1 |
| 2 | 2 | (loop condition `left < right` now `2 < 2` = False) | — | loop ends |

Return `True` ✅ — notice neither the comma nor the space was ever *compared*, only skipped; the algorithm correctly ignored them without ever materializing a cleaned string.

### Time & space complexity

- **Time: O(n)** — each pointer moves forward/backward at most n times total across the *entire* run of the loop (never backtracks), so the total combined work is linear, using the same "each pointer only ever advances" reasoning from the topic overview's monotonic-movement definition.
- **Space: O(1)** — no new string is built; we only use two integer pointers, working directly on the input. This is the entire point of the optimization: same time complexity as the brute force, strictly better space complexity.

---

## Common mistakes & misconceptions

1. **Using `while left < right` but then also needing to `continue` correctly.** A common bug is writing the skip logic with `if`/`elif` instead of separate `if ... continue` statements — with `elif`, if `s[left]` is invalid *and* gets skipped, the code might still fall through and (incorrectly) also try to evaluate `s[right]` in the same iteration before the loop condition is rechecked, potentially comparing a comparison that hasn't been fully validated yet. The explicit `continue` after each skip forces the loop condition to be rechecked from scratch on every character skip, which is the safe, correct structure.
2. **Comparing case-sensitively.** Forgetting `.lower()` on both sides means `"Aa"` would be judged as a mismatch, which is wrong per the problem's explicit case-insensitivity requirement.
3. **Assuming "two pointers" always means "the array must be sorted."** This problem doesn't rely on sortedness at all — it relies on the *mirror* structure of a palindrome. It's worth recognizing this is a different (though related) justification for using two pointers than the "sorted array, provably safe move" justification used in pair-sum problems — see the topic overview for both.
4. **Building the cleaned string "just to be safe," out of habit.** Once you understand *why* the in-place version is correct, defaulting back to the brute force's extra allocation isn't a safety net — it's giving up a real, free space optimization for no benefit.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Clean + Compare to Reverse | O(n) | O(n) | Simple, very readable, uses extra space for a cleaned copy and its reverse. |
| Two Pointers (in-place) | O(n) | O(1) | Same time complexity, but no extra string — the better answer when space matters, and the standard expected solution. |

**Key takeaway:** when a problem is fundamentally about comparing characters from both ends of a sequence, look for a way to do the comparison **directly on the original data** with two pointers, instead of first transforming the data into a cleaned copy — same time complexity, better space complexity, and the justification for the pointer movement here (mirror-position matching) is a genuinely different flavor of "provably safe move" than the sorted-array pair-sum pattern you'll see next in 3Sum.
