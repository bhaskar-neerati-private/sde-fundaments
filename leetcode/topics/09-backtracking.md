# Topic 9: Backtracking

## Core concepts / data structures

### Backtracking

**What it is:** a way of solving problems by **trying** a choice, **recursively exploring** everything that follows from that choice, and then **undoing** the choice ("backtracking") to try the next alternative — systematically exploring every possible combination/arrangement, but abandoning ("pruning") any path early the moment it's clear it can't lead to a valid answer.

**Simple analogy:** imagine navigating a maze by trying a path, and if you hit a dead end, walking back to the last junction and trying a different direction, rather than starting over from the entrance every time. Backtracking is exactly this: try, recurse, and if it doesn't work out, undo the last step and try something else — reusing all the progress made before that point, rather than discarding it.

**Why it's different from plain recursion, precisely:** every backtracking solution *is* recursive, but the defining extra piece is the **undo** step. After a recursive call returns (having fully explored that choice and everything after it), you explicitly reverse whatever change you made before trying the *next* choice at this level. The reason this undo step is non-negotiable, not optional cleanup: backtracking problems typically explore a **single shared, mutable structure** (an array being built up, a grid being marked) across many different branches of exploration — if a change made while exploring one branch isn't undone before exploring a sibling branch, that sibling incorrectly "inherits" state that doesn't actually apply to it. Forgetting the undo step is the single most common backtracking bug, and it's a direct consequence of this shared-mutable-state design.

### The general template
```python
def backtrack(path, choices_remaining):
    if <path is a complete, valid answer>:
        result.append(path.copy())  # or however the answer should be recorded
        return

    for choice in choices_remaining:
        if <choice is invalid right now>:
            continue  # skip it, don't even try

        path.append(choice)               # make the choice
        backtrack(path, updated_choices)  # explore everything that follows
        path.pop()                        # undo the choice - THE KEY STEP
```

**Why `path.copy()` when recording an answer, specifically:** `path` is being mutated (appended to / popped from) throughout the whole recursive exploration — if you store a reference to it directly without copying, every stored "answer" would actually be a reference to the *same* list object, and every one of those stored answers would appear to change (or end up empty) by the time the whole recursion finishes, since `path` is still being modified after being "stored." This is the exact same "don't lose/corrupt a reference" discipline as the Linked List topic's "save before overwrite" rule, just applied to a list reference instead of a node pointer.

## Common patterns / techniques in this topic

| Pattern | When it applies |
|---|---|
| **Subsets / combinations** | "Generate all possible subsets/combinations" — at each step, decide whether to include the current element or not (or which element to pick next). |
| **Permutations** | "Generate all possible orderings" — at each step, pick any not-yet-used element next. |
| **Constraint satisfaction (grid/board search)** | "Find a path/arrangement satisfying rules" (e.g. Word Search, N-Queens) — try a move, recurse, undo if it doesn't pan out, mark/unmark cells as "in use" along the way. |
| **Pruning** | Adding an early exit (`if <clearly can't work>: continue` or `return`) *before* recursing further, to avoid wasted exploration — this is what keeps backtracking from being pure, unfiltered brute force, and it's a genuine optimization, not just style. |
| **Marking visited/used state, then unmarking** | In grid search, marking a cell as visited before recursing into it, then unmarking it after backtracking — because that same cell might legitimately need to be revisited via a *different* path later, and prematurely leaving it "permanently visited" would incorrectly block that. |

## Key terminology

- **Backtrack / undo** — reversing a choice after fully exploring what it leads to, so the next alternative can be tried cleanly, on a "blank slate" with respect to that choice.
- **Prune / pruning** — cutting off a branch of exploration early, once it's clear it can't lead to a valid or better answer, to avoid wasted work — this is the backtracking analog of Binary Search's "provably eliminate half the space," except here we're eliminating a branch based on a specific detected violation, not a general halving.
- **State space / search tree** — the full set of all possible choices/paths being explored; backtracking is a systematic way to explore this space without visiting the same complete state twice, and without exploring dead ends further than necessary.
- **In-place marking** — modifying the actual input/board temporarily (e.g. marking a visited cell) instead of creating new copies, then reverting it — saves memory but requires careful undo discipline, exactly as described above.

## Common beginner mistakes

1. **Forgetting to undo a choice** (`path.pop()`, un-marking a visited cell, etc.) after the recursive call returns. Without this, later branches of the exploration incorrectly "remember" earlier choices that should no longer apply — a bug that often doesn't crash, just silently produces wrong or missing results, since the shared structure quietly drifts out of sync with what each branch actually represents.
2. **Storing a reference instead of a copy** when recording a valid answer, causing all recorded answers to end up identical (or empty) once the underlying mutable structure changes later in the recursion — as explained above, this is the exact same class of bug as losing a linked-list reference, just for a different data type.
3. **Not pruning when possible**, leading to exploring far more of the search space than necessary (e.g. not stopping early once a partial combination sum already exceeds the target, in a "combinations that sum to X" problem) — correctness isn't affected, but performance can be dramatically worse.
4. **Off-by-one in the "start index"** for combination-style problems — forgetting to prevent re-selecting the same element by only allowing subsequent choices to start from *after* the current one (unless the problem explicitly allows repeats, in which case the start index should stay the same, not advance).
5. **Confusing "subsets/combinations" (order doesn't matter, each choice made at most once per position) with "permutations" (order matters, need to track which elements are already used, not just an index)** — these need meaningfully different loop structures: combinations use a "start index" that only ever advances, permutations need a separate "used" tracker since any not-yet-used element can go in any position.
6. **Mutating a shared structure without appropriately marking/unmarking it** in grid-search-style problems (e.g. forgetting to temporarily mark a grid cell as visited before recursing, allowing the same cell to be revisited within a single path when it shouldn't be).

## How this compares to Trees / DFS

Backtracking is DFS applied to an **implicit** tree of choices/decisions, rather than an explicit tree data structure — every recursive call is exploring one more "level" of decisions, and the recursive call stack naturally handles "come back and try the next option" the same way DFS on an actual tree does (per the Trees topic's explanation of why DFS naturally goes deep-then-backtracks, via the call stack's LIFO behavior). The genuinely new idea here is the **explicit undo step**, which matters because — unlike a real tree, where each node only has one path leading to it and no shared mutable state between branches — backtracking problems are usually exploring a shared, mutable structure (an array being built up, a grid being marked) that multiple different branches of exploration will reuse, making the undo step necessary in a way tree traversal never required.

## Starter problems (medium, to warm up — this topic doesn't have many "easy" problems by nature)

1. **Subsets** (LeetCode #78) — not in Blind 75, but the cleanest possible backtracking template to start with.
2. **Permutations** (LeetCode #46) — not in Blind 75, but the standard "track used elements" backtracking variant, good to compare against Subsets.
3. **Combination Sum** (LeetCode #39) — in your Blind 75 list; a great next step once the basic template feels comfortable.

## What carries over from here

Backtracking's "try, recurse, undo" shape reappears (in disguise) as part of some Graph algorithms (like finding all paths, or certain cycle-detection approaches) and is conceptually the ancestor of many Dynamic Programming problems — in fact, a good way to *derive* a DP solution is often to first write the brute-force backtracking/recursive solution, notice it recomputes the same sub-problems repeatedly (the same "overlapping sub-problems" diagnosis the DP topic relies on), and then add memoization to avoid that repeated work (turning backtracking into "top-down DP"). This connection is worth remembering explicitly when you reach the DP topics — many DP recurrences are literally backtracking solutions with a cache added.
