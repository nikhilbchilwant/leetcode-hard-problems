# 329. Longest Increasing Path in a Matrix

> Source: https://algo.monster/liteproblems/329

## Problem Statement

You are given an `m x n` matrix of integers. Your task is to find the length of the longest increasing path in this matrix.

From any cell in the matrix, you can move in four directions: left, right, up, or down (no diagonal moves allowed). You cannot move outside the matrix boundaries, and wrap-around is not permitted.

An increasing path means that each subsequent cell you move to must have a strictly greater value than the current cell. For example, if you're at a cell with value `5`, you can only move to adjacent cells with values greater than `5`.

The goal is to find the maximum length among all possible increasing paths that can be formed starting from any cell in the matrix.

For instance, if you have a matrix:
```
9 9 4
6 6 8
2 1 1
```

Starting from the cell with value `1` (bottom row), you could form the path: `1 → 2 → 6 → 9`, which has length `4`. This would be the longest increasing path in this matrix.

The challenge involves exploring all possible starting positions and all possible paths from each position to determine which path achieves the maximum length.

## Intuition

The key insight is recognizing that this problem is about finding paths in a directed acyclic graph (DAG). Since we can only move to cells with strictly greater values, it's impossible to return to a previously visited cell in the same path - this naturally prevents cycles.

Think of each cell as a starting point for potential paths. From any cell `(i, j)`, we want to know: "What's the longest increasing path I can build starting from here?" This question has an optimal substructure property - if we know the longest path from each of our neighbors, we can determine the longest path from our current position.

Consider a cell with value `5`. If it has adjacent cells with values `7`, `8`, and `3`, we can only move to cells with `7` and `8`. If the longest path from the cell with `7` is length `3` and from the cell with `8` is length `2`, then the longest path from our current cell would be `1 + max(3, 2) = 4` (including the current cell itself).

This recursive relationship naturally leads to a DFS solution. However, naive DFS would repeatedly calculate the same paths. For instance, multiple cells might lead to the same cell, causing us to recalculate that cell's longest path multiple times. This is where memoization becomes crucial - once we've computed the longest path from a cell, we store it and reuse it whenever needed.

The beauty of this approach is that we're essentially performing dynamic programming on a graph structure. Each cell's result depends only on its neighbors with greater values, creating a dependency chain that we can solve recursively. By trying all possible starting positions and keeping track of the maximum, we find the overall longest increasing path in the matrix.

The `@cache` decorator (or memoization) transforms what would be an exponential time complexity problem into a linear one, as each cell is computed exactly once despite potentially being part of many different paths.

## Solution Approach

The solution implements a memoized depth-first search (DFS) approach to find the longest increasing path. Let's walk through the implementation step by step:

**Core Function Design:**
We define a recursive function `dfs(i, j)` that computes the length of the longest increasing path starting from position `(i, j)`. The `@cache` decorator automatically memoizes the results, ensuring each cell is computed only once.

**DFS Logic:**
For each cell `(i, j)`, the function:
1. Initializes `ans = 0` to track the maximum path length from neighboring cells
2. Explores all four directions (up, down, left, right) using the clever `pairwise((-1, 0, 1, 0, -1))` iteration
3. For each valid neighbor `(x, y)` that satisfies:
   - Within bounds: `0 <= x < m` and `0 <= y < n`
   - Has a greater value: `matrix[x][y] > matrix[i][j]`
4. Recursively calls `dfs(x, y)` and updates `ans` with the maximum path length found
5. Returns `ans + 1` (adding 1 for the current cell itself)

**Direction Traversal Technique:**
The code uses `pairwise((-1, 0, 1, 0, -1))` to generate direction pairs:
- `(-1, 0)`: move up
- `(0, 1)`: move right  
- `(1, 0)`: move down
- `(0, -1)`: move left

This compact representation avoids explicitly listing all four direction vectors.

**Main Function:**
The main function:
1. Extracts matrix dimensions `m` and `n`
2. Calls `dfs(i, j)` for every cell in the matrix
3. Returns the maximum value among all starting positions

**Why This Works:**
- The memoization ensures `O(m × n)` time complexity since each cell is computed exactly once
- The recursive structure naturally handles the dependency relationships - a cell's longest path depends on its valid neighbors' longest paths
- By trying all starting positions, we guarantee finding the global maximum

**Space Complexity:**
The solution uses `O(m × n)` space for:
- The memoization cache storing results for each cell
- The recursion stack (at most `O(m × n)` in the worst case of a strictly increasing spiral)

This approach elegantly combines DFS traversal with dynamic programming principles through memoization, making it both intuitive and efficient.

