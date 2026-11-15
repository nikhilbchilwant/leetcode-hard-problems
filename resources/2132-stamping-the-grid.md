# 2132. Stamping the Grid

> Source: https://algo.monster/liteproblems/2132

## Problem Statement

You have a binary matrix `grid` of size `m x n` where each cell contains either `0` (empty) or `1` (occupied).

You're also given stamps with dimensions `stampHeight x stampWidth`. Your task is to determine if you can place these stamps on the grid following these rules:

1. **All empty cells (0s) must be covered** by at least one stamp
2. **No occupied cells (1s) can be covered** by any stamp
3. You can use **unlimited stamps**
4. Stamps **can overlap** with each other
5. Stamps **cannot be rotated**
6. Stamps must be placed **completely inside** the grid boundaries

The problem asks you to return `true` if it's possible to cover all empty cells without covering any occupied cells, and `false` otherwise.

For example, if you have a grid with some empty cells and some occupied cells, you need to strategically place stamps of the given size such that:
- Every `0` in the original grid gets covered by at least one stamp
- No `1` in the original grid gets covered by any stamp
- Each stamp placement must fit entirely within the grid (no part of the stamp can go outside the boundaries)

The key challenge is finding valid positions to place stamps that collectively cover all empty spaces while avoiding all occupied spaces.

## Intuition

The key insight is that we need to solve two sub-problems efficiently:
1. Where can we legally place stamps?
2. After placing all possible stamps, are all empty cells covered?

For the first sub-problem, a stamp can only be placed at position `(i, j)` if the entire `stampHeight × stampWidth` rectangle starting from that position contains no occupied cells (all 0s). Checking this naively for every position would be inefficient - we'd need to examine every cell in the rectangle for each potential stamp position.

This is where **2D prefix sums** come in. If we precompute the cumulative count of occupied cells (1s) for every sub-rectangle from `(0, 0)` to `(i, j)`, we can then check any rectangular region in constant time. For a rectangle from `(i, j)` to `(x, y)`, the number of occupied cells equals:
`s[x][y] - s[x][j-1] - s[i-1][y] + s[i-1][j-1]`

If this value is 0, we can place a stamp at `(i, j)`.

For the second sub-problem, we need to track which cells get covered by stamps. A naive approach would be to mark each cell individually for every stamp placed, but this is inefficient when stamps overlap.

This is where **2D difference arrays** shine. Instead of updating every cell in a rectangle when placing a stamp, we only update the corners of the rectangle in a difference array. Specifically, for a rectangle from `(i, j)` to `(x, y)`:
- Add 1 at `(i, j)` (start of coverage)
- Subtract 1 at `(i, y+1)` (end of horizontal coverage)
- Subtract 1 at `(x+1, j)` (end of vertical coverage)  
- Add 1 at `(x+1, y+1)` (restore after both ends)

After marking all stamps this way, we compute the [prefix sum](/problems/subarray_sum) of the difference array to get the actual coverage count for each cell. Any empty cell (0 in original grid) that has 0 coverage means it wasn't reached by any stamp, making the task impossible.

The elegance of this solution lies in using prefix sums twice - once to efficiently check stamp validity, and once more (on the difference array) to efficiently compute coverage.

**Learn more about [Greedy](/problems/greedy_intro) and [Prefix Sum](/problems/subarray_sum) patterns.**

## Solution Approach

The implementation uses two main techniques: 2D prefix sums for checking stamp validity and 2D difference arrays for tracking coverage.

**Step 1: Build 2D [Prefix Sum](/problems/subarray_sum) Array**

First, we create a [prefix sum](/problems/subarray_sum) array `s` with dimensions `(m+1) × (n+1)` to avoid boundary checks. Each `s[i][j]` stores the count of occupied cells (1s) in the rectangle from `(1,1)` to `(i,j)`:

```python
s[i][j] = s[i-1][j] + s[i][j-1] - s[i-1][j-1] + grid[i-1][j-1]
```

This formula adds the counts from the cell above and to the left, subtracts the overlap (top-left diagonal), and adds the current cell's value.

**Step 2: Initialize 2D Difference Array**

We create a difference array `d` with dimensions `(m+2) × (n+2)` to handle boundary updates safely. This array will track how many stamps cover each cell.

**Step 3: Try Placing Stamps**

