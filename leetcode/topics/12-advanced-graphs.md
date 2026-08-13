# Topic 12: Advanced Graphs

## Core concepts / data structures

This topic builds directly on the Graphs topic - same nodes/edges/adjacency-list foundations - but covers graph problems that need a bit more machinery than plain DFS/BFS: extracting an **ordering** from constraints that aren't given in a simple list form, and (more broadly, for further study beyond this specific list) weighted shortest-path algorithms.

### Topological sort, revisited with a twist
In the Graphs topic, Course Schedule asked a **yes/no** question ("can all courses be finished?"). The problems in this topic often ask you to go one step further: **derive the actual ordering/constraints from a more indirect or unusual input format**, then run topological sort on the graph you've built. The core topological-sort machinery (Kahn's algorithm using in-degrees, or DFS with post-order) is identical to what you already know from Course Schedule - what's new here is the **graph-building step**, which requires more careful reasoning about what the input is actually telling you.

### Example of an indirect input format: ordering from word comparisons
A classic pattern (used in Alien Dictionary): you're given a **list of words**, claimed to be sorted according to some unknown alien alphabet's ordering. To figure out the alphabet's actual letter order, you compare **each pair of adjacent words** in the list - the **first position where they differ** tells you that one letter must come before another in the alien alphabet (this becomes a directed edge in a graph of letters). Once you've built this letter-ordering graph from all such pairwise comparisons, a standard topological sort on it gives you the alphabet order - assuming it's consistent (no cycles - if there's a contradiction, like `'a' before 'b'` AND `'b' before 'a'` both being implied, no valid ordering exists).

## Common patterns / techniques in this topic

| Pattern | When it applies |
|---|---|
| **Build a graph from indirect constraints, then topological sort** | The input isn't a plain edge list - you need to first figure out what the actual ordering constraints (edges) are (e.g. comparing adjacent words letter-by-letter), then apply standard topological sort machinery. |
| **Detecting inconsistent/impossible constraints** | Just like Course Schedule's cycle detection, but the "cycle" might arise from a more subtle contradiction buried in the derived constraints, not an explicit cycle in the raw input. |
| **Edge case: a word that's a prefix of an earlier word, but appears after it in an invalid way** | E.g. if "abc" appears before "ab" in what's claimed to be sorted order, that's *never* valid in any consistent ordering (a shorter string that's a genuine prefix of a longer one must always sort before it, in every real dictionary ordering) - a special-case check some of these problems require, separate from the general topological sort cycle check. |

## Key terminology

- **Topological sort** - see the Graphs topic; ordering nodes of a DAG so every edge points from earlier to later.
- **In-degree** - number of incoming edges to a node; central to Kahn's algorithm.
- **Constraint graph** - a graph built specifically to represent "must come before" relationships extracted from a problem's input, which may not be handed to you as an explicit edge list.

## Common beginner mistakes

1. **Not correctly extracting constraints from the input format.** For word-ordering problems, forgetting to only compare each pair of **adjacent** words (not every pair), and forgetting to stop comparing letters at the **first** point of difference (any letters after that first difference tell you nothing reliable about the ordering).
2. **Missing the "prefix contradiction" edge case** described above (a longer word appearing before its own valid prefix is always invalid, and won't necessarily be caught by ordinary cycle detection alone if handled naively).
3. **Building the graph with edges in the wrong direction** - easy to accidentally reverse "must come before" into "must come after," which silently produces a topological sort in reverse order or misses real cycles.
4. **Forgetting to include every letter/node that appears anywhere in the input**, even ones that never end up needing an edge (e.g. a letter that never differs first in any word comparison still needs to be included somewhere in the final ordering, typically with in-degree 0 from the start).

## How this compares to the Graphs topic

Everything mechanically graph-related here (topological sort itself, cycle detection) is identical to what you already learned in Course Schedule. The added difficulty in this topic is purely in the **translation step**: going from a problem's raw input to a well-defined graph of "must come before" edges, which requires more careful reading and reasoning about what the input actually implies, before any standard graph algorithm can even be applied.

## Starter problems

Given this topic (in your Blind 75 list) only contains one problem (Alien Dictionary), the most useful "warm-up" is really just making sure **Course Schedule** (from the Graphs topic) feels completely comfortable first, since Alien Dictionary is essentially "Course Schedule's topological sort, applied to a graph you have to construct yourself from a trickier input."

## What carries over from here

The general skill of "translate an unusual problem statement into a well-defined graph, then apply a standard graph algorithm" is broadly useful far beyond this one problem - many real-world scheduling, dependency-resolution, and ordering problems require exactly this kind of translation step before any textbook algorithm becomes applicable.
