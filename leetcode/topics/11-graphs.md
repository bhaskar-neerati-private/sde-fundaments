# Topic 11: Graphs

## Core concepts / data structures

### Graph

**What it is:** a collection of **nodes** (also called "vertices") connected by **edges**. Unlike a tree, a graph can have cycles, multiple paths between two nodes, disconnected groups of nodes, and no single designated "root." A tree is actually just a special, restricted kind of graph (connected, no cycles, exactly one path between any two nodes) — everything you learned about DFS/BFS on trees still applies here, with one crucial addition explained below.

**Simple analogy:** think of a graph as a map of cities (nodes) connected by roads (edges). Unlike a family tree, you can potentially get from any city back to where you started by a different route (a cycle), and there might be several separate "islands" of connected cities with no roads between the islands at all (disconnected components).

**Directed vs. Undirected:** in a **directed** graph, an edge from A to B doesn't necessarily mean you can go from B to A (like a one-way street). In an **undirected** graph, every edge works both ways (like a regular two-way street) — this distinction matters enormously for how you build the adjacency list and how you reason about cycles (see below).

**Weighted vs. Unweighted:** edges can optionally have a **weight** (cost/distance) associated with them. Most of the problems in this specific list are unweighted (every edge "costs" the same), which is exactly why BFS — not more complex algorithms like Dijkstra's — is enough to find shortest paths here: BFS's "explore in order of discovery" naturally becomes "explore in order of distance" when every edge costs the same amount.

### How graphs are represented in code

**Adjacency list** (by far the most common representation for these problems): a hash map (or array of lists) where each node maps to a list of its directly connected neighbors.
```python
graph = {
    "A": ["B", "C"],
    "B": ["A", "D"],
    "C": ["A"],
    "D": ["B"],
}
```
Many problems give you the graph as a **grid** (2D array) instead of an explicit adjacency list — in that case, "neighbors" of a cell `(r, c)` are simply its up/down/left/right adjacent cells, computed on the fly rather than looked up in a hash map, but the underlying DFS/BFS logic is otherwise identical.

### DFS and BFS on graphs — the same tools, one crucial addition

The DFS and BFS code from the Trees topic works almost unchanged on graphs — **except graphs can have cycles**, so we must explicitly track a **`visited` set** to avoid infinitely revisiting the same nodes. A tree never needed this, precisely *because* a tree is structurally guaranteed to have no cycles — there's no way to walk from a node back to an ancestor. A graph has no such guarantee, so without explicit cycle protection, a DFS or BFS could loop forever, repeatedly re-discovering the same nodes through a cycle.
```python
def dfs(node, visited, graph):
    if node in visited:
        return
    visited.add(node)
    # process node here
    for neighbor in graph[node]:
        dfs(neighbor, visited, graph)

from collections import deque
def bfs(start, graph):
    visited = {start}
    queue = deque([start])
    while queue:
        node = queue.popleft()
        # process node here
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
```

## Common patterns / techniques in this topic

| Pattern | When it applies |
|---|---|
| **DFS/BFS with a `visited` set** | Basic traversal, connected component counting, flood-fill style problems (e.g. Number of Islands). |
| **BFS for shortest path (unweighted graph)** | "Fewest steps/edges to reach a target" — BFS explores in order of distance from the start, so the first time you reach the target is guaranteed to be via the shortest path, precisely because nothing closer could have been discovered later. |
| **Multi-source BFS** | Starting BFS from **multiple** nodes simultaneously (all pushed into the queue at the start) instead of just one — useful for "distance from the nearest of several sources" problems. |
| **Topological Sort** | Ordering nodes in a **directed acyclic graph (DAG)** such that every edge points from an earlier node to a later one in the ordering — used for "task scheduling with prerequisites"-style problems. Two common implementations: DFS-based (using recursion + a "currently exploring" marker to detect cycles) or Kahn's algorithm (BFS-based, using in-degree counts). |
| **Cycle detection** | In a directed graph: track nodes in the *current* recursion path (not just ever-visited) — revisiting a node that's still in the current path means a cycle. In an undirected graph: track the parent you came from, since walking back to your immediate parent isn't a cycle, but reaching any *other* already-visited node is. |
| **Union-Find (Disjoint Set Union)** | Efficiently tracking which nodes belong to the same connected component, especially when edges are being added incrementally, or when repeatedly asking "are these two nodes connected?" |

