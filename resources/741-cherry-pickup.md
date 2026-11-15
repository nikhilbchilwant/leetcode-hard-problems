# 741. Cherry Pickup

> Source: https://algo.monster/liteproblems/741

## Problem Statement

You have an `n x n` grid representing a cherry field where each cell contains one of three values:
- `0`: empty cell (passable)
- `1`: cell with a cherry (you can pick it and pass through)
- `-1`: cell with a thorn (blocks movement)

Your task is to find the maximum number of cherries you can collect following these rules:

1. Start at position `(0, 0)` and travel to position `(n-1, n-1)` by only moving right or down through valid cells (cells with value `0` or `1`)
2. From `(n-1, n-1)`, return back to `(0, 0)` by only moving left or up through valid cells
3. When you pass through a cell containing a cherry (`1`), you pick it up and the cell becomes empty (`0`)
4. If no valid path exists between `(0, 0)` and `(n-1, n-1)`, return `0`

The key insight in the solution is to reframe the problem: instead of thinking about one trip from `(0, 0)` to `(n-1, n-1)` and back, we can think of it as two people simultaneously traveling from `(0, 0)` to `(n-1, n-1)`. Both people move at the same pace (taking the same number of steps), and when they occupy the same cell, they only collect the cherry once.

The [dynamic programming](/problems/dynamic_programming_intro) approach uses `f[k][i1][i2]` to represent the maximum cherries collected when:
- Both people have taken exactly `k` steps
- Person 1 is at position `(i1, k-i1)`
- Person 2 is at position `(i2, k-i2)`

The state transition considers all valid previous positions for both people, and the final answer is `max(0, f[2n-2][n-1][n-1])`, where `2n-2` is the total number of steps needed to go from `(0, 0)` to `(n-1, n-1)`.

## Intuition

The naive approach might be to find the optimal path from `(0, 0)` to `(n-1, n-1)`, collect cherries along the way, then find the optimal return path on the modified grid. However, this greedy approach fails because the first path might block better options for the return journey.

The key insight is recognizing that going from `(0, 0)` to `(n-1, n-1)` and back is equivalent to having two people start simultaneously from `(0, 0)` and both reach `(n-1, n-1)`. Why? Because any path from start to end and back can be split into two paths from start to end - one person follows the original forward path, and another follows the return path in reverse.

Since both people move simultaneously, after `k` steps, if person 1 is at `(i1, j1)` and person 2 is at `(i2, j2)`, we know that `i1 + j1 = k` and `i2 + j2 = k`. This means we only need to track the row positions `i1` and `i2`, as the column positions can be derived: `j1 = k - i1` and `j2 = k - i2`.

This transforms our problem into a 3-dimensional DP: `f[k][i1][i2]` represents the maximum cherries collected when both people have taken `k` steps, with person 1 at row `i1` and person 2 at row `i2`.

At each step, both people can move either right or down (equivalent to coming from left or up in the previous step). This gives us four possible combinations of previous positions to consider. When both people land on the same cell, we count that cherry only once.

The total number of steps to reach `(n-1, n-1)` from `(0, 0)` is `2n-2` (we need to move `n-1` steps right and `n-1` steps down). The final answer is the maximum cherries collected when both people reach `(n-1, n-1)` after `2n-2` steps.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The implementation uses [dynamic programming](/problems/dynamic_programming_intro) with three dimensions. Let's walk through the key components:

**State Definition:**
- `f[k][i1][i2]` represents the maximum cherries collected when both persons have taken `k` steps total
- Person 1 is at position `(i1, k-i1)` 
- Person 2 is at position `(i2, k-i2)`

**Initialization:**
- Create a 3D array with dimensions `(2n-1) × n × n`
- Initialize all values to negative infinity (`-inf`) to represent invalid states
- Set `f[0][0][0] = grid[0][0]` as both persons start at `(0, 0)`

**State Transition:**
For each step `k` from 1 to `2n-2`:
1. Iterate through all possible positions `(i1, j1)` for person 1 where `j1 = k - i1`
2. Iterate through all possible positions `(i2, j2)` for person 2 where `j2 = k - i2`
3. Check validity:
   - Both `j1` and `j2` must be within bounds `[0, n)`
   - Neither `grid[i1][j1]` nor `grid[i2][j2]` should be `-1` (thorn)
4. Calculate cherries collected at current positions:
   - If both persons are at the same cell (`i1 == i2`), count cherry once: `t = grid[i1][j1]`
   - If at different cells, sum both: `t = grid[i1][j1] + grid[i2][j2]`
5. Consider all valid previous positions:
   - Person 1 could have come from `(i1-1, j1)` or `(i1, j1-1)`, meaning `x1 ∈ {i1-1, i1}`
   - Person 2 could have come from `(i2-1, j2)` or `(i2, j2-1)`, meaning `x2 ∈ {i2-1, i2}`
   - Update: `f[k][i1][i2] = max(f[k][i1][i2], f[k-1][x1][x2] + t)`

