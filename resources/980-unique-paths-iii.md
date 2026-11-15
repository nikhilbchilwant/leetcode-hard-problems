# 980. Unique Paths III

> Source: https://algo.monster/liteproblems/980

## Problem Statement

You are given an `m x n` grid where each cell contains one of four possible values:
- `1`: The starting square (exactly one exists in the grid)
- `2`: The ending square (exactly one exists in the grid)  
- `0`: Empty squares that can be walked through
- `-1`: Obstacles that cannot be walked through

Your task is to find the number of unique paths from the starting square to the ending square that satisfy these conditions:
- You can only move in 4 directions (up, down, left, right)
- Every non-obstacle square must be visited exactly once
- This includes the starting square, ending square, and all empty squares (`0`)

In other words, you need to count all possible paths that:
1. Start at the cell marked with `1`
2. End at the cell marked with `2`
3. Visit every cell that isn't an obstacle (`-1`) exactly one time
4. Only use horizontal and vertical movements (no diagonal moves)

For example, if you have a 2x3 grid with one obstacle, there might be multiple valid paths that visit all non-obstacle cells exactly once while moving from start to end. The function should return the total count of such valid paths.

## Intuition

The key insight is recognizing this as a path enumeration problem with strict visitation constraints. We need to find ALL valid paths, not just one, which immediately suggests we can't use a greedy approach or simple BFS/DFS that finds just one solution.

Think about what makes a path valid:
1. It starts at cell marked `1`
2. It ends at cell marked `2`  
3. It visits EVERY non-obstacle cell exactly once

The third constraint is crucial - we must visit every walkable cell. This means if we have `n` walkable cells total (including start and end), our path must be exactly `n` steps long. Any shorter path would miss some cells, and any longer path would revisit cells.

