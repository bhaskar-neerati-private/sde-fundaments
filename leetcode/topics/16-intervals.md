# Topic 16: Intervals

## Core concepts / data structures

### Interval
**What it is:** a range represented as `[start, end]`, typically representing something like a time span (a meeting from 2pm to 3pm, or a range of numbers). Interval problems ask questions about how a **collection** of these ranges relate to each other - do they overlap, can they be merged, how many overlap at once, etc.

**Simple explanation:** think of intervals as bars on a calendar - each one spans from a start time to an end time. Most interval problems boil down to: **sort the bars by some key (almost always start time, occasionally end time), then sweep through them once, comparing each to whatever came just before it.**

### The core building block: checking if two intervals overlap
```python
def overlaps(a, b):
    return a[0] <= b[1] and b[0] <= a[1]
```
Two intervals **don't** overlap exactly when one ends before the other begins (`a[1] < b[0]` or `b[1] < a[0]`) - so they **do** overlap whenever that's *not* the case. Being comfortable with this single check (and its negation) underlies almost every problem in this topic.

### Why sorting first is (almost) always the first move
Once intervals are **sorted by start time**, a hugely useful property emerges: you only ever need to compare each interval to the **most recently kept/processed** interval, not to every interval seen so far - because anything that could still be relevant to check against is guaranteed to be the most recent one (everything sorted before it has already either been merged in or is definitely non-overlapping with anything coming later, since starts only increase). This is precisely what turns most interval problems into an O(n log n) (dominated by the sort) single-pass algorithm, instead of something that needs to compare every pair of intervals (O(n²)).

## Common patterns / techniques in this topic

| Pattern | When it applies |
|---|---|
| **Sort by start time, then merge overlapping ones in one pass** | Merge Intervals - the canonical pattern; keep a "current merged interval," and either extend it (if the next interval overlaps) or finalize it and start a new one (if it doesn't). |
| **Sort by start time, insert a new interval in the correct position, then merge** | Insert Interval - handle intervals entirely before the new one, then intervals overlapping the new one (merging them all together), then intervals entirely after. |
| **Sort by END time, then greedily count non-overlapping intervals** | Non-overlapping Intervals, Meeting Rooms-style "can everything fit" problems - sorting by end time (not start time!) enables a greedy "always keep the interval that frees up the earliest" strategy. |
| **Track how many intervals are simultaneously active** (using a min-heap of end times, or a sweep-line of +1/-1 events) | Meeting Rooms II-style "how many resources needed at once" problems. |

## Key terminology

- **Overlap** - two intervals share at least one point in common (using the check above).
- **Merge** - combine two overlapping intervals into one covering their full combined range: `[min(a[0],b[0]), max(a[1],b[1])]`.
- **Sweep line** - conceptually "sweeping" through time in order, processing start/end events as they occur - the underlying idea behind Meeting Rooms II's "how many things are active at once" logic.
- **Sort by start vs. sort by end** - a critical, problem-specific choice: merging/overlap-detection problems typically sort by **start** time, while "greedily keep the maximum number of non-overlapping intervals" problems typically sort by **end** time (see Non-overlapping Intervals for why).

## Common beginner mistakes

1. **Forgetting to sort at all**, or sorting by the wrong key (start vs. end) for the specific problem - this single choice determines whether the rest of the greedy logic is even valid.
2. **Getting the overlap condition backward or off-by-one** - particularly around whether touching endpoints (e.g. `[1,3]` and `[3,5]`) count as "overlapping" or not; this genuinely varies by problem statement, so always check the exact definition given.
3. **Only comparing to the previous *original* interval, instead of the current *merged* interval**, when merging - after merging two intervals together, subsequent comparisons need to be against the newly merged (possibly extended) range, not the original un-merged one.
4. **Not handling the "new interval overlaps multiple existing intervals" case correctly** in Insert Interval - needing to merge across *several* consecutive intervals, not just one.
5. **Using a plain list/scan instead of a heap for "how many resources needed simultaneously"** (Meeting Rooms II) - this can turn an O(n log n) heap-based solution into a much slower O(n²) one.

## How this compares to Greedy

Interval problems are, almost without exception, greedy problems wearing a specific costume: sort by the right key, then make a single, never-reconsidered pass, always keeping whatever locally-best choice presents itself. The Greedy topic's core discipline (convince yourself the locally-best choice is provably safe, usually via an exchange-style argument) directly transfers here - and sorting is what *makes* many of these locally-best choices provably safe in the first place.

## Starter problems

Given this topic has 5 (fairly closely related) Blind 75 problems and no separate "easy" warm-ups listed elsewhere, the best approach is to work through them roughly in this order: **Insert Interval** (simplest single-interval-vs-list logic) → **Merge Intervals** (the core merging pattern) → **Non-overlapping Intervals** (introduces sort-by-end greedy reasoning) → **Meeting Rooms** → **Meeting Rooms II** (builds on Meeting Rooms with the heap-based simultaneous-count idea).

## What carries over from here

The "sort first, then make a single greedy pass" discipline built here is broadly transferable to scheduling and resource-allocation problems well beyond LeetCode. The min-heap-tracking-active-things idea from Meeting Rooms II also directly reuses the Heap/Priority Queue topic's core skills, showing how topics build on each other rather than existing in total isolation.
