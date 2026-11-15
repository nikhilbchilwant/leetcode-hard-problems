# 2088. Count Fertile Pyramids in a Land

> Source: https://algo.monster/liteproblems/2088

## Problem Statement

You have a rectangular grid of farmland with `m` rows and `n` columns. Each cell in the grid is either:
- **Fertile** (represented by `1`)
- **Barren** (represented by `0`)

The problem asks you to count two types of special patterns in this grid: **pyramids** and **inverse pyramids**.

**Pyramid Pattern:**
- Must contain more than 1 fertile cell
- Has an apex (top point) at position `(r, c)`
- Expands downward row by row
- For a pyramid of height `h` with apex at `(r, c)`:
  - Row `r` has 1 cell: `(r, c)`
  - Row `r+1` has 3 cells: `(r+1, c-1)`, `(r+1, c)`, `(r+1, c+1)`
  - Row `r+2` has 5 cells: `(r+2, c-2)` to `(r+2, c+2)`
  - And so on...
- All cells in the pyramid must be fertile

**Inverse Pyramid Pattern:**
- Must contain more than 1 fertile cell  
- Has an apex (bottom point) at position `(r, c)`
- Expands upward row by row
- For an inverse pyramid of height `h` with apex at `(r, c)`:
  - Row `r` has 1 cell: `(r, c)`
  - Row `r-1` has 3 cells: `(r-1, c-1)`, `(r-1, c)`, `(r-1, c+1)`
  - Row `r-2` has 5 cells: `(r-2, c-2)` to `(r-2, c+2)`
  - And so on...
- All cells in the inverse pyramid must be fertile

The mathematical formula for cells in a pyramid with apex at `(r, c)` and height `h`:
- Cell `(i, j)` is included if: `r ≤ i ≤ r + h - 1` and `c - (i - r) ≤ j ≤ c + (i - r)`

For an inverse pyramid with apex at `(r, c)` and height `h`:
- Cell `(i, j)` is included if: `r - h + 1 ≤ i ≤ r` and `c - (r - i) ≤ j ≤ c + (r - i)`

Your task is to count the **total number** of valid pyramids and inverse pyramids that can be found in the grid. A pyramid/inverse pyramid is valid if all its cells are fertile and it contains more than one cell.

## Intuition

The key insight is that we can use **[dynamic programming](/problems/dynamic_programming_intro)** to efficiently count pyramids by building upon smaller pyramid heights.

Think about how pyramids grow: if we have a pyramid of height `h` with apex at `(r, c)`, we can check if it can extend to height `h+1` by verifying three positions in the next row: `(r+h, c-h)`, `(r+h, c)`, and `(r+h, c+h)`. Each of these positions must themselves be apexes of pyramids of at least height `h`.

This leads us to define `f[i][j]` as the **maximum height of a pyramid with apex at position `(i, j)`**. For regular pyramids:
- If `grid[i][j] = 0`, then `f[i][j] = -1` (no pyramid possible)
- If we're at the bottom row or edges, `f[i][j] = 0` (single cell, not a valid pyramid)
- Otherwise, `f[i][j] = min(f[i+1][j-1], f[i+1][j], f[i+1][j+1]) + 1`

The reasoning behind the `min` operation: to form a pyramid of height `h+1` at `(i, j)`, we need the three positions below it to support at least a height of `h`. We take the minimum because we're limited by the smallest pyramid among the three supporting positions.

For example, if `f[i+1][j-1] = 2`, `f[i+1][j] = 3`, and `f[i+1][j+1] = 1`, then the maximum pyramid height at `(i, j)` is `1 + 1 = 2`, limited by the rightmost position.

The count of pyramids with apex at `(i, j)` equals `f[i][j]` because if we can build a pyramid of height `h`, we can also build pyramids of heights `h-1`, `h-2`, ..., `2`, and `1` (where height 1 means 2 rows total).

We process the grid **bottom-up** for regular pyramids (starting from the last row) and **top-down** for inverse pyramids (starting from the first row). This ensures that when we calculate `f[i][j]`, we've already computed the values it depends on.

The same logic applies to inverse pyramids, but we look upward instead of downward, using `f[i-1][j-1]`, `f[i-1][j]`, and `f[i-1][j+1]`.

By accumulating `f[i][j]` values for both pyramid types across all valid positions, we get the total count of all possible pyramidal plots in the grid.

**Learn more about [Dynamic Programming](/problems/dynamic_programming_intro) patterns.**

## Solution Approach

The solution implements a [dynamic programming](/problems/dynamic_programming_intro) approach with two passes through the grid - one for counting regular pyramids and one for counting inverse pyramids.

