# 20. Merge Two Sorted Lists

**LeetCode:** [#21 - Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/) · **Topic:** [Linked List](../topics/06-linked-list.md) · **Difficulty:** Easy

## Problem statement

Given the heads of two sorted linked lists `list1` and `list2`, merge them into a single sorted linked list by splicing their nodes together, and return the head of the merged list.

**Example:**
```
Input: list1 = 1->2->4, list2 = 1->3->4
Output: 1->1->2->3->4->4
```

## Applicable approaches

- **Brute Force — Collect all values, sort, rebuild.**
- **Optimal — Two Pointers (one per list) + Dummy Head.** The standard, expected O(n+m) solution.
- **Optimal (alternative) — Recursive.**

## Approach 1: Brute Force — Collect, Sort, Rebuild

### Intuition

Walk both lists, dump every value into one array, sort that array, then build a brand-new linked list from the sorted values. This is correct, but it completely ignores the fact that both inputs are **already sorted** — sorting them again from scratch is redundant work on data that's already almost entirely in the right order (in fact, fully in order within each list; only the *interleaving* between the two lists needs to be figured out).

### Python code
```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

def mergeTwoLists(list1, list2):
    values = []
    for node_list in (list1, list2):
        node = node_list
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

- **Time: O((n+m) log(n+m))** — dominated by the sort, even though both input lists were already individually sorted; the sort algorithm has no way to know that and take advantage of it.
- **Space: O(n+m)** for the values array and the newly created nodes — a second real cost: this approach builds an entirely new list rather than reusing the existing nodes.

---

## Approach 2: Optimal — Two Pointers + Dummy Head

### Intuition

Since **both lists are already sorted**, we don't need to sort anything — the only genuine question is how to *interleave* two already-sorted sequences into one, and that has a direct, greedy answer: repeatedly pick whichever of the two lists' *current* node has the smaller value, attach it to our result, and advance that list's pointer. This is provably correct because, at any point, the smallest remaining value across *both* lists must be at the front of one of them (since each list is individually sorted, nothing smaller could be hiding further inside either list). This is exactly the "merge" step used in merge sort, applied directly to linked-list nodes (via re-linking pointers) instead of array values. A dummy head node (from the topic overview) lets us build the result without special-casing "what's the very first node of the merged list."

### Algorithm

1. Create a `dummy` node, and a `current` pointer starting at `dummy` (this will always point to the *last* node attached so far).
2. While both `list1` and `list2` still have nodes:
   - Compare `list1.val` and `list2.val`. Attach whichever is smaller to `current.next`, and advance *that* list's pointer.
   - Advance `current` to the node just attached.
3. Once one list is exhausted, attach whatever remains of the *other* list directly (it's already sorted, so no more comparisons are needed — just splice it on wholesale).
4. Return `dummy.next` (the real head of the merged list).

### Python code
```python
def mergeTwoLists(list1, list2):
    dummy = ListNode()
    current = dummy

    while list1 and list2:
        if list1.val <= list2.val:
            current.next = list1
            list1 = list1.next
        else:
            current.next = list2
            list2 = list2.next
        current = current.next

    current.next = list1 if list1 else list2

    return dummy.next
```

### Line-by-line explanation

- `dummy = ListNode()` — a throwaway placeholder node; `dummy.next` will end up pointing to the true head of the merged list, whichever node that ends up being, without needing to special-case "who is the first node I attach."
- `current = dummy` — tracks the last node attached to the result so far.
- `while list1 and list2:` — keep merging as long as both lists still have unprocessed nodes.
- `if list1.val <= list2.val:` — compare the two lists' current front values; this is the "smallest remaining value must be at one of the two fronts" argument, made concrete.
- `current.next = list1` — attach `list1`'s current node to the result. **Note we're reusing the existing node, not creating a new one** — this is a genuine advantage over Approach 1, which allocates entirely fresh nodes.
- `list1 = list1.next` — advance `list1`'s pointer past the node we just used.
- (mirror logic in the `else` branch for `list2`.)
- `current = current.next` — move the result's tail pointer forward to the node we just attached.
- `current.next = list1 if list1 else list2` — once the loop ends, exactly one of the two lists is now `None` and the other still has remaining nodes — since that remainder is already sorted internally, we can attach it wholesale in one O(1) step, no further per-node comparison needed (this is what avoids ever needing to "finish off" the merge with more comparisons than necessary).
- `return dummy.next` — skip past the dummy placeholder, returning the actual first real node.

### Dry run

`list1 = 1->2->4`, `list2 = 1->3->4`

| current (before) | list1.val | list2.val | compare | attach | new current | list1 after | list2 after |
|---|---|---|---|---|---|---|---|
| dummy | 1 | 1 | 1<=1 (ties go to list1) | current.next = list1(1) | node(1 from list1) | 2->4 | 1->3->4 |
| 1(from list1) | 2 | 1 | 2>1 | current.next = list2(1) | node(1 from list2) | 2->4 | 3->4 |
| 1(from list2) | 2 | 3 | 2<=3 | current.next = list1(2) | node(2) | 4 | 3->4 |
| 2 | 4 | 3 | 4>3 | current.next = list2(3) | node(3) | 4 | 4 |
| 3 | 4 | 4 | 4<=4 | current.next = list1(4) | node(4 from list1) | None | 4 |

Now `list1` is `None`, loop ends. `current.next = list1 if list1 else list2` → `list1` is falsy (`None`) → `current.next = list2` (the remaining `4` node from list2).

Final merged list, following from `dummy.next`: `1(list1) -> 1(list2) -> 2 -> 3 -> 4(list1) -> 4(list2) -> None` ✅ matches expected `1->1->2->3->4->4`.

### Time & space complexity

- **Time: O(n + m)** where n, m are the lengths of the two lists — each node from each list is visited exactly once, since the comparison-and-attach step consumes exactly one node per iteration.
- **Space: O(1)** extra — we reuse the existing nodes, only creating one throwaway `dummy` node; no new list nodes are allocated. This is the real, measurable advantage over Approach 1's O(n+m) space.

---

## Approach 3: Optimal (alternative) — Recursive

### Intuition

"Merge two sorted lists" can be phrased recursively, in a way that mirrors the iterative logic exactly: compare the two current heads, keep the smaller one as the result's head, and set its `.next` to be "the merge of the rest of its own list with the entirety of the other list" — a smaller version of the exact same problem.

### Python code
```python
def mergeTwoLists(list1, list2):
    if not list1:
        return list2
    if not list2:
        return list1

    if list1.val <= list2.val:
        list1.next = mergeTwoLists(list1.next, list2)
        return list1
    else:
        list2.next = mergeTwoLists(list1, list2.next)
        return list2
```

### Line-by-line explanation

- `if not list1: return list2` / `if not list2: return list1` — base cases: if one list has run out, the answer is simply whatever remains of the other list (already sorted, needs no further processing).
- `if list1.val <= list2.val:` — the smaller current value should be the head of this portion of the merged result.
- `list1.next = mergeTwoLists(list1.next, list2)` — recursively merge everything *after* `list1`'s current node with the *entirety* of `list2` (since `list2`'s current node hasn't been used yet), and attach that result as `list1`'s next pointer.
- `return list1` — `list1`'s current node is confirmed as the head of this portion.
- The `else` branch mirrors this when `list2`'s value is smaller.

### Time & space complexity

- **Time: O(n + m)** — same total work as the iterative version, one comparison-and-recurse per node consumed.
- **Space: O(n + m)** — the recursion depth equals the total number of nodes processed before the deepest base case is hit, using call stack space (unlike the O(1) iterative version).

---

## Common mistakes & misconceptions

1. **Re-sorting instead of merging.** As shown in Approach 1, this is correct but throws away the "already sorted" structure the problem hands you for free — always check whether inputs are pre-sorted before defaulting to a general-purpose sort.
2. **Forgetting the final `current.next = list1 if list1 else list2` step.** Without it, whichever list still has remaining nodes when the loop ends never gets attached — the merged result would be silently truncated, missing however many nodes were left in the longer list.
3. **Creating new nodes instead of reusing existing ones.** Some implementations write `current.next = ListNode(list1.val)` instead of `current.next = list1` — this works and produces correct *values*, but needlessly allocates new memory and discards the actual original node objects, losing the O(1)-extra-space property.
4. **Getting the tie-breaking direction backward or being inconsistent about it.** `list1.val <= list2.val` (ties go to list1) vs. `list1.val < list2.val` (ties go to list2) — both are valid choices for correctness (either order of equal values is a valid sorted output), but be consistent within one solution, since flip-flopping can introduce subtle logic errors elsewhere in a larger program relying on stable ordering.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Collect + Sort + Rebuild | O((n+m) log(n+m)) | O(n+m) | Ignores that inputs are already sorted; allocates new nodes. |
| Two Pointers + Dummy Head | O(n+m) | O(1) | The standard, best-in-class solution. |
| Recursive | O(n+m) | O(n+m) (call stack) | Elegant, but costs stack space. |

**Key takeaway:** when merging two *already sorted* sequences, never re-sort — use the "always take the smaller of the two current fronts" merge step directly, which is provably correct because the global minimum of two sorted sequences must be at one of their two fronts. This exact merge step is also half of how Merge Sort works, and reappears again in Merge K Sorted Lists later in this list.
