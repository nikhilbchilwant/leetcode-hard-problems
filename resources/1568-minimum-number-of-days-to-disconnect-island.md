# 1568. Minimum Number of Days to Disconnect Island

> Source: https://algo.monster/liteproblems/1568

## Problem Statement

You have a binary grid where `1` represents land and `0` represents water. An island is formed by connecting adjacent land cells horizontally or vertically (4-directionally connected). The grid is considered "connected" if it contains exactly one island, and "disconnected" if it has zero islands or more than one island.

Each day, you can change one land cell (`1`) into a water cell (`0`). Your goal is to find the minimum number of days needed to disconnect the grid.

The solution works by first checking if the grid is already disconnected (has 0 or 2+ islands). If so, it returns `0` days. Then it tries removing each land cell one by one to see if that disconnects the grid - if any single removal works, it returns `1` day. Otherwise, it returns `2` days, since removing at most 2 cells is always sufficient to disconnect any connected grid.

The `count` function uses [depth-first search](/problems/dfs_intro) (DFS) to count the number of islands. It marks visited land cells as `2` during traversal, counts each connected component as one island, then restores the original values afterward. This allows checking how many islands exist after each potential cell removal.

## Intuition

The key insight starts with understanding what makes a grid "disconnected" - it needs to have either zero islands or more than one island. If we already have this condition, we need `0` days.

For a connected grid (exactly one island), we need to find critical points that, when removed, would split the island. Think of it like finding the weakest link in a chain - some cells act as bridges connecting different parts of the island.

Why can we always disconnect in at most `2` days? Consider any connected island - even in the worst case where it's a solid rectangle, we can always find a corner cell and its adjacent cell. Removing these two cells will either isolate the corner or create a disconnection. This gives us our upper bound.

The approach becomes clear:
1. First check if we need `0` days (already disconnected)
2. Then try removing each land cell one at a time - if any single removal disconnects the grid, we need `1` day
3. If no single cell removal works, we know `2` days is sufficient

The DFS counting mechanism is elegant - we temporarily mark visited cells as `2` during traversal to avoid revisiting them, count each connected component as one island, then restore the original values. This allows us to non-destructively test each scenario.

The brute force nature (trying each cell) might seem inefficient, but it's actually optimal here because determining the minimum cuts in a graph is inherently complex, and with the small constraint that the answer is at most `2`, checking all single-cell removals is practical and guarantees finding the minimum.

**Learn more about [Depth-First Search](/problems/dfs_intro) and [Breadth-First Search](/problems/bfs_intro) patterns.**

## Solution Approach

The solution implements a brute force approach with DFS to count connected components. Let's walk through the implementation:

**Main Function (`minDays`):**

1. **Initial Check**: First, we call `self.count(grid)` to determine if the grid is already disconnected. If the number of islands is not exactly 1, return `0` days.

2. **Try Single Cell Removal**: Iterate through each cell in the grid. For each land cell (`grid[i][j] == 1`):
   - Temporarily change it to water: `grid[i][j] = 0`
   - Count the islands again using `self.count(grid)`
   - If the count is not 1 (either 0 or 2+ islands), we found a solution with `1` day
   - Restore the cell: `grid[i][j] = 1` for the next iteration

3. **Default Case**: If no single cell removal disconnects the grid, return `2` days.

**Island Counting Function (`count`):**

The counting function uses DFS with a clever marking technique:

1. **DFS Helper**: The nested `dfs(i, j)` function:
   - Marks the current cell as `2` (visited)
   - Explores all 4 directions: `[[0, -1], [0, 1], [1, 0], [-1, 0]]`
   - Recursively visits adjacent land cells that haven't been visited

2. **Counting Process**:
   - Initialize counter `cnt = 0`
   - Iterate through the entire grid
   - When finding an unvisited land cell (`grid[i][j] == 1`), start a DFS from there
   - Increment the counter for each new island found

3. **Restoration**: After counting, restore all cells marked as `2` back to `1`:
   ```python
   for i in range(m):
       for j in range(n):
           if grid[i][j] == 2:
               grid[i][j] = 1
   ```

**Time Complexity**: `O(m² × n²)` where m and n are grid dimensions - we potentially try removing each cell (`O(m × n)`) and for each removal, we run DFS to count islands (`O(m × n)`).

