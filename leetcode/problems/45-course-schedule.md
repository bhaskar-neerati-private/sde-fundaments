# 45. Course Schedule

**LeetCode:** [#207 - Course Schedule](https://leetcode.com/problems/course-schedule/) · **Topic:** [Graphs](../topics/11-graphs.md) · **Difficulty:** Medium

## Problem statement

There are `numCourses` courses labeled `0` to `numCourses - 1`. Some courses have prerequisites, given as pairs `[a, b]` meaning "to take course `a`, you must first take course `b`." Return `True` if it's possible to finish all courses (i.e., there's no impossible circular dependency), `False` otherwise.

## Applicable approaches

- **DFS cycle detection** — the most conceptually direct approach.
- **Kahn's Algorithm (BFS with in-degree counts)** — equally valid, iterative alternative.

## Approach 1: DFS Cycle Detection

### Intuition

"Can I finish all courses" is really asking: **does the prerequisite graph contain a cycle?** If course A requires B, which requires C, which circles back to requiring A, then none of A, B, or C can ever be taken first — there's no valid starting point, so it's genuinely impossible, no matter what order you try. Conversely, if the graph has *no* cycles (it's a DAG, per the topic overview), a valid course order is always guaranteed to exist (this is exactly what topological sort produces). So the entire problem reduces to: **build the prerequisite graph, then detect whether it has a cycle.**

**Why a plain `visited` set isn't enough here (the crucial addition over ordinary graph traversal):** the topic overview draws a sharp distinction between "ever visited" and "currently in the recursion path." A node that was fully explored earlier and has no cycle through it is safe to revisit (e.g., two different courses might both require the same prerequisite — that's not a cycle, just a shared dependency, a completely normal DAG shape). But a node that we're **currently in the middle of exploring**, that gets reached again via some other path, means we've looped back on ourselves — that specifically is a cycle. So we need two separate pieces of state: `visited` (fully done, safe forever) and `in_progress` / `path` (currently being explored on this specific recursive call stack).

### Algorithm

1. Build an adjacency list: course → list of its prerequisites.
2. Maintain two sets: `visited` (fully processed, no cycle found through it) and `in_path` (currently on the active recursion stack).
3. `dfs(course)`:
   - If `course` is in `in_path`, we've found a cycle → return `False`.
   - If `course` is in `visited`, we already know it's safe → return `True`.
   - Otherwise, add `course` to `in_path`, recursively check all its prerequisites (if any return `False`, propagate `False` immediately), then remove `course` from `in_path` and add it to `visited`.
