# 2371. Minimize Maximum Value in a Grid

> Source: https://algo.monster/liteproblems/2371

## Problem Statement

You have an `m x n` matrix filled with distinct positive integers. Your task is to replace each number in the matrix with a new positive integer while following two important rules:

1. **Preserve relative order**: If one number is larger than another in the same row or column in the original matrix, it must remain larger in the new matrix. For example, if `grid[r1][c1] > grid[r2][c2]` and they share the same row (`r1 == r2`) or same column (`c1 == c2`), then after replacement, the relationship `grid[r1][c1] > grid[r2][c2]` must still hold true.

2. **Minimize the maximum**: The largest number in the resulting matrix should be as small as possible.

For example, given `grid = [[2, 4, 5], [7, 3, 9]]`:
- In row 0: the order is 2 < 4 < 5
- In row 1: the order is 3 < 7 < 9
- In column 0: the order is 2 < 7
- In column 1: the order is 3 < 4
- In column 2: the order is 5 < 9

A valid replacement could be `[[1, 2, 3], [2, 1, 4]]` where:
- Row 0 maintains order: 1 < 2 < 3
- Row 1 maintains order: 1 < 2 < 4
- Column 0 maintains order: 1 < 2
- Column 1 maintains order: 1 < 2
- Column 2 maintains order: 3 < 4

## Intuition

The key insight is that we need to assign the smallest possible values while maintaining relative ordering within rows and columns. Think of this problem as a ranking problem - we want to give each element a rank that respects its position relative to other elements in its row and column.

If we process elements from smallest to largest in the original matrix, we can build up our answer incrementally. Why does this work? When we encounter the smallest element, we can safely assign it the value 1 (or the minimum required based on constraints). For each subsequent element, we only need to ensure it's larger than any previously processed elements in its row or column.

Consider what happens when we reach element at position `(i, j)`:
- All smaller elements have already been assigned values
- Some of these smaller elements might be in row `i` or column `j`
- Our new value must be larger than all of these previously assigned values

To efficiently track this, we maintain two arrays:
- `row_max[i]`: the largest value assigned so far in row `i`
- `col_max[j]`: the largest value assigned so far in column `j`

The minimum valid value for position `(i, j)` is then `max(row_max[i], col_max[j]) + 1`. This ensures our new value is larger than everything in its row and column that came before it.

By processing elements in sorted order and always choosing the minimum valid value, we guarantee:
1. The relative ordering is preserved (smaller original values get smaller new values)
2. The maximum value in the result is minimized (we use the smallest possible value at each step)

This greedy approach works because once we assign a value, we never need to change it - the sorted processing order ensures all dependencies are handled correctly.

**Learn more about [Union Find](/problems/dsu_intro), [Graph](/problems/graph_intro), [Topological Sort](/problems/topo_intro) and [Sorting](/problems/sorting_summary) patterns.**

## Solution Approach

The implementation follows a greedy algorithm with [sorting](/problems/sorting_summary) to efficiently assign minimum values while preserving relative order:

1. **Extract and Sort Elements**: First, we create a list of tuples containing `(value, row_index, col_index)` for every element in the grid. This allows us to track both the original value and position of each element. We then sort this list by value in ascending order.

```python
nums = [(v, i, j) for i, row in enumerate(grid) for j, v in enumerate(row)]
nums.sort()
```

2. **Initialize Tracking Arrays**: We maintain two arrays to track the maximum value assigned so far in each row and column:
   - `row_max[i]` stores the maximum value assigned in row `i`
   - `col_max[j]` stores the maximum value assigned in column `j`
   - Both are initialized to 0

```python
row_max = [0] * m
col_max = [0] * n
ans = [[0] * n for _ in range(m)]
```

3. **Process Elements in Sorted Order**: We iterate through the sorted elements. For each element at position `(i, j)`:
   - Calculate the minimum valid value as `max(row_max[i], col_max[j]) + 1`
   - This ensures the new value is greater than all previously assigned values in its row and column
   - Assign this value to `ans[i][j]`
   - Update both `row_max[i]` and `col_max[j]` to this new value

