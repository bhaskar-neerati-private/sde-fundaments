# Topic 8: Heap / Priority Queue

## Core concepts / data structures

### Heap

**What it is:** a tree-shaped data structure (usually visualized as a **binary tree**, but almost always implemented as a plain **array**) that keeps one specific guarantee at all times: in a **min-heap**, every parent node's value is **less than or equal to** both its children's values (so the smallest element is always at the very top/root). A **max-heap** is the mirror image — every parent is **greater than or equal to** its children, so the largest is always at the root.

**The mental model that explains why this is useful, not just what it is:** think of a heap like a company org chart where every manager must have a smaller "score" than everyone reporting to them (for a min-heap) — it doesn't matter how the two people reporting to a manager compare to *each other*, only that both are ≥ their manager. This is a much *weaker* guarantee than full sorting (siblings can be in any order relative to each other), and that weakness is precisely the source of its speed: maintaining a fully sorted structure under insertions/removals is expensive (O(n) to keep everything in order), but maintaining only "every parent ≤ its children" is cheap (O(log n), since fixing this weaker property only ever requires walking up or down *one path* of the tree, not reordering everything).

**Why use a heap instead of just sorting:** if you only ever need the current min/max, and you're repeatedly adding and removing elements, keeping the whole collection **fully sorted** at all times would cost O(n) per insertion (shifting elements, per the Arrays & Hashing topic's explanation of why array insertion is expensive). A heap achieves "always know the min/max" in **O(log n) per insertion or removal**, without maintaining full order among all the other elements — a heap is doing strictly less work than full sorting, and that's the entire reason it's faster for this specific, narrower need.

### Python's `heapq` module

Python doesn't have a dedicated heap class — the `heapq` module provides functions that operate on a plain `list`, treating it as a **min-heap**.
```python
import heapq

heap = []
heapq.heappush(heap, 5)
heapq.heappush(heap, 2)
heapq.heappush(heap, 8)
heapq.heappush(heap, 1)

print(heap[0])          # 1 - the smallest element is always at index 0
smallest = heapq.heappop(heap)  # removes and returns 1
```
**There's no built-in max-heap in Python.** The standard trick: to simulate a max-heap, **negate every value** before pushing, and negate again after popping (`heapq.heappush(heap, -x)`, then `-heapq.heappop(heap)` to get the actual value back) — since "the smallest negative number" corresponds to "the largest original number" (e.g. `-9 < -3`, and `9 > 3`). This isn't a hack specific to Python — it's a reusable trick any time you have a min-based tool but need max-based behavior.

`heapq.heapify(a_list)` converts an existing list into a valid heap **in-place**, in O(n) time (faster than pushing every element one at a time, which would cost O(n log n) — building a heap from scratch all at once can exploit shortcuts that inserting elements one-by-one cannot).

### What operations cost, and why

| Operation | Cost | Why |
|---|---|---|
| Peek at min/max (`heap[0]`) | O(1) | It's always exactly at the root/index 0 — that's the entire point of the heap invariant. |
| Push a new element | O(log n) | The new element is added at the bottom, then "bubbles up," swapping with its parent at most once per level, until the heap property is restored — at most O(log n) swaps, since a heap's height is O(log n). |
| Pop the min/max | O(log n) | Remove the root, move the last element there, then "bubble down" to restore the heap property — same O(log n) bound, for the same height-based reason. |
| Build a heap from n elements | O(n) | `heapify()` is smarter than n individual pushes — it works bottom-up, and a careful accounting shows most nodes are near the bottom (where bubbling down is cheap), so the total work sums to O(n), not O(n log n). |
| Find an arbitrary (non-min/max) element | O(n) | A heap gives **no guarantees** about where other elements are — only the root is guaranteed to be the min/max; siblings and deeper nodes have no ordering relationship to each other at all. |

## Common patterns / techniques in this topic

