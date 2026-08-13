# Topic 6: Linked List

## Core concepts / data structures

### Linked List

**What it is:** a sequence of items where each item (a "node") holds a value **and** a reference (pointer) to the *next* node in the sequence. Unlike an array, the nodes are **not** stored next to each other in memory — each one just knows where to find the next one.

**Simple explanation:** think of a treasure hunt where each clue tells you where to find the *next* clue, rather than a numbered map where you can jump straight to clue #7. To reach clue #7, you must walk through clues 1 through 6 first — you can't skip ahead the way array indexing lets you — but adding a brand new clue between two existing ones is easy (just change what one clue points to), you don't have to renumber everything after it.

**Why this is the direct opposite trade-off from an array, not just "a different structure":** recall from the Arrays & Hashing topic that arrays are fast to index (O(1), via direct address arithmetic) but slow to insert/delete in the middle (O(n), because everything must physically shift to preserve the "no gaps" property the addressing math depends on). A linked list makes the *opposite* trade: because nodes aren't required to be contiguous in memory, inserting a node is just rewiring two pointers (O(1), once you're already at the right spot) — but there's no addressing formula that gets you to "the 7th node," because a node's memory location has no arithmetic relationship to its position in the sequence. You must walk the chain, one pointer at a time.

**Node structure in Python (typical definition you'll see in problems):**
```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next
```
A linked list is just a chain of these, starting from a `head` node:
```python
# building 1 -> 2 -> 3
third = ListNode(3)
second = ListNode(2, third)
head = ListNode(1, second)
```

### Array vs. Linked List: the fundamental trade-off

| Operation | Array | Linked List | Why |
|---|---|---|---|
| Access element at position i | O(1) — direct index | O(n) — must walk from the head | Array position maps to a memory address by arithmetic; a linked list node's position has no such formula, only a chain of pointers to follow. |
| Insert/delete at the **front** | O(n) — everything shifts | O(1) — just change a couple of pointers | Array elements must stay contiguous with no gaps; linked list nodes don't need to be adjacent in memory at all. |
| Insert/delete at a **known node** (with a reference to it) | O(n) — shifting required | O(1) — no shifting, just re-link pointers | Same reasoning as above. |
| Extra memory per element | none | one extra pointer per node | The trade-off's literal cost: flexibility in insertion is paid for with a per-node memory overhead. |

**The core trade-off, stated plainly:** arrays are fast to *read* by position but slow to insert/delete in the middle; linked lists are slow to *read* by position (no shortcuts, must walk node by node) but fast to insert/delete once you're already at the right spot. Neither is strictly better — which one to use depends on whether a problem does more reading-by-position or more inserting/deleting.

### The dummy node trick

A very common technique: when building or modifying a linked list (especially when the **head itself might change**, e.g. removing the first node), create a throwaway `dummy` node that points to the real head *before* you start, do all your work, then return `dummy.next` at the end. This avoids writing special-case code for "what if the node I need to modify is the head" — with a dummy node, the head is never special, it's just "the node after `dummy`," treated the same as every other node.
```python
dummy = ListNode(0, head)
# ... do work, possibly changing what the "real" head is ...
return dummy.next
```

## Common patterns / techniques in this topic

| Pattern | When it applies |
|---|---|
| **Iterative traversal with pointer re-linking** | Reversing a list, deleting a node, inserting a node — carefully rewire `.next` pointers one step at a time. |
| **Fast & slow pointers (Floyd's algorithm)** | Detecting a cycle, finding the middle node, finding where a cycle begins — the fast pointer moves 2 steps for every 1 step the slow pointer takes. |
| **Dummy head node** | Any operation where the head of the list might change, to avoid special-casing "is this the head?" |
| **Merging two lists** | Walk both lists simultaneously with two pointers, always attaching the smaller current node to the result, advancing that list's pointer. |
| **Recursion** | Many linked list operations (like reversal) have an elegant recursive form, since "the rest of the list" is itself a smaller linked list problem — though iterative solutions are usually preferred for O(1) space, since recursion pays for the call stack. |

## Key terminology

- **Node** — a single element, holding a value and a pointer (`.next`) to the following node.
- **Head** — the first node of the list; **Tail** — the last node (its `.next` is `None`/null).
- **Pointer / reference** — a variable holding "where to find" a node, not the node's value directly.
- **Dangling pointer / losing the reference** — a classic bug: if you change `node.next` *before* saving a reference to what it used to point to, you permanently lose access to the rest of the list beyond that point — there's no way to recover it, since nothing else in the program remembers that address.
- **Cycle** — when a node's `.next` chain eventually loops back to a node already visited, instead of ending at `None`.
- **Floyd's Tortoise and Hare** — the formal name for the fast/slow pointer technique used for cycle detection.
- **Sentinel / dummy node** — a placeholder node used to simplify edge cases (see above).

## Common beginner mistakes

1. **Losing the rest of the list by overwriting `.next` too early.** When reversing or rewiring, always save `next_node = current.next` **before** changing `current.next`, or you permanently lose access to everything after `current` — there's no "undo," since the only way to reach those nodes was through the pointer you just overwrote.
2. **Forgetting to handle an empty list (`head is None`) or a single-node list.** These edge cases often break naive implementations that implicitly assume at least 2 nodes exist (e.g. code that accesses `head.next.next` without checking `head.next` isn't `None` first).
3. **Off-by-one with fast/slow pointers.** Starting both pointers at the head vs. starting the slow pointer at the head and fast at `head.next` changes whether you land on the first or second middle node for even-length lists — be deliberate about which convention a specific problem needs, and verify with a concrete small example rather than assuming.
4. **Not updating the previous node's `.next`** after modifying/removing a node, leaving a "dangling" connection that still points to the old (now removed or moved) node — the list *looks* modified from the node you changed, but anything still holding a reference to the old previous node sees stale structure.
5. **Converting to an array first to make things "easier."** This defeats the O(1) space advantage the linked-list-native solution usually offers, and isn't what these problems are testing — the whole point is building comfort with pointer manipulation, not sidestepping it.
6. **Mutating a node's value instead of relinking pointers, when the problem expects pointer manipulation — or the reverse.** Always check what a problem specifically wants; some problems (like "delete a node given only that node," without access to the head) actually *require* a value-copying trick specifically because you can't relink around a node without knowing its predecessor.

## Starter problems (easy, to warm up)

1. **Reverse Linked List** (LeetCode #206) — the foundational linked-list manipulation exercise. Also in your Blind 75 list.
2. **Merge Two Sorted Lists** (LeetCode #21) — also in your Blind 75 list; a great two-pointer-across-two-lists warm-up.
3. **Linked List Cycle** (LeetCode #141) — also in your Blind 75 list; the canonical fast/slow pointer problem.
4. **Middle of the Linked List** (LeetCode #876) — not in Blind 75, but a very clean fast/slow pointer warm-up before tackling more complex versions of the pattern.

## How this compares to Arrays & Hashing / Two Pointers

The fast/slow pointer technique you'll use here is a direct descendant of the Two Pointers topic — same core idea (two indices/references moving through a sequence at different rates or from different starting points, each move justified by a provable guarantee), just applied to a structure where you *can't* jump to an arbitrary index the way you can with an array. Because random access is impossible, almost every linked-list technique compensates by using pointers cleverly instead of indices — this is the main mental shift from array-based topics, and it's why the "save before overwrite" discipline matters so much more here than in array problems (an array index is never destroyed by reading it; a linked list pointer *can* be destroyed by overwriting it).

## What carries over from here

Fast/slow pointers and "explicit pointer manipulation instead of relying on indices" are the two biggest takeaways, and they don't reappear in a totally new form later — but the *discipline* of carefully reasoning about references (not losing access to data, being careful about order of operations when rewiring) is exactly the same discipline you'll need for **Tree** problems next, since a tree node's `.left`/`.right` pointers are conceptually the same idea as a linked list node's `.next` pointer, just with two links instead of one.
