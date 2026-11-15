# 3009. Maximum Number of Intersections on the Chart

> Source: https://algo.monster/liteproblems/3009

## Problem Statement

You have a line chart made up of `n` points connected by line segments. The points are given as a 1-indexed array `y`, where each point has coordinates `(k, y[k])` - meaning the k-th point is located at x-coordinate `k` and y-coordinate `y[k]`.

The chart has these properties:
- Points are connected in order by line segments (point 1 connects to point 2, point 2 to point 3, and so on)
- No two consecutive points have the same y-coordinate (no horizontal segments in the chart)

Your task is to find the maximum number of times a horizontal line can intersect with this chart. You can draw the horizontal line at any y-coordinate you choose, and it extends infinitely in both directions.

For example, if you have points at `(1, 3)`, `(2, 1)`, `(3, 4)`, the chart would consist of a line segment from `(1, 3)` to `(2, 1)` and another from `(2, 1)` to `(3, 4)`. A horizontal line at y = 2 would intersect the chart twice - once on each segment.

The goal is to find the optimal position for the horizontal line that maximizes the number of intersections with the chart's line segments.

## Intuition

The key insight is to think about when a horizontal line at height `h` intersects a line segment. A segment connecting points `(i, y[i])` and `(i+1, y[i+1])` will be intersected by a horizontal line at height `h` if and only if `h` lies between `y[i]` and `y[i+1]` (exclusive of endpoints in most cases).

Instead of checking every possible height, we can use a sweep line approach. Imagine slowly moving a horizontal line from bottom to top. As we move this line:
- When it enters the range between two consecutive points' y-coordinates, we gain an intersection with that segment
- When it exits that range, we lose that intersection

To handle this efficiently, we track "events" - the heights where intersections begin and end. For each line segment from `(i, y[i])` to `(i+1, y[i+1])`:
- If `y[i] < y[i+1]` (ascending), the horizontal line intersects this segment for heights in range `(y[i], y[i+1])`
- If `y[i] > y[i+1]` (descending), the horizontal line intersects this segment for heights in range `(y[i+1], y[i])`

The solution uses a clever trick with coordinates multiplied by 2 to handle edge cases. This allows us to distinguish between:
- Segments that are ascending vs descending at endpoints
- Whether to include or exclude boundary points

By using a TreeMap to store events (sorted by height), we can process them in order. We maintain a running count of active intersections, adding 1 when entering a segment's range and subtracting 1 when leaving it. The maximum value this running count reaches is our answer.

The adjustment `(i == n - 1 ? 0 : y[i] > y[i - 1] ? -1 : 1)` handles the special case at endpoints to ensure we count intersections correctly when the horizontal line passes exactly through a point where the chart changes direction.

**Learn more about [Math](/problems/math-basics) patterns.**

## Solution Approach

The solution implements a sweep line algorithm using interval processing:

1. **Coordinate Transformation**: We multiply all y-coordinates by 2 to work with even numbers. This allows us to use odd numbers (like `2*y - 1` or `2*y + 1`) to represent positions slightly below or above integer y-values, avoiding ambiguity at endpoints.

2. **Processing Each Segment**: For each consecutive pair of points `(i, y[i-1])` and `(i+1, y[i])`:
   - Calculate `start = 2 * y[i-1]` (the doubled y-coordinate of the first point)
   - Calculate `end = 2 * y[i]` with an adjustment:
     - If it's the last segment (`i == n - 1`), no adjustment needed
     - Otherwise, if `y[i] > y[i-1]` (ascending), subtract 1 to exclude the endpoint
     - If `y[i] < y[i-1]` (descending), add 1 to exclude the endpoint

