# 3197. Find the Minimum Area to Cover All Ones II

> Source: https://algo.monster/liteproblems/3197

## Problem Statement

You have a 2D binary array `grid` containing only 0s and 1s. Your task is to find 3 non-overlapping rectangles that cover all the 1s in the grid. These rectangles must have non-zero areas and their sides must be horizontal and vertical (aligned with the grid axes).

The goal is to minimize the total sum of the areas of these three rectangles. The rectangles can touch each other at their boundaries but cannot overlap.

Each rectangle's area is calculated as `(height) × (width)`, where the height and width are determined by the minimum bounding box that contains all the 1s assigned to that rectangle.

The solution approach involves trying different ways to partition the grid into three regions:
- Using two horizontal dividing lines to create three horizontal strips
- Using two vertical dividing lines to create three vertical strips  
- Using one horizontal line followed by one vertical line (creating an L-shape and two rectangles)
- Using one vertical line followed by one horizontal line (creating different L-shape configurations)

For each partition configuration, the algorithm calculates the minimum bounding rectangle for the 1s in each region using the helper function `f(i1, j1, i2, j2)`, which finds the tightest rectangle containing all 1s in the subgrid from position `(i1, j1)` to `(i2, j2)`.

The answer is the minimum sum across all possible valid partitions of the grid into three rectangles.

## Intuition

The key insight is that to divide a grid into exactly 3 non-overlapping rectangles, we need exactly 2 cuts. These cuts can be made in different patterns, and we want to find the pattern that minimizes the total area.

Think about it geometrically - when we need to split a rectangle into 3 parts, there are only a limited number of ways to do this:
1. Make two parallel cuts (either both horizontal or both vertical), creating 3 strips
2. Make one cut in one direction, then make another cut perpendicular to the first one, but only through one of the resulting pieces

Why does this work? Because once we assign each `1` in the grid to one of the three rectangles, each rectangle needs to be sized just large enough to contain all its assigned `1`s. The minimum bounding box for a set of points is uniquely determined - it's the smallest rectangle with sides parallel to the axes that contains all the points.

The challenge is that we don't know in advance which `1`s should belong to which rectangle. However, by trying all possible ways to partition the grid into three regions using straight cuts, we guarantee that we'll find the optimal assignment. Each partition uniquely determines which `1`s belong to which rectangle.

For each partition pattern, we enumerate all possible positions for the cuts. For example, with two horizontal cuts, we try all pairs of row indices `(i1, i2)` where `i1 < i2`. This ensures we don't miss the optimal solution.

The function `f(i1, j1, i2, j2)` efficiently computes the area of the minimum bounding rectangle for all `1`s in a given subgrid by finding the minimum and maximum row and column indices containing a `1`, then calculating `(x2 - x1 + 1) * (y2 - y1 + 1)`.

By exhaustively checking all 6 partition patterns and all possible cut positions within each pattern, we're guaranteed to find the configuration that minimizes the sum of the three rectangular areas.

## Solution Approach

The solution implements a brute-force approach that systematically tries all possible ways to partition the grid into three rectangles.

**Helper Function `f(i1, j1, i2, j2)`:**
This function calculates the minimum bounding rectangle area for all `1`s in the subgrid from `(i1, j1)` to `(i2, j2)`:
- Initialize bounds: `x1 = y1 = inf` (minimum coordinates) and `x2 = y2 = -inf` (maximum coordinates)
- Scan through the subgrid to find all cells containing `1`
- Update the bounds to track the minimum and maximum row/column indices where `1`s appear
- Return the area as `(x2 - x1 + 1) * (y2 - y1 + 1)`

**Main Algorithm:**
The solution enumerates all 6 partition patterns:

1. **Two Horizontal Splits:** 
   - Use two row indices `i1` and `i2` where `i1 < i2`
   - Create three horizontal strips: `[0, i1]`, `[i1+1, i2]`, `[i2+1, m-1]`
   - Calculate: `f(0, 0, i1, n-1) + f(i1+1, 0, i2, n-1) + f(i2+1, 0, m-1, n-1)`

2. **Two Vertical Splits:**
   - Use two column indices `j1` and `j2` where `j1 < j2`
   - Create three vertical strips: `[0, j1]`, `[j1+1, j2]`, `[j2+1, n-1]`
   - Calculate: `f(0, 0, m-1, j1) + f(0, j1+1, m-1, j2) + f(0, j2+1, m-1, n-1)`

