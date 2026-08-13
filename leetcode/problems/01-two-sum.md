# 1. Two Sum

**LeetCode:** [#1 - Two Sum](https://leetcode.com/problems/two-sum/) · **Topic:** [Arrays & Hashing](../topics/01-arrays-hashing.md) · **Difficulty:** Easy

## Problem statement

Given an array of integers `nums` and an integer `target`, return the **indices** of the two numbers that add up to `target`.

- You may assume each input has **exactly one** valid answer.
- You cannot use the same element twice.
- Return the answer in any order.

**Example:**
```
Input: nums = [2, 7, 11, 15], target = 9
Output: [0, 1]
Explanation: nums[0] + nums[1] = 2 + 7 = 9
```

## Applicable approaches

- **Brute Force** — check every pair of numbers.
- **Optimal — Hash Map (one-pass)** — the standard, expected solution.

**What doesn't apply, and why:** Two Pointers (from the next topic) is *not* a good fit here, even though this problem is superficially "find a pair summing to a target" — the exact shape Two Pointers usually loves. The reason: Two Pointers needs the array **sorted** to work, and sorting `nums` would scramble the original indices, which is exactly what the problem asks you to return. You *could* sort `(value, original_index)` pairs and use Two Pointers on those, preserving the indices — but that's strictly more bookkeeping than the hash map approach for no speed benefit (both are effectively O(n), and Two Pointers here would cost an extra O(n log n) sort that the hash map approach doesn't need). So it's not that Two Pointers is *impossible*, it's that it's a worse tool for this specific version of the problem — don't reach for it just because "pair sums to target" pattern-matches.

---

## Approach 1: Brute Force

### Intuition

Before reaching for any clever data structure, ask the most literal possible question the problem allows: for every possible pairing of two numbers in the array, does that pair sum to the target? There's no cleverness here — this is the "try everything" baseline every optimization gets measured against. It's worth explicitly starting here (even in an interview) because it establishes correctness first, and because the inefficiency you're about to fix only makes sense once you've seen what it's fixing.

### Algorithm

1. Loop through the array with index `i`.
2. For each `i`, loop through the rest of the array with index `j` (starting right after `i`, so we never reuse the same element or check the same pair twice).
3. If `nums[i] + nums[j] == target`, return `[i, j]`.

### Python code
```python
def twoSum(nums, target):
    n = len(nums)
    for i in range(n):
        for j in range(i + 1, n):
            if nums[i] + nums[j] == target:
                return [i, j]
    return []  # no answer found (problem guarantees this won't happen)
```

### Line-by-line explanation

- `n = len(nums)` — store the length once. Trivial here, but it's a habit worth having: recomputing `len(nums)` inside a loop condition doesn't change asymptotic complexity in Python (`len()` is O(1)), but caching it makes the loop bounds obviously fixed and easier to reason about.
- `for i in range(n)` — the first number in the pair; try every possible starting position.
- `for j in range(i + 1, n)` — **this `i + 1` is not arbitrary and not just "for style."** Starting `j` at `0` would re-check pairs like `(1, 0)` after already checking `(0, 1)` — wasted work, since addition is commutative and we'd find the same sum twice. Starting `j` at `i` itself would allow `nums[i] + nums[i]`, using the same array slot twice, which the problem explicitly forbids ("you cannot use the same element twice"). `i + 1` is the exact boundary that avoids both problems at once.
- `if nums[i] + nums[j] == target` — the actual check.
- `return [i, j]` — return the moment a match is found; there's no reason to keep scanning once the (guaranteed unique) answer is in hand.
- `return []` — mathematically unreachable given the problem's guarantee of exactly one solution, but it's good practice to make the "no answer" path explicit rather than let the function implicitly return `None` if the loops exhaust.

### Dry run

`nums = [2, 7, 11, 15]`, `target = 9`

| i | j | nums[i] | nums[j] | sum | match? |
|---|---|---|---|---|---|
| 0 | 1 | 2 | 7 | 9 | ✅ yes → return `[0, 1]` |

Found on the very first comparison — this is a best-case dry run. To see the actual cost, imagine the answer were `nums[2]` and `nums[3]` instead: the loop would have to exhaust `j` for `i=0` (checking against 7, 11, 15 — 3 checks), then `j` for `i=1` (checking against 11, 15 — 2 checks), before finally reaching `i=2, j=3`. That's 6 checks for a 4-element array where the naive pair count is `C(4,2) = 6` — i.e., worst case, every pair gets checked.

### Time & space complexity

- **Time: O(n²).** For each of the n elements (outer loop), the inner loop scans up to the remaining elements — summing `(n-1) + (n-2) + ... + 1 = n(n-1)/2` comparisons in the worst case (no match until the very last pair, or genuinely no match at all). This is quadratic because we're comparing every element against every *other* element at least once — there is no shortcut being taken.
- **Space: O(1)** — no data structure grows with input size; just a couple of loop counters.
- **Best case:** O(1) if the first two elements happen to be the answer (as in the dry run above) — but you should never *design around* best case; always reason about worst case unless the problem guarantees favorable structure.

---

## Approach 2: Optimal — Hash Map (one-pass)

### Intuition

Here's the conceptual leap: the brute force is slow specifically because, for every number, it **re-searches the entire rest of the array** looking for a partner — and that search is the wasted work, not the "trying pairs" idea itself. Reframe the question: instead of "for this number, which *other* number completes it?" (a search), ask "have I already walked past the number that would complete *this* one?" (a lookup). Those sound similar but they're fundamentally different operations — one requires scanning, the other requires only remembering.

The trick is that a hash map lets you **remember every number you've walked past so far**, and check "have I seen X?" in O(1) instead of O(n). So instead of searching forward from each position, you search *backward through your own memory* — which costs nothing to query because it's addressed, not scanned (see the topic overview's mental model). This converts the brute force's O(n) *inner search, repeated n times* into an O(1) *lookup, done n times* — that's the entire source of the speedup, not some unrelated cleverness.

### Algorithm

1. Create an empty hash map `seen` that will store `value → index`.
2. Loop through the array once, with index `i` and value `num`.
3. Compute `complement = target - num` (the number we'd need to pair with `num` to reach the target).
4. If `complement` is already a key in `seen`, we've found our pair — return `[seen[complement], i]`.
5. Otherwise, add the current number to the map: `seen[num] = i`.
6. Continue to the next number.

### Python code
```python
def twoSum(nums, target):
    seen = {}  # value -> index
    for i, num in enumerate(nums):
        complement = target - num
        if complement in seen:
            return [seen[complement], i]
        seen[num] = i
    return []
```

### Line-by-line explanation

- `seen = {}` — our hash map, empty at the start. This holds every number we've walked past so far, mapped to its index — this is the "memory" that replaces the brute force's re-search.
- `for i, num in enumerate(nums)` — walk through the array once, giving us both the index `i` and the value `num` at each step.
- `complement = target - num` — the exact value that, paired with `num`, sums to `target`. If `target = 9` and `num = 2`, we need a `7` — this line computes what to look for, before we look for it.
- `if complement in seen` — O(1) hash map lookup: have we already walked past the number we need?
- `return [seen[complement], i]` — `seen[complement]` is the index where the partner was recorded earlier; `i` is the current index. Order doesn't matter per the problem statement.
- `seen[num] = i` — **this line runs *after* the check above, not before — and that ordering is the single most important detail in this solution.** If we inserted `num` into `seen` *before* checking for its complement, then for an input like `nums = [3, 3]`, `target = 6`, the number `3` would find *itself* already in the map and incorrectly return `[0, 0]` — using the same element twice, which the problem forbids. Checking first, inserting second, guarantees a match can only ever be a *different*, earlier-seen element.

### Dry run

`nums = [2, 7, 11, 15]`, `target = 9`

| i | num | complement (9 - num) | complement in seen? | action | seen after this step |
|---|---|---|---|---|---|
| 0 | 2 | 7 | No (`seen` is empty) | add `2 → 0` | `{2: 0}` |
| 1 | 7 | 2 | **Yes**, `seen[2] = 0` | return `[0, 1]` | — |

One pass, done after just 2 steps — but note this is a favorable ordering. Compare against a case where the match is spread further apart: `nums = [3, 2, 4]`, `target = 6`

| i | num | complement | in seen? | action | seen after |
|---|---|---|---|---|---|
| 0 | 3 | 3 | No | add `3 → 0` | `{3: 0}` |
| 1 | 2 | 4 | No | add `2 → 1` | `{3: 0, 2: 1}` |
| 2 | 4 | 2 | **Yes**, `seen[2] = 1` | return `[1, 2]` | — |

Check: `nums[1] + nums[2] = 2 + 4 = 6` ✅. This dry run is worth doing specifically because `3`'s complement (`3`) is never found — it's checking for itself-as-complement on the very first step, when `seen` is still empty, so it correctly moves on rather than incorrectly matching against nothing.

### Time & space complexity

- **Time: O(n).** A single pass through the array; each hash map lookup (`in`) and insert is O(1) *average case* (see the topic overview's note on hash collisions — worst case for a single operation is O(n) under pathological hashing, but this is not something you need to design around in practice, and the standard, expected answer for this problem is O(n)).
- **Space: O(n).** In the worst case (the two matching numbers are the very last two elements checked), the hash map ends up storing nearly every element before the match is found.
- **Contrast with brute force:** brute force is O(n²) time, O(1) space. This solution trades O(n) *extra* space for turning O(n²) time into O(n) — a classic time/space tradeoff, not a free win. If space were extremely constrained and n were small, brute force could theoretically be preferable, but for this problem's constraints, the hash map approach is unambiguously the expected answer.

---

## Common mistakes & misconceptions

1. **Inserting into `seen` before checking for the complement.** As explained above, this breaks the "can't use the same element twice" rule for inputs where a number is its own complement (e.g. `target` is exactly double some element). The fix is check-then-insert, always in that order.
2. **Assuming the hash map should store `index → value` instead of `value → index`.** You need to look up by *value* (does the complement exist?) and retrieve an *index* (what position was it at?) — storing it backward makes the lookup itself O(n) again, defeating the entire point of using a hash map.
3. **Reaching for Two Pointers because the problem "smells like" a pair-sum problem.** As discussed above, this specific version of Two Sum returns *original indices*, which sorting (required for Two Pointers) would destroy without extra bookkeeping. Recognize *why* a pattern applies, not just that it superficially matches.
4. **Forgetting the problem guarantees exactly one solution.** This means you don't need to handle "no answer found" as a real code path, and you don't need to keep searching after the first match — but don't let this guarantee make you sloppy about the check-then-insert ordering above; the "exactly one solution" guarantee is about the *problem's input*, not a license to skip correctness in *how* you find it.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute Force | O(n²) | O(1) | Simple but slow; establishes correctness before optimizing. |
| Hash Map (one-pass) | O(n) | O(n) | The expected answer; trades space for a huge time improvement. |

**Key takeaway:** the brute force's inefficiency is specifically "re-searching the remaining array for every element" — and the fix is specifically "replace searching with remembering," using a hash map's O(1) addressed lookup in place of an O(n) scan. This exact trick — and the exact check-before-insert ordering discipline — reappears constantly (Contains Duplicate, Group Anagrams, and many others in this list).