**Space Complexity**: `O(m × n)` for the recursion stack in worst case when DFS traverses all cells.

The elegance lies in the temporary marking strategy - using `2` to mark visited cells allows us to count islands without creating a separate visited array, and we can easily restore the grid afterward for the next test.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider this 2×3 grid:
```
1 1 1
1 0 1
```

**Step 1: Check if already disconnected**
- Run DFS from position (0,0)
- DFS visits: (0,0) → (0,1) → (0,2) → (1,2) → (1,0)
- All land cells are reachable from one starting point
- Island count = 1 (connected), so we need to proceed

**Step 2: Try removing each land cell one by one**

Try removing (0,0):
```
0 1 1    (temporarily set grid[0][0] = 0)
1 0 1
```
- Count islands: Start DFS from (0,1), visits (0,1) → (0,2) → (1,2)
- Another DFS starts from (1,0), visits only (1,0)
- Island count = 2 → Grid is disconnected! Return 1 day

Let's verify another removal to understand why some cells work and others don't:

Try removing (0,1):
```
1 0 1    (temporarily set grid[0][1] = 0)
1 0 1
```
- Count islands: DFS from (0,0) visits only (0,0)
- DFS from (0,2) visits (0,2) → (1,2)
- DFS from (1,0) visits only (1,0)
- Island count = 3 → This also disconnects the grid

Try removing (1,2):
```
1 1 1    (temporarily set grid[1][2] = 0)
1 0 0
```
- Count islands: DFS from (0,0) visits (0,0) → (0,1) → (0,2) → (1,0)
- Island count = 1 → Still connected, this cell isn't critical

**Result**: Since removing cell (0,0) disconnects the grid, the answer is **1 day**.

The DFS marking process during counting:
- When visiting a cell, mark it as `2`
- After exploring all connected cells, we have one island marked with `2`s
- Restore all `2`s back to `1`s before the next test
- This avoids creating a separate visited array and keeps the grid intact for subsequent checks

## Python Solution

```python
class Solution:
    def minDays(self, grid: List[List[int]]) -> int:
        """
        Find minimum number of days to disconnect the island.
        Each day we can change one land cell (1) to water (0).
        Returns 0 if already disconnected, 1 if one removal suffices, otherwise 2.
        """
        # Check if grid is already disconnected
        if self.count_islands(grid) != 1:
            return 0
      
        rows, cols = len(grid), len(grid[0])
      
        # Try removing each land cell one by one
        for row in range(rows):
            for col in range(cols):
                if grid[row][col] == 1:
                    # Temporarily remove this land cell
                    grid[row][col] = 0
                  
                    # Check if removal disconnects the island
                    if self.count_islands(grid) != 1:
                        return 1
                  
                    # Restore the land cell
                    grid[row][col] = 1
      
        # If no single removal works, we need at most 2 removals
        # (This is always possible for any connected island)
        return 2

    def count_islands(self, grid: List[List[int]]) -> int:
        """
        Count the number of disconnected islands in the grid.
        Uses DFS to mark visited cells.
        """
        def depth_first_search(row: int, col: int) -> None:
            """
            DFS helper to mark all connected land cells starting from (row, col).
            Marks visited cells as 2 temporarily.
            """
            # Mark current cell as visited
            grid[row][col] = 2
          
            # Check all 4 adjacent cells (up, down, left, right)
            directions = [[0, -1], [0, 1], [1, 0], [-1, 0]]
            for delta_row, delta_col in directions:
                new_row, new_col = row + delta_row, col + delta_col
              
                # If adjacent cell is valid and is unvisited land
                if (0 <= new_row < rows and 
                    0 <= new_col < cols and 
                    grid[new_row][new_col] == 1):
                    depth_first_search(new_row, new_col)
      
        rows, cols = len(grid), len(grid[0])
        island_count = 0
      
        # Find and count all islands
        for row in range(rows):
            for col in range(cols):
                if grid[row][col] == 1:
                    # Found a new island, explore it completely
                    depth_first_search(row, col)
                    island_count += 1
      
        # Restore all visited cells (marked as 2) back to land (1)
        for row in range(rows):
            for col in range(cols):
                if grid[row][col] == 2:
                    grid[row][col] = 1
      
        return island_count
```

## Time and Space Complexity

**Time Complexity:** `O(m² × n²)` where `m` is the number of rows and `n` is the number of columns in the grid.

