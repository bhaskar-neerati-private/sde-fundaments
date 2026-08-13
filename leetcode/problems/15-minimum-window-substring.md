# 15. Minimum Window Substring

**LeetCode:** [#76 - Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/) · **Topic:** [Sliding Window](../topics/03-sliding-window.md) · **Difficulty:** Hard

## Problem statement

Given two strings `s` and `t`, return the **smallest substring** of `s` that contains **all** the characters of `t` (including matching duplicate counts — e.g. if `t` has two `'a'`s, the substring needs at least two `'a'`s too). If no such substring exists, return an empty string `""`.

**Example:**
```
Input: s = "ADOBECODEBANC", t = "ABC"
Output: "BANC"
Explanation: "BANC" is the smallest substring of s containing 'A', 'B', and 'C'.
```

## Applicable approaches

- **Brute Force** — check every substring of `s` to see if it contains all of `t`'s characters.
- **Optimal — Sliding Window with Two Frequency Maps** — the standard, expected O(n + m) solution.

## Approach 1: Brute Force

### Intuition

Check every possible substring of `s`, and for each one, verify whether it contains every character of `t` in sufficient quantity. Track the smallest one that qualifies. This is the most literal reading of the problem and is correct by exhaustion, but building and checking a frequency map for every one of the O(n²) substrings is enormously wasteful — most of those substrings share almost all their characters with their neighbors, exactly the situation Sliding Window is built to exploit.

### Algorithm

1. Build a frequency count of `t`'s characters.
2. For every substring `s[i:j+1]`, build its own frequency count and check if it "covers" `t`'s requirements (every character in `t`'s count map appears at least as many times in the substring's count map).
3. Track the smallest substring found that satisfies this.

### Python code
```python
def minWindow(s, t):
    if not t or not s:
        return ""

    need = {}
    for ch in t:
        need[ch] = need.get(ch, 0) + 1

    best = ""
    n = len(s)
    for i in range(n):
        for j in range(i, n):
            window_counts = {}
            for ch in s[i:j + 1]:
                window_counts[ch] = window_counts.get(ch, 0) + 1
            if all(window_counts.get(ch, 0) >= count for ch, count in need.items()):
                if best == "" or (j - i + 1) < len(best):
                    best = s[i:j + 1]
                break  # no need to extend this start further once a valid window is found
    return best
```

### Time & space complexity

- **Time: O(n³)** — O(n²) substrings, each requiring up to O(n) to build its frequency map and check coverage.
- **Space: O(n)** for the window frequency map (worst case).

*(This brute force is extremely slow and mainly useful to understand the problem correctly before optimizing — it wouldn't pass most real judges within the time limit for larger inputs.)*

---

## Approach 2: Optimal — Sliding Window with Two Frequency Maps

### Intuition

This is the "variable window, expand then shrink" pattern again, but the validity condition is more complex than in the previous two problems: the window is valid when it contains **at least as many of each character as `t` requires** — not a single aggregate number (like a sum or a max-frequency count), but a *simultaneous* set of per-character requirements. Checking "does this window satisfy all of `t`'s requirements" naively would mean scanning every required character on every window change — expensive if done repeatedly. The fix: track **how many distinct requirements are currently fully satisfied**, as a single running counter (`have`), updated incrementally exactly like every other quantity in this topic. We compare `have` against `required` (the total number of *distinct* characters `t` needs) — once `have == required`, the window is valid, and we can try shrinking it from the left to find the smallest valid version *before* continuing to expand.

We track two hash maps: `need` (fixed — what `t` requires) and `window` (what the current window currently has).

### Algorithm

1. Build `need`: character → required count, from `t`. Let `required = len(need)` (number of *distinct* characters `t` needs).
2. Maintain `window`: character → count, for characters currently in the sliding window (only tracking characters that are relevant, i.e. appear in `need`, though tracking all characters also works, just slightly less memory-efficient).
3. Maintain `have = 0` (how many distinct characters currently meet their required count in the window) and `left = 0`.
4. For each `right` from 0 to n-1:
   - Add `s[right]` to `window`'s count.
   - If `s[right]` is in `need` and `window[s[right]] == need[s[right]]` (just reached the exact required count for this character), increment `have`.
   - While `have == required` (the window currently satisfies every requirement):
     - If this window is smaller than the best found so far, record it.
     - Try shrinking: remove `s[left]` from `window`'s count. If `s[left]` is in `need` and its count just dropped *below* the required amount, decrement `have` (the window is no longer fully valid) — this will end the `while` loop on the next check.
     - Advance `left`.
5. Return the smallest valid window found (or `""` if none was ever found).

### Python code
```python
def minWindow(s, t):
    if not t or not s:
        return ""

    need = {}
    for ch in t:
        need[ch] = need.get(ch, 0) + 1
    required = len(need)

    window = {}
    have = 0
    left = 0
    best_len = float("inf")
    best_left = 0

    for right in range(len(s)):
        ch = s[right]
        window[ch] = window.get(ch, 0) + 1

        if ch in need and window[ch] == need[ch]:
            have += 1

        while have == required:
            if (right - left + 1) < best_len:
                best_len = right - left + 1
                best_left = left

            left_ch = s[left]
            window[left_ch] -= 1
            if left_ch in need and window[left_ch] < need[left_ch]:
                have -= 1
            left += 1

    return "" if best_len == float("inf") else s[best_left : best_left + best_len]
```

### Line-by-line explanation

- `need` / `required` — what `t` requires, and how many *distinct* characters need to be satisfied.
- `window`, `have`, `left` — the sliding window's current character counts, how many requirements are currently met, and the window's left edge.
- `best_len = float("inf")`, `best_left = 0` — track the smallest valid window found, using its length and starting index (slicing at the very end, rather than storing substrings repeatedly during the loop, avoids unnecessary string-copy overhead on every update).
- `window[ch] = window.get(ch, 0) + 1` — expand the window, including the new character.
- `if ch in need and window[ch] == need[ch]: have += 1` — **this check fires exactly once per character requirement**, the moment its count in the window reaches (not exceeds) what's required — going from "not enough" to "exactly enough." This is deliberately an `==` check, not `>=`: if it were `>=`, adding a *sixth* `'a'` when only one is needed would incorrectly re-increment `have` every time, since `window[ch] >= need[ch]` would stay true on every subsequent addition of that same character.
- `while have == required:` — the window currently satisfies every character requirement; try to shrink it as much as possible while staying valid, since we want the *smallest* such window, and any larger valid window can never be the answer once a smaller one from the same expansion is found.
- `if (right - left + 1) < best_len:` — record a new best if this valid window is smaller than any found before.
- `window[left_ch] -= 1` — remove the leftmost character's contribution.
- `if left_ch in need and window[left_ch] < need[left_ch]: have -= 1` — if removing that character dropped its count *below* what's required, the window is no longer fully valid — this will cause the `while` loop to stop after `left` advances (the condition `have == required` will be `False` on the next check).
- `left += 1` — shrink the window.
- Final line — reconstruct the answer substring from the recorded best start/length, or return `""` if no valid window was ever found (`best_len` never left its initial infinity value).

### Dry run

`s = "ADOBECODEBANC"`, `t = "ABC"` → `need = {A:1, B:1, C:1}`, `required = 3`

Walking `right` forward (abbreviated — showing key moments, not every single step, since this string is long):

- By `right = 5` (character `'C'` at `s[5]`), the window `s[0:6] = "ADOBEC"` has accumulated `A:1, D:2, O:2, B:1, E:2, C:1`. Checking against `need`: `A:1≥1` ✓ (triggered `have+=1` when `'A'` first appeared, at index 0), `B:1≥1` ✓ (triggered at index 3), `C:1≥1` ✓ (triggered at index 5, the moment we reach `right=5`). So `have` reaches `required=3` exactly when `right=5`.
- At `right=5`, `have == required == 3` → enter the `while` loop: current window `[0,5]` length 6, `best_len=6, best_left=0`. Try shrinking: remove `s[0]='A'`, `window['A']` drops to 0, which is `< need['A']=1` → `have -= 1` → `have=2`, `left=1`. Loop condition `have==required` now false (2≠3), stop shrinking.
- Continue expanding `right`. Eventually (after passing through more of the string), the window regains all three requirements again with a tighter left edge, and the shrink-loop runs again, potentially finding a smaller window each time this happens.
- By the end of the full scan — continuing this process through the remaining characters, including the final `"BANC"` segment (indices 9–12) which completes the window once `'A'`, `'N'`, `'C'` are included with `'B'` already present — the smallest valid window found is `"BANC"` (length 4, starting at index 9).

Final answer: `"BANC"` ✅

(The full step-by-step table for all 13 characters would be long; the key mechanic to understand, and the part worth being able to reproduce on a smaller example yourself, is: `have` only changes when a requirement transitions from unmet→met (expanding) or met→unmet (shrinking), and every time `have` hits `required`, the code immediately tries to shrink as tightly as possible before continuing to expand.)

### Time & space complexity

- **Time: O(n + m)** where n = len(s), m = len(t) — building `need` is O(m); the main loop has `right` moving forward n times and `left` also only ever moving forward across the whole algorithm (never resetting), so the combined total movement of both pointers is O(n) — the same amortized argument used throughout this topic.
- **Space: O(m)** for `need` (bounded by distinct characters in `t`), plus O(k) for `window` where k is bounded by the alphabet size (effectively O(1) for a fixed character set, or O(m) worst case).

---

## Common mistakes & misconceptions

1. **Using `window[ch] >= need[ch]` instead of `==` when updating `have`.** As explained above, `>=` re-triggers on every subsequent occurrence of an already-satisfied character, inflating `have` beyond the number of genuinely distinct satisfied requirements — a subtle bug that only manifests on inputs with repeated required characters.
2. **Forgetting the symmetric `<` check when decrementing `have` during shrink.** The decrement must specifically check that the count *dropped below* the requirement (`window[left_ch] < need[left_ch]`), not just that the character was removed — removing a character that's still present in sufficient quantity (e.g. `t` needs one `'a'`, window still has two after removing one) should *not* decrement `have`.
3. **Storing the actual substring inside the `while` loop instead of just tracking start/length.** This works but does unnecessary O(window size) string-copy work on every single valid-window check, rather than deferring the one necessary slice to the very end — a real, avoidable constant-factor cost in an already-hard problem.
4. **Tracking `window` counts for *every* character in `s`, not just characters relevant to `t`.** This still works correctly (extra irrelevant characters just never affect `have`), but wastes a small amount of memory and lookups; restricting `window` conceptually to only characters present in `need` is a minor but real optimization some solutions include.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute Force | O(n³) | O(n) | Correct but far too slow for real inputs. |
| Sliding Window + Two Maps | O(n + m) | O(m) | The standard, expected optimal solution. |

**Key takeaway:** this is the "hardest" version of the Sliding Window pattern in this list, but it's built from the same pieces as every other sliding window problem: an expand step, a validity check, and a shrink step. The new idea here is tracking validity with a **counter** (`have` vs `required`) instead of a single number comparison, because "does the window satisfy *every one* of several distinct requirements" needs more than one running value to check efficiently — and the `==` vs `>=` distinction when updating that counter is the single most important correctness detail in the whole solution.
