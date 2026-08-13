# 24. Merge K Sorted Lists

**LeetCode:** [#23 - Merge K Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/) · **Topic:** [Linked List](../topics/06-linked-list.md) · **Difficulty:** Hard

## Problem statement

Given an array of `k` linked lists, each sorted in ascending order, merge them all into one sorted linked list and return its head.

**Example:**
```
Input: lists = [[1,4,5],[1,3,4],[2,6]]
Output: [1,1,2,3,4,4,5,6]
```

## Applicable approaches

- **Brute Force — Collect all values, sort, rebuild.**
- **Better — Sequential Pairwise Merge** — repeatedly merge two lists at a time using the Merge Two Sorted Lists technique.
- **Optimal — Min-Heap (Priority Queue).** The standard, most efficient solution.
- **Optimal (alternative) — Divide and Conquer Pairwise Merge.** Same time complexity as the heap approach, different mechanism.

## Approach 1: Brute Force — Collect, Sort, Rebuild

### Intuition

Same idea as the brute force for Merge Two Sorted Lists, just across k lists: gather every value, sort, rebuild — correct, but ignores that every input list is already individually sorted, throwing that structure away just as before.

### Python code
```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

def mergeKLists(lists):
    values = []
    for lst in lists:
        node = lst
        while node:
            values.append(node.val)
            node = node.next

    values.sort()

    dummy = ListNode()
    current = dummy
    for val in values:
        current.next = ListNode(val)
        current = current.next

    return dummy.next
```

### Time & space complexity

- **Time: O(N log N)** where N = total number of nodes across all lists (dominated by the sort).
- **Space: O(N)**.

---

## Approach 2: Better — Sequential Pairwise Merge

### Intuition

We already know how to merge **two** sorted lists efficiently and without wasting the "already sorted" structure (Merge Two Sorted Lists). The most direct way to extend that to k lists: repeatedly merge the running result with the next list, one at a time — `merge(merge(merge(list1, list2), list3), list4)...`. This is correct, but notice the running "result" list keeps growing as more lists get folded in, so later merges have to process an ever-larger accumulated result each time — that growth is the specific inefficiency the later approaches target.

### Python code
```python
def mergeTwoLists(l1, l2):
    dummy = ListNode()
    current = dummy
    while l1 and l2:
        if l1.val <= l2.val:
            current.next, l1 = l1, l1.next
        else:
            current.next, l2 = l2, l2.next
        current = current.next
    current.next = l1 if l1 else l2
    return dummy.next

def mergeKLists(lists):
    if not lists:
        return None

    result = lists[0]
    for i in range(1, len(lists)):
        result = mergeTwoLists(result, lists[i])
    return result
```

### Time & space complexity

- **Time: O(k · N)** in the worst case — merging is done k-1 times, and each merge can take up to O(N) time as the running result grows to include most of the nodes. Concretely: the *i*-th merge processes up to roughly `i * (N/k)` nodes (the accumulated result so far plus the next list), and summing that across all k-1 merges gives O(k·N) total, not O(N) — this is the direct consequence of the "growing accumulated result" issue identified above.
- **Space: O(1)** extra (reusing nodes), not counting the input.

*(This is correct but not the most efficient — the "better" options below improve on this by avoiding the growing-result problem entirely.)*

---

## Approach 3: Optimal — Min-Heap (Priority Queue)

### Intuition

The bottleneck in sequential merging is specifically that the running result keeps growing, making each subsequent merge more expensive than the last. Instead, think of the problem as merging k lists **all at once**, rather than two-at-a-time: at every step, we need "the smallest value among all k lists' *current front* nodes" — this is exactly the "top k" style question the Heap/Priority Queue topic is built for. A **min-heap** answers exactly this in O(log k) per query: push all k lists' front nodes in, repeatedly pop the smallest, attach it to the result, and push that list's *next* node back onto the heap. Because the heap never holds more than k elements at once (one per still-active list), every operation costs O(log k), not O(log N) or worse.

### Algorithm

1. Create a min-heap. Push each list's head node onto it (using `(value, unique_tiebreaker, node)` tuples — see the note below on why a tiebreaker is needed).
2. While the heap isn't empty: pop the smallest entry, attach its node to the result, and if that node has a `.next`, push `.next` onto the heap.
3. Return the merged result.

### Python code
```python
import heapq

def mergeKLists(lists):
    heap = []
    for i, node in enumerate(lists):
        if node:
            heapq.heappush(heap, (node.val, i, node))

    dummy = ListNode()
    current = dummy

    while heap:
        val, i, node = heapq.heappop(heap)
        current.next = node
        current = current.next
        if node.next:
            heapq.heappush(heap, (node.next.val, i, node.next))

    return dummy.next
```

### Line-by-line explanation

- `for i, node in enumerate(lists): if node: heapq.heappush(heap, (node.val, i, node))` — seed the heap with the first node of every non-empty list. **Why the tiebreaker `i` is necessary, not just stylistic:** Python's heap compares tuples element by element; if two nodes have the *same* `.val`, Python would try to compare the third element (the `ListNode` objects themselves) to break the tie, and `ListNode` objects have no default ordering comparison defined — this would crash with a `TypeError`. Including `i` (the list's index, always unique and always comparable) as a middle tiebreaker element guarantees the comparison never needs to fall through to the unorderable node objects.
- `val, i, node = heapq.heappop(heap)` — always pops the globally smallest value currently at the front of any list, in O(log k) time (k = current heap size, bounded by the number of input lists, never larger).
- `current.next = node; current = current.next` — attach it to the growing result (reusing the existing node, same space-saving approach as Merge Two Sorted Lists).
- `if node.next: heapq.heappush(heap, (node.next.val, i, node.next))` — that list still has more nodes, so push its new front node back onto the heap for future consideration — this is what keeps the heap always representing "the current front of every still-active list," never more, never less.

