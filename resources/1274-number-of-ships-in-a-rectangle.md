# 1274. Number of Ships in a Rectangle

> Source: https://algo.monster/liteproblems/1274

## Problem Statement

This is an interactive problem where you need to count the number of ships in a rectangular area on a 2D plane without knowing their exact positions.

You're given:
- A rectangular search area defined by two points: `topRight` (upper-right corner) and `bottomLeft` (lower-left corner)
- An API function `Sea.hasShips(topRight, bottomLeft)` that returns `true` if there's at least one ship within the specified rectangle (including boundaries), and `false` otherwise
- Each ship occupies exactly one integer coordinate point
- At most one ship can exist at any integer point
- There are at most 10 ships total in the search area

Your task is to determine the exact count of ships within the given rectangle using only the `hasShips` API function. You cannot directly access the ship positions - you must work "blindfolded" by making strategic calls to `hasShips`.

**Key Constraints:**
- You can make at most 400 calls to `hasShips`
- Coordinates range from `0` to `1000`
- The solution uses a divide-and-conquer approach: recursively split the search area into four quadrants, check each quadrant for ships using `hasShips`, and only continue searching quadrants that contain ships
- When a region is reduced to a single point (`x1 == x2` and `y1 == y2`) and contains a ship, count it as 1
- The algorithm efficiently prunes empty regions to minimize API calls while ensuring all ships are counted

The challenge is to design an efficient search strategy that finds all ships within the API call limit, using the fact that there are at most 10 ships to optimize the search pattern.

## Intuition

The key insight is that we need to efficiently search a potentially large area (up to `1000 x 1000` grid) for at most 10 ships, using a limited number of API calls (400). If we checked each point individually, we'd need up to 1 million calls, which is far too many.

The breakthrough comes from realizing we can use a **divide-and-conquer** strategy similar to binary search, but in 2D space. The `hasShips` API tells us if *any* ships exist in a rectangle, which we can use to intelligently prune our search space.

Here's the thought process:

1. **Start with the whole rectangle** - if `hasShips` returns `false`, we immediately know there are 0 ships and we're done with just 1 API call.

2. **If ships exist, divide the rectangle into smaller parts** - but how should we divide? The natural choice is to split into 4 quadrants by dividing both horizontally and vertically at the midpoints. This gives us four sub-rectangles of roughly equal size.

3. **Recursively search each quadrant** - for each of the 4 quadrants, we call `hasShips`. If a quadrant has no ships, we skip it entirely (pruning that branch). If it has ships, we recursively divide it further.

4. **Base case: single point** - when we've narrowed down to a single coordinate (`x1 == x2` and `y1 == y2`), if `hasShips` returns `true`, we've found exactly 1 ship.

The efficiency comes from the fact that with at most 10 ships, most quadrants will be empty and get pruned early. Each level of recursion divides the area by 4, so we quickly zoom in on the actual ship locations. The worst case would be if ships are spread out maximally, but even then, with only 10 ships, we won't need many recursive calls to isolate each one.

This approach naturally balances between:
- **Broad coverage** - we start by checking large areas
- **Focused search** - we only drill down into areas that actually contain ships
- **Guaranteed completeness** - we won't miss any ships since we systematically cover the entire area

**Learn more about [Divide and Conquer](/problems/divide_and_conquer_intro) patterns.**

## Solution Approach

The solution implements a recursive divide-and-conquer algorithm that systematically narrows down the search space to locate all ships.

**Implementation Details:**

1. **Main Function Structure:**
   - The `countShips` method initiates the search by calling a helper function `dfs` with the initial rectangle boundaries
   - The `dfs` function handles the recursive logic and returns the count of ships in its assigned rectangle

2. **Recursive Function `dfs(topRight, bottomLeft)`:**

   First, extract the coordinates for cleaner code:
   ```python
   x1, y1 = bottomLeft.x, bottomLeft.y
   x2, y2 = topRight.x, topRight.y
   ```

3. **Boundary Validation:**
   ```python
   if x1 > x2 or y1 > y2:
       return 0
   ```
   This handles invalid rectangles where the bottom-left is actually to the right or above the top-right.

4. **Early Termination (Pruning):**
   ```python
   if not sea.hasShips(topRight, bottomLeft):
       return 0
   ```
   If the current rectangle contains no ships, immediately return 0. This is crucial for efficiency - it prevents unnecessary subdivision of empty areas.

