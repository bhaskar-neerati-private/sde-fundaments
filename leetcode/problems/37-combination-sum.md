# 37. Combination Sum

**LeetCode:** [#39 - Combination Sum](https://leetcode.com/problems/combination-sum/) · **Topic:** [Backtracking](../topics/09-backtracking.md) · **Difficulty:** Medium

## Problem statement

Given an array of **distinct** positive integers `candidates` and a `target`, return all **unique combinations** where the chosen numbers sum to `target`. The **same number may be chosen multiple times** (unlimited supply of each candidate). Combinations are considered the same regardless of order (so don't return both `[2,2,3]` and `[3,2,2]`).

**Example:**
```
Input: candidates = [2,3,6,7], target = 7
Output: [[2,2,3],[7]]
```

## Applicable approaches

- **Basic Backtracking (include/skip recursion)**.
- **Optimal — Backtracking with Sorting + Pruning** — avoids exploring clearly-hopeless branches.

## Approach 1: Basic Backtracking

### Intuition

Build combinations one number at a time, using the topic overview's general template directly. At each step, for the current candidate index, decide: either **include** this candidate again (since repeats are allowed) and stay at the same index, or **move on** to the next candidate index without using this one. Stop (record a valid answer) when the running sum exactly hits the target; stop (abandon this branch) if it exceeds the target — this second condition is a form of pruning, cutting off exploration the moment it's provably hopeless.

### Algorithm

1. Recursive helper `backtrack(start, path, remaining)`:
   - If `remaining == 0`: found a valid combination — record a copy of `path`.
   - If `remaining < 0`: overshot the target — abandon this branch.
   - Otherwise, for each candidate index `i` from `start` to the end:
     - Include `candidates[i]`: append it to `path`, recurse with `backtrack(i, path, remaining - candidates[i])` (**note: still `i`, not `i+1`**, since we're allowed to reuse the same number).
     - Undo: remove it from `path` before trying the next candidate.

### Python code
```python
def combinationSum(candidates, target):
    result = []

    def backtrack(start, path, remaining):
        if remaining == 0:
            result.append(path.copy())
            return
        if remaining < 0:
            return

        for i in range(start, len(candidates)):
            path.append(candidates[i])
            backtrack(i, path, remaining - candidates[i])  # same i: can reuse this candidate
            path.pop()  # undo

    backtrack(0, [], target)
    return result
```

### Line-by-line explanation

- `result = []` — collects all valid combinations found.
- `backtrack(start, path, remaining)` — `start` is the earliest candidate index we're allowed to pick next (this is what prevents generating the same combination in different orders, e.g. `[2,3]` and `[3,2]` — we always pick in non-decreasing index order, so a combination's numbers always appear in one canonical order in `path`); `path` is the combination built so far; `remaining` is how much more we still need to sum to.
- `if remaining == 0: result.append(path.copy()); return` — exactly hit the target — a valid combination. **Must copy** `path`, per the topic overview's explanation of why storing a reference would corrupt the recorded answer once `path` is mutated further.
- `if remaining < 0: return` — overshot the target — this branch can never succeed (all candidates are positive, so `remaining` can only get more negative from here) — stop exploring it, a genuine prune.
- `for i in range(start, len(candidates)):` — try every candidate from `start` onward (never going backward, which is what prevents duplicate combinations in different orders).
- `path.append(candidates[i])` — make the choice: include this candidate.
- `backtrack(i, path, remaining - candidates[i])` — recurse, passing `i` (not `i + 1`) as the new `start`, because the same candidate can be reused again — this is the specific detail that distinguishes "unlimited supply" problems from ordinary combination problems, where you'd pass `i + 1`.
- `path.pop()` — **undo** the choice before the loop tries the next `i` — this is the essential backtracking step from the topic overview, restoring `path` to its state before this candidate was tried, so the next iteration of the loop starts from a clean slate.

### Dry run

`candidates = [2,3,6,7]`, `target = 7`

- `backtrack(0, [], 7)`:
  - `i=0` (candidate 2): `path=[2]`, recurse `backtrack(0, [2], 5)`:
    - `i=0` (2 again): `path=[2,2]`, recurse `backtrack(0, [2,2], 3)`:
      - `i=0` (2 again): `path=[2,2,2]`, recurse `backtrack(0,...,1)`:
        - `i=0`: `path=[2,2,2,2]`, recurse with `remaining=-1` → `remaining < 0` → return immediately. Undo → `path=[2,2,2]`.
        - `i=1` (3): `path=[2,2,2,3]`, recurse with `remaining=1-3=-2` → return. Undo.
        - (candidates 6,7 also overshoot) → loop ends. Undo → `path=[2,2]`.
      - `i=1` (3): `path=[2,2,3]`, recurse `backtrack(1,...,0)`: `remaining==0` → **record `[2,2,3]`** ✅. Undo → `path=[2,2]`.
      - `i=2,3` (6,7): overshoot, no match. Undo → `path=[2]`.
    - `i=1` (3): `path=[2,3]`, recurse `backtrack(1,[2,3],2)`: `i=1`(3 again): `path=[2,3,3]`, `remaining=2-3=-1` → return, undo. `i=2,3`: overshoot. Loop ends, undo → `path=[2]`.
    - `i=2,3` (6,7): `2+6=8>7` and `2+7=9>7`, both overshoot immediately upon recursing (remaining becomes negative). Undo. Loop ends → `path=[]`.
  - `i=1` (3): `path=[3]`, recurse `backtrack(1,[3],4)`: (similar exploration) — no combination using only candidates ≥3 sums exactly to 4 (3+3=6 overshoots what's left, and 6,7 alone also overshoot) — no match found in this branch.
  - `i=2` (6): `path=[6]`, recurse `backtrack(2,[6],1)`: only candidates 6,7 available (index≥2), both overshoot 1. No match.
  - `i=3` (7): `path=[7]`, recurse `backtrack(3,[7],0)`: `remaining==0` → **record `[7]`** ✅.

Final `result = [[2,2,3], [7]]` ✅ matches expected output exactly.

### Time & space complexity

- **Time: O(2^target)** in a loose worst-case sense (or more precisely bounded by the branching factor and depth of the search tree, which depends on candidate values) — without the sorting-based pruning shown next, we may explore many branches that are doomed to overshoot before we ever check `remaining < 0`, since we don't know in advance which candidates are "too big" until we've already recursed one level into them.
- **Space: O(target / min(candidates))** for the maximum recursion depth (how many times the smallest candidate could be added before hitting the target), plus space for the output.

---

## Approach 2: Optimal — Sort First, Then Prune Early

### Intuition

If we **sort** the candidates first, we can add a powerful pruning rule: the moment `candidates[i] > remaining`, **every candidate after it (since they're sorted ascending) is also too big** — so we can `break` out of the loop entirely at that point, instead of merely `continue`-ing past this one candidate and checking every remaining one individually. This significantly cuts down wasted exploration compared to Approach 1, which only detects an overshoot *after* recursing one level deeper and hitting the `remaining < 0` base case — this version detects it *before* ever making that (doomed) recursive call at all.

### Python code
```python
def combinationSum(candidates, target):
    candidates.sort()
    result = []

    def backtrack(start, path, remaining):
        if remaining == 0:
            result.append(path.copy())
            return

        for i in range(start, len(candidates)):
            if candidates[i] > remaining:
                break  # sorted, so every candidate after this is also too big - stop entirely

            path.append(candidates[i])
            backtrack(i, path, remaining - candidates[i])
            path.pop()

    backtrack(0, [], target)
    return result
```

### Line-by-line explanation

- `candidates.sort()` — essential setup that makes the pruning rule valid; without sorting, "everything after this candidate is also too big" wouldn't be a safe conclusion, since a smaller candidate could appear later in an unsorted list.
- `if candidates[i] > remaining: break` — **the key optimization, worth being precise about why `break` and not `continue` is correct here**: since candidates are sorted ascending, if the current one already exceeds what's left to reach the target, every candidate from here to the end of the list is *also* too big (they're all ≥ this one) — so we can stop trying candidates at this recursion level entirely, rather than merely skipping this one candidate and needlessly checking the rest (which `continue` would do, wasting comparisons on candidates we can already prove are hopeless).
- Everything else is identical to Approach 1 — we've simply added an earlier, cheaper check that avoids ever making a doomed recursive call in the first place, compared to Approach 1, which only discovered the overshoot *inside* the next recursive call, one level of function-call overhead later.

### Time & space complexity

- **Time:** still exponential in the worst case (this is a fundamentally combinatorial search problem — there's no way around exploring a genuinely large search space in general when many valid combinations exist), but in practice, meaningfully fewer wasted branches are explored compared to Approach 1, since we prune *before* recursing rather than *after*.
- **Space: O(target / min(candidates))** for the recursion depth, same as before, plus O(n log n) for the initial sort (dominated by the search itself in most cases with nontrivial branching).

---

## Common mistakes & misconceptions

1. **Passing `i + 1` instead of `i` in the recursive call.** This is the single most consequential detail in this whole problem: it silently changes the problem from "unlimited reuse allowed" to "each candidate usable at most once" (which is a different LeetCode problem, Combination Sum II) — the bug wouldn't crash, it would just produce a different, incorrect set of combinations for *this* problem's actual rules.
2. **Using `continue` instead of `break` after sorting.** This still produces a correct answer (since `continue` just means "check the next one too," which will still eventually find or rule out every candidate), but it forfeits the efficiency gain of the pruning — worth understanding this is a performance bug, not a correctness one, unlike the `i+1` mistake above.
3. **Forgetting `path.copy()` when recording a result**, per the topic overview's general warning — every recorded combination would end up as a reference to the same, later-emptied `path` list.
4. **Sorting but forgetting that sorting also changes what "not going backward" via `start` actually enforces.** After sorting, `start` still correctly prevents duplicate combinations, but it's worth confirming this reasoning holds: since sorted order groups equal-valued candidates predictably and the `start` index only ever moves forward or stays the same (never backward), the "always pick in non-decreasing index order" duplicate-prevention logic remains valid after sorting — sorting doesn't interact badly with it.

## Summary

| Approach | Notes |
|---|---|
| Basic backtracking (include/skip, unsorted) | Correct, but doesn't prune overshooting branches until one level deeper. |
| Sorted + early-break pruning | The standard, expected optimized solution — same correctness, meaningfully less wasted exploration. |

**Key takeaway:** for backtracking problems involving numeric sums/constraints, **sorting first** often unlocks a cheap and powerful pruning rule (`break` instead of `continue`, or checking a condition *before* recursing rather than after) — this is one of the most common and high-value optimizations across the whole Backtracking topic, and the `i` vs `i+1` choice in the recursive call is the single detail most likely to silently change which variant of a combinatorial problem you're actually solving.
