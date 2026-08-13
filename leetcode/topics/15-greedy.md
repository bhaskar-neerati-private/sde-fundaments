# Topic 15: Greedy

## Core concepts / data structures

### Greedy algorithms
**What it is:** an approach where, at every step, you make whatever choice looks **best right now** (the "locally optimal" choice), without reconsidering that choice later - and for certain problems, this simple strategy provably produces the **globally optimal** answer, without needing the exhaustive exploration (and undoing) that Backtracking or the overlapping-subproblem caching of Dynamic Programming would require.

**Simple explanation:** imagine making change for a customer using the fewest coins, by always grabbing the **largest** coin denomination that doesn't overshoot the remaining amount, repeatedly - never going back to reconsider an earlier coin choice. For "normal" currency systems (like US coins), this greedy strategy happens to always produce the optimal (fewest-coins) answer - though it's worth knowing this isn't universally true for *every* possible set of coin denominations (this is actually why Coin Change, in the DP topic, needed full DP rather than a greedy shortcut - it doesn't assume a "nice" currency system).

**The hard part about greedy problems isn't writing the code - it's PROVING the greedy choice is actually safe.** Unlike DP (where you can often mechanically identify overlapping sub-problems and just cache them) or Backtracking (which explores everything, so it can't be "wrong," just slow), a greedy algorithm requires convincing yourself (or having it be a known result) that making the locally-best choice at each step, and *never reconsidering it*, can't ever lead you away from the true optimal answer. Getting this wrong produces code that runs fast but gives incorrect answers on some inputs - a distinctly dangerous failure mode compared to "just slow."

## Common patterns / techniques in this topic

| Pattern | When it applies |
|---|---|
| **Track a running best/worst as you scan once** | "Maximum subarray sum" (Kadane's algorithm) - at each position, decide whether to extend the previous run or start fresh, based on which is locally better right now. |
| **"Can I still reach the end from here?"** (reachability) | Jump Game-style problems - track the farthest position reachable so far, updating greedily as you scan; if you ever reach a position beyond your current farthest-reachable point before updating it, you're stuck. |
| **Sort first, then make locally greedy choices** | Many greedy problems (especially in the closely related Intervals topic, next) become tractable specifically *after* sorting by some key - the greedy choice is often "safe" only once the input is in the right order. |
| **Exchange argument (a way to justify a greedy choice, conceptually)** | The standard way to *prove* a greedy strategy works: show that any optimal solution could be modified to match the greedy choice at the first point they differ, without making the solution worse - implying the greedy choice was "at least as good," and by induction, greedy overall isn't worse than optimal. |

## Key terminology

- **Locally optimal choice** - the best decision available *right now*, without looking further ahead.
- **Globally optimal** - the actual best possible answer to the whole problem.
- **Greedy-choice property** - the (problem-specific) property that a locally optimal choice at each step leads to a globally optimal solution - this must be true for greedy to be a valid strategy for a given problem; it's not automatically true for every optimization problem.
- **Kadane's algorithm** - the specific, well-known greedy/DP-hybrid technique for Maximum Subarray, tracking a running "best sum ending here," resetting to 0 (or restarting) whenever the running sum goes negative.

## Common beginner mistakes

1. **Applying a greedy strategy without verifying it's actually correct for the specific problem** - a greedy-looking idea that seems intuitive can still be wrong; unlike DP (systematically correct if the recurrence is right) or backtracking (correct by exhaustive search), greedy correctness has to be specifically justified per-problem.
2. **Not resetting/restarting correctly** in "running best" greedy scans (like Kadane's algorithm) - forgetting that once a running sum goes negative, it should never be "carried forward" into future calculations, since starting fresh is always at least as good.
3. **Confusing "greedy" with "brute force with early stopping"** - true greedy algorithms make a single pass with O(1) decisions per step (or close to it), never backtracking; if you find yourself needing to "undo" a greedy choice, it's not actually a valid greedy algorithm for that problem.
4. **Forgetting to sort first when sorting would make the greedy choice provably safe** - many greedy problems that seem hard on unsorted input become straightforward once sorted by the right key.

## How this compares to Dynamic Programming

Greedy and DP both build up a solution incrementally, but DP explores **multiple** possible choices at each step and remembers the best sub-answers (because, for DP problems, the locally-best-looking choice isn't always safe to commit to permanently) - while greedy commits to **one** choice at each step, permanently, banking on the (problem-specific, provable) guarantee that this never costs you the optimal answer. A useful diagnostic: if you can convince yourself a locally-best choice is *always* safe to commit to without reconsideration, you likely have a greedy problem; if the best choice depends on information you don't have yet (like what the rest of the input looks like), you likely need DP instead.

## Starter problems (easy, to warm up)

1. **Maximum Subarray** (LeetCode #53) - in your Blind 75 list; the canonical greedy/DP-hybrid warm-up (Kadane's algorithm).
2. **Best Time to Buy and Sell Stock** (LeetCode #121, already covered in the Sliding Window topic) - worth revisiting here too, since its "track the running minimum" solution is also a valid greedy strategy, illustrating that the same problem can sometimes be framed through multiple topics' lenses.

## What carries over from here

Greedy reasoning is the backbone of the very next topic, **Intervals** - almost every interval-scheduling problem is solved by first sorting intervals by some key (start time, end time), then making a single greedy pass. The "prove the locally-best choice is always safe" discipline built here is exactly what's needed to correctly justify (not just guess at) interval-scheduling greedy strategies.