5. **Base Case - Single Point:**
   ```python
   if x1 == x2 and y1 == y2:
       return 1
   ```
   When the rectangle is reduced to a single point and we know it contains a ship (from the previous `hasShips` check), we've found exactly one ship.

6. **Divide into Four Quadrants:**

   Calculate the midpoints:
   ```python
   midx = (x1 + x2) >> 1  # Equivalent to (x1 + x2) // 2
   midy = (y1 + y2) >> 1
   ```

   The `>> 1` is a bitwise right shift, efficiently dividing by 2.

7. **Recursively Search Each Quadrant:**

   The rectangle is divided into four parts:
   ```python
   a = dfs(topRight, Point(midx + 1, midy + 1))          # Top-right quadrant
   b = dfs(Point(midx, y2), Point(x1, midy + 1))         # Top-left quadrant
   c = dfs(Point(midx, midy), bottomLeft)                # Bottom-left quadrant
   d = dfs(Point(x2, midy), Point(midx + 1, y1))         # Bottom-right quadrant
   ```

   The quadrants are defined as:
   - **Top-right**: from `(midx + 1, midy + 1)` to `(x2, y2)`
   - **Top-left**: from `(x1, midy + 1)` to `(midx, y2)`
   - **Bottom-left**: from `(x1, y1)` to `(midx, midy)`
   - **Bottom-right**: from `(midx + 1, y1)` to `(x2, midy)`

8. **Aggregate Results:**
   ```python
   return a + b + c + d
   ```
   The total count is the sum of ships found in all four quadrants.

**Why This Works:**

- Each recursive call either finds ships in a smaller area or terminates early if no ships exist
- The search space reduces exponentially with each level (by a factor of 4)
- With at most 10 ships, most branches get pruned early, keeping the number of API calls well below 400
- The algorithm guarantees finding all ships since it systematically covers every point in the original rectangle

**Time Complexity Analysis:**
- In the worst case with ships spread out, we might need `O(ships * log(area))` API calls
- With 10 ships and a `1000 x 1000` grid, this stays well within the 400-call limit

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach. Imagine we have a 4×4 grid (coordinates from 0 to 3) with 3 ships located at positions (1,1), (3,0), and (3,3).

**Initial Call:**
- `dfs(topRight=(3,3), bottomLeft=(0,0))`
- `hasShips((3,3), (0,0))` returns `true` (there are ships in the area)
- Not a single point, so we divide into quadrants
- Calculate midpoints: `midx = 1, midy = 1`

**Level 1 - Four Quadrants:**

1. **Top-right quadrant:** `dfs((3,3), (2,2))`
   - `hasShips((3,3), (2,2))` returns `true` (ship at (3,3) is here)
   - Not a single point, divide further with `midx=2, midy=2`
   - Sub-quadrants:
     - `dfs((3,3), (3,3))`: Single point, `hasShips` returns `true` → **Count = 1**
     - `dfs((2,3), (2,3))`: Single point, `hasShips` returns `false` → Count = 0
     - `dfs((2,2), (2,2))`: Single point, `hasShips` returns `false` → Count = 0
     - `dfs((3,2), (3,2))`: Single point, `hasShips` returns `false` → Count = 0
   - Total for this quadrant: **1**

2. **Top-left quadrant:** `dfs((1,3), (0,2))`
   - `hasShips((1,3), (0,2))` returns `false` (no ships here)
   - Early termination → **Count = 0**

3. **Bottom-left quadrant:** `dfs((1,1), (0,0))`
   - `hasShips((1,1), (0,0))` returns `true` (ship at (1,1) is here)
   - Not a single point, divide with `midx=0, midy=0`
   - Sub-quadrants:
     - `dfs((1,1), (1,1))`: Single point, `hasShips` returns `true` → **Count = 1**
     - `dfs((0,1), (0,1))`: Single point, `hasShips` returns `false` → Count = 0
     - `dfs((0,0), (0,0))`: Single point, `hasShips` returns `false` → Count = 0
     - `dfs((1,0), (1,0))`: Single point, `hasShips` returns `false` → Count = 0
   - Total for this quadrant: **1**

4. **Bottom-right quadrant:** `dfs((3,1), (2,0))`
   - `hasShips((3,1), (2,0))` returns `true` (ship at (3,0) is here)
   - Not a single point, divide with `midx=2, midy=0`
   - Sub-quadrants:
     - `dfs((3,1), (3,1))`: Single point, `hasShips` returns `false` → Count = 0
     - `dfs((2,1), (2,1))`: Single point, `hasShips` returns `false` → Count = 0
     - `dfs((2,0), (2,0))`: Single point, `hasShips` returns `false` → Count = 0
     - `dfs((3,0), (3,0))`: Single point, `hasShips` returns `true` → **Count = 1**
   - Total for this quadrant: **1**

