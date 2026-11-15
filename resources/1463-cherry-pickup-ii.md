# 1463. Cherry Pickup II

> Source: https://algo.monster/liteproblems/1463

## Problem Statement

You have a `rows x cols` matrix `grid` representing a field of cherries, where `grid[i][j]` represents the number of cherries at cell `(i, j)`.

Two robots will collect cherries from this field:
- **Robot #1** starts at the top-left corner `(0, 0)`
- **Robot #2** starts at the top-right corner `(0, cols - 1)`

Both robots move simultaneously from the top row to the bottom row following these rules:

**Movement Rules:**
- From any cell `(i, j)`, a robot can move to one of three cells in the next row:
  - `(i + 1, j - 1)` - diagonally left
  - `(i + 1, j)` - straight down  
  - `(i + 1, j + 1)` - diagonally right
- Robots cannot move outside the grid boundaries
- Both robots must reach the bottom row

**Cherry Collection Rules:**
- When a robot passes through a cell, it collects all cherries from that cell
- After collection, the cell becomes empty (0 cherries)
- If both robots occupy the same cell simultaneously, only one robot collects the cherries from that cell

The goal is to find the maximum total number of cherries that both robots can collect together by choosing optimal paths from the top row to the bottom row.

For example, if Robot #1 collects cherries from certain cells and Robot #2 collects from different cells (except when they're at the same position), the total collection is the sum of all cherries collected by both robots throughout their journey.

## Intuition

The key insight is that both robots move simultaneously row by row, so we can think of their movement as a synchronized process. At any given moment, both robots will be in the same row, just at different column positions.

Since both robots start at row 0 and must reach the bottom row, and they can only move down (to the next row) with each step, we can process this problem row by row. This naturally leads to a [dynamic programming](/problems/dynamic_programming_intro) approach where we track the state of both robots together.

The state we need to track is: "What's the maximum cherries we can collect when robot 1 is at position `j1` and robot 2 is at position `j2` in row `i`?" This gives us a 3-dimensional DP state: `f[i][j1][j2]`.

Why do we need to track both robots' positions simultaneously? Because when they land on the same cell, only one can collect the cherries. If we tried to find optimal paths for each robot independently, we'd incorrectly count cherries twice when their paths overlap.

For each state `f[i][j1][j2]`, we need to consider where the robots could have come from in the previous row. Since each robot can move to three possible positions (left diagonal, straight down, right diagonal), and we have two robots, there are `3 × 3 = 9` possible previous states to consider.

The transition is straightforward: 
- If the robots are at different positions (`j1 ≠ j2`), they collect `grid[i][j1] + grid[i][j2]` cherries
- If they're at the same position (`j1 = j2`), they only collect `grid[i][j1]` cherries once

By building up the solution row by row and considering all possible movements, we ensure we find the optimal path for both robots that maximizes the total cherry collection.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

We implement a 3D [dynamic programming](/problems/dynamic_programming_intro) solution where `f[i][j1][j2]` represents the maximum cherries collected when robot 1 is at column `j1` and robot 2 is at column `j2` in row `i`.

**Initialization:**
- Create a 3D array `f` with dimensions `[m][n][n]` initialized to `-1`, where `-1` indicates an unreachable or unvisited state
- Set `f[0][0][n-1] = grid[0][0] + grid[0][n-1]` as the starting position (robot 1 at top-left, robot 2 at top-right)

**State Transition:**
For each row `i` from 1 to `m-1`, and for each possible position combination `(j1, j2)` where both robots can be:

1. Calculate the cherries collected at current positions:
   ```python
   x = grid[i][j1] + (0 if j1 == j2 else grid[i][j2])
   ```
   This handles the case where both robots are at the same cell (collect cherries only once).

2. Check all possible previous positions:
   - Robot 1 could have come from columns: `y1 ∈ {j1-1, j1, j1+1}`
   - Robot 2 could have come from columns: `y2 ∈ {j2-1, j2, j2+1}`

3. For each valid previous state where `f[i-1][y1][y2] != -1`:
   ```python
   f[i][j1][j2] = max(f[i][j1][j2], f[i-1][y1][y2] + x)
   ```

The state transition formula is:
```
f[i][j1][j2] = max(f[i-1][y1][y2] + cherries_at_current_positions)
```
where `y1 ∈ {j1-1, j1, j1+1}` and `y2 ∈ {j2-1, j2, j2+1}` are valid positions within grid bounds.

**Final Answer:**
After filling the entire DP table, the answer is the maximum value among all possible ending positions in the last row:
```python
max(f[m-1][j1][j2] for all valid j1, j2)
```