## Key terminology

- **Vertex / Node**, **Edge** — the two basic components of a graph.
- **Adjacency list** — the standard representation: node → list of its neighbors.
- **Connected component** — a maximal group of nodes all reachable from each other (possibly with several separate components in one graph).
- **Cycle** — a path that starts and ends at the same node without repeating any edge/node in between (other than the start/end).
- **DAG (Directed Acyclic Graph)** — a directed graph with no cycles; this is specifically what topological sort requires — the "no cycles" part is what guarantees a valid ordering can exist at all (a cycle would mean A must come before B, which must come before A, an impossible requirement).
- **Topological order** — an ordering of a DAG's nodes such that every directed edge goes from earlier to later in the ordering.
- **In-degree** — the number of edges pointing **into** a node (used heavily in Kahn's algorithm for topological sort).
- **Union-Find (Disjoint Set Union / DSU)** — a data structure for efficiently tracking and merging groups of connected elements, supporting near-O(1) "are these connected?" and "merge these two groups" operations.

## Common beginner mistakes

1. **Forgetting the `visited` set**, causing infinite loops on graphs with cycles (unlike trees, which structurally can't have this problem, as explained above).
2. **Confusing "visited" (ever visited, ever) with "in the current DFS path"** for directed-graph cycle detection. A node fully explored and popped off the recursion stack earlier is *not* the same as a node currently being explored higher up in the *same* path — conflating these gives false cycle detections, since two separate branches of a DAG can legitimately both lead to the same downstream node without that being a cycle.
3. **Using DFS when BFS is needed for shortest path (or vice versa).** DFS finds *a* path, but not necessarily the *shortest* one in an unweighted graph, since it commits to going deep down one branch before trying alternatives; BFS guarantees shortest path by exploring in increasing distance order, so the first time it reaches any node is guaranteed to be via the fewest possible edges.
4. **Not handling disconnected graphs.** Forgetting that a graph might have multiple separate components, and a single DFS/BFS from one starting node won't reach nodes in a different component — problems asking about "the whole graph" (not just "starting from node X") usually need an outer loop trying every unvisited node as a potential new starting point, the exact same pattern as Number of Islands' outer scan.
5. **Mixing up directed and undirected edge handling.** For an undirected graph represented as an adjacency list, each edge needs to be added to **both** nodes' neighbor lists (since it works both ways) — forgetting this effectively turns an undirected graph into a directed one by accident, breaking any algorithm that assumes true bidirectional traversal.
6. **Off-by-one / wrong bounds checking on grid-based graphs.** Forgetting to check that a neighboring cell is within the grid's bounds before accessing it — the same class of bug flagged in Word Search's backtracking, since grid-as-graph problems inherit all the same boundary-checking discipline.

## How this compares to Trees

Everything from the Trees topic (DFS, BFS, recursive traversal) directly transfers — a tree is just a graph with extra guarantees (no cycles, connected, exactly one path between any two nodes) that let you skip the `visited` set. The genuinely new ideas here are things that only make sense once cycles and multiple paths are possible: cycle detection, topological sort (which is meaningless on a tree, since a tree's parent-child structure already gives an unambiguous ordering with no cycles to worry about), and connected components (a tree, by definition, is always a single connected component, so "count the components" is a trivially-1 question on a tree).

## Starter problems (easy/medium, to warm up)

1. **Flood Fill** (LeetCode #733) — not in Blind 75, but the simplest possible grid-DFS warm-up.
2. **Number of Islands** (LeetCode #200) — in your Blind 75 list; the canonical connected-components-on-a-grid problem.
3. **Clone Graph** (LeetCode #133) — also in your Blind 75 list; a great "DFS/BFS while building a new structure" exercise.

## What carries over from here

BFS's "explore in order of distance" idea is the direct foundation for weighted-graph shortest-path algorithms (like Dijkstra's, covered conceptually in the next topic, Advanced Graphs) — Dijkstra's is essentially BFS with a priority queue instead of a plain queue, to handle edges of different costs, directly reusing the Heap/Priority Queue topic's "always process the current best next" idea. Topological sort and cycle detection also generalize directly into more advanced scheduling and dependency-resolution problems beyond this list.
