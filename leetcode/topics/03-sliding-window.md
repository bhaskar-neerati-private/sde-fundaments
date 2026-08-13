# Topic 3: Sliding Window

## Core concepts / data structures

### The Sliding Window technique

**What it is:** a way to examine every contiguous subarray/substring of a sequence without recomputing everything from scratch for each one. You maintain a "window" — a contiguous range `[left, right]` — and instead of restarting from scratch every time the window changes, you **incrementally** update your tracked information as the window expands (grow `right`) or contracts (grow `left`).

**The conceptual leap, precisely stated:** a brute-force scan of every contiguous subarray inherently considers O(n²) ranges (there are `n(n+1)/2` of them), and if evaluating each range costs even O(1) beyond the scan itself, you're at O(n²) or worse. Sliding Window's insight is that as the window shifts by one position, *most* of its contents don't change — only one element enters (at `right`) and, when shrinking, one element leaves (at `left`). If you can update your tracked state (a sum, a count, a frequency map) in O(1) per element added/removed, rather than recomputing it from the whole window every time, then the *total* work across the entire scan — summed over every window ever considered — is bounded by how many times any single element can enter and leave the window, which is a small constant (each element enters once and leaves at most once). That's what makes the technique O(n) instead of O(n²): it's not "a clever shortcut," it's a direct consequence of doing incremental updates instead of full recomputation.

**Simple analogy:** imagine looking at a stretch of a book through a physical window that you can slide along the page, and stretch or shrink. Instead of re-reading everything inside the window from the start every time you move it a little, you just note what new text came into view and what text left view — much faster than re-reading the whole visible section from scratch each time, and specifically because reading *new* text is proportional to how much text is new, not to how much text is currently visible.

### The two window shapes