The time complexity is `O(m × n² × 9) = O(m × n²)` since for each of the `m × n²` states, we check at most 9 previous states. The space complexity is `O(m × n²)` for the 3D DP array.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider a 3×3 grid:
```
grid = [[3, 1, 1],
        [2, 5, 1],
        [1, 5, 5]]
```

**Initial Setup:**
- Robot 1 starts at (0, 0) with 3 cherries
- Robot 2 starts at (0, 2) with 1 cherry
- Initialize `f[0][0][2] = 3 + 1 = 4`

**Row 0 → Row 1 Transition:**

From `f[0][0][2] = 4`, we explore where both robots can move:
- Robot 1 from column 0 can go to columns: 0 or 1
- Robot 2 from column 2 can go to columns: 1 or 2

Possible transitions:
1. `f[1][0][1]`: Robot 1 at (1,0), Robot 2 at (1,1)
   - Cherries collected: 2 + 5 = 7
   - Total: 4 + 7 = 11

2. `f[1][0][2]`: Robot 1 at (1,0), Robot 2 at (1,2)
   - Cherries collected: 2 + 1 = 3
   - Total: 4 + 3 = 7

3. `f[1][1][1]`: Robot 1 at (1,1), Robot 2 at (1,1) - same cell!
   - Cherries collected: 5 (only once, not 5+5)
   - Total: 4 + 5 = 9

4. `f[1][1][2]`: Robot 1 at (1,1), Robot 2 at (1,2)
   - Cherries collected: 5 + 1 = 6
   - Total: 4 + 6 = 10

After row 1: `f[1][0][1] = 11`, `f[1][0][2] = 7`, `f[1][1][1] = 9`, `f[1][1][2] = 10`

**Row 1 → Row 2 Transition:**

From `f[1][0][1] = 11`:
- Robot 1 from column 0 → columns 0 or 1
- Robot 2 from column 1 → columns 0, 1, or 2

Key transitions:
- `f[2][0][2]`: 11 + (1 + 5) = 17
- `f[2][1][2]`: 11 + (5 + 5) = 21

From `f[1][1][2] = 10`:
- Robot 1 from column 1 → columns 0, 1, or 2
- Robot 2 from column 2 → columns 1 or 2

Key transitions:
- `f[2][0][2]`: max(17, 10 + (1 + 5)) = 17
- `f[2][1][2]`: max(21, 10 + (5 + 5)) = 21
- `f[2][2][2]`: 10 + 5 = 15 (same cell, collect once)

**Final Answer:**
Maximum among all `f[2][j1][j2]` values = 21

The optimal paths are:
- Robot 1: (0,0) → (1,0) → (2,1), collecting 3 + 2 + 5 = 10
- Robot 2: (0,2) → (1,1) → (2,2), collecting 1 + 5 + 5 = 11
- Total: 21 cherries

## Python Solution

```python
class Solution:
    def cherryPickup(self, grid: List[List[int]]) -> int:
        # Get dimensions of the grid
        rows, cols = len(grid), len(grid[0])
      
        # Initialize 3D DP table: dp[row][robot1_col][robot2_col]
        # dp[i][j1][j2] represents max cherries collected when:
        # - Both robots are at row i
        # - Robot 1 is at column j1, Robot 2 is at column j2
        dp = [[[-1] * cols for _ in range(cols)] for _ in range(rows)]
      
        # Base case: robots start at top row
        # Robot 1 starts at column 0, Robot 2 starts at last column
        dp[0][0][cols - 1] = grid[0][0] + grid[0][cols - 1]
      
        # Process each row from top to bottom
        for row in range(1, rows):
            # Try all possible positions for both robots in current row
            for col1 in range(cols):
                for col2 in range(cols):
                    # Calculate cherries collected at current positions
                    # If robots are at same position, only count cherries once
                    current_cherries = grid[row][col1]
                    if col1 != col2:
                        current_cherries += grid[row][col2]
                  
                    # Check all possible previous positions for both robots
                    # Each robot can move diagonally left, straight down, or diagonally right
                    for prev_col1 in range(col1 - 1, col1 + 2):
                        for prev_col2 in range(col2 - 1, col2 + 2):
                            # Validate previous positions are within bounds
                            if (0 <= prev_col1 < cols and 
                                0 <= prev_col2 < cols and 
                                dp[row - 1][prev_col1][prev_col2] != -1):
                                # Update maximum cherries for current state
                                dp[row][col1][col2] = max(
                                    dp[row][col1][col2],
                                    dp[row - 1][prev_col1][prev_col2] + current_cherries
                                )
      
        # Find maximum cherries from all possible ending positions
        from itertools import product
        max_cherries = max(
            dp[-1][col1][col2] 
            for col1, col2 in product(range(cols), range(cols))
            if dp[-1][col1][col2] != -1
        )
      
        return max_cherries
```

