# 218. The Skyline Problem

> Source: https://algo.monster/liteproblems/218

## Problem Statement

The Skyline Problem asks you to find the outline of a city's skyline formed by a collection of rectangular buildings when viewed from a distance.

You're given an array `buildings` where each building is represented as `[left, right, height]`:
- `left`: the x-coordinate of the building's left edge
- `right`: the x-coordinate of the building's right edge  
- `height`: the building's height

All buildings sit on flat ground at height 0.

The output should be a list of "key points" `[[x1, y1], [x2, y2], ...]` that define the skyline, where:
- Each point represents a location where the skyline height changes
- Points must be sorted by x-coordinate
- The last point always has y-coordinate 0 to mark where the rightmost building ends
- Consecutive horizontal lines at the same height must be merged (no redundant points)

For example, if multiple buildings create a continuous horizontal line at height 5 from x=4 to x=12, you should output `[4, 5]` and `[12, next_height]`, not separate points for each building edge.

The solution uses a sweep line algorithm with a priority queue. It processes all building edges (both left and right) in sorted order. At each x-coordinate:
1. Add any buildings that start at or before this position to the priority queue
2. Remove any buildings that have ended from the queue
3. The current maximum height in the queue determines the skyline height at this position
4. Only add a key point if the height changes from the previous point

## Intuition

The key insight is to think about what causes the skyline to change. The skyline only changes height at specific x-coordinates - namely, where buildings start or end. Between these critical points, the skyline remains at a constant height determined by the tallest building present at that location.