For each possible stamp position `(i, j)` where the stamp would fit within bounds:
- Calculate the bottom-right corner: `(x, y) = (i + stampHeight - 1, j + stampWidth - 1)`
- Check if the rectangular region contains any occupied cells using the [prefix sum](/problems/subarray_sum):
  ```python
  occupied_count = s[x][y] - s[x][j-1] - s[i-1][y] + s[i-1][j-1]
  ```
- If `occupied_count == 0`, we can place a stamp here. Update the difference array:
  ```python
  d[i][j] += 1
  d[i][y+1] -= 1
  d[x+1][j] -= 1
  d[x+1][y+1] += 1
  ```

**Step 4: Compute Actual Coverage**

Convert the difference array to actual coverage counts using 2D [prefix sum](/problems/subarray_sum):
```python
d[i][j] += d[i-1][j] + d[i][j-1] - d[i-1][j-1]
```

**Step 5: Verify All Empty Cells Are Covered**

Iterate through the original grid. If any cell is empty (`grid[i-1][j-1] == 0`) but has zero coverage (`d[i][j] == 0`), return `false`. If all empty cells are covered, return `true`.

The time complexity is `O(m × n)` for all operations, and space complexity is `O(m × n)` for the auxiliary arrays. The solution efficiently handles the constraint checking and coverage tracking without redundant computations.

## Example Walkthrough

Let's walk through a small example to illustrate the solution approach.

**Given:**
- Grid (3×4):
  ```
  [0, 0, 0, 0]
  [1, 0, 0, 0]
  [0, 0, 0, 1]
  ```
- Stamp size: 2×2

**Step 1: Build 2D Prefix Sum Array**

We create a prefix sum array `s` (4×5 to include padding):
```
s = [0, 0, 0, 0, 0]
    [0, 0, 0, 0, 0]
    [0, 1, 1, 1, 1]
    [0, 1, 1, 1, 2]
```

For example, `s[3][4] = 2` means there are 2 occupied cells in the rectangle from (1,1) to (3,4).

**Step 2: Try Placing Stamps**

We check each possible 2×2 stamp position:

- Position (1,1): Rectangle covers cells (1,1) to (2,2)
  - Occupied count = s[2][2] - s[2][0] - s[0][2] + s[0][0] = 0 - 0 - 0 + 0 = 0 ✓
  - Can place stamp here!

- Position (1,2): Rectangle covers cells (1,2) to (2,3)
  - Occupied count = s[2][3] - s[2][1] - s[0][3] + s[0][1] = 0 - 0 - 0 + 0 = 0 ✓
  - Can place stamp here!

- Position (1,3): Rectangle covers cells (1,3) to (2,4)
  - Occupied count = s[2][4] - s[2][2] - s[0][4] + s[0][2] = 0 - 0 - 0 + 0 = 0 ✓
  - Can place stamp here!

- Position (2,1): Rectangle covers cells (2,1) to (3,2)
  - Occupied count = s[3][2] - s[3][0] - s[1][2] + s[1][0] = 1 - 0 - 0 + 0 = 1 ✗
  - Cannot place (contains occupied cell at (2,1))

- Position (2,2): Rectangle covers cells (2,2) to (3,3)
  - Occupied count = s[3][3] - s[3][1] - s[1][3] + s[1][1] = 1 - 1 - 0 + 0 = 0 ✓
  - Can place stamp here!

- Position (2,3): Rectangle covers cells (2,3) to (3,4)
  - Occupied count = s[3][4] - s[3][2] - s[1][4] + s[1][2] = 2 - 1 - 0 + 0 = 1 ✗
  - Cannot place (contains occupied cell at (3,4))

**Step 3: Update Difference Array**

For each valid stamp position, we update the difference array `d`:

After placing stamp at (1,1):
```
d[1][1] += 1, d[1][3] -= 1, d[3][1] -= 1, d[3][3] += 1
```

After placing all valid stamps (at positions (1,1), (1,2), (1,3), (2,2)):
```
d = [0, 0, 0, 0, 0, 0]
    [0, 1, 1, 0,-1, 0]
    [0, 0, 1, 0,-1, 0]
    [0,-1,-1, 1, 1, 0]
    [0, 0, 0, 0, 0, 0]
```

**Step 4: Compute Actual Coverage**

