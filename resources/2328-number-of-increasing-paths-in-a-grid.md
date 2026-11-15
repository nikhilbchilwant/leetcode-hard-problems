# 2328. Number of Increasing Paths in a Grid

> Source: https://algo.monster/liteproblems/2328

## Problem Statement

You have an `m x n` grid filled with integers. From any cell in the grid, you can move to any of its 4 adjacent cells (up, down, left, right).

Your task is to count all possible paths in the grid where:
- The values along the path are **strictly increasing** (each cell's value must be greater than the previous cell's value)
- You can start from **any** cell in the grid
- You can end at **any** cell in the grid

Two paths are considered different if they visit a different sequence of cells, even if they have the same starting and ending points.

Since the total number of such paths can be very large, return the result modulo `10^9 + 7`.

For example, if you're at a cell with value `5`, you can only move to adjacent cells with values greater than `5` to maintain a strictly increasing path. Each cell can be the starting point of multiple paths of different lengths - even a single cell by itself counts as a valid path of length 1.

The solution uses [Dynamic Programming](/problems/dynamic_programming_intro) with [memoization](/problems/memoization_intro) through a DFS approach. The function `dfs(i, j)` calculates the number of strictly increasing paths starting from position `(i, j)`. For each cell, it:
1. Counts itself as a path of length 1
2. Recursively explores all adjacent cells with larger values
3. Sums up all possible paths from that starting position

The final answer is the sum of `dfs(i, j)` for all positions `(i, j)` in the grid, representing all possible strictly increasing paths starting from every cell.

## Intuition

When we look at this problem, we need to count all possible strictly increasing paths in a grid. The key insight is that from any cell, we can explore multiple paths, and these paths can have different lengths - even a single cell counts as a valid path.

Think of each cell as a potential starting point for exploration. From any cell with value `v`, we can only move to adjacent cells with values greater than `v`. This creates a natural recursive structure: the number of paths starting from a cell equals 1 (the cell itself) plus the sum of all paths that can be reached from its valid neighbors.

For example, if we're at cell `(i, j)` with value `5`, and it has two adjacent cells with values `7` and `8`, then:
- We have 1 path of length 1 (just staying at the current cell)
- Plus all paths that start from the cell with value `7`
- Plus all paths that start from the cell with value `8`

This recursive relationship suggests using DFS. However, we might visit the same cell multiple times from different starting points or different paths. If we've already calculated the number of paths from cell `(x, y)`, we shouldn't recalculate it - we should reuse that result. This is where [memoization](/problems/memoization_intro) comes in.

The strictly increasing constraint is crucial because it:
1. Prevents infinite loops (we can never return to a previously visited cell in the same path)
2. Creates a natural DAG structure when we consider valid moves
3. Ensures that once we calculate paths from a cell, that count remains valid regardless of how we reached that cell

Therefore, we use DFS with [memoization](/problems/memoization_intro): for each cell in the grid, we calculate how many strictly increasing paths start from that cell, cache the result, and sum up all these counts to get our final answer.