## Example Walkthrough

Let's trace through the algorithm with a small 3×3 matrix:

```
Matrix:
[1, 2, 3]
[6, 5, 4]
[7, 8, 9]
```

**Step 1: Initialize and Start DFS from Each Cell**

We'll call `dfs(i, j)` for every cell. Let's trace a few key calls to understand the process.

**Step 2: DFS from cell (0, 0) with value 1**
- Current value: 1
- Check all 4 neighbors:
  - Up (-1, 0): Out of bounds, skip
  - Right (0, 1): Value = 2 > 1 ✓ Call `dfs(0, 1)`
  - Down (1, 0): Value = 6 > 1 ✓ Call `dfs(1, 0)`
  - Left (0, -1): Out of bounds, skip
- Need to compute `dfs(0, 1)` and `dfs(1, 0)` first

**Step 3: DFS from cell (0, 1) with value 2**
- Current value: 2
- Check neighbors:
  - Up: Out of bounds
  - Right (0, 2): Value = 3 > 2 ✓ Call `dfs(0, 2)`
  - Down (1, 1): Value = 5 > 2 ✓ Call `dfs(1, 1)`
  - Left (0, 0): Value = 1 < 2 ✗ Skip

**Step 4: DFS from cell (0, 2) with value 3**
- Current value: 3
- Check neighbors:
  - Right: Out of bounds
  - Down (1, 2): Value = 4 > 3 ✓ Call `dfs(1, 2)`
  - Others don't satisfy the increasing condition
- `dfs(1, 2)` continues the chain...

**Step 5: Building the Path**

Following the complete recursion, we discover the longest path:
- Start at (0, 0): value = 1
- Move to (0, 1): value = 2
- Move to (0, 2): value = 3
- Move to (1, 2): value = 4
- Move to (1, 1): value = 5
- Move to (1, 0): value = 6
- Move to (2, 0): value = 7
- Move to (2, 1): value = 8
- Move to (2, 2): value = 9

**Step 6: Memoization in Action**

When we later call `dfs(1, 1)` from another starting point:
- The value is already cached as 5 (path: 5→6→7→8→9)
- We immediately return the cached value without recomputation

**Step 7: Final Result**

After checking all starting positions:
- `dfs(0, 0)` returns 9 (the complete path shown above)
- `dfs(2, 2)` returns 1 (no valid moves from value 9)
- Other cells return intermediate values

The maximum among all starting points is **9**.

**Key Observations:**
1. Each cell is computed only once due to memoization
2. The recursion naturally follows increasing values, preventing cycles
3. The algorithm explores all possible paths efficiently by reusing computed results

## Python Solution

```python
class Solution:
    def longestIncreasingPath(self, matrix: List[List[int]]) -> int:
        from functools import cache
      
        @cache
        def dfs(row: int, col: int) -> int:
            """
            Perform depth-first search from position (row, col) to find
            the longest increasing path starting from this cell.
          
            Args:
                row: Current row index
                col: Current column index
          
            Returns:
                Length of the longest increasing path from this cell
            """
            # Initialize the maximum path length from neighboring cells
            max_path_length = 0
          
            # Check all four directions: up, right, down, left
            directions = [(-1, 0), (0, 1), (1, 0), (0, -1)]
          
            for delta_row, delta_col in directions:
                # Calculate neighboring cell coordinates
                next_row = row + delta_row
                next_col = col + delta_col
              
                # Check if the neighbor is within bounds and has a greater value
                if (0 <= next_row < rows and 
                    0 <= next_col < cols and 
                    matrix[next_row][next_col] > matrix[row][col]):
                    # Recursively find the longest path from the neighbor
                    max_path_length = max(max_path_length, dfs(next_row, next_col))
          
            # Return the longest path including the current cell (+1)
            return max_path_length + 1
      
        # Get matrix dimensions
        rows = len(matrix)
        cols = len(matrix[0])
      
        # Find the maximum path length starting from each cell
        max_path = 0
        for i in range(rows):
            for j in range(cols):
                max_path = max(max_path, dfs(i, j))
      
        return max_path
```

## Time and Space Complexity

**Time Complexity:** `O(m * n)` where `m` is the number of rows and `n` is the number of columns in the matrix.

The algorithm uses depth-first search (DFS) with memoization. Each cell `(i, j)` in the matrix is visited and computed at most once due to the `@cache` decorator. When a cell is first visited, the DFS explores its four neighbors and returns the longest path starting from that cell. Once computed, the result is cached and subsequent calls return immediately in `O(1)` time.