3. **Event Recording with TreeMap**: Use a `TreeMap<Integer, Integer>` to store events:
   - At `[Math](/problems/math-basics).min(start, end)`: increment counter by 1 (entering a segment's range)
   - At `Math.max(start, end) + 1`: decrement counter by 1 (leaving a segment's range)
   - The `merge` function accumulates multiple events at the same height

4. **Sweep Line Processing**: Iterate through the TreeMap in sorted order:
   ```java
   for (final int count : line.values()) {
       intersectionCount += count;  // Update running intersection count
       ans = [Math](/problems/math-basics).max(ans, intersectionCount);  // Track maximum
   }
   ```
   - Each value represents the change in intersection count at that height
   - The running sum `intersectionCount` tells us how many segments the horizontal line intersects at each height
   - We track the maximum value reached

The TreeMap ensures events are processed in ascending order of height, simulating the horizontal line moving from bottom to top. The maximum intersection count encountered during this sweep is the answer.

## Example Walkthrough

Let's walk through a small example with points `y = [1, 3, 2, 5]`, which creates a chart with points at coordinates `(1,1)`, `(2,3)`, `(3,2)`, `(4,5)`.

**Step 1: Coordinate Transformation**
We multiply all y-coordinates by 2: `[2, 6, 4, 10]`

**Step 2: Process Each Segment**

For segment 1→2 (from `(1,1)` to `(2,3)`):
- `start = 2 * 1 = 2`
- `end = 2 * 3 = 6`
- Since `3 > 1` (ascending) and it's not the last segment, adjust: `end = 6 - 1 = 5`
- Add events: increment at `min(2,5) = 2`, decrement at `max(2,5) + 1 = 6`

For segment 2→3 (from `(2,3)` to `(3,2)`):
- `start = 2 * 3 = 6`
- `end = 2 * 2 = 4`
- Since `2 < 3` (descending) and it's not the last segment, adjust: `end = 4 + 1 = 5`
- Add events: increment at `min(6,5) = 5`, decrement at `max(6,5) + 1 = 7`

For segment 3→4 (from `(3,2)` to `(4,5)`):
- `start = 2 * 2 = 4`
- `end = 2 * 5 = 10`
- Since it's the last segment, no adjustment: `end = 10`
- Add events: increment at `min(4,10) = 4`, decrement at `max(4,10) + 1 = 11`

**Step 3: TreeMap Events**
Our TreeMap contains:
- Height 2: +1 (entering segment 1→2)
- Height 4: +1 (entering segment 3→4)
- Height 5: +1 (entering segment 2→3)
- Height 6: -1 (leaving segment 1→2)
- Height 7: -1 (leaving segment 2→3)
- Height 11: -1 (leaving segment 3→4)

**Step 4: Sweep Line Processing**
```
Height 2: count = 0 + 1 = 1, max = 1
Height 4: count = 1 + 1 = 2, max = 2
Height 5: count = 2 + 1 = 3, max = 3  ← Maximum!
Height 6: count = 3 - 1 = 2, max = 3
Height 7: count = 2 - 1 = 1, max = 3
Height 11: count = 1 - 1 = 0, max = 3
```

The maximum intersection count is **3**, which occurs when the horizontal line is at height 2.5 (represented as 5 in our doubled coordinates). At this height, the line intersects all three segments of the chart.

## Python Solution

```python
class Solution:
    def maxIntersectionCount(self, y):
        """
        Find the maximum number of line segments that a horizontal line can intersect
        when drawn through a polyline defined by y-coordinates.

        Args:
            y: List of integers representing y-coordinates of polyline points

        Returns:
            Maximum number of intersections possible with any horizontal line
        """
        n = len(y)
        max_intersections = 0
        current_intersections = 0

        # Dictionary to store events: key is position, value is change in intersection count
        # Using a regular dict since we'll sort the keys later
        events = {}

        # Process each line segment from point i-1 to point i
        for i in range(1, n):
            # Double the y-coordinates to handle half-integer positions
            # This allows us to work with integer arithmetic while representing positions
            # that would otherwise be at y + 0.5
            start_y = 2 * y[i - 1]

            # Calculate end position with special handling for the last segment
            if i == n - 1:
                # If it's the last segment, use exact position (2 * y[i])
                end_y = 2 * y[i]
            else:
                # For intermediate segments, adjust position to handle boundary cases
                # If going up (y[i] > y[i-1]), subtract 1 to avoid counting at peak
                # If going down (y[i] < y[i-1]), add 1 to include the valley point
                end_y = 2 * y[i] + (-1 if y[i] > y[i - 1] else 1)

            # Determine the vertical range of the segment
            min_y = min(start_y, end_y)
            max_y = max(start_y, end_y)

            # Add events for line segment:
            # Increment intersection count at the start of the segment
            if min_y in events:
                events[min_y] += 1
            else:
                events[min_y] = 1

            # Decrement intersection count after the end of the segment
            if max_y + 1 in events:
                events[max_y + 1] -= 1
            else:
                events[max_y + 1] = -1

        # Sweep through all events in sorted order to find maximum concurrent intersections
        # Sort events by position (key) to process them in order
        for position in sorted(events.keys()):
            count_change = events[position]
            current_intersections += count_change
            max_intersections = max(max_intersections, current_intersections)

        return max_intersections
```

## Time and Space Complexity

**Time Complexity:** `O(n log n)`

The algorithm iterates through the array once with `n-1` iterations in the first loop, where each iteration performs:
- Two `merge` operations on the TreeMap, each taking `O(log m)` time where `m` is the number of entries in the map
- In the worst case, the TreeMap can have at most `2(n-1)` entries (two entries per line segment)
- Therefore, the first loop takes `O(n log n)` time

The second loop iterates through all values in the TreeMap:
- At most `2(n-1)` entries to iterate through, which is `O(n)`
- Each iteration performs constant time operations

Overall time complexity is dominated by the TreeMap operations: `O(n log n)`

**Space Complexity:** `O(n)`

The TreeMap stores at most `2(n-1)` entries in the worst case (when all line segments have distinct start and end points), which requires `O(n)` space. The remaining variables use constant space.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Mishandling Segment Endpoints at Local Extrema**

The most critical pitfall is incorrectly counting intersections when a horizontal line passes through a local maximum or minimum (peak or valley) of the polyline.

**The Problem:**
When a horizontal line passes through a peak (local maximum), it should count as **one** intersection, not two. Similarly for valleys. However, if we naively consider the y-range of each segment without adjustments, we might double-count at these points.

**Example of the Issue:**
Consider points: `[(1, 1), (2, 3), (3, 2)]`
- Segment 1: from y=1 to y=3
- Segment 2: from y=3 to y=2

Without proper handling, a horizontal line at y=3 would be counted as intersecting both segments (2 intersections), when visually it only touches the peak once (1 intersection).

**The Solution in the Code:**
The code handles this by adjusting the endpoint of non-terminal segments:
```python
if i == n - 1:
    end_y = 2 * y[i]
else:
    end_y = 2 * y[i] + (-1 if y[i] > y[i - 1] else 1)
```

This adjustment ensures:
- At a peak (`y[i] > y[i-1]`): We subtract 1, so the first segment's range ends just below the peak
- At a valley (`y[i] < y[i-1]`): We add 1, so the first segment's range extends just past the valley

### 2. **Integer vs. Half-Integer Position Confusion**

**The Problem:**
Working directly with integer y-coordinates makes it difficult to distinguish between:
- A line passing exactly through a point
- A line passing just above or below a point

**Why This Matters:**
The distinction affects counting. A line at y=2.5 between points at y=2 and y=3 clearly intersects the segment. But what about a line at exactly y=2?

**The Solution:**
Multiplying all y-coordinates by 2 allows us to use odd numbers to represent positions between original integer values:
- Original y=2 becomes 4
- Position "just below y=2" can be represented as 3
- Position "just above y=2" can be represented as 5

### 3. **Off-by-One Errors in Event Processing**

**The Problem:**
When using interval events, it's easy to make off-by-one errors when marking where an interval ends.

**Correct Approach:**
```python
# Mark the end of interval correctly
events[max_y + 1] -= 1  # Not events[max_y] -= 1
```

The `+1` is crucial because we want to decrement the count *after* we've passed through the segment's range, not at its exact endpoint.

### 4. **Not Handling the Last Segment Specially**

**The Problem:**
The last segment doesn't connect to another segment, so it doesn't form a peak or valley at its endpoint. Treating it the same as intermediate segments would incorrectly adjust its endpoint.

**The Solution:**
The code explicitly checks `if i == n - 1` to handle the last segment without adjustment, ensuring its full range is considered for intersections.