### Dry run

`lists = [[1,4,5],[1,3,4],[2,6]]`

Initial heap (as `(val, list_index, node)`): entries for `1` (list 0), `1` (list 1), `2` (list 2). Heap min = smallest by `(val, index)` → `(1, 0, node)` wins the tie over `(1, 1, node)` since index 0 < 1.

| pop | attach | push next |
|---|---|---|
| (1, 0, val=1) | result: 1 | push (4, 0, next node of list 0) |
| (1, 1, val=1) | result: 1,1 | push (3, 1, next node of list 1) |
| (2, 2, val=2) | result: 1,1,2 | push (6, 2, next node of list 2) |
| (3, 1, val=3) | result: 1,1,2,3 | push (4, 1, next node of list 1) |
| (4, 0, val=4) or (4, 1, val=4) - tie, index 0 wins | result: 1,1,2,3,4 | push (5, 0, next of list 0) |
| (4, 1, val=4) | result: 1,1,2,3,4,4 | list 1 exhausted, nothing pushed |
| (5, 0, val=5) | result: 1,1,2,3,4,4,5 | list 0 exhausted, nothing pushed |
| (6, 2, val=6) | result: 1,1,2,3,4,4,5,6 | list 2 exhausted, nothing pushed |

Heap empty, done. Final: `[1,1,2,3,4,4,5,6]` ✅. Notice the heap's size never exceeded 3 (= k) at any point — this bounded size is exactly why every operation stayed at O(log k), never growing toward O(log N).

### Time & space complexity

- **Time: O(N log k)** where N = total nodes, k = number of lists — each of the N nodes is pushed and popped from a heap of size at most k, each operation costing O(log k). This is strictly better than Approach 2's O(k·N) whenever k > 1, since the log factor is much smaller than a linear factor of k for any reasonably large k.
- **Space: O(k)** for the heap (at most one entry per list at any time).

---

## Approach 4: Optimal (alternative) — Divide and Conquer Pairwise Merge