Since we call `dfs(i, j)` for every cell in the matrix (through the `max()` comprehension), and each cell's result is computed exactly once, the total number of DFS computations is `O(m * n)`. For each cell's computation, we check at most 4 neighbors, which is `O(1)` work per cell. Therefore, the overall time complexity is `O(m * n)`.

**Space Complexity:** `O(m * n)`

The space complexity consists of two main components:
1. **Cache storage:** The `@cache` decorator stores the result for each unique `(i, j)` pair. In the worst case, all `m * n` cells will have their results cached, consuming `O(m * n)` space.
2. **Recursion call stack:** In the worst case, the DFS could traverse a path that includes all cells in the matrix (imagine a strictly increasing spiral pattern). This would create a recursion depth of `O(m * n)`.

Therefore, the total space complexity is `O(m * n)`.

## Common Pitfalls

### 1. **Forgetting to Memoize - Leading to Time Limit Exceeded**

One of the most critical pitfalls is implementing the DFS solution without memoization. Without caching results, the algorithm repeatedly recalculates the same paths, resulting in exponential time complexity.

**Problematic Code:**
```python
def longestIncreasingPath(self, matrix: List[List[int]]) -> int:
    def dfs(row: int, col: int) -> int:
        # NO MEMOIZATION - This will cause TLE!
        max_path_length = 0
        directions = [(-1, 0), (0, 1), (1, 0), (0, -1)]
      
        for delta_row, delta_col in directions:
            next_row = row + delta_row
            next_col = col + delta_col
          
            if (0 <= next_row < rows and 
                0 <= next_col < cols and 
                matrix[next_row][next_col] > matrix[row][col]):
                max_path_length = max(max_path_length, dfs(next_row, next_col))
      
        return max_path_length + 1
```

**Solution:**
Always use memoization either through `@cache` decorator or manual dictionary:
```python
from functools import cache

@cache
def dfs(row: int, col: int) -> int:
    # ... rest of the logic
```

Or with manual memoization:
```python
memo = {}
def dfs(row: int, col: int) -> int:
    if (row, col) in memo:
        return memo[(row, col)]
  
    # ... compute result ...
  
    memo[(row, col)] = result
    return result
```

### 2. **Incorrect Boundary Checking Order**

Another common mistake is checking the value comparison before verifying boundaries, which can cause IndexError.

**Problematic Code:**
```python
# WRONG: Checking value before boundaries
if (matrix[next_row][next_col] > matrix[row][col] and  # IndexError!
    0 <= next_row < rows and 
    0 <= next_col < cols):
```

**Solution:**
Always check boundaries first:
```python
# CORRECT: Check boundaries first
if (0 <= next_row < rows and 
    0 <= next_col < cols and 
    matrix[next_row][next_col] > matrix[row][col]):
```

### 3. **Using Wrong Comparison (Non-strict Inequality)**

The problem requires a strictly increasing path, but it's easy to accidentally use `>=` instead of `>`.

**Problematic Code:**
```python
# WRONG: Allows equal values
if matrix[next_row][next_col] >= matrix[row][col]:
    max_path_length = max(max_path_length, dfs(next_row, next_col))
```

**Solution:**
Use strict inequality:
```python
# CORRECT: Strictly greater values only
if matrix[next_row][next_col] > matrix[row][col]:
    max_path_length = max(max_path_length, dfs(next_row, next_col))
```

### 4. **Forgetting to Add 1 for Current Cell**

A subtle bug is forgetting to include the current cell in the path length count.

**Problematic Code:**
```python
def dfs(row: int, col: int) -> int:
    max_path_length = 0
    # ... check neighbors ...
    return max_path_length  # WRONG: Forgot to add 1 for current cell
```

**Solution:**
Always add 1 for the current cell:
```python
def dfs(row: int, col: int) -> int:
    max_path_length = 0
    # ... check neighbors ...
    return max_path_length + 1  # CORRECT: Include current cell
```

### 5. **Improper Handling of Base Case**

When a cell has no valid neighbors (all neighbors are smaller or equal), the function should return 1 (just the cell itself), not 0.

**Problematic Code:**
```python
def dfs(row: int, col: int) -> int:
    if no_valid_neighbors:  # pseudo-code
        return 0  # WRONG: Should be 1
```

**Solution:**
The correct implementation naturally handles this by returning `max_path_length + 1` where `max_path_length` starts at 0:
```python
def dfs(row: int, col: int) -> int:
    max_path_length = 0  # If no valid neighbors, this stays 0
    # ... check neighbors ...
    return max_path_length + 1  # Returns 1 when no valid neighbors
```