| Pattern | When it applies |
|---|---|
| **"Top k" / "k-th largest/smallest"** | Maintain a heap of size k (see Top K Frequent Elements from Arrays & Hashing for this exact pattern) — keep only the k best candidates seen so far, evicting the weakest whenever a better one arrives. |
| **Merging multiple sorted sequences** | Push the current front of each sequence; repeatedly pop the smallest, then push that sequence's next element (see Merge K Sorted Lists). |
| **Simulating a max-heap** | Negate values on push and pop, since Python's `heapq` only provides a min-heap natively. |
| **Two-heap technique** (a max-heap for the smaller half, a min-heap for the larger half) | Finding a running median as a stream of numbers arrives — the median is always at the boundary between the two heaps. |
| **Scheduling / greedy selection by priority** | Whenever "always process whatever's currently most urgent/smallest/largest first" describes the correct strategy for a problem — a heap is the natural implementation. |

## Key terminology

- **Priority Queue** — the general concept (always be able to retrieve the "highest priority" item efficiently); a heap is the most common way to *implement* a priority queue, so the two terms are often used interchangeably in casual conversation, though technically a heap is one specific implementation choice among several.
- **Min-heap / Max-heap** — whether the smallest or largest element sits at the root.
- **Heapify** — the O(n) operation to convert an unordered list into a valid heap in place.
- **Bubble up / sift up** — after inserting a new element (typically at the end), repeatedly swap it with its parent until the heap property is restored.
- **Bubble down / sift down** — after removing the root (typically by moving the last element there), repeatedly swap it with its smaller (or larger, for a max-heap) child until the heap property is restored.
- **Complete binary tree** — a heap's implicit tree shape always fills each level left-to-right before starting a new one, which is exactly what makes an *array* representation possible (a node at array index `i` has children at indices `2i+1` and `2i+2`, no explicit pointers needed) — this "no gaps" property is the tree-shaped analog of an array's own "no gaps" addressing requirement from the Arrays & Hashing topic.

## Common beginner mistakes

1. **Forgetting Python's `heapq` is min-heap only**, and either using it directly for a "largest first" problem (getting wrong results — the smallest, not largest, would keep surfacing) or forgetting to negate the value back after popping (getting the negated value instead of the real one).
2. **Assuming a heap is fully sorted.** Only the root is guaranteed to be the min (or max) — `heap[1]` and `heap[2]` are **not** guaranteed to be the 2nd and 3rd smallest, just "somewhere among the smaller elements, satisfying the parent-child rule relative to their own subtree." Never index into a heap expecting sorted order beyond the root.
3. **Pushing tuples with non-comparable second elements**, causing a crash on ties — as seen in Merge K Sorted Lists, if two entries could tie on their primary sort key, include an always-comparable tiebreaker (like an index) as the second tuple element, especially when the actual objects (like `ListNode`) aren't comparable by default.
4. **Using a heap when you actually just need the overall min/max once.** If you don't need repeated insert/remove-min operations, a heap is overkill — `min()`/`max()`/`sorted()` are simpler, equally correct, and equally fine for a one-time computation; a heap's advantage only shows up when the collection changes *repeatedly* over time.
5. **Not considering bucket sort as an alternative** when the range of possible values/priorities is small and bounded (see Top K Frequent Elements) — bucket sort can beat a heap's O(log n) per operation down to O(1) in the right circumstances, specifically when the "priority" values are bounded integers that can be used directly as array indices.

## How this compares to Sorting

A heap is best understood as "partial, incremental sorting" — it does *just enough* work to always know the current min/max, without fully ordering everything else, and it can do this incrementally as elements are added/removed one at a time (unlike a full sort, which is typically an all-at-once O(n log n) operation on a fixed collection). Reach for a heap specifically when you need **repeated** access to a running min/max as a collection changes over time, not when you just need a one-time sorted result — the moment you need the *full* order of everything, a heap offers no advantage over sorting directly.

## Starter problems (easy/medium, to warm up)

1. **Kth Largest Element in an Array** (LeetCode #215) — not in Blind 75, but the purest heap warm-up — maintain a min-heap of size k.
2. **Last Stone Weight** (LeetCode #1046) — not in Blind 75, but a simple, satisfying max-heap simulation exercise.

## What carries over from here

The "push candidates, always process the current best first" idea reappears directly in **Graphs** (Dijkstra's shortest-path algorithm uses a min-heap, though it's not explicitly in your Blind 75 list, it's a very common extension) and in some **Greedy** problems (like Merge K Sorted Lists' heap approach, or interval-scheduling variants) where always handling the most urgent/smallest thing next is provably optimal, in the same sense the Greedy topic requires every locally-best choice to be *justified*, not just intuitive.