Imagine walking along the x-axis from left to right. At any point, the skyline height equals the maximum height among all buildings that "cover" that x-coordinate. This height changes only when:
1. A new building starts (potentially raising the skyline if it's taller than current buildings)
2. A building ends (potentially lowering the skyline if it was the tallest)

This observation leads us to focus on the building edges rather than entire buildings. We can collect all left and right edges, sort them by x-coordinate, and process them in order.

At each edge position, we need to know: "What's the tallest building currently active?" This naturally suggests using a data structure that can:
- Add new buildings as we encounter their left edges
- Remove buildings as we pass their right edges  
- Quickly tell us the maximum height among active buildings

A max heap (priority queue) fits perfectly. As we sweep through the x-coordinates:
- When we reach a building's left edge, add it to our heap
- When we pass a building's right edge, remove it from the heap
- The top of the heap always gives us the current skyline height

The skyline changes only when the maximum height changes. By comparing the current maximum with the previous one, we know when to add a new key point to our result.

This "sweep line" approach transforms a complex 2D problem into a simpler 1D problem of tracking maximum values over intervals, making it much more manageable to solve.

## Solution Approach

The implementation uses a sweep line algorithm with a priority queue to efficiently track the maximum height at each critical point.

**Step 1: Collect and Sort Critical Points**
```python
lines = []
for build in buildings:
    lines.extend([build[0], build[1]])
lines.sort()
```
We extract all x-coordinates where buildings start or end. These are the only positions where the skyline can change height. Sorting them allows us to process events from left to right.

**Step 2: Initialize Data Structures**
- `pq`: A priority queue (max heap) to track active buildings. Python's `PriorityQueue` is a min heap, so we negate heights to simulate a max heap.
- `skys`: The result list to store skyline key points
- `city`: A pointer to track which buildings we've processed

**Step 3: Process Each Critical Point**
```python
for line in lines:
    # Add buildings that start at or before current position
    while city < n and buildings[city][0] <= line:
        pq.put([-buildings[city][2], buildings[city][0], buildings[city][1]])
        city += 1
```
For each x-coordinate, we first add all buildings that start at or before this position to the priority queue. The queue entry is `[-height, left, right]` where height is negated for max heap behavior.

**Step 4: Remove Expired Buildings**
```python
while not pq.empty() and pq.queue[0][2] <= line:
    pq.get()
```
Remove buildings whose right edge is at or before the current position, as they no longer contribute to the skyline.

**Step 5: Determine Current Height**
```python
high = 0
if not pq.empty():
    high = -pq.queue[0][0]
```
The current skyline height is the maximum height among active buildings (top of the heap), or 0 if no buildings are active.

**Step 6: Add Key Point if Height Changes**
```python
if len(skys) > 0 and skys[-1][1] == high:
    continue
skys.append([line, high])
```
Only add a new key point if the height differs from the previous one. This ensures no consecutive horizontal lines at the same height, satisfying the problem's requirement for a minimal representation.

The algorithm processes each building edge once and performs heap operations, resulting in `O(n log n)` time complexity where `n` is the number of buildings. The space complexity is `O(n)` for storing the edges and heap.

## Example Walkthrough

Let's walk through a small example with three buildings: `[[1,3,3], [2,4,4], [5,7,1]]`

**Initial Setup:**
- Building 1: left=1, right=3, height=3 (covers x from 1 to 3)
- Building 2: left=2, right=4, height=4 (covers x from 2 to 4)  
- Building 3: left=5, right=7, height=1 (covers x from 5 to 7)

**Step 1: Collect critical points**
Extract all x-coordinates where buildings start or end: `[1, 3, 2, 4, 5, 7]`
After sorting: `[1, 2, 3, 4, 5, 7]`

**Step 2: Process each critical point**

*At x=1:*
- Add Building 1 (starts at 1) to heap: `[[-3, 1, 3]]`
- Current max height: 3
- Skyline changes from 0 to 3
- Add key point: `[1, 3]`

*At x=2:*
- Add Building 2 (starts at 2) to heap: `[[-4, 2, 4], [-3, 1, 3]]`
- Current max height: 4
- Skyline changes from 3 to 4
- Add key point: `[2, 4]`

*At x=3:*
- No new buildings start
- The top of the heap is Building 2 (ends at 4), which is still active, so nothing is removed. Building 1 has ended, but it sits below the top, so it stays in the heap for now: `[[-4, 2, 4], [-3, 1, 3]]`
- Current max height: 4
- Skyline stays at 4 (no change)
- No key point added

*At x=4:*
- No new buildings start
- Remove Building 2 (ends at 4) from the top of the heap. The new top is Building 1, whose right edge 3 is also `<= 4`, so it is removed too: `[]`
- Current max height: 0 (no active buildings)
- Skyline changes from 4 to 0
- Add key point: `[4, 0]`

*At x=5:*
- Add Building 3 (starts at 5) to heap: `[[-1, 5, 7]]`
- Current max height: 1
- Skyline changes from 0 to 1
- Add key point: `[5, 1]`

*At x=7:*
- No new buildings start
- Remove Building 3 (ends at 7) from heap: `[]`
- Current max height: 0
- Skyline changes from 1 to 0
- Add key point: `[7, 0]`

**Final Result:** `[[1,3], [2,4], [4,0], [5,1], [7,0]]`

This represents the skyline that:
- Rises to height 3 at x=1
- Rises to height 4 at x=2
- Drops to ground at x=4
- Rises to height 1 at x=5
- Returns to ground at x=7

## Python Solution

```python
from queue import PriorityQueue
from typing import List


class Solution:
    def getSkyline(self, buildings: List[List[int]]) -> List[List[int]]:
        """
        Find the skyline formed by buildings.
      
        Args:
            buildings: List of [left, right, height] representing each building
          
        Returns:
            List of key points [x, height] representing the skyline
        """
        # Result list to store skyline key points
        skyline_points = []
      
        # Collect all x-coordinates (both start and end of buildings)
        x_coordinates = []
        for building in buildings:
            x_coordinates.extend([building[0], building[1]])
        x_coordinates.sort()
      
        # Priority queue to store active buildings (max heap by height)
        # Format: [-height, left, right] (negative for max heap behavior)
        active_buildings = PriorityQueue()
      
        # Index to track which building to process next
        building_index = 0
        total_buildings = len(buildings)
      
        # Process each x-coordinate
        for current_x in x_coordinates:
            # Add all buildings that start at or before current x-coordinate
            while building_index < total_buildings and buildings[building_index][0] <= current_x:
                building = buildings[building_index]
                # Store as negative height for max heap behavior
                active_buildings.put([-building[2], building[0], building[1]])
                building_index += 1
          
            # Remove all buildings that have ended before or at current x-coordinate
            while not active_buildings.empty() and active_buildings.queue[0][2] <= current_x:
                active_buildings.get()
          
            # Get the current maximum height
            current_height = 0
            if not active_buildings.empty():
                # Convert back from negative to get actual height
                current_height = -active_buildings.queue[0][0]
          
            # Only add point if height changes (avoid duplicate heights)
            if skyline_points and skyline_points[-1][1] == current_height:
                continue
              
            skyline_points.append([current_x, current_height])
      
        return skyline_points
```

## Time and Space Complexity

**Time Complexity:** `O(n log n)`

The algorithm processes each building and creates critical points (start and end positions):
- Creating the `lines` array by extracting start and end points from all buildings: `O(n)`
- Sorting the `lines` array containing `2n` points: `O(n log n)`
- Main loop iterating through each line position: `O(n)` iterations
  - Inner while loop for adding buildings: each building is added to the priority queue exactly once across all iterations, `O(log n)` per insertion, `O(n log n)` total
  - Inner while loop for removing expired buildings: every iteration of this loop calls `pq.get()`, which removes a building for good. Each building can be removed at most once, so across the whole sweep this loop runs at most `n` times, `O(n log n)` total. The failed check that ends the loop costs `O(1)` per line position.
  - Reading the maximum with `pq.queue[0]`: `O(1)`

Expired buildings that are not at the top stay in the heap until they reach the top (lazy deletion), but that only delays their removal; it does not add work beyond the one `get()` per building.

**Space Complexity:** `O(n)`

- `lines` array stores `2n` points: `O(n)`
- Priority queue can contain at most `n` buildings: `O(n)`
- `skys` result array can have at most `2n` points: `O(n)`
- Other variables use constant space: `O(1)`

Total space complexity is `O(n)`.

## Common Pitfalls

**Pitfall: Incorrect Handling of Buildings with Same X-Coordinates**

One of the most common pitfalls in the skyline problem occurs when multiple buildings share the same x-coordinate for their edges. An implementation that records a key point after each individual start or end event can produce incorrect intermediate states and spurious key points. The solution above avoids this: at each x it first adds every building that has started, then removes every expired building at the top of the heap, and only then reads the height. A repeated x in `lines` produces the same height again, so it is skipped by the "height unchanged" check.

**Problematic Scenarios:**

1. **Multiple buildings starting at the same x-coordinate**: If buildings A (height 5) and B (height 10) both start at x=2, the algorithm might add A first, record height 5, then add B and record height 10, creating an incorrect intermediate point [2, 5].

2. **Building ending while another starts at the same x**: If building A ends at x=5 and building B starts at x=5, processing the removal before the addition could create a false gap [5, 0] when the skyline should continue seamlessly.

3. **Multiple buildings ending at the same x-coordinate**: Similar issues arise when several buildings end simultaneously.

**Example of the Problem:**
```python
buildings = [[0, 2, 3], [2, 5, 3]]  # Two adjacent buildings of same height
# An event-by-event approach might produce: [[0, 3], [2, 0], [2, 3], [5, 0]]
# Correct output should be: [[0, 3], [5, 0]]
```

**Solution: Process All Events at Same X-Coordinate Together**

Instead of processing events one by one, collect all events at the same x-coordinate and handle them as a batch:

```python
from queue import PriorityQueue
from typing import List

class Solution:
    def getSkyline(self, buildings: List[List[int]]) -> List[List[int]]:
        # Create events: (x, event_type, height)
        # event_type: 0 for start, 1 for end
        events = []
        for left, right, height in buildings:
            events.append((left, 0, height))  # Building start
            events.append((right, 1, height)) # Building end
      
        # Sort events: by x-coordinate, then by type (starts before ends),
        # then by height (taller first for starts, shorter first for ends)
        events.sort(key=lambda x: (x[0], x[1], -x[2] if x[1] == 0 else x[2]))
      
        result = []
        active_heights = PriorityQueue()
        active_heights.put(0)  # Ground level
      
        # Track active buildings with their end positions
        active_buildings = {}  # height -> count
      
        i = 0
        while i < len(events):
            current_x = events[i][0]
          
            # Get previous max height
            prev_max = self._get_max_height(active_buildings)
          
            # Process all events at current x-coordinate
            while i < len(events) and events[i][0] == current_x:
                x, event_type, height = events[i]
              
                if event_type == 0:  # Building starts
                    active_buildings[height] = active_buildings.get(height, 0) + 1
                else:  # Building ends
                    active_buildings[height] -= 1
                    if active_buildings[height] == 0:
                        del active_buildings[height]
              
                i += 1
          
            # Get new max height after processing all events at this x
            new_max = self._get_max_height(active_buildings)
          
            # Only add point if height changed
            if new_max != prev_max:
                result.append([current_x, new_max])
      
        return result
  
    def _get_max_height(self, active_buildings):
        return max(active_buildings.keys()) if active_buildings else 0
```

**Alternative Solution Using Multiset/TreeMap Pattern:**

For better performance with frequent max queries, use a data structure that maintains sorted order:

```python
from sortedcontainers import SortedList

class Solution:
    def getSkyline(self, buildings: List[List[int]]) -> List[List[int]]:
        events = []
        for left, right, height in buildings:
            # Negative height for start events ensures they're processed first
            # when at same x-coordinate
            events.append((left, -height))  # Start event
            events.append((right, height))   # End event
        events.sort()
      
        result = []
        heights = SortedList([0])  # Multiset of active heights
      
        i = 0
        while i < len(events):
            current_x = events[i][0]
          
            # Process all events at same x-coordinate
            while i < len(events) and events[i][0] == current_x:
                x, h = events[i]
                if h < 0:  # Start event
                    heights.add(-h)
                else:  # End event
                    heights.remove(h)
                i += 1
          
            # Check if max height changed
            max_height = heights[-1]
            if not result or result[-1][1] != max_height:
                result.append([current_x, max_height])
      
        return result
```

This approach ensures that all changes at the same x-coordinate are processed atomically, preventing intermediate incorrect states from appearing in the output.

