# 67. Meeting Rooms II

**LeetCode:** [#253 - Meeting Rooms II](https://leetcode.com/problems/meeting-rooms-ii/) (Premium - video/article write-ups are free) · **Topic:** [Intervals](../topics/16-intervals.md) · **Difficulty:** Medium

## Problem statement

Given an array of meeting time intervals, return the **minimum number of meeting rooms required** so that all meetings can happen (possibly simultaneously, using different rooms).

**Example:**
```
Input: intervals = [[0,30],[5,10],[15,20]]
Output: 2
Explanation: room 1 hosts [0,30]; room 2 hosts [5,10] then [15,20].
```

## Applicable approaches

- **Min-Heap of End Times** - the standard, expected solution.
- **Sweep Line (Two Pointers over separately sorted starts and ends)** - an equally valid, elegant alternative.

## Approach 1: Min-Heap of End Times

### Intuition
Sort meetings by **start** time, and process them in that order. Keep a **min-heap** of the end times of all meetings currently "in progress" (i.e. using a room). For each new meeting: first, check if the **earliest-ending** currently-in-progress meeting (the heap's root) has already finished by the time this new meeting starts - if so, that room is now free, so reuse it (pop it off the heap). Then, push the new meeting's end time onto the heap (either into the freed-up slot, or as a genuinely new room if nothing was free). **The number of rooms needed at any point is the heap's size** - track the maximum size the heap ever reaches.

### Algorithm
1. Sort meetings by start time.
2. Create an empty min-heap.
3. For each meeting `(start, end)`:
   - If the heap isn't empty and its smallest end time is ≤ `start` (that room has freed up in time), pop it (reuse that room).
   - Push `end` onto the heap (either reusing a freed slot's "logical position" or genuinely adding a new room).
4. The maximum size the heap ever reaches during this process is the answer.

### Python code
```python
import heapq

def minMeetingRooms(intervals):
    if not intervals:
        return 0

    intervals.sort(key=lambda iv: iv[0])
    heap = []  # min-heap of end times of meetings currently using a room
    max_rooms = 0

    for start, end in intervals:
        if heap and heap[0] <= start:
            heapq.heappop(heap)  # a room has freed up - reuse it

        heapq.heappush(heap, end)
        max_rooms = max(max_rooms, len(heap))

    return max_rooms
```

### Line-by-line explanation
- `intervals.sort(key=lambda iv: iv[0])` - process meetings in the order they start.
- `heap = []` - a min-heap holding the **end times** of every meeting currently occupying a room; the smallest value is always the *next* room to free up.
- `if heap and heap[0] <= start:` - check the earliest-ending currently-active meeting: has it already ended by (or exactly when) the current meeting starts? If so, that room is available.
- `heapq.heappop(heap)` - free up that room (remove its end time from the heap) - **note we don't decrement any separate room counter here**; the heap's size itself represents "rooms currently in use," so removing an entry directly represents freeing a room.
- `heapq.heappush(heap, end)` - the current meeting now occupies a room (either the just-freed one, conceptually, or a brand new one if nothing was free) - push its end time.
- `max_rooms = max(max_rooms, len(heap))` - track the largest the heap ever grows to, which represents the peak number of simultaneously active meetings - i.e. the minimum number of rooms that must have existed to handle that peak.

### Dry run
`intervals = [[0,30],[5,10],[15,20]]` → sorted by start (already in order).

`heap = []`, `max_rooms = 0`.

| meeting (start,end) | heap[0] <= start? | pop? | push end | heap after | max_rooms |
|---|---|---|---|---|---|
| (0,30) | heap empty | no | push 30 | [30] | 1 |
| (5,10) | `30 <= 5`? No | no | push 10 | [10,30] | 2 |
| (15,20) | `10 <= 15`? **Yes** | pop 10 | push 20 | [20,30] | stays 2 (size still 2 after pop+push) |

Final: `max_rooms = 2` ✅ matches expected output.

### Time & space complexity
- **Time: O(n log n)** - sorting is O(n log n); each of the n meetings does at most one heap push and one heap pop, each O(log n).
- **Space: O(n)** for the heap in the worst case (all meetings overlapping simultaneously).

---

## Approach 2: Sweep Line (Two Sorted Arrays of Starts and Ends)

### Intuition
Separate all the **start times** into one sorted list and all the **end times** into another sorted list. Walk through both lists simultaneously (two pointers): whenever the next event chronologically is a "start," a new room is needed (increment a running counter, and track the peak); whenever it's an "end," a room frees up (decrement the counter). This directly simulates the flow of meetings starting and ending over time, without needing a heap at all.

### Python code
```python
def minMeetingRooms(intervals):
    if not intervals:
        return 0

    starts = sorted(iv[0] for iv in intervals)
    ends = sorted(iv[1] for iv in intervals)

    rooms = 0
    max_rooms = 0
    s = e = 0

    while s < len(starts):
        if starts[s] < ends[e]:
            rooms += 1          # a meeting starts before the earliest currently-ending one ends
            max_rooms = max(max_rooms, rooms)
            s += 1
        else:
            rooms -= 1          # a meeting ends at or before the next one starts
            e += 1

    return max_rooms
```

### Line-by-line explanation
- `starts`, `ends` - separately sorted lists of just the start times and just the end times (deliberately decoupled from which interval they originally belonged to - we only care about the *timing* of start/end events, not which specific meeting each belongs to).
- `s, e = 0, 0` - pointers into `starts` and `ends` respectively, tracking which event we're considering next in each list.
- `while s < len(starts):` - process every start event (once every meeting has "started" in this simulation, we're done - any remaining end events don't need explicit processing, since they can't increase the peak room count any further).
- `if starts[s] < ends[e]:` - the next start event happens strictly before the next end event - a new room is needed right now.
- `rooms += 1; max_rooms = max(max_rooms, rooms); s += 1` - occupy a new room, track the peak, advance the start pointer.
- `else: rooms -= 1; e += 1` - the next end event happens at or before the next start event - a room frees up before (or exactly as) the next meeting begins, so it can be reused; advance the end pointer.

### Dry run
`intervals = [[0,30],[5,10],[15,20]]`

`starts = [0,5,15]` (sorted), `ends = [10,20,30]` (sorted).

`rooms=0, max_rooms=0, s=0,e=0`.

| starts[s] | ends[e] | starts[s] < ends[e]? | action | rooms | max_rooms | s,e after |
|---|---|---|---|---|---|---|
| 0 | 10 | Yes | rooms+=1 | 1 | 1 | s=1 |
| 5 | 10 | Yes | rooms+=1 | 2 | 2 | s=2 |
| 15 | 10 | No (`15<10` false) | rooms-=1 | 1 | 2 | e=1 |

`s=2 < len(starts)=3` still true, continue: `starts[2]=15` vs `ends[1]=20`: `15<20`? Yes → `rooms+=1=2`, `max_rooms=max(2,2)=2`, `s=3`. Loop ends (`s=3=len(starts)`).

Final: `max_rooms = 2` ✅ matches expected output.

### Time & space complexity
- **Time: O(n log n)** - dominated by sorting the two separate lists.
- **Space: O(n)** for the two sorted lists.

---

## Common mistakes & misconceptions

1. **Using a plain list scan instead of a heap to find the earliest-ending active meeting.** This is exactly the mistake flagged in the topic overview - repeatedly scanning a list for the minimum end time turns an O(n log n) solution into O(n²); the whole point of the min-heap is O(log n) access to "which room frees up soonest."
2. **Popping from the heap unconditionally on every iteration, instead of only when the earliest end time is actually ≤ the current start.** Popping unconditionally would incorrectly "free" a room that's still in use, undercounting the true number of rooms needed.
3. **Using `<` instead of `<=` in the room-freeing check (`heap[0] <= start`).** A meeting ending at exactly the same time the next one starts should be allowed to reuse that room (back-to-back meetings, no idle gap needed) - using strict `<` would conservatively over-count rooms in this boundary case.
4. **In the sweep-line approach, forgetting that `starts` and `ends` are sorted independently and no longer "linked" to their original interval.** This is intentional and correct (only the *timing* of events matters for counting simultaneous rooms, not which specific meeting each event belongs to) - but it's a common point of confusion when first seeing this approach, since it looks like information is being "lost" compared to the heap approach.
5. **Off-by-one in the sweep line's loop termination** (`while s < len(starts)`) - since every meeting must start, iterating over all start events is sufficient; remaining end events after the last start don't need explicit processing since they can only decrease `rooms`, never increase `max_rooms` further.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Min-heap of end times | O(n log n) | O(n) | The standard, most commonly taught solution; directly ties back to the Heap/Priority Queue topic. |
| Sweep line (two sorted arrays) | O(n log n) | O(n) | Equally valid, avoids an explicit heap, arguably more directly mirrors the "simulate time passing" intuition. |

**Key takeaway:** "how many resources are needed at the peak of simultaneous demand" problems are solved by simulating events **in chronological order** and tracking a running count - either via a min-heap (always knowing the earliest-freeing resource) or via separately sorted start/end event streams (a sweep line). Both approaches represent the exact same underlying idea; picking between them is mostly a matter of which framing feels more natural for a given problem.