4. Run `dfs` on every course (handles disconnected parts of the graph, per the topic overview's "outer loop for disconnected graphs" pattern). If any call returns `False`, the answer is `False`; otherwise `True`.

### Python code
```python
def canFinish(numCourses, prerequisites):
    graph = {i: [] for i in range(numCourses)}
    for course, prereq in prerequisites:
        graph[course].append(prereq)

    visited = set()
    in_path = set()

    def dfs(course):
        if course in in_path:
            return False  # cycle detected
        if course in visited:
            return True  # already confirmed safe

        in_path.add(course)
        for prereq in graph[course]:
            if not dfs(prereq):
                return False
        in_path.remove(course)
        visited.add(course)
        return True

    return all(dfs(course) for course in range(numCourses))
```

### Line-by-line explanation

- `graph = {i: [] for i in range(numCourses)}` then populate — build the adjacency list mapping each course to its prerequisites; note this is a **directed** edge (course → prereq), since "A requires B" is not symmetric (B doesn't require A) — critically different from the undirected grid adjacency in Number of Islands.
- `visited` vs `in_path` — exactly the two-set distinction the topic overview flags as the #1 cycle-detection mistake to avoid: conflating "ever visited" with "currently exploring."
- `if course in in_path: return False` — we've reached a node that's an ancestor of itself in the current DFS call stack — this is precisely what a cycle means in a directed graph.
- `if course in visited: return True` — we've already fully verified this course (and everything under it) has no cycle, on some earlier call — no need to redo the work, and importantly, this is *not* a cycle just because we're seeing it again (it's not in `in_path` anymore, meaning we already finished exploring it safely).
- `in_path.add(course)` before recursing, `in_path.remove(course)` after — this is the critical bookkeeping: a course is only "in the current path" while we're actively still exploring its subtree; once we return from it, it must be removed from `in_path` (even though it stays in `visited` permanently) so that a *different*, unrelated path reaching this same course later isn't mistakenly flagged as a cycle.
- `for prereq in graph[course]: if not dfs(prereq): return False` — if any prerequisite chain leads to a cycle, propagate that failure immediately up the call stack — no need to check remaining prerequisites once we know it's impossible.
- `return all(dfs(course) for course in range(numCourses))` — check every course as a potential starting point, handling courses in separate disconnected parts of the graph, same "outer loop" pattern as Number of Islands.

### Dry run

`numCourses = 3`, `prerequisites = [[1,0],[2,1],[0,2]]` (course 1 needs 0, course 2 needs 1, course 0 needs 2 — a cycle: 0→2→1→0... wait, let's trace it directly as the graph is built: `graph = {0:[2], 1:[0], 2:[1]}`).

- `dfs(0)`: not in `in_path`, not in `visited`. Add `0` to `in_path = {0}`. Its prereqs: `[2]`.
  - `dfs(2)`: not in `in_path`, not in `visited`. Add `2` to `in_path = {0, 2}`. Its prereqs: `[1]`.
    - `dfs(1)`: not in `in_path`, not in `visited`. Add `1` to `in_path = {0, 2, 1}`. Its prereqs: `[0]`.
      - `dfs(0)`: **`0` IS in `in_path`** → return `False` (cycle detected!).
    - `dfs(1)` propagates `False` immediately.
  - `dfs(2)` propagates `False`.
- `dfs(0)` propagates `False`.
- `all(...)` short-circuits to `False` — **cannot finish all courses**, correctly detecting the 0→2→1→0 cycle.

### Time & space complexity

- **Time: O(V + E)** — each course is fully explored (added to `visited`) at most once; the `visited` check ensures we never re-explore a course's entire subtree redundantly, even though it might be reached via multiple different prerequisite chains.
- **Space: O(V + E)** for the graph, plus O(V) for `visited`/`in_path`/recursion depth.

---

## Approach 2: Kahn's Algorithm (BFS with In-Degree Counts)

### Intuition

A completely different way to detect "can this DAG be topologically sorted": repeatedly peel off nodes that currently have **zero remaining prerequisites** (in-degree zero — nothing left pointing into them), since those are always safe to take next. Each time we "take" a course, we remove it and decrement the in-degree of everything that depended on it, potentially freeing up new zero-in-degree courses. **If we can peel off every course this way, there's no cycle (a valid order exists). If we get stuck with courses remaining but none having in-degree zero, those remaining courses must form a cycle** (each one is waiting on another, which is waiting on another, forming a loop with no valid starting point).

### Algorithm

1. Build the graph in the **prerequisite → dependent** direction (opposite of Approach 1: `prereq → [courses that need it]`) and compute each course's in-degree (number of prerequisites it still has).
2. Initialize a queue with every course that has in-degree 0 (no prerequisites at all — safe to take immediately).
3. Repeatedly dequeue a course, count it as "taken," and decrement the in-degree of everything that depended on it; if any of those drop to 0, enqueue them.
4. If the total number of courses "taken" equals `numCourses`, return `True`; otherwise, some courses were never reachable (stuck in a cycle) — return `False`.

### Python code
```python
from collections import deque

def canFinish(numCourses, prerequisites):
    graph = {i: [] for i in range(numCourses)}
    in_degree = [0] * numCourses

    for course, prereq in prerequisites:
        graph[prereq].append(course)  # prereq -> dependent course
        in_degree[course] += 1

    queue = deque([course for course in range(numCourses) if in_degree[course] == 0])
    taken = 0

    while queue:
        course = queue.popleft()
        taken += 1
        for dependent in graph[course]:
            in_degree[dependent] -= 1
            if in_degree[dependent] == 0:
                queue.append(dependent)

    return taken == numCourses
```

### Line-by-line explanation

- `graph[prereq].append(course)` — note the direction is **reversed** compared to Approach 1's `graph[course].append(prereq)` — here we're modeling "once prereq is done, course becomes one step closer to available," so the edge points from prerequisite to dependent.
- `in_degree[course] += 1` — track how many *still-unsatisfied* prerequisites each course has.
- `queue = deque([... in_degree[course] == 0])` — seed the queue with every course that needs nothing else first — these are all immediately safe to take (a multi-source BFS start, per the topic overview).
- The main loop — "take" a course (count it), then for everything that depended on it, reduce their remaining-prerequisite count by one; a course becomes available the moment its very *last* remaining prerequisite is satisfied (`in_degree` hits exactly 0).
- `return taken == numCourses` — if every course was eventually taken, no cycle existed. If some courses never reached in-degree 0 (because they were stuck waiting on each other in a cycle), `taken` will be less than `numCourses`.

### Dry run

Same example: `prerequisites = [[1,0],[2,1],[0,2]]` → `graph = {0:[1], 1:[2], 2:[0]}` (reversed direction), `in_degree = [1, 1, 1]` (every course has exactly one prerequisite, forming the cycle 0→1→2→0).

- `queue = []` (no course has in-degree 0 — every course is waiting on something).
- Loop never executes (queue empty from the start). `taken = 0`.
- `0 == 3`? No → return `False`. Correctly detects the cycle, this time by noticing **no course was ever safe to start with**.

### Time & space complexity

- **Time: O(V + E)** — every node is enqueued/dequeued at most once, and every edge is processed exactly once when decrementing in-degrees.
- **Space: O(V + E)** for the graph and in-degree array.

---

## Common mistakes & misconceptions

1. **Using a single `visited` set for DFS cycle detection instead of the `visited`/`in_path` split.** As shown in the topic overview's mistake list, this is the single most common bug in this exact problem — it either causes false "no cycle" results (if you never re-check already-visited-but-not-currently-in-path nodes, that's actually fine) or, worse, wrongly flags legitimate shared dependencies as cycles if you don't distinguish "in the current path" from "visited at some point, possibly on a totally different branch."
2. **Building the graph in the wrong direction for Kahn's algorithm.** Approach 1 needs `course → prereq` (to recursively ask "what does this course depend on"), while Approach 2 needs `prereq → course` (to ask "what becomes available once this is done") — mixing these up silently breaks the respective algorithm.
3. **Forgetting to remove a course from `in_path` after finishing its DFS exploration.** If a course stays marked "in path" forever after being explored, an unrelated later path that happens to reach it would be incorrectly flagged as a cycle, even though the earlier exploration already completed and returned normally.
4. **Assuming this problem requires producing the actual course order**, when it only asks for a yes/no feasibility answer. (The closely related problem "Course Schedule II" *does* ask for the actual order — that's a small but meaningful variation worth being aware of, solved by collecting the topological order as courses are taken, rather than just counting them.)

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| DFS cycle detection | O(V+E) | O(V+E) | Conceptually direct: "can finish" = "graph has no cycle." |
| Kahn's algorithm (BFS) | O(V+E) | O(V+E) | Iterative; naturally extends to producing the actual valid order. |

**Key takeaway:** "is this set of dependencies satisfiable" is always a cycle-detection question on a directed graph — the same underlying insight applies far beyond this specific problem (build systems, task schedulers, spreadsheet formula dependencies all reduce to exactly this). DFS cycle detection and Kahn's algorithm are two genuinely different but equally valid ways to answer it, and it's worth being comfortable with both since some follow-up variants (like producing the actual order) are more naturally expressed with one or the other.