**Data Structure:**
- `f[i][j]`: A 2D array that stores the maximum height of pyramids/inverse pyramids with apex at position `(i, j)`

**Algorithm Implementation:**

**Pass 1: Count Regular Pyramids (Bottom-Up)**

We iterate through the grid from bottom to top (`i` from `m-1` to `0`) and left to right (`j` from `0` to `n-1`):

```python
for i in range(m - 1, -1, -1):
    for j in range(n):
        if grid[i][j] == 0:
            f[i][j] = -1  # Barren cell, no pyramid possible
        elif not (i == m - 1 or j == 0 or j == n - 1):
            # Check three supporting positions below
            f[i][j] = min(f[i + 1][j - 1], f[i + 1][j], f[i + 1][j + 1]) + 1
            ans += f[i][j]  # Add count of pyramids with apex at (i,j)
```

- If the cell is barren (`grid[i][j] == 0`), mark it as `-1`
- If we're at the bottom row or edges, we can't form a valid pyramid (needs at least 2 rows)
- Otherwise, calculate the maximum pyramid height using the formula: `f[i][j] = min(f[i+1][j-1], f[i+1][j], f[i+1][j+1]) + 1`
- Add `f[i][j]` to the answer (this counts all pyramids of heights 2 to `f[i][j]+1` with apex at `(i,j)`)

**Pass 2: Count Inverse Pyramids (Top-Down)**

We reuse the same `f` array and iterate from top to bottom (`i` from `0` to `m-1`):

```python
for i in range(m):
    for j in range(n):
        if grid[i][j] == 0:
            f[i][j] = -1  # Barren cell, no inverse pyramid possible
        elif i == 0 or j == 0 or j == n - 1:
            f[i][j] = 0  # Edge cases where no valid inverse pyramid exists
        else:
            # Check three supporting positions above
            f[i][j] = min(f[i - 1][j - 1], f[i - 1][j], f[i - 1][j + 1]) + 1
            ans += f[i][j]  # Add count of inverse pyramids with apex at (i,j)
```

- Similar logic but looking upward instead of downward
- For positions not on edges, calculate: `f[i][j] = min(f[i-1][j-1], f[i-1][j], f[i-1][j+1]) + 1`
- Add `f[i][j]` to accumulate the count of inverse pyramids

**Why This Works:**

The `min` operation ensures that we can only build a pyramid as tall as the shortest supporting pyramid allows. For example, if at position `(i,j)` we want to build a pyramid of height 3:
- We need the left diagonal `(i+1, j-1)` to support at least height 2
- We need the center `(i+1, j)` to support at least height 2  
- We need the right diagonal `(i+1, j+1)` to support at least height 2

If any of these positions has a smaller maximum height, it becomes our limiting factor.

**Time Complexity:** `O(m × n)` - we visit each cell twice
**Space Complexity:** `O(m × n)` - for the DP array `f`

## Example Walkthrough

Let me walk through a small example to illustrate the solution approach.

Consider this 4×3 grid:
```
1 1 1
1 1 1  
1 1 1
0 1 0
```

**Pass 1: Counting Regular Pyramids (Bottom-Up)**

We start from the bottom row and work our way up, calculating `f[i][j]` for each position.

Row 3 (bottom row):
- `f[3][0] = 0` (bottom row, no pyramids possible)
- `f[3][1] = 0` (bottom row)
- `f[3][2] = 0` (bottom row)

Row 2:
- `f[2][0] = 0` (left edge, no room for pyramid base)
- `f[2][1] = min(f[3][0], f[3][1], f[3][2]) + 1 = min(0, 0, 0) + 1 = 1`
  - This means we can form 1 pyramid of height 2 with apex at (2,1)
  - Pyramid includes: (2,1) as apex and (3,0), (3,1), (3,2) as base
  - But wait - (3,0) is barren! So actually `f[3][0] = -1`, making `f[2][1] = -1 + 1 = 0`
- `f[2][2] = 0` (right edge)

Let me recalculate with the barren cells properly marked:

Row 3 (with barren cells):
- `f[3][0] = -1` (barren)
- `f[3][1] = 0` (fertile but bottom row)
- `f[3][2] = -1` (barren)

Row 2:
- `f[2][0] = 0` (left edge)
- `f[2][1] = min(-1, 0, -1) + 1 = -1 + 1 = 0` (limited by barren cells)
- `f[2][2] = 0` (right edge)

