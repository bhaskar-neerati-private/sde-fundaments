# 48. Alien Dictionary

**LeetCode:** [#269 - Alien Dictionary](https://leetcode.com/problems/alien-dictionary/) (Premium - video/article write-ups are free) · **Topic:** [Advanced Graphs](../topics/12-advanced-graphs.md) · **Difficulty:** Hard

## Problem statement

Given a list of `words` sorted according to the rules of an **unknown alien alphabet** (using some subset of lowercase English letters), determine a valid ordering of the letters in that alien alphabet. If no valid ordering exists (the input is contradictory), return an empty string. If multiple valid orderings exist, any one is acceptable.

**Example:**
```
Input: words = ["wrt","wrf","er","ett","rftt"]
Output: "wertf"
```

## Applicable approaches

- **Post-order DFS Topological Sort.**
- **Kahn's Algorithm (BFS, in-degree based).**

Both approaches require the same crucial first step: **extracting ordering constraints (edges) from the word list**, which is the genuinely new skill this problem tests (see the Advanced Graphs topic overview).

## Step 0 (shared by both approaches): Extract Edges from the Word List

### Intuition
Compare each pair of **adjacent** words in the list. Find the **first** letter position where they differ - that tells us one letter must come before the other in the alien alphabet (a directed edge). Any letters *after* that first difference tell us nothing reliable, so we stop comparing once a difference is found. There's one special invalid case: if word A is a prefix of word B (e.g. `"abc"` and `"ab"`), but A appears **after** B in the list - that's never valid in any real ordering (a genuine prefix must always sort before the longer word it's a prefix of), so we detect this and immediately return no valid ordering.

### Python code (edge extraction)
```python
def buildGraph(words):
    graph = {ch: set() for word in words for ch in word}

    for w1, w2 in zip(words, words[1:]):
        min_len = min(len(w1), len(w2))
        found_difference = False

        for i in range(min_len):
            if w1[i] != w2[i]:
                graph[w1[i]].add(w2[i])  # w1[i] must come before w2[i]
                found_difference = True
                break

        if not found_difference and len(w1) > len(w2):
            return None  # invalid: w1 is a longer "prefix violation" of w2

    return graph
```

### Line-by-line explanation
- `graph = {ch: set() for word in words for ch in word}` - initialize every distinct letter that appears **anywhere** in the input as a node in the graph, with no edges yet - this ensures letters that never end up needing an edge (e.g. never differ first in any comparison) still appear in the final ordering.
- `for w1, w2 in zip(words, words[1:])` - a compact way to iterate over every **adjacent pair** of words (`zip` pairs up `words[0]` with `words[1]`, `words[1]` with `words[2]`, etc.).
- `min_len = min(len(w1), len(w2))` - only compare up to the shorter word's length (can't compare positions that don't exist in the shorter word).
- `for i in range(min_len): if w1[i] != w2[i]: ... break` - find the **first** differing position; the moment we find it, record the edge and stop - anything after this position is irrelevant to the ordering (it could be anything, in either word, without violating the sorted-order assumption).
- `graph[w1[i]].add(w2[i])` - record that `w1[i]` must come before `w2[i]` in the alien alphabet.
- `if not found_difference and len(w1) > len(w2): return None` - **the special invalid case**: if we scanned through the *entire* shorter length without finding any difference (meaning one word is exactly a prefix of the other) **and** the first (supposedly earlier-sorted) word `w1` is actually the *longer* one, that's a direct contradiction - no valid alien dictionary could ever have a longer word sort before its own prefix.

## Approach 1: Post-order DFS Topological Sort

### Intuition
This uses the same "post-order DFS gives a reverse topological order" idea: fully explore everything reachable from a node **first**, and only add the node itself to the result **after** all of its dependents have been added - then reverse the whole result at the end. We also need cycle detection (using the same "visiting vs. visited" distinction as Course Schedule), since a cycle here means a contradictory ordering.

### Python code
```python
def alienOrder(words):
    graph = buildGraph(words)
    if graph is None:
        return ""

    visited = {}  # letter -> False (currently visiting) or True (fully done)
    result = []

    def dfs(letter):
        if letter in visited:
            return visited[letter]  # True if safely already processed, False if mid-exploration (cycle!)

        visited[letter] = False  # mark as "currently visiting"
        for neighbor in graph[letter]:
            if not dfs(neighbor):
                return False  # cycle detected somewhere downstream

        visited[letter] = True  # fully done, safe
        result.append(letter)
        return True

    for letter in graph:
        if not dfs(letter):
            return ""

    result.reverse()
    return "".join(result)
```

### Line-by-line explanation
- `graph = buildGraph(words); if graph is None: return ""` - run the shared edge-extraction step first; an immediate prefix-violation contradiction means no valid order exists.
- `visited = {}` - maps a letter to `False` (currently being explored, still "in progress" - equivalent to Course Schedule's `visiting` set) or `True` (fully resolved, safe).
- `dfs(letter)`: `if letter in visited: return visited[letter]` - if already processed, return whether it was found safe (`True`) - and crucially, if it's still marked `False` (meaning we're still *actively* exploring it further up the current call chain), that itself signals a cycle.
- `visited[letter] = False` - mark as in-progress before recursing.
- `for neighbor in graph[letter]: if not dfs(neighbor): return False` - recursively verify every letter that must come *after* this one; propagate a cycle detection immediately.
- `visited[letter] = True; result.append(letter)` - **only after all of this letter's dependents are confirmed safe**, mark it done and add it to the result - this is the "post-order" part: a letter is added only once everything that must come after it has already been fully handled.
- `result.reverse()` - since we appended letters in the order they were *finished* (dependents finished before their prerequisites, due to the post-order timing), the result is backward - reverse it to get the correct order (earliest-required letters first).

### Dry run
`words = ["wrt","wrf","er","ett","rftt"]`

Building the graph (comparing adjacent pairs): `"wrt"` vs `"wrf"`: first diff at index 2, `t` vs `f` → edge `t -> f`. `"wrf"` vs `"er"`: first diff at index 0, `w` vs `e` → edge `w -> e`. `"er"` vs `"ett"`: first diff at index 1, `r` vs `t` → edge `r -> t`. `"ett"` vs `"rftt"`: first diff at index 0, `e` vs `r` → edge `e -> r`.

`graph = {w:{e}, r:{t}, t:{f}, f:{}, e:{r}}` (letters present: w,r,t,f,e)

Running DFS from each unvisited letter (order of the outer loop over `graph`'s keys can vary, but let's trace one consistent path): `dfs(w)`: mark False. neighbor `e`: `dfs(e)`: mark False. neighbor `r`: `dfs(r)`: mark False. neighbor `t`: `dfs(t)`: mark False. neighbor `f`: `dfs(f)`: mark False, no neighbors, mark True, `result=[f]`. Back in `dfs(t)`: mark True, `result=[f,t]`. Back in `dfs(r)`: mark True, `result=[f,t,r]`. Back in `dfs(e)`: mark True, `result=[f,t,r,e]`. Back in `dfs(w)`: mark True, `result=[f,t,r,e,w]`.

`result.reverse()` → `[w,e,r,t,f]` → `"wertf"` ✅ matches expected output.

### Time & space complexity
- **Time: O(C)** where C = total number of characters across all words (dominates the edge-building step); the DFS itself is O(V + E) where V, E are bounded by the alphabet size (at most 26) and the number of extracted edges.
- **Space: O(1) for the graph** (bounded by a fixed alphabet size of 26), **O(C)** overall including the input processing.

---

## Approach 2: Kahn's Algorithm (BFS, In-Degree Based)

### Intuition
Same as Course Schedule's Kahn's algorithm: track in-degrees for every letter, start with letters that have zero in-degree (no letters must come before them), and repeatedly "release" letters as their in-degree drops to zero.

### Python code
```python
from collections import deque

def alienOrder(words):
    graph = buildGraph(words)
    if graph is None:
        return ""

    in_degree = {letter: 0 for letter in graph}
    for letter in graph:
        for neighbor in graph[letter]:
            in_degree[neighbor] += 1

    queue = deque([letter for letter in graph if in_degree[letter] == 0])
    result = []

    while queue:
        letter = queue.popleft()
        result.append(letter)
        for neighbor in graph[letter]:
            in_degree[neighbor] -= 1
            if in_degree[neighbor] == 0:
                queue.append(neighbor)

    if len(result) != len(graph):
        return ""  # not all letters processed - a cycle exists somewhere

    return "".join(result)
```

### Line-by-line explanation
- Build the graph and in-degree counts, same as Course Schedule's Kahn's algorithm approach.
- `queue` starts with every letter that has no prerequisites.
- Process the queue: each released letter goes straight into `result` (in correct forward order this time, no reversal needed, since Kahn's algorithm naturally produces the order forward, unlike the post-order DFS approach).
- `if len(result) != len(graph): return ""` - if not every letter made it into `result`, some letters were stuck with permanently nonzero in-degree - a cycle among them, meaning no valid ordering exists.

### Time & space complexity
- **Time: O(C)**, **Space: O(1)** for the graph/in-degree structures (bounded alphabet size), **O(C)** overall.

---

## Common mistakes & misconceptions

1. **Comparing every pair of words instead of only adjacent pairs.** Non-adjacent words provide no new, reliable ordering information beyond what's already implied by the chain of adjacent comparisons between them — comparing `words[0]` directly against `words[4]`, say, can even introduce spurious or redundant edges; the sorted-order guarantee only ever applies between consecutive entries.
2. **Continuing to compare letters *after* the first difference is found.** Once `w1[i] != w2[i]`, every letter after position `i` is irrelevant to the ordering (it could be anything without breaking the sorted-order assumption) — comparing further would extract false constraints and can even fabricate cycles that don't really exist in the alien alphabet.
3. **Missing the prefix-violation edge case entirely.** It's tempting to think "if the loop finds no differing character, there's just nothing to record" and move on — but a longer word appearing *before* its own prefix (`"abc"` before `"ab"`) is a direct contradiction that ordinary cycle detection on the letter graph will **not** catch by itself (there's no cycle among letters here — the violation is about word *lengths*, not letter order) — this must be checked explicitly and separately.
4. **Forgetting to seed the graph with every letter that appears in the input**, not just letters that end up as the source or destination of some edge. A letter that never differs first in any comparison (e.g. it only ever appears in positions after the first difference, or only in a single word) still needs a node in the graph — otherwise it silently vanishes from the final answer instead of appearing somewhere valid in the output.
5. **Assuming there is a meaningful "brute force" alternative worth showing here.** Unlike many problems in this list, there isn't a slower-but-more-obvious approach to progressively optimize away — the real difficulty is entirely in the *translation* step (building the correct graph from the word list), not in choosing between a slow and fast algorithm once the graph exists; both topological sort methods shown above are already optimal, differing only in mechanism (DFS post-order vs. BFS in-degree), not in complexity class. Forcing a "brute force" framing onto this problem would misrepresent where its actual difficulty lies.

## Summary

| Approach | Notes |
|---|---|
| Post-order DFS | Requires reversing the result at the end; needs careful "visiting vs. visited" cycle tracking. |
| Kahn's algorithm (BFS) | Produces the order directly, no reversal needed; arguably slightly more intuitive for this specific problem. |

**Key takeaway:** this problem's real difficulty isn't the topological sort itself (identical machinery to Course Schedule) - it's correctly **translating** the word list into a graph: comparing only adjacent words, stopping at the first difference, and catching the specific prefix-violation edge case. Once the graph is built correctly, everything else is a direct reapplication of what you already know.