**Learn more about [Depth-First Search](/problems/dfs_intro), [Breadth-First Search](/problems/bfs_intro), [Graph](/problems/graph_intro), [Topological Sort](/problems/topo_intro), [Memoization](/problems/memoization_intro) and [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements DFS with [memoization](/problems/memoization_intro) using Python's `@cache` decorator. Let's walk through the implementation step by step:

**Core DFS Function**: We define `dfs(i, j)` which calculates the number of strictly increasing paths starting from position `(i, j)`. The `@cache` decorator automatically memoizes results, storing them in a dictionary to avoid redundant calculations.

**Base Case**: Each cell itself forms a valid path of length 1, so we initialize `ans = 1` for every cell we visit.

**Exploring Neighbors**: We use `pairwise((-1, 0, 1, 0, -1))` to generate direction pairs: `(-1, 0)`, `(0, 1)`, `(1, 0)`, `(0, -1)`, representing moves up, right, down, and left respectively. For each direction:
- Calculate the new position: `x, y = i + a, j + b`
- Check if the new position is valid: `0 <= x < m and 0 <= y < n`
- Check if we can move there (strictly increasing): `grid[i][j] < grid[x][y]`
- If both conditions are met, recursively calculate paths from `(x, y)` and add to our answer

**Modulo Operation**: Since the answer can be very large, we apply modulo `10^9 + 7` at two key points:
1. When accumulating paths in the DFS function: `ans = (ans + dfs(x, y)) % mod`
2. When computing the final sum: `sum(...) % mod`

**Final Aggregation**: We call `dfs(i, j)` for every cell `(i, j)` in the grid and sum all results. This gives us the total count of all strictly increasing paths starting from any cell.

**Time Complexity**: `O(m * n)` where each cell is computed once due to [memoization](/problems/memoization_intro), and each computation checks at most 4 neighbors.

**Space Complexity**: `O(m * n)` for the [memoization](/problems/memoization_intro) cache storing results for each cell.

The [memoization](/problems/memoization_intro) is crucial here - without it, we would repeatedly recalculate paths from the same cells, leading to exponential time complexity. With memoization, each cell's path count is calculated exactly once, making the solution efficient.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

Consider a 2×3 grid:
```
[1, 3, 5]
[2, 4, 6]
```

We'll use DFS with memoization to count all strictly increasing paths.

**Step 1: Start DFS from cell (0,0) with value 1**
- Count itself as 1 path
- Check neighbors:
  - Up: out of bounds
  - Right: (0,1) has value 3 > 1 ✓
  - Down: (1,0) has value 2 > 1 ✓
  - Left: out of bounds
- Recursively call dfs(0,1) and dfs(1,0)

**Step 2: DFS from cell (0,1) with value 3**
- Count itself as 1 path
- Check neighbors:
  - Up: out of bounds
  - Right: (0,2) has value 5 > 3 ✓
  - Down: (1,1) has value 4 > 3 ✓
  - Left: (0,0) has value 1 < 3 ✗
- Recursively call dfs(0,2) and dfs(1,1)

**Step 3: DFS from cell (0,2) with value 5**
- Count itself as 1 path
- Check neighbors:
  - Right: out of bounds
  - Down: (1,2) has value 6 > 5 ✓
  - Others: either out of bounds or smaller values
- Recursively call dfs(1,2)

**Step 4: DFS from cell (1,2) with value 6**
- Count itself as 1 path
- All neighbors have smaller values
- Return 1

Working backwards with memoization:
- dfs(1,2) = 1 (just itself)
- dfs(0,2) = 1 + dfs(1,2) = 1 + 1 = 2
- dfs(1,1) = 1 + dfs(1,2) = 1 + 1 = 2 (can go to cell with value 6)
- dfs(0,1) = 1 + dfs(0,2) + dfs(1,1) = 1 + 2 + 2 = 5
- dfs(1,0) = 1 + dfs(1,1) = 1 + 2 = 3 (can go to cell with value 4)
- dfs(0,0) = 1 + dfs(0,1) + dfs(1,0) = 1 + 5 + 3 = 9

Similarly calculating for all starting positions:
- dfs(0,0) = 9 (already calculated)
- dfs(0,1) = 5 (already cached)
- dfs(0,2) = 2 (already cached)
- dfs(1,0) = 3 (already cached)
- dfs(1,1) = 2 (already cached)
- dfs(1,2) = 1 (already cached)

**Final answer**: 9 + 5 + 2 + 3 + 2 + 1 = **22 total paths**

These 22 paths include:
- 6 paths of length 1 (each cell by itself)
- Paths like [1→3], [1→2], [3→5], [3→4], etc.
- Longer paths like [1→3→5], [1→2→4→6], [1→3→5→6], etc.

The memoization ensures we calculate each cell's contribution only once, even though multiple paths might pass through it.

## Python Solution

```python
class Solution:
    def countPaths(self, grid: List[List[int]]) -> int:
        """
        Count the number of strictly increasing paths in the grid.
        Each path starts from any cell and moves to adjacent cells with larger values.
        """
      
        # Use memoization to cache results of subproblems
        @cache
        def dfs(row: int, col: int) -> int:
            """
            Calculate the number of increasing paths starting from (row, col).
          
            Args:
                row: Current row index
                col: Current column index
          
            Returns:
                Number of paths starting from this cell (including single cell path)
            """
            # Start with 1 to count the path containing only the current cell
            path_count = 1
          
            # Define four directions: up, right, down, left
            directions = [(-1, 0), (0, 1), (1, 0), (0, -1)]
          
            # Explore all four adjacent cells
            for delta_row, delta_col in directions:
                next_row = row + delta_row
                next_col = col + delta_col
              
                # Check if the next cell is within bounds and has a greater value
                if (0 <= next_row < rows and 
                    0 <= next_col < cols and 
                    grid[row][col] < grid[next_row][next_col]):
                    # Add paths from the next cell to current count
                    path_count = (path_count + dfs(next_row, next_col)) % MOD
          
            return path_count
      
        # Define constants
        MOD = 10**9 + 7
        rows = len(grid)
        cols = len(grid[0])
      
        # Calculate total paths by summing paths starting from each cell
        total_paths = 0
        for i in range(rows):
            for j in range(cols):
                total_paths = (total_paths + dfs(i, j)) % MOD
      
        return total_paths
```

## Time and Space Complexity

**Time Complexity:** `O(m × n)`

The algorithm uses dynamic programming with memoization through the `@cache` decorator. Each cell `(i, j)` in the grid is visited and computed exactly once due to memoization. When `dfs(i, j)` is called for a specific cell, it explores up to 4 neighboring cells, which is a constant operation `O(1)`. Since we call `dfs` for all `m × n` cells in the grid (from the main function's sum comprehension), and each cell's result is computed only once and then cached, the total time complexity is `O(m × n)`.

**Space Complexity:** `O(m × n)`

The space complexity consists of two main components:
1. **Memoization cache:** The `@cache` decorator stores the result for each unique `(i, j)` pair. In the worst case, we compute and store results for all `m × n` cells, requiring `O(m × n)` space.
2. **Recursion call stack:** In the worst case (when the grid values form a strictly increasing path through all cells), the recursion depth could reach `O(m × n)`. However, this is typically much less in practice.

The dominant factor is the memoization cache, giving us an overall space complexity of `O(m × n)`, where `m` and `n` are the number of rows and columns in the grid, respectively.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Forgetting to Count Single-Cell Paths**

A critical mistake is forgetting that each cell by itself forms a valid path of length 1. Many developers initially write:

```python
# INCORRECT - Missing single-cell paths
def dfs(row, col):
    path_count = 0  # Wrong! Should start with 1
    for delta_row, delta_col in directions:
        # ... exploring neighbors
    return path_count
```

This error leads to returning 0 for cells that have no adjacent cells with larger values (local maxima), completely missing all single-cell paths. The correct approach initializes `path_count = 1` to account for the current cell as a valid path.

### 2. **Incorrect Modulo Application**

Applying modulo operations incorrectly or inconsistently can cause integer overflow or wrong results:

```python
# INCORRECT - Missing modulo in intermediate calculations
def dfs(row, col):
    path_count = 1
    for delta_row, delta_col in directions:
        if valid_move:
            path_count = path_count + dfs(next_row, next_col)  # Missing modulo!
    return path_count % MOD

# Also INCORRECT - Applying modulo only at the end
total_paths = sum(dfs(i, j) for i in range(rows) for j in range(cols)) % MOD
```

The correct approach applies modulo at every addition to prevent overflow:
- Inside DFS: `path_count = (path_count + dfs(next_row, next_col)) % MOD`
- In final sum: Sum incrementally with modulo rather than summing everything first

### 3. **Using Wrong Inequality for Path Validity**

Using non-strict inequality (`<=`) instead of strict inequality (`<`) violates the problem requirement:

```python
# INCORRECT - Allows equal values
if grid[row][col] <= grid[next_row][next_col]:  # Wrong!
    path_count = (path_count + dfs(next_row, next_col)) % MOD

# CORRECT - Strictly increasing
if grid[row][col] < grid[next_row][next_col]:
    path_count = (path_count + dfs(next_row, next_col)) % MOD
```

### 4. **Manual Memoization Implementation Errors**

Attempting manual memoization without proper initialization or key management:

```python
# INCORRECT - Common manual memoization mistakes
memo = {}  # Might forget to initialize

def dfs(row, col):
    if (row, col) in memo:
        return memo[(row, col)]
  
    path_count = 1
    # ... calculation logic
  
    # Forgot to store result before returning!
    return path_count  # Should be: memo[(row, col)] = path_count; return path_count
```

Using `@cache` decorator eliminates these manual memoization errors and is cleaner.

### 5. **Direction Array Implementation Issues**

Incorrectly defining or iterating through directions:

```python
# INCORRECT - Wrong direction pairs
directions = [-1, 0, 1, 0, -1]  # This is not a list of tuples!
for i in range(4):
    delta_row = directions[i]  # IndexError or wrong values
    delta_col = directions[i+1]

# CORRECT - Clear direction tuples
directions = [(-1, 0), (0, 1), (1, 0), (0, -1)]
for delta_row, delta_col in directions:
    # Clean and readable
```

The solution should use clear, explicit direction pairs to avoid index confusion and improve code readability.

