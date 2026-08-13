# Topic 1: Arrays & Hashing

## Why this topic comes first

Almost every other topic in this list (Two Pointers, Sliding Window, Trees, Graphs, DP) eventually uses an array to store data and a hash map/set to look things up quickly. If this topic isn't solid, everything after it is harder than it needs to be. Take your time here.

## Core concepts / data structures

### Array

**What it is:** a block of memory that stores items one after another, each reachable instantly by its index (position number, starting at 0).

**The mental model that actually matters:** an array isn't just "a list of things" — it's a *contract with memory*. When you create `arr = [10, 20, 30, 40]`, the computer reserves one continuous strip of memory, and it knows the address of the very first element. Because every element is the same fixed size, the address of element `i` is just `start_address + i * element_size` — pure arithmetic, no searching. This is *why* `arr[5]` is O(1): the computer isn't "looking for" position 5, it's computing where position 5 lives and jumping straight there, the same way you don't search for house number 42 on a street — you walk to where 42 must be, because house numbering is arithmetic on the street's starting point.

This mental model is what explains every other array fact below — you don't need to memorize "insert at front is O(n)," you can *derive* it: if the array is a contiguous strip and you want to add something at the front, everything else has to physically shift down to make room, because the arithmetic (`start_address + i * size`) only works if there are no gaps.

**What's fast and what's slow, and why:**
- Read/write by index (`arr[i]`, `arr[i] = x`): **O(1)** — direct address computation, no traversal.
- Append to the end (`arr.append(x)`): **O(1) amortized** — Python's list actually over-allocates extra capacity when it grows, so most appends just write into already-reserved space; only occasionally (when that spare capacity runs out) does it need to allocate a bigger block and copy everything over, an O(n) event. Spread across many appends, the *average* cost per append stays O(1) — this is what "amortized" means, and it's not the same claim as "always O(1)."
- Insert/delete at the front or middle (`arr.insert(0, x)`, `arr.pop(0)`): **O(n)** — every item after that position has to physically shift over by one slot, like everyone in a line taking one step forward to let a new person in at the front.
- Search for a value you don't know the index of (`x in arr`): **O(n)** — there is no shortcut; without an index, you're checking lockers one by one because arithmetic can't tell you *which* index holds a given value, only what value lives at a given index.

```python
arr = [10, 20, 30, 40]
print(arr[2])       # 30 - instant, direct index access
arr.append(50)      # [10, 20, 30, 40, 50] - fast (amortized)
arr.insert(0, 5)    # [5, 10, 20, 30, 40, 50] - slow, everything shifts right
```

### Hash Map (Python `dict`) and Hash Set (Python `set`)

**What it is:** a data structure that stores key→value pairs (map) or just unique keys (set), and can tell you "is this key in here?" or "what's the value for this key?" in O(1) time on average — regardless of how many items are stored.

**How it actually works (the mental model, not just the claim):** a hash map keeps an internal array, exactly like the array above. The difference is *how it decides where to put things*. When you insert a key, it runs the key through a **hash function** — a formula that deterministically turns *any* value (a string, a tuple, whatever) into a number — and uses that number (modulo the array's size) as the index to store it at. Looking up a key later runs the *same* hash function again, computes the *same* index, and jumps straight there — no searching, for the exact same reason array indexing is O(1): it's arithmetic, not search. The insight that makes hashing powerful: **it converts "does this value exist?" from a search problem into an addressing problem.**

**Simple analogy:** imagine a library where, instead of alphabetizing books and scanning shelves, every book's ID number runs through a formula that says "this book lives on shelf 47, slot 3" — you compute the answer, you don't search for it. That's a hash map.

**Why this matters for problem-solving:** any time you catch yourself thinking "I need to check if I've seen this value before" or "I need to count how many times something appears," a hash map/set is almost always the answer, because it replaces an O(n) *search* with an O(1) *computation* — and this is the single most common way an O(n²) brute-force solution becomes an O(n) one in this entire curriculum.

```python
seen = set()
seen.add(5)
seen.add(10)
print(5 in seen)     # True - O(1) lookup, not a loop through the set

counts = {}
counts["apple"] = counts.get("apple", 0) + 1   # counting pattern
```

**Collisions — and why "O(1) on average" is not the same as "always O(1)":** two different keys can hash to the same slot (a "collision") — the hash function's output space is much smaller than the space of all possible inputs, so by the pigeonhole principle, collisions are mathematically inevitable, not a bug. Python's dict/set handle this internally (open addressing — probing for the next open slot), so you never implement collision handling yourself. But it's *why* hash operations are described as **O(1) average-case**, not worst-case: in a pathological scenario where many keys collide, a lookup could degrade toward O(n) (having to check many colliding entries). In practice this is rare enough (and Python's hash function is designed to make it rare) that you should treat hash operations as O(1) by default, but it's worth knowing this isn't a mathematical guarantee the way array indexing is.

**What can be a key — and why:** only **hashable** (effectively: immutable) types — numbers, strings, tuples of hashable things. You cannot use a `list` or another `dict` as a dictionary key or set element. The reason isn't arbitrary: the hash function computes a slot *once*, at insertion time. If the key's value could change afterward (like a list you later mutate), its hash would change too, but the item would still be sitting in the *old* slot — the map would become permanently broken, unable to find its own contents. Immutability is what guarantees a key's slot stays valid for its entire lifetime in the structure.

## Common patterns / techniques in this topic