- The initial call to `count(grid)` takes `O(m × n)` time to perform DFS and count islands
- The main loop iterates through all `m × n` cells
- For each cell with value 1 (worst case all cells), we:
  - Set the cell to 0: `O(1)`
  - Call `count(grid)`: `O(m × n)` for DFS traversal
  - Reset the cell to 1: `O(1)`
- In the worst case where all cells are 1, we perform `O(m × n)` iterations, each calling `count()` which takes `O(m × n)` time
- Total: `O(m × n) × O(m × n) = O(m² × n²)`

**Space Complexity:** `O(m × n)` 

- The recursive DFS in the `count()` function can go as deep as the total number of land cells in the worst case (when all cells form a single connected component in a snake-like pattern)
- Maximum recursion depth: `O(m × n)` for the call stack
- No additional data structures are used besides the recursion stack
- The grid is modified in-place during DFS but restored after each count operation

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Forgetting to Restore Modified Grid State**

One of the most common mistakes is modifying the grid during island counting or cell removal testing without properly restoring it. This leads to incorrect results in subsequent iterations.

**Pitfall Example:**
```python
def count_islands_wrong(self, grid):
    def dfs(i, j):
        grid[i][j] = 2  # Mark as visited
        # ... DFS logic ...
  
    # Count islands...
    # FORGOT to restore cells marked as 2 back to 1!
    return island_count
```

**Solution:**
Always restore the grid after modifications. The correct approach uses a restoration loop:
```python
# After counting islands
for i in range(rows):
    for j in range(cols):
        if grid[i][j] == 2:
            grid[i][j] = 1
```

### 2. **Incorrect Boundary Checking in DFS**

Another frequent error is improper boundary validation when exploring adjacent cells, leading to index out of bounds errors or missed cells.

**Pitfall Example:**
```python
def dfs(i, j):
    grid[i][j] = 2
    for di, dj in directions:
        ni, nj = i + di, j + dj
        # Wrong: using OR instead of AND for boundary check
        if 0 <= ni < rows or 0 <= nj < cols:  # Should be AND!
            if grid[ni][nj] == 1:
                dfs(ni, nj)
```

**Solution:**
Use AND operator for boundary checks to ensure both row and column indices are valid:
```python
if (0 <= new_row < rows and 
    0 <= new_col < cols and 
    grid[new_row][new_col] == 1):
    dfs(new_row, new_col)
```

### 3. **Assuming Maximum Days is Always 2**

While the mathematical proof shows that any connected grid can be disconnected in at most 2 days, directly returning 2 without checking single-cell removals would miss optimal solutions.

**Pitfall Example:**
```python
def minDays_wrong(self, grid):
    if self.count_islands(grid) != 1:
        return 0
    # Wrong: Skip checking 1-day solutions
    return 2  # Missing the check for 1-day disconnection!
```

**Solution:**
Always check if removing a single cell can disconnect the grid before defaulting to 2:
```python
# Try each cell removal first
for i in range(rows):
    for j in range(cols):
        if grid[i][j] == 1:
            grid[i][j] = 0
            if self.count_islands(grid) != 1:
                return 1
            grid[i][j] = 1
# Only return 2 if no single removal works
return 2
```

### 4. **Not Handling Edge Cases Properly**

Failing to consider grids that are already disconnected (0 islands or multiple islands) at the start.

**Pitfall Example:**
```python
def minDays_wrong(self, grid):
    # Wrong: Assumes grid always has exactly 1 island initially
    for i in range(rows):
        for j in range(cols):
            # ... try removing cells ...
```

**Solution:**
Always check the initial state first:
```python
if self.count_islands(grid) != 1:
    return 0  # Already disconnected
```

### 5. **Using a Separate Visited Array Unnecessarily**

Creating a separate 2D visited array increases space complexity and code complexity when the grid itself can serve as the visited tracker.

**Pitfall Example:**
```python
def count_islands_inefficient(self, grid):
    visited = [[False] * cols for _ in range(rows)]  # Extra O(m*n) space
  
    def dfs(i, j):
        visited[i][j] = True
        # ... more complex logic to check both grid and visited ...
```

**Solution:**
Use the grid itself by temporarily marking visited cells with a different value (like 2), then restore them afterward. This maintains O(1) extra space beyond the recursion stack.