3. **L-shaped Partitions:**
   For each position `(i, j)`, try 4 different L-shaped configurations:
 
   - **Top-left L:** Split horizontally at row `i`, then split the top part vertically at column `j`
     - Rectangles: top-left `[0,0,i,j]`, top-right `[0,j+1,i,n-1]`, bottom `[i+1,0,m-1,n-1]`
 
   - **Bottom-left L:** Split horizontally at row `i`, then split the bottom part vertically at column `j`
     - Rectangles: top `[0,0,i,n-1]`, bottom-left `[i+1,0,m-1,j]`, bottom-right `[i+1,j+1,m-1,n-1]`
 
   - **Top-right L:** Split vertically at column `j`, then split the left part horizontally at row `i`
     - Rectangles: top-left `[0,0,i,j]`, bottom-left `[i+1,0,m-1,j]`, right `[0,j+1,m-1,n-1]`
 
   - **Bottom-right L:** Split vertically at column `j`, then split the right part horizontally at row `i`
     - Rectangles: left `[0,0,m-1,j]`, top-right `[0,j+1,i,n-1]`, bottom-right `[i+1,j+1,m-1,n-1]`

**Optimization:**
- Initialize `ans` with `m * n` (the maximum possible area)
- For each valid partition, calculate the sum of the three rectangle areas
- Update `ans` with the minimum sum found

The algorithm has time complexity of `O(m²n² × mn)` where the `m²n²` comes from enumerating all possible cut positions and `mn` from computing each rectangle's bounding box.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider this 4×4 grid:
```
1 1 0 0
1 0 0 0
0 0 1 1
0 0 1 0
```

We need to find 3 non-overlapping rectangles that cover all the 1s with minimum total area.

**Step 1: Try Two Horizontal Splits**

Let's try splitting at rows i1=1 and i2=2:
- Top strip (rows 0-1): Contains 1s at (0,0), (0,1), (1,0)
  - Minimum bounding box: from (0,0) to (1,1) → area = 2×2 = 4
- Middle strip (row 2): Contains 1s at (2,2), (2,3)
  - Minimum bounding box: from (2,2) to (2,3) → area = 1×2 = 2
- Bottom strip (row 3): Contains 1 at (3,2)
  - Minimum bounding box: from (3,2) to (3,2) → area = 1×1 = 1
- Total area = 4 + 2 + 1 = 7

**Step 2: Try Two Vertical Splits**

Let's try splitting at columns j1=1 and j2=2:
- Left strip (columns 0-1): Contains 1s at (0,0), (0,1), (1,0)
  - Minimum bounding box: from (0,0) to (1,1) → area = 2×2 = 4
- Middle strip (column 2): Contains 1s at (2,2), (3,2)
  - Minimum bounding box: from (2,2) to (3,2) → area = 2×1 = 2
- Right strip (column 3): Contains 1 at (2,3)
  - Minimum bounding box: from (2,3) to (2,3) → area = 1×1 = 1
- Total area = 4 + 2 + 1 = 7

**Step 3: Try L-shaped Partition**

Let's try a top-left L configuration with horizontal split at row i=1 and vertical split at column j=1:
- Top-left rectangle (rows 0-1, columns 0-1): Contains 1s at (0,0), (0,1), (1,0)
  - Minimum bounding box: from (0,0) to (1,1) → area = 2×2 = 4
- Top-right rectangle (rows 0-1, columns 2-3): No 1s
  - No valid bounding box → area = 0 (invalid partition)
- Bottom rectangle (rows 2-3, columns 0-3): Contains 1s at (2,2), (2,3), (3,2)
  - Minimum bounding box: from (2,2) to (3,3) → area = 2×2 = 4

This partition is invalid because one rectangle contains no 1s.

**Step 4: Try Another L-shaped Partition**

Let's try a bottom-left L with horizontal split at row i=1 and vertical split at column j=1:
- Top rectangle (rows 0-1, columns 0-3): Contains 1s at (0,0), (0,1), (1,0)
  - Minimum bounding box: from (0,0) to (1,1) → area = 2×2 = 4
- Bottom-left rectangle (rows 2-3, columns 0-1): No 1s
  - Invalid partition (no 1s in this rectangle)
- Bottom-right rectangle (rows 2-3, columns 2-3): Contains 1s at (2,2), (2,3), (3,2)
  - Minimum bounding box: from (2,2) to (3,3) → area = 2×2 = 4

Also invalid.

**Step 5: Finding a Valid L-shaped Partition**

Let's try horizontal split at i=1, vertical split at j=2 in bottom-left L configuration:
- Top rectangle (rows 0-1, columns 0-3): Contains 1s at (0,0), (0,1), (1,0)
  - Minimum bounding box: from (0,0) to (1,1) → area = 2×2 = 4