```python
for _, i, j in nums:
    ans[i][j] = max(row_max[i], col_max[j]) + 1
    row_max[i] = col_max[j] = ans[i][j]
```

The algorithm's correctness relies on processing elements in ascending order of their original values. This guarantees that when we assign a value to position `(i, j)`, all elements that should be smaller (based on the original matrix) have already been processed and assigned smaller values.

**Time Complexity**: `O(m*n*log(m*n))` due to [sorting](/problems/sorting_summary), where `m` and `n` are the dimensions of the grid.

**Space Complexity**: `O(m*n)` for storing the sorted list and the answer matrix.

## Example Walkthrough

Let's trace through a small example to illustrate the solution approach.

**Input**: `grid = [[3, 1], [2, 5]]`

**Step 1: Extract and Sort Elements**
We create tuples of (value, row, col) and sort by value:
- (3, 0, 0) from grid[0][0]
- (1, 0, 1) from grid[0][1]
- (2, 1, 0) from grid[1][0]
- (5, 1, 1) from grid[1][1]

After sorting: `[(1, 0, 1), (2, 1, 0), (3, 0, 0), (5, 1, 1)]`

**Step 2: Initialize Tracking Arrays**
- `row_max = [0, 0]` (one entry per row)
- `col_max = [0, 0]` (one entry per column)
- `ans = [[0, 0], [0, 0]]` (result matrix)

**Step 3: Process Elements in Sorted Order**

*Processing (1, 0, 1) - smallest value at position [0][1]:*
- Current row_max[0] = 0, col_max[1] = 0
- Assign: ans[0][1] = max(0, 0) + 1 = 1
- Update: row_max[0] = 1, col_max[1] = 1
- State: ans = [[0, 1], [0, 0]], row_max = [1, 0], col_max = [0, 1]

*Processing (2, 1, 0) - value 2 at position [1][0]:*
- Current row_max[1] = 0, col_max[0] = 0
- Assign: ans[1][0] = max(0, 0) + 1 = 1
- Update: row_max[1] = 1, col_max[0] = 1
- State: ans = [[0, 1], [1, 0]], row_max = [1, 1], col_max = [1, 1]

*Processing (3, 0, 0) - value 3 at position [0][0]:*
- Current row_max[0] = 1, col_max[0] = 1
- Assign: ans[0][0] = max(1, 1) + 1 = 2
- Update: row_max[0] = 2, col_max[0] = 2
- State: ans = [[2, 1], [1, 0]], row_max = [2, 1], col_max = [2, 1]

*Processing (5, 1, 1) - largest value at position [1][1]:*
- Current row_max[1] = 1, col_max[1] = 1
- Assign: ans[1][1] = max(1, 1) + 1 = 2
- Update: row_max[1] = 2, col_max[1] = 2
- State: ans = [[2, 1], [1, 2]], row_max = [2, 2], col_max = [2, 2]

**Final Result**: `[[2, 1], [1, 2]]`

**Verification**:
- Original row 0: 3 > 1, Result row 0: 2 > 1 ✓
- Original row 1: 2 < 5, Result row 1: 1 < 2 ✓
- Original col 0: 3 > 2, Result col 0: 2 > 1 ✓
- Original col 1: 1 < 5, Result col 1: 1 < 2 ✓

The maximum value in the result is 2, which is the minimum possible while preserving all relative orderings.

## Python Solution

```python
class Solution:
    def minScore(self, grid: List[List[int]]) -> List[List[int]]:
        # Get dimensions of the grid
        m, n = len(grid), len(grid[0])

        # Create a list of tuples containing (value, row_index, col_index) for all cells
        # This allows us to process cells in ascending order of their values
        cells_with_positions = [(value, row_idx, col_idx)
                                for row_idx, row in enumerate(grid)
                                for col_idx, value in enumerate(row)]

        # Sort cells by their values in ascending order
        cells_with_positions.sort()

        # Track the maximum score assigned so far in each row
        max_score_per_row = [0] * m

        # Track the maximum score assigned so far in each column
        max_score_per_col = [0] * n

        # Initialize result grid with zeros
        result = [[0] * n for _ in range(m)]

        # Process cells in ascending order of their original values
        for _, row_idx, col_idx in cells_with_positions:
            # The score for current cell should be 1 more than the maximum score
            # in its row or column (whichever is larger) to maintain ordering
            result[row_idx][col_idx] = max(max_score_per_row[row_idx],
                                          max_score_per_col[col_idx]) + 1

            # Update the maximum score for this row and column
            max_score_per_row[row_idx] = result[row_idx][col_idx]
            max_score_per_col[col_idx] = result[row_idx][col_idx]

        return result
```

