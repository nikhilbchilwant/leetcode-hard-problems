# 1289. Minimum Falling Path Sum II

> Source: https://algo.monster/liteproblems/1289

## Problem Statement

You are given an `n x n` integer matrix called `grid`. Your task is to find the minimum sum of a falling path with non-zero shifts.

A falling path with non-zero shifts means:
- You must select exactly one element from each row of the matrix
- No two elements selected from adjacent rows can be in the same column (this is the "non-zero shift" requirement)

In other words, if you pick an element from column `j` in row `i`, then in row `i+1`, you must pick an element from any column except column `j`.

For example, if you have a 3x3 matrix and you pick element at position `[0][1]` (row 0, column 1), then in row 1 you can pick from columns 0 or 2, but not column 1.

The goal is to find such a path from the first row to the last row that gives you the minimum possible sum of all selected elements.

## Intuition

When we look at this problem, we need to find an optimal path from top to bottom with a constraint - we can't pick elements from the same column in consecutive rows.

Let's think about this step by step. If we're at a certain position in row `i`, how did we get there? We must have come from some position in row `i-1`, but not from the same column. To reach any position optimally, we want to come from the position in the previous row that gives us the minimum cumulative sum.

This naturally leads us to [dynamic programming](/problems/dynamic_programming_intro). For each cell in the current row, we need to know: "What's the minimum sum to reach this cell from the top?"

To calculate this, we consider all possible previous positions (all columns except the current one in the previous row) and pick the one with the minimum cumulative sum. Then we add the current cell's value to get the minimum sum to reach the current position.

The key insight is that we don't need to track the entire path - we only need to know the minimum sum to reach each position in the previous row. This is because the decision at each row only depends on the immediately previous row due to the constraint.

Starting from the first row, we can build up these minimum sums row by row. For each cell `(i, j)`, we compute:
- The minimum sum among all positions in row `i-1` except column `j`
- Add the current cell's value `grid[i][j]` to this minimum

By the time we reach the last row, we'll have the minimum sum to reach each position in that row. The answer is simply the minimum among all these values.

The solution elegantly uses a rolling array technique where `f` stores the minimum sums for the previous row, and `g` computes the new minimum sums for the current row, which then becomes the new `f` for the next iteration.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution uses [dynamic programming](/problems/dynamic_programming_intro) with space optimization. Let's break down the implementation:

**[Dynamic Programming](/problems/dynamic_programming_intro) State Definition:**
We define `f[j]` to represent the minimum sum to reach column `j` in the current row. Initially, `f = [0] * n` represents that we haven't processed any rows yet.

**State Transition:**
For each row, we create a new array `g` that will store the minimum sums for the current row. The transition formula is:

`g[j] = row[j] + min(f[k] for all k ≠ j)`

This means for position `j` in the current row, we add:
- The current cell's value `row[j]`
- The minimum sum from all positions in the previous row except column `j`

**Implementation Details:**

1. **Initialize**: Start with `f = [0] * n`, representing zero cost before processing any rows.

2. **Process each row**: For each row in the grid:
   - Create `g = row[:]` - copy the current row values as the base
   - For each column `i` in the current row:
     - Add to `g[i]` the minimum value from `f[j]` where `j ≠ i`
     - The `min((f[j] for j in range(n) if j != i), default=0)` handles this
     - The `default=0` handles the edge case when `n=1`
   - Update `f = g` for the next iteration

3. **Return result**: After processing all rows, `f` contains the minimum sums to reach each position in the last row. Return `min(f)`.

**Space Optimization:**
Instead of maintaining a 2D DP table `f[i][j]` of size `n×n`, we use a rolling array technique with just two 1D arrays of size `n`. This reduces space complexity from `O(n²)` to `O(n)`.

**Time Complexity:** `O(n³)` - For each of the `n` rows, we process `n` columns, and for each column, we find the minimum among `n-1` values.

**Space Complexity:** `O(n)` - We only maintain arrays `f` and `g` of size `n`.

## Example Walkthrough

Let's walk through a small example with a 3×3 matrix:

```
grid = [[1, 2, 3],
        [4, 5, 6],
        [7, 8, 9]]
```

**Initial State:**
- `f = [0, 0, 0]` (no cost before processing any rows)

**Processing Row 0: [1, 2, 3]**
- Create `g = [1, 2, 3]` (copy of current row)
- For column 0: `g[0] = 1 + min(f[1], f[2]) = 1 + min(0, 0) = 1`
- For column 1: `g[1] = 2 + min(f[0], f[2]) = 2 + min(0, 0) = 2`
- For column 2: `g[2] = 3 + min(f[0], f[1]) = 3 + min(0, 0) = 3`
- Update: `f = [1, 2, 3]`

**Processing Row 1: [4, 5, 6]**
- Create `g = [4, 5, 6]` (copy of current row)
- For column 0: `g[0] = 4 + min(f[1], f[2]) = 4 + min(2, 3) = 4 + 2 = 6`
  - Can't come from column 0, must come from column 1 or 2
- For column 1: `g[1] = 5 + min(f[0], f[2]) = 5 + min(1, 3) = 5 + 1 = 6`
  - Can't come from column 1, must come from column 0 or 2
- For column 2: `g[2] = 6 + min(f[0], f[1]) = 6 + min(1, 2) = 6 + 1 = 7`
  - Can't come from column 2, must come from column 0 or 1
- Update: `f = [6, 6, 7]`