### Intuition

Instead of merging lists one at a time into an ever-growing result (Approach 2's specific weakness), pair them up and merge **in parallel rounds**, like a tournament bracket: merge list 0 with list 1, list 2 with list 3, etc. (round 1), then merge *those* results pairwise (round 2), and so on, halving the number of lists each round until only one remains. This avoids the "growing accumulated result" problem entirely, since no single list ever grows to include more than roughly `N/2` nodes before the very last round.

### Python code
```python
def mergeKLists(lists):
    if not lists:
        return None

    while len(lists) > 1:
        merged_lists = []
        for i in range(0, len(lists), 2):
            l1 = lists[i]
            l2 = lists[i + 1] if i + 1 < len(lists) else None
            merged_lists.append(mergeTwoLists(l1, l2))
        lists = merged_lists

    return lists[0]
```

### Why this is faster than sequential merging

With sequential merging, the *last* merge processes almost all N nodes, and this expensive-merge pattern effectively happens k-1 times, giving O(k·N). With pairwise/divide-and-conquer merging, **every node is only ever involved in O(log k) merge operations total** — once per "round," and there are only `log2(k)` rounds before we're down to 1 list — giving O(N log k), the same complexity as the heap approach, just achieved through a different mechanism (batching merges in parallel rounds instead of using a priority queue to interleave all k lists simultaneously).

### Time & space complexity

- **Time: O(N log k)** — `log k` rounds, each round doing O(N) total work across all its merges (since every node is touched exactly once per round, regardless of how many lists exist in that round).
- **Space: O(1)** extra beyond the temporary `merged_lists` array of list heads (small, O(k)), not counting the input/output nodes themselves.

---

## Common mistakes & misconceptions

1. **Pushing raw `(value, node)` tuples onto the heap without a tiebreaker.** As explained above, this crashes with a `TypeError` the moment two nodes with equal values need to be compared, since `ListNode` objects aren't inherently orderable — the middle tiebreaker index isn't optional defensive code, it's required for correctness on any input with duplicate values across lists.
2. **Believing sequential pairwise merging (Approach 2) is "good enough" because it reuses the correct Merge Two Sorted Lists logic.** It's correct, but its O(k·N) complexity is a real, measurable regression compared to O(N log k) for large k — reusing a correct sub-routine doesn't guarantee the overall composition is efficient.
3. **Forgetting to handle an odd number of lists in the divide-and-conquer round.** The `l2 = lists[i + 1] if i + 1 < len(lists) else None` check handles a leftover unpaired list by merging it with `None` (which `mergeTwoLists` already handles correctly, per its own base cases) — omitting this check causes an index-out-of-range error on any round with an odd list count.
4. **Assuming the heap-based approach's O(k) space is worse than it sounds.** It's easy to mentally round this up to "another O(N)-ish cost," but k (number of lists) is typically much smaller than N (total nodes) — this is a genuinely small, bounded overhead, not a hidden large cost.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Collect + Sort + Rebuild | O(N log N) | O(N) | Ignores that inputs are already sorted. |
| Sequential Pairwise Merge | O(k·N) | O(1) | Correct, reuses known technique, but slow when k is large due to the growing-result problem. |
| Min-Heap | O(N log k) | O(k) | The standard, most commonly expected optimal solution. |
| Divide and Conquer Merge | O(N log k) | O(1) extra | Same complexity as the heap, no heap needed — avoids the growing-result problem via parallel rounds instead. |

**Key takeaway:** "merge k sorted things" generalizes the two-list merge you already know, and the key realization is that merging **one pair at a time sequentially** wastes work by repeatedly re-processing the same growing result — either a heap (to always know the smallest of k current fronts, bounding per-operation cost by k rather than by the growing result size) or a tournament-bracket pairwise merge (to bound how many times each node is touched, by round count rather than list count) brings the k-lists case down to the same O(N log k) complexity, both meaningfully better than the naive O(k·N) sequential approach.