- Bottom-left rectangle (rows 2-3, columns 0-2): Contains 1s at (2,2), (3,2)
  - Minimum bounding box: from (2,2) to (3,2) → area = 2×1 = 2
- Bottom-right rectangle (rows 2-3, column 3): Contains 1 at (2,3)
  - Minimum bounding box: from (2,3) to (2,3) → area = 1×1 = 1
- Total area = 4 + 2 + 1 = 7

After trying all possible partitions, the minimum total area found is 7. The algorithm returns this as the answer.

## Python Solution

```python
class Solution:
    def minimumSum(self, grid: List[List[int]]) -> int:
        def calculate_rectangle_area(row1: int, col1: int, row2: int, col2: int) -> int:
            """
            Calculate the area of the minimum bounding rectangle that contains all 1s
            in the subgrid from (row1, col1) to (row2, col2).
          
            Args:
                row1, col1: Top-left corner of the subgrid
                row2, col2: Bottom-right corner of the subgrid
          
            Returns:
                Area of the minimum bounding rectangle containing all 1s
            """
            # Initialize bounds for the minimum bounding rectangle
            min_row = float('inf')
            min_col = float('inf')
            max_row = -float('inf')
            max_col = -float('inf')
          
            # Find the bounds of all cells containing 1
            for row in range(row1, row2 + 1):
                for col in range(col1, col2 + 1):
                    if grid[row][col] == 1:
                        min_row = min(min_row, row)
                        min_col = min(min_col, col)
                        max_row = max(max_row, row)
                        max_col = max(max_col, col)
          
            # Calculate and return the area of the bounding rectangle
            return (max_row - min_row + 1) * (max_col - min_col + 1)
      
        rows, cols = len(grid), len(grid[0])
        min_total_area = rows * cols
      
        # Case 1: Three horizontal strips
        # Divide the grid into three horizontal rectangles
        for row1 in range(rows - 1):
            for row2 in range(row1 + 1, rows - 1):
                total_area = (
                    calculate_rectangle_area(0, 0, row1, cols - 1) +
                    calculate_rectangle_area(row1 + 1, 0, row2, cols - 1) +
                    calculate_rectangle_area(row2 + 1, 0, rows - 1, cols - 1)
                )
                min_total_area = min(min_total_area, total_area)
      
        # Case 2: Three vertical strips
        # Divide the grid into three vertical rectangles
        for col1 in range(cols - 1):
            for col2 in range(col1 + 1, cols - 1):
                total_area = (
                    calculate_rectangle_area(0, 0, rows - 1, col1) +
                    calculate_rectangle_area(0, col1 + 1, rows - 1, col2) +
                    calculate_rectangle_area(0, col2 + 1, rows - 1, cols - 1)
                )
                min_total_area = min(min_total_area, total_area)
      
        # Case 3-6: Mixed divisions (one horizontal and one vertical cut)
        for row in range(rows - 1):
            for col in range(cols - 1):
                # Configuration 1: Top-left, top-right, bottom
                total_area = (
                    calculate_rectangle_area(0, 0, row, col) +
                    calculate_rectangle_area(0, col + 1, row, cols - 1) +
                    calculate_rectangle_area(row + 1, 0, rows - 1, cols - 1)
                )
                min_total_area = min(min_total_area, total_area)
              
                # Configuration 2: Top, bottom-left, bottom-right
                total_area = (
                    calculate_rectangle_area(0, 0, row, cols - 1) +
                    calculate_rectangle_area(row + 1, 0, rows - 1, col) +
                    calculate_rectangle_area(row + 1, col + 1, rows - 1, cols - 1)
                )
                min_total_area = min(min_total_area, total_area)
              
                # Configuration 3: Top-left, bottom-left, right
                total_area = (
                    calculate_rectangle_area(0, 0, row, col) +
                    calculate_rectangle_area(row + 1, 0, rows - 1, col) +
                    calculate_rectangle_area(0, col + 1, rows - 1, cols - 1)
                )
                min_total_area = min(min_total_area, total_area)
              
                # Configuration 4: Left, top-right, bottom-right
                total_area = (
                    calculate_rectangle_area(0, 0, rows - 1, col) +
                    calculate_rectangle_area(0, col + 1, row, cols - 1) +
                    calculate_rectangle_area(row + 1, col + 1, rows - 1, cols - 1)
                )
                min_total_area = min(min_total_area, total_area)
      
        return min_total_area
```

## Time and Space Complexity

**Time Complexity:** `O(m^2 × n^2)`