**Final Result:** 1 + 0 + 1 + 1 = **3 ships found**

**Key Observations:**
- The top-left quadrant was pruned immediately (1 API call saved multiple recursive calls)
- Each ship was eventually isolated to a single point through recursive subdivision
- The algorithm made approximately 20 API calls for this example, well below the 400 limit
- Empty regions were efficiently skipped, focusing the search on areas with ships

## Python Solution

```python
# """
# This is Sea's API interface.
# You should not implement it, or speculate about its implementation
# """
# class Sea:
#    def hasShips(self, topRight: 'Point', bottomLeft: 'Point') -> bool:
#
# class Point:
#     def __init__(self, x: int, y: int):
#         self.x = x
#         self.y = y


class Solution:
    def countShips(self, sea: "Sea", topRight: "Point", bottomLeft: "Point") -> int:
        """
        Count the number of ships in the given rectangle area using divide and conquer.

        Args:
            sea: Sea object with hasShips API to check if ships exist in a rectangle
            topRight: Top-right corner of the search area
            bottomLeft: Bottom-left corner of the search area

        Returns:
            Total number of ships in the given area
        """

        def search_area(top_right: "Point", bottom_left: "Point") -> int:
            """
            Recursively search for ships in the given rectangular area.
            Uses divide and conquer by splitting the area into 4 quadrants.

            Args:
                top_right: Top-right corner of current search area
                bottom_left: Bottom-left corner of current search area

            Returns:
                Number of ships found in the current area
            """
            # Extract coordinates for clarity
            left_x, bottom_y = bottom_left.x, bottom_left.y
            right_x, top_y = top_right.x, top_right.y

            # Check if the rectangle is valid (not inverted)
            if left_x > right_x or bottom_y > top_y:
                return 0

            # Early termination: no ships in this area
            if not sea.hasShips(top_right, bottom_left):
                return 0

            # Base case: single point, contains exactly one ship
            if left_x == right_x and bottom_y == top_y:
                return 1

            # Calculate midpoints to divide the area into quadrants
            mid_x = (left_x + right_x) >> 1  # Bitwise right shift for integer division by 2
            mid_y = (bottom_y + top_y) >> 1

            # Recursively search all four quadrants
            # Top-right quadrant
            top_right_quadrant = search_area(
                top_right,
                Point(mid_x + 1, mid_y + 1)
            )

            # Top-left quadrant
            top_left_quadrant = search_area(
                Point(mid_x, top_y),
                Point(left_x, mid_y + 1)
            )

            # Bottom-left quadrant
            bottom_left_quadrant = search_area(
                Point(mid_x, mid_y),
                bottom_left
            )

            # Bottom-right quadrant
            bottom_right_quadrant = search_area(
                Point(right_x, mid_y),
                Point(mid_x + 1, bottom_y)
            )

            # Return total count from all quadrants
            return (top_right_quadrant + top_left_quadrant +
                   bottom_left_quadrant + bottom_right_quadrant)

        # Start the recursive search from the entire area
        return search_area(topRight, bottomLeft)
```

## Time and Space Complexity

**Time Complexity:** `O(C × log(max(m, n)))`

The algorithm uses a divide-and-conquer approach by recursively dividing the search area into 4 quadrants. The key insights for time complexity analysis are:

1. When there are no ships in a region (`hasShips` returns false), that branch terminates immediately with `O(1)` work.

2. When a region contains ships, it's divided into 4 smaller regions. The division continues until we reach individual cells (where `x1 == x2` and `y1 == y2`).

3. The depth of recursion is `O(log(max(m, n)))` because we're halving the larger dimension at each level.

4. For each ship, the algorithm explores a path from the root to a leaf node containing that ship. While multiple ships might share parts of their paths, each ship contributes at most `O(log(max(m, n)))` nodes to the recursion tree.

5. Since we have `C` ships total, and each contributes `O(log(max(m, n)))` work, the total time complexity is `O(C × log(max(m, n)))`.

**Space Complexity:** `O(log(max(m, n)))`