Convert difference array to actual coverage using prefix sums:
```
coverage = [0, 0, 0, 0, 0, 0]
           [0, 1, 2, 2, 1, 0]
           [0, 1, 3, 3, 1, 0]
           [0, 0, 1, 1, 0, 0]
           [0, 0, 0, 0, 0, 0]
```

**Step 5: Verify Coverage**

Check each cell in the original grid:
- (1,1): empty, coverage=1 ✓
- (1,2): empty, coverage=2 ✓
- (1,3): empty, coverage=2 ✓
- (1,4): empty, coverage=1 ✓
- (2,1): occupied, skip
- (2,2): empty, coverage=3 ✓
- (2,3): empty, coverage=3 ✓
- (2,4): empty, coverage=1 ✓
- (3,1): empty, coverage=0 ✗

The cell at (3,1) is empty but has no coverage, so we return `false`. It's impossible to cover all empty cells with 2×2 stamps while avoiding occupied cells.

## Python Solution

```python
class Solution:
    def possibleToStamp(
        self, grid: List[List[int]], stampHeight: int, stampWidth: int
    ) -> bool:
        """
        Check if all empty cells (0s) in the grid can be covered by stamps.
        A stamp can only be placed if the entire stamp area contains no occupied cells (1s).
      
        Args:
            grid: 2D binary grid where 0 = empty, 1 = occupied
            stampHeight: Height of the stamp
            stampWidth: Width of the stamp
      
        Returns:
            True if all empty cells can be covered by stamps, False otherwise
        """
        rows, cols = len(grid), len(grid[0])
      
        # Build prefix sum array to quickly check if a region contains any occupied cells
        # prefix_sum[i][j] = sum of all grid values from (0,0) to (i-1,j-1)
        prefix_sum = [[0] * (cols + 1) for _ in range(rows + 1)]
      
        for row_idx, row in enumerate(grid, 1):
            for col_idx, cell_value in enumerate(row, 1):
                # Calculate prefix sum using inclusion-exclusion principle
                prefix_sum[row_idx][col_idx] = (
                    prefix_sum[row_idx - 1][col_idx] 
                    + prefix_sum[row_idx][col_idx - 1] 
                    - prefix_sum[row_idx - 1][col_idx - 1] 
                    + cell_value
                )
      
        # Create a 2D difference array to track stamp coverage
        # Using difference array technique for efficient range updates
        stamp_coverage_diff = [[0] * (cols + 2) for _ in range(rows + 2)]
      
        # Try placing stamps at all valid positions
        for top_row in range(1, rows - stampHeight + 2):
            for left_col in range(1, cols - stampWidth + 2):
                # Calculate bottom-right corner of the stamp
                bottom_row = top_row + stampHeight - 1
                right_col = left_col + stampWidth - 1
              
                # Check if stamp area is completely empty using prefix sum
                occupied_count = (
                    prefix_sum[bottom_row][right_col] 
                    - prefix_sum[bottom_row][left_col - 1] 
                    - prefix_sum[top_row - 1][right_col] 
                    + prefix_sum[top_row - 1][left_col - 1]
                )
              
                if occupied_count == 0:
                    # Mark this stamp placement using 2D difference array
                    # This efficiently marks the entire stamp area
                    stamp_coverage_diff[top_row][left_col] += 1
                    stamp_coverage_diff[top_row][right_col + 1] -= 1
                    stamp_coverage_diff[bottom_row + 1][left_col] -= 1
                    stamp_coverage_diff[bottom_row + 1][right_col + 1] += 1
      
        # Convert difference array to actual coverage count and check validity
        for row_idx, row in enumerate(grid, 1):
            for col_idx, cell_value in enumerate(row, 1):
                # Apply 2D prefix sum to get actual coverage count from difference array
                stamp_coverage_diff[row_idx][col_idx] += (
                    stamp_coverage_diff[row_idx - 1][col_idx] 
                    + stamp_coverage_diff[row_idx][col_idx - 1] 
                    - stamp_coverage_diff[row_idx - 1][col_idx - 1]
                )
              
                # If cell is empty but not covered by any stamp, it's impossible
                if cell_value == 0 and stamp_coverage_diff[row_idx][col_idx] == 0:
                    return False
      
        return True
```

## Time and Space Complexity