The analysis breaks down as follows:
- The helper function `f(i1, j1, i2, j2)` iterates through a rectangular region, which takes `O(m × n)` time in the worst case.
- The code has three main sections that enumerate different ways to partition the grid:
  1. Two horizontal cuts: `O(m^2)` iterations, each calling `f` three times
  2. Two vertical cuts: `O(n^2)` iterations, each calling `f` three times  
  3. One horizontal and one vertical cut: `O(m × n)` iterations, each calling `f` three times (with 4 different partition patterns)
- Since each call to `f` takes `O(m × n)` time, and we have:
  - `O(m^2)` iterations × `O(m × n)` = `O(m^3 × n)`
  - `O(n^2)` iterations × `O(m × n)` = `O(m × n^3)`
  - `O(m × n)` iterations × `O(m × n)` = `O(m^2 × n^2)`
- The dominant term is `O(m^2 × n^2)`, which determines the overall time complexity.

**Space Complexity:** `O(1)`

The space complexity is constant because:
- The function `f` only uses a fixed number of variables (`x1`, `y1`, `x2`, `y2`)
- The main function uses a fixed number of variables (`m`, `n`, `ans`, loop indices)
- No additional data structures are created that scale with input size
- The recursion depth is 1 (no recursive calls)

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Handling Empty Rectangles**
The most critical pitfall in this solution is not handling cases where a partition region contains no 1s. When `calculate_rectangle_area` doesn't find any 1s in a subgrid, the min/max bounds remain at their initial infinity values, resulting in an invalid area calculation.

**Problem Example:**
```python
grid = [[1, 0, 0],
        [0, 0, 0],
        [0, 0, 1]]
```
If we partition this with a horizontal line after row 0 and another after row 1, the middle strip `[1, 0, 2]` contains no 1s, causing the function to return a negative or extremely large area.

**Solution:**
```python
def calculate_rectangle_area(row1: int, col1: int, row2: int, col2: int) -> int:
    min_row = float('inf')
    min_col = float('inf')
    max_row = -float('inf')
    max_col = -float('inf')
  
    has_ones = False  # Track if we found any 1s
  
    for row in range(row1, row2 + 1):
        for col in range(col1, col2 + 1):
            if grid[row][col] == 1:
                has_ones = True
                min_row = min(min_row, row)
                min_col = min(min_col, col)
                max_row = max(max_row, row)
                max_col = max(max_col, col)
  
    # Return 0 if no 1s found, as empty rectangles are invalid
    if not has_ones:
        return float('inf')  # Return infinity to exclude this partition
  
    return (max_row - min_row + 1) * (max_col - min_col + 1)
```

### 2. **Off-by-One Errors in Loop Boundaries**
Another common mistake is incorrectly setting loop boundaries when iterating through possible partition positions.

**Problem:**
```python
# Incorrect: doesn't leave space for third rectangle
for row1 in range(rows):  # Should be rows - 1
    for row2 in range(row1 + 1, rows):  # Should be rows - 1
```

**Solution:**
Ensure each partition leaves at least one row/column for each rectangle:
```python
# Correct: ensures all three rectangles have at least one row
for row1 in range(rows - 2):  # First cut can be at most at rows - 3
    for row2 in range(row1 + 1, rows - 1):  # Second cut leaves room for third rectangle
```

### 3. **Incorrect Partition Configuration Logic**
It's easy to mix up the coordinates when implementing the 6 different partition patterns, especially for L-shaped configurations.

**Problem:**
Accidentally overlapping rectangles or missing parts of the grid:
```python
# Incorrect: rectangles might overlap or miss cells
total_area = (
    calculate_rectangle_area(0, 0, row, col) +
    calculate_rectangle_area(0, col, row, cols - 1) +  # Should be col + 1
    calculate_rectangle_area(row, 0, rows - 1, cols - 1)  # Should be row + 1
)
```

**Solution:**
Carefully verify that:
- Adjacent rectangles share boundaries but don't overlap (use `col + 1` and `row + 1` for the next region)
- All cells in the grid are covered exactly once
- Draw out each configuration to visualize the partitions

### 4. **Not Validating Input Constraints**
The problem requires exactly 3 rectangles to cover all 1s. If the grid has fewer than 3 cells with 1s, the problem becomes impossible.

**Solution:**
Add validation at the beginning:
```python
def minimumSum(self, grid: List[List[int]]) -> int:
    # Count total number of 1s
    ones_count = sum(sum(row) for row in grid)
    if ones_count < 3:
        return -1  # Or handle according to problem requirements
  
    # Rest of the solution...
```