The space complexity is determined by the maximum depth of the recursion stack. Since we divide the search space in half at each recursive level (using `midx` and `midy`), the maximum recursion depth is `O(log(max(m, n)))`, where `m` and `n` represent the dimensions of the initial rectangle. This occurs when we need to drill down from the largest dimension to a single cell.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Quadrant Boundaries**
One of the most common mistakes is incorrectly defining the boundaries when dividing the rectangle into quadrants. This can lead to:
- **Overlapping regions**: Counting the same ship multiple times
- **Gaps between quadrants**: Missing ships that fall in the gaps

**Example of incorrect division:**
```python
# WRONG - Creates overlapping quadrants at the midpoint
top_right_quadrant = search_area(top_right, Point(mid_x, mid_y))
top_left_quadrant = search_area(Point(mid_x, top_y), Point(left_x, mid_y))
# Ships at (mid_x, mid_y) would be counted multiple times!
```

**Correct approach:**
```python
# Ensure no overlap by using mid_x + 1 and mid_y + 1 for appropriate quadrants
top_right_quadrant = search_area(top_right, Point(mid_x + 1, mid_y + 1))
top_left_quadrant = search_area(Point(mid_x, top_y), Point(left_x, mid_y + 1))
bottom_left_quadrant = search_area(Point(mid_x, mid_y), bottom_left)
bottom_right_quadrant = search_area(Point(right_x, mid_y), Point(mid_x + 1, bottom_y))
```

### 2. **Missing the Early Termination Check**
Forgetting to check `hasShips()` before recursing deeper wastes API calls and can exceed the 400-call limit.

**Incorrect implementation:**
```python
def search_area(top_right, bottom_left):
    # WRONG - No early termination check
    if left_x == right_x and bottom_y == top_y:
        # This might return 1 even for empty points!
        return 1 if sea.hasShips(top_right, bottom_left) else 0

    # Continues to divide even for empty areas
    mid_x = (left_x + right_x) >> 1
    mid_y = (bottom_y + top_y) >> 1
    # ... recursive calls
```

**Correct approach:**
```python
def search_area(top_right, bottom_left):
    # Check for ships FIRST
    if not sea.hasShips(top_right, bottom_left):
        return 0  # Don't recurse into empty areas

    # Only then check if it's a single point
    if left_x == right_x and bottom_y == top_y:
        return 1  # We know there's a ship here from the check above
```

### 3. **Incorrect Base Case Logic**
The base case must verify that we've reached a single point AND that point contains a ship (already verified by the early termination check).

**Common mistake:**
```python
# WRONG - Doesn't ensure we're at a single point
if sea.hasShips(top_right, bottom_left):
    return 1  # This could be a large area with multiple ships!
```

**Correct approach:**
```python
# First check if area has ships (done earlier)
# Then check if it's a single point
if left_x == right_x and bottom_y == top_y:
    return 1  # Single point with a ship
```

### 4. **Integer Division Pitfall**
When calculating midpoints, ensure proper integer division to avoid floating-point coordinates.

**Potential issue:**
```python
# In Python 3, this returns a float if not careful
mid_x = (left_x + right_x) / 2  # WRONG - returns float
```

**Correct approaches:**
```python
mid_x = (left_x + right_x) // 2  # Integer division
# OR
mid_x = (left_x + right_x) >> 1  # Bitwise shift (faster)
```

### 5. **Not Handling Edge Cases**
Failing to handle invalid rectangles where bottom_left is actually above or to the right of top_right.

**Missing validation:**
```python
def search_area(top_right, bottom_left):
    # WRONG - No validation
    if not sea.hasShips(top_right, bottom_left):
        return 0
    # This might crash or behave unexpectedly for invalid rectangles
```

**Correct approach:**
```python
def search_area(top_right, bottom_left):
    # Validate rectangle first
    if left_x > right_x or bottom_y > top_y:
        return 0  # Invalid rectangle

    if not sea.hasShips(top_right, bottom_left):
        return 0
```

### 6. **Order of Operations in Base Case**
The order of checks matters for efficiency and correctness.

**Inefficient ordering:**
```python
# WRONG ORDER - Checks single point before checking if ships exist
if left_x == right_x and bottom_y == top_y:
    return 1 if sea.hasShips(top_right, bottom_left) else 0
    # Wastes an API call for the base case
```

**Optimal ordering:**
```python
# Check for ships first (prunes empty branches)
if not sea.hasShips(top_right, bottom_left):
    return 0

# Then check if single point (no additional API call needed)
if left_x == right_x and bottom_y == top_y:
    return 1
```

By avoiding these pitfalls, the solution efficiently finds all ships while minimizing API calls and ensuring correctness.