Row 1:
- `f[1][0] = 0` (left edge)
- `f[1][1] = min(0, 0, 0) + 1 = 1`
  - We can form a pyramid with apex at (1,1) and base at row 2
  - This pyramid has cells: (1,1), (2,0), (2,1), (2,2)
  - Count added: 1
- `f[1][2] = 0` (right edge)

Row 0:
- `f[0][0] = 0` (left edge)
- `f[0][1] = min(0, 1, 0) + 1 = 0 + 1 = 1`
  - We can form a pyramid with apex at (0,1)
  - Count added: 1
- `f[0][2] = 0` (right edge)

Total regular pyramids: 2

**Pass 2: Counting Inverse Pyramids (Top-Down)**

Now we reset `f` and work from top to bottom.

Row 0 (top row):
- `f[0][0] = 0` (top row, no inverse pyramids possible)
- `f[0][1] = 0` (top row)
- `f[0][2] = 0` (top row)

Row 1:
- `f[1][0] = 0` (left edge)
- `f[1][1] = min(f[0][0], f[0][1], f[0][2]) + 1 = min(0, 0, 0) + 1 = 1`
  - We can form an inverse pyramid with apex at (1,1) and base at row 0
  - Count added: 1
- `f[1][2] = 0` (right edge)

Row 2:
- `f[2][0] = 0` (left edge)
- `f[2][1] = min(f[1][0], f[1][1], f[1][2]) + 1 = min(0, 1, 0) + 1 = 1`
  - We can form an inverse pyramid with apex at (2,1)
  - Count added: 1
- `f[2][2] = 0` (right edge)

Row 3:
- `f[3][0] = -1` (barren)
- `f[3][1] = min(0, 1, 0) + 1 = 1`
  - We can form an inverse pyramid with apex at (3,1)
  - Count added: 1
- `f[3][2] = -1` (barren)

Total inverse pyramids: 3

**Final Answer: 2 + 3 = 5 total pyramidal plots**

The key insight demonstrated here is how the DP values build upon each other - each position's maximum pyramid height depends on the minimum of its three supporting positions, ensuring all cells in the pyramid are fertile.

## Python Solution

```python
class Solution:
    def countPyramids(self, grid: List[List[int]]) -> int:
        # Get grid dimensions
        rows, cols = len(grid), len(grid[0])
      
        # dp[i][j] represents the maximum height of pyramid with apex at (i, j)
        dp = [[0] * cols for _ in range(rows)]
        total_pyramids = 0
      
        # Count downward pyramids (apex pointing down)
        # Process from bottom to top since we need values from rows below
        for row in range(rows - 1, -1, -1):
            for col in range(cols):
                if grid[row][col] == 0:
                    # Mark cells with 0 as invalid for pyramid formation
                    dp[row][col] = -1
                elif row == rows - 1 or col == 0 or col == cols - 1:
                    # Boundary cells cannot form pyramids (except size 0)
                    dp[row][col] = 0
                else:
                    # Calculate pyramid height based on three cells below
                    # Take minimum of lower-left, directly below, and lower-right
                    dp[row][col] = min(
                        dp[row + 1][col - 1], 
                        dp[row + 1][col], 
                        dp[row + 1][col + 1]
                    ) + 1
                    # Add number of pyramids with apex at current position
                    total_pyramids += dp[row][col]
      
        # Count upward pyramids (apex pointing up)
        # Process from top to bottom since we need values from rows above
        for row in range(rows):
            for col in range(cols):
                if grid[row][col] == 0:
                    # Mark cells with 0 as invalid for pyramid formation
                    dp[row][col] = -1
                elif row == 0 or col == 0 or col == cols - 1:
                    # Boundary cells cannot form pyramids
                    dp[row][col] = 0
                else:
                    # Calculate pyramid height based on three cells above
                    # Take minimum of upper-left, directly above, and upper-right
                    dp[row][col] = min(
                        dp[row - 1][col - 1], 
                        dp[row - 1][col], 
                        dp[row - 1][col + 1]
                    ) + 1
                    # Add number of pyramids with apex at current position
                    total_pyramids += dp[row][col]
      
        return total_pyramids
```

## Time and Space Complexity

**Time Complexity:** `O(m * n)` where `m` is the number of rows and `n` is the number of columns in the grid.

The algorithm consists of two main passes through the grid:
- First pass: iterates from bottom to top (`i` from `m-1` to `0`) and left to right (`j` from `0` to `n-1`), processing each cell once to count regular pyramids
- Second pass: iterates from top to bottom (`i` from `0` to `m-1`) and left to right (`j` from `0` to `n-1`), processing each cell once to count inverted pyramids