## Time and Space Complexity

**Time Complexity:** `O(m × n²)`

The algorithm uses dynamic programming with three nested loops:
- The outermost loop iterates through all rows: `O(m)`
- The second and third loops iterate through all possible positions for robot 1 and robot 2 in each row: `O(n²)`
- Inside these loops, there are two more nested loops checking previous positions (up to 3 positions each for both robots), but these are constant operations: `O(3 × 3) = O(9) = O(1)`
- The final operation finds the maximum value in the last row, which requires checking all `n²` combinations: `O(n²)`

Therefore, the overall time complexity is `O(m × n² × 1) + O(n²) = O(m × n²)`

**Space Complexity:** `O(m × n²)`

The algorithm creates a 3D DP array `f` with dimensions:
- First dimension (rows): `m`
- Second dimension (robot 1 column positions): `n`
- Third dimension (robot 2 column positions): `n`

This results in a total space requirement of `m × n × n = m × n²`

Therefore, the space complexity is `O(m × n²)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of Same Cell Occupation**

**Pitfall:** A common mistake is double-counting cherries when both robots occupy the same cell. Many solutions incorrectly add `grid[row][col1] + grid[row][col2]` without checking if `col1 == col2`.

**Wrong approach:**
```python
# This always adds both values, causing double-counting
current_cherries = grid[row][col1] + grid[row][col2]
```

**Correct approach:**
```python
# Only count cherries once when robots are at the same position
current_cherries = grid[row][col1]
if col1 != col2:
    current_cherries += grid[row][col2]
```

### 2. **Inefficient State Representation**

**Pitfall:** Using a 4D DP array `dp[row1][col1][row2][col2]` to track both robots' positions independently. Since both robots move simultaneously row by row, they're always in the same row.

**Wrong approach:**
```python
# Unnecessary 4D array - wastes memory and complicates logic
dp = [[[[-1] * cols for _ in range(rows)] for _ in range(cols)] for _ in range(rows)]
```

**Correct approach:**
```python
# 3D array is sufficient since row1 == row2 always
dp = [[[-1] * cols for _ in range(cols)] for _ in range(rows)]
```

### 3. **Boundary Check Errors**

**Pitfall:** Forgetting to validate that previous positions are within grid boundaries when checking state transitions, leading to index out of bounds errors.

**Wrong approach:**
```python
# No boundary checking - will crash if prev_col < 0 or prev_col >= cols
for prev_col1 in [col1 - 1, col1, col1 + 1]:
    for prev_col2 in [col2 - 1, col2, col2 + 1]:
        if dp[row - 1][prev_col1][prev_col2] != -1:  # IndexError possible!
            # update logic
```

**Correct approach:**
```python
for prev_col1 in range(col1 - 1, col1 + 2):
    for prev_col2 in range(col2 - 1, col2 + 2):
        if (0 <= prev_col1 < cols and 
            0 <= prev_col2 < cols and 
            dp[row - 1][prev_col1][prev_col2] != -1):
            # update logic
```

### 4. **Incorrect Base Case Initialization**

**Pitfall:** Initializing the wrong starting positions or forgetting that robots might start at the same cell in edge cases (when grid has only 1 column).

**Wrong approach:**
```python
# Doesn't handle single column grid correctly
dp[0][0][cols - 1] = grid[0][0] + grid[0][cols - 1]
```

**Correct approach:**
```python
# Handle the case where grid has only one column
if cols == 1:
    dp[0][0][0] = grid[0][0]  # Both robots at same cell
else:
    dp[0][0][cols - 1] = grid[0][0] + grid[0][cols - 1]
```

### 5. **Finding Maximum in Final Row**

**Pitfall:** Only checking `dp[rows-1][0][cols-1]` as the final answer, assuming robots must end at specific positions. The robots can end at any valid column positions in the last row.

**Wrong approach:**
```python
# Assumes robots must end at specific positions
return dp[rows - 1][0][cols - 1]
```

**Correct approach:**
```python
# Check all possible ending positions
max_cherries = 0
for col1 in range(cols):
    for col2 in range(cols):
        if dp[rows - 1][col1][col2] != -1:
            max_cherries = max(max_cherries, dp[rows - 1][col1][col2])
return max_cherries
```