**Final Answer:**
- The maximum cherries when both reach `(n-1, n-1)` is `f[2n-2][n-1][n-1]`
- Return `max(0, f[2n-2][n-1][n-1])` to handle cases where no valid path exists (result would be negative infinity)

The time complexity is `O(n³)` as we have three nested loops each running up to `n` times, and the space complexity is also `O(n³)` for the 3D DP array.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider this 3×3 grid:
```
grid = [[0, 1, -1],
        [1, 0,  1],
        [1, 1,  1]]
```

We'll think of this as two people simultaneously moving from (0,0) to (2,2). Each person can only move right or down.

**Step 0 (k=0):** Both start at (0,0)
- Position: Person 1 at (0,0), Person 2 at (0,0)
- Cherries collected: 0 (empty cell)
- `f[0][0][0] = 0`

**Step 1 (k=1):** Each person takes 1 step
- Possible positions after 1 step: (0,1) or (1,0)
- Valid states:
  - Both at (0,1): `f[1][0][0] = 0 + 1 = 1` (one cherry at grid[0][1])
  - Both at (1,0): `f[1][1][1] = 0 + 1 = 1` (one cherry at grid[1][0])
  - One at (0,1), one at (1,0): `f[1][0][1] = 0 + 1 + 1 = 2` (two different cherries)

**Step 2 (k=2):** Each person has taken 2 steps total
- Any state with a person at (0,2) is skipped because grid[0][2] = -1 is a thorn
- For Person 1 at (1,1), Person 2 at (1,1) (same cell):
  - Previous states: `f[1][0][0] = 1`, `f[1][0][1] = 2`, `f[1][1][0] = 2`, `f[1][1][1] = 1`
  - Current cherry: grid[1][1] = 0 (empty)
  - `f[2][1][1] = 2 + 0 = 2`
- For Person 1 at (2,0), Person 2 at (1,1):
  - Previous states: `f[1][1][0] = 2` or `f[1][1][1] = 1`
  - Current cherries: grid[2][0] + grid[1][1] = 1 + 0 = 1
  - `f[2][2][1] = 2 + 1 = 3` (by symmetry `f[2][1][2] = 3`)
- Both at (2,0): `f[2][2][2] = f[1][1][1] + 1 = 2`

**Step 3 (k=3):** Each person has taken 3 steps
- For Person 1 at (1,2), Person 2 at (2,1):
  - Previous states: `f[2][0][1]` and `f[2][0][2]` are invalid (Person 1 would be on the thorn at (0,2)), `f[2][1][1] = 2`, `f[2][1][2] = 3`
  - Current cherries: grid[1][2] + grid[2][1] = 1 + 1 = 2
  - `f[3][1][2] = 3 + 2 = 5` (by symmetry `f[3][2][1] = 5`)
- The same-cell states are smaller: `f[3][1][1] = 3` and `f[3][2][2] = 4`

**Step 4 (k=4):** Both reach (2,2)
- Previous states: `f[3][1][1] = 3`, `f[3][1][2] = 5`, `f[3][2][1] = 5`, `f[3][2][2] = 4`
- Current cherry: grid[2][2] = 1 (both at same cell, count once)
- `f[4][2][2] = 5 + 1 = 6`

**Final Answer:** `max(0, f[4][2][2]) = 6`

One pair of paths that achieves 6 cherries:
- Person 1 path: (0,0) → (0,1) → (1,1) → (1,2) → (2,2)
- Person 2 path: (0,0) → (1,0) → (2,0) → (2,1) → (2,2)
- Cherries: (0,1), (1,0), (2,0), (1,2), (2,1), (2,2) = 6 total, which is every cherry in the grid

## Python Solution

```python
class Solution:
    def cherryPickup(self, grid: List[List[int]]) -> int:
        n = len(grid)
      
        # dp[k][i1][i2] represents the maximum cherries collected when:
        # - Person 1 is at position (i1, j1) where j1 = k - i1
        # - Person 2 is at position (i2, j2) where j2 = k - i2
        # - k represents the sum of coordinates (i + j), indicating the step number
        dp = [[[-float('inf')] * n for _ in range(n)] for _ in range(2 * n - 1)]
      
        # Initialize starting position - both persons start at (0, 0)
        dp[0][0][0] = grid[0][0]
      
        # Iterate through each step (k represents Manhattan distance from origin)
        for k in range(1, 2 * n - 1):
            for row1 in range(n):
                for row2 in range(n):
                    # Calculate column positions based on current step and row
                    col1 = k - row1
                    col2 = k - row2
                  
                    # Check if positions are valid and not blocked by thorns
                    if (not 0 <= col1 < n or 
                        not 0 <= col2 < n or 
                        grid[row1][col1] == -1 or 
                        grid[row2][col2] == -1):
                        continue
                  
                    # Calculate cherries to pick at current positions
                    cherries_picked = grid[row1][col1]
                    # Avoid double counting if both persons are at the same cell
                    if row1 != row2:
                        cherries_picked += grid[row2][col2]
                  
                    # Check all possible previous positions for both persons
                    # Each person could have come from either up or left
                    for prev_row1 in range(row1 - 1, row1 + 1):
                        for prev_row2 in range(row2 - 1, row2 + 1):
                            # Ensure previous positions are within bounds
                            if prev_row1 >= 0 and prev_row2 >= 0:
                                dp[k][row1][row2] = max(
                                    dp[k][row1][row2], 
                                    dp[k - 1][prev_row1][prev_row2] + cherries_picked
                                )
      
        # Return the maximum cherries collected, or 0 if no valid path exists
        return max(0, dp[-1][-1][-1])
```

