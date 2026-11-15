# 1610. Maximum Number of Visible Points

> Source: https://algo.monster/liteproblems/1610

## Problem Statement

You are standing at a fixed position on a 2D plane and can rotate to look in different directions. Your goal is to find the maximum number of points you can see within your field of view.

Given:
- An array `points` where each `points[i] = [xi, yi]` represents a point on the X-Y plane
- Your position `location = [posx, posy]` (you cannot move from this position)
- A viewing `angle` in degrees that determines how wide your field of view is

Key mechanics:
- You start facing directly east (positive x-direction)
- You can rotate counterclockwise by any amount `d` degrees
- When rotated by `d` degrees, your field of view covers all angles in the range `[d - angle/2, d + angle/2]`
- A point is visible if the angle it makes with your position (relative to the east direction) falls within your current field of view

Special cases:
- Points located exactly at your position are always visible regardless of rotation
- Multiple points can exist at the same coordinate
- Points do not block the view of other points

The task is to determine the optimal rotation angle that allows you to see the maximum number of points simultaneously, and return that maximum count.

For example, if you have a 90-degree field of view and points scattered around you, you need to find which 90-degree arc contains the most points when you rotate to face different directions.

## Intuition

The key insight is to transform this geometric problem into a simpler problem about finding the maximum number of points within a [sliding window](/problems/sliding_window_maximum) of angles.

First, let's think about what we're actually looking for. We need to find an orientation where our field of view captures the most points. Since we can rotate to any angle, we're essentially looking for the best "angular window" of size `angle` degrees that contains the most points.

To approach this, we need to convert each point's position into an angle relative to our location. We can use `atan2(y_diff, x_diff)` to calculate the angle each point makes with the positive x-axis (east direction) from our position. This gives us an angle in radians for each point.

Once we have all angles, the problem becomes: find a contiguous range of `angle` degrees that contains the maximum number of angles. This is similar to a [sliding window](/problems/sliding_window_maximum) problem, but with a circular nature since angles wrap around (360° = 0°).

To handle the circular nature, we can use a clever trick: duplicate the angle array by adding `2π` to each angle and appending it to the original array. This way, when we slide our window across the sorted angles, we automatically handle the wrap-around case. For example, if we have angles at 350° and 10°, the duplicated array will also have 370° (350° + 360°), allowing us to capture both points in a window that crosses the 0°/360° boundary.

Points at our exact location need special handling since they don't have a defined angle (division by zero). These points are always visible regardless of our rotation, so we count them separately and add them to our final answer.

The [sliding window](/problems/sliding_window_maximum) can be efficiently implemented using binary search (`bisect_right`) to find how many points fall within `angle` degrees from each starting position, making the solution more efficient than checking every possible window manually.

**Learn more about [Math](/problems/math-basics), [Sorting](/problems/sorting_summary) and [Sliding Window](/problems/sliding_window_maximum) patterns.**

## Solution Approach

Let's walk through the implementation step by step:

**Step 1: Separate points at our location**
```python
v = []
x, y = location
same = 0
for xi, yi in points:
    if xi == x and yi == y:
        same += 1
    else:
        v.append(atan2(yi - y, xi - x))
```
We iterate through all points and separate them into two categories:
- Points at our exact location (`same`): These are always visible
- Other points: Calculate their angle using `atan2(yi - y, xi - x)` and store in array `v`

The `atan2` function returns the angle in radians between the positive x-axis and the line from our location to the point, handling all four quadrants correctly.

**Step 2: Sort the angles**
```python
v.sort()
n = len(v)
```
[Sorting](/problems/sorting_summary) the angles allows us to treat consecutive angles as potentially being in the same field of view.

**Step 3: Handle circular nature by duplicating angles**
```python
v += [deg + 2 * pi for deg in v]
```
We append a copy of all angles increased by `2π` radians (360°). This transforms the circular problem into a linear one. For example, if we have angles `[0.1, 1.5, 6.0]`, the array becomes `[0.1, 1.5, 6.0, 6.38, 7.85, 12.28]`.

**Step 4: Convert angle to radians**
```python
t = angle * pi / 180
```
Convert the field of view angle from degrees to radians for consistency.

**Step 5: Find maximum points in any window**
```python
mx = max((bisect_right(v, v[i] + t) - i for i in range(n)), default=0)
```
For each angle `v[i]` as a starting point:
- `v[i] + t` gives the upper bound of our field of view
- `bisect_right(v, v[i] + t)` finds the rightmost position where we could insert `v[i] + t` while maintaining sorted order - this gives us the index of the first angle outside our window
- `bisect_right(v, v[i] + t) - i` gives the count of points within the angular window `[v[i], v[i] + t]`
- We find the maximum across all possible starting positions

