# Topic 5: Binary Search

## Core concepts / data structures

### Binary Search

**What it is:** a technique for finding a target value (or the boundary between "valid" and "invalid" answers) in a **sorted** sequence, by repeatedly cutting the search space **in half**, instead of checking every element one by one.

**The mental model that explains why it works, not just that it does:** binary search isn't fundamentally about "arrays" — it's about **eliminating half of all remaining possibilities with a single comparison**, which is only possible when the data has a *monotonic* structure (values only increase, or a condition only flips one direction, as you move across the search space). A sorted array is the simplest example: comparing the middle element to the target tells you, with certainty, that the *entire* other half can't contain the target (because everything on the wrong side is definitively too small or too large) — not "probably doesn't," but *provably* doesn't, because of the sort order. That certainty is what licenses discarding half the space outright, which is a fundamentally different move from "checking a spot and moving on" in a linear scan.

**Simple analogy:** imagine looking up a word in a paper dictionary. You don't start at page 1 and flip through every page — you open to roughly the middle, see whether your word comes before or after that page alphabetically, and then repeat the same "jump to the middle of what's left" process on just that half. Each guess eliminates half of what remains, so you find any word in a huge dictionary within a handful of guesses — and this only works because the dictionary is alphabetized; the same strategy on an unsorted pile of papers would tell you nothing.

**Why it's fast, precisely:** cutting the search space in half every step means the number of steps needed grows only as `log2(n)`. A sorted array of 1,000,000 elements needs at most about 20 comparisons, not 1,000,000 — each comparison does the work of eliminating roughly half of whatever's left, so the space shrinks exponentially fast relative to the number of steps taken. This O(log n) time is dramatically faster than a linear O(n) scan for large inputs, and the gap widens enormously as n grows (20 vs. 1,000,000 — not a small difference).

### The classic template
```python
def binary_search(arr, target):
    left, right = 0, len(arr) - 1
    while left <= right:
        mid = (left + right) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1   # target must be in the right half
        else:
            right = mid - 1  # target must be in the left half
    return -1  # not found
```

**Why `mid = (left + right) // 2`:** this picks the middle index of the current search range. (A subtle note for other languages: in languages with fixed-size integers, `left + right` can overflow for very large arrays, so `left + (right - left) // 2` is sometimes used instead — not a practical concern in Python, where integers grow arbitrarily large without overflow, but worth knowing why you might see it written that way elsewhere.)

**Why `left <= right` (not `<`):** the loop needs to still check the case where `left == right` — a single remaining candidate that hasn't been ruled out yet. Using `<` would exit the loop one iteration too early, skipping the check of that last remaining element.

## Common patterns / techniques in this topic

| Pattern | When it applies |
|---|---|
| **Classic search** | Find the exact index of a target value in a sorted array. |
| **Search on a rotated sorted array** | The array was sorted, then "rotated" (shifted) at some pivot point — one half of any given split is still guaranteed to be properly sorted, and you can determine which half based on comparing `arr[mid]` to `arr[left]`/`arr[right]`. |
| **Binary search on the answer** | The array/data itself might not even be the thing you're searching — instead, you binary search over a **range of possible answer values**, checking (via some "is this answer achievable?" function) whether to search higher or lower. Very common in "minimize the maximum" / "maximize the minimum" style problems, not covered directly in this Blind 75 list but worth knowing the shape of. |
| **Find a boundary** (leftmost/rightmost occurrence, or the point where a condition flips from true to false) | Instead of stopping at an exact match, keep narrowing until you've pinpointed exactly where some condition changes — useful for "first/last position of X" or finding the minimum in a rotated array. |

## Key terminology

- **Search space** — the range of indices (or values, for "search on the answer" problems) still being considered.
- **Pivot** — the point where a rotated sorted array "wraps around" (e.g. `[4,5,6,7,0,1,2]` was rotated at index 4).
- **Monotonic** — a function/condition that only ever goes one direction (e.g. always increasing, or flips from `False` to `True` and never back). Binary search fundamentally requires some kind of monotonic structure to work, whether that's sorted values or a predicate that flips exactly once — without monotonicity, eliminating "half the space" based on one comparison is no longer provably safe.
- **O(log n)** — the hallmark time complexity of binary search; if you see "the array is sorted" plus a requirement faster than O(n), that's a strong signal to consider binary search.

## Common beginner mistakes

1. **Forgetting the array must be sorted (or have some other monotonic structure).** Binary search on unsorted data gives wrong answers *silently* — it doesn't crash, it just returns garbage, because the "eliminate half" logic relies on an assumption (sortedness) that no longer holds. This is a genuinely dangerous failure mode, since the bug won't announce itself the way a crash would.
2. **Infinite loops from incorrect boundary updates.** If you write `left = mid` instead of `left = mid + 1` (or `right = mid` instead of `right = mid - 1`) in a branch where `mid` has already been ruled out, the search space might never shrink — `mid` gets re-included in the next iteration's range, and if `left`/`right` converge to the same value without progress, the loop never terminates. Always double check that every branch actually **excludes** whatever's been ruled out from the next range.
3. **Off-by-one errors with `<=` vs `<` in the while condition**, and with whether `right` starts at `len(arr) - 1` or `len(arr)`. These need to be consistent with how `mid` is calculated and how the boundaries update — mixing conventions from different templates half-remembered from different sources is a very common source of subtle bugs.
4. **Not handling duplicate values correctly** when searching for the *first* or *last* occurrence of a value, rather than just "any" occurrence — the update rule needs to specifically keep narrowing toward one side even after finding a match, rather than returning immediately on the first match found.
5. **On rotated sorted arrays: assuming both halves are sorted.** Only *one* of the two halves around `mid` is guaranteed to be properly sorted at any given step — the other half contains the "rotation point" and isn't uniformly ordered. Assuming both are safe to reason about the same way is a common and serious bug in rotated-array problems.

## How this compares to Arrays & Hashing

Hashing answers "does this exact value exist?" in O(1), but it fundamentally cannot answer questions like "where's the *boundary*, or the *closest* value, or which range of values satisfies some condition?" — hashing has no sense of order, because a hash function deliberately scatters values across slots with no relationship between a value's magnitude and its slot. Binary search is what you reach for specifically when order/structure is present and you can eliminate half the possibilities with a single comparison — a fundamentally different, order-dependent kind of speed-up than hashing's order-independent "remember what I've seen."

## Starter problems (easy, to warm up)

1. **Binary Search** (LeetCode #704) — not in Blind 75, but the pure, textbook version of the algorithm — do this first if the template above feels unfamiliar.
2. **Find Minimum in Rotated Sorted Array** (LeetCode #153) — in your Blind 75 list; a great next step once the classic template is comfortable.
3. **Search in Rotated Sorted Array** (LeetCode #33) — also in your Blind 75 list.

## What carries over from here

The "binary search on the answer" pattern (searching over a range of possible *answers*, not array indices) reappears in some Dynamic Programming and Greedy-adjacent problems later, where a brute-force check for "is answer X achievable?" combined with binary search over X can beat a naive scan. The core discipline of this topic — carefully reasoning about what each half of a split guarantees, and proving (not assuming) which half is safe to trust — also directly strengthens comfort with Divide and Conquer thinking, which underlies some Tree and Sorting-related techniques.