## Time and Space Complexity

**Time Complexity:** `O(m * n * log(m * n))`

- Creating the `nums` list with all grid elements takes `O(m * n)` time, where we iterate through all elements in the grid
- Sorting the `nums` list containing `m * n` elements takes `O(m * n * log(m * n))` time
- The final loop iterates through all `m * n` elements once, and each iteration performs constant time operations (comparisons, assignments), taking `O(m * n)` time
- Overall time complexity is dominated by the sorting step: `O(m * n * log(m * n))`

**Space Complexity:** `O(m * n)`

- The `nums` list stores all `m * n` grid elements as tuples, requiring `O(m * n)` space
- The `row_max` array uses `O(m)` space
- The `col_max` array uses `O(n)` space
- The `ans` matrix stores the result with `m * n` elements, requiring `O(m * n)` space
- Total space complexity: `O(m * n) + O(m) + O(n) + O(m * n) = O(m * n)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### Pitfall 1: Processing Elements in Wrong Order

**The Problem**: A common mistake is attempting to process the matrix row by row or column by column instead of processing all elements globally by their sorted values. This approach fails because it doesn't consider cross-dependencies between rows and columns.

**Incorrect Approach**:
```python
# WRONG: Processing row by row
for i in range(m):
    sorted_row = sorted([(grid[i][j], j) for j in range(n)])
    for rank, (_, j) in enumerate(sorted_row, 1):
        result[i][j] = rank
```

This fails because it doesn't maintain the relative ordering across columns. For example, if `grid[0][0] = 5` and `grid[1][0] = 3`, processing rows independently might assign `result[0][0] = 1` and `result[1][0] = 1`, violating the column ordering constraint.

**Solution**: Always sort all elements globally and process them in ascending order of their original values, considering both row and column constraints simultaneously.

### Pitfall 2: Incorrectly Calculating Minimum Valid Value

**The Problem**: Another mistake is using the wrong formula for calculating the minimum valid value. Some might try using the sum instead of maximum, or forget to add 1.

**Incorrect Approaches**:
```python
# WRONG: Using sum instead of max
result[i][j] = row_max[i] + col_max[j] + 1

# WRONG: Forgetting to add 1
result[i][j] = max(row_max[i], col_max[j])

# WRONG: Adding 1 only to one constraint
result[i][j] = max(row_max[i] + 1, col_max[j])
```

**Solution**: The correct formula is `max(row_max[i], col_max[j]) + 1`. This ensures the new value is strictly greater than all previously assigned values in both its row AND column.

### Pitfall 3: Not Updating Both Row and Column Maximums

**The Problem**: After assigning a value to `result[i][j]`, forgetting to update either `row_max[i]` or `col_max[j]` (or both) leads to incorrect assignments for subsequent elements.

**Incorrect Approach**:
```python
for _, i, j in sorted_cells:
    result[i][j] = max(row_max[i], col_max[j]) + 1
    row_max[i] = result[i][j]  # Forgot to update col_max[j]!
```

This causes future elements in the same column to potentially receive values that don't respect the ordering constraint.

**Solution**: Always update both `row_max[i]` and `col_max[j]` after each assignment:
```python
row_max[i] = col_max[j] = result[i][j]
```

### Pitfall 4: Using Original Values Instead of Assigned Values

**The Problem**: Attempting to use the original grid values directly or trying to maintain relative differences from the original matrix instead of just preserving the ordering.

**Incorrect Approach**:
```python
# WRONG: Trying to maintain proportional differences
diff = grid[i][j] - min_value
result[i][j] = base_value + diff
```

**Solution**: The algorithm only needs to preserve the relative ordering, not the actual differences. The greedy approach of assigning the smallest valid value at each step automatically minimizes the maximum value in the result.