**Processing Row 2: [7, 8, 9]**
- Create `g = [7, 8, 9]` (copy of current row)
- For column 0: `g[0] = 7 + min(f[1], f[2]) = 7 + min(6, 7) = 7 + 6 = 13`
- For column 1: `g[1] = 8 + min(f[0], f[2]) = 8 + min(6, 7) = 8 + 6 = 14`
- For column 2: `g[2] = 9 + min(f[0], f[1]) = 9 + min(6, 6) = 9 + 6 = 15`
- Update: `f = [13, 14, 15]`

**Final Result:**
- `min(f) = min(13, 14, 15) = 13`

The minimum path is: `grid[0][0]=1 → grid[1][1]=5 → grid[2][0]=7`, giving sum = 1 + 5 + 7 = 13.

Another optimal path exists: `grid[0][1]=2 → grid[1][0]=4 → grid[2][0]=7`, giving sum = 2 + 4 + 7 = 13.

## Python Solution

```python
class Solution:
    def minFallingPathSum(self, grid: List[List[int]]) -> int:
        """
        Find the minimum sum of a falling path with non-zero shifts.
        A falling path starts from any element in the first row and chooses 
        one element from each row. The next row's choice must be in a different 
        column from the previous row.
      
        Args:
            grid: n x n integer matrix
          
        Returns:
            The minimum sum of a falling path through the grid
        """
        n = len(grid)
      
        # dp[i] represents the minimum path sum ending at column i of the previous row
        dp = [0] * n
      
        # Process each row of the grid
        for row in grid:
            # Create a new dp array for the current row
            new_dp = row[:]  # Copy current row values as base costs
          
            # For each column in the current row
            for col in range(n):
                # Add the minimum from previous row, excluding the same column
                # This ensures we don't pick from the same column as previous row
                min_prev = min((dp[prev_col] for prev_col in range(n) if prev_col != col), default=0)
                new_dp[col] += min_prev
          
            # Update dp for the next iteration
            dp = new_dp
      
        # Return the minimum value from the last row
        return min(dp)
```

## Time and Space Complexity

**Time Complexity:** `O(n³)`

The algorithm iterates through each row of the grid (`n` rows). For each row, it processes each element (`n` elements), and for each element `i`, it computes the minimum of all previous row values except position `i` (which takes `O(n)` time). Therefore, the total time complexity is `O(n × n × n) = O(n³)`.

**Space Complexity:** `O(n)`

The algorithm uses a rolling array optimization technique. Instead of maintaining a 2D DP table of size `n × n`, it only keeps track of:
- Array `f` of size `n` to store the minimum falling path sums from the previous row
- Array `g` of size `n` (created as `row[:]`) to compute the current row's values

Since only two 1D arrays of size `n` are used at any time, and the algorithm alternates between them (rolling array approach), the space complexity is optimized from `O(n²)` to `O(n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Inefficient O(n³) Time Complexity**
The current solution has a time complexity of O(n³) because for each cell (n² cells total), we're finding the minimum among n-1 values. This becomes problematic for large matrices.

**The Pitfall:**
```python
# Inefficient: For each column, we iterate through all other columns
for col in range(n):
    min_prev = min((dp[prev_col] for prev_col in range(n) if prev_col != col), default=0)
```

**Solution - Optimize to O(n²):**
Instead of finding the minimum for each column separately, we can precompute:
- The overall minimum value from the previous row
- The second minimum value from the previous row
- Track which column contains the minimum

```python
class Solution:
    def minFallingPathSum(self, grid: List[List[int]]) -> int:
        n = len(grid)
      
        # Handle edge case
        if n == 1:
            return grid[0][0]
      
        # Initialize with first row
        dp = grid[0][:]
      
        # Process remaining rows
        for i in range(1, n):
            # Find first minimum and second minimum from previous row
            min1 = min2 = float('inf')
            min1_idx = -1
          
            for j in range(n):
                if dp[j] < min1:
                    min2 = min1
                    min1 = dp[j]
                    min1_idx = j
                elif dp[j] < min2:
                    min2 = dp[j]
          
            # Build new dp array
            new_dp = [0] * n
            for j in range(n):
                # If current column is same as min column, use second min
                if j == min1_idx:
                    new_dp[j] = grid[i][j] + min2
                else:
                    new_dp[j] = grid[i][j] + min1
          
            dp = new_dp
      
        return min(dp)
```

### 2. **Edge Case: Single Column Matrix (n=1)**
When n=1, there's only one column, making it impossible to choose different columns in consecutive rows.

**The Pitfall:**
The original code uses `default=0` which incorrectly handles this case:
```python
min_prev = min((dp[prev_col] for prev_col in range(n) if prev_col != col), default=0)
```
When n=1 and col=0, the generator is empty, so it returns 0, which may not be correct.

**Solution:**
Handle n=1 as a special case:
```python
if n == 1:
    return grid[0][0]  # Only one element in the grid
```

### 3. **Unnecessary Array Copying**
The line `new_dp = row[:]` creates a copy of the row which might be confusing and slightly inefficient.

**The Pitfall:**
```python
new_dp = row[:]  # Creates a copy but might be unclear why
```

**Solution - More explicit initialization:**
```python
new_dp = [0] * n
for col in range(n):
    new_dp[col] = grid[row_idx][col] + min_prev_value
```

### 4. **Variable Naming Confusion**
Using generic names like `f`, `g` can make the code harder to understand and maintain.

**Solution:**
Use descriptive variable names:
- `prev_row_min_sums` instead of `f`
- `curr_row_min_sums` instead of `g`
- `col` or `curr_col` instead of `i` or `j`