**1. Fixed-size window** (the window's width never changes, only its position)
```python
window_sum = sum(arr[:k])  # first window
best = window_sum
for right in range(k, len(arr)):
    window_sum += arr[right]        # new element enters on the right
    window_sum -= arr[right - k]    # oldest element leaves on the left
    best = max(best, window_sum)
```
Used for: "find the best sum/average/etc. of every subarray of exactly size k."

**2. Variable-size window** (the window grows and shrinks based on a condition)
```python
left = 0
window_state = ...  # e.g. a running sum, or a hash map of counts inside the window
for right in range(len(arr)):
    # add arr[right] into window_state
    while <window is invalid, e.g. too big / violates a constraint>:
        # remove arr[left] from window_state
        left += 1
    # window [left, right] is now valid; update the answer here
```
Used for: "find the longest/shortest subarray satisfying some condition" — the window expands (via `right`) until it breaks a rule, then shrinks (via `left`) until it's valid again, repeating until `right` reaches the end.

## Common patterns / techniques in this topic

| Pattern | When it applies |
|---|---|
| **Fixed window + running total** | "Max/min sum (or average) of every subarray of exactly size k." |
| **Variable window, expand-then-shrink** | "Longest substring/subarray satisfying condition X" (e.g. no repeating characters, at most k distinct characters). |
| **Variable window with a hash map of counts** | Tracking character/element frequency *within* the current window (e.g. "at most k distinct characters," "contains all characters of another string"). |
| **Shrink-while-invalid loop** | The `while` loop inside the `for` loop that pulls `left` forward until the window becomes valid again — this is what makes the whole thing O(n) instead of O(n²). |

## Key terminology

- **Window** — the current contiguous range `[left, right]` being considered.
- **Expand** — moving `right` forward to include a new element in the window.
- **Shrink / contract** — moving `left` forward to exclude an element from the window (usually because the window became invalid, or because we're specifically searching for the *shortest* valid window).
- **Amortized O(n)** — the rigorous justification, not just a claim: even though there's a `for` loop with a `while` loop nested inside it (which superficially looks like it could be O(n²)), `left` only ever moves forward across the *entire* run of the algorithm — it is **never reset backward**. Since `left` can move forward at most n times total (bounded by the array's length), the combined number of iterations of the inner `while` loop, summed across *every* outer iteration, cannot exceed n. This is the exact same "total movement bounded by n, regardless of how it's distributed across outer iterations" argument used to show Two Pointers algorithms are O(n) — Sliding Window is that same argument applied to a window's boundary instead of two independent pointers converging.

## Common beginner mistakes

1. **Thinking a nested `for` + `while` is automatically O(n²).** As explained above, if `left` (or whatever the inner loop's variable is) never resets backward, the total combined work across the whole algorithm is still O(n). Always check whether the "inner loop" variable resets or just keeps moving forward — this single check distinguishes a true O(n) sliding window from an accidental O(n²) one (which happens if, for instance, `left` gets reset to some earlier value instead of only ever advancing).
2. **Forgetting to shrink the window after it becomes invalid, or shrinking only once instead of using a `while`.** The shrink step needs to run in a loop (potentially removing several elements), not just once conditionally with an `if` — a single expansion can require the window to shrink by more than one element to become valid again, especially if a duplicate character with a very old previous occurrence just entered.
3. **Off-by-one on window size.** For fixed windows, mixing up `right - left + 1` (inclusive window size, correct) vs `right - left` (off by one, a very common bug) — always sanity-check with a window of exactly two elements: `left=0, right=1` should give size 2, and `1 - 0 = 1` is visibly wrong while `1 - 0 + 1 = 2` is correct.
4. **Updating the "best answer" at the wrong point in the loop.** A common bug is recording the max window length *before* the shrink step has actually run, meaning it reflects a window that hasn't yet been confirmed valid — the update to the answer should happen only after the shrink loop has fully executed and the window is guaranteed to satisfy the constraint again.
5. **Not correctly removing an element's effect when it leaves the window.** E.g. decrementing a character's count in a frequency map when `left` moves past it, but forgetting that if your window-validity check depends on something like "number of distinct keys currently in the map," you may also need to delete the key entirely once its count hits 0 — leaving a stale `key: 0` entry in the map can silently corrupt a distinct-count check.

## How this compares to Two Pointers

Sliding Window is best understood as a **specialization of Two Pointers**: both use two indices moving through the array, and both rely on the same "total movement bounded by n" argument for their O(n) complexity. The difference is *what* each pointer pair is tracking. Plain Two Pointers (as in 3Sum or Container With Most Water) usually only cares about the two elements the pointers currently point *at* — the value at `left` and the value at `right` individually. Sliding Window cares about **everything between the pointers** — the whole window's aggregate state (a sum, a set of character counts, etc.). If you've built comfort with the "two indices, moving based on a provably-safe condition" shape from Two Pointers, Sliding Window is the natural next step: same shape, but now the thing you're incrementally maintaining is a summary of an entire *range*, not just two endpoint values.

## Starter problems (easy, to warm up)

1. **Best Time to Buy and Sell Stock** (LeetCode #121) — a simplified sliding window (track the minimum seen so far, i.e. a window that only ever grows from a fixed low point, never needing an explicit shrink step). Also in your Blind 75 list.
2. **Maximum Average Subarray I** (LeetCode #643) — not in Blind 75, but the purest fixed-size window example, good to build the basic pattern before tackling variable windows.
3. **Contains Duplicate II** (LeetCode #219) — not in Blind 75, but a nice variable-window-with-a-set warm-up ("are there two equal values within distance k of each other?").

## What carries over from here

The "expand right, shrink left while invalid" shape shows up again (in a different form) in **Backtracking** (try something, undo it if it doesn't work — a conceptually similar "advance, then retreat when a constraint is violated" rhythm, though achieved through recursion rather than pointer movement) and, more directly, the fixed/variable window bookkeeping habits (running sums, hash maps tracking "what's currently active") reappear in **Intervals** problems, which are conceptually about overlapping ranges — a close cousin of sliding windows over arrays.
