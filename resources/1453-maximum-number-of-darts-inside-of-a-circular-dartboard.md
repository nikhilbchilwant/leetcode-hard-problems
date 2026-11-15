# 1453. Maximum Number of Darts Inside of a Circular Dartboard

> Source: https://algo.monster/liteproblems/1453

## Problem Statement

Alice has thrown `n` darts at positions on a large wall. Each dart's position is given as coordinates `[x_i, y_i]` in the array `darts`.

Bob wants to place a circular dartboard with radius `r` somewhere on the wall to maximize the number of Alice's darts that fall within or on the boundary of the dartboard.

The task is to find the maximum number of darts that can be covered by optimally placing a dartboard of radius `r`.

**Key Points:**
- You have `n` dart positions as 2D coordinates
- You need to place a circle of radius `r` to cover as many darts as possible
- A dart is covered if it lies inside or exactly on the boundary of the circle
- Return the maximum number of darts that can be covered

**Example:**
If you have darts at positions `[[1,2], [3,4], [5,6]]` and radius `r = 2`, you need to find where to place a circle of radius 2 to cover the maximum number of these dart positions.

The solution approach involves:
1. For each pair of darts, calculate potential circle centers where a circle of radius `r` passes through both darts
2. For each potential center, count how many total darts fall within radius `r` 
3. Track the maximum count found

The geometric insight is that an optimal circle placement will have at least two darts on its boundary (unless only one dart can be covered). This is why the solution examines all pairs of darts to find candidate circle centers.

## Intuition

The key insight is that if we want to maximize the number of darts covered by a circle, the optimal placement will often have the circle's boundary passing through at least one or two darts. Why? Because if a circle covers some darts but none of them touch its boundary, we could slightly shift the circle to potentially include more darts.

Think about it this way: if we have an optimal circle placement, we can keep moving it until at least two darts are on its boundary. This movement won't reduce the number of covered darts (since we're not pushing any darts outside), and it might even allow us to include more.

This observation drastically reduces our search space. Instead of considering infinite possible positions for the circle center, we only need to consider positions where the circle passes through pairs of darts.