| Pattern | When it applies | Why it works |
|---|---|---|
| **Frequency counting** (`dict` mapping value → count) | "How many times does X appear?", "find duplicates," "find the most/least common item." | Reduces "how many times have I seen X" from an O(n) rescan to an O(1) lookup+increment. |
| **Seen-before check** (`set` of values you've already visited) | "Have I encountered this value already?" | Turns an O(n²) nested-loop check (compare every pair) into O(n) (check each item against a running memory of what's been seen). |
| **Complement lookup** (store what you *need* to find, not just what you've seen) | Two Sum-style problems: for each number, check if `target - number` is already in the map. | Reframes "find a pair" from "check every pair" (O(n²)) to "for each item, ask one O(1) question" (O(n)). |
| **Sorting as a first move** | When order doesn't matter for the final answer but makes comparison easy (e.g. two anagrams sorted become identical strings). | Costs O(n log n) once, but converts a comparison that would otherwise need O(n) work (comparing letter-by-letter, accounting for reordering) into O(1) (string equality). |
| **Prefix / suffix products or sums** | When you need "everything before index i" and "everything after index i" without recomputing from scratch each time. | Precomputing running totals turns what looks like an O(n) recomputation *per index* (O(n²) total) into two O(n) passes. |
| **Grouping by a computed key** (`dict` mapping key → list of items) | "Group these items by some property" (e.g. group words that are anagrams of each other). | The key acts as an automatic bucket-sorter: anything that should be grouped together is *engineered* to produce the same key, so the hash map does the grouping as a side effect of insertion. |

## Key terminology

- **Hash function** — a formula that converts a value (string, number, etc.) into an integer, used to decide where to store it. Deterministic: the same input always produces the same output.
- **Collision** — when two different keys hash to the same storage slot; handled internally by Python, not something you implement.
- **Amortized O(1)** — "O(1) on average across many operations," even if one specific operation is occasionally slower (e.g. `list.append` is amortized O(1) because Python resizes the underlying array in big jumps, not every single append — most appends are cheap, occasional ones are expensive, and it averages out).
- **In-place** — modifying the existing array/structure directly instead of creating a new one, to save space (O(1) extra space instead of O(n)).
- **Hashable** — a type that can be used as a dict key or set element (must be immutable: int, str, tuple, frozenset — not list or dict).
- **Two-pass vs. one-pass** — whether you loop through the data twice (once to gather info, once to use it) or manage to do it in a single loop. One-pass is usually the "optimal" version once you're comfortable with the pattern, but it's worth explicitly checking whether a one-pass version is *actually* correct, not just shorter — see the misconceptions below.

## Common beginner mistakes (and *why* each one is wrong, not just that it is)

1. **Using `x in some_list` inside a loop.** `in` on a `list` is O(n) — it linearly checks every element, because a plain list has no addressing shortcut for "does this value exist" (only for "what's at this index"). Doing this inside another loop silently creates an O(n²) algorithm that *looks* like it should be O(n) at a glance. Use a `set` instead — `in` on a `set` is O(1) because it's a hash lookup, not a scan.
2. **Forgetting that sorting changes order.** If a problem needs you to return answers in original input order, sorting the actual input array destroys the information needed to answer the question. Sort a copy, or sort `(value, original_index)` pairs so the original position travels with the value.
3. **Trying to use a mutable type as a dict key or set element.** Python raises `TypeError: unhashable type` — this isn't a workaround-able restriction, it's protecting you from the "key's slot becomes permanently wrong" bug described above. Convert to a `tuple` first if you need a sequence as a key.
4. **Off-by-one errors with prefix sums/products.** Specifically: forgetting whether `prefix[i]` means "sum up to and including index i" or "sum up to but *not* including index i" — these are genuinely different definitions that lead to different index arithmetic everywhere they're used. Pick one convention, write it down as a comment while you code, and never mix the two within one solution.
5. **Assuming a "seen it before" hash-set check is safe without thinking about *what* needs to be "seen."** A common subtle bug: checking `if num in seen` and *then* adding `num` to `seen` — versus adding first and checking second — can silently allow (or silently prevent) a value from matching *itself*. In Two Sum, checking the complement *before* inserting the current number is what correctly handles `nums = [3,3], target = 6` without letting a number match itself; get this ordering backward and the same-index-reuse bug creeps in.
6. **Modifying a list while iterating over it.** Removing items from a list during a `for item in list` loop skips elements, because list iteration is index-based internally, and removing an element shifts every later element's index down by one — the loop's internal position counter doesn't know that happened, so it skips whatever slid into the position it just visited. Iterate over a copy (`for item in list[:]`), or build a new list instead.

## Starter problems (easy, do these before Blind 75's Arrays & Hashing set if you want extra warm-up)

1. **Two Sum** (LeetCode #1) — the canonical hash-map lookup problem. (Also the first problem in your Blind 75 list.)
2. **Contains Duplicate** (LeetCode #217) — seen-before check with a set. (Also in your Blind 75 list.)
3. **Valid Anagram** (LeetCode #242) — frequency counting or sorting. (Also in your Blind 75 list.)
4. **Single Number** (LeetCode #136) — not in Blind 75, but a great warm-up: find the one number that doesn't repeat, using either a hash set or the XOR bit trick (covered later in Bit Manipulation).
5. **Majority Element** (LeetCode #169) — not in Blind 75, but reinforces frequency counting with a twist (Boyer-Moore voting is the O(1)-space optimal answer, worth knowing about even if you solve it with a hash map first).

## What carries over from here

Everything in this topic — hash maps for O(1) lookup, frequency counting, the "have I seen this before" pattern — shows up constantly in later topics: Two Pointers and Sliding Window both often pair with a hash map/set to track what's "in the current window," Graphs use hash sets for "visited" tracking, and Backtracking uses them to avoid revisiting states. Arrays & Hashing isn't just topic 1 — it's a toolbox you keep reaching into for the rest of the list. Every time a later topic's approach "uses a hash map," it's leaning on the exact addressing-not-searching mental model built here.