Since we need to explore all possible paths and keep track of which cells we've visited, [backtracking](/problems/backtracking) becomes the natural choice. Why? Because at each step, we have up to 4 choices (up, down, left, right), and we need to:
- Try each valid direction
- Mark the cell as visited (so we don't revisit it in the current path)
- Recursively explore from the new position
- Unmark the cell when we backtrack (so other paths can use it)

The "unmarking" step is critical - it allows us to reuse cells in different paths. For example, if path A goes through cell `(1,1)` early and fails, path B should still be able to use cell `(1,1)` later in its traversal.

We can optimize by first counting all walkable cells. Then, when we reach the ending cell, we simply check if we've visited exactly the right number of cells. If `k` represents the number of cells visited in our current path and `cnt` is the number of empty cells (`0`s), then a valid complete path has `k = cnt + 2` (the `+2` accounts for the start and end cells).

This approach systematically explores all possible paths through exhaustive search with [backtracking](/problems/backtracking), ensuring we find every valid solution.

**Learn more about [Backtracking](/problems/backtracking) patterns.**

## Solution Approach

The implementation uses depth-first search with [backtracking](/problems/backtracking) to explore all possible paths. Let's walk through the key components:

**Initial Setup:**
First, we traverse the entire grid to:
- Find the starting position `(x, y)` where `grid[i][j] == 1`
- Count the number of empty cells (`0`s) and store it in `cnt`
- This preprocessing helps us know when we've visited all required cells

**The DFS Function:**
We define `dfs(i, j, k)` where:
- `(i, j)` is the current cell position
- `k` is the number of cells visited so far in the current path

**Base Case:**
When we reach the ending cell (`grid[i][j] == 2`), we check if `k == cnt + 1`. Why `cnt + 1`?
- `cnt` counts only the empty cells (`0`s)
- We also need to count the starting cell (`1`)
- So a complete path visits `cnt + 1` cells before reaching the end
- If this condition is met, we've found a valid path and return `1`, otherwise `0`

**Recursive Exploration:**
For non-ending cells, we explore all four directions using the `dirs` array `(-1, 0, 1, 0, -1)`. The `pairwise` function creates pairs like `(-1, 0)`, `(0, 1)`, `(1, 0)`, `(0, -1)` representing up, right, down, left movements.

For each adjacent cell `(x, y)`:
1. Check if it's within bounds: `0 <= x < m and 0 <= y < n`
2. Check if it hasn't been visited: `(x, y) not in vis`
3. Check if it's not an obstacle: `grid[x][y] != -1`

**[Backtracking](/problems/backtracking) Mechanism:**
When a valid adjacent cell is found:
1. Mark it as visited: `vis.add((x, y))`
2. Recursively explore from that cell: `dfs(x, y, k + 1)`
3. After exploring, unmark it: `vis.remove((x, y))`

The unmarking step is crucial for [backtracking](/problems/backtracking) - it allows the cell to be used in other potential paths.

**Tracking Visited Cells:**
We use a set `vis` to track visited cells in the current path. Sets provide O(1) lookup and modification, making them efficient for this purpose. We initialize it with the starting position since we begin our path there.

**Final Computation:**
We start the DFS from the starting position with `k = 0` (since we're counting cells visited after the start). The function returns the total count of all valid paths from start to end that visit every non-obstacle cell exactly once.

The time complexity is O(3^(m×n)) in the worst case since at each cell we have at most 3 unvisited directions to explore (we can't go back), and space complexity is O(m×n) for the recursion stack and visited set.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider this 2×3 grid:
```
[1, 0, 0]
[0, 0, 2]
```

**Initial Setup:**
- Starting position: `(0, 0)` where value is `1`
- Ending position: `(1, 2)` where value is `2`
- Count of empty cells (`0`s): `cnt = 4`
- Initialize visited set: `vis = {(0, 0)}`

**Goal:** Find paths that visit all 6 cells exactly once (4 empty cells + start + end).

**DFS Exploration from (0,0), k=0:**

We check all 4 directions from `(0, 0)`:
- Up `(-1, 0)`: Out of bounds ❌
- Right `(0, 1)`: Valid, value is `0` ✓
- Down `(1, 0)`: Valid, value is `0` ✓
- Left `(0, -1)`: Out of bounds ❌

**Path 1: Going right first from (0,0) → (0,1)**
- Mark `(0, 1)` as visited: `vis = {(0, 0), (0, 1)}`
- Call `dfs(0, 1, 1)`

From `(0, 1)`, we can go to `(0, 2)` or `(1, 1)`:

**Path 1a: (0,0) → (0,1) → (0,2)**
- Mark `(0, 2)` visited, call `dfs(0, 2, 2)`
- From `(0, 2)`, only `(1, 2)` is valid (the end cell)
- But going to `(1, 2)` with `k=3` means we've only visited 4 cells total
- We need to visit all 6 cells, so this path fails ❌
- Backtrack: unmark `(0, 2)`

**Path 1b: (0,0) → (0,1) → (1,1)**
- Mark `(1, 1)` visited, call `dfs(1, 1, 2)`
- Continue exploring...
- Eventually find: `(0,0) → (0,1) → (1,1) → (1,0) → (0,2) → (1,2)`
- When reaching `(1, 2)` with `k=5`, we've visited 6 cells total ✓
- This is a valid path! Count = 1

After backtracking from Path 1, we explore:

**Path 2: Going down first from (0,0) → (1,0)**
- Mark `(1, 0)` as visited: `vis = {(0, 0), (1, 0)}`
- Call `dfs(1, 0, 1)`
- Continue exploration...
- Eventually find: `(0,0) → (1,0) → (1,1) → (0,1) → (0,2) → (1,2)`
- This is another valid path! Count = 2

**Key Backtracking Example:**
Notice how cell `(0, 1)` is used in both valid paths:
- In Path 1: visited as the 2nd cell
- In Path 2: visited as the 4th cell

This is possible because after Path 1 completes, we backtrack and unmark all cells except the starting position, allowing Path 2 to use them in a different order.

**Final Result:** The algorithm returns 2, indicating there are exactly 2 unique paths from start to end that visit every cell exactly once.

## Python Solution

```python
class Solution:
    def uniquePathsIII(self, grid: List[List[int]]) -> int:
        def dfs(row: int, col: int, steps: int) -> int:
            """
            Depth-first search to count valid paths.
          
            Args:
                row: Current row position
                col: Current column position
                steps: Number of cells visited so far
          
            Returns:
                Number of valid paths from current position
            """
            # Check if we reached the destination
            if grid[row][col] == 2:
                # Valid path only if we visited all empty cells plus start
                return 1 if steps == empty_cells + 1 else 0
          
            path_count = 0
          
            # Try all four directions: up, right, down, left
            for i in range(4):
                next_row = row + directions[i]
                next_col = col + directions[i + 1]
              
                # Check if next cell is valid and unvisited
                if (0 <= next_row < rows and 
                    0 <= next_col < cols and 
                    (next_row, next_col) not in visited and 
                    grid[next_row][next_col] != -1):
                  
                    # Mark cell as visited
                    visited.add((next_row, next_col))
                  
                    # Recursively explore from next cell
                    path_count += dfs(next_row, next_col, steps + 1)
                  
                    # Backtrack: unmark cell as visited
                    visited.remove((next_row, next_col))
          
            return path_count
      
        # Initialize grid dimensions
        rows, cols = len(grid), len(grid[0])
      
        # Find starting position (cell with value 1)
        start_row, start_col = 0, 0
        for i in range(rows):
            for j in range(cols):
                if grid[i][j] == 1:
                    start_row, start_col = i, j
                    break
      
        # Direction vectors for moving up, right, down, left
        directions = [-1, 0, 1, 0, -1]
      
        # Count empty cells (cells with value 0)
        empty_cells = sum(row.count(0) for row in grid)
      
        # Initialize visited set with starting position
        visited = {(start_row, start_col)}
      
        # Start DFS from starting position with 0 steps taken
        return dfs(start_row, start_col, 0)
```

## Time and Space Complexity

**Time Complexity:** `O(3^(m × n))`

The algorithm uses DFS with backtracking to explore all possible paths from the starting position to the ending position. In the worst case:
- Each cell (except obstacles) can be visited, giving us at most `m × n` cells to explore
- From each cell, we can move in at most 3 directions (we cannot go back to where we came from due to the visited set)
- The recursion depth can be at most `m × n` (visiting all non-obstacle cells)
- This creates a recursion tree where each node has at most 3 branches and the height is at most `m × n`
- Therefore, the time complexity is `O(3^(m × n))`

**Space Complexity:** `O(m × n)`

The space complexity consists of:
- The visited set `vis` which can contain at most `m × n` coordinates: `O(m × n)`
- The recursion call stack depth which can go up to `m × n` in the worst case when we need to visit all cells: `O(m × n)`
- Other variables like `start`, `dirs`, `cnt` which take constant space: `O(1)`

The dominant factor is `O(m × n)` for both the visited set and the recursion stack, resulting in overall space complexity of `O(m × n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Incorrect Path Length Validation

**The Problem:**
A frequent mistake is miscounting the total number of cells that need to be visited. Developers often forget to account for all walkable cells properly:

```python
# INCORRECT: Only counting empty cells
if grid[row][col] == 2:
    return 1 if steps == empty_cells else 0  # Wrong!
```

This fails because the path must include:
- The starting cell (value `1`)
- All empty cells (value `0`)
- The ending cell (value `2`)

**The Solution:**
The correct validation checks if we've visited `empty_cells + 1` cells when reaching the end:
```python
if grid[row][col] == 2:
    return 1 if steps == empty_cells + 1 else 0
```
The `+1` accounts for the starting cell, and the ending cell is implicitly counted when we reach it.

### Pitfall 2: Not Properly Backtracking the Visited Set

**The Problem:**
Forgetting to remove cells from the visited set after exploring a path:

```python
# INCORRECT: Missing backtrack
visited.add((next_row, next_col))
path_count += dfs(next_row, next_col, steps + 1)
# Forgot to remove from visited!
```

This causes cells to remain marked as visited even when exploring alternative paths, leading to undercounting valid paths.

**The Solution:**
Always remove the cell from visited after the recursive call returns:
```python
visited.add((next_row, next_col))
path_count += dfs(next_row, next_col, steps + 1)
visited.remove((next_row, next_col))  # Critical backtracking step
```

### Pitfall 3: Visiting the Starting Cell Multiple Times

**The Problem:**
Not initializing the visited set with the starting position:

```python
# INCORRECT: Empty visited set
visited = set()  # Wrong!
return dfs(start_row, start_col, 0)
```

This allows the path to potentially revisit the starting cell, violating the "exactly once" constraint.

**The Solution:**
Initialize the visited set with the starting position:
```python
visited = {(start_row, start_col)}  # Correct initialization
return dfs(start_row, start_col, 0)
```

### Pitfall 4: Using Wrong Direction Arrays

**The Problem:**
Incorrectly implementing the direction vectors or their usage:

```python
# INCORRECT: Wrong direction array usage
directions = [(-1, 0), (0, 1), (1, 0), (0, -1)]  # Pairs
for dx, dy in directions:
    next_row = row + dx
    next_col = col + dy
```

While this works, mixing it with the flattened array approach can cause index errors:
```python
# INCORRECT: Mismatched iteration
directions = [-1, 0, 1, 0, -1]
for i in range(4):
    next_row = row + directions[i]
    next_col = col + directions[i]  # Wrong index!
```

**The Solution:**
Use consistent indexing with the flattened array:
```python
directions = [-1, 0, 1, 0, -1]
for i in range(4):
    next_row = row + directions[i]
    next_col = col + directions[i + 1]  # Correct offset
```

### Pitfall 5: Counting All Cells Instead of Empty Cells

**The Problem:**
Counting all non-obstacle cells when determining the required path length:

```python
# INCORRECT: Counting start and end cells
empty_cells = 0
for i in range(rows):
    for j in range(cols):
        if grid[i][j] != -1:  # Wrong condition!
            empty_cells += 1
```

This incorrectly includes the start (1) and end (2) cells in the count.

**The Solution:**
Only count cells with value 0:
```python
empty_cells = sum(row.count(0) for row in grid)
```
Or explicitly:
```python
empty_cells = 0
for i in range(rows):
    for j in range(cols):
        if grid[i][j] == 0:  # Only count empty cells
            empty_cells += 1
```