## Time and Space Complexity

**Time Complexity:** `O(n^3)`

The algorithm uses dynamic programming with three nested loops:
- The outermost loop iterates through `k` from `1` to `2n - 2`, which is `O(n)` iterations
- The second loop iterates through `i1` from `0` to `n - 1`, which is `O(n)` iterations  
- The third loop iterates through `i2` from `0` to `n - 1`, which is `O(n)` iterations
- Inside these three loops, there are two more nested loops for `x1` and `x2`, but these only iterate through at most 2 values each (from `i - 1` to `i`), making them `O(1)`

Therefore, the overall time complexity is `O(n) × O(n) × O(n) × O(1) = O(n^3)`.

**Space Complexity:** `O(n^3)`

The algorithm creates a 3-dimensional DP array `f` with dimensions:
- First dimension: `(2n - 1)` elements, which is `O(n)`
- Second dimension: `n` elements, which is `O(n)`
- Third dimension: `n` elements, which is `O(n)`

The total space used by the DP array is `O(n) × O(n) × O(n) = O(n^3)`.

Where `n` is the side length of the grid.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Cherry Counting When Persons Meet**
One of the most common mistakes is forgetting to handle the case when both persons occupy the same cell. If you simply add `grid[i1][j1] + grid[i2][j2]` without checking if `i1 == i2` (which implies `j1 == j2` when `k` is the same), you'll double-count the cherry at that position.

**Wrong approach:**
```python
cherries_picked = grid[row1][col1] + grid[row2][col2]  # Always adds both
```

**Correct approach:**
```python
cherries_picked = grid[row1][col1]
if row1 != row2:  # Only add second person's cherry if at different positions
    cherries_picked += grid[row2][col2]
```

### 2. **Incorrect Previous Position Iteration**
Another frequent error is incorrectly determining which previous positions to check. The code uses `range(row1 - 1, row1 + 1)` which generates `[row1-1, row1]`. Some developers might mistakenly think this should be `range(row1, row1 + 2)` or forget that persons can only come from up (row-1) or left (same row, col-1).

**Wrong approach:**
```python
for prev_row1 in range(row1, row1 + 2):  # This would check row1 and row1+1
    # ...
```

**Correct approach:**
```python
for prev_row1 in range(row1 - 1, row1 + 1):  # Checks row1-1 and row1
    # ...
```

### 3. **Forgetting to Initialize with Negative Infinity**
Initializing the DP array with 0 instead of negative infinity can lead to incorrect results. Using negative infinity ensures that invalid paths (those passing through thorns or going out of bounds) don't contribute to the maximum calculation.

**Wrong approach:**
```python
dp = [[[0] * n for _ in range(n)] for _ in range(2 * n - 1)]  # Wrong!
```

**Correct approach:**
```python
dp = [[[-float('inf')] * n for _ in range(n)] for _ in range(2 * n - 1)]
```

### 4. **Misunderstanding the Step Count**
The total number of steps from `(0,0)` to `(n-1,n-1)` is `2n-2`, not `2n-1` or `n-1`. This is because we need exactly `n-1` right moves and `n-1` down moves. Some might incorrectly set the loop range or array size.

**Wrong approach:**
```python
for k in range(1, 2 * n):  # Goes one step too far
```

**Correct approach:**
```python
for k in range(1, 2 * n - 1):  # Correct: total steps is 2n-2
```

### 5. **Not Handling the "No Valid Path" Case**
If there's no valid path from start to end (due to thorns blocking all routes), the DP value will remain negative infinity. Returning this directly would give an incorrect result. Always use `max(0, dp[-1][-1][-1])` to handle this case.

**Wrong approach:**
```python
return dp[-1][-1][-1]  # Could return -inf if no path exists
```

**Correct approach:**
```python
return max(0, dp[-1][-1][-1])  # Returns 0 if no valid path
```