The time complexity is `O(m × n)`, where `m` is the number of rows and `n` is the number of columns in the grid.

- The first nested loop computes the 2D prefix sum array `s`, iterating through all `m × n` cells once: `O(m × n)`
- The second nested loop checks all possible stamp positions. In the worst case, this iterates through approximately `(m - stampHeight + 1) × (n - stampWidth + 1)` positions, which is still bounded by `O(m × n)`
- The third nested loop applies the 2D difference array technique to compute the final coverage, iterating through all `m × n` cells once: `O(m × n)`

Since all operations are performed sequentially, the overall time complexity is `O(m × n)`.

The space complexity is `O(m × n)`.

- The prefix sum array `s` has dimensions `(m + 1) × (n + 1)`: `O(m × n)`
- The difference array `d` has dimensions `(m + 2) × (n + 2)`: `O(m × n)`

Both auxiliary arrays use space proportional to the size of the input grid, resulting in a total space complexity of `O(m × n)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Boundary Handling in Prefix Sum Queries**

A common mistake is miscalculating the boundaries when querying the prefix sum array to check if a stamp placement is valid. The indices for the rectangle query can easily be off by one.

**Pitfall Example:**
```python
# Incorrect: Using wrong indices for the stamp area check
occupied_count = (
    prefix_sum[bottom_row + 1][right_col + 1]  # Wrong!
    - prefix_sum[bottom_row + 1][left_col]
    - prefix_sum[top_row][right_col + 1]
    + prefix_sum[top_row][left_col]
)
```

**Solution:**
Remember that `prefix_sum[i][j]` represents the sum from `(0,0)` to `(i-1,j-1)` in the original grid. When checking a stamp from `(top_row, left_col)` to `(bottom_row, right_col)` in 1-indexed coordinates:
```python
# Correct: Proper boundary calculation
occupied_count = (
    prefix_sum[bottom_row][right_col] 
    - prefix_sum[bottom_row][left_col - 1] 
    - prefix_sum[top_row - 1][right_col] 
    + prefix_sum[top_row - 1][left_col - 1]
)
```

### 2. **Forgetting to Check if Stamp Fits Within Grid**

Another pitfall is attempting to place stamps that would extend beyond the grid boundaries, leading to index out of bounds errors or incorrect coverage calculations.

**Pitfall Example:**
```python
# Incorrect: Iterating through all positions without checking bounds
for top_row in range(1, rows + 1):  # Goes too far!
    for left_col in range(1, cols + 1):  # Goes too far!
```

**Solution:**
Ensure the loop only considers positions where the entire stamp fits:
```python
# Correct: Only iterate where stamps can fully fit
for top_row in range(1, rows - stampHeight + 2):
    for left_col in range(1, cols - stampWidth + 2):
```

### 3. **Misunderstanding the Difference Array Update Pattern**

The 2D difference array update pattern is counterintuitive. A common mistake is applying the wrong signs or positions for the four corner updates.

**Pitfall Example:**
```python
# Incorrect: Wrong signs or positions in difference array update
stamp_coverage_diff[top_row][left_col] += 1
stamp_coverage_diff[top_row][right_col] -= 1  # Should be right_col + 1
stamp_coverage_diff[bottom_row][left_col] -= 1  # Should be bottom_row + 1
stamp_coverage_diff[bottom_row][right_col] += 1  # Wrong position and sign
```

**Solution:**
The correct 2D difference array update follows this pattern:
- Top-left: +1
- Top-right (one column after stamp): -1
- Bottom-left (one row after stamp): -1
- Bottom-right (one row and column after stamp): +1

```python
# Correct: Proper 2D difference array update
stamp_coverage_diff[top_row][left_col] += 1
stamp_coverage_diff[top_row][right_col + 1] -= 1
stamp_coverage_diff[bottom_row + 1][left_col] -= 1
stamp_coverage_diff[bottom_row + 1][right_col + 1] += 1
```

### 4. **Not Handling Edge Cases Where Stamp is Larger Than Grid**

If the stamp dimensions are larger than the grid dimensions, the solution should immediately return false if there are any empty cells.

**Solution:**
Add an early check:
```python
# Check if stamp is too large for the grid
if stampHeight > rows or stampWidth > cols:
    # If stamp doesn't fit and there are empty cells, impossible
    return all(cell == 1 for row in grid for cell in row)
```