**Step 6: Return the result**
```python
return mx + same
```
Add the points at our location (`same`) to the maximum points we can see by rotation (`mx`).

The time complexity is `O(n log n)` due to [sorting](/problems/sorting_summary) and `n` binary searches, where `n` is the number of points not at our location.

## Example Walkthrough

Let's walk through a concrete example to illustrate the solution approach.

**Given:**
- `location = [1, 1]` (we're standing at position (1, 1))
- `angle = 90` (we have a 90-degree field of view)
- `points = [[2, 1], [2, 2], [3, 3], [1, 0], [1, 1], [0, 1]]`

**Step 1: Separate points at our location**

We check each point to see if it's at our location (1, 1):
- `[2, 1]`: Not at our location → calculate angle
- `[2, 2]`: Not at our location → calculate angle  
- `[3, 3]`: Not at our location → calculate angle
- `[1, 0]`: Not at our location → calculate angle
- `[1, 1]`: **At our location** → increment `same = 1`
- `[0, 1]`: Not at our location → calculate angle

For the non-matching points, we calculate angles using `atan2(yi - 1, xi - 1)`:
- `[2, 1]`: `atan2(1-1, 2-1) = atan2(0, 1) = 0°` (0 radians) - directly east
- `[2, 2]`: `atan2(2-1, 2-1) = atan2(1, 1) = 45°` (π/4 radians) - northeast
- `[3, 3]`: `atan2(3-1, 3-1) = atan2(2, 2) = 45°` (π/4 radians) - northeast
- `[1, 0]`: `atan2(0-1, 1-1) = atan2(-1, 0) = -90°` (-π/2 radians) - directly south
- `[0, 1]`: `atan2(1-1, 0-1) = atan2(0, -1) = 180°` (π radians) - directly west

So `v = [0, π/4, π/4, -π/2, π]` and `same = 1`

**Step 2: Sort the angles**

After sorting: `v = [-π/2, 0, π/4, π/4, π]` (which represents -90°, 0°, 45°, 45°, 180°)

**Step 3: Handle circular nature**

We duplicate each angle by adding 2π:
`v = [-π/2, 0, π/4, π/4, π, 3π/2, 2π, 9π/4, 9π/4, 3π]`

This represents: -90°, 0°, 45°, 45°, 180°, 270°, 360°, 405°, 405°, 540°

**Step 4: Convert viewing angle to radians**

`t = 90 * π / 180 = π/2` radians

**Step 5: Find maximum points in any window**

We check each starting position i (only need to check first n=5 positions):

- `i=0`: Start at -90°, window covers [-90°, 0°]
  - `bisect_right(v, -π/2 + π/2) = bisect_right(v, 0)` = 2
  - Points in window: 2 - 0 = 2 points (angles at -90° and 0°)

- `i=1`: Start at 0°, window covers [0°, 90°]
  - `bisect_right(v, 0 + π/2) = bisect_right(v, π/2)` = 4
  - Points in window: 4 - 1 = 3 points (angles at 0°, 45°, 45°)

- `i=2`: Start at 45°, window covers [45°, 135°]
  - `bisect_right(v, π/4 + π/2) = bisect_right(v, 3π/4)` = 4
  - Points in window: 4 - 2 = 2 points (angles at 45°, 45°)

- `i=3`: Start at 45°, window covers [45°, 135°]
  - `bisect_right(v, π/4 + π/2)` = 4
  - Points in window: 4 - 3 = 1 point (angle at 45°)

- `i=4`: Start at 180°, window covers [180°, 270°]
  - `bisect_right(v, π + π/2) = bisect_right(v, 3π/2)` = 6
  - Points in window: 6 - 4 = 2 points (angles at 180° and 270°, where 270° is the wrapped -90°)

Maximum points visible = 3 (when starting at 0°)

**Step 6: Return final result**

`return mx + same = 3 + 1 = 4`

We can see a maximum of 4 points: the 3 points in the angular window from 0° to 90° (east to north), plus the 1 point at our exact location.

## Python Solution

```python
class Solution:
    def visiblePoints(
        self, points: List[List[int]], angle: int, location: List[int]
    ) -> int:
        from math import atan2, pi
        from bisect import bisect_right
        from typing import List
      
        # Extract observer's location coordinates
        observer_x, observer_y = location
      
        # Count points that are at the same location as the observer
        same_location_count = 0
      
        # Store angles of all points relative to the observer
        angles = []
      
        # Calculate angle for each point relative to the observer
        for point_x, point_y in points:
            if point_x == observer_x and point_y == observer_y:
                # Point is at the same location as observer
                same_location_count += 1
            else:
                # Calculate angle using atan2 (returns angle in radians)
                angle_rad = atan2(point_y - observer_y, point_x - observer_x)
                angles.append(angle_rad)
      
        # Sort angles in ascending order
        angles.sort()
      
        # Get number of unique angle points
        num_angles = len(angles)
      
        # Duplicate the angles array with 2π added to handle circular nature
        # This allows us to check angles that wrap around the circle
        angles += [angle_val + 2 * pi for angle_val in angles]
      
        # Convert viewing angle from degrees to radians
        viewing_angle_rad = angle * pi / 180
      
        # Find maximum number of points visible within the viewing angle
        # Use sliding window approach with binary search
        max_visible = max(
            (bisect_right(angles, angles[i] + viewing_angle_rad) - i 
             for i in range(num_angles)), 
            default=0
        )
      
        # Return total visible points (including those at observer's location)
        return max_visible + same_location_count
```

## Time and Space Complexity

**Time Complexity:** `O(n log n)` where `n` is the number of points.

The time complexity breaks down as follows:
- Converting points to angles: `O(n)` - iterating through all points once and calculating `atan2` for each point (except those at the same location)
- Sorting the angles: `O(n log n)` - using Python's built-in sort
- Extending the array: `O(n)` - appending `n` elements to create the circular array
- Finding maximum visible points: `O(n log n)` - iterating through `n` positions and for each position, performing a binary search with `bisect_right` which takes `O(log n)` time

The dominant operation is either the sorting step or the binary search loop, both being `O(n log n)`.

**Space Complexity:** `O(n)` where `n` is the number of points.

The space complexity breakdown:
- Array `v` for storing angles: `O(n)` - stores at most `n` angles initially
- Extended array: `O(n)` - the array is doubled to handle circular cases, resulting in `2n` elements
- Variables for location, same count, and other constants: `O(1)`

The total auxiliary space used is `O(n)` for storing the angles and their circular extension.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Angle Wrapping Around 360 Degrees**
The most critical pitfall is failing to handle the circular nature of angles. When your field of view crosses the 0°/360° boundary (e.g., viewing from 350° to 20°), a naive approach would miss points in this range.

**Problem Example:**
- You're at origin with a 90° field of view
- Points at angles: 10°, 20°, 350°, 355°
- If you face 15° (covering -30° to 60°), you should see points at 10° and 20°
- If you face 5° (covering -40° to 50°), you should see points at 10°, 20°, 350°, and 355°

**Why the solution works:** By duplicating all angles with `+2π` added, we linearize the circular problem. The array `[10°, 20°, 350°, 355°]` becomes `[10°, 20°, 350°, 355°, 370°, 380°, 710°, 715°]`. Now checking the range `[350°, 440°]` correctly captures the wraparound.

### 2. **Using Degrees Instead of Radians**
`atan2` returns angles in radians (range: -π to π), but the input angle is in degrees. Mixing these units causes incorrect calculations.

**Incorrect approach:**
```python
# Wrong: comparing radians with degrees
if atan2(y, x) <= angle:  # atan2 returns radians, angle is degrees
```

**Correct approach:**
```python
# Convert angle to radians first
viewing_angle_rad = angle * pi / 180
if atan2(y, x) <= viewing_angle_rad:  # Both in radians
```

### 3. **Incorrect Angle Calculation**
Using `atan(y/x)` instead of `atan2(y, x)` loses quadrant information and fails when x=0.

**Problem with atan:**
- `atan(1/1)` and `atan(-1/-1)` both return π/4, but these represent different directions
- Division by zero when point is directly north or south

**Solution:** Always use `atan2(y_diff, x_diff)` which:
- Handles all four quadrants correctly
- Returns angles in the full range (-π to π)
- Handles x=0 gracefully

### 4. **Off-by-One Errors in Binary Search**
Using `bisect_left` instead of `bisect_right` can undercount visible points when angles are exactly on the boundary.

**Example scenario:**
- Field of view: [0, 1.57] radians
- Point at exactly 1.57 radians
- `bisect_left` would not include this boundary point
- `bisect_right` correctly includes it

### 5. **Not Handling Edge Cases**
Forgetting special cases leads to incorrect results:
- Empty points array → should return 0
- All points at observer's location → should return all points
- Angle ≥ 360° → all non-collocated points are always visible

**Robust handling:**
```python
# Handle angle >= 360 degrees
if angle >= 360:
    return len(points)

# Handle empty angles array
max_visible = max((bisect_right(...) for i in range(num_angles)), default=0)
```