Each cell operation involves:
- Checking if the cell value is 0: `O(1)`
- Checking boundary conditions: `O(1)`
- Computing the minimum of three values: `O(1)`
- Updating the answer: `O(1)`

Since we visit each cell exactly twice with constant time operations per cell, the total time complexity is `O(2 * m * n) = O(m * n)`.

**Space Complexity:** `O(m * n)`

The algorithm uses:
- A 2D array `f` of size `m × n` to store the dynamic programming values: `O(m * n)`
- A few constant variables (`m`, `n`, `ans`, loop indices): `O(1)`

The dominant space usage comes from the 2D array `f`, making the overall space complexity `O(m * n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Boundary Handling for Pyramid Formation**

A common mistake is not properly handling the boundary conditions when determining if a pyramid can be formed at a given position. Many developers forget that:
- A pyramid needs space to expand both horizontally and vertically
- Edge cells (first/last column) cannot be apex points for pyramids of height > 1
- The bottom row cannot be an apex for regular pyramids, and the top row cannot be an apex for inverse pyramids

**Incorrect Implementation:**
```python
# Missing proper boundary checks
for row in range(rows - 1, -1, -1):
    for col in range(cols):
        if grid[row][col] == 0:
            dp[row][col] = -1
        else:
            # This will cause index out of bounds errors!
            dp[row][col] = min(
                dp[row + 1][col - 1], 
                dp[row + 1][col], 
                dp[row + 1][col + 1]
            ) + 1
```

**Correct Implementation:**
```python
for row in range(rows - 1, -1, -1):
    for col in range(cols):
        if grid[row][col] == 0:
            dp[row][col] = -1
        elif row == rows - 1 or col == 0 or col == cols - 1:
            # Properly handle boundaries where pyramids cannot form
            dp[row][col] = 0
        else:
            dp[row][col] = min(
                dp[row + 1][col - 1], 
                dp[row + 1][col], 
                dp[row + 1][col + 1]
            ) + 1
```

### 2. **Mixing Up Barren Cell Representation**

Another pitfall is inconsistent handling of barren cells in the DP array. Using `-1` to mark barren cells but then not handling this value correctly in the `min()` operation can lead to incorrect pyramid counts.

**Problematic Approach:**
```python
# If barren cells are marked as -1, this min operation gives wrong results
dp[row][col] = min(
    dp[row + 1][col - 1], 
    dp[row + 1][col], 
    dp[row + 1][col + 1]
) + 1
# If any supporting cell is -1, the min will be -1, giving dp[row][col] = 0
# This incorrectly allows pyramids over barren land
```

**Solution:**
Either:
1. Check if any supporting cell is barren before calculating:
```python
if (dp[row + 1][col - 1] == -1 or 
    dp[row + 1][col] == -1 or 
    dp[row + 1][col + 1] == -1):
    dp[row][col] = -1 if grid[row][col] == 0 else 0
else:
    dp[row][col] = min(
        dp[row + 1][col - 1], 
        dp[row + 1][col], 
        dp[row + 1][col + 1]
    ) + 1
```

2. Or initialize barren cells to 0 (simpler approach used in the solution):
```python
if grid[row][col] == 0:
    dp[row][col] = -1  # Mark as invalid
# Later, when -1 is encountered in min(), it naturally prevents pyramid formation
```

### 3. **Forgetting to Reset DP Array Between Passes**

Some implementations try to use separate DP arrays for regular and inverse pyramids but forget to properly initialize them, or they reuse the same array without understanding how the values should be interpreted.

**Incorrect Approach:**
```python
dp_regular = [[0] * cols for _ in range(rows)]
dp_inverse = dp_regular  # This creates a reference, not a copy!
# Modifying dp_inverse will also modify dp_regular
```

**Correct Approach:**
The provided solution cleverly reuses the same DP array because:
- The first pass processes rows from bottom to top
- The second pass processes rows from top to bottom
- When processing inverse pyramids at row `i`, we only need values from row `i-1`, which have already been computed in the second pass
- This works because we're overwriting values in a controlled manner

### 4. **Off-by-One Errors in Pyramid Counting**

Understanding what `dp[i][j]` represents is crucial. If `dp[i][j] = k`, it means:
- We can form pyramids of heights 1, 2, ..., k with apex at (i,j)
- Since pyramids must have height ≥ 2 (more than 1 cell), we count k pyramids, not k+1

**Wrong Interpretation:**
```python
# Incorrectly adding 1 to include single-cell "pyramids"
total_pyramids += dp[row][col] + 1
```

**Correct Interpretation:**
```python
# dp[row][col] already represents the count of valid pyramids
total_pyramids += dp[row][col]
```