For any two darts within distance `2r` of each other (if they're farther apart, no circle of radius `r` can contain both), there are exactly two possible positions for a circle center such that both darts lie on the circle's boundary. These two positions are found by:
1. Finding the midpoint between the two darts
2. Moving perpendicular to the line connecting the darts by a specific distance

The perpendicular distance is calculated using the Pythagorean theorem: if the distance between two darts is `d`, and they both lie on a circle of radius `r`, then the center is at distance `sqrt(r² - (d/2)²)` from the midpoint, perpendicular to the line connecting the darts.

By checking all pairs of darts and their corresponding possible circle centers, we ensure we don't miss the optimal placement. For each candidate center, we count how many darts fall within radius `r` and keep track of the maximum count.

The algorithm handles the edge case where only one dart can be covered by initializing the maximum to 1, and the floating-point comparison uses a small epsilon (`1e-7`) to handle numerical precision issues.

**Learn more about [Math](/problems/math-basics) patterns.**

## Solution Approach

The implementation consists of three main components:

**1. Finding Possible Circle Centers (`possibleCenters` function)**

For two darts at positions `(x1, y1)` and `(x2, y2)`, we calculate where a circle of radius `r` can be centered such that both darts lie on its boundary:

- First, calculate the distance `d` between the two darts: `d = sqrt((x2-x1)² + (y2-y1)²)`
- If `d > 2r`, no circle of radius `r` can contain both darts, return empty list
- Find the midpoint: `(mid_x, mid_y) = ((x1+x2)/2, (y1+y2)/2)`
- Calculate the perpendicular distance from midpoint to center: `dist_to_center = sqrt(r² - (d/2)²)`
- The perpendicular direction is obtained by rotating the vector `(dx, dy)` by 90 degrees: `(dy, -dx)`
- Normalize and scale this perpendicular vector to get the offset:
  - `offset_x = dist_to_center * dy / d`
  - `offset_y = dist_to_center * -dx / d`
- Return two possible centers: one on each side of the line connecting the darts

**2. Counting Darts Within Circle (`countDarts` function)**

For a given center `(x, y)`, count how many darts fall within or on the circle:
- Iterate through all darts
- Calculate distance from each dart to the center using the distance formula
- If distance ≤ `r + 1e-7`, increment the count (the small epsilon handles floating-point precision)

**3. Main Algorithm**

The main algorithm exhaustively checks all possible optimal placements:

```python
max_darts = 1  # At least one dart can always be covered
for i in range(n):
    for j in range(i + 1, n):
        # Get possible centers where circle passes through darts i and j
        centers = possibleCenters(darts[i], darts[j])
      
        # For each center, count darts and update maximum
        for center in centers:
            max_darts = max(max_darts, countDarts(center))
```

**Time Complexity:** `O(n² × n) = O(n³)` where `n` is the number of darts
- We check `O(n²)` pairs of darts
- For each pair, we generate at most 2 centers
- For each center, we count darts in `O(n)` time

**Space Complexity:** `O(1)` as we only use a constant amount of extra space regardless of input size

The algorithm guarantees finding the optimal solution because it checks all geometrically significant positions where the circle boundary passes through pairs of darts, which includes all potential optimal placements.

## Example Walkthrough

Let's walk through a concrete example with darts at positions `[[1,2], [3,2], [2,4]]` and radius `r = 2`.

**Step 1: Initialize**
- Start with `max_darts = 1` (we can always cover at least one dart)

**Step 2: Check all pairs of darts**

**Pair 1: Darts [1,2] and [3,2]**
- Distance between them: `d = sqrt((3-1)² + (2-2)²) = 2`
- Since `d = 2 ≤ 2r = 4`, we can find circle centers
- Midpoint: `(2, 2)`
- Perpendicular distance: `sqrt(4 - 1) = sqrt(3) ≈ 1.732`
- The line between darts is horizontal, so perpendicular is vertical
- Two possible centers: `(2, 2+1.732)` and `(2, 2-1.732)`
  - Center at `(2, 3.732)`: Check distances to all darts
    - Dart [1,2]: `sqrt((1-2)² + (2-3.732)²) = sqrt(1 + 3) = 2` ✓ (on boundary)
    - Dart [3,2]: `sqrt((3-2)² + (2-3.732)²) = sqrt(1 + 3) = 2` ✓ (on boundary)
    - Dart [2,4]: `sqrt((2-2)² + (4-3.732)²) = 0.268 < 2` ✓ (inside)
    - **Count: 3 darts**
  - Center at `(2, 0.268)`: Would only cover the first two darts

**Pair 2: Darts [1,2] and [2,4]**
- Distance: `d = sqrt(1 + 4) = sqrt(5) ≈ 2.236`
- Since `d ≤ 4`, find centers
- Midpoint: `(1.5, 3)`
- Perpendicular distance: `sqrt(4 - 1.25) ≈ 1.658`
- Calculate two centers and count covered darts
  - Best count from this pair: 2 darts

**Pair 3: Darts [3,2] and [2,4]**
- Distance: `d = sqrt(1 + 4) = sqrt(5) ≈ 2.236`
- Similar process yields at most 2 darts covered

**Step 3: Return maximum**
- The maximum count found is **3 darts**, achieved when the circle is centered at approximately `(2, 3.732)`

This example demonstrates how the algorithm systematically checks all pairs of darts to find circle placements where two darts lie on the boundary, then counts all darts within each such circle to find the optimal placement.

## Python Solution

```python
class Solution:
    def numPoints(self, darts: list[list[int]], r: int) -> int:
        """
        Find the maximum number of darts that can be enclosed within a dartboard of radius r.
      
        The algorithm works by finding all possible circle centers that pass through pairs of darts,
        then counting how many darts fall within each circle.
      
        Args:
            darts: List of [x, y] coordinates representing dart positions
            r: Radius of the dartboard
          
        Returns:
            Maximum number of darts that can be enclosed in a circle of radius r
        """
        from math import sqrt
      
        def count_darts_in_circle(center_x: float, center_y: float) -> int:
            """
            Count how many darts are within the circle centered at (center_x, center_y).
          
            Args:
                center_x: X-coordinate of circle center
                center_y: Y-coordinate of circle center
              
            Returns:
                Number of darts within the circle (including boundary)
            """
            count = 0
            for dart_x, dart_y in darts:
                # Calculate distance from dart to center
                distance = sqrt((center_x - dart_x) ** 2 + (center_y - dart_y) ** 2)
                # Check if dart is within circle (with small epsilon for floating point comparison)
                if distance <= r + 1e-7:
                    count += 1
            return count
      
        def find_circle_centers(x1: float, y1: float, x2: float, y2: float) -> list[tuple[float, float]]:
            """
            Find the two possible centers of circles with radius r that pass through 
            points (x1, y1) and (x2, y2).
          
            Args:
                x1, y1: Coordinates of first point
                x2, y2: Coordinates of second point
              
            Returns:
                List of up to 2 possible circle centers as (x, y) tuples
            """
            # Calculate distance between the two points
            dx = x2 - x1
            dy = y2 - y1
            distance_between_points = sqrt(dx * dx + dy * dy)
          
            # If points are too far apart, no circle of radius r can pass through both
            if distance_between_points > 2 * r:
                return []
          
            # Find midpoint between the two points
            mid_x = (x1 + x2) / 2
            mid_y = (y1 + y2) / 2
          
            # Calculate distance from midpoint to circle center
            # Using Pythagorean theorem: r^2 = (d/2)^2 + dist_to_center^2
            dist_to_center = sqrt(r * r - (distance_between_points / 2) * (distance_between_points / 2))
          
            # Calculate perpendicular offset from midpoint to circle centers
            # The perpendicular direction is obtained by rotating the vector (dx, dy) by 90 degrees
            offset_x = dist_to_center * dy / distance_between_points
            offset_y = dist_to_center * (-dx) / distance_between_points
          
            # Return both possible circle centers
            return [
                (mid_x + offset_x, mid_y + offset_y),
                (mid_x - offset_x, mid_y - offset_y),
            ]
      
        n = len(darts)
        max_darts_in_circle = 1  # At minimum, we can always enclose one dart
      
        # Try all pairs of darts to find circle centers
        for i in range(n):
            for j in range(i + 1, n):
                # Find possible circle centers that pass through darts[i] and darts[j]
                possible_centers = find_circle_centers(
                    darts[i][0], darts[i][1], 
                    darts[j][0], darts[j][1]
                )
              
                # For each possible center, count darts and update maximum
                for center_x, center_y in possible_centers:
                    darts_count = count_darts_in_circle(center_x, center_y)
                    max_darts_in_circle = max(max_darts_in_circle, darts_count)
      
        return max_darts_in_circle
```

## Time and Space Complexity

**Time Complexity:** `O(n³)` where `n` is the number of darts.

The analysis breaks down as follows:
- The outer two nested loops iterate through all pairs of darts: `O(n²)` pairs in total
- For each pair of darts, `possibleCenters` computes at most 2 circle centers in `O(1)` time
- For each center (at most 2 per pair), `countDarts` is called, which iterates through all `n` darts to count how many fall within the radius: `O(n)` time
- Overall: `O(n²) × O(1) × 2 × O(n) = O(n³)`

**Space Complexity:** `O(1)`

The space analysis:
- The `possibleCenters` function creates a list with at most 2 coordinate pairs: `O(1)`
- The `countDarts` function uses only a counter variable: `O(1)` 
- No recursive calls or data structures that scale with input size
- Only a constant amount of variables (`max_darts`, `centers`, loop indices, temporary calculations) are used
- Overall auxiliary space: `O(1)`

Note: The input array `darts` is not counted in space complexity as it's part of the input.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Floating-Point Precision Errors**

**The Problem:**
When checking if a dart is within the circle, direct floating-point comparison can fail due to precision errors. A dart that should be exactly on the boundary might be calculated as slightly outside due to accumulated floating-point arithmetic errors.

```python
# Problematic code:
if distance <= r:  # May incorrectly exclude boundary points
    count += 1
```

**The Solution:**
Add a small epsilon value to account for floating-point imprecision:

```python
# Correct approach:
if distance <= r + 1e-7:  # Small epsilon handles precision issues
    count += 1
```

### 2. **Division by Zero When Points Coincide**

**The Problem:**
When two darts are at the exact same position (distance = 0), the normalization step divides by zero:

```python
# This causes division by zero when distance_between_points = 0:
offset_x = dist_to_center * dy / distance_between_points
offset_y = dist_to_center * (-dx) / distance_between_points
```

**The Solution:**
Check for coincident points before calculating centers:

```python
def find_circle_centers(x1, y1, x2, y2):
    dx = x2 - x1
    dy = y2 - y1
    distance_between_points = sqrt(dx * dx + dy * dy)
  
    # Handle coincident points
    if distance_between_points < 1e-9:
        # Return the point itself as the only center
        return [(x1, y1)]
  
    # Continue with normal calculation...
    if distance_between_points > 2 * r:
        return []
```

### 3. **Missing Edge Case: Single Dart Coverage**

**The Problem:**
The algorithm only checks circles passing through pairs of darts, but might miss the optimal placement when the best solution is a circle centered directly on a single dart (with no other darts on the boundary).

**The Solution:**
Additionally check circles centered at each dart position:

```python
# Add this to the main algorithm:
for i in range(n):
    # Check circle centered at dart i
    count = count_darts_in_circle(darts[i][0], darts[i][1])
    max_darts_in_circle = max(max_darts_in_circle, count)
```

### 4. **Numerical Instability in Square Root Calculation**

**The Problem:**
When two points are very close to being exactly 2r apart, the expression `r * r - (distance_between_points / 2) * (distance_between_points / 2)` can become slightly negative due to floating-point errors, causing `sqrt()` to fail.

**The Solution:**
Clamp the value to ensure it's non-negative:

```python
# Instead of:
dist_to_center = sqrt(r * r - (distance_between_points / 2) * (distance_between_points / 2))

# Use:
squared_dist = r * r - (distance_between_points / 2) * (distance_between_points / 2)
dist_to_center = sqrt(max(0, squared_dist))  # Clamp to non-negative
```

### 5. **Not Handling the Case Where All Darts Are Far Apart**

**The Problem:**
When all darts are more than 2r apart from each other, no pair will generate valid circle centers, but the algorithm should still return 1 (covering at least one dart).

**The Solution:**
Initialize `max_darts_in_circle = 1` at the start, which handles this case automatically, or explicitly check circles centered at each individual dart as mentioned in pitfall #3.

